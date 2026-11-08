# Code organization convention

Code organization follows **semantic locality** at every scale: Package,
module, source file, type, implementation block and function. The same
responsibility rule applies recursively, not just to top-level directories.

A reader should be able to find an object's state, invariants, valid
transitions and principal behavior together. Do not make them reconstruct
one concept from distant declarations and unrelated implementation files.

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
3. Are state transitions and the methods that apply them near the
   objects whose correctness they determine?
4. Does the layout remain understandable **inside** each module, type,
   implementation block and function, not only in the directory tree?
5. Did the change preserve observable behavior, interfaces and safety
   invariants? If it changed semantics, is that change reviewed and
   verified separately?

Do not impose universal file-length, file-count or type-count limits.
Optimize for local comprehension and stable responsibility boundaries,
not symmetry of directory shape.
