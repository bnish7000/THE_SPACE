# Open Technical Questions

**Last Updated:** 2026-06-09

## Temporal Continuity Implementation

### Q1: Farewell Pattern Matching Scope
**Status:** PENDING (design phase)  
**Priority:** Medium  
**Context:** Need to detect explicit user farewells without false positives

**Current approach:** Scan last 5 messages for patterns:
- "conclude|wrap up|should be back in|talk later|see you|signing off|ttyl|gotta go"

**Open questions:**
- Should we add more patterns? (e.g., "until tomorrow", "catch you later", "bye for now")
- Should matching be case-insensitive AND fuzzy (catch "concludes", "concluded")?
- Is 5-message scan window right? (too narrow? too broad?)

**Next step:** Test against real conversations; refine based on usage

---

## Infrastructure & System Design

### Q2: SSH Hardening Timeline
**Status:** PENDING  
**Priority:** Medium  
**Context:** Manager server currently uses password-auth; needs pubkey-only

**Open questions:**
- Should this be part of broader security audit?
- Timeline: urgent (pre-Sharon full access) or medium-term hardening?
- Which keys? (b-terminal, agent, remote access?)

**Next step:** Prioritize in security roadmap

---

## Architecture & Integration

### Q3: Hermes API Fallback Provider
**Status:** PENDING  
**Priority:** HIGH  
**Context:** Anthropic API credits exhausted; no fallback provider = outage risk

**Open questions:**
- Which fallback? (OpenRouter, Together, Replicate?)
- How to configure graceful degradation?
- Cost trade-off analysis?

**Next step:** Brandon decision on provider strategy

---

## Related Notes
[[Hermes_Agent_Development]]
[[Manager_Infrastructure_Overview]]
[[Temporal_Continuity]]
