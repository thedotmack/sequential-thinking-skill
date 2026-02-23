# ultrathink

A Claude Code skill that replicates the [Sequential Thinking MCP server](https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking) with full feature parity — no MCP required.

Structured problem-solving through a chain of numbered thoughts that can **branch**, **revise**, and **adapt dynamically**.

## Why

The Sequential Thinking MCP server is useful but requires MCP infrastructure. This skill provides the same capabilities as a standalone Claude Code skill with a TypeScript state machine that persists `thoughtHistory[]` and `branches{}` to disk.

## Features

- Sequential numbered thoughts with formatted box output
- **Revisions** — reconsider and correct previous thoughts without losing the original
- **Branching** — explore alternative paths from any thought, tracked by branch ID
- **Dynamic depth** — adjust `totalThoughts` up or down as the problem unfolds
- **Hypothesis/verification loops** — generate and verify solutions iteratively
- **Persistent state** — `think.ts` maintains history across invocations via JSON on disk
- **`--status` inspection** — dump full thought history and branch details at any point

## Installation

Copy the `sequential-thinking/` folder into `~/.claude/skills/`:

```bash
cp -r sequential-thinking ~/.claude/skills/
```

Or extract the packaged `.skill` file:

```bash
unzip sequential-thinking.skill -d ~/.claude/skills/
```

Requires [`tsx`](https://github.com/privatenumber/tsx) for the state machine script.

## Usage

The skill triggers automatically in Claude Code when problems benefit from structured multi-step reasoning. It uses `scripts/think.ts` to track state:

```bash
# Reset for a new thinking session
tsx scripts/think.ts --reset

# Submit a thought
tsx scripts/think.ts --thought "Initial analysis" --thoughtNumber 1 --totalThoughts 5 --nextThoughtNeeded true

# Revise a previous thought
tsx scripts/think.ts --thought "Reconsidering..." --thoughtNumber 3 --totalThoughts 5 --nextThoughtNeeded true --isRevision --revisesThought 1

# Branch from a thought
tsx scripts/think.ts --thought "Alternative approach" --thoughtNumber 4 --totalThoughts 7 --nextThoughtNeeded true --branchFromThought 2 --branchId alt-path

# Check full state
tsx scripts/think.ts --status
```

Each invocation outputs a formatted thought box and returns JSON status:

```
┌────────────────────────────────────────────────┐
│ 🌿 Branch 4/7 (from thought 2, ID: alt-path)   │
├────────────────────────────────────────────────┤
│ Alternative approach                            │
└────────────────────────────────────────────────┘
```

```json
{
  "thoughtNumber": 4,
  "totalThoughts": 7,
  "nextThoughtNeeded": true,
  "branches": ["alt-path"],
  "thoughtHistoryLength": 4
}
```

## Structure

```
sequential-thinking/
├── SKILL.md                       # Skill protocol and instructions
├── scripts/
│   └── think.ts                   # TypeScript state machine (tsx)
└── references/
    └── example-session.md         # Worked example with all features
```

## License

MIT
