# Branch convention

`main` is the only permanent development branch. Start temporary work from
`main` and use one branch per independently reviewable change.

| Prefix | Purpose |
| --- | --- |
| `feat/` | New behavior |
| `fix/` | Defect fix |
| `refactor/` | Structural change |
| `exp/` | Experiment |
| `docs/` | Documentation |
| `test/` | Verification |
| `build/` | Build system |
| `ci/` | Automation |

Use a lowercase kebab-case description, such as `feat/frame-transfer` or
`docs/engineering-standards`. A release line that needs separate maintenance
may use `release/<codename>` with a lowercase kebab-case branch suffix. Retire
the branch when that maintenance ends.

## Protection and cleanup

Make changes to `main` through PRs and require applicable checks before merge.
Protect `main` against force-push and deletion. Single-maintainer review follows
the [PR convention](pull-request.md).

Delete temporary branches after their PRs merge. Before deleting an older
branch, record its tip and check unique changes, open PRs, dependency pins,
gitlinks and workflow references. Compare the branch's changes against the
merged PR when a squash merge leaves no ancestry relationship. Preserve needed
unmerged work and keep published dependency commits fetchable.
