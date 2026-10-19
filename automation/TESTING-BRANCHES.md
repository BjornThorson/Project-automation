# Nex project testing branches

Every authorised project has a dedicated `testing` branch created from its original default branch. The central Nex Delivery Router runs from `Project-automation/main`; **all file submissions target `testing`**.

## Routing safety

The router rejects file delivery manifests with `base_branch` other than `testing`. The allowlist also hardcodes `testing` as the permitted destination for all 14 repositories. Approved submissions are committed to the testing branch via the scoped GitHub App token. No code-delivery step pushes to `main` or `master`.

The router still writes compact delivery receipts into `Project-automation/main` using its own `GITHUB_TOKEN`. This is automation audit metadata, not new project code.

GitHub Actions code checks must run on `push` to `testing`, and ideally `pull_request` to production for promotion checks. Existing CI triggers and deployment permissions require a per-repository review. Never assume tests ran just because delivery succeeded.

## Promotion

Once tests are green, compare `testing` to the project's existing default branch (`main`, or `master` for `phaser-rpg`). Open a **pull request from `testing` into the default branch** and review it before merging.

The Nex Delivery Router does not merge promotion pull requests automatically. Branch protection and required status checks must be configured in GitHub if enforcement beyond router policy is desired. No production deployment should be triggered from `testing`.

Because `testing` is a shared integration branch, compare **all** changes since its last promotion. A failed change may need reverting on `testing` before it can be safely promoted. Feature-specific branches are preferable for parallel or conflicting experiments.

## Queue migration

Before enabling this policy, pause the Active Inbox and bootstrap the `testing` branches across all 14 authorised repositories. Then update and verify the router on `Project-automation/main` as a one-time control-plane change. Only after that should the paused delivery queue be resumed with `base_branch: testing`.
