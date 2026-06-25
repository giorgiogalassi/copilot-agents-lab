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

> Format: each item shows your own answer (unchanged) followed by a fuller model answer for study.

### 1. An agent fails a multi-step task involving a REST API. The logs show that the required tool was never invoked. What is the most likely cause?

**Your answer:**

_*A:* Not invoked suggest that there was no error at all, at the same time the question writes that the multi-step task failed. This can be caused by a missing tool in the allow-tools for the specific MPC/SKILL/AGENT or copilit has been started with a specific flag that deny that particular tool._

**Model answer:**

"Never invoked" locates the failure in the availability/permission layer, upstream of execution: the tool was not callable, so the agent never reached it. Most likely causes: the tool is not in the allow list, or was denied by a `--deny-tool` flag; the MCP server that provides the tool is not configured or failed to connect; or an org/repo policy blocks it. Contrast this with "invoked but errored", where the tool did run and returned an error (bad arguments, auth failure, the external service down). Diagnostic rule: never-invoked points to a permission/availability problem; invoked-and-failed points to an execution/integration problem. Your answer is correct on the cause; just note that "never invoked" does not mean "no error at all", it means the failure happened before tool selection, and add "MCP server not connected" and "org policy" to the list.

### 2. What is the difference between a remote MCP server (such as GitHub MCP) and an MCP allow list? What does each one control?

**Your answer:**

_A: A Remote MPC Server (such as GitHub MPC) controls and gives to the Agent the list of the APIs that the said Agent can invoke (ie. list of issues from GitHub or branches from a project). The MCP allow list controls which are the APIs that the Agent is allowed to call — which can be a subset of the all available ones._

**Model answer:**

The MCP server defines and exposes the universe of tools available (GitHub MCP exposes issues, branches, PRs; Context7 exposes documentation lookups). The allow list governs which of those exposed tools the agent may actually call in this session or config, a permitted subset. Server = what exists and is connectable; allow list = what is permitted. They compose: a tool is usable only if it is both exposed by a connected server and permitted by the allow list. Your answer is correct and well framed; the only addition is that composition rule.

### 3. Why does least-privilege scoping reduce risk in cases of prompt injection, not just software bugs?

**Your answer:**

_A: Becuase with least-privileg scoping we can scope what our Agent can and cannot do, handling the errors by our self or let the platform — in this case GitHub — do it. We could give deny the write tool when we known that only the reading tool is necessary, lovering significantly the risk of uwnanted writing._

**Model answer:**

Prompt injection works by hijacking the agent's own instructions and reasoning, which is exactly the advisory layer ("the agent knows not to") meant to restrain it. Any control that relies on the agent behaving well fails at the moment injection succeeds. Least-privilege enforced outside the agent (allow list, tool scoping, and above all platform policy) holds regardless of what the manipulated agent is convinced to attempt, because the capability was never granted in the first place. It caps the blast radius: a fully hijacked agent still cannot write, delete, or exfiltrate if those tools are not permitted. Against ordinary bugs least-privilege also helps, by shrinking the surface for accidental damage, but against injection it is essential, because it is the one restraint the attacker cannot talk the agent past. Your answer described the scoping but not the injection-specific reason; note that you actually gave the correct reasoning in Q4, so port it back here.

### 4. Both your Cody prompt tool list and an MCP allow list can block tools. Which one still holds if the agent is compromised or manipulated, and why?

**Your answer:**

_A: If the Agent is compromised at system-prompt level means that the Agent itself cannot be trusted. MCP allow list on the other hand are handled and checked by Copilot itself — if the tools (or an API subset) is not present on inside the allow-list the tool call will fail._

**Model answer:**

The MCP allow list (tool scoping enforced by the harness/platform) holds; the prompt tool list does not. A prompt restriction lives inside the agent's instructions, the same layer a compromise subverts, so a manipulated agent can ignore it. The allow list is checked by Copilot outside the agent's reasoning, so an unpermitted call fails whatever the agent intends. Your core answer is right. Elevate it with the three layers of strength that this lab's own evidence revealed:

1. Prompt tool list = advisory. Fails under compromise.
2. CLI allow/deny flags (`--allow-tool` / `--deny-tool`) = enforced against the agent's autonomous action. In interactive mode Copilot may still offer you, the human operator, an override prompt (the behavior observed in this lab); in non-interactive `-p` or autopilot runs there is no human, so the denied tool simply fails. So: enforced against the agent, mode-dependent against the operator.
3. Org Copilot policy, repository permissions, branch protection = platform-enforced, not overridable in-session by either agent or operator.

The exam wants you to place a given control on this ladder and pick the right layer for the threat. For prompt injection in autonomous or CI runs, layer 2 or higher is what saves you. For "no one merges their own PR", only layer 3 works.

## Connection to the Squad (Informational Only)

Your squad already uses MCP indirectly through Claude and Codex connectors. In this lab, you view MCP as a governed surface: registry management, allow lists, and tool-level permissions.

If this approach proves valuable over time, it becomes a natural candidate for a future `copilot/` tree within the squad.

For now, it stays here.

## Definition of Done

You have:

- all three files saved under `evidence/`
- the ability to answer all four self-assessment questions without reopening the documentation