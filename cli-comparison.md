# Claude Code vs Copilot CLI: confronto di riferimento

I primitivi di estensibilita mappano quasi 1:1. Cio che cambia, ed e la parte
che l'esame testa, e il control plane: Copilot mette GitHub al centro (issue,
branch, PR come tool nativi) dove Claude Code mette il filesystem locale.

Nota per lo squad: le tue skill (`SKILL.md`) sono sullo stesso standard Agent
Skills che Copilot CLI usa, quindi piu vicine alla portabilita di quanto
sembri. Gli agenti (cody/reven) vanno tradotti di formato, come gia fai tra
`claude/` e `codex/`.

## Mappa concetto per concetto

| Concetto | Claude Code | Copilot CLI |
|----------|-------------|-------------|
| Install | `npm i -g @anthropic-ai/claude-code` | `npm i -g @github/copilot`, poi `/init` |
| Istruzioni progetto | `CLAUDE.md` (sempre caricato) | `AGENTS.md`, `copilot-instructions.md`, `.instructions.md` con `applyTo` |
| Skill | `SKILL.md`, on-demand | `SKILL.md` (stesso standard), `.github/skills/` o `~/.copilot/skills/` |
| Agent custom | `.claude/agents/*.md`, YAML frontmatter | `.github/agents/*.agent.md` o `~/.copilot/agents/`, `infer: true` per auto-delega |
| Subagent / paralleli | Task tool, Agent Teams | built-in (explore, task, code-review, research) + `/fleet` |
| Hook | PreToolUse/PostToolUse/Stop, exit 2 blocca | sessionStart/preToolUse/postToolUse/agentStop in `.github/hooks/*.json` |
| MCP | `.mcp.json`, `claude mcp add` | GitHub MCP gia incluso + custom in `~/.copilot/mcp-config.json`, `/mcp add` |
| Permessi tool | `tools`/`disallowedTools` (allow/deny list) | `--allow-tool`/`--deny-tool`, `--allow-all`/`--yolo` |
| Plugin | bundle di skill/agent/hook/MCP | `plugin.json`, `/plugin install owner/repo`, marketplace |
| Headless | `claude -p` one-shot | `copilot -p`, `--output-format=json`, `--silent` |
| Autonomia | permission mode | Shift+Tab cicla standard / plan / autopilot |

Curiosita che conferma la vicinanza: Copilot CLI sa leggere hook in formato
Claude (preToolUse, permissionRequest) per matcher come Bash, Read, *.

## Le differenze che contano per l'esame

1. GitHub e il control plane (D1). Il GitHub MCP nativo ti fa lavorare su
   issue, branch e PR, non solo modificare file: crei branch, implementi e
   apri una PR con un comando. Nello squad la accountability vive nel vault
   (status.md, decisions.md); qui vive nei primitivi GitHub.

2. Esecuzione in CI come prima classe (D1, D2). Puoi avviare un cloud agent su
   GitHub e portarlo in locale; oppure assegnare un'issue a Copilot, che crea
   un dev environment in GitHub Actions, scrive codice e apre la PR. Lo squad
   non fa questo.

3. Livelli di autonomia espliciti nell'interfaccia (D6). Shift+Tab tra plan e
   autopilot. Il tuo severity model assegna autonomia per rischio a livello
   concettuale; qui e un toggle operativo.

4. Memoria come prodotto gestito (D3). Repository memory (convenzioni del
   codebase tra sessioni) e cross-session memory (PR e lavoro passato). E il
   tuo Lore e il second-brain, ma nativo.

## La lezione meta

Squad: l'LLM e il control plane, il giudizio sta negli agenti, le regole nei
prompt (advisory). GH-600: GitHub e il control plane, l'enforcement sta nei
controlli della piattaforma. Quasi tutti i concetti reggono; quello da
spostare e il livello di enforcement, da "l'agente sa che non deve" a "la
piattaforma non glielo lascia fare".

## Fonti

- Copilot CLI GA: https://github.blog/changelog/2026-02-25-github-copilot-cli-is-now-generally-available/
- Using Copilot CLI: https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/overview
- Feature page: https://github.com/features/copilot/cli
- Claude Code subagents: https://code.claude.com/docs/en/sub-agents