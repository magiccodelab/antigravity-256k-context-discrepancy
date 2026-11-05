# Volume 4: Community and Official Responses to the 256K Limit

- **Language**: **English** | [简体中文 (zh-cn/04-community-and-official-responses.md)](./zh-cn/04-community-and-official-responses.md)
- **Archive Navigation**: [Repository Overview](../README.md) | [Overview Report (dossier.md)](./dossier.md) | [Full Sources Archive (sources.md)](./sources.md) | [Static Forensic Report](../evidence/evidence-report.md)
- **Thematic Volumes**: [01-Incident Overview & Timeline](./01-incident-overview-and-timeline.md) | [02-256K Hardcoded Gates & Binary Teardown](./02-256k-hardcoded-gates-and-binary-teardown.md) | [03-agy-p Policy Reversal & Bans](./03-agy-p-headless-policy-reversal-and-bans.md) | **04-Community & Official Responses** | [05-Verbatim Quotes & Primary Records](./05-verbatim-quotes-and-evidence-vault.md)
- **Research ID (UUID)**: `df866cf1-da40-4b82-8084-34c6b0729544` | **Research Date**: 2026-11-04 (updated 2026-11-05) | **Scope**: How users, community developers, and Google staff responded to the 256K compaction threshold, and what those responses show about where the limit lives

---

## 1. Summary

Neither the desktop app nor the CLI offers a documented setting or menu option for the compaction threshold. Since CLI `v1.5.0` (2026-11-04), the binary again reads the undocumented `AGY_CLI_AUTO_COMPACT_OVERRIDE` environment variable, but in `v1.5.1` it only applies in a narrow case (see §2.4). Google staff confirmed on 2026-11-04 that the ~250K threshold "is a fixed default and isn't user-configurable yet"[^57]. Everything recorded in this volume is a response to that gap:

1. **A community binary patch removes the limit on the client.** The patch changes three locations in the local `agy` binary and nothing on the server. That it works is further evidence that the 256K ceiling is enforced by client code, as Volume 2 shows through disassembly[^1][^2].
2. **Official guidance treats the limit as something to work around.** Google staff recommend manual `/compact`, re-reading files after compaction, and keeping noise out of the context, without offering a way to raise the threshold[^43][^57].
3. **Google's own engineering team designed around compaction.** The ADK case study on the Google Developers Blog ran an 11-hour unattended session only after adding repository skills and `/goal` mode, so the objective survived repeated compaction[^16].
4. **Third parties built the visibility the product lacks.** Context monitors, a desktop `/context` panel, and context-reduction MCP servers appeared because the clients did not show or control context usage[^44][^49][^50][^51][^52][^53][^54].
5. **Cross-tool use is bounded by forum statements.** After the `agy -p` reversal, staff described which integration patterns remain allowed under consumer plans (see Volume 3 for the reversal itself)[^34][^36][^37][^38][^41].

---

## 2. The Community Binary Patch: Evidence That the Limit Is Client-Side

### 2.1 What `patch_agy.py` Changes
In GitHub Issue `#1245`, systems engineer `@LiraNuna` published **[`patch_agy.py`](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26)**, a patcher that works across `agy 1.3.5`, `1.4.x`, and `1.5.0` by resolving function locations from the binary's Go `pclntab` symbol table instead of fixed addresses[^1][^2]. It modifies the same three locations that Volume 2 identifies as the hardcoded gates. This archive records what each change does. It does not reproduce the replacement bytes, the script source, or the commands for running it.

| Change | Target function (`v1.5.0` file offset) | Original behavior | Effect of the change |
|---|---|---|---|
| 1 | `cli/model/model.SelectorModel.offersContextWindow` (`0x9e92140`) | Shows the context tier selector only for `"Gemini 4 Argon"` | Selector appears for every model |
| 2 | `cli/store/store.(*Store).ContextWindow` (`0x9c49fe0`) | Re-checks the same whitelist and drops the saved tier for other models | Saved `512K` / `900K` tier is kept for every model |
| 3 | `cortex/utils/utils.GetCheckpointConfig` (`0x7fe5802`) | `cmp $0x3e800, %edx; mov $0x3e800, %esi; cmovb %edx, %esi` clamps the threshold to `min(calculated, 256000)` | Threshold follows the computed `max_context - 40,000` |

All three changes are in client code. That the clamp can be removed this way agrees with Volume 2: the 256K ceiling is set by the client rather than reported by the server[^1][^2]. How the server handles requests above 256K after patching has not been verified in this archive (see [dossier.md §10](./dossier.md)).

### 2.2 Community Packaging and Third-Party Thresholds
- On 2026-11-02, developer `@AsabaLazy` announced in Issue `#1245` an Arch Linux AUR package, `antigravity-cli-fix-bin`, that tracks upstream releases with the patch applied[^1].
- Nous Research's Hermes Agent plugin documentation sets its own compaction threshold at `239,616` tokens (`256K - 16K` output margin) to stay below `agy`'s internal compaction[^48].

### 2.3 Costs and Risks Recorded by the Community
- **Quota cost**: `@Emily2040` noted in Issue `#1245` that at `512K` (`1.3x` multiplier) or `900K` (`1.8x`), a single uncached full-context resend after a prefix cache miss costs roughly **4x** the quota of the same event at `256K`[^4]. This is the most concrete cost argument for an early compaction default, and it supports making the threshold configurable rather than fixed.
- **Maintenance**: `agy` replaces its own binary on every auto-update, so any local modification is lost and must be reapplied[^1][^4].
- **Account risk**: Modifying the official client may conflict with the Antigravity Terms of Service. This archive does not recommend it.

### 2.4 The Environment Variable Restored in CLI `v1.5.0`–`v1.5.1`
`AGY_CLI_AUTO_COMPACT_OVERRIDE` had been compiled out in `v1.3.5`–`v1.4.x`. CLI `v1.5.0` reads it again, and `CHANGELOG.md` for `v1.5.0` and `v1.5.1` does not mention it[^1][^11]. Static analysis of the official `v1.5.1` binary (SHA-256 `72738cf0…26669`) shows how narrow it is:

- **Absolute token counts only**: The parser accepts either a plain token count or a percentage such as `80%`, but the percentage form needs a positive base value. Both call sites in `v1.5.1` pass a base of `0` (`xor %eax,%eax` before the call at VA `0x9c34c9a` and VA `0x9e875aa`), so percentage values are ignored and one value cannot scale across models.
- **Whitelisted models only, lowest priority**: In `Store.ContextWindow()` the model whitelist is checked first, then the tier saved in `settings.json`, and only then the environment variable. Non-Argon models never reach it, and a saved tier takes precedence over it.
- **Interaction with the clamp is unverified**: The `min(calculated, 256000)` clamp in `GetCheckpointConfig` is still present. Whether a value above `256,000` survives that clamp has not been tested in this archive; a long session that crosses 256K, read back from the `max_context_tokens` field in the session database, would settle it.

In short, `v1.5.0` restored an undocumented variable that, in `v1.5.1`, works at most for Gemini 4 Argon with a fixed token count. It is neither a user-facing setting nor a confirmed way past the 256K clamp.

---

## 3. Official Guidance From Google Staff

| Date | Thread | Staff | Guidance |
|---|---|---|---|
| 2026-09-25 | `#184694` | `@chunduriv` | For context visibility, use `/context` in the CLI; the desktop app had no equivalent panel[^44] |
| 2026-10-20 | `#189476` | `@chunduriv` | Add a standing rule (in `GEMINI.md` or `.agents/rules/`) telling the agent to re-read target files after `# Resuming from a compaction`, and run `/compact` with instructions to preserve exact identifiers and file paths[^43] |
| 2026-11-04 | `#198191` | `@chunduriv` | The ~250K threshold is a fixed default and not user-configurable; run `/compact` at a natural break point, keep reference material in workspace files, and exclude build artifacts and dependencies. The request for a selectable size was passed to engineering, with no timeline[^57] |

All three replies accept the threshold as given and move the burden of avoiding information loss to the user. None of them addresses the advertised 1M window.

---

## 4. Google's Own Case Study: Designing Around Compaction

In October 2026, the Google Developers Blog published *"[Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/)"*, describing two runs against `google/adk-python` (about 205,000 lines, 800 source files, and 16,892 unit tests)[^16]:

| | Round 1 (no repository guidance) | Round 2 (`.agents/skills` + `/goal`) |
|---|---|---|
| Runtime | Stopped after about 2 hours | Ran 11 hours unattended (9:11 PM to 8:19 AM) |
| Before editing | Wrote a benchmark at minute 7, edited code from minute 70 | Read 12 skill files on demand, finished baselines by minute 10 |
| Public API | Broke the public `AuthConfig` schema | Kept `AuthConfig` intact |
| Tests | Ran 216 nearby tests | Added regression tests and ran all 16,892 |

The post attributes the long run to `/goal` mode keeping the objective in the harness, so it is not lost when the conversation is compacted. Google's own team therefore treated compaction as a constraint to engineer around, not as a non-issue[^16].

---

## 5. Third-Party Tools Built to Fill the Gap

| Tool | What it adds | Source |
|---|---|---|
| `weby-homelab/antigravity-cli-statusline` | Live context usage, model, and 5-hour / 7-day quota in the CLI status line | [^52] |
| `dimitr-io/antigravity-desktop-context` | A `/context`-style side panel for the Antigravity 2.0 desktop app, which had none | [^44][^51] |
| `AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor` | Context watermarks across sessions in the Antigravity IDE | [^50] |
| OTLP export hook (`@UesugiErii`, Issue `#366`) | Per-turn token counters exported to OpenTelemetry for cost auditing | [^7] |
| `mksglu/context-mode` | Indexes large tool output in a local sandbox and returns only matching snippets; reports 70%–90% less context bloat | [^49] |
| `oraios/serena` | LSP symbol-level lookup in place of full-file reads | [^53] |
| `xdotech/goatlas` | AST and call-graph queries in place of repository-wide reads | [^54] |

These tools share one purpose: keeping sessions below a threshold that users cannot see or change in the product.

---

## 6. Cross-Tool Integration Boundaries Stated by Staff

After the `agy -p` reversal (Volume 3), staff replies describe the following patterns as permitted under consumer plans. They are recorded here as policy statements, not as recommendations:

1. **ACP clients**: Signing in with a personal subscription in clients that support the Agent Client Protocol, using Google's own ACP server (built into `agy` from `v1.5.0`), is supported[^11][^34][^36][^41].
2. **API-key mode**: When `agy` authenticates with a Gemini API key, no consumer account session exists, so the consumer-plan restriction on third-party orchestration does not apply[^36][^38].
3. **Human-triggered runs**: Files drafted with another AI assistant do not make a user's own deterministic script a prohibited wrapper, provided the assistant never invokes `agy` and every run is started by the account owner[^37].

ACP over `stdio` and `agy -p` over `stdio` are technically the same kind of local subprocess exchange, yet the first is allowed and the second is not. The overview report lists this as an open contradiction (see [dossier.md §7](./dossier.md)).

---

## Reference Sources Cited in This Volume (Footnote Definitions)

[^1]: [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) — GitHub `google-antigravity/antigravity-cli` (Author: `Burntgogi`; Commenters: `NuCl34R`, `Circumsized`, `Emily2040`, `LiraNuna`, `AsabaLazy`), Published: 2026-10-14 to 2026-11-02 (Accessed: 2026-11-04 | Type: Primary Official Repository Issue & Binary Reverse-Engineering Proof | Confidence: High)
[^2]: [Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) — GitHub Gist (Author: `LiraNuna`), Published: 2026-10-23 (Accessed: 2026-11-04 | Type: Primary Reverse-Engineering Patch Source Code | Confidence: High)
[^4]: [[Bug]: Five measured places the CLI spends time and tokens the work does not need: serial tool calls, uncached full-context calls and 70 s compactions, 12 s headless overhead, summary store rewritten every 1.5 s, build-time CPU instruction choice (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) — GitHub `google-antigravity/antigravity-cli` (Author: `Emily2040`), Published: 2026-10-19 to 2026-10-25 (Accessed: 2026-11-04 | Type: Primary Official Repository Issue & Benchmark Audit | Confidence: High)
[^7]: [Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue #366)](https://github.com/google-antigravity/antigravity-cli/issues/366) — GitHub `google-antigravity/antigravity-cli` (Author: `UesugiErii` et al.), Published: 2026-06-11 to 2026-10-20 (Accessed: 2026-11-04 | Type: Primary Official Repository Issue | Confidence: High)
[^11]: [Google Antigravity CLI Releases (`v1.4.2` – `v1.5.0`)](https://github.com/google-antigravity/antigravity-cli/releases) — GitHub `google-antigravity/antigravity-cli`, Published: 2026-10-18 to 2026-11-04 (Accessed: 2026-11-04 | Type: Primary Official Release Notes | Confidence: High)
[^16]: [Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/) — Google Developers Blog, Published: 2026-10 (Accessed: 2026-11-04 | Type: Primary Official Engineering Blog | Confidence: High)
[^34]: [Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241) — Google AI Developers Forum (Official Reply: `@chunduriv`), Published: 2026-10-07 to 2026-10-08 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Confidence: High)
[^36]: [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689) — Google AI Developers Forum (Official Reply: `@Rebbanpalli_Naveen`), Published: 2026-10-23 to 2026-10-27 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Confidence: High)
[^37]: [Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998)](https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998) — Google AI Developers Forum (Official Reply: `@Rebbanpalli_Naveen`), Published: 2026-10-26 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Confidence: High)
[^38]: [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815) — Google AI Developers Forum (Official Reply: `@Ambati_Rajendra`), Published: 2026-10-19 to 2026-10-28 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Confidence: High)
[^41]: [Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131)](https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131) — Google AI Developers Forum (Official Reply: `@Rebbanpalli_Naveen`), Published: 2026-10-31 to 2026-11-02 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Confidence: High)
[^43]: [Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476) — Google AI Developers Forum (Author: `@Hello_there`; Official Reply: `@chunduriv`), Published: 2026-10-15 to 2026-10-20 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Confidence: High)
[^44]: [Feature request: Show context window usage in Antigravity Desktop (#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694) — Google AI Developers Forum (Author: `@Soro`; Official Reply: `@chunduriv`; Extension: `@dimitr-io`), Published: 2026-09-24 to 2026-10-22 (Accessed: 2026-11-04 | Type: Primary Official Forum Reply | Confidence: High)
[^48]: [Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (`239,616` Compaction Threshold)](https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk) — Nous Research Hermes Agent Documentation, Published: 2026-09-23 (Accessed: 2026-11-04 | Type: Technical Documentation | Confidence: High)
[^49]: [mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management](https://github.com/mksglu/context-mode) — GitHub (Author: `mksglu`), Published: 2026 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Confidence: High)
[^50]: [AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE](https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor) — GitHub (Author: `AGI-is-going-to-arrive`), Published: 2026 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Confidence: High)
[^51]: [dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context) — GitHub (Author: `dimitr-io`), Published: 2026-10-22 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Confidence: High)
[^52]: [weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI](https://github.com/weby-homelab/antigravity-cli-statusline) — GitHub (Author: `weby-homelab`), Published: 2026 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Confidence: High)
[^53]: [oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit ("The IDE for Your Coding Agent")](https://github.com/oraios/serena) — GitHub (Author: Oraios AI), Published: 2026 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Confidence: High)
[^54]: [xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server)](https://github.com/xdotech/goatlas) — GitHub (Author: `xdotech`), Published: 2026 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Confidence: High)
[^57]: [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) — Google AI Developers Forum (Author: `@Sharper`; Official Reply: `@chunduriv`), Published: 2026-11-04 (Accessed: 2026-11-05 | Type: Primary Official Staff Forum Reply | Confidence: High)
