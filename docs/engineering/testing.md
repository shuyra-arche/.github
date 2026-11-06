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

| Scope | Responsibility | Preferred form |
| --- | --- | --- |
| Repository | Local logic, public contracts, and declared build targets | Host integration tests and target compilation |
| Cluster | Behavior shared by selected providers or repositories | Minimal cross-provider scenario |
| System | Composition, boot, and runtime behavior of the assembled system | Targeted QEMU or hardware scenario |

A Cluster is a source/build organization boundary, not a runtime Unit. Its
integration tests select the Units needed to establish a contract. Tests belong
with the Package or integration owner responsible for the behavior. Cross-repository
and system tests belong to the repository that composes the relevant providers
or system.

## Test code placement

All executable test code, test fixtures, and test-only support code must live
under a `tests/` directory in the owning Package or composition repository.
Production source directories contain production code only.

Do not place test modules or test bodies in `src/`. In particular, do not use
inline `#[cfg(test)] mod tests`, `src/**/tests.rs`, or test-only fixture
modules in production source. A Package does not need a `tests/` directory
unless it has a meaningful test to maintain.

Organize each test tree by domain and behavior, not by individual source file.
Use a top-level Rust test file as the suite entry point and a same-named
subdirectory for its child modules:

```text
server/
├── src/
│   └── ...
└── tests/
    ├── memory.rs
    ├── memory/
    │   ├── mapping.rs
    │   ├── ownership.rs
    │   └── fault.rs
    ├── ipc.rs
    ├── ipc/
    │   ├── transfer.rs
    │   └── communication.rs
    └── support/
        └── mod.rs
```

For example, `tests/memory.rs` declares `mod mapping;`, `mod ownership;`,
and `mod fault;`. Cargo treats each `.rs` file directly under `tests/` as
an integration-test target; nested files are modules of that target, not
independent targets. Shared support belongs under `tests/` and is included only
by the suites that use it. Do not create empty directories or mirror the entire
source tree.

For cross-repository or system scenarios, use the same convention in the
composition owner:

```text
vanllia/
└── tests/
    ├── cluster.rs
    ├── cluster/
    │   └── kernel/
    │       ├── memory.rs
    │       └── memory/
    │           └── transfer.rs
    ├── system.rs
    └── system/
        ├── boot.rs
        └── recovery.rs
```

The owning test entry point composes the nested modules. Do not create a
parallel top-level `integration/` directory for test code.

## no_std and Rust test execution

Production libraries keep `#![no_std]` unconditionally. Do not use
`#![cfg_attr(not(test), no_std)]` to make the library become `std` during
testing.

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
4. Use the owning test suite's existing entry point; record selected Units,
   dependency revisions, toolchain, and execution command.
5. Assert the result and preserve enough evidence to reproduce a failure.

Do not introduce a mandatory test DSL, manifest, orchestration service, or
`cluster.toml` solely to describe scenarios. Extend Vanllia's build and test
selection only when repeated scenarios demonstrate a concrete need.

## Execution and CI

Run verification according to cost and affected risk:

| Stage | Typical checks | Trigger |
| --- | --- | --- |
| Fast | Formatting, applicable static checks, target compilation, and focused host tests | Every PR |
| Affected scenario | Contract and cross-provider checks; minimal QEMU scenario when needed | PRs that alter the affected behavior or boundary |
| Full | Longer boot paths, multi-platform scenarios, recovery, and hardware tests | Scheduled verification or release preparation |

A high-impact change must not bypass relevant verification merely because it
is expensive. Select the smallest adequate check; if it cannot run before merge,
state the unverified risk and obtain an explicit review decision.

Do not make every PR boot the full OS. Keep required checks deterministic and
bounded; move unrelated or costly coverage out of the fast stage. Code
repositories use their declared toolchain and check entry points. Documentation
repositories check applicable links and templates. Require a CI check only after
its real command has succeeded. Keep Actions permissions minimal and pin
third-party Actions to full commit SHAs.

Distinguish host execution, target compilation, QEMU execution, and physical
hardware execution: none is evidence that the others succeeded.

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
