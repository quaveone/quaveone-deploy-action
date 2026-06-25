# AI Instructions

## Project

This repository is the public Quave ONE deploy GitHub Action. Customer examples use `quaveone/quaveone-deploy-action@main`, so `main` must always point at the latest stable Quave ONE CLI image.

## Release rule

When a new stable Quave ONE CLI version is released, `action.yml` must be updated to `docker://quaveone/quaveone-cli:<version>`, `main` must contain that update, and the matching action release must be marked **Latest**.

Prefer the automation in `quaveone/quaveone-client-utils` Release workflow. For manual repair or standalone action release, use `.agents/skills/release-deploy-action/SKILL.md`.
