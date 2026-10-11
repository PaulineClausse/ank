---
id: ADR-a2faf59f6ce0
type: adr
slug: done-reuses-a-verifier-result-whose-definition-a
title: done reuses a verifier result whose definition and inputs have not changed, locally or from a hermetic attestation
created: 2026-10-11T17:42:34Z
author: claude-code/opus-5.5
status: accepted
scope:
  - crates/ank-cli/src/done.rs
  - crates/ank-cli/src/verify.rs
  - crates/ank-core/src/model.rs
  - crates/ank-core/src/config.rs
constraint: |
  A verifier result is keyed by name@hash and a digest of the content on disk at execution time (tracked, or untracked and not ignored, files matching inputs, or the whole tree without inputs); every run by done except exit 9 is recorded in an unversioned registry under <git-common-dir>/ank/runs/; done may reuse only a passing local entry with the same key no older than proof.reuse_max_age (default 24h), or a passing attestation with the same key from an executor honouring env for a hermetic verifier, never another machine's local proof and never a failure; a reused entry keeps the via of the run that produced it and carries reused and by, so it is never indistinguishable from an execution for this task; done --fresh reuses nothing.
ratified: 981093c546c0
verified:
  - by: haksolot@omarchy
    at: 2026-10-11T18:24:00Z
schema: 4
version: 2
---

Depends on ADR-d8a328e311b9 (a verifier declares its execution environment):
the definition hash this decision keys on is the one that ADR extends to cover
`env` and `inputs`, and hermeticity is the property it derives.

## Context

`done` runs every verifier listed by the task, every time. On a growing
repository this makes `done` the slowest step of the loop. Most of that time is
spent re-running verifiers whose definition and inputs have not changed since
they last passed: after a `done` that failed on another verifier, or in a
sibling worktree on the same machine working on the same content.

`done` runs on the working tree as it stands, and an agent's nominal case is an
uncommitted tree (SPEC-77d99d8d1ef2, "What local proof anchors"). A key built
on a commit would name a state that was never tested, and two different dirty
trees over one HEAD would share it. So the key is taken over content on disk at
execution time, the way `tree: scope/<hash>` already is.

## Decision

1. **Result key.** A verifier result is identified by `name@hash` (the
   definition hash, which covers `env` and `inputs`) and an inputs digest,
   computed over the working tree as it stands when the verifier runs:
   - with `inputs`: a hash of the sorted list of `(path, blob id)`, blob ids in
     the `git hash-object` form of the content on disk, for every file
     matching the globs that is tracked, or untracked and not ignored;
   - without `inputs`: the same digest over every such file of the tree.
   Ignored files are never part of the digest. A verifier that depends on
   generated files has no correct `inputs` unless it also declares their
   sources; otherwise it omits `inputs` and its key is the whole tree.
2. **Local run registry.** Every verifier run by `done` is recorded, pass or
   fail, in a registry under the git common directory
   (`<git-common-dir>/ank/runs/`), never versioned. Sibling worktrees therefore
   share it. An entry holds the key, the exit code, the identity that ran it,
   and the time. Exit code 9 is not recorded.
3. **What `done` may reuse:**
   - a passing entry of the local registry with the same key, no older than
     `proof.reuse_max_age` (config, default 24h);
   - a passing detached attestation with the same key, produced by an executor
     that honours `env`, for hermetic verifiers only.
   Proofs produced by other agents on other machines are never reused: a local
   run does not claim its environment, so its result says nothing about another
   host. Failures are never reused.
4. **Recording.** A reused result is written as a proof entry like any other,
   with two additional fields: `reused` (`local`, or the ref of the
   attestation) and `by` (the identity of the run that produced it). Its `via`
   is the route of the run that produced it -- `verifier` for the local
   registry, `attested` for an attestation -- so no new route is added to the
   trust hierarchy. A reused entry is never indistinguishable from an execution
   done for this task.
5. **Escape.** `done --fresh` ignores the registry and attestations and runs
   every verifier.

## Example proof entry

```yaml
proof:
  - type: test
    ref: local/e51b22c09a4f@a3f9c21
    tree: scope/4be2d10c
    criteria: 9d0c1e7a
    verifier: auth-tests@1f2e3d4c5b6a
    via: verifier
    inputs: 7c09e1b2
    reused: local
    by: claude-code/opus-5
```

## Consequences

- The common case (fix one failing verifier, run `done` again) costs only the
  verifiers whose key changed.
- Sibling worktrees on one machine share results without any coordination.
- Without an executor that honours `env`, there is no cross-machine reuse. This
  is deliberate.
- Wrong `inputs` can produce a false local reuse. The definition hash makes it
  traceable, and an executor that mounts only declared inputs detects it.
- Environment drift on one machine (a toolchain upgrade) within
  `reuse_max_age` can produce a stale reuse. `--fresh` and the age bound limit
  it.
- SPEC-77d99d8d1ef2 says `done` runs all the verifiers of the list; it is
  amended to say what a reused entry is when this is implemented.

## Rejected alternatives

- Reusing any passing proof in the corpus: trusts another host's environment
  for this one.
- Keying on the commit's tree and tracked blobs: `done` runs on an uncommitted
  tree, so the key would name a state that was never tested, and a file an
  agent created without `git add` would escape it.
- A new `via: reused` route: the trust a reader extends rests on who ran the
  verifier, which the original route already says; `reused` and `by` carry the
  rest.
- Verifiers made incremental from changed files: the proof would depend on a
  base commit and no longer prove what it states.
- Reusing failures: prevents a plain retry and hides flakiness.
- A new `ank verify` verb as a prerequisite: may be useful later so agents get
  feedback through ank rather than raw test commands, but not needed for this
  gain.
