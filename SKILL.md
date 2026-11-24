---
name: ship-feature
description: "End-to-end feature delivery: grill requirements, write a spec, slice tickets, implement each ready ticket in a fresh agent, integrate parallel batches, run final tests and review, and create a PR."
disable-model-invocation: true
---

# Ship Feature

Autonomous end-to-end engineering pipeline unifying repository initialization, interactive grilling with domain docs, formal specification, vertical tracer-bullet slicing, autonomous context compaction, parallel TDD execution in isolated Git worktrees, two-axis code review, PR creation, and user-confirmed merge.

Designed for universal execution across all coding agents (Claude Code, Antigravity, OpenAI Codex, Cursor, ChatGPT, and open-source LLM runners).

---

## 🛡️ Core Architecture: Zero-Dump-Zone Context Isolation

To prevent context window bloat and reasoning degradation (the 100k–150k+ "dump zone"), this skill strictly enforces context isolation:

1. **Context Pointer Protocol**: Coordinator and subagents communicate **strictly via file pointers** (`.scratch/<feature-slug>/...`, `GLOSSARY.md`, `docs/adr/`). **NEVER** dump conversational transcripts, raw file contents, or tool call logs into subagent prompts.
2. **Dedicated Fresh Subagents**: Each ticket and each delegated phase runs in a fresh context window. The coordinator retains the ticket frontier and starts a new implementer for every ticket.
3. **Receipts Only**: Subagents write any reports to disk and reply with compact receipts containing report links or the branch and commit needed for the next handoff.
4. **Bundled References**: Subagents follow the relevant workflow references bundled in `./references/`.
5. **Autonomous Compaction**: The coordinator automatically issues `/compact` after ticket decomposition to flush conversational grilling history before implementation.

```
Coordinator (Main Thread)
├── Step 0: Pre-flight & Fast-Track Init
├── Step 1: Grilling with Docs ──► .scratch/<slug>/grill-brief.md
├── [MANDATORY GATE: User confirms grilling is complete]
├── Step 3.5: Autonomous Context Compaction (/compact)
├── Step 4: Ticket frontier and batch dispatch
├── Step 5: Collect final test and review receipts
└── Step 7: Final User Confirmation Gate & Merge into main

Subagents (Clean Context Windows)
├── Step 2: Spec Writer Subagent       ──► .scratch/<slug>/spec.md
├── Step 3: Ticket Slicer Subagent     ──► .scratch/<slug>/tickets/NN-*.md
├── Step 4: One Implementer per ticket ──► ticket branches/worktrees
├── Step 4: Batch Integrator if parallel ──► feat/<slug>
├── Step 5: Test and Code Review Agents ──► repair agent for findings
└── Step 6: PR Creator Subagent        ──► gh pr create with evidence
```

---

## 📚 Bundled Reference Skills (`./references/`)

Subagents and coordinator must strictly consult and adhere to their respective reference files in the local `./references/` directory:

| Phase | Bundled Reference Document | Purpose |
| :--- | :--- | :--- |
| **Grilling** | [`references/grilling.md`](./references/grilling.md)<br/>[`references/grill-with-docs.md`](./references/grill-with-docs.md) | Design-tree interview in iterative frontier rounds |
| **Domain Docs** | [`references/domain-modeling.md`](./references/domain-modeling.md)<br/>[`references/ADR-FORMAT.md`](./references/ADR-FORMAT.md)<br/>[`references/GLOSSARY-FORMAT.md`](./references/GLOSSARY-FORMAT.md) | In-flight glossary sharpening & minimal ADR recording |
| **Spec** | [`references/to-spec.md`](./references/to-spec.md) | Highest public testing seams, exhaustive user stories, decisions |
| **Tickets** | [`references/to-tickets.md`](./references/to-tickets.md) | Vertical tracer-bullet slices, blocking edges, expand-contract |
| **Implementation** | [`references/implement-spec.md`](./references/implement-spec.md)<br/>[`references/tdd.md`](./references/tdd.md) | One fresh implementer per ticket, isolated parallel worktrees, batch integration |
| **Code Review** | [`references/code-review.md`](./references/code-review.md) | Two-axis separation: Standards (Fowler smells) + Spec validation |
| **PR** | [`references/pr.md`](./references/pr.md) | Structured PR body: Mermaid/call-tree summary, test evidence, merge danger |

---

## 🌐 Universal Multi-Agent Execution Contract

This pipeline is platform-agnostic and runs across any coding agent harness:

- **Antigravity**: Launch subagents via `invoke_subagent(Role="...", Prompt="...")`. Recommend or execute compaction.
- **Claude Code**: Launch subagents via the `Task` tool (or subagent commands) and trigger compaction via `/compact`.
- **OpenAI Codex / ChatGPT / Cursor / OpenRouter / Local LLMs**: Dispatch fresh subagent sessions or subshells with isolated contexts.

**The Golden Rule of Prompts**: When launching any subagent, the prompt contains only:
1. The subagent's role and target objective.
2. The relative path to its bundled reference file (e.g. `./references/to-spec.md`).
3. The relevant file paths, issue number, base commit, and worktree path.
4. The output artifact path, when the role writes a report.
5. The instruction to reply with a concise receipt and the report link or resulting branch and commit.

---

## Step 0: Repository Pre-flight & Fast-Track Init

The coordinator verifies the repository and GitHub environment before starting:

1. **Git & Remote Verification**:
   - `git rev-parse --is-inside-work-tree` (if uninitialized: `git init`).
   - `gh auth status` and `gh repo view` (verify GitHub CLI authentication and repo access).
2. **Config Scaffolding** (create immediately if missing, without asking):
   - `docs/agents/issue-tracker.md`: specifies GitHub Issues via `gh` CLI.
   - `docs/agents/domain.md`: points to `GLOSSARY.md` and `docs/adr/`.
   - `docs/agents/triage-labels.md`: canonical labels (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`).
   - Create `GLOSSARY.md` when the first domain term is resolved; create `docs/adr/` when a decision warrants an ADR.
   - `AGENTS.md` (or `CLAUDE.md`): ensure agent skills block exists.
   - Canonical labels: `gh label create <name> --force` for missing labels.

**Completion**: Git worktree ready, `gh` authenticated, domain config files present.

---

## Step 1: Interactive Grilling with Docs (`grill-with-docs`)

The coordinator conducts the requirements and architecture interview in the **main conversation thread**, strictly following [`references/grilling.md`](./references/grilling.md) and [`references/domain-modeling.md`](./references/domain-modeling.md):

1. **Iterative Multi-Round Grilling**:
   - Map requirements as a **design tree**.
   - Identify the **frontier**: every decision whose prerequisites are settled.
   - Present rounds of numbered questions formatted so "yes" accepts the recommended answer:
     ```
     ❓ **Q1** - **<title>**: <details and trade-offs>
     ➡️ <recommended answer>
     ```
   - **Do NOT stop prematurely**: Grilling is NOT limited to a single pass or 5 questions. After every user reply, recompute the frontier and ask the next round. Continue until every ambiguous branch is settled.
2. **Inline Domain Modeling**:
   - Immediately update `GLOSSARY.md` whenever domain terms are clarified or disambiguated.
   - Record ADRs in `docs/adr/NNNN-<slug>.md` if decisions are hard to reverse, surprising, and trade-off heavy.
3. **Persist Grill Brief**:
   - Write `.scratch/<feature-slug>/grill-brief.md`:
     - Problem statement and target outcome.
     - Settled architectural decisions, seams, and constraints.
     - Pointers to updated `GLOSSARY.md` entries and ADRs.

### 🚨 Mandatory Grilling Frontier Gate (User Approval Required)

When the coordinator believes the design tree is completely settled, it **MUST HALT** and explicitly ask:

> *"The requirements, domain terms, and architectural decisions appear settled. Is this grilling enough or do you have any additional questions or doubts? Shall we proceed to `/to-spec`?"*

**STRICT ENFORCEMENT**: The coordinator is **strictly forbidden** from initiating Step 2 (`to-spec`) until the user explicitly grants affirmative confirmation.

---

## Step 2: Spec Synthesis Subagent (`to-spec`)

Upon user approval, the coordinator launches a fresh subagent:
- **Role**: `Spec Writer`
- **Pointers**: `.scratch/<feature-slug>/grill-brief.md`, `GLOSSARY.md`, `docs/adr/`, [`references/to-spec.md`](./references/to-spec.md)
- **Output**: `.scratch/<feature-slug>/spec.md`

**Subagent Execution**:
1. Reads `references/to-spec.md` and adopts the `<spec-template>`.
2. Proposes public testing seams (highest viable external boundary).
3. Produces `.scratch/<feature-slug>/spec.md` with sections:
   - Problem Statement & Solution (user perspective)
   - Exhaustive numbered User Stories (`As an <actor>, I want <feature>, so that <benefit>`)
   - Implementation Decisions (modules, interfaces, schema, API contracts — no fragile code dumps)
   - Testing Decisions (external behavior only, modules under test, seams)
   - Out of Scope & Further Notes
4. Returns a 3-sentence summary receipt and markdown link: `[spec.md](.scratch/<feature-slug>/spec.md)`.

---

## Step 3: Vertical Ticket Slicing Subagent (`to-tickets`)

Immediately upon receiving the spec receipt, the coordinator launches a fresh subagent:
- **Role**: `Ticket Slicer`
- **Pointers**: `.scratch/<feature-slug>/spec.md`, `GLOSSARY.md`, [`references/to-tickets.md`](./references/to-tickets.md)
- **Output**: `.scratch/<feature-slug>/tickets/`

**Subagent Execution**:
1. Reads `references/to-tickets.md` and applies the vertical tracer-bullet rules.
2. Decomposes the spec into vertical slices cutting through schema $\rightarrow$ API $\rightarrow$ UI $\rightarrow$ tests.
3. Sizes each ticket to fit comfortably in a single subagent context window.
4. For wide refactors, applies expand-contract sequencing.
5. Writes individual ticket files: `.scratch/<feature-slug>/tickets/NN-<slug>.md` (with title, what to build, acceptance criteria, and `Blocked by: Ticket NN` or `None`).
6. Returns a receipt with:
   - Ticket dependency table (Number, Title, Delivers, Blocked By).
   - Initial **frontier** (unblocked tickets ready to start).

---

## Step 3.5: Autonomous Coordinator Compaction (`/compact`)

Immediately upon receiving the ticket receipt, the coordinator **automatically executes context compaction** in the main thread before starting implementation:

1. **Zero Human Intervention**: The coordinator executes compaction autonomously without pausing or prompting the user.
2. **Compaction Directive**:
   - On Claude Code or harnesses supporting slash commands, output:
     ```
     /compact Retain only feature slug <feature-slug>, branch feat/<feature-slug>, and pointers to .scratch/<feature-slug>/spec.md and .scratch/<feature-slug>/tickets/. Prune all conversational grilling history, questions, and exploration logs.
     ```
   - On Antigravity and generic LLM harnesses, prune working memory down to the minimal pointer manifest linking to the spec, tickets, and repository status.
3. **Seamless Transition**: Immediately proceed to Step 4 without waiting for human input.

---

## Step 4: Coordinator Ticket Loop (`implement-spec` + `tdd`)

The coordinator reads [`references/implement-spec.md`](./references/implement-spec.md) and keeps the ticket graph in the main thread. A fresh setup subagent creates and pushes `feat/<feature-slug>` from `main`, creates the parent spec issue and ticket issues, and returns their identifiers. The coordinator then repeats this loop until every ticket is integrated:

1. **Choose a ready batch**: Select every ticket whose blockers are already integrated into `feat/<feature-slug>`. Record the feature branch commit that the batch will start from. A closed issue alone does not unblock a dependent ticket.
2. **Dispatch one fresh implementer per ready ticket**: Start independent subagents concurrently when the batch has multiple tickets. Give each only its ticket path and issue number, spec path, base commit, worktree path, and pointers to [`references/implement-spec.md`](./references/implement-spec.md) and [`references/tdd.md`](./references/tdd.md). Each implementer works in its own worktree and `work/<ticket-slug>` branch based on the batch commit. It implements only that ticket, commits its work, and returns a receipt with its branch and commit. It does not take the next ticket or orchestrate the queue.
3. **Single-ticket batch**: The implementer verifies its ticket branch descends from the batch base, then fast-forwards the remote feature branch with `git push origin HEAD:refs/heads/feat/<feature-slug>`. It closes its own GitHub issue after the push succeeds and reports the resulting feature branch commit. The coordinator starts the next ready batch from that commit; no integration subagent is needed.
4. **Parallel batch**: Each implementer pushes only its own `work/<ticket-slug>` branch and reports its commit. Once all implementers in the batch finish, the coordinator launches one fresh integration subagent with pointers to those branches and issues. That subagent combines the branches into `feat/<feature-slug>`, resolves merge conflicts, pushes the feature branch once, closes the batch's issues only after the push succeeds, and returns the new feature branch commit. Implementers do not race to push the feature branch.
5. **Advance on feature branch receipts**: The coordinator reports the completed batch to the user, marks those tickets integrated, recomputes the frontier, and launches fresh implementers for the next batch. If an implementation or integration is incomplete, dispatch a fresh repair subagent for that work and hold its dependent tickets until a receipt confirms the updated feature branch commit.

The coordinator delegates GitHub writes and branch integration to subagents. It never hands the entire ticket queue to one implementation subagent. Testing and code review of the whole feature follow after the final batch is integrated.

---

## Step 5: Final Test and Code Review Agents

After all tickets are integrated, the coordinator launches two fresh subagents in separate worktrees against the same feature branch commit:

1. **Test Agent**: Runs the project's full test suite, typechecker, and other required checks. Writes `.scratch/<feature-slug>/test-report.md` with commands, results, and the commit tested.
2. **Code Reviewer**: Reviews `feat/<feature-slug>` against `main`, following [`references/code-review.md`](./references/code-review.md) for separate Standards and Spec review passes. Writes `.scratch/<feature-slug>/review-report.md` with findings and the commit reviewed.
3. **Repair loop**: For any test failure or blocking review finding, the coordinator launches a fresh repair subagent with the relevant reports. The repair agent fixes and pushes the feature branch, then returns its new commit. The coordinator reruns both final agents against that commit. Continue until tests pass and the two-axis review has no blocking findings.

The coordinator proceeds to PR creation only when both reports refer to the same passing feature branch commit.

---

## Step 6: PR Creation Subagent (`pr`)

The coordinator launches a fresh subagent:
- **Role**: `PR Creator`
- **Pointers**: `feat/<feature-slug>`, `.scratch/<feature-slug>/spec.md`, `.scratch/<feature-slug>/test-report.md`, `.scratch/<feature-slug>/review-report.md`, [`references/pr.md`](./references/pr.md)

**Subagent Execution**:
1. Formats PR body strictly adhering to `references/pr.md`:
   - **Summary**: Mermaid sequence/flowchart or component call-tree showing architecture.
   - **Evidence**: The final test report and any repair and rerun results.
   - **Merge Danger**: Door (`one-way` | `two-way`) and Blast Radius (`narrow` | `moderate` | `wide`).
   - Reference: `Closes #<ParentSpecIssueNumber>`.
2. Pushes branch: `git push -u origin feat/<feature-slug>`.
3. Creates PR: `gh pr create --base main --head feat/<feature-slug> --title "feat: <Feature Title>" --body-file <temp-body-file>`.
4. Returns PR URL, title, and summary receipt to coordinator.

---

## Step 7: Final User Confirmation & PR Merge into Main

The coordinator presents the completed PR to the user in the main chat thread:
1. Displays PR URL, executive summary, and test evidence.
2. **Presents the Final Confirmation Prompt**:
   > *"PR #NN is ready with all tests passing, two-axis review clean, and zero blocking issues. Shall I squash-merge this PR into `main` and sync your workspace?"*
3. **Execution Upon Confirmation**:
   - Merge PR: `gh pr merge --squash --delete-branch`
   - Switch and sync:
     ```bash
     git checkout main
     git pull origin main
     ```
   - Confirm to the user that the feature is live on `main`, remote branch deleted, and workspace is synced and clean.
