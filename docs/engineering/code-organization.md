# Code design convention

Shuyra code should make correctness, responsibilities and costs possible to
understand locally. A good abstraction reduces the knowledge needed to
maintain an invariant; it does not merely reduce line count or hide
dependencies.

Apply these principles recursively to packages, modules, source files,
types, implementations and functions. They are criteria for judging a
design, not a required arrangement of files or transition frameworks.

## Core principles

1. **Correctness before structure.** Invariants, authorization, resource
   safety, observable contracts and failure recovery must survive refactors.
   A cleaner layout or shorter critical section proves none of these.
2. **Concepts determine boundaries.** Group facts by real responsibility
   and the invariants they maintain, not by field count, source length,
   operation verb or superficial symmetry.
3. **State is explicit and local.** Keep meaningful states and legal
   transitions inspectable near their owner. Independent concerns can
   evolve independently; do not invent a universal state machine.
4. **Dependencies are direct and directional.** Use the actual owners
   required by an operation, as narrowly and visibly as possible.
   Coordinate cross-owner changes at the nearest real operation boundary.
5. **Synchronization protects invariants, not structures.** Choose
   exclusion and commit scopes for shared facts and atomicity requirements.
   Types, state machines, allocations and physical locks need not match
   one-to-one.
6. **Costs and outstanding obligations have owners.** Resource growth,
   in-flight effects, retries, cleanup and reclamation remain accountable
   after failures, cancellation and shutdown.

These principles have priorities. Correctness and resource safety are
requirements, never tradeoffs for elegance or speed. Within those
constraints, choose faithful semantic boundaries before visual symmetry,
and smaller local models before extra abstraction. A wider scope is
justified only by the invariant or contract that genuinely requires it.
Claimed performance improvements need evidence.

## Classify facts before grouping fields

Identify what each field represents before designing its container:

| Fact | Question to answer |
| --- | --- |
| Identity | What does it identify; how is it allocated and reused? |
| Authoritative state | Who can mutate it, and which invariant governs that mutation? |
| Local lifecycle | Which phases and associated data belong to this object? |
| Independent property | Can it change without participating in another transition? |
| Index or cache | Which owner supplies its truth and invalidation rules? |
| Snapshot or observation | How fresh is it, and can it be mistaken for live authority? |
| Synchronization or completion evidence | What exact exclusion, revision or event does it prove? |
| Retained obligation | Who must finish or recover it if the caller disappears? |

Each mutable fact has one authoritative representation and each invariant
one enforcing owner. Derived records, weak discovery structures and cached
snapshots must not become competing authorities. Keep state private where
possible; make every mutation path preserve the owner's invariants.

Facts that must change together can belong under one owner even when they
describe different concepts. Truly independent properties need not share
one lifecycle. One owner can contain multiple small state machines, and
multiple local machines can share one allocation or lock. Conversely,
wrapping unrelated fields in `State`, `Life` or nominal sub-managers does
not give them a new invariant.

Do not use one untyped integer for unrelated identities such as an object
ID, hardware slot and revision. Distinguish non-reusing sequences, finite
recyclable slots and generational handles; specify exhaustion and stale
reference behavior. Do not add generations unless reuse needs them.

Lifetime, authorization, publication and safe reclamation can have
different conditions. A reference count proves retained storage, not
permission to access it. Cancellation or caller completion does not prove
that asynchronous users have stopped touching a resource.

## Model transitions where they belong

Use types to represent mutually exclusive states and their associated
data, rather than independent flags that permit impossible combinations.
Keep genuinely independent properties independent; do not create an
explosive combined enum just to put everything in one machine.

A local transition should be understandable from its own state and
explicit inputs. Ordinary methods are appropriate for simple changes.
A complex rule may benefit from a side-effect-free decision, conceptually
`State × Event -> Result<Transition, Error>`, but this is not a mandated
Rust signature or a required decision/execute framework.

A decision computed from a snapshot is a proposal, not permission to
apply it later. The invariant owner validates live authority, identity,
revision and assumptions when committing under appropriate synchronization.
When effects such as page-table updates, I/O or notifications matter,
expose their executor, visibility and completion points, ordering,
partial-failure handling and retry behavior. Pure decisions do not
make hardware effects atomic.

Illegal events, duplicates, cancellation, stale decisions and terminal
states must be intentional. Partial execution requires rollback,
compensation or an explicit recoverable state. Keep states, transition
rules and the operations using them in the same semantic neighborhood.

Names must describe real completion semantics. For example,
`begin_release` is suitable only when release is admitted now and
settled later; it is not a stylistic replacement for `release`.
Prefer symmetrical operation names for genuinely dual operations, but
meaning is more important than matching word lengths.

## Local dependencies and cross-owner coordination

A local transition should not secretly acquire unrelated owners, look
up global services, wait for external effects or run arbitrary callbacks.
Use explicit inputs. An operation may hold narrow, direct references to
the actual owners it needs when their lifetime and authority conditions
are clear. Reachability alone proves neither authorization nor completion.

Avoid circular parent/manager lookups, forwarding layers that carry
unrelated data, and all-purpose `Context` or service locators. Do not
shuttle values through callers that do not use them. Equally, do not
hide live authority checks, expected revisions or completion witnesses
merely to shorten a signature.

For invariants spanning owners, identify the concrete coordinator,
participants, validation, commit point and partial-failure responsibility.
It may admit work under synchronization, prepare outside a critical
section and later settle under revalidation. This is a reasoning pattern,
not a mandatory API template.

Global address conflicts, quotas, indices and hardware domains remain
shared where correctness requires them. Sharing storage or batching
hardware work does not require centralizing every participant's local
state machine. A local operation should not scan or lock unrelated
members without a correctness or measured implementation reason.

## Synchronization and outstanding work

Choose lock boundaries for the facts that must be observed or changed
together, independently of types and source files. Use lexical scopes
and RAII guards to make protected regions visible. Check acquisition
cycles, error paths, cancellation, destruction, final references and
shutdown as well as the successful path. Locks do not themselves prove
live authority, exact identity or external completion.

When the contract permits, perform expensive or fallible preparation,
waiting, notifications, potentially cross-owner releases and unbounded
iteration outside critical sections. Revalidate after reacquisition
or retain a stated guarantee across the gap. Keep necessary atomic
commits and synchronized hardware writes protected. A lock split must
not silently alter revocation, observable atomicity or partial results.

Before an effect can escape, ensure its unfinished obligation has a
durable owner and that the needed recovery capacity is reserved.
Registries and workers may select and schedule that owner, but may not
copy its authoritative state or infer settlement from a wakeup.
Define exact operation or publication identity, repeated execution,
late or uncertain completion, retry and safe reuse. Release resources
only when matching evidence proves the relevant use is complete.

Closing admission must serialize with final publication. Cleanup must
continue if ordinary callers vanish. Do not require unbounded work
or allocation in a last destructor to honor a routine obligation.
Historical handles can retain metadata without retaining payloads
whose obligations have already settled.

## Bounds, costs and verification

For Kernel, temporary memory per operation must have a determinable
upper bound. Persistent growth needs an accountable owner and enforced
quota or capacity. Allocation failure and recovery must also be safe
and bounded. An outer batch limit does not bound an inner scan,
allocation, destructor cascade or pending-work queue.

Space bounds, elapsed time and lock duration are different properties.
Allocation-free does not imply short critical sections. Choose data
structures according to actual access patterns and complexity.
Measure locality, contention, retained space and failure behavior
before claiming an optimization.

Tests protect enduring invariants and observable contracts, not
arbitrary implementation shapes. Test consequential invalid
transitions, stale proofs, failures and races at the smallest useful
boundary. Follow the [testing convention](testing.md) for placement,
execution and lifecycle. A structural refactor preserves contracts;
an intentional contract change is declared and verified separately.

## Names and source layout

Use short, concrete names for stable domain concepts. Name operations
for their actual action and completion state. Use consistent vocabulary
and meaningful pairs; visual or character-count symmetry must not
obscure semantics.

Organize around the owner of an invariant or coherent behavior, not
generic `types/`, `impl/`, `helpers/`, `create/`, `update/` or
`delete/` buckets. Keep a type and principal methods close together.
Put a helper near its sole use; extract reusable helpers when they own
a meaningful responsibility. Avoid separating one type's methods
across files solely to reduce file size or combining unrelated
responsibilities into an enormous entry module.

Introduce modules for independent concepts, lifecycles, safety
boundaries, interfaces or verification responsibilities; not merely
because a source file is long. Apply semantic locality inside modules,
implementation blocks and functions, not only to the directory tree.

Do not put a file and directory with the same sibling stem
(`space.rs` beside `space/`), or a redundant file stem
(`space/space.rs`). For a directory-owned Rust module, prefer
`space/mod.rs` for its principal definition and add justified
submodules. Apply the corresponding convention to tests; see
[testing](testing.md). Migrate existing paths with their owners;
avoid unrelated mass renaming.

Do not impose universal file-length, module-count, method-count or
type-count limits. A large coherent owner is better than several
nominal managers; a small scattered invariant can still be a defect.

## Review questions

1. Which facts are authoritative or derived, and who enforces each
   invariant across every construction and mutation path?
2. Which states exclude one another, which change independently, and
   which invalid combinations should types rule out?
3. Can the local rule be understood from its owner and explicit inputs,
   without tracing unrelated managers or forwarding layers?
4. Which constraints really cross owners, where is the commit point,
   and who handles authorization changes or partial failure?
5. What does each critical section protect, and what remains valid
   when a guard is released? Are release and cancellation paths safe?
6. Who retains unfinished work, and what exact evidence permits
   settlement, reclamation and reuse?
7. What bounds temporary and retained resources, and which costs
   have actually been verified?
8. Does the structure improve understanding without extra wrappers,
   artificial naming symmetry or unnecessary compatibility scaffolding?

See [dependency](dependency.md) for contracts and cross-repository
migration and [repository](repository.md) for repository-level
boundaries.
