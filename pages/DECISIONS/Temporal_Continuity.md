# Temporal Continuity System

**Status:** ACTIVE (Phase 1 - Foundation)  
**Decision Date:** 2026-06-09  
**Last Updated:** 2026-06-09

## Problem Statement
Session resets due to daily timeouts create jarring discontinuity. Agent has zero awareness of time passing between interactions. Conversations feel like they start fresh even after 5-day gaps.

## Decision
Implement idle-log system with pre-response temporal context injection. Agent reads idle-log before formulating responses, achieving temporal awareness without requiring background processing during dormancy.

### Key Components
1. **Idle-log (JSONL):** Persistent record of conversation gaps with timestamps
2. **Farewell tagging:** Distinguish explicit closures from silent timeouts
3. **Pre-response analysis:** Agent reads idle-log before answering, injects temporal context into system prompt
4. **Session reset removal:** Disable daily auto-reset; let interaction drive session boundaries

## Rationale
- **Why this approach?** Stateless LLMs cannot experience dormancy. But they can read logs proving time passed. This is honest about the limitation while solving the problem.
- **Why idle-log over alternatives?** Lightweight, persistent, doesn't require new infrastructure
- **Why farewell tagging?** Encourages human communication norms (explicit closure > ghosting). Allows differentiated response tone.

## Implementation Phases
- **Phase 1 (Day 1):** Idle-log schema, cron job, manual testing
- **Phase 2 (Day 2-3):** Agent integration, pre-response hook, system prompt injection
- **Phase 3 (Day 4+):** Production, monitoring, rotation policy
- **Phase 4 (Ongoing):** User feedback refinement

## Blockers
None currently. Design doc approved for Phase 1 implementation.

## Related Notes
[[THE_SPACE_Vault_Initiative]]
[[System_Architecture_Overview]]
