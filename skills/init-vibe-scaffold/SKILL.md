---
name: init-vibe-scaffold
description: "Use when starting a new project with Claude Code OR when user asks to set up Claude-friendly project structure, scaffold spec templates, or initialize CLAUDE.md. Triggers on: 'new project init', 'scaffold project', 'set up Claude project', 'project structure for AI', 'CLAUDE.md template', 'spec template', 'starting a project', 'init vibe', 'kickstart project'. Targets large projects (≥10 features, multi-domain) but adapts to smaller ones."
---

# Init-Vibe Scaffold — Claude-Friendly Project Structure

You are scaffolding a new project. Goal: structure specs so Claude implements features fast without context bloat.

**Companion skill**: After scaffolding, use `layer-loading-strategy` for efficient spec reading during implementation.

---

## Decision Tree — Use This Skill?

**Use init-vibe-scaffold when**:
- New project ≥10 user-facing features OR multi-domain (auth + payment + corpus + ...)
- Project will live ≥3 months
- Multi-agent or multi-session implementation expected
- User says "init project", "scaffold", "set up structure"

**Skip when**:
- Quick prototype, ≤5 features
- Single-domain script, no cross-feature shared logic
- Existing project with established structure (don't disrupt)

---

## Project Layout

```
project-root/
├── CLAUDE.md                       # Top-level instruction (auto-loaded by Claude Code)
├── specs/
│   ├── VISION.md                   # ~150 line — what + why + non-goals
│   ├── STRATEGY.md                 # ~250 line — wedge + phase + pricing/business
│   ├── ARCHITECTURE.md             # ~400 line — tech stack + component map + NFR
│   ├── MASTER-INDEX.md             # ~250 line — function inventory + domain map
│   ├── MODULE-INTERFACES.md        # ~200 line — TypeScript signatures cross-feature
│   ├── DECISIONS.md                # ~300 line — append-only decision log
│   └── domains/
│       ├── {domain-1}/             # vd auth, payment, corpus
│       │   ├── F-{feature}.md
│       │   └── ...
│       └── {domain-2}/
│           └── ...
└── (code folders — src/, tests/, etc. per project tech stack)
```

**Rules**:
- Domain folder name = lowercase (`auth`, `payment`)
- Feature file name = `F-{KEBAB-CASE}.md` (vd `F-LOGIN-EMAIL.md`)
- No `README.md` per domain (folder name = boundary; redundant)
- No `features/` subfolder inside domain (folder name already implies)

---

## File Templates

### Template 1 — `CLAUDE.md` (top-level instruction)

```markdown
# Project — {project-name}

**Read first**: specs/VISION.md, specs/STRATEGY.md (foundation context).

**When implementing a feature**:
1. Read specs/MASTER-INDEX.md function inventory row for the feature
2. Read the feature file (specs/domains/{domain}/F-{feature}.md)
3. Follow `load_order` header in that feature file
4. Use `layer-loading-strategy` skill for efficient reading

**Decisions**: see specs/DECISIONS.md (append-only log; never delete entries).

**Cross-feature interfaces**: see specs/MODULE-INTERFACES.md.

**HARD VETO triggers**: see specs/MASTER-INDEX.md §HARD VETO.

**Don't**:
- Read all specs at once (use layer-loading discipline)
- Skip writing tests for new features
- Modify spec without updating DECISIONS.md if rationale changes
```

### Template 2 — `specs/VISION.md` (~150 line target)

```markdown
---
title: {project-name} — Vision
status: locked
last_updated: YYYY-MM-DD
---

# Vision

## 1. What
[1-paragraph product description. What does it do?]

USP statement (1 sentence):
> [Unique value proposition]

## 2. Why
[1-2 paragraph problem space. Why does this exist? What pain does it solve?]

## 3. Who (target users)
[Bulleted segments. Who pays? Who uses?]

## 4. Core capabilities
[Table: capability + purpose + output]

## 5. Quality bar — North Star
[1-3 metrics that define "good". E.g., accuracy ≥X%, latency <Ys, churn <Z%]

## 6. Non-goals (out of scope)
| # | Out of scope | Lý do |
|---|---|---|
| 1 | [item] | [why] |
| 2 | [item] | [why] |

## 7. Tech stack & constraints (1 line — full detail in ARCHITECTURE)
[1 sentence: language, framework, hosting, key dependencies]

## 8. Success criteria (vision-level)
[3-5 bullet: how do we know vision is achieved at month 6, 12, ...]
```

### Template 3 — `specs/STRATEGY.md` (~250 line target)

```markdown
---
title: {project-name} — Strategy
status: locked
last_updated: YYYY-MM-DD
related: VISION.md, ARCHITECTURE.md
---

# Strategy

## 1. Wedge (where to start)
[Initial market segment + reason]

## 2. Phasing
| Phase | Window | Goal | Pricing | Gate |
|---|---|---|---|---|
| 0 Beta | M1-2 | [validate] | free | [criteria] |
| 1 Ship | M3-4 | [launch] | [pricing] | [criteria] |
| ... | | | | |

## 3. Pricing
[Tier table or freemium model]

## 4. Positioning
[vs competitor table — 1-line per competitor]

## 5. Anti-positioning (refuse claims)
[What we explicitly do NOT say/do]

## 6. Moat
[3-5 bullet: real moats vs features anyone can copy]

## 7. R0 dangerous assumptions (validate phase 0)
[Numbered assumptions that, if wrong, kill the project. Plan to validate each.]

## 8. Risks (R1-RN)
[Mitigation table]
```

### Template 4 — `specs/ARCHITECTURE.md` (~400 line target)

```markdown
---
title: {project-name} — Architecture
status: locked
last_updated: YYYY-MM-DD
related: VISION.md, STRATEGY.md
---

# Architecture

## 1. Tech stack (locked)
| Layer | Choice | Notes |
|---|---|---|
| Runtime | [e.g., Cloudflare Workers] | [reason] |
| DB | [e.g., D1 SQLite] | [reason] |
| ... | | |

## 2. C4 Level 1 — System context
[Actor → edge → storage → external]

## 3. Core pipeline (if applicable)
[Sequence diagram or flow description for primary flow]

## 4. State coordination
[DO instances / state machines / queues — when to use what]

## 5. Tier router / auth model
[Day-1 tier list + rate-limits]

## 6. Day-0 prerequisite checklist
[External services, secrets, DNS, accounts to set up before first feature dispatch]

## 7. NFR (Non-functional requirements)
| ID | Name | Target | Phase |
|---|---|---|---|
| N1 | [name] | [target] | Day-1 |
| ... | | | |

## 8. Cost & latency budget
[Per-request budget; alerts at threshold]

## 9. Security & compliance
[High-level only; details in domain features]

## 10. Observability
[Logs, metrics, traces — where + retention]
```

### Template 5 — `specs/MASTER-INDEX.md` (~250 line target)

```markdown
---
title: {project-name} — Master Index
status: living document
last_updated: YYYY-MM-DD
purpose: Index/registry — discover features + domain boundary + cross-domain flows. NOT a design doc.
related: VISION.md, STRATEGY.md, ARCHITECTURE.md
---

# Master Index

## TOC (line ranges — for layer-loading skill Read offset)
<!-- Update line ranges when sections grow. Used by layer-loading-strategy skill. -->
- §1 Function inventory: line 15-X
- §2 Domain map: line X-Y
- §3 Cross-domain flows: line Y-Z
- §4 Phase plan: line Z-W
- §5 HARD VETO triggers: line W-end

## §1 Function inventory

| Feature ID | Name | Domain | Phase | Detail file | Status |
|---|---|---|---|---|---|
| F-LOGIN-EMAIL | Email + OTP login | auth | Day-1 | `domains/auth/F-LOGIN-EMAIL.md` | pending |
| F-LOGIN-OAUTH | OAuth Google login | auth | Day-1 | `domains/auth/F-LOGIN-OAUTH.md` | pending |
| ... | | | | | |

Status enum: pending | in-progress | done | deprecated

## §2 Domain map

| Domain | Boundary (1 sentence) | Feature count |
|---|---|---|
| auth | Identity + session + tokens | N |
| payment | Stripe + delivery + refund | N |
| ... | | |

## §3 Cross-domain flows
<!-- 1-line per critical user journey, link feature IDs in order -->
- New user signup: F-LOGIN-EMAIL → F-CONSENT-BANNER → F-DASHBOARD-FIRST-VISIT
- Purchase flow: F-LOGIN-EMAIL (optional) → F-CHECKOUT → F-DELIVERY → F-RECEIPT
- ...

## §4 Phase plan

| Phase | Features | Gate criteria |
|---|---|---|
| Day-1 | F-LOGIN-EMAIL, F-CHECKOUT, ... | [metrics] |
| Phase 2 | F-PRO-SUBSCRIPTION, ... | [metrics] |

## §5 HARD VETO triggers
<!-- Conditions that block ship; resolution required before continuing -->
| # | Trigger | Severity | Module owner |
|---|---|---|---|
| 1 | [condition] | critical | [feature/module] |
```

### Template 6 — `specs/MODULE-INTERFACES.md` (~200 line target)

```markdown
---
title: {project-name} — Module Interfaces
status: living document
purpose: Single source of truth for cross-feature shared helper signatures. Phase 1 spec; Phase 2+ derived from code.
last_updated: YYYY-MM-DD
---

# Module Interfaces (cross-feature signatures)

Update when adding/changing a function consumed by ≥2 features.

## TOC
- AuthHelpers: line X
- GuardHelpers: line Y
- ...

---

## AuthHelpers (used by domains/auth/* and domains/payment/*)

```typescript
export interface AuthHelpers {
  /** Verify JWT signature + expiry + revocation list. */
  verifyJWT(token: string): Promise<JWTPayload | null>

  /** Issue JWT + refresh token after successful auth. */
  issueJWT(payload: { user_id: string; email: string; tier: 'authed' }):
    Promise<{ access_token: string; refresh_token: string; expires_at: string }>

  /** Hash OTP for KV storage (deterministic salt). */
  hashOTP(code: string, email: string): Promise<string>
}

export type JWTPayload = {
  sub: string         // user_id (UUID v4)
  email: string
  tier: 'authed' | 'admin'
  iat: number
  exp: number
  jti: string         // JWT ID for revocation
}
```

---

## GuardHelpers (used by all domains)

```typescript
export interface GuardHelpers {
  /** Scan text for banned patterns; return matches. */
  scanForViolations(text: string, lang: 'vi' | 'en'): {
    violation: boolean
    matches: string[]
  }

  /** Redact PII (email, phone, IDs) before persistence. */
  redactPII(text: string): string
}
```

---

[Add more module interfaces as project grows. Keep file ≤300 line; if longer, split per domain.]
```

### Template 7 — `specs/DECISIONS.md` (~300 line target, append-only)

```markdown
---
title: {project-name} — Decisions log
status: append-only
purpose: Single source of truth for decision rationale. NEVER delete entries; supersede by appending new entry referencing old.
last_updated: YYYY-MM-DD
---

# Decisions Log

**Convention**:
- Append entries chronologically (newest at top).
- Each entry has `decision_id: YYYY-MM-DD-{slug}` for cross-reference.
- When superseding old decision, append new entry with `supersedes: YYYY-MM-DD-{old-slug}`.
- Never delete entries (audit trail).

---

## YYYY-MM-DD-{decision-slug}

**Context**: [What were we deciding? When?]

**Decision**: [The choice. State concretely with numbers/names.]

**Rationale**: [Why? Bullet 2-4 reasons.]

**Alternatives rejected**: [What else considered + why not]

**Owner**: [Role(s) who decided]

**Source**: [File/section/event that triggered this decision]

**Affects**: [Feature IDs F-X, F-Y this decision impacts]

---

[Continue with more entries above this line...]
```

### Template 8 — Feature file `F-{FEATURE}.md` (~150-250 line target)

**Lean header** (10 lines, NOT 25):

```markdown
---
feature_id: F-LOGIN-EMAIL
name: Email + OTP login
domain: auth
phase: Day-1
load_order:
  required:
    - specs/MASTER-INDEX.md (function inventory row F-LOGIN-EMAIL)
    - specs/MODULE-INTERFACES.md (AuthHelpers section)
  reference_only:
    - specs/ARCHITECTURE.md (Resend email integration if implementing email send)
---

# F-LOGIN-EMAIL — Email + OTP Login

## Purpose
[2-3 sentences: what user does, what system returns]

## User flow (e2e behavior)
1. User submits email
2. Backend issues OTP code, sends email via Resend
3. User submits code
4. Backend verifies, issues JWT + refresh token
5. User authed

## Modules touched (this feature implementation)
- `src/workers/api/auth/login-otp.ts` (new)
- `src/workers/api/auth/verify-otp.ts` (new)
- `src/lib/auth/otp.ts` (new)

## Cross-feature dependencies
- Consumes: `AuthHelpers.issueJWT` from MODULE-INTERFACES.md
- Consumes: `GuardHelpers.redactPII` from MODULE-INTERFACES.md
- Triggers: `F-WELCOME-EMAIL` (post-login, async)

## Decisions specific to this feature
[Local decisions not global enough for DECISIONS.md. If rationale matters cross-feature, MOVE to DECISIONS.md and link here.]

- OTP TTL = 10 min (per DECISIONS 2026-05-03-OTP-TTL-10MIN)
- Rate-limit 5 attempts/email/15min

## Acceptance criteria
- [ ] OTP issuance success rate ≥ 99%
- [ ] OTP code 6-digit, bias-corrected random
- [ ] Rate-limit triggers at 6th attempt within 15min → 429
- [ ] JWT verifiable + revocable

## Open questions (blocking implementation)
- [ ] [question] — owner: [role]

## Reference
- `MASTER-INDEX.md` row F-LOGIN-EMAIL
- `DECISIONS.md` 2026-05-03-OTP-TTL-10MIN
- `MODULE-INTERFACES.md` AuthHelpers
```

**Body sections** can adapt to project — common sections: User flow, Modules touched, Cross-feature dependencies, Decisions, Acceptance, Open questions, Reference.

---

## Init Checklist

When user asks to scaffold:

1. **Confirm scope** — ≥10 features or multi-domain? If yes proceed; if no, recommend simpler structure.
2. **Ask project name + tech stack** — needed to populate templates.
3. **Ask domain list** — top-level domains (auth, payment, ...). Suggest 3-7 domains; if more, project may need decomposition into sub-projects first.
4. **Create folder structure** — `specs/`, `specs/domains/{each-domain}/`.
5. **Generate 6 core files** from templates 2-7 (VISION/STRATEGY/ARCHITECTURE/MASTER-INDEX/MODULE-INTERFACES/DECISIONS) — fill placeholders with project-specific values.
6. **Generate `CLAUDE.md`** at project root from Template 1.
7. **Generate 1 example feature file** in first domain — show the team the template.
8. **Recommend** — "Use `layer-loading-strategy` skill when implementing each feature."

---

## Anti-patterns to Avoid

| Anti-pattern | Why bad | Fix |
|---|---|---|
| Module-based file structure (1 file per code module) | Tight to code, miss business view | Use feature-based; module signatures go in MODULE-INTERFACES.md |
| README.md per domain | Redundant with folder name + Master Index | Skip; folder name = boundary |
| Heavy header (25 lines metadata) | Bloats context when scanning | Lean header (10 lines): id, name, domain, phase, load_order |
| Code snippets in feature files | Spec drifts from code; Claude copy-pastes | Reference paths only; let Claude read code for patterns |
| Test enumeration in feature files | Tests emergent during implementation | List acceptance criteria, not test cases |
| Duplicate decision rationale across files | Drift on update | Single source: DECISIONS.md, link from feature |

---

## Adaptation Notes

- **Smaller projects** (5-9 features, single domain): skip `domains/` subfolder; place feature files directly in `specs/`. Skip MODULE-INTERFACES.md if no cross-feature helpers.
- **Tech stack varies**: adapt MODULE-INTERFACES.md syntax (TypeScript shown; can use Python/Go/Rust signatures).
- **Multi-language docs**: keep file names English, content can be Vietnamese/English mixed (templates support both).
- **Existing project migration**: don't force-convert; introduce MASTER-INDEX.md first as overlay, gradually move detail to feature files.

---

## Reference

- Companion skill: `layer-loading-strategy` (efficient spec reading during implementation)
- Related: `pdca-agent-automation` (multi-agent dispatch workflow)
- Pattern based on: feature-based domain-driven design adapted for AI-first development workflows
