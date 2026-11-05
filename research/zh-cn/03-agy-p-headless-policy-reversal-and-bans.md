# 分册三：`agy -p` 无头模式政策变更记录、政策背景与账号申诉情况

- **语言 / Language**：[English (../03-agy-p-headless-policy-reversal-and-bans.md)](../03-agy-p-headless-policy-reversal-and-bans.md) | **简体中文**
- **档案库全局导航**：[仓库总览](../../README.zh-CN.md) ｜ [调研总览 (dossier.md)](./dossier.md) ｜ [全量来源档案 (sources.md)](./sources.md) ｜ [静态逆向取证报告](../../evidence/zh-cn/evidence-report.md)
- **专题分册导航**：[01-事件背景与时间线](./01-incident-overview-and-timeline.md) ｜ [02-256K硬编码与逆向拆包](./02-256k-hardcoded-gates-and-binary-teardown.md) ｜ **03-agy-p政策反转与封号** ｜ [04-社区与官方应对](./04-community-and-official-responses.md) ｜ [05-原文引用与记录汇总](./05-verbatim-quotes-and-evidence-vault.md)
- **研究 ID (UUID)**：`df866cf1-da40-4b82-8084-34c6b0729544` ｜ **调研时间**：2026-11-04 ｜ **置信度校准**：已核验 Google AI 开发者论坛全部 20+ 原始 JSON 帖子记录、ToS 原文与官网 FAQ

---

## 1. 条款文本背景：《附加服务条款》第 6 条 vs. 官网《FAQ 页面》

### 1.1 正式法律合同文本：《Google Antigravity 附加服务条款》第 6 条（自 2026 年 9 月起未发生任何修改）
根据 [Google Antigravity Additional Terms of Service](https://antigravity.google/terms) 原文，具有约束力的合同协议（Agreement）仅由三份文件构成：①《Google 通用服务条款》、②《Google Antigravity 附加服务条款》、③《Google 隐私权政策》（注：**官网 FAQ 页面并不在合同第 1 条与第 2 条列明的具有法律约束力的协议文件清单之中**）[^14]。
其中第 6 条（Section 6）全文如下：
> *“6. You must not abuse, harm, interfere with, or disrupt the Service. This includes, but is not limited to, using the Service in connection with products not provided by us. Using third party software, tools, or services to access the Service (e.g. using OpenClaw with Antigravity OAuth) is a breach of this Agreement. Such actions may be grounds for suspension or termination of your Antigravity and/or Gemini CLI accounts.”*[^14]
> （中文精译：“6. 您不得滥用、损害、干扰或破坏本服务。这包括但不限于将本服务与非我们提供的产品结合使用。使用第三方软件、工具或服务访问本服务（例如将 OpenClaw 与 Antigravity OAuth 配合使用）属于违反本协议的行为。此类行为可能成为暂停或终止您的 Antigravity 和/或 Gemini CLI 账号的依据。”）

### 1.2 10 月中旬静默新增的《官方 FAQ》条目
由于 `agy` CLI 官方专门开发并文档化了无头模式（`agy -p "<prompt>"`、`--output-format json`、`--output-format stream-json`），其设计目的就是供外部进程通过标准输入输出（`stdio`）进行脚本化调用。然而在 **2026 年 10 月 6 日至 8 日左右**，Google 在未改动上述第 6 条任何文字、也未向付费订阅用户发送合同变更通知邮件的情况下，在非合同性质的 [官方 FAQ 页面](https://antigravity.google/docs/faq/) 下方静默新增了如下问答条目[^13][^36]：
> *“## Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?*
> *Antigravity CLI headless mode (`agy -p`) is intended for direct scripting and CI workflows. Wrapping `agy` inside third-party coding agents or applications, or automating account rotation, is not supported under consumer plans.”*[^13]

---

## 2. 第一阶段（2026-09-16 至 2026-10-08 16:23 UTC）：Google 官方人员连续 9 次书面承诺“本地子进程调用 `agy -p` 完全合规”

在 2026 年 7 月至 10 月初，大量开发者为了严格遵守第 6 条、避免账号被误封，在编写集成脚本或购买年费订阅前，主动前往官方指定的支持论坛 `discuss.ai.google.dev/c/antigravity/64` 发帖求证。在长达三个多星期的时间里，三位 Google 官方技术支持工程师（`@Rebbanpalli_Naveen`、`@Ambati_Rajendra`、`@chunduriv`）给出了**高度一致、毫不含糊的肯定答复**：只要调用的是官方未修改的 `agy` / `agy.exe` 二进制文件，且不提取、不代理 OAuth 令牌，无论是个人编排器、开源工具、OpenAI Codex 还是 Anthropic Claude Code 通过本地子进程调用 `agy -p`，均**完全合规（Permitted / Compliant / Completely fine to use）**[^23][^24][^27][^28][^29][^30][^31][^32][^33]。

### 表 1：反转前 Google 官方人员书面确认“子进程调用 `agy` 合规”的 9 份一手证据全集

| 序号 | 论坛帖号与标题 | 官方回复时间 (UTC) | 提问者与具体场景 | 回复的 Google 官方人员 | 官方核心定性原话摘录 | 溯源 |
| :---: | :--- | :---: | :--- | :---: | :--- | :---: |
| **1** | **#175905**<br>*Is local IPC/PTY wrapping of `agy` CLI for open-standard editor bridges (e.g., ACP) ToS compliant?* | `2026-09-16 12:45:42` | `@shindgew`：开源项目 `agy-acp` 通过本地单用户子进程/PTY 包装未修改的 `agy` 对接 Zed/Neovim | `@Rebbanpalli_Naveen` | *"**Yes, your described pattern is compliant.** The ToS restriction on third-party access tools is aimed at credential harvesting, unofficial API reverse-engineering, and multi-tenant proxying. Spawning the official, unmodified `agy` binary locally as a single-user child process/PTY under the user’s own `agy login` session complies..."* | [^23] |
| **2** | **#183429**<br>*Is it allowed for an open-source tool to call the official `agy` CLI in headless mode?* | `2026-09-17 12:06:16` | `@darkest_alex`：开源 Python 工具 `sprite-gen` 在本地调用 `agy --output-format json -p` | `@Ambati_Rajendra` | *"Is calling `agy` headless allowed? **Yes.** Calling the official `agy` binary locally via `agy -p` is an officially supported feature built for programmatic scripting... **It does not violate Section 6**, provided your tool never extracts, stores, or intercepts the user’s OAuth credentials."* | [^24] |
| **3** | **#185494**<br>*Headless agy runs launched by a script from another AI coding tool: within the Additional Terms?* | `2026-09-28 07:14:31` | `@Diggy_Styon`：Google AI Pro 订阅下，由另一个 AI 编程助手启动脚本运行 `agy -p`（含调用计划内 Claude 模型） | `@Ambati_Rajendra` | *"**Yes, your use case is permitted.** Running the official `agy` binary locally (even headless via a script) under your own account is compliant with the terms... As long as you are executing the unmodified `agy` CLI and not proxying your auth tokens into another application, **you are good to go!**"* | [^27] |
| **4** | **#185460**<br>*One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI?* | `2026-09-28 12:22:17` | `@Winez`：Windows 11 本地单用户编排器将未修改的 `agy` 作为子进程调用（询问多账号轮换） | `@Ambati_Rajendra` | *"While **we permit running the official `agy` CLI locally as a subprocess for your own personal use** (as long as you are using your single personal account and interacting with it directly), using an orchestrator to cycle through multiple Google AI Pro accounts... is a direct violation..."* | [^26] |
| **5** | **#185992**<br>*Is invoking the official `agy` CLI from a local AI coding assistant allowed on an individual Google AI account?* | `2026-09-30 23:25:25` | `@Ivan_Valenzuela`：Anthropic 的 **Claude Code** 在本地 Windows 机器上将 `agy.exe` 作为子进程拉起并读写 `stdio` JSON | `@chunduriv` | *"Is invoking the unmodified `agy.exe` binary in headless mode from a local assistant permitted on an individual account? **Yes.** Section 6... prohibits extracting, proxying, or reusing Antigravity OAuth credentials inside third-party harnesses to call Antigravity’s backend endpoints directly."* | [^28] |
| **6** | **#185949**<br>*Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper* | `2026-10-02 20:20:25` | `@nothing2`：**Codex CLI 或 Claude Code** → 本地 Skill → `scripts/invoke.ps1` → 官方未修改 `agy` CLI | `@Ambati_Rajendra` | *"Short answer: **yes, your setup is fine.** ... **Codex or Claude calling the Skill: Yes**, as long as it sends `agy` a real task (like a code review or research prompt) and `agy` runs with your own local login."* | [^29] |
| **7** | **#186954**<br>*Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6?* | `2026-10-05 10:17:42` | `@local_ai_builder`：本地编排器自动将官方未修改的 `agy` 作为子进程拉起（无需每次人工点击） | `@Rebbanpalli_Naveen` | *"**Yes, the single-user local workflow you described is supported through the official CLI Headless Mode.** Local Child Process vs. Section 6: **Yes, this is permitted and materially different.**"* | [^30] |
| **8** | **#186952**<br>*Is invoking the official Antigravity CLI from Codex allowed?* | `2026-10-05 10:24:45` | `@Hossy`：Windows PC 上 **OpenAI Codex** 将官方未修改的 `agy.exe` 作为本地进程调用并通过 `stdin/stdout` 传文本 | `@Rebbanpalli_Naveen` | *"**Yes, the workflow you described is completely fine to use with your personal Google AI Pro subscription.** ... Having Codex invoke the official, unmodified `agy.exe` binary as a local process on your Windows PC—while authentication (`agy auth login`) and all backend communication stay strictly inside `agy.exe`—**is a supported use of CLI Headless Mode.**"* | [^31] |
| **9** | **#186246**<br>*Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in?* | `2026-10-07 15:03:46` | `@Croixofwar`：**Claude Code** 或自建本地协调器将未修改的 `agy` 作为子进程拉起，询问与手动运行脚本是否有区别 | `@chunduriv` | *"1. Is launching the official `agy` CLI as a local subprocess allowed under Section 6? **Yes.** ... 2. Does the answer change if you launch the script yourself vs. Claude Code launching it? **No. From a policy standpoint, there is no difference whether the unmodified `agy` subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code)**..."* | [^32] |
| **10** | **#187536**<br>*Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator* | `2026-10-08 16:23:34` | `@BuddhaRyu`：私人 Linux 服务器（Oracle Cloud / Vultr）上协调 `agy`、**OpenAI Codex** 与 **Claude Code** 自动交接任务 | `@Ambati_Rajendra` | *"**Running `agy` from your orchestrator: Yes.** ... **Handoffs between `agy`, Codex and Claude Code: Yes.** Passing results like diffs or review notes between tools in your own project is fine... **Does section 6 apply? No.** Only the official `agy` program talks to Antigravity, under your own sign-in..."* | [^33] |

---

## 3. 第二阶段（2026-10-08 17:34 UTC 至 2026-11-02）：10 月 8 日政策变更、FAQ 更新与限制范围扩大

### 3.1 转折节点：26.5 小时内口径变更（`#186246` → `#187241`）
对比论坛原始 UTC 时间戳可以看到以下时间顺序：
- **2026-10-07 15:03:46 UTC**，Google 官方人员 `@chunduriv` 在 **#186246** 中刚刚白纸黑字写下：*“从政策角度来看，未修改的 `agy` 子进程究竟是由你自己的脚本启动，还是由另一个本地编程助手（如 Claude Code）启动，没有任何区别（there is no difference）。”*[^32]
- **2026-10-08 16:23:34 UTC**，`@Ambati_Rajendra` 在 **#187536** 中仍在回复用户：在个人编排器中将 `agy` 与 Codex、Claude Code 组合自动交接任务“完全没问题（Yes... Does section 6 apply? No）”[^33]。
- **2026-10-08 17:34:41 UTC**（**距 `#187536` 仅隔 71 分钟，距 `@chunduriv` 自己在 `#186246` 的回复仅隔 26 小时 31 分钟**），`@chunduriv` 在 **#187241** 帖回复年费 Google AI Pro 用户 `@ALAMDAR_BALOCH` 提出的完全相同的“用 Codex 调用官方 `agy` CLI”问题时，突然在开头加上法务免责声明，并给出截然相反的封号警告[^34]：
  > *“While we cannot provide formal legal interpretations of the Terms of Service... While `agy` offers a documented headless mode (`agy -p`) for your own local scripts and CI pipelines, **we do not support using third-party coding agents (like Codex, Claude Code, OpenClaw, or OpenCode) to orchestrate or wrap `agy` under a consumer login. Routing third-party agent prompts or automated loops through Antigravity can trigger automated ToS enforcement and account suspension.**”*[^34]

### 3.2 宣布“FAQ 取代早期论坛回复”与被付费用户当面质问（`#189905` & `#191689`）
1. **2026-10-16（`#189905` 正式宣布废除旧回复）**：
   开发者 `@Amane2418` 在发现官网 FAQ 与论坛 `#185992` 等帖子的官方回复完全冲突后，发帖要求给出明确的 Yes/No 答复。`@Rebbanpalli_Naveen`（即 11 天前在 `#186952` 中亲口对 `@Hossy` 说 Codex 调用 `agy.exe` 完全合规的同一位工程师）出面回复称：
   > *“Current Official Guidance: **The Antigravity FAQ reflects our current policy for Antigravity CLI headless mode and supersedes earlier forum comments where they differ.** ... Even if you invoke the official, unmodified `agy` binary as a local subprocess without extracting OAuth tokens or reverse-engineering internal endpoints, wrapping `agy` inside third-party coding agents or applications—or automating account rotation—is not supported under consumer plans... violates our Terms of Service and can result in account suspension or termination.”*[^35]
2. **2026-10-23 至 10-27（`#191689` 用户连环质问与官方承认未改 ToS）**：
   在帖子 `#191689` 中，付费 Google AI Pro 用户 `@eliavb` 指出 Claude 与 OpenAI 均允许个人订阅用户通过官方 CLI（Claude Code、Codex）在自建本地环境中使用订阅额度，唯独 Google 要求用户“既付订阅月费、又额外按 Token 付 API 费”[^36]。随后，用户 `@b_w`（第 4、6 楼）与 `@Tora39`（第 7 楼）直接贴出 `#186952`（10 月 5 日）与 `#186246`（10 月 7 日）的链接当面质问 `@Rebbanpalli_Naveen`：
   - `@b_w` 质问：*“You wrote ‘After that October 5 reply in #186952 was posted, the policy on using Antigravity CLI headless mode in third-party coding agents was updated’ - **Can you please point me to the evidence of this? I am unable to find any evidence of a policy update after October 5th. The link only directs me to a frequently asked questions page. The most recent terms of service updates I am able to find are from September.**”*[^36]
   - 在第 5 楼（10 月 26 日）与第 8 楼（10 月 27 日），`@Rebbanpalli_Naveen` 被迫公开承认：
     > *“To clarify the timeline for `@b_w` and answer `@Tora39`’s question (a): **the Google Antigravity Terms of Service themselves were not modified after September; rather, the Official FAQ was updated in mid-October 2026 to add the section ‘Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?’** ... Earlier forum replies in #186246 and #186952 gave an overly broad interpretation before that FAQ entry was published.”*[^36]

### 3.3 连环封堵：连“本地文件传任务”与“独立 Python 协调器”也一律定性为违规（`#190815` / `#196472` / `#190756`）
在 10 月下旬的后续回复中，Google 官方人员进一步将禁令范围从“直接子进程 `stdio` 管道调用”扩大到了几乎所有与外部 AI 协同的本地工作流：
1. **禁止通过本地文件夹在 Claude Code 与独立 Python 脚本之间传递任务（`#190815`，2026-10-28）**：
   用户 `@Lars` 提出：由一个独立的 Python 脚本读取本地文件夹中的任务并调用 `agy -p`，Claude Code 从不接触 `agy` 进程或 Google 凭据，仅负责在本地文件夹中准备任务文件。`@Ambati_Rajendra` 明确否决：
   > *“**That still counts as third-party agent coordination. Passing tasks from Claude Code to `agy -p` through local files instead of stdin/stdout pipes still uses consumer quota as a backend worker for an external coding agent**, so it requires a Gemini API key or Gemini Enterprise.”*[^38]
2. **禁止通过独立本地 Python 协调服务启动 `agy -p`（`#196472`，2026-10-29）**：
   用户 `@Dsssyc` 询问：由一个独立的本地 Python 任务协调服务（而非 Host Agent 直接）负责调度并启动原版 `agy -p` 进程是否合规。`@Rebbanpalli_Naveen` 同样否决，称只要 `agy` 在个人登录下被用作外部多智能体框架的执行后端，即可能触发自动封号[^40]。面对 `@Dsssyc` 在第 3 楼进一步追问“如果框架只生成脚本、由人类用户在终端手动复制粘贴运行命令，再把结果传回框架审阅是否也违规”，截至 11 月 4 日官方未能给出自洽答复[^40]。
3. **禁止个人监控应用的无人值守自动触发（`#190756`，2026-10-28）**：
   用户 `@xorz` 询问在 Windows 上用自编私有监控程序在检测到系统报错时自动低频调用 `agy -p` 分析日志是否允许；`@Ambati_Rajendra` 回复称：手动运行脚本可以，但**任何无人值守/后台自动触发请求（unattended/automatic background requests，如事件驱动守护进程或轮询循环）在个人计划下均不支持，即使设置了本地速率限制也可能触发自动滥用/ToS 封禁**[^39]。

---

## 4. 技术检测限制、协议差异与二振出局申诉流程

### 4.1 操作系统底层的检测限制与 `agy acp` / `agy -p` 口径差异（`@oli_hlmpn` 的分析）
在论坛 **#190756**（10 月 29 日）与 **#197131**（11 月 2 日）中，开发者 `@oli_hlmpn` 与 `@Jay10`（`#189905`）指出了 Google 当前政策在计算机科学原理与用户体验上的三处矛盾[^35][^39][^41]：
1. **进程树与文本来源的不可区分性（“掷骰子式封号”）**：
   在 Linux/Windows 操作系统层面，当 `agy -p "<prompt>"` 被执行时，`agy` 进程只能看到拉起它的父进程是 `bash`、`powershell` 或 `python3`。至于这段 Prompt 字符串或本地 `.md` 任务文件在 5 分钟前究竟是人类手敲进 `vim` 的，还是由 Claude Code / ChatGPT 辅助生成的，操作系统内核与 `agy` 二进制根本无从知晓[^37][^39]。正如 `@oli_hlmpn` 所言：
   > *“And lastly, the most concerning part, **since determining if the command was executed by a local go server, a python script, or a 3 line bash script, is borderline impossible, does this not mean that even perfectly acceptable, fair use, explicitly allowed usage ends up being a dice roll?** If something that is explicitly allowed comes with the caveat of potentially causing a permanent suspension of a business account, does it not become a significant problem when that is not communicated clearly?”*[^39]
2. **`agy acp`（允许第三方客户端通过 `stdio` 调用）与 `agy -p`（不允许第三方客户端通过 `stdio` 调用）的政策差异**：
   在 2026 年 11 月 2 日的 `#197131` 帖及 11 月 4 日发布的 CLI `v1.5.0` 中，Google 官方明确允许第三方编辑器/客户端（如 Zed、Neovim、T3 Code）通过标准输入输出（`stdio`）拉起 `agy acp`（或 `antigravity-acp`）并使用个人订阅 OAuth 登录（`oauth-personal`）[^11][^41]；但如果在完全相同的机器、完全相同的个人账号下，用第三方工具通过 `stdio` 拉起 `agy --output-format stream-json -p`，却被归为信任与安全违规（Trust & Safety Violation）并面临账号暂停[^39][^41]。
3. **客户端缺乏前置警告提示（`@Jay10`）**：
   当用户配置编码助手调用 `agy -p` 时，命令在本地顺畅执行且返回结果，没有任何报错、没有任何交互式确认弹窗；即便 CLI 输出任何提示，也只会被父进程捕获而不会被人类用户看到，直到用户的 Google 账号收到 `403 TOS_VIOLATION` 封禁通知[^35]。

### 4.2 分析：为何一线支持人员先连续 10 次认可合规、后于 10 月 8 日改变口径？【分析推断】
*(注：本小节基于上述已核验的时间戳、论坛问答记录与企业合规运作机制进行客观推演，标注为【分析推断】)*
1. **一线工程师最初的 10 次肯定答复符合命令行工具设计常规与第 6 条原文**：
   从 Unix CLI 设计规范来看，`agy` 专门提供 `-p`（`--print`）和 `--output-format json / stream-json`，目的是供外部程序通过 `stdio` 调用。而在安全与计费架构上，只要外部程序调用的是未修改的官方 `agy` 二进制且不提取 OAuth Token，所有的身份认证、遥测上报以及 5 小时/周配额扣减都处于 Google 官方二进制与服务端（`cloudcode-pa.googleapis.com`）的管理之下[^31][^32]。因此，`@Ambati_Rajendra`、`@Rebbanpalli_Naveen` 和 `@chunduriv` 在 9 月 16 日至 10 月 8 日 16:23 UTC 期间给出的答复，是基于《附加服务条款》第 6 条原始文义（打击类似早期 OpenClaw 提取 OAuth Token 直接调用私有接口的行为）做出的技术解答[^24][^31][^32]。
2. **订阅计划商业边界与通过更新 FAQ 调整政策口径**：
   进入 10 月第一周后，随着 `#185992`、`#186246`、`#186952`、`#187536` 等帖子中越来越多的付费用户询问“能否将个人订阅接入 Claude Code / Codex / 多智能体编排器以省去按量 API 费用”并确认该回复是否代表官方解释[^28][^31][^32][^33]，这一用法触及了不同产品线的计费边界——Google 希望将固定月费的个人订阅（$20 Pro / $200 Ultra）限制在直接交互场景，而要求多智能体自动化工作流使用按 Token 计费的 Google AI Studio API 或企业版 Vertex AI（Gemini Enterprise）[^36][^38][^40]。
   由于正式修改《附加服务条款（ToS）》需要经过法务审批并向付费用户发送变更通知，实际采取的做法是：**在 10 月 8 日前后直接更新《官网 FAQ 页面》而未修改 ToS 正文，同时支持工程师在后续新帖回复中统一附上法务免责声明，说明此前的早期答复“给出了过于宽泛的解释（gave an overly broad interpretation）”**[^34][^35][^36]。

### 4.3 `cloudcode-pa.googleapis.com` 封号机制与超过 100 天未处理的申诉案例
1. **底层拦截与跨产品连坐**：
   Antigravity 的反滥用风控位于 `cloudcode-pa.googleapis.com` 后端网关。一旦账号被风控标记，客户端会收到 `HTTP 403 PERMISSION_DENIED`（`reason: TOS_VIOLATION`），并在终端打印：
   `Eligibility check failed: PERMISSION_DENIED (code 403): This service has been disabled in this account for violation of Terms of Service. Please submit an appeal to continue using this product.`[^8][^10][^45]
   由于共用同一后端层，该封禁会**同时锁死同一 Google 账号下的 Antigravity 2.0 桌面端、Antigravity IDE、`agy` CLI、旧版 `@google/gemini-cli` 以及 `Gemini Code Assist`**（另有部分个人 `@gmail.com` 付费账号遭遇 `Error #3501` 被错误标记为需要企业许可证或 `SUBSCRIPTION_REQUIRED` 而无法使用）[^8][^9][^45]。
2. **二振出局制（Two-Strike Policy）与申诉同步失灵**：
   尽管 Google 维护者 `@jackwotherspoon` 在 `Discussion #20632` 中承诺“首次违规填写 Google Form 重新认证后将通过周期性自动同步在 1–2 天内解封，第二次违规永久封禁”[^9]，但 GitHub **Issue #927**（`@jlwainwright`，自 2026 年 5 月 25 日起被封禁超 100 天，Google One 工单号 `2-9245000040533`）與 **Issue #29576 / 论坛 #186089**（`@myessaylab1234`，多次提交申诉表单每次提示 24–72 小时内审核却石沉大海）均证实：**自动解封同步机制对大量账号完全失效，而当付费用户联系 Google One 客服与 Google Developer Profile 支持团队时，双方均回复称“无权查看或升级处理 ToS 封禁状态”，使付费用户陷入投诉无门的死循环**[^8][^10][^45]。

---

## 本分册引用来源清单（脚注定义）

[^8]: [agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync (Issue #927)](https://github.com/google-antigravity/antigravity-cli/issues/927) — GitHub `google-antigravity/antigravity-cli`（作者：`jlwainwright`），发布日期：2026-09-02（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^9]: [Addressing Antigravity Bans & Reinstating Access (Discussion #20632)](https://github.com/google-gemini/gemini-cli/discussions/20632) — GitHub `google-gemini/gemini-cli`（作者：Google 维护者 `@jackwotherspoon`），发布日期：2026-02-27（访问日期：2026-11-04｜类型：一手官方公告｜置信度：高）
[^10]: [Appeal submitted but TOS_VIOLATION suspension persists (request for private follow-up) (Issue #29576)](https://github.com/google-gemini/gemini-cli/issues/29576) — GitHub `google-gemini/gemini-cli`（作者：`myessaylab1234`），发布日期：2026-09-30（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^11]: [Google Antigravity CLI Releases (`v1.4.2` – `v1.5.0`)](https://github.com/google-antigravity/antigravity-cli/releases) — GitHub `google-antigravity/antigravity-cli`，发布日期：2026-10-18 至 2026-11-04（访问日期：2026-11-04｜类型：一手官方发版日志｜置信度：高）
[^13]: [Google Antigravity Docs — Official FAQ](https://antigravity.google/docs/faq/) — Google Antigravity 官方文档，发布日期：2026-10（访问日期：2026-11-04｜类型：一手官方文档｜置信度：高）
[^14]: [Google Antigravity — Additional Terms of Service](https://antigravity.google/terms) — Google Antigravity 官方法律条款，发布日期：2026-09（访问日期：2026-11-04｜类型：一手官方合同条款｜置信度：高）
[^23]: [Clarification Request: Is local IPC/PTY wrapping of `agy` CLI for open-standard editor bridges (e.g., ACP) ToS compliant? (#175905)](https://discuss.ai.google.dev/t/clarification-request-is-local-ipc-pty-wrapping-of-agy-cli-for-open-standard-editor-bridges-e-g-acp-tos-compliant/175905) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-07-23 至 2026-09-16（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^24]: [Is it allowed for an open-source tool to call the official `agy` CLI in headless mode? (#183429)](https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-17（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^26]: [One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? (#185460)](https://discuss.ai.google.dev/t/one-person-several-paid-google-ai-pro-accounts-may-a-local-orchestrator-use-them-with-the-official-agy-cli/185460) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^27]: [Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (#185494)](https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^28]: [Is invoking the official `agy` CLI from a local AI coding assistant allowed on an individual Google AI account? (#185992)](https://discuss.ai.google.dev/t/is-invoking-the-official-agy-cli-from-a-local-ai-coding-assistant-allowed-on-an-individual-google-ai-account/185992) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-09-29 至 2026-10-01（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^29]: [Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper (#185949)](https://discuss.ai.google.dev/t/clarification-on-codex-claude-skill-invoking-the-official-agy-cli-through-a-local-powershell-wrapper/185949) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-09-29 至 2026-10-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^30]: [Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? (#186954)](https://discuss.ai.google.dev/t/is-automatically-launching-the-official-agy-cli-from-a-local-ai-orchestrator-allowed-under-additional-terms-6/186954) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-05（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^31]: [Is invoking the official Antigravity CLI from Codex allowed? (#186952)](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-05（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^32]: [Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (#186246)](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-10-01 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^33]: [Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator (#187536)](https://discuss.ai.google.dev/t/terms-clarification-personal-antigravity-cli-integration-with-a-multi-ai-orchestrator/187536) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-10-08（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^34]: [Is Google AI Pro Antigravity CLI automation with Codex permitted? (#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241) — Google AI Developers Forum（官方回复：`@chunduriv`），发布日期：2026-10-07 至 2026-10-08（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^35]: [Clarification on consumer-plan policy for invoking official agy via local subprocess (#189905)](https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`，用户反馈：`@Jay10`），发布日期：2026-10-16 至 2026-10-26（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^36]: [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`，用户质问：`@eliavb`、`@b_w`、`@Tora39`），发布日期：2026-10-23 至 2026-10-27（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^37]: [Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (#194998)](https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-26（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^38]: [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`），发布日期：2026-10-19 至 2026-10-28（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^39]: [Policy clarification: private monitoring app using official Antigravity CLI and subscription quota (#190756)](https://discuss.ai.google.dev/t/policy-clarification-private-monitoring-app-using-official-antigravity-cli-and-subscription-quota/190756) — Google AI Developers Forum（官方回复：`@Ambati_Rajendra`，评论：`@oli_hlmpn`），发布日期：2026-10-19 至 2026-10-29（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^40]: [Clarification on subscription-based agy -p integration with a local multi-agent framework (#196472)](https://discuss.ai.google.dev/t/clarification-on-subscription-based-agy-p-integration-with-a-local-multi-agent-framework/196472) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-28 至 2026-10-29（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^41]: [Conflicting guidance on using T3 Code with personal Google AI subscriptions (#197131)](https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131) — Google AI Developers Forum（官方回复：`@Rebbanpalli_Naveen`），发布日期：2026-10-31 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^45]: [Persistent Gemini CLI TOS_VIOLATION after appeal — request for private review or sync check (#186089)](https://discuss.ai.google.dev/t/persistent-gemini-cli-tos-violation-after-appeal-request-for-private-review-or-sync-check/186089) — Google AI Developers Forum（作者：`@myessaylab1234`），发布日期：2026-09-30（访问日期：2026-11-04｜类型：专业社区报告｜置信度：高）
