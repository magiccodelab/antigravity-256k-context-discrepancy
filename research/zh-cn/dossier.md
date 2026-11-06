# 【Google Antigravity 256K 上下文硬编码限制与 `agy -p` 政策反转事件】调研总览报告

- **语言 / Language**：[English (../dossier.md)](../dossier.md) | **简体中文**
- **档案库全局导航**：[仓库总览 (../../README.zh-CN.md)](../../README.zh-CN.md) ｜ **第一部分：调研总览（本文件）** ｜ [第二部分：静态逆向与数据库取证报告](../../evidence/zh-cn/evidence-report.md)（[English](../../evidence/evidence-report.md)）
- **研究 ID (UUID)**：`df866cf1-da40-4b82-8084-34c6b0729544`
- **GitHub 仓库地址**：https://github.com/magiccodelab/antigravity-256k-context-discrepancy
- **目录与文件路径**：`research/zh-cn/dossier.md`（全量来源档案：[sources.md](./sources.md)）
- **调研时间**：2026-11-04（2026-11-05 更新）｜ **正文引用来源**：56 条 ｜ **信息截至**：2026-11

---

## 0. 专题分册导航

为避免单文件过长影响阅读与按需查阅，本目录在保留本总览报告（`dossier.md`）与来源索引（`sources.md`）的基础上，将技术分析、政策时间线、各方应对与原文引用按主题拆分为 **5 份独立专题分册**（所有文件共享统一的脚注编号 `[^1]`–`[^56]`）：

| 分册编号 | 专题文件名称与直达链接 | 核心涵盖内容与适用场景 |
| :---: | :--- | :--- |
| **总览** | **[dossier.md](./dossier.md)**（本文件） | 核心事实汇总、关键数据表、多方观点、56 条完整脚注定义 |
| **分册一** | **[01-incident-overview-and-timeline.md](./01-incident-overview-and-timeline.md)** | 事件背景、24 亿美元收购 Windsurf 团队与核心维护者背景、旧版 `gemini-cli` 关停、从 2025-07 至 2026-11-05 的完整时间线 |
| **分册二** | **[02-256k-hardcoded-gates-and-binary-teardown.md](./02-256k-hardcoded-gates-and-binary-teardown.md)** | 上下文窗口统计、`AGY_CLI_AUTO_COMPACT_OVERRIDE` 演变、三处二进制门禁（含 `0x3e800` 汇编指令）、70 秒压缩耗时、Argon 3 倍配额消耗（`#196351`）与 Desktop `v2.29.0` / CLI `v1.5.0–v1.5.1` 拆包 |
| **分册三** | **[03-agy-p-headless-policy-reversal-and-bans.md](./03-agy-p-headless-policy-reversal-and-bans.md)** | `agy -p` 无头模式政策变更经过：变更前连续 10 帖官方书面确认 vs. 26.5 小时内口径变更、确认未改 ToS 仅改 FAQ（`#191689`）、限制本地文件交接（`#190815`）及 `cloudcode-pa.googleapis.com` 账号申诉记录 |
| **分册四** | **[04-community-and-official-responses.md](./04-community-and-official-responses.md)** | 社区与官方对 256K 限制的应对：社区补丁 `patch_agy.py` 改了什么及其为何说明限制位于客户端、Google 官方人员的建议、Google 自己的 ADK 案例、第三方上下文工具，以及官方人员划定的跨工具边界 |
| **分册五** | **[05-verbatim-quotes-and-evidence-vault.md](./05-verbatim-quotes-and-evidence-vault.md)** | 原文引用与记录汇总：收录逆向报告、汇编注释、Google 工作人员变更前后论坛回复原文、ToS 与 FAQ 原文及中文译文 |
| **来源库** | **[sources.md](./sources.md)** | 全量 56 条正文引用来源 + 9 条备查来源的完整元数据与原始摘录 |

---

## 1. 一句话概览

> [!IMPORTANT]
> **官方最新进展（2026-11-06 11:50 UTC —— [论坛帖 #198430](https://discuss.ai.google.dev/t/1m-advertised-context-vs-a-256k-compaction-clamp-in-the-shipped-binaries-plus-quota-and-agy-p-policy-questions-desktop-2-29-0-cli-1-5-1/198430)）**：本档案结论发布到开发者论坛后（接口快照见 [`../../evidence/raw/forum_198430.json`](../../evidence/raw/forum_198430.json)；发帖 8 秒后被 `discourse_ai_spam` 隐藏，约 11 小时 47 分钟后恢复公开），论坛支持工程师 `@Rebbanpalli_Naveen` 回复：桌面端 2.29.0 与 CLI 1.5.1 默认在约 250K–256K Tokens 处自动压缩，“以维持延迟、可靠性和 Token 效率（to maintain latency, reliability, and token efficiency）”，目前不支持用户配置；已把“可配置压缩阈值”和“在 `/context` 中更清楚地显示触发点”的反馈转交工程团队；不公布分模型配额倍率；`agy -p` 适用《服务条款》第 6 条与 FAQ“共同适用”。这次给出的理由不再包含前一天 `#1432` 中提到的“每轮配额消耗”，回复也没有回应在文档和模型列表中写明 256K 限制、`base = 0` 解析缺陷以及受白名单控制的 512K/900K 档位。

> [!NOTE]
> **官方进展（2026-11-05 19:56–20:00 UTC —— [GitHub Issue #1432](https://github.com/google-antigravity/antigravity-cli/issues/1432)）**：在本档案库的反汇编证据作为 `[Bug]` 提交至官方仓库 [`google-antigravity/antigravity-cli#1432`](https://github.com/google-antigravity/antigravity-cli/issues/1432) 约 5 小时后（接口快照见 [`../../evidence/raw/issue_1432.json`](../../evidence/raw/issue_1432.json) 与 [`../../evidence/raw/issue_1432_comments.json`](../../evidence/raw/issue_1432_comments.json)），官方仓库协作者 `@chandrashakherkamasani` 认领了该工单，将其打上 `duplicate`、`feature request` 与 `feedback` 三个标签（未加 `bug`，但保持 `OPEN` 开启状态），并首次在 GitHub 上书面确认：
> 1. Antigravity CLI 在 1M（`1,048,576`）上下文模型上**有意（intentionally）**将默认自动压缩阈值设为 `256,000` Tokens，理由是智能体每一步都会重发当前活跃上下文，256K 用于平衡多步推理质量、单轮延迟与**每轮配额消耗**；256K 以上的用户可选档位暂未开放。
> 2. 承认 `/context` 目前显示的是所选模型的底层总容量（`1,048,576`）而非实际生效的 `256,000` 自动压缩阈值，同意这种展示方式会让用户误以为遇到了意外上限。

本报告汇总了 Google Antigravity（含桌面端 Antigravity 2.0 至 `v2.29.0`、Antigravity IDE 及命令行工具 `agy` 至 `v1.5.0–v1.5.1`）[^11][^12]在官方说明与 `/context` 面板标称 `Gemini 4 Argon` 及 `Gemini 3.8 Flash` 支持 100 万（`1,048,576`）Tokens 上下文窗口的情况下，客户端通过设置模型白名单以及在底层 Cortex 引擎 `cortex/utils/utils.GetCheckpointConfig` 中硬编码 `cmp $0x3e800, %edx; cmovb %edx, %esi` 汇编指令，将实际自动压缩触发阈值限制在 `256,000` Tokens（标称值的 25%），同时以公开 API 价格的 3.0 倍（Web 端纯输入为 7.64–9.25 倍）扣减 `Gemini 4 Argon` 订阅配额并内置 `512K`（`1.3x`）与 `900K`（`1.8x Expensive`）档位的技术记录[^1][^2][^3][^4][^18][^42]；以及 Google 官方支持工程师在开发者论坛连续 10 帖书面回复确认“第三方工具、Codex 或 Claude Code 通过本地子进程调用原版 `agy -p` 符合条款”后，在未修改正式《附加服务条款（ToS）》正文的情况下，于 10 月 8 日（距最后一次确认 71 分钟）改变口径、随后以 10 月中旬更新的 FAQ 页面为依据限制跨工具本地调用的完整记录，以及社区与官方对该限制的应对[^9][^13][^14][^31][^32][^34][^35][^36][^38]。

---

## 2. 范围与假设

- **时间范围**：回溯自 2025 年 7 月 Google 以 24 亿美元引进 Windsurf（Codeium）核心团队[^56]，覆盖 2026 年 2 月 27 日 `gemini-cli` 封号与二振出局制确立[^9]、2026 年 6 月 18 日旧版开源 `@google/gemini-cli` 个人访问关停[^10][^45]、2026 年 8–11 月 `256K` 硬编码截断逆向与 `agy -p` 政策反转全过程，数据核验截至 **2026-11-04**（桌面端 `v2.29.0`、CLI `v1.5.0`）[^11][^12][^17]。
- **产品与订阅范围**：涵盖 Google Antigravity 四大客户端表面（Antigravity 2.0 桌面端、Antigravity IDE、`agy` CLI、Python SDK）[^12]；涉及个人消费者订阅计划（Free、Google AI Plus、Google AI Pro $20/月、Google AI Ultra 5x/20x $200–$250/月）[^13][^35][^42]；涉及模型包括 `Gemini 4 Argon`、`Gemini 3.8 Flash`、`Gemini 3.7/3.6/3.5 Flash`、`Gemini 3.1 Pro` 以及订阅内第三方模型（`Claude Opus 5.5`、`Claude Sonnet 5.5`、`GPT-OSS 120B`）[^1][^12][^27]。
- **合同主体与条款基准**：《Google Antigravity 附加服务条款》签约与运营实体为位于美国加利福尼亚州山景城（1600 Amphitheatre Parkway, Mountain View, CA 94043）的 Google LLC，协议明确规定完整合同由《Google 服务条款》《Google Antigravity 附加服务条款》及《Google 隐私政策》组成[^14]。

---

## 3. 关键事实（按子问题分组）

*(注：以下为各核心维度的事实纲要；完整的反汇编代码、逐帖对话还原与配置脚本详见顶部导航表对应的 6 份专题分册。)*

### 3.1 团队背景、旧版开源 CLI 关停与双层滚动配额池机制（详见 [分册一](./01-incident-overview-and-timeline.md)）
- **24 亿美元 Windsurf 团队引进与 `jetski` 底座**：2025 年 7 月，Google 斥资 24 亿美元达成 Windsurf（Codeium）技术许可与人才引进协议，联合创始人 Varun Mohan（`@_mohansolo`）与产品工程负责人 Kevin Hou（`@kevinhou22` / GitHub `@khou22`）加入 Google DeepMind 主导 Antigravity[^55][^56]。二进制符号表证实 `agy` 与 Cortex 引擎源码位于 `google3/third_party/jetski/` 目录下[^2]。（数据日期：2026-11-04｜置信度：高）
- **强制关停开源 `@google/gemini-cli`**：2026 年 6 月 18 日，Google 正式终止个人消费者账号对旧版开源 `@google/gemini-cli` 的访问，强制迁移至闭源 Go 编译的 `agy` CLI（`v1.5.0` 体积达 `238.6 MB`）[^10][^45]，同时移除了旧版原生支持的 OpenTelemetry（OTLP）Token 用量导出功能（GitHub Issue `#366`）[^7]。（数据日期：2026-09-30｜置信度：高）
- **5 小时 + 7 天周上限共享配额池**：所有个人客户端表面与 `agy -p` 无头模式共享同一“5 小时滚动窗口（5-Hourly Pool）+ 周上限池（Weekly Pool）”[^31][^32]，其中 **`1 Weekly Pool = 20 × 5-Hourly Pool`**，Ultra 5x 计划的单个 5 小时池按 `Gemini 3.8 Flash` 挂牌价折算隐含名义容量约为 **`$150.0 USD`**[^42]。用户需同时在 `~/.gemini/config/config.json` 设置 `"useAiCredits": false` 并在 `~/.gemini/antigravity-cli/settings.json` 设置 `"useG1Credits": false` 方可阻止配额耗尽后自动扣减额外点数[^38][^39]。（数据日期：2026-10-29｜置信度：高）

### 3.2 1M 上下文窗口差异与二进制底层的“三道硬编码门禁”（详见 [分册二](./02-256k-hardcoded-gates-and-binary-teardown.md)）
- **生产环境止步于 ~256K 的统计数据**：在 GitHub Issue `#743` 与 `#878` 中，开发者 `@thealgebraist` 指出长会话约 70% Token 被冗余历史输出占据[^6]，Eddy Murias（`@murias002`）对 310 个会话、65,613 次调用的数据库统计显示，单次调用最高输入为 `255,999` Tokens，超过 300K 的调用为 **0 次**，276 次压缩事件触发前的中位上下文为 **`255,070` Tokens**[^3]；本次对本机补丁前（`< 2026-11-04 00:00:00 UTC+8`）237 个 Antigravity CLI SQLite 会话数据库（含 209 个 1M 上下文 Gemini 会话、15,575 次调用）的独立核验同样显示，**100.0% 的调用（`15,575 / 15,575`）记录 `max_context_tokens = 256,000`**，单次无重试调用超过 300K 为 **0 次**，并记录了 79 次 `CORTEX_STEP_TYPE_CHECKPOINT`（`step_type = 23`）压缩步骤与 52 次 `truncation_reason = 2`（`MAX_TOKEN_LIMIT`）截断事件（见[取证报告 §2.5](../../evidence/zh-cn/evidence-report.md)）；在 Issue `#1245` 与 `#1291` 中，`@Burntgogi`、`@NuCl34R` 与 `@Emily2040`（1,279 次 `Gemini 4 Argon High` 调用、22 次检查点）进一步证实所有压缩均发生在 `235K–279K` 之间[^1][^4]；外部插件如 Nous Research 的 Hermes Agent 被迫将自身安全压缩阈值设为 `239,616` Tokens（`256K - 16K`）以避开 `agy` 的内部压缩[^48]。（数据日期：2026-11-05｜置信度：高）
- **SQLite `max_context_tokens = 256000` 与环境变量 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 的两阶段演变**：`@Circumsized` 证实客户端在每次请求的 SQLite `ContextWindowMetadata` 记录中直接写入 `max_context_tokens = 256000`（恰为标称 `1,048,576` 的 25%）及 `truncation_reason = 2`（静默请求前截断）[^1]；`@LiraNuna` 反汇编证实 `v1.3.5`–`v1.4.x` 中 `store.AutoCompactOverride()` 的 `os.Getenv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` 在发布编译时被内联替换为寄存器清零指令（空字符串 `""`：`xor %eax, %eax; xor %ebx, %ebx`）[^1][^2]。尽管 Google 在 `v1.5.0`–`v1.5.1`（`VA 0x9c49d00`）中未写进 `CHANGELOG.md` 便悄悄恢复了 `os.LookupEnv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` 的读取逻辑，但底层 `GetCheckpointConfig` 中的 `0x3e800`（`256,000`）钳制指令依然原封未动。在 `v1.5.1` 中，该变量只认绝对 Token 数（百分比写法收到的基数为 `0`），只对未在 `settings.json` 保存档位的白名单模型读取，也尚无证据表明它能越过钳制（见分册四 §2.4）。（数据日期：2026-11-05｜置信度：高）
- **三道二进制硬编码门禁（已在 `v1.3.5` 至 `v1.5.0` 及 `v1.5.1` 二进制中交叉核验）**：
  1. **第一道门禁（`SelectorModel.offersContextWindow`，`v1.5.0` 偏移 `0x9e92140`）**：检查硬编码模型白名单切片（`0xc592be0`），仅对 `"Gemini 4 Argon"` 返回 `true`，对同样具备 1M 窗口的 `Gemini 3.8 Flash` 等模型返回 `false`，直接隐藏 `/model` 菜单中的 `Tab` 键上下文档位切换栏（`Default | 256K | 512K | 900K`）[^1][^2]。
  2. **第二道门禁（`store.(*Store).ContextWindow()`，`v1.3.5` 偏移 `0x917d892` / `v1.5.0` 偏移 `0x9c49fe0`）**：包含第二次完全相同的 `"Gemini 4 Argon"` 白名单校验；对非 Argon 模型直接跳转至退出分支返回 `("", 0)`，根本不读取 `settings.json` 中已保存的 `"contextWindow": "512K"`，导致 `Manager.syncContextWindow()` 调用 `backend.SetContextWindow("")` 丢弃用户配置[^1][^2]。
  3. **第三道门禁（`cortex/utils/utils.GetCheckpointConfig`，`v1.3.5` 偏移 `0x7861894` / `v1.5.0` 偏移 `0x7fe57fe` / `v1.5.1` 偏移 `0x7fb0fde`）**：在计算出真实可用阈值（`context - 40,000` 安全余量）存入 `%edx` 后，紧接着执行 14 字节硬编码钳制指令 `cmp $0x3e800, %edx; mov $0x3e800, %esi; cmovb %edx, %esi; mov %esi, 0x50(%rsp)`（十六进制 `0x3e800` 即十进制 `256,000`），将所有模型（含旗舰 `Gemini 4 Argon`）的压缩触发阈值 `TokenThreshold` 强制限制在 `min(计算值, 256000)`[^1][^2]。（数据日期：2026-11-05｜置信度：高）
- **官方人员书面承认约 250K 阈值固定且不可配置（2026-11-04）**：在论坛 `#198191` 帖中，Google 官方人员 `@chunduriv` 回复 `@Sharper`：*"As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. That threshold is a fixed default and isn't user-configurable yet."*（桌面端中的 Gemini 模型，例如 Gemini 4 Argon，目前会在约 250K Tokens 时自动压缩；该阈值是固定默认值，暂不支持用户配置。）回复将该问题归类为功能请求，建议以手动 `/compact`、工作区参考文件和排除构建产物作为临时缓解办法，并表示已把“用户可选压缩阈值”的需求转交工程团队，未给出时间表。这是本档案收录的第一条写明该阈值、并确认用户无法修改的官方书面答复[^57]。（数据日期：2026-11-05｜置信度：高）

### 3.3 提前压缩的工程影响、70 秒压缩耗时与 Argon 3 倍配额消耗（详见 [分册二](./02-256k-hardcoded-gates-and-binary-teardown.md)）
- **代码符号丢失与幻觉错误**：跨越 ~250K 后，系统将 25 万 Tokens 历史有损压缩为仅 `15K–17K` Tokens 的散文摘要（`CHECKPOINT: "# Resuming from a compaction..."`），丢失此前 `view_file` 读取的精确变量名、Schema 与初始化约束，导致模型在未重读文件的情况下捏造 6 个不存在的标识符（论坛 `#189476`）[^43]、将 `1000..2000` 数值比例尺错误缩小 200 倍并在不同模块混用连字符与下划线键名（Issue `#1245`）[^1]，或在多步工程中破坏前后端 DTO 契约并引入认证绕过逻辑（论坛 `#176775`）[^46]。（数据日期：2026-10-15｜置信度：高）
- **70 秒压缩停顿、缓存失效与 31% 未缓存输入开销**：`@Emily2040` 在 Issue `#1291` 中实测 1,279 次 `Gemini 4 Argon High` 调用发现：① 每次自动压缩中位耗时达 **70 秒**（35–602 秒），摘要调用消耗的算力不在轨迹元数据中报告但扣减配额，且压缩后前缀重写导致 240K+ 隐式缓存归零（`cache_read_tokens → 0`），紧随其后需发起 45K–70K 未缓存调用重建缓存[^1][^4]；② 仅 17 次中途前缀缓存失效（其中 15 次发生在 `system_message` 步骤后两步内）就产生了 `3,292,077` 未缓存输入 Tokens，占整个运行周期全部未缓存输入的 **31.0%**[^4]；③ 每次无头调用 `agy -p` 存在固定的 **11.3–12.2 秒**客户端启停开销，且 943 KB 的会话摘要存储文件在 90 秒内被全量重写 27 次（每 1.5–3.3 秒重写一次）[^4]。（数据日期：2026-10-19｜置信度：高）
- **`Gemini 4 Argon` 按公开 API 价格 3.0 倍（Web 纯输入 7.64–9.25 倍）消耗配额**：`@Yiji_Kong` 在论坛 `#196351` 的控制变量 A/B 测试证实，在 Antigravity 同一 5 小时配额桶内，按公开 API 挂牌价归一化后，**`Gemini 4 Argon` 每美元消耗的配额等于 `Gemini 3.8 Flash` 的 `3.00 倍`**（相当于按 `$6.00 / $30.00` 每百万 Token 扣费，而非公开标价的 `$2.00 / $10.00`）[^42]；在 Gemini Web 端 8 位小数周配额池（`ΔWeekly`）纯输入零思考测试中，Argon 单位美元配额消耗达 Flash 的 **7.64 倍至 9.25 倍**，在 Argon (Low) 上仅发送一句 28 字符的 `"Reply with only the word: OK"` 所消耗的周配额（`+0.00015849`），比在 3.8 Flash (High) 上写完一篇 4,013 词的 Paxos/Raft 技术长文（`+0.00014157`）高出 **12%**[^42]；此外 `gemini-3.8-flash-tiered`（`M322`）子代理探测调用出现了标准 Flash 预期值 **22 倍**（`0.01048544`）的扣费[^42]。Google 官方人员 `@Maniraj_Chervirala` 回应称公开 API 价格仅为按量计费促销价，订阅配额内部乘数不予公开[^42]。（数据日期：2026-10-29｜置信度：高）
- **受控高倍率上下文档位与 `v2.29.0` 动向**：客户端内部已将 `512K` 设为 `1.3x` 配额倍率、`900K` 设为带 `Expensive` 标记的 `1.8x` 配额倍率[^1][^18][^19]。Google 维护者 `@chandrashakherkamasani` 于 10 月 15 日接管 Issue `#1245` 后仅标 `feedback` 标签（未标 `bug`）并保持 20+ 天无公开回复[^1]，同时在 CLI `1.4.3–1.4.10` 中修补压缩后丢失图片/PDF 及子代理丢失父代理 ID 等问题[^11]，并在 11 月 3 日桌面端 `v2.29.0`（`language_server.exe` 增长 `+4.47 MB`）中新增 `cortex/tokens/compaction` 与 `cortex/shared/snapshot` 包及受控开关 `enable-context-usage-indicator`[^17]。（数据日期：2026-11-05｜置信度：高）

### 3.4 `agy -p` 无头模式政策变更与账号申诉情况（详见 [分册三](./03-agy-p-headless-policy-reversal-and-bans.md)）
- **变更前（2026-09-16 至 10-08 16:23 UTC）：10 个独立论坛帖连续书面确认**：Google 官方支持工程师 `@Rebbanpalli_Naveen`（`#175905`、`#186954`、`#186952`）、`@Ambati_Rajendra`（`#183429`、`#184829`、`#185494`、`#185460`、`#185949`、`#187536`）与 `@chunduriv`（`#185992`、`#186246`）连续书面确认：只要执行未修改的官方 `agy` / `agy.exe` 二进制且不提取 OAuth 令牌（且不封装为对外 MCP Server[^25]），由开源工具、本地编排器、OpenAI Codex 或 Anthropic Claude Code 通过本地子进程调用 `agy -p` 符合《附加服务条款》第 6 条[^23][^24][^26][^27][^28][^29][^30][^31][^32][^33]。其中 `@chunduriv` 在 **10 月 7 日 15:03 UTC（`#186246`）** 明确写道：*“从政策角度来看，未修改的 `agy` 子进程究竟是由你自己的脚本启动，还是由另一个本地编程助手（如 Claude Code）启动，没有任何区别（there is no difference）。”*[^32]（数据日期：2026-10-08｜置信度：高）
- **变更后（2026-10-08 17:34 UTC 起）：26.5 小时内口径变更、确认未改 ToS 仅改 FAQ、限制本地文件交接**：
  1. 仅隔 **26 小时 31 分钟**（距 `#187536` 仅隔 71 分钟），`@chunduriv` 于 **10 月 8 日 17:34 UTC（`#187241`）** 改变口径称不支持用 Codex、Claude Code、OpenClaw 或 OpenCode 编排或包装 `agy`，否则可能触发自动 ToS 执法与封号[^34]。
  2. 10 月 16 日（`#189905`），`@Rebbanpalli_Naveen` 表示更新后的官网 FAQ *“取代早期论坛评论（supersedes earlier forum comments where they differ）”*[^35]。
  3. 10 月 26–27 日（`#191689`），面对付费用户 `@b_w` 与 `@Tora39` 询问 10 月 5 日之后的合同修改记录，`@Rebbanpalli_Naveen` 在第 8 楼确认：**《Google Antigravity 服务条款》本身在 9 月之后并未修改，而是在 10 月中旬更新了非合同性质的《官方 FAQ》，并称 `#186246` 与 `#186952` 的早期官方回复“给出了过于宽泛的解释”**[^36]；日本开发者佐藤賢治（`horizon_it00`）等亦公开记录因该 FAQ 更新停用跨工具编排架构[^47]。
  4. 10 月 28–29 日（`#190815`、`#196472`、`#190756`），官方进一步表示“由 Claude Code 将任务写入本地文件夹再由独立 Python 脚本调用 `agy -p`”以及“本地独立 Python 任务协调服务”均不被个人订阅支持，需购买 Google AI Studio API Key 或企业版 Gemini Enterprise[^38][^39][^40]。（数据日期：2026-10-29｜置信度：高）
- **二振出局封号与 100+ 天申诉停滞**：`cloudcode-pa.googleapis.com` 返回的 `HTTP 403 TOS_VIOLATION` 会同时限制同一账号下的 Antigravity、Gemini CLI 与 Gemini Code Assist，且实行“首次填表重认证解封、第二次标记直接永久封禁”的二振出局制（`Discussion #20632`）[^9]。GitHub Issue `#927` 与 `#29576`（论坛 `#186089`）记录显示，多名用户提交申诉表单后超过 **100 天**仍未获同步解封[^8][^10][^45]。（数据日期：2026-09-30｜置信度：高）

### 3.5 社区与官方对 256K 限制的应对（详见 [分册四](./04-community-and-official-responses.md)）
- **印证限制固定且位于客户端的各方应对**：社区补丁 `patch_agy.py` 通过修改三处客户端函数去掉 256K 钳制，与分册二的反汇编结论一致[^1][^2]；Hermes Agent 把自身阈值设为 `239,616` Tokens 以避开 `agy` 的压缩[^48]；Google 官方人员建议手动 `/compact`、压缩后重读文件、减少上下文噪音，并确认约 250K 阈值不能由用户配置[^43][^57]；Google 的 ADK 案例借助 `.agents/skills` 与 `/goal` 模式，才完成一次在多次压缩后仍能继续的 11 小时运行[^16]；第三方监控插件与 MCP 服务器之所以出现，是因为客户端既不显示也不允许控制上下文用量[^49][^50][^51][^52][^53][^54]。（置信度：高）

---

## 4. 关键数据表

| 维度分类 | 指标 / 项目名称 | 实测数值 / 机器码状态 | 统计口径与时间窗口 | 溯源序号 | 置信度 |
| :--- | :--- | :--- | :--- | :---: | :---: |
| **上下文容量与硬编码门禁** | `Gemini 4 Argon` / `3.8 Flash` 标称物理窗口 | `1,048,576` Tokens (`1.0M`) | 官方文档、模型卡片及 CLI `/context` 面板显示值（截至 2026-11） | [^1][^3][^12][^44] | 高 |
| **上下文容量与硬编码门禁** | SQLite 会话库持久化上限字段 | `max_context_tokens = 256000`<br>`truncation_reason = 2` | `~/.gemini/antigravity-cli/conversations/<id>.db` 中 `ContextWindowMetadata`（本机 209 个补丁前 1M Gemini 会话 / 15,575 次调用 100.0% 命中，及 `#1245` `@Circumsized` 实证） | [^1] | 高 |
| **上下文容量与硬编码门禁** | Cortex 引擎底层压缩触发硬截断指令 | `cmp $0x3e800, %edx`<br>`mov $0x3e800, %esi`<br>`cmovb %edx, %esi` (`256,000`) | `google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig`（`v1.3.5` 偏移 `0x7861894`，`v1.5.0` 偏移 `0x7fe57fe`，`v1.5.1` 偏移 `0x7fb0fde`） | [^1][^2] | 高 |
| **上下文容量与硬编码门禁** | `AGY_CLI_AUTO_COMPACT_OVERRIDE` 状态 | `1.3.5–1.4.x`：编译期抹除为 `""`（`xor %eax, %eax; xor %ebx, %ebx`）；`1.5.0–1.5.1`：静默恢复 `os.LookupEnv`；`1.5.1` 中仅认绝对 Token 数、仅对白名单模型生效 | `1.5.0`/`1.5.1` 中 `store.AutoCompactOverride()`（`VA 0x9c49d00`）已恢复，但 `GetCheckpointConfig` 的 `0x3e800` 钳制仍保留 | [^1][^2][^11] | 高 |
| **上下文容量与硬编码门禁** | 310 个生产会话（65,613 次调用）实测上限 | 最大输入 `255,999` Tokens；`>300K` 调用为 **`0 次`**；压缩前中位数 **`255,070` Tokens** | `@murias002` 对 2026-08-30 至 10-04 全量会话数据库统计（276 次压缩事件） | [^3] | 高 |
| **上下文容量与硬编码门禁** | 触发自动压缩后剩余摘要大小 | `15,363` – `17,336` Tokens（散文摘要） | `@Burntgogi` 94 轮 Argon 实测（从 `276,110` 与 `249,867` 暴跌，抹除 >93% 历史） | [^1] | 高 |
| **上下文容量与硬编码门禁** | 外部插件躲避强压的安全阈值 | `239,616` Tokens (`239K`) | Nous Research Hermes Agent 插件（`256K - 16K output tokens`） | [^48] | 高 |
| **缓存失效与隐性开销** | 单次自动压缩停顿耗时与汇报状态 | 中位耗时 **`70 秒`**（`35–602 秒`）；轨迹汇报用量为 **`0`（隐藏扣费）** | `@Emily2040` 对 15 份无头日志（1,279 次 Argon High 调用、22 次检查点）实测 | [^1][^4] | 高 |
| **缓存失效与隐性开销** | 压缩后 Prompt Cache 留存率与重建成本 | 留存率 **`0%`**（瞬间清零）；紧随一次 **`44,990–70,041` Tokens** 未缓存调用 | 前缀被重写为散文摘要导致 ~240K 隐式缓存作废 | [^1][^4] | 高 |
| **缓存失效与隐性开销** | 中途前缀缓存失效（Mid-run Prefix Miss）占比 | 仅 **17 次调用（1.3%）** 消耗 **`3,292,077` Tokens（占全部未缓存输入的 `31.0%`）** | 17 次中有 15 次发生在客户端注入 `system_message` 步骤后两步之内 | [^4][^5] | 高 |
| **缓存失效与隐性开销** | `agy -p` 每次无头调用固定客户端启停开销 | **`11.3 – 12.2 秒`**（且 943 KB 摘要库在 90 秒内被全量重写 27 次） | `@Emily2040` 5 轮基准测试（服务端仅耗时 4.5–8.2 秒，201–238 MB 二进制启停占 12 秒） | [^4] | 高 |
| **配额倍率与计费权重** | 隐藏上下文档位与每轮配额消耗乘数 | `256K`: **`1.0x`**<br>`512K`: **`1.3x`** (`about 1.3x quota per turn`)<br>`900K`: **`1.8x`** (`Expensive: true`) | CLI `/model` 解锁界面实测与 Desktop `v2.27.0–v2.28.1` 二进制拆包 | [^1][^18][^19] | 高 |
| **配额倍率与计费权重** | `Gemini 4 Argon` vs `3.8 Flash` 同桶配额消耗比 | 按公开 API 挂牌价归一化后，Argon 消耗配额为 Flash 的 **`3.00 倍 / 美元`** | `@Yiji_Kong` Antigravity `v2.27.0` Ultra 5x 控制变量 A/B 测试（隐含 Argon 按 `$6/$30` 计费，5 小时桶约 `$150 USD`） | [^42] | 高 |
| **配额倍率与计费权重** | Web 端 8 位小数周配额池纯输入消耗比 | Argon 纯输入消耗为 Flash 的 **`24.67x` 原始 / `9.25x` 每美元**（扣除底座后 **`7.64x` 每美元**） | `@Yiji_Kong` Gemini Web 8 位小数 `percentageUsed` 实测（Argon Low 发一句 `"OK"` 比 Flash High 写 4,013 词长文多耗 **12%** 配额） | [^42] | 高 |
| **配额倍率与计费权重** | `gemini-3.8-flash-tiered` (`M322`) 子代理异常扣费 | 单次跌落 `0.01048544`（约为标准 Flash 预期值 `0.00047` 的 **`22 倍`**） | `12,773` 输入 / `16,332` 输出 Tokens，0 缓存子代理探测 | [^42] | 高 |
| **规则上限与版本演进** | Antigravity 常驻规则（Rules）硬上限 | 单文件 **`24 KB` (`24,000` 字节)** 截断；总计 **`20,000` Tokens** (`defaultRulesBudget`) | 官方规则文档与内置指南（超出 20K Tokens 自动降级为文件路径指针） | [^15] | 高 |
| **规则上限与版本演进** | Desktop `v2.29.0` 后端体积增长 | `language_server.exe` **`+4.47 MB`**，内嵌 Web UI **`+36%`** | 2026-11-03 发布，新增 `cortex/tokens/compaction`、`cortex/shared/snapshot` 及 `enable-context-usage-indicator` | [^12][^17] | 高 |
| **政策反转与封号机制** | 官方确认 `agy -p` 子进程合规至口径变更的时间差 | 连续 **10 帖确认合规**（9月16日–10月8日 16:23 UTC）→ **71 分钟 / 26.5 小时后变更口径**（10月8日 17:34 UTC） | 正式 ToS 自 9 月起未修改；10 月 8 日先以论坛回复变更口径，10 月中旬新增的 FAQ 条目随后被宣布取代此前官方答复 | [^31][^32][^33][^34][^36] | 高 |
| **政策反转与封号机制** | ToS 违规封禁容错次数与申诉停滞时长 | **1 次重认证机会，第 2 次直接永久封禁**；申诉后超 **100+ 天** 未同步解封 | `Discussion #20632` 官方规则及 `Issue #927`、`#29576`、`#186089` 实测 | [^8][^9][^10][^45] | 高 |

---

## 5. 外部公开界面外链索引

与 `256K` 上下文限制及 `/context` 面板显示差异相关的两份公开界面截图可通过以下原始地址查看：

1. **解锁 `SelectorModel.offersContextWindow` 后的 `agy` CLI `/model` 菜单截图**（`@LiraNuna` 发布于 [GitHub Issue #1245](https://github.com/google-antigravity/antigravity-cli/issues/1245)，[查看 GitHub 原始截图](https://github.com/user-attachments/assets/dda2af37-98a9-4d89-b78e-8cae8c6361cc)）[^1]：展示将 `offersContextWindow` 补丁为返回 `true` 后，`Gemini 3.8 Flash` 下方暴露出的隐藏 `Context: Default | 256K | [512K] | 900K` 切换栏及红色 `about 1.3x quota per turn` 配额警告。
2. **`agy` CLI 内置 `/context` 面板与桌面端侧边栏同会话对比截图**（`@dimitr-io` 发布于 [Google AI Developers Forum #184694](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694)，[查看论坛原始截图](https://d3qe71uytubmmx.cloudfront.net/original/3X/e/d/ed8ffb40b5376fcc45cc7cf33c70aa687083b01e.jpeg)）[^44][^51]：展示会话在 `222.4k–225.0k`（逼近 `256,000` 强制压缩红线）且已发生历史检查点摘要（`1 older checkpoint(s) summarized, not in context`）时，界面仍标称总窗口为 `1.0M tokens` 并显示 `Free space: 823.6k (78.5%)`。

---

## 6. 多方观点

- **立场 A：付费订阅开发者与逆向工程社区（`@Burntgogi`、`@LiraNuna`、`@Circumsized`、`@Emily2040`、`@Yiji_Kong`、`@murias002`、`@b_w`、`@Tora39`、`@oli_hlmpn`）**
  - **关于 256K 硬编码与 3 倍配额消耗**：认为 Google 将 `1,048,576`（1M）上下文作为卖点吸引用户订阅 $20/月（Pro）至 $250/月（Ultra）计划，却在客户端二进制 `GetCheckpointConfig` 中用 `0x3e800` 将所有模型限制在 25% 容量处，并在每次压缩时停顿 70 秒、使 240K+ 缓存失效且以公开价 3 倍以上的权重扣减 Argon 配额，导致宣传参数与实际可用容量不符[^1][^3][^4][^42]。即便出于成本控制需要限制默认窗口，也应将其作为公开文档化的可调配置并在 `/context` 中展示实际触发阈值[^1]。
  - **关于 `agy -p` 政策反转**：指出开发者在集成前专门在官方论坛获得 Google 工程师连续 10 次明确书面许可，而后 Google 不修改具有法律效力的《服务条款（ToS）》，仅靠静默修改 FAQ 就推翻承诺并威胁永久封禁 Google 账号，不仅违背商业契约精神，而且在操作系统层面根本无法区分“合法的 3 行個人脚本”与“外部 AI 辅助触发的脚本”，使所有合规脚本用户沦为随时可能被误封的牺牲品[^31][^32][^36][^39]。
- **立场 B：Google 一线技术支持与社区工程师（`@Ambati_Rajendra`、`@Rebbanpalli_Naveen`、`@chunduriv`、`@Maniraj_Chervirala`）**
  - **反转前（2026-09-16 至 10-08 16:23 UTC）**：基于 Unix `stdio` 工具设计常理与 ToS 第 6 条打击盗取 OAuth Token 的本意，一致认为只要执行原版未修改的 `agy` 二进制且认证留在 `agy` 内部，无论由个人脚本、Codex 还是 Claude Code 启动子进程均完全合规[^23][^24][^27][^28][^31][^32][^33]。
  - **反转后（2026-10-08 17:34 UTC 起）**：迫于内部政策指令，不得不統一添加免责声明（*“While we cannot provide formal legal interpretations of the Terms of Service...”*），承认正式 ToS 未变但宣称 10 月中旬新增的 FAQ 取代了自己在 `#186246` 与 `#186952` 中“过于宽泛的早期解释”，引导用户额外购买 Google AI Studio API Key（设置 `"modelProvider": "gemini"`）或联系销售购买 Gemini Enterprise，或改用 `agy acp` 协议[^34][^35][^36][^38][^41]。对于 256K 压缩失忆问题，建议用户在 `GEMINI.md` 中添加强制重读规则缓解症状[^43]。
- **立场 C：Google 产品管理、商业化与法务风控层（规则制定方）**
  - **核心逻辑**：在固定月费的个人消费级订阅（Pro / Ultra）与按量计费的 Google AI Studio API / 企业级 Vertex AI（Gemini Enterprise）之间筑起严格的商业隔离墙[^36][^40][^42]。由于长上下文推理在缓存失效时单步成本激增 4 倍以上，且多智能体自动化循环会高频消耗算力池，管理层选择在底层用 `0x3e800` 锁死默认 256K 上限、在后台筹备带 `1.3x` / `1.8x Expensive` 倍率的 `512K` / `900K` 档位，并通过修改 FAQ 一刀切禁止任何第三方编程智能体调用个人订阅会话下的 `agy -p`[^1][^13][^17][^18][^42]。

---

## 7. 争议与矛盾信息

1. **矛盾一：宣传界面与 `/context` 面板标称 `1,048,576`（1.0M）可用窗口 vs. SQLite `max_context_tokens = 256000` 与汇编 `0x3e800` 硬截断**
   - **表现与原因**：CLI `/context` 面板在 225K 时仍显示 `21.5%` 占用与 `823.6k (78.5%) Free space`[^44][^51]，且 `SelectorModel.offersContextWindow` 已为 `Gemini 4 Argon` 开放了 `512K`/`900K` 选项；但底层 Cortex 引擎 `GetCheckpointConfig` 却因遗留或刻意的 `cmp $0x3e800, %edx; cmovb %edx, %esi` 指令将所有模型一律钳制在 `256,000`，形成前后端代码左右手互搏[^1][^2]。
2. **矛盾二：正式《附加服务条款（ToS）》自 9 月起未改一字 vs. 仅凭修改非合同性质的《FAQ 页面》推翻 10 次官方书面许可并威胁封号**
   - **表现与原因**：合同第 1–2 条明确规定协议仅由通用条款、附加条款与隐私政策三份文件组成，不包含 FAQ[^14]；官方人员 `@Rebbanpalli_Naveen` 在 `#191689` 第 8 楼亦亲口承认 ToS 在 9 月后从未修改[^36]。Google 用不具合同修改效力的 FAQ 网页单方面废除官方代表此前一个月的书面解释，造成正式合同文本与实际风控执法口径之间的直接脱节[^14][^36]。
3. **矛盾三：允许第三方客户端通过 `stdio` 调用 `agy acp` vs. 禁止第三方客户端通过 `stdio` 调用 `agy -p`**
   - **表现与原因**：`agy v1.5.0` 内置的 `agy acp` 与 `agy -p --output-format stream-json` 在物理本质上完全相同（均是在本地启动原版 `agy` 子进程、通过 `stdio` 收发 JSON、使用同一个个人订阅 OAuth 会话与配额池）[^11][^39][^41]；但官方却规定前者在第三方工具中合规、后者在第三方工具中可触发永久封禁，导致技术边界自相矛盾[^39][^41]。

---

## 8. 时间线（核心节点速览，完整逐日时间线见 [分册一](./01-incident-overview-and-timeline.md)）

- **2025-07**：Google 以 **24 亿美元**达成 Windsurf（Codeium）技术许可与核心团队引进协议，Varun Mohan 与 Kevin Hou 加入 Google DeepMind 构建 Antigravity（内部代码路径 `google3/third_party/jetski`）[^2][^55][^56]。
- **2026-02-27**：Google 维护者 `@jackwotherspoon` 发布 **Discussion #20632**，确立 `cloudcode-pa.googleapis.com` 反滥用封号的“首次填表重认证解封、第二次直接永久封禁”二振出局规则[^9]。
- **2026-05-30**：Reddit `r/google_antigravity` 爆发针对激进自动压缩毁掉 Gemini 1M 上下文卖点的集中控诉（`#1trr4dy`）[^21]。
- **2026-06-18**：Google 正式关停个人消费者对旧版开源 `@google/gemini-cli` 的访问，强制迁移至闭源 Go 二进制 `agy`[^10][^45]。
- **2026-08-26**：开发者 `@schalatap` 提交 **GitHub Issue #878**，要求允许关闭刚性自动压缩并解锁完整 1M 窗口[^3]。
- **2026-09-16 至 2026-10-08 16:23 UTC**：Google 官方人员 `@Rebbanpalli_Naveen`、`@Ambati_Rajendra`、`@chunduriv` 在论坛连续 **10 个主题帖**（`#175905`、`#183429`、`#185494`、`#185460`、`#185992`、`#185949`、`#186954`、`#186952`、`#186246`、`#187536`）中书面确认：本地子进程调用未修改的 `agy -p`（含由 Codex 或 Claude Code 启动）完全合规[^23][^24][^26][^27][^28][^29][^30][^31][^32][^33]。
- **2026-10-08 17:34 UTC**：距 `#187536` 仅隔 71 分钟、距 `#186246` 仅隔 26.5 小时，`@chunduriv` 在 **#187241** 突然反转改口，宣布用 Codex / Claude Code 调用 `agy` 属违规并可触发自动封号[^34]；此后被引为依据的官网 FAQ 条目于 10 月中旬才加入[^13][^36]。
- **2026-10-14 至 10-23**：`@Burntgogi` 提交 **GitHub Issue #1245**；`@Circumsized` 曝出 SQLite `max_context_tokens = 256000`；`@Emily2040` 提交 **Issue #1291** 曝出 70 秒黑盒压缩与 31% 未缓存输入浪费；`@LiraNuna` 反汇编扒出三道硬编码门禁与 `0x3e800` 汇编指令并发布 **`patch_agy.py`** 补丁脚本；Google 维护者 `@chandrashakherkamasani` 领单但拒打 `bug` 标签且零回复[^1][^2][^4]。
- **2026-10-16 至 10-29**：`@Rebbanpalli_Naveen` 在 **#189905** 宣布新 FAQ 废除早期回复[^35]，在 **#191689** 被用户质问后承认正式 ToS 自 9 月起从未修改[^36]，并与 `@Ambati_Rajendra` 在 **#190815**、**#196472** 中封杀通过本地文件或独立 Python 服务与外部 Agent 协同[^38][^40]。
- **2026-10-28 至 10-29**：`@Yiji_Kong` 在论坛 **#196351** 发布严格 A/B 实测，证实 `Gemini 4 Argon` 按公开价 3.0 倍（Web 纯输入 7.64–9.25 倍）消耗订阅配额[^42]；`u/Lberondroth` 发布 `v2.27.0 → v2.28.1` 拆包（`#1xij7jx`）曝出 `512K`/`900K Expensive` 字段[^18]。
- **2026-11-02 至 11-04**：Reddit 双版块置顶追问何时解除 256K 限制[^19][^20]；Google 发布桌面端 **`v2.29.0`**（`language_server.exe` +4.47 MB，新增 `cortex/tokens/compaction` 与受控进度条开关）[^12][^17]及 CLI **`v1.5.0`**（新增 `agy acp`，但经本机反汇编验证 `0x3e800` 硬截断依然保留在文件偏移 `0x7fe5802` 处）[^11]。
- **2026-11-04 23:39 UTC**：Google 官方人员 `@chunduriv` 在论坛 **#198191** 帖回复 `@Sharper`，书面确认 Antigravity 桌面端中的 Gemini 模型会在约 250K Tokens 时自动压缩，且该阈值“is a fixed default and isn't user-configurable yet”（是固定默认值，暂不支持用户配置）；“用户可选压缩阈值”的需求已转交工程团队，未给出时间表[^57]。

---

## 9. 可引用原文摘录（完整 20+ 条中英对照见 [分册五](./05-verbatim-quotes-and-evidence-vault.md)）

> *“2. Cortex has a hardcoded 256K (`0x3e800`) ceiling on compaction triggers: Even with the backend receiving the context window limit, Cortex's checkpoint calculator (`cortex/utils/utils.GetCheckpointConfig`) explicitly clamps the compaction trigger (`TokenThreshold`) for conversational cascades: `7861898: cmp $0x3e800, %edx # 0x3e800 == 256,000 tokens ... 78618a3: cmovb %edx, %esi # Clamps to min(calculated, 256000)`.”*
> —— 系统工程师 `@LiraNuna`，GitHub Issue `#1245` Comment #6，2026-10-23[^1]

> *“Data: across 15 headless stream-json logs on 1.4.0–1.4.2 with Gemini 4 Argon High (1,279 model calls), 22 `checkpoint` steps fired, every one with the preceding call's context between 235k and 268k tokens (`input + cache_read`), never higher... Each checkpoint took a median 70 s (35–224 s, one 602 s), reported no usage for the summary call, and was followed by a 45–70k uncached call before the cache rebuilt.”*
> —— 开发者 `@Emily2040`，GitHub Issue `#1245` Comment #4 & Issue `#1291`，2026-10-19[^1][^4]

> *“Normalized by each model’s published API price, Argon consumes exactly 3.0x more quota per dollar than Flash... In Step 1C, sending just `"Reply with only the word: OK"` (28 chars) in a fresh chat on 4 Argon (Low) burned `+0.00015849` of weekly quota (~0.32% of the 5-hour pool), which is 12% more than 3.8 Flash (High) writing a full 4,013-word technical article on Paxos and Raft (`+0.00014157`).”*
> —— 开发者 `@Yiji_Kong`，Google AI Developers Forum `#196351`，2026-10-28 / 10-29[^42]

> *“2. Does the answer change if you launch the script yourself vs. Claude Code launching it? No. From a policy standpoint, there is no difference whether the unmodified `agy` subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code)...”*
> —— Google 官方人员 `@chunduriv`（反转前 26.5 小时），Google AI Developers Forum `#186246` Post #2，2026-10-07 15:03 UTC[^32]

> *“To clarify the timeline for `@b_w` and answer `@Tora39`’s question (a): the Google Antigravity Terms of Service themselves were not modified after September; rather, the Official FAQ was updated in mid-October 2026 to add the section ‘Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?’ ... Earlier forum replies in #186246 and #186952 gave an overly broad interpretation before that FAQ entry was published.”*
> —— Google 官方人员 `@Rebbanpalli_Naveen`（被用户当面质问后的承认），Google AI Developers Forum `#191689` Post #8，2026-10-27 12:31 UTC[^36]

> *“As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. That threshold is a fixed default and isn't user-configurable yet. We understand that for workflows that rely on a large, stable, cached starting context, compacting this early costs both cache efficiency and detail.”*
> —— Google 官方人员 `@chunduriv`，Google AI Developers Forum `#198191` Post #2，2026-11-04 23:39 UTC[^57]

---

## 10. 信息缺口与局限

1. **云端何时拨开 `v2.29.0` / `v1.5.0` 中的 `ContextWindowOptions` 与新压缩包开关尚未公布**：尽管本地 `agy 1.5.0` 与桌面端 `v2.29.0` 二进制中已编译入 `cortex/tokens/compaction` 与 `enable-context-usage-indicator`，但截至 2026 年 11 月 4 日，`GetCheckpointConfig` 中的 `0x3e800` 常数仍未被移除，Google 官方亦未公布具体的云端 Feature Flag 全量放开日期[^11][^17][^19]。11 月 4 日，官方人员在论坛 `#198191` 帖确认约 250K 阈值是固定默认值，只表示“用户可选压缩阈值”的需求已转交工程团队评估，没有给出时间表[^57]。
2. **服务端 `cloudcode-pa.googleapis.com` 对非 Argon 模型破解超过 300K 后的风控阈值属黑盒**：客户端通过 `patch_agy.py` 解锁非 Argon 模型（如 `Gemini 3.8 Flash`）至 `512K`/`900K` 后，Google 服务端是否设有針對单次请求超过 256K 的异常审计规则尚无公开服务端源码验证[^1][^2]。

---

## 11. 参考来源清单（脚注定义）

[^1]: [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) — GitHub `google-antigravity/antigravity-cli`（作者：`Burntgogi`，评论：`NuCl34R`、`Circumsized`、`Emily2040`、`LiraNuna`、`AsabaLazy`），发布日期：2026-10-14 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与代码逆向实证｜置信度：高）
[^2]: [Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) — GitHub Gist（作者：`LiraNuna`），发布日期：2026-10-23（访问日期：2026-11-04｜类型：一手逆向工程补丁源码｜置信度：高）
[^3]: [[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878)](https://github.com/google-antigravity/antigravity-cli/issues/878) — GitHub `google-antigravity/antigravity-cli`（作者：`schalatap`，评论：`bash99`、`murias002`、`Corsagee`、`Infinnerty`、`jlukic`），发布日期：2026-08-26 至 2026-10-31（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与大规模会话统计｜置信度：高）
[^4]: [[Bug]: Five measured places the CLI spends time and tokens the work does not need: serial tool calls, uncached full-context calls and 70 s compactions, 12 s headless overhead, summary store rewritten every 1.5 s, build-time CPU instruction choice (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) — GitHub `google-antigravity/antigravity-cli`（作者：`Emily2040`），发布日期：2026-10-19 至 2026-10-25（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与基准实测｜置信度：高）
[^5]: [[Feature Request]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue #1072)](https://github.com/google-antigravity/antigravity-cli/issues/1072) — GitHub `google-antigravity/antigravity-cli`（作者：`flotoz`，评论：`Emily2040`），发布日期：2026-09-20 至 2026-10-19（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^6]: [Feature Request: Add `max_context_tokens` setting to control per-turn token budget (Issue #743)](https://github.com/google-antigravity/antigravity-cli/issues/743) — GitHub `google-antigravity/antigravity-cli`（作者：`thealgebraist`），发布日期：2026-08-03（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^7]: [Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue #366)](https://github.com/google-antigravity/antigravity-cli/issues/366) — GitHub `google-antigravity/antigravity-cli`（作者：`UesugiErii` 等），发布日期：2026-06-11 至 2026-10-20（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^8]: [agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync (Issue #927)](https://github.com/google-antigravity/antigravity-cli/issues/927) — GitHub `google-antigravity/antigravity-cli`（作者：`jlwainwright`），发布日期：2026-09-02（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^9]: [Addressing Antigravity Bans & Reinstating Access (Discussion #20632)](https://github.com/google-gemini/gemini-cli/discussions/20632) — GitHub `google-gemini/gemini-cli`（作者：Google 维护者 `@jackwotherspoon`），发布日期：2026-02-27（访问日期：2026-11-04｜类型：一手官方公告｜置信度：高）
[^10]: [Appeal submitted but TOS_VIOLATION suspension persists (request for private follow-up) (Issue #29576)](https://github.com/google-gemini/gemini-cli/issues/29576) — GitHub `google-gemini/gemini-cli`（作者：`myessaylab1234`），发布日期：2026-09-30（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^11]: [Google Antigravity CLI Releases (`v1.4.2` – `v1.5.0`)](https://github.com/google-antigravity/antigravity-cli/releases) — GitHub `google-antigravity/antigravity-cli`，发布日期：2026-10-18 至 2026-11-04（访问日期：2026-11-04｜类型：一手官方发版日志｜置信度：高）
[^12]: [Google Antigravity Docs — Changelog (`v2.0.1` – `v2.29.0`)](https://antigravity.google/docs/changelog) — Google Antigravity 官方文档，发布日期：2026-11-03（访问日期：2026-11-04｜类型：一手官方文档｜置信度：高）
[^13]: [Google Antigravity Docs — Official FAQ](https://antigravity.google/docs/faq/) — Google Antigravity 官方文档，发布日期：2026-10（访问日期：2026-11-04｜类型：一手官方文档｜置信度：高）
[^14]: [Google Antigravity — Additional Terms of Service](https://antigravity.google/terms) — Google Antigravity 官方法律条款，发布日期：2026-09（访问日期：2026-11-04｜类型：一手官方合同条款｜置信度：高）
[^15]: [Google Antigravity Docs — Workspace Rules (24 KB Per-File Limit & 20,000-Token Aggregate Rules Budget)](https://antigravity.google/docs/rules/) — Google Antigravity 官方文档（及内置技能 `agy-customizations/docs/rules.md`），发布日期：2026-10（访问日期：2026-11-04｜类型：一手官方文档｜置信度：高）
[^16]: [Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/) — Google Developers Blog，发布日期：2026-10（访问日期：2026-11-04｜类型：一手官方工程博客｜置信度：高）
[^17]: [[Teardown / Binary Diff] Antigravity Updated to v2.29.0](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/) — Reddit `r/google_antigravity`（作者：`u/Lberondroth`），发布日期：2026-11-03（访问日期：2026-11-04｜类型：专业社区二进制拆包报告｜置信度：高）
[^18]: [[Deep-Dive] Antigravity v2.27.0 → v2.28.1 Binary & ASAR Diff: 'Script Jail', the `oneharness` Migration, Teamwork Leaving Preview, and What's Coming Next](https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/) — Reddit `r/google_antigravity`（作者：`u/Lberondroth`），发布日期：2026-10-28（访问日期：2026-11-04｜类型：专业社区二进制拆包报告｜置信度：高）
[^19]: [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon?](https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/) — Reddit `r/AntigravityGoogle`，发布日期：2026-11-02（访问日期：2026-11-04｜类型：专业社区讨论｜置信度：高）
[^20]: [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon? (`r/google_antigravity`)](https://www.reddit.com/r/google_antigravity/comments/1xmc89l/timeline_for_enabling_the_512k_900k/) — Reddit `r/google_antigravity`，发布日期：2026-11-02（访问日期：2026-11-04｜类型：专业社区讨论｜置信度：高）
[^21]: [Antigravity's aggressive history compaction is ruining Gemini's 1M+ context selling point](https://www.reddit.com/r/google_antigravity/comments/1trr4dy/antigravitys_aggressive_history_compaction_is/) — Reddit `r/google_antigravity`，发布日期：2026-05-30（访问日期：2026-11-04｜类型：专业社区讨论｜置信度：高）
[^22]: [How to force AG to use subagents? (Orchestrator + Ephemeral Subagents & File-based Handoff)](https://www.reddit.com/r/google_antigravity/comments/1uo7g2k/how_to_force_ag_to_use_subagents/) — Reddit `r/google_antigravity`，发布日期：2026-07（访问日期：2026-11-04｜类型：专业社区工程实践｜置信度：高）
[^23]: [Clarification Request: Is local IPC/PTY wrapping of `agy` CLI for open-standard editor bridges (e.g., ACP) ToS compliant? (#175905)](https://discuss.ai.google.dev/t/clarification-request-is-local-ipc-pty-wrapping-of-agy-cli-for-open-standard-editor-bridges-e-g-acp-tos-compliant/175905) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-07-23 至 2026-09-16（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^24]: [Is it allowed for an open-source tool to call the official `agy` CLI in headless mode? (#183429)](https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-17（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^25]: [Is using the official agy CLI through a local MCP server with third-party AI agents permitted? (#184829)](https://discuss.ai.google.dev/t/is-using-the-official-agy-cli-through-a-local-mcp-server-with-third-party-ai-agents-permitted/184829) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-25（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^26]: [One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? (#185460)](https://discuss.ai.google.dev/t/one-person-several-paid-google-ai-pro-accounts-may-a-local-orchestrator-use-them-with-the-official-agy-cli/185460) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^27]: [Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (#185494)](https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^28]: [Is invoking the official `agy` CLI from a local AI coding assistant allowed on an individual Google AI account? (#185992)](https://discuss.ai.google.dev/t/is-invoking-the-official-agy-cli-from-a-local-ai-coding-assistant-allowed-on-an-individual-google-ai-account/185992) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-09-29 至 2026-10-01（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^29]: [Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper (#185949)](https://discuss.ai.google.dev/t/clarification-on-codex-claude-skill-invoking-the-official-agy-cli-through-a-local-powershell-wrapper/185949) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-29 至 2026-10-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^30]: [Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? (#186954)](https://discuss.ai.google.dev/t/is-automatically-launching-the-official-agy-cli-from-a-local-ai-orchestrator-allowed-under-additional-terms-6/186954) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-05（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^31]: [Is invoking the official Antigravity CLI from Codex allowed? (#186952)](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-05（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^32]: [Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (#186246)](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-10-01 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^33]: [Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator (#187536)](https://discuss.ai.google.dev/t/terms-clarification-personal-antigravity-cli-integration-with-a-multi-ai-orchestrator/187536) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-10-08（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^34]: [Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-10-07 至 2026-10-08（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^35]: [Clarification on consumer-plan policy for invoking official agy via local subprocess (#189905)](https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-16 至 2026-10-26（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^36]: [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-23 至 2026-10-27（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^37]: [Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998)](https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-26（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^38]: [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-10-19 至 2026-10-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^39]: [Policy clarification: private monitoring app using official Antigravity CLI and subscription quota (#190756)](https://discuss.ai.google.dev/t/policy-clarification-private-monitoring-app-using-official-antigravity-cli-and-subscription-quota/190756) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`，评论：`@oli_hlmpn`），发布日期：2026-10-19 至 2026-10-29（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^40]: [Clarification on subscription-based agy -p integration with a local multi-agent framework (#196472)](https://discuss.ai.google.dev/t/clarification-on-subscription-based-agy-p-integration-with-a-local-multi-agent-framework/196472) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-28 至 2026-10-29（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^41]: [Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131)](https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-31 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^42]: [Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351)](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351) — Google AI Developers Forum（作者：`@Yiji_Kong`，官方回复：`@Maniraj_Chervirala`），发布日期：2026-10-28 至 2026-10-29（访问日期：2026-11-04｜类型：一手实测报告与官方论坛回复｜置信度：高）
[^43]: [Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476) — Google AI Developers Forum（提问：`@Hello_there`，官方回复：`@chunduriv`），发布日期：2026-10-15 至 2026-10-20（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^44]: [Feature request: Show context window usage in Antigravity Desktop (#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694) — Google AI Developers Forum（提问：`@Soro`，官方回复：`@chunduriv`，插件：`@dimitr-io`），发布日期：2026-09-24 至 2026-10-22（访问日期：2026-11-04｜类型：一手官方论坛回复｜置信度：高）
[^45]: [Persistent Gemini CLI TOS_VIOLATION after appeal — request for private review or sync check (#186089)](https://discuss.ai.google.dev/t/persistent-gemini-cli-tos-violation-after-appeal-request-for-private-review-or-sync-check/186089) — Google AI Developers Forum（作者：`@myessaylab1234`），发布日期：2026-09-30（访问日期：2026-11-04｜类型：专业社区报告｜置信度：高）
[^46]: [Antigravity CLI Agent reliability issue (#176775)](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775) — Google AI Developers Forum（作者：`@Pitt_zhang`），发布日期：2026-08-01（访问日期：2026-11-04｜类型：专业社区报告｜置信度：高）
[^47]: [Google Antigravity FAQ 政策更新与停止第三方工具連携说明](https://note.com/horizon_it00/n/nded285d721d1) — note.com（作者：`horizon_it00`），发布日期：2026-10（访问日期：2026-11-04｜类型：专业技术博客｜置信度：高）
[^48]: [Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (`239,616` Compaction Threshold)](https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk) — Nous Research Hermes Agent 文档，发布日期：2026-09-23（访问日期：2026-11-04｜类型：专业技术文档｜置信度：高）
[^49]: [mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management](https://github.com/mksglu/context-mode) — GitHub（作者：`mksglu`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^50]: [AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE](https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor) — GitHub（作者：`AGI-is-going-to-arrive`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^51]: [dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context) — GitHub（作者：`dimitr-io`），发布日期：2026-10-22（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^52]: [weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI](https://github.com/weby-homelab/antigravity-cli-statusline) — GitHub（作者：`weby-homelab`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^53]: [oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit ("The IDE for Your Coding Agent")](https://github.com/oraios/serena) — GitHub（作者：Oraios AI），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^54]: [xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server)](https://github.com/xdotech/goatlas) — GitHub（作者：`xdotech`），发布日期：2026（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^55]: [Kevin Hou — "Get Out of the Model's Way" (AI Engineer Summit Presentation & Speaker Profile)](https://www.youtube.com/watch?v=xsVD9_cJNYs) — AI Engineer / YouTube（及 `https://ai.engineer/speakers/kevin-hou`），发布日期：2026（访问日期：2026-11-04｜类型：一手高管公开演讲｜置信度：高）
[^56]: [Varun Mohan Education Qualifications: How an Indian-Origin MIT Engineer Secured Google's $2.4 Billion AI Investment](https://timesofindia.indiatimes.com/education/news/varun-mohan-education-qualifications-how-an-indian-origin-mit-engineer-secured-googles-2-4-billion-ai-investment/articleshow/122405531.cms) — Times of India，发布日期：2025-07（访问日期：2026-11-04｜类型：权威媒体报道｜置信度：高）
[^57]: [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) — Google AI Developers Forum（提问：`@Sharper`，官方回复：`@chunduriv`），发布日期：2026-11-04（访问日期：2026-11-05｜类型：一手官方人员论坛回复｜置信度：高）
