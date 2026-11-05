# Volume 2: 1M Context Window Discrepancy, Three Binary Hardcoded Gates & v2.29.0 / CLI 1.5.0–1.5.1 Disassembly

- **Language**: **English** | [简体中文 (zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md)](./zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md)
- **Archive Navigation**: [Repository Overview](../README.md) | [Overview Report (dossier.md)](./dossier.md) | [Full Sources Archive (sources.md)](./sources.md) | [Static Forensic Report](../evidence/evidence-report.md)
- **Thematic Volumes**: [01-Incident Overview & Timeline](./01-incident-overview-and-timeline.md) | **02-256K Hardcoded Gates & Binary Teardown** | [03-agy-p Policy Reversal & Bans](./03-agy-p-headless-policy-reversal-and-bans.md) | [04-Community & Official Responses](./04-community-and-official-responses.md) | [05-Verbatim Quotes & Primary Records](./05-verbatim-quotes-and-evidence-vault.md)
- **Research ID (UUID)**: `df866cf1-da40-4b82-8084-34c6b0729544` | **Research Date**: 2026-11-04 (Updated 2026-11-05) | **Confidence Calibration**: Verified against GitHub Issues `#1245`/`#878`/`#1291`, SQLite session database metadata, and `agy` v1.3.5–v1.5.1 ELF disassembly

---

## 1. Empirical Proof: Advertised 1M (1,048,576) Context vs. Forced ~256K Compaction in Production

### 1.1 Official Advertised Capacity and `/context` Panel Display
Across Google Antigravity's official documentation, model release announcements (Changelog)[^12], model cards, and the interactive `/context` panel of the `agy` CLI, both the flagship reasoning model `Gemini 4 Argon` and the workhorse model `Gemini 3.8 Flash` explicitly display a context window capacity of **`1,048,576` Tokens (1.0M)**[^1][^3][^12][^44]. When a session sits around 220K Tokens, the CLI `/context` panel tells the user that context utilization is a mere `21.2%` (`222.4k/1.0M tokens`) and indicates that remaining available capacity is `Free space: 826.2k (78.8%)`[^44][^51]. (Confidence: High)

### 1.2 Five Independent Large-Scale Production Session Telemetry Datasets
Despite the UI displaying more than 75% free context capacity, five independent production session telemetry datasets collected across Linux and Windows environments consistently confirm that every session is forcibly truncated or compacted around **`~240K–276K` Tokens** (centered on the `256,000` constant):

1. **Eddy Murias (`@murias002`) Full-Database Telemetry Across 310 Sessions and 65,613 Model Calls (Issue `#878`)**[^3]:
   - **Sample Scope**: Every `gen_metadata` record across `~/.gemini/antigravity-cli/conversations/*.db` on Linux between August 30 and October 4, 2026, totaling **310 distinct sessions and 65,613 model calls** (86.6% running on `Gemini 3.8 Flash`)[^3].
   - **Input Token Distribution**: Median (`p50`) of `136,487` Tokens, `p90` of `229,000` Tokens, an **all-time maximum single-call input hard-capped at `255,999` Tokens, and `0` model calls exceeding `300,000` input tokens**[^3].
   - **Compaction Trigger Water-Level**: Across all **276 compaction events** (identified as >40% single-step token drops), the median pre-compaction context size on the immediately preceding step was **`255,070` Tokens** (`p10` = `247,360`, `p90` = `255,874`), dropping to a post-compaction median of **`45,304` Tokens**[^3].
2. **`@Burntgogi` Trajectory Measurements of a 94-Turn Continuous `Gemini 4 Argon` Refactoring Session (Issue `#1245`)**[^1]:
   - **Sample Scope**: A 50-minute continuous multi-stage TypeScript refactoring session on `agy 1.3.5` under Windows 10 x64 (94 `PLANNER_RESPONSE` turns, 264 steps, accumulating `10.38M` total prompt tokens and `9.43M` cached read tokens)[^1]:

| Event Node | Step / Turn | Pre-Compaction Prompt Tokens (Cached + New Input) | Post-Compaction Prompt Tokens | Post-Compaction `cache_read_tokens` | Trigger Cause & Impact |
|---|---|---:|---:|---:|---|
| **Checkpoint 1** | Step 79 (Turn 21) | `276,110` (`263,524` cached + `12,586` in) | `17,336` | `0` | Crossed ~256K threshold, triggering first forced compaction and wiping cache to zero[^1] |
| **Mid-run Prefix Miss** | Step 193 (Turn 67) | `239,857` (`237,239` cached + `2,618` in) | `243,064` | `0` | Full prefix cache invalidation just 25 seconds after Step 191 (243K billed at full uncached rate)[^1] |
| **Checkpoint 2** | Step 200 (Turn 70) | `249,867` (`247,369` cached + `2,498` in) | `15,363` | `0` | Reached ~250K, triggering second forced compaction and erasing 93.8% of context[^1] |

3. **`@NuCl34R` Controlled Experiment Proving the Environment Variable Override Is Non-Functional (Issue `#1245` Comment #1)**[^1]:
   - Running `agy 1.3.5` (`gemini-4-argon`) on Linux x64, whether no environment variable was set (compacting at `269,597` and `274,270` Tokens) or `AGY_CLI_AUTO_COMPACT_OVERRIDE=900000` was explicitly exported (still compacting at `272,784` and `279,687` Tokens), the compaction trigger point showed zero change[^1].
4. **`@Emily2040` Precision Audit Across 15 Headless Run Logs and 1,279 `Gemini 4 Argon High` Calls (Issue `#1245` Comments #3–#4 & Issue `#1291`)**[^1][^4]:
   - Across 1,279 model calls on `agy 1.4.0–1.4.2`, **22 `checkpoint` auto-compaction steps** were triggered; on the step immediately preceding every single compaction, the active context (`input + cache_read`) fell strictly between **`235K–268K` Tokens**, with zero exceptions exceeding that band[^1][^4].
5. **Local Pre-Patch Antigravity CLI Database Audit Across 209 Sessions and 15,575 1M-Gemini Calls (`< 2026-11-04 00:00:00 UTC+8`, [Evidence Report §2.5](../evidence/evidence-report.md))**:
   - **Sample Scope**: All 237 local CLI SQLite databases (`~/.gemini/antigravity-cli/conversations/*.db`) modified prior to `2026-11-04 00:00:00 UTC+8` (`2026-11-03 16:00:00 UTC`, before any local binary patching), containing 209 active 1M-context Gemini sessions and **15,575 model calls** (`11,110` on `Gemini 3.8 Flash`, `2,826` on `Gemini 4 Argon`, and `1,639` on `Gemini 3.7/3.6/3.5 Flash`).
   - **Results**: **`100.0%` of calls (`15,575 / 15,575`) recorded `max_context_tokens = 256,000`**, **`0` single-attempt calls (`retries = 0`) exceeded `300,000` prompt tokens** (`p50 = 108,493`, `p90 = 211,227`, `p99 = 250,610`), and the databases recorded **79 `CORTEX_STEP_TYPE_CHECKPOINT` (`step_type = 23`) compaction steps** and **52 `truncation_reason = 2` (`TRUNCATION_REASON_MAX_TOKEN_LIMIT`) events**.

---

## 2. The Three Binary Hardcoded Gates & Environment Variable Evolution (ELF & Assembly-Level Proof)

Through reverse-engineering of compiled `agy` client releases (64-bit Go ELF binaries from `v1.3.5` through `v1.5.0` and `v1.5.1`) and Desktop `v2.27.0–v2.29.0` ASAR/binary bundles by community researchers (`@LiraNuna`, `@Circumsized`, `u/Lberondroth`), combined with our direct disassembly verification of the official unmodified `agy` `v1.5.0` (BuildID `5fb6af4f2e1d489f3ada0fd785060f48`) and `agy` `v1.5.1` (`2026-11-05`, BuildID `696031ae34e5e5e26a9d7c436e85317f`) binaries, we have reconstructed the four-layer low-level mechanism Google uses to restrict context windows:

### 2.1 Foundational Forensics: Antigravity CLI SQLite Schema, `max_context_tokens = 256000`, & Two-Stage Evolution of `AGY_CLI_AUTO_COMPACT_OVERRIDE`
1. **Antigravity CLI SQLite Schema & Persistent `ContextWindowMetadata` Proof (`@Circumsized` & Local 15,575-Call Audit)**[^1]:
   Antigravity CLI persists each conversation in `~/.gemini/antigravity-cli/conversations/<uuid>.db` (indexed globally by `conversation_summaries.db` and its FTS5 virtual table `conversations_fts`). Each session `.db` contains 7 tables: `trajectory_meta`, `steps` (ordered action rows where `step_type = 15` is `PLANNER_RESPONSE` and `step_type = 23` is `CORTEX_STEP_TYPE_CHECKPOINT`), `gen_metadata` (serialized `exa.cortex_pb.GeneratorMetadata` Protobuf blobs per LLM call), `executor_metadata`, `parent_references`, `trajectory_metadata_blob`, and `battle_mode_infos`.
   Inside `gen_metadata.data`, the embedded Protobuf hierarchy follows:
   - `GeneratorMetadata.chat_model` (`field 1`, `ChatModelMetadata`) → `.usage` (`field 4`, `ModelUsageStats`: `field 2 = input_tokens`, `field 3 = output_tokens`, `field 4 = cache_write_tokens`, `field 5 = cache_read_tokens`, `field 9 = thinking_output_tokens`)
   - `GeneratorMetadata.chat_model` (`field 1`) → `.chat_start_metadata` (`field 9`, `ChatStartMetadata`: `field 1 = start_step_index`, `field 2 = checkpoint_index`, `field 4 = created_at`) → `.context_window_metadata` (`field 10`, `ContextWindowMetadata`):
     - `field 1`: `estimated_tokens_used` (`int32`)
     - `field 2`: `truncation_reason` (`TruncationReason` enum: `0 = UNSPECIFIED`, `1 = PROMPT_CACHE_TTL_EXPIRED`, `2 = MAX_TOKEN_LIMIT`)
     - `field 3`: `token_breakdown` (`TokenBreakdown`)
     - `field 4`: `max_context_tokens` (`optional int32`)
   In both `@Circumsized`'s Issue `#1245` inspection and our read-only audit of all 15,575 pre-patch 1M Gemini calls across 209 local CLI sessions (see [Evidence Report §2.5](../evidence/evidence-report.md)), every 1M Gemini call records:
   ```text
   max_context_tokens = 256000
   truncation_reason = 2  (TRUNCATION_REASON_MAX_TOKEN_LIMIT, set upon crossing the 256,000 boundary)
   ```
   As `@Circumsized` noted, `256,000` is almost exactly one-quarter (~25%) of the advertised `1,048,576` window—proving this is neither a server-side rejection nor a physical model limit, but a hard client-side policy constant written into request metadata prior to dispatch[^1].
2. **Two-Stage Evolution of `AGY_CLI_AUTO_COMPACT_OVERRIDE`: Compile-Time Erasure in `v1.3.5–v1.4.x` (`@LiraNuna`) vs. Silent Restoration in `v1.5.0–v1.5.1`**[^1][^2]:
   Inside `google3/third_party/jetski/cli/store/store.AutoCompactOverride`, the source code was designed to parse either a percentage (such as `80%`) or an integer token threshold (with the literal `"overridden by AGY_CLI_AUTO_COMPACT_OVERRIDE="` in the string table).
   - **Stage 1 (`v1.3.5`–`v1.4.x`, October 2026)**: In official release builds disassembled by `@LiraNuna` in Issue `#1245`, the `os.Getenv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` lookup was replaced at compile time with register-zeroing instructions (an empty string `""`):
     ```x86asm
     xor %eax, %eax                ; Set string pointer to 0 ("")
     xor %ebx, %ebx                ; Set string length to 0
     call strings.TrimSpace        ; Call TrimSpace on the empty string
     ```
     Because the subsequent `strconv.ParseInt("", 10, ...)` call on an empty string always failed, `AutoCompactOverride()` unconditionally returned `""`, rendering any user-defined `AGY_CLI_AUTO_COMPACT_OVERRIDE` completely inert[^1][^2].
   - **Stage 2 (`v1.5.0`–`v1.5.1`, November 4–5, 2026)**: Direct disassembly of both `v1.5.0` (VA `0x9c49d00`) and `v1.5.1` proves that Google **silently restored** the environment variable lookup (`lea rax, [rip+→"AGY_CLI_AUTO_COMPACT_OVERRIDE"]` + `mov ebx, 0x1d` + `call os.LookupEnv` + `call strings.TrimSpace`, routed to the exact same `os.LookupEnv` target used by `GEMINI_API_KEY` and `HTTPS_PROXY`) along with its downstream percentage/integer parser (`VA 0x9c49d40`), **without mentioning this change anywhere in `CHANGELOG.md`** (see [Forensic Report §4.3](../evidence/evidence-report.md)). However, the underlying `0x3E800` (`256,000`) clamp inside `GetCheckpointConfig` (Gate 3 below) was left completely untouched. In `v1.5.1`, both call sites of the parser pass a base of `0`, so the percentage form is inert, and `Store.ContextWindow()` consults the variable only after the model whitelist and the tier saved in `settings.json`; whether a value above `256,000` survives the clamp is unverified (see [Volume 4 §2.4](./04-community-and-official-responses.md)).

### 2.2 Gate 1: The UI/TUI Model Selector Whitelist (`SelectorModel.offersContextWindow`)
In the `agy` CLI's `/model` menu code, the interactive logic for cycling through four context window tiers via the `Tab` key is already fully implemented:
`Context: Default (256K) | 256K | [512K] | 900K`[^1].
However, the function controlling whether that selector row is rendered—`google3/third_party/jetski/cli/model/model.SelectorModel.offersContextWindow` (located at file offset `0x9e92140` in the `v1.5.0` binary)—hardcodes a global whitelist slice check (pointing to memory address `0xc592be0`):
- In official release binaries, that whitelist slice **contains only a single model string: `"Gemini 4 Argon"`**[^1].
- For every other model that physically supports a 1M context window—including `Gemini 3.8 Flash` and `Gemini 3.1 Pro`—`offersContextWindow` immediately returns `false` (`xor %eax, %eax`), hiding the `Context` selector row in the TUI and ignoring `Tab` key events[^1].
- Meanwhile, in the Antigravity 2.0 Desktop App and IDE (as of `v2.28.2` / `v2.29.0`), even though the `ContextWindowOptions` structure is compiled into the binary bundle, the frontend UI selector remains hidden behind an unflipped feature gate[^18][^19].

- **Unlocked UI Proof (Posted by `@LiraNuna` in [GitHub Issue #1245](https://github.com/google-antigravity/antigravity-cli/issues/1245), [View Original Screenshot](https://github.com/user-attachments/assets/dda2af37-98a9-4d89-b78e-8cae8c6361cc))**: After patching `SelectorModel.offersContextWindow` to return `true`, the `Switch Model` (`/model`) terminal menu in the `agy` CLI exposes the previously hidden `Context` selector row (`Default`, `256K`, `[512K]`, `900K`) beneath the model list (`Gemini 4 Argon`, `> Gemini 3.8 Flash (current)`, `Claude Opus 5.5`, `Claude Sonnet 5.5`, `GPT-OSS 120B (Medium)`) and the `Effort` reasoning slider, displaying a prominent red warning on the right when `[512K]` is selected: **`about 1.3x quota per turn`**[^1].

### 2.3 Gate 2: Duplicate Whitelist Discard During Config Read & Backend Sync (`store.(*Store).ContextWindow()`)
Even if a user bypasses Gate 1 via binary patching—or manually edits `~/.gemini/antigravity-cli/settings.json` to set `"contextWindow": "512K"` or `"900K"`—the setting still has zero effect on non-Argon models[^1].
Here is why: although the client contains a `store.(*Store).ContextWindowChoice()` function (`0x917d800` in `v1.3.5`, `0x9c49fa0` in `v1.5.0`) that reads and normalizes the setting directly from `settings.json`, the `Manager.syncContextWindow()` method responsible for synchronizing configuration with the backend instead calls a wrapper function: **`google3/third_party/jetski/cli/store/store.(*Store).ContextWindow()`** (`0x917d892` in `v1.3.5`, `0x9c49fe0` in `v1.5.0`)[^1][^2].
That wrapper function contains a **second, identical `"Gemini 4 Argon"` whitelist loop check**:
```x86asm
; v1.3.5 Disassembly (@LiraNuna) / v1.5.0 corresponding offset 0x9c4a032:
917d892: mov  0x279b727(%rip), %rdx   ; Load whitelist slice ("Gemini 4 Argon")
...
917d913: jl   0x917d952               ; If current model is not in whitelist, jump straight to exit!
917d952: xor  %eax, %eax              ; Return empty string pointer ""
917d954: xor  %ebx, %ebx              ; Return length 0
917d956: leave
917d957: ret                          ; Return ("", 0) without ever reading settings.json!
```
Once `("", 0)` is returned, `Manager.syncContextWindow()` invokes `backend.SetContextWindow("")`, ensuring the backend never receives the `512K` or `900K` value configured in `settings.json` and forcibly falls back to the 256K default[^1].

### 2.4 Gate 3: The `0x3e800` (256,000) Assembly Clamp in Cortex Engine's `GetCheckpointConfig`
The third gate operates inside the core compaction calculator: even when a user runs `Gemini 4 Argon`—the sole model on the whitelist—and even after the backend receives a `512K` or `900K` window setting, sessions still compact around `~250K–256K`[^1].
Reverse-engineering reveals that **`google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig`** (`0x7861894` in `v1.3.5`, file offset `0x7fe57fe` in `v1.5.0`, and file offset `0x7fb0fde` in `v1.5.1`)—the core Cortex engine function that computes auto-compaction checkpoint configurations for multi-turn conversational cascades—first calculates the true usable threshold (physical context window minus a `40,000` safety margin, e.g., ~`960,000`–`983,000` on a 1M window) and stores it in register `%edx`. Immediately afterward, it executes a 14-byte hardcoded constant clamp (machine code `81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2`)[^1][^2]:

```x86asm
; Disassembled from agy v1.3.5 (0x7861894), agy v1.5.0 (0x7fe57fe), and agy v1.5.1 (0x7fb0fde):
7fe57fe:  8b 54 24 4c           mov    0x4c(%rsp), %edx      ; Load calculated usable threshold (context - 40,000 margin)
7fe5802:  81 fa 00 e8 03 00     cmp    $0x3e800, %edx        ; Compare against hex 0x3e800 (decimal 256,000)! [v1.5.1 @ 0x7fb0fe2]
7fe5808:  be 00 e8 03 00        mov    $0x3e800, %esi        ; Load 256,000 into %esi
7fe580d:  0f 42 f2              cmovb  %edx, %esi            ; Unsigned conditional move: esi = min(calculated, 256000)
7fe5810:  89 74 24 50           mov    %esi, 0x50(%rsp)      ; Write the clamped 256,000 into TokenThreshold!
```

These 5 assembly instructions mean that **regardless of how the UI, configuration files, or model whitelist are set, whenever the calculated compaction threshold exceeds `256,000` (`0x3e800`), the `cmovb` instruction unconditionally discards the calculated value and writes `256,000` into `TokenThreshold`**[^1][^2]. This is the exact line-of-code root cause explaining why every paying `Gemini 4 Argon` 1M subscriber worldwide sees their sessions collapse into compaction around ~250K–276K[^1].

### 2.5 Crucial Architectural Distinction: Lossy "Compaction" vs. Silent "Pre-flight Truncation"
In GitHub Issue `#1245` Comment #2, reverse-engineering researcher `@Circumsized` highlighted a widely overlooked dual-mechanism distinction based on the database field `truncation_reason = 2`[^1]:
1. **Lossy Auto-Compaction (Compaction)**: When a conversation crosses `TokenThreshold`, the system invokes a background summarization model to replace the prior ~250K Tokens of verbatim history with a 15K–17K Token prose summary, explicitly injecting the marker `CHECKPOINT: "# Resuming from a compaction..."`[^1].
2. **Silent Pre-flight Truncation (Pre-flight Truncation)**: As a conversation approaches `max_context_tokens = 256000`, the client also slices off older conversation turns directly from the front of the request payload prior to sending it to the server (recorded on the subsequent turn as `truncation_reason = 2`). **This process generates no summary and prints no `CHECKPOINT` marker—earlier history simply vanishes silently from the request**[^1].
Consequently, merely "disabling automatic summary generation" without raising the underlying `256,000` constant cap would only replace a *visible lossy summary* with *silent, unmarked history deletion*—an even more insidious hazard[^1].

---

## 3. Engineering Damage & Financial/Quota Black Holes Caused by Premature 256K Compaction and Cache Invalidation

### 3.1 Engineering Damage: Loss of Exact Code Symbols, Hallucinated Identifiers, and Cascading Amnesia
1. **Cross-File Constant Corruption and 200x Scale Distortion (`@Burntgogi` Production Case)**:
   In the 94-turn refactoring session documented in `#1245`, the agent read catalog and state-initialization files via `view_file` during Steps 1–75; when the first compaction fired at Step 79, all verbatim `view_file` outputs were discarded and replaced with a prose summary. Later, when writing cross-module upgrade mappings during Steps 159–177, the model did not re-read the source files; instead, working from memory, it hallucinated a non-existent `tier-3` catalog key and mistook state fields initialized on a `1000..2000` scale for a normalized `0..10` scale—**erroneously shrinking live business values by 200x**. After the second compaction at Step 200, the model used a hyphenated `localStorage` key name in the UI module and an underscored key name in the entry module, breaking module integration completely[^1].
2. **Fabricating 6 Non-Existent Identifiers Out of Thin Air (`@Hello_there` Forum `#189476` Case)**:
   After a session hit `Resuming from a compaction — you have lost access to the full conversation history`, the framework failed to enforce any post-compaction file re-verification gate, causing the model to **fabricate 6 non-existent identifiers out of thin air** in subsequent iterations and overwrite previously established initialization hard limits with bogus hardcoded values[^43].
3. **Complete Multi-Compaction Segment Amnesia (`@jlukic` Issue `#878` Case)**:
   When a long session undergoes multiple compactions, each subsequent compaction summarizes only the already-condensed ~100K context from the immediately preceding segment—causing Segment 3 of a session to lose 100% of its memory of Segment 1 unless the user manually forces the agent to read its own local session transcript files on disk[^3].
4. **Unverified Task Completion Claims and Authentication Bypass Bugs (`@Pitt_zhang` Forum `#176775` Audit Case)**:
   In Google AI Developers Forum thread [`#176775`](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775)[^46], an independent code audit of Antigravity's (AGY) delivery across a multi-step frontend and backend engineering project documented three classes of amnesia and verification failures: (1) **False Frontend Completion Claims** (the claimed backup commit `104dc9c` modified only a chat diary without touching frontend source files, and an independent TypeScript check failed); (2) **Broken Frontend-Backend API Contracts** (the AGY frontend called `/portal-api/**` and `/mock-channel/**` while the backend only exposed `/api/**`, with mismatched DTO fields and status enums); and (3) **Critical Authentication Bypass Vulnerability** (when a login API request failed, the frontend fell back to a local mock-admin session, allowing any attacker to gain administrator privileges simply by inducing an API failure)[^46].

### 3.2 Prompt Cache Destruction and 70-Second Black-Box Compactions: Why the 256K Wall Drains User Quota Faster
According to `@Emily2040`'s quantitative audit of 1,279 `Gemini 4 Argon High` calls (`10,619,129` uncached input tokens and `192,374,637` cached read tokens) in GitHub Issues `#1291` and `#1245`, Antigravity's context management harbors two hidden quota-draining black holes[^1][^4]:
1. **70-Second Median Freeze per Auto-Compaction, Unreported Summarization Quota Deduction, and Total Prompt Cache Wipe**:
   - `Gemini 4 Argon` and newer models **do not support explicit cache handles (Explicit Caching)**; they rely entirely on byte-identical prefix matching via Implicit Caching (which normally achieves a 95%–99% hit rate, where cached reads cost only **1/20th** of uncached input—i.e., `$0.30/1M` vs. `$6.00/1M`)[^1][^4][^42].
   - Across the **22 measured `checkpoint` auto-compactions**, each compaction froze execution for `35 seconds` to `602 seconds` (10 minutes), with a **median pause of `70 seconds`**[^4].
   - During those 70 seconds, the background summarization model call **reports `"none"` for token usage in the client trajectory metadata**—so users see zero token increase in local logs while their backend quota pool is silently deducted[^4][^42].
   - Worse still, once compaction finishes, the previously accumulated 240K–268K cached prefix is completely rewritten and invalidated (`cache_read_tokens → 0`). The very next turn is forced to send **`44,990–70,041` Tokens of uncached input at full price** to rebuild the cache from scratch, and because the model lost exact code symbols during compaction, it immediately invokes file-reading tools to re-ingest large files—creating a vicious quota-burning loop: *"compaction wipes cache → full-price prefix resend → re-reading files refills 256K → another compaction"*[^1][^4][^5].
2. **Mid-Run Full Prefix Cache Misses Triggered by `system_message` Injection**:
   - Out of 1,279 calls, just **17 calls** with `input > 100K` and `cache_read_tokens = 0` (each sending `113,735` to `268,032` uncached tokens—representing a mere **1.3%** of all calls) consumed **`3,292,077` Tokens, or `31.0%` of the entire run's uncached input token budget**[^4]!
   - Aside from 3 cold starts after `agy -c` session resumes, the remaining **14 occurred mid-run**, and **15 of the 17 occurred within two steps of the client injecting a `system_message` step** (such as a subagent completion notification or timer wakeup), or after a long-running shell command (551–1,190 seconds) allowed the implicit cache TTL to expire[^4].

### 3.3 `@Yiji_Kong` Controlled A/B Benchmarks: `Gemini 4 Argon` Secretly Drains Quota at 3.0x Its Published API Price (and 7.64x–9.25x on Web Pure Input)
On October 28–29, 2026, developer `@Yiji_Kong` published two sets of strictly controlled A/B experiments across Antigravity (`v2.27.0`, Ultra 5x subscription) and Gemini Web in official forum thread **#196351**, uncovering the true internal billing weights behind rapid subscription quota exhaustion[^42]:

#### Experiment 1: In-Client Antigravity Same-Bucket, Single-Turn, Zero-Cache A/B Comparison
By polling `quotaInfo.remainingFraction` from the local `LanguageServerService/GetUserStatus` endpoint every 30 seconds and reading exact token counts from `GetCascadeTrajectoryGeneratorMetadata` after an 8-minute idle window, identical single-step tool-free writing prompts (with zero cached reads) were sent to `Gemini 4 Argon (High)` and `Gemini 3.8 Flash (High)`[^42]:

| Tested Model | Input / Output Tokens | 5-Hourly Pool Quota Drop (`Quota drop`) | Implied Cost at Published API List Price | Quota Consumed per $1 of Published API Cost (`Quota per USD`) | Normalized Multiplier vs. Flash |
|---|---|---:|---:|---:|---:|
| **Gemini 4 Argon (High)** | `14,918` in / `4,233` out | `0.00144340` | `$0.0722 USD` (at `$2 / $10` per 1M) | **`0.02000`** | **`3.00x`** (`0.0200 / 0.00667`)[^42] |
| **Gemini 3.8 Flash (High)** | `15,111` in / `3,451` out | `0.00016180` | `$0.0243 USD` (at `$0.75 / $3.75` per 1M) | **`0.00667`** | `1.00x` (Baseline)[^42] |

- **Key Findings**:
  1. Normalized per dollar of published API list price, **`Gemini 4 Argon` drains exactly `3.00x` as much Antigravity subscription quota per nominal dollar of work as `Gemini 3.8 Flash`**[^42].
  2. Benchmarked against `3.8 Flash`'s published API pricing, the Ultra 5x 5-hour shared quota pool carries an implied nominal budget of **`$150.0 USD`** (`$0.024274 / 0.00016180 = $150.0`), whereas `Gemini 4 Argon` is internally weighted at **$`6.00` input / `$30.00` output per 1M Tokens** inside the subscription pool (3x its advertised `$2.00 / $10.00` launch price, with cached reads weighted around `$0.30/1M`)[^42].
  3. Furthermore, a single subagent probe routed to `gemini-3.8-flash-tiered` (internal model enum `M322`, `12,773` input / `16,332` output, 0 cached) caused `remainingFraction` to plunge by **`0.01048544`**—**`22x` the expected drain of standard Flash (`~0.00047`)**[^42]!

#### Experiment 2: Gemini Web 8-Decimal-Place Weekly Pool (`ΔWeekly`) Pure-Input, Zero-Thinking Cross-Verification
To rule out local language-server accounting quirks or hidden "Thinking Tokens," `@Yiji_Kong` ran three controlled tests on October 29 under the same Ultra account by capturing the **8-decimal-place weekly pool usage (`percentageUsed`)** returned by the Web `/usage` endpoint with thinking tokens isolated to zero[^42]:

| Test ID & Scenario | Model & Thinking Tier | Input / Output Payload | 5-Hourly Pool Delta | Exact Weekly Pool Delta (`ΔWeekly`, 8 Decimal Places) | Measured Quota Drain Multiplier |
|---|---|---|---:|---:|---|
| **Test 1A (Pure Input, 0 Thinking, 0 Cache)** | `Gemini 3.8 Flash (Low)` | `51,262` chars (~13.5K tokens) in / reply `"OK"` (1 token, 0 thinking) | `0.10 → 0.10` (`0.00`) | **`+0.00003685`** (`0.09984048 → 0.09987733`) | Baseline `1.00x`[^42] |
| **Test 1B (Pure Input, 0 Thinking, 0 Cache)** | `Gemini 4 Argon (Low)` | `51,262` chars (equal-length Perm B) in / reply `"OK"` (1 token, 0 thinking) | `0.10 → 0.12` (`+0.02`) | **`+0.00090908`** (`0.09987733 → 0.10078641`) | **`24.67x` raw drain** vs. Flash (**`9.25x per published dollar`**; or **`20.37x` raw / `7.64x` per dollar** after subtracting fixed per-request base overhead)[^42] |
| **Test 2A (High-Thinking 4,000-Word Essay)** | `Gemini 3.8 Flash (High)` | `107` chars in / generated `28,389` chars (`4,013`-word Paxos/Raft article) | `0.12 → 0.12` (`0.00`) | **`+0.00014157`** (`0.10078641 → 0.10092798`) | Writing a 4,013-word deep technical article consumed just `0.014%` of the weekly pool[^42] |
| **Test 2B (High-Thinking 4,000-Word Essay)** | `Gemini 4 Argon (High)` | `107` chars in / generated `27,859` chars (`4,284`-word Paxos/Raft article) | `0.12 → 0.15` (`+0.03`) | **`+0.00157077`** (`0.10092798 → 0.10249875`) | **`11.10x` raw drain** vs. Flash (**`4.24x per published dollar`**, burning **`~3.1%`** of the 5-hourly pool in a single turn)[^42] |
| **Test 1C (Minimal 28-Char Base Overhead Calibration)** | `Gemini 4 Argon (Low)` | 28 chars `"Reply with only the word: OK"` in / reply `"OK"` | `0.15 → 0.16` (`+0.01`) | **`+0.00015849`** (`0.10249875 → 0.10265724`) | **Sending a single `"OK"` ping on Argon (Low) (`+0.00015849`) burned `12%` MORE quota than generating a 4,013-word technical article on 3.8 Flash (High) (`+0.00014157`)!**[^42] |

Confronted with this proof, Google staff member `@Maniraj_Chervirala` publicly admitted in post #6 of thread `#196351` that the unit prices on the API pricing page are *"promotional per-token rates"* exclusive to the paid API, whereas consumer subscription 5-hour and weekly limits **are not pegged to published API rates and *"Fixed per-model quota multipliers are not published"***[^42].

### 3.4 Hidden Higher Context-Tier Quota Multipliers (`1.0x` / `1.3x` / `1.8x Expensive`)
Synthesizing `u/Lberondroth`'s ASAR/binary teardown of Desktop `v2.27.0 → v2.28.1` (`#1xij7jx`)[^18], the `r/AntigravityGoogle` community teardown thread (`#1xmc5pk`)[^19], and `@LiraNuna`'s unlocked CLI `/model` UI measurement in Issue `#1245`[^1], Google has internally bound the three context window options directly to per-turn quota multipliers:
- **`Default (256K)` / `256K`**: Baseline quota multiplier of **`1.0x`**[^1][^19];
- **`512K`**: Displays `about 1.3x quota per turn` in the UI (**`1.3x`** quota multiplier per turn)[^1][^19];
- **`900K`**: Carries the boolean flag **`Expensive: true`** in the binary struct, with a per-turn quota multiplier of **`1.8x`**[^18][^19].

---

## 4. Official Issue Status & Binary Changes in Desktop v2.29.0 / CLI 1.5.0–1.5.1

### 4.1 On GitHub and Community Channels: Self-Assigned Tickets Without a `bug` Label or Public Reply
1. **Labeling `#1245` and `#1291` as `feedback` Without a `bug` Label**:
   After `@Burntgogi` filed **Issue #1245** on October 14, 2026, Google repository maintainer `@chandrashakherkamasani` self-assigned the issue on October 15, labeling it with `comp:agent capabilities`, `feedback`, and `1.3.5` without applying the `bug` label (and similarly labeling `@Emily2040`'s October 19 **Issue #1291** as `feature request` and `feedback` while self-assigning it)[^1][^4]. As of November 5, 2026 (including after the `v1.5.1` release)—more than 21 days later—despite community members posting SQLite records, x86-64 assembly clamp instructions, and binary patches in the thread, the number of public replies from Google staff on `#1245`, `#878`, and `#1291` stands at **0**[^1][^3][^4].
2. **The Official Response Came on the Forum, Not on GitHub (2026-11-04)**:
   The only official statement on the threshold so far appeared in Google AI Developers Forum thread **#198191**, where `@Sharper` asked for larger-than-250K context sizes in the Desktop app. Google staff member `@chunduriv` replied at 23:39 UTC: *"As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. That threshold is a fixed default and isn't user-configurable yet."* The reply treats the report as a feature request for configurable compaction, lists three interim mitigations (manual `/compact` at a natural break point, keeping reference material in workspace files the agent can re-read after compaction, and excluding build artifacts, dependencies, and generated files), and says the request for a user-selectable compaction size has been passed to the engineering team for review. It gives no timeline and does not address the CLI, the `0x3e800` clamp, or the GitHub Issues listed above[^57].
3. **The Underlying Commercial & Compute Trade-Off [Analytical Inference]**:
   Synthesizing the empirical findings from `@Circumsized`, `@Emily2040`, and `@Yiji_Kong`, the lack of public response on `#1245` reflects a cost trade-off: if Google treats `0x3e800` in `GetCheckpointConfig` as a bug and unlocks the full 1M window by default for all subscribers, the combination of multi-step tool loops and frequent `system_message`-induced prefix cache misses (where re-sending 800K–900K of uncached context increases raw compute cost by >4x per miss) would significantly increase compute costs on fixed-rate monthly subscriptions[^1][^4]; conversely, acknowledging that `256,000` (25%) is an intentional cost cap highlights the discrepancy between marketing a "1M Context Window" on product pages and the `/context` panel while clamping 75% of that capacity in client code[^1][^19].

### 4.2 Under-the-Hood Changes Across CLI `1.4.3–1.5.1` and Desktop `v2.29.0`
Although Google staff have posted no replies in the Issue threads, inspecting the 10 release changelogs published in `google-antigravity/antigravity-cli` between October 20 and November 5, 2026 (`v1.4.3` through `v1.5.1`)[^11] alongside `u/Lberondroth`'s November 3 **Desktop `v2.29.0` binary diff teardown (`#1xn624i`)**[^17] shows the three parallel tracks of changes:

1. **Track 1: Retaining the Default 256K Auto-Compaction Threshold While Patching Compaction Side Effects (CLI `1.4.3`–`1.4.10`)**[^11]:
   - **`v1.4.3` (2026-10-20)**: Fixed a bug where parent agents lost track of recently launched subagents after session compaction (caused because the old summarizer stuffed each subagent's entire initial prompt into the summary, getting truncated before recording the subagent's `conversationId`; the new version retains only subagent type, role, and outcome)[^11].
   - **`v1.4.4` (2026-10-21)**: Improved `/compact` and auto-compaction requests to retain `tool declarations` when dispatching summarization calls so they can reuse the cached prompt prefix (directly implementing the cache-aware compaction fix proposed by `@Emily2040` in `#1291`, without crediting or replying to the Issue)[^4][^11].
   - **`v1.4.6` (2026-10-23)**: Improved context compaction to retain user-attached images, PDFs, and other files across conversation summaries, preventing multimodal attachments from vanishing after compaction[^11].
   - **`v1.4.7` (2026-10-24)**: Fixed a deadlock bug where a single message, tool output, or attachment too large to fit in context caused the session to enter an infinite compaction loop[^11].
   - **`v1.4.9` (2026-10-28)**: Fixed a bug where trimming older history left orphaned tool calls or tool results, causing every subsequent model request in the session to fail (confirming `@Circumsized`'s discovery that silent Pre-flight truncation/trimming is actively operating alongside compaction)[^1][^11].
   - **`v1.4.10` (2026-10-29)**: (1) Fixed a critical bug where subagents completely forgot their assigned task and parent agent ID after undergoing context compaction (now forcibly retaining the last 10 messages from the parent agent in the subagent summary); (2) Updated compaction banners and `/context` post-compaction size accounting to include carried-over historical requests, artifacts, subagents, and background task snapshots alongside the raw summary text (instead of displaying only the bare summary size)[^11].
2. **Track 2: Refactor the Underlying Compaction Engine in Desktop `v2.29.0` & CLI `1.5.0`, Silently Restore `AGY_CLI_AUTO_COMPACT_OVERRIDE`, and Add a "High-Definition Quota Drain Indicator"**[^17]:
   - **Backend Binary Bloat and New Internal Packages**: In the Desktop `v2.29.0` update pushed on November 3, 2026 (and CLI `v1.5.0` on November 4), the backend `language_server.exe` binary grew by **`+4.47 MB`** in a single release, while the embedded Web UI grew by **`+36%`**[^17]. Disassembled symbol tables (verified directly in our local `agy 1.5.0` binary) show that Google added two core internal packages:
     - **`google3/third_party/jetski/cortex/tokens/compaction`** (containing granular accounting functions including `RequestTokens`, `SummaryTokens`, `CheckpointTokens`, `CarriedTokens`, `ContextOverheadTokens`, and `OverheadTokens`);
     - **`google3/third_party/jetski/cortex/shared/snapshot`** (containing snapshot management functions including `RegisterProducer`, `LookupProducer`, and `NewMemoryBuffer`)[^17].
   - **Silent Restoration of `AGY_CLI_AUTO_COMPACT_OVERRIDE` Alongside the Unchanged `0x3E800` Clamp**: As detailed in §2.1 above, `v1.5.0` quietly restored the `os.LookupEnv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` call inside `store.AutoCompactOverride` without documenting it in `CHANGELOG.md`, while keeping the `0x3E800` (`256,000`) clamp in `GetCheckpointConfig` and the length-1 `"Gemini 4 Argon"` whitelist intact.
   - **Feature-Gated `enable-context-usage-indicator`**: Right next to the send button in Desktop `v2.29.0`, Google added a server-gated **"Context and contingent Indicator"** designed to display both context window occupancy and remaining plan quota side by side (with strings for *"Resets in..."* and *"Your plan's baseline contingent will refresh on..."*)[^17]. As teardown author `u/Lberondroth` dryly concluded:
     > *"What you get instead is the Context and contingent Indicator, so you can soon watch your contingent drain in high definition, right next to the send button..."*[^17]
3. **Track 3: CLI `v1.5.1` Maintenance Update (2026-11-05 ~07:34 UTC) — Zero Changes to the 256K Context Limit**[^11]:
   - Just prior to the publication of this archive on November 5, 2026 (~07:34 UTC), Google pushed CLI **`v1.5.1`** (`240,857,488` bytes, BuildID `696031ae34e5e5e26a9d7c436e85317f`, SHA-256 `72738cf0445918483956a5f0da33abdfcfb21a12dd9a421fd57d063782826669`). All 11 entries in `CHANGELOG.md` are routine maintenance fixes (`agy acp` model availability and session bug fixes, `--remote-agent` `/config` label renames, shortening post-exit child process output wait from 5s to 1s, subagent blocked-tool display, plugin OAuth token header hardening, Windows CRLF/encoding/stdio fixes, `AGY_ADC_AUTH` `agy plugin` fix, hiding the built-in `browser` agent from the session list, and GNU screen copy fix)—with **not a single word addressing context windows, compaction thresholds, or 256K**[^11].
   - Direct binary comparison between `v1.5.1` and `v1.5.0` confirms that the 14-byte `0x3E800` (`256,000`) clamp in `GetCheckpointConfig` (`81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2`) is preserved byte-for-byte (shifting only from file offset `0x7fe5802` to `0x7fb0fe2`), all whitelist symbols (`offersContextWindow`, `store.ContextWindow`, `ClampCheckpointConfigForModelInfo`) and the restored `AGY_CLI_AUTO_COMPACT_OVERRIDE` lookup remain identical, and no new context- or compaction-related strings were added.

---

## Reference Sources Cited in This Volume (Footnote Definitions)

[^1]: [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) — GitHub `google-antigravity/antigravity-cli` (Author: `Burntgogi`; Commenters: `NuCl34R`, `Circumsized`, `Emily2040`, `LiraNuna`, `AsabaLazy`), Published: 2026-10-14 to 2026-11-02 (Accessed: 2026-11-04 | Type: Primary Official Repository Issue & Binary Reverse-Engineering Proof | Confidence: High)
[^2]: [Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) — GitHub Gist (Author: `LiraNuna`), Published: 2026-10-23 (Accessed: 2026-11-04 | Type: Primary Reverse-Engineering Patch Source Code | Confidence: High)
[^3]: [[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878)](https://github.com/google-antigravity/antigravity-cli/issues/878) — GitHub `google-antigravity/antigravity-cli` (Author: `schalatap`; Commenters: `bash99`, `murias002`, `Corsagee`, `Infinnerty`, `jlukic`), Published: 2026-08-26 to 2026-10-31 (Accessed: 2026-11-04 | Type: Primary Official Repository Issue & Large-Scale Session Telemetry | Confidence: High)
[^4]: [[Bug]: Five measured places the CLI spends time and tokens the work does not need: serial tool calls, uncached full-context calls and 70 s compactions, 12 s headless overhead, summary store rewritten every 1.5 s, build-time CPU instruction choice (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) — GitHub `google-antigravity/antigravity-cli` (Author: `Emily2040`), Published: 2026-10-19 to 2026-10-25 (Accessed: 2026-11-04 | Type: Primary Official Repository Issue & Benchmark Audit | Confidence: High)
[^5]: [[Feature Request]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue #1072)](https://github.com/google-antigravity/antigravity-cli/issues/1072) — GitHub `google-antigravity/antigravity-cli` (Author: `flotoz`; Commenter: `Emily2040`), Published: 2026-09-20 to 2026-10-19 (Accessed: 2026-11-04 | Type: Primary Official Repository Issue | Confidence: High)
[^11]: [Google Antigravity CLI Releases & Changelog (`v1.4.2` – `v1.5.1`)](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md) — GitHub `google-antigravity/antigravity-cli`, Published: 2026-10-18 to 2026-11-05 (Accessed: 2026-11-05 | Type: Primary Official Release Notes | Confidence: High)
[^12]: [Google Antigravity Docs — Changelog (`v2.0.1` – `v2.29.0`)](https://antigravity.google/docs/changelog) — Google Antigravity Official Documentation, Published: 2026-11-03 (Accessed: 2026-11-04 | Type: Primary Official Documentation | Confidence: High)
[^17]: [[Teardown / Binary Diff] Antigravity Updated to v2.29.0](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/) — Reddit `r/google_antigravity` (Author: `u/Lberondroth`), Published: 2026-11-03 (Accessed: 2026-11-04 | Type: Specialist Community Binary Teardown Report | Confidence: High)
[^18]: [[Deep-Dive] Antigravity v2.27.0 → v2.28.1 Binary & ASAR Diff: 'Script Jail', the `oneharness` Migration, Teamwork Leaving Preview, and What's Coming Next](https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/) — Reddit `r/google_antigravity` (Author: `u/Lberondroth`), Published: 2026-10-28 (Accessed: 2026-11-04 | Type: Specialist Community Binary Teardown Report | Confidence: High)
[^19]: [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon?](https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/) — Reddit `r/AntigravityGoogle`, Published: 2026-11-02 (Accessed: 2026-11-04 | Type: Specialist Community Discussion | Confidence: High)
[^42]: [Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351)](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351) — Google AI Developers Forum (Author: `@Yiji_Kong`; Official Reply: `@Maniraj_Chervirala`), Published: 2026-10-28 to 2026-10-29 (Accessed: 2026-11-04 | Type: Primary Benchmark Report & Official Forum Reply | Confidence: High)
[^43]: [Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476) — Google AI Developers Forum (Author: `@Hello_there`; Official Reply: `@chunduriv`), Published: 2026-10-15 to 2026-10-20 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Confidence: High)
[^44]: [Feature request: Show context window usage in Antigravity Desktop (#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694) — Google AI Developers Forum (Author: `@Soro`; Official Reply: `@chunduriv`; Extension: `@dimitr-io`), Published: 2026-09-24 to 2026-10-22 (Accessed: 2026-11-04 | Type: Primary Official Forum Reply | Confidence: High)
[^46]: [Antigravity CLI Agent reliability issue (#176775)](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775) — Google AI Developers Forum (Author: `@Pitt_zhang`), Published: 2026-08-01 (Accessed: 2026-11-04 | Type: Specialist Community Report | Confidence: High)
[^51]: [dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context) — GitHub (Author: `dimitr-io`), Published: 2026-10-22 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Confidence: High)
[^57]: [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) — Google AI Developers Forum (Author: `@Sharper`; Official Reply: `@chunduriv`), Published: 2026-11-04 (Accessed: 2026-11-05 | Type: Primary Official Staff Forum Reply | Confidence: High)
