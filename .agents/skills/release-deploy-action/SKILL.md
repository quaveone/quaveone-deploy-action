---
name: release-deploy-action
description: Release or repair quaveone/quaveone-deploy-action when action metadata, inputs, wrapper behavior, docs, main, tags, or GitHub Latest markers change. Do not use for ordinary CLI-only releases; the action runs quaveone/quaveone-cli:latest.
---
# Release Deploy Action

Use this workflow when the deploy action itself changes: `action.yml`, inputs, wrapper shell behavior, documentation, the `main` branch pointer, tags, or the GitHub Release **Latest** marker. The action intentionally runs `docker://quaveone/quaveone-cli:latest`, so ordinary CLI releases reach action users through the CLI Docker `latest` promotion and do **not** require a deploy-action release.

## Issue tracking before release changes

Before an AI agent modifies `action.yml`, wrapper behavior, workflows, public README/docs, or release automation, create or reuse an issue in `quaveone/quaveone-deploy-action` using `.agents/skills/create-issue/SKILL.md`. Add it to the Quave ONE project, set current iteration, assign `@me`, and move it to **In Development** when implementation starts. A pure release/promotion run that only executes an already-reviewed action release may use the originating implementation/release issue; if no issue exists and the agent will change files, create one first.

## Golden rule: CLI latest is the runtime handoff

Customers use `quaveone/quaveone-deploy-action@main`. The action metadata should stay stable and reference the promoted CLI alias:

```yaml
runs:
  using: docker
  image: docker://quaveone/quaveone-cli:latest
```

For a CLI-only release:

1. Publish the CLI fixed version.
2. Smoke-test the exact CLI release asset and versioned Docker images.
3. Promote the CLI/Docker `latest` aliases from that tested version.
4. If the CLI change affects action-used behavior or logs, run the deploy-action `Smoke Deploy Action` workflow on `main` and wait for the published `uses: quaveone/quaveone-deploy-action@main` job to pass.
5. Do not create a deploy-action tag/release and do not change `action.yml`.

## When an action release is needed

Create a deploy-action PR/release only when at least one is true:

- `action.yml` changed;
- action inputs changed;
- wrapper shell behavior changed;
- action docs changed and should be packaged in a release marker;
- `main`, tags, or GitHub Release **Latest** markers need manual repair.

## Safe order for action changes

1. Create a branch from `main`.
2. Make the action/docs/workflow change. Keep the CLI image as `docker://quaveone/quaveone-cli:latest`.
3. Run local validation:

   ```bash
   ruby -e 'require "yaml"; YAML.load_file("action.yml"); YAML.load_file(".github/workflows/smoke.yml"); YAML.load_file(".github/workflows/release.yml")'
   ! grep -En '\$\{\{ *(github|env)\.' action.yml
   test "$(ruby -ryaml -e 'puts YAML.load_file("action.yml").dig("runs", "image")')" = 'docker://quaveone/quaveone-cli:latest'
   ```

4. Open a PR. The PR smoke must pass the local `uses: ./` job.
5. Merge/update `main` only after fixed-ref/PR smoke passes.
6. Run `Smoke Deploy Action` on `main` and wait for the published-main job (`uses: quaveone/quaveone-deploy-action@main`) to pass.
7. Create or update the deploy-action release tag and mark it **Latest** only after the published-main smoke passes.

If `main` is already broken, immediately restore it to the last tested-good action commit, prove that rollback with smoke, then work on the new candidate in a separate branch.

## Action metadata safety

GitHub parses `action.yml` before the Docker container starts. Docker action metadata must not reference runner contexts such as `${{ github.* }}` or `${{ env.* }}` in `runs.env` or `runs.args`; those can make every `uses: quaveone/quaveone-deploy-action@main` workflow fail during the `Set up job` phase. Keep metadata limited to supported values such as `${{ inputs.* }}` and move event-specific logic into the shell script, using runner-provided environment variables (`GITHUB_EVENT_NAME`, `GITHUB_REF_NAME`, `GITHUB_HEAD_REF`, `GITHUB_EVENT_PATH`) inside the container. Parse pull request `action` from `$GITHUB_EVENT_PATH`; do not use `${{ github.event.action }}` in `action.yml`.

## Manual action release workflow

Use this only after the fixed-ref/PR smoke and published-main smoke are already green. Replace `v1.0.45` with the action release version and set the confirmation exactly:

```bash
gh workflow run release.yml \
  --repo quaveone/quaveone-deploy-action \
  --ref main \
  -f version=v1.0.45 \
  -f promotion_confirmation=tested-action-v1.0.45
```

The workflow will refuse to run without the matching confirmation. It will:

1. Verify `action.yml` uses `docker://quaveone/quaveone-cli:latest`.
2. Create tag `v1.0.45` at the current `main` commit if it does not exist.
3. Create or update GitHub Release `v1.0.45` and mark it **Latest**.
4. Refuse to move an existing tag that points to a different commit.

Because this workflow moves the action's GitHub Latest marker, prefer the PR path unless this is a deliberate, already-tested action release or emergency repair.

## Verification

Replace `v1.0.45` with the action release version.

```bash
gh api repos/quaveone/quaveone-deploy-action/contents/action.yml?ref=main --jq '.content' | base64 --decode > /tmp/quaveone-deploy-action-main.yml
test "$(ruby -ryaml -e 'puts YAML.load_file("/tmp/quaveone-deploy-action-main.yml").dig("runs", "image")')" = 'docker://quaveone/quaveone-cli:latest'
gh api repos/quaveone/quaveone-deploy-action/releases/latest --jq '.tag_name'
gh api repos/quaveone/quaveone-deploy-action/git/ref/tags/v1.0.45 --jq '.object.sha'
docker buildx imagetools inspect quaveone/quaveone-cli:latest
```

Run and wait for the action smoke workflow after `main` changes. It must prove both local metadata (`uses: ./`) and published metadata (`uses: quaveone/quaveone-deploy-action@main`) load in real GitHub Actions:

```bash
gh workflow run smoke.yml --repo quaveone/quaveone-deploy-action --ref main
gh run list --repo quaveone/quaveone-deploy-action --workflow smoke.yml --branch main --limit 5
```

## Do not finish until

- Action changes were tested by PR, fixed branch, tag, or SHA before `main` moved.
- `main` still uses `docker://quaveone/quaveone-cli:latest`.
- The published-main action smoke passed after `main` changed.
- Any deploy-action tag/release was created only after published-main smoke passed.
- CLI-only releases did not retarget or retag the deploy action; they only promoted CLI/Docker `latest` and, when relevant, reran action smoke.
