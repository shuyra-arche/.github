# Pull request convention

## Title and description

Use `<type>(<scope>): <description>`. The scope identifies the affected module
or domain. For example, `fix(iommu): prevent mapping leaks`.

| Type | Use |
| --- | --- |
| `feat` | New behavior |
| `fix` | Defect fix |
| `refactor` | Structural change |
| `exp` | Experiment |
| `docs` | Documentation |
| `test` | Verification |
| `build` | Build system |
| `ci` | Automation |
| `chore` | Maintenance |

Use the [PR template](../../PULL_REQUEST_TEMPLATE.md). Describe the purpose,
changes, verification, compatibility and related work. Use `N/A` where a section
does not apply. Review the final diff and report commands and results, including
failed or skipped checks.

## Review and merge

Open a Draft PR while work or required verification is in progress. Resolve
review discussions and pass applicable checks before merging. Use squash merge
and remove the completed working branch.

| Situation | Review |
| --- | --- |
| Single maintainer | Author reviews the final diff and verifies required checks |
| Multiple maintainers | At least one maintainer other than the author approves |
| Core API or ABI change | Explicit compatibility review and migration notes when needed |
| Security-sensitive change | Focused review of the affected trust and failure boundaries |

## Cross-repository changes

Open one PR per affected repository and link the related PRs. Record the merge
order, minimum compatible versions and any compatibility window. Land the
provider first, then update consumers against a published version or a full
commit SHA. Verify the assembled change at the smallest applicable integration
boundary.

Merge and publication are separate steps. See the
[release convention](release.md) for tagging and channel updates.
