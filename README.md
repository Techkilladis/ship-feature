# Ship Feature

An agent skill for taking a feature from requirements through a spec, tickets, implementation, final tests and review, and a pull request. The coordinator starts a fresh implementer for each ready ticket. Independent tickets can run in parallel; a separate agent integrates each parallel batch. A single-ticket batch is integrated by its implementer.

## Install

Requires Node.js 22.20 or later. From a terminal, run:

```sh
npx skills add Techkilladis/ship-feature
```

The installer lets you choose the skill, the coding agents, and project or global scope. Choose **Global** to use the skill across projects for the agents you select. Select every agent if you want it available in all your installed coding agents. You can instead select only Antigravity, Codex, Claude Code, or another supported agent.

For an Antigravity-only global install, run this in PowerShell or another system terminal:

```sh
npx skills add Techkilladis/ship-feature -g -a antigravity
```

The installer places the skill under `~/.gemini/config/skills/ship-feature/`, where Antigravity loads global skills. In Antigravity, invoke it with `/ship-feature`. Restart the app or reload skills if it was already open. For another agent, use the interactive picker or pass its ID with `-a`.

Run `npx skills list -g` to check global installations.

See the [Skills CLI agent table](https://github.com/vercel-labs/skills/blob/main/docs/agents.md) and [Antigravity skill docs](https://antigravity.google/docs/skills) for installation paths and invocation.

## Bundled references

The files under `references/` include material from [Matt Pocock's skills](https://github.com/mattpocock/skills). They are distributed under the MIT license in [references/LICENSE](references/LICENSE).
