---
id: TASK-dbbf4e6316bb
type: task
slug: the-test-verifier-and-ci-run-the-suite-under-car
title: The test verifier and CI run the suite under cargo-nextest
created: 2026-10-13T10:11:06Z
author: claude-code/2.x
status: open
scope:
  - .ank/config.yml
  - .github/workflows/ci.yml
  - CONTRIBUTING.md
  - crates/ank-tui/tests/**
  - .config/nextest.toml
blocked_by: [TASK-d08e4110f6c1]
done_criteria: |
  The cargo-test verifier and the CI test job run cargo nextest run --workspace; cargo nextest list counts exactly as many tests as cargo test -- --list, both counts recorded with ank log; the ank-tui tests serialised today by an in-process lock run in a nextest test group with max-threads 1; ank done with cargo-nextest absent from PATH exits 9; CONTRIBUTING says how to install it; the CI run on the change is green on all three platforms.
criteria_by: creator
verify: [cargo-test, fmt-check]
schema: 4
version: 6
---

Applies ADR-8c12a38531e3 once it is ratified; do not claim before. The
baseline to compare against afterwards: on CI runs 38373336710 and
38279153781 the Windows test leg took 16m19s and 13m57s, with `cli.rs` at
407.6s, and a green `done` measured 114.7s (LOG-eb0b8).
