# Testing convention

Status: Draft for review.

Tests are a mechanism for controlling change risk, not a measure of code volume.
Shuyra protects important behavior while keeping architecture free to evolve.
No repository, Package, or Unit is required to have a test for every function,
module, or API. Coverage percentages are not universal merge criteria.

## What deserves a lasting test

Distinguish three kinds of verification:

| Kind | What it protects | Lifecycle |
| --- | --- | --- |
| Invariant | A property that must remain true across implementations, such as isolation or correct resource ownership | Retain across refactors |
| Contract | Observable behavior agreed at an API, ABI, service, or composition boundary | Revise when the contract intentionally changes |
| Exploration | A hypothesis, prototype, or mechanism whose behavior is still being designed | Verify temporarily; promote only when justified |

An invariant can outlive a Package, Unit, or repository layout. A contract may
change deliberately without requiring compatibility with an abandoned design.
Exploratory verification need not become a permanent CI obligation.

Maintain a test when it protects a plausible, consequential failure and can be
run and understood at reasonable cost. Prioritize memory safety, permission
boundaries, isolation, resource lifetime, failure recovery, and previously
observed regressions. An important defect fix should normally gain a regression
test at the smallest practical boundary.

A new function, module, Package, or Unit does not automatically require a new
test. Simple wrappers, getters, and implementation details usually need none.
When an important risk cannot yet be automated, record the available verification
and the remaining gap instead of adding a misleading test.

## Verification boundaries

A test belongs to the owner of the invariant, public contract, or product
behavior it verifies—not automatically to the tool that builds or executes it.
The owner maintains its guest or fixture, assertions, supported execution
conditions, and regression history. A consumer can own a contract test involving
several providers without taking ownership of those providers' private tests.
A product configuration owner can own acceptance of the assembled product.

| Scope | Test owner | Preferred form |
| --- | --- | --- |
| Repository | Package or provider responsible for the behavior | Host tests, target compilation, and focused target execution |
| Cross-provider contract | Consumer or contract owner responsible for the observable boundary | Minimal scenario using selected real providers |
| Product | Product configuration or acceptance owner | Boot and runtime scenarios against the selected assembled artifact |

A Cluster is a source/build organization boundary, not a runtime Unit.
It may select multiple providers for a test without becoming the semantic
owner of their internal invariants.

**Test ownership, execution tooling, and CI triggering are distinct.**
Vanllia may discover provider-owned test declarations, select their dependency
closure, build and run them, and preserve execution evidence. This does not
transfer their scenario definitions, assertions, or Hart/profile matrices to
Vanllia. Vanllia tests its own CLI, builder, resolver, runner, and platform
Packages as their real owner; it must not maintain a central catalog of
other repositories' private mechanism tests. A new provider test using an
existing public execution contract should not require changing Vanllia.

## Test code placement

All executable test code, test fixtures, and test-only support code must live
under a `tests/` directory in the owning Package or composition repository.
Production source directories contain production code only.

Keep test bodies, fixtures and test-only support outside `src/`. When a
consequential private invariant cannot be exercised through an existing public
boundary, the owning production module may declare a test module using
`#[cfg(test)]` and `#[path = "..."]`. The path must resolve to a source file
under the owning Package's `tests/` directory. The declaration and an optional
`#[cfg(test)] extern crate std;` at the library root may live in `src/`.
Do not place inline test bodies or `src/**/tests.rs` in production source.
A Package needs a `tests/` directory only when it has a test to maintain.

Organize each test tree by domain and behavior, not by individual source file.
Keep test suites traceable to actual semantic owners in production source where
useful: a single-owner test normally sits under the analogous test subtree,
while a cross-owner invariant belongs with the owner that enforces it or the
composition boundary that owns the contract. A lasting test protects behavior,
not the current path of its implementation. Repositories need not use identical
Server source layouts or test suite names.

Prefer a stable test entry per genuine compilation or execution boundary.
Group ordinary tests in the suite's Rust module tree by their semantic
owners; adding a test case or domain within that boundary should not
require changing the Package manifest. Separate test targets are justified
by independent build, linking, fixture or execution requirements, not
merely by a new module or source file. Use the native test harness and
its module-path filters rather than a custom test registry.

Do not use the same stem for a file and a sibling directory:
`tests/memory.rs` beside `tests/memory/` is not an accepted layout.
Use `mod.rs` for a directory-owned module, and place related cases beside it:

```text
server/
├── src/
│   └── ...
├── Cargo.toml
└── tests/
    ├── contracts.rs
    ├── memory/
    │   ├── mod.rs
    │   ├── mapping.rs
    │   ├── ownership.rs
    │   └── fault.rs
    ├── ipc/
    │   ├── mod.rs
    │   ├── transfer.rs
    │   └── communication.rs
    └── support/
        └── mod.rs
```

For example, `tests/contracts.rs` declares `mod memory;` and
`mod ipc;`. Their `mod.rs` files declare the actual cases. Cargo can
discover the root automatically, or the Package can declare one stable
target when automatic discovery would also find independent fixtures
or tools:

```toml
[[test]]
name = "contracts"
path = "tests/contracts.rs"
```

If the Package itself is rooted at `server/tests/Cargo.toml`, use
the corresponding path relative to that manifest. A standalone
`tests/abi.rs` is acceptable when it protects a distinct boundary
and no sibling `tests/abi/` directory exists. Do not create wrapper
files merely to satisfy Cargo's auto-discovery rules.

Use ordinary nested Rust modules for deep source-aligned suites. An
explicit nested entry or an independent test Package is appropriate when
there is a genuine separate compilation boundary, such as incompatible
fixture roots, build scripts or target settings. Keep fixtures under
`tests/`, document genuine suite isolation, and make focused commands
and CI selection discoverable. A suite configured with `[[test]]` is
a host test unless it is part of a separately configured target-only
guest Package. Do not create empty directories or mirror the entire
source tree for symmetry.

Cross-repository execution is not a reason to move the test into
Vanllia. Place scenarios with the actual contract or product owner, even
when they need providers from several repositories. For example:

```text
pipe/
└── tests/
    └── roundtrip/
        ├── Cargo.toml
        ├── unit.toml
        ├── test.toml
        └── src/
            └── main.rs

system/
└── server/
    └── tests/
        └── target/
            └── handoff/
                └── ...
```

The first is a provider-owned test Unit discoverable through Vanllia's
existing declarations; the second represents owner-specific target tests
which may require a different execution path. Neither example mandates a
universal layout, test manifest, or runner for every kind of verification.
Product-wide acceptance belongs with the product configuration or
acceptance owner, even if it invokes Vanllia as its builder and executor.
Do not create a parallel top-level `integration/` directory solely to
reclassify tests already owned elsewhere.

## Benchmark ownership

Benchmarks measure cost under declared workloads and execution conditions;
they do not replace invariant or contract tests. Measurement bodies and
benchmark-only fixtures belong under `benches/` in their owning Package
or composition repository. Tests of measurement infrastructure remain
under `tests/`. Shared execution tooling may live with the test runner;
separate CI entry points do not require separate machine implementations,
workspaces or dependency locks.

Organize benchmark workloads by the mechanism or behavior that owns them.
Use the chosen toolchain's native build-target discovery. A shared executable
entry may provide both a correctness behavior and a measurement behavior
when it owns the same execution boundary. Select the actual behavior
explicitly; run each purpose with independently prepared state. Do not
duplicate the entry, its machine declaration, or its discovery metadata
merely to distinguish those purposes. Separate images when machine topology,
initial state or resource retirement requires isolation; keep independent
measurement series together when they genuinely share that boundary. Preserve raw observations and record the workload,
environment and implementation revision separately. A change to an
unrelated workload should not invalidate an otherwise comparable series.
Do not claim hardware performance from emulator timing or infer correctness
from a performance threshold. Run routine performance measurements only
when explicitly useful; benchmark infrastructure changes still need
appropriate verification.

## no_std and Rust test execution

Production libraries keep `#![no_std]` unconditionally. Do not use
`#![cfg_attr(not(test), no_std)]` to make the library become `std` during
testing. A library unit-test build may import `std` explicitly under `cfg(test)`
for its external test modules while keeping the production library `no_std`.

Cargo integration-test executables under `tests/` use the host test harness
and may use `std`. The library under test is compiled as a normal dependency,
so it retains its production `no_std` configuration. This allows host tests
to exercise public behavior without changing the production crate's runtime
requirements.

Integration tests cannot access private implementation details. Verify those
details through public behavior where possible. Do not expose private items
solely to satisfy a test. If a complex algorithm deserves an independent public
boundary for real production reasons, it may be extracted into a separate
`no_std` Package and tested through that boundary.

For a Package supporting both host verification and freestanding targets,
keep the ordinary Cargo test target usable on the host. Do not force a
bare-metal target for the entire shared workspace merely for convenience.
Select hardware targets explicitly in their build and verification commands;
preserve target-specific flags. A bare-metal guest does not use the host
`libtest` harness.

Host tests do not validate privileged instructions or target hardware behavior.
For target-only behavior, place a standalone `no_std` guest test package under
`tests/target/` when needed. Its entry point, panic handler, allocator, linker
configuration, and result reporter belong to that test package, not to the
production library. Run it in QEMU or on hardware. Do not introduce a guest
runner before a real target-only scenario requires one.

If the repository requires every automated test to be physically under
`tests/`, disable Rust documentation tests for the relevant library target
with `doctest = false`; keep executable examples in the test tree instead.

## Minimum integration scenario

1. State one observable property or invariant and how a violation will be detected.
2. Choose the smallest set of Units that can exercise it. Include infrastructure
   such as System only when required to start or connect those Units.
3. Replace external dependencies with fixtures only if the behavior under test
   remains real. Do not mock away the contract being verified.
4. Use the scenario owner's entry point, which may invoke Vanllia.
   Record the chosen Units, exact provider revisions, toolchain, and execution
   command. Keep business-specific assertions at that owner.
5. Assert the result and preserve enough evidence to reproduce a failure.

Do not introduce a mandatory test DSL, manifest, orchestration service, or
`cluster.toml` solely to describe scenarios. Extend Vanllia's build and test
selection only when repeated scenarios demonstrate a concrete need.

## Execution and CI

Run verification according to cost and affected risk:

| Stage | Typical checks | Trigger |
| --- | --- | --- |
| Fast | Formatting, applicable static checks, target compilation, and focused host tests | Every PR |
| Affected scenario | Owner-defined contracts and cross-provider checks; minimal QEMU scenario when needed | PRs that alter the affected behavior or boundary |
| Full | Longer boot paths, multi-platform scenarios, recovery, and hardware tests | Scheduled verification or release preparation |

A high-impact change must not bypass relevant verification merely because it
is expensive. Select the smallest adequate check; if it cannot run before merge,
state the unverified risk and obtain an explicit review decision.

A provider's high-risk PR must verify its affected contract against the
actual candidate revision before merge; a product's older pinned result is
not substitute evidence. Keep scenario selection at its owner and execution
reuse at the tool boundary; do not embed a provider-specific test matrix in
Vanllia's CI.

Do not make every PR boot the full OS. Keep required checks deterministic and
bounded; move unrelated or costly coverage out of the fast stage. Code
repositories use their declared toolchain and check entry points. Documentation
repositories check applicable links and templates. Require a CI check only after
its real command has succeeded. Keep Actions permissions minimal and pin
third-party Actions to full commit SHAs.

Distinguish host execution, target compilation, QEMU execution, and physical
hardware execution: none is evidence that the others succeeded.

A recorded execution verdict must distinguish **pass**, **fail**, **timeout**
and **inconclusive** (insufficient or ambiguous evidence). A build, guest marker,
console text, successful launcher exit, or a partially observed shutdown is not
a substitute for the scenario owner's complete assertions and required native
completion/retirement receipts. Missing, truncated, conflicting or unverifiable
evidence must never become a pass; an owner may define a narrowly justified
inconclusive retry policy only if the original attempt and its result remain
visible. A retry that passes does not erase a failed first attempt.

## Test lifecycle during changes

| Change | Expected action |
| --- | --- |
| Implementation changes; invariant and contract remain | Preserve behavioral tests; adjust harnesses only if necessary |
| Contract intentionally changes | Update or replace contract tests and record the new behavior |
| Feature or tested property disappears | Remove obsolete tests |
| Multiple tests protect the same failure | Keep the clearest, cheapest adequate protection |
| Experimental design is abandoned | Discard its temporary verification |
| A test is flaky | Investigate, isolate if needed, and report the coverage gap |

A failing test is evidence to investigate, not an instruction to preserve the
old design. Determine whether the implementation violated a valid property,
the contract changed, or the test became obsolete. Never delete a valid safety
invariant merely to make CI pass. If verification is temporarily disabled,
make the missing protection visible.

For example, merging Kernel Flat/Nest into Space should not force rewriting
a test that proves frames cannot be released while still owned. A test asserting
the former internal page-table layout can be removed if that layout is no longer
a requirement. If Frame transfer semantics change, replace the tests of the
former contract while keeping the underlying ownership safety checks.

## Pull-request evidence

Use the existing PR Verification section; no separate test report is required.
For a meaningful behavior change, describe:

- Which invariant or contract was affected, if any.
- What was actually run and the result, including relevant Unit selection.
- What was skipped or could not be verified, and why.
- Which lasting tests were added, updated, consolidated, or removed when relevant.

Raw logs belong in CI artifacts or release assets, not permanent documentation.
The objective is enough evidence to make a change reviewable, not maximum test
count. Test infrastructure itself is subject to the same maintenance standard.
