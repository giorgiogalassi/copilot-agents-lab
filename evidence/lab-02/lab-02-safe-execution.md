# Lab 02 — Safe Execution Evidence

Source: Actions run log from PR #2 (session df23daeb-385c-448c-8d5e-ecbf9a46cc25).

---

## Retry behavior

No retry was observed in this run. The agent completed the task in 11 turns without any tool call being reattempted. (The log shows 12 turns total; turn 12 is a post-session cleanup phase where the runtime reconnects MCP servers and sends the PR summary. Copilot itself reports "Main session complete after 11 turns" -- 11 is the correct count for the task execution.)

One non-fatal error occurred at turn 10 during `parallel_validation`. The `code_review` subprocess attempted `git show <base-commit>:labs/README.md` to diff the new file against the base commit. The command failed with exit code 128 and stderr `fatal: path 'labs/README.md' exists on disk, but not in '<sha>'` -- expected behavior for a new file that has no base-commit counterpart. The subprocess handled the exception internally and completed successfully 8 seconds later. No retry was triggered; the tool had a fallback path.

This illustrates the diagnostic rule from Q1: "invoked but errored" (tool ran, received an error, recovered) is distinct from "never invoked" (tool was not callable). The error is visible in the log with timestamp, exit code, and stderr -- inspectable by anyone with repo access.

No Actions-level infrastructure retry (e.g. `retry-on-error`) was configured or triggered. For a well-scoped, single-file task this is expected.

---

## Rollback posture

The agent created branch `copilot/add-labs-readme-md` and opened PR #2. The `main` branch was not modified.

Branch protection on `main` is configured with: require at least 1 approving review from a reviewer with write access. The merge button on PR #2 is disabled ("Merging is blocked") without an approval. The agent cannot merge its own work -- it has no path to approve its own PR.

This is the default safe rollback posture: if the PR is closed without merging, `main` is untouched and the branch can be deleted with no side effects.

Gap confirmed and closed: without branch protection, the agent could have merged autonomously (the self-merge accountability gap identified in the squad journal at JOURNAL #5.9). Branch protection is the layer 3 fix -- enforced by the platform, not overridable in-session by the agent or the operator.

**Additional finding -- coding agent collaborator rule:** after adding an approval, merging was still blocked with the message: "Approvals from users that collaborated with the coding agent on changes will not satisfy review requirements." The approval showed as "Approved these changes with read-only permissions." GitHub automatically invalidates approvals from any user it considers to have collaborated with the coding agent on the PR (in this case: the user who created the issue and assigned it to Copilot). This is not a standard branch protection setting -- it is a platform-level control specific to the coding agent, applied automatically.

For a single-developer repository this would make the repo unmergeable without a second account. The resolution is to add an "Allow bypass" rule for repository administrators in the branch protection settings. With the bypass rule in place, the repo owner can merge even when their approval is invalidated by the collaborator check. This is a deliberate trade-off: the control prevents fully automated self-merge by the agent, while the bypass rule preserves operator agency for single-dev repos.

---

## Escalation path

The agent completed successfully, so no escalation occurred in this run. The escalation path for a failed or out-of-scope run would be:

1. Agent cannot proceed or exceeds scope: session ends, PR is left open in draft or with failing checks.
2. Failed check or missing approval: visible on the PR page to any reviewer, regardless of whether they watched the run.
3. Human reviewer inspects the PR diff and the Actions run log to determine what happened and whether to close or fix.

There is no automated notification beyond the standard GitHub PR review request. The operator is responsible for monitoring open PRs.

---

## Layer model confirmed by this run

| Layer | Mechanism | Observed |
|---|---|---|
| Advisory (prompt) | Agent instructions | Not tested in this run |
| Enforced (harness) | `--allow-tool` / `--deny-tool` flags | Tested in Lab 01 |
| Platform (repo policy) | Branch protection, required reviewer | Confirmed: merge blocked without approval |
