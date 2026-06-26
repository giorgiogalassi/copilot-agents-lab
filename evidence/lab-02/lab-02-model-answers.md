# Lab 02 — Model Answers (restore after self-assessment)

### 1.

The PR diff is the primary detection surface: every file the agent touched is visible in the diff, regardless of whether the reviewer was present during the run. The Actions run log shows which tools were invoked and in what order, making it possible to trace how the out-of-scope file was reached. The issue the PR references provides the intended scope, so the reviewer can compare intended vs. actual changes directly. This is the structural traceability guarantee of the GitHub-as-control-plane model: the audit trail exists independently of whether a human was watching. The mitigation is CODEOWNERS or required reviewers on sensitive paths, which turns detection into prevention: the PR cannot be merged without approval from the owner of the affected file.

### 2.

Two layers: (1) Actions infrastructure retry, configured at the step or job level (`retry-on-error`, or a third-party retry action), which reruns the step regardless of what failed. (2) Agent-level retry, where the agent's own logic decides to attempt an alternative approach or re-invoke a tool after an error response. The scenario describes the agent retrying twice, which points to agent-level retry logic (the agent decided to try again), not infrastructure retry (which would show as re-run steps in the Actions log with no agent awareness). The distinction matters for debugging: infrastructure retry is visible as duplicate step entries in the log; agent retry is visible as repeated tool calls within a single step's output.

### 3.

A server at `latest` means the set of tools it exposes, their signatures, and their behavior can change between runs without any change to your configuration. A pipeline that worked today may fail or behave differently in six months if the server updated a tool name, removed a tool, or changed a parameter. The mitigation is version pinning: specify an exact version in `mcp-config.json` so the tool surface is stable. The exam maps this to the same reproducibility principle that applies to Actions (`actions/checkout@v4`, not `actions/checkout@latest`) and to package management (`npm ci` with a lockfile, not `npm install`). For production pipelines, `latest` is an anti-pattern; it is acceptable only for interactive exploration.

### 4.

The agent can, in effect, merge its own work: it opens the PR, the checks pass (or there are none), and auto-merge closes it without any human ever seeing the diff. This is the self-merge accountability gap: the agent is both author and merger, with no independent verification. The platform-level fix is branch protection on the target branch: require at least one human reviewer (not the PR author), disable auto-merge or require a manual approval step, and optionally add CODEOWNERS for sensitive paths. This is a layer 3 control (org/repo policy, not overridable in-session), which is the only layer that holds against an agent whose instructions have been manipulated. Advisory controls ("the agent knows not to merge its own PR") fail the moment the agent is compromised.
