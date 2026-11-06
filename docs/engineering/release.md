# Release convention

## Release identity

| Release | Git tag format | Example |
| --- | --- | --- |
| Non-stable | `YY.MM.DD-CHANNEL-GREEK` | `26.11.06-explore-Alpha` |
| Stable | `YY.MM.DD-CODENAME` | `26.11.15-Orion` |

Use the actual publication date in `YY.MM.DD`. Non-stable channels use the
lowercase names `explore`, `develop` and `preview`. Within a series, advance
through the Greek alphabet in order using full English names with an initial
capital, from `Alpha` through `Omega`. The stable tag uses its codename as the
sole suffix after the date.

A release tag identifies the published Git revision. Cargo package versions
carry their own compatibility semantics. Record the association when publishing
both.

## Cargo packages

Repository release tags and Cargo package versions are independent. Do not
require a release tag to match a Cargo workspace or package version. Packages
may share a workspace version when they intentionally share a compatibility
lifecycle, but a repository release does not require every package version to
change.

Maintain each package's compatibility policy in its manifest and public
contracts. A GitHub source release does not require publishing packages to a
Cargo registry. Set `publish = false` for packages not intended for registry
publication. If publishing packages, validate the publishable dependency graph
and publication order separately, and record the published package names,
versions, registry and corresponding source tag/commit.

## Publication

1. Select the candidate revision from `main` or the applicable maintenance
   branch.
2. Run release validation on that exact revision and its locked inputs.
   Validate the tag's date, channel and suffix independently of Cargo versions.
3. Create the release tag at the verified revision. Keep published tags fixed
   and protect release tags against updates or deletion.
4. Record relevant changes, compatibility, migration requirements and
   verification. Update the release channel after publication.

Record the tag and commit SHA used for reproducible builds, along with package
versions where applicable. See the [dependency convention](dependency.md) for
consumer pins.
