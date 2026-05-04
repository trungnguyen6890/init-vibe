# init-vibe

> Vibe-coding starter pack - scaffold Claude-friendly project structure, manage spec context loading, automate multi-agent PDCA workflows, enforce ASCII-only document style.

For large projects (>=10 features, multi-domain) where you want Claude to implement features fast without context bloat or hallucination.

## What's inside

4 complementary skills that work together:

| Skill | Purpose | When to use |
|---|---|---|
| **init-vibe-scaffold** | Scaffold project structure (specs/ + domains/ + 6 core file templates) | Starting a new project |
| **layer-loading-strategy** | Load only relevant specs when implementing - prevent context bloat & hallucination | Implementing each feature |
| **pdca-agent-automation** | Multi-agent workflow (PM -> builder/reviewer/marketer) with full Plan-Do-Check-Act automation | Dispatching agents |
| **no-ai-chars** | Enforce ASCII + Vietnamese diacritics only in docs - no em-dash, smart quotes, emoji, or AI-typographic chars | Writing/cleaning any document |

Use them independently OR as a stack:
1. `init-vibe-scaffold` -> structure the project
2. `layer-loading-strategy` -> read specs efficiently while implementing
3. `pdca-agent-automation` -> orchestrate agents to implement features
4. `no-ai-chars` -> ensure docs use only standard typeable characters

## Installation

In Claude Code:

```
/plugin marketplace add trungnguyen6890/init-vibe
/plugin install init-vibe@claude-skills
```

Replace `trungnguyen6890` with your GitHub username if you fork.

The marketplace identifier is `claude-skills` (the bundle host); the plugin name is `init-vibe`. After install, skills appear as plugin `init-vibe@claude-skills`.

## Update later

Manual via Claude Code:

```
/plugin marketplace update claude-skills
/plugin update init-vibe@claude-skills
```

Or use the included shell script `scripts/claude-sync` for one-command sync of all git-based marketplaces:

```bash
# One-time setup (per device)
mkdir -p ~/bin
cp ~/.claude/plugins/marketplaces/claude-skills/scripts/claude-sync ~/bin/
chmod +x ~/bin/claude-sync
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
echo "alias cs='claude-sync'" >> ~/.zshrc
source ~/.zshrc

# Then anytime to sync
claude-sync                  # sync all git marketplaces
claude-sync claude-skills    # sync only this one
cs                           # short alias

# After sync, restart Claude Code session to reload skills
```

The script:
- Pulls latest from each git-based marketplace (`~/.claude/plugins/marketplaces/*/`)
- Skips non-git marketplaces (e.g. anthropics official uses different mechanism)
- Shows commit deltas (before -> after + last 5 changes)
- Reminds to restart Claude Code session after sync

## Project structure scaffolded by `init-vibe-scaffold`

```
project-root/
+-- CLAUDE.md
+-- specs/
    +-- VISION.md                   # what + why + non-goals (~150 line)
    +-- STRATEGY.md                 # wedge + phase + business (~250 line)
    +-- ARCHITECTURE.md             # tech stack + NFR (~400 line)
    +-- MASTER-INDEX.md             # function inventory + domain map (~250 line)
    +-- MODULE-INTERFACES.md        # cross-feature TS signatures (~200 line)
    +-- DECISIONS.md                # append-only decision log (~300 line)
    +-- domains/
        +-- auth/
        |   +-- F-LOGIN-EMAIL.md
        |   +-- F-LOGOUT.md
        +-- payment/
        |   +-- F-CHECKOUT.md
        +-- ...
```

**Feature-based** (not module-based). Each feature file ~150-250 line, self-contained, with `load_order` header for layer-loading skill.

## Why this structure?

Tested on real production project (Asklaw.vn - 22 modules, 125 features, 6000+ line specs). Original module-based 1168-line Master Design with 22 detail design files at 600+ line each caused context bloat and spec hallucination.

This bundle is the **lessons learned**:
- Feature-based beats module-based for Claude implementation
- Lean files (~200 line) beat heavy spec docs (600+ line)
- Single source of truth per concern (Master = index, Interfaces = signatures, Decisions = rationale)
- Layer loading discipline (~1100 line active context max) beats naive "load all specs"
- ASCII + Vietnamese diacritics only in docs - no AI typographic signature

## Adding new skills to this bundle

Drop new skill folder into `skills/` with valid `SKILL.md` (YAML frontmatter required: `name`, `description`).

Test locally first:
```bash
cp -r skills/<new-skill> ~/.claude/skills/
# Verify it appears in skill list when starting Claude Code
```

Then commit + push:
```bash
git add skills/<new-skill>
git commit -m "feat: add <new-skill>"
git push
```

## License

MIT - use, fork, modify freely.

---

**Author**: [@trungnguyen6890](https://github.com/trungnguyen6890) - built while working on [Asklaw.vn](https://asklaw.vn) (AI legal copilot for Vietnam).

#vibecoding #initvibe #claudecode
