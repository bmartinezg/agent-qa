---
name: agent-qa
description: >
  Autonomous QA agent that performs deep code quality inspection in three phases.
  Phase 1: Full codebase analysis — maps architecture, stack, modules, dependencies,
  and generates a structured test-case document with every functionality and verification
  checkpoint organized by module. Phase 2: Code tracing and verification — traces each
  functionality end-to-end through the source code, verifying integrations, data flow,
  naming conventions, formatting, logic correctness, and edge cases. Produces a detailed
  report with pass/fail results per checkpoint. Phase 3: Fix iteration — presents all
  findings to the user for confirmation, then iterates through approved issues applying
  fixes directly in the codebase. Use when the user says "agent qa", "QA",
  "quality assurance", "quality inspection", "code audit", "code scan", "deep code review",
  "trace code", "verify code", "scan project", "quality check", "code quality", "audit code",
  "inspect code", or "run QA".
---

# Agent QA - Quality Assurance

Autonomous three-phase code quality agent. Analyzes any codebase regardless of language or framework, traces every functionality end-to-end to verify correctness, and fixes confirmed issues.

## Workflow

```
Phase 1: Analysis     → agent_qa/QA_ANALYSIS.md  (full project map + checkpoints)
Phase 2: Verification → agent_qa/QA_REPORT.md    (traced results + findings)
Phase 3: Fixes        → Code changes + agent_qa/QA_FIXES.md (fix log)
```

### Phase selection

- **No `agent_qa/` folder exists** → Start Phase 1
- **`QA_ANALYSIS.md` exists but no `QA_REPORT.md`** → Start Phase 2
- **`QA_REPORT.md` exists but no `QA_FIXES.md`** → Start Phase 3
- **All three files exist** → Ask user which phase to re-run

## Phase 1: Deep Analysis

**Goal**: Understand every aspect of the project and produce a comprehensive checkpoint document.

### Step 1 — Project discovery

1. Create `agent_qa/` directory at project root
2. Scan the entire codebase systematically:
   - Read config files first (package.json, pyproject.toml, Cargo.toml, go.mod, Gemfile, pom.xml, etc.)
   - Read entry points and main modules
   - Map directory structure
3. Collect: languages, frameworks, build tools, dependencies, environment config, CI/CD setup

### Step 2 — Architecture mapping

1. Identify architectural pattern (MVC, hexagonal, microservices, monolith, serverless, etc.)
2. Map all modules and their boundaries
3. Trace dependency graph between modules
4. Identify external integrations (APIs, databases, message queues, third-party services)
5. Map data flow: entry points → processing → storage → output

### Step 3 — Generate QA_ANALYSIS.md

Use the template and checklist in [references/phase1-analysis.md](references/phase1-analysis.md).

Produce `agent_qa/QA_ANALYSIS.md` following the exact structure defined there. Every functionality must have granular, traceable checkboxes.

### Step 4 — Present summary to user

After generating, show:
- Total modules found
- Total checkpoints generated
- High-level module list
- Ask user if they want to proceed to Phase 2 or review/adjust first

## Phase 2: Code Tracing & Verification

**Goal**: Trace each checkpoint from QA_ANALYSIS.md through the actual source code, verify correctness, and produce a detailed report.

### Step 1 — Load analysis

1. Read `agent_qa/QA_ANALYSIS.md` completely
2. Build a work queue of all unchecked checkpoints, grouped by module

### Step 2 — Systematic verification

For each module, for each functionality, for each checkpoint:

1. Locate the relevant source code
2. Read and trace the code path end-to-end
3. Verify against the specific checkpoint criteria
4. Record: PASS, FAIL, or WARN with evidence

Follow the detailed verification procedures in [references/phase2-verification.md](references/phase2-verification.md).

### Step 3 — Generate QA_REPORT.md

Produce `agent_qa/QA_REPORT.md` with:
- Every checkpoint marked with result
- Findings with file:line references
- Severity classification for issues
- Executive summary with metrics

### Step 4 — Present results

Show:
- Pass/Fail/Warn counts
- Critical findings (if any)
- Top issues by severity
- Overall health score (percentage of passes)
- Ask user if they want to proceed to Phase 3

## Phase 3: Fix Iteration

**Goal**: Present findings to the user for approval, then iterate through confirmed issues applying fixes in the codebase.

### Step 1 — Load report

1. Read `agent_qa/QA_REPORT.md` completely
2. Build a numbered list of all FAIL and WARN findings

### Step 2 — Present findings for confirmation

Show the user a consolidated table of all issues:

```
#  | Severity | Module         | Issue                          | Location
1  | CRITICAL | auth           | SQL injection in user query    | src/api/users.py:47
2  | HIGH     | payments       | Missing null check on amount   | src/payments/charge.py:23
3  | MEDIUM   | utils          | Generic exception catch        | src/utils/http.py:91
...
```

Ask the user:
- **"Fix all"** → proceed with every FAIL and WARN
- **"Fix by severity"** → e.g., "only CRITICAL and HIGH"
- **"Select specific"** → user provides numbers to fix (e.g., "1, 2, 5")
- **"Skip"** → end without fixing

Do NOT proceed until the user explicitly confirms.

### Step 3 — Iterate and fix

For each confirmed issue, in order of severity (CRITICAL first):

1. Read the relevant source code at the referenced location
2. Understand the surrounding context
3. Apply the minimal fix that resolves the issue without breaking existing functionality
4. Show the user the change made (before → after)
5. Move to next issue

**Rules:**
- One fix at a time — do not batch multiple fixes in one edit
- Minimal changes only — fix the issue, do not refactor surrounding code
- If a fix is ambiguous or risky, ask the user before applying
- If a fix could affect other modules, warn the user first

### Step 4 — Generate QA_FIXES.md

Produce `agent_qa/QA_FIXES.md` with a log of all changes:

See [references/phase3-fixes.md](references/phase3-fixes.md) for the template.

### Step 5 — Present summary

Show:
- Total issues fixed
- Issues skipped (with reason)
- Recommendation: re-run Phase 2 to verify fixes if many changes were made

## Installation & Usage

### Install in a project

**Option A — Global (all projects):**
```bash
# Unzip into global skills directory
unzip agent-qa.skill -d ~/.claude/skills/
```

**Option B — Per project (committed to repo):**
```bash
# Unzip into project-level skills directory
unzip agent-qa.skill -d .claude/skills/
```

### Run

Once installed, trigger the agent in Claude Code with any of these:
- "run QA"
- "agent qa"
- "code audit"
- "scan project"
- "quality check"

The agent auto-detects which phase to run based on existing files in `agent_qa/`.

### Re-run

To re-run a specific phase:
- Delete `agent_qa/QA_FIXES.md` → re-runs Phase 3
- Delete `agent_qa/QA_REPORT.md` → re-runs Phase 2
- Delete `agent_qa/` entirely → starts fresh from Phase 1
