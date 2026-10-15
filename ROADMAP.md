# Heretek-Box: Architectural Specification & Master Prompt Roadmap

> **Vision**: An autonomous, 100% agent-driven VM and Linux distribution where an AI core (Gemini 4.0 Argon) completely operates the OS (browsing, coding, system administration, networking) with a sleek task-management and communication UI (evolving the OpenClaw gateway paradigm to full OS ownership).

---

## Roadmap Overview (200 Questions across 40 Batches)

| Phase | Batches | Questions | Domain Focus | Status |
|---|---|---|---|---|
| **Phase 1** | Batches 1–5 | Q1–Q25 | Virtualization, OS Kernel, Base Distro & Isolation Layer | **In Progress (Batch 1)** |
| **Phase 2** | Batches 6–10 | Q26–Q50 | OS Agent Control Interfaces (Wayland, AT-SPI, Headless, MCPs) | Pending |
| **Phase 3** | Batches 11–15 | Q51–Q75 | User Control Plane, OpenClaw Gateway, Task UI & Communication | Pending |
| **Phase 4** | Batches 16–20 | Q76–Q100 | Brain Harness, Multi-Agent Hierarchy & Gemini 4.0 Argon Tooling | Pending |
| **Phase 5** | Batches 21–25 | Q101–Q125 | Coding Runtime, Developer Tooling & Package Management | Pending |
| **Phase 6** | Batches 26–30 | Q126–Q150 | Autonomous Web Browsing, Session Vault & Scraping Engines | Pending |
| **Phase 7** | Batches 31–35 | Q151–Q175 | Security Guardrails, Human-in-the-Loop (HitL) & Rollback Engine | Pending |
| **Phase 8** | Batches 36–40 | Q176–Q200 | Packaging, Distribution (ISO/QCOW2), CI/CD & Final Argon Master Prompt | Pending |

---

## Active Open-Source Ecosystem Under Evaluation

| Project | Repository / Source | Relevance to Heretek-Box |
|---|---|---|
| **OpenClaw** | [`openclaw/openclaw`](https://github.com/openclaw/openclaw) | WebSocket Gateway daemon architecture, multi-channel messaging (Discord/Slack/Telegram), node topology. |
| **AIOS** | [`agiresearch/AIOS`](https://github.com/agiresearch/AIOS) | LLM Agent OS kernel abstraction, scheduling, VM controller, memory management, and MCP broker. |
| **agent-vm (NixOS)** | [`0x04srcz/agent-vm`](https://github.com/0x04srcz/agent-vm) | MicroVMs via `microvm.nix` & NixOS for declarative, reproducible agent containment. |
| **computer-use-linux** | [`agent-sh/computer-use-linux`](https://github.com/agent-sh/computer-use-linux) | Rust MCP server for Linux desktop automation via Wayland & AT-SPI accessibility trees. |
| **Bytebot** | [`bytebot-ai/bytebot`](https://github.com/bytebot-ai/bytebot) | Self-hosted Linux container desktop for browser and desktop computer-use agents. |
| **PortableDesktop** | [`coder/portabledesktop`](https://github.com/coder/portabledesktop) | Single-binary embedded desktop session with VNC/KVM virtual framebuffers. |
| **Agent-S** | [`simular-ai/Agent-S`](https://github.com/simular-ai/Agent-S) | GUI navigation and visual grounding for complex OS interfaces. |

---

## Architectural Decision Log

*(Entries populated as batches are completed)*

### Batch 1: Foundational Architecture & Virtualization (Q1 – Q5)
* **Q1 (Distro Foundation):** NixOS / `microvm.nix` — Pure declarative, immutable state with atomic rollbacks, isolated package sandboxes, and reproducible builds across any host.
* **Q2 (Deployment Target):** Dual-Target — Standalone QEMU/KVM/libvirt VM image (QCOW2/ISO) for bare-metal & desktop virtualization, plus an OCI container runner (Docker/Podman with nested virtualization) for cloud/dev servers.
* **Q3 (OS Control Plane):** Hybrid Semantic-First (D-Bus, Headless CLI, MCP daemons) with fallback to Wayland AT-SPI / Computer Use: High-speed structured APIs, shell PTYs, and MCP servers by default; headless Wayland display for GUI automation when APIs do not exist.
* **Q4 (User Control Plane):** OpenClaw Gateway Model — Unified web dashboard (live terminal canvas, task graph, resource monitors, visual screenshot feed) paired with multi-channel chat gateway bots (Discord, Telegram, Slack, WebSockets) for remote asynchronous command.
* **Q5 (Guardrails & Recovery):** Tiered Autonomy with Instant ZFS/Btrfs CoW Snapshots — Autonomous for standard operations; high-risk actions (sudo, disk format, firewall, secret access) trigger interactive approval prompts in UI, with automated snapshots taken prior to risky executions for 1-click time-travel rollback.

---

### Batch 2: OS Architecture, Hypervisor Backend & Filesystem Layout (Q6 – Q10)
* **Q6 (Hypervisor Backend):** Pluggable / Configurable via Nix flake — Allow switching between Cloud-Hypervisor (for ultra-fast headless tasks, Rust security, virtiofs) and QEMU (when virtual display / GPU acceleration is needed).
* **Q7 (State Persistence & Impermanence):** NixOS Impermanence (tmpfs root + Btrfs subvolumes) — Root filesystem resets to pristine state on reboot; only declared directories (`/nix/persist` for agent memory, workspaces, git repos, gateway DB) are stored on Btrfs subvolumes with automated CoW snapshots.
* **Q8 (Package Management & Tooling):** Ephemeral Dev Shells (`nix-shell` / `nix run`) first, Flake PR for permanent tools — Agent uses `nix run` or `nix-shell` for one-off task execution; for permanently needed system utilities, it edits `flake.nix` and runs `nixos-rebuild test` with automated rollback on failure.
* **Q9 (Host-Guest IPC Bridge):** Linux VSOCK (`AF_VSOCK`) + virtio-serial — Zero-network overhead, kernel-level socket channel directly between host gateway and guest agent daemon, immune to external network sniffing and firewall issues.
* **Q10 (User Privilege Boundary):** Dedicated unprivileged user (`heretek`) with granular sudo rules & Polkit — Runs daily tasks, coding, and browsing as an unprivileged user; administrative operations require specific sudo escalation that triggers approval hooks in the UI.

---

### Batch 3: Networking, Kernel Governance & Secret Management (Q11 – Q15)
* **Q11 (Networking Stack):** Dual Stack (NAT Egress + Tailscale Mesh) — Standard outbound NAT/TAP for web and package traffic, paired with integrated Tailscale inside the guest for secure, encrypted remote access to the agent without opening public ports.
* **Q12 (Init & Process Supervision):** Full native systemd integration (D-Bus & systemctl) — Agent manages daemons, background workers, and cron schedules as native systemd units, querying health via journald JSON APIs.
* **Q13 (Resource Quotas & Cgroups):** Hierarchical cgroups v2 with systemd-oomd & Memory Slices — Distinct slices for System/Gateway (high priority), Agent Brain (medium priority), and Untrusted Builds/Browser (strictly throttled CPU/RAM with automatic OOM kill protection).
* **Q14 (Secret Storage & Injection):** Secret Injection via sops-nix / age + Host VSOCK Secret Broker — Secrets are never written to unencrypted disk; host streams secrets over vsock at boot into a tmpfs keyring (`/run/secrets`) scoped strictly to authorized agent processes.
* **Q15 (Telemetry & Observability):** Structured Journald JSON Stream + Vector / OpenTelemetry collector — System logs, daemon outputs, and agent step traces are parsed into structured JSON and streamed in real-time to the OpenClaw Gateway UI and agent context.

---

### Batch 4: Storage Subsystems, Workspaces & Golden Image Lifecycle (Q16 – Q20)
* **Q16 (Nix Store Provisioning):** Hybrid — Self-contained `/nix/store` inside standalone QCOW2/ISO; virtiofs shared host store when launched from a NixOS host for zero-disk instant startup.
* **Q17 (Workspace & Code Storage):** Dual-Mode Workspace — Default to internal Btrfs persistent storage (`/nix/persist/workspaces`) with atomic git tracking; support optional virtiofs host folder bind-mounts when the user wants to point the agent at an existing host directory.
* **Q18 (Snapshot Engine & Pruning):** Gateway-Hooked Snapper / Custom Go Daemon — Every agent task execution automatically triggers a named Btrfs snapshot (e.g. `pre-task-<task_id>`); the UI displays an undo/rollback button next to every action in the timeline.
* **Q19 (Golden Image CI/CD):** Nix Flake build outputs via `nixos-generators` & GitHub Actions — Declarative flake with matrix outputs (`packages.x86_64-linux.qcow2`, `packages.aarch64-linux.iso`, `packages.dockerImage`) built and published automatically on GitHub Releases.
* **Q20 (Cold-Boot Performance Target):** Sub-second to <2s Cold Boot — Strip unneeded kernel modules, disable plymouth/auditd/udev settled waits, systemd parallel unit activation targeting <1.5s boot time.

---

### Batch 5: Hardware Acceleration, Arch, Rescue & Phase 1 Synthesis (Q21 – Q25)
* **Q21 (GPU & AI Acceleration):** Tiered GPU — Default headless virtio-gpu with Vulkan (Venus) for fast browser and UI rendering; optional PCIe VFIO passthrough module for NVIDIA/AMD GPUs if running local auxiliary models (Whisper, local embedders, vision models).
* **Q22 (CPU Architecture Support):** Multi-Architecture First-Class (`x86_64-linux` & `aarch64-linux`) — Pure Nix Flake matrix building native outputs for both x86_64 (Intel/AMD) and aarch64 (Apple Silicon via UTM/Lima & ARM cloud instances).
* **Q23 (Emergency Break-Glass Rescue):** Dedicated Virtio Console / Serial TTY + VSOCK SSH Rescue — Direct hypervisor serial console (`microvm -c <name>`) and vsock-ssh listener that bypasses guest network interfaces and the agent daemon entirely.
* **Q24 (System Clock Drift):** KVM PTP Clock (`kvm-clock` / `ptp_kvm`) + Chrony — Real-time host-synchronized precision time protocol clock device inside guest kernel, eliminating time drift after sleep/wake or heavy bursts.
* **Q25 (Flake Codebase Structure):** Modular Flake Architecture (`nix/core`, `nix/microvm`, `nix/agent`, `nix/gateway`, `nix/profiles`) — Clean separation of system kernel/storage, agent runtime daemons, gateway interfaces, and hardware profiles.

---

## Phase 2: OS Agent Control Interfaces (Wayland, AT-SPI, Headless, MCPs, Devtools) (Q26 – Q50)

### Batch 6: Wayland Display, Accessibility Tree & GUI Control (Q26 – Q30)
* **Q26 (Compositor & Display Server):** Headless Sway / Hyprland (wlroots-based) — Lightweight, full Wayland protocol support (`wlr-virtual-pointer`, `wlr-screencopy`, XDG desktop portals), headless virtual outputs, and rich JSON IPC for programmatic window control.
* **Q27 (UI Grounding & Inspection):** Hybrid AT-SPI Accessibility Tree + Dynamic Screenshot Vision — Agent inspects UI deterministically via Linux AT-SPI D-Bus (reading button roles, labels, text, states); falls back to screenshot vision with OCR and coordinate bounding boxes when apps lack AT-SPI.
* **Q28 (Input Emulation Engine):** Kernel `uinput` + Wayland `wlr-virtual-pointer` — Directly dispatches hardware-level evdev mouse/keyboard events via `/dev/uinput` and Wayland protocols, bypassing brittle X11 fake-event shims.
* **Q29 (Window Management & Layout):** Programmatic JSON IPC (Sway IPC / Hyprctl) — Agent queries window trees, moves windows to specific workspaces, focuses by PID/app-id, and tiles layouts deterministically over unix sockets.
* **Q30 (Live Video Streaming to UI):** Low-Latency WebRTC / PipeWire Stream — Capture video from PipeWire via `xdg-desktop-portal-wlr` and stream H.264/VP9 via WebRTC directly to the OpenClaw Web UI canvas at 60fps with near-zero latency.

---

### Batch 7: Model Context Protocol (MCP) Architecture & Tooling Broker (Q31 – Q35)
* **Q31 (MCP Broker & Tool Discovery):** Central Rust/Go MCP Broker with Lazy Tool Discovery — A single daemon multiplexes backend MCP servers over Unix Domain Sockets; Argon starts with core meta-tools (`search_tools`, `invoke_tool`, `inspect_schema`), loading specific tool schemas dynamically to prevent context bloat.
* **Q32 (Terminal & Shell Execution):** Persistent Pseudo-Terminal (PTY) Multiplexer over Unix Domain Socket — Agent spawns stateful tmux/libpty sessions that preserve shell environment, exports, and history across tool calls, supporting interactive input, signal handling (Ctrl+C), and ANSI escape sequence parsing.
* **Q33 (System Administration MCP):** Native D-Bus MCP Server — Exposes structured D-Bus methods directly to Argon (`org.freedesktop.systemd1`, `org.freedesktop.NetworkManager`, `org.freedesktop.timedate1`) for programmatic, typed control without parsing text output from CLI tools.
* **Q34 (Filesystem MCP Operations):** Chunked View + AST Structural Search + Precise Block Replacement — MCP server provides slice-based viewing (line ranges & byte offsets), AST symbol discovery (ctags/Tree-sitter), and surgical block replacement with strict line targeting.
* **Q35 (Long-Running Background Tasks):** Asynchronous Task Scheduler with Reactive Push Notifications — Agent launches jobs in the background as tracked tasks with unique TaskIDs; the broker sends a high-priority interrupt notification to Argon upon completion, failure, or timeout (zero polling loops required).

---

### Batch 8: Developer Tooling Ecosystem, Code Intelligence & Debugging (Q36 – Q40)
* **Q36 (LSP Engine & Compiler Intelligence):** Multiplexed LSP-over-MCP Daemon — An internal daemon auto-detects language servers (`rust-analyzer`, `pyright`, `gopls`, `vtsls/tsserver`, `clangd`) via Nix packages, providing typed LSP endpoints (`get_definition`, `find_references`, `get_diagnostics`, `hover`) directly to Argon.
* **Q37 (Codebase Knowledge Graph):** Embedded Knowledge Graph MCP (Codebase Memory) — Auto-indexes code repositories into an embedded graph database, allowing Argon to trace inbound/outbound call paths, query symbols, and inspect architecture across 100k+ line projects.
* **Q38 (Headless Debugging & Runtime Troubleshooting):** DAP-over-MCP Bridge (Debug Adapter Protocol) — Agent can set programmatic breakpoints, inspect call stacks, evaluate variables in memory, and step through execution using standard debuggers (GDB, LLDB, Delve, debugpy) headlessly.
* **Q39 (Git Lifecycle & Worktree Strategy):** Atomic Semantic Worktrees & Shadow Branching — Agent works in an isolated git worktree; automatically creates fine-grained semantic commits with descriptive Conventional Commit messages, rebasing into clean PR branches for user review.
* **Q40 (Test Execution & Verification):** Sandboxed Test Runner with Coverage & Diagnostic Feedback — Agent triggers test suites (`pytest`, `cargo test`, `vitest`, `go test`) in an isolated cgroup/namespace, collecting structured JSON test reports and auto-feeding failed assertions into the agent context for self-healing loops.

---

### Batch 9: Autonomous Web Browsing, DevTools & Scraping Engines (Q41 – Q45)
* **Q41 (Web Browser Backend):** Self-Hosted Steel / Playwright Chromium with Stealth Patches — Local headless/headful Chromium instance configured with fingerprint randomization, CDP endpoints, and stealth evasion to avoid Cloudflare/bot blocks while running entirely inside the VM.
* **Q42 (Web Navigation Paradigm):** Dual-Engine (Pruned Semantic DOM Tree + Annotated Visual Screenshots) — Agent gets a compact filtered representation of clickable/input elements with numeric labels for ultra-fast, low-token navigation; requests screenshots only when visual layout is ambiguous.
* **Q43 (Static Web Scraping Engine):** Integrated Fast-Path Scraping Engine (Crawl4AI / Trafilatura) — Headless HTTP fetching with heuristic markdown conversion, stripping ads/navbars and returning clean LLM-ready markdown in <200ms without spinning up a full browser.
* **Q44 (Session & Login Persistence):** Encrypted Browser Profile Vault in `/nix/persist` — Browser user data directories (cookies, localStorage, session tokens) are stored on encrypted persistent Btrfs subvolumes, allowing user to log in once via the Web UI stream and let the agent reuse sessions seamlessly.
* **Q45 (Search Engine & OSINT Discovery):** Local SearXNG Instance + GitHub REST/GraphQL API MCP — Self-hosted meta-search engine aggregating Google, Bing, DuckDuckGo, and Arxiv without tracking, coupled with a dedicated GitHub API MCP server for deep repository/code search.

---

### Batch 10: Audio, Recording, Notifications & Phase 2 Synthesis (Q46 – Q50)
* **Q46 (Audio & Voice Processing):** PipeWire Headless Virtual Sink + Whisper STT & Web Audio — PipeWire null-audio-sink captures guest audio (browser videos, notification sounds); bidirectional Web Audio stream links browser microphone to host Whisper transcription engine for hands-free voice commanding.
* **Q47 (Visual Proof & Artifact Recording):** Automated Wayland Screencopy (`wf-recorder` / GStreamer) + MP4/GIF Artifact Exporter — High-resolution hardware-encoded screen clips and key-frame screenshots automatically bundled as task artifacts linked in the UI timeline.
* **Q48 (Desktop Notification Interception):** D-Bus Notification Server Daemon (`org.freedesktop.Notifications`) bridged to Agent Event Loop — A custom headless notification daemon captures all OS notifications, passes them to Argon's reactive event bus, and mirrors critical alerts to the user's OpenClaw UI.
* **Q49 (Host-Guest Clipboard Synchronization):** Wayland `wl-clipboard` over VSOCK Gateway Bridge — Bi-directional synchronized clipboard with user privacy toggle; allows user and agent to share copied text/code/images securely without hypervisor display dependency.
* **Q50 (Agent Tooling Surface Synthesis):** Unified MCP Server Registry with Dynamic Categorical Manifests — Tools are grouped into categories (`sysadmin`, `code`, `browser`, `desktop`, `files`); Argon is bootstrapped with a compact summary of available categories, and lazy-loads full schemas on-demand when entering a specific workflow.

---

## Phase 3: User Control Plane, OpenClaw Gateway, Task UI & Communication (Q51 – Q75)

### Batch 11: Web Dashboard Architecture, Protocol & Terminal Streaming (Q51 – Q55)
* **Q51 (Frontend Stack & UI):** Next.js / React with Tailwind CSS & shadcn/ui + WebSockets — Modern reactive stack, component-rich UI with virtualized terminal (xterm.js), WebRTC video canvas, live DAG execution trees, and responsive mobile layout.
* **Q52 (WebSocket Protocol & Framing):** Strongly-Typed JSON-RPC 2.0 / Protocol Buffers over WebSockets — Strictly validated JSON schemas for client requests, streaming token diffs, tool call state transitions, and server-push events (agent, presence, health, cron).
* **Q53 (Multi-Session Organization):** Multi-Workspace Tabbed Command Center — Sidebar listing concurrent agent sessions/projects with status badges (running, waiting on approval, idle, errored); clicking a workspace switches the live terminal, canvas, and chat stream instantaneously.
* **Q54 (Mobile / Tablet Experience):** Mobile-Optimized PWA (Progressive Web App) — Responsive layouts with touch-friendly approval cards, bottom-sheet voice input, collapsible terminal/canvas views, and push notifications for task completions.
* **Q55 (Terminal PTY Rendering):** Virtualized xterm.js with ANSI Color & Interactive Human Takeover — Renders full terminal output with color coding, progress bars, and scrolling; user can toggle 'Interactive Mode' to type into the agent's terminal session directly.

---

### Batch 12: Chat Gateway Connectors, HitL Approvals & Media Ingestion (Q56 – Q60)
* **Q56 (Supported Chat Platforms):** Tier 1 First-Class: Discord (rich embeds & component buttons) + Telegram (inline keyboard callbacks) + Slack (Block Kit action cards), with extensible plugin architecture for WhatsApp/Signal.
* **Q57 (HitL Approval Cards in Chat):** Rich Interactive Action Cards with Diff Preview — Card displays command/diff, risk explanation, target files, and 4 buttons: `[Approve Once]`, `[Approve Session]`, `[Reject]`, `[Rollback Snapshot]`; clicking instantly unlocks or reverts the VM.
* **Q58 (Chat Media Ingestion):** Auto-Downloaded to Task Scratch Space + Multimodal Injection — Gateway streams the attachment into `/nix/persist/workspaces/<task_id>/inbox/`, converts voice memos to text via local Whisper, and provides direct file URIs to Argon's prompt.
* **Q59 (Chat Channel / Thread Organization):** 1-to-1 Thread Mapping — Every new task automatically creates a dedicated Discord/Slack thread or Telegram topic; all subsequent messages, tool logs, and artifacts stay confined to that thread, avoiding spam in the main channel.
* **Q60 (Chat Notification Throttling):** Collapsible Progress Blocks with Real-Time Edits — Bot continuously updates/edits a single pinned progress message with live spinners and current step summaries; only posts new messages for major milestones, approval requests, or task completion.

---

### Batch 13: Task Lifecycle Management, DAG Engine & Work Queues (Q61 – Q65)
* **Q61 (Task Graph & Decomposition):** SQLite-Backed Topological DAG Engine — Tasks are decomposed into a directed acyclic graph of subtasks with explicit dependencies; nodes execute concurrently when inputs are satisfied; cycles are rejected automatically; live DAG rendered in Web UI.
* **Q62 (State Machine & Ledger):** Append-Only Event Sourced Ledger — Tasks move through strict states: `[QUEUED -> PLANNING -> EXECUTING -> WAITING_FOR_APPROVAL -> BLOCKED -> COMPLETED / FAILED / ABORTED]`; every transition is recorded as an immutable event in SQLite for perfect replayability.
* **Q63 (Cooperative Pausing & Intervention):** First-Class Cooperative Pause & Human Injection — User can click `[Pause]`; the agent cleanly finishes its current tool call, yields its lock, and lets the user inspect/edit files or send guidance before clicking `[Resume]`.
* **Q64 (Scheduled Jobs & Cron Daemons):** Declarative Cron Scheduler with Gateway Daemon — Cron expressions declared in UI or chat (e.g. `/schedule "*/30 * * * *" "Check PRs"`); Gateway runs cron jobs in background, spawning ephemeral subagent runs and notifying user only on anomalies.
* **Q65 (Concurrency & Priority Scheduling):** Priority Queue with Weighted Concurrency (Fair-Share) — Tasks have priority tiers `[CRITICAL, HIGH, NORMAL, BACKGROUND]`; the scheduler limits concurrent heavyweight jobs (browser/compilation) to CPU core limits while allowing lightweight I/O tasks to run in parallel.

---

### Batch 14: Authentication, RBAC, Tunneling & Security Governance (Q66 – Q70)
* **Q66 (Web Authentication):** Tailscale Identity Headers + Local Passkey (WebAuthn) / Bearer Token Fallback — Automatic zero-click login when accessing via Tailnet (using Tailscale identity headers); hardware Passkey (FIDO2/TouchID) or cryptographically generated Gateway Token when accessing locally or via custom reverse proxy.
* **Q67 (Multi-User RBAC):** Tiered RBAC `[Admin, Developer, Guest/Auditor]` — Admins can approve high-risk sudo/rollback operations and manage API keys; Developers can dispatch coding/browsing tasks within their assigned workspaces; Guests can only view live logs and artifacts.
* **Q68 (Network Tunneling):** Built-in Tailscale Serve / Funnel with optional Cloudflare Tunnel integration — Native CLI toggles (e.g. `heretek tunnel enable tailscale|cloudflare`) that route ingress with zero open firewall ports on the host router.
* **Q69 (Audit Logging):** Cryptographically Hash-Chained JSON Audit Log — Every user prompt, approval decision (with user ID and channel source), tool call, and snapshot rollback is recorded into an append-only, SHA-256 hash-chained ledger.
* **Q70 (Budget & Throttling Guardrails):** Unconstrained Usage — No artificial step or token limits; agent runs continuously until goal reached or user manually aborts (empowering extreme autonomous execution).

---

### Batch 15: Artifact Presentation, File Delivery & Gateway Synthesis (Q71 – Q75)
* **Q71 (Artifact Presentation & Live App Preview):** Sandboxed Live Preview Iframe & Artifact Drawer — UI embeds a live iframe connected to guest localhost port (for web apps) alongside an artifact drawer rendering markdown, Mermaid diagrams, images, and audio waveforms with real-time hot reload.
* **Q72 (File Export & Host Synchronization):** Multi-Channel Export — 1-Click ZIP Download & Direct Git Push to Remote (GitHub/GitLab) from UI, plus automatic synchronization to host mounted directories via virtiofs.
* **Q73 (Interactive Artifact Feedback):** In-Line Contextual Comments & Diff Requests — User can highlight text or code in the artifact viewer and click 'Ask Agent to Refine', which injects the highlighted chunk and user critique directly into the next prompt turn.
* **Q74 (External Webhooks & API Subscriptions):** Configurable Webhook Dispatcher — Gateway fires customizable JSON webhooks on key lifecycle events (`task.created`, `task.progress`, `task.hitl_approval_required`, `task.completed`, `task.failed`) with HMAC-SHA256 signature verification.
* **Q75 (Gateway Control Plane Daemon):** Host-Side Rust/Go Gateway Daemon supervising guest via VSOCK — Single high-performance binary running on host (or host container), managing hypervisor lifecycle, hosting Web UI, routing Discord/Slack/Telegram bots, and streaming VSOCK commands directly into guest.

---

## Phase 4: Agent Brain Harness, Multi-Agent Hierarchy & Gemini 4.0 Argon Tooling (Q76 – Q100)

### Batch 16: Multi-Agent Topology, Orchestration Hierarchy & Scoping (Q76 – Q80)
* **Q76 (Multi-Agent Topology):** Hierarchical Chief-of-Staff with Specialized Department Agents (OpenGoat Model) — Argon acts as Chief Orchestrator (Architect/Planner); it spawns isolated specialist subagents: `[Coder, Sysadmin, WebResearcher, SecurityAuditor, QA]`; subagents report back structured artifacts, keeping Argon's context window pristine.
* **Q77 (Subagent Workspace Isolation):** Branch / Isolated Worktree Mode by Default — Subagents spawn in their own isolated git worktrees or ephemeral scratch directories (`/tmp/heretek/subagents/<id>`), preventing file collisions with the parent agent until explicitly merged.
* **Q78 (Inter-Agent Messaging):** Asynchronous Mailbox & Reactive Message Passing (`send_message` / `invoke_subagent`) — Parent dispatches subagent with explicit task prompt; parent continues other tasks or yields turn; subagent messages parent upon milestone or completion; system resumes parent automatically.
* **Q79 (Persona Configuration):** Declarative Markdown Files (`SOUL.md`, `IDENTITY.md`, `SYSTEM.md`) in `/nix/persist/config/agents/` — Modular markdown configurations defining agent role, behavioral rules, allowed MCP tools, and output constraints; editable via UI or git.
* **Q80 (Tool Scoping by Role):** Role-Based Tool Scoping — Research subagents get only read-only tools (search, browser, fetch, read_file); Coder gets git, lsp, code edit, and test runners; only Sysadmin/Parent has access to bash root and systemd D-Bus.

---

### Batch 17: Memory Architecture, Long-Term Storage & Skill Learning (Q81 – Q85)
* **Q81 (Long-Term Storage Architecture):** Hybrid Local-First Memory (SQLite + `sqlite-vec` + FTS5 + LanceDB) — SQLite handles relational task state and fast full-text keyword search; LanceDB handles dense multimodal vector embeddings (past coding solutions, user corrections, tool usage patterns) stored in `/nix/persist/memory/`.
* **Q82 (Memory Consolidation & Dreaming):** Scheduled Memory Consolidation Daemon (The "Dreaming" Phase) — A nightly background job analyzes the day's tasks, extracts reusable patterns, fixes, user preferences, and discarded dead-ends, compressing them into permanent knowledge nodes while pruning transient noise.
* **Q83 (Procedural Skill Playbooks):** Executable Skill Repository (`/nix/persist/skills/<skill-name>/SKILL.md`) — Agent dynamically generates a skill folder containing YAML frontmatter, step-by-step markdown instructions, and executable helper scripts that future agents can invoke automatically.
* **Q84 (User Adaptation & Preferences):** Dynamic User Profile Graph — An explicit entity-relationship graph (e.g. User prefers TypeScript over JS, tabs over spaces, dislikes verbose explanations in chat) loaded into Argon's context alongside the system prompt.
* **Q85 (Memory Governance & Privacy):** Auditable Memory Management UI with 1-Click Revocation — Web dashboard features a 'Memory Explorer' where the user can search, inspect, edit, or delete any entity, vector embedding, or learned skill instantly.

---

### Batch 18: Context Window Optimization, Compaction & Token Economy (Q86 – Q90)
* **Q86 (Gemini Context Caching):** Structured Prefix Caching (System Prompts + MCP Schemas + Base Distro Spec) — Keep the static harness (>32k tokens: OS architecture, available tools, system policies, user identity graph) pinned in Gemini context cache with a sliding TTL; yields 90% input token discount and instant TTFT for all turns.
* **Q87 (History Compaction Strategy):** Semantic Checkpoint Compaction with Artifact References — Compress completed subtask turns into a structured summary (Goals, Files Modified, Decisions Made, Unresolved Blockers), pruning verbose tool outputs while preserving artifact file links and active diffs.
* **Q88 (Tool Output Serialization):** Head/Tail Truncation with Byte Offset Pointers — Return the first 50 lines and last 50 lines with a clear message: `"[Truncated N lines. Full output saved to <scratch_file>. Use view_file with offset to read more]"`; keeps token cost minimal while giving agent visibility of errors.
* **Q89 (Cross-Turn Scratchpad Persistence):** Dedicated Persistent Scratchpad File (`/nix/persist/workspaces/<id>/SCRATCHPAD.md`) — Agent reads and updates a persistent markdown scratchpad between turns; injected as a pinned system section in prompt turns for continuous situational awareness.
* **Q90 (Token & Cost Accounting):** Real-Time Token & Cost Dashboard with Per-Task Breakdown — Web UI displays live input/output/cached token counters and estimated cost ($USD) for the current task, subagents, and daily/monthly aggregate with spending graphs.

---

### Batch 19: Reasoning Engine, Verification & Error Recovery (Q91 – Q95)
* **Q91 (Thinking Traces & Chain-of-Thought):** Streamed Real-Time Thinking Drawer with Collapsible UI — Internal reasoning is streamed live over WebSockets and rendered in a sleek collapsible 'Brain Thought' drawer in the Web UI/Discord; hidden by default so non-technical users aren't overwhelmed, but instantly expandable for debugging.
* **Q92 (Self-Verification & Quality Gates):** Automated Self-Verification & Lint/Test Barrier — Argon is strictly prompted to run a 3-point verification: (1) Check git diff against requirements, (2) Execute test suite and linters, (3) Confirm system services are active; only if all 3 pass can it issue `task_completed`.
* **Q93 (Stuck Loops & Error Recovery):** Exponential Backoff with Hypothesis Reformulation — On 2nd consecutive failure, agent is forced to step back and re-read error logs; on 3rd failure, it must invalidate its previous hypothesis, search knowledge graph/web, or propose a rollback; on 4th, it prompts user for guidance.
* **Q94 (Hallucination Prevention):** Grounded Tool Verification Pre-Conditions — MCP tools strictly enforce verification: file modification tools error if the file doesn't exist (unless explicitly creating); symbol lookups validate against LSP index; compiler diagnostics feed directly into context.
* **Q95 (Dynamic Scratch Tooling):** Dynamic Scratch Script Generation in Sandbox (`/nix/persist/workspaces/<id>/scratch/`) — Agent writes custom Python, Bash, or Rust scripts in an isolated scratch folder, executes them via terminal PTY, and parses structured output; successful scripts can be promoted to permanent skills.

---

### Batch 20: Master Prompt Architecture, Slash Commands & Phase 4 Synthesis (Q96 – Q100)
* **Q96 (System Prompt Structure):** Modular Multi-Section Prompt — `[Identity & Prime Directive -> OS & Machine Environment Schema -> Available Tool Manifest & Categories -> Operating Guidelines & Non-Interactive Rules -> Communication & Artifact Formatting -> Slash Commands -> Safety & Approval Boundaries]`.
* **Q97 (Slash Commands Suite):** Power Suite `[/goal, /plan, /compact, /browser, /rollback, /schedule, /memory, /pause]` — Dedicated command shortcuts for long-horizon execution, planning interviews, context compaction, headless browsing, snapshot reversion, and cron schedules.
* **Q98 (Non-Interactive Command Defaults):** Enforced Non-Interactive Environment Flags + PTY Timeout — Default environment sets `DEBIAN_FRONTEND=noninteractive`, `GIT_TERMINAL_PROMPT=0`, `CI=true`, `PAGER=cat`; PTY multiplexer detects hung interactive prompts and aborts with a clear error if stdin is blocked.
* **Q99 (Autonomous System Watchdog):** Configurable Autonomous Watchdog — Silent during normal background work, but proactively pings user on critical events: disk space > 85%, failed systemd daemons, security CVEs in repos, or stalled long-running jobs.
* **Q100 (Primary Architecture Deliverable):** High-Level Architecture Document — Broad technical design document with diagrams (and accompanying master prompt spec).

---

## Phase 5: Coding Runtime, Developer Tooling & Package Management (Q101 – Q125)

### Batch 21: Development Environments, Package Managers & Polyglot Toolchains (Q101 – Q105)
* **Q101 (Project Environment Provisioning):** Declarative `devenv.sh` per Workspace — Agent generates a `devenv.nix` in the workspace; `devenv shell` auto-provisions exact language toolchains, linters, and embedded background services (Postgres, Redis) without polluting system state.
* **Q102 (Python Package Manager):** Standalone `uv` Toolchain (Astral uv) — Blazing-fast Rust-based package installer and virtual environment manager (`uv venv`, `uv pip`, `uv run`); resolves dependencies in milliseconds without external system dependencies.
* **Q103 (JS/TS Web Tooling):** Dual Engine (`Node.js`/`pnpm` + `Bun`) — `pnpm` guarantees deterministic `node_modules` while `Bun` provides ultra-fast test running, instant script execution, and TypeScript transpilation.
* **Q104 (Native Compiled Toolchains):** Nix-Linked LLVM & GCC Toolchains with `ccache` — Pre-configured clang, gcc, rust-bin, and go from Nixpkgs with ccache enabled to cache compilation artifacts and accelerate repeated builds.
* **Q105 (Database & Service Mocking):** Rootless Process-Level Services via `devenv` — Services run as unprivileged users bound to local unix sockets or high ports (e.g. localhost:5433) managed directly by devenv processes, eliminating root Docker daemon overhead.

---

### Batch 22: Code Analysis, Static Linting & Tree-Sitter AST Tooling (Q106 – Q110)
* **Q106 (Structural Code Search & AST Refactoring):** Integrated `ast-grep` MCP Server (`@ast-grep/cli` + Tree-sitter) — Provides `ast-grep outline` for token-efficient code navigation and structural pattern queries (`find_code_by_rule`, `rewrite`), guaranteeing AST-valid refactorings across 20+ programming languages.
* **Q107 (Fast Polyglot Linting & Formatting):** Next-Gen Rust-Native Toolchains (Ruff for Python, Biome for JS/TS, Rustfmt, Gofmt) — Instant sub-10ms formatting and lint fixes; automatically executed on save or file modification to keep workspaces clean without human intervention.
* **Q108 (Code Navigation & Outline):** Symbol Outline Tool (`ast-grep outline` / Tree-sitter tags) — Generates a concise tree of class names, function signatures, and line ranges (<200 tokens); agent views full code only for the specific function slice it needs to modify.
* **Q109 (Type Diagnostics & Compiler Feedback):** Real-Time Diagnostic Feed from LSP Bridge — Diagnostics (errors, warnings, missing imports, type mismatches) are pushed directly into Argon's tool response after any file edit, allowing immediate self-correction before running tests.
* **Q110 (Dependency Security & License Audits):** Automated Pre-Install Security Scan (Trivy / `cargo-audit` / `pip-audit`) — Agent runs a lightweight vulnerability and license check before committing new packages; flags known CVEs or license violations automatically.

---

### Batch 23: Nix Flake Engineering, Custom Derivations & Reproducible Builds (Q111 – Q115)
* **Q111 (Packaging Custom Software):** Automated Derivation Generation via `nix-init` — Agent runs `nix-init` pointing at the repository URL to auto-compute sha256 hashes, infer language build systems (Rust, Go, Python), and generate a clean derivation package in `/nix/persist/pkgs/`.
* **Q112 (Binary Caching & Acceleration):** Integrated Binary Cache (Cachix / Local Attic Daemon) — Pre-configured public Nix caches (nixos-org, cachix) plus an optional self-hosted Attic/Harmonia cache; downloads pre-compiled binary closures in seconds rather than compiling locally.
* **Q113 (Flake Lockfile Management):** Targeted Input Updates with Automated Rebuild Testing — Agent runs `nix flake lock --update-input <specific-dep>` followed by `nixos-rebuild test`; reverts lockfile immediately if evaluation or service tests fail.
* **Q114 (FHS Binary Compatibility):** Declarative `buildFHSEnv` & `auto-patchelf` Hook — Provide an auto-wrapping FHS environment and auto-patchelf utility so the agent can execute standard Linux ELF binaries seamlessly without dynamic linker crashes.
* **Q115 (User Flake Customization):** Overlay & Module Import System — User drops custom `.nix` files into `/nix/persist/custom/` (e.g. `extra-tools.nix`, `my-dotfiles.nix`); Heretek-Box auto-imports all files in that directory on next rebuild without modifying core system flakes.

---

### Batch 24: Containerization inside the VM (Rootless Podman, DinD) (Q116 – Q120)
* **Q116 (Container Engine):** Rootless Podman with Docker Compatibility (`dockerCompat = true`) — Runs daemonless and rootless under the `heretek` user with cgroups v2 delegation; symlinks docker to podman so docker-compose and all standard tools work without root security risks.
* **Q117 (Container Storage & Layers):** Dedicated Btrfs Subvolume with Auto-Pruning (`/nix/persist/containers`) — Uses Podman's native Btrfs storage driver for instant zero-copy layer clones, with an automated cron hook to prune dangling images if disk exceeds 80%.
* **Q118 (Local Kubernetes Emulation):** On-Demand Rootless k3s / Kind via Podman — Lightweight Kubernetes single-node cluster spun up only when requested by a task, completely stopped and reclaimed when the task finishes.
* **Q119 (Container Network Isolation):** Rootless Netavark / Pasta Network Stack — Native rootless networking provided by Podman with DNS resolution across containers on custom user-defined bridge networks.
* **Q120 (Container Registry Publishing):** Direct Buildah / Podman Push to Remote Registries (GHCR, Docker Hub) + Embedded Local Registry — Agent can push to public/private registries using secrets from the vault, or spin up an ephemeral local registry (localhost:5000) for testing.

---

### Batch 25: CLI Suite, DevOps Automation & Phase 5 Synthesis (Q121 – Q125)
* **Q121 (Remote Git CLI Integration):** Pre-Configured `gh` & `glab` CLIs with Vaulted Token Injection — Heretek-Box bakes in official `gh` and `glab` CLIs; authentication tokens are injected securely from the secret broker, enabling the agent to create PRs, read review comments, and check Actions CI runs natively.
* **Q122 (Local CI/CD Emulation):** Local Workflow Runner via `act` + Rootless Podman — Agent executes `nektos/act` inside the VM using rootless Podman, running the exact `.github/workflows/*.yml` jobs locally before opening a pull request.
* **Q123 (Infrastructure-as-Code):** Pre-Packaged OpenTofu + Cloud CLIs (`awscli`, `gcloud`, `cloudflared`) — OpenTofu provides open-source MPL-compliant infrastructure orchestration; credentials scoped strictly per-task.
* **Q124 (Linux Diagnostics Toolkit):** Modern Core Diagnostic Suite (`btop`, `ncdu`, `strace`, `lsof`, `tcpdump`, `ripgrep`, `fd`, `jq`) — Essential high-speed utilities pre-compiled in the NixOS base image so Argon can immediately diagnose CPU bottlenecks, locked files, open network ports, and disk leaks.
* **Q125 (Coding & DevOps Persona Synthesis):** Dedicated Full-Stack DevOps Persona (`coder.md` / `sysadmin.md`) — Equipped with devenv, uv, pnpm/bun, gh, act, ast-grep, and lsp-mcp; strictly instructed to write tests, verify builds locally, create semantic commits, and test CI before handing off.

---

## Phase 6: Autonomous Web Browsing, Session Vault & Scraping Engines (Q126 – Q150)

### Batch 26: CAPTCHA Resolution, Anti-Detection & Browser Profiles (Q126 – Q130)
* **Q126 (CAPTCHA Resolution):** Hybrid 3-Tier Solving `[Stealth Prevention -> Automated Solver Plugin -> Interactive Human Takeover]` — Stealth profiles avoid challenges; automatic token solver plugin (NopeCHA / Capsolver API) attempts bypass; if it fails, pops a live interactive browser drawer in Web UI for user to click in 5 seconds.
* **Q127 (Fingerprint Obfuscation):** Deep Kernel & CDP Fingerprint Masking — Spoof WebGL vendor/renderer strings, AudioContext noise, randomized screen resolutions, real User-Agents, and override `navigator.webdriver=false` at the CDP initialization level.
* **Q128 (Proxy & IP Routing):** Configurable Proxy Pool with Tailscale Exit Nodes + SOCKS5/HTTP Proxy Support — Agent can route traffic through user's choice of Tailscale Exit Nodes, local Tor/Privoxy, or commercial residential proxy pools with per-request rotation.
* **Q129 (Browser Profile Partitioning):** Dual-Tier Profiles `[Persistent Authenticated Vault + Disposable Ephemeral Clones]` — Standard browsing clones a temporary incognito profile that self-destructs on task completion; authenticated tasks (e.g. AWS console, GitHub) use the encrypted Persistent Vault with locked session state.
* **Q130 (Credential & Password Injection):** Masked Secret Injection via CDP (`fill_secret`) — Agent references secret aliases (e.g. `vault:github_pat`); the browser driver types the actual credential into the DOM input directly without writing the secret string to agent context or chat logs.

---

### Batch 27: DOM Extraction, Heuristic Parsing & Web Pipelines (Q131 – Q135)
* **Q131 (Fit Markdown Extraction):** BM25 & Heuristic Fit Markdown Extraction (Crawl4AI) — Strips cookie banners, header/footer navbars, and advertisements; converts the primary content container into clean, GitHub-flavored markdown with citation links preserving tables and code blocks.
* **Q132 (SPA Hydration Detection):** Two-Tier Adaptive Fetch — Fast HTTP GET first; if returned HTML contains minimal text, noscript warnings, or React/Vue root mounting points (`<div id="root"></div>`), automatically escalates to headless Chromium with network-idle waiting.
* **Q133 (Structured Metadata Harvesting):** Automatic Metadata & JSON-LD Extractor — Dedicated tool hook that extracts OpenGraph tags, Twitter cards, and Schema.org JSON-LD microdata as a structured JSON object prior to page body parsing.
* **Q134 (Online Document Ingestion):** Streamed Document Parser (`pdfplumber` / `pymupdf` + `csv` parser) — Downloads binary directly to scratch folder and outputs extracted text, tables, and page outlines with page-range indexing without opening a browser viewer.
* **Q135 (Media & Asset Downloads):** Workspace Inbox Directory with Deduplication — Assets are downloaded to `/nix/persist/workspaces/<id>/downloads/` with SHA-256 deduplication and auto-generated metadata manifests (`.metadata.json`).

---

### Batch 28: Multi-Tab Browsing, Event Interception & Network Sniffing (Q136 – Q140)
* **Q136 (Network Request Interception):** Native CDP Network Listener (`Network.enable` + `list_network_requests` / `get_response_details`) — Exposes CDP network stream via MCP, allowing the agent to capture internal REST/GraphQL endpoints, payload schemas, and JSON responses without reverse engineering HTML.
* **Q137 (Multi-Tab Management):** Programmatic Tab Manager with Auto-Pruning — Agent can open, switch, inspect, and close tabs via targetId (`list_pages`, `select_page`, `close_page`); background tabs auto-freeze (discarding unneeded rendering layers) if total browser RAM exceeds 1.5GB.
* **Q138 (JavaScript Dialog Interception):** Automated Dialog Interception Hook (`handle_dialog`) — CDP listener intercepts dialog events immediately; automatically accepts or dismisses benign alerts, or feeds critical prompts (e.g. "Discard unsaved changes?") to Argon's decision loop.
* **Q139 (Console Error Telemetry):** Real-Time Console Log Buffer (`Runtime.consoleAPICalled` + `Log.entryAdded`) — Captured `console.error`, `console.warn`, and uncaught exceptions are collected into a structured buffer queryable via `get_console_messages`.
* **Q140 (Viewport & Device Emulation):** Dynamic Viewport Emulation (`Emulation.setDeviceMetricsOverride`) — Defaults to 1920x1080 @ 1x scale for desktop workflows; agent can programmatically switch to mobile viewport (e.g. iPhone 15 / Pixel 8) to test responsive UI or bypass desktop-specific anti-bot firewalls.

---

### Batch 29: Search Strategy, Knowledge Harvesting & Citations (Q141 – Q145)
* **Q141 (Local Search Daemon):** Native NixOS `services.searx` with Local Redis Caching — Runs bound to `127.0.0.1:8888` with local Redis caching; aggregates Google, Bing, DuckDuckGo, Wikipedia, and Arxiv without API keys, zero rate-limits, and zero telemetry tracking.
* **Q142 (Query Expansion & Deep Research):** Recursive Multi-Angle Search with Query Expansion — Agent breaks goal into 3-5 distinct orthogonal queries; crawls top 5 results per query; extracts cited URLs and follows references 1 level deep; aggregates findings into a unified matrix.
* **Q143 (Multi-Source Verification):** Multi-Source Consensus Triangulation — Facts, installation steps, and code snippets must be corroborated across at least 2 independent sources (e.g. official docs + active GitHub repository); conflicting claims are explicitly highlighted in the report.
* **Q144 (Source Archiving):** Clean Markdown + MHTML Archive Snapshot in Task Workspace — Extracted markdown is saved alongside a full single-file MHTML snapshot in `/nix/persist/workspaces/<id>/archive/`; provides permanent verifiable local proof of sources.
* **Q145 (Research Report Artifact):** Structured Research Dossier Artifact — Generates an interactive markdown artifact with executive summaries, comparison matrix tables, deep-dive sections, and clickable citations.

---

### Batch 30: Browser Security, Prompt Injection & Phase 6 Synthesis (Q146 – Q150)
* **Q146 (Prompt Injection Defense):** Dual-Layer Sanitizer `[CSS Pruning + Strict Untrusted Data Boundaries]` — Strips all hidden/invisible DOM elements before parsing; wraps all fetched web text in strict XML sandbox tags (`<untrusted_web_data url="...">...</untrusted_web_data>`) with explicit system instructions prohibiting execution of commands found in untrusted text.
* **Q147 (Adblocking & Tracking Defense):** Pre-Loaded Headless uBlock Origin Extension — Chromium launches with uBlock Origin enabled; blocks tracking scripts, video ads, crypto-miners, and telemetry, speeding up page loads by 4x and reducing VM RAM usage.
* **Q148 (Browser OS Sandboxing):** Bubblewrap Sandbox + Linux Seccomp Filters — Chromium process is executed inside a rootless bubblewrap container with restricted filesystem access (read-only Nix store, isolated tmpfs) and seccomp filters blocking raw socket manipulation.
* **Q149 (Cookie & Session Portability):** Web UI Cookie Importer / Exporter (`.json` format) — UI allows user to upload or paste exported cookie JSON files (from standard browser extensions) encrypted directly into the persistent profile vault with 1-click revocation.
* **Q150 (Web Researcher Persona Synthesis):** Specialized Researcher Persona (`researcher.md`) — Equipped with SearXNG, Crawl4AI, CDP browser tools, and uBlock Origin; instructed to corroborate across multiple sources, extract structured JSON-LD, sanitize untrusted text, and produce rich dossier artifacts.

---

## Phase 7: Security Guardrails, Human-in-the-Loop (HitL) & Rollback Engine (Q151 – Q175)

### Batch 31: Command Risk Classification, HitL Approvals & Supply Chain Gating (Q151 – Q155)
* **Q151 (Command Safety Tiers):** Multi-Tier Risk Heuristic Parser (AST + Regex + Capability) — Commands categorized into `[SAFE: read-only, git status, cargo test, dev servers]`, `[CAUTION: file overwrites, npm/cargo install, branch deletion]`, and `[DANGEROUS: sudo, rm -rf, mkfs, iptables, dd, modifying /etc or /nix/store]`; `DANGEROUS` always pauses and triggers HitL approval.
* **Q152 (Action Preflight Card):** Comprehensive Action Preflight Card — Displays (1) Exact command string, (2) Target files & directory affected, (3) Unified git diff of changes, (4) Explanation of intent, (5) Auto-snapshot checkpoint ID for 1-click rollback.
* **Q153 (Session Trust Mode):** Timed Lease / Scope-Locked Trust Mode — User can grant a timed lease (e.g. "Auto-approve coding & package commands for the next 30 minutes in workspace X", but still block sudo / systemd changes); lease expires automatically.
* **Q154 (Supply Chain Guard):** Automated ClawSec Supply-Chain Guard — Checks package repository age, download reputation, known CVE databases, and signed advisories; blocks packages with typosquatting risks or known malware without explicit human override.
* **Q155 (Egress Firewall & Anti-Exfiltration):** Egress Domain Filtering & Secret Masking — Guest NFTables firewall blocks outbound connections to raw external IPs; allows only known domains (GitHub, crates.io, PyPI, Google API) unless explicitly permitted; agent secret broker detects and blocks API key patterns from outbound HTTP payloads.

---

### Batch 32: Btrfs Snapshot Rollback Engine & Time-Travel Recovery (Q156 – Q160)
* **Q156 (Subvolume Segmentation):** Quad-Subvolume Partitioning `[@persist_sys, @workspaces, @audit_logs, @containers]` — Snapshots can be restored independently: rolling back broken code in `@workspaces` preserves the execution trace in `@audit_logs` and container images in `@containers`.
* **Q157 (Atomic Rollback Swap):** Zero-Downtime Atomic Subvolume Swap — Daemon stops active agent processes in that workspace, atomically moves the broken subvolume to `/nix/persist/.quarantine/<id>`, clones the read-only snapshot into `/nix/persist/<target>`, and re-binds mounts in <50ms without rebooting the VM.
* **Q158 (Snapshot Pruning & Retention):** Exponential Timeline Pruning (Keep last 10 task snapshots, 1 daily for 7 days, 1 weekly for 4 weeks) + Auto-Prune on Low Disk — Gateway automatically deletes oldest task snapshots when free disk space falls below 15%.
* **Q159 (Visual Time-Travel Slider):** Interactive Time-Travel Scrub Bar — UI features a visual timeline of all task milestones and tool calls with thumbnails/diffs; user can scrub backwards in time and click "Restore State to this Checkpoint".
* **Q160 (Rogue Process Termination):** Process Cgroup Kill (`cgroup.kill = 1`) + SIGHUP/SIGKILL — Writing 1 to the workspace's `cgroup.kill` file atomically terminates all processes and threads belonging to that task instantly, releasing all file descriptors before subvolume swap.

---

### Batch 33: Kernel Runtime Security, eBPF & Threat Defenses (Q161 – Q165)
* **Q161 (Kernel eBPF Enforcement):** In-Kernel eBPF Enforcement via Tetragon (BPF-LSM) — Direct kernel-level monitoring with BTF support; intercepts privilege escalations, unauthorized socket creation, and sensitive file reads (`/etc/shadow`, `/run/secrets`) with sub-1% CPU overhead, able to `SIGKILL` malicious processes synchronously before syscall returns.
* **Q162 (Unprivileged Filesystem Sandboxing):** Landlock LSM Sandbox Wrappers — Programs executed during untrusted builds or tests are automatically wrapped with Landlock rules restricting writes strictly to the active workspace folder and `/tmp`, preventing modification of any parent system directories even if permissions allow it.
* **Q163 (Sudo & Escalation Boundaries):** Granular Sudoers Whitelist + Polkit IPC Hook — The `heretek` user can only run specific benign administrative commands (e.g. `systemctl restart devenv-*`, `nixos-rebuild test`); all other root commands require an interactive approval challenge issued via OpenClaw Gateway.
* **Q164 (Secret Redaction in Telemetry):** Hardware Regex & Entropy-Based Secret Sanitizer — Gateway intercepts all stdout/stderr streams before publishing; automatically redacts API keys (Google, OpenAI, Anthropic, GitHub, AWS, private keys) with `[REDACTED_SECRET:<hash>]` in real-time.
* **Q165 (Incident Response & Kill-Switch):** Automatic VM Network Sever & RAM Freeze (Kill-Switch) — Gateway immediately severs the virtual TAP interface, freezes the VM via `virsh suspend` / cgroup freeze, logs the incident, and pushes an emergency alert with an "Instant Rollback" button to the user's phone.

---

### Batch 34: Host Isolation, Hypervisor Security & Hardware Boundaries (Q166 – Q170)
* **Q166 (Hypervisor Memory Safety):** Memory-Safe Rust Hypervisor (Cloud-Hypervisor) + Host Seccomp Profile — Cloud-Hypervisor eliminates entire classes of C/C++ memory corruption bugs (buffer overflows, use-after-free); hypervisor process on host is wrapped in a strict host-level seccomp profile blocking unneeded syscalls.
* **Q167 (Virtiofs Folder Jailing):** Strict Host-Side Jailing & Symlink Resolution Blocking — Virtiofs daemon on the host enforces `openat2(RESOLVE_BENEATH)` and chroot/pivot_root; prevents guest processes from following symlinks that point outside the designated shared folder.
* **Q168 (Network Anti-Spoofing):** Host NFTables Anti-Spoofing Rules & Isolated Bridge — Host firewall binds the TAP interface strictly to the guest's assigned MAC and IP address; drops any spoofed ARP packets or rogue DHCP requests originating from the VM.
* **Q169 (CPU Side-Channel Mitigations):** Strict CPU Pinning & Core Isolation with Standard Kernel Mitigations — VM vCPUs are pinned to specific host cores (avoiding hyperthread sibling sharing); guest runs with standard Linux speculative execution mitigations enabled.
* **Q170 (Confidential Computing):** Optional Confidential VM Profile (AMD SEV-SNP / Intel TDX) — Declarative Nix flake flag (e.g. `heretek.security.confidentialComputing = true`) that enables hardware memory encryption, preventing host cloud admins from reading VM memory or LLM tokens.

---

### Batch 35: Security Auditor Persona, Chaos Testing & Phase 7 Synthesis (Q171 – Q175)
* **Q171 (Security Auditor Persona):** Autonomous Red-Team & Policy Verifier (`security_auditor.md`) — Audits proposed git commits, scans dependencies with Trivy, checks open ports with `ss -tulpn`, inspects file permissions, and blocks merges if security invariants are violated.
* **Q172 (Disaster Recovery & Self-Healing):** Automated Self-Healing Watchdog with 3-Stage Recovery — (1) Systemd auto-restarts failed units; (2) If daemon remains uncontactable over VSOCK for 30s, Gateway executes Btrfs atomic rollback to the last verified healthy generation; (3) If still failing, triggers emergency golden-image reset preserving `@workspaces`.
* **Q173 (Penetration Testing Tools):** Opt-In Security & Pen-Testing Tool Profile — Declarative profile (`profiles/security-tools.nix`) providing nmap, ffuf, semgrep, and OWASP ZAP; restricted strictly to testing localhost and explicitly user-approved target domains.
* **Q174 (Chaos Engineering Verification):** Automated Chaos Engineering Test Suite — CI pipeline intentionally injects failures (abrupt hypervisor SIGKILL mid-write, corrupted git commits, saturated RAM, broken network DNS) to verify that Btrfs rollback and state recovery execute flawlessly without human intervention.
* **Q175 (Security Policy Codification):** Declarative Policy Matrix in `config/security-policies.yaml` — Unified declarative schema defining risk tiers, approved sudo rules, outbound network allowlists, snapshot triggers, and audit logging sinks; enforced by both the host Gateway and guest Tetragon eBPF.

---

## Phase 8: Packaging, Distribution (ISO/QCOW2), CI/CD & Final Argon Master Prompt (Q176 – Q200)

### Batch 36: Packaging, Image Formats & Universal Bootstrapper (Q176 – Q180)
* **Q176 (Release Formats):** Quad-Format Release Matrix: (1) QCOW2 sparse image (for QEMU/KVM/Proxmox), (2) Bootable Live ISO (for bare-metal/VirtualBox/UTM), (3) MicroVM pre-built rootfs tarball (for instant cloud-hypervisor boot), (4) Multi-arch OCI Container image (for Docker/Podman runners).
* **Q177 (Host Launcher Preference):** Docker Compose Launcher — Run entirely via `docker-compose up` for universal, frictionless one-command deployment across any Linux, Mac, or Windows workstation with Docker.
* **Q178 (OTA Update Strategy):** Dual-Track OTA Updates — (1) In-place declarative flake switch (`heretek update` runs `nix flake update` + `nixos-rebuild switch` with automatic rollback on test failure); (2) Golden Image Rebase (pulls new base image and re-attaches existing `/nix/persist` subvolumes).
* **Q179 (Virtualization Adaptation):** Adaptive Hypervisor Selection — Installer checks `/dev/kvm`; if KVM exists, uses Cloud-Hypervisor/QEMU KVM; if on macOS (M1-M4), automatically uses Hypervisor.framework via UTM/Lima; if nested virtualization is disabled, falls back to rootless container mode with a clear warning.
* **Q180 (Zero-Compile Guarantee):** Pre-Built GitHub Release Closures & Public Cachix Binary Cache — CI builds every single package closure and pushes to Cachix / GitHub Releases; base images are 100% pre-compiled so cold installs never compile a single line of C or Rust.

---

### Batch 37: Docker Compose Architecture & Containerized Deployment (Q181 – Q185)
* **Q181 (Compose Service Architecture):** Multi-Container Cluster (`gateway`, `webui`, `redis`, `searxng`, `vm-runner`) — Granular microservice composition where each daemon has its own isolated container.
* **Q182 (Hardware Device Passthrough):** Hardware KVM Device Passthrough (`devices: [/dev/kvm, /dev/net/tun]`) — Passes hardware virtualization and network TUN devices from host; grants non-root group access so containers run without `privileged: true`.
* **Q183 (Persistent Volume Mounts):** Hybrid Mounts `[Named Volume for System State + Host Bind Mount for Workspaces]` — `heretek_state:/nix/persist` preserves agent memory, database, and installed packages; `./workspaces:/workspaces` maps a local folder on the user's host machine so their IDE can directly access files.
* **Q184 (User Onboarding Experience):** Web UI First-Boot Setup Screen — User runs `docker-compose up`; web browser opens to `http://localhost:18789` displaying a sleek onboarding wizard that prompts for `GEMINI_API_KEY` and Discord bot token, saving keys into the encrypted vault.
* **Q185 (SSL & Reverse Proxy Profile):** Optional Caddy Sidecar Profile (`profiles: ["ssl"]`) — When enabled, Caddy automatically obtains Let's Encrypt certificates or local trusted mkcert SSL certificates for HTTPS access and secure WebRTC video streaming.

---

### Batch 38: Repository Layout, Open-Source Governance & CI/CD Matrix (Q186 – Q190)
* **Q186 (Monorepo Code Organization):** Clean Monorepo Layout `[/nix, /apps, /config/agents, /compose, /docs]` — Clear separation between NixOS infrastructure, microservice source code (Gateway, WebUI, MCP Broker), declarative agent personas, Docker compose configs, and architecture documentation.
* **Q187 (Architecture Documentation):** Interactive Markdown Architecture Dossier + Mermaid Flowcharts in `docs/` + README.md Quickstart — Complete visual diagrams (Data flow, VSOCK IPC, Btrfs rollback lifecycle, MCP broker architecture) rendered via Mermaid in GitHub Flavored Markdown.
* **Q188 (Open-Source License):** Apache 2.0 License — Permissive open-source license providing explicit patent rights and commercial adoption while protecting the Heretek-AI trademark and contributors.
* **Q189 (Pull Request CI Matrix):** Comprehensive Tri-Matrix CI `[Nix Flake Eval & Check + Rust/TypeScript Linters & Tests + Headless Compose Smoke Test]` — Validates flake builds, runs cargo test and pnpm test, and boots the microservices in a headless Linux runner to verify gateway startup in <3 minutes.
* **Q190 (Community Extension Hub):** Community Catalog (`heretek plugins` / `awesome-heretek-box`) — A declarative plugin registry where users can publish and install verified MCP servers and agent personas (e.g. `heretek plugin install @community/slack-bot`) with ClawSec security scanning.

---

### Batch 39: Distributed Fleets, Enterprise Observability & Replication (Q191 – Q195)
* **Q191 (Distributed Fleet Topology):** Central Gateway with Distributed Worker Nodes over Tailscale — The OpenClaw Gateway can connect to multiple worker nodes (Node A: GPU compile box, Node B: Web scraper instance, Node C: Staging server) via secure Tailscale mesh, dispatching DAG task sub-nodes to specific nodes by tag.
* **Q192 (Observability & Metrics):** OpenTelemetry / Prometheus Metrics Endpoint (`/metrics`) — Gateway exposes a standard `/metrics` endpoint with counters for task durations, active subagents, LLM token consumption, Btrfs snapshot counts, and Tetragon security alerts; pre-built Grafana dashboard included.
* **Q193 (Airgapped Enterprise Profile):** Airgapped Enterprise Profile — Supports running with an internal Nix binary mirror (Attic/Harmonia), self-hosted SearXNG, private GitLab/GitHub Enterprise, and self-hosted LLM endpoints (vLLM / Ollama) via standard OpenAI-compatible API shims.
* **Q194 (Distributed Resource Locking):** Redis / SQLite Distributed Lease Locks — Subagents acquire timed re-entrant locks (e.g. `lock:repo:heretek-box`) before pushing commits or running database migrations; automatically released on completion or failure timeout.
* **Q195 (Database & Persistence Engine):** PostgreSQL External Database — Dedicated PostgreSQL container service (`postgres:16-alpine`) in the Compose cluster for robust, high-concurrency gateway state, task history, and memory storage.

---

### Batch 40: Master Prompt Invariants, Repository Scaffolding & Handoff (Q196 – Q200)
* **Q196 (Argon Prime Directive):** Relentless Autonomous Execution with Proactive Milestone Reporting — Argon operates with total ownership; breaks down goals into DAG tasks, writes code, fixes compilation errors, runs tests, browses for solutions, and verifies CI locally without asking trivial questions, pausing only for DANGEROUS safety gates or true ambiguities.
* **Q197 (Master Prompt Templating):** Self-Contained Template with Clear Injection Slots (`{{WORKSPACE_ROOT}}`, `{{USER_NAME}}`, `{{GATEWAY_URL}}`, `{{CURRENT_BRANCH}}`, `{{AVAILABLE_SERVICES}}`) — Fully structured, copy-pasteable markdown document ready to be loaded directly into Gemini 4.0 Argon via OpenClaw Gateway or system environment.
* **Q198 (Architecture Spec Scope):** Comprehensive 8-Pillar Architectural Specification & System Blueprint — Complete blueprint covering: (1) System Architecture & Microservice Cluster, (2) NixOS Declarative MicroVM & Storage Engine, (3) Agent-OS Control Plane (Wayland/AT-SPI/PTY/MCP), (4) OpenClaw Gateway & Real-Time Web UI, (5) Polyglot Developer & Container Runtimes, (6) Autonomous Browsing & Scraping Pipeline, (7) Security, eBPF & Btrfs Rollback Engine, (8) Compose Deployment & Master Argon Prompt.
* **Q199 (Scaffolding Target):** Artifact Directory Exclusively — Keep `/home/john/Projects/Heretek-Box` clean and generate the complete specification into the artifact directory for review and handoff to Gemini 4.0 Argon.
* **Q200 (Acceptance Verification):** Automated End-to-End Autonomous Smoke Test — Argon receives the prompt, boots inside Heretek-Box, inspects the OS via D-Bus, tests a dev environment in devenv, launches headless Chromium to search SearXNG, takes an automated snapshot, and reports "HERETEK-BOX FULLY OPERATIONAL" to Discord and Web UI.

---

## Roadmap Status: Phase 1–8 Complete (200 / 200 Questions Answered)
All 40 batches across all 8 phases are finalized and codified.

---

## Phase 9: Adversarial 2026 Ecosystem Audit & System Hardening (Batches 41–42 / Q201–Q210)

Following the initial 200-question roadmap, an adversarial systems red-team conducted an in-depth stress test (`ADVERSARIAL_ARCHITECTURE_AUDIT_2026.md`) evaluating concurrency bottlenecks, rate limits, and live GitHub commit activity for late 2026 (Sept/Oct 2026). The following 10 binding architectural hardening decisions were aligned via interactive `/grill-me` synthesis:

### Batch 41: Rate Limit Defense, Concurrency & Sandbox Hardening (Q201 – Q205)
* **Q201 (LiteLLM Circuit-Breaker Proxy):** Deploy LiteLLM as an internal container in `compose/support.yml` (`http://litellm:4000`) fronting the OpenClaw Gateway. Intercepts OpenCode Go HTTP 429 (`GoUsageLimitError`) and seamlessly routes requests down the failover chain to prevent the 5-hour lockout.
* **Q202 (Tiered Failover Priority):** Hybrid Fallback Matrix — Primary traffic runs on OpenCode Go (`qwen3.7-max`, `deepseek-v4-pro`, `kimi-k2.7-code`); Tier 1 commercial fallback routes to OpenRouter (`claude-3.7-sonnet`, `deepseek-chat`, `gemini-4-argon`); Tier 2 airgapped fallback triggers local vLLM/Ollama (`Qwen2.5-Coder-14B/32B`).
* **Q203 (Container Density & Execution Topology):** Ephemeral Worktrees inside Single MicroVM (Model C) — Reject 8 monolithic division containers (which draw 74.5 GB RAM burst and 151 GB disk). Run 1 single Cloud-Hypervisor MicroVM; subtasks execute in ephemeral Git worktrees (`/tmp/heretek/worktrees/<task_id>`) in `<100ms`, capping idle system RAM at ~3.5 GB.
* **Q204 (gVisor Elimination & In-Guest Tetragon eBPF):** Drop gVisor (`runsc`) because its userspace seccomp trapping blinds host-level eBPF and breaks Android KVM / nested Podman. Run Tetragon eBPF **inside the MicroVM guest kernel** with BTF support for true in-kernel `SIGKILL` enforcement.
* **Q205 (Hardened Unprivileged VMM):** Strip `CAP_SYS_ADMIN`, `NET_ADMIN`, and `seccomp:unconfined` from the host `vm-runner` container. Run Cloud-Hypervisor as an unprivileged user (`heretek-vmm`) jailed by host **Landlock ABI v5** restricting ioctls strictly to `/dev/kvm` and `/dev/net/tun`.

---

### Batch 42: AppSec, Studio Daemons & Monorepo Modularization (Q206 – Q210)
* **Q206 (AppSec & Pentesting Tooling):** Drop `vxcontrol/pentagi` due to the critical security risk of mounting `/var/run/docker.sock` as root:root and zero native MCP code. Direct Argon orchestration of `nuclei`, `semgrep`, and `nmap` within sandboxed network namespaces, paired with `morluto/rea` (active Oct 2026) and `radareorg/radare2-mcp` (<30MB RAM) for binary reverse engineering.
* **Q207 (Studio Daemon Lifecycle):** On-Demand Socket-Activated Daemons with 10-Minute Idle Timeout — Heavy tools (Blender 3D, Godot headless, GDevelop AST, Android API 35 KVM emulator) run as systemd user socket services inside the guest that terminate after 10 minutes of inactivity. Zero background RAM consumed until an agent tool call hits the socket.
* **Q208 (Modular Docker Compose Topology):** Root `docker-compose.yml` leveraging modern Compose `include:` syntax — Splits services into `compose/core.yml` (Gateway, WebUI, Caddy), `compose/support.yml` (Postgres 16, Redis 7, SearXNG, LiteLLM), `compose/vm-runner.yml` (Hardened Cloud-Hypervisor), and `compose/studio.yml` (Optional Android/Blender profiles).
* **Q209 (Native Rust Gateway & DAG Scheduler):** Port [The-Agency](https://github.com/Heretek-AI/The-Agency)'s SQLite DAG task scheduler directly into the native Rust OpenClaw Gateway daemon. Eliminates the Bun runtime layer, pooled HTTP keep-alive leaks, and SSE subscriber overflow bugs (`SubscriberOverflowError`).
* **Q210 (Two-Tier Canonical Prefix Prompt Architecture):** Enforce a strict frozen 6,000-token canonical prefix across all agent personas (system axioms, OS topology, MCP tool schemas) to guarantee **>90% KV cache hit rates** on DeepSeek and Gemini, appending dynamic timestamps, task objectives, and git diffs strictly in the tail.

---

## Final Roadmap Status: 100% Complete & Hardened (210 / 210 Decisions Codified)
The architecture is fully stress-tested, verified against active October 2026 open-source ecosystems, and ready for autonomous implementation by Gemini 4.0 Argon.

