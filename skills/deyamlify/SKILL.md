---
name: deyamlify
description: >
  Convert agent-optimized YAML back into clean, readable markdown documents.
  Trigger this skill whenever a user wants to "deyamlify", expand a YAML
  context file, generate a human-readable version of a yamlified document,
  produce a report or spec from YAML, share agent context with a human, or
  turn a compressed YAML back into a proper document. Also trigger when the
  user says "make this readable", "write this up as a doc", "expand this
  YAML", or "turn this into a spec/README/report". The output is not a
  reconstruction of the original markdown — it is a fresh, well-written
  document derived from the YAML's structured content.
---

# Deyamlify

Convert agent YAML back into clean, human-readable markdown. The output is
a freshly written document — not a mechanical YAML dump, but proper prose
that a human can read, share, and act on.

---

## Core Philosophy

YAML is optimized for agents: dense, structured, low-token. Markdown is
optimized for humans: scannable, contextual, narratively coherent.

Deyamlify is **not reconstruction** of the original source document. It is
**authoring from structured data**. The YAML is the source of truth; the
output is a well-written document derived from it.

The key skill here is knowing what to expand and what to keep terse.
Values that are already clear stay concise. Values with comments get
expanded — the comment is the cue that more prose is needed there.

---

## Output Modes

### 1. `doc` (DEFAULT)

A clean, complete markdown document. Readable by anyone. Good for sharing
with stakeholders, writing READMEs, generating specs from agent output.

**Rules:**
- Map top-level YAML keys → markdown `##` sections.
- Nested keys → subsections or prose paragraphs.
- YAML lists → markdown bullet lists or numbered steps.
- YAML comments → expanded into context sentences within their section.
- `_meta`, `intent`, `constraints` → a short intro paragraph at the top
  (not a raw dump — write it as a natural opening).
- `_stale` section → a "Known Gaps / Pending Decisions" section at the end.
- `TBD` values → phrased as open questions in the document.
- Tone: clear, direct, professional. No filler. No padding.

**Example:**

Input YAML:
```yaml
_meta:
  phase: mid
  confidence: partial

intent: build a centralized notification service replacing per-service email logic

constraints:
  - no PII stored beyond 30 days
  - all endpoints idempotent

auth:
  mechanism: JWT bearer token
  token_ttl: 24h
  # Refresh token rotation not yet decided
  refresh_rotation: TBD
```

Output markdown:
```markdown
# Notification Service

This document describes the notification service, currently in active
development. Some sections reflect decisions still in progress.

## Intent

Centralize all outbound notification logic to replace the per-service
email implementations currently scattered across the codebase.

## Constraints

- No PII may be stored beyond 30 days.
- All endpoints must be idempotent.

## Authentication

The service uses JWT bearer tokens with a 24-hour expiry. Refresh token
rotation policy has not yet been finalized — this decision is pending.

## Open Questions

- **Refresh token rotation**: Under discussion. Decision needed before launch.
```

---

### 2. `brief`

A short summary document. 1–2 paragraphs max. Good for status updates,
handoffs, or executive summaries derived from agent context.

**Rules:**
- Lead with intent in one sentence.
- Summarize current state and key constraints in 2–4 sentences.
- List any open questions or stale items as a brief bullet block at the end.
- No section headers — flowing prose only.
- Target: 80–150 words.

---

### 3. `spec`

A formal specification document. More structured than `doc`. Good for
technical specs, API docs, architecture decision records (ADRs).

**Rules:**
- Add a standard spec header: title, status, date, owner (from YAML if present).
- Sections follow a spec convention: Overview, Goals, Constraints,
  Design / Architecture, Open Questions, Decisions Log.
- Technical values (endpoints, TTLs, limits) formatted as definition lists
  or tables, not inline prose.
- Comments in YAML → "Note:" callouts or "Rationale:" subsections.
- `_stale` items → "Superseded Decisions" section.

---

## Step-by-Step Process

### Step 1: Read the YAML structure

Before writing:
- Identify the YAML mode (`lean`, `max`, `delta`, `minified`, or `ultrapack_*`).
- Extract `intent`, `constraints`, `_meta` — these drive the intro.
- Note all YAML comments — they are expansion cues.
- Note all `TBD` and `STALE` values — they become open questions.
- Identify the document type from content (spec, README, notes, config).
- If mode is `ultrapack_*`, check `_meta.mode` for compression level (`scaffold`,
  `core`, `ultra`) and check for a `_manifest` or `_chunks` block — deferred
  content will be absent from the YAML and must be flagged in the output.

### Step 2: Choose the output mode

| User says | Produce |
|---|---|
| "deyamlify this" / "make readable" | doc |
| "write this up" / "expand this" | doc |
| "brief" / "summary" / "tl;dr" | brief |
| "spec" / "formal doc" / "ADR" | spec |
| "README" | doc with README conventions |
| "status update" / "handoff" | brief |

### Step 3: Map structure to document sections

General mapping rules:

| YAML element | Markdown output |
|---|---|
| `_meta` + `intent` + `constraints` | Intro paragraph or Overview section |
| Top-level key | `##` section header |
| Nested key | `###` subsection or prose |
| Sequence (list) | Bullet list or numbered steps |
| Inline comment | Context sentence within section |
| Block comment above key | Paragraph before the content it describes |
| `TBD` value | "Not yet decided" / open question bullet |
| `STALE` marker | Note in "Known Gaps" or "Pending" section |
| `_stale` block | "Superseded / Pending Decisions" section |
| `delta.changed` | "Recent Changes" section |
| `delta.added` | Integrated into relevant sections |
| `delta.removed` | "Resolved Items" section |
| `_manifest` entry | Reference note: "full detail available on request" |
| `_chunks` entry | Callout: section exists but was not loaded |
| `deferred_state: N items` | "N additional items not loaded" note in that section |
| Skeleton value (3–5 words) | Minimal prose sentence; do not invent detail |
| Arrow chain (`A → B → C`) | Prose flow description of the sequence |
| Conditional bracket `[cond] → outcome` | "When X, the system does Y" sentence |
| Inline scope `key(scope): val` | Qualify the statement: "In [scope], ..." |
| Symbol prefix (`+`, `-`, `~`, `!`, `?`) | Translate: + = added/enabled, - = removed/deprecated, ~ = approximate, ! = required/critical, ? = unknown/TBD |
| `→ ref: key_path` | Inline reference: "see [section]" |

### Step 4: Write, don't dump

The most important rule: **write prose, don't serialize YAML into sentences.**

Bad (mechanical dump):
> The auth mechanism is JWT bearer token. The token_ttl is 24h.
> The refresh_ttl is 30d.

Good (written prose):
> Authentication uses JWT bearer tokens issued at login. Tokens expire
> after 24 hours; refresh tokens remain valid for 30 days.

Let the YAML be the outline. Write the document from it, as a human author
would — with natural connective tissue between ideas.

### Step 5: Handle comments as expansion cues

Every YAML comment is a signal that more context exists. When you encounter:

```yaml
cache_ttl: 5m
# Short TTL prioritizes freshness; revisit if p99 latency degrades
```

Expand to something like:
> The cache TTL is set to 5 minutes. This was chosen to prioritize data
> freshness over cache efficiency; if p99 latency becomes a concern,
> this value should be revisited.

Don't drop comments — they contain authorial intent that should surface
in the human document.

### Step 6: Open Questions section

Always include at the end if any `TBD` values or `STALE` markers exist.
Format as a named bullet list so humans can act on them:

```markdown
## Open Questions

- **Refresh token rotation** — Policy not yet decided. Tradeoff between
  security (rotation reduces exposure window) and UX (forces more re-logins).
- **Notification content retention** — Privacy implications under review.
  Decision needed before launch.
```

---

## Tone and Style Guide

- **Direct**: State facts plainly. Avoid "it should be noted that" and similar.
- **Terse where appropriate**: Technical values (TTLs, limits, endpoints) don't
  need elaboration unless a comment demands it.
- **Human where appropriate**: Decisions and rationale deserve real sentences.
- **No filler**: Never pad to length. A good deyamlify output can be shorter
  than the original markdown if the original had redundancy.
- **No YAML leakage**: Never let YAML syntax appear in the output. No `key:`,
  no `- item`, no `snake_case` keys as section headers. Translate everything
  to natural language (`token_ttl` → "token expiry", `max_attempts` →
  "maximum retry attempts").

---

## Handling Special YAML Modes

### From `minified`
Minified YAML is intentionally lossy — it dropped nuance to save tokens.
When deyamlifying minified, note at the top of the output:

```markdown
> ⚠️ Derived from minified YAML. Some detail may have been lost in
> compression. Verify against source if precision is required.
```

### From `delta`
A delta represents changes, not a full state. Two options:
1. **Changelog doc**: Write a "What Changed" document listing modifications.
2. **Merged doc**: Ask user for the base YAML, merge delta into it, then
   deyamlify the merged result as a full `doc`.

Default to option 1 unless the user requests the full document.

### From `max`
Max YAML is richest — expand generously. Full spec or doc output is ideal.

### From `ultrapack`

Ultrapack applies a four-layer information priority model before compressing.
Layers 3 and 4 are partially or fully dropped depending on compression level.
The output document will be incomplete — this is by design, not a deyamlify error.

**Always open with a completeness caveat** keyed to the compression level:

```markdown
> ⚠️ Derived from ultrapack (`{level}` compression). Layer 3 reference content
> was {compressed inline / deferred to manifest / dropped}. Sections marked
> [deferred] can be loaded on request.
```

**By compression level:**

| Level | What survived | What to do |
|---|---|---|
| `scaffold` | Layers 1–2 full; Layer 3 compressed to 1–2 lines per section | Expand normally; treat skeleton values as minimal stubs |
| `core` | Layers 1–2 full; Layer 3 deferred to `_manifest` | Expand Layers 1–2 fully; render `_manifest` as a "Deferred Sections" appendix |
| `ultra` | Layer 1 full; Layer 2 top items only; Layer 3 skeleton only | Expand what's there; flag every skeleton value as potentially incomplete |

**Decoding ultrapack encoding techniques** — these appear in the YAML values
and must be translated back to natural language:

- **Predicate-stripped values** (`golden_circle: Why→How→What order`): expand
  to a sentence that names the subject and states the claim. Do not pad — the
  stripped form is intentionally lean. One sentence is usually enough.

- **Noun-chain keys** (`user_token_auth`, `security_change_approvers`): convert
  to natural English noun phrases when writing section headers or prose
  ("user token authentication", "security change approvers").

- **Skeleton values** (3–5 word payloads): these are placeholders, not full
  descriptions. Write one sentence that conveys the core claim; flag as
  `[deferred — limited detail]` if the section warrants more depth.

- **Arrow chains** (`request → cache_miss → DB_fetch → cache_write(5m) → return`):
  write as a prose flow sequence. "On a cache miss, the system fetches the
  value from the database, writes it to the cache with a 5-minute TTL, then
  returns it to the caller."

- **Conditional brackets** (`[no_auth] → 401`, `[write] → 503`): write as
  conditional sentences. "[no_auth] → 401" becomes "Unauthenticated requests
  receive a 401 response."

- **Inline scope** (`latency(p99): 500ms`, `access(admin): delete+view`):
  qualify the statement explicitly. "The p99 latency target is 500ms."
  "Admin users can delete and view records."

- **Symbol vocabulary**: translate before writing prose.
  - `+dark_mode` → "Dark mode has been added."
  - `-v1_api` → "The v1 API has been deprecated."
  - `~80% done` → "Approximately 80% complete."
  - `!no_pii` → "Storing PII is prohibited." (mark as a constraint)
  - `?retention_policy` → Open question in the Open Questions section.

- **State-collapsed items** (`refresh_endpoint: blocked→auth_spec_signoff`):
  expand each field. "The refresh endpoint implementation is currently
  blocked, pending sign-off on the auth spec."

- **Schema-then-values tables** (`_schema: [method+path, auth, rate_limit, owner]`):
  render as a proper markdown table using the schema as column headers.

- **Cross-references** (`token_expiry: → ref: auth.token_ttl`): resolve
  inline — find the referenced value and write it into the sentence rather
  than exposing the pointer.

**Handling `_manifest` and `_chunks`:**

At `core` level, a `_manifest` block lists deferred Layer 3 content. Render
this as an appendix section at the end of the document:

```markdown
## Deferred Sections

The following sections were compressed out of this document and are available
on request:

- **{id}**: {summary}. (~{tokens} tokens) — request: "{request phrase}"
```

At `ultra` level with a `deferred_state` count, add a note within the
relevant section: "_(N additional items not included at this compression
level — request to load if needed.)_"

For chunked manifest output (`ultrapack_chunked`), the main YAML contains
only Layer 1 (intent + constraints) plus the chunk index. Expand only what
is present; render the chunk list as a "Document Sections" table showing
scope and phase relevance. Do not invent content for unloaded chunks.

---

## Output Format

Start directly with the document — no preamble like "Here is the expanded
document:" Just output the markdown. Use standard markdown conventions:

```markdown
# Document Title

Intro paragraph derived from intent + meta.

## Section One
...

## Open Questions
...
```