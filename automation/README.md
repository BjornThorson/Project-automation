# Nex P0 automation toolkit

The private Drive Active Inbox is our submission transport. All deliveries must name an allowlisted repository and use SHA-256 verified files.

## Queue and receipts

Schema 2 delivers one `ready` submission, or an `idle` marker. Schema 3 supports an append-only queue:
```json
{"schema_version":3,"status":"active","queue":[{"schema_version":2,"submission_id":"example-20261019-01","status":"ready","repository":"BjornThorson/Project-automation","base_branch":"main","files":[]}]}
```
The example is structural only: file deliveries require 1-20 valid file entries.

The router processes the first queued submission that has no receipt, one per run. Completed and failed deliveries create `automation/receipts/<submission_id>.json` on `main`, with a link to the corresponding run. A failed submission must use a new ID for retry. Queue capacity is 50 requests and the Drive export limit is 512 KiB. Archive processed entries periodically.

A queued diagnostic request uses `"kind":"diagnostic"` plus approved `"repository"`, `"base_branch"`, `"profile"` and `"ref"`, with no files. A completed diagnostic receipt means the diagnostic was dispatched; inspect its separate Actions run for the actual test outcome.

## Quality gate

`nex-reusable-quality.yml` is a reusable `workflow_call` job. It only runs when another repository installs a caller workflow:

```yaml
name: PR validation
on: pull_request
permissions:
  contents: read
jobs:
  check:
    uses: BjornThorson/Project-automation/.github/workflows/nex-reusable-quality.yml@main
    with:
      profile: npm-build
```

Profiles: `npm-build`, `npm-test`, `playwright`, `shader`, `gradle`. Tests and builds execute checked-out project scripts. No deployment secrets are passed.

## Diagnostics

`nex-diagnostics.yml` can run through its GitHub Actions `workflow_dispatch` form or through a queued `diagnostic` request. Only explicit repositories, refs and profiles are supported. Diagnostics have read-only checkout permissions and cannot deploy.

## Setup mode

`NEX_SETUP_MODE=true` commits file changes directly to authorised repositories. Otherwise, the router creates pull requests. Repo protection and GitHub App installation permissions still apply. Workflow updates may start CI or deployment jobs as soon as they land. Keep the inbox private and never stage credentials.
