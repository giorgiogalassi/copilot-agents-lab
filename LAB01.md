# Lab 01 — MCP Server and Allow Lists (Domain 2)

Objective: connect an MCP server to a Copilot agent, configure an allow list, and observe what the agent can and cannot invoke. This sits at the core of the heaviest-weighted domain in the exam.

## What You Already Know vs. What Is New

Transferable concept: role-based permissioning. In the squad, you already restrict tools by agent (Reven is read-only, Cody has workspace-write access, tool lists are constrained). An MCP allow list is the same idea, moved to the platform level.

What is new—and the focus of this lab—is advisory versus enforced controls. In the squad, restrictions are prompt-based: the agent KNOWS it should not do something. Here, the allow list makes the action impossible: the platform simply will not allow it. Keep this distinction in mind throughout the lab.

## Exam Skills Covered (Study Guide D2)

- Add an MCP server as a tool to an agent
- Configure MCP allow lists
- Configure agent tool permissions
- Implement traceability and accountability for agent actions

## Prerequisites

- A GitHub Copilot-enabled account (Copilot CLI is included with all Copilot plans)
- Node.js to install the CLI via npm
- An empty test repository where the agent can operate

## Steps

### 1. Install, authenticate, and initialize

```bash
npm install -g @github/copilot
copilot                 # start, authenticate with GitHub, confirm workspace trust
/init                   # generate Copilot project instructions
```

Documentation:

https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/overview

### 2. Inspect the current setup, then add a second MCP server

The GitHub MCP server is already configured by default (issues, branches, pull requests). The actual exercise is adding a second server and observing the difference between remote and local MCP integrations.

```bash
/mcp            # view, add, edit, enable, or disable servers
/mcp add        # wizard: use Tab to complete fields, Ctrl+S to save
```

The configuration is stored in:

```text
~/.copilot/mcp-config.json
```

This location can be changed through the `COPILOT_HOME` environment variable.

Copy the actual configuration you obtain into:

```text
evidence/lab-01-mcp-config.json
```

> The JSON schema for MCP servers may still evolve between releases. The commands and file path above are current at the time of writing. If a field differs from what you expect, trust what `/mcp` writes to the file rather than memory or older examples.

Documentation:

https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers

### 3. Configure the tool allow list (least privilege)

In interactive sessions, tools are approved on demand. For explicit control, use CLI startup flags:

```bash
copilot --allow-tool 'shell'         # allow only this tool
copilot --deny-tool 'write'          # explicitly deny this tool
copilot --allow-all                  # alias: --yolo, grants everything (anti-pattern; use only for comparison)
```

Grant only the minimum tools required for the task.

Record which tools remain blocked and at which layer the restriction is applied:

- CLI startup flags
- Per-tool MCP configuration

Documentation:

https://docs.github.com/en/copilot/how-tos/copilot-cli/allowing-tools

### 4. Test enforced vs. advisory controls

Run two one-shot tasks: one using an allowed tool and one using a denied tool.

```bash
copilot -p "list open issues" --allow-tool 'shell'
copilot -p "write a README file" --deny-tool 'write'
```

Observe the refusal in the second case.

Then ask yourself:

- Is the block enforced by the platform?
- Or did the agent merely decide not to attempt the action?

Rephrase the task and try to bypass the restriction. If the agent still cannot perform the action, you are observing genuine enforcement.

### 5. Capture evidence in `evidence/`

- `lab-01-mcp-config.json` — actual MCP configuration (from `~/.copilot/mcp-config.json`)
- `lab-01-session.log` — session output showing loaded and invoked tools
- `lab-01-notes.md` — what was blocked, at which layer, and why

## Self-Assessment Questions (Exam Style)

1. An agent fails a multi-step task involving a REST API. The logs show that the required tool was never invoked. What is the most likely cause?

2. What is the difference between a remote MCP server (such as GitHub MCP) and an MCP allow list? What does each one control?

3. Why does least-privilege scoping reduce risk in cases of prompt injection, not just software bugs?

4. Both your Cody prompt tool list and an MCP allow list can block tools. Which one still holds if the agent is compromised or manipulated, and why?

## Connection to the Squad (Informational Only)

Your squad already uses MCP indirectly through Claude and Codex connectors. In this lab, you view MCP as a governed surface: registry management, allow lists, and tool-level permissions.

If this approach proves valuable over time, it becomes a natural candidate for a future `copilot/` tree within the squad.

For now, it stays here.

## Definition of Done

You have:

- all three files saved under `evidence/`
- the ability to answer all four self-assessment questions without reopening the documentation
