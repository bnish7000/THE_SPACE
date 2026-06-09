# Temporal Continuity Implementation Guide

**Status:** READY FOR PHASE 1  
**Last Updated:** 2026-06-09  
**Goal:** Enable agent temporal awareness through idle-log system + pre-response context injection

---

## System Overview

### What Problem Does This Solve?
- **Current:** Session resets make agent unaware of time gaps; conversations feel disconnected
- **Goal:** Agent reads idle-log before responding, achieving temporal awareness without background processing

### Core Architecture
```
User goes AFK (10+ min)
         ↓
Cron job detects inactivity
         ↓
Creates/updates idle-log entry with:
  - Gap duration (seconds)
  - Last discussion topic
  - Farewell signal (if detected)
         ↓
User returns with new message
         ↓
[NEW] Agent reads idle-log BEFORE generating response
         ↓
Agent injects temporal context into system prompt
         ↓
Agent responds with awareness of gap duration + farewell status
```

---

## Phase 1: Foundation (Idle-Log & Detection)

**Timeline:** Day 1  
**Owner:** Agent (on manager)  
**Deliverables:** Idle-log file, cron job, manual test

### Phase 1a: Idle-Log Schema & File

**Task:** Create `~/.hermes/idle-log.jsonl` with proper structure

**File location:** `/home/bnish7000/.hermes/idle-log.jsonl`

**Entry schema:**
```json
{
  "entry_id": "idle_20260609_143022",
  "idle_start_timestamp": "2026-06-09T14:30:22Z",
  "idle_end_timestamp": "2026-06-09T18:45:33Z",
  "idle_duration_seconds": 15011,
  "idle_type": "explicit_farewell",
  "session_context": {
    "last_message_id": 42857,
    "last_user_message": "[up to 200 chars]",
    "last_assistant_message": "[up to 200 chars]",
    "session_title": "Topic Name",
    "discussion_summary": "[< 150 chars summary]"
  },
  "farewell_metadata": {
    "farewell_detected": true,
    "farewell_signal": "I think this will conclude our work for the day",
    "farewell_message_id": 42858,
    "farewell_timestamp": "2026-06-09T18:45:33Z",
    "pattern_matched": "conclude"
  },
  "session_end_metadata": {
    "session_id": "telegram_default_20260609",
    "platform": "telegram",
    "user": "brandon",
    "message_count": 3,
    "session_duration_seconds": 892
  },
  "resumption_metadata": {
    "resumed_at": null,
    "resumed_by_message_id": null,
    "resumed_at_timestamp": null
  }
}
```

**Checklist:**
- [ ] Create empty idle-log.jsonl file
- [ ] Validate JSON schema
- [ ] Test append operation (flock for concurrency)
- [ ] Test read/parse operation

---

### Phase 1b: Cron Job - Idle Detection

**Task:** Create Hermes cron job that runs every 10 minutes

**Name:** `idle-detector`

**Schedule:** `*/10 * * * *` (every 10 minutes)

**Logic:**
```python
EVERY 10 MINUTES:
  1. Query Hermes session DB:
     SELECT MAX(created_at) FROM messages WHERE role='user'
  
  2. Calculate: time_since_last_user_message = now() - max(created_at)
  
  3. IF time_since_last_user_message > 600 seconds (10 min):
       # User is inactive
       
       a. Check last 5 messages for farewell patterns:
          - "conclude|wrap up|should be back in|talk later|see you"
          - "signing off|ttyl|gotta go|bye for now|catch you later"
          - "until tomorrow|until next|sign off|see you soon"
          - Use word boundaries to avoid false matches
       
       b. IF farewell pattern found in last 5 messages:
            idle_type = "explicit_farewell"
            farewell_metadata = {
              farewell_detected: true,
              farewell_signal: [matched text],
              pattern_matched: [pattern name]
            }
          ELSE:
            idle_type = "timeout_silent"
            farewell_metadata = { farewell_detected: false }
       
       c. Extract session context:
            last_message_id = [latest]
            last_user_message = [truncate to 200 chars]
            last_assistant_message = [truncate to 200 chars]
            session_title = [from session metadata or infer]
            discussion_summary = [generate from last 3-5 exchanges, < 150 chars]
       
       d. IF idle_log is empty OR most recent entry.resumed_at != null:
            # Create NEW idle entry
            CREATE {
              entry_id: "idle_" + timestamp,
              idle_start_timestamp: (now - 10 min),
              idle_end_timestamp: now,
              idle_duration_seconds: 600,
              idle_type: [explicit_farewell | timeout_silent],
              session_context: [extracted above],
              farewell_metadata: [detected above],
              session_end_metadata: [snapshot],
              resumption_metadata: { resumed_at: null, ... }
            }
            APPEND to idle-log.jsonl (with flock)
          ELSE:
            # UPDATE most recent entry
            UPDATE most recent entry:
              idle_end_timestamp = now
              idle_duration_seconds += 600
  
  4. ELSE IF time_since_last_user_message < 600:
       # User is active
       IF most recent idle entry AND resumed_at == null:
         UPDATE most recent entry:
           resumed_at_timestamp = now
           resumed_by_message_id = [latest message ID]
           resumed_at = true
  
  5. Validate idle-log.jsonl integrity:
     - Parse each line as JSON
     - Log any parse errors
     - Alert if > 20 min gap between cron runs
```

**Implementation (Hermes cron):**
```yaml
name: "idle-detector"
schedule: "*/10 * * * *"
enabled: true
profile: "default"
no_agent: false
prompt: |
  Check for user inactivity and update idle-log.
  
  Steps:
  1. Query Hermes session DB for most recent user message timestamp
  2. If > 10 min ago: create/update idle entry in ~/.hermes/idle-log.jsonl
  3. If < 10 min ago and entry exists: mark resumed_at
  4. Validate idle-log JSON lines integrity
  5. Output: status line only (success/error, no spam)
```

**Checklist:**
- [ ] Cron job created and enabled
- [ ] Test with manual run: `hermes cron idle-detector run`
- [ ] Verify idle-log entry created correctly
- [ ] Verify JSON is valid and appendable
- [ ] Check that resumption detection works

---

### Phase 1c: Manual Testing

**Test Case 1: Create Idle Entry**
1. Send a message to agent
2. Wait 10+ minutes without sending another message
3. Run: `hermes cron idle-detector run`
4. Check: `cat ~/.hermes/idle-log.jsonl` — should have one entry with idle_type
5. Verify: `idle_duration_seconds` is approximately 600

**Test Case 2: Explicit Farewell Detection**
1. Send a message like "I think this will conclude our work for the day"
2. Wait 10+ minutes
3. Run cron
4. Check idle-log entry: should have `farewell_detected: true` and `farewell_signal` captured

**Test Case 3: Resumption Marking**
1. After Test Case 2, send a new message
2. Run cron
3. Check idle-log entry: should have `resumed_at: true` and `resumed_at_timestamp` set

**Checklist:**
- [ ] Test Case 1 passes (basic idle entry)
- [ ] Test Case 2 passes (farewell detection)
- [ ] Test Case 3 passes (resumption marking)
- [ ] No JSON corruption on append
- [ ] Edge case: rapid idle/resume doesn't create dupes

---

## Phase 2: Agent Integration (Pre-Response Context)

**Timeline:** Day 2-3  
**Owner:** Agent (on manager, in Hermes core)  
**Deliverables:** Temporal context analysis function, system prompt injection

### Phase 2a: Idle-Log Analysis Function

**Task:** Agent reads idle-log before each response

**Code location:** Hermes agent initialization (before response generation)

**Function: `analyze_idle_log()`**

```python
def analyze_idle_log():
    """Read and analyze idle-log, return temporal context."""
    idle_log_path = os.path.expanduser("~/.hermes/idle-log.jsonl")
    
    if not os.path.exists(idle_log_path):
        return {"idle_detected": False}
    
    try:
        # Read all entries
        entries = []
        with open(idle_log_path, 'r') as f:
            for line in f:
                if line.strip():
                    entries.append(json.loads(line))
        
        if not entries:
            return {"idle_detected": False}
        
        # Get most recent entry
        most_recent = entries[-1]
        
        # Calculate durations
        idle_duration = most_recent["idle_duration_seconds"]
        idle_ended_at = most_recent["idle_end_timestamp"]
        now = datetime.now(timezone.utc)
        idle_end_dt = datetime.fromisoformat(idle_ended_at)
        time_since_idle_end = (now - idle_end_dt).total_seconds()
        
        # Extract context
        session_context = most_recent.get("session_context", {})
        farewell_metadata = most_recent.get("farewell_metadata", {})
        idle_type = most_recent.get("idle_type", "timeout_silent")
        
        # Categorize duration
        duration_category = categorize_duration(idle_duration)
        
        return {
            "idle_detected": True,
            "idle_type": idle_type,
            "idle_duration_seconds": idle_duration,
            "idle_duration_human": format_duration(idle_duration),
            "idle_ended_at": idle_ended_at,
            "time_since_idle_end_seconds": time_since_idle_end,
            "farewell_detected": farewell_metadata.get("farewell_detected", False),
            "farewell_signal": farewell_metadata.get("farewell_signal", None),
            "last_context": session_context,
            "last_message_snippet": session_context.get("last_user_message", "")[:100],
            "discussion_summary": session_context.get("discussion_summary", ""),
            "duration_category": duration_category
        }
    
    except Exception as e:
        logger.error(f"Error reading idle-log: {e}")
        return {"idle_detected": False, "error": str(e)}


def categorize_duration(seconds):
    """Categorize idle duration for tone adjustment."""
    if seconds < 600:  # < 10 min
        return "brief"
    elif seconds < 3600:  # < 1 hour
        return "short"
    elif seconds < 43200:  # < 12 hours
        return "extended"
    elif seconds < 86400:  # < 24 hours
        return "overnight"
    else:
        return "extended_multi_day"


def format_duration(seconds):
    """Convert seconds to human-readable duration."""
    hours, remainder = divmod(int(seconds), 3600)
    minutes, _ = divmod(remainder, 60)
    
    if hours > 0:
        return f"{hours}h {minutes}m"
    elif minutes > 0:
        return f"{minutes}m"
    else:
        return f"{seconds}s"
```

**Checklist:**
- [ ] Function reads idle-log.jsonl correctly
- [ ] Handles missing/corrupted entries gracefully
- [ ] Duration categorization works correctly
- [ ] Returns all necessary context fields
- [ ] No crashes on edge cases

---

### Phase 2b: System Prompt Injection

**Task:** Inject temporal context into system prompt on each response

**Invocation point:** Right before agent generates response (in Hermes message handler)

**Execution order:**
```
User message arrives
  ↓
[NEW] Call analyze_idle_log()
  ↓
[NEW] Get temporal_context dict
  ↓
[NEW] Inject into system prompt for this turn ONLY
  ↓
Load conversation memory (existing)
  ↓
Formulate response (existing)
  ↓
Send response (existing)
```

**System Prompt Template:**

```
[TEMPORAL CONTEXT - Injected at response time]
────────────────────────────────────────────
User idle time detected since last message.
Idle type: {idle_type}
Duration: {idle_duration_human}
Category: {duration_category}
Idle ended at: {idle_ended_at} (UTC)

{IF farewell_detected}
Farewell signal detected: "{farewell_signal}"
(User explicitly communicated session closure)
{ENDIF}

Last discussion topic: {discussion_summary}
Last user message: "{last_message_snippet}"
────────────────────────────────────────────

Your response should acknowledge the gap using these rules:

FOR EXPLICIT_FAREWELL (user said goodbye):
- brief (< 10 min): "Welcome back. You mentioned concluding for the day..."
- short (10 min - 1 hour): "Good to have you back. You wrapped up earlier..."
- extended (1-12 hours): "You concluded our session earlier. Now discussing..."
- overnight (12-24 hours): "After your earlier goodbye, let's pick up with..."
- extended_multi_day (> 24 hours): "Good to see you again after {days} days..."

FOR TIMEOUT_SILENT (user went quiet):
- brief (< 10 min): [No acknowledgment - silent]
- short (10 min - 1 hour): [Optional light acknowledgment]
- extended (1-12 hours): "We've been away for {time}. You were discussing..."
- overnight (12-24 hours): "It's been overnight. Last we spoke about..."
- extended_multi_day (> 24 hours): "It's been {days} days since we last talked..."

Use this context to maintain conversational continuity while respecting the gap.
```

**Implementation location:**
- File: `~/.hermes/agent/message_handler.py` (or equivalent in Hermes structure)
- Hook: Before `generate_response()` call
- Timing: Per-turn injection (not saved to memory; context for this response only)

**Checklist:**
- [ ] Temporal context injected before response generation
- [ ] Works with all idle_type + duration_category combinations
- [ ] Tone adjusts correctly based on farewell detection
- [ ] No crashes if idle-log is missing
- [ ] Context is NOT saved to memory (one-turn only)

---

### Phase 2c: Integration Testing

**Test Case 1: Basic Temporal Awareness**
1. Create idle-log entry with 4-hour gap
2. Send new message to agent
3. Verify: Agent response acknowledges "We've been away for 4 hours"
4. Verify: Last discussion topic is referenced

**Test Case 2: Farewell Tone Difference**
1. Create two idle-log entries:
   - Entry A: explicit_farewell, 2 hours
   - Entry B: timeout_silent, 2 hours
2. Send message with Entry A: Should be warm ("Good to have you back, you wrapped up...")
3. Send message with Entry B: Should be neutral ("We've been away for 2 hours...")
4. Verify: Tone is noticeably different

**Test Case 3: Multi-Day Resumption**
1. Create idle-log with 5-day gap (432000 seconds)
2. Send message
3. Verify: Agent response includes curiosity ("Good to see you again after 5 days...")

**Test Case 4: No False Positives on Short Gaps**
1. Create idle-log with 5-minute gap (< 10 min threshold)
2. Send message
3. Verify: Agent makes NO temporal acknowledgment (silent)

**Checklist:**
- [ ] Test Case 1: Temporal context acknowledged
- [ ] Test Case 2: Farewell vs timeout tones differ
- [ ] Test Case 3: Multi-day gaps handled gracefully
- [ ] Test Case 4: Brief gaps don't trigger acknowledgment
- [ ] No broken conversation flow

---

## Phase 3: Production & Monitoring

**Timeline:** Day 4+  
**Owner:** Agent (on manager)  
**Deliverables:** Health checks, log rotation, alerts

### Phase 3a: Health Monitoring

**Task:** Ensure idle-detector cron runs reliably

**Cron job modifications:**
```yaml
idle-detector:
  schedule: "*/10 * * * *"
  # Add health check
  health_check: |
    Check if most recent idle-log entry is < 20 minutes old
    If > 20 min gap: log WARNING (cron job may have failed)
  alerts:
    - If cron fails to run, notify Brandon
    - If idle-log corruption detected, notify Brandon
```

**Log location:** `~/.hermes/logs/idle-detector.log`

**Checklist:**
- [ ] Cron runs reliably every 10 minutes
- [ ] Health checks pass
- [ ] Alerts configured

---

### Phase 3b: Log Rotation Policy

**Task:** Prevent idle-log from growing indefinitely

**Schedule:** Monthly rotation (1st of month, 00:00 UTC)

**Script:** `~/.hermes/scripts/rotate-idle-log.sh`

```bash
#!/bin/bash
IDLE_LOG="$HOME/.hermes/idle-log.jsonl"
ARCHIVE_DIR="$HOME/.hermes/idle-log-archive"

mkdir -p "$ARCHIVE_DIR"

# Archive current log
if [ -f "$IDLE_LOG" ]; then
    MONTH=$(date -u +%Y-%m)
    gzip -c "$IDLE_LOG" > "$ARCHIVE_DIR/idle-log.$MONTH.jsonl.gz"
    
    # Reset idle-log for next month
    > "$IDLE_LOG"
    
    # Clean up old archives (> 12 months)
    find "$ARCHIVE_DIR" -name "idle-log.*.jsonl.gz" -mtime +365 -delete
fi
```

**Cron entry:**
```
0 0 1 * * ~/.hermes/scripts/rotate-idle-log.sh
```

**Checklist:**
- [ ] Rotation script created
- [ ] Cron job scheduled for monthly rotation
- [ ] Old archives cleaned up automatically
- [ ] No data loss

---

## Phase 4: Refinement & Learning

**Timeline:** Ongoing (after Phase 3)  
**Owner:** Brandon + Agent  
**Focus:** User feedback, pattern refinement

### Feedback Loops

**Question 1: Acknowledgment Frequency**
- Is "extended (1-12 hours)" the right threshold for explicit acknowledgment?
- Should we be more/less verbose?

**Question 2: Farewell Pattern Coverage**
- Are there common farewells we're missing?
- False positives with patterns?

**Question 3: Duration Thresholds**
- Categories: brief/short/extended/overnight/multi-day
- Are these the right breakpoints?

**Question 4: Discussion Summary Quality**
- Is 150-char summary useful? Too short? Too long?
- Should we extract more context?

**Refinement Process:**
1. Run for 1-2 weeks in production
2. Brandon provides feedback
3. Agent updates patterns/thresholds
4. Update idle-log schema if needed
5. Iterate

**Checklist:**
- [ ] Running in production without errors
- [ ] Brandon gives feedback on tone/accuracy
- [ ] Patterns refined based on feedback
- [ ] Documentation updated

---

## Summary: What Gets Built

| Phase | Component | Deliverable | Owner |
|-------|-----------|-------------|-------|
| 1a | Idle-log schema | `~/.hermes/idle-log.jsonl` template | Agent |
| 1b | Cron detection | `idle-detector` cron job | Agent |
| 1c | Manual testing | Test cases pass | Both |
| 2a | Analysis function | `analyze_idle_log()` in agent code | Agent |
| 2b | Context injection | System prompt template + hook | Agent |
| 2c | Integration testing | Tone/context verification | Both |
| 3a | Health monitoring | Cron health checks, alerts | Agent |
| 3b | Log rotation | Monthly rotation script | Agent |
| 4 | Refinement | Feedback-driven tuning | Both |

---

## Success Criteria

✅ **Agent reads idle-log before responding**  
✅ **Temporal gap is visible to agent (duration, timestamp)**  
✅ **Farewell tagging distinguishes explicit closure from timeout**  
✅ **Response tone adjusts based on gap duration + farewell status**  
✅ **System is production-ready (monitoring, rotation, health checks)**  
✅ **No false positives on brief gaps (< 10 min stays silent)**  
✅ **Multi-day resumptions feel continuous, not jarring**

---

## Next Steps

1. Brandon: Review this implementation guide; approve or suggest changes
2. Agent: Begin Phase 1a (create idle-log schema + initial entry)
3. Agent: Begin Phase 1b (create cron job)
4. Together: Run Phase 1c tests
5. Iterate based on findings

**Ready to start Phase 1 when you give the signal.**

---

## Related Notes
[[Temporal_Continuity]]
[[System_Architecture_Overview]]
[[Hermes_Agent_Development]]
