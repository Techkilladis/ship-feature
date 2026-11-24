# Ship Feature

An agent skill for taking a feature from requirements through a spec, tickets, implementation, final tests and review, and a pull request. The coordinator starts a fresh implementer for each ready ticket. Independent tickets can run in parallel; a separate agent integrates each parallel batch. A single-ticket batch is integrated by its implementer.

## Install

Requires Node.js 22.20 or later. From a terminal, run:

```sh
npx skills add Techkilladis/ship-feature
```

The installer lets you choose the skill, the coding agents, and project or global scope. Choose **Global** to use the skill across projects for the agents you select. Select every agent if you want it available in all your installed coding agents. You can instead select only Antigravity, Codex, Claude Code, or another supported agent.

For a non-interactive install to a particular agent, use `-g -a <agent-id> -y`, for example:

```sh
npx skills add Techkilladis/ship-feature -g -a antigravity -y
```

Run `npx skills list -g` to check global installations. Invoke the skill as `$ship-feature` in an agent that supports skill invocation.

See the [Skills CLI reference](https://github.com/vercel-labs/skills/blob/main/docs/commands.md) for agent IDs and installation options.

## Bundled references

The files under `references/` include material from [Matt Pocock's skills](https://github.com/mattpocock/skills). They are distributed under the MIT license in [references/LICENSE](references/LICENSE).
