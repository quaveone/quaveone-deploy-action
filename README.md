# Quave ONE Deploy Action

This action deploys source code or container images to
[Quave ONE](https://www.quave.one).

The action is non-interactive and token-based. It passes inputs to the Quave ONE CLI through `QUAVEONE_*` environment variables and does not use local `quaveone login` config. Keep product behavior in zcloud public API/CLI paths; do not duplicate deploy or preview logic in the action.

## Inputs

## `user-token`

**Optional** Token to authenticate the user.

## `env-token`

**Optional** Token to authenticate the app environment.

## `env`

**Required** Environment name to create or update.

## `dir`

**Optional** Directory usage as source code to send to deploy (default value is current dir)

## `image`

**Optional** Image name to deploy.

## `app`

**Optional** Name of the app in which the environment will be created or updated (requires usage together with the --create argument).

## `copy-env-vars-from`

**Optional** Name of the environment from which the env vars will be copied to the new environment (requires usage together with the --create argument).

## `api-cli-uri`

**Optional** Custom API URL for full private region deployments.

## `cli-extra-args`

**Optional** Additional `quaveone deploy` arguments, including repeatable
`--arg` values, startup command overrides, wait options, and `--clear-command`.
Do not pass application environment variables through GitHub Actions; configure
them persistently through Quave ONE before deploying.


## Branch preview inputs

Set `preview: "true"` to run the single `quaveone preview deploy` command. It creates or reuses the preview for the source environment and branch, then deploys the checked-out source; it also forwards `dir` for monorepos. The Action derives the branch from `GITHUB_HEAD_REF` or `GITHUB_REF_NAME`. Preview URLs and deployment output appear in workflow logs.

| Input | Required when `preview=true` | Description |
| --- | --- | --- |
| `env` | No | Not used while creating/deploying a preview. For `delete-on-pr-close`, supply the exact CLI preview name printed by an earlier preview deployment. It remains required for a normal deployment. |
| `app` | Yes | App slug/name/id resolved by the CLI. |
| `from` | Yes | Source env appEnvId, cliEnvName, or display name scoped to the app. |
| `ttl-hours` | Yes | Absolute preview TTL in hours. |
| `idle-hours` | No | Idle timeout in hours. |
| `commit-sha` | No | Source commit SHA stored in the preview metadata. |
| `prevent-destroy` | No | Create the preview with Prevent destroy enabled. |
| `delete-on-pr-close` | No | On a closed pull request, call `quaveone preview delete --env`. `env` must be the exact CLI preview name printed by an earlier preview deployment; otherwise rely on the required TTL cleanup. |

Example:

```yaml
uses: quaveone/quaveone-deploy-action@main
with:
  user-token: ${{ secrets.QUAVEONE_USER_TOKEN }}
  app: my-app
  preview: "true"
  from: production
  ttl-hours: "8"
  idle-hours: "2"
  commit-sha: ${{ github.sha }}
  dir: website/
  cli-extra-args: "--wait"
```

## Example usage

```yaml
uses: quaveone/quaveone-deploy-action@main
with:
  user-token: USER_TOKEN
  env: ENV_NAME
  dir: app1
  cli-extra-args: "--wait"
```

The same image can run a different long-lived process without an entrypoint
script. Quote the complete value so spaces inside `--command` stay grouped;
shell metacharacters still need normal shell escaping:

```yaml
uses: quaveone/quaveone-deploy-action@main
with:
  env-token: ${{ secrets.QUAVEONE_ENV_TOKEN }}
  env: worker-production
  image: ghcr.io/acme/app:sha
  cli-extra-args: '--command "bun run start:worker" --wait'
```

For an image without `/bin/sh`, use direct execution and repeat `--arg`:

```yaml
cli-extra-args: >-
  --command bun
  --shell=false
  --arg run
  --arg start:worker
  --working-dir /app
  --wait
```

The override is saved on the Quave ONE environment and remains active when
later deployments omit these flags. Pass `--clear-command` to restore the
image's `ENTRYPOINT`, `CMD`, and working directory.

# Quave ONE CLI

- [Documentation](https://docs.quave.one/docs/cli/)
- [Docker](https://hub.docker.com/r/quaveone/quaveone-cli)


## Release process

Customer examples use `quaveone/quaveone-deploy-action@main`, and this action runs `docker://quaveone/quaveone-cli:latest`. CLI releases reach action users when the CLI release workflow promotes the tested Docker image to `quaveone/quaveone-cli:latest`; the action should not be retargeted or released for every CLI version.

Treat the CLI `latest` alias as customer-facing: publish a fixed CLI version first, smoke-test the exact asset/image, then promote CLI/Docker `latest`. When a CLI change affects action users (for example deploy/preview/`--wait` behavior or logs), run this repository's `Smoke Deploy Action` workflow on `main` after the CLI promotion so the published `uses: quaveone/quaveone-deploy-action@main` path proves it loads the promoted CLI.

Create a deploy-action PR/release only when `action.yml`, inputs, wrapper shell, or action docs changed. For manual repair or a standalone action release after the fixed-ref and published-main smoke tests pass, run this repository's **Release Deploy Action** workflow:

```shell
gh workflow run release.yml --repo quaveone/quaveone-deploy-action --ref main \
  -f version=v1.0.45 -f promotion_confirmation=tested-action-v1.0.45
```

The workflow tags the already-tested `main` commit and marks the matching GitHub Release as **Latest**. It does not change the CLI Docker image; action runtime follows the promoted `quaveone/quaveone-cli:latest` image. AI agents should follow `.agents/skills/release-deploy-action/SKILL.md` before declaring an action release complete.
