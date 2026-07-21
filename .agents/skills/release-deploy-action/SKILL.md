---
name: release-deploy-action
description: Release or repair quaveone/quaveone-deploy-action so its main branch, action.yml Docker image, Git tag, and GitHub Latest release match the latest Quave ONE CLI version. Use when publishing a deploy action release, syncing the action after a CLI release, or fixing stale main/latest markers.
---

# Release Deploy Action

Use this workflow when the Quave ONE CLI version changes and the deploy action must point to the new CLI image. The action's `main` branch is the version customers use in docs examples, so `main` must always match the latest stable CLI release.

## Action metadata safety

GitHub parses `action.yml` before the Docker container starts. Docker action metadata must not reference runner contexts such as `${{ github.* }}` or `${{ env.* }}` in `runs.env` or `runs.args`; those can make every `uses: quaveone/quaveone-deploy-action@main` workflow fail during the `Set up job` phase. Keep metadata limited to supported values such as `${{ inputs.* }}` and move event-specific logic into the shell script, using runner-provided environment variables (`GITHUB_EVENT_NAME`, `GITHUB_REF_NAME`, `GITHUB_HEAD_REF`, `GITHUB_EVENT_PATH`) inside the container. Parse pull request `action` from `$GITHUB_EVENT_PATH`; do not use `${{ github.event.action }}` in `action.yml`.

## Preferred path

The `quaveone/quaveone-client-utils` Release workflow should update this repo automatically when run with `mode=release`, `publish_docker=true`, and `set_latest=true`.

If that automation fails or a manual repair is needed, run this repo's **Release Deploy Action** workflow with the CLI version, for example `v1.0.35`.

## Manual workflow dispatch

```bash
gh workflow run release.yml \
  --repo quaveone/quaveone-deploy-action \
  --ref main \
  -f version=v1.0.35
```

The workflow will:

1. Update `action.yml` to `docker://quaveone/quaveone-cli:1.0.35`.
2. Commit the change directly to `main` when needed.
3. Create tag `v1.0.35` if it does not exist.
4. Create or update GitHub Release `v1.0.35` and mark it **Latest**.
5. Refuse to move an existing tag that points to a different commit.

## Verification

Replace `v1.0.35` and `1.0.35` with the target version.

```bash
gh api repos/quaveone/quaveone-deploy-action/contents/action.yml?ref=main --jq '.content' | base64 --decode | grep 'docker://quaveone/quaveone-cli:1.0.35'
gh api repos/quaveone/quaveone-deploy-action/releases/latest --jq '.tag_name'
gh api repos/quaveone/quaveone-deploy-action/git/ref/tags/v1.0.35 --jq '.object.sha'
```

Also confirm the CLI image exists:

```bash
docker buildx imagetools inspect quaveone/quaveone-cli:1.0.35
```

Run and wait for the action smoke workflow after `main` changes. It must prove both local metadata (`uses: ./`) and published metadata (`uses: quaveone/quaveone-deploy-action@main`) load in real GitHub Actions:

```bash
gh workflow run smoke.yml --repo quaveone/quaveone-deploy-action --ref main
gh run list --repo quaveone/quaveone-deploy-action --workflow smoke.yml --branch main --limit 5
```

## Do not finish until

- `main` contains the new CLI image version.
- The matching tag exists.
- The matching GitHub Release is marked **Latest**.
- The action smoke workflow passed on `main`, including the published `uses: quaveone/quaveone-deploy-action@main` job.
- Customer docs can safely keep using `quaveone/quaveone-deploy-action@main`.
