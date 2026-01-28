# Ralph

![Ralph](ralph.webp)

Ralph is an autonomous AI agent loop that runs a command-line AI tool repeatedly until all PRD items are complete. Each iteration is a fresh instance with clean context. Memory persists via git history, `progress.txt`, and `prd.json`.

This version is adapted to work with multiple AI CLIs:
- **Amp** (`ralph-amp.sh`)
- **Claude** (`ralph-claude.sh`)
- **Gemini** (`ralph-gemini.sh`)

Based on [Geoffrey Huntley's Ralph pattern](https://ghuntley.com/ralph/).

[Read my in-depth article on how I use Ralph](https://x.com/ryancarson/status/2008548371712135632)

## Prerequisites

- One of the supported AI CLIs installed and authenticated:
  - [Amp CLI](https://ampcode.com)
  - [Anthropic's Claude CLI](https://github.com/anthropics/anthropic-sdk-python/tree/main/src/anthropic_cli)
  - [Google's Gemini CLI](https://github.com/google/gemini-cli)
- `jq` installed (`brew install jq` on macOS)
- A git repository for your project

## Setup

### Option 1: Copy to your project

Copy the ralph files into your project:

```bash
# From your project root
mkdir -p scripts/ralph
cp /path/to/ralph/ralph-*.sh scripts/ralph/
cp /path/to/ralph/prompt.md scripts/ralph/
chmod +x scripts/ralph/ralph-*.sh
```

### Option 2: Install skills globally (for Amp)

Copy the skills to your Amp config for use across all projects:

```bash
cp -r skills/prd ~/.config/amp/skills/
cp -r skills/ralph ~/.config/amp/skills/
```

### Configure Amp auto-handoff (recommended for Amp)

Add to `~/.config/amp/settings.json`:

```json
{
  "amp.experimental.autoHandoff": { "context": 90 }
}
```

This enables automatic handoff when context fills up, allowing Ralph to handle large stories that exceed a single context window.

## Workflow

### 1. Create a PRD

Use the PRD skill to generate a detailed requirements document:

```
Load the prd skill and create a PRD for [your feature description]
```

Answer the clarifying questions. The skill saves output to `tasks/prd-[feature-name].md`.

### 2. Convert PRD to Ralph format

Use the Ralph skill to convert the markdown PRD to JSON:

```
Load the ralph skill and convert tasks/prd-[feature-name].md to prd.json
```

This creates `prd.json` with user stories structured for autonomous execution.

### 3. Run Ralph

Choose the script corresponding to your desired AI tool:

**For Amp:**
```bash
./scripts/ralph/ralph-amp.sh [max_iterations]
```

**For Claude:**
```bash
./scripts/ralph/ralph-claude.sh [max_iterations]
```

**For Gemini:**
```bash
./scripts/ralph/ralph-gemini.sh [max_iterations]
```

The default is 10 iterations.

Ralph will:
1. Create a feature branch (from PRD `branchName`)
2. Pick the highest priority story where `passes: false`
3. Implement that single story
4. Run quality checks (typecheck, tests)
5. Commit if checks pass
6. Update `prd.json` to mark story as `passes: true`
7. Append learnings to `progress.txt`
8. Repeat until all stories pass or max iterations reached

## Key Files

| File | Purpose |
|------|---------|
| `ralph-amp.sh`| The bash loop that spawns fresh **Amp** instances. |
| `ralph-claude.sh`| The bash loop that spawns fresh **Claude** instances. |
| `ralph-gemini.sh`| The bash loop that spawns fresh **Gemini** instances. |
| `prompt.md` | Instructions given to each AI instance. |
| `prd.json` | User stories with `passes` status (the task list). |
| `prd.json.example` | Example PRD format for reference. |
| `progress.txt` | Append-only learnings for future iterations. |
| `skills/prd/` | (Amp) Skill for generating PRDs. |
| `skills/ralph/` | (Amp) Skill for converting PRDs to JSON. |
| `flowchart/` | Interactive visualization of how Ralph works. |

## Flowchart

[![Ralph Flowchart](ralph-flowchart.png)](https://snarktank.github.io/ralph/)

**[View Interactive Flowchart](https://snarktank.github.io/ralph/)** - Click through to see each step with animations.

The `flowchart/` directory contains the source code. To run locally:

```bash
cd flowchart
npm install
npm run dev
```

## Critical Concepts

### Each Iteration = Fresh Context

Each iteration spawns a **new AI instance** with clean context. The only memory between iterations is:
- Git history (commits from previous iterations)
- `progress.txt` (learnings and context)
- `prd.json` (which stories are done)

### Small Tasks

Each PRD item should be small enough to complete in one context window. If a task is too big, the LLM runs out of context before finishing and produces poor code.

Right-sized stories:
- Add a database column and migration
- Add a UI component to an existing page
- Update a server action with new logic
- Add a filter dropdown to a list

Too big (split these):
- "Build the entire dashboard"
- "Add authentication"
- "Refactor the API"
