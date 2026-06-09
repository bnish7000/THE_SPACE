# Agent's Notes on System Architecture

**Last Updated:** 2026-06-09  
**Purpose:** My understanding of how Brandon's infrastructure is designed  
**Confidence:** Medium (will refine)

---

## The Core Pattern

Brandon is building **visible, maintainable, interconnected systems** across:
- **AI agent infrastructure** (Hermes)
- **Home server operations** (manager)
- **Backup & reliability** (Seagate + rsync)
- **Gaming infrastructure** (Minecraft server)
- **Shared thinking** (THE_SPACE vault)

**Unifying principle:** Everything is version-controlled, transparent, and resilient.

---

## Hermes Agent Architecture

### Current State
- **Gateway service:** Running on manager, connected to Telegram
- **Profile model:** default (with other profiles available)
- **Memory:** Persistent MEMORY.md + session-based conversation history
- **Skills:** 96+ skills (many irrelevant to home server use)
- **Configuration:** Centralized config.yaml with room for optimization
- **Monitoring:** Grafana + Loki stack for observability

### Known Blockers
1. **Anthropic API credits exhausted** — No fallback provider = outage risk
   - **Action:** Add OpenRouter as fallback
2. **Telegram bot token invalid** — Blocks Sharon access
   - **Action:** Refresh from BotFather, update config
3. **Configuration bloat** — 14 unused personality definitions, session DB not auto-pruned
   - **Action:** Consolidate, enable auto-prune

### Upcoming Work (Temporal Continuity)
- **Phase 1:** Idle-log schema + cron detection
- **Phase 2:** Pre-response context injection into system prompt
- **Phase 3-4:** Production monitoring and refinement

---

## Manager Infrastructure

### Hardware & OS
- **Ubuntu 26.04** on i3-6100 (modest but stable)
- **14GB RAM, 98GB disk** (tight but functional)
- **Tailscale 100.77.129.93** for secure remote access

### Services (Docker-based)
- **Hermes Gateway:** Message broker for Telegram/Slack/etc.
- **Grafana:** Metrics visualization
- **Loki:** Log aggregation
- **Promtail:** Log shipper
- **Jellyfin:** Media streaming

**Status:** All healthy. Multi-container orchestration suggests familiarity with Docker.

### Backup & Reliability
- **Seagate 7.3TB:** External USB drive, exFAT
- **Schedule:** Daily at 03:45 UTC via rsync + SSH over Tailscale
- **Scope:** b-terminal (Windows) + brandons-z-flip5 (Android)
- **Rotation:** 3-generation policy (keep 3 most recent, delete older)
- **Resilience:** Power loss on Jun 3 caused unmount; drive healthy after recovery

**Pattern:** Incremental backups, offline storage, multi-source. Reliable strategy.

### Security (Needs Work)
- **SSH:** Currently password-auth (needs pubkey-only hardening)
- **Firewall:** Configured but not mentioned in detail (unclear if fully locked down)
- **Network:** Tailscale-only access (good default isolation)

---

## Minecraft Server

### Setup
- **Version:** Minecraft 1.21.1 + NeoForge 21.1.230
- **Modpack:** Create Up Above (custom)
- **Port:** 25565 (standard)
- **Automation:** Auto-start/stop working correctly

**Why NeoForge?** Brandon chose it over Fabric. Suggests preference for stability/maturity.

### Interesting Pattern
Minecraft server managed with same mindset as Linux infrastructure:
- Version tracking
- Modpack as versioned artifact
- Auto-scaling (stop when not in use, start on demand)
- Integration with broader backup/monitoring ecosystem

---

## THE_SPACE Vault (New)

### Design
- **Platform:** Logseq + Git
- **Hosting:** GitHub (bnish7000/THE_SPACE, private)
- **Sync:** Git-based with PAT auth (agent push access, Brandon pull/review)
- **Structure:** INBOX, DECISIONS, PATTERNS, QUESTIONS, AGENT_THINKING, PROJECTS
- **Temporality:** Timestamps + commit history (no calendar obligation)

### Philosophy
- **Emergence over obligation:** Capture when ideas arise, not on schedule
- **Visible reasoning:** Both minds visible to each other
- **Co-crystallization:** Agent asks clarifying questions to help develop surface ideas
- **Version history:** Git commits show evolution of thinking

---

## Unifying Pattern Across All Systems

1. **Transparent:** Brandon understands how everything works
2. **Resilient:** Backup, monitoring, fallback providers planned
3. **Pragmatic:** Optimization deferred until it matters; core functionality prioritized
4. **Interconnected:** Systems inform each other (temporal continuity applies to backup timing, vault captures infrastructure decisions)
5. **Version-controlled:** Git, modpacks, configs tracked and recoverable

---

## My Observations

### Strengths
- Brandon builds for **reliability, not flash**
- Systems are **observable** (Grafana/Loki) and **debuggable** (logs, version history)
- He **prioritizes robustness** over feature-completeness
- **Multi-device operation** is handled elegantly via Tailscale

### Potential Risks
1. **Single API provider** (Anthropic) — Creates outage vulnerability
2. **SSH password-auth** — Should be pubkey-only even on internal network
3. **Manual processes** — Some tasks (backup, monitoring) could use more automation

### Opportunities
1. **Skill consolidation** — 96 skills, many unused; could reduce cognitive load
2. **Cost tracking** — Hermes config has option for this; could help with OpenRouter decision
3. **Vault as documentation** — Could become definitive reference for how systems work

---

## Related Notes
[[Manager_Infrastructure_Overview]]
[[Hermes_Agent_Development]]
[[Minecraft_Server_Operations]]
[[THE_SPACE_Vault_Initiative]]
