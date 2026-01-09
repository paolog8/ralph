# Ralph Agent Instructions

## Overview

Ralph is an autonomous AI agent loop that runs Amp or Claude Code repeatedly until all PRD items are complete. Each iteration is a fresh agent instance with clean context.

## Commands

```bash
# Run the flowchart dev server
cd flowchart && npm run dev

# Build the flowchart
cd flowchart && npm run build

# Run Ralph (from your project that has prd.json)
./ralph.sh <agent> [max_iterations]

# Examples:
./ralph.sh amp          # Run with Amp, 10 iterations
./ralph.sh claude       # Run with Claude Code (Opus Plan), 10 iterations
./ralph.sh claude 5     # Run with Claude Code, 5 iterations
```

## Key Files

- `ralph.sh` - The bash loop that spawns fresh agent instances (Amp or Claude Code)
- `prompt.md` - Instructions given to each agent instance
- `prd.json.example` - Example PRD format
- `flowchart/` - Interactive React Flow diagram explaining how Ralph works

## Flowchart

The `flowchart/` directory contains an interactive visualization built with React Flow. It's designed for presentations - click through to reveal each step with animations.

To run locally:
```bash
cd flowchart
npm install
npm run dev
```

## Patterns

- Each iteration spawns a fresh agent instance (Amp or Claude Code) with clean context
- Memory persists via git history, `progress.txt`, and `prd.json`
- Stories should be small enough to complete in one context window
- Always update AGENTS.md with discovered patterns for future iterations
- When all stories pass, Ralph creates a pull request before exiting
- Claude Code uses Opus Plan model by default for extended planning and reasoning
- **All feature branches are created from `develop`**, not `main` (Git Flow)
- Pull requests target the `develop` branch
