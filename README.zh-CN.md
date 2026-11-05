# Google Antigravity 256K 上下文限制与无头模式政策变更档案

**语言 / Language**：[English](./README.md) | **简体中文**

**网页档案**：[https://antigravity-256k-context-discrepancy.pages.dev/](https://antigravity-256k-context-discrepancy.pages.dev/)

Google Antigravity（包括 Antigravity 2.0 桌面端应用、Antigravity IDE 与 `agy` 命令行工具）在对外说明与客户端界面中均标称 `Gemini 4 Argon` 及 `Gemini 3.8 Flash` 支持 `1,048,576`（1M）Tokens 上下文窗口。但对官方客户端二进制文件的静态反汇编与本地会话数据库统计表明，底层代码通过硬编码机器码指令将自动上下文压缩触发阈值限制在 `256,000` Tokens（标称窗口的 25%）。同一时期，Google 官方支持人员在未修改正式《附加服务条款（ToS）》正文的情况下，通过更新非合同性质的 FAQ 页面，撤回了此前在开发者论坛连续给出的 10 次关于“第三方编程工具通过本地子进程调用 `agy -p` 合规”的书面答复。

本仓库将相关的技术分析、反汇编记录与公开论坛/Issue 讨论整理为两部分纯文本材料：**第一部分 —— 事件调研与分析**（`research/`）与**第二部分 —— 二进制静态反汇编与本地数据库报告**（`evidence/`）。各目录顶层默认为英文版，`zh-cn/` 子目录为对应的简体中文版。本仓库不存放任何第三方二进制可执行文件或安装包；所有被分析的官方二进制文件均提供 Google 官方云存储（`storage.googleapis.com`）下载直链与 SHA-256 / SHA-512 校验哈希，以便独立核对。

---

## 核心结论

1. **标称 `1,048,576`（1M）窗口与底层 `0x3E800`（`256,000`）硬编码限制**：在桌面端 `v2.29.0`（`language_server.exe`）与 CLI `v1.5.0`–`v1.5.1`（`antigravity`）中，`defaultmodels.Gemini4Argon` 注册的最大上下文窗口确为 `0x100000`（`1,048,576` Tokens）。但底层检查点计算函数 `cortex/utils/utils.GetCheckpointConfig` 内包含硬编码 x86-64 指令序列 `cmp $0x3e800, %edx; mov $0x3e800, %esi; cmovb %edx, %esi`（机器码 `81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2`，`v1.5.0` 文件偏移 `0x7fe5802`，`v1.5.1` 文件偏移 `0x7fb0fe2`），将所有模型的自动压缩触发阈值限制为 `min(计算值, 256,000)`。尽管 `v1.5.0` 与 `v1.5.1` 恢复了此前在 `v1.3.5`–`v1.4.x` 中被编译为空字符串的 `os.LookupEnv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` 读取逻辑（`v1.5.1` 中只认绝对 Token 数、只对白名单模型生效），但 `0x3E800` 钳制指令本身并未改动（见 [分册二](./research/zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md) 与 [取证报告 §2.3 & §4](./evidence/zh-cn/evidence-report.md)）。2026 年 11 月 4 日，Google 官方人员 `@chunduriv` 在开发者论坛 [`#198191`](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) 帖中确认，桌面端中的 Gemini 模型会在约 250K Tokens 时自动压缩，且该阈值“is a fixed default and isn't user-configurable yet”（是固定默认值，暂不支持用户配置）（见 [分册五 引语 1.8](./research/zh-cn/05-verbatim-quotes-and-evidence-vault.md)）。
2. **会话实际容量上限固定为 `256,000` Tokens，单次压缩中位耗时 70 秒**：本机补丁前（`< 2026-11-04 00:00:00 UTC+8`）209 个 Antigravity CLI 1M-Gemini 会话数据库（`~/.gemini/antigravity-cli/conversations/*.db`，共 15,575 次调用）中，15,575 条请求元数据均记录 `max_context_tokens = 256000`，单次无重试（`retries = 0`）调用超过 300K Tokens 的次数为 0。社区开发者 `@murias002` 在 Issue `#878` 对 310 个 CLI 会话（65,613 次调用）的独立统计从输入规模得出一致的上限：单次最高输入 `255,999` Tokens，超过 300K 的调用为 0 次。每次自动压缩中位停顿 **70 秒**，摘要调用本身的 Token 消耗不写入 `gen_metadata` 表，将约 25 万 Tokens 历史改写为 `15K–17K` 散文摘要（`CORTEX_STEP_TYPE_CHECKPOINT = 23`、`truncation_reason = 2`），并导致原有的 240K+ 隐式提示词缓存失效（见 [取证报告 §2.5–§3](./evidence/zh-cn/evidence-report.md) 与 [分册二 §3](./research/zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md)）。
3. **`Gemini 4 Argon` 3.0 倍配额消耗比与受控上下文选项**：在同一 5 小时配额窗口内的控制变量 A/B 测试表明，按公开 API 挂牌价折算后，`Gemini 4 Argon` 每美元消耗的订阅配额是 `Gemini 3.8 Flash` 的 **3.00 倍**（折合 `$6/$30` 每百万 Tokens，公开价为 `$2/$10`；Web 纯输入测试中为 `7.64–9.25 倍`）。此外，客户端二进制内已包含 `512K`（`1.3x` 配额倍率）与 `900K`（`1.8x` 配额倍率，`Expensive: true`）上下文选项，受仅含 `"Gemini 4 Argon"` 一项的白名单与服务端功能开关限制（见 [分册二 §3–§4](./research/zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md) 与 [取证报告 §2.4](./evidence/zh-cn/evidence-report.md)）。
4. **未修改 ToS 的 `agy -p` 政策反转**：2026 年 9 月 16 日至 10 月 8 日 16:23 UTC 期间，三位 Google 官方支持工程师在开发者论坛连续 10 帖书面回复确认，由第三方编程工具（如 Codex、Claude Code）通过本地子进程调用未修改的 `agy -p` 符合《附加服务条款》第 6 条。10 月 8 日 17:34 UTC，即最后一次确认 71 分钟后、`#186246` 的确认 26.5 小时后，官方支持人员在 `#187241` 改变口径并提示该用法可触发自动封号（`cloudcode-pa.googleapis.com` HTTP 403 `TOS_VIOLATION`），随后在论坛 `#191689` 第 8 楼确认正式《服务条款》自 9 月后未作修改，仅在 10 月中旬更新了非合同性质的《官方 FAQ》（见 [分册三](./research/zh-cn/03-agy-p-headless-policy-reversal-and-bans.md) 与 [分册五](./research/zh-cn/05-verbatim-quotes-and-evidence-vault.md)）。

---

## 第一部分：事件调研与分析（`research/`）

记录从 2025 年 7 月至 2026 年 11 月 5 日（`v1.5.1`）的完整时间线、二进制上下文限制反汇编、`AGY_CLI_AUTO_COMPACT_OVERRIDE` 在不同版本中的变化、Argon 配额消耗测试、`agy -p` 政策变更前后的论坛答复记录、社区与官方对该限制的应对以及原始引文汇总。

| 文档 | English Edition (Default) | 简体中文版 (`zh-cn/`) | 内容说明 |
| :--- | :--- | :--- | :--- |
| **总览报告** | [research/dossier.md](./research/dossier.md) | [research/zh-cn/dossier.md](./research/zh-cn/dossier.md) | 核心事实汇总、关键指标数据表、多方观点与全部 57 条脚注定义 |
| **分册一** | [01-incident-overview-and-timeline.md](./research/01-incident-overview-and-timeline.md) | [zh-cn/01-incident-overview-and-timeline.md](./research/zh-cn/01-incident-overview-and-timeline.md) | 项目背景、Windsurf 团队收购、旧版 `gemini-cli` 停用、配额池规则与按时间排序的事件时间线 |
| **分册二** | [02-256k-hardcoded-gates-and-binary-teardown.md](./research/02-256k-hardcoded-gates-and-binary-teardown.md) | [zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md](./research/zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md) | 上下文容量统计、`AGY_CLI_AUTO_COMPACT_OVERRIDE` 变化（`1.3.5` vs. `1.5.0–1.5.1`）、三处二进制限制（`0x3e800`）、70 秒压缩耗时与 Argon 配额实测 |
| **分册三** | [03-agy-p-headless-policy-reversal-and-bans.md](./research/03-agy-p-headless-policy-reversal-and-bans.md) | [zh-cn/03-agy-p-headless-policy-reversal-and-bans.md](./research/zh-cn/03-agy-p-headless-policy-reversal-and-bans.md) | 政策变更前连续 10 帖官方论坛确认、10 月 8 日口径变更、确认未改 ToS 仅改 FAQ 的经过及账号申诉情况 |
| **分册四** | [04-community-and-official-responses.md](./research/04-community-and-official-responses.md) | [zh-cn/04-community-and-official-responses.md](./research/zh-cn/04-community-and-official-responses.md) | 社区与官方对限制的应对：社区补丁作为限制位于客户端的佐证、Google 官方人员建议、Google 的 ADK 案例、第三方上下文工具与跨工具边界 |
| **分册五** | [05-verbatim-quotes-and-evidence-vault.md](./research/05-verbatim-quotes-and-evidence-vault.md) | [zh-cn/05-verbatim-quotes-and-evidence-vault.md](./research/zh-cn/05-verbatim-quotes-and-evidence-vault.md) | 逆向分析原文、汇编代码片段、政策变更前后 Google 工作人员论坛回复原文及 ToS/FAQ 文本对照 |
| **来源索引** | [research/sources.md](./research/sources.md) | [research/zh-cn/sources.md](./research/zh-cn/sources.md) | 57 条正文引用来源与 9 条补充参考来源的元数据及原文摘录 |
| **单文件完整报告** | [antigravity-256k-report.md](./antigravity-256k-report.md) | — | 单文件英文完整报告，汇总核心结论、时间线、反汇编、配额实测、政策记录、原文引语、二进制哈希与全部 57 条引用来源 |

---

## 第二部分：二进制静态反汇编与本地数据库报告（`evidence/`）

提供针对官方桌面端安装包 `Antigravity-x64.exe`（`v2.29.0`）内嵌的 `language_server.exe`（PE32+ x86-64，SHA-256 `c98e7c33...e1c3d5`）与 CLI `antigravity`（`v1.5.0` BuildID `5fb6af4f...`，以及 `v1.5.1` BuildID `696031ae...`）的只读静态反汇编比对、Antigravity CLI 本地 SQLite 数据库表结构与 Protobuf 字段拆解（含本机补丁前 237 个 CLI 会话库、209 个 1M-Gemini 会话共 15,575 次调用全量核验），以及 10 个相关 GitHub Issue 的接口快照。

| 文档 / 目录 | 路径 | 说明 |
| :--- | :--- | :--- |
| **英文版报告 (Default)** | [evidence/evidence-report.md](./evidence/evidence-report.md) | 0–9 共 10 章英文静态分析报告（安装包校验、机器码偏移对比、SQLite 统计与复现命令） |
| **中文版报告 (`zh-cn/`)** | [evidence/zh-cn/evidence-report.md](./evidence/zh-cn/evidence-report.md) | 0–9 共 10 章中文静态分析报告（与英文版章节结构一一对应） |
| **原始 JSON 快照与清单** | [evidence/raw/](./evidence/raw/) | 包含官方 `manifest_linux_amd64.json`、21 份 GitHub API 原始 JSON 响应及 1 份论坛主题 JSON 快照（共 23 个文本文件） |

### 被分析官方二进制文件与校验哈希

本仓库不托管任何二进制样本。可通过以下 Google 官方云存储（`storage.googleapis.com/antigravity-public`）公开地址直接下载原件并核对哈希：

| 组件 | 版本 / 平台 | 官方下载直链 / 来源 | 文件大小与校验哈希 |
| :--- | :--- | :--- | :--- |
| **桌面端官方安装包** | `v2.29.0` (Windows x64) | [`Antigravity-x64.exe`](https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-x64/Antigravity-x64.exe) | `169,295,640` 字节<br>SHA-256: `944b109bcb377f9f94f86eeea7cfa9315d4b42d84fb9d5bad689058fc11c12ba` |
| **桌面端语言服务程序** | `v2.29.0` (`resources/bin/language_server.exe`) | 随上述 `Antigravity-x64.exe` 安装包内嵌 `app-64.7z` 本地打包下发 | `187,342,336` 字节<br>SHA-256: `c98e7c331da5bcc37002a550e7d625b4c636f775fa2804f73777c6a500e1c3d5` |
| **CLI 官方发布包 (`v1.5.0`)** | `v1.5.0` (Linux x64, 2026-11-04) | [`cli_linux_x64.tar.gz`](https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64/cli_linux_x64.tar.gz)（清单：[`linux_amd64.json`](https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/linux_amd64.json)） | 压缩包 `69,781,910` 字节（解压后 `antigravity` 为 `238,604,688` 字节，BuildID `5fb6af4f2e1d489f3ada0fd785060f48`）<br>SHA-512: `e6031f850929210705d8c2393a5665b82b8c3b19684a6d3c556cafa31f26f77acbf164dfee96ca84a5ae7670ac0cc238a974c981c517b269194bdccff271cbd5` |
| **CLI 二进制文件 (`v1.5.1`)** | `v1.5.1` (Linux x64, 2026-11-05) | [`cli_linux_x64.tar.gz`](https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.1-4523430503120896/linux-x64/cli_linux_x64.tar.gz)（清单：[`linux_amd64.json`](https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/linux_amd64.json)；更新日志：[`CHANGELOG.md`](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md)） | 压缩包 `72,209,583` 字节，SHA-512: `f70406654ebd966f211424338259e2d38d099c48734e901dc06975063e5a0137d55b6eb9366b75983c6908bbc20ae92a2617656bb191d8ed6fe429d654bca4f5`<br>解压后 `antigravity`：`240,857,488` 字节（BuildID `696031ae34e5e5e26a9d7c436e85317f`），SHA-256: `72738cf0445918483956a5f0da33abdfcfb21a12dd9a421fd57d063782826669` |
| **CLI 官方安装脚本** | 全平台入口 | [`https://antigravity.google/cli/install.sh`](https://antigravity.google/cli/install.sh) | `7,643` 字节<br>SHA-256: `62966c07365423bd4dc209355060744058fb30d60f5323e2d360e39de64e5042`<br>（本仓库不转载原文；2026-11-04 获取，2026-11-05 复核未变） |

---

## 目录结构

```text
antigravity-256k-context-discrepancy/
├── README.md                                  # 英文说明、核心结论、目录索引与校验哈希
├── README.zh-CN.md                            # 中文说明、核心结论、目录索引与校验哈希
├── LICENSE                                    # 多重许可协议与合理使用说明（中英双语）
├── antigravity-256k-report.md                 # 单文件英文完整报告（汇总第一、二部分）
├── research/                                  # [第一部分] 事件调研报告（顶层英文 + zh-cn/ 中文版）
│   ├── dossier.md                             # 总览报告与关键指标数据表
│   ├── 01-incident-overview-and-timeline.md
│   ├── 02-256k-hardcoded-gates-and-binary-teardown.md
│   ├── 03-agy-p-headless-policy-reversal-and-bans.md
│   ├── 04-community-and-official-responses.md
│   ├── 05-verbatim-quotes-and-evidence-vault.md
│   ├── sources.md                             # 57 条正文来源 + 9 条补充来源元数据与原文摘录
│   └── zh-cn/                                 # 简体中文版（文件名与英文版 1:1 对应）
│       ├── dossier.md
│       ├── 01-incident-overview-and-timeline.md
│       ├── 02-256k-hardcoded-gates-and-binary-teardown.md
│       ├── 03-agy-p-headless-policy-reversal-and-bans.md
│       ├── 04-community-and-official-responses.md
│       ├── 05-verbatim-quotes-and-evidence-vault.md
│       └── sources.md
├── evidence/                                  # [第二部分] 二进制静态反汇编与本地数据库报告
│   ├── evidence-report.md                     # 英文版 10 章静态分析报告
│   ├── zh-cn/
│   │   └── evidence-report.md                 # 中文版 10 章静态分析报告
│   └── raw/                                   # 原始文本材料（共 23 个文本/JSON 文件，不含二进制文件）
│       ├── manifest_linux_amd64.json          # 官方 CLI 版本下载清单（v1.5.0 SHA-512 来源）
│       ├── forum_198191.json                  # 论坛 #198191 帖快照（官方确认约 250K 阈值固定）
│       └── *.json                             # 21 份 GitHub API 原始 JSON 响应
└── index.html                                 # 独立单文件静态归档网页（https://antigravity-256k-context-discrepancy.pages.dev/）
```

---

## 许可协议与引用说明

本仓库对原创材料与引用材料分别适用不同条款（完整双语条款见 [LICENSE](./LICENSE)）：

1. **原创调研报告、反汇编分析与统计表格（`research/*.md`、`research/zh-cn/*.md`、`evidence/evidence-report.md`、`evidence/zh-cn/evidence-report.md`、`antigravity-256k-report.md`、`index.html`）**：采用 **[CC BY 4.0（知识共享 署名 4.0 国际）](https://creativecommons.org/licenses/by/4.0/deed.zh-hans)** 许可。你可以自由复制、转载、翻译或引用这些内容，只需在引用时注明本 GitHub 仓库链接（`https://github.com/magiccodelab/antigravity-256k-context-discrepancy`）或作者博客文章链接（`https://afengblog.com/blog/antigravity-256k-context-discrepancy.html`）即可。
2. **第三方接口快照（`evidence/raw/`）与引用原文**：版本清单、21 份 GitHub API JSON 快照、论坛主题 JSON 快照，以及报告中引用的论坛回复、《附加服务条款（ToS）》与《官方 FAQ》原文，其著作权仍归各自权利人（包括 Google LLC 及相关社区贡献者）所有，仅依据**“合理使用（Fair Use，17 U.S.C. § 107）”**原则用于技术研究与文档记录，不适用上述 CC BY 4.0 许可。第三方脚本不在本仓库转载：官方安装脚本仅以 URL 与 SHA-256 引用，社区补丁脚本 `patch_agy.py` 的著作权归作者 [`@LiraNuna`](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) 所有。

---

## 说明与免责声明

- **整理与核对说明**：本仓库中的材料由作者配合 **Google Antigravity（`Gemini 4 Argon`）** 共同整理。受限于个人时间与精力，文中可能存在疏漏；如有任何疑问或出入，请直接以仓库内的原始引用信源（[`research/zh-cn/sources.md`](./research/zh-cn/sources.md)）及原始接口快照（[`evidence/raw/`](./evidence/raw/)）为准。
- **时区说明（`UTC+8`）**：由于作者日常处于 **`UTC+8`（东八区）** 时区，文档中部分按本地日历或本地文件时间戳记录的日期，与 GitHub、开发者论坛使用的 `UTC`（或美西时间）时间戳相比可能存在跨日偏差（前后相差 1 天以内）。这类日期差异属于时区换算偏差而非事实错误；需要核对精确发生时刻时，请参考时间线中标注的 `UTC` 时间或 [`research/zh-cn/sources.md`](./research/zh-cn/sources.md)、[`evidence/raw/`](./evidence/raw/) 中的原始记录。

> **有趣的是**：如果仅依赖官方 Google Antigravity CLI 现有的默认 `256,000` Tokens（`0x3E800`）上下文自动压缩阈值，Antigravity 本身会在中途因强制有损压缩而丢失上下文，根本无法配合我完成这份档案库的整理任务。

**🙏 特别致谢（Special Acknowledgements）：**

我们由衷感谢 Google DeepMind Gemini 团队训练出如此真正卓越的 1M 上下文模型（Gemini 4 Argon），使其能够综合整理出这整套取证档案库。

同样，也要特别感谢 Google Antigravity 产品与商业化团队（Product & Monetization team）做出的极具远见的产品决策——尤其是他们秉持“不挡模型的路（Get Out of the Model's Way）”的理念，将 0x3E800（256,000）硬编码进二进制文件，并通过修改一页 FAQ 推翻了 10 次官方书面承诺。如果没有你们这种“用户至上（user-first）”的工程实践，本仓库根本不会有存在的理由。
