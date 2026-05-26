# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **Learn Claude Code — Harness Engineering for Real Agents**, a 20-lesson progressive tutorial that teaches how to build the harness (tools, context management, permissions, coordination) around an LLM agent. The core thesis: agency comes from model training; the harness is just the vehicle.

## Setup

```bash
# Install dependencies
uv sync
# or: pip install anthropic python-dotenv

# Configure environment
cp .env.example .env
# Edit .env: set ANTHROPIC_API_KEY and MODEL_ID (e.g. claude-sonnet-4-6)
```

## Running Lessons

Each lesson is a standalone script:

```bash
python s01_agent_loop/code.py
python s02_tool_use/code.py
# ... through s20_comprehensive/code.py
```

All scripts read `MODEL_ID` and `ANTHROPIC_API_KEY` from `.env`. To use an alternative provider, set `ANTHROPIC_BASE_URL` in `.env`.

## Running Tests

```bash
# Smoke tests: compile-check all legacy agent scripts in agents/
uv run pytest tests/test_agents_smoke.py

# Run a single test
uv run pytest tests/test_agents_smoke.py::test_agent_scripts_exist
```

The smoke tests only cover the `agents/` (legacy) directory, not the root-level `s01-s20` lesson scripts.

## Repository Structure

**Canonical track (current):** Root-level `s01_agent_loop/` through `s20_comprehensive/` — each folder has `code.py` and trilingual READMEs (`README.md`, `README.en.md`, `README.ja.md`).

**Legacy track:** `agents/` (Python scripts) + `docs/` (markdown) + `web/` (Next.js visualizer) — the older 12-lesson version, kept for existing readers.

**`skills/`** — skill definitions for Claude Code agents: `agent-builder/`, `code-review/`, `mcp-builder/`, `pdf/`. Each has a `SKILL.md` with a YAML frontmatter `name`/`description` block.

## Lesson Architecture Pattern

Every lesson `code.py` follows a strict additive pattern:

```
# FROM sNN — unchanged   ← copied verbatim from previous lesson
# NEW in sNN             ← the single mechanism this lesson teaches
```

The `agent_loop()` function itself never changes across lessons — only the tools and surrounding harness mechanisms evolve. This is intentional: the loop belongs to the model; the mechanisms belong to the harness.

### The Core Loop (s01, unchanged through s20)

```python
def agent_loop(messages):
    while True:
        response = client.messages.create(model=MODEL, system=SYSTEM,
                                          messages=messages, tools=TOOLS)
        messages.append({"role": "assistant", "content": response.content})
        if response.stop_reason != "tool_use":
            return
        results = [{"type": "tool_result", "tool_use_id": b.id,
                    "content": TOOL_HANDLERS[b.name](**b.input)}
                   for b in response.content if b.type == "tool_use"]
        messages.append({"role": "user", "content": results})
```

### Lesson Progression

| Lesson | Mechanism | Key addition |
|--------|-----------|--------------|
| s01 | Agent Loop | bare loop + bash tool |
| s02 | Tool Use | dispatch map (`TOOL_HANDLERS`), 4 more tools |
| s03 | Permission | allow/block/approve per-tool policy |
| s04 | Hooks | extension points around the loop (pre/post tool) |
| s05 | TodoWrite | planning before acting |
| s06 | Subagent | spawn child agents with fresh `messages[]` |
| s07 | Skill Loading | on-demand `SKILL.md` injection |
| s08 | Context Compact | multi-layer compaction strategies |
| s09 | Memory | selection, extraction, consolidation |
| s10 | System Prompt | runtime assembly from sections |
| s11 | Error Recovery | retry, fallback model, make-room |
| s12 | Task System | file-backed task graph |
| s13 | Background Tasks | threading + notification injection |
| s14 | Cron Scheduler | time-triggered task execution |
| s15 | Agent Teams | persistent teammates + JSONL mailboxes |
| s16 | Team Protocols | fixed request-reply wire format |
| s17 | Autonomous Agents | self-organizing work claiming from shared board |
| s18 | Worktree Isolation | task ID ↔ git worktree binding |
| s19 | MCP Plugin | external tool routing via MCP |
| s20 | Comprehensive | all mechanisms in one loop |

## Environment Variables

| Variable | Required | Purpose |
|----------|----------|---------|
| `ANTHROPIC_API_KEY` | Yes | Anthropic API key |
| `MODEL_ID` | Yes | Model to use (e.g. `claude-sonnet-4-6`) |
| `ANTHROPIC_BASE_URL` | No | Override for compatible providers (MiniMax, GLM, Kimi, DeepSeek) |
| `FALLBACK_MODEL_ID` | No | Used by s11/s20 error recovery |

## Key Design Principles (for harness code)

- **The loop never changes.** New capabilities register into `TOOL_HANDLERS`, not into the loop itself.
- **Subagents get fresh `messages[]`.** Context isolation prevents noise from leaking across subtasks.
- **Skills load on demand.** `SKILL.md` files are read and injected into the system prompt only when relevant, not upfront.
- **Tool implementations are path-sandboxed.** The `safe_path()` pattern (resolving against `WORKDIR` and checking `is_relative_to`) appears in every file tool from s02 onward.
