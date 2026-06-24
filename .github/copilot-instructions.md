Repository: copilot-agents-lab — Copilot session instructions

1) Build, test, and lint commands

- No build/test/lint scripts detected in this repository (no package.json or CI config present).
- Project includes lab instructions that use the Copilot CLI (Node.js required).
- Copilot CLI example commands used in labs:
  - npm install -g @github/copilot
  - copilot                 # start, authenticate, confirm workspace trust
  - /init                   # generate Copilot project instructions
  - /mcp                    # view/add/edit MCP servers
  - copilot --allow-tool 'shell'
  - copilot --deny-tool 'write'
  - copilot -p "list open issues" --allow-tool 'shell'

- If/when tests are added (Node.js/Jest, etc.), run a single test with the test runner directly (example patterns):
  - npm test -- -t "<test name regex>"    # Jest
  - pytest tests/test_file.py::test_name   # pytest
  - go test ./pkg -run TestName             # Go

2) High-level architecture (big picture)

- Purpose: an exam-focused sandbox for GH-600 (Copilot agentic developer) labs. Not a production app.
- Top-level layout (from README):
  - labs/      — one lab per skill (disposable, guided exercises)
  - evidence/  — canonical output artifacts from labs (logs, configs, PR links). This is the primary durable area.
  - README.md, lab markdowns — authoritative guidance for each lab

- Principal workflow:
  1. Execute a lab in labs/ (interactive CLI + Copilot agent).
  2. Capture concrete artifacts into evidence/ (e.g., mcp config JSON, session logs, notes).
  3. Only promote working artifacts to `agent-squad` after validation per the Promotion Rule.

- MCP & Copilot CLI: labs exercise MCP servers and allow lists. The Copilot CLI reads/writes config under ~/.copilot (config file: ~/.copilot/mcp-config.json). COPILOT_HOME overrides this path.

3) Key conventions and patterns specific to this repository

- evidence/ is authoritative: every lab's "definition of done" requires saving real outputs into evidence/. File naming conventions used in labs:
  - lab-01-mcp-config.json
  - lab-01-session.log
  - lab-01-notes.md

- Promotion rule: Nothing moves into agent-squad until validated. Treat this repo as a staging/sandbox area for reproducible artifacts, not as an upstream integration point.

- Labs are single-purpose, disposable, and documented: follow each lab's Definition of Done and capture exactly the named artifacts.

- MCP-focused flags and checks: labs rely on explicit allow/deny flags to exercise enforcement vs advisory controls. When reproducing a lab, record both the CLI flags used and the resulting ~/.copilot/mcp-config.json copied into evidence/.

- COPILOT_HOME: if running multiple sessions or CI, set COPILOT_HOME to isolate config between runs.

4) Files to incorporate into Copilot prompts

- README.md and LAB01.md are the canonical context sources. When initiating an agent session for a lab, include the relevant lab markdown and evidence/ naming rules in the prompt.

5) AI assistant / other tooling configs

- No existing AI assistant config files were found (CLAUDE.md, .cursorrules, AGENTS.md, .windsurfrules, CONVENTIONS.md, etc.). If such files are added later, include their content into Copilot session context.

6) Quick checklist for starting a Copilot session for a lab

- Start Copilot CLI and authenticate: copilot
- Run /init to generate project instructions if needed
- Add or inspect MCP servers: /mcp and copy ~/.copilot/mcp-config.json to evidence/
- Use explicit --allow-tool / --deny-tool flags for repeatability
- Save session logs and notes into evidence/ using the lab's recommended filenames

Summary of changes

- Created .github/copilot-instructions.md containing build/test discovery, Copilot CLI usage from LAB01, high-level architecture, and repository-specific conventions.

Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>