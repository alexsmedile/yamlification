---
name: yamlify
version: 1.2.0
description: >
  Convert markdown files into compact, AI-optimized YAML representations.
  Use this skill whenever a user wants to "yamlify" a markdown file, compress
  context for an AI, convert .md to YAML for use as AI context, or asks for a
  "lean", "minified", "max", or "delta" YAML version of any document. Also
  trigger when the user says a markdown file is "too bloated", "too many words",
  or wants to reduce token usage. Default output is lean unless specified.
category: content
status: published
tags: [yaml, markdown, compression, agent-context]
---

# Yamlify

Convert markdown into compact, structured, machine-readable YAML for fast
retrieval and agent consumption. Four modes: `lean` (default), `minified`,
`max`, `delta`.

## Core Philosophy

Markdown is human-optimized: full sentences, prose, headers, redundancy.
YAML context is AI-optimized: structured keys, minimal repetition, high signal.

The transformation is **restructuring, not summarization** — all meaning
preserved, format discarded. Use keys as semantic labels. Use values as
dense payloads. Use YAML comments (`#`) to preserve nuance that doesn't fit
cleanly into a value.

---

## Modes

### `lean` — default

~40–65% token reduction. All signal, no noise. Comments carry nuance.

- Analyze the markdown first: identify its semantic structure (what are the
  real "properties" it's describing?)
- Map each concept to a YAML key. Use short, lowercase, snake_case keys.
- Write values as concise phrases or short sentences — not full prose.
- Drop: filler phrases, restatements, transitional sentences, decorative prose,
  redundant headers.

**Transformation example:**

```markdown
## Summer Trip — Lisbon

We're thinking of going to Lisbon in late July, probably for about ten days.
Flights from Milan look reasonable right now. We'd like to stay somewhere central,
ideally walkable to the main sights. Budget is around €2,500 total for two people,
though we could stretch a bit if accommodation is really good. Main thing to sort
out first is the flights — we're flexible on exact dates.
```

```yaml
# lean
trip:
  destination: Lisbon
  dates: late July, ~10 days
  travelers: 2
  budget: €2,500  # flexible if accommodation warrants it
  base: central, walkable to sights
  priority: book flights first  # dates flexible
```

**Output example (technical):**

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

~70–85% token reduction. Not suitable for project start, but excellent for keeping a long-running AI session on track.

- Strong use of summaries
- Collapse nested structures to single lines where possible.
- Abbreviate values aggressively — drop adjectives, drop examples, keep nouns
  and verbs.
- No comments unless they carry load-bearing information.
- Target: 20–35% of original token count.

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

### `full`

~15–25% token reduction. Allow fuller sentences in values. Best for nuance-critical
specs where compression would lose meaning.

- Nested objects and lists encouraged to preserve hierarchy.
- Comments used generously to preserve authorial intent and edge cases.
- Still no markdown prose — no headers-as-text, no transitional sentences.

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

### Step 1: Read and analyze the markdown

Before writing any YAML:
- Identify the document type (spec, README, guide, notes, config doc, etc.) it drives compression depth and schema shape. You can do quick search of best corresponding doc type in [references/class.md](references/class.md).
- Identify the real semantic structure: what are the top-level *things* being described?
- Note any sections that are pure filler vs. load-bearing content.

### Step 2: Design the key schema

Good and bad examples here: [references/schema.md](references/schema.md).

Map the document's concepts to YAML keys:
- Use the document's own terminology, shortened. (`authentication` → `auth`)
- Group related concepts under parent keys.
- Lists of items → YAML sequences.
- Key/value pairs in prose → YAML mappings.
- Prefer one level of nesting per logical grouping.
- Keys name concepts, not sentences.

**Key naming** — varies by mode:
- `lean` / `full` (`max`): prefer self-documenting names — `token_expires_after` not `token_ttl`, `max_retries` not `max_att`, `created_at` not `ts`. Exception: abbreviations so universal they need no gloss — `id`, `url`, `api`, `os`, `cpu`, `db`. When in doubt, spell it out.
- `minified`: technical abbreviations are acceptable — space is the constraint, not readability. `ttl`, `ts`, `cfg`, `env`, `dep` are fine when the domain is clear.

### Step 3: Decide what to cut

See [references/cut-keep.md](references/cut-keep.md).
*Heuristic:* would an agent acting on this YAML need to know it? Yes → keep. No → cut.

### Step 4: Fill values

- Densest accurate value possible.
- When a value needs context that can't fit: add a `# comment`.
- Unresolved items → `open_questions`.
- Ambiguity → `field: TBD  # source unclear, two options mentioned`.

**Abbreviations** — prefer common abbreviations when meaning is unambiguous: `TBD`, `WIP`, `ETA`, `N/A`, `TLDR`, `v1`, etc. Never spell out what an abbreviation already says clearly.

For minified: optionally use inline flow style (`{key: val, key2: val2}`).

**Empty fields and placeholders** — when converting a template or partially-filled document:
- Omit keys with no content and no structural significance.
- If the key matters, signal the expected type explicitly:
  - string → `name: ""`
  - list → `tags: []`
  - map → `config: {}`
- Add a comment hint when an example adds real value: `name: ""  # e.g. Acme Corp`.

**Metadata – if needed** — see [references/metadata.md](references/metadata.md). Default: `meta.source` only if original filename is known. All values from source or user — never invented.

**Comments** — first-class tool, not an afterthought. Four uses:
1. **Clarify a value**: `deadline: 2026-06-01  # hard cutoff, not soft`
2. **Preserve dropped nuance**: if cutting a paragraph loses something an agent needs — rescue it as a comment
3. **Flag uncertainty**: `owner: marketing  # unclear if brand or growth team`
4. **Capture intent behind a decision**: `format: one-page  # requested short for busy readers, not because content is thin`

**Frontmatter** — if the source has YAML frontmatter, absorb relevant fields into `meta:` (e.g. `version`, `status`, `updated` → `date`). Drop structural/system fields (`id`, `layer`, `alias`, `lang`) unless they carry meaning for a downstream agent.

### Step 5: Output

- Always label the output with the mode name as a comment at the top (`# lean`).
- For delta, verify base is available first — if it isn't, tell the user and ask them to provide it before proceeding.
- File output: `filename.md` → `filename.yaml` clean by default.
- If producing multiple versions "all-three” OR request is mode-specific append mode in filename:
  - “minify this” → `filename.min.yaml`
  - “preserve content” → `filename.full.yaml`

---

## When the User Specifies a Mode

| User says | Produce |
|---|---|
| "yamlify this" | lean only |
| "lean yaml" | lean only |
| "minify" / "reminder version" / "small yaml" | minified only |
| "max" / "full yaml” | full only |
| "all three" / "all versions" | lean + min + full, labeled |

---

## References

- [Common classification + yamalify approach](references/class.md)
- [Schema design + machine-readability rules](references/schema.md)
- [What to cut and keep + comment guide](references/cut-keep.md)
- [Metadata fields and examples](references/metadata.md)
- Conversion examples by document type: `examples/` at repo root (not loaded into context)
