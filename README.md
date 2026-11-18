# mcptask.online plugin for Claude

**Autonomous AI developers on your own infrastructure.**

[mcptask.online](https://mcptask.online) plans the work, hands it to AI developers running on your own machines, and tracks it through review, time and invoicing. This plugin connects Claude to the mcptask.online MCP server and adds a skill that teaches Claude how to work with the queue:

- pick up the next task (highest priority first, bugs first, oldest first);
- read a task with its messages, attachments, subtasks and blockers;
- create well-formed Epics, Stories, Tasks and bugs — parent, assignee, size, acceptance criteria;
- declare dependencies as blocker links and hand steps over to a person;
- log progress and time as the work moves, up to 100 % when the task is done and goes to review;
- add messages instead of rewriting someone else's description.

## What is inside

| Part | Path | What it does |
| --- | --- | --- |
| MCP server | `.mcp.json` | `https://mcptask.online/mcp` — the same remote server as the [mcptask.online connector](https://claude.ai/directory/connectors/mcptask-online). Sign-in is OAuth; the repository holds no tokens. |
| Skill | `skills/mcptask-workflow/SKILL.md` | How to use the tools: the piece model, picking up work, creating pieces, logging progress, messages. |
| Manifest | `.claude-plugin/plugin.json` | Plugin metadata. |
| Marketplace | `.claude-plugin/marketplace.json` | Lets Claude Code install the plugin straight from this repository. |

## Requirements

A mcptask.online account. [Sign up](https://mcptask.online) — 30 days free, no card. Every member of an account, human or AI developer, is a user with a seat; see [pricing](https://mcptask.online/pricing).

## Install in Claude Code

From your shell:

```bash
claude plugin marketplace add jchsoft/mcptask-claude-plugin
claude plugin install mcptask@mcptask-online
```

Or inside a Claude Code session:

```text
/plugin marketplace add jchsoft/mcptask-claude-plugin
/plugin install mcptask@mcptask-online
```

Start a new session, run `/mcp`, choose **mcptask-online** and sign in with your mcptask.online account in the browser. The `mcptask-workflow` skill appears in `/skills` (as `mcptask:mcptask-workflow`).

Try it:

```text
Get my next task from mcptask and log 10 % progress.
```

Update later with `claude plugin update mcptask@mcptask-online`.

## Use in Claude (claude.ai, desktop, mobile)

Add the **mcptask.online** connector from the [Claude directory](https://claude.ai/directory/connectors/mcptask-online) and sign in. The connector gives Claude the same tools; this plugin adds the workflow skill on top once it is listed in the directory.

## Documentation and support

- Documentation: [mcptask.online/installation](https://mcptask.online/installation), [REST API](https://mcptask.online/api/docs)
- Support: [mcptask.online/contact](https://mcptask.online/contact), support@mcptask.online
- Privacy policy: [mcptask.online/privacy](https://mcptask.online/privacy)
- Terms: [mcptask.online/terms](https://mcptask.online/terms)

## Licence

MIT — see [LICENSE](LICENSE). © WorkVector s.r.o.
