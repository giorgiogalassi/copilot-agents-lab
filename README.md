# copilot-agents-lab

Study sandbox for the **GH-600: GitHub Certified Agentic AI Developer** certification.

This repository is not part of `agent-squad`. It is a clean, GitHub-centric environment for learning Copilot primitives (cloud/coding agent, Copilot CLI, SDK, MCP, Memory, and guardrails) by rebuilding them from scratch rather than importing them from the squad.

Mental model: GitHub is both the platform and the control plane. Issues, branches, pull requests, checks, Actions, and branch protection are the accountability primitives. No vault, no `.squad/`. The squad remains separate and clean.

## Promotion Rule

Nothing moves from this repository into `agent-squad` until it has been validated.

When a Copilot-native agent actually works and has been properly tested, it can then be evaluated for inclusion in a `copilot/` tree within the squad. The relationship between sandbox and squad is the same as the relationship between project decisions and `development.md`: local practices are promoted to the global level only after validation.

## Exam Domain Map

| # | Domain | Weight | Status | Lab |
|---|---|---|---|---|
| 1 | Prepare agent architecture and SDLC processes | 15–20% | To do | - |
| 2 | Implement tool use and environment interaction | 20–25% | In progress | lab-01 |
| 3 | Manage memory, state, and execution | 10–15% | To do | - |
| 4 | Perform evaluation, error analysis, and tuning | 15–20% | To do | - |
| 5 | Orchestrate multi-agent coordination | 15–20% | To do | - |
| 6 | Implement guardrails and accountability | 10–15% | To do | - |

Study order (prioritized by weight and current gaps): D2, D1, D5, D4, D3, D6.

## Structure

```text
copilot-agents-lab/
  README.md            this file
  labs/                one lab per skill, GitHub-native and disposable
  evidence/            real outputs captured from labs (logs, configs, PR links)
```

`evidence/` is the key area. The exam is scenario-based and requires recognizing implementation evidence. Each lab concludes by saving a real artifact (MCP configuration, session logs, PR links, check outputs, etc.) in this directory.

## Official References

- Study Guide: https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/gh-600
- Reference Squad: https://github.com/giorgiogalassi/agent-squad
- Squad-to-Exam Crosswalk: squad-crosswalk.md
- CLI Comparison (Claude Code vs Copilot CLI): cli-comparison.md
- Copilot Coding Agent: https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent
- MCP and Coding Agent: https://docs.github.com/en/copilot/concepts/agents/coding-agent/mcp-and-coding-agent
- Copilot Memory: https://docs.github.com/en/copilot/concepts/agents/copilot-memory
- Guardrails: https://docs.github.com/en/copilot/concepts/agents/coding-agent/risks-and-mitigations
