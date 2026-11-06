# Dependency convention

Cross-repository source dependencies form a directed acyclic graph. Consumers
depend on public interfaces. The composition layer selects concrete providers.

## Interface boundaries

| Component | Allowed dependencies |
| --- | --- |
| API or shared contract | Foundational types needed to express the interface |
| Client | Public API or contract and required transport |
| Server | Public API or contract and required runtime infrastructure |
| Kernel core | Kernel internals and public contracts |
| Driver | Public contracts and required runtime capabilities |

Clients and servers share a contract. A server can separately act as a consumer
of another service's public API. Kernel core must stay independent of concrete
driver implementations. If the kernel and a driver need a shared concept,
define it at the public boundary that owns it.

Separate ABI (call convention, representation and layout), service contracts
(messages and semantics) and code generation (macros and tools that emit
bindings). Keep service binding generation outside the kernel ABI package.

## Dependency kinds

| Kind | Where to declare it |
| --- | --- |
| Source | Package manifest |
| Build | Build configuration or the package's build dependency declaration |
| Runtime | Unit configuration and service contracts |
| Test | Test configuration or development dependencies |
| Development tooling | Toolchain and development configuration |

Runtime service discovery and calls can use a shared contract without adding a
source dependency on the provider implementation.

## Versioning and lock files

Prefer published packages with explicit version requirements. Pin
cross-repository Git dependencies to full commit SHAs. Treat branch names and
other movable references as development inputs only. Keep personal path
overrides in local configuration. Track lock files for
reproducibly built applications and integration projects. For libraries, use
the repository's documented lock-file policy.

Describe the public package's compatibility guarantees. After a stable
`1.0.0`, use SemVer for compatible additions, fixes and breaking changes. For
`0.x`, state the compatibility policy and still record breaking changes.
Document ABI compatibility and the migration path independently when relevant.

## Changes across repositories

Explain new dependencies in the PR, including the capability they provide, the
public boundary they use, and their effects on the build graph and release
order. Review transitive dependencies when they introduce a new boundary or
prevent independent builds.

Publish and validate the provider first. Update consumer pins and lock files,
document the minimum compatible versions, then verify the assembled revision
set. Use [linked PRs](pull-request.md) to make the order visible.
