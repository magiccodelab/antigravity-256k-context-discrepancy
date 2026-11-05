# [Google Antigravity 256K Hardcoded Context Limit & `agy -p` Policy Reversal Incident] Full Reference Sources & Search Archive

- **Language**: **English** | [简体中文 (zh-cn/sources.md)](./zh-cn/sources.md)
- **Archive Navigation**: [Repository Overview](../README.md) | [Overview Report (dossier.md)](./dossier.md) | **Full Sources Archive (This File)** | [Static Forensic Report](../evidence/evidence-report.md)
- **Thematic Volumes**: [01-Incident Overview & Timeline](./01-incident-overview-and-timeline.md) | [02-256K Hardcoded Gates & Binary Teardown](./02-256k-hardcoded-gates-and-binary-teardown.md) | [03-agy-p Policy Reversal & Bans](./03-agy-p-headless-policy-reversal-and-bans.md) | [04-Community & Official Responses](./04-community-and-official-responses.md) | [05-Verbatim Quotes & Primary Records](./05-verbatim-quotes-and-evidence-vault.md)
- **Research ID (UUID)**: `df866cf1-da40-4b82-8084-34c6b0729544` | **Directory Path**: `research/` | **Research Date**: 2026-11-04 | **Cited Body Sources**: 57 | **Supplementary Sources**: 9

---

## 1. Master Sources Index Table

| ID | Source Title & Link | Organization / Author | Publication Date | Source Tier | Confidence | Citation Locations in Master Dossier & Volumes |
|---|---|---|---|---|---|---|
| `[1]` (`[^1]`) | [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) | GitHub `google-antigravity/antigravity-cli` / `Burntgogi`, `NuCl34R`, `Circumsized`, `Emily2040`, `LiraNuna`, `AsabaLazy` | 2026-10-14 – 2026-11-02 | Primary Official Repo Issue & Binary Reverse-Engineering | High | `dossier.md` §1, §3.2, §3.3, §4, §5, §6–§9; Volumes 1–5 |
| `[2]` (`[^2]`) | [Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) | GitHub Gist / `@LiraNuna` | 2026-10-23 | Primary Reverse-Engineering Patch Source Code | High | `dossier.md` §1, §3.1, §3.2, §3.5, §4; Volumes 1, 2, 4 |
| `[3]` (`[^3]`) | [[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878)](https://github.com/google-antigravity/antigravity-cli/issues/878) | GitHub `google-antigravity/antigravity-cli` / `schalatap`, `murias002`, `Corsagee`, `Infinnerty`, `jlukic` | 2026-08-26 – 2026-10-31 | Primary Official Repo Issue & 65K-Call Telemetry | High | `dossier.md` §1, §3.2, §4, §8, §9; Volumes 1, 2, 4, 5 |
| `[4]` (`[^4]`) | [[Bug]: Five measured places the CLI spends time and tokens the work does not need (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) | GitHub `google-antigravity/antigravity-cli` / `@Emily2040` | 2026-10-19 – 2026-10-25 | Primary Official Repo Issue & 1,279-Call Benchmark | High | `dossier.md` §1, §3.2, §3.3, §4, §8, §9; Volumes 1, 2, 4, 5 |
| `[5]` (`[^5]`) | [[Feature Request]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue #1072)](https://github.com/google-antigravity/antigravity-cli/issues/1072) | GitHub `google-antigravity/antigravity-cli` / `@flotoz`, `@Emily2040` | 2026-09-20 – 2026-10-19 | Primary Official Repo Issue | High | `dossier.md` §3.3, §4; Volumes 1, 2 |
| `[6]` (`[^6]`) | [Feature Request: Add `max_context_tokens` setting to control per-turn token budget (Issue #743)](https://github.com/google-antigravity/antigravity-cli/issues/743) | GitHub `google-antigravity/antigravity-cli` / `@thealgebraist` | 2026-08-03 | Primary Official Repo Issue | High | `dossier.md` §3.2; Volumes 1, 4 |
| `[7]` (`[^7]`) | [Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue #366)](https://github.com/google-antigravity/antigravity-cli/issues/366) | GitHub `google-antigravity/antigravity-cli` / `@UesugiErii`, `@Mandark-droid`, `@evilfurryone`, `@rajatim` | 2026-06-11 – 2026-10-20 | Primary Official Repo Issue | High | `dossier.md` §3.1; Volumes 1, 4 |
| `[8]` (`[^8]`) | [agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync (Issue #927)](https://github.com/google-antigravity/antigravity-cli/issues/927) | GitHub `google-antigravity/antigravity-cli` / `@jlwainwright` | 2026-09-02 | Primary Official Repo Issue | High | `dossier.md` §3.4, §4; Volumes 1, 3 |
| `[9]` (`[^9]`) | [Addressing Antigravity Bans & Reinstating Access (Discussion #20632)](https://github.com/google-gemini/gemini-cli/discussions/20632) | GitHub `google-gemini/gemini-cli` / Google Maintainer `@jackwotherspoon` | 2026-02-27 | Primary Official Maintainer Announcement | High | `dossier.md` §1, §2, §3.4, §4, §8; Volumes 1, 3 |
| `[10]` (`[^10]`) | [Appeal submitted but TOS_VIOLATION suspension persists (Issue #29576)](https://github.com/google-gemini/gemini-cli/issues/29576) | GitHub `google-gemini/gemini-cli` / `@myessaylab1234` | 2026-09-30 | Primary Official Repo Issue | High | `dossier.md` §2, §3.1, §3.4, §4; Volumes 1, 3 |
| `[11]` (`[^11]`) | [Google Antigravity CLI Official Releases (`v1.4.2` – `v1.5.0`)](https://github.com/google-antigravity/antigravity-cli/releases) | GitHub `google-antigravity/antigravity-cli` | 2026-10-18 – 2026-11-04 | Primary Official Release Notes | High | `dossier.md` §1, §2, §3.3, §3.5, §7, §8, §10; Volumes 1–4 |
| `[12]` (`[^12]`) | [Google Antigravity Docs — Changelog (`v2.0.1` – `v2.29.0`)](https://antigravity.google/docs/changelog) | Google Antigravity Official Documentation | 2026-11-03 | Primary Official Documentation | High | `dossier.md` §1, §2, §4, §8; Volumes 1, 2, 4 |
| `[13]` (`[^13]`) | [Google Antigravity Docs — Official FAQ](https://antigravity.google/docs/faq/) | Google Antigravity Official Documentation | 2026-10 | Primary Official Documentation | High | `dossier.md` §1, §2, §3.4, §6, §8; Volumes 1, 3, 4 |
| `[14]` (`[^14]`) | [Google Antigravity — Additional Terms of Service](https://antigravity.google/terms) | Google LLC | 2026-09 | Primary Official Contractual Terms | High | `dossier.md` §1, §2, §7; Volumes 1, 3 |
| `[15]` (`[^15]`) | [Google Antigravity Docs — Workspace Rules (24 KB Cap & 20,000-Token Budget)](https://antigravity.google/docs/rules/) | Google Antigravity Official Documentation | 2026-10 | Primary Official Documentation | High | `dossier.md` §3.5, §4 |
| `[16]` (`[^16]`) | [Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/) | Google Developers Blog | 2026-10 | Primary Official Technical Blog | High | `dossier.md` §3.5; Volume 4 |
| `[17]` (`[^17]`) | [[Teardown / Binary Diff] Antigravity Updated to v2.29.0](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/) | Reddit `r/google_antigravity` / `u/Lberondroth` | 2026-11-03 | Specialist Community Binary Teardown Report | High | `dossier.md` §2, §3.3, §4, §6, §8, §10; Volumes 1, 2, 5 |
| `[18]` (`[^18]`) | [[Deep-Dive] Antigravity v2.27.0 → v2.28.1 Binary & ASAR Diff](https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/) | Reddit `r/google_antigravity` / `u/Lberondroth` | 2026-10-28 | Specialist Community Binary Teardown Report | High | `dossier.md` §1, §3.3, §4, §6, §8; Volumes 1, 2 |
| `[19]` (`[^19]`) | [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon?](https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/) | Reddit `r/AntigravityGoogle` | 2026-11-02 | Specialist Community Discussion | High | `dossier.md` §3.3, §4, §8, §10; Volumes 1, 2 |
| `[20]` (`[^20]`) | [Timeline for enabling the 512k / 900k ContextWindowOptions (`r/google_antigravity`)](https://www.reddit.com/r/google_antigravity/comments/1xmc89l/timeline_for_enabling_the_512k_900k/) | Reddit `r/google_antigravity` | 2026-11-02 | Specialist Community Discussion | High | `dossier.md` §8; Volume 1 |
| `[21]` (`[^21]`) | [Antigravity's aggressive history compaction is ruining Gemini's 1M+ context selling point](https://www.reddit.com/r/google_antigravity/comments/1trr4dy/antigravitys_aggressive_history_compaction_is/) | Reddit `r/google_antigravity` | 2026-05-30 | Specialist Community Discussion | High | `dossier.md` §8; Volume 1 |
| `[22]` (`[^22]`) | [How to force AG to use subagents? (Orchestrator + Ephemeral Subagents)](https://www.reddit.com/r/google_antigravity/comments/1uo7g2k/how_to_force_ag_to_use_subagents/) | Reddit `r/google_antigravity` | 2026-07 | Specialist Community Engineering Practice | High | `dossier.md` §11 (footnote list only) |
| `[23]` (`[^23]`) | [Clarification Request: Is local IPC/PTY wrapping of `agy` CLI for open-standard editor bridges (e.g., ACP) ToS compliant? (#175905)](https://discuss.ai.google.dev/t/clarification-request-is-local-ipc-pty-wrapping-of-agy-cli-for-open-standard-editor-bridges-e-g-acp-tos-compliant/175905) | Google AI Developers Forum / `@shindgew`, Official `@Rebbanpalli_Naveen` | 2026-07-23 – 2026-09-16 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §6, §8; Volumes 1, 3 |
| `[24]` (`[^24]`) | [Is it allowed for an open-source tool to call the official `agy` CLI in headless mode? (#183429)](https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429) | Google AI Developers Forum / `@darkest_alex`, Official `@Ambati_Rajendra` | 2026-09-17 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §6, §8; Volumes 1, 3, 5 |
| `[25]` (`[^25]`) | [Is using the official agy CLI through a local MCP server with third-party AI agents permitted? (#184829)](https://discuss.ai.google.dev/t/is-using-the-official-agy-cli-through-a-local-mcp-server-with-third-party-ai-agents-permitted/184829) | Google AI Developers Forum / `@snowstone`, Official `@Ambati_Rajendra` | 2026-09-25 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4; Volume 1 |
| `[26]` (`[^26]`) | [One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? (#185460)](https://discuss.ai.google.dev/t/one-person-several-paid-google-ai-pro-accounts-may-a-local-orchestrator-use-them-with-the-official-agy-cli/185460) | Google AI Developers Forum / `@Winez`, Official `@Ambati_Rajendra` | 2026-09-28 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §8; Volumes 1, 3 |
| `[27]` (`[^27]`) | [Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (#185494)](https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494) | Google AI Developers Forum / `@Diggy_Styon`, Official `@Ambati_Rajendra` | 2026-09-28 | Primary Official Staff Forum Reply | High | `dossier.md` §2, §3.4, §6, §8; Volumes 1, 3, 5 |
| `[28]` (`[^28]`) | [Is invoking the official `agy` CLI from a local AI coding assistant allowed on an individual Google AI account? (#185992)](https://discuss.ai.google.dev/t/is-invoking-the-official-agy-cli-from-a-local-ai-coding-assistant-allowed-on-an-individual-google-ai-account/185992) | Google AI Developers Forum / `@Ivan_Valenzuela`, Official `@chunduriv` | 2026-09-29 – 2026-10-01 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §6, §8; Volumes 1, 3 |
| `[29]` (`[^29]`) | [Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper (#185949)](https://discuss.ai.google.dev/t/clarification-on-codex-claude-skill-invoking-the-official-agy-cli-through-a-local-powershell-wrapper/185949) | Google AI Developers Forum / `@nothing2`, Official `@Ambati_Rajendra` | 2026-09-29 – 2026-10-02 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §8; Volumes 1, 3 |
| `[30]` (`[^30]`) | [Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? (#186954)](https://discuss.ai.google.dev/t/is-automatically-launching-the-official-agy-cli-from-a-local-ai-orchestrator-allowed-under-additional-terms-6/186954) | Google AI Developers Forum / `@local_ai_builder`, Official `@Rebbanpalli_Naveen` | 2026-10-05 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §8; Volumes 1, 3 |
| `[31]` (`[^31]`) | [Is invoking the official Antigravity CLI from Codex allowed? (#186952)](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952) | Google AI Developers Forum / `@Hossy`, Official `@Rebbanpalli_Naveen` | 2026-10-05 | Primary Official Staff Forum Reply | High | `dossier.md` §1, §3.1, §3.4, §4, §6, §8; Volumes 1, 3, 5 |
| `[32]` (`[^32]`) | [Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (#186246)](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246) | Google AI Developers Forum / `@Croixofwar`, Official `@chunduriv` | 2026-10-01 – 2026-11-02 | Primary Official Staff Forum Reply | High | `dossier.md` §1, §3.1, §3.4, §4, §6, §8, §9; Volumes 1, 3, 5 |
| `[33]` (`[^33]`) | [Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator (#187536)](https://discuss.ai.google.dev/t/terms-clarification-personal-antigravity-cli-integration-with-a-multi-ai-orchestrator/187536) | Google AI Developers Forum / `@BuddhaRyu`, Official `@Ambati_Rajendra` | 2026-10-08 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §4, §6, §8; Volumes 1, 3 |
| `[34]` (`[^34]`) | [Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241) | Google AI Developers Forum / `@ALAMDAR_BALOCH`, Official `@chunduriv` | 2026-10-07 – 2026-10-08 | Primary Official Staff Forum Reply | High | `dossier.md` §1, §3.4, §4, §6, §8; Volumes 1, 3, 4, 5 |
| `[35]` (`[^35]`) | [Clarification on consumer-plan policy for invoking official agy via local subprocess (#189905)](https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905) | Google AI Developers Forum / `@Amane2418`, `@Jay10`, Official `@Rebbanpalli_Naveen` | 2026-10-16 – 2026-10-26 | Primary Official Staff Forum Reply | High | `dossier.md` §1, §2, §3.4, §6, §8; Volumes 1, 3, 4, 5 |
| `[36]` (`[^36]`) | [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689) | Google AI Developers Forum / `@eliavb`, `@b_w`, `@Tora39`, Official `@Rebbanpalli_Naveen` | 2026-10-23 – 2026-10-27 | Primary Official Staff Forum Reply | High | `dossier.md` §1, §3.4, §3.5, §4, §6–§9; Volumes 1, 3, 4, 5 |
| `[37]` (`[^37]`) | [Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998)](https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998) | Google AI Developers Forum / `@Herzchen`, Official `@Rebbanpalli_Naveen` | 2026-10-26 | Primary Official Staff Forum Reply | High | `dossier.md` §3.5; Volumes 1, 3, 4 |
| `[38]` (`[^38]`) | [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815) | Google AI Developers Forum / `@Lars`, Official `@Ambati_Rajendra` | 2026-10-19 – 2026-10-28 | Primary Official Staff Forum Reply | High | `dossier.md` §1, §3.1, §3.4, §6, §8; Volumes 1, 3, 4, 5 |
| `[39]` (`[^39]`) | [Policy clarification: private monitoring app using official Antigravity CLI and subscription quota (#190756)](https://discuss.ai.google.dev/t/policy-clarification-private-monitoring-app-using-official-antigravity-cli-and-subscription-quota/190756) | Google AI Developers Forum / `@xorz`, `@oli_hlmpn`, Official `@Ambati_Rajendra` | 2026-10-19 – 2026-10-29 | Primary Official Staff Forum Reply | High | `dossier.md` §3.1, §3.4, §6, §7; Volumes 1, 3 |
| `[40]` (`[^40]`) | [Clarification on subscription-based agy -p integration with a local multi-agent framework (#196472)](https://discuss.ai.google.dev/t/clarification-on-subscription-based-agy-p-integration-with-a-local-multi-agent-framework/196472) | Google AI Developers Forum / `@Dsssyc`, Official `@Rebbanpalli_Naveen` | 2026-10-28 – 2026-10-29 | Primary Official Staff Forum Reply | High | `dossier.md` §3.4, §6, §8; Volumes 1, 3 |
| `[41]` (`[^41]`) | [Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131)](https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131) | Google AI Developers Forum / `@sannikov`, `@tkosub`, `@oli_hlmpn`, Official `@Rebbanpalli_Naveen` | 2026-10-31 – 2026-11-02 | Primary Official Staff Forum Reply | High | `dossier.md` §3.5, §6, §7; Volumes 3, 4 |
| `[42]` (`[^42]`) | [Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351)](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351) | Google AI Developers Forum / `@Yiji_Kong`, `@RobW`, `@shuai_guo`, Official `@Maniraj_Chervirala` | 2026-10-28 – 2026-10-29 | Primary Benchmark Report & Official Reply | High | `dossier.md` §1, §2, §3.1, §3.3, §4, §6, §8, §9; Volumes 1, 2, 4, 5 |
| `[43]` (`[^43]`) | [Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476) | Google AI Developers Forum / `@Hello_there`, Official `@chunduriv` | 2026-10-15 – 2026-10-20 | Primary Official Staff Forum Reply | High | `dossier.md` §3.3, §3.5, §6; Volumes 1, 2, 4 |
| `[44]` (`[^44]`) | [Feature request: Show context window usage in Antigravity Desktop (#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694) | Google AI Developers Forum / `@Soro`, Official `@chunduriv`, `@dimitr-io` | 2026-09-24 – 2026-10-22 | Primary Official Staff Forum Reply | High | `dossier.md` §4, §5, §7; Volumes 1, 2, 4 |
| `[45]` (`[^45]`) | [Persistent Gemini CLI TOS_VIOLATION after appeal — request for private review or sync check (#186089)](https://discuss.ai.google.dev/t/persistent-gemini-cli-tos-violation-after-appeal-request-for-private-review-or-sync-check/186089) | Google AI Developers Forum / `@myessaylab1234` | 2026-09-30 | Specialist Community Report | High | `dossier.md` §2, §3.1, §3.4, §4; Volumes 1, 3 |
| `[46]` (`[^46]`) | [Antigravity CLI Agent reliability issue (#176775)](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775) | Google AI Developers Forum / `@Pitt_zhang` | 2026-08-01 | Specialist Community Audit Report | High | `dossier.md` §3.3; Volumes 1, 2 |
| `[47]` (`[^47]`) | [Google Antigravity FAQ Policy Update and Termination of Third-Party Tool Integration](https://note.com/horizon_it00/n/nded285d721d1) | note.com / `horizon_it00` (Kenji Sato) | 2026-10 | Technical Blog | High | `dossier.md` §3.4; Volume 1 |
| `[48]` (`[^48]`) | [Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (`239,616` Compaction Threshold)](https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk) | Nous Research Hermes Agent Docs | 2026-09-23 | Technical Documentation | High | `dossier.md` §3.2, §4; Volume 1 |
| `[49]` (`[^49]`) | [mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management](https://github.com/mksglu/context-mode) | GitHub / `@mksglu` | 2026 | Open-Source Code Repository | High | `dossier.md` §3.5; Volume 4 |
| `[50]` (`[^50]`) | [AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE](https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor) | GitHub / `@AGI-is-going-to-arrive` | 2026 | Open-Source Code Repository | High | `dossier.md` §3.5; Volume 4 |
| `[51]` (`[^51]`) | [dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context) | GitHub / `@dimitr-io` | 2026-10-22 | Open-Source Code Repository | High | `dossier.md` §3.5, §5, §7; Volumes 1, 2, 4 |
| `[52]` (`[^52]`) | [weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI](https://github.com/weby-homelab/antigravity-cli-statusline) | GitHub / Weby Homelab | 2026 | Open-Source Code Repository | High | `dossier.md` §3.5; Volume 4 |
| `[53]` (`[^53]`) | [oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit ("The IDE for Your Coding Agent")](https://github.com/oraios/serena) | GitHub / Oraios AI | 2026 | Open-Source Code Repository | High | `dossier.md` §3.5; Volume 4 |
| `[54]` (`[^54]`) | [xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server)](https://github.com/xdotech/goatlas) | GitHub / `@xdotech` | 2026 | Open-Source Code Repository | High | `dossier.md` §3.5; Volume 4 |
| `[55]` (`[^55]`) | [Kevin Hou — "Get Out of the Model's Way" (AI Engineer Summit Presentation & Speaker Profile)](https://www.youtube.com/watch?v=xsVD9_cJNYs) | AI Engineer / YouTube (`https://ai.engineer/speakers/kevin-hou`) / Kevin Hou | 2026 | Primary Executive Public Keynote | High | `dossier.md` §3.1, §8; Volume 1 |
| `[56]` (`[^56]`) | [Varun Mohan Education Qualifications: How an Indian-Origin MIT Engineer Secured Google's $2.4 Billion AI Investment](https://timesofindia.indiatimes.com/education/news/varun-mohan-education-qualifications-how-an-indian-origin-mit-engineer-secured-googles-2-4-billion-ai-investment/articleshow/122405531.cms) | Times of India | 2025-07 | Authoritative Media Report | High | `dossier.md` §2, §3.1, §8; Volume 1 |
| `[57]` (`[^57]`) | [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) | Google AI Developers Forum / `@Sharper` (Official Reply: `@chunduriv`) | 2026-11-04 | Primary Official Staff Forum Reply | High | `dossier.md` §3.2, §8, §9, §10; Volumes 1, 2, 4, 5 |

---

## 2. Detailed Cited Body Sources (Strictly Aligned One-to-One With `[^1]`..`[^57]` Across `dossier.md` and All Volumes)

### [1] [Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245) (Corresponds to `[^1]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/issues/1245
- **Organization / Author**: GitHub `google-antigravity/antigravity-cli` official repository | Reporter: `@Burntgogi`; Core Technical Forensic Commenters: `@NuCl34R`, `@Circumsized`, `@Emily2040`, `@LiraNuna`, `@AsabaLazy`; Assigned Maintainer: `@chandrashakherkamasani`
- **Publication Date**: 2026-10-14 (Last updated: 2026-11-02) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official code repository issue tracker & binary reverse-engineering proof) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > - `@Burntgogi` (OP): "During multi-phase TypeScript refactoring and application scaffolding runs on `agy 1.3.5` with **Gemini 4 Argon** (`1,048,576` token context window), `antigravity-cli` triggers hard context compactions (`CHECKPOINT: "# Resuming from a compaction..."`) at **~250K–276K tokens** — less than 27% of the model's 1M context window... Checkpoint 1 (Step 79): 276,110 -> 17,336 (`cache_read_tokens`: 0); Mid-run prefix miss (Step 193): 239,857 -> 243,064 (`cache_read_tokens`: 0); Checkpoint 2 (Step 200): 249,867 -> 15,363 (`cache_read_tokens`: 0)."
  > - `@Circumsized` (Comment #2): "Every model call writes a `ContextWindowMetadata` record into the conversation database at `~/.gemini/antigravity-cli/conversations/<session-id>.db`... `max_context_tokens = 256000`, `truncation_reason = 2`... 256,000 is exactly 25% of 1,048,576 — a round fraction of the window, which points at a policy constant rather than a technical ceiling."
  > - `@LiraNuna` (Comments #5 & #6): "1. `SelectorModel.offersContextWindow`: hardcoded to only contain `'Gemini 4 Argon'`. 2. `os.Getenv('AGY_CLI_AUTO_COMPACT_OVERRIDE')` was compiled out / inlined into an empty string `""`. 3. `Store.ContextWindow()` (`0x917d892`) drops the selection for non-Argon models. 4. `cortex/utils/utils.GetCheckpointConfig` (`0x7861894`): `cmp $0x3e800, %edx; mov $0x3e800, %esi; cmovb %edx, %esi; mov %esi, 0x50(%rsp)`."

### [2] Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`) (Corresponds to `[^2]`)
- **Original URL**: https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26
- **Organization / Author**: GitHub Gist / `@LiraNuna`
- **Publication Date**: 2026-10-23 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Reverse-engineering patch source code) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Parses ELF `.text` section headers and the Go runtime `pclntab` symbol table to dynamically locate `google3/third_party/jetski/cli/model/model.SelectorModel.offersContextWindow`, `google3/third_party/jetski/cli/store/store.(*Store).ContextWindow`, `store.(*Store).ContextWindowChoice`, `google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig`, and `store.AutoCompactOverride`, and modifying three code locations: forcing the context tier selector to appear for all models, routing `ContextWindow` to `ContextWindowChoice`, and removing the 256,000-token clamp in `GetCheckpointConfig` (original bytes `81fa00e80300be00e803000f42f2`). The script itself is not reproduced in this archive.

### [3] [Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878) (Corresponds to `[^3]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/issues/878
- **Organization / Author**: GitHub `google-antigravity/antigravity-cli` / `@schalatap`, Eddy Murias (`@murias002`), `@Corsagee`, `@Infinnerty`, `@jlukic` (Assigned maintainer: `@manirajc`)
- **Publication Date**: 2026-08-26 (Updated through 2026-10-31) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official repository issue & large-scale database statistics) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@murias002`: "Conversations: 310 | Model calls: 65,613 (86.6% on Gemini 3.8 Flash) | Input tokens per call: p50 136,487 · p90 229,000 · max 255,999 | Calls above 300k input tokens: 0 | Compaction events (drop > 40%): 276 | Context right before compaction: median 255,070 (p10 247,360 · p90 255,874) | Context right after compaction: median 45,304."

### [4] [Bug]: Five measured places the CLI spends time and tokens the work does not need (Issue #1291) (Corresponds to `[^4]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/issues/1291
- **Organization / Author**: GitHub `google-antigravity/antigravity-cli` / `@Emily2040` (Assigned maintainer: `@chandrashakherkamasani`)
- **Publication Date**: 2026-10-19 (Updated through 2026-10-25) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official repository issue & 1,279-call empirical report) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "all model calls: uncached input 10,619,129 | cache_read 192,374,637. 17 calls with input > 100k and cache_read 0: sum 3,292,077 = 31.0 % of all uncached input... 15 of the 17 came within two steps of a `system_message` step... 22 `checkpoint` steps in the same logs: durations (s): 35..602, median 70; checkpoint step usage: none reported; next call: input 44,990–70,041, cache_read 0."

### [5] [Feature Request]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue #1072) (Corresponds to `[^5]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/issues/1072
- **Organization / Author**: GitHub `google-antigravity/antigravity-cli` / `@flotoz`, `@Emily2040` (Assigned maintainer: `@manirajc`)
- **Publication Date**: 2026-09-20 (Updated through 2026-10-19) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official repository issue) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "Even on Gemini 3.8 Flash Low, whose per-token price is very low, my usage burns through the 5-hour and weekly rate-limit windows roughly 3x faster than I'd expect from token cost alone."

### [6] Feature Request: Add `max_context_tokens` setting to control per-turn token budget (Issue #743) (Corresponds to `[^6]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/issues/743
- **Organization / Author**: GitHub `google-antigravity/antigravity-cli` / `@thealgebraist`
- **Publication Date**: 2026-08-03 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official repository issue) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "379 steps, 610 KB of transcript, Estimated 500K–1M+ total input+output tokens, ~70% of tokens were redundant compiler error output from earlier failed iterations."

### [7] Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue #366) (Corresponds to `[^7]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/issues/366
- **Organization / Author**: GitHub `google-antigravity/antigravity-cli` / `@UesugiErii`, `@Mandark-droid`, `@evilfurryone`, `@rajatim`, `@Emily2040`
- **Publication Date**: 2026-06-11 (Updated through 2026-10-20) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official repository issue) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Documents how `agy` stripped out the native OTLP telemetry export interface previously available in `gemini-cli`, details the `gen_metadata` / `steps` Protobuf BLOB schemas inside `~/.gemini/antigravity-cli/conversations/<id>.db`, and provides a daemonless OTLP export script powered by the `PostInvocation` hook.

### [8] agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync (Issue #927) (Corresponds to `[^8]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/issues/927
- **Organization / Author**: GitHub `google-antigravity/antigravity-cli` / `@jlwainwright`
- **Publication Date**: 2026-09-02 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official repository issue) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "Eligibility check failed: PERMISSION_DENIED (code 403): This service has been disabled in this account for violation of Terms of Service... Suspended since at least 25 May 2026 (Google One case 2-9245000040533, which cannot escalate ToS blocks)... Nothing has happened in 100+ days."

### [9] Addressing Antigravity Bans & Reinstating Access (Discussion #20632) (Corresponds to `[^9]`)
- **Original URL**: https://github.com/google-gemini/gemini-cli/discussions/20632
- **Organization / Author**: GitHub `google-gemini/gemini-cli` / Google Maintainer `@jackwotherspoon`
- **Publication Date**: 2026-02-27 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google announcement) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "Because of the backend layer where abuse prevention occurs, bans for Antigravity usage also blocked access to Gemini CLI and Gemini Code Assist... 1. User notification... 2. Recertification... 3. Automatic unban: within a day or two. 4. Permanent ban: If an account is flagged for a second violation of ToS, it will be permanently banned."

### [10] Appeal submitted but TOS_VIOLATION suspension persists (request for private follow-up) (Issue #29576) (Corresponds to `[^10]`)
- **Original URL**: https://github.com/google-gemini/gemini-cli/issues/29576
- **Organization / Author**: GitHub `google-gemini/gemini-cli` / `@myessaylab1234`
- **Publication Date**: 2026-09-30 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official repository issue) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "The backend response is HTTP 403 `PERMISSION_DENIED`, reason `TOS_VIOLATION`, from `cloudcode-pa.googleapis.com`... Asked Google One Support, which could not view the appeal status. Google Developer Profile Support confirmed this is outside its scope. I understand that consumer access to the legacy Gemini CLI ended on June 18, 2026."

### [11] Google Antigravity CLI Official Releases (`v1.4.2` – `v1.5.0`) (Corresponds to `[^11]`)
- **Original URL**: https://github.com/google-antigravity/antigravity-cli/releases
- **Organization / Author**: Google (`google-antigravity/antigravity-cli`)
- **Publication Date**: 2026-10-18 to 2026-11-04 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official release notes) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Covers `v1.4.3` (fixing lost subagent IDs after compaction), `v1.4.4` (retaining tool declarations in compaction requests to reuse prompt cache prefixes), `v1.4.6` (preserving images and PDFs across compaction summaries), `v1.4.7` (fixing infinite compaction loops caused by oversized single messages), `v1.4.9` (fixing orphaned tool calls/results left behind after history trimming), `v1.4.10` (fixing subagents losing parent tasks and parent IDs after compaction, and improving post-compaction size accounting in `/context`), and `v1.5.0` (adding `agy acp` and `--remote-agent` on 2026-11-04).

### [12] Google Antigravity Docs — Changelog (`v2.0.1` – `v2.29.0`) (Corresponds to `[^12]`)
- **Original URL**: https://antigravity.google/docs/changelog
- **Organization / Author**: Google Antigravity Official Documentation
- **Publication Date**: 2026-11-03 (Verified 2026-11-04) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official documentation) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Documents the full version evolution of the Antigravity App from `v2.0.1` through `v2.29.0` (2026-11-03), including `v2.24.0` (2026-10-15 launch of `/compact` and `Gemini 4 Argon`), `v2.27.0` (10-26), `v2.28.0` (10-27), `v2.28.1` (10-28), `v2.28.2` (10-29), and `v2.29.0` (11-03).

### [13] Google Antigravity Docs — Official FAQ (Corresponds to `[^13]`)
- **Original URL**: https://antigravity.google/docs/faq/
- **Organization / Author**: Google Antigravity Official Documentation
- **Publication Date**: Updated mid-October 2026 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official documentation) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "Why can’t I use third-party software (such as Claude Code, OpenClaw, or OpenCode) with my Antigravity login? Using third-party software, tools, or services to access Antigravity is a violation of our Terms of Service... Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching? Antigravity CLI headless mode (`agy -p`) is intended for direct scripting and CI workflows. Wrapping `agy` inside third-party coding agents or applications, or automating account rotation, is not supported under consumer plans."

### [14] Google Antigravity — Additional Terms of Service (Corresponds to `[^14]`)
- **Original URL**: https://antigravity.google/terms
- **Organization / Author**: Google LLC
- **Publication Date**: 2026-09 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official legal terms) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "6. You must not abuse, harm, interfere with, or disrupt the Service. This includes, but is not limited to, using the Service in connection with products not provided by us. Using third party software, tools, or services to access the Service (e.g. using OpenClaw with Antigravity OAuth) is a breach of this Agreement. Such actions may be grounds for suspension or termination of your Antigravity and/or Gemini CLI accounts."

### [15] Google Antigravity Docs — Workspace Rules (24 KB Per-File Limit & 20,000-Token Aggregate Rules Budget) (Corresponds to `[^15]`)
- **Original URL**: https://antigravity.google/docs/rules/
- **Organization / Author**: Google Antigravity Official Documentation (and local built-in `agy-customizations/docs/rules.md`)
- **Publication Date**: 2026-10 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official documentation) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "Per-File Limit (`24 KB` / `24,000` bytes): Each rule file is capped at 24,000 bytes (after expanding `@[label](path)` includes) and truncated on line boundaries when over the cap. Aggregate Rules Budget (`20,000` tokens): Always-on and global rules share a dedicated 20,000-token rules budget (`defaultRulesBudget`)... Over-budget rules are demoted from full inline text to file path pointers so the agent can read them on demand."

### [16] Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon (Corresponds to `[^16]`)
- **Original URL**: https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/
- **Organization / Author**: Google Developers Blog
- **Publication Date**: 2026-10 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google technical blog) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "The Python repository (`google/adk-python`) spans roughly 205,000 lines of code across nearly 800 source files, backed by nearly 17,000 unit tests... First, we put the team's maintainer guidelines into the repository's `.agents/skills` directory... Each skill starts with a short `SKILL.md` index that points to focused `references/` files on demand, so we don't flood the agent's context window on every turn... Second, we turned on `/goal` mode in Google Antigravity, which preserves the goal across context compactions so the agent keeps running until the task is finished. With `.agents/skills` in the repo and `/goal` mode active, Argon ran unattended for 11 hours overnight."

### [17] [Teardown / Binary Diff] Antigravity Updated to v2.29.0 (Corresponds to `[^17]`)
- **Original URL**: https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/
- **Organization / Author**: Reddit `r/google_antigravity` / `u/Lberondroth`
- **Publication Date**: 2026-11-03 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community (Primary binary diff teardown report) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `language_server.exe` grew by `+4.47 MB`, embedded Web UI grew by `+36%`, adding underlying packages `cortex/tokens/compaction` and `cortex/shared/snapshot` alongside the gated feature flag `enable-context-usage-indicator`: "What you get instead is the Context and contingent Indicator, so you can soon watch your contingent drain in high definition, right next to the send button..."

### [18] [Deep-Dive] Antigravity v2.27.0 → v2.28.1 Binary & ASAR Diff (Corresponds to `[^18]`)
- **Original URL**: https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/
- **Organization / Author**: Reddit `r/google_antigravity` / `u/Lberondroth`
- **Publication Date**: 2026-10-28 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community (Primary binary & ASAR teardown report) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Dissects Protobuf and binary structural changes from `v2.27.0` to `v2.28.1`, uncovering `ContextWindowOptions` (`256K`, `512K`, `900K`) and the `Expensive` boolean flag.

### [19] Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon? (`r/AntigravityGoogle`) (Corresponds to `[^19]`)
- **Original URL**: https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/
- **Organization / Author**: Reddit `r/AntigravityGoogle`
- **Publication Date**: 2026-11-02 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community Discussion | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Synthesizes public community demands for an official timeline on enabling the `512K` (`1.3x`) and `900K` (`1.8x Expensive`) tiers in the desktop app and lifting the `256K` clamp inside `GetCheckpointConfig`.

### [20] Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon? (`r/google_antigravity`) (Corresponds to `[^20]`)
- **Original URL**: https://www.reddit.com/r/google_antigravity/comments/1xmc89l/timeline_for_enabling_the_512k_900k/
- **Organization / Author**: Reddit `r/google_antigravity`
- **Publication Date**: 2026-11-02 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community Discussion | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Cross-posted concurrently with `[19]` to the main `r/google_antigravity` subreddit; as of 2026-11-04, the official team has provided no timeline response in the thread.

### [21] Antigravity's aggressive history compaction is ruining Gemini's 1M+ context selling point (Corresponds to `[^21]`)
- **Original URL**: https://www.reddit.com/r/google_antigravity/comments/1trr4dy/antigravitys_aggressive_history_compaction_is/
- **Organization / Author**: Reddit `r/google_antigravity`
- **Publication Date**: 2026-05-30 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community Discussion | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Early community report documenting how Antigravity executes aggressive history summarization well below the 1M context limit, undermining Gemini's flagship 1M+ long-context advantage.

### [22] How to force AG to use subagents? (Orchestrator + Ephemeral Subagents) (Corresponds to `[^22]`)
- **Original URL**: https://www.reddit.com/r/google_antigravity/comments/1uo7g2k/how_to_force_ag_to_use_subagents/
- **Organization / Author**: Reddit `r/google_antigravity`
- **Publication Date**: 2026-07 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community Engineering Practice | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Details the engineering pattern of using `AGENTS.md` to force the main agent to act strictly as an Orchestrator, spawning short-lived Ephemeral Subagents and handing off state via local `.md` files to prevent monolithic main-thread context bloat.

### [23] Clarification Request: Is local IPC/PTY wrapping of `agy` CLI for open-standard editor bridges (e.g., ACP) ToS compliant? (#175905) (Corresponds to `[^23]`)
- **Original URL**: https://discuss.ai.google.dev/t/clarification-request-is-local-ipc-pty-wrapping-of-agy-cli-for-open-standard-editor-bridges-e-g-acp-tos-compliant/175905
- **Organization / Author**: Google AI Developers Forum / Question: `@shindgew`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-07-23 (Official reply: 2026-09-16 12:45:42 UTC) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Rebbanpalli_Naveen`: "1. ToS Compliance (Local CLI Wrapping): Yes, your described pattern is compliant. The ToS restriction on third-party access tools is aimed at credential harvesting, unofficial API reverse-engineering, and multi-tenant proxying. Spawning the official, unmodified `agy` binary locally as a single-user child process/PTY under the user’s own `agy login` session complies with single-user usage, provided tokens are not extracted, stored, or proxied."

### [24] Is it allowed for an open-source tool to call the official `agy` CLI in headless mode? (#183429) (Corresponds to `[^24]`)
- **Original URL**: https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429
- **Organization / Author**: Google AI Developers Forum / Question: `@darkest_alex`; Official Reply: `@Ambati_Rajendra`
- **Publication Date**: 2026-09-17 12:06:16 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra`: "Is calling `agy` headless allowed? Yes. Calling the official `agy` binary locally via `agy -p` is an officially supported feature built for programmatic scripting (see the Headless Mode Documentation). It does not violate Section 6, provided your tool never extracts, stores, or intercepts the user’s OAuth credentials."

### [25] Is using the official agy CLI through a local MCP server with third-party AI agents permitted? (#184829) (Corresponds to `[^25]`)
- **Original URL**: https://discuss.ai.google.dev/t/is-using-the-official-agy-cli-through-a-local-mcp-server-with-third-party-ai-agents-permitted/184829
- **Organization / Author**: Google AI Developers Forum / Question: `@snowstone`; Official Reply: `@Ambati_Rajendra`
- **Publication Date**: 2026-09-25 12:47:46 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra`: "Third-Party Agent MCP Wrapper: Wrapping `agy -p` in a local MCP server to use with third-party agents (like Claude Code or Codex CLI) is not permitted."

### [26] One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? (#185460) (Corresponds to `[^26]`)
- **Original URL**: https://discuss.ai.google.dev/t/one-person-several-paid-google-ai-pro-accounts-may-a-local-orchestrator-use-them-with-the-official-agy-cli/185460
- **Organization / Author**: Google AI Developers Forum / Question: `@Winez`; Official Reply: `@Ambati_Rajendra`
- **Publication Date**: 2026-09-28 12:22:17 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra`: "While we permit running the official `agy` CLI locally as a subprocess for your own personal use (as long as you are using your single personal account and interacting with it directly), using an orchestrator to cycle through multiple Google AI Pro accounts to bypass quota limits, pool capacity, or distribute tasks is a direct violation of the Terms of Service."

### [27] Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (#185494) (Corresponds to `[^27]`)
- **Original URL**: https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494
- **Organization / Author**: Google AI Developers Forum / Question: `@Diggy_Styon`; Official Reply: `@Ambati_Rajendra`
- **Publication Date**: 2026-09-28 07:14:31 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra`: "Yes, your use case is permitted. Running the official `agy` binary locally (even headless via a script) under your own account is compliant with the terms... As long as you are executing the unmodified `agy` CLI and not proxying your auth tokens into another application, you are good to go!"

### [28] Is invoking the official `agy` CLI from a local AI coding assistant allowed on an individual Google AI account? (#185992) (Corresponds to `[^28]`)
- **Original URL**: https://discuss.ai.google.dev/t/is-invoking-the-official-agy-cli-from-a-local-ai-coding-assistant-allowed-on-an-individual-google-ai-account/185992
- **Organization / Author**: Google AI Developers Forum / Question: `@Ivan_Valenzuela`; Official Reply: `@chunduriv`
- **Publication Date**: 2026-09-30 23:25:25 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@chunduriv`: "Is invoking the unmodified `agy.exe` binary in headless mode from a local assistant permitted on an individual account? Yes. Section 6 of the Antigravity Additional Terms of Service (and our FAQ regarding third-party tools) prohibits extracting, proxying, or reusing Antigravity OAuth credentials inside third-party harnesses to call Antigravity’s backend endpoints directly."

### [29] Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper (#185949) (Corresponds to `[^29]`)
- **Original URL**: https://discuss.ai.google.dev/t/clarification-on-codex-claude-skill-invoking-the-official-agy-cli-through-a-local-powershell-wrapper/185949
- **Organization / Author**: Google AI Developers Forum / Question: `@nothing2`; Official Reply: `@Ambati_Rajendra`
- **Publication Date**: 2026-10-02 20:20:25 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra`: "Short answer: yes, your setup is fine. What’s not allowed is extracting sign-in tokens, calling private Antigravity endpoints, or using `agy` as a raw model backend for another agent. Running the official `agy` CLI locally to do a specific task and reading its output is allowed. That’s the same rule behind all the forum answers you found. To your questions: Codex or Claude calling the Skill: Yes, as long as it sends `agy` a real task..."

### [30] Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? (#186954) (Corresponds to `[^30]`)
- **Original URL**: https://discuss.ai.google.dev/t/is-automatically-launching-the-official-agy-cli-from-a-local-ai-orchestrator-allowed-under-additional-terms-6/186954
- **Organization / Author**: Google AI Developers Forum / Question: `@local_ai_builder`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-10-05 10:17:42 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Rebbanpalli_Naveen`: "Yes, the single-user local workflow you described is supported through the official CLI Headless Mode. To answer your questions directly: Local Child Process vs. Section 6 (Questions 1 & 3): Yes, this is permitted and materially different."

### [31] Is invoking the official Antigravity CLI from Codex allowed? (#186952) (Corresponds to `[^31]`)
- **Original URL**: https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952
- **Organization / Author**: Google AI Developers Forum / Question: `@Hossy`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-10-05 10:24:45 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Rebbanpalli_Naveen`: "Yes, the workflow you described is completely fine to use with your personal Google AI Pro subscription... Having Codex invoke the official, unmodified `agy.exe` binary as a local process on your Windows PC—while authentication (`agy auth login`) and all backend communication stay strictly inside `agy.exe`—is a supported use of CLI Headless Mode."

### [32] Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (#186246) (Corresponds to `[^32]`)
- **Original URL**: https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246
- **Organization / Author**: Google AI Developers Forum / Question: `@Croixofwar`; Official Reply: `@chunduriv`; Follow-up questions: `@Kevin_Lopez`, `@Lorde_Lorde`, `@Ripu_Yadav`, `@AbMedhat`
- **Publication Date**: 2026-10-01 (Official reply: 2026-10-07 15:03:46 UTC; updated through 2026-11-02) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@chunduriv` (2026-10-07 15:03:46 UTC): "1. Is launching the official `agy` CLI as a local subprocess allowed under Section 6? Yes... 2. Does the answer change if you launch the script yourself vs. Claude Code launching it? No. From a policy standpoint, there is no difference whether the unmodified `agy` subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code)..."

### [33] Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator (#187536) (Corresponds to `[^33]`)
- **Original URL**: https://discuss.ai.google.dev/t/terms-clarification-personal-antigravity-cli-integration-with-a-multi-ai-orchestrator/187536
- **Organization / Author**: Google AI Developers Forum / Question: `@BuddhaRyu`; Official Reply: `@Ambati_Rajendra`
- **Publication Date**: 2026-10-08 16:23:34 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra`: "Running `agy` from your orchestrator: Yes. Running the official `agy` program from your app (e.g. `agy -p '<prompt>' --output-format json` or `stream-json`) for your own development work is fine... Handoffs between `agy`, Codex and Claude Code: Yes... Does section 6 apply? No."

### [34] Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241) (Corresponds to `[^34]`)
- **Original URL**: https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241
- **Organization / Author**: Google AI Developers Forum / Question: `@ALAMDAR_BALOCH`; Official Reversal Reply: `@chunduriv`
- **Publication Date**: 2026-10-07 (Official reply: 2026-10-08 17:34:41 UTC) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@chunduriv` (2026-10-08 17:34:41 UTC): "While `agy` offers a documented headless mode (`agy -p`) for your own local scripts and CI pipelines, we do not support using third-party coding agents (like Codex, Claude Code, OpenClaw, or OpenCode) to orchestrate or wrap `agy` under a consumer login. Routing third-party agent prompts or automated loops through Antigravity can trigger automated ToS enforcement and account suspension."

### [35] Clarification on consumer-plan policy for invoking official agy via local subprocess (#189905) (Corresponds to `[^35]`)
- **Original URL**: https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905
- **Organization / Author**: Google AI Developers Forum / Questions: `@Amane2418`, `@Jay10`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-10-16 12:35:16 UTC (Updated through 2026-10-26) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Rebbanpalli_Naveen`: "Current Official Guidance: The Antigravity FAQ reflects our current policy for Antigravity CLI headless mode and supersedes earlier forum comments where they differ... Even if you invoke the official, unmodified `agy` binary as a local subprocess without extracting OAuth tokens or reverse-engineering internal endpoints, wrapping `agy` inside third-party coding agents or applications—or automating account rotation—is not supported under consumer plans... violates our Terms of Service and can result in account suspension or termination."

### [36] Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689) (Corresponds to `[^36]`)
- **Original URL**: https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689
- **Organization / Author**: Google AI Developers Forum / Questions & Challenges: `@eliavb`, `@b_w`, `@Tora39`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-10-23 to 2026-10-27 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Rebbanpalli_Naveen` (Post #8, 2026-10-27 12:31:42 UTC): "To clarify the timeline for `@b_w` and answer `@Tora39`’s question (a): the Google Antigravity Terms of Service themselves were not modified after September; rather, the Official FAQ was updated in mid-October 2026 to add the section ‘Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?’... Earlier forum replies in #186246 and #186952 gave an overly broad interpretation before that FAQ entry was published."

### [37] Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998) (Corresponds to `[^37]`)
- **Original URL**: https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998
- **Organization / Author**: Google AI Developers Forum / Question: `@Herzchen`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-10-26 13:54:18 UTC | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Distinguishes static file drafting from automated closed-loop orchestration: if an external AI automatically writes requests, automatically triggers `agy -p`, and reads the results in a closed loop, it violates policy; it only falls within the permitted direct-CI boundary when explicitly and manually triggered by the human owner (e.g., manual `workflow_dispatch`) without an external AI coupled into an automated loop.

### [38] Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815) (Corresponds to `[^38]`)
- **Original URL**: https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815
- **Organization / Author**: Google AI Developers Forum / Question: `@Lars`; Official Reply: `@Ambati_Rajendra`
- **Publication Date**: 2026-10-19 (Official reply: 2026-10-28 13:10:46 UTC) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra`: "That still counts as third-party agent coordination. Passing tasks from Claude Code to `agy -p` through local files instead of stdin/stdout pipes still uses consumer quota as a backend worker for an external coding agent, so it requires a Gemini API key or Gemini Enterprise."

### [39] Policy clarification: private monitoring app using official Antigravity CLI and subscription quota (#190756) (Corresponds to `[^39]`)
- **Original URL**: https://discuss.ai.google.dev/t/policy-clarification-private-monitoring-app-using-official-antigravity-cli-and-subscription-quota/190756
- **Organization / Author**: Google AI Developers Forum / Question: `@xorz`; Official Reply: `@Ambati_Rajendra`; Technical Challenge: `@oli_hlmpn`
- **Publication Date**: 2026-10-19 to 2026-10-29 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply & technical inquiry) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Ambati_Rajendra` bans unattended automatic background requests while documenting that disabling overage credits requires configuring both `"useAiCredits": false` in `~/.gemini/config/config.json` and `"useG1Credits": false` in `~/.gemini/antigravity-cli/settings.json`; `@oli_hlmpn` highlights that it is impossible at the OS level to distinguish whether a process was launched by a local Go/Python server or a 3-line Bash script, and that permitting third-party calls to `acp` while permanently banning `agy -p` creates a severe contradiction.

### [40] Clarification on subscription-based agy -p integration with a local multi-agent framework (#196472) (Corresponds to `[^40]`)
- **Original URL**: https://discuss.ai.google.dev/t/clarification-on-subscription-based-agy-p-integration-with-a-local-multi-agent-framework/196472
- **Organization / Author**: Google AI Developers Forum / Question: `@Dsssyc`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-10-28 to 2026-10-29 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Rebbanpalli_Naveen`: "Having an intermediate local Python coordination service spawn and manage the `agy -p` process (rather than the Host or Router Agent directly) does not change this restriction, as `agy` is still being used as an execution backend for an external multi-agent framework under a consumer sign-in."

### [41] Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131) (Corresponds to `[^41]`)
- **Original URL**: https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131
- **Organization / Author**: Google AI Developers Forum / Questions: `@sannikov`, `@tkosub`, `@oli_hlmpn`, `@christian-taillon`; Official Reply: `@Rebbanpalli_Naveen`
- **Publication Date**: 2026-10-31 to 2026-11-02 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Rebbanpalli_Naveen` confirms that launching the official Google-signed `antigravity-acp` binary inside ACP-compatible third-party editors/clients using personal OAuth sign-in (for eligible Gemini models in the plan) complies with policy.

### [42] Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351) (Corresponds to `[^42]`)
- **Original URL**: https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351
- **Organization / Author**: Google AI Developers Forum / Author: `@Yiji_Kong`; Reproductions: `@RobW`, `@shuai_guo`; Official Reply: `@Maniraj_Chervirala`
- **Publication Date**: 2026-10-28 to 2026-10-29 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Controlled telemetry benchmark & official staff reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "Normalized by each model’s published API price, Argon consumes exactly 3.0x more quota per dollar than Flash. If Flash is billed at its published rate, the implied bucket size is ~150 USD, and Argon’s consumption matches 6.00 / 30.00 USD per 1M input/output tokens (3x the advertised 2.00 / 10.00 launch price)... 1 Weekly Pool = 20 × 5-Hourly Pool... In Step 1C, sending just 'Reply with only the word: OK' (28 chars) in a fresh chat on 4 Argon (Low) burned +0.00015849 of weekly quota, which is 12% more than 3.8 Flash (High) writing a full 4,013-word technical article on Paxos and Raft (+0.00014157)."

### [43] Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476) (Corresponds to `[^43]`)
- **Original URL**: https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476
- **Organization / Author**: Google AI Developers Forum / Question: `@Hello_there`; Official Reply: `@chunduriv`
- **Publication Date**: 2026-10-15 (Official reply: 2026-10-20 23:03:17 UTC) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Hello_there` reports the agent fabricating 6 non-existent identifiers and overwriting initialization hard limits after compaction; `@chunduriv` provides the official workaround: add standing rules in `~/.gemini/GEMINI.md` or `.agents/rules/` forcing the agent to re-read target files via `view_file` upon encountering `# Resuming from a compaction`, and run `/compact Preserve exact contract identifiers, initialization limits and inspected file paths verbatim`.

### [44] Feature request: Show context window usage in Antigravity Desktop (#184694) (Corresponds to `[^44]`)
- **Original URL**: https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694
- **Organization / Author**: Google AI Developers Forum / Question: `@Soro`; Official Reply: `@chunduriv`; Extension Author: `@dimitr-io`
- **Publication Date**: 2026-09-24 to 2026-10-22 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Official forum reply & empirical screenshot) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@chunduriv` (2026-09-25): "If you ever use the Antigravity CLI, it already has this. Just type `/context` and it shows how much of the context window you’re using..."; `@dimitr-io` (2026-10-22) releases the `antigravity-desktop-context` sidebar extension with a side-by-side comparison screenshot.

### [45] Persistent Gemini CLI TOS_VIOLATION after appeal — request for private review or sync check (#186089) (Corresponds to `[^45]`)
- **Original URL**: https://discuss.ai.google.dev/t/persistent-gemini-cli-tos-violation-after-appeal-request-for-private-review-or-sync-check/186089
- **Organization / Author**: Google AI Developers Forum / `@myessaylab1234`
- **Publication Date**: 2026-09-30 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community Report | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Confirms that individual consumer access to the legacy `@google/gemini-cli` ended on June 18, 2026, and that `TOS_VIOLATION` suspensions returned by `cloudcode-pa.googleapis.com` remain unsynchronized for extended periods after appeal submission.

### [46] Antigravity CLI Agent reliability issue (#176775) (Corresponds to `[^46]`)
- **Original URL**: https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775
- **Organization / Author**: Google AI Developers Forum / `@Pitt_zhang`
- **Publication Date**: 2026-08-01 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Specialist Community Audit Report | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Documents a case where `agy` CLI forgot project instructions during a complex multi-step engineering run, claimed completion without verification, mismatched frontend/backend API paths, and introduced an authentication-bypass fallback.

### [47] Google Antigravity FAQ Policy Update and Termination of Third-Party Tool Integration (Corresponds to `[^47]`)
- **Original URL**: https://note.com/horizon_it00/n/nded285d721d1
- **Organization / Author**: note.com / `horizon_it00` (Kenji Sato)
- **Publication Date**: 2026-10 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Technical Blog | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Documents how Japanese developers were forced to shut down existing cross-tool orchestration architectures after Google updated the Antigravity FAQ in October 2026 to ban wrapping or invoking `agy -p` from third-party coding agents.

### [48] Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (`239,616` Compaction Threshold) (Corresponds to `[^48]`)
- **Original URL**: https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk
- **Organization / Author**: Nous Research Hermes Agent Documentation
- **Publication Date**: 2026-09-23 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Technical Documentation | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Documents setting the external plugin safety compaction threshold at `239,616` tokens (`256K - 16K output tokens`) specifically to dodge Antigravity's internal 256K hard ceiling.

### [49] mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management (Corresponds to `[^49]`)
- **Original URL**: https://github.com/mksglu/context-mode
- **Organization / Author**: GitHub / `@mksglu`
- **Publication Date**: 2026 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Open-Source Code Repository | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Intercepts raw file reads and long command outputs inside a local sandbox backed by a SQLite FTS5 full-text index, returning only queried, high-precision snippets to the agent and reducing context consumption by 70%–90%.

### [50] AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE (Corresponds to `[^50]`)
- **Original URL**: https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor
- **Organization / Author**: GitHub / `@AGI-is-going-to-arrive`
- **Publication Date**: 2026 (Updated through `v1.13.0`) | **Verification Access Date**: 2026-11-04
- **Source Tier**: Open-Source Code Repository | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Supports macOS, Linux, Windows, and WSL (`v1.12.0+` / `v1.13.0`), automatically discovering local `agy` Hub and language server processes to monitor context window water levels across all active conversations in real time.

### [51] dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app (Corresponds to `[^51]`)
- **Original URL**: https://github.com/dimitr-io/antigravity-desktop-context
- **Organization / Author**: GitHub / `@dimitr-io`
- **Publication Date**: 2026-10-22 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Open-Source Code Repository | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "this panel reads the token breakdown that Antigravity records for every model call, the same data the CLI's `/context` uses. The accounting is ported from the CLI binary itself, so the categories, totals, checkpoints and files match `agy /context` on the same conversation."

### [52] weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI (Corresponds to `[^52]`)
- **Original URL**: https://github.com/weby-homelab/antigravity-cli-statusline
- **Organization / Author**: GitHub / Weby Homelab
- **Publication Date**: 2026 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Open-Source Code Repository | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "The Bash renderer supports Linux and macOS; the PowerShell renderer supports Windows. Both read the CLI’s JSON payload from standard input and supplement it with local Git and host data when available."

### [53] oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit ("The IDE for Your Coding Agent") (Corresponds to `[^53]`)
- **Original URL**: https://github.com/oraios/serena
- **Organization / Author**: GitHub / Oraios AI
- **Publication Date**: 2026 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Open-Source Code Repository | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "Serena provides essential semantic code retrieval, editing, refactoring and debugging tools that are akin to an IDE's capabilities, operating at the symbol level and exploiting relational structure. It integrates with any client/LLM via the model context protocol (MCP)."

### [54] xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server) (Corresponds to `[^54]`)
- **Original URL**: https://github.com/xdotech/goatlas
- **Organization / Author**: GitHub / `@xdotech`
- **Publication Date**: 2026 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Open-Source Code Repository | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > "**GoAtlas** is an AI-powered code intelligence platform that helps LLMs and developers deeply understand large codebases — combining multi-language AST parsing, a Neo4j knowledge graph, pgvector semantic search, and a pluggable LLM backend, all exposed via MCP."

### [55] Kevin Hou — "Get Out of the Model's Way" (AI Engineer Summit Presentation & Speaker Profile) (Corresponds to `[^55]`)
- **Original URL**: https://www.youtube.com/watch?v=xsVD9_cJNYs
- **Organization / Author**: AI Engineer Summit / YouTube (and speaker profile `https://ai.engineer/speakers/kevin-hou`, personal site `https://khou22.com`) | Speaker: Kevin Hou (Head of Product Engineering, Google DeepMind Antigravity, `@kevinhou22` / GitHub `@khou22`)
- **Publication Date**: 2026 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Primary Source (Executive public keynote) | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Articulates Antigravity's core design philosophy *"Get Out of the Model's Way"*: avoid building rigid scaffolding structures around the model that constrain it; as base models improve, over-engineered hard limits become depreciating assets, and engineers should let the model plan and breathe freely.

### [56] Varun Mohan Education Qualifications: How an Indian-Origin MIT Engineer Secured Google's $2.4 Billion AI Investment (Corresponds to `[^56]`)
- **Original URL**: https://timesofindia.indiatimes.com/education/news/varun-mohan-education-qualifications-how-an-indian-origin-mit-engineer-secured-googles-2-4-billion-ai-investment/articleshow/122405531.cms
- **Organization / Author**: Times of India
- **Publication Date**: 2025-07 | **Verification Access Date**: 2026-11-04
- **Source Tier**: Authoritative Media Report | **Confidence**: High
- **Key Verbatim Excerpts & Statistical Scope**:
  > Documents Google's July 2025 $2.4 billion technology licensing and talent deal with Windsurf (Codeium), under which co-founder and CEO Varun Mohan and his core engineering team joined Google DeepMind to lead agentic coding platform development.

### [57] Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191) (Corresponds to `[^57]`)
- **Original URL**: https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191
- **Organization / Author**: Google AI Developers Forum / Question: `@Sharper`; Official Reply: `@chunduriv` (Veeranjaneyulu Ch, forum staff & moderator)
- **Publication Date**: 2026-11-04 16:01:05 UTC (Official reply: 2026-11-04 23:39:13 UTC) | **Verification Access Date**: 2026-11-05
- **Source Tier**: Primary Source (Official Google staff forum reply) | **Confidence**: High
- **Raw Snapshot**: [`evidence/raw/forum_198191.json`](../evidence/raw/forum_198191.json) (Discourse topic JSON, accessed 2026-11-05)
- **Key Verbatim Excerpts & Statistical Scope**:
  > `@Sharper`: "In practice, the desktop app forces cache-losing and information-lossy compaction at 250K of used context... Choosing the context compaction size must be at least an option for users."; `@chunduriv`: "This is a feature request for configurable context compaction in the Antigravity App. As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. That threshold is a fixed default and isn't user-configurable yet. ... We've passed your request for a user-selectable compaction size to the engineering team for review." Interim mitigations offered: manual `/compact` at a natural break point, keeping the stable starting set in workspace files, and excluding build artifacts, dependencies, and generated files.

---

## 3. Supplementary & Cross-Verification Sources (Verified and Archived with Zero Omission)

- **[S1]** [Question about personal wrapper around official Antigravity CLI headless mode (#178472)](https://discuss.ai.google.dev/t/question-about-personal-wrapper-around-official-antigravity-cli-headless-mode/178472) — Google AI Developers Forum (Author: `@Ahmad3`), Published: 2026-08-16 (Accessed: 2026-11-04 | Type: Specialist Community | Archive Note: Representative early thread inquiring about ToS compliance when wrapping `agy -p` in a private single-user Web UI / Telegram bot)
- **[S2]** [Is a local single-user orchestrator allowed to invoke the official agy -p CLI? (#179013)](https://discuss.ai.google.dev/t/is-a-local-single-user-orchestrator-allowed-to-invoke-the-official-agy-p-cli/179013) — Google AI Developers Forum (Author: `@JJPop`; Reply: `@Ambati_Rajendra`), Published: 2026-08-20 (Accessed: 2026-11-04 | Type: Primary Official Forum | Archive Note: Verifies early user inquiries regarding Section 6 during the `agy 1.1.12` era)
- **[S3]** [Google AI Studio Gemini API invoking a consumer-authenticated agy CLI is allowed? (#194944)](https://discuss.ai.google.dev/t/google-ai-studio-gemini-api-invoking-a-consumer-authenticated-agy-cli-is-allowed/194944) — Google AI Developers Forum (Question: `@oooolee`; Official Reply: `@Ambati_Rajendra`), Published: 2026-10-26 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Archive Note: Confirms that reverse-triggering a consumer-authenticated `agy` CLI from an external Gemini API function tool is prohibited)
- **[S4]** [Can I let my grok bot use agy cli on their computer? (#190429)](https://discuss.ai.google.dev/t/can-i-let-my-grok-bot-use-agy-cli-on-their-computer/190429) — Google AI Developers Forum (Question: `@Chua_Poplar`; Official Reply: `@Ambati_Rajendra`), Published: 2026-10-19 (Accessed: 2026-11-04 | Type: Primary Official Staff Forum Reply | Archive Note: Confirms that driving `agy` across VMs via a third-party bot triggers automated `403 TOS_VIOLATION` bans)
- **[S5]** [AsabaLazy/aur-ASLZ: Arch Linux AUR Repository containing `antigravity-cli-fix-bin`](https://github.com/AsabaLazy/aur-ASLZ) — GitHub (Author: `@AsabaLazy`), Published: 2026-11-02 (Accessed: 2026-11-04 | Type: Open-Source Code Repository | Archive Note: Arch Linux repository that automatically syncs upstream twice daily and pre-applies the 256K unlock patch)
- **[S6]** [mcpservers.org — sshahzaiib/agy-bridge MCP Server](https://mcpservers.org/ja/servers/sshahzaiib/agy-bridge) — MCP Servers Directory (Author: `sshahzaiib`), Published: 2026 (Accessed: 2026-11-04 | Type: Technical Directory | Archive Note: Corroborates background on early community-built `agy-bridge` MCP bridging tools)
- **[S7]** [Claude Code vs Google Antigravity vs GitHub Copilot Pricing & Quota Comparison](https://www.r-jc.jp/news-blog/6256/claude-code-antigravity-%E6%AF%94%E8%BC%83copilot-%E9%81%95%E3%81%84%E6%96%99%E9%87%91%E8%A1%A8/) — R-JC News Blog, Published: 2026 (Accessed: 2026-11-04 | Type: Industry Comparative Analysis | Archive Note: Cross-compares subscription pricing and model quota structures across Claude Code, Antigravity, and GitHub Copilot)
- **[S8]** [Agy unusable after false violation of Terms of Service (#185325)](https://discuss.ai.google.dev/t/agy-unusable-after-false-violation-of-terms-of-service/185325) — Google AI Developers Forum, Published: 2026-09-27 (Accessed: 2026-11-04 | Type: Specialist Community Report | Archive Note: Corroborates cases of personal accounts rendered unusable by false-positive `TOS_VIOLATION` flags)
- **[S9]** [BUG: AGY CLI Error 3501 (403 Forbidden) - Personal Account Wrongly Flagged as Enterprise (#185714)](https://discuss.ai.google.dev/t/bug-agy-cli-error-3501-403-forbidden-personal-account-wrongly-flagged-as-enterprise/185714) — Google AI Developers Forum, Published: 2026-09-28 (Accessed: 2026-11-04 | Type: Specialist Community Report | Archive Note: Corroborates background on personal `@gmail.com` accounts encountering `Error 3501` 403 blocks)
