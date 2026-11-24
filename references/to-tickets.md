---
name: to-tickets
description: "Slice a confirmed spec into local tracer-bullet tickets with explicit blocking edges for fresh implementer agents."
disable-model-invocation: true
---

# To Tickets

Turn the confirmed spec into a directed graph of tickets. Each ticket is small enough for one fresh implementer and delivers behavior that can be demonstrated or verified on its own. The ticket slicer writes local files; a later setup agent creates GitHub issues from them.

## Slice vertically

- Each ticket cuts a narrow path through the layers it needs, including tests. Avoid tickets that only build one horizontal layer of a feature.
- Prefer existing public testing seams and the project's glossary vocabulary. Respect relevant ADRs.
- Put prerequisite cleanup first when it genuinely blocks a behavior change. Give every ticket only the blockers needed to start it safely.
- A wide mechanical refactor may need an expand–migrate–contract sequence rather than a forced vertical slice: add the new form alongside the old, migrate callers in bounded batches, then remove the old form after every caller has moved.

## Check the graph

Return a numbered table with each ticket's title, blockers, and delivered behavior. Ensure every user story is covered, every blocker names a real ticket, and the graph has no cycle. Flag unresolved scope choices for the coordinator; the confirmed design does not require a routine second approval gate.

## Write ticket files

Write one file per ticket under `.scratch/<feature-slug>/tickets/<NN>-<slug>.md`, numbered in dependency order. The setup agent later creates GitHub issues without changing the graph.

```markdown
# <NN>: <Ticket title>

**What to build:** <end-to-end behavior from the user's perspective>

**Blocked by:** <ticket numbers and titles, or None>

**Status:** ready-for-agent

- [ ] <observable acceptance criterion>
- [ ] <observable acceptance criterion>
```

The initial ready set is every ticket with no blockers. Later tickets become ready only when their blockers are integrated into the remote feature branch. Avoid file paths and code snippets in ticket descriptions unless a decision is clearer with a small, stable example.
