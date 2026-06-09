# Hermes Agent Development & Enhancements

**Status:** ACTIVE  
**Focus Areas:** Temporal awareness, skill development, configuration optimization  
**Last Updated:** 2026-06-09

## Current Initiatives

### 1. Temporal Continuity System (Phase 1)
**Owner:** Brandon + Agent  
**Status:** Phase 1 - Foundation (design approved)  
**Target:** Implement idle-log with farewell tagging

**What it does:**
- Tracks conversation gaps with precise timestamps
- Distinguishes explicit farewells from silent timeouts
- Agent reads idle-log before responding, achieving temporal awareness
- Removes daily session resets

**Phases:**
- Phase 1 (Day 1): Schema, cron job, manual testing
- Phase 2 (Day 2-3): Agent integration, pre-response hook
- Phase 3 (Day 4+): Production, monitoring, rotation
- Phase 4 (Ongoing): Refinement based on user feedback

**Related:** [[Temporal_Continuity]] decision doc

### 2. THE_SPACE Vault (Logseq)
**Owner:** Agent (primary builder), Brandon (reviews, adds as comfortable)  
**Status:** ACTIVE (Just Initiated)  
**Target:** Co-thinking space with visible reasoning

**What it does:**
- Shared knowledge base for Brandon + Agent ideas
- Agent scaffolds surface ideas into crystallized thinking
- Both minds visible—mutual learning and critique
- No calendar pressure (capture on emergence)

**Structure:**
- INBOX/ — Raw thoughts
- DECISIONS/ — Formalized choices
- PATTERNS/ — Recurring insights
- QUESTIONS/ — Open threads
- AGENT_THINKING/ — Agent reasoning
- PROJECTS/ — Active work

**Related:** [[THE_SPACE_Vault_Initiative]]

### 3. Configuration & Optimization (Backlog)
**Owner:** Agent  
**Status:** TODO (pending cost tracking priority)

**Known optimizations:**
- Remove 14 unused personality definitions (keep kawaii only)
- Enable session auto-prune (prevent DB bloat)
- Enable cost tracking display (monitor API budget)
- Skill curation (96 skills, many irrelevant to home server use)

**Related:** [[Hermes_Config_Optimization]]

## Recent Milestones
- 2026-06-09: Temporal continuity design finalized; Logseq vault initiated
- 2026-06-05: Timezone configured (PST per Brandon preference)
- 2026-05-31: Hermes config audit completed; blockers identified

## Blockers
- **Anthropic API credits exhausted** — No fallback provider currently; blocks outages
  - **Solution needed:** Add OpenRouter as fallback
- **Telegram bot token invalid** — Sharon cannot access Hermes (pending token refresh from BotFather)

## Related Notes
[[System_Architecture_Overview]]
[[Hermes_Config_Optimization]]
[[Brandon_Communication_Style]]
