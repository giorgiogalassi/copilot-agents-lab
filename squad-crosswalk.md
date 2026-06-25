# Squad to esame: crosswalk

Per ogni concetto trasferibile, il link diretto al punto nel repo
`giorgiogalassi/agent-squad` (branch `main`).

I `.toml` non hanno heading, quindi il link apre il file: cerca il campo
indicato (`sandbox_mode`, `tools:`) con ctrl-F.

## D1 Architecture e SDLC

| Concetto | Link |
|----------|------|
| Piano strutturato prima dell'azione | [cody.md #3-plan](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/cody.md#3-plan) |
| Stesso, versione Codex (step "4. Plan") | [cody.toml](https://github.com/giorgiogalassi/agent-squad/blob/main/codex/agents/cody.toml) |
| Gate umani (PRD review, issue review) | [JOURNAL #iteration-6-manual-review-gates](https://github.com/giorgiogalassi/agent-squad/blob/main/JOURNAL.md#iteration-6-manual-review-gates) |
| Skill vs agent (anti-pattern) | [JOURNAL #iteration-2-adding-scout-and-forge](https://github.com/giorgiogalassi/agent-squad/blob/main/JOURNAL.md#iteration-2-adding-scout-and-forge) |
| Input/output/success criteria | [forge/SKILL.md #required-slots](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/forge/SKILL.md#required-slots) |

## D2 Tool use e MCP

| Concetto | Link |
|----------|------|
| Permessi tool per agente, read-only (campo `tools:`) | [reven.md](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/reven.md) |
| Permessi tool per agente, write (campo `tools:`) | [cody.md](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/cody.md) |
| Sandbox read-only Codex (campo `sandbox_mode`) | [reven.toml](https://github.com/giorgiogalassi/agent-squad/blob/main/codex/agents/reven.toml) |
| Sandbox workspace-write Codex (campo `sandbox_mode`) | [cody.toml](https://github.com/giorgiogalassi/agent-squad/blob/main/codex/agents/cody.toml) |
| Retry ed escalation | [ralph/SKILL.md #2c-escalation](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/ralph/SKILL.md#2c-escalation) |
| Max 3 retry, regola | [ralph/SKILL.md #rules](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/ralph/SKILL.md#rules) |

Nota: lo squad si ferma al consumo di tool. Configurare MCP server, registry e
allow list e il lavoro nuovo del Lab 01, non e nel repo.

## D3 Memory, state, execution

| Concetto | Link |
|----------|------|
| context = RAM, filesystem = disk | [SQUAD #principles](https://github.com/giorgiogalassi/agent-squad/blob/main/SQUAD.md#principles) |
| Loading discipline (scoping memoria) | [SQUAD #loading-discipline](https://github.com/giorgiogalassi/agent-squad/blob/main/SQUAD.md#loading-discipline-all-companions-follow-this) |
| Loading discipline, Lore | [lore.md #loading-discipline](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/lore.md#loading-discipline) |
| Cap 100 righe (pruning) | [lore.md #lore-prefer](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/lore.md#lore-prefer-decision) |
| Snapshot riscritto intero (reset) | [seed/SKILL.md #phase-4](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/seed/SKILL.md#phase-4-write-scout-cachemd) |
| Drift: timestamp mismatch + staleness (step 8 e 9) | [lore.md #lore-start](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/lore.md#lore-start) |
| Instance namespacing cross-tool | [lore.md #rules](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/lore.md#rules) |

## D4 Evaluation e tuning

| Concetto | Link |
|----------|------|
| Verdetto strutturato vs acceptance criteria | [reven.md #output](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/reven.md#output) |
| Criteri di review | [reven.md #review-criteria](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/reven.md#review-criteria) |
| Root cause su diff merged (caso GG-18) | [JOURNAL #iteration-14](https://github.com/giorgiogalassi/agent-squad/blob/main/JOURNAL.md#iteration-14-global-installation-and-vault-first-state) |

## D5 Multi-agent coordination

| Concetto | Link |
|----------|------|
| Pipeline come orchestration pattern | [README #mvp-flow](https://github.com/giorgiogalassi/agent-squad/blob/main/README.md#mvp-flow) |
| Roster agenti | [SQUAD #agent-roster](https://github.com/giorgiogalassi/agent-squad/blob/main/SQUAD.md#agent-roster) |
| Ralph orchestratore, esecuzione | [ralph/SKILL.md #phase-2](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/ralph/SKILL.md#phase-2-execute-in-order) |
| Grafo Blocked-by + cycle detection | [ralph/SKILL.md #phase-1](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/ralph/SKILL.md#phase-1-build-the-execution-order) |
| Formato dipendenza (definizione) | [chisel/SKILL.md #dependency-format](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/skills/chisel/SKILL.md#dependency-format) |
| Stalled / draft PR a maxTurns | [cody.md #rules](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/cody.md#rules) |
| Lifecycle: quando aggiungere agenti | [JOURNAL #6-when-to-add](https://github.com/giorgiogalassi/agent-squad/blob/main/JOURNAL.md#6-when-to-add-the-next-agent-layer) |
| Lifecycle: cosa e fuori MVP | [JOURNAL #what-is-out-of-the-mvp](https://github.com/giorgiogalassi/agent-squad/blob/main/JOURNAL.md#what-is-out-of-the-mvp) |

## D6 Guardrails e accountability

| Concetto | Link |
|----------|------|
| Regole Cody (no secret, no test off) | [cody.md #rules](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/cody.md#rules) |
| Severity model (autonomia per rischio) | [SQUAD #severity-model](https://github.com/giorgiogalassi/agent-squad/blob/main/SQUAD.md#severity-model) |
| Severity model, origine | [JOURNAL #iteration-4-severity-model](https://github.com/giorgiogalassi/agent-squad/blob/main/JOURNAL.md#iteration-4-severity-model) |
| Stripping tag private (data handling) | [lore.md #rules](https://github.com/giorgiogalassi/agent-squad/blob/main/claude/agents/lore.md#rules) |
| Gap self-merge (accountability) | [JOURNAL #5.9](https://github.com/giorgiogalassi/agent-squad/blob/main/JOURNAL.md#59-pr-assignment-and-branch-protection) |

Quest'ultima riga e la piu importante: hai gia identificato da solo il buco di
accountability (Cody apre e Reven rivede sullo stesso account `gh`, nessun
blocco al self-merge) prima di studiare. L'esame testa esattamente la sua
soluzione enforced: branch protection e required reviewer.