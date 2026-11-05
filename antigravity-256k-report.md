# **Google Antigravity: 256K Context Window Limit & Headless Policy Changes**

**Technical Report & Primary Evidence Archive  •**  Updated: November 5, 2026 (Desktop v2.29.0 / CLI v1.5.1)  •  **GitHub Archive:** [magiccodelab/antigravity-256k-context-discrepancy](https://github.com/magiccodelab/antigravity-256k-context-discrepancy)  •  **Web Archive:** [https://antigravity-256k-context-discrepancy.pages.dev/](https://antigravity-256k-context-discrepancy.pages.dev/)

# **Executive Summary**

Google Antigravity—Google's agent-oriented software development platform spanning the Antigravity 2.0 desktop application, Antigravity IDE, the closed-source agy command-line interface, and the Python SDK \[11\]\[12\]—advertises and displays in its client interface a 1,048,576 (1M) token context window for Gemini 4 Argon and Gemini 3.8 Flash \[1\]\[3\]\[12\]. However, static disassembly of the official client binaries and read-only audits of local SQLite session databases show that client-side code enforces model whitelists and executes a hardcoded machine-code instruction sequence that clamps the automatic context compaction threshold at 256,000 tokens (25% of the advertised window) \[1\]\[2\]\[3\]. Around the same period in October 2026, Google support staff reversed 10 prior written forum confirmations regarding local subprocess invocations of agy \-p from third-party coding tools by updating a non-contractual FAQ page without amending the formal Additional Terms of Service (ToS) \[13\]\[14\]\[31\]\[32\]\[34\]\[36\].

**Scope & Methodology Note:** This document consolidates the primary technical evidence, x86-64 disassembly records, local SQLite trajectory statistics, controlled quota benchmarks, verbatim forum transcripts, and cryptographic binary hashes verified through November 5, 2026\. All analyzed official binaries are referenced by their direct Google Cloud Storage (storage.googleapis.com) download URLs and SHA-256 / SHA-512 digests for independent verification.

* **1\. Advertised 1,048,576 (1M) Window vs. Hardcoded 0x3E800 (256,000) Binary Clamp:** In both Desktop v2.29.0 (language\_server.exe) and CLI v1.5.0–v1.5.1 (antigravity), defaultmodels.Gemini4Argon registers a maximum context window of 0x100000 (1,048,576 tokens). However, the checkpoint configuration calculator cortex/utils/utils.GetCheckpointConfig executes the hardcoded x86-64 instruction sequence cmp \$0x3e800, %edx; mov \$0x3e800, %esi; cmovb %edx, %esi (81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2, at file offset 0x176cca2 in Desktop v2.29.0, 0x7fe5802 in CLI v1.5.0, and 0x7fb0fe2 in CLI v1.5.1), clamping the automatic compaction trigger threshold to min(calculated, 256,000) across all models. While CLI v1.5.0 and v1.5.1 restored the os.LookupEnv("AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE") lookup that had been compiled out as an empty string in v1.3.5–v1.4.x, the 0x3E800 clamp itself remains unchanged \[1\]\[2\]\[11\]. On November 4, 2026, Google staff member @chunduriv confirmed in Forum Thread \#198191 that Gemini models in the Desktop app compact automatically at around 250K tokens and that the threshold "is a fixed default and isn't user-configurable yet" \[57\].  
* **2\. Consistent 256,000 Token Ceiling & 70-Second Compaction Overhead:** In 209 pre-patch local Antigravity CLI 1M-Gemini sessions (15,575 model calls prior to November 4, 2026\), all 15,575 request metadata records set max\_context\_tokens \= 256000, and no single-attempt call (retries \= 0) exceeded 300K tokens. @murias002's independent Issue \#878 audit of 310 CLI sessions (65,613 model calls) measured a consistent ceiling from input sizes: a maximum single-call input of 255,999 tokens and no calls above 300K. Each automatic compaction pauses execution for a median of 70 seconds, omits the summarization call's token usage from gen\_metadata, replaces \~250K tokens of history with a 15K–17K prose summary (CORTEX\_STEP\_TYPE\_CHECKPOINT \= 23, truncation\_reason \= 2), and invalidates the 240K+ implicit prompt cache \[1\]\[3\]\[4\].  
* **3\. Gemini 4 Argon 3.0x Quota Consumption Ratio & Gated Context Tiers:** Controlled A/B testing within the same 5-hour quota window shows that, when normalized by published API list prices, Gemini 4 Argon consumes subscription quota at 3.00x the rate of Gemini 3.8 Flash (equivalent to \$6.00/\$30.00 per 1M input/output tokens vs. \$2.00/\$10.00 published, and 7.64x–9.25x per dollar in web pure-input tests) \[42\]. Meanwhile, the client binaries already include 512K (1.3x quota multiplier) and 900K (1.8x quota multiplier, Expensive: true) context window options restricted by a length-1 "Gemini 4 Argon" whitelist and server-side feature flags \[1\]\[18\]\[19\].  
* **4\. agy \-p Policy Reversal Without a ToS Amendment:** Between September 16 and October 8, 2026 (16:23 UTC), three Google support engineers posted 10 written confirmations on the official developer forum stating that running the unmodified agy \-p binary as a local subprocess from third-party coding tools (such as OpenAI Codex and Anthropic Claude Code) complied with Section 6 of the Additional Terms of Service \[23\]\[24\]\[26\]\[27\]\[28\]\[29\]\[30\]\[31\]\[32\]\[33\]. At 17:34 UTC on October 8, 71 minutes after the last of these confirmations and 26.5 hours after the confirmation in \#186246, staff reversed this position in \#187241 and warned of automated account suspensions (cloudcode-pa.googleapis.com HTTP 403 TOS\_VIOLATION) \[34\], later confirming in Forum Thread \#191689 Post \#8 that the formal ToS had not been modified since September and that only a non-contractual FAQ page had been updated in mid-October \[13\]\[14\]\[36\].

## **Summary Table of Verified Metrics**

The table below summarizes the primary quantitative measurements, binary states, and policy milestones documented across the static binary analysis, SQLite session audits, and public GitHub/Forum records.

| Category | Metric / Component | Measured Value / Binary State | Scope & Evidence Source | Citations |
| :---- | :---- | :---- | :---- | :---- |
| Context Capacity | Gemini 4 Argon / 3.8 Flash advertised window | 1,048,576 Tokens (1.0M) | Official documentation, model cards, CLI /context UI, and defaultmodels.Gemini4Argon (0x100000) | \[1\]\[3\]\[12\]\[44\] |
| Context Capacity | SQLite ContextWindowMetadata ceiling field | max\_context\_tokens \= 256000; truncation\_reason \= 2 | 100.0% of calls (15,575 / 15,575) across 209 pre-patch local 1M Gemini CLI sessions & Issue \#1245 (@Circumsized) | \[1\] |
| Binary Gate \#3 | Cortex GetCheckpointConfig hard clamp | cmp \$0x3e800, %edx; mov \$0x3e800, %esi; cmovb %edx, %esi (256,000) | Desktop v2.29.0 @ 0x14176d6a2; CLI v1.5.0 @ 0x7fe5802; CLI v1.5.1 @ 0x7fb0fe2 (sole occurrence in .text) | \[1\]\[2\] |
| Binary Gate \#1 & \#2 | CLI Context selector model whitelist | Slice len=1 at 0xc592be0: \["Gemini 4 Argon"\] | Consumed by SelectorModel.offersContextWindow (0x9e92140) and store.(\*Store).ContextWindow (0x9c49fe0) | \[1\]\[2\] |
| Env Override | AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE lookup | v1.3.5–v1.4.x: compiled to "" (xor %eax,%eax); v1.5.0–v1.5.1: os.LookupEnv restored; v1.5.1 accepts absolute token counts only, whitelisted models only | Restored at VA 0x9c49d00 in CLI v1.5.0/v1.5.1 without changelog note; 0x3E800 clamp remains active | \[1\]\[2\]\[11\] |
| Production Telemetry | 310 sessions / 65,613 model calls (@murias002) | Max input: 255,999; Calls \>300K: 0; Pre-compaction median: 255,070 | Full CLI database audit from 2026-08-30 to 2026-10-04 across 276 compaction events (post-compaction median: 45,304) | \[3\] |
| Compaction Output | Post-compaction prose summary size | 15,363 – 17,336 Tokens (\>93% history reduction) | @Burntgogi 94-turn Argon session (compacting from 276,110 and 249,867 tokens) | \[1\] |
| External Threshold | Third-party plugin safety compaction ceiling | 239,616 Tokens (256K \- 16K output margin) | Nous Research Hermes Agent plugin configuration to avoid agy's internal 256K compaction | \[48\] |
| Official Statement | Desktop compaction threshold confirmed by Google staff | \~250K tokens; "a fixed default and isn't user-configurable yet" | @chunduriv reply in Forum \#198191 (2026-11-04 23:39 UTC); classified as a feature request and passed to engineering with no timeline | \[57\] |
| Compaction Latency | Auto-compaction pause duration & usage logging | Median 70 seconds (range 35–602 s); 0 tokens reported in gen\_metadata | @Emily2040 audit across 15 headless logs (1,279 Argon High calls, 22 checkpoints) | \[1\]\[4\] |
| Cache Invalidation | Post-compaction cache retention & rebuild call | 0% cache retention; followed by 44,990–70,041 uncached input tokens | Rewriting the conversation prefix into a prose summary invalidates the \~240K implicit prompt cache | \[1\]\[4\] |
| Cache Invalidation | Mid-run prefix cache misses | 17 calls (1.3%) \= 3,292,077 tokens (31.0% of all uncached input) | 15 of the 17 mid-run cache misses occurred within two steps of a client-injected system\_message step | \[4\]\[5\] |
| CLI Overhead | Headless agy \-p startup/teardown & store I/O | 11.3 – 12.2 s fixed overhead; 943 KB summary store rewritten 27x in 90 s | @Emily2040 5-run benchmark (server execution took 4.5–8.2 s; binary startup/exit took \~12 s) | \[4\] |
| Quota Multipliers | Built-in context window tier multipliers | 256K: 1.0x \| 512K: 1.3x \| 900K: 1.8x (Expensive: true) | CLI string table (0x5cf26c0), ContextWindowOption.GetExpensive, unlocked /model UI, and Desktop teardowns | \[1\]\[18\]\[19\] |
| Quota Consumption | Gemini 4 Argon vs. 3.8 Flash same-bucket ratio | 3.00x quota per published API dollar (\$6/\$30 vs. \$2/\$10 published) | @Yiji\_Kong controlled A/B test on Antigravity v2.27.0 Ultra 5x (implied 5h bucket size: \~\$150 USD) | \[42\] |
| Quota Consumption | Gemini Web 8-decimal ΔWeekly pure-input ratio | 24.67x raw / 9.25x per dollar (7.64x net of base overhead) | 28-char "OK" prompt on Argon Low (+0.00015849) burned 12% more quota than a 4,013-word essay on Flash High (+0.00014157) | \[42\] |
| Policy Change | Time between 10th forum approval & reversal | 71 minutes (\#187536 \-\> \#187241) / 26.5 hours (\#186246 \-\> \#187241) | 10 written approvals (Sept 16 – Oct 8 16:23 UTC) reversed in a forum reply on Oct 8; FAQ entry added mid-October; ToS unmodified since Sept | \[31\]\[32\]\[33\]\[34\]\[36\] |
| Account Appeals | Two-strike enforcement & appeal stall duration | 1 recertification opportunity; permanent ban on 2nd flag; 100+ days stalled | Discussion \#20632 policy vs. unresolved appeals in GitHub Issues \#927, \#29576, and Forum \#186089 | \[8\]\[9\]\[10\]\[45\] |

As shown in the summary table, the discrepancy between the advertised 1M context window and the 256,000-token operational limit is enforced consistently across both the desktop language server and the CLI binary, while subscription quota consumption and headless CLI usage policies underwent parallel adjustments in October 2026\.

# **1\. Product Background, Architecture & Chronological Timeline**

## **1.1 The \$2.4B Windsurf Team Deal, Key Personnel & the jetski Codebase**

In July 2025, Google entered a \$2.4 billion non-exclusive technology licensing and talent acquisition agreement with the AI coding startup Windsurf (formerly Codeium). Windsurf co-founder and CEO Varun Mohan (now Director of Software Engineering at Google DeepMind) and Head of Product Engineering Kevin Hou (now Head of Product Engineering for Antigravity at Google DeepMind) joined Google DeepMind with their core engineering team to build Google Antigravity \[55\]\[56\].

Inspection of the Go runtime pclntab function symbol tables inside both the Windows desktop language server (language\_server.exe, 214,663 symbols) and the Linux CLI binary (antigravity v1.5.0, 177,405 symbols) confirms that roughly 92,000 functions in the desktop binary and 90,000 functions in the CLI are compiled from the same internal Google monorepo source tree carrying Windsurf/Codeium's legacy codename: google3/third\_party/jetski/ \[2\]. Key shared and CLI-specific symbols include:

`google3/third_party/jetski/cli/model/model.SelectorModel.offersContextWindow`  
`google3/third_party/jetski/cli/store/store.(*Store).ContextWindow`  
`google3/third_party/jetski/cli/store/store.(*Store).ContextWindowChoice`  
`google3/third_party/jetski/cli/store/store.AutoCompactOverride`  
`google3/third_party/jetski/cortex/utils/utils.GetCheckpointConfig`  
`google3/third_party/jetski/cortex/tokens/compaction/compaction.go`  
`google3/third_party/jetski/cortex/shared/snapshot/snapshot.go`

The table below lists the key leadership and official repository/forum personnel referenced across the public records in this report:

| Name & Title | Platform Handles | Role & Involvement in Public Records | Citations |
| :---- | :---- | :---- | :---- |
| Varun Mohan — Director of Software Engineering, Google DeepMind | X: @\_mohansolo | Former Co-Founder & CEO of Windsurf (Codeium); joined Google DeepMind in July 2025 via the \$2.4B licensing and talent agreement to lead Antigravity. | \[56\] |
| Kevin Hou — Head of Product Engineering, Antigravity, Google DeepMind | X: @kevinhou22 \| GitHub: @khou22 | Former Head of Product Engineering at Windsurf/Codeium; presented the AI Engineer Summit keynote "Get Out of the Model's Way" advocating against rigid scaffolding around models. | \[1\]\[55\] |
| Chandra Shakher Kamasani — Core Maintainer, Antigravity CLI Repository | GitHub: @chandrashakherkamasani | Self-assigned Issue \#1245 (256K hardcoded compaction) on Oct 15, 2026 and Issue \#1291 (70s compactions & cache invalidation) on Oct 19, 2026, applying the feedback label rather than bug, with zero public comments as of Nov 5, 2026\. | \[1\]\[4\] |
| Maniraj Chervirala — Antigravity CLI Maintainer & Forum Support | GitHub: @manirajc \| Forum: @Maniraj\_Chervirala | Assignee for GitHub Issues \#878, \#1072, \#927, and \#743; replied officially on Oct 29, 2026 in Forum Thread \#196351 confirming that published API prices apply only to pay-as-you-go billing and subscription multipliers are not published. | \[3\]\[5\]\[6\]\[8\]\[42\] |
| Rebbanpalli Naveen, Ambati Rajendra & Chunduriv — Official Forum Support Engineers | Forum: @Rebbanpalli\_Naveen, @Ambati\_Rajendra, @chunduriv | Authored the 10 written confirmations between Sept 16 and Oct 8, 2026 (16:23 UTC) stating local subprocess calls to agy \-p were compliant, and the subsequent posts starting Oct 8 (17:34 UTC) announcing the FAQ-based prohibition. | \[23\]–\[41\] |

## **1.2 Client Surfaces, Legacy gemini-cli Shutdown & Shared Quota Pool Mechanics**

According to Google Antigravity's official documentation and release notes, the platform comprises four client surfaces sharing a unified agent harness and subscription quota pool \[12\]: (1) Antigravity 2.0 Desktop App (an Electron-based multi-agent desktop application bundling language\_server.exe, browser automation, Jupyter viewer, and voice mode; v2.29.0 released November 3, 2026\) \[12\]\[17\]; (2) Antigravity IDE & Extensions (a VS Code-derived editor and IDE plugins) \[12\]; (3) Antigravity CLI (agy, a monolithic closed-source Go binary providing an interactive TUI, headless mode agy \-p / \--print, and the agy acp Agent Client Protocol stdio server introduced in v1.5.0) \[11\]\[12\]; and (4) Antigravity Python SDK (google-antigravity/antigravity-sdk-python) \[12\]\[33\]\[40\].

On June 18, 2026, Google terminated individual consumer account access to the earlier open-source Node.js @google/gemini-cli, requiring all free and paid individual subscribers to migrate to the closed-source agy binary \[10\]\[45\]. In addition to ending source-level visibility into the client, this transition removed the native OpenTelemetry (OTLP) token-usage exporter that enterprise and automated workflows previously used to track input, output, cached, and thinking tokens outside the terminal (documented in GitHub Issue \#366) \[7\].

Consumer subscriptions are divided into Free, Google AI Plus, Google AI Pro (\~\$20/month), and Google AI Ultra (5x and 20x tiers at \$200–\$250/month), providing access to first-party Gemini models (Gemini 3.5/3.6/3.7/3.8 Flash, Gemini 3.1 Pro, and flagship Gemini 4 Argon launched in mid-October 2026\) as well as a third-party model bucket (Claude Opus 5.5, Claude Sonnet 5.5, and GPT-OSS 120B Medium) \[1\]\[12\]\[13\]\[27\]\[42\]. All client surfaces—including Gemini Web and headless agy \-p—draw from a shared 5-Hour Rolling Refresh Pool (5-Hourly Pool) and 7-Day Weekly Cap Pool (Weekly Pool) \[31\]\[32\]\[42\]. Telemetry from LanguageServerService/GetUserStatus and the 8-decimal-place /usage endpoint shows that 1 Weekly Pool strictly equals 20 × 5-Hourly Pools, with a single Ultra 5x 5-hour pool corresponding to approximately \$150 USD at Gemini 3.8 Flash list prices (\$0.75/\$3.75 per 1M tokens) \[42\]. To prevent automatic deduction of paid Google One AI Credits once baseline quota is exhausted, users must set both "useAiCredits": false in \~/.gemini/config/config.json and "useG1Credits": false in \~/.gemini/antigravity-cli/settings.json \[38\]\[39\].

## **1.3 Chronological Timeline of Events (July 2025 – November 5, 2026\)**

The following chronological timeline is reconstructed from GitHub Issue and Release timestamps, UTC timestamps in the Google AI Developers Forum JSON archives, and official documentation releases.

### **Phase 1: Team Formation, Early Account Suspensions & Closed-Source Migration (2025-07 to 2026-08)**

* **2025-07:** Google completes a \$2.4B technology licensing and core team acquisition agreement with Windsurf (Codeium); Varun Mohan and Kevin Hou join Google DeepMind to build Antigravity atop the internal google3/third\_party/jetski codebase \[2\]\[55\]\[56\].  
* **2026-02-27 23:18 UTC:** Following an automated ban wave at cloudcode-pa.googleapis.com triggered by third-party OAuth proxy tools—which also locked out legitimate @google/gemini-cli and Gemini Code Assist users with HTTP 403 PERMISSION\_DENIED (TOS\_VIOLATION)—Google maintainer @jackwotherspoon posts Discussion \#20632 in google-gemini/gemini-cli. The announcement performs a one-time unban reset and establishes the "Two-Strike Policy": first-time flags can submit a Recertification Google Form for periodic automated reinstatement (1–2 days), while a second ToS violation flag results in a permanent ban \[9\].  
* **2026-05-25 to 2026-05-30:** Google AI Pro annual subscriber @jlwainwright is flagged with TOS\_VIOLATION after briefly testing an OpenClaw plugin prior to the policy clarification (remaining suspended for 100+ days despite appealing) \[8\]. On May 30, a widely discussed thread on r/google\_antigravity (\#1trr4dy) reports that aggressive history compaction is truncating sessions long before reaching Gemini's advertised 1M context window \[21\].  
* **2026-06-11 & 2026-06-18:** Developer @UesugiErii files GitHub Issue \#366 regarding the removal of OTLP token usage export in agy \[7\]. On June 18, Google terminates consumer account access to the open-source @google/gemini-cli, requiring migration to the closed-source Go binary agy \[10\]\[45\].  
* **2026-07-23 to 2026-08-26:** Open-source agy-acp author @shindgew (\#175905), @Ahmad3 (\#178472), and @JJPop (\#179013) post inquiries on the official forum asking whether spawning the unmodified agy binary as a single-user local subprocess complies with Section 6 \[23\]\[S1\]\[S2\]. Meanwhile, @Pitt\_zhang (\#176775) documents agent context loss in multi-step tasks \[46\], @thealgebraist files Issue \#743 requesting a configurable max\_context\_tokens setting \[6\], and on August 26 @schalatap files GitHub Issue \#878 requesting an option to disable automatic compaction and unlock the full 1M context window \[3\].

### **Phase 2: Ten Consecutive Official Forum Confirmations Approving Local Subprocess Calls (2026-09-02 to 2026-10-08 16:23 UTC)**

* **2026-09-02 to 2026-09-25:** @jlwainwright files Issue \#927 documenting a 100+ day unresolved ToS appeal \[8\]. On Sept 16 (\#175905, Confirmation \#1) and Sept 17 (\#183429, Confirmation \#2), Google staff @Rebbanpalli\_Naveen and @Ambati\_Rajendra confirm in writing that spawning the unmodified agy binary locally as a child process or calling agy \-p from an open-source tool is officially supported and does not violate Section 6 provided OAuth credentials are not extracted \[23\]\[24\]. On Sept 20, @flotoz files Issue \#1072 on rapid quota depletion \[5\]; on Sept 23, Nous Research's Hermes Agent sets a 239,616-token threshold to avoid agy's internal 256K compaction \[48\]; on Sept 25 (\#184829), @Ambati\_Rajendra notes that wrapping agy \-p as an MCP server for Claude Code is disallowed while direct local scripting remains supported \[25\].  
* **2026-09-28 to 2026-10-02:** Google staff issue Confirmations \#3 through \#6: @Ambati\_Rajendra confirms in \#185494 (Sept 28\) that launching a script running agy \-p from another AI coding tool is permitted ("you are good to go\!") \[27\], and in \#185460 (Sept 28\) that running agy locally as a subprocess under a single personal account is permitted (while multi-account rotation is prohibited) \[26\]. On Sept 30, @chunduriv confirms in \#185992 (Confirmation \#5) that having Anthropic's Claude Code invoke unmodified agy.exe in headless mode over stdio on an individual account is permitted \[28\]. On Oct 2, @Ambati\_Rajendra confirms in \#185949 (Confirmation \#6) that Codex CLI or Claude Code invoking a local Skill that calls a PowerShell wrapper for agy is compliant \[29\].  
* **2026-10-05 to 2026-10-08 16:23 UTC:** On Oct 5 (00:51 UTC), Eddy Murias (@murias002) publishes a 310-session / 65,613-call SQLite audit in Issue \#878 showing a maximum input of 255,999 tokens and a median pre-compaction context of 255,070 tokens \[3\]. Later on Oct 5, @Rebbanpalli\_Naveen issues Confirmation \#7 (\#186954, approving local AI orchestrator subprocess calls) \[30\] and Confirmation \#8 (\#186952, explicitly confirming that OpenAI Codex invoking unmodified agy.exe under a personal Google AI Pro plan is "completely fine to use" and "a supported use of CLI Headless Mode") \[31\]. On Oct 7 (15:03 UTC), @chunduriv issues Confirmation \#9 (\#186246), stating "there is no difference whether the unmodified agy subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code)" \[32\]. On Oct 8 (16:23 UTC), @Ambati\_Rajendra issues Confirmation \#10 (\#187536), approving automated task handoffs between agy, Codex, and Claude Code in a personal orchestrator \[33\].

### **Phase 3: The October 8 Policy Reversal, FAQ Update & Binary Disassembly of the 0x3E800 Gates (2026-10-08 17:34 UTC to 2026-11-05)**

* **2026-10-08 17:34 UTC & Mid-October FAQ Edit:** Just 71 minutes after \#187536 and 26 hours 31 minutes after his own reply in \#186246, @chunduriv posts in \#187241 that Google does not support using third-party coding agents (Codex, Claude Code, OpenClaw, OpenCode) to orchestrate or wrap agy under a consumer login, warning of automated ToS enforcement and account suspension \[34\]. Around the same time, Google adds an entry to the Official FAQ page prohibiting wrapping agy inside third-party coding agents under consumer plans, without modifying the September Additional Terms of Service \[13\]\[14\]\[36\].  
* **2026-10-14 to 2026-10-20:** @Burntgogi files Issue \#1245 on Oct 14 documenting forced compaction at \~250K–276K on Gemini 4 Argon \[1\]. On Oct 15, maintainer @chandrashakherkamasani self-assigns \#1245 with the feedback label (no bug label), @NuCl34R and @Circumsized publish experimental and SQLite ContextWindowMetadata (max\_context\_tokens \= 256000\) proofs \[1\], @Hello\_there reports post-compaction identifier fabrication in Forum \#189476 \[43\], and Desktop v2.24.0 officially launches Gemini 4 Argon and /compact \[12\]. On Oct 16 (\#189905), @Rebbanpalli\_Naveen states that the FAQ supersedes earlier forum comments \[35\]. On Oct 19, @Emily2040 files Issue \#1291 documenting 70s median compactions and 31% uncached input from mid-run prefix cache misses \[1\]\[4\].  
* **2026-10-22 to 2026-10-29:** On Oct 22–23, @LiraNuna publishes the complete disassembly of the three binary gates in Issue \#1245 (Comments \#5 & \#6) and releases patch\_agy.py \[1\]\[2\]. In Forum \#191689 (Oct 26–27), pressed by subscribers @b\_w and @Tora39, @Rebbanpalli\_Naveen confirms that the formal ToS was not modified after September and that only the FAQ was updated in mid-October \[36\]. Between Oct 26 and Oct 29 (\#194998, \#190756, \#190815, \#196472), staff extend the restriction to local file handoffs from Claude Code and standalone Python coordinators \[37\]\[38\]\[39\]\[40\]. On Oct 28–29, @Yiji\_Kong publishes controlled A/B quota benchmarks in Forum \#196351 showing Argon consumes 3.00x quota per published dollar vs. Flash, which @Maniraj\_Chervirala confirms is expected because subscription multipliers are unpublished \[42\], while u/Lberondroth's v2.28.1 diff (\#1xij7jx) uncovers ContextWindowOptions with Expensive: true \[18\].  
* **2026-11-02 to 2026-11-05 (Desktop v2.29.0 & CLI v1.5.0–v1.5.1):** On Nov 2, @AsabaLazy releases the AUR package antigravity-cli-fix-bin \[1\]\[S5\]. On Nov 3, Google releases Desktop v2.29.0 (+4.47 MB language\_server.exe, adding cortex/tokens/compaction, cortex/shared/snapshot, and gated enable-context-usage-indicator) \[12\]\[17\]. On Nov 4, Google releases CLI v1.5.0 (adding agy acp and silently restoring os.LookupEnv("AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE") at VA 0x9c49d00 while keeping the 0x3E800 clamp at 0x7fe5802) \[11\]. Later on Nov 4 (23:39 UTC), replying to @Sharper in Forum \#198191, @chunduriv confirms that Gemini models in the Desktop app compact automatically at around 250K tokens, that the threshold is a fixed default and not user-configurable yet, and that the request for a user-selectable compaction size has been passed to engineering, without a timeline \[57\]. On Nov 5 (\~07:34 UTC), Google releases CLI v1.5.1 (BuildID 696031ae...); all 11 changelog items are routine maintenance fixes, the 0x3E800 clamp remains byte-for-byte identical at file offset 0x7fb0fe2, and core GitHub Issues \#1245, \#878, and \#1291 remain without official response \[1\]\[3\]\[4\]\[11\].

# **2\. The 256K Hardcoded Gates, Static Binary Disassembly & SQLite Database Audit**

## **2.1 Advertised 1M Capacity vs. Five Independent Production Session Datasets**

Across Google Antigravity's official documentation, release notes \[12\], model cards, and the interactive /context panel of the agy CLI, both Gemini 4 Argon and Gemini 3.8 Flash display a context window capacity of 1,048,576 Tokens (1.0M) \[1\]\[3\]\[12\]\[44\]. When a session reaches approximately 222K tokens, the CLI /context panel reports utilization at 21.2% (222.4k/1.0M tokens) with Free space: 826.2k (78.8%) \[44\]\[51\].

**Public UI Reference Screenshots: (1)** [Unlocked agy CLI /model Menu Screenshot](https://github.com/user-attachments/assets/dda2af37-98a9-4d89-b78e-8cae8c6361cc) (posted by @LiraNuna in GitHub Issue \#1245 \[1\], showing the hidden Context: Default \| 256K \| \[512K\] \| 900K row and the red "about 1.3x quota per turn" warning); (2) [Side-by-Side CLI /context and Desktop Context Usage Panel Screenshot](https://d3qe71uytubmmx.cloudfront.net/original/3X/e/d/ed8ffb40b5376fcc45cc7cf33c70aa687083b01e.jpeg) (posted by @dimitr-io in Forum Thread \#184694 \[44\]\[51\], showing a session at 222.4k–225.0k tokens with 1 older checkpoint already summarized while reporting 823.6k / 78.5% free space out of 1.0M).

Despite the interface reporting more than 75% available capacity, five independent production session telemetry datasets collected across Linux and Windows environments confirm that all unmodified sessions are truncated or compacted around 240K–276K tokens (centered on the 256,000 constant):

* **1\. Eddy Murias (@murias002) Full-Database Audit Across 310 Sessions and 65,613 Model Calls (Issue \#878) \[3\]:** Auditing every gen\_metadata record across \~/.gemini/antigravity-cli/conversations/\*.db on Linux between August 30 and October 4, 2026 (310 sessions, 65,613 calls, 86.6% on Gemini 3.8 Flash) showed a median (p50) input of 136,487 tokens, p90 of 229,000 tokens, an all-time maximum single-call input of 255,999 tokens, and 0 calls exceeding 300,000 input tokens. Across all 276 compaction events (\>40% single-step token drop), the median pre-compaction context size was 255,070 tokens (p10 \= 247,360, p90 \= 255,874), dropping to a post-compaction median of 45,304 tokens.  
* **2\. @Burntgogi Trajectory Measurements of a 94-Turn Gemini 4 Argon Refactoring Session (Issue \#1245) \[1\]:** In a 50-minute continuous multi-stage TypeScript refactoring session on agy 1.3.5 under Windows 10 x64 (94 PLANNER\_RESPONSE turns, 264 steps, 10.38M total prompt tokens, 9.43M cached read tokens), two forced compactions and one mid-run prefix cache miss occurred at the exact boundaries shown in the table below.  
* **3\. @NuCl34R Controlled Environment Variable Override Experiment (Issue \#1245 Comment \#1) \[1\]:** On Linux x64 agy 1.3.5 (gemini-4-argon), sessions without an environment variable compacted at 269,597 and 274,270 tokens, while sessions exporting AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE=900000 compacted at 272,784 and 279,687 tokens, confirming the environment variable had no effect in 1.3.5.  
* **4\. @Emily2040 Audit Across 15 Headless Run Logs and 1,279 Gemini 4 Argon High Calls (Issue \#1245 & \#1291) \[1\]\[4\]:** Across 1,279 model calls on agy 1.4.0–1.4.2, 22 checkpoint auto-compaction steps fired; on the step immediately preceding every compaction, total prompt context (input \+ cache\_read) fell strictly between 235K and 268K tokens.  
* **5\. Local Pre-Patch Antigravity CLI Database Audit Across 209 Sessions and 15,575 1M-Gemini Calls (\< 2026-11-04 00:00:00 UTC+8):** Across all 237 local CLI SQLite databases modified prior to November 4, 2026 (containing 209 active 1M-context Gemini sessions and 15,575 model calls: 11,110 on Gemini 3.8 Flash, 2,826 on Gemini 4 Argon, and 1,639 on Gemini 3.7/3.6/3.5 Flash), 100.0% of calls (15,575 / 15,575) recorded max\_context\_tokens \= 256,000, 0 single-attempt calls (retries \= 0\) exceeded 300,000 prompt tokens (p50 \= 108,493, p90 \= 211,227, p99 \= 250,610), with 79 CORTEX\_STEP\_TYPE\_CHECKPOINT (step\_type \= 23\) compaction steps and 52 truncation\_reason \= 2 (MAX\_TOKEN\_LIMIT) events.

The table below summarizes the key trajectory events measured by @Burntgogi in Issue \#1245 during the 94-turn Gemini 4 Argon session \[1\]:

| Event Node | Step / Turn | Pre-Event Prompt Tokens (Cached \+ Input) | Post-Event Prompt Tokens | Post-Event cache\_read\_tokens | Observed Mechanism & Impact |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Checkpoint 1 | Step 79 (Turn 21\) | 276,110 (263,524 cached \+ 12,586 in) | 17,336 | 0 | Crossed \~256K threshold; triggered first forced compaction and zeroed prompt cache \[1\] |
| Mid-run Prefix Miss | Step 193 (Turn 67\) | 239,857 (237,239 cached \+ 2,618 in) | 243,064 | 0 | Full prefix cache invalidation 25 seconds after Step 191 (243,064 tokens billed uncached) \[1\] |
| Checkpoint 2 | Step 200 (Turn 70\) | 249,867 (247,369 cached \+ 2,498 in) | 15,363 | 0 | Reached \~250K; triggered second forced compaction, replacing 93.8% of context with prose summary \[1\] |

## **2.2 Antigravity CLI SQLite Architecture, Protobuf Schema & Compaction Boundary Replay**

Antigravity CLI persists session data locally under \~/.gemini/antigravity-cli/ across two SQLite tiers: (1) a global session index (conversation\_summaries.db, containing the conversation\_summaries table and the conversations\_fts FTS5 virtual table); and (2) per-session trajectory databases (conversations/\<uuid\>.db, each containing 7 tables: trajectory\_meta, steps, gen\_metadata, executor\_metadata, parent\_references, trajectory\_metadata\_blob, and battle\_mode\_infos).

In the steps table, CORTEX\_STEP\_TYPE\_PLANNER\_RESPONSE \= 15 represents a model planning turn and CORTEX\_STEP\_TYPE\_CHECKPOINT \= 23 represents an automatic context compaction checkpoint. In the gen\_metadata table, each row stores a serialized exa.cortex\_pb.GeneratorMetadata Protobuf record for a model call. When background compaction fires (step\_type \= 23), the client inserts a row into steps without writing a corresponding row into gen\_metadata, which is why the summarization call's token usage is omitted from trajectory statistics. Extracted directly from the embedded google.protobuf.FileDescriptorProto tables in agy (file offsets 0x66298e2–0x662a950 and 0x65d62cd), the Protobuf hierarchy inside gen\_metadata.data is:

`exa.cortex_pb.GeneratorMetadata`  
 `├── field 1: chat_model (exa.cortex_pb.ChatModelMetadata)`  
 `│    ├── field 3: model (exa.codeium_common_pb.Model enum)`  
 `│    ├── field 4: usage (exa.codeium_common_pb.ModelUsageStats)`  
 `│    │    ├── field 2: input_tokens (uint64)          # Uncached prompt input tokens billed`  
 `│    │    ├── field 3: output_tokens (uint64)         # Generated output tokens`  
 `│    │    ├── field 4: cache_write_tokens (uint64)    # Tokens written to prompt cache`  
 `│    │    ├── field 5: cache_read_tokens (uint64)     # Cached prefix tokens read`  
 `│    │    └── field 9: thinking_output_tokens (uint64)`  
 `│    ├── field 9: chat_start_metadata (exa.cortex_pb.ChatStartMetadata)`  
 `│    │    ├── field 1: start_step_index (uint32)      # 0 before compaction; jumps to checkpoint idx after`  
 `│    │    ├── field 2: checkpoint_index (int32)       # -1 before compaction`  
 `│    │    ├── field 3: steps_covered_by_checkpoint (repeated uint32)`  
 `│    │    ├── field 4: created_at (google.protobuf.Timestamp)`  
 `│    │    └── field 10: context_window_metadata (exa.cortex_pb.ContextWindowMetadata)`  
 `│    │         ├── field 1: estimated_tokens_used (int32)`  
 `│    │         ├── field 2: truncation_reason (0=UNSPECIFIED, 1=CACHE_TTL_EXPIRED, 2=MAX_TOKEN_LIMIT)`  
 `│    │         ├── field 3: token_breakdown (exa.cortex_pb.TokenBreakdown)`  
 `│    │         └── field 4: max_context_tokens (optional int32)  # Recorded as 256000 on 100% of 1M Gemini calls`  
 `│    ├── field 14: retries (uint32)`  
 `│    ├── field 19: response_model (string)`  
 `│    └── field 21: model_display_name (string)`  
 `└── field 2: step_indices (repeated uint32)`

The table below replays a representative compaction boundary from local pre-patch CLI session 5681d8b4-f18f-4efa-be2b-35091d09b953.db (gen\_idx 338..346). On gen\_idx \= 343, estimated\_tokens\_used reaches 255,912 (88 tokens below the 256,000 ceiling). At step 700, CORTEX\_STEP\_TYPE\_CHECKPOINT (step\_type \= 23\) fires without writing a gen\_metadata row. On the next call (gen\_idx \= 344, steps 701–702), start\_step\_index jumps from 0 to 700, truncation\_reason is recorded as 2 (MAX\_TOKEN\_LIMIT), estimated\_tokens\_used drops to 35,148, and cache\_read\_tokens falls from 267,056 to 20,399:

| gen\_idx | steps.idx | input\_tokens | cache\_read\_tokens | prompt\_total | estimated\_tokens\_used | start\_step\_index | truncation\_reason | max\_context\_tokens |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 338 | 688, 689 | 5,264 | 263,016 | 268,280 | 253,367 | 0 (ckpt=-1) | 0 | 256,000 |
| 340 | 692, 693 | 5,728 | 263,000 | 268,728 | 253,770 | 0 (ckpt=-1) | 0 | 256,000 |
| 342 | 696, 697 | 2,220 | 267,033 | 269,253 | 254,211 | 0 (ckpt=-1) | 0 | 256,000 |
| 343 | 698, 699 | 4,942 | 267,056 | 271,998 | 255,912 | 0 (ckpt=-1) | 0 | 256,000 |
| (step 700\) | 700 (type=23) | (omitted) | (omitted) | (omitted) | (CHECKPOINT) | — | — | — |
| 344 | 701, 702 | 5,962 | 20,399 | 26,361 | 35,148 | 700 (ckpt=0) | 2 (MAX\_TOKEN\_LIMIT) | 256,000 |
| 345 | 703, 704 | 3,662 | 24,476 | 28,138 | 36,719 | 700 (ckpt=0) | 0 | 256,000 |
| 346 | 705, 706 | 4,778 | 24,472 | 29,250 | 37,813 | 700 (ckpt=0) | 0 | 256,000 |

## **2.3 Static Binary Disassembly: Model Registration, Three Hardcoded Gates & Env Var Evolution**

Static disassembly of the official Desktop v2.29.0 language\_server.exe binary (PE32+ x86-64, SHA-256 c98e7c33...e1c3d5) and the official Linux CLI binaries (v1.5.0 BuildID 5fb6af4f... and v1.5.1 BuildID 696031ae...), together with @LiraNuna's disassembly of v1.3.5–v1.4.x in Issue \#1245 \[1\]\[2\], establishes the exact instruction-level implementation across four layers:

### **Layer 1: Model Registration (True 1,048,576 Token Capability Exists in Code)**

In both the desktop and CLI binaries, Gemini 4 Argon's model descriptor is constructed by defaultmodels.Gemini4Argon \-\> defaultmodels.GeminiModelInfo, which writes the maximum context window of 0x100000 (1,048,576 tokens) into struct offset \+0x14 and a 65,536-token (0x10000) output/margin field into struct offset \+0x70, passing family IDs 1417/1418/1419 (0x589/0x58a/0x58b) by thinking level:

`; Desktop language_server.exe (VA 0x141ce5015) & CLI v1.5.0 (VA 0x8540415):`  
`mov DWORD PTR [rax+0x14], 0x100000    ; MaxContext = 1,048,576 tokens (1M)`  
`mov DWORD PTR [rax+0x70], 0x10000     ; Secondary field = 65,536 tokens (64K)`

### **Layer 2 (Gate \#1 & Gate \#2): The Length-1 "Gemini 4 Argon" Whitelist Slice in CLI & Experiment Flag in Desktop**

In the CLI binary (v1.5.0), the string table stores "512K", "1.3x", "900K", "1.8x", and "256K" adjacently around file offset 0x5cf26c0, and the Protobuf message ContextWindowOption includes the boolean accessor (\*ContextWindowOption).GetExpensive. In the /model TUI menu, cycling across Context: Default (256K) \| 256K \| \[512K\] \| 900K via the Tab key is already implemented \[1\]. However, it is blocked by two identical checks against a global whitelist slice resolved via the ELF .rela.dyn relocation table (file offset 0x9fa8):

`Relocation 1: r_offset = 0xc592be0  ->  addend 0xc580ef0  (Slice header at 0xc592be0: {ptr=0xc580ef0, len=1, cap=1})`  
`Relocation 2: r_offset = 0xc580ef0  ->  addend 0x5dcc4d3  (String header at 0xc580ef0: {ptr=0x5dcc4d3, len=14})`  
`Target string at 0x5dcc4d3 (14 bytes): "Gemini 4 Argon"`

This length-1 whitelist slice is consumed at two separate gates in the CLI binary: (1) Gate \#1 (google3/third\_party/jetski/cli/model/model.SelectorModel.offersContextWindow, VA 0x9e92140), which compares the active model name against the slice (@0x9e922ba–0x9e922e0) and returns false (xor %eax, %eax) for Gemini 3.8 Flash and all other non-Argon 1M models, hiding the Context selector chip in /model; and (2) Gate \#2 (google3/third\_party/jetski/cli/store/store.(\*Store).ContextWindow, v1.3.5 offset 0x917d892 / v1.5.0 VA 0x9c49fe0), which repeats the exact same whitelist loop before reading settings.json \[1\]\[2\]:

`; Gate #2: store.(*Store).ContextWindow() — v1.3.5 (@LiraNuna) / v1.5.0 (VA 0x9c49fe0, load @ 0x9c4a032):`  
`917d892: mov  0x279b727(%rip), %rdx   ; Load whitelist slice header at 0xc592be0 ("Gemini 4 Argon")`  
`...`  
`917d913: jl   0x917d952               ; If active model is not in whitelist, jump to exit branch`  
`917d952: xor  %eax, %eax              ; Return empty string pointer "" (VA 0x9c4a0f2 in v1.5.0)`  
`917d954: xor  %ebx, %ebx              ; Return string length 0`  
`917d956: leave`  
`917d957: ret                          ; Returns ("", 0) without reading settings.json`

Because Manager.syncContextWindow() calls store.(\*Store).ContextWindow() rather than the direct configuration reader store.(\*Store).ContextWindowChoice() (VA 0x9c49fa0), any non-Argon model receives backend.SetContextWindow(""), discarding any "contextWindow": "512K" or "900K" value saved in \~/.gemini/antigravity-cli/settings.json \[1\]. On the desktop side, language\_server.exe does not contain the CLI-specific offersContextWindow symbol; instead, model\_config.ContextWindowOptions (VA 0x141242e20) reads options from the server-controlled feature flag key "context-window-options" (string @ 0x1436d7c0a).

### **Layer 3 (Gate \#3): The Hardcoded 0x3E800 (256,000) Clamp in cortex/utils/utils.GetCheckpointConfig**

Even when a user selects Gemini 4 Argon (passing Gates \#1 and \#2), sessions still compact at \~256K tokens. Disassembly of google3/third\_party/jetski/cortex/utils/utils.GetCheckpointConfig (Desktop v2.29.0 VA 0x14176d580, CLI v1.3.5 offset 0x7861894, CLI v1.5.0 VA 0x7fe57fe, CLI v1.5.1 offset 0x7fb0fde) shows that after checking a special-case fast path for internal model family ID 323 (0x143, which receives a 1,000,000 threshold at 0x14176da77 / 0x7fe5bd7) and subtracting a safety margin of max(model\_field, 40,000) (0x9c40), the function executes a 14-byte hardcoded clamp sequence (81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2) \[1\]\[2\]:

`; Desktop v2.29.0 (VA 0x14176d5aa..0x14176d6b0) & CLI v1.5.0 (VA 0x7fe570a..0x7fe5810) / v1.5.1 (0x7fb0fde):`  
`14176d5aa:  cmp   DWORD PTR [rcx+0x8], 0x143    ; Special case check for internal family ID 323 (0x143)`  
`14176d5b1:  je    0x14176da2b                   ; If family == 323 -> jump to 1,000,000 (0xf4240) fast path`  
`...`  
`14176d5dc:  cmp   rdi, 0x9c40                   ; Compare model margin field against 40,000 (0x9c40)`  
`14176d5e9:  cmovg r8, rdi                       ; margin = max(model_field, 40000)  [65,536 for Argon]`  
`14176d5f6:  sub   rdx, r8                       ; candidate = context_window - margin [983,040 for Argon]`  
`...`  
`14176d69e:  mov   edx, DWORD PTR [rsp+0x4c]     ; Load candidate threshold into %edx (CLI v1.5.0 @ 0x7fe57fe)`  
`14176d6a2:  cmp   edx, 0x3e800                  ; Compare %edx against 0x3e800 (256,000) [CLI v1.5.0 @ 0x7fe5802]`  
`14176d6a8:  mov   esi, 0x3e800                  ; Load 256,000 into %esi`  
`14176d6ad:  cmovb esi, edx                      ; Unsigned conditional move: esi = min(candidate, 256000)`  
`14176d6b0:  mov   DWORD PTR [rsp+0x50], esi     ; Write clamped 256,000 into TokenThreshold!`  
`14176d6ba:  cmp   edx, 0x4000                   ; Secondary field clamped against 16,384 (0x4000)`

A whole-section scan across the entire 56 MB .text section of Desktop language\_server.exe and the entire 60 MB .text section of CLI antigravity (v1.5.0 and v1.5.1) confirms that the 14-byte sequence 81 fa 00 e8 03 00 be 00 e8 03 00 0f 42 f2 appears exactly once in each binary. The adjacent function ClampCheckpointConfigForModelInfo (Desktop VA 0x14176dce0) only bounds the threshold by max\_context \- 0x9c40 (40,000) without any 256K constant. Substituting Gemini 4 Argon's registered parameters yields threshold \= min(1,048,576 \- 65,536, 256,000) \= 256,000.

### **Layer 4: Two-Stage Evolution of AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE & Compaction vs. Pre-Flight Truncation**

In google3/third\_party/jetski/cli/store/store.AutoCompactOverride, the function was written to parse a percentage (e.g., 80%) or an integer token count from the undocumented environment variable AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE. Across release versions, its implementation underwent two distinct stages:

* **Stage 1 (CLI v1.3.5–v1.4.x, October 2026 — Compiled Out to Empty String) \[1\]\[2\]:** As disassembled by @LiraNuna in Issue \#1245 Comment \#5, the os.Getenv("AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE") call was replaced at compile time with register-zeroing instructions (xor %eax, %eax; xor %ebx, %ebx; call strings.TrimSpace), passing an empty string "" to strconv.ParseInt and unconditionally returning no override.  
* **Stage 2 (CLI v1.5.0–v1.5.1, November 4–5, 2026 — Silently Restored in Binary) \[11\]:** In CLI v1.5.0 (VA 0x9c49d00) and v1.5.1, Google restored the lookup sequence: lea rax, \[rip+-\>"AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE"\] (string @ 0x5ec74c5, len 29\) \+ mov ebx, 0x1d \+ call os.LookupEnv (0x6b21260) \+ call strings.TrimSpace (0x6b50780), followed by the percentage/integer parser at VA 0x9c49d40 (invoked by Store.ContextWindow at 0x9c4a0fa when no window choice is saved). However, this change was not mentioned in CHANGELOG.md, and the 0x3E800 clamp in GetCheckpointConfig was left intact. In v1.5.1, both calls to the parser pass a base of 0 (xor %eax,%eax before the calls at VA 0x9c34c9a and 0x9e875aa), so the percentage form is ignored and only absolute token counts apply; Store.ContextWindow reads the variable only after the model whitelist passes and no tier is saved in settings.json. Whether a value above 256,000 survives the clamp has not been verified.  
* **Distinction Between Lossy Compaction and Silent Pre-Flight Truncation (@Circumsized, Issue \#1245) \[1\]:** Database records and CLI v1.4.9 release notes confirm two separate context-reduction mechanisms: (a) Compaction (CORTEX\_STEP\_TYPE\_CHECKPOINT \= 23), which replaces \~250K tokens of history with a 15K–17K prose summary marked by CHECKPOINT: "\# Resuming from a compaction..."; and (b) Pre-flight truncation (truncation\_reason \= 2), which trims earlier history from the front of the outgoing request payload as tokens approach max\_context\_tokens \= 256000 without generating a summary or checkpoint banner.

# **3\. Engineering Impact, 70-Second Compaction Stalls & Gemini 4 Argon Quota Consumption**

## **3.1 Context Loss & Post-Compaction Regressions in Multi-Step Engineering**

When a multi-step engineering session crosses the \~256K threshold and undergoes lossy prose compaction (\>93% token reduction) or silent pre-flight trimming, exact symbol definitions, numeric scales, and cross-file constraints read during earlier turns are discarded. Public issues and forum reports document four representative failure modes:

* **Cross-File Symbol Hallucination & 200x Scale Distortion (@Burntgogi, Issue \#1245) \[1\]:** In the 94-turn TypeScript refactoring session on Gemini 4 Argon, the agent read catalog and state-initialization files via view\_file during Steps 1–75. After Checkpoint 1 fired at Step 79 (276,110 \-\> 17,336 tokens), all verbatim view\_file outputs were replaced by a prose summary. When writing upgrade mappings at Steps 159–177 without re-reading the files, the model hallucinated a non-existent tier-3 catalog key and mistook state fields initialized on a 1000..2000 scale for a normalized 0..10 scale (shrinking values by 200x). After Checkpoint 2 at Step 200 (249,867 \-\> 15,363 tokens), it used a hyphenated localStorage key in the UI module and an underscored key in the entry module.  
* **Post-Compaction Identifier Fabrication (@Hello\_there, Forum \#189476) \[43\]:** After hitting "Resuming from a compaction — you have lost access to the full conversation history," the absence of a post-compaction file re-verification check caused the agent to introduce 6 non-existent identifiers and overwrite initialization constraints with hardcoded values. Official staff (@chunduriv) advised adding manual rules in \~/.gemini/GEMINI.md instructing the agent to re-run view\_file after compaction.  
* **Multi-Segment Context Loss (@jlukic & @Infinnerty, Issue \#878) \[3\]:** Across sessions undergoing multiple compactions, each new summary condenses only the already-truncated \~100K context from the preceding segment, leaving Segment 3 with zero access to Segment 1 details, and causing summaries to cite non-existent function names or line numbers beyond file ends.  
* **API Contract Mismatches & Verification Gaps (@Pitt\_zhang, Forum \#176775) \[46\]:** In an audit of a multi-step full-stack task, context loss across turns resulted in the frontend calling /portal-api/\*\* and /mock-channel/\*\* while the backend exposed /api/\*\* with incompatible DTO schemas, alongside a login fallback branch that granted local admin state when an API request failed.

## **3.2 Prompt Cache Invalidation & 70-Second Compaction Overhead**

In GitHub Issues \#1291 and \#1245, @Emily2040 analyzed 15 headless stream-json logs on agy v1.4.0–v1.4.2 running Gemini 4 Argon High (1,279 model calls totaling 10,619,129 uncached input tokens and 192,374,637 cached read tokens) \[1\]\[4\]. Because Gemini 4 Argon relies on byte-identical prefix matching via Implicit Caching (where cached reads cost 1/20th of uncached input, i.e., \$0.30/1M vs. \$6.00/1M) \[4\]\[42\], premature compaction and mid-run prefix changes significantly increase token and quota consumption:

* **70-Second Median Compaction Stall & Unreported Summarization Usage \[1\]\[4\]:** Across the 22 measured checkpoint steps (all triggered between 235K and 268K tokens), each compaction paused execution for 35 to 602 seconds, with a median duration of 70 seconds. Because step\_type \= 23 writes no row to gen\_metadata, the summarization call reports zero token usage in user-visible logs while deducting from the user's subscription quota pool \[4\]\[42\].  
* **Total Prompt Cache Wipeout After Every Compaction \[1\]\[4\]\[5\]:** Rewriting the \~250K conversation prefix into a 15K–17K summary invalidates the entire cached prefix (cache\_read\_tokens drops to 0). Every checkpoint in the 15 logs was immediately followed by a 44,990 to 70,041 token uncached input call to rebuild the cache, followed by additional file re-reads as the agent recovered lost context.  
* **Mid-Run Prefix Cache Misses After system\_message Injection (31.0% of Uncached Input) \[4\]:** Out of 1,279 calls, just 17 calls with input \> 100K and cache\_read\_tokens \= 0 (113,735 to 268,032 tokens each, or 1.3% of total calls) consumed 3,292,077 uncached input tokens—accounting for 31.0% of all uncached input across the entire workload. Fifteen of those 17 misses occurred within two steps of a client-injected system\_message step (such as a subagent notification or timer wakeup).

## **3.3 Controlled A/B Quota Benchmarks (@Yiji\_Kong, Forum \#196351) & Tier Multipliers**

On October 28–29, 2026, developer @Yiji\_Kong published two sets of controlled A/B experiments in Google AI Developers Forum Thread \#196351 across Antigravity Desktop (v2.27.0, Google AI Ultra 5x plan) and the Gemini Web interface to measure internal subscription quota deduction rates \[42\].

### **Experiment 1: In-Client Antigravity Same-Bucket, Single-Turn, Zero-Cache A/B Comparison**

By polling quotaInfo.remainingFraction from LanguageServerService/GetUserStatus every 30 seconds and reading exact token counts from GetCascadeTrajectoryGeneratorMetadata after an 8-minute idle period, identical single-turn tool-free prompts (0 cached reads) were sent to Gemini 4 Argon (High) and Gemini 3.8 Flash (High) \[42\]:

| Model | Input / Output Tokens | 5-Hourly Pool Quota Drop | Implied Cost at Published API List Price | Quota Drop per \$1 of Published API Cost | Normalized Quota Ratio vs. Flash |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Gemini 4 Argon (High) | 14,918 in / 4,233 out | 0.00144340 | \$0.0722 USD (at \$2.00 / \$10.00 per 1M) | 0.02000 | 3.00x (0.02000 / 0.00667) \[42\] |
| Gemini 3.8 Flash (High) | 15,111 in / 3,451 out | 0.00016180 | \$0.0243 USD (at \$0.75 / \$3.75 per 1M) | 0.00667 | 1.00x (Baseline) \[42\] |

Normalized by each model's published API list price, Gemini 4 Argon consumes exactly 3.00x more subscription quota per published dollar than Gemini 3.8 Flash. If Flash is weighted at its published rate (\$0.75/\$3.75 per 1M), the Ultra 5x 5-hour quota pool corresponds to \~\$150.0 USD (\$0.024274 / 0.00016180), and Argon is weighted internally at \$6.00 input / \$30.00 output per 1M tokens (3x its advertised \$2.00 / \$10.00 API launch rate) \[42\]. Additionally, a single subagent probe routed to gemini-3.8-flash-tiered (internal model enum M322, 12,773 input / 16,332 output, 0 cache) dropped remainingFraction by 0.01048544—roughly 22x the expected drop (\~0.00047) for standard Flash \[42\].

### **Experiment 2: Gemini Web 8-Decimal Weekly Pool (ΔWeekly) Pure-Input, Zero-Thinking Cross-Verification**

To eliminate local client accounting variables and thinking-token overhead, @Yiji\_Kong recorded the 8-decimal-place weekly quota usage (percentageUsed) from the Web /usage endpoint across five test cases on October 29, 2026 \[42\]:

| Test Case & Scenario | Model & Tier | Input / Output Payload | 5-Hourly Pool Delta | Exact Weekly Pool Delta (ΔWeekly, 8 Decimals) | Measured Drain Ratio & Finding |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Test 1A (Pure Input, 0 Thinking, 0 Cache) | Gemini 3.8 Flash (Low) | 51,262 chars (\~13.5K tokens) in / reply "OK" (1 token, 0 thinking) | 0.10 \-\> 0.10 (0.00) | \+0.00003685 (0.09984048 \-\> 0.09987733) | Baseline 1.00x \[42\] |
| Test 1B (Pure Input, 0 Thinking, 0 Cache) | Gemini 4 Argon (Low) | 51,262 chars (equal-length Perm B) in / reply "OK" (1 token, 0 thinking) | 0.10 \-\> 0.12 (+0.02) | \+0.00090908 (0.09987733 \-\> 0.10078641) | 24.67x raw drain vs. Flash (9.25x per published dollar; 20.37x raw / 7.64x per dollar net of base overhead) \[42\] |
| Test 2A (High-Thinking 4,013-Word Essay) | Gemini 3.8 Flash (High) | 107 chars in / 28,389 chars (4,013-word Paxos/Raft article) out | 0.12 \-\> 0.12 (0.00) | \+0.00014157 (0.10078641 \-\> 0.10092798) | Generating a 4,013-word technical article used 0.014% of weekly pool \[42\] |
| Test 2B (High-Thinking 4,284-Word Essay) | Gemini 4 Argon (High) | 107 chars in / 27,859 chars (4,284-word Paxos/Raft article) out | 0.12 \-\> 0.15 (+0.03) | \+0.00157077 (0.10092798 \-\> 0.10249875) | 11.10x raw drain vs. Flash (4.24x per published dollar; \~3.1% of 5-hourly pool in one turn) \[42\] |
| Test 1C (Minimal 28-Char Base Calibration) | Gemini 4 Argon (Low) | 28 chars "Reply with only the word: OK" in / reply "OK" | 0.15 \-\> 0.16 (+0.01) | \+0.00015849 (0.10249875 \-\> 0.10265724) | Sending a 28-char "OK" prompt on Argon Low (+0.00015849) used 12% more quota than writing a 4,013-word essay on 3.8 Flash High (+0.00014157) \[42\] |

In Post \#6 of Thread \#196351, Google maintainer @Maniraj\_Chervirala responded that the per-token rates on https\://ai.google.dev/gemini-api/docs/pricing apply only to pay-as-you-go API billing, whereas consumer 5-hour and weekly subscription limits reflect available capacity, model compute, and total agent work, adding that "Fixed per-model quota multipliers are not published because subscription limits adjust dynamically with system capacity" \[42\].

### **Gated Context Window Multipliers (1.0x / 1.3x / 1.8x) & Recent Releases (v1.4.3–v1.5.1 / Desktop v2.29.0)**

Binary string tables and Protobuf descriptors in CLI v1.3.5–v1.5.1 and Desktop v2.27.0–v2.29.0 bind the three built-in context window options to explicit per-turn quota multipliers: Default (256K) / 256K at 1.0x, 512K at 1.3x ("about 1.3x quota per turn"), and 900K at 1.8x with Expensive: true \[1\]\[18\]\[19\]. Between October 20 and November 5, 2026, Google shipped CLI v1.4.3–v1.4.10 to patch compaction side effects (retaining subagent IDs in v1.4.3, retaining tool declarations for cache reuse in v1.4.4, retaining images/PDFs in v1.4.6, fixing oversized-message compaction loops in v1.4.7, fixing orphaned tool calls after history trimming in v1.4.9, and retaining the last 10 parent messages in subagent summaries in v1.4.10) \[11\], added cortex/tokens/compaction, cortex/shared/snapshot, and the gated enable-context-usage-indicator in Desktop v2.29.0 (+4.47 MB language\_server.exe) \[17\], and shipped CLI v1.5.0 and v1.5.1 on November 4–5 while leaving the 0x3E800 (256,000) clamp in GetCheckpointConfig unchanged \[11\].

# **4\. The agy \-p Headless Mode Policy Reversal & Account Suspension Appeals**

## **4.1 Contractual Text Background: Section 6 of the Additional Terms of Service vs. the Official FAQ**

Under Sections 1 and 2 of the Google Antigravity Additional Terms of Service (https\://antigravity.google/terms), the legally binding Agreement between Google LLC and the user consists of three documents: (1) the Google Terms of Service, (2) the Google Antigravity Additional Terms of Service, and (3) the Google Privacy Policy \[14\]. The website FAQ page is not enumerated among the binding contractual documents. Section 6 of the Additional Terms of Service (unmodified since September 2026\) reads in full:

*"6. You must not abuse, harm, interfere with, or disrupt the Service. This includes, but is not limited to, using the Service in connection with products not provided by us. Using third party software, tools, or services to access the Service (e.g. using OpenClaw with Antigravity OAuth) is a breach of this Agreement. Such actions may be grounds for suspension or termination of your Antigravity and/or Gemini CLI accounts."*

*— Google Antigravity Additional Terms of Service, Section 6 (September 2026\) \[14\] ([Source Link](https://antigravity.google/terms))*

Because the official agy CLI provides a documented headless execution mode (agy \-p "\<prompt\>", \--output-format json, \--output-format stream-json) designed for external processes to invoke over standard input/output (stdio), developers inquired on the official support forum (discuss.ai.google.dev/c/antigravity/64) whether invoking the unmodified agy binary locally as a subprocess complied with Section 6\. Around mid-October 2026, without modifying Section 6 of the Additional Terms of Service, Google added the following entry to the Official FAQ page (https\://antigravity.google/docs/faq/) \[13\]\[36\]:

*"Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?*

*Antigravity CLI headless mode (agy \-p) is intended for direct scripting and CI workflows. Wrapping agy inside third-party coding agents or applications, or automating account rotation, is not supported under consumer plans."*

*— Google Antigravity Official FAQ (Updated Mid-October 2026\) \[13\] ([Source Link](https://antigravity.google/docs/faq/))*

## **4.2 Phase I (Sept 16 – Oct 8, 2026, 16:23 UTC): Ten Consecutive Written Confirmations by Google Staff**

Between September 16, 2026 and October 8, 2026 at 16:23 UTC, three official Google support engineers (@Rebbanpalli\_Naveen, @Ambati\_Rajendra, and @chunduriv) issued 10 consecutive written confirmations across independent forum threads stating that invoking the unmodified agy / agy.exe binary locally as a child process—whether from an open-source tool, a personal orchestrator, OpenAI Codex, or Anthropic Claude Code—complied with Section 6 provided OAuth credentials were not extracted or proxied \[23\]\[24\]\[26\]\[27\]\[28\]\[29\]\[30\]\[31\]\[32\]\[33\]. The table below records all 10 confirmations in chronological order:

| \# | Forum Thread ID & Title | Reply Timestamp (UTC) | Inquirer & Scenario | Google Staff Member | Verbatim Core Official Ruling Excerpt | Cite |
| :---- | :---- | :---- | :---- | :---- | :---- | :---- |
| 1 | \#175905 — Is local IPC/PTY wrapping of agy CLI for open-standard editor bridges (e.g., ACP) ToS compliant? | 2026-09-16 12:45:42 | @shindgew: Open-source agy-acp wrapping unmodified agy via local single-user child process/PTY for Zed/Neovim | @Rebbanpalli\_Naveen | "Yes, your described pattern is compliant. The ToS restriction on third-party access tools is aimed at credential harvesting, unofficial API reverse-engineering, and multi-tenant proxying. Spawning the official, unmodified agy binary locally as a single-user child process/PTY under the user’s own agy login session complies..." | \[23\] |
| 2 | \#183429 — Is it allowed for an open-source tool to call the official agy CLI in headless mode? | 2026-09-17 12:06:16 | @darkest\_alex: Open-source Python tool sprite-gen locally calling agy \--output-format json \-p | @Ambati\_Rajendra | "Is calling agy headless allowed? Yes. Calling the official agy binary locally via agy \-p is an officially supported feature built for programmatic scripting... It does not violate Section 6, provided your tool never extracts, stores, or intercepts the user’s OAuth credentials." | \[24\] |
| 3 | \#185494 — Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? | 2026-09-28 07:14:31 | @Diggy\_Styon: Under Google AI Pro, another AI coding assistant launches a script running agy \-p (including in-plan Claude models) | @Ambati\_Rajendra | "Yes, your use case is permitted. Running the official agy binary locally (even headless via a script) under your own account is compliant with the terms... As long as you are executing the unmodified agy CLI and not proxying your auth tokens into another application, you are good to go\!" | \[27\] |
| 4 | \#185460 — One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? | 2026-09-28 12:22:17 | @Winez: Windows 11 local single-user orchestrator invoking unmodified agy as a subprocess | @Ambati\_Rajendra | "While we permit running the official agy CLI locally as a subprocess for your own personal use (as long as you are using your single personal account and interacting with it directly), using an orchestrator to cycle through multiple Google AI Pro accounts... is a direct violation..." | \[26\] |
| 5 | \#185992 — Is invoking the official agy CLI from a local AI coding assistant allowed on an individual Google AI account? | 2026-09-30 23:25:25 | @Ivan\_Valenzuela: Anthropic's Claude Code spawning agy.exe as a child process on Windows and reading/writing stdio JSON | @chunduriv | "Is invoking the unmodified agy.exe binary in headless mode from a local assistant permitted on an individual account? Yes. Section 6... prohibits extracting, proxying, or reusing Antigravity OAuth credentials inside third-party harnesses to call Antigravity’s backend endpoints directly." | \[28\] |
| 6 | \#185949 — Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper | 2026-10-02 20:20:25 | @nothing2: Codex CLI or Claude Code \-\> local Skill \-\> scripts/invoke.ps1 \-\> official unmodified agy CLI | @Ambati\_Rajendra | "Short answer: yes, your setup is fine. ... Codex or Claude calling the Skill: Yes, as long as it sends agy a real task (like a code review or research prompt) and agy runs with your own local login." | \[29\] |
| 7 | \#186954 — Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? | 2026-10-05 10:17:42 | @local\_ai\_builder: Local orchestrator automatically spawning unmodified agy as a child process | @Rebbanpalli\_Naveen | "Yes, the single-user local workflow you described is supported through the official CLI Headless Mode. Local Child Process vs. Section 6: Yes, this is permitted and materially different." | \[30\] |
| 8 | \#186952 — Is invoking the official Antigravity CLI from Codex allowed? | 2026-10-05 10:24:45 | @Hossy: OpenAI Codex on Windows PC invoking unmodified agy.exe as a local process via stdin/stdout | @Rebbanpalli\_Naveen | "Yes, the workflow you described is completely fine to use with your personal Google AI Pro subscription. ... Having Codex invoke the official, unmodified agy.exe binary as a local process on your Windows PC—while authentication (agy auth login) and all backend communication stay strictly inside agy.exe—is a supported use of CLI Headless Mode." | \[31\] |
| 9 | \#186246 — Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? | 2026-10-07 15:03:46 | @Croixofwar: Claude Code or custom local coordinator spawning unmodified agy as a subprocess | @chunduriv | "1. Is launching the official agy CLI as a local subprocess allowed under Section 6? Yes. ... 2\. Does the answer change if you launch the script yourself vs. Claude Code launching it? No. From a policy standpoint, there is no difference whether the unmodified agy subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code)..." | \[32\] |
| 10 | \#187536 — Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator | 2026-10-08 16:23:34 | @BuddhaRyu: Private Linux server orchestrating automated task handoffs between agy, OpenAI Codex, and Claude Code | @Ambati\_Rajendra | "Running agy from your orchestrator: Yes. ... Handoffs between agy, Codex and Claude Code: Yes. Passing results like diffs or review notes between tools in your own project is fine... Does section 6 apply? No. Only the official agy program talks to Antigravity, under your own sign-in..." | \[33\] |

## **4.3 Phase II (Oct 8, 2026, 17:34 UTC – Nov 2, 2026): Policy Reversal, FAQ Update & Expanded Restrictions**

On October 8, 2026 at 17:34:41 UTC—71 minutes after Confirmation \#10 in Thread \#187536 and 26 hours 31 minutes after @chunduriv's own confirmation in Thread \#186246—@chunduriv replied in Thread \#187241 to annual Google AI Pro subscriber @ALAMDAR\_BALOCH (who asked the same question regarding invoking agy from Codex) with a legal disclaimer and a reversal of the prior guidance \[34\]:

* **1\. Initial Reversal & Suspension Warning (\#187241, Oct 8, 2026 at 17:34 UTC) \[34\]:** @chunduriv stated that while agy offers a documented headless mode (agy \-p) for a user's own local scripts and CI pipelines, "we do not support using third-party coding agents (like Codex, Claude Code, OpenClaw, or OpenCode) to orchestrate or wrap agy under a consumer login. Routing third-party agent prompts or automated loops through Antigravity can trigger automated ToS enforcement and account suspension."  
* **2\. Declaring That the FAQ Supersedes Prior Staff Replies (\#189905, Oct 16, 2026\) \[35\]:** When developer @Amane2418 pointed out the conflict between the website FAQ and prior staff replies such as \#185992, @Rebbanpalli\_Naveen replied: "The Antigravity FAQ reflects our current policy for Antigravity CLI headless mode and supersedes earlier forum comments where they differ," adding that even invoking the unmodified agy binary as a local subprocess without extracting OAuth tokens violates the Terms of Service if wrapped inside Claude Code, Codex, OpenClaw, or OpenCode.  
* **3\. Confirmation That the Formal ToS Was Not Modified (\#191689, Oct 23–27, 2026\) \[36\]:** In Thread \#191689, after paying subscribers @eliavb, @b\_w (Posts \#4 & \#6), and @Tora39 (Post \#7) cited \#186952 and \#186246 and asked for evidence of any Terms of Service modification after October 5, @Rebbanpalli\_Naveen confirmed in Post \#8 (Oct 27, 12:31 UTC): "the Google Antigravity Terms of Service themselves were not modified after September; rather, the Official FAQ was updated in mid-October 2026 to add the section 'Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?' ... Earlier forum replies in \#186246 and \#186952 gave an overly broad interpretation before that FAQ entry was published."  
* **4\. Expansion to Local File Task Handoffs, Standalone Python Coordinators & Unattended Scripts (\#190815, \#196472, \#190756, Oct 28–29, 2026\) \[38\]\[39\]\[40\]:** In subsequent threads, Google staff ruled that: (a) having Claude Code write task files into a local folder for a standalone Python script to pass to agy \-p "still counts as third-party agent coordination" and requires a paid Gemini API key or Gemini Enterprise (\#190815) \[38\]; (b) launching agy \-p from an independent local Python task-coordination service is unsupported under consumer logins (\#196472) \[40\]; and (c) unattended or event-driven background monitoring scripts invoking agy \-p under consumer plans are unsupported even with local rate limiting and may trigger automated suspensions (\#190756) \[39\].

## **4.4 OS-Level Detection Boundaries, Protocol Discrepancies & Account Appeal Records**

Both static binary inspection of the client executables and developer commentary on the official forum highlight three technical and operational issues surrounding the enforcement of the headless policy:

* **1\. Absence of Client-Side Detection & OS-Level Indistinguishability \[35\]\[39\]:** Whole-binary string and symbol audits of both Desktop language\_server.exe (v2.29.0) and CLI antigravity (v1.5.0/v1.5.1) confirm 0 occurrences of CLAUDECODE / CLAUDE\_CODE environment variable checks, parent-process inspection, or non-TTY anti-automation checks (all 13 BOT\_CHECK\_\* strings belong to browser CAPTCHA handling instructions, and Getppid / isatty are standard Go runtime symbols). As developer @oli\_hlmpn noted in Thread \#190756, at the OS level agy only sees bash, powershell, or python3 as its parent process and cannot distinguish whether a prompt string or local .md file was written by a human or assisted by another tool \[39\], nor does the CLI display any warning when invoked \[35\].  
* **2\. Policy Discrepancy Between agy acp (stdio Permitted) and agy \-p (stdio Prohibited) \[11\]\[39\]\[41\]:** In Forum Thread \#197131 (Nov 2, 2026\) and CLI v1.5.0 (Nov 4, 2026), Google officially permits third-party clients and editors (such as Zed, Neovim, and T3 Code) to spawn agy acp over stdio using personal subscription OAuth sign-in (oauth-personal) \[11\]\[41\], whereas spawning agy \-p \--output-format stream-json over stdio under the same personal subscription is classified as a ToS violation.  
* **3\. Backend Enforcement at cloudcode-pa.googleapis.com & 100+ Day Unresolved Appeals \[8\]\[9\]\[10\]\[45\]:** Account suspensions are enforced at the cloudcode-pa.googleapis.com gateway via HTTP 403 PERMISSION\_DENIED (reason: TOS\_VIOLATION), which simultaneously disables Antigravity Desktop, Antigravity IDE, agy CLI, @google/gemini-cli, and Gemini Code Assist on the user's Google account \[8\]\[9\]\[10\]. Although Discussion \#20632 stated that first-time flags would be automatically reinstated within 1–2 days after submitting the recertification form \[9\], GitHub Issue \#927 (@jlwainwright, suspended since May 25, 2026 for 100+ days; Google One Case 2-9245000040533) and Issue \#29576 / Forum \#186089 (@myessaylab1234) document that reinstatement sync has failed to restore access for extended periods, while both Google One Support and Google Developer Profile Support state that ToS suspensions are outside their scope \[8\]\[10\]\[45\].

# **5\. Verbatim Quotes & Primary Records Vault**

This section compiles the verbatim primary excerpts from GitHub Issues \#1245, \#878, and \#1291 and the Google AI Developers Forum threads examined in this report, together with their exact timestamps and source URLs.

## **5.1 Verbatim Quotes on the 256K (0x3E800) Binary Clamp, 70s Compaction & 3x Quota Ratio**

### **Quote 1.1: @LiraNuna on Binary Gate \#1 and Compile-Time Removal of AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE (2026-10-22)**

*"In the /model menu, there is already full UI and backend support for choosing the compaction limit: Context: Default (256K) \| 256K \| \[512K\] \| 900K (cycled with Tab). However, in SelectorModel.offersContextWindow: The client checks the selected model against an internal whitelist slice. In the release binary, this slice is hardcoded to only contain "Gemini 4 Argon". For other models (e.g. Gemini 3.8 Flash), offersContextWindow returns false, which completely hides the Context chip and causes the Tab key handler to be ignored.*

*Why AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE does nothing: In store.AutoCompactOverride(), the code was designed to parse either percentages (e.g. 80%) or integer token counts. However, in the compiled release binaries, the call to os.Getenv("AGY\_CLI\_AUTO\_COMPACT\_OVERRIDE") was compiled out / inlined into an empty string "" (xor %eax, %eax; xor %ebx, %ebx; call strings.TrimSpace). Because strconv.ParseInt("", 10, ...) always fails, AutoCompactOverride() unconditionally returns an empty string (no override)."*

*— @LiraNuna, GitHub google-antigravity/antigravity-cli Issue \#1245 Comment \#5, 2026-10-22 16:37:11 UTC \[1\] ([Source Link](https://github.com/google-antigravity/antigravity-cli/issues/1245))*

### **Quote 1.2: @LiraNuna on Binary Gate \#2 and Cortex Engine's 0x3E800 (256,000) Assembly Clamp (2026-10-23)**

*"1. Store.ContextWindow() drops the selection for non-Argon models: Even though the UI saves "contextWindow": "512K" into settings.json, store.(\*Store).ContextWindow() contains a second, identical whitelist check against "Gemini 4 Argon". For all other models (e.g. Gemini 3.8 Flash), it aborts and returns ("", 0\) without ever reading the saved setting... Consequently, Manager.syncContextWindow() calls backend.SetContextWindow(""), and the backend never receives the 512K/900K selection.*

*2\. Cortex has a hardcoded 256K (0x3e800) ceiling on compaction triggers: Even with the backend receiving the context window limit, Cortex's checkpoint calculator (cortex/utils/utils.GetCheckpointConfig) explicitly clamps the compaction trigger (TokenThreshold) for conversational cascades:*

*7861894: mov 0x4c(%rsp), %edx \# Calculated threshold (context \- 40,000 margin)*

*7861898: cmp \$0x3e800, %edx \# 0x3e800 \== 256,000 tokens*

*786189e: mov \$0x3e800, %esi*

*78618a3: cmovb %edx, %esi \# Clamps to min(calculated, 256000\)*

*78618a6: mov %esi, 0x50(%rsp) \# Writes 256,000 to TokenThreshold\!*

*Because of cmovb, any calculated threshold greater than 256,000 tokens is clamped down to 256,000 tokens."*

*— @LiraNuna, GitHub google-antigravity/antigravity-cli Issue \#1245 Comment \#6, 2026-10-23 01:37:57 UTC \[1\] ([Source Link](https://github.com/google-antigravity/antigravity-cli/issues/1245))*

### **Quote 1.3: @Circumsized on SQLite max\_context\_tokens \= 256000 and Compaction vs. Pre-Flight Truncation (2026-10-15)**

*"Every model call writes a ContextWindowMetadata record into the conversation database at \~/.gemini/antigravity-cli/conversations/\<session-id\>.db. On agy 1.3.5 with Gemini 4 Argon, the records for calls immediately preceding the token drops in this report read: max\_context\_tokens \= 256000, truncation\_reason \= 2, while /context reports a 1.0M window. So this is not the server rejecting anything, and it is not the model's window: it is a number the client writes into its own request metadata before sending. 256,000 is exactly 25% of 1,048,576 — a round fraction of the window, which points at a policy constant rather than a technical ceiling.*

*There appear to be two distinct mechanisms, and only one of them is compaction:*

*\- Compaction — history is replaced by a prose summary, with the CHECKPOINT: "\# Resuming from a compaction..." marker. This is the lossy case described in the report.*

*\- Pre-flight truncation — as the conversation approaches max\_context\_tokens, earlier history is trimmed out of the outgoing request before it is sent. No summary is produced, no checkpoint marker appears, and the exact content is simply absent from that request onward."*

*— @Circumsized, GitHub google-antigravity/antigravity-cli Issue \#1245 Comment \#2, 2026-10-15 19:35:43 UTC \[1\] ([Source Link](https://github.com/google-antigravity/antigravity-cli/issues/1245))*

### **Quote 1.4: Eddy Murias (@murias002) on 310 Conversations and 65,613 Model Calls (2026-10-05)**

*"Conversations: 310 \| Model calls: 65,613 (86.6% on Gemini 3.8 Flash) \| Input tokens per call: p50 136,487 · p90 229,000 · max 255,999 \| Calls above 300k input tokens: 0 \| Compaction events (drop \> 40%): 276 \| Context right before compaction: median 255,070 (p10 247,360 · p90 255,874) \| Context right after compaction: median 45,304.*

*So on a model that accepts about 1M tokens, the session is always summarized at roughly 256k, and nothing in \~/.gemini/antigravity-cli/settings.json, the CLI flags or the environment changes it."*

*— Eddy Murias (@murias002), GitHub google-antigravity/antigravity-cli Issue \#878 Comment \#2, 2026-10-05 00:51:10 UTC \[3\] ([Source Link](https://github.com/google-antigravity/antigravity-cli/issues/878))*

### **Quote 1.5: @Emily2040 on 1,279 Model Calls, 22 Checkpoints, and 70-Second Median Compaction Stalls (2026-10-19)**

*"Data: across 15 headless stream-json logs on 1.4.0–1.4.2 with Gemini 4 Argon High (1,279 model calls), 22 checkpoint steps fired, every one with the preceding call's context between 235k and 268k tokens (input \+ cache\_read), never higher. That is consistent with the 256,000 constant read out of the request metadata above. Each checkpoint took a median 70 s (35–224 s, one 602 s), reported no usage for the summary call, and was followed by a 45–70k uncached call before the cache rebuilt."*

*— @Emily2040, GitHub google-antigravity/antigravity-cli Issue \#1245 Comment \#4 & Issue \#1291, 2026-10-19 \[1\]\[4\] ([Source Link](https://github.com/google-antigravity/antigravity-cli/issues/1291))*

### **Quote 1.6: @Yiji\_Kong on 3.0x Argon Quota Drain & @Maniraj\_Chervirala Official Response (2026-10-28 & 10-29)**

*"Normalized by each model’s published API price, Argon consumes exactly 3.0x more quota per dollar than Flash. If Flash is billed at its published rate, the implied bucket size is \~150 USD, and Argon’s consumption matches 6.00 / 30.00 USD per 1M input/output tokens (3x the advertised 2.00 / 10.00 launch price)... Context compaction model calls do not appear in the trajectory token metadata, so their quota cost is invisible to users.*

*In Step 1C, sending just "Reply with only the word: OK" (28 chars) in a fresh chat on 4 Argon (Low) burned \+0.00015849 of weekly quota (\~0.32% of the 5-hour pool), which is 12% more than 3.8 Flash (High) writing a full 4,013-word technical article on Paxos and Raft (+0.00014157)."*

*— @Yiji\_Kong, Google AI Developers Forum Thread \#196351 Posts \#1 & \#5, 2026-10-28 to 2026-10-29 \[42\] ([Source Link](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351))*

*"Higher subscription quota consumption on Gemini 4 Argon compared to Gemini 3.8 Flash is expected: promotional per-token rates on https\://ai.google.dev/gemini-api/docs/pricing apply only to pay-as-you-go API billing, whereas 5-hour and weekly subscription limits reflect available capacity, model compute, and total agent work rather than matching API token list prices... Fixed per-model quota multipliers are not published because subscription limits adjust dynamically with system capacity."*

*— Google Staff @Maniraj\_Chervirala, Google AI Developers Forum Thread \#196351 Post \#6, 2026-10-29 \[42\] ([Source Link](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351))*

### **Quote 1.7: u/Lberondroth on the Desktop v2.29.0 Context and Contingent Indicator (2026-11-03)**

*"Resets in... and Your plan's baseline contingent will refresh on... What you get instead is the Context and contingent Indicator, so you can soon watch your contingent drain in high definition, right next to the send button..."*

*— u/Lberondroth, Reddit r/google\_antigravity Thread \#1xn624i, 2026-11-03 \[17\] ([Source Link](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/))*

### **Quote 1.8: @chunduriv Confirming the \~250K Compaction Threshold "Is a Fixed Default and Isn't User-Configurable Yet" (2026-11-04 23:39 UTC)**

*"As you noted, Gemini models in the App, such as Gemini 4 Argon, currently compact automatically at around 250K tokens. That threshold is a fixed default and isn't user-configurable yet. We understand that for workflows that rely on a large, stable, cached starting context, compacting this early costs both cache efficiency and detail.*

*Keep noise out of the context: Exclude build artifacts, dependencies and generated files so more of the \~250K budget goes to the material you actually need. We've passed your request for a user-selectable compaction size to the engineering team for review."*

*— @chunduriv (Google staff), Google AI Developers Forum Thread \#198191 Post \#2, replying to @Sharper, 2026-11-04 23:39 UTC \[57\] ([Source Link](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191))*

This is the first official written reply in the archive that names the \~250K threshold and states that users cannot change it. The figure matches the 0x3E800 (256,000) clamp in GetCheckpointConfig and the max\_context\_tokens \= 256000 SQLite records. The reply classifies the report as a feature request, offers only interim mitigations (manual /compact, workspace reference files, excluding build artifacts), covers the Desktop app only, and gives no timeline.

## **5.2 Verbatim Quotes Prior to the agy \-p Policy Reversal (Confirming Subprocess Compliance)**

### **Quote 2.1: @Ambati\_Rajendra Confirming Open-Source Tools Calling agy \-p Do Not Violate Section 6 (2026-09-17)**

*"Is calling agy headless allowed? Yes. Calling the official agy binary locally via agy \-p is an officially supported feature built for programmatic scripting (see the Headless Mode Documentation). It does not violate Section 6, provided your tool never extracts, stores, or intercepts the user’s OAuth credentials. As noted in the Antigravity FAQ and Terms of Service, the restriction specifically targets unauthorized third-party clients intercepting tokens to proxy private endpoints."*

*— Google Staff @Ambati\_Rajendra, Google AI Developers Forum Thread \#183429 Post \#2, 2026-09-17 12:06:16 UTC \[24\] ([Source Link](https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429))*

### **Quote 2.2: @Ambati\_Rajendra Confirming Launching agy \-p via Script from Another AI Coding Tool Is Permitted (2026-09-28)**

*"Yes, your use case is permitted. Running the official agy binary locally (even headless via a script) under your own account is compliant with the terms. The restriction against third-party software strictly applies to unauthorized tools (like OpenClaw) that intercept or extract your Antigravity OAuth tokens to bypass official clients. As long as you are executing the unmodified agy CLI and not proxying your auth tokens into another application, you are good to go\!"*

*— Google Staff @Ambati\_Rajendra, Google AI Developers Forum Thread \#185494 Post \#2, 2026-09-28 07:14:31 UTC \[27\] ([Source Link](https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494))*

### **Quote 2.3: @Rebbanpalli\_Naveen Confirming Invoking agy.exe from OpenAI Codex Is Officially Supported (2026-10-05)**

*"Yes, the workflow you described is completely fine to use with your personal Google AI Pro subscription. Here are a few helpful details on how this aligns with our policies:*

*Section 6 & Official CLI Usage: Section 6 of the Google Antigravity Additional Terms of Service and our Third-Party Software FAQ are specifically intended to prevent third-party harnesses from extracting, proxying, or reusing Antigravity OAuth credentials to call private backend endpoints directly. Having Codex invoke the official, unmodified agy.exe binary as a local process on your Windows PC—while authentication (agy auth login) and all backend communication stay strictly inside agy.exe—is a supported use of CLI Headless Mode."*

*— Google Staff @Rebbanpalli\_Naveen, Google AI Developers Forum Thread \#186952 Post \#2, 2026-10-05 10:24:45 UTC \[31\] ([Source Link](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952))*

### **Quote 2.4: @chunduriv Stating "There Is No Difference Whether agy Is Started by Your Own Script or Claude Code" (2026-10-07 15:03 UTC)**

*"1. Is launching the official agy CLI as a local subprocess allowed under Section 6? Yes. Running the official, unmodified agy binary locally in headless mode (for example, agy \-p "\<prompt\>" \--output-format json or stream-json) using your own personal Google AI subscription sign-in for your own individual work is permitted.*

*2\. Does the answer change if you launch the script yourself vs. Claude Code launching it? No. From a policy standpoint, there is no difference whether the unmodified agy subprocess is started by your own script, a local coordinator, or another local coding assistant (such as Claude Code), provided that:*

*\- All Antigravity service access happens exclusively through the unmodified official agy binary (or the official Antigravity Python SDK).*

*\- No OAuth tokens, cookies, or credentials are extracted, read, or passed to the external tool.*

*\- The setup is strictly for your own personal, single-user workflow and not shared, pooled, or exposed as a multi-user service."*

*— Google Staff @chunduriv, Google AI Developers Forum Thread \#186246 Post \#2, 2026-10-07 15:03:46 UTC (26.5h prior to reversal) \[32\] ([Source Link](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246))*

## **5.3 Verbatim Quotes After the agy \-p Policy Reversal (FAQ Supersession & ToS Confirmation)**

### **Quote 3.1: @chunduriv Reversing Position 26.5 Hours Later and Warning of Account Suspension (2026-10-08 17:34 UTC)**

*"While we cannot provide formal legal interpretations of the Terms of Service... Having Codex invoke agy CLI: While agy offers a documented headless mode (agy \-p) for your own local scripts and CI pipelines, we do not support using third-party coding agents (like Codex, Claude Code, OpenClaw, or OpenCode) to orchestrate or wrap agy under a consumer login. Routing third-party agent prompts or automated loops through Antigravity can trigger automated ToS enforcement and account suspension."*

*— Google Staff @chunduriv, Google AI Developers Forum Thread \#187241 Post \#2, 2026-10-08 17:34:41 UTC \[34\] ([Source Link](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241))*

### **Quote 3.2: @Rebbanpalli\_Naveen Stating the FAQ Supersedes Earlier Forum Comments (2026-10-16)**

*"Current Official Guidance: The Antigravity FAQ reflects our current policy for Antigravity CLI headless mode and supersedes earlier forum comments where they differ. ... Wrapping agy Inside Third-Party Coding Agents or Applications (Claude Code, Codex, OpenClaw, OpenCode) — Not Supported: No under consumer plans. Even if you invoke the official, unmodified agy binary as a local subprocess without extracting OAuth tokens or reverse-engineering internal endpoints, wrapping agy inside third-party coding agents or applications—or automating account rotation—is not supported under consumer plans (FAQ — Headless mode in third-party coding agents). Using third-party coding agents or tools (such as Claude Code, OpenClaw, or OpenCode) to access Antigravity violates our Terms of Service and can result in account suspension or termination."*

*— Google Staff @Rebbanpalli\_Naveen, Google AI Developers Forum Thread \#189905 Post \#2, 2026-10-16 12:35:16 UTC \[35\] ([Source Link](https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905))*

### **Quote 3.3: Exchange in \#191689 Confirming the Terms of Service Were Not Modified After September (2026-10-26 & 10-27)**

*"Codex vs. OpenClaw & Why the Earlier Reply (\#186952) Differs: There is no difference between Codex and OpenClaw, having either tool invoke agy / agy.exe is not supported under a consumer Google Account login. After that October 5 reply in \#186952 was posted, the policy on using Antigravity CLI headless mode in third-party coding agents was updated to explicitly clarify that wrapping agy inside third-party coding agents or applications is not supported under consumer plans. This updated policy supersedes the earlier guidance in \#186952 and applies equally to Codex, OpenClaw, Claude Code, and OpenCode."*

*— Google Staff @Rebbanpalli\_Naveen, Google AI Developers Forum Thread \#191689 Post \#5, 2026-10-26 11:24 UTC \[36\] ([Source Link](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689))*

*"You wrote 'After that October 5 reply in \#186952 was posted, the policy on using Antigravity CLI headless mode in third-party coding agents was updated' \- Can you please point me to the evidence of this? I am unable to find any evidence of a policy update after October 5th. The link only directs me to a frequently asked questions page. The most recent terms of service updates I am able to find are from September."*

*— Subscriber @b\_w, Google AI Developers Forum Thread \#191689 Post \#6, 2026-10-26 20:08 UTC \[36\] ([Source Link](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689))*

*"To clarify the timeline for @b\_w and answer @Tora39’s question (a): the Google Antigravity Terms of Service themselves were not modified after September; rather, the Official FAQ was updated in mid-October 2026 to add the section ‘Can I use Antigravity CLI headless mode in third-party coding agents or automate account switching?’ That FAQ entry reflects the current policy for consumer plans (Free, Plus, Pro, and Ultra)... Earlier forum replies in \#186246 and \#186952 gave an overly broad interpretation before that FAQ entry was published."*

*— Google Staff @Rebbanpalli\_Naveen, Google AI Developers Forum Thread \#191689 Post \#8, 2026-10-27 12:31 UTC \[36\] ([Source Link](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689))*

### **Quote 3.4: @Ambati\_Rajendra Ruling That Passing Tasks via Local Disk Files Is Also Prohibited (2026-10-28)**

*"No. Per the FAQ and Additional Terms, using agy \-p as a secondary worker coordinated by a third-party coding agent like Claude Code isn’t permitted on consumer plans... That still counts as third-party agent coordination. Passing tasks from Claude Code to agy \-p through local files instead of stdin/stdout pipes still uses consumer quota as a backend worker for an external coding agent, so it requires a Gemini API key or Gemini Enterprise."*

*— Google Staff @Ambati\_Rajendra, Google AI Developers Forum Thread \#190815 Post \#2, 2026-10-28 13:10:46 UTC \[38\] ([Source Link](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815))*

# **6\. Analyzed Official Binaries, GitHub Issue Records, Scope Boundaries & Full Reference Sources**

## **6.1 Analyzed Official Binaries & Cryptographic Hashes**

No binary executables are hosted in the archive repository. All original binaries examined in this report can be downloaded directly from Google's official cloud storage (storage.googleapis.com/antigravity-public) and verified against the SHA-256 and SHA-512 digests below:

| Component | Version / Platform | Official Download URL / Source | File Size & Cryptographic Hash |
| :---- | :---- | :---- | :---- |
| Desktop Installer | v2.29.0 (Windows x64) | https\://storage.googleapis.com/antigravity-public/antigravity-hub/2.29.0-5723181030440960/windows-x64/Antigravity-x64.exe | 169,295,640 bytes \| SHA-256: 944b109bcb377f9f94f86eeea7cfa9315d4b42d84fb9d5bad689058fc11c12ba |
| Desktop Language Server | v2.29.0 (resources/bin/language\_server.exe) | Bundled inside app-64.7z within Antigravity-x64.exe above (PE32+ x86-64, 214,663 pclntab symbols) | 187,342,336 bytes \| SHA-256: c98e7c331da5bcc37002a550e7d625b4c636f775fa2804f73777c6a500e1c3d5 |
| CLI Release Archive (v1.5.0) | v1.5.0 (Linux x64, 2026-11-04) | https\://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.0-5269980577660928/linux-x64/cli\_linux\_x64.tar.gz | Archive: 69,781,910 bytes (antigravity binary: 238,604,688 bytes, BuildID 5fb6af4f2e1d489f3ada0fd785060f48) \| SHA-512: e6031f850929210705d8c2393a5665b82b8c3b19684a6d3c556cafa31f26f77acbf164dfee96ca84a5ae7670ac0cc238a974c981c517b269194bdccff271cbd5 |
| CLI Binary (v1.5.1) | v1.5.1 (Linux x64, 2026-11-05) | https\://storage.googleapis.com/antigravity-public/antigravity-cli/1.5.1-4523430503120896/linux-x64/cli_linux_x64.tar.gz (listed in the official linux\_amd64.json manifest) | Archive 72,209,583 bytes, SHA-512: f70406654ebd966f211424338259e2d38d099c48734e901dc06975063e5a0137d55b6eb9366b75983c6908bbc20ae92a2617656bb191d8ed6fe429d654bca4f5 \| Extracted binary 240,857,488 bytes (BuildID 696031ae34e5e5e26a9d7c436e85317f), SHA-256: 72738cf0445918483956a5f0da33abdfcfb21a12dd9a421fd57d063782826669 |
| CLI Install Script & Manifest | Cross-platform / Linux amd64 | https\://antigravity.google/cli/install.sh \| https\://antigravity-cli-auto-updater-974169037036.us-central1.run.app/manifests/linux\_amd64.json | install.sh: 7,643 bytes, SHA-256 62966c07365423bd4dc209355060744058fb30d60f5323e2d360e39de64e5042 (referenced, not redistributed); manifest archived in evidence/raw/manifest\_linux\_amd64.json |

## **6.2 Public GitHub Issue Records (google-antigravity/antigravity-cli)**

The table below lists all 10 related GitHub Issues in google-antigravity/antigravity-cli verified via the GitHub API (gh api) and archived in evidence/raw/:

| Issue \# | Title | Author & Date Filed | State & Assignee | Key Technical Contributions & Commenters |
| :---- | :---- | :---- | :---- | :---- |
| \#1245 | \[Bug\]: Premature \~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache | @Burntgogi (2026-10-14) | open \| @chandrashakherkamasani (labeled feedback, no bug label) | 94-turn Argon telemetry (@Burntgogi); env override test (@NuCl34R); SQLite max\_context\_tokens=256000 & pre-flight truncation (@Circumsized); 1,279-call / 70s checkpoint audit (@Emily2040); 3 binary gates disassembly & patch\_agy.py (@LiraNuna); AUR package (@AsabaLazy). 22+ thumbs-up. |
| \#878 | \[Feature Request\]: Disable automatic context auto-compaction and unlock full 1M context window | @schalatap (2026-08-26) | open \| @manirajc (labeled feature request) | 310-session / 65,613-call SQLite audit showing max input 255,999 and 0 calls \>300K (@murias002); AntigravityCompactionConfig.token\_threshold inspection (@Corsagee); 1.3.5 headless compaction logs (@Infinnerty); multi-segment context loss (@jlukic). 22 thumbs-up. |
| \#1371 | \[Feature Request\] User-configurable auto-compaction threshold in the desktop app (\~256K hardcoded while the model window is 1M) | @floveey (2026-10-30) | open \| Official reply by @manirajc | Reports \~256K compaction in Desktop app; @manirajc redirected Desktop/IDE requests to the Google AI Developers Forum; @LiraNuna linked \#1245. |
| \#1291 | \[Bug\]: Five measured places the CLI spends time and tokens the work does not need | @Emily2040 (2026-10-19) | open \| @chandrashakherkamasani (labeled feature request, feedback) | Quantifies 31.0% uncached input from 17 mid-run prefix misses (15 after system\_message), 70s median compactions across 22 checkpoints, 11.3–12.2s headless startup/teardown overhead, and summary store rewrites every 1.5–3.3s. |
| \#1072 | \[Feature Request\]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest | @flotoz (2026-09-20) | open \| @manirajc | Documents \>3x faster-than-expected quota burn on Gemini 3.8 Flash Low; cross-referenced by @Emily2040. |
| \#743 | Feature Request: Add max\_context\_tokens setting to control per-turn token budget | @thealgebraist (2026-08-03) | open \| @manirajc | Documents a 379-step session (610 KB transcript) where \~70% of tokens were redundant compiler error outputs. |
| \#927 | agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync | @jlwainwright (2026-09-02) | open \| @manirajc | Documents HTTP 403 PERMISSION\_DENIED persisting 100+ days since May 25, 2026 (Google One Case 2-9245000040533). |
| \#1316, \#1372, \#1367, \#919 | Subagent compaction controls (\#1316); Active warning before degradation (\#1372); /goal loop burning 37M tokens (\#1367); Trajectory loops (\#919) | @aeronmiles, @zhangr930103, @thhart, @ozl | open | Corroborate headless subagent compaction controls, pre-compaction warnings, and runaway token loops in /goal mode. |

## **6.3 Scope Boundaries & Unverified Items**

To maintain strict evidentiary clarity between verified client-side/database facts and unverified external claims, the following boundaries are explicitly noted:

* **1\. Server-Side Enforcement of 512K/900K Tiers:** While the client binaries contain the 512K (1.3x) and 900K (1.8x, Expensive: true) strings, Protobuf fields, and UI selectors, actual server-side handling and billing multipliers for unlocked 512K/900K requests cannot be verified from client binaries alone.  
* **2\. Absence of Client-Side Subprocess Ban Detectors:** Neither language\_server.exe nor agy contains CLAUDECODE / CLAUDE\_CODE environment checks, parent-process inspection, or non-TTY anti-automation checks. Enforcement of TOS\_VIOLATION occurs at the cloudcode-pa.googleapis.com backend gateway.  
* **3\. Desktop Electron Frontend (app.asar):** Static binary verification on the desktop focused on resources/bin/language\_server.exe (187,342,336 bytes); frontend Electron bundle strings (such as enable-context-usage-indicator reported in community ASAR diffs \[17\]\[18\]) are cited to their respective community teardown reports.  
* **4\. Reproduction Commands:** All binary findings can be reproduced on Linux x86-64 using standard tools: sha256sum / sha512sum for binary integrity, python3 (stdlib struct and sqlite3 in mode=ro) for ELF/PE section and Go pclntab parsing, data.find(bytes.fromhex('81fa00e80300be00e803000f42f2')) for locating the 0x3E800 clamp, and objdump \-d \-M intel for instruction disassembly.

## **6.4 Complete Reference Sources Index (\[1\]–\[57\] & \[S1\]–\[S9\])**

Below is the complete list of all 57 primary cited sources (\[1\]–\[57\]) and 9 supplementary verification sources (\[S1\]–\[S9\]) from the archive:

**\[1\] [\[Bug\]: Premature \~250K context compaction on 1M-context models drops exact symbol/schema context and invalidates prompt cache (Issue \#1245)](https://github.com/google-antigravity/antigravity-cli/issues/1245)** — GitHub google-antigravity/antigravity-cli (@Burntgogi, @NuCl34R, @Circumsized, @Emily2040, @LiraNuna, @AsabaLazy; assignee: @chandrashakherkamasani) — 2026-10-14 to 2026-11-02

**\[2\] [Patch agy to unlock 512K/900K context window selection and fix early compaction (patch\_agy.py)](https://gist.github.com/LiraNuna/4c058aed3f636f5193f920f64284ff26)** — GitHub Gist (@LiraNuna) — 2026-10-23

**\[3\] [\[Feature Request\]: Disable automatic context auto-compaction and unlock full 1M context window (Issue \#878)](https://github.com/google-antigravity/antigravity-cli/issues/878)** — GitHub google-antigravity/antigravity-cli (@schalatap, @bash99, @murias002, @Corsagee, @Infinnerty, @jlukic; assignee: @manirajc) — 2026-08-26 to 2026-10-31

**\[4\] [\[Bug\]: Five measured places the CLI spends time and tokens the work does not need: serial tool calls, uncached full-context calls and 70 s compactions, 12 s headless overhead, summary store rewritten every 1.5 s, build-time CPU instruction choice (Issue \#1291)](https://github.com/google-antigravity/antigravity-cli/issues/1291)** — GitHub google-antigravity/antigravity-cli (@Emily2040; assignee: @chandrashakherkamasani) — 2026-10-19 to 2026-10-25

**\[5\] [\[Feature Request\]: Lack of aggressive context compaction / prompt caching burns the 5h and weekly quota far faster than token cost would suggest (Issue \#1072)](https://github.com/google-antigravity/antigravity-cli/issues/1072)** — GitHub google-antigravity/antigravity-cli (@flotoz, @Emily2040; assignee: @manirajc) — 2026-09-20 to 2026-10-19

**\[6\] [Feature Request: Add max\_context\_tokens setting to control per-turn token budget (Issue \#743)](https://github.com/google-antigravity/antigravity-cli/issues/743)** — GitHub google-antigravity/antigravity-cli (@thealgebraist; assignee: @manirajc) — 2026-08-03

**\[7\] [Feature request: Support Gemini CLI-compatible OTLP/OpenTelemetry token usage export (Issue \#366)](https://github.com/google-antigravity/antigravity-cli/issues/366)** — GitHub google-antigravity/antigravity-cli (@UesugiErii, @Mandark-droid, @evilfurryone, @rajatim, @Emily2040) — 2026-06-11 to 2026-10-20

**\[8\] [agy 1.1.23: 403 ToS eligibility block persists 100+ days after appeal form, no reinstatement sync (Issue \#927)](https://github.com/google-antigravity/antigravity-cli/issues/927)** — GitHub google-antigravity/antigravity-cli (@jlwainwright; assignee: @manirajc) — 2026-09-02

**\[9\] [Addressing Antigravity Bans & Reinstating Access (Discussion \#20632)](https://github.com/google-gemini/gemini-cli/discussions/20632)** — GitHub google-gemini/gemini-cli (Google Maintainer @jackwotherspoon) — 2026-02-27

**\[10\] [Appeal submitted but TOS\_VIOLATION suspension persists (request for private follow-up) (Issue \#29576)](https://github.com/google-gemini/gemini-cli/issues/29576)** — GitHub google-gemini/gemini-cli (@myessaylab1234) — 2026-09-30

**\[11\] [Google Antigravity CLI Official Releases & Changelog (v1.4.2 – v1.5.1)](https://github.com/google-antigravity/antigravity-cli/blob/main/CHANGELOG.md)** — GitHub google-antigravity/antigravity-cli — 2026-10-18 to 2026-11-05

**\[12\] [Google Antigravity Docs — Changelog (v2.0.1 – v2.29.0)](https://antigravity.google/docs/changelog)** — Google Antigravity Official Documentation — 2026-11-03

**\[13\] [Google Antigravity Docs — Official FAQ](https://antigravity.google/docs/faq/)** — Google Antigravity Official Documentation — Updated Mid-October 2026

**\[14\] [Google Antigravity — Additional Terms of Service](https://antigravity.google/terms)** — Google LLC — Published September 2026

**\[15\] [Google Antigravity Docs — Workspace Rules (24 KB Per-File Limit & 20,000-Token Aggregate Rules Budget)](https://antigravity.google/docs/rules/)** — Google Antigravity Official Documentation — 2026-10

**\[16\] [Optimizing a 200,000-line Python codebase overnight with Gemini 4 Argon](https://developers.googleblog.com/optimizing-a-200000-line-python-codebase-overnight-with-gemini-4-argon/)** — Google Developers Blog — 2026-10

**\[17\] [\[Teardown / Binary Diff\] Antigravity Updated to v2.29.0](https://www.reddit.com/r/google_antigravity/comments/1xn624i/teardown_binary_diff_antigravity_updated_to_v2290/)** — Reddit r/google\_antigravity (u/Lberondroth) — 2026-11-03

**\[18\] [\[Deep-Dive\] Antigravity v2.27.0 \-\> v2.28.1 Binary & ASAR Diff: 'Script Jail', the oneharness Migration, Teamwork Leaving Preview, and What's Coming Next](https://www.reddit.com/r/google_antigravity/comments/1xij7jx/deepdive_antigravity_v2270_v2281_binary_asar_diff/)** — Reddit r/google\_antigravity (u/Lberondroth) — 2026-10-28

**\[19\] [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon? (r/AntigravityGoogle)](https://www.reddit.com/r/AntigravityGoogle/comments/1xmc5pk/timeline_for_enabling_the_512k_900k/)** — Reddit r/AntigravityGoogle — 2026-11-02

**\[20\] [Timeline for enabling the 512k / 900k ContextWindowOptions and lifting the 256k autocompact clamp on Argon? (r/google\_antigravity)](https://www.reddit.com/r/google_antigravity/comments/1xmc89l/timeline_for_enabling_the_512k_900k/)** — Reddit r/google\_antigravity — 2026-11-02

**\[21\] [Antigravity's aggressive history compaction is ruining Gemini's 1M+ context selling point](https://www.reddit.com/r/google_antigravity/comments/1trr4dy/antigravitys_aggressive_history_compaction_is/)** — Reddit r/google\_antigravity — 2026-05-30

**\[22\] [How to force AG to use subagents? (Orchestrator \+ Ephemeral Subagents)](https://www.reddit.com/r/google_antigravity/comments/1uo7g2k/how_to_force_ag_to_use_subagents/)** — Reddit r/google\_antigravity — 2026-07

**\[23\] [Clarification Request: Is local IPC/PTY wrapping of agy CLI for open-standard editor bridges (e.g., ACP) ToS compliant? (\#175905)](https://discuss.ai.google.dev/t/clarification-request-is-local-ipc-pty-wrapping-of-agy-cli-for-open-standard-editor-bridges-e-g-acp-tos-compliant/175905)** — Google AI Developers Forum (@shindgew; Official Reply: @Rebbanpalli\_Naveen) — 2026-07-23 to 2026-09-16

**\[24\] [Is it allowed for an open-source tool to call the official agy CLI in headless mode? (\#183429)](https://discuss.ai.google.dev/t/is-it-allowed-for-an-open-source-tool-to-call-the-official-agy-cli-in-headless-mode/183429)** — Google AI Developers Forum (@darkest\_alex; Official Reply: @Ambati\_Rajendra) — 2026-09-17

**\[25\] [Is using the official agy CLI through a local MCP server with third-party AI agents permitted? (\#184829)](https://discuss.ai.google.dev/t/is-using-the-official-agy-cli-through-a-local-mcp-server-with-third-party-ai-agents-permitted/184829)** — Google AI Developers Forum (@snowstone; Official Reply: @Ambati\_Rajendra) — 2026-09-25

**\[26\] [One person, several paid Google AI Pro accounts: may a local orchestrator use them with the official agy CLI? (\#185460)](https://discuss.ai.google.dev/t/one-person-several-paid-google-ai-pro-accounts-may-a-local-orchestrator-use-them-with-the-official-agy-cli/185460)** — Google AI Developers Forum (@Winez; Official Reply: @Ambati\_Rajendra) — 2026-09-28

**\[27\] [Headless agy runs launched by a script from another AI coding tool: within the Additional Terms? (\#185494)](https://discuss.ai.google.dev/t/headless-agy-runs-launched-by-a-script-from-another-ai-coding-tool-within-the-additional-terms/185494)** — Google AI Developers Forum (@Diggy\_Styon; Official Reply: @Ambati\_Rajendra) — 2026-09-28

**\[28\] [Is invoking the official agy CLI from a local AI coding assistant allowed on an individual Google AI account? (\#185992)](https://discuss.ai.google.dev/t/is-invoking-the-official-agy-cli-from-a-local-ai-coding-assistant-allowed-on-an-individual-google-ai-account/185992)** — Google AI Developers Forum (@Ivan\_Valenzuela; Official Reply: @chunduriv) — 2026-09-29 to 2026-10-01

**\[29\] [Clarification on Codex/Claude Skill invoking the official agy CLI through a local PowerShell wrapper (\#185949)](https://discuss.ai.google.dev/t/clarification-on-codex-claude-skill-invoking-the-official-agy-cli-through-a-local-powershell-wrapper/185949)** — Google AI Developers Forum (@nothing2; Official Reply: @Ambati\_Rajendra) — 2026-09-29 to 2026-10-02

**\[30\] [Is automatically launching the official agy CLI from a local AI orchestrator allowed under Additional Terms §6? (\#186954)](https://discuss.ai.google.dev/t/is-automatically-launching-the-official-agy-cli-from-a-local-ai-orchestrator-allowed-under-additional-terms-6/186954)** — Google AI Developers Forum (@local\_ai\_builder; Official Reply: @Rebbanpalli\_Naveen) — 2026-10-05

**\[31\] [Is invoking the official Antigravity CLI from Codex allowed? (\#186952)](https://discuss.ai.google.dev/t/is-invoking-the-official-antigravity-cli-from-codex-allowed/186952)** — Google AI Developers Forum (@Hossy; Official Reply: @Rebbanpalli\_Naveen) — 2026-10-05

**\[32\] [Can a local AI coordinator launch the official Antigravity CLI with subscription sign-in? (\#186246)](https://discuss.ai.google.dev/t/can-a-local-ai-coordinator-launch-the-official-antigravity-cli-with-subscription-sign-in/186246)** — Google AI Developers Forum (@Croixofwar; Official Reply: @chunduriv) — 2026-10-01 to 2026-11-02

**\[33\] [Terms Clarification: Personal Antigravity CLI Integration with a Multi-AI Orchestrator (\#187536)](https://discuss.ai.google.dev/t/terms-clarification-personal-antigravity-cli-integration-with-a-multi-ai-orchestrator/187536)** — Google AI Developers Forum (@BuddhaRyu; Official Reply: @Ambati\_Rajendra) — 2026-10-08

**\[34\] [Is Google AI Pro Antigravity CLI automation with Codex permitted? (\#187241)](https://discuss.ai.google.dev/t/is-google-ai-pro-antigravity-cli-automation-with-codex-permitted/187241)** — Google AI Developers Forum (@ALAMDAR\_BALOCH; Official Reply: @chunduriv) — 2026-10-07 to 2026-10-08

**\[35\] [Clarification on consumer-plan policy for invoking official agy via local subprocess (\#189905)](https://discuss.ai.google.dev/t/clarification-on-consumer-plan-policy-for-invoking-official-agy-via-local-subprocess/189905)** — Google AI Developers Forum (@Amane2418, @Jay10; Official Reply: @Rebbanpalli\_Naveen) — 2026-10-16 to 2026-10-26

**\[36\] [Paying Google AI Pro subscriber: allow subscription use with third-party agents like OpenClaw (\#191689)](https://discuss.ai.google.dev/t/paying-google-ai-pro-subscriber-allow-subscription-use-with-third-party-agents-like-openclaw/191689)** — Google AI Developers Forum (@eliavb, @b\_w, @Tora39; Official Reply: @Rebbanpalli\_Naveen) — 2026-10-23 to 2026-10-27

**\[37\] [Consumer plan: does my own deterministic CI remain permitted if its test input is prepared by another AI? (\#194998)](https://discuss.ai.google.dev/t/consumer-plan-does-my-own-deterministic-ci-remain-permitted-if-its-test-input-is-prepared-by-another-ai/194998)** — Google AI Developers Forum (@Herzchen; Official Reply: @Rebbanpalli\_Naveen) — 2026-10-26

**\[38\] [Google AI Pro – Is standalone Antigravity CLI scripting with external task coordination permitted? (\#190815)](https://discuss.ai.google.dev/t/google-ai-pro-is-standalone-antigravity-cli-scripting-with-external-task-coordination-permitted/190815)** — Google AI Developers Forum (@Lars; Official Reply: @Ambati\_Rajendra) — 2026-10-19 to 2026-10-28

**\[39\] [Policy clarification: private monitoring app using official Antigravity CLI and subscription quota (\#190756)](https://discuss.ai.google.dev/t/policy-clarification-private-monitoring-app-using-official-antigravity-cli-and-subscription-quota/190756)** — Google AI Developers Forum (@xorz, @oli\_hlmpn; Official Reply: @Ambati\_Rajendra) — 2026-10-19 to 2026-10-29

**\[40\] [Clarification on subscription-based agy \-p integration with a local multi-agent framework (\#196472)](https://discuss.ai.google.dev/t/clarification-on-subscription-based-agy-p-integration-with-a-local-multi-agent-framework/196472)** — Google AI Developers Forum (@Dsssyc; Official Reply: @Rebbanpalli\_Naveen) — 2026-10-28 to 2026-10-29

**\[41\] [Conflicting guidance on using T3 Code with personal Google AI subscriptions (\#197131)](https://discuss.ai.google.dev/t/conflicting-guidance-on-using-t3-code-with-personal-google-ai-subscriptions/197131)** — Google AI Developers Forum (@sannikov, @tkosub, @oli\_hlmpn; Official Reply: @Rebbanpalli\_Naveen) — 2026-10-31 to 2026-11-02

**\[42\] [Gemini 4 Argon draws Antigravity quota at 3x Gemini 3.8 Flash per published dollar (clean A/B, same bucket) (\#196351)](https://discuss.ai.google.dev/t/gemini-4-argon-draws-antigravity-quota-at-3x-gemini-3-8-flash-per-published-dollar-clean-a-b-same-bucket/196351)** — Google AI Developers Forum (@Yiji\_Kong, @RobW, @shuai\_guo; Official Reply: @Maniraj\_Chervirala) — 2026-10-28 to 2026-10-29

**\[43\] [Context Compaction Amnesia and Missing Post-Write Verification Gates (\#189476)](https://discuss.ai.google.dev/t/context-compaction-amnesia-and-missing-post-write-verification-gates/189476)** — Google AI Developers Forum (@Hello\_there; Official Reply: @chunduriv) — 2026-10-15 to 2026-10-20

**\[44\] [Feature request: Show context window usage in Antigravity Desktop (\#184694)](https://discuss.ai.google.dev/t/feature-request-show-context-window-usage-in-antigravity-desktop/184694)** — Google AI Developers Forum (@Soro, @dimitr-io; Official Reply: @chunduriv) — 2026-09-24 to 2026-10-22

**\[45\] [Persistent Gemini CLI TOS\_VIOLATION after appeal — request for private review or sync check (\#186089)](https://discuss.ai.google.dev/t/persistent-gemini-cli-tos-violation-after-appeal-request-for-private-review-or-sync-check/186089)** — Google AI Developers Forum (@myessaylab1234) — 2026-09-30

**\[46\] [Antigravity CLI Agent reliability issue (\#176775)](https://discuss.ai.google.dev/t/antigravity-cli-agent-reliability-issue/176775)** — Google AI Developers Forum (@Pitt\_zhang) — 2026-08-01

**\[47\] [Google Antigravity FAQ Policy Update and Termination of Third-Party Tool Integration](https://note.com/horizon_it00/n/nded285d721d1)** — note.com / horizon\_it00 (Kenji Sato) — 2026-10

**\[48\] [Hermes Agent Plugin Catalog — Antigravity Subscription DirectSDK (239,616 Compaction Threshold)](https://hermes-agent.nousresearch.com/docs/plugins/antigravity-subscription-directsdk)** — Nous Research Hermes Agent Documentation — 2026-09-23

**\[49\] [mksglu/context-mode: MCP Server & FTS5 Sandbox for Token-Efficient Agent Context Management](https://github.com/mksglu/context-mode)** — GitHub (@mksglu) — 2026

**\[50\] [AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor: Real-time Monitoring of Context Window Usage for Antigravity IDE](https://github.com/AGI-is-going-to-arrive/Antigravity-Context-Window-Monitor)** — GitHub (@AGI-is-going-to-arrive) — 2026

**\[51\] [dimitr-io/antigravity-desktop-context: Live /context side panel for the Antigravity 2.0 desktop app](https://github.com/dimitr-io/antigravity-desktop-context)** — GitHub (@dimitr-io) — 2026-10-22

**\[52\] [weby-homelab/antigravity-cli-statusline: Adaptive Statusline for Antigravity CLI](https://github.com/weby-homelab/antigravity-cli-statusline)** — GitHub (Weby Homelab) — 2026

**\[53\] [oraios/serena: Semantic Code Retrieval & Symbol-Level MCP Toolkit](https://github.com/oraios/serena)** — GitHub (Oraios AI) — 2026

**\[54\] [xdotech/goatlas: AI-Powered Code Intelligence Engine (AST, Neo4j Knowledge Graph & pgvector MCP Server)](https://github.com/xdotech/goatlas)** — GitHub (@xdotech) — 2026

**\[55\] [Kevin Hou — "Get Out of the Model's Way" (AI Engineer Summit Presentation & Speaker Profile)](https://www.youtube.com/watch?v=xsVD9_cJNYs)** — AI Engineer Summit / YouTube (Kevin Hou, @kevinhou22) — 2026

**\[56\] [Varun Mohan Education Qualifications: How an Indian-Origin MIT Engineer Secured Google's \$2.4 Billion AI Investment](https://timesofindia.indiatimes.com/education/news/varun-mohan-education-qualifications-how-an-indian-origin-mit-engineer-secured-googles-2-4-billion-ai-investment/articleshow/122405531.cms)** — Times of India — 2025-07

**\[57\] [Enable larger than 250K Gemini context sizes in Antigravity Desktop! (\#198191)](https://discuss.ai.google.dev/t/enable-larger-than-250k-gemini-context-sizes-in-antigravity-desktop/198191)** — Google AI Developers Forum (@Sharper; Official Reply: @chunduriv) — 2026-11-04

### **Supplementary & Cross-Verification Sources (\[S1\]–\[S9\])**

**\[S1\] [Question about personal wrapper around official Antigravity CLI headless mode (\#178472)](https://discuss.ai.google.dev/t/question-about-personal-wrapper-around-official-antigravity-cli-headless-mode/178472)** — Google AI Developers Forum (@Ahmad3) — 2026-08-16

**\[S2\] [Is a local single-user orchestrator allowed to invoke the official agy \-p CLI? (\#179013)](https://discuss.ai.google.dev/t/is-a-local-single-user-orchestrator-allowed-to-invoke-the-official-agy-p-cli/179013)** — Google AI Developers Forum (@JJPop; Reply: @Ambati\_Rajendra) — 2026-08-20

**\[S3\] [Google AI Studio Gemini API invoking a consumer-authenticated agy CLI is allowed? (\#194944)](https://discuss.ai.google.dev/t/google-ai-studio-gemini-api-invoking-a-consumer-authenticated-agy-cli-is-allowed/194944)** — Google AI Developers Forum (@oooolee; Reply: @Ambati\_Rajendra) — 2026-10-26

**\[S4\] [Can I let my grok bot use agy cli on their computer? (\#190429)](https://discuss.ai.google.dev/t/can-i-let-my-grok-bot-use-agy-cli-on-their-computer/190429)** — Google AI Developers Forum (@Chua\_Poplar; Reply: @Ambati\_Rajendra) — 2026-10-19

**\[S5\] [AsabaLazy/aur-ASLZ: Arch Linux AUR Repository containing antigravity-cli-fix-bin](https://github.com/AsabaLazy/aur-ASLZ)** — GitHub (@AsabaLazy) — 2026-11-02

**\[S6\] [mcpservers.org — sshahzaiib/agy-bridge MCP Server](https://mcpservers.org/ja/servers/sshahzaiib/agy-bridge)** — MCP Servers Directory (sshahzaiib) — 2026

**\[S7\] [Claude Code vs Google Antigravity vs GitHub Copilot Pricing & Quota Comparison](https://www.r-jc.jp/news-blog/6256/claude-code-antigravity-%E6%AF%94%E8%BC%83copilot-%E9%81%95%E3%81%84%E6%96%99%E9%87%91%E8%A1%A8/)** — R-JC News Blog — 2026

**\[S8\] [Agy unusable after false violation of Terms of Service (\#185325)](https://discuss.ai.google.dev/t/agy-unusable-after-false-violation-of-terms-of-service/185325)** — Google AI Developers Forum — 2026-09-27

**\[S9\] [BUG: AGY CLI Error 3501 (403 Forbidden) \- Personal Account Wrongly Flagged as Enterprise (\#185714)](https://discuss.ai.google.dev/t/bug-agy-cli-error-3501-403-forbidden-personal-account-wrongly-flagged-as-enterprise/185714)** — Google AI Developers Forum — 2026-09-28

*Google Antigravity 256K Context Limit & Headless Policy Change Archive  •  Licensed under CC BY 4.0 (Third-party quotes & API snapshots included under Fair Use, 17 U.S.C. § 107\)*

