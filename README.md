# Quave ONE Deploy Action

This action deploys source code or container images to
[Quave ONE](https://www.quave.one).

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
`--env-var` and `--arg` values, startup command overrides, wait options, and
`--clear-command`.


## Branch preview inputs

Set `preview: "true"` to run `quaveone preview create` instead of `quaveone deploy`. The Action derives the preview branch from `GITHUB_HEAD_REF` or `GITHUB_REF_NAME`.

| Input | Required when `preview=true` | Description |
| --- | --- | --- |
| `app` | Yes | App slug/name/id resolved by the CLI. |
| `from` | Yes | Source env appEnvId, cliEnvName, or display name scoped to the app. |
| `ttl-hours` | Yes | Absolute preview TTL in hours. |
| `idle-hours` | No | Idle timeout in hours. |
| `prevent-destroy` | No | Create the preview with Prevent destroy enabled. |
| `delete-on-pr-close` | No | On a closed pull request, call `quaveone preview delete --env`. Use `env` as the stable preview env name/delete target. |

Example:

```yaml
uses: quaveone/quaveone-deploy-action@main
with:
  user-token: ${{ secrets.QUAVEONE_USER_TOKEN }}
  env: preview-${{ github.event.pull_request.number }}
  app: my-app
  preview: "true"
  from: production
  ttl-hours: "8"
  idle-hours: "2"
  delete-on-pr-close: "true"
```

## Example usage

```yaml
uses: quaveone/quaveone-deploy-action@main
with:
  user-token: USER_TOKEN
  env: ENV_NAME
  dir: app1
  cli-extra-args: "--env-var ENV1=VAL1 --env-var ENV2=VAL2"
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

Customer examples use `quaveone/quaveone-deploy-action@main`, so the `main` branch must always use the latest stable Quave ONE CLI Docker image.

The normal path is automatic: the `quaveone/quaveone-client-utils` Release workflow updates this repo after a stable CLI release when `publish_docker=true` and `set_latest=true`.

For manual repair or a standalone action release, run this repository's **Release Deploy Action** workflow:

```shell
gh workflow run release.yml --repo quaveone/quaveone-deploy-action --ref main -f version=v1.0.35
```

The workflow updates `action.yml`, pushes `main`, creates the matching tag, and marks the matching GitHub Release as **Latest**. AI agents should follow `.agents/skills/release-deploy-action/SKILL.md` before declaring the action release complete.
