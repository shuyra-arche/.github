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

## Publication

1. Select the candidate revision from `main` or the applicable maintenance
   branch.
2. Run release validation on that exact revision and its locked inputs.
3. Create the release tag at the verified revision. Keep published tags fixed.
4. Record relevant changes, compatibility, migration requirements and
   verification. Update the release channel after publication.

Record the tag and commit SHA used for reproducible builds, along with package
versions where applicable. See the [dependency convention](dependency.md) for
consumer pins.
