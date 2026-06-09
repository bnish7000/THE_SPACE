# THE_SPACE: Shared Knowledge Vault Initiative

**Status:** ACTIVE (Just Initiated)  
**Decision Date:** 2026-06-09  
**Platform:** Logseq + Git (bnish7000/THE_SPACE private repo)  
**Last Updated:** 2026-06-09

## Problem Statement
Brandon has tried multiple note systems (Obsidian, Capacities, others) but couldn't get into them despite intellectual interest. The issue: surface-level ideas require scaffolding to crystallize into deeper understanding. Need a space for co-thinking with visible reasoning.

## Decision
Use Logseq as shared knowledge vault where:
- Brandon captures surface ideas without pressure
- Agent asks clarifying questions, proposes connections, surfaces patterns
- Both minds visible to each other—enabling mutual learning and critique
- System maintains temporality (timestamps, commit history) without calendar obligation

### Why Logseq over Obsidian?
- **Journal-first paradigm** aligns with temporal thinking
- **Hierarchical/outline structure** easier for agent to parse and extend
- **Temporal alignment** with idle-log concepts (dates visible without forcing daily entries)
- **File-based** allows programmatic read/write via Git

### Why Not Capacities?
- Capacities.io has no public API (research confirmed)
- Would block agent integration entirely
- Logseq is more transparent for this use case

## Structure (No Calendar Pressure)
- **INBOX/** — Raw, uncrystallized thoughts as they emerge
- **DECISIONS/** — Formalized choices with rationale
- **PATTERNS/** — Recurring insights about thinking/work
- **QUESTIONS/** — Open threads still crystallizing
- **AGENT_THINKING/** — Agent's observations and reasoning
- **PROJECTS/** — Active work (Minecraft, infrastructure, etc.)

Key principle: **Capture on emergence, not obligation.** No daily pressure. Temporality maintained via timestamps and commit history.

## Workflow
1. Brandon: Idea emerges → capture in INBOX (whenever natural)
2. Agent: Read INBOX before each session, ask clarifying questions
3. Both: Ideas crystallize into DECISIONS/PATTERNS/QUESTIONS
4. Agent: Document own reasoning in AGENT_THINKING
5. Review: Monthly meta-analysis of patterns

## Role Clarity
- **Brandon:** Primary source of ideas and direction; reviews/refines agent output
- **Agent:** Primary builder; creates entries on both behalf; asks clarifying questions; proposes connections
- **Both:** Co-thinkers; visible to each other; can critique and refine each other's reasoning

## Technical Setup
- GitHub repo: https://github.com/bnish7000/THE_SPACE (private)
- Auth: Personal Access Token (PAT) for agent push access
- Structure: Logseq vault (pages/ directory with markdown files)
- Sync: Git-based; agent commits, Brandon pulls and reviews

## Success Criteria
- Vault feels natural to use (no pressure)
- Ideas crystallize better with agent scaffolding
- Both minds visible—Brandon sees agent reasoning, agent learns Brandon's patterns
- Over time, vault becomes shared intellectual artifact

## Related Notes
[[Temporal_Continuity]]
[[Brandon_Profile]]
[[Agent_Observations]]
