# Google Antigravity 256K Context Limit & Headless Policy Change Archive

**Language**: **English** | [简体中文](./README.zh-CN.md)

**Web Archive**: [https://antigravity-256k-context-discrepancy.pages.dev/](https://antigravity-256k-context-discrepancy.pages.dev/)

Google Antigravity (including the Antigravity 2.0 desktop application, Antigravity IDE, and the `agy` command-line interface) advertises and displays in its client interface a `1,048,576` (1M) token context window for `Gemini 4 Argon` and `Gemini 3.8 Flash`. However, static disassembly of the official client binaries and analysis of local session databases show that a hardcoded machine-code instruction clamps the automatic context compaction threshold at `256,000` tokens (25% of the advertised window). Around the same period, Google support staff reversed 10 prior written forum confirmations regarding local subprocess invocations of `agy -p` from third-party coding tools by updating a non-contractual FAQ page without amending the formal *Additional Terms of Service (ToS)*.

This repository organizes the technical records, disassembly data, and public forum/issue history into two text-only parts: **Part I — Incident Research & Analysis** (`research/`) and **Part II — Static Binary Disassembly & Local Database Report** (`evidence/`). All documents default to English at the top level of each directory, with corresponding Chinese editions in `zh-cn/` subdirectories. No third-party binary executables or installers are stored in this repository; all analyzed official binaries are referenced by their direct Google Cloud Storage (`storage.googleapis.com`) download URLs and SHA-256 / SHA-512 cryptographic hashes so they can be verified independently.

---

## Key Findings

1. **Advertised `1,048,576` (1M) Window vs. Hardcoded `0x3E800` (`256,000`) Binary Clamp**: In both Desktop `v2.29.0` (`language_server.exe`) and CLI `v1.5.0`–`v1.5.1` (`antigravity`), `defaultmodels.Gemini4Argon` registers a maximum context window of `0x100000` (`1,048,576` tokens). However, the checkpoint configuration calculator `cortex/utils/utils.GetCheckpointConfig` executes the hardcoded x86-64 instruction sequence `cmp $0x3e800, %edx; mov $0x3e800, %esi; cmovb %edx, %esi` (`81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2`, at file offset `0x7fe5802` in `v1.5.0` and `0x7fb0fe2` in `v1.5.1`), clamping the automatic compaction trigger threshold to `min(calculated, 256,000)` across all models. While `v1.5.0` and `v1.5.1` quietly restored the `os.LookupEnv("AGY_CLI_AUTO_COMPACT_OVERRIDE")` lookup that had been compiled out as an empty string in `v1.3.5`–`v1.4.x` (in `v1.5.1` it accepts only absolute token counts and applies only to whitelisted models), the `0x3E800` clamp itself remains unchanged (see [Volume 2](./research/02-256k-hardcoded-gates-and-binary-teardown.md) and [Evidence Report §2.3 & §4](./evidence/evidence-report.md)). On 2026-11-04, Google staff member `@chunduriv` confirmed in developer forum thread [`#198191`](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191) that Gemini models in the Desktop app compact automatically at around 250K tokens and that the threshold "is a fixed default and isn't user-configurable yet" (see [Volume 5, Quote 1.8](./research/05-verbatim-quotes-and-evidence-vault.md)).
2. **Consistent `256,000` Token Ceiling & 70-Second Compaction Overhead**: In 209 pre-patch local Antigravity CLI 1M-Gemini sessions (`~/.gemini/antigravity-cli/conversations/*.db` prior to `2026-11-04 00:00:00 UTC+8`, 15,575 model calls), all 15,575 request metadata records set `max_context_tokens = 256000`, and no single-attempt call (`retries = 0`) exceeded 300K tokens. `@murias002`'s independent Issue `#878` audit of 310 CLI sessions (65,613 model calls) measured a consistent ceiling from input sizes: a maximum single-call input of `255,999` tokens and no calls above 300K. Each automatic compaction pauses the session for a median of **70 seconds**, omits the summarization call's token usage from `gen_metadata`, replaces ~250K tokens of history with a `15K–17K` prose summary (`CORTEX_STEP_TYPE_CHECKPOINT = 23`, `truncation_reason = 2`), and invalidates the 240K+ implicit prompt cache (see [Evidence Report §2.5–§3](./evidence/evidence-report.md) and [Volume 2 §3](./research/02-256k-hardcoded-gates-and-binary-teardown.md)).
3. **`Gemini 4 Argon` 3.0x Quota Consumption Ratio & Gated Context Tiers**: Controlled A/B testing within the same 5-hour quota window shows that, when normalized by published API list prices, `Gemini 4 Argon` consumes subscription quota at **3.00x the rate of `Gemini 3.8 Flash`** (equivalent to `$6/$30` per 1M tokens vs. `$2/$10` published, and `7.64x–9.25x` in web pure-input tests). Meanwhile, the client binaries already include `512K` (`1.3x` quota multiplier) and `900K` (`1.8x` quota multiplier, `Expensive: true`) context window options restricted by a length-1 `"Gemini 4 Argon"` whitelist and server-side feature flags (see [Volume 2 §3–§4](./research/02-256k-hardcoded-gates-and-binary-teardown.md) and [Evidence Report §2.4](./evidence/evidence-report.md)).
4. **`agy -p` Policy Reversal Without a ToS Amendment**: Between September 16 and October 8, 2026 (16:23 UTC), three Google support engineers posted 10 written confirmations on the official developer forum stating that running the unmodified `agy -p` binary as a local subprocess from third-party coding tools (such as Codex and Claude Code) complied with Section 6 of the *Additional Terms of Service*. On October 8 at 17:34 UTC, 71 minutes after the last of these confirmations and 26.5 hours after the confirmation in `#186246`, staff reversed this position in `#187241` and warned of automated account suspensions (`cloudcode-pa.googleapis.com` HTTP 403 `TOS_VIOLATION`), later confirming in Forum Thread `#191689` Post #8 that the formal ToS had not changed since September and that only a non-contractual FAQ page had been edited in mid-October (see [Volume 3](./research/03-agy-p-headless-policy-reversal-and-bans.md) and [Volume 5](./research/05-verbatim-quotes-and-evidence-vault.md)).

---

## Part I: Incident Research & Analysis (`research/`)

Documents the timeline from July 2025 through November 5, 2026 (`v1.5.1`), covering the binary disassembly of the context window gates, the `AGY_CLI_AUTO_COMPACT_OVERRIDE` behavior across versions, empirical quota consumption measurements, the forum records before and after the `agy -p` policy change, community and official responses to the limit, and verbatim source excerpts.

| Document | English Edition (Default) | 简体中文版 (`zh-cn/`) | Contents |
| :--- | :--- | :--- | :--- |
| **Overview Report** | [research/dossier.md](./research/dossier.md) | [research/zh-cn/dossier.md](./research/zh-cn/dossier.md) | Summary of findings, key metrics table, stakeholder perspectives, and all 57 footnote definitions |
| **Volume 1** | [01-incident-overview-and-timeline.md](./research/01-incident-overview-and-timeline.md) | [zh-cn/01-incident-overview-and-timeline.md](./research/zh-cn/01-incident-overview-and-timeline.md) | Project background, Windsurf team acquisition, legacy `gemini-cli` retirement, quota pool rules, and chronological timeline |
| **Volume 2** | [02-256k-hardcoded-gates-and-binary-teardown.md](./research/02-256k-hardcoded-gates-and-binary-teardown.md) | [zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md](./research/zh-cn/02-256k-hardcoded-gates-and-binary-teardown.md) | Context window statistics, `AGY_CLI_AUTO_COMPACT_OVERRIDE` changes (`1.3.5` vs. `1.5.0–1.5.1`), three binary gates (`0x3e800`), 70s compaction overhead, and Argon quota measurements |
| **Volume 3** | [03-agy-p-headless-policy-reversal-and-bans.md](./research/03-agy-p-headless-policy-reversal-and-bans.md) | [zh-cn/03-agy-p-headless-policy-reversal-and-bans.md](./research/zh-cn/03-agy-p-headless-policy-reversal-and-bans.md) | Record of the 10 prior forum confirmations, the October 8 reversal, confirmation that only the FAQ was edited, and account suspension appeal cases |
| **Volume 4** | [04-community-and-official-responses.md](./research/04-community-and-official-responses.md) | [zh-cn/04-community-and-official-responses.md](./research/zh-cn/04-community-and-official-responses.md) | Community and official responses to the limit: the community patch as client-side evidence, Google staff guidance, Google's ADK case study, third-party context tools, and cross-tool boundaries |
| **Volume 5** | [05-verbatim-quotes-and-evidence-vault.md](./research/05-verbatim-quotes-and-evidence-vault.md) | [zh-cn/05-verbatim-quotes-and-evidence-vault.md](./research/zh-cn/05-verbatim-quotes-and-evidence-vault.md) | Reverse-engineering posts, assembly listings, forum replies before and after the policy change, and side-by-side ToS/FAQ texts |
| **Sources Index** | [research/sources.md](./research/sources.md) | [research/zh-cn/sources.md](./research/zh-cn/sources.md) | Metadata and verbatim excerpts for 57 cited sources and 9 supplementary references |
| **Single-File Report** | [antigravity-256k-report.md](./antigravity-256k-report.md) | — | Consolidated English report in one file covering findings, timeline, disassembly, quota tests, policy records, verbatim quotes, binary hashes, and all 57 cited sources |

---

## Part II: Static Binary Disassembly & Local Database Report (`evidence/`)

Provides a read-only static disassembly comparison between `language_server.exe` (PE32+ x86-64, SHA-256 `c98e7c33...e1c3d5`) extracted from the official desktop installer `Antigravity-x64.exe` (`v2.29.0`) and the `antigravity` CLI binary (`v1.5.0` BuildID `5fb6af4f...`, plus `v1.5.1` BuildID `696031ae...`), a complete breakdown of the Antigravity CLI SQLite schema and Protobuf metadata decoded across 237 local pre-patch CLI conversation databases (209 1M-Gemini sessions, 15,575 model calls), and raw GitHub API records across 10 related Issues.

| Document / Directory | Path | Description |
| :--- | :--- | :--- |
| **English Report (Default)** | [evidence/evidence-report.md](./evidence/evidence-report.md) | 10-section static analysis report in English (installer verification, machine-code offsets, SQLite statistics, and reproduction commands) |
| **Chinese Report (简体中文版)** | [evidence/zh-cn/evidence-report.md](./evidence/zh-cn/evidence-report.md) | 10-section static analysis report in Chinese (matching the English structure section by section) |
| **Raw JSON Snapshots & Manifests** | [evidence/raw/](./evidence/raw/) | Official `manifest_linux_amd64.json`, 21 raw GitHub API JSON responses, and 1 forum thread JSON snapshot (23 text files) |

### Analyzed Official Binaries & Cryptographic Hashes

No binary executables are hosted in this repository. The original files can be downloaded directly from Google's official cloud storage (`storage.googleapis.com/antigravity-public`) and verified against the hashes below:

| Component | Version / Platform | Official Download URL / Source | Size & Cryptographic Hash |
| :--- | :--- | :--- | :--- |
| **Desktop Installer** | `v2.29.0` (Windows x64) | [`Antigravity-x64.exe`](https://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-x64/Antigravity-x64.exe) | `169,295,640` bytes<br>SHA-256: `944b109bcb377f9f94f86eeea7cfa9315d4b42d84fb9d5bad689058fc11c12ba` |
| **Desktop Language Server** | `v2.29.0` (`resources/bin/language_server.exe`) | Bundled inside `app-64.7z` within the `Antigravity-x64.exe` installer above | `187,342,336` bytes<br>SHA-256: `c98e7c331da5bcc37002a550e7d625b4c636f775fa2804f73777c6a500e1c3d5` |
| **CLI Release Archive (`v1.5.0`)** | `v1.5.0` (Linux x64, 2026-11-04) | [`cli_linux_x64.tar.gz`](https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64/cli_linux_x64.tar.gz) (Manifest: [`linux_amd64.json`](https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/linux_amd64.json)) | Archive: `69,781,910` bytes (`antigravity` binary: `238,604,688` bytes, BuildID `5fb6af4f2e1d489f3ada0fd785060f48`)<br>SHA-512: `e6031f850929210705d8c2393a5665b82b8c3b19684a6d3c556cafa31f26f77acbf164dfee96ca84a5ae7670ac0cc238a974c981c517b269194bdccff271cbd5` |
| **CLI Binary (`v1.5.1`)** | `v1.5.1` (Linux x64, 2026-11-05) | [`cli_linux_x64.tar.gz`](https://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.1-4523430503120896/linux-x64/cli_linux_x64.tar.gz) (manifest: [`linux_amd64.json`](https://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/linux_amd64.json); [`CHANGELOG.md`](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md)) | Archive `72,209,583` bytes, SHA-512: `f70406654ebd966f211424338259e2d38d099c48734e901dc06975063e5a0137d55b6eb9366b75983c6908bbc20ae92a2617656bb191d8ed6fe429d654bca4f5`<br>Extracted `antigravity`: `240,857,488` bytes (BuildID `696031ae34e5e5e26a9d7c436e85317f`), SHA-256: `72738cf0445918483956a5f0da33abdfcfb21a12dd9a421fd57d063782826669` |
| **CLI Install Script** | Cross-platform entry point | [`https://antigravity.google/cli/install.sh`](https://antigravity.google/cli/install.sh) | `7,643` bytes<br>SHA-256: `62966c07365423bd4dc209355060744058fb30d60f5323e2d360e39de64e5042`<br>(not redistributed here; retrieved 2026-11-04, unchanged on 2026-11-05) |

---

## Repository Structure

```text
antigravity-256k-context-discrepancy/
├── README.md                                  # English overview, key findings, index & binary hashes
├── README.zh-CN.md                            # Chinese overview, key findings, index & binary hashes
├── LICENSE                                    # Multi-licensing terms & Fair Use notice (English & Chinese)
├── antigravity-256k-report.md                 # Single-file English report consolidating Parts I and II
├── research/                                  # [Part I] Incident research reports (English default + zh-cn/)
│   ├── dossier.md                             # Overview report & key metrics table
│   ├── 01-incident-overview-and-timeline.md
│   ├── 02-256k-hardcoded-gates-and-binary-teardown.md
│   ├── 03-agy-p-headless-policy-reversal-and-bans.md
│   ├── 04-community-and-official-responses.md
│   ├── 05-verbatim-quotes-and-evidence-vault.md
│   ├── sources.md                             # Metadata & excerpts for 57 cited sources + 9 supplementary sources
│   └── zh-cn/                                 # Chinese edition (filenames 1:1 identical to English)
│       ├── dossier.md
│       ├── 01-incident-overview-and-timeline.md
│       ├── 02-256k-hardcoded-gates-and-binary-teardown.md
│       ├── 03-agy-p-headless-policy-reversal-and-bans.md
│       ├── 04-community-and-official-responses.md
│       ├── 05-verbatim-quotes-and-evidence-vault.md
│       └── sources.md
├── evidence/                                  # [Part II] Static binary disassembly & SQLite database report
│   ├── evidence-report.md                     # English 10-section static analysis report
│   ├── zh-cn/
│   │   └── evidence-report.md                 # Chinese 10-section static analysis report
│   └── raw/                                   # Raw text artifacts (23 text/JSON files, zero binaries)
│       ├── manifest_linux_amd64.json          # Official CLI release manifest (v1.5.0 SHA-512 source)
│       ├── forum_198191.json                  # Forum thread #198191 snapshot (official reply on the fixed ~250K threshold)
│       └── *.json                             # 21 raw GitHub API JSON responses
└── index.html                                 # Self-contained single-file static web archive (https://antigravity-256k-context-discrepancy.pages.dev/)
```

---

## License & Citation

This repository applies different terms to original and quoted material (see [LICENSE](./LICENSE) for full bilingual terms):

1. **Research Reports, Disassembly Notes & Data Tables (`research/*.md`, `research/zh-cn/*.md`, `evidence/evidence-report.md`, `evidence/zh-cn/evidence-report.md`, `antigravity-256k-report.md`, `index.html`)**: Licensed under **[Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)**. You may copy, redistribute, translate, or quote these materials for any purpose, provided you credit either this GitHub repository (`https://github.com/magiccodelab/antigravity-256k-context-discrepancy`) or the author's blog post (`https://afengblog.com/blog/antigravity-256k-context-discrepancy.html`).
2. **Third-Party Raw API Snapshots (`evidence/raw/`) & Quoted Texts**: The release manifest, 21 GitHub API JSON snapshots, the forum thread JSON snapshot, and quoted forum replies, *Additional Terms of Service (ToS)*, and *Official FAQ* texts remain the copyright property of their respective rightsholders (including Google LLC and respective community contributors). They are included here under **Fair Use (17 U.S.C. § 107)** for technical research and documentation, and are excluded from the CC BY 4.0 license above. Third-party scripts are not redistributed: the official install script is cited by URL and SHA-256, and the community `patch_agy.py` script remains with its author [`@LiraNuna`](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26).

---

## Notes & Disclaimer

- **Compilation & Verification**: The materials in this repository were organized by the author in collaboration with **Google Antigravity (`Gemini 4 Argon`)**. Given the volume of material, minor omissions or inaccuracies may exist; if you notice any discrepancy, please refer directly to the primary cited sources ([`research/sources.md`](./research/sources.md)) and raw API snapshots ([`evidence/raw/`](./evidence/raw/)).
- **Timezone Note (`UTC+8`)**: Because the author is based in the **UTC+8** timezone, some calendar dates or local file/build timestamps mentioned in the reports reflect `UTC+8` local time and may differ by up to one calendar day from `UTC` (or US Pacific Time) timestamps on GitHub and the developer forum. These date differences are timezone offsets rather than factual errors; where exact timestamps are needed, please check the explicit `UTC` timestamps in the timeline or the original records in [`research/sources.md`](./research/sources.md) and [`evidence/raw/`](./evidence/raw/).

> **Interestingly**: Had this investigation relied solely on the unmodified default `256,000`-token (`0x3E800`) auto-compaction threshold enforced by the official Google Antigravity CLI, Antigravity itself would have lost context mid-way and been incapable of assisting in completing this archive.

**🙏 Special Acknowledgements:**

Our sincere gratitude goes to the Google DeepMind Gemini team for training such a genuinely remarkable 1M-context model (Gemini 4 Argon) capable of synthesizing this entire forensic archive.

Equally, a special thank-you to the Google Antigravity Product & Monetization team for their truly visionary product decisions—most notably their commitment to "Get Out of the Model's Way" by hardcoding 0x3E800 (256,000) into the binary and overturning 10 official written assurances via an FAQ edit. Without your "user-first" engineering, this repository would have had no reason to exist.
