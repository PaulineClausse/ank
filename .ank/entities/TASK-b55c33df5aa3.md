---
id: TASK-b55c33df5aa3
type: task
slug: a-verb-that-runs-git-adds-the-refs-ank-refspec-o
title: A verb that runs git adds the refs/ank refspec origin lacks, and fetches refs/ank once
created: 2026-10-11T20:46:29Z
author: haksolot@omarchy
status: done
scope:
  - crates/ank-cli/src/git.rs
  - crates/ank-cli/src/init.rs
  - crates/ank-cli/src/status.rs
  - docs/claims.md
  - crates/ank-cli/src/human.rs
  - crates/ank-cli/tests/init_refspec.rs
  - crates/ank-cli/tests/cli.rs
  - crates/ank-cli/tests/watch.rs
blocked_by: []
done_criteria: |
  Through the binary, per ADR-1b030fe70442, in a scratch clone of a bare origin carrying a done task and its refs/ank/proof/<id> ref, with remote.origin.fetch holding only the branch refspec: the first ank status exits 0, leaves +refs/ank/*:refs/ank/* in remote.origin.fetch, prints one stderr line naming the refspec added and the fetch, and leaves this clone with the same refs/ank/* names origin holds; ank check then reports no 'done with no test proof' signal for that task and no refspec signal; a second ank status starts no git fetch, counted with GIT_TRACE at an absolute path; with origin unreachable the refspec is still written, the verb exits as it would with the refspec present, and a later git fetch brings the refs; in a repository with no origin nothing is written and git remote add origin then succeeds; ank help and ank --version start no git process, counted the same way; docs/claims.md describes the repair in a replay that the suite runs.
criteria_by: creator
verify: [cargo-test, fmt-check]
method: tdd
proof:
  - type: test
    ref: local/7c25d8d1f923@e71d74b
    tree: scope/c4a284d02674
    criteria: 51417791ce41
    verifier: cargo-test@15ba026d69c2
    via: verifier
  - type: test
    ref: local/e3b0c44298fc@e71d74b
    tree: scope/c4a284d02674
    criteria: 51417791ce41
    verifier: fmt-check@5ca6d10bcd55
    via: verifier
schema: 4
version: 6
---

Implements ADR-1b030fe70442. Measured before (fb17a26a, fresh clone of this repo): 0 refs/ank against 218 on origin, ank check 744 signals of which 214 'done with no test proof'; with the refs fetched, 531 and 0. Count processes, never time them (CLAUDE.md). The refspec is written by init::ensure_refspec today; reuse it rather than writing a second writer.
