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
| Repository | Local logic, public contracts, and declared build targets | Native language tests and static checks |
| Cluster | Behavior shared by selected providers or repositories | Minimal cross-provider scenario |
| System | Composition, boot, and runtime behavior of the assembled system | Targeted QEMU or hardware scenario |

A Cluster is a source/build organization boundary, not a runtime Unit. Its
integration tests select the Units needed to establish a contract. Tests belong
with the Package or integration owner responsible for the behavior, not in a
central directory merely to mirror the source tree. A Package does not need a
`tests/` directory unless it has tests that benefit from one.

Test internal algorithms locally when they have independent, high-risk
invariants. Prefer public boundaries for durable behavioral tests, so internal
refactoring does not invalidate them.

## Minimum integration scenario

1. State one observable property or invariant and how a violation will be detected.
2. Choose the smallest set of Units that can exercise it. Include infrastructure
   such as System only when required to start or connect those Units.
3. Replace external dependencies with fixtures only if the behavior under test
   remains real. Do not mock away the contract being verified.
4. Use the owning integration project's existing entry point; record selected
   Units, dependency revisions, toolchain, and execution command.
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
