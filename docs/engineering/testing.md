# Testing convention

Status: Draft for review.

## Verification boundaries

| Scope | What it verifies |
| --- | --- |
| Repository | Its code, public interfaces and declared build targets |
| Cluster | A contract or behavior shared by a selected group of providers |
| System | Composition, boot and behavior that depend on the assembled system |

Use the repository's declared toolchain and check entry points. Code
repositories cover applicable builds, unit tests, public interfaces and static
checks in PR CI. Documentation repositories check links and templates. Require
a CI check after it has run successfully with its real command. Keep Actions
permissions minimal and pin third-party Actions to full commit SHAs.

A Cluster test owns a cross-provider behavior and exercises only the Units and
fixtures needed to establish it. System integration belongs to the composition
owner. Use full-system coverage for behaviors that need it, such as a complete
boot path.

## Minimal integration scenario

1. State the observable behavior and failure condition.
2. Select the Unit under test and the smallest set of Units needed by its
   runtime contracts. Include infrastructure required by the scenario, such as
   System when it starts the Units from initrd.
3. Use fixtures only where they preserve the contract being exercised.
4. Record the selected Units, revision pins, toolchain and commands. Build the
   scenario using the integration project's declared entry point.
5. Assert the intended behavior and report the result in the PR.

## Evidence

Distinguish host test results, target compilation, QEMU execution and physical
hardware execution. Report which layer ran, what it established, and which
checks were skipped or failed. Keep raw logs in CI artifacts or release assets.
Preserve assertions when fixing a failure.
