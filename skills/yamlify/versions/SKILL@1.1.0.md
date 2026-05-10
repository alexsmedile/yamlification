---
name: yamlify
description: >
  Convert markdown files into compact, AI-optimized YAML representations.
  Use this skill whenever a user wants to "yamlify" a markdown file, compress
  context for an AI, convert .md to YAML for use as AI context, or asks for a
  "lean", "minified", "max", or "delta" YAML version of any document. Also
  trigger when the user says a markdown file is "too bloated", "too many words",
  or wants to reduce token usage. Default output is lean unless specified.
version: 1.1.0
category: content
status: published
tags: [yaml, markdown, compression, agent-context]
---

# Yamlify

Convert markdown into compact, structured, machine-readable YAML for fast
retrieval and agent consumption. Four modes: `lean` (default), `minified`,
`max`, `delta`.

The transformation is **restructuring, not summarization** — all meaning
preserved, format discarded. Keys are semantic labels. Values are dense
payloads. Comments carry nuance that doesn't fit a value cleanly.

---

## Modes

### `lean` — default

~40–65% token reduction. All signal, no noise. Comments carry nuance.

```yaml
# lean
auth:
  mechanism: JWT bearer token
  token_expires_after: 24h
  refresh_expires_after: 30d
  public_endpoints: [/health, /login]
  # Refresh token rotation policy not yet decided

open_questions:
  refresh_rotation: undecided  # security vs UX tradeoff unresolved
```

### `minified`

~70–85% token reduction. Mid-session reminder only — not for project start.
Goal + active constraints + next action. No comments unless load-bearing.

```yaml
# minified
goal: JWT auth for internal APIs
auth:
  mech: bearer token
  token_ttl: 24h
  refresh_ttl: 30d
  public_eps: [/health, /login]
next: implement refresh endpoint
```

### `max`

~15–25% token reduction. Full sentences in values. Best for nuance-critical
specs where compression would lose meaning.

```yaml
# max
auth:
  mechanism: JWT bearer tokens issued at login and validated on every request.
  token_expires_after: 24 hours. Chosen to balance session convenience with security exposure.
  refresh_expires_after: 30 days. Single-use, rotated on each redemption.
  public_endpoints:
    - /health  # liveness probe — must stay unauthenticated for load balancer
    - /login   # entry point — auth happens here, not before
  # Rotation policy under discussion: rotate-always vs rotate-on-suspicion

open_questions:
  refresh_rotation:
    status: undecided
    options: [rotate-always, rotate-on-suspicion]
    blocker: security review 2025-06-01
```

### `delta`

Tracks changes between two YAML versions or markdown sources. Always cite
related documents in a `refs` block. Include only non-empty sections.

```yaml
# delta
refs:
  base: auth-spec.lean.yaml
  source: auth-spec-v2.md

changed:
  status: in_progress → complete
  owner: TBD → platform-team
  sla: sub-30s → 2min  # revised after load testing

added:
  retry_logic: {max_attempts: 3, backoff: [1m, 5m, 30m], on_exhaustion: page on-call}

removed:
  - open_question: refresh_rotation  # resolved — rotation not required v1
```

---

## Conversion Process

**1. Classify** — identify document type; it drives compression depth and schema shape.

| Type | Approach |
|---|---|
| Spec / ADR | High compression. Map sections directly to keys. |
| README | Medium. Keep purpose, install, constraints. Drop marketing prose. |
| Meeting notes | High. Map source sections to keys. Compress prose to dense values; drop discussion narrative. Preserve section order. If a thematic restructure (e.g. decisions/open_questions/actions) would significantly improve agent scannability, propose it — but default to source structure unless the user confirms. |
| Config / reference | Low. Preserve structure; collapse prose descriptions. |
| Narrative / essay | Do not convert silently — tell the user it's argument-first and ask if they want conclusions-only YAML. If yes, mark output as partial and reference the source. |

**2. Design the key schema** — see [references/schema.md](references/schema.md).
One level of nesting per logical grouping. Keys name concepts, not sentences.
Values are atomic. Rationale goes in comments.

**Key naming** — varies by mode:
- `lean` / `max`: prefer self-documenting names — `token_expires_after` not `token_ttl`, `max_retries` not `max_att`, `created_at` not `ts`. Exception: abbreviations so universal they need no gloss — `id`, `url`, `api`, `os`, `cpu`, `db`. When in doubt, spell it out.
- `minified`: technical abbreviations are acceptable — space is the constraint, not readability. `ttl`, `ts`, `cfg`, `env`, `dep` are fine when the domain is clear.

**3. Decide what to cut** — see [references/cut-keep.md](references/cut-keep.md).
Heuristic: would an agent acting on this YAML need to know it? Yes → keep. No → cut.
Keep TBD only when tied to an owner, deadline, dependency, or explicit open question — otherwise cut it. Lost uncertainty is worse than lost facts, but empty placeholders are noise.

**4. Fill values** — densest accurate value possible. Unresolved items → `open_questions`.
Ambiguity → `field: TBD  # brief note`.

**Abbreviations** — prefer common abbreviations when meaning is unambiguous: `TBD`, `TBC`, `WIP`, `ETA`, `N/A`, `TLDR`, `POC`, `OOO`, `EOD`, `v1`, etc. Never spell out what an abbreviation already says clearly.

**Empty fields and placeholders** — when converting a template or partially-filled document:
- Omit keys with no content and no structural significance.
- If the key matters, signal the expected type explicitly:
  - string → `name: ""`
  - list → `tags: []`
  - map → `config: {}`
- Add a comment hint when an example adds real value: `name: ""  # e.g. Acme Corp`.

**5. Add metadata if needed** — see [references/metadata.md](references/metadata.md).
Default: `meta.source` only if filename is known. All values from source or user — never invented.

**6. Output** — label mode as top comment (`# lean`). For delta, verify base is available first — if it isn't, tell the user and ask them to provide it before proceeding.

**7. Frontmatter** — if the source has YAML frontmatter, absorb relevant fields into `meta:` (e.g. `version`, `status`, `updated` → `date`). Drop structural/system fields (`id`, `layer`, `alias`, `lang`) unless they carry meaning for a downstream agent.

---

## References

- [Schema design + machine-readability rules](references/schema.md)
- [What to cut and keep + comment guide](references/cut-keep.md)
- [Metadata fields and examples](references/metadata.md)
- Conversion examples by document type: `examples/` at repo root (not loaded into context)
