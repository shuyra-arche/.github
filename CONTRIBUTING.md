# Contributing to Shuyra

Start with the repository's README for its responsibilities, toolchain and
verification commands. Use a [bug report](.github/ISSUE_TEMPLATE/bug_report.md)
for a reproducible problem and a [scoped proposal](.github/ISSUE_TEMPLATE/feature_request.md)
for a feature, maintenance task or coordinated change.

## Submit a change

1. Start from the current `main` and create a short-lived branch using the
   [branch convention](docs/engineering/branch.md). Keep each PR independently
   reviewable.
2. Follow the [PR convention](docs/engineering/pull-request.md). Open a draft
   while the change or its required verification is in progress.
3. Run the repository's applicable checks and record commands and results in the
   PR. Report any failures and checks you skipped, with the reason.
4. For cross-repository work, link the PRs, state the integration order and
   update consumer pins after the provider is available. Follow the
   [dependency convention](docs/engineering/dependency.md).

## Coordinate work

Give concurrent workers distinct repositories or worktrees. A handoff records
the scope, commit, interface constraints, checks and open problems. Agents follow
the same review and validation rules. Use actual check output as evidence.

Keep repository-specific `AGENTS.md` guidance focused on local entry points and
exceptions. Get task authorization for settings changes, merges, closing
unrelated work or repository retirement.

See the [engineering standards](docs/engineering/README.md) for the full policy.
