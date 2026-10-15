---
id: TASK-625d3bf0c9a1
type: task
slug: the-watcher-s-mirror-fetch-leaves-refs-ank-claim
title: the watcher's mirror fetch leaves refs/ank/claims alone in a clone that carries the refs/ank refspec
created: 2026-10-12T12:47:30Z
author: claude-code/opus-5.5
status: done
scope:
  - crates/ank-daemon/src/fetch.rs
  - crates/ank-cli/tests/watch.rs
blocked_by: []
done_criteria: |
  Through the binary: in a clone whose remote.origin.fetch carries +refs/ank/*:refs/ank/* and which holds a claim of its own, one ank watch --once cycle, with origin holding that clone's claim rewritten and a claim on another task, mirrors the remote claim under refs/ank/watch/origin/claims/ and leaves every ref under refs/ank/claims/ of that clone exactly as it was, the same assertion the existing negative test makes of a clone without the refspec.
criteria_by: creator
verify: [cargo-test, fmt-check]
method: tdd
proof:
  - type: test
    ref: local/64e309f77bcb@98e2515
    tree: scope/26dc2d5851e7
    criteria: 252c94e6407c
    verifier: cargo-test@15ba026d69c2
    via: verifier
  - type: test
    ref: local/e3b0c44298fc@98e2515
    tree: scope/26dc2d5851e7
    criteria: 252c94e6407c
    verifier: fmt-check@5ca6d10bcd55
    via: verifier
schema: 4
version: 3
---

## Goal

ADR-24e21cb83793 says the watcher touches no local refs/ank/claims. The mirror fetch passes an explicit refspec, `+refs/ank/claims/*:refs/ank/watch/origin/claims/*`, and git then also performs its opportunistic remote-tracking update for every configured fetch refspec the fetched refs match. In a clone carrying init's `+refs/ank/*:refs/ank/*`, that update lands on refs/ank/claims/* itself: found while working TASK-b55c33df5aa3, where a test simulating the watcher's fetch saw a remote claim appear on the local plane once the fixture clone carried the refspec. `--refmap=` (empty) is git's switch that turns the opportunistic update off.

The existing negative test, a_watching_cycle_moves_the_tracking_namespace_and_nothing_else, runs on a clone without the refspec, so it could not see this. TASK-b55c33df5aa3 makes every clone carry the refspec after its first status, so the case becomes the ordinary one.
