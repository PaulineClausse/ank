---
id: ADR-5c6db7a161b8
type: adr
slug: signers-live-in-config-yml-ank-signers-writes-th
title: Signers live in config.yml, ank signers writes them, and a key is retired by date rather than removed
created: 2026-10-11T18:26:17Z
author: claude-code/opus-5.5
status: accepted
scope:
  - crates/ank-cli/src/config.rs
  - crates/ank-core/src/config.rs
  - crates/ank-cli/src/human.rs
  - crates/ank-cli/src/git.rs
  - .ank/config.yml
  - docs/signing.md
constraint: |
  Who may ratify is declared in config.yml under signers:, keyed by principal, each key carrying an optional note and an optional valid_before date, and ank signers (add, retire, remove, migrate) is its only writer, through the surgical config writer, while ank config refuses signers.*; retire sets valid_before so the key ratifies nothing dated after it and what it signed before stays verified, remove is the act for a compromised key and the faults it raises are intended; check reads signers: only, hands git a generated allowed-signers file carrying valid_before as git's valid-before option, judges gpg keys against the committer date itself, and reports a remaining .ank/allowed_signers as a fault naming ank signers migrate; ank signers is a trust act the skills reserve to a human, as they reserve accept.
amends: [ADR-e45e1a29fe91, ADR-e64dfaafd578]
ratified: 4571c324e036
verified:
  - by: haksolot@omarchy
    at: 2026-10-11T19:04:21Z
schema: 4
version: 2
---

## Context

`.ank/allowed_signers` lists who may ratify, in git's allowed-signers layout.
It is the one file under `.ank/` that the format asks a human to edit by hand
(TASK-8a80b590b356): no verb writes it, and ADR-e45e1a29fe91 closes the list of
verbs that write `.ank/` without naming one.

Measured on this repository on 2026-10-11, the cost of that:

- Four keys declared for one maintainer in under two months: a gpg key, a VPS
  ssh key, a laptop ssh key, and a second laptop ssh key after a reinstall lost
  the first. Every new machine is a hand edit and a pull request before it can
  ratify anything.
- A key cannot be retired. `check` judges every ratification against the file
  as it stands, so removing a key turns every ratification it ever signed into
  a fault. The lost laptop key therefore stays declared, able to ratify, with
  nobody holding it -- or, if somebody does, with nobody able to tell.

git's allowed-signers format already carries the retirement this lacks.
Measured with git 2.55 in a scratch repository: two commits signed by one ssh
key, committer dates 2026-09-01 and 2026-10-11, read through the same
`rev-list --format=%G?` path `check` uses. With no option both read `G`. With
`valid-before="20261001"` the first reads `G` and the second `U`; with
`valid-after="20261001"` the reverse. git compares the committer date against
the bound.

## Decision

1. **Signers live in `config.yml`**, under `signers:`, keyed by principal. Each
   principal carries a list of keys; each key carries the key itself (`gpg
   <fingerprint>` or `<ssh key type> <base64>`), an optional `note`, and an
   optional `valid_before` date. A principal may carry a `github:` login, which
   `ank signers sync` reads (decided separately, filed beside this one).

   ```yaml
   signers:
     sean.lamet@dekrow.com:
       github: haksolot
       keys:
         - key: gpg 739A603FB05F9F2F7D3C8D50624FCFCC1482554A
         - key: ssh-ed25519 AAAA...QGI6
           note: VPS
         - key: ssh-ed25519 AAAA...K4ul
           note: omarchy, lost on reinstall
           valid_before: 2026-10-11
   ```

2. **`ank signers` is the only writer of `signers:`**, through the surgical
   writer `config.yml` already has (ADR-e64dfaafd578): comments, order and
   quoting survive. `ank config` refuses `signers.*` and names `ank signers`.
   - `ank signers add` declares the key `user.signingkey` names (or `--key
     <file>`), under the principal `user.email` names (or `--principal`), with
     an optional `--note`.
   - `ank signers retire <key>` sets `valid_before` (today, or `--date`): the
     key ratifies nothing dated after it, and what it signed before stays
     verified.
   - `ank signers remove <key>` deletes the key. That is the act for a
     compromised key, and the faults it raises on what the key signed are the
     point: the committer date is chosen by whoever signs, so a stolen retired
     key can still sign a backdated commit that `retire` would accept.
   - `ank signers migrate` moves `.ank/allowed_signers` into `signers:`,
     carrying its comments into `note`, and deletes the file.
3. **`check` reads `signers:` only.** It hands git a scratch allowed-signers
   file generated from it, `valid_before` written as git's `valid-before`
   option, and judges `gpg` entries itself against the committer date. A
   `.ank/allowed_signers` still present is a fault naming `ank signers migrate`.
   The four outcomes of section 8 and the regime ADR-964be4d940b2 states are
   unchanged.
4. **`ank signers` is a trust act, and the skills say so** as they say it of
   `accept`: an agent proposes, a human runs it. The tool refuses on state and
   never on who is asking.

This amends ADR-e45e1a29fe91 (the writing verbs gain `signers`) and
ADR-e64dfaafd578 (the key set gains `signers`, written by `ank signers` and not
by `ank config`). Both stay binding for the rest.

## Consequences

- A new machine is `ank signers add` and a pull request, with no file opened.
- A lost key is retired without breaking the history it ratified.
- SPEC-77d99d8d1ef2 section 8 names `allowed_signers` as the key file; it is
  amended to name `signers:` when this is implemented, and `docs/signing.md`
  with it.
- git's floor is 2.34 (`git.rs`); whether `valid-before` is honoured that far
  back has not been measured, and the implementing task measures it on the
  three platforms.

## Rejected alternatives

- Keeping `allowed_signers` and adding a verb that writes it: a second file of
  configuration with its own format, comments and parser, for no gain over
  the writer `config.yml` already has.
- A flat list of git allowed-signers lines in `config.yml`: keeps the format
  git reads but leaves the options as quoted strings nobody edits safely.
- Reading both sources during a transition: two truths about who may ratify.
- `retire` as the answer to a compromised key: the committer date is the
  signer's to choose.
