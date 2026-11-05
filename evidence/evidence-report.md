# Google Antigravity 256K Context Compaction Gate — Static Binary & SQLite Forensic Evidence Report

- **Language**: **English** | [简体中文 (zh-cn/evidence-report.md)](./zh-cn/evidence-report.md)
- **Archive Navigation**: [Repository Overview (../README.md)](../README.md) | [Part I: Deep-Research Dossier](../research/dossier.md) ([简体中文](../research/zh-cn/dossier.md)) | **Part II: Static Forensic Evidence Report (This File)** | [Primary Raw JSON & Script Artifacts (./raw/)](./raw/)
- **Report Scope**: Compiles all independently verifiable static binary disassembly and local SQLite telemetry proving that Google Antigravity (CLI `v1.5.0` and Desktop `v2.29.0`) enforces a hardcoded `256,000`-token (`0x3E800`) auto-compaction clamp on models advertised with a `1,048,576` (1M) token context window.
- **Compilation Date**: 2026-11-04 | **File Path**: `evidence/evidence-report.md`
- **Forensic Methodology**: Contains strictly two categories of primary evidence — (1) read-only static disassembly of official release binaries and read-only SQLite queries against local conversation databases; (2) public GitHub repository records verified item-by-item via the GitHub CLI (`gh api`). Zero bytes of any analyzed binary were modified, no analyzed binary was executed, and no abnormal requests were sent to any online service.
- **Primary Raw Materials**: The `./raw/` directory contains all 21 raw GitHub API JSON responses, one Google AI Developers Forum thread snapshot (`forum_198191.json`, the official reply confirming the fixed ~250K Desktop compaction threshold), and the official release manifest (`manifest_linux_amd64.json`). The official CLI install script is referenced by URL (`https://antigravity.google/cli/install.sh`, `7,643` bytes, SHA-256 `62966c07365423bd4dc209355060744058fb30d60f5323e2d360e39de64e5042`) and not redistributed; the `patch_agy.py` gist is recorded as metadata plus a content SHA-256 (`f9115f46d5949afcf6e07a5ace39dfc81f7f891a8d59a88c109ab74fe479731f`) without its script body. This repository does not host third-party binary executables; the analyzed official desktop installer `Antigravity-x64.exe` (`v2.29.0`, SHA-256 `944b109b…11c12ba`), the embedded desktop binary `language_server.exe` (SHA-256 `c98e7c33…e1c3d5`, verified byte-identical to the file bundled inside the official installer), and the official CLI release archive `cli_linux_x64.tar.gz` (`v1.5.0`, SHA-512 matching the manifest) can be downloaded and independently verified via the official Google Cloud Storage (`storage.googleapis.com/antigravity-public`) direct links and cryptographic hashes documented in Sections 1 and 9.

---

## 0. One-Page Conclusions

1. The Gemini 4 Argon model is registered in the client binaries with a true context window of 1,048,576 tokens (`0x100000`), consistent with the advertised 1M window. The capability genuinely exists in the code.
2. In those same binaries, the compaction-threshold calculation function `cortex/utils/utils.GetCheckpointConfig` contains a hardcoded instruction sequence that clamps the threshold to `min(computed value, 256,000)` (constant `0x3E800`). In the desktop binary's 56MB code section and the CLI's 60MB code section, this constant appears exactly once each.
3. The CLI and the desktop are compiled from the same source tree (`google3/third_party/jetski`): roughly 92,000 function symbols in the desktop binary and 90,000 in the CLI carry that path prefix; the bytes surrounding the clamp in both binaries are identical.
4. Across the user's 237 local pre-patch Antigravity CLI conversation databases (`~/.gemini/antigravity-cli/conversations/*.db` prior to `2026-11-04 00:00:00 UTC+8`, before any local binary patching) — comprising 209 active 1M-context Gemini sessions and 15,575 model calls (`11,110` on `Gemini 3.8 Flash`, `2,826` on `Gemini 4 Argon`, and `1,639` on `Gemini 3.7/3.6/3.5 Flash`) — the request metadata records `max_context_tokens = 256,000` on **100.0% of calls (`15,575 / 15,575`)**, with 79 `CORTEX_STEP_TYPE_CHECKPOINT` (`step_type = 23`) compaction steps and 52 `truncation_reason = 2` (`TRUNCATION_REASON_MAX_TOKEN_LIMIT`) events.
5. The CLI's `/model` interface already contains a 256K / 512K / 900K context-window selector (including the 1.3x / 1.8x cost-multiplier strings), but it is gated in two code paths by a hardcoded whitelist of length 1 whose sole entry is the string `"Gemini 4 Argon"`; on the desktop, the corresponding feature is the `context-window-options` experiment key, controlled server-side by a feature flag.
6. Official issue tracking: Issue #1245 (filed 2026-10-14, 22 👍, assigned to Google member chandrashakherkamasani, still open); #878 (filed 2026-08-26, 22 👍, open); #1371 is the desktop-side equivalent request (filed 2026-10-30; the official response redirected it to the forum). As of this dossier's date, Google has not shipped any user-facing threshold setting.
7. Notable countervailing evidence: in the latest CLI version 1.5.0 (built 2026-11-04), the read-and-parse logic for the environment variable `AGY_CLI_AUTO_COMPACT_OVERRIDE` — previously proven to be a no-op — has been restored. This shows Google is aware of the issue and able to change it, while the default 256K clamp and the whitelist remain untouched in the latest release.

---

## 1. Inventory of Files Under Examination

### 1.1 Desktop language_server.exe

| Item | Value |
|---|---|
| Origin | Copy taken from a local installation (`%LOCALAPPDATA%\Programs\antigravity\resources\bin\language_server.exe`), verified byte-identical to the file inside the official installer |
| Corresponding official release | Antigravity App v2.29.0. Official Windows x64 installer: `https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-x64/Antigravity-x64.exe` (169,295,640 bytes, SHA-256 `944b109bcb377f9f94f86eeea7cfa9315d4b42d84fb9d5bad689058fc11c12ba`, directly verifiable via this official download link) |
| Packaging model (verified) | language_server.exe is **bundled locally inside the installer**, not downloaded from the cloud at runtime: the installer is an NSIS self-extracting archive embedding `app-64.7z` (built 2026-11-03 17:36, the same moment as the user's file timestamp); the payload contains `Antigravity.exe` (the Electron shell, 245,737,472 bytes), `resources/app.asar` (the frontend package), `resources/bin/language_server.exe`, and 425 files in total |
| Provenance (verified) | The `resources/bin/language_server.exe` extracted from the official installer (187,342,336 bytes) matches the user's K: backup **byte-for-byte (SHA-256 identical, cmp passed)**: `c98e7c331da5bcc37002a550e7d625b4c636f775fa2804f73777c6a500e1c3d5`. The desktop binary analyzed in this dossier is therefore the official v2.29.0 release file |
| File size | 187,342,336 bytes |
| Format | PE32+ x86-64 console executable, 14 sections, ImageBase 0x140000000 |
| SHA-256 | `c98e7c331da5bcc37002a550e7d625b4c636f775fa2804f73777c6a500e1c3d5` (analysis copy verified identical to the original on K:) |
| Toolchain | Go (Go buildinf header at file offset 0x9610c00; symbol data consistent with the go1.28 toolchain) |
| Function symbols | 214,663 (pclntab at file offset 0x44b7440) |

### 1.2 CLI antigravity (agy) 1.5.0

| Item | Value |
|---|---|
| Acquisition | Official install script `https://antigravity.google/cli/install.sh` (not executed); the script's manifest URL `https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/linux_amd64.json` |
| Manifest contents (fetched 2026-11-04) | version `1.5.0`; url `https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64/cli_linux_x64.tar.gz`; sha512 `e6031f850929210705d8c2393a5665b82b8c3b19684a6d3c556cafa31f26f77acbf164dfee96ca84a5ae7670ac0cc238a974c981c517b269194bdccff271cbd5` |
| Verification | sha512sum of the downloaded archive matches the manifest exactly |
| Packaged file | `antigravity`, 238,604,688 bytes, build timestamp 2026-11-04 02:54 |
| Format | ELF 64-bit LSB PIE x86-64, dynamically linked, stripped, BuildID(md5/uuid) `5fb6af4f2e1d489f3ada0fd785060f48` |
| Function symbols | 177,405 (pclntab at file offset 0x1676b70; although stripped, Go pclntab function-name tables are fully preserved) |

### 1.3 Local conversation databases (Antigravity CLI)

- Directory `~/.gemini/antigravity-cli/conversations/` (per-session trajectory SQLite databases, `<uuid>.db`) and `~/.gemini/antigravity-cli/conversation_summaries.db` (global index + FTS5 full-text search database).
- Pre-patch cutoff window: 237 SQLite `.db` files with modification timestamps strictly prior to `2026-11-04 00:00:00 UTC+8` (`2026-11-03 16:00:00 UTC`, epoch `< 1793721600`, spanning `2026-07-28` through `2026-11-03 UTC+8`). All 237 files predate the user's local binary patch on November 4 and reflect the unmodified CLI client's native runtime behavior (audited strictly via read-only `mode=ro` queries against schema and numeric Protobuf metadata fields, without reading conversation text).

---

## 2. The Chain of Facts: Five Layers of Evidence

### 2.1 The advertising layer

The Gemini 4 Argon context window is 1,048,576 tokens. This number appears in:

- The title and body of the official repository's Issue #1245 ("1M-context models", "`1,048,576` token context window");
- The title of Issue #878 ("unlock full 1M context window");
- The title of Issue #1371 ("the model window is 1M").

### 2.2 The model layer (binary evidence: the capability genuinely exists)

In both binaries, Argon's model info is constructed by `defaultmodels.Gemini4Argon` → `defaultmodels.GeminiModelInfo`, writing the same struct fields:

- Desktop: `mov DWORD PTR [rax+0x14],0x100000` (1,048,576) @ VA 0x141ce5015; `mov DWORD PTR [rax+0x70],0x10000` (65,536) @ VA 0x141ce501c
- CLI 1.5.0: the same two instructions @ VA 0x8540415 / 0x854041c

That is, the registered maximum context is 1M. `Gemini4Argon` passes family IDs 1417/1418/1419 (`0x589/0x58a/0x58b`) according to thinking level.

### 2.3 The enforcement layer (binary evidence: the 256K clamp)

The core function is `google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig`. Disassembly (desktop, from VA 0x14176d580):

```
14176d5aa:  cmp DWORD PTR [rcx+0x8],0x143      ; family-ID 323 special case
14176d5b1:  je  0x14176da2b                     ; if matched → 1M fast path
...
14176d5dc:  cmp rdi,0x9c40                      ; 40,000
14176d5e9:  cmovg r8,rdi                        ; margin = max(model field, 40000)
14176d5f6:  sub rdx,r8                          ; candidate = context − margin
14176d6a2:  cmp edx,0x3e800                     ; ← the 256,000 clamp (sole occurrence)
14176d6a8:  mov esi,0x3e800
14176d6ad:  cmovb esi,edx                       ; threshold = min(candidate, 256000)
14176d6b0:  mov DWORD PTR [rsp+0x50],esi        ; write to TokenThreshold
14176d6ba:  cmp edx,0x4000                      ; secondary field clamped against 16,384
...
14176da77:  mov DWORD PTR [rax+0xc],0xf4240     ; family-323 path: threshold = 1,000,000
```

The byte pattern `81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2` (`cmp $0x3e800,%edx; mov $0x3e800,%esi; cmovb %edx,%esi`):

- Desktop: appears **once** across the entire 56MB `.text` (file offset 0x176cca2, VA 0x14176d6a2); 256,000 as an immediate exists in the entire code section only in this one function's two instructions;
- CLI 1.5.0: appears **once** across the entire 60MB `.text` (file offset/VA 0x7fe5802); the surrounding instruction sequence (`mov edx,[rsp+0x4c]` … `mov [rsp+0x50],esi` … `cmp edx,0x4000`) is byte-identical to the desktop.

The adjacent `ClampCheckpointConfigForModelInfo` (desktop VA 0x14176dce0) only shrinks the threshold relative to the model's true window (`maxcontext − 0x9c40`); it contains no 256K-type lock. The 256,000 value is therefore the only fixed ceiling in the entire chain.

Substituting Argon's values: threshold = min(1,048,576 − 65,536, 256,000) = 256,000.

**Common-origin corroboration**: the CLI and desktop share roughly 90,000 `google3/third_party/jetski/`-prefixed function symbols; `cortex/utils/utils.GetCheckpointConfig`, `GetCheckpointConfigForModelInfo`, `ClampCheckpointConfigForModelInfo`, `ValidateCheckpointConfigForModelInfo`, `GetDefaultCheckpointConfigForModel` exist under the same names and order in both binaries.

### 2.4 The interface layer (binary evidence: the feature exists but is gated)

CLI 1.5.0:

- In the string table, `512K`, `1.3x`, `900K`, `1.8x`, `256K` are stored adjacently (around file offset 0x5cf26c0), and the proto message `ContextWindowOption` carries an `Expensive` field (accessors such as `(*ContextWindowOption).GetExpensive` are present in the symbol table). The data structures for the three-tier window selector fully exist.
- The whitelist, proven via the ELF relocation chain: the slice header is at data VA 0xc592be0, contents `{ptr→0xc580ef0, len=1, cap=1}`; the backing array @0xc580ef0 is a single string header `{ptr→0x5dcc4d3, len=14}`; the string at 0x5dcc4d3 is **`"Gemini 4 Argon"`** (14 characters). Both R_X86_64_RELATIVE relocations are recoverable from `.rela.dyn`.
- The whitelist is consumed in two places:
  - `model.SelectorModel.offersContextWindow` (function body VA 0x9e92140; the loop compares string lengths first, then calls memequal, @0x9e922c6-0x9e922d4) — controls whether the `/model` interface shows the Context option;
  - `store.(*Store).ContextWindow` (function body VA 0x9c49fe0; loads the whitelist @0x9c4a032/0x9c4a03e; non-members → `xor eax,eax; xor ebx,ebx; ret`, returning `("", 0)` @0x9c4a0f2-0x9c4a0f7) — controls whether a saved 512K/900K selection takes effect.
- The prologue of `Store.ContextWindowChoice` (the direct-config bypass) @0x9c49fa0 is `49 3b 66 10 76 27 55 48 89 e5 48 83 ec 10 48 8b 48 10 48 85 c9` — byte-identical to the anchor pattern published by LiraNuna in Issue #1245.

Desktop language_server.exe:

- Does not contain the CLI-only symbols `offersContextWindow`, `ContextWindowChoice`, or `AGY_CLI_AUTO_COMPACT_OVERRIDE` (zero hits on a whole-file search).
- Contains `model_config.ContextWindowOptions` (VA 0x141242e20): it reads configuration via the experiment key `context-window-options` (string @0x1436d7c0a); the nearby string `"Ignoring unparsable %s experiment: %v"` shows the data source is the feature-flag/experiment system; the code validates each option name and special-cases `"Default"`. The tier values are not in the binary — they are delivered via server-side experiment configuration.

### 2.5 The runtime layer (Antigravity CLI SQLite architecture & 15,575-call pre-patch audit)

#### 2.5.1 Antigravity CLI SQLite Database Architecture & Core Protobuf Schema

Antigravity CLI persists session state locally under `~/.gemini/antigravity-cli/` across two SQLite tiers:

1. **Global Session Index (`~/.gemini/antigravity-cli/conversation_summaries.db`)**:
   - `conversation_summaries`: Stores session-level index metadata (`conversation_id` PRIMARY KEY, `title`, `preview`, `step_count`, `last_modified_time`, `workspace_uris`, `status`, `source`, `project_id`, `agent_name`, `parent_conversation_id`, `nesting_depth`, `last_user_input_time`, `last_user_input_step_index`, `raw_summary` BLOB, `group_id`).
   - `conversations_fts`: An FTS5 virtual table (`tokenize = 'unicode61 remove_diacritics 2'`, `prefix = '1 2 3'`) indexing chunked conversation steps (`chunk_first_step`, `chunk_last_step`, `title`, `workspace`, `user_body`, `agent_body`).
2. **Per-Session Trajectory Store (`~/.gemini/antigravity-cli/conversations/<uuid>.db`, 7 tables)**:
   - `trajectory_meta` (`trajectory_id` PRIMARY KEY, `cascade_id`, `trajectory_type`, `source`): Trajectory identity and invocation source.
   - `steps` (`idx` PRIMARY KEY, `step_type` INTEGER, `status` INTEGER, `has_subtrajectory` NUMERIC, `metadata` BLOB, `error_details` BLOB, `permissions` BLOB, `task_details` BLOB, `render_info` BLOB, `step_payload` BLOB, `step_format` INTEGER): Every trajectory step in execution order, indexed by `idx_steps_status` and `idx_steps_step_type` (`CORTEX_STEP_TYPE_PLANNER_RESPONSE = 15`, `CORTEX_STEP_TYPE_CHECKPOINT = 23`).
   - `gen_metadata` (`idx` PRIMARY KEY, `data` BLOB, `size` INTEGER): Serialized `exa.cortex_pb.GeneratorMetadata` Protobuf record for each model call. Notably, background compaction steps (`CORTEX_STEP_TYPE_CHECKPOINT = 23`) insert a row into `steps` without writing a corresponding `gen_metadata` record, which is why summarization token usage is omitted from trajectory statistics.
   - `executor_metadata` (`idx` PRIMARY KEY, `data` BLOB), `parent_references` (`idx` PRIMARY KEY, `data` BLOB), `trajectory_metadata_blob` (`id` PRIMARY KEY DEFAULT `"main"`, `data` BLOB), and `battle_mode_infos` (`idx` PRIMARY KEY, `data` BLOB).

3. **Authoritative Protobuf Hierarchy Inside `gen_metadata.data` (`exa.cortex_pb.GeneratorMetadata`)** (extracted directly from the embedded `google.protobuf.FileDescriptorProto` tables in `agy` at `0x66298e2–0x662a950` and `0x65d62cd`):
   - `GeneratorMetadata.chat_model` (`field 1`, `exa.cortex_pb.ChatModelMetadata`) & `.step_indices` (`field 2`, `repeated uint32`):
     - `ChatModelMetadata.model` (`field 3`, `exa.codeium_common_pb.Model` enum)
     - `ChatModelMetadata.usage` (`field 4`, `exa.codeium_common_pb.ModelUsageStats`):
       - `input_tokens` (`field 2`, `uint64`): Uncached prompt input tokens billed on the call
       - `output_tokens` (`field 3`, `uint64`): Generated output tokens
       - `cache_write_tokens` (`field 4`, `uint64`): Tokens written to prompt cache
       - `cache_read_tokens` (`field 5`, `uint64`): Cached prefix tokens read (`input_tokens + cache_read_tokens` = total prompt context sent to the model)
       - `thinking_output_tokens` (`field 9`, `uint64`) & `response_output_tokens` (`field 10`, `uint64`)
     - `ChatModelMetadata.chat_start_metadata` (`field 9`, `exa.cortex_pb.ChatStartMetadata`):
       - `start_step_index` (`field 1`, `uint32`): First step index included in the active prompt window (`0` before compaction; jumps to the checkpoint step index immediately after compaction)
       - `checkpoint_index` (`field 2`, `int32`): Active checkpoint index (`-1` when no checkpoint has fired)
       - `steps_covered_by_checkpoint` (`field 3`, `repeated uint32`): Step indices summarized by the checkpoint
       - `created_at` (`field 4`, `google.protobuf.Timestamp`): UTC timestamp of the model call
       - `context_window_metadata` (`field 10`, `exa.cortex_pb.ContextWindowMetadata`):
         - `estimated_tokens_used` (`field 1`, `int32`): Cortex's internal token estimate compared against the `256,000` (`0x3E800`) threshold
         - `truncation_reason` (`field 2`, `exa.cortex_pb.TruncationReason` enum: `0 = TRUNCATION_REASON_UNSPECIFIED`, `1 = TRUNCATION_REASON_PROMPT_CACHE_TTL_EXPIRED`, `2 = TRUNCATION_REASON_MAX_TOKEN_LIMIT`)
         - `token_breakdown` (`field 3`, `exa.cortex_pb.TokenBreakdown`)
         - `max_context_tokens` (`field 4`, `optional int32`): Effective context window limit enforced on the call
     - `ChatModelMetadata.retries` (`field 14`, `uint32`), `.response_model` (`field 19`, `string`), and `.model_display_name` (`field 21`, `string`).

#### 2.5.2 Pre-Patch Local CLI Database Audit (`< 2026-11-04 00:00:00 UTC+8`)

Full read-only decoding of `gen_metadata` across all 237 local CLI `.db` files modified prior to `2026-11-04 00:00:00 UTC+8` (`2026-11-03 16:00:00 UTC`, 215 non-empty sessions containing 15,871 total model calls):

- **1M-Context Gemini Model Calls**: **209 sessions, 15,575 model calls** (`11,110` on `Gemini 3.8 Flash` variants, `2,826` on `Gemini 4 Argon` High/Medium/Low variants, and `1,639` on `Gemini 3.7 / 3.6 / 3.5 Flash` variants);
- **Recorded `max_context_tokens` on 1M Gemini Models**: **`256,000` on 100.0% of calls (`15,575 / 15,575`)** — zero pre-patch 1M Gemini calls recorded `1,048,576`, `900,000`, or `512,000` (the remaining 296 non-1M-Gemini calls across the 215 DBs were `114` calls on legacy `Gemini 3.1 Pro / gemini-pro-default` at `128,000`, `61` calls on `claude-opus-4-6-thinking` at `160,000`, and `121` calls on `claude-opus-5-5-medium` at `840,000`);
- **Prompt Token Distribution (`input_tokens + cache_read_tokens`)**: Median (`p50`) = `108,493`, `p90` = `211,227`, `p99` = `250,610`; **0 single-attempt calls (`retries = 0`) exceeded 300,000 tokens** ( aligns with `@murias002`'s independent community CLI audit of 310 sessions / 65,613 calls in Issue `#878`, which likewise found 0 calls above 300K and a maximum input of `255,999` tokens);
- **Compaction & Truncation Events**: **79 `CORTEX_STEP_TYPE_CHECKPOINT` (`step_type = 23`) steps** and **52 calls with `truncation_reason = 2` (`TRUNCATION_REASON_MAX_TOKEN_LIMIT`)**.

Replay of a representative compaction boundary in local CLI session `5681d8b4-f18f-4efa-be2b-35091d09b953.db` (`gen_idx 338..346`) — `estimated_tokens_used` climbs to `255,912` (88 tokens below the `256,000` ceiling), `CORTEX_STEP_TYPE_CHECKPOINT` (`step_type = 23`) fires at `step 700` without recording its own `gen_metadata` row, and on `gen_idx = 344` (`steps 701–702`) `start_step_index` jumps from `0` to `700`, `truncation_reason` is set to `2` (`MAX_TOKEN_LIMIT`), `estimated_tokens_used` drops to `35,148`, and `cache_read_tokens` collapses from `267,056` to `20,399`:

| gen_idx | steps | input_tokens | cache_read_tokens | prompt_total | estimated_tokens_used | start_step_index | truncation_reason | max_context_tokens |
|---|---|---|---|---|---|---|---|---|
| 338 | 688, 689 | 5,264 | 263,016 | 268,280 | 253,367 | 0 (`ckpt=-1`) | `0` | `256,000` |
| 340 | 692, 693 | 5,728 | 263,000 | 268,728 | 253,770 | 0 (`ckpt=-1`) | `0` | `256,000` |
| 342 | 696, 697 | 2,220 | 267,033 | 269,253 | 254,211 | 0 (`ckpt=-1`) | `0` | `256,000` |
| 343 | 698, 699 | 4,942 | 267,056 | 271,998 | **255,912** | 0 (`ckpt=-1`) | `0` | `256,000` |
| *(step 700)* | *700 (`type=23`)* | *(omitted)* | *(omitted)* | *(omitted)* | *(CHECKPOINT)* | — | — | — |
| 344 | 701, 702 | 5,962 | **20,399** | **26,361** | **35,148** | **700** (`ckpt=0`) | **`2` (`MAX_TOKEN_LIMIT`)** | `256,000` |
| 345 | 703, 704 | 3,662 | 24,476 | 28,138 | 36,719 | 700 (`ckpt=0`) | `0` | `256,000` |
| 346 | 705, 706 | 4,778 | 24,472 | 29,250 | 37,813 | 700 (`ckpt=0`) | `0` | `256,000` |

---

## 3. Desktop language_server.exe — Detailed Static-Analysis Record

### 3.1 File structure

PE section summary (`.text` VA 0x140001000, RawPtr 0x600, VSize 0x3570556; `.rdata` VA 0x143572000, etc.). Go buildinf header @0x9610c00 (`\xff Go buildinf:`). pclntab @0x44b7440, nfunc=214,663.

### 3.2 Whole-surface scan for the constant 256,000

Scanning the entire `.text` for the immediate 256,000 (both encodings: `81 f8..ff` cmp-to-register and `b8..bf` mov-imm): **only GetCheckpointConfig matches** (cmp @0x14176d6a2 + mov-imm @0x14176d6a8 — the two instructions of the single clamp). No second 256K preset exists anywhere.

### 3.3 Locations of 1,000,000 (0xF4240)

- `GetCheckpointConfig` @0x14176da77: the family-323 fast path writes the threshold directly as 1,000,000 (the comparison `cmp [mem],0x143` occurs exactly once in the entire code section @0x14176d5aa);
- `defaultmodels.GetModelInfosWithOverride` @0x141ce4c9e: constructs an override model with family ID 0x146 (326), MaxContext=1,000,000, field +0x70 = 65,536;
- `geminimodels.MapModel` @0x141ce63e1: mov-imm 1,048,576 (`0x100000`).
- The product names behind family IDs 323 and 326 cannot be determined statically from the binary (family IDs arrive via data/protocol, not as code immediates).

### 3.4 Other related functions (cortex/utils/utils package)

`GetDefaultCheckpointConfigForModel`, `GetCheckpointConfigForModelInfo`, `GetCheckpointConfig`, `ClampCheckpointConfigForModelInfo`, `ValidateCheckpointConfigForModelInfo`, `IsIgnorableCheckpointView`, spanning VA 0x14176d200-0x14176e0c0; `prodvariants.withoutCheckpointing` (@0x24aa300) shows that a compaction-disabled variant exists at the code level.

### 3.5 String audit (record of refuted "detection/anti-abuse" claims)

| Probe | Result | Notes |
|---|---|---|
| `CLAUDECODE` / `CLAUDE_CODE` | 0 hits | No environment-variable detection strings exist |
| `CLAUDE` (38 hits) | All are model names such as `CLAUDE_4_OPUS` | |
| `BOT_CHECK_*` | 13 hits | All contexts are the browser tooling's CAPTCHA/Cloudflare-handling instruction text (e.g. "navigating directly to a bot-verification challenge"); unrelated to "detecting calls from other AI tools" |
| `Getppid` / `isatty` / `IsTerminal` | Standard-library symbols only | Present in any Go binary |
| `attestation` | 13 hits | Belong to an unrelated Notary proto descriptor and workspace-training enums; no binary self-verification mechanism found |
| `offersContextWindow` / `ContextWindowChoice` / `AGY_CLI_AUTO_COMPACT_OVERRIDE` | 0 hits | CLI-only; absent from the desktop binary |
| `enable-context-usage-indicator` | 0 hits | Not in this file (cannot be verified from this file; the Electron frontend was not analyzed) |
| `ANTIGRAVITY_*` env vars | 47 distinct names | Including `ANTIGRAVITY_SENTRY_SAMPLE_RATE` (Sentry telemetry sampling rate); all serve configuration/identification purposes |
| `512K` / `256K` hits | Respectively the Go runtime's "512KiB minimum heap" and a compression block-size string | Unrelated to context-window options (false positives ruled out) |

### 3.6 Model registration path

`defaultmodels.Gemini4Argon` (VA 0x141ce4e00) passes family IDs `0x589/0x58a/0x58b` by thinking level (parameter 1/2/3) and calls `GeminiModelInfo` (0x141ce4ea0); the "Gemini 4 Argon" string occurs 4 times in the desktop binary with 5 code references, all inside `defaultmodels.Gemini4Argon` and `geminimodels.Catalog` (model registration, not the whitelist).

---

## 4. CLI 1.5.0 — Detailed Static-Analysis Record

### 4.1 The clamp

The byte pattern occurs exactly once @0x7fe5802; `cmp [mem],0x143` occurs exactly once across the section @0x7fe570a; `mov DWORD PTR [rax+0xc],0xf4240` (the 1M threshold fast path) sits inside `GetCheckpointConfigForModelInfo` @0x7fe5bd7 (a different function location than the desktop, identical behavior). The `GeminiModelInfo` constants match the desktop (0x8540415/0x854041c).

### 4.2 The whitelist (complete chain of evidence)

1. In `.rela.dyn` (file offset 0x9fa8, 979,494 entries): `r_offset=0xc592be0` → addend `0xc580ef0`; `r_offset=0xc580ef0` → addend `0x5dcc4d3`.
2. The 24 bytes at data 0xc592be0 are `00…00 | 01 00…00 | 01 00…00`, i.e. the slice header `{ptr=relocation-filled, len=1, cap=1}`.
3. The 16 bytes at 0xc580ef0 are `{ptr=relocation-filled, len=14}` — the single element, a 14-character string header.
4. The string at 0x5dcc4d3 is `Gemini 4 Argon`.
5. Consumer 1: `SelectorModel.offersContextWindow` (body @0x9e92140), loading the slice @0x9e9219e-0x9e921ad, comparison loop @0x9e922ba-0x9e922e0.
6. Consumer 2: `store.(*Store).ContextWindow` (body @0x9c49fe0), loading @0x9c4a032/0x9c4a03e, returning empty for non-members @0x9c4a0f2.

### 4.3 The evolution of AGY_CLI_AUTO_COMPACT_OVERRIDE (three independent pieces of evidence)

1. **Ineffective in the 1.3.5/1.4.x era**: in Issue #878, @Infinnerty (2026-10-15) measured that setting `compactionThreshold` / `checkpointIntervalTokens` to 30000 in `settings.json` had no effect; in Issue #1245, @LiraNuna (2026-10-22) proved by disassembly that in 1.4.x the call to `os.Getenv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` was compiled to an empty string (`xor %eax,%eax; xor %ebx,%ebx; call strings.TrimSpace`).
2. **Restored in 1.5.0**: this analysis found that in 1.5.0, `store.AutoCompactOverride` (VA 0x9c49d00) is `lea rax,[rip+→"AGY_CLI_AUTO_COMPACT_OVERRIDE"]` (string @0x5ec74c5, length 29) + `mov ebx,0x1d` + `call os.LookupEnv` (0x6b21260) + `call strings.TrimSpace` (0x6b50780).
3. **Parsing and consumption**: the parser immediately following (VA 0x9c49d40) supports two formats — `<percentage>%` (ParseFloat, upper-bound check at 100, division by 100 via the magic constant 0xa3d70a3d70a3d70b) and absolute token counts; `Store.ContextWindow` invokes this parser when no window choice is saved (call @0x9c4a0fa). The variable is undocumented.
4. **Callers**: `AutoCompactOverride` has exactly 3 direct call sites — its parser, plus two in `model.overrideNote` (UI-hint purposes); `Store.ContextWindow` consumes it indirectly through the parser.
5. **Limits in 1.5.1** (official binary, SHA-256 `72738cf0…26669`): both calls to the parser (VA 0x9c348e0) are preceded by `xor %eax,%eax` (call sites @0x9c34c9a and @0x9e875aa), passing a base of 0, so the `<percentage>%` form is rejected and only absolute token counts apply. In `Store.ContextWindow`, the variable is read only after the model whitelist passes and no tier is saved in `settings.json`. Whether a value above 256,000 survives the `GetCheckpointConfig` clamp was not tested.

### 4.4 Other CLI checks & 1.5.1 follow-up verification (2026-11-05)

- `CLAUDECODE`: 0 hits.
- The `context-window-options` experiment key: present (1 hit).
- The ELF-parsing entry of LiraNuna's gist script `patch_agy.py` checks for `\x7fELF`, so the script does not apply to PE-format desktop binaries (a mechanical fact, not a behavioral verification).
- **CLI `1.5.1` verification (`2026-11-05 ~07:34 UTC`, `240,857,488` bytes, BuildID `696031ae34e5e5e26a9d7c436e85317f`, SHA-256 `72738cf0...26669`; re-downloaded on 2026-11-05 from the official manifest's `cli_linux_x64.tar.gz`, SHA-512 `f7040665…bca4f5` matching the manifest, extracted binary byte-identical)**: all 11 `CHANGELOG.md` entries are routine maintenance fixes (`agy acp` session fixes, Windows CRLF/encoding/stdio fixes, plugin OAuth token header hardening, and UI status tweaks) with zero mention of the 256K context limit; the 14-byte `0x3E800` (`256,000`) clamp in `GetCheckpointConfig` remains byte-for-byte identical at file offset `0x7fb0fe2` (shifted from `0x7fe5802` in `1.5.0`), and the whitelist functions (`offersContextWindow`, `store.ContextWindow`) plus the restored `AGY_CLI_AUTO_COMPACT_OVERRIDE` lookup remain unchanged.

---

## 5. Complete Record of Public GitHub Corroboration

All data below was verified in real time via `gh api` on 2026-11-04; raw JSON is in the parent directory's `../raw/` folder.

### 5.1 Repository

- `google-antigravity/antigravity-cli` (https://github.com/google-antigravity/antigravity-cli)
- Created 2026-05-13; description "Antigravity CLI brings the reasoning, execution, and orchestration capabilities of Antigravity agent harness directly into your terminal."
- 2,714 stars; 841 open issues; no License file.

### 5.2 Issue #1245 (the primary issue)

- Title: `[Bug]: Premature ~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache`
- URL: https://github.com/google-antigravity/antigravity-cli/issues/1245
- Filed by @Burntgogi, 2026-10-14T19:34:38Z; state open; labels `comp:agent capabilities`, `feedback`, `1.3.5`; assignee **@chandrashakherkamasani** (self-assigned 2026-10-15).
- Body highlights (agy 1.3.5 + Gemini 4 Argon; a 94-turn / 264-step, 50-minute autonomous run, 10.38M total prompt tokens):
  - Checkpoint 1 @ Step 79: 276,110 tokens before compaction (263,524 cached + 12,586 input) → 17,336 after; cache_read dropped to zero;
  - Mid-run prefix miss @ Step 193: 243,064 tokens fully uncached;
  - Checkpoint 2 @ Step 200: 249,867 before → 15,363 after;
  - Reported consequences: after compaction the model referenced catalog item IDs that did not exist, rewrote a 1000..2000 numeric scale as 0..10 (a 200x clamping error), and named one localStorage key differently across two modules.
- All 7 comments (chronological, each verified via gh):

| Time (UTC) | Author | Summary | 👍 |
|---|---|---|---|
| 2026-10-15T18:12 | @NuCl34R | Linux measurements: compactions at 241,901 / 278,129 / 249,445 / 269,597 / 274,270 etc.; `AGY_CLI_AUTO_COMPACT_OVERRIDE=900000/100000` had no effect; the binary contains that env-var string. In a recall test 40/40 hidden codes were found (the summaries happened to retain them), but 21 of 64 facts in one report and 13 of 47 in another were wrong — including compaction summaries that "completed" 9-10 character SHA prefixes into full SHAs that do not exist | 6 |
| 2026-10-15T19:35 | @Circumsized | Database measurements: `max_context_tokens = 256000`, `truncation_reason = 2` (`~/.gemini/antigravity-cli/conversations/<id>.db`); distinguishes compaction from pre-flight truncation as two distinct history-loss mechanisms; notes that the ineffective override means the cap's source is unreachable by users | 3 |
| 2026-10-19T04:22 | @Emily2040 | Across 15 headless stream-json logs (1.4.0-1.4.2, 1,279 calls): 17 calls with input >100k and cache_read=0; 22 checkpoints with median duration 70s (35-224s, one 602s); checkpoints report no usage and are followed by 45-70k uncached calls | 1 |
| 2026-10-19T22:48 | @Emily2040 | Addendum: all 22 checkpoints fired with preceding context between 235k and 268k, consistent with the 256,000 constant; notes a public post the same day describing patching the constant to 980,000 with better long-run results, does not recommend that route, but states that "a decision this important being a hidden constant" is the root cause | 4 |
| 2026-10-22T16:37 | @LiraNuna | Disassembly findings: `/model` already has a Context selector (`Default (256K) | 256K | [512K] | 900K`), gated by a hardcoded whitelist in `SelectorModel.offersContextWindow` (only `"Gemini 4 Argon"`); `AGY_CLI_AUTO_COMPACT_OVERRIDE` compiled to an empty string in release binaries; `Store.ContextWindow()` falls back to a 256K preset | 3 |
| 2026-10-23T01:37 | @LiraNuna | Follow-up: two additional gates that keep compaction at ~250K after unlocking the UI — a second identical whitelist inside `Store.ContextWindow()`, and the 0x3E800 clamp in `cortex/utils/utils.GetCheckpointConfig` (with a 40,000 margin). Published an automated patch script to a Gist | 5 |
| 2026-11-02T14:43 | @AsabaLazy | Published an AUR package fixing this issue, `antigravity-cli-fix-bin` (https://github.com/AsabaLazy/aur-ASLZ) | 0 |

- Complete roster of reaction users (28 reaction events, 25 unique): @scarletkc, @NuCl34R, @asyrafng, @chunyoupeng, @bluekyu, @sundanc, @fikrisaman, @S-Elisei, @LiraNuna, @heswithme (×2), @x-spy, @carterfleet, @nurimator, @vegerot, @cqjjjzr, @NekoApocalypse, @Raychanan (×2), @msackman, @md-1985, @uYanJX (×2), @xuyang2, @jkooooo99, @Antisophy, @nullgit, @Sharper31.

### 5.3 Issue #878 (the earliest feature request)

- Title: `[Feature Request]: Disable automatic context auto-compaction and unlock full 1M context window`
- URL: https://github.com/google-antigravity/antigravity-cli/issues/878
- Filed by @schalatap, 2026-08-26T13:36:35Z; open; labels `feature request`, `comp:conversations`; 22 👍.
- All 5 comments (verified via gh):

| Time (UTC) | Author | Summary | 👍 |
|---|---|---|---|
| 2026-09-03T04:03 | @bash99 | Requests `contextWindowLimit` and `historyCompactionThreshold` configuration options | 1 |
| 2026-10-05T00:51 | @murias002 (Eddy Murias) | Measurements over 310 conversations / 65,613 calls: per-call input p50=136,487, p90=229,000, **max=255,999**; zero calls above 300k; 276 compaction events with pre-compaction context median 255,070 (p10 247,360, p90 255,874) and post-compaction median 45,304; nothing in settings.json, CLI flags, or the environment changes it | 1 |
| 2026-10-05T09:17 | @Corsagee | Requests exposing `compactionThreshold` in settings.json; binary inspection finds the proto schema already defines `AntigravityCompactionConfig.token_threshold`, replaced in practice by a hardcoded `ContextWindow − MaxOutputTokens` fallback | 4 |
| 2026-10-15T07:58 | @Infinnerty | agy 1.3.5 headless measurements: 5 compactions in 16 minutes (triggered at 87,386 / 99,639 / 91,934 / 84,678 / 115,605 tokens, dropping to 25k-29k); `compactionThreshold`/`checkpointIntervalTokens` set to 30000 had no effect; after the second compaction, summaries named functions that do not exist in the repo and the final report cited lines beyond file ends | 2 |
| 2026-10-31T15:49 | @jlukic | Notes the gap blocks long planning tasks; observed that after multiple compactions the summary only compacts already-truncated context ("segment 3 has no access…") | 0 |

- Complete reaction roster (22 users): @angelo0098, @asror-z, @aur9ra, @bash99, @ccl-123, @dyokism, @gengel, @ghisguth, @HarryPi, @hernancerm, @Infinnerty, @jlukic, @jomp16, @mkarimdev, @nisel11, @NuCl34R, @RodNSS, @rrhuffy, @rs3vans, @schalatap, @vegerot, @z1r0-.

### 5.4 Issue #1371 (the desktop-side equivalent request)

- Title: `[Feature Request] User-configurable auto-compaction threshold in the desktop app (~256K hardcoded while the model window is 1M)`
- URL: https://github.com/google-antigravity/antigravity-cli/issues/1371
- Filed by @floveey, 2026-10-30T07:28:59Z; open; desktop sessions compact at roughly 256K (input + cache read); reaction: @Sharper31.
- 2 comments (verified via gh):
  - 2026-10-30T11:35 @manirajc (official reply): "this repository is exclusively for the Antigravity CLI. For feature requests related to Antigravity 2.0 or IDE please reach out to [Antigravity Forum](https://discuss.ai.google.dev/c/antigravity/64)" — i.e. the desktop request was redirected to the Google forum.
  - 2026-11-01T14:05 @LiraNuna: points to #1245 and the agy patch.

### 5.5 Other related issues (all verified via gh)

| # | Title (abbreviated) | Author | Filed | State | 👍 | Commenters |
|---|---|---|---|---|---|---|
| 743 | Add max_context_tokens setting to control per-turn token budget | @thealgebraist | 2026-08-03 | open | 2 | (none) Reactions: @LogIN-, @persano |
| 1291 | Five measured places the CLI spends time and tokens the work does not need (incl. 70s compactions and uncached full-context resends) | @Emily2040 | 2026-10-19 | open | 2 | @Emily2040 (2026-10-25); reactions: @LordOfCorners, @vegerot |
| 1316 | Subagent context & compaction controls for headless orchestrator backends (--no-auto-compact, --compact-threshold, eviction modes) | @aeronmiles | — | open | 1 | @sleegme (2026-10-29); reaction: @vegerot |
| 1372 | Active warning and compaction prompt before sessions degrade | @zhangr930103 | — | open | 0 | @LiraNuna (2026-11-01) |
| 1367 | /goal autonomous loop lacks token circuit breaker & mid-flight compaction, burning 37M tokens in under an hour | @thhart | — | open | 0 | (none) |
| 919 | Implement context compaction to prevent INVALID_ARGUMENT 400 trajectory loops | @ozl | — | open | 0 | (none) |
| 1072 | Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost | @flotoz | — | open | 0 | @Emily2040 (2026-10-19) |

### 5.6 LiraNuna's Gist (patch_agy.py)

- URL: https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26
- Description: "Patch agy to unlock 512K/900K context window selection and fix early compaction"
- Owner: @LiraNuna; created 2026-10-23T01:37:08Z; last updated 2026-10-23T23:12:51Z; 2 revisions (rev1 → rev2, with rev2 switching to version-agnostic pclntab symbol resolution; 174 additions / 86 deletions).
- File: `patch_agy.py` (11,290 bytes).
- The four modifications declared by the script (mutually corroborated by this dossier's binary analysis): ① force `SelectorModel.offersContextWindow` to return true; ② keep `store.AutoCompactOverride` clean; ③ redirect `Store.ContextWindow` to `Store.ContextWindowChoice`; ④ remove the 0x3E800 clamp in `GetCheckpointConfig`.
- This dossier cites the script solely as public evidence that the community located the same set of functions and constants; the script was never executed.

### 5.7 Complete participant roster (deduplicated)

**Issue authors (10)**: @Burntgogi, @schalatap, @thealgebraist, @Emily2040, @floveey, @aeronmiles, @zhangr930103, @thhart, @ozl, @flotoz

**Commenters (12, deduplicated)**: @NuCl34R, @Circumsized, @Emily2040 (comments on #1245/#1291/#1072), @LiraNuna (comments on #1245×2/#1371/#1372), @AsabaLazy, @bash99, @murias002, @Corsagee, @Infinnerty, @jlukic, @sleegme, @manirajc (official)

**Reaction participants (48, merged and deduplicated across issues)**: @scarletkc, @NuCl34R, @asyrafng, @chunyoupeng, @bluekyu, @sundanc, @fikrisaman, @S-Elisei, @LiraNuna, @heswithme, @x-spy, @carterfleet, @nurimator, @vegerot, @cqjjjzr, @NekoApocalypse, @Raychanan, @msackman, @md-1985, @uYanJX, @xuyang2, @jkooooo99, @Antisophy, @nullgit, @Sharper31, @angelo0098, @asror-z, @aur9ra, @bash99, @ccl-123, @dyokism, @gengel, @ghisguth, @HarryPi, @hernancerm, @Infinnerty, @jlukic, @jomp16, @mkarimdev, @nisel11, @RodNSS, @rrhuffy, @rs3vans, @schalatap, @z1r0-, @LogIN-, @persano, @LordOfCorners

**Official personnel**: @chandrashakherkamasani (Issue #1245 assignee), @manirajc (official reply on #1371)

---

## 6. Timeline of Version Behavior

| Date | Event | Evidence |
|---|---|---|
| 2026-08-03 | #743 requests a max_context_tokens setting | gh |
| 2026-08-26 | #878 requests unlocking the full 1M window | gh |
| 2026-08-30~10-04 | @murias002's measurement window over 310 conversations (max input 255,999) | gh (#878 comment) |
| 2026-10-14 | #1245 filed; assigned 10-15 | gh |
| 2026-10-19 | @Emily2040's two quantified addenda (all 22 checkpoints fired at 235k-268k) | gh (#1245/#1291) |
| 2026-10-22~23 | @LiraNuna publishes the disassembly analysis and the Gist patch | gh (#1245 + gist metadata) |
| 2026-10-30 | #1371 (desktop) filed; redirected to the forum by an official reply the next day | gh |
| 2026-11-02 | @AsabaLazy publishes the AUR fix package | gh (#1245) |
| 2026-11-04 | CLI 1.5.0 built (02:54) and released: `AGY_CLI_AUTO_COMPACT_OVERRIDE`'s Getenv + parsing restored; the 256,000 clamp, the length-1 `"Gemini 4 Argon"` whitelist, and the 40,000 margin all unchanged | this dossier's static analysis |

---

## 7. Unverified and Unverifiable Items (Statement of Intellectual Honesty)

The following are **not cited as facts** in this dossier:

1. **Server-side capability**: actual server-side support for the 512K/900K tiers and the enforcement of 1.3x/1.8x billing. Local evidence is limited to client-side UI strings (`512K`/`1.3x`/`900K`/`1.8x` stored adjacently) and proto fields (the existence of `ContextWindowOption.Expensive` and `BidiGenerateContentSetup.context_window_compression{trigger_tokens, sliding_window}`). Server behavior was not verified.
2. **Ban and anti-abuse claims**: neither binary contains evidence of `CLAUDECODE`/`CLAUDE_CODE` environment detection, parent-process inspection, or non-TTY detection (`BOT_CHECK_*` is browser CAPTCHA-handling text; `Getppid`/`isatty` are standard-library symbols). Earlier claims about bans related to invocation method (involving forum users Man_A and Dai_L_u) were not corroborated by local evidence or GitHub.
3. **Desktop Electron frontend (app.asar)**: not analyzed. `enable-context-usage-indicator`, the frontend presentation of the three ContextWindowOptions tiers, and the "v2.30 next week" timeline cannot be verified from language_server.exe.
4. **"Lberondroth's v2.28.1→v2.29.0 unpacking comparison"**: GitHub user Lberondroth (Louis Berndroth) exists (account created 2021-03-26) but has zero public gists; no verifiable analysis report could be found. Claims from that source are not cited in the factual sections of this dossier.
5. **Reddit user u/z1r0_**: no such GitHub user; Reddit content not verified (the #878 reaction list contains @z1r0-, a different username).
6. **Patch effects**: this dossier executed no patches and measured no post-modification behavior. Community reports (the public post mentioned by @Emily2040 in #1245, and @AsabaLazy's AUR package) are recorded only as evidence of community response.
7. **The exact location of the 262,144 (256×1024) preset**: Issue #1245's author LiraNuna described the 1.3.5-era `Store.ContextWindow()` fallback preset as 262,144; in 1.5.0 that function's structure has changed (options-table iteration + environment-variable parsing), and this analysis did not pinpoint the exact instruction holding that fallback value in the new version. This does not affect the main conclusion — the effective ceiling is determined by `GetCheckpointConfig`'s 256,000 clamp, a point doubly confirmed by local pre-patch CLI runtime data (15,575 1M Gemini calls across 209 sessions with 100% `max_context_tokens=256,000`) and community measurements (`@murias002` in Issue #878: 65,613 calls across 310 sessions with max input 255,999 and pre-compaction median 255,070).

---

## 8. Reproduction Method

Environment: WSL2 Ubuntu (x86_64); tools: python3 (stdlib struct/sqlite3), objdump (GNU Binutils 2.42), `gh` (GitHub CLI, logged in as magiccodelab).

1. Verify the desktop copy: `sha256sum` matches the original (`c98e7c33…e1c3d5`).
2. PE/ELF structure and pclntab parsing: python3 struct reads section tables; Go pclntab (magic `f1 ff ff ff`, ptrSize=8) parsed for the function-name table.
3. Byte-pattern search: whole-file scan with `data.find(bytes.fromhex('81fa00e80300be00e803000f42f2'))`.
4. Disassembly: `objdump -d -M intel --start-address=<VA> --stop-address=<VA> <binary>`.
5. Whitelist relocation chain: parse `.rela.dyn` (file offset 0x9fa8) for the two R_X86_64_RELATIVE entries with r_offset=0xc592be0 and 0xc580ef0; read the corresponding data-section string headers and target strings.
6. Conversation-store decoding: open read-only via `sqlite3 "file:<db>?mode=ro"`; proto definitions (GenMetadata/ChatModelMetadata/ModelUsageStats/ChatStartMetadata/ContextWindowMetadata and the CORTEX_STEP_TYPE enum values) were extracted from the embedded protobuf-descriptor string area of language_server.exe; a generic protobuf decoder validated the field paths by hand (see 2.5).
7. CLI acquisition: `curl -fsSL https://antigravity.google/cli/install.sh` (script saved, not executed) → `curl` the manifest → `curl` the tarball → `sha512sum` verified → `tar -xzf` unpacked. The binary was never executed.
8. GitHub data: `gh api repos/google-antigravity/antigravity-cli/issues/{1245,878,743,1291,1316,1371,1372,1367,919,1072}`, `.../issues/<n>/comments`, `.../issues/<n>/reactions`, `search/issues?q=repo:...+compaction+in:title`, `gists/4c058aed...`. Raw responses are stored in `./raw/`.

---

## 9. Complete Index of References

- Issue #1245: https://github.com/google-antigravity/antigravity-cli/issues/1245
- Issue #878: https://github.com/google-antigravity/antigravity-cli/issues/878
- Issue #743: https://github.com/google-antigravity/antigravity-cli/issues/743
- Issue #1291: https://github.com/google-antigravity/antigravity-cli/issues/1291
- Issue #1371: https://github.com/google-antigravity/antigravity-cli/issues/1371
- Issue #1316: https://github.com/google-antigravity/antigravity-cli/issues/1316
- Issue #1372: https://github.com/google-antigravity/antigravity-cli/issues/1372
- Issue #1367: https://github.com/google-antigravity/antigravity-cli/issues/1367
- Issue #919: https://github.com/google-antigravity/antigravity-cli/issues/919
- Issue #1072: https://github.com/google-antigravity/antigravity-cli/issues/1072
- Gist patch_agy.py: https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26
- AUR fix package: https://github.com/AsabaLazy/aur-ASLZ
- Official CLI install script: https://antigravity.google/cli/install.sh
- Official download page (all desktop/CLI/IDE entry points): https://antigravity.google/download
- Antigravity App v2.29.0 official direct links (GCS bucket `antigravity-public/antigravity-hub`):
  - Windows x64: https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-x64/Antigravity-x64.exe
  - Windows ARM64: https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-arm/Antigravity-arm64.exe
  - macOS Apple Silicon: https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/darwin-arm/Antigravity.dmg
  - macOS Intel: https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/darwin-x64/Antigravity.dmg
  - Linux x64: https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/linux-x64/Antigravity.tar.gz
  - Linux ARM64: https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/linux-arm/Antigravity.tar.gz
- CLI 1.5.0 official direct links for all platforms (GCS bucket `antigravity-public/antigravity-cli`):
  - Windows x64 (.exe): https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/windows-x64/cli_windows_x64.exe
  - macOS x64: https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/darwin-x64/cli_mac_x64.tar.gz
  - macOS ARM64: https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/darwin-arm/cli_mac_arm64.tar.gz
  - Linux x64: https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64/cli_linux_x64.tar.gz
  - Linux x64 musl: https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64-musl/cli_linux_x64_musl.tar.gz
  - Linux ARM64: https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-arm/cli_linux_arm64.tar.gz
- CLI manifests (authoritative version and SHA-512 source for all platforms): https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/{platform}.json (platform values such as linux_amd64, windows_amd64, darwin_arm64, etc.)
- Google official forum, Antigravity board (referenced by the official reply on #1371): https://discuss.ai.google.dev/c/antigravity/64
