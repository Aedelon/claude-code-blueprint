# Claude Code Blueprint

A production-grade Claude Code configuration: skills, agents, hooks, rules, and permissions working as a system, not as isolated features.

**Companion repo for the article**: *"I Turned Claude Code Into an Operating System. Here's the Blueprint."*

---

## Architecture

```
+---------------------------------------------------+
|                 LAYER 6: MCP                      |
|        6 servers (context7, fetch, git,           |
|         huggingface, jupyter, ide)                |
+---------------------------------------------------+
|              LAYER 5: SECURITY                    |
|     40 allow + 38 deny + hooks (17 events)        |
|     + regex secrets + fail-closed trap            |
+---------------------------------------------------+
|              LAYER 4: AGENTS                      |
|        Specialized sub-agents (by domain)         |
|     Multi-agent brainstorm (parallel)             |
+---------------------------------------------------+
|              LAYER 3: SKILLS                      |
|     32 skills (10 categories) auto-loaded         |
|     Common pattern: 5 phases AAPEV               |
+---------------------------------------------------+
|            LAYER 2: MEMORY                        |
|   memory/ persistent + rules/ per project         |
|   Cross-session learning                          |
+---------------------------------------------------+
|          LAYER 1: CLAUDE.md (KERNEL)              |
|   Anti-hallucination + Confidence levels +        |
|   Toolchain + Code standards + /compact rules     |
+---------------------------------------------------+
```

Each layer reinforces the others. The anti-hallucination protocol works **because** Context7 is an accessible MCP server. Security hooks work **because** static permissions define a perimeter. Multi-agent brainstorm works **because** agents have dedicated tools and models.

---

## What's Included

| Component | Count | Description |
|-----------|-------|-------------|
| **CLAUDE.md** | 1 | The kernel. Anti-hallucination protocol, confidence levels, security rules, code standards. |
| **Rules** | 2 | Path-scoped conventions: `python.md` and `typescript.md`. Load only when editing matching files. |
| **Skills** | 8 / 32 | Representative subset. Each follows the 5-phase AAPEV pattern. |
| **Agents** | 4 | Safe examples across different domains. |
| **Hooks** | 11 | 9 complete + 2 sanitized (security patterns generalized). |
| **Commands** | 3 | Slash commands: `/agent`, `/docs`, `/review`. |
| **Settings** | 1 | Template with permission structure (adapt to your needs). |
| **Memory** | dir | Empty directory with `.gitkeep`. Claude writes here across sessions. |

### What's NOT Included

This is a curated subset, not a mirror of a production setup. The full system has 32 skills, more agents, and exact security regex patterns. The hooks `bash-guard` and `write-guard` are **sanitized**: they show what categories of threats to block without exposing exact regex. Adapt them to your threat model.

---

## Quick Start

```bash
# 1. Clone
git clone https://github.com/Aedelon/claude-code-blueprint.git
cd claude-code-blueprint

# 2. Copy to your Claude Code config
cp CLAUDE.md ~/.claude/CLAUDE.md
cp -r rules/ ~/.claude/rules/
cp -r skills/ ~/.claude/skills/
cp -r agents/ ~/.claude/agents/
cp -r hooks/ ~/.claude/hooks/
cp -r commands/ ~/.claude/commands/

# 3. Adapt settings (DO NOT copy blindly -- review permissions first)
# Use settings.template.json as a reference for your own ~/.claude/settings.json

# 4. Install hook dependencies
brew install jq  # or: apt install jq
```

---

## Directory Structure

```
claude-code-blueprint/
├── README.md
├── LICENSE                          # Apache 2.0
├── CLAUDE.md                        # The kernel
├── settings.template.json           # Permission structure template
│
├── rules/
│   ├── python.md                    # Python conventions (uv, ruff, pytest)
│   └── typescript.md                # TypeScript conventions (prettier, vitest)
│
├── skills/
│   ├── anti-hallucination/SKILL.md  # Context7-first verification
│   ├── core-protocols/SKILL.md      # Systematic debugging
│   ├── brainstorm/SKILL.md          # Multi-agent orchestration
│   ├── code-patterns/SKILL.md       # Reference patterns
│   ├── security-audit/SKILL.md      # OWASP, secrets, deps
│   ├── uv-workflow/SKILL.md         # Python toolchain
│   ├── commit-message/SKILL.md      # Git workflow
│   └── research-protocol/SKILL.md   # Citations & sources
│
├── agents/
│   ├── research-synthesizer.md      # Academic research
│   ├── midjourney-expert.md         # Creative / image generation
│   ├── finance-advisor.md           # Wealth management
│   └── prompt-engineer.md           # LLM prompt design
│
├── hooks/scripts/
│   ├── session-start.sh             # Project detection, git context
│   ├── session-end.sh               # Cleanup
│   ├── user-prompt-secrets.sh       # Scan secrets before API
│   ├── bash-guard.sh                # SANITIZED: dangerous commands
│   ├── write-guard.sh               # SANITIZED: protected files + secrets
│   ├── write-format.sh              # Auto-format (prettier/ruff)
│   ├── bash-vuln.sh                 # npm audit after install
│   ├── posttooluse-failure.sh       # Log tool failures
│   ├── stop.sh                      # Git summary
│   ├── pre-compact.sh               # Context preservation
│   └── permission-git.sh            # Destructive git warning
│
├── commands/
│   ├── agent.md                     # /agent <name>
│   ├── docs.md                      # /docs <library>
│   └── review.md                    # /review [file]
│
├── memory/
│   └── .gitkeep                     # Claude writes here
│
└── docs/
    └── architecture.md              # System design notes
```

---

## The 5-Phase Pattern (AAPEV)

Every skill follows the same cognitive discipline:

```
Phase 1: ASSESS / CLARIFY    --> Understand the problem
Phase 2: ANALYZE / RESEARCH   --> Search before acting
Phase 3: PLAN / DESIGN        --> Propose a plan
Phase 4: EXECUTE / IMPLEMENT  --> Apply changes
Phase 5: VALIDATE / VERIFY    --> Test the result
```

This prevents the model from jumping to Phase 4 (code) without understanding (Phase 1), researching (Phase 2), and planning (Phase 3).

---

## Build Your Own

### Adding a Skill

Create `~/.claude/skills/your-skill/SKILL.md`:

```yaml
---
name: your-skill
description: |-
  What this skill does. MUST BE USED when user asks: "trigger phrases".
allowed-tools:
  - Read
  - WebSearch
---

# Your Skill

## Process
Phase 1: ASSESS ...
Phase 2: ANALYZE ...
Phase 3: PLAN ...
Phase 4: EXECUTE ...
Phase 5: VALIDATE ...
```

### Adding an Agent

Create `~/.claude/agents/your-agent.md`:

```yaml
---
name: your-agent
model: sonnet  # or opus for complex tasks
description: |-
  What this agent specializes in.
  MUST BE USED when user asks: "trigger phrases".
tools: [Read, Write, WebSearch]
color: "#hex"
---

Your agent's personality, methodology, and domain knowledge.
```

### Adding a Hook

1. Create the script in `~/.claude/hooks/scripts/your-hook.sh`
2. Make it executable: `chmod +x your-hook.sh`
3. Register it in `~/.claude/settings.json` under the appropriate lifecycle event

Claude Code supports **17 lifecycle events**: `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PostToolUseFailure`, `Notification`, `SubagentStart`, `SubagentStop`, `Stop`, `TeammateIdle`, `TaskCompleted`, `ConfigChange`, `WorktreeCreate`, `WorktreeRemove`, `PreCompact`, `SessionEnd`. This blueprint covers 9 of them.

Hooks support three types:
- **`"type": "command"`** — a shell script (used in this blueprint)
- **`"type": "http"`** — POST JSON to a URL, receive JSON back
- **`"type": "prompt"`** — an LLM sub-agent interprets the event

Key patterns for command hooks:
- Always start with `set -euo pipefail`
- Use `trap ... ERR` for fail-closed behavior on PreToolUse hooks
- Read JSON from stdin: `input=$(cat)`
- Parse with jq: `$(echo "$input" | jq -r '.field')`

---

## Sanitized Hooks

The `bash-guard.sh` and `write-guard.sh` hooks contain **generic patterns** instead of exact regex. This is intentional:

- **bash-guard**: Categories of dangerous commands (sudo, rm -rf, fork bombs, obfuscation). Add your own regex patterns.
- **write-guard**: Categories of protected files and secret formats. Add patterns for your specific API keys and cloud providers.

The article explains the categories. This repo gives you the skeleton.

---

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI installed
- `jq` for JSON parsing in hooks
- `prettier` (for JS/TS auto-format hook)
- `ruff` (for Python auto-format hook)
- MCP servers configured in your `settings.json` (context7, fetch, git at minimum)

---

## License

Apache 2.0 -- Delanoe Pirard / Aedelon

---

## Credits

Built with Claude Code. The system described is the system used.
