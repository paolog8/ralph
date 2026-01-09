# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What is Ralph?

Ralph is an autonomous AI agent loop that spawns fresh Amp instances repeatedly until all PRD items are complete. Each iteration has clean context - memory persists only through git history, `progress.txt`, and `prd.json`.

## Commands

### Running Ralph (in projects that use it)
```bash
./ralph.sh [max_iterations]  # Default: 10 iterations
```

### Flowchart Development
```bash
cd flowchart
npm install       # First time setup
npm run dev       # Development server
npm run build     # Build for production
npm run lint      # Run linter
```

### Debugging Ralph State
```bash
# Check which stories are complete
cat prd.json | jq '.userStories[] | {id, title, passes}'

# View learnings from previous iterations
cat progress.txt

# Check recent commits
git log --oneline -10
```

## Architecture

### Core Loop (ralph.sh)
- Spawns fresh Amp instances in a loop (max 10 iterations by default)
- Pipes `prompt.md` into each Amp instance with `--dangerously-allow-all`
- Archives previous runs when `branchName` changes
- Exits when `<promise>COMPLETE</promise>` appears in output
- Each iteration is stateless - no memory carries over except:
  - Git history (committed work)
  - `progress.txt` (learnings and context)
  - `prd.json` (story completion status)

### Key Files
- `ralph.sh` - Main bash loop that orchestrates iterations
- `prompt.md` - Instructions given to each Amp instance
- `prd.json` - User stories with `passes` boolean (task list)
- `prd.json.example` - Reference for PRD format
- `progress.txt` - Append-only log of learnings for future iterations
- `skills/` - Amp skills for generating and converting PRDs
- `flowchart/` - React Flow visualization (Vite + TypeScript + React)

### PRD Format (prd.json)
```json
{
  "project": "string",
  "branchName": "ralph/feature-name",
  "description": "string",
  "userStories": [
    {
      "id": "US-001",
      "title": "string",
      "description": "As a [user], I want [feature] so that [benefit]",
      "acceptanceCriteria": ["criterion1", "Typecheck passes"],
      "priority": 1,
      "passes": false,
      "notes": ""
    }
  ]
}
```

### Iteration Workflow
Each Amp instance follows this cycle:
1. Read `prd.json` and `progress.txt`
2. Checkout/create branch from `branchName`
3. Pick highest priority story where `passes: false`
4. Implement that single story
5. Run quality checks (typecheck, tests, etc.)
6. Commit if checks pass: `feat: [Story ID] - [Story Title]`
7. Update `prd.json` to mark `passes: true`
8. Append learnings to `progress.txt`
9. If all stories pass:
   - Push branch to remote
   - Create pull request with summary of all completed stories
   - Output `<promise>COMPLETE</promise>`

## Skills

### PRD Skill (`skills/prd/`)
Generates structured Product Requirements Documents:
- Asks 3-5 clarifying questions with lettered options
- Creates markdown PRD in `tasks/prd-[feature-name].md`
- User stories must be small (completable in one context window)
- Always includes "Typecheck passes" in acceptance criteria
- UI stories include "Verify in browser using dev-browser skill"

### Ralph Skill (`skills/ralph/`)
Converts markdown PRDs to `prd.json` format:
- Splits large features into small stories
- Orders stories by dependency (schema → backend → UI)
- Ensures acceptance criteria are verifiable
- Archives previous runs when branch changes

## Critical Concepts

### Story Size
Each story must be completable in ONE Amp iteration (one context window). Too large = LLM runs out of context = broken code.

**Right-sized:**
- Add database column and migration
- Add UI component to existing page
- Add filter dropdown to list

**Too large (split these):**
- "Build entire dashboard"
- "Add authentication"
- "Refactor the API"

### Story Dependencies
Stories execute in priority order. Earlier stories cannot depend on later ones.

**Correct:** Schema → Backend → UI → Dashboard
**Wrong:** UI (depends on schema) → Schema

### AGENTS.md Updates
Ralph updates relevant `AGENTS.md` files with learnings after each iteration. Amp automatically reads these files, so discoveries benefit future iterations.

**Add to AGENTS.md:**
- Patterns discovered ("this codebase uses X for Y")
- Gotchas ("don't forget to update Z when changing W")
- Useful context ("settings panel is in component X")

### Browser Verification for UI
Frontend stories must include "Verify in browser using dev-browser skill" in acceptance criteria. Ralph uses dev-browser skill to navigate, interact, and confirm changes visually.

### Quality Checks
Ralph only commits code that passes quality checks. Define project-specific checks in your project's setup. Common checks:
- Typecheck (always required)
- Tests
- Lint
- Build

### Archiving
Ralph automatically archives previous runs when `branchName` in `prd.json` changes. Archives saved to `archive/YYYY-MM-DD-feature-name/`.

## Flowchart

Interactive React Flow visualization at `flowchart/`:
- Built with Vite + React + TypeScript + React Flow
- Click-through presentation with animations
- Deployed to GitHub Pages via workflow

## Configuration

### Recommended Amp Settings
Add to `~/.config/amp/settings.json`:
```json
{
  "amp.experimental.autoHandoff": { "context": 90 }
}
```

Enables automatic handoff when context fills, allowing Ralph to handle large stories.

### Installing Skills Globally
```bash
cp -r skills/prd ~/.config/amp/skills/
cp -r skills/ralph ~/.config/amp/skills/
```

## Working with This Repository

This repo contains Ralph's core files - it's meant to be copied into projects or installed as Amp skills. It's not a standalone application.

### Typical Usage
1. Copy `ralph.sh` and `prompt.md` to your project's `scripts/ralph/`
2. Or install skills globally to `~/.config/amp/skills/`
3. In your project, use the PRD skill to generate requirements
4. Use the Ralph skill to convert PRD to `prd.json`
5. Run `./scripts/ralph/ralph.sh` to execute autonomously
