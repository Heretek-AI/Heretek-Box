Adversarial Architecture Audit & 2026 Ecosystem Stress-Test: Heretek-Box 

  Auditor: Principal Systems Architect & Adversarial Ecosystem Research Group
  Target: Heretek-AI/Heretek-Box & The 50-Agent Fleet Blueprint
  Evaluation Date: October 15, 2026
  Primary Artifact: ADVERSARIAL_ARCHITECTURE_AUDIT_2026.md
  Detailed Subagent Dossiers:
  • Squad A (OpenCode v2 & Agent IPC): OPENCODE_V2_FLEET_STRESS_TEST_REPORT.md
  • Squad B (Sandboxing & Security): sandboxing_security_jailbreak_audit.md
  • Squad C (Reverse Engineering & Pentesting): squad_c_tooling_audit.md
  • Squad D (Game Studio & GUI Automation): squad_d_audit_report.md
  • Squad E (Resource Budget & Fleet Density): SQUAD_E_RESOURCE_BUDGET_AUDIT.md
  ──────
  ## 1. The Red-Team Critique (The "Tear-Down")

  ### A. Fatal Fragilities & Architectural Deadlocks

  1. The OpenCode v2 SQLite Single-Writer Bottleneck:
  In anomalyco/opencode https://github.com/anomalyco/opencode (packages/core/src/database/database.ts), the SQLite database hardcodes PRAGMA busy_timeout = 5000 (5
  seconds). While WAL mode permits concurrent reads, SQLite permits strictly one exclusive writer. A 50-agent fleet generating tool telemetry, execution claims
  (store.claim), and git snapshots generates 15–30 write transactions/sec. Queue delays immediately exceed 5,000ms, crashing sessions with SQLITE_BUSY: database is
  locked.
  2. The-Agency Global Freeze Bug:
  In Heretek-AI/The-Agency https://github.com/Heretek-AI/The-Agency (dispatcher/src/limits.ts & engine.ts), receiving an HTTP 429 or quota limit error triggers pause-
  missions. The dispatcher sets pausedUntil = now + 60s and freezes all missions across all 50 agents globally. It ignores OpenCode Go's Retry-After: 17700 header,
  fires a probe every 60 seconds, hits 429 again, and re-parks in an infinite loop for up to 5 hours.
  3. IPC Socket Exhaustion & SSE Drops:
  The-Agency's OpenCodeClient lacks HTTP Keep-Alive connection pooling, causing high-frequency polling to exhaust ephemeral TCP ports (ECONNRESET / EADDRNOTAVAIL).
  Concurrently, OpenCode's SSE event bus (event-feed.ts) hardcodes SubscriberCapacity = 4_096, dropping agent subscribers with SubscriberOverflowError under event
  bursts.
  4. The gVisor / eBPF Telemetry Blindspot:
  Deploying agent containers under gVisor (runsc) blinds host-level Tetragon eBPF. gVisor's Sentry intercepts syscalls in Go userspace via seccomp-bpf
  (SECCOMP_RET_TRAP). Host kernel syscall tracepoints (sys_enter_execve, sys_enter_openat2) never execute, completely disarming Tetragon's in-kernel SIGKILL
  enforcement.
  5. The Container Escape Vulnerability in SPEC.md:
  In SPEC.md:253-265 Pillar 8, vm-runner was defined with devices: [/dev/kvm], cap_add: [SYS_ADMIN, NET_ADMIN], and security_opt: [seccomp:unconfined]. If an attacker
  or untrusted binary breaks out of Cloud-Hypervisor via a KVM vulnerability, they land in a root container with full host CAP_SYS_ADMIN and unconfined seccomp,
  making host compromise trivial.
  ──────
  ### B. OpenCode Go $12 / 5-Hour Rolling Quota Choke-Point

  Source verification of opencode.ai/zen/go/v1 (packages/console/core/src/subscription.ts) reveals that OpenCode Go enforces a fixed-anchor 5-hour window capped at
  $12.00 (1,200,000,000 microcents). It does not use a sliding leaky bucket; time_rolling_updated anchors at the first call and resets exactly 18,000 seconds later.

  50-Agent Fleet Burnout Timeline (Assumptions: 50 agents, 20k input tokens, 400 output tokens/turn, 15s turn cadence):

  • qwen3.7-max ($2.50 / $7.50 per 1M): Fleet cost = 2.65/turn → Exhausted in 4.5 turns (1.1 minutes / 68 seconds).
  • kimi-k2.7-code ($0.95 / $4.00 per 1M): Fleet cost = 1.03/turn → Exhausted in 11.6 turns (2.9 minutes / 174 seconds).
  • deepseek-v4-pro ($0.435 / $0.87 per 1M): Fleet cost = 0.452/turn → Exhausted in 26.5 turns (6.6 minutes / 398 seconds).
  • Lockout Penalty: When breached, the API throws HTTP 429 Too Many Requests (GoUsageLimitError) with Retry-After: ~17700 (up to 4.9 hours of total lockout).
  • Prefix Cache Destruction: DeepSeek V4 Pro offers a 99.17% prompt cache discount. However, distinct persona system prompts and dynamic timestamps (Date.now(),
  task.id) placed at token 0 break byte-level prefix equality, generating 50 disjoint cache keys and forcing 100% full-rate token billing.
  ──────
  ### C. Vanity Persona Consolidation: 50 Roles Down to 6 Multi-Skilled Agents
  The 50 personas introduce extreme overhead: 2,000–6,000 tokens of serialization tax per hop, 15–45s IPC delay, information degradation through the "telephone game,"
  and prompt cache invalidation.
  │ Diagram exceeds terminal width (601 > 168 cols)
  │ Displayed as code block. Widen terminal to view inline.
    graph TD
        subgraph Dropped_Vanity_Personas ["Purged Redundant Roles"]
            VP1["frontend_dev + css_specialist + nextjs_specialist\n(Merge: Single TSX/Tailwind file ownership)"]
            VP2["rust_engineer + compiler_diagnostic_agent\n(Merge: Agent must see LSP feedback directly)"]
            VP3["director + planner + chief_of_staff + standup_scribe\n(Merge: Gemini 4.0 Argon reasons natively)"]
            VP4["qa_author + qa_judge + diff_reviewer\n(Merge: Eliminates 3x diff re-reading token tax)"]
        end
    
        subgraph Consolidated_Core ["Hardened 6-Role Fleet"]
            R1["1. Chief Orchestrator (Argon / Frontier)\n[Plan, Route, Synthesize, User Approvals]"]
            R2["2. Polyglot Systems Builder (Standard Coder)\n[Rust, TS, Python, Go, LSPs, Tests, AST Refactor]"]
            R3["3. Web & OSINT Scout (Fast / Flash)\n[SearXNG, Crawl4AI, Headless Chromium, Docs]"]
            R4["4. SRE & Invariant Auditor (Review / Standard)\n[Tetragon eBPF, Trivy, Btrfs Snapshots, Diffs]"]
            R5["5. AppSec & RE Lab (On-Demand Systemd Daemon)\n[morluto/rea, Ghidra Headless, radare2, Nuclei]"]
            R6["6. Spatial & Mobile Lab (On-Demand Systemd Daemon)\n[Godot, Blender, Android KVM, Maestro]"]
        end
  ──────
  ## 2. Verified Active 2026 OSS Component Matrix
  Audited against live GitHub repositories and package registries with a strict September / October 2026 cutoff:
   Role / Domain         | Proposed Tool / MCP     | 2026 Maintenance Status (Commit Date / Repo) | Adversarial Verdi… | Rationale & Tradeoffs
  -----------------------|-------------------------|----------------------------------------------|--------------------|----------------------------------------------
   Core Coding Agent     | anomalyco/opencode    | Active (Daily)v1.19.1 released 2026-10-       | KEEP (Core Engine) | Production-grade coding agent. 139 V2 HTTP
                         |                         | 15anomalyco/opencode[1]                      | Engine)           | API endpoints. Must patch SQLite busy_timeout
                         |                         |                                              |                   | and client socket pooling.
   Dispatcher / Control  | Heretek-AI/The-Agency   | Active (Daily)Pushed 2026-10-15 (SwarmOS     | KEEP (Patched)    | Pure DAG scheduler, SQLite ledger, worktrees.
                         |                         | v1.1.0)Heretek-AI/The-Agency[2]              |                    | worktrees. Must patch limits.ts to prevent
                         |                         |                                              |                    | global 429 mission freezing.
   RE Agent Engine       | morluto/rea             | Active (Daily)Pushed 2026-10-15 (266k        | KEEP (Core RE)     | Production RE MCP server connecting Ghidra
                         |                         | stars)morluto/rea[3]                         |                    | 12.1.x, Rizin, GDB. Includes cryptographic
                         |                         |                                              |                    | evidence ledgers and DEBUGINFOD_URLS
                         |                         |                                              |                    | cleansing.
   OpenCode RE Bridge    | Heretek-AI/opencode-rea | ActivePushed 2026-10-12Heretek-AI/opencode-  | KEEP               | Purpose-built OpenCode v2 plugin wiring REA
                         |                         | rea[4]                                       |                    | MCP into static, runtime, and auditor
                         |                         |                                              |                    | subagents.
   Autonomous Pentesting | vxcontrol/pentagi       | ActivePushed 2026-10-09 (30.4k               | DROP / REPLACE     | Severe security risk: Requires mounting
                         |                         | stars)vxcontrol/pentagi[5]                   |                    | /var/run/docker.sock as root:root to run
                         |                         |                                              |                    | worker containers. Open-source edition
                         |                           |                                             |                    | contains zero MCP server code. Replace with
                         |                           |                                              |                    | with direct Argon orchestration of nuclei,
                         |                           |                                              |                    | semgrep, and nmap.
   Ghidra MCP Server      | 0xflux/ghidra-mcp         | STALE (6+ Months)Last push 2026-04-04 (7     | DROP / REPLACE     | Abandoned fork. Replace with native
                          |                           | stars)0xflux/ghidra-mcp[6]                   |                    | morluto/rea Ghidra runner or upstream
                          |                           |                                              |                    | bethington/ghidra-mcp (active Oct 2026,
                          |                           |                                              |                    | 5.6k stars).
   Radare2 MCP Server     | radareorg/radare2-mcp     | Active (Daily)Pushed 2026-10-15 (325         | KEEP (Fast RE)     | Official C implementation. <30MB RAM
                          |                           | stars)radareorg/radare2-mcp[7]               |                    | footprint (vs Ghidra's 2GB+ JVM), sub-
                          |                           |                                              |                    | 10ms startup. Essential for low-overhead
                          |                           |                                              |                    | binary triage.
   Sandboxing Engine      | google/gvisor (runsc)     | Active (Daily)Pushed 2026-10-                | DROP (for          | Breaks Android KVM, breaks nested Podman
                          |                           | 15google/gvisor[8]                           | Dev/RE)REPLACE     | overlayfs, degrades JVM/Ghidra JIT, and
                          |                           |                                              | with Cloud-        | blinds host Tetragon eBPF.
                          |                           |                                              | Hypervisor +       |
                          |                           |                                              | Landlock           |
   Kernel Security        | cilium/tetragon           | Active (Daily)Pushed 2026-10-                | KEEP (In-Guest)    | In-kernel eBPF enforcement with
                          |                           | 13cilium/tetragon[9]                         |                    | synchronous SIGKILL. Must be deployed
                          |                           |                                              |                    | inside the MicroVM guest kernel, not on
                          |                           |                                              |                    | the host.
   Desktop Automation     | agent-sh/computer-use-    | Active (Daily)Pushed 2026-10-12agent-        | KEEP (Core         | Native Rust MCP server supporting Wayland
                          | linux                     | sh/computer-use-linux[10]                    | Desktop)           | (Sway/Hyprland), AT-SPI accessibility
                          |                           |                                              |                    | tree, and kernel uinput/ydotool.
   Container Desktop      | bytebot-ai/bytebot        | ARCHIVED & DEADArchived March 2026bytebot-   | DROP               | Officially archived and abandoned
                          |                           | ai/bytebot[11]                               |                    | upstream. Do not use as base image.
   Desktop Automation     | tanob/mcp-desktop-        | DEAD / UNMAINTAINEDLast push Nov             | DROP               | Unmaintained. RobotJS breaks on Node
                          | automation                | 2025tanob/mcp-desktop-automation[12]         |                    | 20/22+. X11 only, zero Wayland or AT-SPI
                          |                           |                                              |                    | support.
   Godot OpenCode Plugin  | Heretek-AI/godot-opencode | Active (v1.0.0)Pushed 2026-10-12 (123/124    | KEEP (Core Game)   | Production Bun/TypeScript Effect plugin
                          |                           | tests pass)Heretek-AI/godot-opencode[13]     |                    | with GDScript companion addon, loopback
                          |                           |                                              |                    | JSON-RPC, atomic UndoRedo, and GUT
                          |                           |                                              |                    | runner.
   GDevelop Automation    | Heretek-AI/GDevelop-      | Active (v1.0.0)Pushed 2026-10-13 (258/258    | KEEP (Core 2D)     | Complete zero-fork patch stack with
                          | Unlocked                  | tests pass)Heretek-AI/GDevelop-Unlocked[14]  |                    | Electron launcher hook, BYOK AI runtime,
                          |                           |                                              |                    | AST game.json mutation, and live
                          |                           |                                              |                    | debugger.
   Blender MCP Server     | sandraschi/blender-mcp    | ActivePushed Oct 2026sandraschi/blender-     | KEEP / ADOPT       | Headless-first Blender automation (150+
                          |                           | mcp[15]                                      |                    | tools, batch mesh, VSE, VRM). Spawns
                          |                           |                                              |                    | background headless Blender processes on
                          |                           |                                              |                    | demand.
   Android Emulation      | budtmo/docker-android     | Active (v3.8.0)Updated 2026budtmo/docker-    | KEEP               | Android 15 (API 35) support, native MCP
                          |                           | android[16]                                  |                    | server for LLM-driven emulator control.
                          |                           |                                              |                    | Requires /dev/kvm.
   Mobile Test Automation | mobile-dev-inc/maestro    | Active (Daily)Pushed 2026-10-15mobile-dev-   | KEEP (Gold         | Industry standard declarative YAML mobile
                          |                           | inc/maestro[17]                              | Standard)          | automation. Fast, deterministic, zero-
                          |                           |                                              |                    | flakiness.
   Mobile MCP Server      | mobile-next/mobile-mcp    | Active (v1.0.9)Pushed Oct 2026mobile-        | KEEP               | Semantic mobile MCP using Android
                          |                           | next/mobile-mcp[18]                          |                    | accessibility tree dumps
                          |                           |                                              |                    | (mobile_list_elements_on_screen) rather
                          |                           |                                              |                    | than brittle vision.

   [1]: anomalyco/opencode https://github.com/anomalyco/opencode
   [2]: Heretek-AI/The-Agency https://github.com/Heretek-AI/The-Agency
   [3]: morluto/rea https://github.com/morluto/rea
   [4]: Heretek-AI/opencode-rea https://github.com/Heretek-AI/opencode-rea
   [5]: vxcontrol/pentagi https://github.com/vxcontrol/pentagi
   [6]: 0xflux/ghidra-mcp https://github.com/0xflux/ghidra-mcp
   [7]: radareorg/radare2-mcp https://github.com/radareorg/radare2-mcp
   [8]: google/gvisor https://github.com/google/gvisor
   [9]: cilium/tetragon https://github.com/cilium/tetragon
  [10]: agent-sh/computer-use-linux https://github.com/agent-sh/computer-use-linux
  [11]: bytebot-ai/bytebot https://github.com/bytebot-ai/bytebot
  [12]: tanob/mcp-desktop-automation https://github.com/tanob/mcp-desktop-automation
  [13]: Heretek-AI/godot-opencode https://github.com/Heretek-AI/godot-opencode
  [14]: Heretek-AI/GDevelop-Unlocked https://github.com/Heretek-AI/GDevelop-Unlocked
  [15]: sandraschi/blender-mcp https://github.com/sandraschi/blender-mcp
  [16]: budtmo/docker-android https://github.com/budtmo/docker-android
  [17]: mobile-dev-inc/maestro https://github.com/mobile-dev-inc/maestro
  [18]: mobile-next/mobile-mcp https://github.com/mobile-next/mobile-mcp
  ──────
  ## 3. Resolution of the 5 Wave 1 Architectural Dilemmas

  ### 1. Container Density: Ephemeral Worktrees inside MicroVM (Model C)

  • Rejection of 8 Monolith Containers: Sinks 8.6–9.5 GB RAM at idle, bursts to 74.5 GB RAM, and consumes 151 GB of disk (bringing the local 179 GB disk to 99%
  capacity). Triggers fatal OOM kills on 32GB machines and severe swap thrashing on 64GB machines.
  • Rejection of 50 Ephemeral Containers: Adds 2.5–8.0 seconds of cold-start latency per task with extreme container layer maintenance.
  • Binding Verdict: A lean 3-container Compose cluster (gateway, support, vm-runner). Subagent tasks execute in ephemeral Git worktrees
  (/tmp/heretek/worktrees/<task_id>) in <100ms. Language toolchains provision on-demand via devenv.nix and uv from a deduplicated /nix/store. Heavy tools (Android KVM,
  Ghidra, Blender) run as socket-activated systemd user daemons that terminate after 10 minutes of inactivity, capping active RAM at 16–24 GB and disk at 35 GB.

  ### 2. Communication Topology: Hybrid Hub-and-Spoke with Redis Streams

  • Rejection of Pure Hub-and-Spoke: Funneling all tool logs through Director creates context bloat and reasoning bottlenecks.
  • Rejection of Unconstrained Squad Handoffs: Agents form runaway communication loops and lose synchronization with the human-in-the-loop audit log.
  • Binding Verdict: Director acts as the Root DAG Scheduler & Safety Authority. For collaborative tasks (e.g. AppSec scanning Builder's web service), Director spawns
  an ephemeral squad communicating over a private Redis Stream (squad:<task_id>:events). Squad agents exchange typed JSON-RPC events directly with <5ms latency, but
  cannot modify shared branches or commit code without pushing a signed completion event back to Director's SQLite DAG for invariant verification.

  ### 3. OpenCode Go 429 Failover: Circuit-Breaker Matrix via LiteLLM

  When OpenCode Go throws HTTP 429 (GoUsageLimitError), the local proxy catches the error, parses Retry-After, opens the circuit for that model, and seamlessly fails
  over down the chain:

   Tier                     | Primary OpenCode Go Model | Tier 1 Commercial Fallback (OpenRouter/Direct) | Tier 2 Local Airgapped Fallback | Fallback Trigger Condit…
  --------------------------|---------------------------|------------------------------------------------|---------------------------------|--------------------------
   Frontier(Director,       | opencode-go/qwen3.7-      | OpenRouter: anthropic/claude-3.7-sonnet(or     | Local vLLM: Qwen2.5-Coder-32B-  | HTTP 429, 503, or
   Architect)               | max(or Gemini 4.0 Argon)  | Direct Google AI Studio)                       | Instruct(AWQ 4-bit, 20GB VRAM)  | latency > 45s
   Standard(Systems         | opencode-go/kimi-k2.7-    | DeepSeek Direct: deepseek-chat (V3)OpenRouter: | Local vLLM: Qwen2.5-Coder-14B-  | HTTP 429, 500, or
   Builder, Coder)          | code                      | meta-llama/llama-3.3-70b                       | Instruct(8-bit, 16GB VRAM)      | context > 64k
   Fast(Web Scout,          | opencode-go/deepseek-v4-  | DeepSeek Direct: deepseek-chatGroq: llama-3.1- | Local Ollama: qwen2.5-coder:7b- | HTTP 429 or rate limit
   Scrapers)                | flash                     | 8b-instant                                     | instruct-q8_0(CPU/GPU, 8GB RAM) |
   Review(SRE, Invariant    | opencode-go/deepseek-v4-  | OpenRouter: openai/gpt-4oDirect Anthropic:     | Local vLLM: Qwen2.5-Coder-32B-  | HTTP 429 or structured
   Auditor)                 | pro                       | claude-3.5-haiku                               | Instruct                        | schema error

  Canonical Prefix Prompt Rule: Standardize the first 6,000 tokens across all personas (tool manifests and core axioms). Persona specializations append after the
  prefix. This guarantees >90% KV cache hit rates on DeepSeek and Gemini, cutting token burn by up to 10x.

  ### 4. Sandboxing Isolation: 4-Tier Defense-in-Depth

  • Reject gVisor for developer and agent execution: It breaks Android KVM, breaks nested Podman overlayfs, degrades JVM/Ghidra JIT, and blinds host Tetragon eBPF.
  • Harden Cloud-Hypervisor: Strip SYS_ADMIN, NET_ADMIN, and seccomp:unconfined from the VMM container. Run Cloud-Hypervisor as an unprivileged user (heretek-vmm),
  jailed by host Landlock ABI v5 restricting ioctl access strictly to /dev/kvm.
  • Deploy In-Guest Tetragon eBPF: Run Tetragon inside the MicroVM kernel with BTF enabled. This restores 100% syscall visibility and enables instantaneous in-kernel
  SIGKILL on policy violations.
  • Wrap Agent Runtimes in Guest Landlock LSM: Agent execution wrappers enforce Landlock path rules, confining writes strictly to /workspaces/<task_id> and /tmp.

  ### 5. Compose Packaging: Modular Compose with include: & Profiles

  • Rejection of Monolithic Compose: A 500-line single file creates merge conflicts, makes selective component testing impossible, and forces all services into memory
  simultaneously.
  • Binding Verdict: Root docker-compose.yml orchestrates modular service files via include::
    include:
      - compose/core.yml           # Gateway, Caddy SSL, OpenCode Server
      - compose/support.yml        # PostgreSQL 16, Redis 7, SearXNG
      - compose/vm-runner.yml      # Cloud-Hypervisor MicroVM Engine
      - compose/scraper-swarm.yml  # Optional: Crawl4AI, Playwright, Scrapling
      - compose/appsec-lab.yml     # Optional Profile: REA, Ghidra, Nuclei
      - compose/game-studio.yml    # Optional Profile: Godot, Blender, Android

  • Heavy divisions are guarded by profiles (profiles: ["appsec"], profiles: ["game"]). They remain completely dormant until explicitly launched (docker compose --
  profile appsec up -d), keeping cold boot memory under 3.5 GB RAM.
  ──────
  ## 4. The Counter-Proposal Blueprint

  │ Diagram exceeds terminal width (626 > 168 cols)
  │ Displayed as code block. Widen terminal to view inline.

    flowchart TD
        subgraph Host_Machine ["Host Developer Workstation (Linux / macOS / Cloud)"]
            User_UI["Web Dashboard & Chat Gateway\n(Next.js PWA / Discord / Telegram)"]
            LiteLLM["LiteLLM / OmniRoute Proxy\n(Circuit Breaker & Fallback Matrix)"]
        end
    
        subgraph Core_Compose ["Heretek-Box Core Cluster (Docker Compose with include:)"]
            Gateway["OpenClaw Gateway Daemon\n(Rust/Go + Bun Dispatcher)"]
            OpenCode["OpenCode v2 Headless Server\n(opencode serve :4096, WAL SQLite)"]
            Postgres[("PostgreSQL 16\n(Task Ledger & State)")]
            Redis[("Redis 7\n(Task Queues & PubSub)")]
            SearXNG["SearXNG Meta-Search"]
        end
    
        subgraph MicroVM_Runner ["Hardware-Isolated Guest (Cloud-Hypervisor via VSOCK)"]
            Guest_Kernel["Linux Kernel 7.x (BTF, Landlock ABI v5, BPF LSM)"]
            InGuest_Tetragon["In-Guest Tetragon eBPF Daemon\n(Synchronous In-Kernel SIGKILL)"]
            Btrfs_Engine["Btrfs CoW Snapshot Engine\n(<50ms pre-task rollback)"]
            Wayland_Sway["Headless Sway Compositor\n(PipeWire 60fps WebRTC Canvas)"]
            
            subgraph Ephemeral_Workspaces ["Ephemeral Execution Units (Zero Container Overhead)"]
                Agent_1["Task 1: Worktree + Landlock Jail\n(devenv / uv / rustc / pnpm)"]
                Agent_2["Task 2: Worktree + Landlock Jail\n(ast-grep / lsp-mcp / tests)"]
            end
    
            subgraph Socket_Daemons ["On-Demand Socket-Activated Daemons (10m Idle Timeout)"]
                REA_Daemon["morluto/rea + Ghidra 12.1.x Headless"]
                Godot_Daemon["godot-opencode v1.0.0 Headless"]
                GDevelop_Daemon["GDevelop-Unlocked v1.0.0 AST Engine"]
                Android_Daemon["docker-android (API 35 KVM) + Maestro"]
            end
        end
    
        User_UI --> Gateway
        Gateway --> LiteLLM
        LiteLLM --> OpenCode
        Gateway <--> Postgres
        Gateway <--> Redis
        Gateway <--> SearXNG
        Gateway <-->|Linux AF_VSOCK| Guest_Kernel

        Guest_Kernel --> InGuest_Tetragon
        Guest_Kernel --> Btrfs_Engine
        Guest_Kernel --> Wayland_Sway
        Guest_Kernel --> Ephemeral_Workspaces
        Ephemeral_Workspaces -.->|Socket Activation| Socket_Daemons

  ### Summary of Tactical Changes:

  1. Fleet Topology: Consolidate 50 fragmented personas into 6 multi-skilled roles (Chief Orchestrator, Polyglot Systems Builder, Web/OSINT Scout, SRE Invariant
  Auditor, AppSec/RE Lab, Spatial/Mobile Lab).
  2. Quota & Rate Limit Defense: Deploy a local LiteLLM / OmniRoute proxy fronting OpenCode Go with an automatic circuit-breaker fallback to OpenRouter (claude-3.7-
  sonnet, deepseek-chat) and local vLLM (Qwen2.5-Coder-32B). Enforce a 6,000-token canonical prefix prompt to achieve >90% KV caching.
  3. Execution Density: Replace 8 always-running division containers with ephemeral Git worktrees inside a single Cloud-Hypervisor NixOS MicroVM. Toolchains provision
  instantly via devenv and uv from /nix/store.
  4. Sandboxing & Telemetry: Drop gVisor. Isolate the MicroVM via Cloud-Hypervisor run as an unprivileged user jailed by host Landlock ABI v5. Move Tetragon eBPF
  inside the guest kernel to gain 100% syscall visibility and instantaneous in-kernel SIGKILL enforcement.
  5. AppSec & Reverse Engineering: Drop vxcontrol/pentagi (docker.sock security risk) and 0xflux/ghidra-mcp (stale). Adopt morluto/rea and radareorg/radare2-mcp with
  read-only target mounts, airgapped network namespaces, and Tetragon eBPF outbound connection filters.
  6. Game & Desktop Automation: Drop bytebot (archived March 2026) and tanob/mcp-desktop-automation (dead). Adopt agent-sh/computer-use-linux over headless Sway
  (Wayland) with AT-SPI semantic inspection, paired with Heretek-AI/godot-opencode (v1.0.0) and Heretek-AI/GDevelop-Unlocked (v1.0.0).
  7. Packaging: Modularize docker-compose.yml using include: and profile gates (profiles: ["appsec", "game"]), dropping cold-boot memory to 3.2 GB RAM and enabling
  complete operation on any standard 32GB developer workstation.
  ──────
  ### Key Open Decisions for Alignment:

  1. LiteLLM Proxy Placement: Should the LiteLLM fallback proxy run as a container in compose/support.yml or as a native process in the Gateway runtime?
  2. Local Model Hardware Allocation: If enabling the local vLLM/Ollama fallback tier, what GPU VRAM budget (e.g., 16GB vs 24GB VRAM) should be allocated on your host
  workstation?
  3. AppSec / RE Profile Auto-Spawn: Should socket-activated daemons (Ghidra, Android emulator) be spawned dynamically via systemd socket units or started explicitly
  via /profile <name> slash commands in chat?
