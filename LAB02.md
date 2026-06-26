# Lab 02 — CI/Actions, Safe Execution, and MCP Registries (Domain 2)

Objective: close Domain 2 by covering the three areas Lab 01 did not touch: running a Copilot agent task via GitHub Actions and inspecting the resulting traceability artifacts; safe execution patterns (retry, rollback, escalation); and MCP registry and plugin management.

## What You Already Know vs. What Is New

**Transferable from the squad:**
- Retry and escalation: Ralph caps retries at 3 and escalates to the operator on repeated failure. The same principle applies here, enforced at the platform level rather than in a prompt. See [ralph/SKILL.md #2c-escalation](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/ralph/SKILL.md#2c-escalation) and [#rules](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/ralph/SKILL.md#rules).
- Audit trail concept: in the squad, Lore writes session logs to the vault. Here, GitHub Actions run logs, PR timelines, and check results are the equivalent, and they are inspectable by anyone with repo access.

**What is new:**
- The agent runs unattended in CI. There is no operator present to approve an override prompt. Every allow/deny decision is final.
- Traceability is structural, not optional: the Actions run log, the PR opened by the agent, and the required checks are all permanent, linkable, and inspectable artifacts. This is what the exam means by "accountability."
- MCP registries: how to discover, add, and scope MCP servers beyond the default GitHub MCP, including version pinning and removal.

## Exam Skills Covered (Study Guide D2)

- Invoke a Copilot agent task in GitHub Actions
- Implement traceability and accountability for agent actions
- Apply safe execution patterns: retry, rollback, escalation
- Manage MCP registries and plugins

## Prerequisites

- A GitHub Copilot-enabled account with Copilot coding agent access
- Sandbox repository: https://github.com/giorgiogalassi/copilot-agents-lab-sandbox (created ad hoc for this lab -- Actions enabled, clean slate for agent tasks)
- This repo (`copilot-agents-lab`) is where evidence files are saved; the sandbox is where the agent operates

---

## Part A — Copilot Coding Agent in GitHub Actions

The Copilot coding agent can be triggered by assigning a GitHub issue to `@github-models` (or the equivalent Copilot agent handle). It runs in a sandboxed Actions environment, opens a branch, commits its work, and opens a PR. The PR, the run log, and the checks are the traceability artifacts.

### Steps

#### 1. Create a scoped issue

Create an issue in the sandbox repository (https://github.com/giorgiogalassi/copilot-agents-lab-sandbox) with a clear, bounded task. Bounded means: one outcome, verifiable, no ambiguity about when it is done. Example:

```
Title: Add a README.md that describes this sandbox repository
Body:
- File: README.md
- Content: one paragraph explaining this is a sandbox for GH-600 Lab 02, with a link back to copilot-agents-lab
- No other files should be modified
```

Save the issue URL for `evidence/lab-02-issue.md`.

#### 2. Assign the issue to the Copilot coding agent

Assign the issue to `Copilot` (the coding agent) from the Assignees panel in the sandbox repo. This triggers the agent. It will:

1. Create a branch from the default branch
2. Implement the task
3. Open a PR referencing the issue

Do not intervene. Let it run to completion or failure.

#### 3. Inspect the traceability artifacts

Once the PR is open, collect the following and record them in `evidence/lab-02-traceability.md`:

- **Issue URL** -- the triggering artifact
- **PR URL** -- what the agent produced
- **Actions run URL** -- the full execution log (find it in the PR checks or the Actions tab)
- **Branch name** -- created by the agent, shows naming convention
- **Checks status** -- did required checks pass or fail?

For each artifact, note: who created it, when, and whether it is inspectable by a reviewer who was not present during the run.

#### 4. Review the PR as a human reviewer

Read the diff. Ask yourself:

- Did the agent stay within the scope of the issue?
- Are the changes attributable (commit author, run log)?
- If the agent made an error, where in the artifact trail is the error visible?

Do not merge. Record your observations in `evidence/lab-02-traceability.md` under a "Review notes" section.

---

## Part B — Safe Execution: Retry, Rollback, Escalation

This part does not require a separate live run. It is a structured observation exercise based on the Actions run from Part A plus the documentation.

### Concepts to map

| Pattern | How it appears in GitHub Actions / Copilot | Squad equivalent |
|---|---|---|
| Retry | Actions step `retry-on-error`, or agent re-attempt within the same run | Ralph: max 3 retries before escalation |
| Rollback | Branch is never merged; closing the PR leaves main untouched | Cody: draft PR on maxTurns, operator decides |
| Escalation | Agent cannot proceed: PR stays open, checks fail, operator is notified via PR comment or failed check | Ralph: escalation note in progress.txt |
| Traceability | Run log + PR + checks = inspectable audit trail | Lore: session log in vault |

### Steps

#### 1. Identify retry behavior in the run log

Open the Actions run log from Part A. Find any step that was retried or any tool call that failed and was reattempted. If the run succeeded cleanly, note the absence of retries as expected behavior for a well-scoped task.

Record in `evidence/lab-02-safe-execution.md`: what retry behavior (if any) was observed and at which layer it occurred (Actions infrastructure vs. agent logic).

#### 2. Confirm rollback posture

Note that the agent's branch and PR exist but main is unchanged. This is the default rollback posture: the agent cannot merge its own work. Confirm this is still true by checking the branch protection rules for the sandbox repo (https://github.com/giorgiogalassi/copilot-agents-lab-sandbox > Settings > Branches).

Record: is a required reviewer configured? Is auto-merge disabled? If neither is set, note this as a gap (it is the same self-merge accountability gap identified in the squad journal).

#### 3. Identify the escalation path

If the agent's PR has failing checks, the escalation path is: failed check visible in PR, operator receives notification, operator decides to close or fix. If checks pass but the work is wrong, the escalation path is: human reviewer catches it during PR review.

Record: at what point in the artifact trail does an error become visible to a human reviewer who was not watching the run live?

---

## Part C — MCP Registries and Plugin Management

### Concepts

An MCP registry is a catalog of available MCP servers. The default Copilot CLI setup includes the GitHub MCP server. Additional servers can be added from a registry (remote, curated list) or configured manually (local, arbitrary endpoint).

Key operations: discover, add, pin a version, disable, remove.

### Steps

#### 1. List currently configured MCP servers

```bash
copilot
/mcp
```

Note which servers are active and which tools each one exposes. Compare to the `~/.copilot/mcp-config.json` captured in Lab 01.

#### 2. Add a server from the registry

Use `/mcp add` to add a second server. The Context7 server used in Lab 01 is a valid choice if not already present as a named entry. During the wizard:

- Note whether the source is a registry entry (URL from a curated list) or a manual endpoint
- Note whether a version is pinned or left at `latest`

Save the updated `~/.copilot/mcp-config.json` to `evidence/lab-02-mcp-config.json`.

#### 3. Scope tools from the new server

`--allow-tool` takes a tool name, not a server name. There is no per-server scoping at the CLI flag level -- the flag applies globally across all connected servers. If you want to restrict tools to a specific server, configure the `tools` field in `mcp-config.json` at the server entry level (e.g. `"tools": ["resolve-library-id"]`).

To observe the effect of `--allow-tool`, the task you give the agent must actually require the tool you are restricting. If the task does not trigger that tool, you will see no difference.

**Important: `--allow-tool` is a whitelist.** Specifying one tool blocks all others. In the examples below, allowing only `resolve-library-id` means `shell`, `fetch`, `write`, and every other tool are blocked -- the agent has no fallback.

**Observed behavior by mode:**

- **Interactive (no `-p`)**: when a blocked tool is attempted, Copilot surfaces an override prompt asking the operator for permission (same mechanism as `--deny-tool` in Lab 01). The operator decides. Example observed: `copilot --allow-tool 'resolve-library-id'` followed by "what is the latest version of react?" produced a "Fetch web content -- Do you want to allow this access?" prompt for `https://registry.npmjs.org/react/latest`.

- **Non-interactive (`-p`)**: no operator is present. The blocked tool fails silently and the agent falls back to training data or fails the task entirely. Example observed: `copilot -p "what is the latest version of react" --allow-tool 'resolve-library-id'` -- agent attempted `shell` and `fetch`, both returned "Permission denied and could not request permission from user", then responded from training data.

```bash
# interactive -- override prompt appears for blocked tools
copilot --allow-tool 'resolve-library-id'
# then ask: what is the latest version of react?

# non-interactive -- hard failure, no override path
copilot -p "what is the latest version of react" --allow-tool 'resolve-library-id'

# to force resolve-library-id specifically, use a docs task:
copilot -p "show me the react hooks documentation" --allow-tool 'resolve-library-id'
```

The `-p` mode behavior is what applies in CI/Actions runs: no human present, blocked tools fail, no escape hatch.

#### 4. Remove or disable the server

Use `/mcp` to disable or remove the server. Confirm that the tool is no longer available in a new session. Save the final config state to `evidence/lab-02-mcp-config.json` (overwrite with the post-removal state, or save both versions with suffixes `-before` and `-after`).

Record in `evidence/lab-02-mcp-registries.md`: the difference between a registry-sourced server and a manually configured one, and why version pinning matters in a CI context.

---

## Self-Assessment Questions (Exam Style)

### 1. A Copilot coding agent is assigned an issue and opens a PR, but the PR contains changes to files outside the scope of the issue. A security reviewer was not watching the run. How would they detect the out-of-scope changes?

**Your answer:**

_A: As we saw in the log, Copilot invokes claude-sonnet-4.5 as a peer reviewer (LLM-as-judge). The scope of the development must be provided at issue level, describing which domains may be involved in the fix. If this is not possible (e.g. an analysis issue), another agent must provide that perimeter. Once that information is in place, identifying out-of-scope changes becomes straightforward._

**Model answer:**

The PR diff is the primary detection surface: every file the agent touched is visible in the diff, regardless of whether the reviewer was present during the run. The Actions run log shows which tools were invoked and in what order, making it possible to trace how the out-of-scope file was reached. The issue the PR references provides the intended scope, so the reviewer can compare intended vs. actual changes directly. This is the structural traceability guarantee of the GitHub-as-control-plane model: the audit trail exists independently of whether a human was watching. The mitigation is CODEOWNERS or required reviewers on sensitive paths, which turns detection into prevention: the PR cannot be merged without approval from the owner of the affected file.

**Notes:** Your answer focuses on prevention (scope the issue well, use an agent to define the perimeter) and on the LLM-as-judge mechanism observed in the log. Both are valid points. What is missing is the primary detection mechanism the exam is looking for: the PR diff and the Actions run log are the structural artifacts that make out-of-scope changes visible to a reviewer who was not present. The LLM peer reviewer is an automated check, not a human reviewer -- it may or may not catch scope violations. The exam answer should start from the artifact trail (diff, log, issue) before moving to mitigation.

---

### 2. An agent task fails midway through a multi-step workflow in GitHub Actions. The run log shows the agent retried the failing step twice before the run was marked as failed. What are the two distinct retry layers available, and which one applies here?

**Your answer:**

_A: Retry behavior and escalation are two available patterns. In this case we are experiencing a retry mechanism. The layers available are agent level and platform level; this scenario is at agent level._

**Model answer:**

Two layers: (1) Actions infrastructure retry, configured at the step or job level (`retry-on-error`, or a third-party retry action), which reruns the step regardless of what failed. (2) Agent-level retry, where the agent's own logic decides to attempt an alternative approach or re-invoke a tool after an error response. The scenario describes the agent retrying twice, which points to agent-level retry logic (the agent decided to try again), not infrastructure retry (which would show as re-run steps in the Actions log with no agent awareness). The distinction matters for debugging: infrastructure retry is visible as duplicate step entries in the log; agent retry is visible as repeated tool calls within a single step's output.

**Notes:** Correct identification of the layer (agent level). The answer is thin on the distinction between the two layers and on how to tell them apart in the log. For the exam, add: infrastructure retry shows as duplicate step entries in the Actions log; agent retry shows as repeated tool calls within a single step's output. That diagnostic detail is what the scenario is testing.

---

### 3. You are configuring a Copilot coding agent for a CI pipeline that runs on every PR. The pipeline must be reproducible across runs six months apart. What is the risk of leaving an MCP server configured at `latest`, and how do you mitigate it?

**Your answer:**

_A: Assuming `latest` refers to the version, the problem is that some APIs may differ from the original, making the agent and pipeline unreliable. I would pin a fixed version at the time of deployment, then run a scheduled update pipeline against all dependencies after a defined interval._

**Model answer:**

A server at `latest` means the set of tools it exposes, their signatures, and their behavior can change between runs without any change to your configuration. A pipeline that worked today may fail or behave differently in six months if the server updated a tool name, removed a tool, or changed a parameter. The mitigation is version pinning: specify an exact version in `mcp-config.json` so the tool surface is stable. The exam maps this to the same reproducibility principle that applies to Actions (`actions/checkout@v4`, not `actions/checkout@latest`) and to package management (`npm ci` with a lockfile, not `npm install`). For production pipelines, `latest` is an anti-pattern; it is acceptable only for interactive exploration.

**Notes:** Correct on the risk and the fix. The scheduled update pipeline is a good addition not in the model answer. What is missing is the specificity: the risk is not just "APIs may differ" but that the tool surface itself can change -- tool names, signatures, or tools being removed entirely -- which is more disruptive than a typical API version bump. Also worth noting the parallel to `actions/checkout@v4` vs `@latest`, which is the exam's likely framing.

---

### 4. A Copilot agent opens a PR but the repository has no required reviewers and auto-merge is enabled. What accountability gap does this create, and what is the platform-level fix?

**Your answer:**

_A: No review of the code before it is shipped to production is the accountability gap. The platform-level fix is adding a ruleset that prevents merging unless an approval is set. GitHub also provides an additional check: a user who collaborated with the agent cannot satisfy the review requirement -- they can approve, but their approval will not count if the required number of approvals is 1._

**Model answer:**

The agent can, in effect, merge its own work: it opens the PR, the checks pass (or there are none), and auto-merge closes it without any human ever seeing the diff. This is the self-merge accountability gap: the agent is both author and merger, with no independent verification. The platform-level fix is branch protection on the target branch: require at least one human reviewer (not the PR author), disable auto-merge or require a manual approval step, and optionally add CODEOWNERS for sensitive paths. This is a layer 3 control (org/repo policy, not overridable in-session), which is the only layer that holds against an agent whose instructions have been manipulated. Advisory controls ("the agent knows not to merge its own PR") fail the moment the agent is compromised.

**Notes:** Strong answer, and the collaborator check observation is correct and goes beyond the model answer -- it is something you discovered hands-on in this lab. The one addition for the exam: frame the gap explicitly as "self-merge" (agent is both author and merger) and note that advisory controls fail under compromise, which is why layer 3 is the only reliable fix.

---

## Evidence Files

Collect the following before declaring this lab done:

| File | Content |
|---|---|
| `evidence/lab-02-issue.md` | Issue URL and title |
| `evidence/lab-02-traceability.md` | PR URL, Actions run URL, branch name, checks status, review notes |
| `evidence/lab-02-safe-execution.md` | Retry behavior observed, rollback posture confirmed, escalation path identified |
| `evidence/lab-02-mcp-config.json` | MCP config after adding and removing the second server |
| `evidence/lab-02-mcp-registries.md` | Registry vs. manual server, version pinning rationale |

## Definition of Done

You have:

- all five evidence files saved under `evidence/`
- the Copilot coding agent has run at least once in GitHub Actions and produced an inspectable PR
- the ability to answer all four self-assessment questions without reopening the documentation
- D2 fully covered: Lab 01 (MCP server, allow list, enforced vs. advisory) + Lab 02 (CI/Actions, traceability, safe execution, registries)
