# Agent guidance convention

Every `shuyra-arche` repository must have a short root `AGENTS.md` that loads
the shared standards and identifies local entry points and exceptions.
The standards remain authoritative in this repository; consumer repositories
should not copy the full policy text.

## Session startup

At each new session, before substantive analysis or edits:

1. Read the working repository's root `AGENTS.md` and `README.md`, then check
   for applicable nested instructions before working in a subtree.
2. Resolve the current `main` revision of `shuyra-arche/.github` and read
   `CONTRIBUTING.md`, `docs/engineering/README.md`, this guide,
   `code-organization.md` and `testing.md` from that same revision.
3. Read task-relevant standards from the index: `repository.md` for layout
   or lifecycle work, `branch.md` and `pull-request.md` for changes and
   publication, `dependency.md` for dependencies, and `release.md` for
   releases. Read local build and verification instructions as applicable.

When working on `.github` itself, use the local checkout instead of remote
`main`. Within one session, reuse already-read guidance unless it changes or
the task enters a different scope. A new session must resolve and read it
again; memory of an earlier conversation is not evidence of current policy.

A link is a retrieval instruction, not the contents of the linked file.
Actually read the files through an available connector, Git checkout or
authenticated API. Keep the source revision available for handoff or review.
If current guidance cannot be retrieved, use a known cached revision when
available, state its revision and the limitation, and do not claim the latest
rules were read. Continue work that does not depend on the missing guidance.

These repository instructions operate within the agent runtime's instruction
hierarchy and the user's authorized task. They do not grant permission for
unrelated changes or override explicit user instructions.

## Consumer root entry

Use the following as the starting content of a consumer repository's root
`AGENTS.md`, then add only actual local entry points or exceptions:

```markdown
# Agent entry point

This repository follows the shared standards in `shuyra-arche/.github`.

At the start of every new session, before substantive analysis or edits:

1. Resolve the current `main` revision of
   https://github.com/shuyra-arche/.github and read, at that revision:
   - `CONTRIBUTING.md`
   - `docs/engineering/README.md`
   - `docs/engineering/agent.md`
   - `docs/engineering/code-organization.md`
   - `docs/engineering/testing.md`
2. Follow the startup procedure in `agent.md` and read task-relevant
   standards from the index; changes and publication require `branch.md`
   and `pull-request.md` from the same revision.
3. Read this repository's `README.md` and applicable nested instructions.

Read actual file contents; do not substitute links or previous-session
summaries. If retrieval fails, report the limitation and any cached revision
used. Continue work independent of missing guidance without claiming it was read.
```

## Runtime integration

Codex discovers instructions at startup from its global configuration and
the project root down to the working directory. A root `AGENTS.md` therefore
provides the repository entry point; an `AGENTS.override.md` at the same
level takes precedence. See the
[Codex instruction discovery reference](https://developers.openai.com/codex/guides/agents-md).

The organization `.github` repository is not an ancestor of other repository
checkouts. Its `AGENTS.md` is not automatically inherited by those repositories.
Each consumer needs its own root entry. Adding a template here does not deploy
it to existing repositories.

For other agent clients, configure their supported startup instruction
mechanism to read the repository's root `AGENTS.md`. For chat sessions without
a checkout, put an explicit instruction in the project's persistent settings:

> For each new session working on Shuyra, read
> `https://github.com/shuyra-arche/.github/blob/main/AGENTS.md` and the files
> it requires before substantive analysis or edits. Without a local checkout,
> resolve `.github/main` once and read all those paths at that revision.
> Also read the target repository's root `AGENTS.md` and local guidance when
> working on that repository. Report any retrieval limitation.

A personal Codex installation can additionally place an instruction scoped
to `shuyra-arche` work in `~/.codex/AGENTS.md`. This is machine-local setup;
it does not replace repository entries used by collaborators or hosted agents.

## Verify the entry point

Start a fresh session in the target repository and ask the agent to list the
instruction files it actually read, the shared standards revision, and the
rules applicable to a small proposed change. Check the recorded reads when
available; an assertion of compliance alone is insufficient. Test a nested
working directory if the repository uses nested instructions or overrides.

Do not claim that writing this file configures every client or proves future
compliance. Reload or start a new session after changing startup instructions.
Automated checks can enforce file presence and link validity, but cannot prove
that an agent understood and applied the rules.
