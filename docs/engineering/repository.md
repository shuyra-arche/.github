# Repository convention

## Boundaries and names

A repository represents a unit with a clear interface and an independent reason
to develop, maintain, release or verify it. Keep internal implementation in
packages or modules when it shares the same lifecycle.

Use lowercase kebab-case. Use `<cluster>-<unit>` for a repository named after a
Unit within a Cluster, such as `driver-iommu`. Standalone repositories and
projects can use their own names, such as `limen`, `vanllia` and `anallia`.
Choose names that express the repository's responsibility and avoid generic
suffixes such as `-core` or `-lib` when they add no meaning.

A Cluster describes source and build organization. A Unit describes runtime
organization. A Cluster name alone creates no requirement for a `cluster.toml`.

## Layout and documentation

| Location | Purpose |
| --- | --- |
| `README.md` | Current responsibilities, entry points, prerequisites and build and test commands |
| `AGENTS.md` | Agent startup entry for shared standards and local instructions |
| `LICENSE` | License for the repository, with any required notices and attribution |
| `.gitignore` | Rules for generated and local files, when needed |
| `.github/workflows/` | Repository-specific automation, when needed |
| `CONTRIBUTING.md` | Local contribution details, when needed |
| `docs/architecture/` | Design decisions and enduring contracts, when needed |

Use the language's normal package layout and apply the
[code organization convention](code-organization.md) recursively within every
module, file, type and function. Add source, test, example and script
directories when they have a defined purpose. Describe ownership, failure
conditions and unsafe preconditions next to the relevant API. Use Issues for
future work, PRs for change rationale and verification, and CI artifacts or
release assets for raw evidence. Check code, tests and publishing references
before removing stale documentation.

Follow the [agent guidance convention](agent.md) for the root `AGENTS.md`.
Keep shared policy in the organization standards and local guidance focused
on entry points and exceptions. Organization community-file defaults do not
automatically load agent instructions into other repositories.

## Design repositories

Use `document-<component>` when a component needs an independently maintained
design contract. Start with [template-document](https://github.com/shuyra-arche/template-document).
The component repository owns its design, dependencies and verification.
The `document` repository indexes the components and pins the versions of its
submodules. It can describe compatibility gaps and cross-component scenarios
by linking to the contracts of the actual owners.

Use the root README to explain the component and its reading order. Put
contracts beside their semantic owners. Add cross-owner scenarios and
executable models when they have concrete work to do. Record the exact
provider version when the reference is used as verification or compatibility
evidence. Keep candidate designs, Host evidence and verified implementation
status explicit.

## GitHub settings

Use `main` as the default branch. Protect it with the rules in the
[branch convention](branch.md) and use squash merge with deletion of merged
working branches. Enable Issues and use PRs for changes. Enable Discussions as
needed and keep Wiki disabled by default. Choose visibility according to the
project's release and collaboration stage.

For a new design repository, start private with `main`, squash-only merging,
and automatic deletion of merged working branches. Configure repository
settings after generation, then verify the effective values. The template flag
belongs to the starter repository. Apply branch protection when the repository's
plan supports it. Record unavailable protections and follow the review procedure.

Each code repository provides its own applicable build and verification entry
points. The integration owner verifies assembled systems. See the proposed
[testing convention](testing.md) for coverage boundaries.

## Lifecycle

| State | Meaning |
| --- | --- |
| Active | Under development |
| Maintained | Focused on fixes and maintenance |
| Deprecated | Existing users have a migration path or a documented limitation |
| Archived | Read-only historical reference |

Document deprecation and any successor before archiving. Prefer archiving retired
repositories to deleting their history.
