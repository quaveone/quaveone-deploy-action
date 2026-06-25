# Quave One Deploy Action

This action deploy content on [Quave One](https://www.quave.one) platform

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

**Optional** CLI extra arguments.

## Example usage

```yaml
uses: quaveone/quaveone-deploy-action@main
with:
  user-token: USER_TOKEN
  env: ENV_NAME
  dir: app1
  cli-extra-args: "--env-var ENV1=VAL1 --env-var ENV2=VAL2"
```

# Quave One CLI

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
