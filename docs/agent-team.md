# Agent team

I'm using GitHub Copilot CLI inside a Codespace to orchestrate a team of four custom subagents that together will build Mona's Project Pulse dashboard. Each agent has a focused responsibility, a target model, and a definition file under `.github/agents/`.

## The team

### Orchestrator

- **Model:** Claude Opus 4.7 (copilot)
- **Definition:** `.github/agents/orchestrator.agent.md`
- **Responsibility:** Coordinates the team from the Copilot CLI. Takes the Planner's plan, breaks it into phases with explicit file scopes, delegates to the Coder and Designer in parallel when scopes don't overlap, and verifies the integrated result. Does not write code itself.

### Planner

- **Model:** Claude Opus 4.7 (copilot)
- **Definition:** `.github/agents/planner.agent.md`
- **Responsibility:** Researches the repository, docs, and dependencies, then produces an ordered implementation plan with file assignments, sequencing, parallelizable work, edge cases, and open questions. Does not write code.

### Coder

- **Model:** GPT-5.5 (copilot)
- **Definition:** `.github/agents/coder.agent.md`
- **Responsibility:** Implements the Project Pulse HTML/JS logic and supporting runnable-app configuration (for example `.vscode/launch.json` with `cwd` set to `${workspaceFolder}/app` opening `index.html`) within the file scope assigned by the Orchestrator. Keeps code explicit, deterministic, and testable.

### Designer

- **Model:** Gemini 3.1 Pro (copilot)
- **Definition:** `.github/agents/designer.agent.md`
- **Responsibility:** Owns UI/UX, accessibility, information hierarchy, and visual design for the dashboard. Delivers a polished Project Pulse look with project cards, status badges, priority treatment, responsive layout, and deterministic CSS hooks like `.dashboard` and `.project-card`.

## Orchestration environment

I'm running this work through **GitHub Copilot CLI in a GitHub Codespace**. The Orchestrator agent invokes the Planner, Coder, and Designer as subagents from the CLI, and I control all git operations (stage, commit, push) myself — the agents never touch git.
