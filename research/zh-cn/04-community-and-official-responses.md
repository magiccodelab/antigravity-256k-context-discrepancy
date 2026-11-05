# 分册四：社区与官方对 256K 限制的应对

- **语言 / Language**：[English (../04-community-and-official-responses.md)](../04-community-and-official-responses.md) | **简体中文**
- **档案库全局导航**：[仓库总览](../../README.zh-CN.md) ｜ [调研总览 (dossier.md)](./dossier.md) ｜ [全量来源档案 (sources.md)](./sources.md) ｜ [静态逆向取证报告](../../evidence/zh-cn/evidence-report.md)
- **专题分册导航**：[01-事件背景与时间线](./01-incident-overview-and-timeline.md) ｜ [02-256K硬编码与逆向拆包](./02-256k-hardcoded-gates-and-binary-teardown.md) ｜ [03-agy-p政策反转与封号](./03-agy-p-headless-policy-reversal-and-bans.md) ｜ **04-社区与官方应对** ｜ [05-原文引用与记录汇总](./05-verbatim-quotes-and-evidence-vault.md)
- **研究 ID (UUID)**：`df866cf1-da40-4b82-8084-34c6b0729544` ｜ **调研时间**：2026-11-04（2026-11-05 更新）｜ **范围**：用户、社区开发者与 Google 官方人员如何应对 256K 压缩阈值，以及这些应对说明了该限制位于何处

---

## 1. 概要

桌面端与 CLI 都没有提供修改压缩阈值的公开设置或菜单选项。自 CLI `v1.5.0`（2026-11-04）起，二进制重新读取未公开的环境变量 `AGY_CLI_AUTO_COMPACT_OVERRIDE`，但在 `v1.5.1` 中它只在很窄的条件下生效（见 §2.4）。Google 官方人员已于 2026-11-04 确认，约 250K 的阈值“is a fixed default and isn't user-configurable yet”（是固定默认值，暂不支持用户配置）[^57]。本分册记录的内容都是对这一空缺的应对：

1. **社区二进制补丁在客户端去掉了限制。** 补丁只修改本地 `agy` 二进制中的三处位置，不涉及服务端。它能生效，进一步印证了分册二通过反汇编得出的结论：256K 上限由客户端代码执行[^1][^2]。
2. **官方建议把该限制当作只能绕开的约束。** Google 官方人员建议手动执行 `/compact`、压缩后重新读取文件、减少上下文中的噪音，但没有提供提高阈值的办法[^43][^57]。
3. **Google 自己的工程团队也围绕压缩做了设计。** Google Developers Blog 的 ADK 案例中，团队在加入仓库技能文件和 `/goal` 模式后，才完成 11 小时无人值守运行，使任务目标在多次压缩后仍得以保留[^16]。
4. **第三方补上了产品缺失的可见性。** 上下文监控插件、桌面端 `/context` 面板和减少上下文占用的 MCP 服务器之所以出现，是因为客户端既不显示也不允许控制上下文用量[^44][^49][^50][^51][^52][^53][^54]。
5. **跨工具使用的边界由论坛答复划定。** `agy -p` 口径变更之后，官方人员说明了消费者订阅下仍被允许的集成方式（口径变更本身见分册三）[^34][^36][^37][^38][^41]。

---

## 2. 社区二进制补丁：限制位于客户端的佐证

### 2.1 `patch_agy.py` 修改了什么
在 GitHub Issue `#1245` 中，系统工程师 `@LiraNuna` 发布了 **[`patch_agy.py`](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26)**。该脚本通过二进制中的 Go `pclntab` 符号表定位函数，不依赖固定地址，因此可用于 `agy 1.3.5`、`1.4.x` 与 `1.5.0`[^1][^2]。它修改的正是分册二认定的三道硬编码门禁。本档案只记录每处修改的作用，不转载替换字节、脚本源码或运行命令。

| 修改 | 目标函数（`v1.5.0` 文件偏移） | 原始行为 | 修改后的效果 |
|---|---|---|---|
| 1 | `cli/model/model.SelectorModel.offersContextWindow`（`0x9e92140`） | 仅对 `"Gemini 4 Argon"` 显示上下文档位选择器 | 所有模型都显示选择器 |
| 2 | `cli/store/store.(*Store).ContextWindow`（`0x9c49fe0`） | 再次校验同一白名单，丢弃其他模型已保存的档位 | 所有模型都保留已保存的 `512K` / `900K` 档位 |
| 3 | `cortex/utils/utils.GetCheckpointConfig`（`0x7fe5802`） | `cmp $0x3e800, %edx; mov $0x3e800, %esi; cmovb %edx, %esi` 将阈值钳制为 `min(计算值, 256000)` | 阈值采用计算值 `max_context - 40,000` |

三处修改都在客户端代码中。能用这种方式去掉钳制，与分册二的结论一致：256K 上限由客户端设定，而不是由服务端返回[^1][^2]。打补丁后服务端如何处理超过 256K 的请求，本档案尚未核实（见 [dossier.md §10](./dossier.md)）。

### 2.2 社区打包与第三方阈值
- 2026-11-02，开发者 `@AsabaLazy` 在 Issue `#1245` 宣布了 Arch Linux AUR 软件包 `antigravity-cli-fix-bin`，该包跟随上游版本并已应用补丁[^1]。
- Nous Research 的 Hermes Agent 插件文档把自身的压缩阈值设为 `239,616` Tokens（`256K - 16K` 输出余量），以避开 `agy` 内部的压缩[^48]。

### 2.3 社区记录的代价与风险
- **配额代价**：`@Emily2040` 在 Issue `#1245` 中指出，在 `512K`（`1.3x` 倍率）或 `900K`（`1.8x`）下，一次前缀缓存失效后的全量未缓存重发，消耗的配额约为 `256K` 下同类事件的 **4 倍**[^4]。这是支持“默认提前压缩”最具体的成本理由，同时也说明阈值更适合做成可配置项，而非固定值。
- **维护成本**：`agy` 每次自动更新都会替换自身二进制，本地修改随之失效，需要重新处理[^1][^4]。
- **账号风险**：修改官方客户端可能与 Antigravity 服务条款冲突。本档案不建议这样做。

### 2.4 CLI `v1.5.0`–`v1.5.1` 恢复的环境变量
`AGY_CLI_AUTO_COMPACT_OVERRIDE` 在 `v1.3.5`–`v1.4.x` 中被编译期抹除，CLI `v1.5.0` 重新读取它，而 `v1.5.0` 与 `v1.5.1` 的 `CHANGELOG.md` 都没有提到这一变化[^1][^11]。对官方 `v1.5.1` 二进制（SHA-256 `72738cf0…26669`）的静态分析显示，它的适用范围很窄：

- **只认绝对 Token 数**：解析函数支持纯数字 Token 数和 `80%` 这类百分比两种写法，但百分比写法需要调用方传入大于 0 的基数。`v1.5.1` 中的两个调用点传入的基数都是 `0`（VA `0x9c34c9a` 与 VA `0x9e875aa` 处的调用前均为 `xor %eax,%eax`），因此百分比写法不会生效，也就无法用一个值适配所有模型。
- **只对白名单模型生效，且优先级最低**：`Store.ContextWindow()` 先检查模型白名单，再读取 `settings.json` 中保存的档位，最后才读取环境变量。非 Argon 模型走不到这一步；已保存的档位也会优先于环境变量。
- **与钳制指令的关系尚未验证**：`GetCheckpointConfig` 中的 `min(计算值, 256000)` 钳制仍然存在。大于 `256,000` 的值能否越过这道钳制，本档案尚未实测；跑一次跨过 256K 的长会话，再读取会话数据库中的 `max_context_tokens` 字段即可确认。

总之，`v1.5.0` 恢复的是一个未公开的环境变量，在 `v1.5.1` 中最多只对 Gemini 4 Argon、以固定 Token 数的形式生效。它既不是面向用户的设置，也还不能确认可以绕过 256K 钳制。

---

## 3. Google 官方人员给出的建议

| 日期 | 帖子 | 官方人员 | 建议内容 |
|---|---|---|---|
| 2026-09-25 | `#184694` | `@chunduriv` | 需要查看上下文用量时，在 CLI 中使用 `/context`；桌面端没有对应面板[^44] |
| 2026-10-20 | `#189476` | `@chunduriv` | 在 `GEMINI.md` 或 `.agents/rules/` 中添加常驻规则，要求 agent 在出现 `# Resuming from a compaction` 后重新读取目标文件；执行 `/compact` 时附上保留准确标识符与文件路径的指令[^43] |
| 2026-11-04 | `#198191` | `@chunduriv` | 约 250K 阈值是固定默认值，不能由用户配置；在自然断点手动执行 `/compact`，把参考资料放在工作区文件中，排除构建产物和依赖。“可选阈值”的需求已转交工程团队，未给出时间表[^57] |

三条答复都把阈值当作既定条件，把避免信息丢失的责任交给用户，也都没有回应标称的 1M 窗口。

---

## 4. Google 自己的案例：围绕压缩进行设计

2026 年 10 月，Google Developers Blog 发表《[Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/)》，记录了对 `google/adk-python`（约 205,000 行代码、800 个源文件、16,892 个单元测试）的两轮运行[^16]：

| | 第一轮（无仓库指引） | 第二轮（`.agents/skills` + `/goal`） |
|---|---|---|
| 运行时长 | 约 2 小时后停止 | 无人值守运行 11 小时（晚 9:11 至次日早 8:19） |
| 改代码之前 | 第 7 分钟写基准脚本，第 70 分钟开始改代码 | 按需读取 12 份技能文件，第 10 分钟完成基线测试 |
| 公开 API | 破坏了公开的 `AuthConfig` 结构 | `AuthConfig` 保持不变 |
| 测试 | 只运行附近的 216 个测试 | 新增回归测试并运行全部 16,892 个 |

文章把长时间运行归功于 `/goal` 模式在执行框架层保存任务目标，使其不会随对话压缩而丢失。可见 Google 自己的团队也把压缩当作需要工程手段应对的约束，而不是无关紧要的问题[^16]。

---

## 5. 为填补空缺而出现的第三方工具

| 工具 | 补充的能力 | 来源 |
|---|---|---|
| `weby-homelab/antigravity-cli-statusline` | 在 CLI 状态栏实时显示上下文用量、模型及 5 小时 / 7 天配额 | [^52] |
| `dimitr-io/antigravity-desktop-context` | 为原本没有该功能的 Antigravity 2.0 桌面端提供 `/context` 式侧边栏 | [^44][^51] |
| `AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor` | 在 Antigravity IDE 中显示各会话的上下文水位 | [^50] |
| OTLP 导出 Hook（`@UesugiErii`，Issue `#366`） | 每轮把 Token 计数导出到 OpenTelemetry，用于成本审计 | [^7] |
| `mksglu/context-mode` | 在本地沙盒中索引大段工具输出，只返回匹配片段；作者报告可减少 70%–90% 的上下文占用 | [^49] |
| `oraios/serena` | 用 LSP 符号级查询代替整文件读取 | [^53] |
| `xdotech/goatlas` | 用 AST 与调用图查询代替全仓库读取 | [^54] |

这些工具的目的相同：让会话停留在一个用户在产品中既看不到、也改不了的阈值之下。

---

## 6. 官方人员划定的跨工具集成边界

`agy -p` 口径变更之后（见分册三），官方人员的答复把以下方式描述为消费者订阅下允许的用法。这里按政策表态记录，不作为使用建议：

1. **ACP 客户端**：在支持 Agent Client Protocol 的客户端中用个人订阅登录，并使用 Google 自己的 ACP 服务（自 `v1.5.0` 起内置于 `agy`），属于受支持的用法[^11][^34][^36][^41]。
2. **API Key 模式**：`agy` 使用 Gemini API Key 认证时不建立消费者账号会话，因此消费者订阅对第三方编排的限制不适用[^36][^38]。
3. **由本人触发的运行**：借助其他 AI 助手起草的文件，不会让用户自己的确定性脚本变成被禁止的封装，前提是该助手从不调用 `agy`，且每次运行都由账号所有者本人发起[^37]。

通过 `stdio` 运行 ACP 与通过 `stdio` 运行 `agy -p`，在技术上都是同一类本地子进程交互，前者被允许，后者却被禁止。总览报告把这一点列为尚未解决的矛盾（见 [dossier.md §7](./dossier.md)）。

---

## 本分册引用来源清单（脚注定义）

[^1]: [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) — GitHub `google-antigravity/antigravity-cli`（作者：`Burntgogi`，评论：`NuCl34R`、`Circumsized`、`Emily2040`、`LiraNuna`、`AsabaLazy`），发布日期：2026-10-14 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与代码逆向实证｜置信度：高）
[^2]: [Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) — GitHub Gist（作者：`LiraNuna`），发布日期：2026-10-23（访问日期：2026-11-04｜类型：一手逆向工程补丁源码｜置信度：高）
[^4]: [[Bug]: Five measured places the CLI spends time and tokens the work does not need: serial tool calls, uncached full-context calls and 70 s compactions, 12 s headless overhead, summary store rewritten every 1.5 s, build-time CPU instruction choice (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) — GitHub `google-antigravity/antigravity-cli`（作者：`Emily2040`），发布日期：2026-10-19 至 2026-10-25（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与基准实测｜置信度：高）
[^7]: [Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue #366)](https://github.com/google-antigravity/antigravity-cli/issues/366) — GitHub `google-antigravity/antigravity-cli`（作者：`UesugiErii` 等），发布日期：2026-06-11 至 2026-10-20（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^11]: [Google Antigravity CLI Releases (`v1.4.2` – `v1.5.0`)](https://github.com/google-antigravity/antigravity-cli/releases) — GitHub `google-antigravity/antigravity-cli`，发布日期：2026-10-18 至 2026-11-04（访问日期：2026-11-04｜类型：一手官方发版日志｜置信度：高）
[^16]: [Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/) — Google Developers Blog，发布日期：2026-10（访问日期：2026-11-04｜类型：一手官方工程博客｜置信度：高）
[^34]: [Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-10-07 至 2026-10-08（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^36]: [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-23 至 2026-10-27（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^37]: [Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998)](https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-26（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^38]: [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-10-19 至 2026-10-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^41]: [Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131)](https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-31 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^43]: [Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476) — Google AI Developers Forum（提问：`@Hello_there`，官方回复：`@chunduriv`），发布日期：2026-10-15 至 2026-10-20（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^44]: [Feature request: Show context window usage in Antigravity Desktop (#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694) — Google AI Developers Forum（提问：`@Soro`，官方回复：`@chunduriv`，插件：`@dimitr-io`），发布日期：2026-09-24 至 2026-10-22（访问日期：2026-11-04｜类型：一手官方论坛回复｜置信度：高）
[^48]: [Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (`239,616` Compaction Threshold)](https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk) — Nous Research Hermes Agent 文档，发布日期：2026-09-23（访问日期：2026-11-04｜类型：专业技术文档｜置信度：高）
[^49]: [mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management](https://github.com/mksglu/context-mode) — GitHub（作者：`mksglu`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^50]: [AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE](https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor) — GitHub（作者：`AGI-is-going-to-arrive`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^51]: [dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context) — GitHub（作者：`dimitr-io`），发布日期：2026-10-22（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^52]: [weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI](https://github.com/weby-homelab/antigravity-cli-statusline) — GitHub（作者：`weby-homelab`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^53]: [oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit ("The IDE for Your Coding Agent")](https://github.com/oraios/serena) — GitHub（作者：Oraios AI），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^54]: [xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server)](https://github.com/xdotech/goatlas) — GitHub（作者：`xdotech`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^57]: [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) — Google AI Developers Forum（提问：`@Sharper`，官方回复：`@chunduriv`），发布日期：2026-11-04（访问日期：2026-11-05｜类型：一手官方人员论坛回复｜置信度：高）
