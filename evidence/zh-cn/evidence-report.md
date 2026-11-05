# Google Antigravity 256K 上下文压缩门禁 —— 客户端二进制静态逆向与本地数据库工程取证报告

- **语言 / Language**：[English (../evidence-report.md)](../evidence-report.md) | **简体中文**
- **档案库全局导航**：[仓库总览 (../../README.zh-CN.md)](../../README.zh-CN.md) ｜ [第一部分：事件调研总览](../../research/zh-cn/dossier.md)（[English](../../research/dossier.md)）｜ **第二部分：工程静态取证报告（本文件）** ｜ [原始 JSON 与脚本取证附件 (../raw/)](../raw/)
- **报告范围**：汇总 Google Antigravity（CLI `v1.5.0` 与桌面端 `v2.29.0`）在标称 `1,048,576`（1M）上下文的模型上强制执行 `256,000` Tokens（`0x3E800`）硬编码自动压缩的全部可独立复核的静态反汇编与本地 SQLite 数据库证据。
- **编制日期**：2026-11-04 ｜ **文件路径**：`evidence/zh-cn/evidence-report.md`
- **取证方法论**：本报告仅包含两类一手材料——（1）对官方发行二进制文件的只读静态反汇编分析与对本地会话数据库的只读读取；（2）经 GitHub CLI（`gh api`）逐条核实的公开 GitHub 记录。全程未修改任何被分析文件的字节，未运行被分析的二进制，未对任何在线服务发起异常请求。
- **原始取证材料**：上级目录 `../raw/` 内含全部 21 份 GitHub API 原始 JSON 响应、1 份 Google AI Developers Forum 主题快照（`forum_198191.json`，官方确认桌面端约 250K 压缩阈值固定的回复）、以及官方发布清单（`manifest_linux_amd64.json`）。官方 CLI 安装脚本仅以 URL 引用（`https://antigravity.google/cli/install.sh`，`7,643` 字节，SHA-256 `62966c07365423bd4dc209355060744058fb30d60f5323e2d360e39de64e5042`），不转载原文；`patch_agy.py` Gist 仅记录元数据与内容 SHA-256（`f9115f46d5949afcf6e07a5ace39dfc81f7f891a8d59a88c109ab74fe479731f`），不含脚本正文。本仓库不直接存放第三方二进制可执行文件或安装包；被分析的官方桌面端安装包 `Antigravity-x64.exe`（`v2.29.0`，SHA-256 `944b109b…11c12ba`）、内嵌的桌面端 `language_server.exe`（SHA-256 `c98e7c33…e1c3d5`，已实证与官方安装包内文件逐字节一致）及 CLI 官方发布包 `cli_linux_x64.tar.gz`（`v1.5.0`，SHA-512 与 manifest 一致）均已在第 1 章与第 9 章提供 Google 官方云存储（`storage.googleapis.com/antigravity-public`）直链与完整加密哈希，可供独立下载复核。

---

## 0. 一页结论

1. Gemini 4 Argon 模型在客户端二进制中注册的真实上下文窗口为 1,048,576 tokens（`0x100000`），与对外宣传的 1M 一致。该能力真实存在于代码中。
2. 同一套二进制中，压缩阈值的计算函数 `cortex/utils/utils.GetCheckpointConfig` 内含一条硬编码指令序列，将阈值钳制为 `min(计算值, 256,000)`（常量 `0x3E800`）。该常量在桌面端 56MB 代码段、CLI 60MB 代码段中各只出现这一处。
3. CLI 与桌面端由同一源码树（`google3/third_party/jetski`）编译，桌面端约 92,000 个、CLI 约 90,000 个函数符号携带该路径前缀；两处钳制指令前后字节逐字节一致。
4. 用户本机在 `2026-11-04 00:00:00 UTC+8`（打补丁前）生成的 237 个 Antigravity CLI 会话数据库（`~/.gemini/antigravity-cli/conversations/*.db`）中，209 个 1M 上下文 Gemini 会话、共计 **15,575 次模型调用**（含 `Gemini 3.8 Flash` `11,110` 次、`Gemini 4 Argon` `2,826` 次、`Gemini 3.7/3.6/3.5 Flash` `1,639` 次）的请求元数据中，`max_context_tokens` **100.0%（`15,575 / 15,575`）记录为 `256,000`**，并记录了 79 次 `CORTEX_STEP_TYPE_CHECKPOINT`（`step_type = 23`）压缩步骤与 52 次 `truncation_reason = 2`（`TRUNCATION_REASON_MAX_TOKEN_LIMIT`）截断事件。
5. CLI 的 `/model` 界面已内置 256K / 512K / 900K 三档上下文选择（含 1.3x / 1.8x 计费标记字符串），但被一个长度为 1 的硬编码白名单（唯一条目为字符串 `"Gemini 4 Argon"`）在两处代码中门控；桌面端对应功能为 `context-window-options` 实验键，处于服务端灰度控制之下。
6. 官方 Issue 跟踪：Issue #1245（2026-10-14 提出，22 个👍，已被 Google 成员 chandrashakherkamasani 认领，至今 open）；#878（2026-08-26 提出，22 个👍，open）；#1371 为桌面端同类请求（2026-10-30 提出，官方回复将该问题导向论坛）。截至本档案日期，Google 未发布任何面向用户的阈值设置。
7. 值得注意的反向证据：CLI 最新版 1.5.0（2026-11-04 构建）中，此前被证明无效的环境变量 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 的读取与解析逻辑已被恢复实现。这表明 Google 内部了解该问题且具备修改能力，但默认的 256K 钳制与白名单在最新版中原样保留。

---

## 1. 涉事文件清单

### 1.1 桌面端 language_server.exe

| 项目 | 值 |
|---|---|
| 来源 | 取自本机安装目录（`%LOCALAPPDATA%\Programs\antigravity\resources\bin\language_server.exe`），已核验与官方安装包内文件逐字节一致 |
| 对应官方发行 | Antigravity App v2.29.0。官方 Windows x64 安装包直链：`https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-x64/Antigravity-x64.exe`（169,295,640 字节，SHA-256 `944b109bcb377f9f94f86eeea7cfa9315d4b42d84fb9d5bad689058fc11c12ba`，可通过该官方直链直接下载校验） |
| 打包方式（已实证） | language_server.exe **随安装包本地打包**，非运行时云端下载：该安装包为 NSIS 自解压结构，内嵌 `app-64.7z`（构建时间 2026-11-03 17:36，与用户文件时间戳同刻），载荷含 `Antigravity.exe`（Electron 主程序，245,737,472 字节）、`resources/app.asar`（前端包）、`resources/bin/language_server.exe` 等 425 个文件 |
| 来源一致性（已实证） | 从官方安装包内提取的 `resources/bin/language_server.exe`（187,342,336 字节）与用户 K 盘备份 **SHA-256 一致且逐字节一致（cmp 通过）**，哈希同为 `c98e7c331da5bcc37002a550e7d625b4c636f775fa2804f73777c6a500e1c3d5`。即本档案分析的桌面端二进制即官方 v2.29.0 发行文件 |
| 文件大小 | 187,342,336 字节 |
| 格式 | PE32+ x86-64 console executable，14 个 section，ImageBase 0x140000000 |
| SHA-256 | `c98e7c331da5bcc37002a550e7d625b4c636f775fa2804f73777c6a500e1c3d5`（分析与 K 盘原件校验一致） |
| 编译工具链 | Go（Go buildinf 头位于文件偏移 0x9610c00；符号表含 go1.28 工具链痕迹） |
| 函数符号数 | 214,663（pclntab 位于文件偏移 0x44b7440） |

### 1.2 CLI antigravity（agy）1.5.0

| 项目 | 值 |
|---|---|
| 获取方式 | 官方安装脚本 `https://antigravity.google/cli/install.sh`（未执行），脚本内 manifest 地址 `https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/linux_amd64.json` |
| manifest 内容（2026-11-04 拉取） | version `1.5.0`；url `https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64/cli_linux_x64.tar.gz`；sha512 `e6031f850929210705d8c2393a5665b82b8c3b19684a6d3c556cafa31f26f77acbf164dfee96ca84a5ae7670ac0cc238a974c981c517b269194bdccff271cbd5` |
| 校验 | 下载后 sha512sum 与 manifest 完全一致 |
| 包内文件 | `antigravity`，238,604,688 字节，构建时间 2026-11-04 02:54 |
| 格式 | ELF 64-bit LSB PIE x86-64，dynamically linked，stripped，BuildID(md5/uuid) `5fb6af4f2e1d489f3ada0fd785060f48` |
| 函数符号数 | 177,405（pclntab 位于文件偏移 0x1676b70；虽 stripped，Go pclntab 函数名表完整保留） |

### 1.3 本地会话数据库（Antigravity CLI）

- 目录 `~/.gemini/antigravity-cli/conversations/`（单会话轨迹 SQLite 数据库，按 UUID `<uuid>.db` 命名）与 `~/.gemini/antigravity-cli/conversation_summaries.db`（全局会话索引与 FTS5 全文检索数据库）。
- 补丁前截止窗口：修改时间严格早于 `2026-11-04 00:00:00 UTC+8`（`2026-11-03 16:00:00 UTC`，Unix 时间戳 `< 1793721600`，时间跨度为 `2026-07-28` 至 `2026-11-03 UTC+8`）的全部 **237 个 `.db` 文件**。这 237 个数据库全部早于 11 月 4 日的任何本地补丁修改，属于未修改官方 CLI 客户端的原生运行数据（全程通过 `mode=ro` 只读连接仅解析表结构与数值型 Protobuf 元数据字段，不读取任何对话文本内容）。

---

## 2. 事实链：五层证据

### 2.1 宣传层

Gemini 4 Argon 的上下文窗口为 1,048,576 tokens。该数字同时出现在：

- Google 官方仓库 Issue #1245 标题与正文（"1M-context models"、"`1,048,576` token context window"）；
- Issue #878 标题（"unlock full 1M context window"）；
- Issue #1371 标题（"the model window is 1M"）。

### 2.2 模型层（二进制证据：能力真实存在）

两份二进制中，Argon 模型信息由 `defaultmodels.Gemini4Argon` → `defaultmodels.GeminiModelInfo` 构造，写入同一结构体字段：

- 桌面端：`mov DWORD PTR [rax+0x14],0x100000`（1,048,576）@ VA 0x141ce5015；`mov DWORD PTR [rax+0x70],0x10000`（65,536）@ VA 0x141ce501c
- CLI 1.5.0：同两条指令 @ VA 0x8540415 / 0x854041c

即模型注册的最大上下文就是 1M。`Gemini4Argon` 按思考档位传入家族号 1417/1418/1419（`0x589/0x58a/0x58b`）。

### 2.3 执行层（二进制证据：256K 钳制）

核心函数 `google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig`。反汇编（桌面端，VA 0x14176d580 起）：

```
14176d5aa:  cmp DWORD PTR [rcx+0x8],0x143      ; 家族号 323 特判
14176d5b1:  je  0x14176da2b                     ; 命中 → 走 1M 直通路径
...
14176d5dc:  cmp rdi,0x9c40                      ; 40,000
14176d5e9:  cmovg r8,rdi                        ; margin = max(模型字段, 40000)
14176d5f6:  sub rdx,r8                          ; candidate = context − margin
14176d6a2:  cmp edx,0x3e800                     ; ← 256,000 钳制（唯一一处）
14176d6a8:  mov esi,0x3e800
14176d6ad:  cmovb esi,edx                       ; threshold = min(candidate, 256000)
14176d6b0:  mov DWORD PTR [rsp+0x50],esi        ; 写入 TokenThreshold
14176d6ba:  cmp edx,0x4000                      ; 次级字段按 16,384 钳制
...
14176da77:  mov DWORD PTR [rax+0xc],0xf4240     ; 家族 323 特判路径：阈值=1,000,000
```

字节特征 `81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2`（`cmp $0x3e800,%edx; mov $0x3e800,%esi; cmovb %edx,%esi`）：

- 桌面端：整个 56MB `.text` 中出现 **1 次**（文件偏移 0x176cca2，VA 0x14176d6a2），256,000 作为立即数在整个代码段仅存在于这一个函数的这两条指令；
- CLI 1.5.0：整个 60MB `.text` 中出现 **1 次**（文件偏移/VA 0x7fe5802），前后指令序列（`mov edx,[rsp+0x4c]` … `mov [rsp+0x50],esi` … `cmp edx,0x4000`）与桌面端逐字节一致。

相邻的 `ClampCheckpointConfigForModelInfo`（桌面端 VA 0x14176dce0）只按模型真实窗口收缩阈值（`maxcontext − 0x9c40`），不含 256K 类锁。即 256,000 是整条链路中唯一的固定上限。

对 Argon 代入：threshold = min(1,048,576 − 65,536, 256,000) = 256,000。

**同源佐证**：CLI 与桌面端共有约 9 万个 `google3/third_party/jetski/` 前缀函数符号；`cortex/utils/utils.GetCheckpointConfig`、`GetCheckpointConfigForModelInfo`、`ClampCheckpointConfigForModelInfo`、`ValidateCheckpointConfigForModelInfo`、`GetDefaultCheckpointConfigForModelInfo` 在两份二进制中同名同序存在。

### 2.4 界面层（二进制证据：功能已内置但被门控）

CLI 1.5.0：

- 字符串表中 `512K`、`1.3x`、`900K`、`1.8x`、`256K` 相邻存放（文件偏移 0x5cf26c0 一带），proto 消息 `ContextWindowOption` 含 `Expensive` 字段（`(*ContextWindowOption).GetExpensive` 等访问器存在于符号表）。三档窗口选择 UI 的数据结构完整存在。
- 白名单数据链（ELF 重定位链）：切片头位于数据段 VA 0xc592be0，内容 `{ptr→0xc580ef0, len=1, cap=1}`；backing array @0xc580ef0 为单个字符串头 `{ptr→0x5dcc4d3, len=14}`；0x5dcc4d3 处字符串为 **`"Gemini 4 Argon"`**（14 字符）。两条 R_X86_64_RELATIVE 重定位（`.rela.dyn`）完整可查。
- 该白名单被两处消费：
  - `model.SelectorModel.offersContextWindow`（函数体 VA 0x9e92140；循环内先比字符串长度、再调 memequal，@0x9e922c6-0x9e922d4）——控制 `/model` 界面是否显示 Context 选项；
  - `store.(*Store).ContextWindow`（函数体 VA 0x9c49fe0；加载白名单 @0x9c4a032/0x9c4a03e；不在白名单 → `xor eax,eax; xor ebx,ebx; ret`，返回 `("", 0)` @0x9c4a0f2-0x9c4a0f7）——控制已保存的 512K/900K 选择是否生效。
- `Store.ContextWindowChoice`（直接读取配置的旁路函数）的序言 @0x9c49fa0 为 `49 3b 66 10 76 27 55 48 89 e5 48 83 ec 10 48 8b 48 10 48 85 c9`——与 LiraNuna 于 Issue #1245 公布的锚点特征串逐字节一致。

桌面端 language_server.exe：

- 不含 `offersContextWindow`、`ContextWindowChoice`、`AGY_CLI_AUTO_COMPACT_OVERRIDE` 等 CLI 独有符号（全文件搜索 0 命中）。
- 含 `model_config.ContextWindowOptions`（VA 0x141242e20）：按实验键 `context-window-options`（字符串 @0x1436d7c0a）读取配置；附近字符串 `"Ignoring unparsable %s experiment: %v"` 表明其数据来源为实验（feature flag）体系；代码逐项校验选项名并特判 `"Default"`。选项档位值不在二进制内，由服务端实验配置下发。

### 2.5 运行层（Antigravity CLI SQLite 数据库结构拆解与 15,575 次调用补丁前统计）

#### 2.5.1 Antigravity CLI 本地 SQLite 数据库结构与核心 Protobuf 字段定义

Antigravity CLI 将会话数据保存在 `~/.gemini/antigravity-cli/` 下的两级 SQLite 数据库中：

1. **全局会话索引库（`~/.gemini/antigravity-cli/conversation_summaries.db`）**：
   - `conversation_summaries` 表：存放会话级索引元数据（主键 `conversation_id`、`title`、`preview`、`step_count`、`last_modified_time`、`workspace_uris`、`status`、`source`、`project_id`、`agent_name`、`parent_conversation_id`、`nesting_depth`、`last_user_input_time`、`last_user_input_step_index`、`raw_summary` BLOB、`group_id`）。
   - `conversations_fts` 虚拟表：基于 SQLite FTS5（`tokenize = 'unicode61 remove_diacritics 2'`，`prefix = '1 2 3'`）按步骤分块索引会话标题、工作区路径与正文的全文检索表（含 `chunk_first_step`、`chunk_last_step`、`title`、`workspace`、`user_body`、`agent_body`）。
2. **单会话轨迹数据库（`~/.gemini/antigravity-cli/conversations/<uuid>.db`，共 7 张表）**：
   - `trajectory_meta`（主键 `trajectory_id`、`cascade_id`、`trajectory_type`、`source`）：记录会话轨迹标识与调用来源。
   - `steps`（主键 `idx`、`step_type` INTEGER、`status` INTEGER、`has_subtrajectory` NUMERIC、`metadata` BLOB、`error_details` BLOB、`permissions` BLOB、`task_details` BLOB、`render_info` BLOB、`step_payload` BLOB、`step_format` INTEGER）：按执行顺序记录每一步动作，含 `idx_steps_status` 与 `idx_steps_step_type` 两个索引（例如 `CORTEX_STEP_TYPE_PLANNER_RESPONSE = 15` 为模型规划回复，`CORTEX_STEP_TYPE_CHECKPOINT = 23` 为自动上下文压缩检查点）。
   - `gen_metadata`（主键 `idx`、`data` BLOB、`size` INTEGER）：按模型调用序号存放序列化的 `exa.cortex_pb.GeneratorMetadata` Protobuf 二进制记录。值得注意的是，当后台触发自动压缩步骤（`CORTEX_STEP_TYPE_CHECKPOINT = 23`）时，客户端只在 `steps` 表插入一条 `step_type = 23` 的记录，**不会在 `gen_metadata` 表写入对应的模型调用记录**，这就是压缩摘要调用本身的 Token 消耗在轨迹统计中不可见的底层原因。
   - 其余 4 张辅助表：`executor_metadata`（主键 `idx`、`data` BLOB）、`parent_references`（主键 `idx`、`data` BLOB，记录子代理/分支父节点引用）、`trajectory_metadata_blob`（主键 `id` 默认 `"main"`、`data` BLOB）与 `battle_mode_infos`（主键 `idx`、`data` BLOB）。

3. **`gen_metadata.data`（`exa.cortex_pb.GeneratorMetadata`）核心 Protobuf 字段路径**（直接从 `agy` 二进制文件偏移 `0x66298e2–0x662a950` 与 `0x65d62cd` 内嵌的 `google.protobuf.FileDescriptorProto` 描述符表中提取）：
   - `GeneratorMetadata.chat_model`（`field 1`，类型 `exa.cortex_pb.ChatModelMetadata`）与 `.step_indices`（`field 2`，`repeated uint32`，关联 `steps.idx`）：
     - `ChatModelMetadata.model`（`field 3`，`exa.codeium_common_pb.Model` 枚举值）
     - `ChatModelMetadata.usage`（`field 4`，类型 `exa.codeium_common_pb.ModelUsageStats`）：
       - `input_tokens`（`field 2`，`uint64`）：当次调用计费的未命中缓存输入 Token 数
       - `output_tokens`（`field 3`，`uint64`）：模型总输出 Token 数
       - `cache_write_tokens`（`field 4`，`uint64`）：写入提示词缓存的 Token 数
       - `cache_read_tokens`（`field 5`，`uint64`）：命中的隐式前缀缓存 Token 数（`input_tokens + cache_read_tokens` 即当次发送给模型的完整提示词上下文总量）
       - `thinking_output_tokens`（`field 9`，`uint64`）与 `response_output_tokens`（`field 10`，`uint64`）
     - `ChatModelMetadata.chat_start_metadata`（`field 9`，类型 `exa.cortex_pb.ChatStartMetadata`）：
       - `start_step_index`（`field 1`，`uint32`）：当前上下文窗口保留的起始步骤序号（未压缩前为 `0`；一旦触发检查点压缩，下一轮调用立即跳变至该检查点所在的 `steps.idx`）
       - `checkpoint_index`（`field 2`，`int32`）：当前生效的检查点序号（未发生压缩时为 `-1`）
       - `steps_covered_by_checkpoint`（`field 3`，`repeated uint32`）：被检查点摘要折叠覆盖的历史步骤序号列表
       - `created_at`（`field 4`，`google.protobuf.Timestamp`）：当次模型调用的 UTC 时间戳
       - `context_window_metadata`（`field 10`，类型 `exa.cortex_pb.ContextWindowMetadata`）：
         - `estimated_tokens_used`（`field 1`，`int32`）：Cortex 引擎内部估算并用于同 `256,000`（`0x3E800`）阈值比对的已用 Token 数
         - `truncation_reason`（`field 2`，`exa.cortex_pb.TruncationReason` 枚举：`0 = TRUNCATION_REASON_UNSPECIFIED`，`1 = TRUNCATION_REASON_PROMPT_CACHE_TTL_EXPIRED`，`2 = TRUNCATION_REASON_MAX_TOKEN_LIMIT`）
         - `token_breakdown`（`field 3`，`exa.cortex_pb.TokenBreakdown`）
         - `max_context_tokens`（`field 4`，`optional int32`）：当次请求实际执行的上下文容量上限
     - `ChatModelMetadata.retries`（`field 14`，`uint32`）、`.response_model`（`field 19`，`string`）与 `.model_display_name`（`field 21`，`string`）。

#### 2.5.2 本机补丁前 CLI 会话数据库全量核验（`< 2026-11-04 00:00:00 UTC+8`）

对修改时间早于 `2026-11-04 00:00:00 UTC+8`（`2026-11-03 16:00:00 UTC`）的全部 237 个本地 CLI `.db` 文件（其中 215 个非空会话库包含 15,871 次模型调用）的 `gen_metadata` 进行只读全量解码：

- **1M 上下文 Gemini 模型调用规模**：共计 **209 个会话、15,575 次模型调用**（其中 `Gemini 3.8 Flash` 系列 `11,110` 次、`Gemini 4 Argon` High/Medium/Low 系列 `2,826` 次、`Gemini 3.7 / 3.6 / 3.5 Flash` 系列 `1,639` 次）；
- **1M Gemini 模型 `max_context_tokens` 记录值**：**全部 209 个会话、15,575 次调用 100.0%（`15,575 / 15,575`）记录为 `256,000`**，没有任何一次调用记录为 `1,048,576`、`900,000` 或 `512,000`（其余 296 次非 1M Gemini 调用分别为旧版 `Gemini 3.1 Pro / gemini-pro-default` 的 `114` 次 `128,000`、`claude-opus-4-6-thinking` 的 `61` 次 `160,000`，以及 `claude-opus-5-5-medium` 的 `121` 次 `840,000`）；
- **单次调用上下文总量分布（`input_tokens + cache_read_tokens`）**：中位数（`p50`）为 `108,493`，`p90` 为 `211,227`，`p99` 为 `250,610`；**单次无重试调用（`retries = 0`）超过 300,000 Tokens 的次数为 0 次**（与社区开发者 `@murias002` 在 Issue `#878` 对 310 个 CLI 会话、65,613 次调用的独立统计完全吻合，后者同样测得超 300K 调用为 0 次、单次最高输入 `255,999` Tokens）；
- **压缩与截断事件统计**：共记录 **79 次 `CORTEX_STEP_TYPE_CHECKPOINT`（`step_type = 23`）压缩步骤**与 **52 次 `truncation_reason = 2`（`TRUNCATION_REASON_MAX_TOKEN_LIMIT`）截断事件**。

以下为本机 CLI 会话 `5681d8b4-f18f-4efa-be2b-35091d09b953.db` 在第 338–346 次调用（`gen_idx 338..346`）跨越 `256,000` 压缩边界时的真实记录重放——可以看到 `estimated_tokens_used` 在 `gen_idx = 343` 爬升至 `255,912`（距 `256,000` 仅差 88 Tokens），紧接着 `step 700` 触发 `CORTEX_STEP_TYPE_CHECKPOINT`（`step_type = 23`，不写入 `gen_metadata`），随后在 `gen_idx = 344`（对应 `steps 701–702`）中，`start_step_index` 从 `0` 跳变至 `700`（丢弃前 700 步原始历史），`truncation_reason` 被标记为 `2`（`MAX_TOKEN_LIMIT`），`estimated_tokens_used` 骤降至 `35,148`，且 `cache_read_tokens` 从 `267,056` 跌至 `20,399`：

| gen_idx | 关联 steps.idx | input_tokens | cache_read_tokens | 提示词合计 | estimated_tokens_used | start_step_index | truncation_reason | max_context_tokens |
|---|---|---|---|---|---|---|---|---|
| 338 | 688, 689 | 5,264 | 263,016 | 268,280 | 253,367 | 0 (`ckpt=-1`) | `0` | `256,000` |
| 340 | 692, 693 | 5,728 | 263,000 | 268,728 | 253,770 | 0 (`ckpt=-1`) | `0` | `256,000` |
| 342 | 696, 697 | 2,220 | 267,033 | 269,253 | 254,211 | 0 (`ckpt=-1`) | `0` | `256,000` |
| 343 | 698, 699 | 4,942 | 267,056 | 271,998 | **255,912** | 0 (`ckpt=-1`) | `0` | `256,000` |
| *(step 700)* | *700 (`type=23`)* | *(未记录)* | *(未记录)* | *(未记录)* | *(CHECKPOINT)* | — | — | — |
| 344 | 701, 702 | 5,962 | **20,399** | **26,361** | **35,148** | **700** (`ckpt=0`) | **`2` (`MAX_TOKEN_LIMIT`)** | `256,000` |
| 345 | 703, 704 | 3,662 | 24,476 | 28,138 | 36,719 | 700 (`ckpt=0`) | `0` | `256,000` |
| 346 | 705, 706 | 4,778 | 24,472 | 29,250 | 37,813 | 700 (`ckpt=0`) | `0` | `256,000` |

---

## 3. 桌面端 language_server.exe 静态分析详录

### 3.1 文件结构

PE section 摘要（`.text` VA 0x140001000，RawPtr 0x600，VSize 0x3570556；`.rdata` VA 0x143572000 等）。Go buildinf 头 @0x9610c00（`\xff Go buildinf:`）。pclntab @0x44b7440，nfunc=214,663。

### 3.2 256,000 常量全量扫描

对 `.text` 全段扫描立即数 256,000（`81 f8..ff` 的 cmp 与 `b8..bf` 的 mov-imm 两种编码）：**仅 GetCheckpointConfig 一个函数命中**（cmp @0x14176d6a2 + mov-imm @0x14176d6a8，即同一处钳制的两条指令）。不存在第二处 256K 预设。

### 3.3 1,000,000（0xF4240）相关位置

- `GetCheckpointConfig` @0x14176da77：家族 323 特判路径直接写阈值 1,000,000（`cmp [mem],0x143` 在全代码段仅此一处 @0x14176d5aa）；
- `defaultmodels.GetModelInfosWithOverride` @0x141ce4c9e：构造一个家族号 0x146（326）、MaxContext=1,000,000、字段+0x70=65,536 的覆盖模型；
- `geminimodels.MapModel` @0x141ce63e1：mov-imm 1,048,576（`0x100000`）。
- 323、326 两个家族对应的具体产品名无法从二进制静态确定（家族号经数据/协议下发，非代码内立即数）。

### 3.4 其余相关函数（cortex/utils/utils 包）

`GetDefaultCheckpointConfigForModel`、`GetCheckpointConfigForModelInfo`、`GetCheckpointConfig`、`ClampCheckpointConfigForModelInfo`、`ValidateCheckpointConfigForModelInfo`、`IsIgnorableCheckpointView` 等，VA 0x14176d200-0x14176e0c0 一带；另有 `prodvariants.withoutCheckpointing`（@0x24aa300）表明关闭压缩的变体在代码层存在。

### 3.5 字符串核查（对"检测/风控"类断言的证伪记录）

| 检索项 | 结果 | 说明 |
|---|---|---|
| `CLAUDECODE` / `CLAUDE_CODE` | 0 次 | 不存在环境变量检测字符串 |
| `CLAUDE`（38 次） | 均为 `CLAUDE_4_OPUS` 等模型名 | |
| `BOT_CHECK_*` | 13 次 | 上下文全部为浏览器工具的 CAPTCHA/Cloudflare 处理指令文案（如 "navigating directly to a bot-verification challenge"），与"检测第三方 AI 调用"无关 |
| `Getppid` / `isatty` / `IsTerminal` | 仅标准库符号 | 任何 Go 二进制均有 |
| `attestation` | 13 次 | 属无关的 Notary proto 描述符与 workspace 培训枚举，未发现二进制自校验机制 |
| `offersContextWindow` / `ContextWindowChoice` / `AGY_CLI_AUTO_COMPACT_OVERRIDE` | 0 次 | CLI 独有，不存在于桌面端 |
| `enable-context-usage-indicator` | 0 次 | 不在本文件（无法从本文件核实，Electron 前端未分析） |
| `ANTIGRAVITY_*` 环境变量 | 47 个不同名字 | 含 `ANTIGRAVITY_SENTRY_SAMPLE_RATE`（Sentry 遥测采样率）等，均为配置/标识用途 |
| `512K` / `256K` 命中 | 分别为 Go runtime "512KiB 最小堆"、压缩块大小字符串 | 与上下文窗口选项无关（误报排除记录） |

### 3.6 模型注册路径

`defaultmodels.Gemini4Argon`（VA 0x141ce4e00）按思考档位（参数 1/2/3）传入家族号 `0x589/0x58a/0x58b` 并调用 `GeminiModelInfo`（0x141ce4ea0）；"Gemini 4 Argon" 字符串在桌面端共 4 处，代码引用 5 处，全部位于 `defaultmodels.Gemini4Argon` 与 `geminimodels.Catalog` 两个函数内（模型注册，非白名单）。

---

## 4. CLI 1.5.0 静态分析详录

### 4.1 钳制

字节特征唯一出现 @0x7fe5802；`cmp [mem],0x143` 全段唯一 @0x7fe570a；`mov DWORD PTR [rax+0xc],0xf4240`（1M 阈值直通）位于 `GetCheckpointConfigForModelInfo` 内 @0x7fe5bd7（与桌面端不同函数位置、同一行为）。`GeminiModelInfo` 常量与桌面端一致（0x8540415/0x854041c）。

### 4.2 白名单（完整证据链）

1. `.rela.dyn`（文件偏移 0x9fa8，979,494 条）中：`r_offset=0xc592be0` → addend `0xc580ef0`；`r_offset=0xc580ef0` → addend `0x5dcc4d3`。
2. 数据段 0xc592be0 处 24 字节为 `00…00 | 01 00…00 | 01 00…00`，即切片头 `{ptr=重定位填充, len=1, cap=1}`。
3. 0xc580ef0 处 16 字节为 `{ptr=重定位填充, len=14}`，即唯一元素——一个 14 字符的字符串头。
4. 0x5dcc4d3 处字符串 = `Gemini 4 Argon`。
5. 消费方一：`SelectorModel.offersContextWindow`（函数体 @0x9e92140），读取该切片 @0x9e9219e-0x9e921ad，循环比较 @0x9e922ba-0x9e922e0。
6. 消费方二：`store.(*Store).ContextWindow`（函数体 @0x9c49fe0），读取 @0x9c4a032/0x9c4a03e，不在白名单即返回空 @0x9c4a0f2。

### 4.3 环境变量 AGY_CLI_AUTO_COMPACT_OVERRIDE 的演变（三段独立证据）

1. **1.3.5/1.4.x 时期无效**：Issue #878 中 @Infinnerty（2026-10-15）实测 `settings.json` 的 `compactionThreshold` / `checkpointIntervalTokens` 设为 30000 无效；Issue #1245 中 @LiraNuna（2026-10-22）反汇编证明 1.4.x 里 `os.Getenv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` 被编译为空字符串（`xor %eax,%eax; xor %ebx,%ebx; call strings.TrimSpace`）。
2. **1.5.0 已恢复**：本次分析在 1.5.0 中实测 `store.AutoCompactOverride`（VA 0x9c49d00）为 `lea rax,[rip+→"AGY_CLI_AUTO_COMPACT_OVERRIDE"]`（字符串 @0x5ec74c5，长度 29）+ `mov ebx,0x1d` + `call os.LookupEnv`（0x6b21260）+ `call strings.TrimSpace`（0x6b50780）。
3. **解析与消费**：紧随其后的解析器（VA 0x9c49d40）支持两种格式——`<百分比>%`（ParseFloat、上限校验 100、经魔数 0xa3d70a3d70a3d70b 除以 100）与绝对 token 数；`Store.ContextWindow` 在无已选窗口时调用该解析器（call @0x9c4a0fa）。该环境变量无官方文档。
4. **调用方**：`AutoCompactOverride` 的直接调用者共 3 处——其解析器自身、`model.overrideNote` 两处（UI 提示用途）；`Store.ContextWindow` 经解析器间接消费。
5. **1.5.1 中的限制**（官方二进制，SHA-256 `72738cf0…26669`）：对解析器（VA 0x9c348e0）的两次调用之前都是 `xor %eax,%eax`（调用点 @0x9c34c9a 与 @0x9e875aa），即传入基数 0，因此 `<百分比>%` 写法被拒绝，只有绝对 token 数生效。在 `Store.ContextWindow` 中，只有模型白名单通过且 `settings.json` 未保存档位时才会读取该变量。大于 256,000 的值能否越过 `GetCheckpointConfig` 的钳制未做实测。

### 4.4 CLI 其余核查与 1.5.1 追补核验（2026-11-05）

- `CLAUDECODE`：0 次。
- `context-window-options` 实验键：存在（1 次）。
- LiraNuna gist 补丁脚本 `patch_agy.py` 的 ELF 解析入口检查 `\x7fELF`，不适用于 PE 格式的桌面端二进制（机制性事实，非行为验证）。
- **CLI `1.5.1` 追补核验（`2026-11-05 ~07:34 UTC`，`240,857,488` 字节，BuildID `696031ae34e5e5e26a9d7c436e85317f`，SHA-256 `72738cf0...26669`）**：`CHANGELOG.md` 的全部 11 条更新均为常规维护修复（`agy acp` 会话修复、Windows CRLF/编码/stdio 修复、插件 OAuth 凭据防泄露与 UI 状态显示调整），未涉及任何 256K 上下文限制；`GetCheckpointConfig` 中的 14 字节 `0x3E800`（`256,000`）钳制指令在文件偏移 `0x7fb0fe2` 处逐字节原样保留（`1.5.0` 为 `0x7fe5802`），白名单函数（`offersContextWindow`、`store.ContextWindow`）与已恢复的 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 读取指令亦无任何变化。

---

## 5. GitHub 公开佐证完整档案

以下全部数据经 `gh api` 于 2026-11-04 实时核实，原始 JSON 见 `raw/` 目录。

### 5.1 仓库

- `google-antigravity/antigravity-cli`（https://github.com/google-antigravity/antigravity-cli）
- 创建于 2026-05-13，描述 "Antigravity CLI brings the reasoning, execution, and orchestration capabilities of Antigravity agent harness directly into your terminal."
- 2,714 stars；841 open issues；无 License 文件。

### 5.2 Issue #1245（主 Issue）

- 标题：`[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache`
- URL：https://github.com/google-antigravity/antigravity-cli/issues/1245
- 提出：@Burntgogi，2026-10-14T19:34:38Z；状态 open；标签 `comp:agent capabilities`、`feedback`、`1.3.5`；认领人 **@chandrashakherkamasani**（2026-10-15 自行认领）。
- 正文要点（agy 1.3.5 + Gemini 4 Argon，94 轮 / 264 步 50 分钟自主运行，10.38M 总 prompt tokens）：
  - Checkpoint 1 @ Step 79：压缩前 276,110 tokens（263,524 cached + 12,586 input）→ 压缩后 17,336，cache_read 归零；
  - Mid-run prefix miss @ Step 193：243,064 全量未命中缓存；
  - Checkpoint 2 @ Step 200：压缩前 249,867 → 压缩后 15,363；
  - 报告的后果：压缩后模型引用不存在的目录条目 ID、把 1000..2000 量纲误写为 0..10（200 倍偏差）、同一 localStorage 键在两个模块中命名不一致。
- 全部 7 条评论（按时间序，均经 gh 核实）：

| 时间 (UTC) | 作者 | 内容要点 | 👍 |
|---|---|---|---|
| 2026-10-15T18:12 | @NuCl34R | Linux 实测：多次运行压缩分别发生在 241,901 / 278,129 / 249,445 / 269,597 / 274,270 等；`AGY_CLI_AUTO_COMPACT_OVERRIDE=900000/100000` 均无效果；二进制内存在该环境变量字符串。抽测 40 个隐藏代码 recall 40/40（摘要碰巧保留），但 64 事实报告 21 处错误、47 事实报告 13 处错误，其中压缩摘要把 9-10 位 SHA 前缀"补全"成不存在的完整 SHA | 6 |
| 2026-10-15T19:35 | @Circumsized | 会话库实测 `max_context_tokens = 256000`、`truncation_reason = 2`（`~/.gemini/antigravity-cli/conversations/<id>.db`）；区分"压缩"与"预发送截断"两种历史丢失机制；指出 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 无效说明上限读取源用户不可达 | 3 |
| 2026-10-19T04:22 | @Emily2040 | 15 个 headless stream-json 日志（1.4.0-1.4.2，1,279 次调用）：17 次调用 input>100k 且 cache_read=0；22 次 checkpoint 中位耗时 70 秒（35-224s，一次 602s），checkpoint 不报告 usage，其后 45-70k 未命中调用 | 1 |
| 2026-10-19T22:48 | @Emily2040 | 补充：22 次 checkpoint 触发前上下文全部在 235k-268k 之间，与 256,000 常量一致；同日公开帖描述将常量改为 980,000 后效果改善，其不建议该路线但指出"如此重要的决策被隐藏为常量"是根因 | 4 |
| 2026-10-22T16:37 | @LiraNuna | 反汇编结论：`/model` 已内置 Context 选择（`Default (256K) | 256K | [512K] | 900K`），被 `SelectorModel.offersContextWindow` 的硬编码白名单（仅 `"Gemini 4 Argon"`）门控；`AGY_CLI_AUTO_COMPACT_OVERRIDE` 在发行二进制中被编译为空串；`Store.ContextWindow()` 回退 256K 预设 | 3 |
| 2026-10-23T01:37 | @LiraNuna | 跟进：解锁 UI 后仍压缩在 ~250K 的两个额外门控——`Store.ContextWindow()` 内第二个同样的白名单；`cortex/utils/utils.GetCheckpointConfig` 的 0x3E800 钳制（含 40,000 margin）。发布自动补丁脚本至 Gist | 5 |
| 2026-11-02T14:43 | @AsabaLazy | 发布修复此问题的 AUR 打包 `antigravity-cli-fix-bin`（https://github.com/AsabaLazy/aur-ASLZ） | 0 |

- 点赞（👍等全部表情）用户完整名单（28 个 reaction 事件、25 个去重用户）：@scarletkc、@NuCl34R、@asyrafng、@chunyoupeng、@bluekyu、@sundanc、@fikrisaman、@S-Elisei、@LiraNuna、@heswithme（×2）、@x-spy、@carterfleet、@nurimator、@vegerot、@cqjjjzr、@NekoApocalypse、@Raychanan（×2）、@msackman、@md-1985、@uYanJX（×2）、@xuyang2、@jkooooo99、@Antisophy、@nullgit、@Sharper31。

### 5.3 Issue #878（最早的功能请求）

- 标题：`[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window`
- URL：https://github.com/google-antigravity/antigravity-cli/issues/878
- 提出：@schalatap，2026-08-26T13:36:35Z；open；标签 `feature request`、`comp:conversations`；22 👍。
- 全部 5 条评论（gh 核实）：

| 时间 (UTC) | 作者 | 内容要点 | 👍 |
|---|---|---|---|
| 2026-09-03T04:03 | @bash99 | 请求 `contextWindowLimit` 与 `historyCompactionThreshold` 配置项 | 1 |
| 2026-10-05T00:51 | @murias002（Eddy Murias） | 310 个会话、65,613 次调用实测：单次调用 input p50=136,487、p90=229,000、**max=255,999**；>300k 的调用 0 次；276 次压缩事件，压缩前中位 255,070（p10 247,360、p90 255,874），压缩后中位 45,304；settings.json、CLI flag、环境变量均无法改变 | 1 |
| 2026-10-05T09:17 | @Corsagee | 请求 `settings.json` 暴露 `compactionThreshold`；其二进制检查发现 proto schema 已定义 `AntigravityCompactionConfig.token_threshold`，但被硬编码的 `ContextWindow − MaxOutputTokens` 回退取代 | 4 |
| 2026-10-15T07:58 | @Infinnerty | agy 1.3.5 headless 实测：16 分钟内压缩 5 次（87,386 / 99,639 / 91,934 / 84,678 / 115,605 tokens 处触发，压后 25k-29k）；`compactionThreshold`/`checkpointIntervalTokens` 设 30000 无效；二次压缩后摘要开始引用仓库中不存在的函数、报告引用超出文件末尾的行号 | 2 |
| 2026-10-31T15:49 | @jlukic | 指出该差异阻碍长规划任务；观察到多次压缩后摘要只压缩已被截断的上下文（"segment 3 has no access…"） | 0 |

- 点赞用户完整名单（22 个）：@angelo0098、@asror-z、@aur9ra、@bash99、@ccl-123、@dyokism、@gengel、@ghisguth、@HarryPi、@hernancerm、@Infinnerty、@jlukic、@jomp16、@mkarimdev、@nisel11、@NuCl34R、@RodNSS、@rrhuffy、@rs3vans、@schalatap、@vegerot、@z1r0-。

### 5.4 Issue #1371（桌面端同类请求）

- 标题：`[Feature Request] User-configurable auto-compaction threshold in the desktop app (~256K hardcoded while the model window is 1M)`
- URL：https://github.com/google-antigravity/antigravity-cli/issues/1371
- 提出：@floveey，2026-10-30T07:28:59Z；open；桌面端会话约 256K（input+cache read）即压缩；点赞用户：@Sharper31。
- 评论 2 条（gh 核实）：
  - 2026-10-30T11:35 @manirajc（官方回复）："this repository is exclusively for the Antigravity CLI. For feature requests related to Antigravity 2.0 or IDE please reach out to [Antigravity Forum](https://discuss.ai.google.dev/c/antigravity/64)"——即桌面端请求被引导至 Google 论坛。
  - 2026-11-01T14:05 @LiraNuna：指向 #1245 及其 agy 补丁。

### 5.5 其余相关 Issue（均经 gh 核实）

| # | 标题（缩写） | 作者 | 提出时间 | 状态 | 👍 | 评论者 |
|---|---|---|---|---|---|---|
| 743 | Add max_context_tokens setting to control per-turn token budget | @thealgebraist | 2026-08-03 | open | 2 | （无）点赞：@LogIN-、@persano |
| 1291 | Five measured places the CLI spends time and tokens the work does not need（含 70 秒压缩、未缓存全量重发等五项实测） | @Emily2040 | 2026-10-19 | open | 2 | @Emily2040（2026-10-25）；点赞：@LordOfCorners、@vegerot |
| 1316 | Subagent context & compaction controls for headless orchestrator backends（--no-auto-compact、--compact-threshold 等） | @aeronmiles | — | open | 1 | @sleegme（2026-10-29）；点赞：@vegerot |
| 1372 | Active warning and compaction prompt before sessions degrade | @zhangr930103 | — | open | 0 | @LiraNuna（2026-11-01） |
| 1367 | /goal autonomous loop lacks token circuit breaker & mid-flight compaction, burning 37M tokens in under an hour | @thhart | — | open | 0 | （无） |
| 919 | Implement context compaction to prevent INVALID_ARGUMENT 400 trajectory loops | @ozl | — | open | 0 | （无） |
| 1072 | Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost | @flotoz | — | open | 0 | @Emily2040（2026-10-19） |

### 5.6 LiraNuna 的 Gist（patch_agy.py）

- URL：https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26
- 描述："Patch agy to unlock 512K/900K context window selection and fix early compaction"
- 所有者：@LiraNuna；创建 2026-10-23T01:37:08Z；最后更新 2026-10-23T23:12:51Z；共 2 个修订版本（rev1 → rev2，rev2 改为版本无关的 pclntab 符号动态解析，174 处新增/86 处删除）。
- 文件：`patch_agy.py`（11,290 字节）。
- 脚本声明的四项修改（与本次二进制分析相互印证）：① `SelectorModel.offersContextWindow` 强制返回 true；② `store.AutoCompactOverride` 保持干净；③ `Store.ContextWindow` 重定向至 `Store.ContextWindowChoice`；④ 移除 `GetCheckpointConfig` 的 0x3E800 钳制。
- 本档案仅将该脚本作为"社区已定位到同一批函数与常量"的公开佐证引用，未执行该脚本。

### 5.7 参与者完整名单汇总（去重）

**Issue 提出者（10 人）**：@Burntgogi、@schalatap、@thealgebraist、@Emily2040、@floveey、@aeronmiles、@zhangr930103、@thhart、@ozl、@flotoz

**评论者（12 人，去重）**：@NuCl34R、@Circumsized、@Emily2040（#1245/#1291/#1072 三处留言）、@LiraNuna（#1245×2/#1371/#1372 四处留言）、@AsabaLazy、@bash99、@murias002、@Corsagee、@Infinnerty、@jlukic、@sleegme、@manirajc（官方）

**点赞/表情参与者（48 人，跨 issue 去重合并）**：@scarletkc、@NuCl34R、@asyrafng、@chunyoupeng、@bluekyu、@sundanc、@fikrisaman、@S-Elisei、@LiraNuna、@heswithme、@x-spy、@carterfleet、@nurimator、@vegerot、@cqjjjzr、@NekoApocalypse、@Raychanan、@msackman、@md-1985、@uYanJX、@xuyang2、@jkooooo99、@Antisophy、@nullgit、@Sharper31、@angelo0098、@asror-z、@aur9ra、@bash99、@ccl-123、@dyokism、@gengel、@ghisguth、@HarryPi、@hernancerm、@Infinnerty、@jlukic、@jomp16、@mkarimdev、@nisel11、@RodNSS、@rrhuffy、@rs3vans、@schalatap、@z1r0-、@LogIN-、@persano、@LordOfCorners

**官方人员**：@chandrashakherkamasani（Issue #1245 认领人）、@manirajc（#1371 官方回复）

---

## 6. 版本行为演变时间线

| 时间 | 事件 | 证据 |
|---|---|---|
| 2026-08-03 | #743 请求 max_context_tokens 设置 | gh |
| 2026-08-26 | #878 请求解锁完整 1M 窗口 | gh |
| 2026-08-30~10-04 | @murias002 的 310 会话实测期（max input 255,999） | gh（#878 评论） |
| 2026-10-14 | #1245 提出；10-15 被认领 | gh |
| 2026-10-19 | @Emily2040 两次量化补充（22 次 checkpoint 全部在 235k-268k 触发） | gh（#1245/#1291） |
| 2026-10-22~23 | @LiraNuna 发布反汇编分析与 Gist 补丁 | gh（#1245 + gist 元数据） |
| 2026-10-30 | #1371 桌面端请求提出，次日被官方引导至论坛 | gh |
| 2026-11-02 | @AsabaLazy 发布 AUR 修复包 | gh（#1245） |
| 2026-11-04 | CLI 1.5.0 构建（02:54）并发布：`AGY_CLI_AUTO_COMPACT_OVERRIDE` 的 Getenv+解析已恢复实现；256,000 钳制、`"Gemini 4 Argon"` 长度为 1 的白名单、40,000 margin 原样保留 | 本次静态分析 |

---

## 7. 未验证与无法核实事项（诚实声明）

以下内容在本档案中**不作为事实引用**：

1. **服务器端能力**：512K/900K 档位在服务端的实际支持与 1.3x/1.8x 计费执行。本地证据仅为客户端 UI 字符串（`512K`/`1.3x`/`900K`/`1.8x` 相邻存放）与 proto 字段（`ContextWindowOption.Expensive`、`BidiGenerateContentSetup.context_window_compression{trigger_tokens, sliding_window}` 的存在）。服务端行为未验证。
2. **封号与风控断言**：两个二进制中均未发现 `CLAUDECODE`/`CLAUDE_CODE` 环境变量检测、父进程检查或非 TTY 检测的证据（`BOT_CHECK_*` 为浏览器 CAPTCHA 处理文案；`Getppid`/`isatty` 为标准库符号）。此前对话中"因调用方式被封号"的说法（涉及论坛用户 Man_A、Dai_L_u）未在本地证据或 GitHub 中核实。
3. **桌面端 Electron 前端（app.asar）**：未分析。`enable-context-usage-indicator`、ContextWindowOptions 三档在前端的具体呈现、"下周 v2.30 放开"的时间表，均无法从 language_server.exe 核实。
4. **"Lberondroth 的 v2.28.1→v2.29.0 拆包比对"**：GitHub 用户 Lberondroth（Louis Berndroth）存在（账号创建于 2021-03-26），但公开 gist 数为 0，未找到可核实的分析报告。该来源的说法不在本档案事实部分引用。
5. **Reddit 用户 u/z1r0_**：GitHub 无此用户，Reddit 内容未核实（#878 点赞名单中有 @z1r0-，为另一用户名）。
6. **补丁效果**：本档案未执行任何补丁，未测量修改后的行为。社区报告（#1245 中 @Emily2040 提及的公开帖、@AsabaLazy 的 AUR 包）仅作为社区反应记录。
7. **262,144（256×1024）预设的准确落点**：Issue #1245 中 LiraNuna 描述 1.3.5 版 `Store.ContextWindow()` 回退预设为 262,144；1.5.0 中该函数结构已变化（选项表遍历 + 环境变量解析），本次分析未逐条定位新版本中该回退值的确切指令。这不影响主结论——有效上限由 `GetCheckpointConfig` 的 256,000 钳制决定，该点已由本机补丁前 CLI 运行数据（209 个会话、15,575 次 1M Gemini 调用 100% 记录 `max_context_tokens=256,000`）与社区实测（Issue #878 `@murias002`：310 个会话、65,613 次调用 max input 255,999、压缩前中位 255,070）双重印证。

---

## 8. 复现方法

环境：WSL2 Ubuntu（x86_64）；工具：python3（内置 struct/sqlite3）、objdump (GNU Binutils 2.42)、`gh` (GitHub CLI，已登录账号 magiccodelab)。

1. 校验桌面端副本：`sha256sum` 与原件一致（`c98e7c33…e1c3d5`）。
2. PE/ELF 结构与 pclntab 解析：python3 struct 按节表读取；Go pclntab（magic `f1 ff ff ff`，ptrSize=8）解析函数名表。
3. 字节特征检索：`data.find(bytes.fromhex('81fa00e80300be00e803000f42f2'))` 全文件扫描。
4. 反汇编：`objdump -d -M intel --start-address=<VA> --stop-address=<VA> <binary>`。
5. 白名单重定位链：解析 `.rela.dyn`（文件偏移 0x9fa8）中 r_offset=0xc592be0 与 0xc580ef0 两条 R_X86_64_RELATIVE，读取对应数据段字符串头与目标字符串。
6. 会话库解码：`sqlite3 "file:<db>?mode=ro"` 只读打开；proto 定义（GenMetadata/ChatModelMetadata/ModelUsageStats/ChatStartMetadata/ContextWindowMetadata 及 CORTEX_STEP_TYPE 枚举值）从 language_server.exe 内嵌 protobuf 描述符字符串区提取；通用 protobuf 解码器手工验证字段路径（见 2.5 节）。
7. CLI 获取：`curl -fsSL https://antigravity.google/cli/install.sh`（仅保存脚本未执行）→ `curl` manifest → `curl` tarball → `sha512sum` 校验一致 → `tar -xzf` 解包。全程未运行二进制。
8. GitHub 数据：`gh api repos/google-antigravity/antigravity-cli/issues/{1245,878,743,1291,1316,1371,1372,1367,919,1072}`、`.../issues/<n>/comments`、`.../issues/<n>/reactions`、`search/issues?q=repo:...+compaction+in:title`、`gists/4c058aed...`。原始响应存于 `../raw/`。

---

## 9. 引用链接总表

- Issue #1245：https://github.com/google-antigravity/antigravity-cli/issues/1245
- Issue #878：https://github.com/google-antigravity/antigravity-cli/issues/878
- Issue #743：https://github.com/google-antigravity/antigravity-cli/issues/743
- Issue #1291：https://github.com/google-antigravity/antigravity-cli/issues/1291
- Issue #1371：https://github.com/google-antigravity/antigravity-cli/issues/1371
- Issue #1316：https://github.com/google-antigravity/antigravity-cli/issues/1316
- Issue #1372：https://github.com/google-antigravity/antigravity-cli/issues/1372
- Issue #1367：https://github.com/google-antigravity/antigravity-cli/issues/1367
- Issue #919：https://github.com/google-antigravity/antigravity-cli/issues/919
- Issue #1072：https://github.com/google-antigravity/antigravity-cli/issues/1072
- Gist patch_agy.py：https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26
- AUR 修复包：https://github.com/AsabaLazy/aur-ASLZ
- 官方 CLI 安装脚本：https://antigravity.google/cli/install.sh
- 官方下载页（桌面端/CLI/IDE 全部入口）：https://antigravity.google/download
- 桌面端 Antigravity App v2.29.0 官方直链（GCS bucket `antigravity-public/antigravity-hub`）：
  - Windows x64：https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-x64/Antigravity-x64.exe
  - Windows ARM64：https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-arm/Antigravity-arm64.exe
  - macOS Apple Silicon：https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/darwin-arm/Antigravity.dmg
  - macOS Intel：https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/darwin-x64/Antigravity.dmg
  - Linux x64：https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/linux-x64/Antigravity.tar.gz
  - Linux ARM64：https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/linux-arm/Antigravity.tar.gz
- CLI 1.5.0 全平台官方直链（GCS bucket `antigravity-public/antigravity-cli`）：
  - Windows x64（.exe）：https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/windows-x64/cli_windows_x64.exe
  - macOS x64：https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/darwin-x64/cli_mac_x64.tar.gz
  - macOS ARM64：https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/darwin-arm/cli_mac_arm64.tar.gz
  - Linux x64：https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64/cli_linux_x64.tar.gz
  - Linux x64 musl：https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64-musl/cli_linux_x64_musl.tar.gz
  - Linux ARM64：https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-arm/cli_linux_arm64.tar.gz
- CLI manifest（全平台版本与 SHA-512 权威来源）：https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/{platform}.json（platform 取值如 linux_amd64、windows_amd64、darwin_arm64 等）
- Google 官方论坛 Antigravity 板块（#1371 官方回复所指向）：https://discuss.ai.google.dev/c/antigravity/64
