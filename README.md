# Agent QA

Autonomous three-phase QA agent skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Analyzes any codebase regardless of language or framework, traces every functionality end-to-end to verify correctness, and fixes confirmed issues.

## How it works

```
Phase 1: Analysis     → Scans codebase, maps architecture, generates checkpoints per module
Phase 2: Verification → Traces each checkpoint through source code, produces PASS/FAIL/WARN report
Phase 3: Fixes        → Presents findings for approval, applies fixes, logs all changes
```

The agent auto-detects which phase to run based on existing files in `agent_qa/`:

| State | Action |
|-------|--------|
| No `agent_qa/` folder | Start Phase 1 |
| `QA_ANALYSIS.md` exists, no `QA_REPORT.md` | Start Phase 2 |
| `QA_REPORT.md` exists, no `QA_FIXES.md` | Start Phase 3 |
| All three files exist | Ask which phase to re-run |

## Phase 1: Deep Analysis

Scans the entire codebase systematically:

- Reads config files, entry points, and main modules
- Identifies architecture pattern (MVC, hexagonal, microservices, etc.)
- Maps module boundaries and dependency graph
- Traces data flow: entry points > processing > storage > output
- Generates `agent_qa/QA_ANALYSIS.md` with granular, traceable checkpoints per module

## Phase 2: Code Tracing & Verification

For each checkpoint, the agent:

1. Locates the relevant source code
2. Traces the execution path end-to-end (forward, backward, and data traces)
3. Evaluates against checkpoint criteria
4. Records **PASS**, **FAIL**, or **WARN** with file:line evidence and severity

Produces `agent_qa/QA_REPORT.md` with an executive summary, health score, and findings sorted by severity (CRITICAL > HIGH > MEDIUM > LOW).

## Phase 3: Fix Iteration

Presents all findings in a consolidated table and asks for confirmation:

- **Fix all** — proceed with every FAIL and WARN
- **Fix by severity** — e.g., "only CRITICAL and HIGH"
- **Select specific** — pick issues by number
- **Skip** — end without fixing

Applies minimal fixes one at a time, shows before/after for each, and generates `agent_qa/QA_FIXES.md` as a change log.

## Installation

**Global (all projects):**

```bash
git clone https://github.com/bmartinezg/agent-qa.git ~/.claude/skills/agent-qa
```

**Per project (committed to repo):**

```bash
git clone https://github.com/bmartinezg/agent-qa.git .claude/skills/agent-qa
```

## Usage

Once installed, trigger the agent in Claude Code with any of these:

- `agent qa`
- `run QA`
- `code audit`
- `scan project`
- `quality check`
- `deep code review`
- `inspect code`

## Re-running phases

Delete the corresponding output file to re-run a phase:

```bash
rm agent_qa/QA_FIXES.md    # re-run Phase 3
rm agent_qa/QA_REPORT.md   # re-run Phase 2
rm -rf agent_qa/            # start fresh from Phase 1
```

## Output files

All outputs are written to `agent_qa/` at the project root:

| File | Phase | Contents |
|------|-------|----------|
| `QA_ANALYSIS.md` | 1 | Project map, architecture, module checkpoints |
| `QA_REPORT.md` | 2 | Verification results with PASS/FAIL/WARN per checkpoint |
| `QA_FIXES.md` | 3 | Log of all applied fixes with before/after code |

## License

MIT
