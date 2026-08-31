# Template format

One file per concept. The body is written for a person to read top to bottom.
The YAML frontmatter and the fenced-block metadata exist so an agent can parse
the same file without a second source of truth.

## Why one file and not two

A separate machine format drifts from the prose within a month, and the prose
is the part people actually correct. Keeping both in one file means an edit to
the lesson is an edit to the spec.

## Frontmatter fields

| Field | Required | Meaning |
| --- | --- | --- |
| `id` | yes | Stable slug. Never reused, never renamed. |
| `concept` | yes | The thing being taught, in plain words. |
| `role` | yes | Job family this appears in. |
| `language` | yes | Language of the code rungs. |
| `prerequisites` | yes | Concept ids a reader needs first. Empty list is allowed. |
| `taxonomy` | yes | Skill names from a public taxonomy. IDs stay `TBD` until verified against the source, so an unverified mapping is never mistaken for a real one. |
| `rungs` | yes | Ordered list. See below. |
| `sources` | yes | URLs backing every factual claim in the body. |
| `verified` | yes | Date the code and the doc claims were last checked. |

## The five rungs

The ordering is not stylistic. Dropping a novice into a large codebase to learn
a basic construct is the approach with the most evidence against it, so each
rung removes context the reader does not have yet.

1. `minimal` — smallest correct usage. No dependencies, no framework, runnable.
2. `idiomatic` — how a working developer writes it, shown against the naive
   version so the difference is visible rather than asserted.
3. `excerpt` — real code from a named repository with everything unrelated
   stubbed out. Must state what was removed.
4. `in_situ` — the unedited code where it lives, with a reading path. Name files
   and symbols, not line numbers; line numbers rot.
5. `failure` — what breaks at scale and what production does instead.

Rung 5 carries most of the value. Every existing resource stops at rung 3.

## Rules for rung 3 and 4

- Name the repository, the file, and the symbol. Never a line number.
- Quote only what the concept needs. Say what was cut.
- If a claim about library behaviour appears, it goes in `sources` with a link.
- Check licences before copying a substantive block. Attribution and licence go
  in the source entry.

## Checks

Every rung ends with a check the reader can run or answer. A rung without a
check is a paragraph, not a lesson.
