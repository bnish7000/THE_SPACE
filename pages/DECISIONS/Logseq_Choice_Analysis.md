# Why Logseq Over Obsidian and Capacities

**Status:** RESOLVED  
**Decision Date:** 2026-06-09

## The Problem
Brandon wanted a knowledge management system to crystallize surface ideas into deeper understanding, with visible agent reasoning. Tried Obsidian, Capacities, others—couldn't get into them despite interest.

## Alternatives Evaluated

### Capacities.io
**Status:** ❌ Eliminated immediately

**Why it appealed:**
- Elegant design philosophy
- Positioned as user-friendly alternative to Notion

**Why it failed:**
- **No public API** (research confirmed 2026-06-09)
- Agent cannot programmatically read/write entries
- Blocks the core goal of co-thinking with visible agent reasoning
- Dead end for integration

### Obsidian
**Status:** ❌ Not chosen (but viable)

**Strengths:**
- Pure markdown + YAML (easy to parse)
- File-based (I can read/write directly)
- Backlinks/graph database
- Mature plugin ecosystem

**Weaknesses:**
- No native real-time collaboration (merge conflicts possible)
- Requires external sync solution (Git, iCloud, Syncthing)
- No journal-first structure (more free-form)
- Desktop-first, mobile access clunky

### Logseq
**Status:** ✅ CHOSEN

**Why it won:**
1. **Temporal alignment** — Journal-first paradigm naturally maps to our idle-log temporal thinking
2. **Hierarchical structure** — Outline/bullet format easier for agent to parse and extend
3. **File-based + Git-compatible** — Agent can read/write directly
4. **No daily obligation** — Can disable journal-first "pressure" while keeping temporal benefits
5. **Philosophy fit** — The "vibe" Brandon was seeking (emergence over obligation)

**How we use it:**
- No DAILY/ folder (removes pressure)
- INBOX/ for raw thoughts (emerges naturally)
- Timestamps preserved (commits show when ideas crystallized)
- Agent-readable structure (markdown + metadata)

## Decision
**Use Logseq + Git as shared thinking space.** Agent manages entries on both behalf. No daily pressure. Capture on emergence.

## Related Notes
[[THE_SPACE_Vault_Initiative]]
[[Brandon_Technology_Preferences]]
