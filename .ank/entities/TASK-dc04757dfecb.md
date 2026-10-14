---
id: TASK-dc04757dfecb
type: task
slug: a-verifier-declares-inputs-and-env-and-its-defin
title: A verifier declares inputs, its definition hash covers them, and env is refused by name
created: 2026-10-13T10:11:05Z
author: claude-code/2.x
status: open
scope:
  - crates/ank-core/src/config.rs
  - crates/ank-cli/src/config.rs
  - crates/ank-cli/src/verify.rs
blocked_by: []
done_criteria: |
  Through the binary: ank config verifiers.<name>.inputs writes inputs globs to config.yml and reads them back; the verifier definition hash changes when inputs changes and is unchanged for a verifier declaring none; ank config verifiers.<name>.env.image is refused by name as a key the parser does not know, and a config.yml carrying an env block under a verifier fails to load naming the key; check reports a changed inputs as it reports any verifier change; SPEC-fe67fbaa2708 names inputs, states what the definition hash covers, and names no env.
criteria_by: creator
verify: [cargo-test, fmt-check]
schema: 4
version: 3
---

Implements ADR-d8a328e311b9, accepted and never coded. Today `VerifierFile`
(crates/ank-cli/src/config.rs) is `deny_unknown_fields` with `run`, `timeout`
and `default` only, so an `inputs` key in config.yml is refused, and
`verify::definition_hash` covers `run` and `timeout` only.

ank keeps running verifiers on the host: `env` is parsed, hashed and
validated, never executed here (the ADR leaves that to an executor).

SPEC-fe67fbaa2708 is proposed and still says the hash covers `run` and
`timeout`; it is amended here, before it is ratified, so it does not contradict
the accepted ADR.
