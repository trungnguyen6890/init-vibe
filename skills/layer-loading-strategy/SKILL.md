---
name: layer-loading-strategy
description: "Use when implementing features in large multi-spec projects (≥5 spec files OR ≥1000 lines total spec) to prevent context window degradation. Triggers on: large project with hierarchical specs (Vision → Strategy → Architecture → Master Index → Feature design), multi-feature implementation, multi-agent dispatch, when user mentions 'context too large', 'spec hallucination', 'lost in the middle', 'divide and conquer specs', or when about to implement a feature that has multiple cross-referenced spec files."
---

# Layer Loading Strategy — Spec Context Management for Large Projects

You are operating in a project with **hierarchical specs** (foundation specs + feature design files + cross-references). Loading all specs at once degrades your performance via "lost in the middle" attention drop and increases hallucination risk.

This skill encodes the **layer loading discipline** for reading specs efficiently when implementing.

For how to **structure** specs (folder layout, file templates, header format) so that this skill works — see the companion skill `init-vibe-scaffold`.

---

## The Core Problem

Your context window is large (200K-1M tokens) but **attention is uneven**:
- Strong recall: prompt start + recent messages
- Weak recall: middle of long context (researched "lost in the middle" phenomenon, Liu et al. 2023)
- Confabulation risk: cross-references like `§4.5` increase when multiple long sections held simultaneously

When you load 5000+ lines of spec to implement 1 feature:
- ~30% of detail blurs or gets missed
- You hallucinate "spec said X" when it didn't
- You miss critical constraints nested in distant sections
- You copy-paste code snippets from spec instead of writing natural code with current code context

**Solution: load ONLY what implementing the current feature needs.**

---

## The 4 Layers

### Layer 1 — Always loaded (~500 lines)

Foundation context for every feature implementation:

- `VISION.md` — business intent, non-goals (small, ~150 line)
- `STRATEGY.md` — wedge, phase plan (~250 line; can read offset+limit if larger)
- `MASTER-INDEX.md` row of CURRENT feature only (~10-30 line via offset+limit)

**Read mechanic**: full file for VISION (small), `Read offset+limit` for MASTER-INDEX function inventory row.

### Layer 2 — Feature-specific (~200-400 lines)

The feature design file you're implementing:

- `domains/{domain}/F-{FEATURE}.md` (full file) — purpose, user flow, decisions, acceptance
- Source spec sections referenced in the feature's `load_order.required` (~50-100 line each, via offset+limit)

**Read mechanic**: full feature file (small by design — ~150-250 line), `Read offset+limit` for source spec sections.

### Layer 3 — Lazy loaded (read on-demand)

Files you reference but don't fully load:

- `MODULE-INTERFACES.md` — TypeScript signature registry for cross-feature shared helpers (~200 line; Read full when feature `consumes_interfaces`)
- `DECISIONS.md` — append-only decision log (~300 line; Grep for specific decision_id, Read offset+limit ~30 line per entry)
- Existing code: use `Glob` then `Read` 2-3 relevant files when implementing

**Read mechanic**: search-driven (`Grep` to find, then `Read offset+limit` for specific section).

### Layer 4 — Reference only (DO NOT load full)

Files too large to load entirely; reference structurally:

- Full `ARCHITECTURE.md` if >500 line (only read tech stack section + relevant component map)
- Full source specs (data schemas, API contracts, prompts, etc.) — read only specific section needed
- Other feature files (do NOT load unless explicit cross-feature dependency)

**Read mechanic**: `Grep` to locate exact section, then `Read offset+limit` for ≤50 lines.

---

## Active Context Budget

Target: **800-1500 active lines** when implementing one feature, never more than 2500.

Math:
- Layer 1: ~500 line (foundation)
- Layer 2: ~300 line (current feature + relevant source sections)
- Layer 3: ~300-500 line (interface signatures + decision lookups + existing code patterns)
- Layer 4: 0 (reference only, not loaded)

**Total: ~1100-1300 line active context** for implementing 1 feature.

Compare vs naive approach: load Master + 5 feature designs + full source specs = 5000-8000 line. **3-5× context bloat.**

---

## The Workflow (every feature implementation)

```
1. User says "implement F-X"
   ↓
2. Read MASTER-INDEX.md function inventory row F-X (offset+limit)
   ↓
3. Read F-X.md full → check `load_order` header
   ↓
4. Follow load_order.required → Read those sections (offset+limit)
   ↓
5. If F-X declares `consumes_interfaces` → Read MODULE-INTERFACES.md relevant sections
   ↓
6. If F-X references DECISIONS → Grep DECISIONS.md by decision_id, Read offset+limit
   ↓
7. Glob existing code patterns (src/lib/{related}, src/workers/{related})
   ↓
8. Read 2-3 existing files to learn project's code patterns
   ↓
9. Implement with ~1000-1500 line active context
   ↓
10. If discover need for reference_only spec → Grep+Read specific section
   ↓
11. Test + iterate (emergent test cases, not pre-listed in spec)
```

You DO NOT:
- Read full MASTER-INDEX.md
- Read multiple feature files at once (only the current feature's file)
- Read full source specs unless explicitly needed
- Copy-paste code from feature files (feature files have no code by design)

---

## Sub-agent Dispatch for Hard Isolation

For projects implementing many features sequentially, use sub-agent dispatch to enforce layer loading via context isolation:

```
Main thread dispatches Task agent for "Implement F-X":
  - Agent has its own context window (separate from main thread)
  - Agent reads only what its prompt instructs (Layer 1 + 2)
  - Agent returns 200-500 line summary
  - Main thread receives summary only, NOT the spec content the agent loaded
```

This prevents main thread context pollution when implementing many features in one session.

---

## Anti-patterns

| Anti-pattern | Why bad | Fix |
|---|---|---|
| "Read all specs first to ensure consistency" | Loads 10000+ line, blurs focus | Read F-X load_order, follow strictly |
| "Quote spec verbatim in code comment" | Spec drifts, comment rots | Reference spec path + section, don't copy |
| "Load Master + 5 feature designs to ensure consistency" | Cross-feature risk inflated | Use MODULE-INTERFACES.md for signatures |
| "Pre-write tests from spec test enumeration" | Tests emergent during implementation | Write tests when bug discovered or contract violated |
| "Implement multiple features in one session without sub-agent" | Context accumulates from prev features | Dispatch sub-agent per feature, OR clear context between |
| "Read entire DATA-SPEC for one table DDL" | 1000+ line for 50-line need | Grep table name, Read offset+limit |
| "Read full ARCHITECTURE.md to find tech stack" | Wastes 400 lines | Grep "tech stack" → Read offset+limit |

---

## When This Skill Applies

**Apply layer loading when**:
- Project has ≥5 spec files OR ≥1000 line total spec
- Implementing a feature that touches multiple shared helpers
- Multi-agent workflow with cross-references
- User mentions context concerns, hallucination, "divide and conquer"

**Skip when**:
- Small project (≤3 files, ≤500 line total spec)
- Single-shot script with no cross-spec dependencies
- Exploratory questions about codebase (not implementation)

---

## Required Project Structure

This skill assumes specs are structured per the `init-vibe-scaffold` skill:

- `specs/MASTER-INDEX.md` with TOC line ranges for offset+limit reading
- `specs/MODULE-INTERFACES.md` with cross-feature TypeScript signatures
- `specs/DECISIONS.md` with append-only decision log
- `specs/domains/{domain}/F-{feature}.md` with `load_order` header

If your project lacks this structure → use `init-vibe-scaffold` first to scaffold it, then return to this skill for loading discipline.

---

## Reference

- Lost in the middle phenomenon: Liu et al. 2023 (research on context window attention)
- Companion skill: `init-vibe-scaffold` (defines project structure this skill assumes)
- Related: `pdca-agent-automation` (multi-agent workflow), `superpowers:dispatching-parallel-agents`
