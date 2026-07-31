---
name: create-issue
description: Create or project-track GitHub issues for Quave ONE repositories and the shared Quave ONE project board. Use when the user asks to open an issue, file a ticket, add work to the board, move an issue/task through project statuses, or before an AI agent modifies code/public docs without an existing issue. New issues must include a complete description plus an implementation-ready checklist in the issue body so agents can later implement or test item by item.
---

# Create Issue

Create a GitHub issue in the repository whose files will change, then add that issue to the shared **Quave ONE** project board at `https://github.com/orgs/quaveone/projects/3`.

Canonical policy lives in `zcloud-ws/zcloud` at `internal-docs/ai/issue-first-workflow.md`; this repo mirrors the commands needed to follow it locally.

> **Important:** The issue belongs in the target repo (`zcloud-ws/zcloud`, `zcloud-ws/zcloud-infra`, `quaveone/quaveone-client-utils`, `quaveone/cli`, or `quaveone/quaveone-deploy-action`). Do not create a `zcloud-ws/zcloud` issue for a change that will be committed in another repo. Always add the resulting issue URL to the single Quave ONE project (`quaveone`, project number `3`). GitHub Projects accepts issue URLs from other organizations, so `zcloud-ws/*` issues can still be tracked there without moving repos.

## Mandatory issue-first trigger

Before an AI agent modifies code, tests, scripts, workflows, configuration, generated files, release automation, public docs, README customer instructions, examples, changelogs that customers read, or any other customer-facing artifact in a Quave ONE repo, create or reuse a GitHub issue first. For work the agent will implement immediately, the issue must be:

- in the repo whose files will change;
- added to the Quave ONE project;
- assigned to the current logged-in GitHub user (`gh api user --jq .login`, or `--assignee "@me"`);
- set to the current project iteration;
- moved to **In Development** before implementation starts;
- populated with a complete description and `## Execution checklist`.

Exception: no issue is required for internal-only AI-agent documentation/workflow-maintenance changes such as `AGENTS.md`, `CLAUDE.md`, `.agents/skills/**`, `.claude/skills/**`, `.cursor/**`, repo-local `.ai/**`, or internal AI docs, as long as the branch does not also change product code, workflows, config, public docs, or customer-facing files.

## Target repositories

| Area | GitHub repo | Default reviewer for PRs |
| --- | --- | --- |
| Main app, MCP, public docs, website | `zcloud-ws/zcloud` | `filipenevola` |
| Infra, operators, manifests, cluster tooling | `zcloud-ws/zcloud-infra` | `edimarlnx` |
| Private CLI source/release automation | `quaveone/quaveone-client-utils` | `filipenevola` |
| Public CLI installer/release repo | `quaveone/cli` | `filipenevola` |
| Public deploy GitHub Action | `quaveone/quaveone-deploy-action` | `filipenevola` |

## Issue body requirements

Every issue created by this skill must be ready for later implementation or QA. Do not create an issue with only prose unless the user explicitly asks for a text-only note; even exploratory issues should have a discovery/decision checklist.

Build the body with:

1. **Full description:** context, problem, desired outcome, constraints, links, screenshots, Slack/GitHub references, and known non-goals.
2. **Acceptance criteria:** bullets that define the observable behavior or decision needed.
3. **Execution checklist:** an `## Execution checklist` section with concrete, checkable steps and proof expectations.
4. **Testing/QA expectations:** include UI/API/infra/manual checks and screenshot requirements when relevant.
5. **Project tracking:** include a board/status item when the issue is actionable or will move through implementation.

Use this template for actionable work:

```markdown
## Description

<Complete problem statement, context, constraints, links, and non-goals.>

## Acceptance criteria

- <Observable behavior or outcome>
- <Required compatibility, UI, API, docs, or operational expectation>

## Execution checklist

- [ ] (TODO) (PLAN) Confirm scope, acceptance criteria, impacted areas, and owner repo(s).
- [ ] (TODO) (CODE) Implement <specific behavior or change>. Proof: <focused test/check>.
- [ ] (TODO) (TEST) Add or update regression coverage for <behavior>.
- [ ] (TODO) (DOCS, if needed) Update public/internal docs for <changed behavior>.
- [ ] (TODO) (VERIFY) Run <command, UI/API proof, staging check, or manual verification>.
- [ ] (TODO) (REVIEW) Run review closeout and resolve accepted findings.
- [ ] (TODO) (PR) Open PR and link it here.
- [ ] (TODO) (BOARD) Keep Quave ONE project status/comment in sync with the lifecycle step.
```

Adapt the checklist to the issue. Remove irrelevant items and add concrete steps for migrations, API docs, infra split, rollout, cleanup, or customer evidence. For non-ready discovery issues, use checklist items such as `(RESEARCH)`, `(DESIGN)`, `(DECISION)`, and `(FOLLOW-UP)` instead of fake implementation tasks.

## Hardcoded project IDs (do NOT look these up unless they fail)

| Resource | Value |
| --- | --- |
| Project owner | `quaveone` |
| Project name | Quave ONE |
| Project number | `3` |
| Project URL | `https://github.com/orgs/quaveone/projects/3` |
| Project node ID | `PVT_kwDODx0MXs4BfBKc` |
| Status field ID | `PVTSSF_lADODx0MXs4BfBKczhZXJxo` |
| Iteration field ID | `PVTIF_lADODx0MXs4BfBKczhZXL84` |

### Status options

| Status | Option ID |
| --- | --- |
| Deprioritized | `0a70e862` |
| Shaping | `494009bb` |
| Betting Table | `1b158a94` |
| Ready for Design | `f75ad846` |
| In Design | `9fbfdcde` |
| Design Review | `74436942` |
| Backlog | `191f9fd9` |
| Ready for Development | `a264972f` |
| Awaiting | `abd365a1` |
| Timing | `02a31019` |
| In Development | `47fc9ee4` |
| In Review | `da31bef0` |
| In Staging | `776d9f11` |
| Ready to Prod | `c6075b28` |
| Done | `98236657` |

## Lifecycle defaults

- Immediate AI implementation: **In Development** + current iteration + assign `@me`.
- Actionable issue not starting now: **Ready for Development**.
- Captured but not ready: **Backlog** or **Shaping**.
- PR open: **In Review**.
- Staging verification underway: **In Staging**.
- Staging passed and production is next: **Ready to Prod**.
- Production/main shipped and verified: **Done**.

## Current iteration helper

Query the current iteration dynamically; do not hardcode a soon-stale iteration ID.

```bash
TODAY=$(date +%F)
ITERATION_JSON=$(gh api graphql -f query='query {
  organization(login: "quaveone") {
    projectV2(number: 3) {
      fields(first: 50) {
        nodes {
          ... on ProjectV2IterationField {
            id
            name
            configuration { iterations { id title startDate duration } }
          }
        }
      }
    }
  }
}')

read -r ITERATION_FIELD_ID ITERATION_ID ITERATION_TITLE <<EOF_ITERATION
$(ITERATION_JSON="$ITERATION_JSON" TODAY="$TODAY" python3 - <<'PY'
import datetime, json, os
payload = json.loads(os.environ['ITERATION_JSON'])
today = datetime.date.fromisoformat(os.environ['TODAY'])
for field in payload['data']['organization']['projectV2']['fields']['nodes']:
    if field and field.get('name') == 'Iteration':
        for iteration in field['configuration']['iterations']:
            start = datetime.date.fromisoformat(iteration['startDate'])
            end = start + datetime.timedelta(days=iteration['duration'])
            if start <= today < end:
                print(field['id'], iteration['id'], iteration['title'])
                raise SystemExit
raise SystemExit('No current Quave ONE iteration found')
PY
)
EOF_ITERATION
```

## CLI workflow

```bash
TARGET_REPO="<owner>/<repo>"
STATUS_OPTION_ID="47fc9ee4" # In Development for immediate AI implementation
BODY_FILE="/tmp/quave-one-issue-body.md"

ISSUE_URL=$(gh issue create --repo "$TARGET_REPO" \
  --title "<title>" \
  --body-file "$BODY_FILE" \
  --assignee "@me" | tail -n 1)

ITEM_ID=$(gh project item-add 3 --owner quaveone --url "$ISSUE_URL" \
  --format json --jq .id)

gh api graphql -f query='mutation($itemId: ID!, $optionId: String!) {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_kwDODx0MXs4BfBKc"
    itemId: $itemId
    fieldId: "PVTSSF_lADODx0MXs4BfBKczhZXJxo"
    value: { singleSelectOptionId: $optionId }
  }) { projectV2Item { id } }
}' -f itemId="$ITEM_ID" -f optionId="$STATUS_OPTION_ID"

gh api graphql -f query='mutation($itemId: ID!, $iterationId: String!) {
  updateProjectV2ItemFieldValue(input: {
    projectId: "PVT_kwDODx0MXs4BfBKc"
    itemId: $itemId
    fieldId: "PVTIF_lADODx0MXs4BfBKczhZXL84"
    value: { iterationId: $iterationId }
  }) { projectV2Item { id } }
}' -f itemId="$ITEM_ID" -f iterationId="$ITERATION_ID"
```

If `gh project item-add` fails, first refresh/verify GitHub CLI project scope and repo/project permissions:

```bash
gh auth refresh -s project
```

If assignment to `@me` fails in a cross-org repo, create or edit the issue without the assignee and add a short comment naming the pilot user. Do not use a duplicate zcloud issue as a workaround for project permissions or assignment limitations.

## Existing issues

For an existing issue, get the issue URL, add it to the project with `gh project item-add 3 --owner quaveone --url "$ISSUE_URL" --format json --jq .id`, then use the same GraphQL mutations above to set status and iteration. If `item-add` reports the item already exists, query the issue's project items and select `project.number == 1`.

## Notes

- Add issues only to the Quave ONE project at `https://github.com/orgs/quaveone/projects/3` unless Filipe explicitly says otherwise.
- For immediate AI implementation, set **In Development**, current iteration, and `@me` before editing files.
- Keep issue bodies concise but complete; never omit `## Execution checklist`.
- If a repo-specific issue template exists, satisfy it and still include `## Execution checklist`.
