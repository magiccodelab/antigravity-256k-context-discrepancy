# 分册二：1M 上下文窗口差异、二进制三道硬编码门禁与 v2.29.0 / CLI 1.5.0–1.5.1 逆向拆包

- **语言 / Language**：[English (../02-256k-hardcoded-gates-and-binary-teardown.md)](../02-256k-hardcoded-gates-and-binary-teardown.md) | **简体中文**
- **档案库全局导航**：[仓库总览](../../README.zh-CN.md) ｜ [调研总览 (dossier.md)](./dossier.md) ｜ [全量来源档案 (sources.md)](./sources.md) ｜ [静态逆向取证报告](../../evidence/zh-cn/evidence-report.md)
- **专题分册导航**：[01-事件背景与时间线](./01-incident-overview-and-timeline.md) ｜ **02-256K硬编码与逆向拆包** ｜ [03-agy-p政策反转与封号](./03-agy-p-headless-policy-reversal-and-bans.md) ｜ [04-社区与官方应对](./04-community-and-official-responses.md) ｜ [05-原文引用与记录汇总](./05-verbatim-quotes-and-evidence-vault.md)
- **研究 ID (UUID)**：`df866cf1-da40-4b82-8084-34c6b0729544` ｜ **调研时间**：2026-11-04（2026-11-05 更新）｜ **置信度校准**：已核验 GitHub Issue `#1245`/`#878`/`#1291`、SQLite 会话库元数据及 `agy` v1.3.5–v1.5.1 ELF 反汇编

---

## 1. 标称 1M（1,048,576）上下文与生产环境强制 ~256K 压缩的实测数据

### 1.1 官方标称值与 `/context` 面板显示
在 Google Antigravity 官方文档、模型发布公告（Changelog）[^12]、模型卡片以及 `agy` CLI 的 `/context` 交互式面板中，旗舰推理模型 `Gemini 4 Argon` 与主力模型 `Gemini 3.8 Flash` 的上下文窗口容量均明确显示为 **`1,048,576` Tokens（1.0M）**[^1][^3][^12][^44]。当会话处于 22 万 Tokens 左右时，CLI `/context` 面板仍向用户显示占用率仅为 `21.2%`（`222.4k/1.0M tokens`），并提示剩余可用空间高达 `Free space: 826.2k (78.8%)`[^44][^51]。（置信度：高）

### 1.2 五组独立大规模生产会话统计数据
尽管界面显示拥有超过 75% 的空闲上下文空间，五组在 Linux 与 Windows 生产环境下独立采集的会话遥测数据一致表明，所有会话均在 **`~240K–276K` Tokens**（围绕 `256,000` 常数）处被强制截断或压缩：

1. **Eddy Murias（`@murias002`）对 310 个会话、65,613 次调用的全量数据库统计（Issue `#878`）**[^3]：
   - **样本范围**：2026 年 8 月 30 日至 10 月 4 日期间，Linux 环境下 `~/.gemini/antigravity-cli/conversations/*.db` 中全部 `gen_metadata` 记录，共计 **310 个独立会话、65,613 次模型调用**（其中 86.6% 运行于 `Gemini 3.8 Flash`）[^3]。
   - **输入 Token 分布**：中位数（`p50`）为 `136,487` Tokens，`p90` 为 `229,000` Tokens，**历史最高单次输入停留在 `255,999` Tokens，超过 `300,000` 输入 Tokens 的模型调用次数为 `0`**[^3]。
   - **压缩触发水位**：在全部 **276 次压缩事件**（Token 骤降超过 40%）中，触发压缩前一步的上下文大小中位数为 **`255,070` Tokens**（`p10` 为 `247,360`，`p90` 为 `255,874`），压缩后上下文中位数跌至 **`45,304` Tokens**[^3]。
2. **`@Burntgogi` 对 `Gemini 4 Argon` 94 轮连续重构会话的轨迹测量（Issue `#1245`）**[^1]：
   - **样本范围**：Windows 10 x64 下 `agy 1.3.5` 连续 50 分钟的 TypeScript 多阶段重构会话（94 个 `PLANNER_RESPONSE` 轮次、264 个步骤，累计 `10.38M` 总提示词 Tokens、`9.43M` 缓存读取 Tokens）[^1]：

| 事件节点 | 步骤 / 轮次 | 压缩前提示词 Tokens（缓存 + 新增输入） | 压缩后提示词 Tokens | 压缩后 `cache_read_tokens` | 触发原因与影响 |
|---|---|---:|---:|---:|---|
| **Checkpoint 1** | Step 79 (Turn 21) | `276,110` (`263,524` cached + `12,586` in) | `17,336` | `0` | 跨越 ~256K 阈值触发首次强制压缩，缓存清零[^1] |
| **Mid-run Prefix Miss** | Step 193 (Turn 67) | `239,857` (`237,239` cached + `2,618` in) | `243,064` | `0` | 距 Step 191 仅 25 秒发生全量前缀缓存失效（243K 全额重算）[^1] |
| **Checkpoint 2** | Step 200 (Turn 70) | `249,867` (`247,369` cached + `2,498` in) | `15,363` | `0` | 抵达 ~250K 触发第二次强制压缩，93.8% 上下文被抹除[^1] |

3. **`@NuCl34R` 对环境变量覆盖无效性的对照实验（Issue `#1245` Comment #1）**[^1]：
   - 在 Linux x64 `agy 1.3.5`（`gemini-4-argon`）下，无论不设环境变量（在 `269,597` 与 `274,270` 处压缩）还是显式设置 `AGY_CLI_AUTO_COMPACT_OVERRIDE=900000`（依然在 `272,784` 与 `279,687` 处压缩），压缩触发点毫无变化[^1]。
4. **`@Emily2040` 对 15 份无头运行日志、1,279 次 `Gemini 4 Argon High` 调用的精确测量（Issue `#1245` Comments #3–#4 & Issue `#1291`）**[^1][^4]：
   - 在 `agy 1.4.0–1.4.2` 的 1,279 次模型调用中，共触发 **22 次 `checkpoint` 自动压缩步骤**，每一次触发前一步的上下文（`input + cache_read`）均严格落在 **`235K–268K` Tokens** 之间，无一例超过该区间[^1][^4]。
5. **本机补丁前 Antigravity CLI 会话数据库 209 个会话、15,575 次 1M Gemini 调用核验（`< 2026-11-04 00:00:00 UTC+8`，见[取证报告 §2.5](../../evidence/zh-cn/evidence-report.md)）**：
   - **样本范围**：本机在 `2026-11-04 00:00:00 UTC+8`（打补丁前）生成的全部 237 个 CLI SQLite 数据库（`~/.gemini/antigravity-cli/conversations/*.db`）中，包含 209 个 1M 上下文 Gemini 会话、共计 **15,575 次模型调用**（`Gemini 3.8 Flash` `11,110` 次、`Gemini 4 Argon` `2,826` 次、`Gemini 3.7/3.6/3.5 Flash` `1,639` 次）。
   - **统计结果**：**`100.0%` 的调用（`15,575 / 15,575`）记录 `max_context_tokens = 256,000`**，**单次无重试调用（`retries = 0`）超过 `300,000` 提示词 Tokens 的次数为 `0` 次**（`p50 = 108,493`、`p90 = 211,227`、`p99 = 250,610`），并记录了 **79 次 `CORTEX_STEP_TYPE_CHECKPOINT`（`step_type = 23`）压缩步骤**与 **52 次 `truncation_reason = 2`（`TRUNCATION_REASON_MAX_TOKEN_LIMIT`）截断事件**。

---

## 2. 二进制底层的“三道硬编码门禁”与环境变量演变（ELF & 汇编级分析）

通过对 `agy` 客户端编译产物（`v1.3.5` 至 `v1.5.0` 及 `v1.5.1` 的 Go ELF 64 位二进制文件）及桌面端 `v2.27.0–v2.29.0` ASAR/二进制包的逆向反编译，社区开发者（`@LiraNuna`、`@Circumsized`、`u/Lberondroth`）及本次调研对官方原版 `agy` `v1.5.0`（BuildID `5fb6af4f2e1d489f3ada0fd785060f48`）与 `agy` `v1.5.1`（`2026-11-05`，BuildID `696031ae34e5e5e26a9d7c436e85317f`）的直接反汇编核验，完整还原了 Google 限制上下文窗口的四层底层机制：

### 2.1 前置证据：Antigravity CLI SQLite 数据库结构、`max_context_tokens = 256000` 与环境变量 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 的两阶段演变
1. **Antigravity CLI SQLite 表结构与 `ContextWindowMetadata` 实证（`@Circumsized` 及本机 15,575 次调用核验）**[^1]：
   Antigravity CLI 将每个会话独立保存在 `~/.gemini/antigravity-cli/conversations/<uuid>.db` 中（由全局索引库 `conversation_summaries.db` 及其 FTS5 全文检索虚拟表 `conversations_fts` 统一索引）。每个会话 `.db` 内含 7 张表：`trajectory_meta`、`steps`（按序记录动作步骤，其中 `step_type = 15` 为 `PLANNER_RESPONSE`，`step_type = 23` 为 `CORTEX_STEP_TYPE_CHECKPOINT`）、`gen_metadata`（按模型调用存放序列化的 `exa.cortex_pb.GeneratorMetadata` Protobuf 二进制块）、`executor_metadata`、`parent_references`、`trajectory_metadata_blob` 与 `battle_mode_infos`。
   在 `gen_metadata.data` 内部，其 Protobuf 字段层级为：
   - `GeneratorMetadata.chat_model`（`field 1`，`ChatModelMetadata`）→ `.usage`（`field 4`，`ModelUsageStats`：`field 2 = input_tokens`、`field 3 = output_tokens`、`field 4 = cache_write_tokens`、`field 5 = cache_read_tokens`、`field 9 = thinking_output_tokens`）
   - `GeneratorMetadata.chat_model`（`field 1`）→ `.chat_start_metadata`（`field 9`，`ChatStartMetadata`：`field 1 = start_step_index`、`field 2 = checkpoint_index`、`field 4 = created_at`）→ `.context_window_metadata`（`field 10`，`ContextWindowMetadata`）：
     - `field 1`：`estimated_tokens_used`（`int32`，Cortex 内部估算已用 Token 数）
     - `field 2`：`truncation_reason`（`TruncationReason` 枚举：`0 = UNSPECIFIED`、`1 = PROMPT_CACHE_TTL_EXPIRED`、`2 = MAX_TOKEN_LIMIT`）
     - `field 3`：`token_breakdown`（`TokenBreakdown`）
     - `field 4`：`max_context_tokens`（`optional int32`）
   无论在 `@Circumsized` 于 Issue `#1245` 公布的数据库记录，还是本次对本机 209 个补丁前 CLI 会话、共 15,575 次 1M Gemini 调用的只读核验中（见[取证报告 §2.5](../../evidence/zh-cn/evidence-report.md)），所有 1M Gemini 调用均记录：
   ```text
   max_context_tokens = 256000
   truncation_reason = 2  (TRUNCATION_REASON_MAX_TOKEN_LIMIT，在跨越 256,000 边界触发压缩/截断时写入)
   ```
   正如 `@Circumsized` 指出的那样：`256,000` 恰为标称窗口 `1,048,576` 的整整四分之一（~25%），证明这不是服务端拒绝或模型物理上限，而是客户端在发送请求前主动写入请求元数据的硬性策略常数（Policy Constant）[^1]。
2. **环境变量 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 的两阶段演变：`v1.3.5–v1.4.x` 编译期抹除（`@LiraNuna`） vs. `v1.5.0–v1.5.1` 静默恢复**[^1][^2]：
   在 `google3/third_party/jetski/cli/store/store.AutoCompactOverride` 函数中，源码设计用于解析百分比（如 `80%`）或整数 Token 阈值（字符串表中存有 `"overridden by AGY_CLI_AUTO_COMPACT_OVERRIDE="` 字面量）。
   - **第一阶段（`v1.3.5`–`v1.4.x`，2026 年 10 月）**：在 `@LiraNuna` 于 Issue `#1245` 反汇编的正式发布版本中，读取环境变量的 `os.Getenv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` 调用在编译期被直接替换为寄存器清零指令（空字符串 `""`）：
     ```x86asm
     xor %eax, %eax                ; 将字符串指针置为 0 ("")
     xor %ebx, %ebx                ; 将字符串长度置为 0
     call strings.TrimSpace        ; 对空字符串调用 TrimSpace
     ```
     由于后续的 `strconv.ParseInt("", 10, ...)` 对空字符串永远解析失败，`AutoCompactOverride()` 无条件返回空字符串，导致用户在 Shell 中设置 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 完全失效[^1][^2]。
   - **第二阶段（`v1.5.0`–`v1.5.1`，2026 年 11 月 4–5 日）**：对本机 `v1.5.0`（VA `0x9c49d00`）与 `v1.5.1` 二进制的反汇编交叉核验显示，Google **在未写进任何 `CHANGELOG.md` 更新日志的情况下，静默恢复了该环境变量的读取与解析逻辑**（`lea rax, [rip+→"AGY_CLI_AUTO_COMPACT_OVERRIDE"]` + `mov ebx, 0x1d` + `call os.LookupEnv` + `call strings.TrimSpace`，其 `os.LookupEnv` 调用目标与 `GEMINI_API_KEY`、`HTTPS_PROXY` 路由至同一函数，并接续 `VA 0x9c49d40` 处的百分比/整数解析器；详见 [工程取证报告 §4.3](../../evidence/zh-cn/evidence-report.md)）。然而，底层 `GetCheckpointConfig` 中的 `0x3E800`（`256,000`）钳制指令（见下文第三道门禁）本身并未移除。在 `v1.5.1` 中，解析函数的两个调用点传入的基数都是 `0`，百分比写法因此失效；`Store.ContextWindow()` 也只在模型白名单与 `settings.json` 档位之后才读取该变量。大于 `256,000` 的值能否越过钳制尚未验证（见 [分册四 §2.4](./04-community-and-official-responses.md)）。

### 2.2 第一道门禁：UI/TUI 模型选择器白名单（`SelectorModel.offersContextWindow`）
在 `agy` CLI 的 `/model` 菜单代码中，原本已完整实现了通过 `Tab` 键在四个上下文档位之间循环切换的交互逻辑：
`Context: Default (256K) | 256K | [512K] | 900K`[^1]。
然而，控制该选项是否显示的 `google3/third_party/jetski/cli/model/model.SelectorModel.offersContextWindow` 函数（在 `v1.5.0` 二进制中位于文件偏移 `0x9e92140`）内部硬编码了一个全局白名单切片校验（指向内存地址 `0xc592be0`）：
- 在官方发布二进制中，该白名单切片**仅包含 `"Gemini 4 Argon"` 一个模型字符串**[^1]。
- 对于同样具备 1M 物理上下文窗口的 `Gemini 3.8 Flash`、`Gemini 3.1 Pro` 等所有其他模型，`offersContextWindow` 直接返回 `false`（`xor %eax, %eax`），从而在 TUI 界面中彻底隐藏 `Context` 切换栏并忽略用户的 `Tab` 按键事件[^1]。
- 而在 Antigravity 2.0 桌面端与 IDE 中（截至 `v2.28.2` / `v2.29.0`），尽管二进制包内已编译入 `ContextWindowOptions` 结构，但前端 UI 选择器仍处于未放开的受控隐藏状态[^18][^19]。

- **解锁界面实证（`@LiraNuna` 发布于 [GitHub Issue #1245](https://github.com/google-antigravity/antigravity-cli/issues/1245)，[查看原始截图](https://github.com/user-attachments/assets/dda2af37-98a9-4d89-b78e-8cae8c6361cc)）**：将 `SelectorModel.offersContextWindow` 补丁为返回 `true` 后，`agy` CLI 的 `Switch Model`（`/model`）终端菜单在 `Gemini 4 Argon`、`> Gemini 3.8 Flash (current)`、`Claude Opus 5.5`、`Claude Sonnet 5.5`、`GPT-OSS 120B (Medium)` 及 `Effort` 滑块下方暴露出原本被隐藏的 `Context` 选择行（`Default`、`256K`、`[512K]`、`900K`），且在选中 `[512K]` 时右侧以红字提示 **`about 1.3x quota per turn`**（每轮消耗约 1.3 倍配额）[^1]。

### 2.3 第二道门禁：配置读取与后端同步的二次白名单丢弃（`store.(*Store).ContextWindow()`）
即便用户通过补丁绕过了第一道 UI 门禁，或者手动打开 `~/.gemini/antigravity-cli/settings.json` 写入 `"contextWindow": "512K"` 或 `"900K"`，在非 Argon 模型下依然无效[^1]。
原因在于：客户端内部虽然存在一个直接从 `settings.json` 读取并规范化配置的 `store.(*Store).ContextWindowChoice()` 函数（`v1.3.5` 偏移 `0x917d800`，`v1.5.0` 偏移 `0x9c49fa0`），但负责向后端同步配置的 `Manager.syncContextWindow()` 调用的却是另一个包装函数 **`google3/third_party/jetski/cli/store/store.(*Store).ContextWindow()`**（`v1.3.5` 偏移 `0x917d892`，`v1.5.0` 偏移 `0x9c49fe0`）[^1][^2]。
该函数内部包含**第二次完全相同的 `"Gemini 4 Argon"` 白名单循环比对**：
```x86asm
; v1.3.5 反汇编 (@LiraNuna) / v1.5.0 对应偏移 0x9c4a032:
917d892: mov  0x279b727(%rip), %rdx   ; 加载白名单切片 ("Gemini 4 Argon")
...
917d913: jl   0x917d952               ; 若当前模型不在白名单内，直接跳转至退出分支！
917d952: xor  %eax, %eax              ; 返回空字符串指针 ""
917d954: xor  %ebx, %ebx              ; 返回长度 0
917d956: leave
917d957: ret                          ; 返回 ("", 0)，根本不读取 settings.json！
```
一旦返回 `("", 0)`，`Manager.syncContextWindow()` 便会调用 `backend.SetContextWindow("")`，导致后端永远收不到用户在 `settings.json` 中配置的 `512K` 或 `900K`，并强制回退至默认 256K[^1]。

### 2.4 第三道门禁：Cortex 引擎 `GetCheckpointConfig` 中的 `0x3e800`（256,000）汇编级钳制
第三道门禁位于压缩检查点计算函数内部：即便用户使用的是白名单内唯一受支持的旗舰模型 `Gemini 4 Argon`，且后端已收到 `512K` 或 `900K` 的窗口配置，会话依然会在 `~250K–256K` 处被强制压缩[^1]。
逆向反编译揭示，底层 Cortex 引擎负责计算多轮对话级联（Conversational Cascades）自动压缩检查点配置的核心函数 **`google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig`**（在 `v1.3.5` 中位于 `0x7861894`，在本机 `v1.5.0` 中位于文件偏移 `0x7fe57fe`，在 `v1.5.1` 中位于文件偏移 `0x7fb0fde`），在计算出真实可用阈值（物理上下文窗口减去 `40,000` 安全余量，例如 1M 窗口算出约 `960,000`–`983,000`）并存入 `%edx` 寄存器后，紧接着执行了 14 字节的硬编码常数钳制指令（机器码 `81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2`）[^1][^2]：

```x86asm
; 摘自 agy v1.3.5 (0x7861894)、本机 agy v1.5.0 (0x7fe57fe) 及 agy v1.5.1 (0x7fb0fde) 实测反汇编：
7fe57fe:  8b 54 24 4c           mov    0x4c(%rsp), %edx      ; 读取计算出的可用阈值 (context - 40,000 margin)
7fe5802:  81 fa 00 e8 03 00     cmp    $0x3e800, %edx        ; 与十六进制 0x3e800 (十进制 256,000) 比较！[v1.5.1 位于 0x7fb0fe2]
7fe5808:  be 00 e8 03 00        mov    $0x3e800, %esi        ; 将 256,000 载入 %esi
7fe580d:  0f 42 f2              cmovb  %edx, %esi            ; 无符号条件传送：取 min(计算阈值, 256000)
7fe5810:  89 74 24 50           mov    %esi, 0x50(%rsp)      ; 将被锁死的 256,000 写入 TokenThreshold！
```

这 5 行汇编指令意味着：**无论上层 UI、配置文件还是模型白名单如何设置，只要计算出的压缩触发阈值大于 `256,000`（`0x3e800`），`cmovb` 指令就会无条件丢弃计算值，将 `256,000` 强行写入 `TokenThreshold`**[^1][^2]。这就是全网所有 `Gemini 4 Argon` 1M 付费用户的会话无一例外在 ~250K–276K 处崩溃压缩的直接代码根因[^1]。

### 2.5 关键机制区分：“有损摘要压缩（Compaction）” vs. “静默请求前截断（Pre-flight Truncation）”
在 GitHub Issue `#1245` Comment #2 中，逆向研究者 `@Circumsized` 基于数据库 `truncation_reason = 2` 字段指出了一个被普遍忽视的双重机制区分[^1]：
1. **自动摘要压缩（Compaction）**：当对话跨越 `TokenThreshold` 时，系统调用后台摘要模型，将此前 25 万 Tokens 的完整历史替换为一段 15K–17K Tokens 的散文式摘要，并显式注入 `CHECKPOINT: "# Resuming from a compaction..."` 标记[^1]。
2. **请求前静默截断（Pre-flight Truncation）**：当对话逼近 `max_context_tokens = 256000` 时，客户端还会在向服务端发出请求前直接从请求体头部裁掉早期对话历史（对应下一条请求的 `truncation_reason = 2`）。**这一过程不生成任何摘要、不打印任何 `CHECKPOINT` 标记，早期内容直接从请求中静默消失**[^1]。
因此，若仅仅“禁用自动摘要生成”而不提高底层的 `256,000` 常数上限，只会把“有标记的有损摘要”变成“无标记的静默丢弃历史”，反而更加隐蔽危险[^1]。

---

## 3. 256K 提前压缩与缓存失效造成的工程破坏与财务/配额黑洞

### 3.1 工程破坏：精确代码符号丢失、捏造标识符与级联失忆
1. **跨文件常量与比例尺篡改（`@Burntgogi` 实测案例）**：
   在 `#1245` 记录的 94 轮重构会话中，Agent 在第 1–75 步通过 `view_file` 读取了数据目录与状态初始化文件；第 79 步触发首次压缩后，所有原始 `view_file` 输出被丢弃并替换为散文摘要。随后在第 159–177 步编写跨模块升级映射时，模型没有重新读取源文件，而是直接凭印象编造了原目录中根本不存在的 `tier-3` 键名，并将原本按 `1000..2000` 比例尺初始化的状态字段误以为是 `0..10` 归一化比例尺，**直接将现有业务数值错误缩小了 200 倍**；在第 200 步第二次压缩后，模型更是在 UI 模块与入口模块中对同一个 `localStorage` 键分别使用了连字符（hyphen）和下划线（underscore）命名，导致模块集成彻底断裂[^1]。
2. **凭空捏造 6 个不存在的标识符（`@Hello_there` 论坛 `#189476` 实测案例）**：
   当会话出现 `Resuming from a compaction — you have lost access to the full conversation history` 后，框架未强制要求模型重新校验已读文件，导致模型在后续迭代中**凭空引入 6 个不存在的标识符**，并用错误的硬编码值覆盖了此前确立的初始化约束条件[^43]。
3. **多轮压缩后的“分段彻底失联”（`@jlukic` Issue `#878` 实测案例）**：
   当一次长会话经历多次压缩后，后续压缩摘要仅对前一段已缩写过的 ~100K 上下文再次做二次摘要，导致第 3 段会话彻底失去第 1 段会话的所有记忆，除非人工强迫 Agent 去读取它自己的本地历史会话日志文件[^3]。
4. **复杂工程未验证即宣称完成与安全绕过（`@Pitt_zhang` 论坛 `#176775` 审计案例）**：
   在 Google AI Developers Forum 帖子 [`#176775`](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775)[^46] 中，开发者对 Antigravity（AGY）在多步前后端工程中的交付质量进行了独立代码核查，记录了三类典型失忆与失控错误：① **前端完成声明与实际交付不符**（所谓备份提交 `104dc9c` 仅修改了聊天日记而未包含前端源码，独立执行 TypeScript 检查失败）；② **前后端接口契约脱节**（AGY 前端调用 `/portal-api/**` 与 `/mock-channel/**`，而后端实际仅有 `/api/**`，DTO 字段与状态枚举完全不一致）；③ **认证绕过漏洞**（前端登录在 API 请求失败时直接回退至本地管理员分支，导致攻击者可通过制造 API 失败获取管理员权限）[^46]。

### 3.2 缓存失效与 70 秒压缩停顿：为何 256K 限制反而加快配额消耗？
根据 `@Emily2040` 在 GitHub Issue `#1291` 与 `#1245` 中对 1,279 次 `Gemini 4 Argon High` 调用（总计 `10,619,129` 未缓存输入 Tokens、`192,374,637` 缓存读取 Tokens）的量化审计，Antigravity 的上下文管理在两处产生额外的配额消耗[^1][^4]：
1. **每次自动压缩中位耗时 70 秒、摘要算力未在元数据汇报且使 Prompt Cache 失效**：
   - `Gemini 4 Argon` 及后续模型**不支持显式缓存句柄（Explicit Caching）**，完全依赖前缀字节级一致的隐式缓存（Implicit Caching，正常命中率达 95%–99%，缓存读取计费权重仅为未缓存输入的 **1/20**，即 `$0.30/1M` vs `$6.00/1M`）[^1][^4][^42]。
   - 在实测的 **22 次 `checkpoint` 自动压缩**中，每次压缩耗时分布在 `35 秒`至 `602 秒`（10 分钟）之间，**中位停顿时间达 `70 秒`**[^4]。
   - 在这 70 秒内，后台发起的摘要生成调用**在客户端轨迹元数据中报告的 Token 消耗为“无（none reported）”**，用户界面不显示该次 Token 增长，但后台配额池仍会扣减[^4][^42]。
   - 此外，压缩完成后，原本已积累 240K–268K 的缓存前缀被重写作废（`cache_read_tokens → 0`），紧随其后的下一次请求需发起 **`44,990–70,041` Tokens 的全额未缓存输入调用**重建缓存，且模型因丢失代码细节不得不再次调用工具重读大文件，形成“压缩使缓存失效 → 全价重发前缀 → 重读文件再次填满 256K → 再次压缩”的循环开销[^1][^4][^5]。
2. **`system_message` 注入引发的中途全量前缀缓存失效（Mid-run Prefix Cache Miss）**：
   - 在 1,279 次调用中，仅 **17 次** `input > 100K` 且 `cache_read_tokens = 0` 的全量缓存失效调用（单次达 `113,735` 至 `268,032` Tokens，仅占总调用次数的 **1.3%**），就消耗了 **`3,292,077` Tokens，占整个运行周期全部未缓存输入 Token 总量的 `31.0%`**[^4]。
   - 除 3 次为 `agy -c` 恢复会话的冷启动外，其余 **14 次全部发生在运行中途**，且 17 次中有 **15 次发生在客户端注入 `system_message` 步骤（如子代理完成通知或定时唤醒）后的两步之内**，或发生在长耗时命令（551–1,190 秒）导致隐式缓存 TTL 过期之后[^4]。

### 3.3 `@Yiji_Kong` 控制变量实测：`Gemini 4 Argon` 按公开 API 价格的 3.0 倍（Web 端纯输入达 7.64–9.25 倍）扣减配额
2026 年 10 月 28 日至 29 日，开发者 `@Yiji_Kong` 在官方论坛 **#196351** 发布了针对 Antigravity（`v2.27.0`，Ultra 5x 订阅）与 Gemini Web 端的两组控制变量 A/B 实验，测算了订阅配额的内部计费权重[^42]：

#### 实验一：Antigravity 客户端内同桶（Same Bucket）单步无缓存 A/B 对照
通过每 30 秒轮询本地语言服务 `LanguageServerService/GetUserStatus` 的 `quotaInfo.remainingFraction` 字段，并读取 `GetCascadeTrajectoryGeneratorMetadata` 的精确 Token 数，在完全空闲 8 分钟后对 `Gemini 4 Argon (High)` 与 `Gemini 3.8 Flash (High)` 发送完全相同的无工具单步写作任务（零缓存读取）[^42]：

| 测试模型 | 输入 / 输出 Tokens | 5 小时配额池扣减比例 (`Quota drop`) | 按公开 API 挂牌价折算成本 | 每 1 美元挂牌价消耗的配额比例 (`Quota per USD`) | 归一化倍率对比 |
|---|---|---:|---:|---:|---:|
| **Gemini 4 Argon (High)** | `14,918` in / `4,233` out | `0.00144340` | `$0.0722 USD` (按 `$2 / $10` 每百万) | **`0.02000`** | **`3.00x`** (`0.0200 / 0.00667`)[^42] |
| **Gemini 3.8 Flash (High)** | `15,111` in / `3,451` out | `0.00016180` | `$0.0243 USD` (按 `$0.75 / $3.75` 每百万) | **`0.00667`** | `1.00x` (基准)[^42] |

- **核心结论**：
  1. 若按各自公开的 API 挂牌价归一化，**`Gemini 4 Argon` 在 Antigravity 内部每 1 美元名义工作量消耗的订阅配额精确等于 `Gemini 3.8 Flash` 的 `3.00 倍`**[^42]。
  2. 若以 `3.8 Flash` 的公开价格为基准，Ultra 5x 的 5 小时共享配额池隐含总额约为 **`$150.0 USD`**（`$0.024274 / 0.00016180 = $150.0`），而 `Gemini 4 Argon` 在订阅池内部的实际扣费权重高达 **输入 `$6.00` / 输出 `$30.00` 每百万 Tokens**（即公开宣传首发价 `$2.00 / $10.00` 的整整 3 倍；缓存读取权重约为 `$0.30/1M`）[^42]。
  3. 此外，一次路由至 `gemini-3.8-flash-tiered`（内部代号 `M322`，`12,773` 输入 / `16,332` 输出，0 缓存）的子代理探测调用，竟导致 `remainingFraction` 骤降 **`0.01048544`**——**高达标准 Flash 预期消耗（`~0.00047`）的 `22 倍`**[^42]！

#### 实验二：Gemini Web 端 8 位小数周配额池（`ΔWeekly`）纯输入零思考交叉验证
为排除客户端本地语言服务统计偏差或“隐藏思考 Token（Thinking Tokens）”的干扰，`@Yiji_Kong` 于 10 月 29 日在同一 Ultra 账号下通过抓取 Web 端 `/usage` 接口返回的 **8 位小数周配额池使用率（`percentageUsed`）** 进行了三组剥离思考变量的对照测试[^42]：

| 测试编号与场景 | 模型与思考档位 | 输入 / 输出负载 | 5 小时池变化 | 周配额池精确变化 (`ΔWeekly`, 8 位小数) | 实测消耗倍率结论 |
|---|---|---|---:|---:|---|
| **Test 1A（纯输入、0 思考、0 缓存）** | `Gemini 3.8 Flash (Low)` | `51,262` 字符 (~13.5K tokens) 输入 / 仅回复 `"OK"` (1 token, 0 thinking) | `0.10 → 0.10` (`0.00`) | **`+0.00003685`** (`0.09984048 → 0.09987733`) | 基准 `1.00x`[^42] |
| **Test 1B（纯输入、0 思考、0 缓存）** | `Gemini 4 Argon (Low)` | `51,262` 字符 (等长异序 Perm B) 输入 / 仅回复 `"OK"` (1 token, 0 thinking) | `0.10 → 0.12` (`+0.02`) | **`+0.00090908`** (`0.09987733 → 0.10078641`) | 原始消耗为 Flash 的 **`24.67x`**（按挂牌价折算为 **`9.25x / 美元`**；扣除固定底座后仍达 **`20.37x` 原始 / `7.64x` 每美元**）[^42] |
| **Test 2A（高思考 4000 词长文）** | `Gemini 3.8 Flash (High)` | `107` 字符输入 / 生成 `28,389` 字符 (`4,013` 词 Paxos/Raft 文章) | `0.12 → 0.12` (`0.00`) | **`+0.00014157`** (`0.10078641 → 0.10092798`) | 写一篇 4,013 词深度技术文章仅耗周池 `0.014%`[^42] |
| **Test 2B（高思考 4000 词长文）** | `Gemini 4 Argon (High)` | `107` 字符输入 / 生成 `27,859` 字符 (`4,284` 词 Paxos/Raft 文章) | `0.12 → 0.15` (`+0.03`) | **`+0.00157077`** (`0.10092798 → 0.10249875`) | 消耗为 Flash 的 **`11.10x`**（按挂牌价折算 **`4.24x / 美元`**，单轮烧掉 5 小时池的 **`~3.1%`**）[^42] |
| **Test 1C（极简 28 字符底座校准）** | `Gemini 4 Argon (Low)` | 仅输入 28 字符 `"Reply with only the word: OK"` / 回复 `"OK"` | `0.15 → 0.16` (`+0.01`) | **`+0.00015849`** (`0.10249875 → 0.10265724`) | **在 Argon (Low) 上仅发一句 `"OK"` 消耗的配额（`+0.00015849`），比在 3.8 Flash (High) 上写完一篇 4,013 词技术长文（`+0.00014157`）还要高出 `12%`！**[^42] |

针对上述实测数据，Google 官方人员 `@Maniraj_Chervirala` 在 `#196351` 第 6 楼回应确认：API 价格页面上的单价属于仅针对按量计费 API 的“促销费率（promotional per-token rates）”，而个人订阅的 5 小时与周限额**不与 API 标价挂钩，且单模型固定配额乘数不予公开（Fixed per-model quota multipliers are not published）**[^42]。

### 3.4 隐藏的高阶上下文档位倍率（`1.0x` / `1.3x` / `1.8x Expensive`）
综合 `u/Lberondroth` 对桌面端 `v2.27.0 → v2.28.1` 的 ASAR/二进制拆包（`#1xij7jx`）[^18]、Reddit `r/AntigravityGoogle` 社区拆包帖（`#1xmc5pk`）[^19]以及 `@LiraNuna` 在 Issue `#1245` 解锁的 CLI `/model` 界面实测[^1]，Google 内部已将三个上下文窗口档位与不同的每轮配额消耗乘数（Quota Multipliers）直接绑定：
- **`Default (256K)` / `256K`**：基准配额消耗倍率 **`1.0x`**[^1][^19]；
- **`512K`**：界面显示 `about 1.3x quota per turn`（每轮约 **`1.3x`** 配额消耗）[^1][^19]；
- **`900K`**：在二进制结构中附带布尔字段 **`Expensive: true`** 标记，每轮配额消耗倍率高达 **`1.8x`**[^18][^19]。

---

## 4. 官方 Issue 处理状态与 Desktop v2.29.0 / CLI 1.5.0–1.5.1 底层变动

### 4.1 GitHub 与社区层面：认领工单、未标 `bug` 标签且 20+ 天无公开回复
1. **将 `#1245` 与 `#1291` 标记为 `feedback` 而未标 `bug`**：
   在 2026 年 10 月 14 日 `@Burntgogi` 提交 **Issue #1245** 后，Google 仓库维护者 `@chandrashakherkamasani` 于 10 月 15 日将该 Issue 指派给自己（self-assigned），在添加分类标签时标注了 `comp:agent capabilities`、`feedback`（用户反馈）与 `1.3.5`，未添加 `bug` 标签（同理，对 `@Emily2040` 10 月 19 日提交的 **Issue #1291** 也标注了 `feature request` 与 `feedback` 标签并指派给自己）[^1][^4]。截至 2026 年 11 月 5 日（含 `v1.5.1` 发布后），超过 21 天过去，尽管社区已在评论区贴出 SQLite 记录、x86-64 汇编钳制指令与修复补丁，Google 官方人员在 `#1245`、`#878`、`#1291` 下方的公开回复数均为 **0**[^1][^3][^4]。
2. **官方回应出现在论坛而非 GitHub（2026-11-04）**：
   截至目前，官方对该阈值的唯一表态出现在 Google AI Developers Forum **#198191** 帖中。用户 `@Sharper` 在帖中要求桌面端支持大于 250K 的上下文，Google 官方人员 `@chunduriv` 于 23:39 UTC 回复：*“As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. That threshold is a fixed default and isn't user-configurable yet.”*（桌面端中的 Gemini 模型，例如 Gemini 4 Argon，目前会在约 250K Tokens 时自动压缩；该阈值是固定默认值，暂不支持用户配置。）回复把该问题视为“可配置压缩”的功能请求，列出三条临时缓解办法（在自然断点手动执行 `/compact`；把参考资料放在工作区文件中，供压缩后重新读取；排除构建产物、依赖与生成文件），并表示已把“用户可选压缩阈值”的需求转交工程团队评估。回复未给出时间表，也未涉及 CLI、`0x3e800` 钳制指令或上述 GitHub Issue[^57]。
3. **背后的商业与算力权衡【分析推断】**：
   结合 `@Circumsized`、`@Emily2040` 与 `@Yiji_Kong` 的实测数据，官方对 `#1245` 未作公开回应反映了成本与宣传之间的权衡：如果将 `GetCheckpointConfig` 中的 `0x3e800` 作为 Bug 移除并默认放开 1M 窗口，在多步工具调用与 `system_message` 频繁引发前缀缓存失效（单次重发 800K–900K 未缓存上下文的算力成本增长 4 倍以上）的情况下，固定月费订阅池的算力成本将显著上升[^1][^4]；反之，如果公开确认 `256,000`（25%）是人为设定的成本上限，则会凸显宣传页面与 `/context` 面板展示“1M 上下文”而客户端代码限制 75% 容量的不一致[^1][^19]。

### 4.2 CLI `1.4.3–1.5.1` 与 Desktop `v2.29.0` 的底层改动
虽然官方在 Issue 评论区未作回复，但核查 `google-antigravity/antigravity-cli` 在 2026 年 10 月 20 日至 11 月 5 日期间发布的 10 个版本更新日志（`v1.4.3` 至 `v1.5.1`）[^11]，以及 `u/Lberondroth` 于 11 月 3 日发布的桌面端 **`v2.29.0` 二进制差分拆包报告（`#1xn624i`）**[^17]，可以看到工程团队实际推进的三条更新路线：

1. **路线一：保留默认 256K 自动压缩阈值，修补压缩引发的会话异常与失忆问题（CLI `1.4.3`–`1.4.10`）**[^11]：
   - **`v1.4.3`（2026-10-20）**：修复会话被压缩后父代理丢失最近启动的子代理记录（原因是旧版摘要把每个子代理的完整初始 Prompt 放入摘要，在记录子代理 `conversationId` 前被截断；新版改为仅保留子代理类型、角色和结果）[^11]。
   - **`v1.4.4`（2026-10-21）**：改进 `/compact` 与自动压缩请求，在发起摘要请求时保留工具声明（tool declarations），以便复用已缓存的提示词前缀（对应 `@Emily2040` 在 `#1291` 中提出的 cache-aware compaction 建议）[^4][^11]。
   - **`v1.4.6`（2026-10-23）**：改进上下文压缩，在对话摘要中保留用户附加的图片、PDF 与其他文件，防止压缩后丢失多模态附件[^11]。
   - **`v1.4.7`（2026-10-24）**：修复当单条消息、工具输出或附件过大无法放入上下文时导致会话陷入无限循环反复压缩的 Bug[^11]。
   - **`v1.4.9`（2026-10-28）**：修复旧历史被裁剪（trimmed）后遗留未配对的工具调用或结果导致后续每一轮模型请求持续失败的 Bug（印证了 `@Circumsized` 所说的 Pre-flight truncation 静默裁剪机制存在）[^1][^11]。
   - **`v1.4.10`（2026-10-29）**：① 修复子代理在经历上下文压缩后丢失父代理下达的任务及父代理 ID 的 Bug（新版在子代理摘要中保留父代理最近的 10 条消息）；② 改进压缩标记与 `/context` 中显示的“压缩后体积”，将随摘要一起带入下一轮提示词的历史请求、Artifact、子代理与后台任务快照一并计入[^11]。
2. **路线二：在 Desktop `v2.29.0` 与 CLI `1.5.0` 底层重构压缩引擎、恢复 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 读取，并新增配额指示器开关**[^17]：
   - **后端二进制体积增长与新包引入**：在 2026 年 11 月 3 日推送的桌面端 `v2.29.0`（及 11 月 4 日的 CLI `v1.5.0`）中，后端 `language_server.exe` 体积单次增长 **`+4.47 MB`**，内嵌 Web UI 体积增长 **`+36%`**[^17]。反编译符号表（已在本地 `agy 1.5.0` 二进制中核验确认）显示，Google 新增了两个核心底层包：
     - **`google3/third_party/jetski/cortex/tokens/compaction`**（包含 `RequestTokens`、`SummaryTokens`、`CheckpointTokens`、`CarriedTokens`、`ContextOverheadTokens`、`OverheadTokens` 等精细化核算函数）；
     - **`google3/third_party/jetski/cortex/shared/snapshot`**（包含 `RegisterProducer`、`LookupProducer`、`NewMemoryBuffer` 等快照管理函数）[^17]。
   - **静默恢复 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 读取但保留 `0x3E800` 钳制**：如上文 §2.1 所述，`v1.5.0` 在未写入 `CHANGELOG.md` 的情况下恢复了 `store.AutoCompactOverride` 中的 `os.LookupEnv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` 读取，但 `GetCheckpointConfig` 中的 `0x3E800`（`256,000`）钳制指令与长度为 1 的 `"Gemini 4 Argon"` 白名单原样保留。
   - **受控功能开关 `enable-context-usage-indicator`**：在桌面端 `v2.29.0` 的发送按钮旁，Google 新增了一个处于 Gated（待云端开关激活）状态的 **“上下文与配额指示器（Context and contingent Indicator）”**，用于同时展示上下文占用与计划配额剩余比例（显示 *"Resets in..."* 与 *"Your plan's baseline contingent will refresh on..."*）[^17]。拆包作者 `u/Lberondroth` 在报告中对此评价道：
     > *“What you get instead is the Context and contingent Indicator, so you can soon watch your contingent drain in high definition, right next to the send button...”*（你没有等来任何配额重置，取而代之的是一个上下文与配额指示器，好让你很快就能在发送按钮旁边，以高清画质看着自己的配额流干。）[^17]
3. **路线三：CLI `v1.5.1` 维护性更新（2026-11-05 ~07:34 UTC）—— 对 256K 上下文限制只字未动**[^11]：
   - 就在本档案库发布前的 2026 年 11 月 5 日清晨（约 `07:34 UTC`），Google 推送了 CLI **`v1.5.1`**（`240,857,488` 字节，BuildID `696031ae34e5e5e26a9d7c436e85317f`，SHA-256 `72738cf0445918483956a5f0da33abdfcfb21a12dd9a421fd57d063782826669`）。其 `CHANGELOG.md` 中的全部 11 条更新均为常规维护修复（`agy acp` 开放全部模型及会话 Bug 修复、`--remote-agent` 下 `/config` 标签改名、终端命令退出后等待后台进程释放输出时间从 5 秒缩至 1 秒、子代理阻塞时显示具体等待工具、插件 OAuth 凭据 `Authorization` 头防泄露加固、Windows 下 CRLF 编辑/非 ASCII 乱码/stdio MCP 子进程残留三项修复、`AGY_ADC_AUTH` 下 `agy plugin` 修复、隐藏内置 `browser` 代理、GNU screen 复制修复），**没有任何一条涉及上下文窗口、自动压缩阈值或 256K**[^11]。
   - 对本机 `v1.5.1` 与 `v1.5.0` 二进制的直接比对证实：`GetCheckpointConfig` 中的 14 字节 `0x3E800`（`256,000`）钳制指令（`81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2`）原样保留（仅文件偏移从 `0x7fe5802` 挪至 `0x7fb0fe2`），白名单函数（`offersContextWindow`、`store.ContextWindow`、`ClampCheckpointConfigForModelInfo`）与已恢复的 `AGY_CLI_AUTO_COMPACT_OVERRIDE` 读取指令字节级一致，且未新增任何上下文或压缩相关字符串。

---

## 本分册引用来源清单（脚注定义）

[^1]: [[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue #1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245) — GitHub `google-antigravity/antigravity-cli`（作者：`Burntgogi`，评论：`NuCl34R`、`Circumsized`、`Emily2040`、`LiraNuna`、`AsabaLazy`），发布日期：2026-10-14 至 2026-11-02（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与代码逆向实证｜置信度：高）
[^2]: [Patch agy to unlock 512K/900K context window selection and fix early compaction (`patch_agy.py`)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26) — GitHub Gist（作者：`LiraNuna`），发布日期：2026-10-23（访问日期：2026-11-04｜类型：一手逆向工程补丁源码｜置信度：高）
[^3]: [[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window (Issue #878)](https://github.com/google-antigravity/antigravity-cli/issues/878) — GitHub `google-antigravity/antigravity-cli`（作者：`schalatap`，评论：`bash99`、`murias002`、`Corsagee`、`Infinnerty`、`jlukic`），发布日期：2026-08-26 至 2026-10-31（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与大规模会话统计｜置信度：高）
[^4]: [[Bug]: Five measured places the CLI spends time and tokens the work does not need: serial tool calls, uncached full-context calls and 70 s compactions, 12 s headless overhead, summary store rewritten every 1.5 s, build-time CPU instruction choice (Issue #1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291) — GitHub `google-antigravity/antigravity-cli`（作者：`Emily2040`），发布日期：2026-10-19 至 2026-10-25（访问日期：2026-11-04｜类型：一手官方仓库 Issue 与基准实测｜置信度：高）
[^5]: [[Feature Request]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue #1072)](https://github.com/google-antigravity/antigravity-cli/issues/1072) — GitHub `google-antigravity/antigravity-cli`（作者：`flotoz`，评论：`Emily2040`），发布日期：2026-09-20 至 2026-10-19（访问日期：2026-11-04｜类型：一手官方仓库 Issue｜置信度：高）
[^11]: [Google Antigravity CLI Releases & Changelog (`v1.4.2` – `v1.5.1`)](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md) — GitHub `google-antigravity/antigravity-cli`，发布日期：2026-10-18 至 2026-11-05（访问日期：2026-11-05｜类型：一手官方发版日志｜置信度：高）
[^12]: [Google Antigravity Docs — Changelog (`v2.0.1` – `v2.29.0`)](https://antigravity.google/docs/changelog) — Google Antigravity 官方文档，发布日期：2026-11-03（访问日期：2026-11-04｜类型：一手官方文档｜置信度：高）
[^17]: [[Teardown / Binary Diff] Antigravity Updated to v2.29.0](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/) — Reddit `r/google_antigravity`（作者：`u/Lberondroth`），发布日期：2026-11-03（访问日期：2026-11-04｜类型：专业社区二进制拆包报告｜置信度：高）
[^18]: [[Deep-Dive] Antigravity v2.27.0 → v2.28.1 Binary & ASAR Diff: 'Script Jail', the `oneharness` Migration, Teamwork Leaving Preview, and What's Coming Next](https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/) — Reddit `r/google_antigravity`（作者：`u/Lberondroth`），发布日期：2026-10-28（访问日期：2026-11-04｜类型：专业社区二进制拆包报告｜置信度：高）
[^19]: [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon?](https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/) — Reddit `r/AntigravityGoogle`，发布日期：2026-11-02（访问日期：2026-11-04｜类型：专业社区讨论｜置信度：高）
[^42]: [Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (#196351)](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351) — Google AI Developers Forum（作者：`@Yiji_Kong`，官方回复：`@Maniraj_Chervirala`），发布日期：2026-10-28 至 2026-10-29（访问日期：2026-11-04｜类型：一手实测报告与官方论坛回复｜置信度：高）
[^43]: [Context Compaction Amnesia and Missing Post-Write Verification Gates (#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476) — Google AI Developers Forum（提问：`@Hello_there`，官方回复：`@chunduriv`），发布日期：2026-10-15 至 2026-10-20（访问日期：2026-11-04｜类型：一手官方人员论坛回复｜置信度：高）
[^44]: [Feature request: Show context window usage in Antigravity Desktop (#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694) — Google AI Developers Forum（提问：`@Soro`，官方回复：`@chunduriv`，插件：`@dimitr-io`），发布日期：2026-09-24 至 2026-10-22（访问日期：2026-11-04｜类型：一手官方论坛回复｜置信度：高）
[^46]: [Antigravity CLI Agent reliability issue (#176775)](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775) — Google AI Developers Forum（作者：`@Pitt_zhang`），发布日期：2026-08-01（访问日期：2026-11-04｜类型：专业社区报告｜置信度：高）
[^51]: [dimitr-io/antigravity-desktop-context: Live `/context` side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context) — GitHub（作者：`dimitr-io`），发布日期：2026-10-22（访问日期：2026-11-04｜类型：开源代码仓库｜置信度：高）
[^57]: [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) — Google AI Developers Forum（提问：`@Sharper`，官方回复：`@chunduriv`），发布日期：2026-11-04（访问日期：2026-11-05｜类型：一手官方人员论坛回复｜置信度：高）
