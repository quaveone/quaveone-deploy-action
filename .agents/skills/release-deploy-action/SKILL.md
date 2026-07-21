---
name: release-deploy-action
description: Release or repair quaveone/quaveone-deploy-action so its main branch, action.yml Docker image, Git tag, and GitHub Latest release match the latest Quave ONE CLI version. Use when publishing a deploy action release, syncing the action after a CLI release, or fixing stale main/latest markers.
---
# Release Deploy Action

Use this workflow when the Quave ONE CLI version changes and the deploy action must point to the new CLI image. The action's `main` branch is a customer-facing alias, so it must only move after a fixed candidate has passed smoke tests in real GitHub Actions. The GitHub Release **Latest** marker must only move after the published `@main` smoke passes.

## Golden rule: test fixed candidate before moving main/latest

Customers use `quaveone/quaveone-deploy-action@main`. Treat `main` exactly like Docker `latest`: mutable, customer-facing, and dangerous if untested.

Safe order:

1. Prepare a candidate branch or SHA with `action.yml` updated to `docker://quaveone/quaveone-cli:X.Y.Z`.
2. Test that candidate by fixed ref in GitHub Actions. Local YAML parsing and direct Docker runs are necessary but not sufficient.
3. Merge/update `main` only after the fixed-ref candidate passed.
4. Run `Smoke Deploy Action` on `main` and wait for the published-main job (`uses: quaveone/quaveone-deploy-action@main`) to pass.
5. Mark the deploy-action release `vX.Y.Z` as **Latest** only after the published-main smoke passes.

If `main` is already broken, immediately restore `main` to the last tested-good action version, prove the rollback with the smoke workflow, then work on the new candidate in a separate branch.

## Action metadata safety

GitHub parses `action.yml` before the Docker container starts. Docker action metadata must not reference runner contexts such as `${{ github.* }}` or `${{ env.* }}` in `runs.env` or `runs.args`; those can make every `uses: quaveone/quaveone-deploy-action@main` workflow fail during the `Set up job` phase. Keep metadata limited to supported values such as `${{ inputs.* }}` and move event-specific logic into the shell script, using runner-provided environment variables (`GITHUB_EVENT_NAME`, `GITHUB_REF_NAME`, `GITHUB_HEAD_REF`, `GITHUB_EVENT_PATH`) inside the container. Parse pull request `action` from `$GITHUB_EVENT_PATH`; do not use `${{ github.event.action }}` in `action.yml`.

## Preferred path

The safest path is a PR-based promotion:

1. Create a branch from `main`.
2. Update `action.yml` to the target CLI image.
3. Run local validation:

   ```bash
   ruby -e 'require "yaml"; YAML.load_file("action.yml"); YAML.load_file(".github/workflows/smoke.yml")'
   ! grep -En '\$\{\{ *(github|env)\.' action.yml
   docker buildx imagetools inspect quaveone/quaveone-cli:1.0.39
   ```

4. Open a PR. The PR smoke must pass the local `uses: ./` job.
5. If practical, run an additional fixed-ref consumer workflow using the branch or commit SHA.
6. Merge to `main` only after fixed-ref smoke passes.
7. Wait for `Smoke Deploy Action` on `main` to pass, including the published-main job.
8. Create or update release `vX.Y.Z` and mark it **Latest**.

The `quaveone/quaveone-client-utils` release workflow and this repo's manual **Release Deploy Action** workflow are promotion tools. Do not run them against an untested candidate. When a workflow asks for `promotion_confirmation`, enter `tested-vX.Y.Z` only after the fixed-ref smoke has passed.

## Manual workflow dispatch

Use this only after fixed-ref candidate tests are already green. Replace `v1.0.39` with the target version and set the confirmation exactly:

```bash
gh workflow run release.yml \
  --repo quaveone/quaveone-deploy-action \
  --ref main \
  -f version=v1.0.39 \
  -f promotion_confirmation=tested-v1.0.39
```

The workflow will refuse to run without the matching confirmation. It will:

1. Update `action.yml` to `docker://quaveone/quaveone-cli:1.0.39`.
2. Commit the change directly to `main` when needed.
3. Create tag `v1.0.39` if it does not exist.
4. Create or update GitHub Release `v1.0.39` and mark it **Latest**.
5. Refuse to move an existing tag that points to a different commit.

Because this workflow can move customer-facing aliases, prefer the PR path unless this is a deliberate, already-tested promotion or an emergency repair.

## Verification

Replace `v1.0.39` and `1.0.39` with the target version.

```bash
gh api repos/quaveone/quaveone-deploy-action/contents/action.yml?ref=main --jq '.content' | base64 --decode | grep 'docker://quaveone/quaveone-cli:1.0.39'
gh api repos/quaveone/quaveone-deploy-action/releases/latest --jq '.tag_name'
gh api repos/quaveone/quaveone-deploy-action/git/ref/tags/v1.0.39 --jq '.object.sha'
docker buildx imagetools inspect quaveone/quaveone-cli:1.0.39
```

Run and wait for the action smoke workflow after `main` changes. It must prove both local metadata (`uses: ./`) and published metadata (`uses: quaveone/quaveone-deploy-action@main`) load in real GitHub Actions:

```bash
gh workflow run smoke.yml --repo quaveone/quaveone-deploy-action --ref main
gh run list --repo quaveone/quaveone-deploy-action --workflow smoke.yml --branch main --limit 5
```

## Do not finish until

- The candidate action was tested by fixed branch, tag, or SHA before `main` moved.
- `main` contains the new CLI image version only after the fixed-ref candidate passed.
- The matching tag exists and was not moved from a different commit.
- The matching GitHub Release is marked **Latest** only after the published-main smoke passed.
- The action smoke workflow passed on `main`, including the published `uses: quaveone/quaveone-deploy-action@main` job.
- Customer docs and downstream zcloud workflows can safely keep using `quaveone/quaveone-deploy-action@main`.
