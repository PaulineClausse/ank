---
id: TASK-68db19c9ed93
type: task
slug: ank-accept-takes-several-ids-and-pr-carries-them
title: ank accept takes several ids, and --pr carries them to a pull request
created: 2026-10-11T18:26:57Z
author: claude-code/opus-5.5
status: open
scope:
  - crates/ank-cli/src/human.rs
  - crates/ank-cli/src/git.rs
  - crates/ank-contract/src/verbs.rs
  - docs/ratifying.md
blocked_by: []
done_criteria: |
  Through the binary, per ADR-2a63c0c87691, in a scratch repository with a bare origin and gh replaced on PATH by a stub recording its invocations: ank accept A B on the default branch writes two commits whose subjects are ratify A and ratify B and pushes nothing; ank accept A B --pr additionally leaves branch ratify/A at the second commit on origin, the local default branch equal to its upstream, exactly one gh pr create with --base set to the default branch and --head ratify/A, no gh pr merge, and prints gh pr merge ratify/A --merge; with no upstream, with gh absent or unauthenticated, or with a tracked file outside .ank/ modified, --pr refuses before any commit with the documented code and the tree unchanged; with the push refused by origin, the commits remain on ratify/A and the message names the command that finishes the job; docs/ratifying.md leads with accept --pr and keeps the manual recipe.
criteria_by: creator
verify: [cargo-test, fmt-check]
method: tdd
schema: 4
version: 1
---

Implements ADR-2a63c0c87691. Independent of the signers tasks.
