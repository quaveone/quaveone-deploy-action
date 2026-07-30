# AI Instructions

## Project

This repository is the public Quave ONE deploy GitHub Action. Customer examples use `quaveone/quaveone-deploy-action@main`, so `main` is a customer-facing alias and must only point at a version that has already been tested.

## Mandatory issue-first workflow for code and public docs

Before an AI agent modifies code, tests, scripts, workflows, configuration, generated files, release automation, public docs, README customer instructions, examples, changelogs that customers read, or any customer-facing artifact in any Quave ONE related repo, it must create or reuse a GitHub issue first. This is mandatory for bugs, features, refactors, chores, public documentation, and release/action changes. The issue ID must exist before commits so every change can be tracked over time.

**Reuse before create:** work that is part of an already-tracked issue — autoreview findings on its PR, follow-ups discovered during its implementation, refinements of its acceptance criteria — must **not** get a new issue. Add execution-checklist lines to the parent issue and implement under the parent's branch/PR. Create a new issue only for genuinely separable work, or when the parent issue is already Done/closed.

Exception: no GitHub issue is required for internal-only AI-agent documentation and workflow-maintenance changes, such as `AGENTS.md`, `CLAUDE.md`, `.agents/skills/**`, `.claude/skills/**`, `.cursor/**`, repo-local `.ai/**`, or internal AI docs, as long as the branch does not also change product code, tests, workflows, config, public docs, or customer-facing files.

For work the AI agent will implement immediately, use `.agents/skills/create-issue/SKILL.md` before editing, create the issue in the repo whose files will change, add it to the Quave ONE project (`https://github.com/orgs/zcloud-ws/projects/1`), assign it to the current logged-in GitHub user (`gh api user --jq .login`), set the current project iteration, and move it to **In Development**. This applies equally to `zcloud-ws/zcloud`, `zcloud-ws/zcloud-infra`, `quaveone/quaveone-client-utils`, `quaveone/cli`, and `quaveone/quaveone-deploy-action`; do not create a zcloud issue for a change that will be committed in another repo. GitHub Projects accepts issue URLs from other organizations, so cross-org `quaveone/*` issues should still be added to the shared `zcloud-ws` Quave ONE project instead of moving repos or duplicating issues.

Canonical policy lives in `zcloud-ws/zcloud` at `internal-docs/ai/issue-first-workflow.md`. When this workflow changes, update zcloud first and mirror the concise `AGENTS.md` rule plus `.agents/skills/create-issue` / `.agents/skills/implement-issue` to the Quave ONE related repos.

## Shared product behavior

This action is a thin non-interactive wrapper around the Quave ONE CLI. Do not duplicate deploy, preview, JobRun, account, or environment product logic in `action.yml` or shell snippets. Pass inputs to the CLI and let the CLI call zcloud public API endpoints that reuse app/server validation, permission, service, formatter, and audit logic. If the action needs behavior the CLI does not expose, update the CLI/zcloud API path first instead of adding an action-only product path.

The action must stay token-based and CI-safe. It should use explicit inputs and `QUAVEONE_*` environment variables, never local `quaveone login` config or any interactive prompt.

## Release rule

This action intentionally runs `docker://quaveone/quaveone-cli:latest`. The CLI release workflow promotes that Docker `latest` alias only after a fixed CLI version has been published and smoke-tested, which gives action users the same vetted CLI version as installer users without changing `action.yml` for every CLI release.

Do **not** release or retag this action for ordinary CLI releases. CLI-only changes reach `quaveone/quaveone-deploy-action@main` by promoting `quaveone/quaveone-cli:latest`; after a CLI change that affects action users, run the action smoke workflow on `main` to prove the current action metadata still loads the promoted CLI. Create an action PR/release only when `action.yml`, inputs, wrapper shell, or action docs changed.

Safe promotion order:

1. Publish the CLI as a fixed version and test that exact CLI asset/image.
2. Promote the CLI/Docker `latest` aliases only after the fixed-version smoke passes.
3. If the CLI change affects action-used behavior, run `Smoke Deploy Action` on deploy-action `main`; the published `uses: quaveone/quaveone-deploy-action@main` job must pass against the promoted CLI `latest` image.
4. For action metadata/wrapper/docs changes, prepare a candidate branch or SHA, run the pull-request/fixed-ref action smoke, merge/update `main` only after that passes, then run the published `@main` smoke.
5. Mark a deploy-action GitHub Release **Latest** only for action changes, and only after the published-main smoke passes.

If `main` is broken, first restore it to the last tested-good action commit, prove that rollback with smoke, and only then continue fixing the new candidate.

The retired `quaveone/quaveone-client-utils` `sync_deploy_action` path must not retarget this action to fixed CLI images anymore. For manual repair or standalone action release, use `.agents/skills/release-deploy-action/SKILL.md`.

## Action metadata safety

`action.yml` is parsed by GitHub Actions before the Docker container starts. Do **not** put `${{ github.* }}`, `${{ env.* }}`, or other runner contexts directly in Docker action metadata such as `runs.env` or `runs.args`; GitHub can reject the action before any shell script runs. Prefer `${{ inputs.* }}` in metadata and read runner-provided variables such as `GITHUB_EVENT_NAME`, `GITHUB_REF_NAME`, `GITHUB_HEAD_REF`, and `GITHUB_EVENT_PATH` inside the container script. For pull request event actions, parse `$GITHUB_EVENT_PATH` inside the script instead of using `${{ github.event.action }}` in `action.yml`.

Every change to `action.yml` must run the smoke workflow. Pull requests must pass the local `uses: ./` smoke, fixed-candidate tests must pass before `main` moves, and after merging to `main` the published `uses: quaveone/quaveone-deploy-action@main` smoke must pass before agents tell zcloud or customers to use `@main`.
