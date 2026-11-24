---
name: implement-spec
description: "Implement a spec's tickets through fresh per-ticket agents and integrate each ready batch."
disable-model-invocation: true
---

# Implement Spec

The coordinator owns the ticket graph. A ticket is ready when every blocking ticket is integrated into the shared feature branch. Read the spec and ticket files to determine each ready batch. Keep the coordinator in the main thread; no implementation subagent receives the whole queue.

## Setup

Use a fresh setup subagent to create and push `feat/<feature-slug>` from `main`, create the parent GitHub issue and ticket issues, and return the issue numbers and feature branch commit. The coordinator records that commit as the first batch base.

## Ticket batches

For each ready ticket, start a fresh implementer in its own worktree on `work/<ticket-slug>`, based on the same feature branch commit for that batch. Give it the spec and one ticket, its issue number, the base commit, and the `tdd` reference. It implements and commits only that ticket. Parallel implementers do not pull or push the moving feature branch while their batch is running.

- **One ready ticket:** Its implementer verifies that its branch descends from the batch base, then fast-forwards the remote feature branch with `git push origin HEAD:refs/heads/feat/<feature-slug>`. It closes its issue after the push succeeds and reports the new feature branch commit. No integration subagent is needed.
- **Multiple ready tickets:** Each implementer pushes only its own `work/<ticket-slug>` branch and reports the branch and commit. After every implementer in the batch has reported, start one fresh integration subagent. It combines those branches into `feat/<feature-slug>`, resolves actual conflicts, pushes that branch once, closes the batch's issues after the push succeeds, and reports the new feature branch commit. A branch that could not be integrated keeps its issue open.

Only a receipt confirming the updated feature branch commit unblocks dependent tickets. The coordinator reports each completed batch to the user, recomputes the frontier, and starts new agents for the next batch. If a ticket or batch cannot finish, use a fresh repair subagent and keep dependent tickets waiting for integration.

## Final checks

After all batches, run a fresh test agent and a fresh code review agent in separate worktrees against the same feature branch commit. A fresh repair agent fixes and pushes failures or blocking review findings. Rerun both checks against each repaired commit until both pass. Then create the PR using the final reports and leave the merge into `main` for the user's final confirmation.
