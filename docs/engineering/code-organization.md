# Code organization convention

Code organization follows **semantic locality** at every scale: Package,
module, source file, type, implementation block and function. The same
responsibility rule applies recursively, not just to top-level directories.

A reader should be able to find an object's state, invariants, valid
transitions and principal behavior together. Do not make them reconstruct
one concept from distant declarations and unrelated implementation files.

## Keep invariant ownership explicit

**Fragmented invariant ownership** means that a correctness rule depends on
several callers remembering an agreement, with no clear owner
that enforces it. This is an architectural problem even when the directory
tree looks modular. Its symptoms include duplicated authoritative records,
unrelated identities represented by the same integer, independently mutable
fields that must change together, and cleanup that relies on a particular
caller sequence.

Align the semantic boundary with the state representation, transition
interface and synchronization boundary:

- Give each mutable fact one authoritative representation and each invariant
  an explicit enforcing owner. Shared references can refer to that owner;
  copied records must not become competing authorities. A snapshot is an
  observation with a defined freshness contract, not live authorization.
- Represent mutually exclusive states and their associated data together.
  Use distinct types for values with different meanings, such as object
  identity, hardware slot and revision. Independent concerns can remain
  separate; moving arbitrary fields into a `State` or `Life` wrapper does
  not establish an invariant.
- Keep state private where possible and route every mutation path through
  the enforcing interface. When separating a pure decision from effects,
  specify what must be revalidated before application and who handles
  partial failure or rollback. State the synchronization that prevents
  observers from seeing an invalid intermediate state.
- Make identity allocation and reuse policies explicit. Distinguish a
  non-reusing sequence, a recyclable finite pool and a generational handle;
  specify exhaustion and stale-reference behavior. Add a generational
  allocator only when actual slot reuse and handle validation require it.
- Model lifetime, authority, publication and safe reclamation separately
  when their conditions differ. A reference count proves storage is still
  retained, not that access remains authorized. Cancellation or business
  completion alone does not prove asynchronous work can no longer touch
  a resource; name the evidence required before reclamation.
- For an invariant spanning several owners, identify the coordinating
  operation and its commit, failure and concurrency rules. One semantic
  owner does not require one global manager or one global lock.

For example, a permission record should own revocation state rather than
requiring every copied permission view to be updated. A hardware address-space
slot that can be reused should not silently double as the stable identity
of a software object. Fix the ownership and transition model before moving
files or adding wrappers.

## Explicit state modeling and functional transitions

When correctness depends on an object's changing state, make the legal states,
events and transitions explicit enough to inspect independently of the
operations that trigger them. Prefer a **functional style for transition
decisions**: describe what a transition means before performing its effects.
This is a modeling preference, not a requirement to make every function pure.

- Represent mutually exclusive states and their associated data with types,
  such as enum variants, rather than combinations of flags that admit
  impossible states. Keep genuinely independent properties independent; do not
  invent a single state machine where several concerns have distinct owners.
- Where a transition has nontrivial rules, prefer a decision function whose
  inputs are the current state, an event and any required read-only context.
  Its output should describe the next state, rejection or intended effects.
  Conceptually, `State × Event → Transition`. The decision should not mutate
  shared state or perform I/O. An ordinary method is fine for simple,
  self-contained changes; do not create a transition framework for its own sake.
- Keep authorization and commitment with the owner of the invariant. A
  transition computed from a snapshot is a *proposal*, not permission to apply
  it later. Revalidate assumptions, identities and revisions at the commit
  boundary under the required synchronization. Specify how concurrent events,
  retries and stale decisions are resolved.
- Separate decisions from effects when doing so improves reasoning. Effects
  such as page-table changes, hardware commands, notifications and I/O have
  ordering and failure semantics that a pure function cannot enforce. Name
  their executor, required preconditions, completion evidence and the point
  at which the new state becomes authoritative. Handle partial execution with
  rollback, compensation or an explicit recoverable intermediate state.
- Make rejected events, repeated events, cancellation and terminal states
  intentional. Do not silently accept an illegal transition or infer success
  merely because an external operation was requested. Verify important legal
  and illegal transitions, including races and failures, at the smallest
  meaningful boundary.
- Keep state representations, transition decisions and commit operations
  close to their semantic owner. A pure helper may be extracted when it makes
  the rule easier to test or understand, not to manufacture layers of
  indirection or a universal event-processing abstraction.

For a nontrivial state machine, a useful design sketch is
`decide(&State, Event) -> Result<Transition, Error>`, followed by an
owner-controlled commit that validates and executes the proposed transition.
This sketch is not a mandatory Rust signature or a promise of atomic hardware
effects. Choose the smallest implementation that keeps the invariant and
failure boundary visible.

## Group by ownership and invariants

- Organize around the object or mechanism that owns a state transition,
  enforces an invariant or provides a coherent interface. Do not divide
  code merely into declarations versus implementations, or into generic
  `types/`, `impl/`, `helpers/`, `create/`, `update/` and `delete/` buckets.
- Keep a type and its principal methods close together. Put a state
  representation, transition rules and the operations that use them in
  the same semantic neighborhood. Prefer explicit states and transitions
  when they clarify correctness; do not hide state changes in scattered
  methods.
- Within a file, place related definitions and implementations together,
  followed by tightly related helpers. Within an implementation block,
  group operations by the behavior or invariant they maintain. Within a
  function, keep each transformation understandable at its own scale.
- Locate a helper near its sole use. Extract a shared helper only when
  it has a meaningful, reusable responsibility; avoid forwarding layers
  and re-export chains whose only purpose is to satisfy a layout.
- Keep an object responsible for its own state and invariants. If a
  manager accumulates methods that really belong to its members, move
  those methods to their semantic owners before splitting files.

## Module boundaries

A new module or file needs a reason beyond file length or a one-to-one
mapping from type names to filenames. Split when an independent concept
has meaningful state, invariants, lifecycle, interface or verification
responsibility. Conversely, combine tiny fragments that must always be
read together to understand a single concept.

Do not solve an oversized module by moving arbitrary groups of methods
into separate files with additional `impl` blocks for the same type.
Do not solve scattered code by indiscriminately combining everything
into an enormous `mod.rs`. First identify the actual owners and
boundaries, then choose the smallest clear layout.

Distinct architecture backends, safety boundaries or generated
implementations may justify separating a type's behavior; keep the
connection and rationale discoverable.

## Naming and layout

Choose short, concrete module names that describe one responsibility.
Avoid mechanical combinations of multiple concepts in filenames such as
`mapping_control_record.rs` or `source_authorization.rs` when the
type and its behavior can live under a coherent parent concept. An
established multiword term is acceptable when splitting it would change
its meaning; naming must not obscure the domain model.

Avoid duplicate directory/file stems such as `space/space.rs`.
For a directory-based Rust module, prefer `space/mod.rs` for the
principal `Space` definition and behavior, with meaningful submodules
only where justified. Do not create a file solely to hold a trivial
declaration and another file solely to implement it.

Keep a module's entry point readable: declare its own principal
responsibility, expose only necessary names and make navigation to
real submechanisms straightforward. Avoid both deep wrapper layers
and a flat collection of unrelated files.

## Review questions

Before introducing or moving code, check:

1. Can a reader find this concept's state, invariants and behavior
   without searching across unrelated files?
2. Does each extracted module own a genuine responsibility, rather
   than a syntactic category, operation verb or arbitrary slice?
3. Who enforces each invariant, and can any construction or mutation path
   bypass that owner? Are transitions near the state they govern?
4. Does the layout remain understandable **inside** each module, type,
   implementation block and function, not only in the directory tree?
5. Did the change preserve observable behavior, interfaces and safety
   invariants? If it changed semantics, is that change reviewed and
   verified separately?
6. Which invalid combinations are excluded by types, and which need runtime
   checks? Are failure, cancellation, stale references and exhaustion handled
   at the relevant ownership boundary?
7. For nontrivial state changes, are decisions, commit authority and effects
   distinguishable? What invalidates a proposed transition before commit?

Do not impose universal file-length, file-count or type-count limits.
Optimize for local comprehension and stable responsibility boundaries,
not symmetry of directory shape.
