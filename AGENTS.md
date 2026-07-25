# AI Instructions

## Project

This repository is the public Quave ONE deploy GitHub Action. Customer examples use `quaveone/quaveone-deploy-action@main`, so `main` is a customer-facing alias and must only point at a version that has already been tested.

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
