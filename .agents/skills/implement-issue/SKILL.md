---
name: implement-issue
description: Implement a feature or task from a GitHub issue by using the issue body's execution checklist as the tracker, creating or repairing that checklist before code changes when needed, then completing each item with focused tests, review closeout, PR linkage, and Quave ONE project lifecycle updates. If no issue exists for code/public-doc work, first create one with create-issue.
---

# Implement Issue

Implement work from a GitHub issue without losing the plan, proof, or review trail. Issues created through `create-issue` should already include an `## Execution checklist`; refine that checklist rather than replacing it.

Canonical policy lives in `zcloud-ws/zcloud` at `internal-docs/ai/issue-first-workflow.md`; this repo mirrors the required local flow.

## Mandatory issue-first rule

If the user asks the AI agent to modify code, tests, scripts, workflows, configuration, generated files, release automation, public docs, README customer instructions, examples, changelogs that customers read, or any customer-facing artifact and does **not** provide an existing issue, stop before editing and run `.agents/skills/create-issue/SKILL.md` first. Create the issue in the repo whose files will change, add it to the Quave ONE project, assign the current logged-in GitHub user, set the current iteration, and move it to **In Development**.

Only internal AI-agent documentation/workflow-maintenance changes are exempt (`AGENTS.md`, `.agents/skills/**`, `.claude/skills/**`, `.cursor/**`, repo-local `.ai/**`, and similar) when no product code, public docs, workflows, config, or customer-facing files change.

## Core rules

1. **Use the checklist before implementation.** The issue body must contain an `## Execution checklist` section before code changes start. If an older issue lacks one, add it first.
2. **The issue body is the tracker.** Update checklist lines as work progresses. Comments are for detail and evidence, not status.
3. **Preserve the original issue.** Do not rewrite the user's problem statement away. Append or update the execution section.
4. **Work item by item.** Mark the current line `(IN PROGRESS)`, implement it, run focused proof, review it, then mark it done or blocked.
5. **Do not silently expand scope.** If required work is discovered, add a checklist line. If it is separable, create or propose a follow-up issue.
6. **Every done code/public-doc item needs proof.** Use focused tests, compile checks, lint, rendered docs checks, action smoke, UI screenshots, or clear manual verification notes.
7. **Use issue-linked commit messages.** Commit subjects must be `#<number> <issue title>` plus `- ` body bullets for the specific changes in that commit.
8. **Keep Quave ONE project status in sync automatically.** Move the issue to **In Development** when implementation starts, **In Review** when the PR opens, **In Staging** when staging verification begins, **Ready to Prod** when staging passes, and **Done** when production/main is shipped and verified.
9. **Keep branches clean.** Branch from the target base, avoid unrelated refactors, preserve user changes, and use disposable worktrees when a shared checkout is dirty.
10. **Cross-link multi-repo work.** Keep separate branches/PRs per repo and link every PR to the originating issue or explicit split issues.
11. **Request default reviewers.** `zcloud` and `quaveone/*` PRs request `filipenevola`; `zcloud-infra` PRs request `edimarlnx`, unless the issue says otherwise.

## Checklist format

Append or repair this section before coding:

```markdown
## Execution checklist

- [ ] (TODO) (PLAN) Confirm scope and acceptance criteria from the issue body.
- [ ] (TODO) (CODE) Implement <specific behavior>. Proof: <focused test/check>.
- [ ] (TODO) (TEST) Add or update regression coverage for <behavior>.
- [ ] (TODO) (DOCS, if needed) Update public/internal docs for <changed behavior>.
- [ ] (TODO) (VERIFY) Run <command or UI/API verification>.
- [ ] (TODO) (REVIEW) Run review closeout and resolve accepted findings.
- [ ] (TODO) (PR) Open PR and link it here.
- [ ] (TODO) (BOARD) Keep Quave ONE project status/comment in sync with the current lifecycle step.
```

Status rules:

- Not started: `- [ ] (TODO) ...`
- Active item: `- [ ] (IN PROGRESS) ...`
- Done: `- [x] (DONE) ... Proof: ...`
- Blocked required work: `- [ ] (BLOCKED) ... Reason: ...`
- Split to another issue/PR: `- [x] (SPLIT) ... Follow-up: <url>`
- Skipped because no longer needed: `- [x] (SKIP) ... Reason: ...`

## Workflow

1. Resolve the issue repo from the URL or current checkout. If no issue exists for code/public-doc work, create it first with `create-issue`.
2. Load the issue with `gh issue view <number> --repo <owner>/<repo> --json title,body,comments,url`.
3. Repair the `## Execution checklist` before editing if needed.
4. Add/move the issue on the Quave ONE project using `create-issue` helpers; immediate implementation means **In Development** + current iteration + assignee.
5. Create a branch named `codex/issue-<number>-<short-slug>` when practical.
6. Execute checklist items one by one, updating the issue body as the tracker.
7. Run focused proof for each done item and review closeout for non-trivial changes.
8. Open the PR, request the default reviewer, link the PR on the issue checklist, and move the project item to **In Review**.
9. Refetch the issue before final response and confirm checklist, PR, project status, and proof are synchronized.

## Final response

Return the issue URL, PR URL(s), Quave ONE board status, checklist counts, proof run, review result, and any blocked/deferred items.
