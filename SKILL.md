---
name: ship-feature
description: "Deliver a feature through a confirmed design, spec, dependency-aware tickets, fresh per-ticket agents, final verification, and a pull request."
disable-model-invocation: true
---

# Ship Feature

Coordinate a feature from requirements through a reviewed pull request. The main agent owns the ticket graph and every handoff. Each implementer receives exactly one ticket in a fresh context. Use separate agents for spec writing, ticket slicing, parallel-batch integration, final testing, code review, repairs, and PR creation.

Resolve every `references/` link relative to **this skill's directory**, even while working in another repository. Read only the reference needed for the current phase.

## Operating constraints

- This workflow requires a GitHub repository, authenticated `gh`, Git worktrees, and a host that can launch fresh subagents. If a required capability is missing, explain the blocked phase; keep the ticket graph with the coordinator rather than handing it to one long-running implementer.
- Inspect the target repository and any uncommitted changes before creating branches. Preserve the user's work. Detect the repository's default branch instead of assuming `main`.
- Keep the feature on `feat/<feature-slug>`. Nothing enters the default branch before the user confirms the final PR merge.
- A ticket becomes available when all of its blockers are **integrated into the remote feature branch**. GitHub issue state and a pushed ticket branch alone do not satisfy a dependency.
- Close ticket issues only after their commits reach the remote feature branch. Keep test and review results tied to the exact feature commit they checked.

## Durable state and handoffs

Keep the run's working artifacts under `.scratch/<feature-slug>/` in the target repository. Keep this scratch directory out of feature commits. Create or update `.scratch/<feature-slug>/run-state.md` before delegation and after every receipt. Record the repository, default and feature branches, current remote feature commit, parent issue, ticket files, issue numbers, blocking edges, ticket status, and branch/commit receipts. On resume, reconcile the state with the remote feature branch and GitHub issues before dispatching more work.

Subagent prompts contain the role, one objective, the relevant reference path, file pointers, applicable issue number and base commit, worktree location, and expected output. Use fresh contexts and compact receipts:

- **Implementer:** ticket/issue, status, ticket branch, commit, and any blocked condition.
- **Integrator or singleton implementer:** integrated ticket IDs, pushed feature commit, and issue-close results.
- **Test/review/repair/PR agent:** target commit, report path or resulting commit, and outcome.

Write full reports to disk and return links to them. The coordinator reports each completed batch in the main conversation.

## 0. Preflight

Verify the Git repository and remote, `gh auth status`, the GitHub repository, the default branch, and the available subagent/worktree capabilities. Check for an existing run with the same feature slug so a restart does not duplicate branches or GitHub issues. Create project domain files only when the workflow needs them: `GLOSSARY.md` when a term is resolved and `docs/adr/` when a decision warrants an ADR.

**Done when:** the target repository, default branch, issue tracker, agent capability, and safe starting state are known.

## 1. Grill with docs

Conduct the interview in the main conversation using [grilling](references/grilling.md), [grill-with-docs](references/grill-with-docs.md), and [domain modeling](references/domain-modeling.md). Work through the design tree in rounds, resolve terms in the glossary as they settle, and record only decisions that meet the ADR threshold. Confirm the public testing seams during this design phase. Write `.scratch/<feature-slug>/grill-brief.md` with the problem, decisions, constraints, testing seams, and links to domain docs.

When the frontier is empty, summarize the settled design and ask the user to confirm that grilling is complete. **Wait for an affirmative answer before writing the spec.**

**Done when:** the user confirms the design and the grill brief captures it.

## 2. Spec

Launch one fresh spec-writing agent with the grill brief, domain docs, and [to-spec](references/to-spec.md). It writes `.scratch/<feature-slug>/spec.md` with the user stories, implementation decisions, scope, and agreed testing seams. The coordinator checks that every settled design decision is represented before moving on.

**Done when:** the spec covers the confirmed design and its testing boundaries.

## 3. Tickets

Launch one fresh ticket-slicing agent with the spec, glossary, and [to-tickets](references/to-tickets.md). It writes one vertical slice per file under `.scratch/<feature-slug>/tickets/`, with acceptance criteria and explicit blockers. The coordinator checks for cycles, missing story coverage, and tickets too large for a fresh implementer. Record the graph in `run-state.md`. Resolve any remaining scope decision with the user; do not create a routine extra approval gate for an already confirmed design.

**Done when:** every story has a ticket path and the initial ready set is known.

## 3.5. Preserve context

Persist the run state, spec path, and ticket paths before context compaction. If the host provides an executable compaction action, use it and continue from those pointers. Otherwise continue with the compact state manifest; writing `/compact` as ordinary chat text does not execute it. Do not ask the user to perform compaction.

## 4. Implement the ticket graph

Read [implement-spec](references/implement-spec.md) and [tdd](references/tdd.md). Launch a fresh setup agent to create or resume the feature branch and GitHub parent/ticket issues; record its receipts. The **coordinator** repeats the following loop:

1. Select ready tickets from the recorded graph. Dispatch one fresh implementer per ticket, concurrently up to host capacity. Give every implementer in a parallel batch the same remote feature commit as its base and a distinct worktree and `work/<ticket-slug>` branch. Each agent implements and commits only its assigned ticket.
2. **One-ticket batch:** Its implementer verifies ancestry, fast-forwards the remote feature branch from its ticket commit, closes its issue after the push succeeds, and reports the feature commit. No integration agent is needed.
3. **Parallel batch:** Implementers push only their own ticket branches. After their receipts arrive, launch one fresh integration agent with those exact branch commits. It combines the batch, resolves actual conflicts, pushes the feature branch once, then closes the batch's issues and reports the feature commit. Implementers never compete to push a moving feature branch.
4. Update `run-state.md` from the feature-commit receipt, report the batch, and recompute the ready set. If a ticket or integration cannot finish, use a fresh repair agent for that work and hold dependent tickets. If issue closure fails after a successful push, record and repair the issue state without treating the code as absent.

Continue until every ticket is present in the remote feature branch. Clean up completed worktrees after their commits are safely integrated.

**Done when:** all ticket commits are integrated and the run state identifies the remote feature commit.

## 5. Final verification

Launch a fresh test agent and a fresh code review agent in separate worktrees at the **same feature commit**:

- The test agent runs the repository's required test, type, lint, and build checks and writes `.scratch/<feature-slug>/test-report.md` with commands, outcomes, and commit.
- The reviewer compares the feature branch with the detected default branch, following [code-review](references/code-review.md), and writes `.scratch/<feature-slug>/review-report.md` with separate Standards and Spec findings and the commit.

For any failure or blocking finding, launch a fresh repair agent, push its change to the feature branch, then rerun **both** final agents against the new commit. If the feature branch moves for any reason while checks run, rerun them at its new head.

**Done when:** tests pass, review has no blocking findings, and both reports name the current remote feature commit.

## 6. Create the PR

Launch a fresh PR agent with the spec, final reports, feature commit, and [PR guidance](references/pr.md). It confirms the remote feature head still matches the checked commit, creates or updates a PR against the default branch, and returns its URL. Include a concise summary, test evidence, review outcome, and parent issue reference. Check required GitHub CI checks if configured; repair and rerun verification if they expose a defect.

**Done when:** the PR points at the checked feature commit and required checks are reported.

## 7. Ask before merging

Present the PR URL, delivered behavior, test and review evidence, and any remaining risk. Ask the user whether to merge. On explicit confirmation, merge the PR with the repository's agreed strategy, then sync the workspace without discarding local changes. Report the merged commit and final state. If the user does not confirm, leave the PR ready for review.
