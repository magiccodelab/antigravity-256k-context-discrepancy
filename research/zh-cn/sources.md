# 【Google Antigravity 256K 上下文硬编码限制与 `agy -p` 政策反转事件】全量参考来源与检索档案

- **语言 / Language**：[English (../sources.md)](../sources.md) | **简体中文**
- **档案库全局导航**：[仓库总览](../../README.zh-CN.md) ｜ [调研总览 (dossier.md)](./dossier.md) ｜ **全量来源档案（本文件）** ｜ [静态逆向取证报告](../../evidence/zh-cn/evidence-report.md)
- **专题分册导航**：[01-事件背景与时间线](./01-incident-overview-and-timeline.md) ｜ [02-256K硬编码与逆向拆包](./02-256k-hardcoded-gates-and-binary-teardown.md) ｜ [03-agy-p政策反转与封号](./03-agy-p-headless-policy-reversal-and-bans.md) ｜ [04-社区与官方应对](./04-community-and-official-responses.md) ｜ [05-原文引用与记录汇总](./05-verbatim-quotes-and-evidence-vault.md)
- **研究 ID (UUID)**：`df866cf1-da40-4b82-8084-34c6b0729544` ｜ **目录路径**：`research/zh-cn/` ｜ **调研时间**：2026-11-04 ｜ **正文引用来源**：57 条 ｜ **备查补充来源**：9 条

---

## 1. 来源总览索引表

| 编号 | 来源标题与链接 | 机构 / 作者 | 发布日期 | 来源层级 | 置信度 | 主档案与分册引用位置 |
|---|---|---|---|---|---|---|
| `[1]`（`[^1]`） | [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) | GitHub `google-antigravity/antigravity-cli` / `Burntgogi`, `NuCl34R`, `Circumsized`, `Emily2040`, `LiraNuna`, `AsabaLazy` | 2026-10-14 – 2026-11-02 | 一手官方仓库 Issue & 代码逆向 | 高 | `dossier.md` §1, §3.2, §3.3, §4, §5, §6–§9；分册一至五 |
| `[2]`（`[^2]`） | [Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) | GitHub Gist / `@LiraNuna` | 2026-10-23 | 一手逆向工程补丁源码 | 高 | `dossier.md` §1, §3.1, §3.2, §3.5, §4；分册一、二、四 |
| `[3]`（`[^3]`） | [[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878)](https://github.com/google-antigravity/antigravity-cli/issues/878) | GitHub `google-antigravity/antigravity-cli` / `schalatap`, `murias002`, `Corsagee`, `Infinnerty`, `jlukic` | 2026-08-26 – 2026-10-31 | 一手官方仓库 Issue & 6.5万次调用统计 | 高 | `dossier.md` §1, §3.2, §4, §8, §9；分册一、二、四、五 |
| `[4]`（`[^4]`） | [[Bug]: Five measured places the CLI spends time and tokens the work does not need (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) | GitHub `google-antigravity/antigravity-cli` / `@Emily2040` | 2026-10-19 – 2026-10-25 | 一手官方仓库 Issue & 1279次调用实测 | 高 | `dossier.md` §1, §3.2, §3.3, §4, §8, §9；分册一、二、四、五 |
| `[5]`（`[^5]`） | [[Feature Request]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue #1072)](https://github.com/google-antigravity/antigravity-cli/issues/1072) | GitHub `google-antigravity/antigravity-cli` / `@flotoz`, `@Emily2040` | 2026-09-20 – 2026-10-19 | 一手官方仓库 Issue | 高 | `dossier.md` §3.3, §4；分册一、二 |
| `[6]`（`[^6]`） | [Feature Request: Add `max_context_tokens` setting to control per-turn token budget (Issue #743)](https://github.com/google-antigravity/antigravity-cli/issues/743) | GitHub `google-antigravity/antigravity-cli` / `@thealgebraist` | 2026-08-03 | 一手官方仓库 Issue | 高 | `dossier.md` §3.2；分册一、四 |
| `[7]`（`[^7]`） | [Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue #366)](https://github.com/google-antigravity/antigravity-cli/issues/366) | GitHub `google-antigravity/antigravity-cli` / `@UesugiErii`, `@Mandark-droid`, `@evilfurryone`, `@rajatim` | 2026-06-11 – 2026-10-20 | 一手官方仓库 Issue | 高 | `dossier.md` §3.1；分册一、四 |
| `[8]`（`[^8]`） | [agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync (Issue #927)](https://github.com/google-antigravity/antigravity-cli/issues/927) | GitHub `google-antigravity/antigravity-cli` / `@jlwainwright` | 2026-09-02 | 一手官方仓库 Issue | 高 | `dossier.md` §3.4, §4；分册一、三 |
| `[9]`（`[^9]`） | [Addressing Antigravity Bans & Reinstating Access (Discussion #20632)](https://github.com/google-gemini/gemini-cli/discussions/20632) | GitHub `google-gemini/gemini-cli` / Google 维护者 `@jackwotherspoon` | 2026-02-27 | 一手官方维护者公告 | 高 | `dossier.md` §1, §2, §3.4, §4, §8；分册一、三 |
| `[10]`（`[^10]`） | [Appeal submitted but TOS_VIOLATION suspension persists (Issue #29576)](https://github.com/google-gemini/gemini-cli/issues/29576) | GitHub `google-gemini/gemini-cli` / `@myessaylab1234` | 2026-09-30 | 一手官方仓库 Issue | 高 | `dossier.md` §2, §3.1, §3.4, §4；分册一、三 |
| `[11]`（`[^11]`） | [Google Antigravity CLI Official Releases (`v1.4.2` – `v1.5.0`)](https://github.com/google-antigravity/antigravity-cli/releases) | GitHub `google-antigravity/antigravity-cli` | 2026-10-18 – 2026-11-04 | 一手官方发版日志 | 高 | `dossier.md` §1, §2, §3.3, §3.5, §7, §8, §10；分册一至四 |
| `[12]`（`[^12]`） | [Google Antigravity Docs — Changelog (`v2.0.1` – `v2.29.0`)](https://antigravity.google/docs/changelog) | Google Antigravity 官方文档 | 2026-11-03 | 一手官方文档 | 高 | `dossier.md` §1, §2, §4, §8；分册一、二、四 |
| `[13]`（`[^13]`） | [Google Antigravity Docs — Official FAQ](https://antigravity.google/docs/faq/) | Google Antigravity 官方文档 | 2026-10 | 一手官方文档 | 高 | `dossier.md` §1, §2, §3.4, §6, §8；分册一、三、四 |
| `[14]`（`[^14]`） | [Google Antigravity — Additional Terms of Service](https://antigravity.google/terms) | Google LLC | 2026-09 | 一手官方合同条款 | 高 | `dossier.md` §1, §2, §7；分册一、三 |
| `[15]`（`[^15]`） | [Google Antigravity Docs — Workspace Rules (24 KB Cap & 20,000-Token Budget)](https://antigravity.google/docs/rules/) | Google Antigravity 官方文档 | 2026-10 | 一手官方文档 | 高 | `dossier.md` §3.5, §4 |
| `[16]`（`[^16]`） | [Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/) | Google Developers Blog | 2026-10 | 一手官方技术博客 | 高 | `dossier.md` §3.5；分册四 |
| `[17]`（`[^17]`） | [[Teardown / Binary Diff] Antigravity Updated to v2.29.0](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/) | Reddit `r/google_antigravity` / `u/Lberondroth` | 2026-11-03 | 专业社区二进制拆包报告 | 高 | `dossier.md` §2, §3.3, §4, §6, §8, §10；分册一、二、五 |
| `[18]`（`[^18]`） | [[Deep-Dive] Antigravity v2.27.0 → v2.28.1 Binary & ASAR Diff](https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/) | Reddit `r/google_antigravity` / `u/Lberondroth` | 2026-10-28 | 专业社区二进制拆包报告 | 高 | `dossier.md` §1, §3.3, §4, §6, §8；分册一、二 |
| `[19]`（`[^19]`） | [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon?](https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/) | Reddit `r/AntigravityGoogle` | 2026-11-02 | 专业社区讨论 | 高 | `dossier.md` §3.3, §4, §8, §10；分册一、二 |
| `[20]`（`[^20]`） | [Timeline for enabling the 512k / 900k ContextWindowOptions (`r/google_antigravity`)](https://www.reddit.com/r/google_antigravity/comments/1xmc89l/timeline_for_enabling_the_512k_900k/) | Reddit `r/google_antigravity` | 2026-11-02 | 专业社区讨论 | 高 | `dossier.md` §8；分册一 |
| `[21]`（`[^21]`） | [Antigravity's aggressive history compaction is ruining Gemini's 1M+ context selling point](https://www.reddit.com/r/google_antigravity/comments/1trr4dy/antigravitys_aggressive_history_compaction_is/) | Reddit `r/google_antigravity` | 2026-05-30 | 专业社区讨论 | 高 | `dossier.md` §8；分册一 |
| `[22]`（`[^22]`） | [How to force AG to use subagents? (Orchestrator + Ephemeral Subagents)](https://www.reddit.com/r/google_antigravity/comments/1uo7g2k/how_to_force_ag_to_use_subagents/) | Reddit `r/google_antigravity` | 2026-07 | 专业社区工程实践 | 高 | `dossier.md` §11（仅列于脚注） |
| `[23]`（`[^23]`） | [Clarification Request: Is local IPC/PTY wrapping of `agy` CLI for open-standard editor bridges (e.g., ACP) ToS compliant? (#175905)](https://discuss.ai.google.dev/t/clarification-request-is-local-ipc-pty-wrapping-of-agy-cli-for-open-standard-editor-bridges-e-g-acp-tos-compliant/175905) | Google AI Developers Forum / `@shindgew`, 官方 `@Rebbanpalli_Naveen` | 2026-07-23 – 2026-09-16 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §6, §8；分册一、三 |
| `[24]`（`[^24]`） | [Is it allowed for an open-source tool to call the official `agy` CLI in headless mode? (#183429)](https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429) | Google AI Developers Forum / `@darkest_alex`, 官方 `@Ambati_Rajendra` | 2026-09-17 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §6, §8；分册一、三、五 |
| `[25]`（`[^25]`） | [Is using the official agy CLI through a local MCP server with third-party AI agents permitted? (#184829)](https://discuss.ai.google.dev/t/is-using-the-official-agy-cli-through-a-local-mcp-server-with-third-party-ai-agents-permitted/184829) | Google AI Developers Forum / `@snowstone`, 官方 `@Ambati_Rajendra` | 2026-09-25 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4；分册一 |
| `[26]`（`[^26]`） | [One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? (#185460)](https://discuss.ai.google.dev/t/one-person-several-paid-google-ai-pro-accounts-may-a-local-orchestrator-use-them-with-the-official-agy-cli/185460) | Google AI Developers Forum / `@Winez`, 官方 `@Ambati_Rajendra` | 2026-09-28 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §8；分册一、三 |
| `[27]`（`[^27]`） | [Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (#185494)](https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494) | Google AI Developers Forum / `@Diggy_Styon`, 官方 `@Ambati_Rajendra` | 2026-09-28 | 一手官方人员论坛回复 | 高 | `dossier.md` §2, §3.4, §6, §8；分册一、三、五 |
| `[28]`（`[^28]`） | [Is invoking the official `agy` CLI from a local AI coding assistant allowed on an individual Google AI account? (#185992)](https://discuss.ai.google.dev/t/is-invoking-the-official-agy-cli-from-a-local-ai-coding-assistant-allowed-on-an-individual-google-ai-account/185992) | Google AI Developers Forum / `@Ivan_Valenzuela`, 官方 `@chunduriv` | 2026-09-29 – 2026-10-01 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §6, §8；分册一、三 |
| `[29]`（`[^29]`） | [Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper (#185949)](https://discuss.ai.google.dev/t/clarification-on-codex-claude-skill-invoking-the-official-agy-cli-through-a-local-powershell-wrapper/185949) | Google AI Developers Forum / `@nothing2`, 官方 `@Ambati_Rajendra` | 2026-09-29 – 2026-10-02 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §8；分册一、三 |
| `[30]`（`[^30]`） | [Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? (#186954)](https://discuss.ai.google.dev/t/is-automatically-launching-the-official-agy-cli-from-a-local-ai-orchestrator-allowed-under-additional-terms-6/186954) | Google AI Developers Forum / `@local_ai_builder`, 官方 `@Rebbanpalli_Naveen` | 2026-10-05 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §8；分册一、三 |
| `[31]`（`[^31]`） | [Is invoking the official Antigravity CLI from Codex allowed? (#186952)](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952) | Google AI Developers Forum / `@Hossy`, 官方 `@Rebbanpalli_Naveen` | 2026-10-05 | 一手官方人员论坛回复 | 高 | `dossier.md` §1, §3.1, §3.4, §4, §6, §8；分册一、三、五 |
| `[32]`（`[^32]`） | [Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (#186246)](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246) | Google AI Developers Forum / `@Croixofwar`, 官方 `@chunduriv` | 2026-10-01 – 2026-11-02 | 一手官方人员论坛回复 | 高 | `dossier.md` §1, §3.1, §3.4, §4, §6, §8, §9；分册一、三、五 |
| `[33]`（`[^33]`） | [Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator (#187536)](https://discuss.ai.google.dev/t/terms-clarification-personal-antigravity-cli-integration-with-a-multi-ai-orchestrator/187536) | Google AI Developers Forum / `@BuddhaRyu`, 官方 `@Ambati_Rajendra` | 2026-10-08 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §4, §6, §8；分册一、三 |
| `[34]`（`[^34]`） | [Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241) | Google AI Developers Forum / `@ALAMDAR_BALOCH`, 官方 `@chunduriv` | 2026-10-07 – 2026-10-08 | 一手官方人员论坛回复 | 高 | `dossier.md` §1, §3.4, §4, §6, §8；分册一、三、四、五 |
| `[35]`（`[^35]`） | [Clarification on consumer-plan policy for invoking official agy via local subprocess (#189905)](https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905) | Google AI Developers Forum / `@Amane2418`, `@Jay10`, 官方 `@Rebbanpalli_Naveen` | 2026-10-16 – 2026-10-26 | 一手官方人员论坛回复 | 高 | `dossier.md` §1, §2, §3.4, §6, §8；分册一、三、四、五 |
| `[36]`（`[^36]`） | [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689) | Google AI Developers Forum / `@eliavb`, `@b_w`, `@Tora39`, 官方 `@Rebbanpalli_Naveen` | 2026-10-23 – 2026-10-27 | 一手官方人员论坛回复 | 高 | `dossier.md` §1, §3.4, §3.5, §4, §6–§9；分册一、三、四、五 |
| `[37]`（`[^37]`） | [Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998)](https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998) | Google AI Developers Forum / `@Herzchen`, 官方 `@Rebbanpalli_Naveen` | 2026-10-26 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.5；分册一、三、四 |
| `[38]`（`[^38]`） | [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815) | Google AI Developers Forum / `@Lars`, 官方 `@Ambati_Rajendra` | 2026-10-19 – 2026-10-28 | 一手官方人员论坛回复 | 高 | `dossier.md` §1, §3.1, §3.4, §6, §8；分册一、三、四、五 |
| `[39]`（`[^39]`） | [Policy clarification: private monitoring app using official Antigravity CLI and subscription quota (#190756)](https://discuss.ai.google.dev/t/policy-clarification-private-monitoring-app-using-official-antigravity-cli-and-subscription-quota/190756) | Google AI Developers Forum / `@xorz`, `@oli_hlmpn`, 官方 `@Ambati_Rajendra` | 2026-10-19 – 2026-10-29 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.1, §3.4, §6, §7；分册一、三 |
| `[40]`（`[^40]`） | [Clarification on subscription-based agy -p integration with a local multi-agent framework (#196472)](https://discuss.ai.google.dev/t/clarification-on-subscription-based-agy-p-integration-with-a-local-multi-agent-framework/196472) | Google AI Developers Forum / `@Dsssyc`, 官方 `@Rebbanpalli_Naveen` | 2026-10-28 – 2026-10-29 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.4, §6, §8；分册一、三 |
| `[41]`（`[^41]`） | [Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131)](https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131) | Google AI Developers Forum / `@sannikov`, `@tkosub`, `@oli_hlmpn`, 官方 `@Rebbanpalli_Naveen` | 2026-10-31 – 2026-11-02 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.5, §6, §7；分册三、四 |
| `[42]`（`[^42]`） | [Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351)](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351) | Google AI Developers Forum / `@Yiji_Kong`, `@RobW`, `@shuai_guo`, 官方 `@Maniraj_Chervirala` | 2026-10-28 – 2026-10-29 | 一手实测报告与官方回复 | 高 | `dossier.md` §1, §2, §3.1, §3.3, §4, §6, §8, §9；分册一、二、四、五 |
| `[43]`（`[^43]`） | [Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476) | Google AI Developers Forum / `@Hello_there`, 官方 `@chunduriv` | 2026-10-15 – 2026-10-20 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.3, §3.5, §6；分册一、二、四 |
| `[44]`（`[^44]`） | [Feature request: Show context window usage in Antigravity Desktop (#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694) | Google AI Developers Forum / `@Soro`, 官方 `@chunduriv`, `@dimitr-io` | 2026-09-24 – 2026-10-22 | 一手官方人员论坛回复 | 高 | `dossier.md` §4, §5, §7；分册一、二、四 |
| `[45]`（`[^45]`） | [Persistent Gemini CLI TOS_VIOLATION after appeal — request for private review or sync check (#186089)](https://discuss.ai.google.dev/t/persistent-gemini-cli-tos-violation-after-appeal-request-for-private-review-or-sync-check/186089) | Google AI Developers Forum / `@myessaylab1234` | 2026-09-30 | 专业社区报告 | 高 | `dossier.md` §2, §3.1, §3.4, §4；分册一、三 |
| `[46]`（`[^46]`） | [Antigravity CLI Agent reliability issue (#176775)](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775) | Google AI Developers Forum / `@Pitt_zhang` | 2026-08-01 | 专业社区审计报告 | 高 | `dossier.md` §3.3；分册一、二 |
| `[47]`（`[^47]`） | [Google Antigravity FAQ 政策更新与停止第三方工具連携说明](https://note.com/horizon_it00/n/nded285d721d1) | note.com / `horizon_it00`（佐藤賢治） | 2026-10 | 专业技术博客 | 高 | `dossier.md` §3.4；分册一 |
| `[48]`（`[^48]`） | [Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (`239,616` Compaction Threshold)](https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk) | Nous Research Hermes Agent Docs | 2026-09-23 | 专业技术文档 | 高 | `dossier.md` §3.2, §4；分册一 |
| `[49]`（`[^49]`） | [mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management](https://github.com/mksglu/context-mode) | GitHub / `@mksglu` | 2026 | 开源代码仓库 | 高 | `dossier.md` §3.5；分册四 |
| `[50]`（`[^50]`） | [AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE](https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor) | GitHub / `@AGI-is-going-to-arrive` | 2026 | 开源代码仓库 | 高 | `dossier.md` §3.5；分册四 |
| `[51]`（`[^51]`） | [dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context) | GitHub / `@dimitr-io` | 2026-10-22 | 开源代码仓库 | 高 | `dossier.md` §3.5, §5, §7；分册一、二、四 |
| `[52]`（`[^52]`） | [weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI](https://github.com/weby-homelab/antigravity-cli-statusline) | GitHub / Weby Homelab | 2026 | 开源代码仓库 | 高 | `dossier.md` §3.5；分册四 |
| `[53]`（`[^53]`） | [oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit ("The IDE for Your Coding Agent")](https://github.com/oraios/serena) | GitHub / Oraios AI | 2026 | 开源代码仓库 | 高 | `dossier.md` §3.5；分册四 |
| `[54]`（`[^54]`） | [xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server)](https://github.com/xdotech/goatlas) | GitHub / `@xdotech` | 2026 | 开源代码仓库 | 高 | `dossier.md` §3.5；分册四 |
| `[55]`（`[^55]`） | [Kevin Hou — "Get Out of the Model's Way" (AI Engineer Summit Presentation & Speaker Profile)](https://www.youtube.com/watch?v=xsVD9_cJNYs) | AI Engineer / YouTube (`https://ai.engineer/speakers/kevin-hou`) / Kevin Hou | 2026 | 一手高管公开演讲 | 高 | `dossier.md` §3.1, §8；分册一 |
| `[56]`（`[^56]`） | [Varun Mohan Education Qualifications: How an Indian-Origin MIT Engineer Secured Google's $2.4 Billion AI Investment](https://timesofindia.indiatimes.com/education/news/varun-mohan-education-qualifications-how-an-indian-origin-mit-engineer-secured-googles-2-4-billion-ai-investment/articleshow/122405531.cms) | Times of India | 2025-07 | 权威媒体报道 | 高 | `dossier.md` §2, §3.1, §8；分册一 |
| `[57]`（`[^57]`） | [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) | Google AI Developers Forum / `@Sharper`（官方回复：`@chunduriv`） | 2026-11-04 | 一手官方人员论坛回复 | 高 | `dossier.md` §3.2、§8、§9、§10；分册一、二、四、五 |

---

## 2. 正文引用来源详单（与 `dossier.md` 及各分册的 `[^1]`..`[^57]` 严格一一对应）

### [1] [Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)（对应 `[^1]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/issues/1245
- **机构 / 作者**：GitHub `google-antigravity/antigravity-cli` 官方仓库｜发起人：`@Burntgogi`；核心技术取证评论者：`@NuCl34R`、`@Circumsized`、`@Emily2040`、`@LiraNuna`、`@AsabaLazy`；被指派维护者：`@chandrashakherkamasani`
- **发布日期**：2026-10-14（最后更新：2026-11-02）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方代码仓库 Issue 追踪器与二进制反编译实证）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > - `@Burntgogi` (OP): "During multi-phase TypeScript refactoring and application scaffolding runs on `agy 1.3.5` with **Gemini 4 Argon** (`1,048,576` token context window), `antigravity-cli` triggers hard context compactions (`CHECKPOINT: "# Resuming from a compaction..."`) at **~250K–276K tokens** — less than 27% of the model's 1M context window... Checkpoint 1 (Step 79): 276,110 -> 17,336 (`cache_read_tokens`: 0); Mid-run prefix miss (Step 193): 239,857 -> 243,064 (`cache_read_tokens`: 0); Checkpoint 2 (Step 200): 249,867 -> 15,363 (`cache_read_tokens`: 0)."
  > - `@Circumsized` (Comment #2): "Every model call writes a `ContextWindowMetadata` record into the conversation database at `~/.gemini/antigravity-cli/conversations/<session-id>.db`... `max_context_tokens = 256000`, `truncation_reason = 2`... 256,000 is exactly 25% of 1,048,576 — a round fraction of the window, which points at a policy constant rather than a technical ceiling."
  > - `@LiraNuna` (Comments #5 & #6): "1. `SelectorModel.offersContextWindow`: hardcoded to only contain `'Gemini 4 Argon'`. 2. `os.Getenv('AGY_CLI_AUTO_COMPACT_OVERRIDE')` was compiled out / inlined into an empty string `""`. 3. `Store.ContextWindow()` (`0x917d892`) drops the selection for non-Argon models. 4. `cortex/utils/utils.GetCheckpointConfig` (`0x7861894`): `cmp $0x3e800, %edx; mov $0x3e800, %esi; cmovb %edx, %esi; mov %esi, 0x50(%rsp)`."

### [2] Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)（对应 `[^2]`）
- **原始 URL**：https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26
- **机构 / 作者**：GitHub Gist / `@LiraNuna`
- **发布日期**：2026-10-23 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（逆向工程补丁源码）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 解析 ELF `.text` 节与 Go `pclntab` 符号表，定位 `google3/third_party/jetski/cli/model/model.SelectorModel.offersContextWindow`、`google3/third_party/jetski/cli/store/store.(*Store).ContextWindow`、`store.(*Store).ContextWindowChoice`、`google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig` 及 `store.AutoCompactOverride`，修改三处代码位置：让上下文档位选择器对所有模型显示、把 `ContextWindow` 转交 `ContextWindowChoice` 处理、去掉 `GetCheckpointConfig` 中的 256,000 钳制（原始字节 `81fa00e80300be00e803000f42f2`）。本档案不转载脚本本身。

### [3] [Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878)（对应 `[^3]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/issues/878
- **机构 / 作者**：GitHub `google-antigravity/antigravity-cli` / `@schalatap`、Eddy Murias (`@murias002`)、`@Corsagee`、`@Infinnerty`、`@jlukic`（指派维护者：`@manirajc`）
- **发布日期**：2026-08-26（更新至 2026-10-31）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方仓库 Issue 与大规模数据库统计）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@murias002`: "Conversations: 310 | Model calls: 65,613 (86.6% on Gemini 3.8 Flash) | Input tokens per call: p50 136,487 · p90 229,000 · max 255,999 | Calls above 300k input tokens: 0 | Compaction events (drop > 40%): 276 | Context right before compaction: median 255,070 (p10 247,360 · p90 255,874) | Context right after compaction: median 45,304."

### [4] [Bug]: Five measured places the CLI spends time and tokens the work does not need (Issue #1291)（对应 `[^4]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/issues/1291
- **机构 / 作者**：GitHub `google-antigravity/antigravity-cli` / `@Emily2040`（指派维护者：`@chandrashakherkamasani`）
- **发布日期**：2026-10-19（更新至 2026-10-25）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方仓库 Issue 与 1,279 次调用实测报告）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "all model calls: uncached input 10,619,129 | cache_read 192,374,637. 17 calls with input > 100k and cache_read 0: sum 3,292,077 = 31.0 % of all uncached input... 15 of the 17 came within two steps of a `system_message` step... 22 `checkpoint` steps in the same logs: durations (s): 35..602, median 70; checkpoint step usage: none reported; next call: input 44,990–70,041, cache_read 0."

### [5] [Feature Request]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue #1072)（对应 `[^5]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/issues/1072
- **机构 / 作者**：GitHub `google-antigravity/antigravity-cli` / `@flotoz`、`@Emily2040`（指派维护者：`@manirajc`）
- **发布日期**：2026-09-20（更新至 2026-10-19）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方仓库 Issue）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "Even on Gemini 3.8 Flash Low, whose per-token price is very low, my usage burns through the 5-hour and weekly rate-limit windows roughly 3x faster than I'd expect from token cost alone."

### [6] Feature Request: Add `max_context_tokens` setting to control per-turn token budget (Issue #743)（对应 `[^6]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/issues/743
- **机构 / 作者**：GitHub `google-antigravity/antigravity-cli` / `@thealgebraist`
- **发布日期**：2026-08-03 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方仓库 Issue）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "379 steps, 610 KB of transcript, Estimated 500K–1M+ total input+output tokens, ~70% of tokens were redundant compiler error output from earlier failed iterations."

### [7] Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue #366)（对应 `[^7]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/issues/366
- **机构 / 作者**：GitHub `google-antigravity/antigravity-cli` / `@UesugiErii`、`@Mandark-droid`、`@evilfurryone`、`@rajatim`、`@Emily2040`
- **发布日期**：2026-06-11（更新至 2026-10-20）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方仓库 Issue）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 记录 `agy` 移除 `gemini-cli` 原生 OTLP 遥测导出接口、本地 SQLite `~/.gemini/antigravity-cli/conversations/<id>.db` 中 `gen_metadata` / `steps` Protobuf BLOB 结构，以及基于 `PostInvocation` Hook 的 OTLP 导出脚本。

### [8] agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync (Issue #927)（对应 `[^8]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/issues/927
- **机构 / 作者**：GitHub `google-antigravity/antigravity-cli` / `@jlwainwright`
- **发布日期**：2026-09-02 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方仓库 Issue）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "Eligibility check failed: PERMISSION_DENIED (code 403): This service has been disabled in this account for violation of Terms of Service... Suspended since at least 25 May 2026 (Google One case 2-9245000040533, which cannot escalate ToS blocks)... Nothing has happened in 100+ days."

### [9] Addressing Antigravity Bans & Reinstating Access (Discussion #20632)（对应 `[^9]`）
- **原始 URL**：https://github.com/google-gemini/gemini-cli/discussions/20632
- **机构 / 作者**：GitHub `google-gemini/gemini-cli` / Google 维护者 `@jackwotherspoon`
- **发布日期**：2026-02-27 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方公告）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "Because of the backend layer where abuse prevention occurs, bans for Antigravity usage also blocked access to Gemini CLI and Gemini Code Assist... 1. User notification... 2. Recertification... 3. Automatic unban: within a day or two. 4. Permanent ban: If an account is flagged for a second violation of ToS, it will be permanently banned."

### [10] Appeal submitted but TOS_VIOLATION suspension persists (request for private follow-up) (Issue #29576)（对应 `[^10]`）
- **原始 URL**：https://github.com/google-gemini/gemini-cli/issues/29576
- **机构 / 作者**：GitHub `google-gemini/gemini-cli` / `@myessaylab1234`
- **发布日期**：2026-09-30 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方仓库 Issue）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "The backend response is HTTP 403 `PERMISSION_DENIED`, reason `TOS_VIOLATION`, from `cloudcode-pa.googleapis.com`... Asked Google One Support, which could not view the appeal status. Google Developer Profile Support confirmed this is outside its scope. I understand that consumer access to the legacy Gemini CLI ended on June 18, 2026."

### [11] Google Antigravity CLI Official Releases (`v1.4.2` – `v1.5.0`)（对应 `[^11]`）
- **原始 URL**：https://github.com/google-antigravity/antigravity-cli/releases
- **机构 / 作者**：Google (`google-antigravity/antigravity-cli`)
- **发布日期**：2026-10-18 至 2026-11-04 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方发版日志）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 涵盖 `v1.4.3`（修复压缩后丢失子代理 ID）、`v1.4.4`（压缩请求保留工具声明复用前缀缓存）、`v1.4.6`（压缩摘要保留图片与 PDF）、`v1.4.7`（修复超大消息无限循环压缩）、`v1.4.9`（修复裁剪历史后遗留未配对工具调用）、`v1.4.10`（修复子代理压缩后丢失父代理任务与 ID，改进 `/context` 压缩后体积统计）及 `v1.5.0`（2026-11-04 新增 `agy acp` 与 `--remote-agent`）。

### [12] Google Antigravity Docs — Changelog (`v2.0.1` – `v2.29.0`)（对应 `[^12]`）
- **原始 URL**：https://antigravity.google/docs/changelog
- **机构 / 作者**：Google Antigravity 官方文档
- **发布日期**：2026-11-03（核验于 2026-11-04）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方文档）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 记录 Antigravity App 从 `v2.0.1` 至 `v2.29.0`（2026-11-03）的全部版本演进，含 `v2.24.0`（2026-10-15 推出 `/compact` 与 `Gemini 4 Argon`）、`v2.27.0`（10-26）、`v2.28.0`（10-27）、`v2.28.1`（10-28）、`v2.28.2`（10-29）及 `v2.29.0`（11-03）。

### [13] Google Antigravity Docs — Official FAQ（对应 `[^13]`）
- **原始 URL**：https://antigravity.google/docs/faq/
- **机构 / 作者**：Google Antigravity 官方文档
- **发布日期**：2026-10 中旬更新 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方文档）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "Why can’t I use third-party software (such as Claude Code, OpenClaw, or OpenCode) with my Antigravity login? Using third-party software, tools, or services to access Antigravity is a violation of our Terms of Service... Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching? Antigravity CLI headless mode (`agy -p`) is intended for direct scripting and CI workflows. Wrapping `agy` inside third-party coding agents or applications, or automating account rotation, is not supported under consumer plans."

### [14] Google Antigravity — Additional Terms of Service（对应 `[^14]`）
- **原始 URL**：https://antigravity.google/terms
- **机构 / 作者**：Google LLC
- **发布日期**：2026-09 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方法律条款）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "6. You must not abuse, harm, interfere with, or disrupt the Service. This includes, but is not limited to, using the Service in connection with products not provided by us. Using third party software, tools, or services to access the Service (e.g. using OpenClaw with Antigravity OAuth) is a breach of this Agreement. Such actions may be grounds for suspension or termination of your Antigravity and/or Gemini CLI accounts."

### [15] Google Antigravity Docs — Workspace Rules (24 KB Per-File Limit & 20,000-Token Aggregate Rules Budget)（对应 `[^15]`）
- **原始 URL**：https://antigravity.google/docs/rules/
- **机构 / 作者**：Google Antigravity 官方文档（及本机内置 `agy-customizations/docs/rules.md`）
- **发布日期**：2026-10 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方文档）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "Per-File Limit (`24 KB` / `24,000` bytes): Each rule file is capped at 24,000 bytes (after expanding `@[label](path)` includes) and truncated on line boundaries when over the cap. Aggregate Rules Budget (`20,000` tokens): Always-on and global rules share a dedicated 20,000-token rules budget (`defaultRulesBudget`)... Over-budget rules are demoted from full inline text to file path pointers so the agent can read them on demand."

### [16] Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon（对应 `[^16]`）
- **原始 URL**：https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/
- **机构 / 作者**：Google Developers Blog
- **发布日期**：2026-10 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方技术博客）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "The Python repository (`google/adk-python`) spans roughly 205,000 lines of code across nearly 800 source files, backed by nearly 17,000 unit tests... First, we put the team's maintainer guidelines into the repository's `.agents/skills` directory... Each skill starts with a short `SKILL.md` index that points to focused `references/` files on demand, so we don't flood the agent's context window on every turn... Second, we turned on `/goal` mode in Google Antigravity, which preserves the goal across context compactions so the agent keeps running until the task is finished. With `.agents/skills` in the repo and `/goal` mode active, Argon ran unattended for 11 hours overnight."

### [17] [Teardown / Binary Diff] Antigravity Updated to v2.29.0（对应 `[^17]`）
- **原始 URL**：https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/
- **机构 / 作者**：Reddit `r/google_antigravity` / `u/Lberondroth`
- **发布日期**：2026-11-03 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区（一手二进制差分拆包报告）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `language_server.exe` 增长 `+4.47 MB`，内嵌 Web UI 增长 `+36%`，新增 `cortex/tokens/compaction` 与 `cortex/shared/snapshot` 底层包，引入受控开关 `enable-context-usage-indicator`："What you get instead is the Context and contingent Indicator, so you can soon watch your contingent drain in high definition, right next to the send button..."

### [18] [Deep-Dive] Antigravity v2.27.0 → v2.28.1 Binary & ASAR Diff（对应 `[^18]`）
- **原始 URL**：https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/
- **机构 / 作者**：Reddit `r/google_antigravity` / `u/Lberondroth`
- **发布日期**：2026-10-28 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区（一手二进制与 ASAR 拆包报告）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 拆解 `v2.27.0` 至 `v2.28.1` 的 Protobuf 与二进制结构变化，发现 `ContextWindowOptions`（`256K`、`512K`、`900K`）及布尔标记 `Expensive`。

### [19] Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon? (`r/AntigravityGoogle`)（对应 `[^19]`）
- **原始 URL**：https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/
- **机构 / 作者**：Reddit `r/AntigravityGoogle`
- **发布日期**：2026-11-02 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区讨论｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 汇总社区对 `512K`（`1.3x`）、`900K`（`1.8x Expensive`）档位何时在桌面端启用及何时解除 `GetCheckpointConfig` 中 `256K` 钳制的公开追问。

### [20] Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon? (`r/google_antigravity`)（对应 `[^20]`）
- **原始 URL**：https://www.reddit.com/r/google_antigravity/comments/1xmc89l/timeline_for_enabling_the_512k_900k/
- **机构 / 作者**：Reddit `r/google_antigravity`
- **发布日期**：2026-11-02 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区讨论｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 与 `[19]` 同步发布于 `r/google_antigravity` 主版块，截至 2026-11-04 官方团队未在帖内给出时间表回复。

### [21] Antigravity's aggressive history compaction is ruining Gemini's 1M+ context selling point（对应 `[^21]`）
- **原始 URL**：https://www.reddit.com/r/google_antigravity/comments/1trr4dy/antigravitys_aggressive_history_compaction_is/
- **机构 / 作者**：Reddit `r/google_antigravity`
- **发布日期**：2026-05-30 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区讨论｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 早期记录 Antigravity 在远低于 1M 上下文处执行激进历史摘要压缩、破坏 Gemini 1M+ 长上下文核心优势的社区反馈帖。

### [22] How to force AG to use subagents? (Orchestrator + Ephemeral Subagents)（对应 `[^22]`）
- **原始 URL**：https://www.reddit.com/r/google_antigravity/comments/1uo7g2k/how_to_force_ag_to_use_subagents/
- **机构 / 作者**：Reddit `r/google_antigravity`
- **发布日期**：2026-07 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区工程实践｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 阐述通过 `AGENTS.md` 强制主代理扮演 Orchestrator、派发短生命周期子代理（Ephemeral Subagents）并通过本地 `.md` 状态文件交接以规避单主线程上下文爆炸的工程模式。

### [23] Clarification Request: Is local IPC/PTY wrapping of `agy` CLI for open-standard editor bridges (e.g., ACP) ToS compliant? (#175905)（对应 `[^23]`）
- **原始 URL**：https://discuss.ai.google.dev/t/clarification-request-is-local-ipc-pty-wrapping-of-agy-cli-for-open-standard-editor-bridges-e-g-acp-tos-compliant/175905
- **机构 / 作者**：Google AI Developers Forum / 提问：`@shindgew`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-07-23（官方回复：2026-09-16 12:45:42 UTC）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Rebbanpalli_Naveen`: "1. ToS Compliance (Local CLI Wrapping): Yes, your described pattern is compliant. The ToS restriction on third-party access tools is aimed at credential harvesting, unofficial API reverse-engineering, and multi-tenant proxying. Spawning the official, unmodified `agy` binary locally as a single-user child process/PTY under the user’s own `agy login` session complies with single-user usage, provided tokens are not extracted, stored, or proxied."

### [24] Is it allowed for an open-source tool to call the official `agy` CLI in headless mode? (#183429)（对应 `[^24]`）
- **原始 URL**：https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429
- **机构 / 作者**：Google AI Developers Forum / 提问：`@darkest_alex`；官方回复：`@Ambati_Rajendra`
- **发布日期**：2026-09-17 12:06:16 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra`: "Is calling `agy` headless allowed? Yes. Calling the official `agy` binary locally via `agy -p` is an officially supported feature built for programmatic scripting (see the Headless Mode Documentation). It does not violate Section 6, provided your tool never extracts, stores, or intercepts the user’s OAuth credentials."

### [25] Is using the official agy CLI through a local MCP server with third-party AI agents permitted? (#184829)（对应 `[^25]`）
- **原始 URL**：https://discuss.ai.google.dev/t/is-using-the-official-agy-cli-through-a-local-mcp-server-with-third-party-ai-agents-permitted/184829
- **机构 / 作者**：Google AI Developers Forum / 提问：`@snowstone`；官方回复：`@Ambati_Rajendra`
- **发布日期**：2026-09-25 12:47:46 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra`: "Third-Party Agent MCP Wrapper: Wrapping `agy -p` in a local MCP server to use with third-party agents (like Claude Code or Codex CLI) is not permitted."

### [26] One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? (#185460)（对应 `[^26]`）
- **原始 URL**：https://discuss.ai.google.dev/t/one-person-several-paid-google-ai-pro-accounts-may-a-local-orchestrator-use-them-with-the-official-agy-cli/185460
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Winez`；官方回复：`@Ambati_Rajendra`
- **发布日期**：2026-09-28 12:22:17 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra`: "While we permit running the official `agy` CLI locally as a subprocess for your own personal use (as long as you are using your single personal account and interacting with it directly), using an orchestrator to cycle through multiple Google AI Pro accounts to bypass quota limits, pool capacity, or distribute tasks is a direct violation of the Terms of Service."

### [27] Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (#185494)（对应 `[^27]`）
- **原始 URL**：https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Diggy_Styon`；官方回复：`@Ambati_Rajendra`
- **发布日期**：2026-09-28 07:14:31 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra`: "Yes, your use case is permitted. Running the official `agy` binary locally (even headless via a script) under your own account is compliant with the terms... As long as you are executing the unmodified `agy` CLI and not proxying your auth tokens into another application, you are good to go!"

### [28] Is invoking the official `agy` CLI from a local AI coding assistant allowed on an individual Google AI account? (#185992)（对应 `[^28]`）
- **原始 URL**：https://discuss.ai.google.dev/t/is-invoking-the-official-agy-cli-from-a-local-ai-coding-assistant-allowed-on-an-individual-google-ai-account/185992
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Ivan_Valenzuela`；官方回复：`@chunduriv`
- **发布日期**：2026-09-30 23:25:25 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@chunduriv`: "Is invoking the unmodified `agy.exe` binary in headless mode from a local assistant permitted on an individual account? Yes. Section 6 of the Antigravity Additional Terms of Service (and our FAQ regarding third-party tools) prohibits extracting, proxying, or reusing Antigravity OAuth credentials inside third-party harnesses to call Antigravity’s backend endpoints directly."

### [29] Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper (#185949)（对应 `[^29]`）
- **原始 URL**：https://discuss.ai.google.dev/t/clarification-on-codex-claude-skill-invoking-the-official-agy-cli-through-a-local-powershell-wrapper/185949
- **机构 / 作者**：Google AI Developers Forum / 提问：`@nothing2`；官方回复：`@Ambati_Rajendra`
- **发布日期**：2026-10-02 20:20:25 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra`: "Short answer: yes, your setup is fine. What’s not allowed is extracting sign-in tokens, calling private Antigravity endpoints, or using `agy` as a raw model backend for another agent. Running the official `agy` CLI locally to do a specific task and reading its output is allowed. That’s the same rule behind all the forum answers you found. To your questions: Codex or Claude calling the Skill: Yes, as long as it sends `agy` a real task..."

### [30] Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? (#186954)（对应 `[^30]`）
- **原始 URL**：https://discuss.ai.google.dev/t/is-automatically-launching-the-official-agy-cli-from-a-local-ai-orchestrator-allowed-under-additional-terms-6/186954
- **机构 / 作者**：Google AI Developers Forum / 提问：`@local_ai_builder`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-10-05 10:17:42 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Rebbanpalli_Naveen`: "Yes, the single-user local workflow you described is supported through the official CLI Headless Mode. To answer your questions directly: Local Child Process vs. Section 6 (Questions 1 & 3): Yes, this is permitted and materially different."

### [31] Is invoking the official Antigravity CLI from Codex allowed? (#186952)（对应 `[^31]`）
- **原始 URL**：https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Hossy`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-10-05 10:24:45 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Rebbanpalli_Naveen`: "Yes, the workflow you described is completely fine to use with your personal Google AI Pro subscription... Having Codex invoke the official, unmodified `agy.exe` binary as a local process on your Windows PC—while authentication (`agy auth login`) and all backend communication stay strictly inside `agy.exe`—is a supported use of CLI Headless Mode."

### [32] Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (#186246)（对应 `[^32]`）
- **原始 URL**：https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Croixofwar`；官方回复：`@chunduriv`；后续追问：`@Kevin_Lopez`、`@Lorde_Lorde`、`@Ripu_Yadav`、`@AbMedhat`
- **发布日期**：2026-10-01（官方回复：2026-10-07 15:03:46 UTC；更新至 2026-11-02）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@chunduriv` (2026-10-07 15:03:46 UTC): "1. Is launching the official `agy` CLI as a local subprocess allowed under Section 6? Yes... 2. Does the answer change if you launch the script yourself vs. Claude Code launching it? No. From a policy standpoint, there is no difference whether the unmodified `agy` subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code)..."

### [33] Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator (#187536)（对应 `[^33]`）
- **原始 URL**：https://discuss.ai.google.dev/t/terms-clarification-personal-antigravity-cli-integration-with-a-multi-ai-orchestrator/187536
- **机构 / 作者**：Google AI Developers Forum / 提问：`@BuddhaRyu`；官方回复：`@Ambati_Rajendra`
- **发布日期**：2026-10-08 16:23:34 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra`: "Running `agy` from your orchestrator: Yes. Running the official `agy` program from your app (e.g. `agy -p '<prompt>' --output-format json` or `stream-json`) for your own development work is fine... Handoffs between `agy`, Codex and Claude Code: Yes... Does section 6 apply? No."

### [34] Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)（对应 `[^34]`）
- **原始 URL**：https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241
- **机构 / 作者**：Google AI Developers Forum / 提问：`@ALAMDAR_BALOCH`；官方反转回复：`@chunduriv`
- **发布日期**：2026-10-07（官方回复：2026-10-08 17:34:41 UTC）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@chunduriv` (2026-10-08 17:34:41 UTC): "While `agy` offers a documented headless mode (`agy -p`) for your own local scripts and CI pipelines, we do not support using third-party coding agents (like Codex, Claude Code, OpenClaw, or OpenCode) to orchestrate or wrap `agy` under a consumer login. Routing third-party agent prompts or automated loops through Antigravity can trigger automated ToS enforcement and account suspension."

### [35] Clarification on consumer-plan policy for invoking official agy via local subprocess (#189905)（对应 `[^35]`）
- **原始 URL**：https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Amane2418`、`@Jay10`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-10-16 12:35:16 UTC（更新至 2026-10-26）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Rebbanpalli_Naveen`: "Current Official Guidance: The Antigravity FAQ reflects our current policy for Antigravity CLI headless mode and supersedes earlier forum comments where they differ... Even if you invoke the official, unmodified `agy` binary as a local subprocess without extracting OAuth tokens or reverse-engineering internal endpoints, wrapping `agy` inside third-party coding agents or applications—or automating account rotation—is not supported under consumer plans... violates our Terms of Service and can result in account suspension or termination."

### [36] Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)（对应 `[^36]`）
- **原始 URL**：https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689
- **机构 / 作者**：Google AI Developers Forum / 提问与质问：`@eliavb`、`@b_w`、`@Tora39`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-10-23 至 2026-10-27 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Rebbanpalli_Naveen` (Post #8, 2026-10-27 12:31:42 UTC): "To clarify the timeline for `@b_w` and answer `@Tora39`’s question (a): the Google Antigravity Terms of Service themselves were not modified after September; rather, the Official FAQ was updated in mid-October 2026 to add the section ‘Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?’... Earlier forum replies in #186246 and #186952 gave an overly broad interpretation before that FAQ entry was published."

### [37] Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998)（对应 `[^37]`）
- **原始 URL**：https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Herzchen`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-10-26 13:54:18 UTC ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 区分静态文件起草与自动化循环编排：若外部 AI 自动写入请求、自动触发 `agy -p` 并读取结果形成自动循环，即属违规；仅当由人类所有者显式手动触发（如手动 `workflow_dispatch`）且外部 AI 不耦合在自动循环中时才符合直接 CI 边界。

### [38] Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)（对应 `[^38]`）
- **原始 URL**：https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Lars`；官方回复：`@Ambati_Rajendra`
- **发布日期**：2026-10-19（官方回复：2026-10-28 13:10:46 UTC）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra`: "That still counts as third-party agent coordination. Passing tasks from Claude Code to `agy -p` through local files instead of stdin/stdout pipes still uses consumer quota as a backend worker for an external coding agent, so it requires a Gemini API key or Gemini Enterprise."

### [39] Policy clarification: private monitoring app using official Antigravity CLI and subscription quota (#190756)（对应 `[^39]`）
- **原始 URL**：https://discuss.ai.google.dev/t/policy-clarification-private-monitoring-app-using-official-antigravity-cli-and-subscription-quota/190756
- **机构 / 作者**：Google AI Developers Forum / 提问：`@xorz`；官方回复：`@Ambati_Rajendra`；技术质疑：`@oli_hlmpn`
- **发布日期**：2026-10-19 至 2026-10-29 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复与技术质询）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Ambati_Rajendra` 宣布禁止无人值守后台自动请求，同时说明关闭超额点数需同时配置 `~/.gemini/config/config.json` 中 `"useAiCredits": false` 与 `~/.gemini/antigravity-cli/settings.json` 中 `"useG1Credits": false`；`@oli_hlmpn` 指出底层无法区分进程是由本地 Go/Python 服务还是 3 行 Bash 脚本启动，且 `acp` 允许第三方调用而 `agy -p` 永封形成严重矛盾。

### [40] Clarification on subscription-based agy -p integration with a local multi-agent framework (#196472)（对应 `[^40]`）
- **原始 URL**：https://discuss.ai.google.dev/t/clarification-on-subscription-based-agy-p-integration-with-a-local-multi-agent-framework/196472
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Dsssyc`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-10-28 至 2026-10-29 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Rebbanpalli_Naveen`: "Having an intermediate local Python coordination service spawn and manage the `agy -p` process (rather than the Host or Router Agent directly) does not change this restriction, as `agy` is still being used as an execution backend for an external multi-agent framework under a consumer sign-in."

### [41] Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131)（对应 `[^41]`）
- **原始 URL**：https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131
- **机构 / 作者**：Google AI Developers Forum / 提问：`@sannikov`、`@tkosub`、`@oli_hlmpn`、`@christian-taillon`；官方回复：`@Rebbanpalli_Naveen`
- **发布日期**：2026-10-31 至 2026-11-02 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Rebbanpalli_Naveen` 确认在支持 ACP 协议的编辑器/客户端中启动官方签名的 `antigravity-acp` 二进制并使用个人 OAuth 登录（针对计划内的 Gemini 模型）符合规定。

### [42] Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351)（对应 `[^42]`）
- **原始 URL**：https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351
- **机构 / 作者**：Google AI Developers Forum / 作者：`@Yiji_Kong`；复现：`@RobW`、`@shuai_guo`；官方回复：`@Maniraj_Chervirala`
- **发布日期**：2026-10-28 至 2026-10-29 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（控制变量遥测实证与官方人员回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "Normalized by each model’s published API price, Argon consumes exactly 3.0x more quota per dollar than Flash. If Flash is billed at its published rate, the implied bucket size is ~150 USD, and Argon’s consumption matches 6.00 / 30.00 USD per 1M input/output tokens (3x the advertised 2.00 / 10.00 launch price)... 1 Weekly Pool = 20 × 5-Hourly Pool... In Step 1C, sending just 'Reply with only the word: OK' (28 chars) in a fresh chat on 4 Argon (Low) burned +0.00015849 of weekly quota, which is 12% more than 3.8 Flash (High) writing a full 4,013-word technical article on Paxos and Raft (+0.00014157)."

### [43] Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)（对应 `[^43]`）
- **原始 URL**：https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Hello_there`；官方回复：`@chunduriv`
- **发布日期**：2026-10-15（官方回复：2026-10-20 23:03:17 UTC）｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@Hello_there` 报告压缩后捏造 6 个不存在的标识符并覆写初始化硬限制；`@chunduriv` 给出官方 Workaround：在 `~/.gemini/GEMINI.md` 或 `.agents/rules/` 中添加常驻规则强制遇到 `# Resuming from a compaction` 后用 `view_file` 重读目标文件，并执行 `/compact Preserve exact contract identifiers, initialization limits and inspected file paths verbatim`。

### [44] Feature request: Show context window usage in Antigravity Desktop (#184694)（对应 `[^44]`）
- **原始 URL**：https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Soro`；官方回复：`@chunduriv`；插件作者：`@dimitr-io`
- **发布日期**：2026-09-24 至 2026-10-22 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（官方论坛回复与实证图）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > `@chunduriv` (2026-09-25): "If you ever use the Antigravity CLI, it already has this. Just type `/context` and it shows how much of the context window you’re using..."；`@dimitr-io` (2026-10-22) 发布 `antigravity-desktop-context` 侧边栏插件并附对比截图。

### [45] Persistent Gemini CLI TOS_VIOLATION after appeal — request for private review or sync check (#186089)（对应 `[^45]`）
- **原始 URL**：https://discuss.ai.google.dev/t/persistent-gemini-cli-tos-violation-after-appeal-request-for-private-review-or-sync-check/186089
- **机构 / 作者**：Google AI Developers Forum / `@myessaylab1234`
- **发布日期**：2026-09-30 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区报告｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 确认旧版 `@google/gemini-cli` 个人访问权限于 2026 年 6 月 18 日终止，且 `cloudcode-pa.googleapis.com` 返回的 `TOS_VIOLATION` 申诉后长期无法同步解封。

### [46] Antigravity CLI Agent reliability issue (#176775)（对应 `[^46]`）
- **原始 URL**：https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775
- **机构 / 作者**：Google AI Developers Forum / `@Pitt_zhang`
- **发布日期**：2026-08-01 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业社区审计报告｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 记录 `agy` CLI 在复杂多步工程中遗忘项目指令、未完成验证即汇报完成、前后端接口路径错位及引入认证绕过回退分支的案例。

### [47] Google Antigravity FAQ 政策更新与停止第三方工具連携说明（对应 `[^47]`）
- **原始 URL**：https://note.com/horizon_it00/n/nded285d721d1
- **机构 / 作者**：note.com / `horizon_it00`（佐藤賢治）
- **发布日期**：2026-10 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业技术博客｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 记录日本开发者因 Google 在 2026 年 10 月更新 Antigravity FAQ 禁止第三方编程智能体包装调用 `agy -p` 而被迫停用原有跨工具编排架构。

### [48] Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (`239,616` Compaction Threshold)（对应 `[^48]`）
- **原始 URL**：https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk
- **机构 / 作者**：Nous Research Hermes Agent 文档
- **发布日期**：2026-09-23 ｜ **访问核验日期**：2026-11-04
- **来源层级**：专业技术文档｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 记录为规避 Antigravity 256K 硬上限，将外部插件安全压缩阈值设定为 `239,616`（`256K - 16K output tokens`）。

### [49] mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management（对应 `[^49]`）
- **原始 URL**：https://github.com/mksglu/context-mode
- **机构 / 作者**：GitHub / `@mksglu`
- **发布日期**：2026 ｜ **访问核验日期**：2026-11-04
- **来源层级**：开源代码仓库｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 通过本地沙盒与 SQLite FTS5 全文索引拦截原始文件读取与长命令输出，仅向智能体返回检索后的精确片段，降低 70%–90% 上下文消耗。

### [50] AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE（对应 `[^50]`）
- **原始 URL**：https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor
- **机构 / 作者**：GitHub / `@AGI-is-going-to-arrive`
- **发布日期**：2026（更新至 `v1.13.0`）｜ **访问核验日期**：2026-11-04
- **来源层级**：开源代码仓库｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 支持 macOS、Linux、Windows 与 WSL（`v1.12.0+` / `v1.13.0`），自动发现 `agy` Hub 与语言服务进程并实时监控所有会话的上下文窗口水位。

### [51] dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app（对应 `[^51]`）
- **原始 URL**：https://github.com/dimitr-io/antigravity-desktop-context
- **机构 / 作者**：GitHub / `@dimitr-io`
- **发布日期**：2026-10-22 ｜ **访问核验日期**：2026-11-04
- **来源层级**：开源代码仓库｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "this panel reads the token breakdown that Antigravity records for every model call, the same data the CLI's `/context` uses. The accounting is ported from the CLI binary itself, so the categories, totals, checkpoints and files match `agy /context` on the same conversation."

### [52] weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI（对应 `[^52]`）
- **原始 URL**：https://github.com/weby-homelab/antigravity-cli-statusline
- **机构 / 作者**：GitHub / Weby Homelab
- **发布日期**：2026 ｜ **访问核验日期**：2026-11-04
- **来源层级**：开源代码仓库｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "The Bash renderer supports Linux and macOS; the PowerShell renderer supports Windows. Both read the CLI’s JSON payload from standard input and supplement it with local Git and host data when available."

### [53] oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit ("The IDE for Your Coding Agent")（对应 `[^53]`）
- **原始 URL**：https://github.com/oraios/serena
- **机构 / 作者**：GitHub / Oraios AI
- **发布日期**：2026 ｜ **访问核验日期**：2026-11-04
- **来源层级**：开源代码仓库｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "Serena provides essential semantic code retrieval, editing, refactoring and debugging tools that are akin to an IDE's capabilities, operating at the symbol level and exploiting relational structure. It integrates with any client/LLM via the model context protocol (MCP)."

### [54] xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server)（对应 `[^54]`）
- **原始 URL**：https://github.com/xdotech/goatlas
- **机构 / 作者**：GitHub / `@xdotech`
- **发布日期**：2026 ｜ **访问核验日期**：2026-11-04
- **来源层级**：开源代码仓库｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > "**GoAtlas** is an AI-powered code intelligence platform that helps LLMs and developers deeply understand large codebases — combining multi-language AST parsing, a Neo4j knowledge graph, pgvector semantic search, and a pluggable LLM backend, all exposed via MCP."

### [55] Kevin Hou — "Get Out of the Model's Way" (AI Engineer Summit Presentation & Speaker Profile)（对应 `[^55]`）
- **原始 URL**：https://www.youtube.com/watch?v=xsVD9_cJNYs
- **机构 / 作者**：AI Engineer Summit / YouTube（及演讲者档案 `https://ai.engineer/speakers/kevin-hou`、个人主页 `https://khou22.com`）｜演讲者：Kevin Hou（Google DeepMind Antigravity 产品工程负责人，`@kevinhou22` / GitHub `@khou22`）
- **发布日期**：2026 ｜ **访问核验日期**：2026-11-04
- **来源层级**：一手来源（高管公开演讲）｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 阐述 Antigravity 的核心设计哲学 "Get Out of the Model's Way"（不要在模型周围构建死板的脚手架结构限制模型，随着基座模型能力提升，过度设计的硬性限制是贬值资产，应让模型自主规划与呼吸）。

### [56] Varun Mohan Education Qualifications: How an Indian-Origin MIT Engineer Secured Google's $2.4 Billion AI Investment（对应 `[^56]`）
- **原始 URL**：https://timesofindia.indiatimes.com/education/news/varun-mohan-education-qualifications-how-an-indian-origin-mit-engineer-secured-googles-2-4-billion-ai-investment/articleshow/122405531.cms
- **机构 / 作者**：Times of India
- **发布日期**：2025-07 ｜ **访问核验日期**：2026-11-04
- **来源层级**：权威媒体报道｜ **置信度**：高
- **关键原文与统计口径摘录**：
  > 记录 2025 年 7 月 Google 以 24 亿美元（$2.4 Billion）达成 Windsurf（Codeium）技术许可协议，联合创始人兼 CEO Varun Mohan 及核心工程团队加入 Google DeepMind 主导智能体编程平台研发。

### [57] Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)（对应 `[^57]`）
- **原始 URL**：https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191
- **机构 / 作者**：Google AI Developers Forum / 提问：`@Sharper`；官方回复：`@chunduriv`（Veeranjaneyulu Ch，论坛 staff 及版主）
- **发布日期**：2026-11-04 16:01:05 UTC（官方回复：2026-11-04 23:39:13 UTC）｜ **访问核验日期**：2026-11-05
- **来源层级**：一手来源（Google 官方人员论坛回复）｜ **置信度**：高
- **原始快照**：[`evidence/raw/forum_198191.json`](../../evidence/raw/forum_198191.json)（Discourse 主题 JSON，访问于 2026-11-05）
- **关键原文与统计口径摘录**：
  > `@Sharper`："In practice, the desktop app forces cache-losing and information-lossy compaction at 250K of used context... Choosing the context compaction size must be at least an option for users."；`@chunduriv`："This is a feature request for configurable context compaction in the Antigravity App. As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. That threshold is a fixed default and isn't user-configurable yet. ... We've passed your request for a user-selectable compaction size to the engineering team for review." 官方给出的临时缓解办法：在自然断点手动执行 `/compact`；把稳定的初始资料放在工作区文件中；排除构建产物、依赖与生成文件。

---

## 3. 备查与交叉验证补充来源（已核验留档，零丢弃）

- **[S1]** [Question about personal wrapper around official Antigravity CLI headless mode (#178472)](https://discuss.ai.google.dev/t/question-about-personal-wrapper-around-official-antigravity-cli-headless-mode/178472) — Google AI Developers Forum（作者：`@Ahmad3`），发布日期：2026-08-16（访问日期：2026-11-04｜类型：专业社区｜留档说明：早期询问私有单用户 Web UI / Telegram 包装 `agy -p` 合规性的代表性帖子）
- **[S2]** [Is a local single-user orchestrator allowed to invoke the official agy -p CLI? (#179013)](https://discuss.ai.google.dev/t/is-a-local-single-user-orchestrator-allowed-to-invoke-the-official-agy-p-cli/179013) — Google AI Developers Forum（作者：`@JJPop`，回复：`@Ambati_Rajendra`），发布日期：2026-08-20（访问日期：2026-11-04｜类型：一手官方论坛｜留档说明：验证 `agy 1.1.12` 时期用户对第 6 条的早期垂询）
- **[S3]** [Google AI Studio Gemini API invoking a consumer-authenticated agy CLI is allowed? (#194944)](https://discuss.ai.google.dev/t/google-ai-studio-gemini-api-invoking-a-consumer-authenticated-agy-cli-is-allowed/194944) — Google AI Developers Forum（提问：`@oooolee`，官方回复：`@Ambati_Rajendra`），发布日期：2026-10-26（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜留档说明：确认禁止从外部 Gemini API 函数工具反向触发个人订阅登录的 `agy` CLI）
- **[S4]** [Can I let my grok bot use agy cli on their computer? (#190429)](https://discuss.ai.google.dev/t/can-i-let-my-grok-bot-use-agy-cli-on-their-computer/190429) — Google AI Developers Forum（提问：`@Chua_Poplar`，官方回复：`@Ambati_Rajendra`），发布日期：2026-10-19（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜留档说明：确认跨 VM 用第三方 Bot 驱动 `agy` 会触发自动 `403 TOS_VIOLATION` 封禁）
- **[S5]** [AsabaLazy/aur-ASLZ: Arch Linux AUR Repository containing `antigravity-cli-fix-bin`](https://github.com/AsabaLazy/aur-ASLZ) — GitHub（作者：`@AsabaLazy`），发布日期：2026-11-02（访问日期：2026-11-04｜类型：开源代码仓库｜留档说明：每日两次自动同步上游并预打 256K 解锁补丁的 Arch Linux 仓库）
- **[S6]** [mcpservers.org — sshahzaiib/agy-bridge MCP Server](https://mcpservers.org/ja/servers/sshahzaiib/agy-bridge) — MCP Servers Directory（作者：`sshahzaiib`），发布日期：2026（访问日期：2026-11-04｜类型：专业技术目录｜留档说明：佐证社区早期开发的 `agy-bridge` MCP 桥接工具背景）
- **[S7]** [Claude Code vs Google Antigravity vs GitHub Copilot 料金・制限比較](https://www.r-jc.jp/news-blog/6256/claude-code-antigravity-%E6%AF%94%E8%BC%83copilot-%E9%81%95%E3%81%84%E6%96%99%E9%87%91%E8%A1%A8/) — R-JC News Blog，发布日期：2026（访问日期：2026-11-04｜类型：行业对比分析｜留档说明：横向对比 Claude Code、Antigravity 与 Copilot 的订阅定价与模型配额结构）
- **[S8]** [Agy unusable after false violation of Terms of Service (#185325)](https://discuss.ai.google.dev/t/agy-unusable-after-false-violation-of-terms-of-service/185325) — Google AI Developers Forum，发布日期：2026-09-27（访问日期：2026-11-04｜类型：专业社区报告｜留档说明：佐证个人账号误触发 `TOS_VIOLATION` 导致无法使用的案例）
- **[S9]** [BUG: AGY CLI Error 3501 (403 Forbidden) - Personal Account Wrongly Flagged as Enterprise (#185714)](https://discuss.ai.google.dev/t/bug-agy-cli-error-3501-403-forbidden-personal-account-wrongly-flagged-as-enterprise/185714) — Google AI Developers Forum，发布日期：2026-09-28（访问日期：2026-11-04｜类型：专业社区报告｜留档说明：佐证个人 `@gmail.com` 账号遭遇 `Error 3501` 403 拦截的故障背景）
