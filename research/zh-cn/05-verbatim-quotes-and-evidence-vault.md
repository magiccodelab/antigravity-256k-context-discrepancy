# 分册五：核心人物与原文引用汇总（原文 + 中文译文 + 精确出处对照）

- **语言 / Language**：[English (../05-verbatim-quotes-and-evidence-vault.md)](../05-verbatim-quotes-and-evidence-vault.md) | **简体中文**
- **档案库全局导航**：[仓库总览](../../README.zh-CN.md) ｜ [调研总览 (dossier.md)](./dossier.md) ｜ [全量来源档案 (sources.md)](./sources.md) ｜ [静态逆向取证报告](../../evidence/zh-cn/evidence-report.md)
- **专题分册导航**：[01-事件背景与时间线](./01-incident-overview-and-timeline.md) ｜ [02-256K硬编码与逆向拆包](./02-256k-hardcoded-gates-and-binary-teardown.md) ｜ [03-agy-p政策反转与封号](./03-agy-p-headless-policy-reversal-and-bans.md) ｜ [04-社区与官方应对](./04-community-and-official-responses.md) ｜ **05-原文引用与记录汇总**
- **研究 ID (UUID)**：`df866cf1-da40-4b82-8084-34c6b0729544` ｜ **调研时间**：2026-11-04 ｜ **用途**：供技术报道、学术研究或合规分析时直接引用（核对原始页面与 JSON 记录）

---

## 1. 关于 256K (`0x3e800`) 硬编码截断、70 秒黑盒压缩与 3 倍配额消耗的一手原话

### 引语 1.1：`@LiraNuna` 揭露二进制第一道门禁与 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 编译期抹除
- **出处**：GitHub `google-antigravity/antigravity-cli` Issue `#1245` Comment #5（作者：`@LiraNuna`，2026-10-22 16:37:11 UTC）[^1]
- **英文原文**：
  > *“In the `/model` menu, there is already full UI and backend support for choosing the compaction limit: `Context: Default (256K) | 256K | [512K] | 900K` (cycled with `Tab`). However, in `SelectorModel.offersContextWindow`: The client checks the selected model against an internal whitelist slice. In the release binary, this slice is hardcoded to only contain `"Gemini 4 Argon"`. For other models (e.g. `Gemini 3.8 Flash`), `offersContextWindow` returns `false`, which completely hides the `Context` chip and causes the `Tab` key handler to be ignored.*
  > *Why `AGY_CLI_AUTO_COMPACT_OVERRIDE` does nothing: In `store.AutoCompactOverride()`, the code was designed to parse either percentages (e.g. `80%`) or integer token counts. However, in the compiled release binaries, the call to `os.Getenv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` was compiled out / inlined into an empty string `""` (`xor %eax, %eax; xor %ebx, %ebx; call strings.TrimSpace`). Because `strconv.ParseInt("", 10, ...)` always fails, `AutoCompactOverride()` unconditionally returns an empty string (no override).”*[^1]
- **中文精译**：
  “在 `/model` 菜单中，其实已经具备选择压缩上限的完整 UI 和后端支持：`Context: Default (256K) | 256K | [512K] | 900K`（通过 `Tab` 键循环切换）。然而，在 `SelectorModel.offersContextWindow` 中：客户端会将所选模型与一个内部白名单切片进行比对。在发布版二进制文件中，该切片被硬编码为仅包含 `"Gemini 4 Argon"`。对于其他模型（如 `Gemini 3.8 Flash`），`offersContextWindow` 返回 `false`，这会彻底隐藏 `Context` 标签并导致 `Tab` 按键处理函数被忽略。
  为什么 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 毫无作用：在 `store.AutoCompactOverride()` 中，代码原本设计为解析百分比（如 `80%`）或整数 Token 数。然而，在编译后的发布版二进制文件中，对 `os.Getenv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` 的调用被编译裁掉/内联成了一个空字符串 `""`（`xor %eax, %eax; xor %ebx, %ebx; call strings.TrimSpace`）。因为 `strconv.ParseInt("", 10, ...)` 总是失败，`AutoCompactOverride()` 会无条件返回空字符串（无覆盖）。”

### 引语 1.2：`@LiraNuna` 揭露第二道门禁与 Cortex 引擎 `0x3e800`（256,000）汇编级硬截断
- **出处**：GitHub `google-antigravity/antigravity-cli` Issue `#1245` Comment #6（作者：`@LiraNuna`，2026-10-23 01:37:57 UTC）[^1]
- **英文原文**：
  > *“1. `Store.ContextWindow()` drops the selection for non-Argon models: Even though the UI saves `"contextWindow": "512K"` into `settings.json`, `store.(*Store).ContextWindow()` contains a second, identical whitelist check against `"Gemini 4 Argon"`. For all other models (e.g. `Gemini 3.8 Flash`), it aborts and returns `("", 0)` without ever reading the saved setting... Consequently, `Manager.syncContextWindow()` calls `backend.SetContextWindow("")`, and the backend never receives the 512K/900K selection.*
  > *2. Cortex has a hardcoded 256K (`0x3e800`) ceiling on compaction triggers: Even with the backend receiving the context window limit, Cortex's checkpoint calculator (`cortex/utils/utils.GetCheckpointConfig`) explicitly clamps the compaction trigger (`TokenThreshold`) for conversational cascades:*
  > `7861894: mov 0x4c(%rsp), %edx # Calculated threshold (context - 40,000 margin)`
  > `7861898: cmp $0x3e800, %edx # 0x3e800 == 256,000 tokens`
  > `786189e: mov $0x3e800, %esi`
  > `78618a3: cmovb %edx, %esi # Clamps to min(calculated, 256000)`
  > `78618a6: mov %esi, 0x50(%rsp) # Writes 256,000 to TokenThreshold!`
  > *Because of `cmovb`, any calculated threshold greater than 256,000 tokens is clamped down to 256,000 tokens.”*[^1]
- **中文精译**：
  “1. `Store.ContextWindow()` 会丢弃非 Argon 模型的选择：尽管 UI 将 `"contextWindow": "512K"` 保存到了 `settings.json` 中，但 `store.(*Store).ContextWindow()` 包含针对 `"Gemini 4 Argon"` 的第二次完全相同的白名单检查。对于所有其他模型（如 `Gemini 3.8 Flash`），它会直接中止并返回 `("", 0)`，根本不读取已保存的设置……结果，`Manager.syncContextWindow()` 调用了 `backend.SetContextWindow("")`，后端永远收不到 512K/900K 的选择。
  2. Cortex 在压缩触发器上有一个硬编码的 256K（`0x3e800`）上限：即便后端收到了上下文窗口限制，Cortex 的检查点计算器（`cortex/utils/utils.GetCheckpointConfig`）也会显式地将多轮对话级联的压缩触发阈值（`TokenThreshold`）钳制在 `min(计算值, 256000)`……由于 `cmovb` 指令的存在，任何大于 256,000 Tokens 的计算阈值都会被强行压低到 256,000 Tokens。”

### 引语 1.3：`@Circumsized` 揭示 SQLite 数据库 `max_context_tokens = 256000` 及“有损压缩”与“静默截断”的区别
- **出处**：GitHub `google-antigravity/antigravity-cli` Issue `#1245` Comment #2（作者：`@Circumsized`，2026-10-15 19:35:43 UTC）[^1]
- **英文原文**：
  > *“Every model call writes a `ContextWindowMetadata` record into the conversation database at `~/.gemini/antigravity-cli/conversations/<session-id>.db`. On `agy 1.3.5` with Gemini 4 Argon, the records for calls immediately preceding the token drops in this report read: `max_context_tokens = 256000`, `truncation_reason = 2`, while `/context` reports a 1.0M window. So this is not the server rejecting anything, and it is not the model's window: it is a number the client writes into its own request metadata before sending. 256,000 is exactly 25% of 1,048,576 — a round fraction of the window, which points at a policy constant rather than a technical ceiling.*
  > *There appear to be two distinct mechanisms, and only one of them is compaction:*
  > *- **Compaction** — history is replaced by a prose summary, with the `CHECKPOINT: "# Resuming from a compaction..."` marker. This is the lossy case described in the report.*
  > *- **Pre-flight truncation** — as the conversation approaches `max_context_tokens`, earlier history is trimmed out of the outgoing request before it is sent. No summary is produced, no checkpoint marker appears, and the exact content is simply absent from that request onward.”*[^1]
- **中文精译**：
  “每次模型调用都会向 `~/.gemini/antigravity-cli/conversations/<session-id>.db` 会话数据库中写入一条 `ContextWindowMetadata` 记录。在搭载 Gemini 4 Argon 的 `agy 1.3.5` 上，本报告中 Token 骤降前夕的调用记录显示：`max_context_tokens = 256000`，`truncation_reason = 2`，而与此同时 `/context` 却报告 1.0M 窗口。因此，这不是服务器拒绝了任何内容，也不是模型的物理窗口限制：这是客户端在发送请求前写入自身请求元数据的数字。256,000 恰好是 1,048,576 的整整 25%——窗口的一个整数分数，这指向一个策略常数而非技术上限。
  目前似乎存在两种截然不同的机制，其中只有一种是压缩：
  - **压缩（Compaction）**——历史记录被替换为散文式摘要，并带有 `CHECKPOINT: "# Resuming from a compaction..."` 标记。这是报告中描述的有损情况。
  - **请求前截断（Pre-flight truncation）**——当对话接近 `max_context_tokens` 时，早期历史记录在发出请求前就被直接裁剪掉。不生成摘要，不出现检查点标记，从那次请求开始，精确内容就这样悄无声息地消失了。”

### 引语 1.4：Eddy Murias (`@murias002`) 对 310 个会话、65,613 次调用的实测总结
- **出处**：GitHub `google-antigravity/antigravity-cli` Issue `#878` Comment #2（作者：Eddy Murias `@murias002`，2026-10-05 00:51:10 UTC）[^3]
- **英文原文**：
  > *“Conversations: 310 | Model calls: 65,613 (86.6% on Gemini 3.8 Flash) | Input tokens per call: p50 136,487 · p90 229,000 · max 255,999 | Calls above 300k input tokens: 0 | Compaction events (drop > 40%): 276 | Context right before compaction: median 255,070 (p10 247,360 · p90 255,874) | Context right after compaction: median 45,304.*
  > *So on a model that accepts about 1M tokens, the session is always summarized at roughly 256k, and nothing in `~/.gemini/antigravity-cli/settings.json`, the CLI flags or the environment changes it.”*[^3]
- **中文精译**：
  “会话数：310｜模型调用次数：65,613（86.6% 在 Gemini 3.8 Flash 上）｜单次调用输入 Tokens：中位数 136,487 · p90 229,000 · 最大值 255,999｜超过 300k 输入 Tokens 的调用：0 次｜压缩事件（下降 > 40%）：276 次｜压缩前瞬间的上下文：中位数 255,070（p10 247,360 · p90 255,874）｜压缩后瞬间的上下文：中位数 45,304。
  因此，在一个支持约 100 万 Token 的模型上，会话总是被强制在大约 256K 处进行摘要压缩，无论是 `~/.gemini/antigravity-cli/settings.json`、CLI 参数还是环境变量都无法改变这一点。”

### 引语 1.5：`@Emily2040` 关于 31% 未缓存输入来自中途前缀失效及 70 秒黑盒压缩的实证
- **出处**：GitHub `google-antigravity/antigravity-cli` Issue `#1245` Comment #4 & Issue `#1291`（作者：`@Emily2040`，2026-10-19）[^1][^4]
- **英文原文**：
  > *“Data: across 15 headless stream-json logs on 1.4.0–1.4.2 with Gemini 4 Argon High (1,279 model calls), 22 `checkpoint` steps fired, every one with the preceding call's context between 235k and 268k tokens (`input + cache_read`), never higher. That is consistent with the 256,000 constant read out of the request metadata above. Each checkpoint took a median 70 s (35–224 s, one 602 s), reported no usage for the summary call, and was followed by a 45–70k uncached call before the cache rebuilt.”*[^1][^4]
- **中文精译**：
  “数据：在 1.4.0–1.4.2 版本使用 Gemini 4 Argon High 运行的 15 份无头 stream-json 日志（共 1,279 次模型调用）中，共触发了 22 个 `checkpoint`（压缩检查点）步骤，每一个检查点前一步调用的上下文（`input + cache_read`）都介于 235k 到 268k Tokens 之间，从未高于此值。这与上方从请求元数据中读出的 256,000 常数完全一致。每个检查点中位耗时高达 70 秒（35–224 秒，最高一次达 602 秒），没有报告摘要调用的任何 Token 用量，且随后都紧跟着一次 45k–70k 的未缓存调用来重建缓存。”

### 引语 1.6：`@Yiji_Kong` 关于 `Gemini 4 Argon` 按公开价格 3 倍（Web 纯输入 7.64–9.25 倍）消耗配额的实测与官方回应
- **出处**：Google AI Developers Forum 帖子 `#196351` Posts #1, #5, #6（作者：`@Yiji_Kong` & 官方人员 `@Maniraj_Chervirala`，2026-10-28 至 10-29）[^42]
- **英文原文（`@Yiji_Kong`）**：
  > *“Normalized by each model’s published API price, **Argon consumes exactly 3.0x more quota per dollar than Flash**. If Flash is billed at its published rate, the implied bucket size is ~150 USD, and Argon’s consumption matches **6.00 / 30.00 USD per 1M input/output tokens** (3x the advertised 2.00 / 10.00 launch price)... Context compaction model calls do not appear in the trajectory token metadata, so their quota cost is invisible to users.*
  > *In Step 1C, sending just `"Reply with only the word: OK"` (28 chars) in a fresh chat on 4 Argon (Low) burned `+0.00015849` of weekly quota (~0.32% of the 5-hour pool), which is **12% more** than 3.8 Flash (High) writing a full 4,013-word technical article on Paxos and Raft (`+0.00014157`).”*[^42]
- **英文原文（Google 官方人员 `@Maniraj_Chervirala` 回应）**：
  > *“Higher subscription quota consumption on Gemini 4 Argon compared to Gemini 3.8 Flash is expected: promotional per-token rates on https://ai.google.dev/gemini-api/docs/pricing apply only to pay-as-you-go API billing, whereas 5-hour and weekly subscription limits reflect available capacity, model compute, and total agent work rather than matching API token list prices... Fixed per-model quota multipliers are not published because subscription limits adjust dynamically with system capacity.”*[^42]
- **中文精译**：
  （`@Yiji_Kong`）：“按每个模型公布的 API 价格归一化后，**Argon 每美元消耗的配额恰好是 Flash 的 3.0 倍**。如果 Flash 按其公布的价格计费，则隐含的配额池大小约为 150 美元，而 Argon 的消耗相当于**每百万输入/输出 Tokens 6.00 / 30.00 美元**（宣传的 2.00 / 10.00 首发价的 3 倍）……上下文压缩模型调用不会出现在轨迹 Token 元数据中，因此它们的配额成本对用户是隐形的。在步骤 1C 中，在 4 Argon (Low) 的新对话中仅发送一句 `"Reply with only the word: OK"`（28 个字符）就烧掉了 `+0.00015849` 的周配额（约占 5 小时池的 0.32%），这比让 3.8 Flash (High) 写一篇完整的 4,013 词 Paxos 与 Raft 技术长文（`+0.00014157`）**还要多消耗 12%**。”
  （`@Maniraj_Chervirala`）：“Gemini 4 Argon 相比 Gemini 3.8 Flash 更高的订阅配额消耗是符合预期的：定价页面上的每 Token 促销费率仅适用于按量付费 API 计费，而 5 小时和周订阅限额反映的是可用容量、模型算力和总智能体工作量，而非匹配 API Token 挂牌价……固定的单模型配额乘数不予公布，因为订阅限额会随系统容量动态调整。”

### 引语 1.7：`u/Lberondroth` 对桌面端 `v2.29.0` 新增“上下文与配额指示器”的拆包评述
- **出处**：Reddit `r/google_antigravity` 帖子 `#1xn624i`（作者：`u/Lberondroth`，2026-11-03）[^17]
- **英文原文**：
  > *“Resets in... and Your plan's baseline contingent will refresh on... What you get instead is the Context and contingent Indicator, so you can soon watch your contingent drain in high definition, right next to the send button...”*[^17]
- **中文精译**：
  “‘距离重置还有……’以及‘您计划的基础配额将于……刷新’。你没有等来配额重置，取而代之的是一个上下文与配额指示器，好让你很快就能在发送按钮旁边，以高清画质眼睁睁看着自己的配额飞速流干……”

### 引语 1.8：`@chunduriv` 确认约 250K 压缩阈值“is a fixed default and isn't user-configurable yet”（2026-11-04 23:39 UTC）
- **出处**：Google AI Developers Forum 帖子 `#198191` Posts #1 & #2（提问：`@Sharper`，2026-11-04 16:01 UTC；官方人员：`@chunduriv`，2026-11-04 23:39 UTC）[^57]
- **英文原文（`@Sharper`）**：
  > *“In theory, Gemini models like Argos support 1M of context. In practice, the desktop app forces cache-losing and information-lossy compaction at 250K of used context... Choosing the context compaction size must be at least an option for users. Until then, please stop advertising that your products work with context sizes you then don't allow the user to actually use.”*[^57]
- **英文原文（Google 官方人员 `@chunduriv` 回应）**：
  > *“As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. **That threshold is a fixed default and isn't user-configurable yet.** We understand that for workflows that rely on a large, stable, cached starting context, compacting this early costs both cache efficiency and detail.”*
  > *“Keep noise out of the context: Exclude build artifacts, dependencies and generated files so more of the ~250K budget goes to the material you actually need. We've passed your request for a user-selectable compaction size to the engineering team for review.”*[^57]
- **中文精译**：
  （`@Sharper`）：“理论上，Argos 这类 Gemini 模型支持 1M 上下文。实际上，桌面端在已用上下文达到 250K 时就强制执行会丢失缓存、丢失信息的压缩……至少应当让用户自己选择压缩阈值。在那之前，请不要再宣传你们的产品支持用户实际用不上的上下文规模。”
  （`@chunduriv`）：“正如你所说，桌面端中的 Gemini 模型（例如 Gemini 4 Argon）目前会在约 250K Tokens 时自动压缩。**该阈值是固定默认值，暂不支持用户配置。**我们理解，对于依赖大量、稳定且已缓存的初始上下文的工作流来说，这么早压缩会同时损失缓存效率和细节。”“把噪音挡在上下文之外：排除构建产物、依赖和生成文件，让约 250K 的预算更多留给你真正需要的材料。我们已将你提出的‘用户可选压缩阈值’需求转交工程团队评估。”
- **证据要点**：
  这是本档案收录的第一条写明约 250K 压缩阈值、并承认其固定且不可由用户配置的官方书面答复。该数值与 `GetCheckpointConfig` 中的 `0x3e800`（`256,000`）钳制指令以及 SQLite 中记录的 `max_context_tokens = 256000` 一致（见引语 1.2 与 1.3）。回复把问题定性为功能请求而非缺陷，没有反驳用户“标称 1M 窗口实际无法使用”的说法，只提供了临时缓解办法：在自然断点手动执行 `/compact`、把参考资料放在工作区文件中、排除构建产物。回复只承诺转交工程团队评估，没有时间表，且仅针对桌面端；CLI 的 GitHub Issue（`#1245`、`#878`、`#1291`、`#1316`）中至今没有对应的官方表态。

---

## 2. 关于 `agy -p` 政策反转前：Google 官方人员确认“本地子进程调用合规”的原文记录

### 引语 2.1：`@Ambati_Rajendra` 确认开源工具调用 `agy -p` 不违反第 6 条（2026-09-17）
- **出处**：Google AI Developers Forum 帖子 `#183429` Post #2（官方人员：`@Ambati_Rajendra`，2026-09-17 12:06:16 UTC）[^24]
- **英文原文**：
  > *“Is calling `agy` headless allowed? **Yes.** Calling the official `agy` binary locally via `agy -p` is an officially supported feature built for programmatic scripting (see the Headless Mode Documentation). **It does not violate Section 6, provided your tool never extracts, stores, or intercepts the user’s OAuth credentials.** As noted in the Antigravity FAQ and Terms of Service, the restriction specifically targets unauthorized third-party clients intercepting tokens to proxy private endpoints.”*[^24]
- **中文精译**：
  “允许以无头模式调用 `agy` 吗？**是的。** 通过 `agy -p` 在本地调用官方 `agy` 二进制文件是为程序化脚本构建的官方受支持功能（参见无头模式文档）。**只要你的工具从不提取、存储或拦截用户的 OAuth 凭据，它就不违反第 6 条。** 正如 Antigravity FAQ 和服务条款中所述，该限制专门针对拦截令牌以代理私有端点的未经授权的第三方客户端。”

### 引语 2.2：`@Ambati_Rajendra` 确认由另一个 AI 编程工具通过脚本启动 `agy -p` 完全合规（2026-09-28）
- **出处**：Google AI Developers Forum 帖子 `#185494` Post #2（官方人员：`@Ambati_Rajendra`，2026-09-28 07:14:31 UTC）[^27]
- **英文原文**：
  > *“**Yes, your use case is permitted.** Running the official `agy` binary locally (even headless via a script) under your own account is compliant with the terms. The restriction against third-party software strictly applies to unauthorized tools (like OpenClaw) that intercept or extract your Antigravity OAuth tokens to bypass official clients. **As long as you are executing the unmodified `agy` CLI and not proxying your auth tokens into another application, you are good to go!**”*[^27]
- **中文精译**：
  “**是的，你的使用场景是允许的。** 在你自己的账号下于本地运行官方 `agy` 二进制文件（即使是通过脚本以无头模式运行）符合条款规定。针对第三方软件的限制严格适用于拦截或提取你的 Antigravity OAuth 令牌以绕过官方客户端的未经授权工具（如 OpenClaw）。**只要你执行的是未修改的 `agy` CLI 且没有将认证令牌代理到其他应用中，就完全没问题（you are good to go）！**”

### 引语 2.3：`@Rebbanpalli_Naveen` 确认 OpenAI Codex 调用 `agy.exe` 是官方受支持用法（2026-10-05）
- **出处**：Google AI Developers Forum 帖子 `#186952` Post #2（官方人员：`@Rebbanpalli_Naveen`，2026-10-05 10:24:45 UTC）[^31]
- **英文原文**：
  > *“**Yes, the workflow you described is completely fine to use with your personal Google AI Pro subscription.** Here are a few helpful details on how this aligns with our policies:*
  > *Section 6 & Official CLI Usage: Section 6 of the Google Antigravity Additional Terms of Service and our Third-Party Software FAQ are specifically intended to prevent third-party harnesses from extracting, proxying, or reusing Antigravity OAuth credentials to call private backend endpoints directly. **Having Codex invoke the official, unmodified `agy.exe` binary as a local process on your Windows PC—while authentication (`agy auth login`) and all backend communication stay strictly inside `agy.exe`—is a supported use of CLI Headless Mode.**”*[^31]
- **中文精译**：
  “**是的，你描述的工作流在你的个人 Google AI Pro 订阅下完全可以使用。** 以下是它如何符合我们政策的具体细节：
  第 6 条与官方 CLI 用法：《Google Antigravity 附加服务条款》第 6 条和我们的第三方软件 FAQ 专门旨在防止第三方框架提取、代理或重用 Antigravity OAuth 凭据来直接调用私有后端接口。**让 Codex 在你的 Windows 电脑上将官方、未修改的 `agy.exe` 二进制文件作为本地进程调用——同时身份验证（`agy auth login`）和所有后端通信严格保留在 `agy.exe` 内部——是 CLI 无头模式的受支持用法。**”

### 引语 2.4：`@chunduriv` 明确声明“自己运行脚本与 Claude Code 启动 `agy` 在政策上毫无区别”（2026-10-07 15:03 UTC，反转前 26.5 小时）
- **出处**：Google AI Developers Forum 帖子 `#186246` Post #2（官方人员：`@chunduriv`，2026-10-07 15:03:46 UTC）[^32]
- **英文原文**：
  > *“1. Is launching the official `agy` CLI as a local subprocess allowed under Section 6? **Yes.** Running the official, unmodified `agy` binary locally in headless mode (for example, `agy -p "<prompt>" --output-format json` or `stream-json`) using your own personal Google AI subscription sign-in for your own individual work is **permitted**.*
  > *2. Does the answer change if you launch the script yourself vs. Claude Code launching it? **No. From a policy standpoint, there is no difference whether the unmodified `agy` subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code)**, provided that:*
  > *- All Antigravity service access happens exclusively through the unmodified official `agy` binary (or the official Antigravity Python SDK).*
  > *- No OAuth tokens, cookies, or credentials are extracted, read, or passed to the external tool.*
  > *- The setup is strictly for your own personal, single-user workflow and not shared, pooled, or exposed as a multi-user service.”*[^32]
- **中文精译**：
  “1. 在第 6 条下，允许将官方 `agy` CLI 作为本地子进程启动吗？**允许（Yes）。** 使用你自己的个人 Google AI 订阅登录，为你自己的个人工作在本地以无头模式运行官方、未修改的 `agy` 二进制文件（例如 `agy -p "<prompt>" --output-format json` 或 `stream-json`）是**允许的**。
  2. 如果你自己启动脚本与由 Claude Code 启动它，答案会改变吗？**不会（No）。从政策角度来看，未修改的 `agy` 子进程究竟是由你自己的脚本、本地协调器，还是由另一个本地编程助手（例如 Claude Code）启动，没有任何区别**，前提是：
  - 所有 Antigravity 服务访问均完全通过未修改的官方 `agy` 二进制文件（或官方 Antigravity Python SDK）进行；
  - 没有任何 OAuth 令牌、Cookie 或凭据被提取、读取或传递给外部工具；
  - 该设置严格用于你自己的个人单用户工作流，且未被共享、池化或暴露为多用户服务。”

---

## 3. 关于 `agy -p` 政策反转后：变更口径、提示封号风险与确认未改 ToS 的原文记录

### 引语 3.1：`@chunduriv` 在 26.5 小时后突然自我否定并威胁封号（2026-10-08 17:34 UTC）
- **出处**：Google AI Developers Forum 帖子 `#187241` Post #2（同一位官方人员：`@chunduriv`，2026-10-08 17:34:41 UTC）[^34]
- **英文原文**：
  > *“While we cannot provide formal legal interpretations of the Terms of Service... Having Codex invoke `agy` CLI: While `agy` offers a documented headless mode (`agy -p`) for your own local scripts and CI pipelines, **we do not support using third-party coding agents (like Codex, Claude Code, OpenClaw, or OpenCode) to orchestrate or wrap `agy` under a consumer login. Routing third-party agent prompts or automated loops through Antigravity can trigger automated ToS enforcement and account suspension.**”*[^34]
- **中文精译**：
  “虽然我们无法提供对服务条款的正式法律解释……让 Codex 调用 `agy` CLI：虽然 `agy` 为你自己的本地脚本和 CI 流水线提供了文档化的无头模式（`agy -p`），但**我们不支持在个人消费者登录下使用第三方编程智能体（如 Codex、Claude Code、OpenClaw 或 OpenCode）来编排或包装 `agy`。通过 Antigravity 路由第三方智能体提示词或自动化循环可能会触发自动服务条款执法和账号封禁。**”

### 引语 3.2：`@Rebbanpalli_Naveen` 正式宣布“新 FAQ 废除并取代早期论坛回复”（2026-10-16）
- **出处**：Google AI Developers Forum 帖子 `#189905` Post #2（官方人员：`@Rebbanpalli_Naveen`，2026-10-16 12:35:16 UTC）[^35]
- **英文原文**：
  > *“Current Official Guidance: **The Antigravity FAQ reflects our current policy for Antigravity CLI headless mode and supersedes earlier forum comments where they differ.** ... Wrapping `agy` Inside Third-Party Coding Agents or Applications (Claude Code, Codex, OpenClaw, OpenCode) — Not Supported: **No** under consumer plans. Even if you invoke the official, unmodified `agy` binary as a local subprocess without extracting OAuth tokens or reverse-engineering internal endpoints, wrapping `agy` inside third-party coding agents or applications—or automating account rotation—is not supported under consumer plans (FAQ — Headless mode in third-party coding agents). Using third-party coding agents or tools (such as Claude Code, OpenClaw, or OpenCode) to access Antigravity violates our Terms of Service and can result in account suspension or termination.”*[^35]
- **中文精译**：
  “当前官方指南：**Antigravity FAQ 反映了我们当前针对 Antigravity CLI 无头模式的政策，凡与早期论坛评论存在差异之处，均取代早期评论。**……在第三方编程智能体或应用程序（Claude Code、Codex、OpenClaw、OpenCode）内部包装 `agy` —— 不支持：在个人消费者计划下**不允许**。即使你将官方、未修改的 `agy` 二进制文件作为本地子进程调用，且不提取 OAuth 令牌、不逆向工程内部端点，在个人消费者计划下将 `agy` 包装在第三方编程智能体或应用内部也是不支持的。使用第三方编程智能体或工具访问 Antigravity 违反我们的服务条款，并可能导致账号暂停或终止。”

### 引语 3.3：被付费用户 `@b_w` 与 `@Tora39` 当面质问后，`@Rebbanpalli_Naveen` 承认正式 ToS 从未修改（2026-10-26 & 10-27）
- **出处**：Google AI Developers Forum 帖子 `#191689` Posts #5, #6, #8（用户 `@b_w`、`@Tora39` 与官方人员 `@Rebbanpalli_Naveen`，2026-10-26 至 10-27）[^36]
- **英文原文（Post #5 by `@Rebbanpalli_Naveen`，2026-10-26 11:24 UTC）**：
  > *“Codex vs. OpenClaw & Why the Earlier Reply (#186952) Differs: There is no difference between Codex and OpenClaw, having either tool invoke `agy` / `agy.exe` is not supported under a consumer Google Account login. After that October 5 reply in #186952 was posted, the policy on using Antigravity CLI headless mode in third-party coding agents was updated to explicitly clarify that wrapping `agy` inside third-party coding agents or applications is not supported under consumer plans. This updated policy supersedes the earlier guidance in #186952 and applies equally to Codex, OpenClaw, Claude Code, and OpenCode.”*[^36]
- **英文原文（Post #6 by `@b_w` 追问证据，2026-10-26 20:08 UTC）**：
  > *“You wrote ‘After that October 5 reply in #186952 was posted, the policy on using Antigravity CLI headless mode in third-party coding agents was updated’ - Can you please point me to the evidence of this? I am unable to find any evidence of a policy update after October 5th. The link only directs me to a frequently asked questions page. The most recent terms of service updates I am able to find are from September.”*[^36]
- **英文原文（Post #8 by `@Rebbanpalli_Naveen` 承认未改 ToS，2026-10-27 12:31 UTC）**：
  > *“To clarify the timeline for `@b_w` and answer `@Tora39`’s question (a): **the Google Antigravity Terms of Service themselves were not modified after September; rather, the Official FAQ was updated in mid-October 2026 to add the section ‘Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?’** That FAQ entry reflects the current policy for consumer plans (Free, Plus, Pro, and Ultra)... **Earlier forum replies in #186246 and #186952 gave an overly broad interpretation before that FAQ entry was published.**”*[^36]
- **中文精译（Post #8）**：
  “为了向 `@b_w` 澄清时间线并回答 `@Tora39` 的问题 (a)：**《Google Antigravity 服务条款》本身在 9 月之后并未修改；而是《官方 FAQ》在 2026 年 10 月中旬进行了更新，新增了‘我可以在第三方编程智能体中使用 Antigravity CLI 无头模式或自动切换账号吗？’一节。** 该 FAQ 条目反映了当前针对个人消费者计划（Free、Plus、Pro 和 Ultra）的政策……**此前在 #186246 和 #186952 中的论坛回复在该 FAQ 条目发布之前给出了过于宽泛的解释。**”

### 引语 3.4：`@Ambati_Rajendra` 宣布连“通过本地文件传任务”也算违规（2026-10-28）
- **出处**：Google AI Developers Forum 帖子 `#190815` Post #2（官方人员：`@Ambati_Rajendra`，2026-10-28 13:10:46 UTC）[^38]
- **英文原文**：
  > *“No. Per the FAQ and Additional Terms, using `agy -p` as a secondary worker coordinated by a third-party coding agent like Claude Code isn’t permitted on consumer plans... **That still counts as third-party agent coordination. Passing tasks from Claude Code to `agy -p` through local files instead of stdin/stdout pipes still uses consumer quota as a backend worker for an external coding agent, so it requires a Gemini API key or Gemini Enterprise.**”*[^38]
- **中文精译**：
  “不允许。根据 FAQ 和附加条款，在个人消费者计划下，不允许将 `agy -p` 用作由 Claude Code 等第三方编程智能体协调的辅助 Worker……**那依然算作第三方智能体协调。通过本地文件而非 stdin/stdout 管道将任务从 Claude Code 传递给 `agy -p`，仍然是在将个人消费者配额用作外部编程智能体的后端 Worker，因此需要 Gemini API 密钥或 Gemini Enterprise。**”

---

## 本分册引用来源清单（脚注定义）

[^1]: [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) — GitHub `google-antigravity/antigravity-cli`（作者：`Burntgogi`，评论：`NuCl34R`、`Circumsized`、`Emily2040`、`LiraNuna`、`AsabaLazy`），发布日期：2026-10-14 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与代码逆向实证｜置信度：高）
[^3]: [[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878)](https://github.com/google-antigravity/antigravity-cli/issues/878) — GitHub `google-antigravity/antigravity-cli`（作者：`schalatap`，评论：`murias002` 等），发布日期：2026-08-26 至 2026-10-31（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与大规模会话统计｜置信度：高）
[^4]: [[Bug]: Five measured places the CLI spends time and tokens the work does not need (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) — GitHub `google-antigravity/antigravity-cli`（作者：`Emily2040`），发布日期：2026-10-19 至 2026-10-25（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^17]: [[Teardown / Binary Diff] Antigravity Updated to v2.29.0](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/) — Reddit `r/google_antigravity`（作者：`u/Lberondroth`），发布日期：2026-11-03（访问日期：2026-11-04｜类型：专业社区二进制拆包报告｜置信度：高）
[^24]: [Is it allowed for an open-source tool to call the official `agy` CLI in headless mode? (#183429)](https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-17（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^27]: [Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (#185494)](https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^31]: [Is invoking the official Antigravity CLI from Codex allowed? (#186952)](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-05（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^32]: [Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (#186246)](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-10-01 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^34]: [Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-10-07 至 2026-10-08（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^35]: [Clarification on consumer-plan policy for invoking official agy via local subprocess (#189905)](https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-16 至 2026-10-26（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^36]: [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-23 至 2026-10-27（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^38]: [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-10-19 至 2026-10-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^42]: [Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351)](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351) — Google AI Developers Forum（作者：`@Yiji_Kong`，官方回复：`@Maniraj_Chervirala`），发布日期：2026-10-28 至 2026-10-29（访问日期：2026-11-04｜类型：一手实测报告与官方论坛回复｜置信度：高）
[^57]: [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) — Google AI Developers Forum（提问：`@Sharper`，官方回复：`@chunduriv`），发布日期：2026-11-04（访问日期：2026-11-05｜类型：一手官方人员论坛回复｜置信度：高）
