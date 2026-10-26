# TTPOS skills: Pivot Master suite

Seven Agent Skills (open SKILL.md format, agentskills.io) that teach an AI assistant the Pivot Master workflows (Franklin Ochoa's CPR method and Trend X) on top of the TTPOS MCP server.

| Skill | What it does |
| --- | --- |
| `pivot-screener` | Finds candidates by CPR width, regime and price position |
| `pivot-flightplan` | Flight plan of one symbol: CPR, regime, position, router, levels, setups |
| `pivot-calendar` | Upcoming announcements and currency strength |
| `pivot-risk` | Accounts, daily risk gauge and decision journal (read only) |
| `pivot-entry` | Entry Timer: is price on a zone with a reversal signal, and a one-shot alert |
| `pivot-propose` | Proposes a decision to the human in TTPOS after your explicit agreement (Avancé plan); never approves or executes |
| `pivot-desk` | Morning briefing adapted to your profile, chaining the others |

`pivot-timemachine` will arrive in a later release.

## Prerequisite

The TTPOS MCP server must be connected to your AI client: in TTPOS, open « Connecter votre IA » in the top bar and follow the steps for your client. The skills only describe the method and the order of the calls; all data comes from the MCP tools. Some tools need the Abonné or Avancé plan.

## Install

A skill is a folder containing a `SKILL.md`. Copy the folders you want into your client's skills directory:

- Claude Code: `~/.claude/skills/` (all projects) or `.claude/skills/` (one project)
- Cursor, Codex and other Agent Skills clients: the skills directory documented by the client (often `~/.agents/skills/` or `.agents/skills/`)
- Clients without skills support: paste the content of a `SKILL.md` into the instructions of a project or custom assistant

Restart the client if it does not pick the skills up.

## Behavior

- Answers come in the user's language (French by default, formal "vous"); the `summary` of each tool is reused word for word in French.
- No invented numbers, no orders: approving and executing happen in TTPOS. This is not investment advice.

## License

MIT, see `LICENSE`.
