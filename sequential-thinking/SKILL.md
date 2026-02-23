---
name: sequential-thinking
description: "Dynamic, reflective problem-solving through structured sequential thoughts with support for branching, revision, and adaptive depth. Use this skill when: (1) Breaking down complex problems into steps, (2) Planning and design with room for revision, (3) Analysis that might need course correction, (4) Problems where the full scope is not clear initially, (5) Multi-step solutions requiring maintained context, (6) Situations where irrelevant information must be filtered out, (7) Any task benefiting from hypothesis generation, verification, and iterative refinement. Triggers: think through, step by step, break this down, sequential thinking, reason through, analyze step by step, think carefully, or when a problem clearly benefits from structured multi-step reasoning."
---

# Sequential Thinking

Structured problem-solving through a chain of numbered thoughts that can branch, revise, and adapt dynamically. Full parity with the Sequential Thinking MCP server.

## State Machine Script

Use `scripts/think.ts` (run via `tsx`) to track state programmatically. This maintains `thoughtHistory[]` and `branches{}` persistently across invocations — identical to the MCP server's in-memory state.

**IMPORTANT:** Run `tsx scripts/think.ts --reset` at the start of every new thinking session.

### Commands

```bash
# Reset state for a new session
tsx scripts/think.ts --reset

# Submit a thought (required flags: --thought, --thoughtNumber, --totalThoughts, --nextThoughtNeeded)
tsx scripts/think.ts --thought "analysis here" --thoughtNumber 1 --totalThoughts 5 --nextThoughtNeeded true

# Submit a revision
tsx scripts/think.ts --thought "revised analysis" --thoughtNumber 3 --totalThoughts 5 --nextThoughtNeeded true --isRevision --revisesThought 1

# Submit a branch
tsx scripts/think.ts --thought "alternative path" --thoughtNumber 4 --totalThoughts 7 --nextThoughtNeeded true --branchFromThought 2 --branchId alt-approach

# Signal need for more thoughts
tsx scripts/think.ts --thought "more to explore" --thoughtNumber 6 --totalThoughts 8 --nextThoughtNeeded true --needsMoreThoughts

# View full state (history, branches, counts)
tsx scripts/think.ts --status
```

### Response Format

Each invocation prints the formatted thought box to stderr and returns JSON to stdout:

```json
{
  "thoughtNumber": 3,
  "totalThoughts": 7,
  "nextThoughtNeeded": true,
  "branches": ["alt-path"],
  "thoughtHistoryLength": 3
}
```

`--status` additionally includes `fullHistory` and `branchDetails` for inspecting the complete chain.

## Protocol

For each thought, the script enforces these rules automatically:

- **Auto-adjust**: If `thoughtNumber` exceeds `totalThoughts`, `totalThoughts` is raised to match
- **Append-only history**: Thoughts are never deleted, only appended
- **Branch tracking**: Branches require both `--branchFromThought` and `--branchId`
- **Revision tracking**: Revisions require both `--isRevision` and `--revisesThought`

### Rules

1. **Start** with an initial estimate of `totalThoughts`, but adjust freely as understanding deepens
2. **Auto-adjust**: If `thoughtNumber` exceeds `totalThoughts`, raise `totalThoughts` to match
3. **Revise** previous thoughts by setting `isRevision: true` and `revisesThought: N` — the original stays in history; the revision is a new entry
4. **Branch** by setting both `branchFromThought: N` and `branchId: "label"` — explore alternative paths without losing the main line
5. **Extend** beyond the initial estimate at any time by setting `needsMoreThoughts: true` and increasing `totalThoughts`
6. **Express uncertainty** — not every thought needs confidence; questioning and exploring is encouraged
7. **Filter noise** — ignore information irrelevant to the current step
8. **Generate hypotheses** when appropriate, then verify them against prior chain-of-thought steps
9. **Repeat** hypothesis-verification cycles until satisfied
10. **Terminate** only when a satisfactory answer is reached by setting `nextThoughtNeeded: false`
11. **Non-linear paths are first-class** — branching, backtracking, and revision are not failures but features

### Process

1. Assess the problem and estimate `totalThoughts`
2. Begin thought 1 with initial analysis
3. For each subsequent thought, decide:
   - **Continue linearly** → increment `thoughtNumber`
   - **Revise** → set `isRevision` + `revisesThought`
   - **Branch** → set `branchFromThought` + `branchId`
   - **Extend** → increase `totalThoughts` + set `needsMoreThoughts`
4. Generate a solution hypothesis when evidence is sufficient
5. Verify the hypothesis against the chain of thought
6. If verification fails, revise or branch and continue
7. Set `nextThoughtNeeded: false` only when confident in the final answer
8. Provide the final answer as the last thought

### Example Session

See [references/example-session.md](references/example-session.md) for a complete worked example demonstrating normal thoughts, revisions, branches, and dynamic depth adjustment.
