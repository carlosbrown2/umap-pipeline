# UMAP Pipeline - Agent Instructions

## Overview

This project uses the Ralph autonomous agent loop for development. Ralph runs AI coding tools repeatedly until all PRD items are complete.

## Commands

```bash
# Run Ralph with Amp (default)
./scripts/ralph/ralph.sh [max_iterations]

# Run Ralph with Claude Code
./scripts/ralph/ralph.sh --tool claude [max_iterations]

# Check PRD status
cat scripts/ralph/prd.json | jq '.userStories[] | {id, title, passes}'
```

## Key Files

- `scripts/ralph/ralph.sh` - The bash loop that spawns fresh AI instances
- `scripts/ralph/prompt.md` - Instructions given to each Amp instance
- `scripts/ralph/prd.json` - User stories with pass/fail status
- `scripts/ralph/progress.txt` - Learnings from previous iterations
- `tasks/` - PRD markdown files

## Workflow

1. Create a PRD in `tasks/prd-[feature-name].md`
2. Convert PRD to `scripts/ralph/prd.json`
3. Run `./scripts/ralph/ralph.sh`

## Codebase Patterns

(Add patterns discovered during development here)
