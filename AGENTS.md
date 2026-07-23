# AI Instructions

## Project

This repository is the public Quave ONE deploy GitHub Action. Customer examples use `quaveone/quaveone-deploy-action@main`, so `main` is a customer-facing alias and must only point at a version that has already been tested.

## Shared product behavior

This action is a thin non-interactive wrapper around the Quave ONE CLI. Do not duplicate deploy, preview, JobRun, account, or environment product logic in `action.yml` or shell snippets. Pass inputs to the CLI and let the CLI call zcloud public API endpoints that reuse app/server validation, permission, service, formatter, and audit logic. If the action needs behavior the CLI does not expose, update the CLI/zcloud API path first instead of adding an action-only product path.

The action must stay token-based and CI-safe. It should use explicit inputs and `QUAVEONE_*` environment variables, never local `quaveone login` config or any interactive prompt.

## Release rule

Deploy-action releases also have two phases: **test a fixed candidate** and **promote customer aliases**. Do not move `main` or mark a release **Latest** until the candidate action has passed the complete smoke in real GitHub Actions by fixed branch, tag, or SHA.

Safe promotion order:

1. Prepare the candidate in a branch or other fixed ref with `action.yml` pointing at the target `docker://quaveone/quaveone-cli:<version>`.
2. Test that fixed ref in GitHub Actions. The smoke must load the action metadata and exercise deploy, preview create, and preview delete paths when those paths exist.
3. Only after the fixed-ref smoke passes, update/merge `main`.
4. Run `Smoke Deploy Action` on `main` and wait for the published `uses: quaveone/quaveone-deploy-action@main` job to pass.
5. Only after the `main` smoke passes, mark the matching deploy-action release as **Latest**.

If `main` is broken, first restore it to the last tested-good action version, prove that rollback with smoke, and only then continue fixing the new candidate.

Prefer the automation in `quaveone/quaveone-client-utils` Release workflow only after the CLI version and action candidate were tested by fixed refs. For manual repair or standalone action release, use `.agents/skills/release-deploy-action/SKILL.md`.

## Action metadata safety

`action.yml` is parsed by GitHub Actions before the Docker container starts. Do **not** put `${{ github.* }}`, `${{ env.* }}`, or other runner contexts directly in Docker action metadata such as `runs.env` or `runs.args`; GitHub can reject the action before any shell script runs. Prefer `${{ inputs.* }}` in metadata and read runner-provided variables such as `GITHUB_EVENT_NAME`, `GITHUB_REF_NAME`, `GITHUB_HEAD_REF`, and `GITHUB_EVENT_PATH` inside the container script. For pull request event actions, parse `$GITHUB_EVENT_PATH` inside the script instead of using `${{ github.event.action }}` in `action.yml`.

Every change to `action.yml` must run the smoke workflow. Pull requests must pass the local `uses: ./` smoke, fixed-candidate tests must pass before `main` moves, and after merging to `main` the published `uses: quaveone/quaveone-deploy-action@main` smoke must pass before agents tell zcloud or customers to use `@main`.
