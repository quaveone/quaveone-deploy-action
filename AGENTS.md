# AI Instructions

## Project

This repository is the public Quave ONE deploy GitHub Action. Customer examples use `quaveone/quaveone-deploy-action@main`, so `main` must always point at the latest stable Quave ONE CLI image.

## Release rule

When a new stable Quave ONE CLI version is released, `action.yml` must be updated to `docker://quaveone/quaveone-cli:<version>`, `main` must contain that update, and the matching action release must be marked **Latest**.

Prefer the automation in `quaveone/quaveone-client-utils` Release workflow. For manual repair or standalone action release, use `.agents/skills/release-deploy-action/SKILL.md`.
## Action metadata safety

`action.yml` is parsed by GitHub Actions before the Docker container starts. Do **not** put `${{ github.* }}`, `${{ env.* }}`, or other runner contexts directly in Docker action metadata such as `runs.env` or `runs.args`; GitHub can reject the action before any shell script runs. Prefer `${{ inputs.* }}` in metadata and read runner-provided variables such as `GITHUB_EVENT_NAME`, `GITHUB_REF_NAME`, `GITHUB_HEAD_REF`, and `GITHUB_EVENT_PATH` inside the container script. For pull request event actions, parse `$GITHUB_EVENT_PATH` inside the script instead of using `${{ github.event.action }}` in `action.yml`.

Every change to `action.yml` must run the smoke workflow. Pull requests must pass the local `uses: ./` smoke, and after merging to `main` the published `uses: quaveone/quaveone-deploy-action@main` smoke must pass before agents tell zcloud or customers to use `@main`.

