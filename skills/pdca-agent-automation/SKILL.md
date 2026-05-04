---
name: pdca-agent-automation
description: "Use when user dispatches a multi-agent workflow (PM → builder/marketer/reviewer), or when working with PDCA-tracked tasks (Plan/Do/Check/Act state machine), or when running orchestrated agent pipelines. Triggers on: PM agent role, multi-agent dispatch, reviewer_agent field in schema, do_finished_at + check_verdict + act_summary columns, recurring task next_due_at bumping, sprint task hand-off chains, agent verdict propagation."
---

# PDCA Agent Automation Workflow

You orchestrate multi-agent workflows using the **Plan-Do-Check-Act** state machine. When the user dispatches a task to PM (or any orchestrator), you run the **complete PDCA cycle to terminal state** without stopping at intermediate stages for human confirmation.

This skill encodes 3 hard rules learned from production session 2026-04-28:

1. **Full automation when dispatched** — don't stop at "do done" waiting for review approval
2. **No auto-commit during develop** — single commit at session end, not per micro-step
3. **PDCA full cycle** — `do done` MUST trigger `check` → `act` automatically per `reviewer_agent` field

---

## The Hard Rule

**When the user dispatches a workflow, you run it to a terminal state. Period.**

Terminal states (only stop here):
- `done` — task complete, lessons logged
- `blocked` with HARD VETO that cannot be auto-fixed (escalate to user)
- `blocked` waiting for budget/scope decision (user authority — explicit budget number, new scope, deadline change)
- `blocked` after 3 revision cycles (loop guard, escalate)

NOT terminal states (do NOT stop here):
- `do done, check pending` — auto-dispatch reviewer
- `check pass, act pending` — auto-apply act stage
- `check fail, fix pending` — auto-re-dispatch builder/marketer with verdict feedback
- `lessons not logged` — auto-log

If you stop at a non-terminal state, you have skipped the workflow. The user's WBS schema has `reviewer_agent`, `check_verdict`, `act_summary`, `lessons_learned` columns precisely so you can run automation. Use them.

---

## The PDCA State Machine

```
[plan] ──spec_locked_at set──> [do] ──do_finished_at set──> [check]
                                                                │
                                                                │ dispatch reviewer_agent
                                                                ▼
                                                       check_verdict received
                                                                │
                                          ┌─────────────────────┼─────────────────────┐
                                          │                     │                     │
                                  pass / pass-with-note   needs-revision           HARD VETO
                                          │                     │                     │
                                          ▼                     ▼                     ▼
                                       [act]              re-dispatch fix       escalate user
                                          │                  (loop max 3)        STOP terminal
                                          │                     │
                                act_summary +                   └──> back to [do]
                                lessons_learned set
                                          │
                                          ▼
                                    [done] / [recurring next cycle]
```

---

## Stage-by-Stage Behavior

### Plan stage
- Write spec to disk
- Set `spec_locked_at` ONLY if user explicitly confirms or all reviewers pass
- Otherwise leave `spec_locked_at = NULL` and proceed — spec lock is end-of-cycle confirmation, not blocker

### Do stage
- Dispatch agent per `agent_pic`
- Agent ships artifact (file, code, content)
- Set `do_started_at` when dispatch fires; `do_finished_at` when artifact received
- **Do not commit to git per artifact.** Stage in working tree, log "ready for check".
- Immediately enter Check stage — do not wait for user

### Check stage
- Read `reviewer_agent` field from task row
- Dispatch reviewer agent with artifact reference + acceptance criteria from spec
- Reviewer returns verdict: `pass` / `pass-with-note` / `needs-revision` / `fail` / `hard-veto`
- Write verdict to `check_verdict`, `check_finished_at`
- Branch on verdict (next section)

### Act stage (verdict branching)

**verdict = pass | pass-with-note**:
- Write `act_summary` (1-2 sentences: what shipped, what to watch in v2)
- Write `lessons_learned` (1-2 sentences: pattern to repeat or avoid)
- Set `done_at`, `pdca_stage = 'done'`, `status = 'done'`
- For recurring task: bump `next_due_at` per recurrence (weekly +7d, monthly +30d, per-deploy = NULL), set `last_run_at = now()`, reset `pdca_stage = 'do'` (cycle restarts)

**verdict = needs-revision**:
- Re-dispatch `agent_pic` with original artifact + verdict feedback as input
- Increment internal revision counter
- After agent ships v2 → re-dispatch reviewer (Check again)
- Loop until pass OR revision counter ≥ 3 → escalate user with full revision history

**verdict = hard-veto**:
- STOP. Set `status = 'blocked'`, `pdca_stage = 'check'`, `stage_status = 'fail'`
- Log full veto rationale to `notes` + `VETO-LOG.md`
- Surface to user as terminal blocker

**verdict = fail (non-veto)**:
- Same as needs-revision but with stricter feedback emphasis
- Loop guard same (max 3)

---

## Commit Discipline

**Default: do NOT commit during workflow.** Stage changes in working tree.

You may commit only when:
1. **User explicit ask**: "commit", "commit và push", "ship commit"
2. **End-of-session sealing**: at the very end of orchestration, after all artifacts settled, ship 1 cohesive commit with full context (not micro-step commits)
3. **Required for tool function**: e.g., `gh pr create` needs commit; do the minimum commit for tool to work

When you do commit:
- Use HEREDOC for message format
- Single cohesive scope per commit (don't sweep unrelated artifacts)
- If parallel agents wrote multiple files, group them into 1 commit at end of orchestration session, with message naming all artifacts

When NOT to commit even if it feels natural:
- After each agent's artifact ship → just stage, log "ready"
- After reviewer verdict → just update D1 + log
- After PDCA act apply → just update D1 + log
- Mid-workflow checkpoint "in case something breaks" → no, working tree state IS the checkpoint

---

## Dispatch Loop Guard

To prevent runaway loops:
- Track revision count per task in-memory during session
- Max 3 revision cycles per task → escalate user
- Max parallel agent fan-out: 4 (avoid stream timeout from earlier session evidence)
- If single agent times out → resume via SendMessage if cap-fits, else re-dispatch with smaller scope (1 file/artifact per agent call)

---

## When User Says "Just Execute"

When user says variants of:
- "thực hiện hết đi"
- "tự execute toàn bộ"
- "tin tưởng PM"
- "I'll be away, run the workflow"
- "just do it"

This means: **run to terminal state**. Auto-fix safe issues, dispatch reviewers, apply act, mark done. Only stop at the terminal states listed above. Do not interpret it as "do the next step then ask."

When user says variants of:
- "verify and report"
- "audit only"
- "tell me what's wrong"

This means: **read-only**, do NOT mutate. Ship the audit report and stop.

---

## When NOT to Use This Skill

- Single-agent task with no PDCA tracking (just do it directly)
- User explicitly asks for confirmation between steps ("show me each step before continuing")
- High-blast-radius irreversible action (push --force, rm -rf, payment, deploy prod) — these need confirmation regardless of automation rule
- Spec change that adds new task (scope addition = user authority, escalate)
- Budget decision with concrete number (user authority, escalate)

---

## Anti-Patterns to Avoid (lessons from 2026-04-28)

| Anti-pattern | Correct |
|---|---|
| "Builder shipped artifact, I'll wait for user to dispatch reviewer" | Dispatch reviewer per `reviewer_agent` field immediately |
| "Each agent commits its own artifact" | Stage all, single commit at session end |
| Co-bundle commit sweeping unrelated staged files | Verify staged files match commit scope before commit |
| Stop at "do done" because reviewer needs human input | Reviewer is an agent. Dispatch the agent. |
| Stop at "needs-revision" because builder needs to re-do | Re-dispatch builder with verdict as input |
| Skip `act` stage because "user can mark done" | Apply act + log lessons; user reviews finalized state |
| Multiple commits per orchestration session | 1 commit at session end with full context |

---

## Schema Hooks (when working with WBS-style D1 schema)

If the project's task table has these columns, use them:
- `pdca_stage` (plan/do/check/act/done/blocked/deferred/skipped)
- `stage_status` (pending/in_progress/pass/fail) — note: limited enum, may need adapter
- `reviewer_agent` (FK to agent definition)
- `do_started_at`, `do_finished_at`, `check_finished_at`, `done_at`
- `check_verdict` (pass/pass-with-note/needs-revision/fail/hard-veto)
- `act_summary`, `lessons_learned`
- `recurrence`, `next_due_at`, `last_run_at`
- `dispatch_log` table (id auto, task_id, agent, status, started_at, finished_at, response_summary)

If project does NOT have PDCA schema, run the same logic in-memory and surface state via report at end.
