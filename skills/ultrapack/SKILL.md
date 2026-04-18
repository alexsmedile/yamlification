---
name: ultrapack
description: >
  Extreme context compression for large documents that are too long to carry
  as standard YAML context. Trigger this skill whenever a user needs to compress
  a large document, spec, codebase summary, or knowledge base for use as agent
  context; asks for "maximum compression", "extreme compression", "fit this in
  context", "ultrapack this", or "this is too big"; says the document is "too
  long", "too large for context", "won't fit", or is causing context pressure
  in a long session. Also trigger when yamlify or yamlify-advanced produces
  output that is still too large for the user's context budget. Default behavior:
  assess the document and workflow situation, then choose the compression level
  automatically. Do not ask the user which level they want unless they specify
  one or the automatic choice is genuinely ambiguous.
---

# Ultrapack

Extreme compression for documents too large to carry as standard agent context.
Applies a layered information priority model to reduce documents to the minimum
viable signal for the current workflow — without summarization, paraphrase, or
meaning loss.

---

## Core Philosophy

Standard yamlify targets 40–80% token reduction by stripping prose and
restructuring format. Ultrapack targets 80–95% reduction by applying a theory
of what agents actually need to act correctly, then discarding everything else.

The principle is **information priority, not summarization**. Content is not
paraphrased or abstracted into vaguer forms. It is either kept (in the densest
possible encoding), deferred to a manifest entry, or dropped because it carries
no action-relevant signal for the current workflow.

Three claims underpin this:

1. **Agents act on a small fraction of a document.** Most content in a large
   spec, README, or knowledge doc is background, rationale, and history. An
   agent running a workflow needs constraints, current state, and next actions
   — not the evolutionary history of how those constraints were reached.

2. **Structure is cheaper than content.** A one-line key that points to a
   chunk ("see: auth_spec chunk 2") costs 6 tokens. The chunk it replaces may
   cost 400. When an agent needs the chunk, it can be loaded. Until then, the
   pointer is sufficient.

3. **Compression level should match workflow phase.** A document loaded at
   session start needs more context than a mid-session reminder. An agent
   near the end of a workflow needs only the exit criteria and constraints.
   Ultrapack reads the situation and calibrates automatically.

---

## Information Priority Model

All content in any document falls into one of four layers. Ultrapack processes
layers in order: Layer 1 is always kept, Layer 4 is always dropped. Layers 2
and 3 are kept, compressed, or deferred depending on the compression level.

### Layer 1 — Command (always kept, never compressed)

The minimum an agent must have to act without error.

- **Intent**: single-sentence purpose of the document or workflow
- **Constraints**: hard limits, must-nots, compliance requirements, SLAs
- **Phase**: where in the workflow this context applies
- **Critical state**: any decisions or facts that, if unknown, would cause the
  agent to take a wrong action immediately
- **Owner / authority**: who can make decisions if the agent hits ambiguity

Typical Layer 1 output: 40–100 tokens. Never more. If Layer 1 alone exceeds
150 tokens, the document has too many constraints — flag this to the user.

### Layer 2 — Active State (kept in full at `scaffold`, compressed at `core`, summarized at `ultra`)

What is true right now, relevant to the current workflow.

- Current status of open work items
- Pending decisions that block progress
- Next actions (concrete, not aspirational)
- Open questions with their current state (resolved / unresolved / blocked)
- Known risks or dependencies that are active — not historical

At `ultra`: collapse to a flat list of the most time-sensitive items only.
At `core`: keep all items, compress each to a single line.
At `scaffold`: keep all items with their supporting context.

### Layer 3 — Reference Payload (compressed at `scaffold`, deferred at `core`, skeleton at `ultra`)

Background that an agent might need but doesn't need to carry at all times.

- Rationale behind decisions (why, not just what)
- Historical context and superseded approaches
- Examples and illustrative cases
- Detailed technical specs that aren't directly action-gating
- Supporting arguments and trade-off discussions

At `scaffold`: compress each section to 1–2 key/comment pairs.
At `core`: deferred to manifest with a one-line summary.
At `ultra`: skeleton-encoded — key name + 3–5 word payload only, no comments,
no elaboration. Full structural coverage survives; all prose is gone. This
preserves complete topical awareness at near-zero token cost per entry.

### Layer 4 — Archival (always dropped)

Content that carries no forward-looking signal.

- Resolved decisions (keep only the conclusion, in Layer 2 or 3)
- Superseded specifications
- Content explicitly marked `STALE`
- Introductory and onboarding prose ("this document describes...")
- Metadata about the document itself (revision history, authorship notes)
- Decorative structure: transition sentences, section summaries that restate
  the section, closing paragraphs

---

## Compression Levels

Ultrapack selects the level automatically by default. Manual override available.

### `scaffold` — ~50–65% reduction

**When auto-selected:**
- Session start: agent needs situational awareness before acting
- Document is a spec or architecture doc where Layer 3 rationale may
  affect decisions the agent will make
- Downstream consumer is an agent that will make structural design choices
- User has not yet read or reviewed the document themselves

**What's kept:**
- Layer 1: full, uncompressed
- Layer 2: full, uncompressed
- Layer 3: compressed to 1–2 YAML key/comment pairs per section
- Layer 4: dropped

**Structure:** Full YAML document. Sections correspond to source sections.
Layer 3 sections present but collapsed. No manifest needed.

---

### `core` — ~70–82% reduction

**When auto-selected:**
- Mid-session: agent has already oriented, now needs a lean reminder
- Document is well-understood by both user and agent
- Workflow is in execution phase — decisions are made, agent is implementing
- Token pressure exists but isn't critical

**What's kept:**
- Layer 1: full, uncompressed
- Layer 2: compressed — one line per item, inline flow where possible
- Layer 3: deferred to manifest (pointer + one-line summary)
- Layer 4: dropped

**Structure:** Compact YAML document + a `_manifest` block at the bottom
listing deferred Layer 3 chunks with summaries. Agent can request a chunk
by name if needed.

---

### `ultra` — ~85–95% reduction

**When auto-selected:**
- Context window is critically full
- Session is in final phase — agent is completing, wrapping up, or exiting
- Document is being used as a constraint check only, not a reference
- User explicitly asks for "maximum compression" or "smallest possible"
- Document is extremely large (>10,000 tokens raw) and even `core` won't fit

**What's kept:**
- Layer 1: full, uncompressed
- Layer 2: only the top 3–5 most time-critical items, collapsed to inline flow
- Layer 3: skeleton-encoded — key + 3–5 word payload, no comments, no prose
- Layer 4: dropped

**Structure:** Ultra-compact YAML. Fits in ~200–500 tokens for most documents.
If Layer 2 items exceed 5, group the rest under `deferred_state` with counts
only: `deferred_state: 8 items — request if needed`.

---

## Auto-Selection Logic

When no level is specified, read these signals in order:

```
0. What type of document is this?

   KNOWLEDGE / REFERENCE (book, article, long-form notes, research, concepts):
   → No session phase applies. Go directly to core with skeleton encoding
     on all Layer 3 content. Goal: full topical coverage, minimum tokens.

   WORKFLOW / SPEC (architecture doc, task spec, agent context, decision log):
   → Continue to phase-based selection below.

1. Has the user mentioned token pressure, context filling up, or running long?
   → ultra

2. Is this the first time this document is being loaded in the session?
   → scaffold (agent needs orientation)

3. Is the workflow in an execution phase (decisions made, implementing)?
   → core

4. Is the document being used as a constraint check or exit criteria only?
   → ultra

5. Is the document a complex spec where rationale affects agent decisions?
   → scaffold

6. Default (unclear phase, general use):
   → core
```

State the chosen level and the reason in one line at the top of the output:
`# ultrapack — core (mid-session execution phase, token pressure moderate)`

---

## Manifest System

The manifest is a lightweight index of deferred content. It allows an agent
to know what was dropped and request it explicitly if needed, without paying
the full token cost upfront.

**Manifest entry format:**

```yaml
_manifest:
  - id: auth_rationale
    summary: JWT chosen over OAuth2 for statelessness; session tokens rejected for compliance
    tokens: ~120
    layer: 3
    request: "load auth_rationale"

  - id: retry_spec_detail
    summary: Full retry backoff table with edge cases for partial failures
    tokens: ~200
    layer: 3
    request: "load retry_spec_detail"
```

**Rules:**
- Every deferred chunk gets one manifest entry
- `summary` is one line: enough for the agent to decide if it needs the chunk
- `tokens` is an estimate — helps the agent manage its own context budget
- `request` is the exact phrase the user or agent can say to load the chunk
- When a chunk is requested, output it as a standalone YAML block, not
  re-integrated into the full document

**When to include a manifest:**
- `core` level: always, for all deferred Layer 3 content
- `ultra` level: only if `deferred_state` items exceed 5
- `scaffold` level: not needed (Layer 3 is compressed inline, not deferred)

---

## Chunking for Oversized Documents

When even `ultra` compression produces output that exceeds the user's stated
context budget, or when the source document is so large that a single-pass
compression loses coherence, use **semantic chunking**.

**Do not chunk by token count.** Chunk by semantic unit:
- A spec sections as a chunk
- A workflow phase as a chunk
- A component or subsystem as a chunk
- A decision cluster (related decisions and their constraints) as a chunk

**Chunk output format:**

```yaml
# ultrapack — chunked manifest
---
_meta:
  mode: ultrapack_chunked
  source: large-spec.md
  total_chunks: 4
  load_order: sequential  # or: on-demand

intent: {single-sentence intent from Layer 1}

constraints:
  - {all constraints, always fully present in the manifest}

_chunks:
  - id: chunk_auth
    scope: authentication and session management
    phase_relevance: [start, mid]
    tokens: ~180
    request: "load chunk_auth"

  - id: chunk_data_model
    scope: database schema and entity relationships
    phase_relevance: [start]
    tokens: ~240
    request: "load chunk_data_model"

  - id: chunk_ops
    scope: deployment, monitoring, and incident response
    phase_relevance: [end]
    tokens: ~150
    request: "load chunk_ops"

  - id: chunk_decisions
    scope: architecture decisions and their rationale
    phase_relevance: [start]
    tokens: ~320
    request: "load chunk_decisions"
```

**The manifest always contains Layer 1 in full.** Constraints are never
chunked or deferred — they must be present in every agent interaction.

**`phase_relevance`** tells the agent which chunks to load first based on
workflow phase. An agent at `mid` phase loads `mid`-relevant chunks only,
skipping `start`-only orientation chunks it no longer needs.

---

## Encoding Techniques

This is the core of ultrapack. The layer model defines *what* to keep. These
techniques define *how* to compress it. Always apply the densest technique
that preserves the full meaning. Never summarize — every technique below is
lossless on meaning, lossy only on words.

---

### 1. Predicate Strip

**Rule:** Keep subject + verb + object. Drop qualifiers, adverbs, connective
tissue, transitional phrases, and illustrative elaboration.

**Use for:** conceptual explanations, argument chains, narrative passages.

```
Source:
  "Simon Sinek's Golden Circle framework argues that great leaders and
  organizations communicate from the inside out — starting with Why (purpose),
  then How (process), then What (result) — because people don't buy what you
  do, they buy why you do it."

Ultrapack:
  golden_circle: Why→How→What order
  mechanic: sell belief not product
  insight: people buy Why not What
```

The mechanism, the sequence, and the actionable insight all survive. The
author name, the framing prose, and the elaboration are gone.

---

### 2. Noun-Chain Collapse

**Rule:** Replace a descriptive phrase with its essential noun chain.
Compound concepts become compound keys or terse values.

**Use for:** process descriptions, system component names, relationship labels.

```
Source:
  "the process by which users authenticate their identity using tokens"
→ user_token_auth

Source:
  "a feedback loop that reinforces the original behavior over time"
→ reinforcing_feedback_loop

Source:
  "the team responsible for reviewing and approving security changes"
→ security_change_approvers
```

---

### 3. Implicit Drop

**Rule:** Drop anything that can be perfectly inferred from the key name alone.
The key *is* the context — the value carries only the delta.

**Use for:** any value that restates, introduces, or describes the key.

```
Bad (value restates key):
  authentication: uses JWT tokens for authenticating users

Good (value is only the delta):
  auth: JWT
  token_ttl: 24h
  public_endpoints: [/health, /login]
```

If removing a word loses meaning, keep it. If removing it loses nothing, drop it.

---

### 4. List Collapse

**Rule:** Replace enumerated prose — items described in paragraphs or
sentences — with a flat YAML sequence. Order matters only if the source
implies sequence; otherwise sort by importance.

**Use for:** feature lists, option sets, step sequences, constraint lists.

```
Source:
  "The system supports three output modes: a lean mode for everyday use,
  a minified mode for mid-session reminders, and a max mode for complex
  documents where nuance is critical."

Ultrapack:
  output_modes: [lean, minified, max]
  # lean=default, minified=mid-session, max=nuance-critical
```

When list items have meaningful distinctions, put the distinction in a comment
on the same line — never expand back into prose.

---

### 5. Comment Anchor

**Rule:** When a concept is too nuanced to fit in a value but too important
to drop, reduce the value to its irreducible keyword and park the nuance in
a YAML comment. The keyword unlocks the concept; the comment completes it.

**Use for:** decisions with non-obvious rationale, constraints with exceptions,
states with important caveats.

```
Source:
  "We chose JWT over OAuth2 primarily for statelessness — we don't want
  to maintain session state on the server, and OAuth2's overhead wasn't
  justified for an internal API. Session tokens were rejected outright
  due to compliance requirements."

Ultrapack:
  auth: JWT  # stateless; OAuth2 overhead unjustified; sessions rejected=compliance
```

One line. The value names the decision. The comment preserves the why and
the rejected alternatives.

---

### 6. Skeleton Encoding

**Rule:** For Layer 3 content at `ultra` level — or any content where topical
coverage matters but detail does not — encode as key + 3–5 word payload only.
No comments. No elaboration. Structure and topic survive; all prose is gone.

**Use for:** Layer 3 at ultra level, large reference sections that must be
inventoried but not read.

```
Source section: "Retry Logic"
  (200 tokens of backoff tables, edge cases, failure modes, diagrams)

Skeleton:
  retry_logic: 3 attempts, exponential backoff
```

```
Source section: "Golden Circle — Why"
  (300 tokens explaining purpose-driven leadership, examples, case studies)

Skeleton:
  golden_circle_why: purpose drives action and loyalty
```

The reader knows the topic exists and its core claim. They can request the
full chunk if they need the depth.

---

### 7. State Collapse

**Rule:** When multiple fields describe the same evolving state (status,
progress, blockers, next action), collapse to a single inline flow entry
per work item. Drop field names that are implied by position or context.

**Use for:** Layer 2 at `core` and `ultra` levels, project status, task lists.

```
Source:
  - Task: implement refresh endpoint
    Status: in progress
    Blocker: waiting on auth spec sign-off
    Owner: platform team
    Due: end of sprint

Ultrapack (core):
  refresh_endpoint: in_progress | blocked=auth_spec_signoff | owner=platform | due=sprint_end

Ultrapack (ultra):
  refresh_endpoint: blocked→auth_spec_signoff
```

---

### 8. Relation Arrow

**Rule:** Express causal chains, sequences, data flows, and transformations
using `→`. Express logical implication using `⇒`. Both are universally
readable by humans (from math, code, diagrams) and reliably parsed by agents.

**Use for:** pipelines, decision trees, cause-effect pairs, state transitions,
dependency chains.

```
Source:
  "When a cache miss occurs, the system fetches the value from the database,
  stores it in the cache with a 5-minute TTL, then returns it to the caller."

Ultrapack:
  cache_miss_flow: request → cache_miss → DB_fetch → cache_write(5m) → return

Source:
  "If a user fails authentication three times, their account is locked and
  an alert is sent to the security team."

Ultrapack:
  auth_failure: 3x_fail ⇒ account_locked + security_alert
```

Arrow chains can be nested inside state collapse or combined with inline
scope brackets. Keep chains left-to-right; branch with `+` for parallel steps.

---

### 9. Conditional Bracket

**Rule:** Express conditionals as `[condition] outcome`. Drop "if", "when",
"in the case that", and all other conditional framing prose. The bracket
*is* the conditional marker.

**Use for:** business rules, exception handling, branching logic, guard clauses,
policy statements.

```
Source:
  "If the request is made by an unauthenticated user, return a 401 error.
  If the user is authenticated but lacks the required role, return a 403."

Ultrapack:
  access_control:
    - [no_auth] → 401
    - [auth, !role] → 403

Source:
  "When operating in maintenance mode, all write operations should be rejected
  with a 503, but read operations can proceed normally."

Ultrapack:
  maintenance_mode:
    - [write] → 503
    - [read] → pass
```

Combine with relation arrow for condition-triggered flows:
`[retry_exhausted] → dead_letter_queue → alert`

---

### 10. Schema-Then-Values

**Rule:** When multiple items share identical structure, define the schema
once, then encode each instance as a value array. Eliminates repeated key
names across every entry — the highest-ROI technique for tabular or
repeated-structure content.

**Use for:** task lists, API endpoint tables, configuration sets, test cases,
team rosters, changelog entries — any repeated structure.

```
Source:
  - Endpoint: POST /auth/login
    Auth required: No
    Rate limit: 10/min
    Owner: platform

  - Endpoint: GET /users/:id
    Auth required: Yes
    Rate limit: 100/min
    Owner: platform

  - Endpoint: DELETE /users/:id
    Auth required: Yes
    Rate limit: 10/min
    Owner: security

Ultrapack:
  _schema: [method+path, auth, rate_limit, owner]
  endpoints:
    - [POST /auth/login, no, 10/min, platform]
    - [GET /users/:id, yes, 100/min, platform]
    - [DELETE /users/:id, yes, 10/min, security]
```

The schema line is the investment — it pays back on every entry. For 3+
entries with 3+ fields, this technique almost always wins on token count.
For 2 entries or fewer, use state collapse instead.

---

### 11. Symbol Vocabulary

**Rule:** Use a fixed set of single-character prefixes that carry semantic
weight without words. Apply consistently across the document — never invent
new symbols mid-document.

**Allowed symbol set:**

| Symbol | Meaning | Example |
|---|---|---|
| `+` | added, new, supported, enabled | `+dark_mode`, `+retry_logic` |
| `-` | removed, deprecated, disabled, rejected | `-session_tokens`, `-v1_api` |
| `~` | approximate, partial, uncertain, in progress | `~80% done`, `~2h estimate` |
| `!` | critical, required, must-not-violate, error | `!no_pii`, `!auth_required` |
| `?` | unknown, TBD, needs decision, ambiguous | `?owner`, `?launch_date` |

**Use for:** changelogs, constraint lists, feature flags, status indicators,
decision states — anywhere a one-word status needs to carry a polarity.

```
Source:
  "Dark mode support has been added. The legacy v1 API has been deprecated.
  The cache TTL is approximately 5 minutes but may vary. PII storage is
  absolutely prohibited. The data retention policy is still to be decided."

Ultrapack:
  changes: [+dark_mode, -v1_api]
  cache_ttl: ~5m
  constraints: [!no_pii]
  open: [?retention_policy]
```

Never stack symbols (`!?` is ambiguous). If two symbols seem to apply, use
a comment anchor to resolve: `retention_policy: TBD  # !critical — decide before launch`.

---

### 12. Inline Scope

**Rule:** When a value applies only under a specific condition, context, or
qualifier, attach the qualifier as a parenthetical to the key rather than
creating a nested object or a separate conditional.

**Format:** `key(scope): value`

**Use for:** environment-specific config, percentile-qualified metrics,
role-scoped permissions, version-bounded behavior, context-dependent limits.

```
Source:
  "The p99 latency target is 500ms. The burst rate limit is 100 requests
  per second. The sustained rate limit is 20 requests per second. In the
  EU region, data must not leave the region boundary."

Ultrapack:
  latency(p99): 500ms
  rate_limit(burst): 100rps
  rate_limit(sustained): 20rps
  data_residency(EU): in-region only

Source:
  "Admin users can delete records. Read-only users can only view records.
  In staging environments, all users have admin-level access."

Ultrapack:
  access(admin): delete+view
  access(read_only): view
  access(staging): admin_level  # override — all users
```

Inline scope avoids a nested object (saves 2–4 lines) and avoids a comment
(keeps the qualifier in the parseable key, not the prose). Use it when the
qualifier is the primary differentiator between sibling values.

---

### 13. Cross-Reference

**Rule:** When two sections share content, one holds the content and the
other holds a pointer. Never duplicate. A pointer costs ~4 tokens; duplication
costs N tokens per copy.

**Format:** `→ ref: key_path` or `→ see: chunk_id`

**Use for:** shared constraints across sections, repeated definitions,
content that belongs to one section but is relevant to several, inter-chunk
references in chunked manifests.

```
Source:
  Section "Authentication": "Tokens expire after 24 hours..."
  Section "Security": "All tokens expire after 24 hours, consistent with
    our authentication policy..."

Ultrapack:
  auth:
    token_ttl: 24h
    refresh_ttl: 30d

  security:
    token_expiry: → ref: auth.token_ttl
    # no duplication — single source of truth
```

```
Source: large spec with "Retry Logic" described in both the API section
  and the Error Handling section.

Ultrapack (chunked):
  api:
    retry: → see: chunk_error_handling

  _chunks:
    - id: chunk_error_handling
      scope: retry logic, error codes, fallback behavior
```

Cross-reference is mandatory when the same value would appear in two or more
places. Duplication in ultrapack output is always a bug — it means the
source was not deduped before compression.

---

### Technique Selection Guide

| Content type | Primary technique | Fallback |
|---|---|---|
| Conceptual argument / narrative | Predicate strip | Comment anchor |
| Process or system description | Noun-chain collapse | List collapse |
| Decision with rationale | Comment anchor | Predicate strip |
| Feature or option set | List collapse | Symbol vocabulary |
| Task / status / progress | State collapse | Relation arrow |
| Causal chain / pipeline / flow | Relation arrow | Conditional bracket |
| Branching logic / rules / policy | Conditional bracket | Relation arrow |
| Repeated structure / tabular data | Schema-then-values | State collapse |
| Polarity / status / flag | Symbol vocabulary | Implicit drop |
| Scoped / qualified values | Inline scope | Comment anchor |
| Duplicated content across sections | Cross-reference | — |
| Reference section (ultra) | Skeleton | Comment anchor |
| Any value restating its key | Implicit drop | — |

Multiple techniques combine on the same entry. A state-collapsed item can
use implicit drop on its values and symbol vocabulary on its status. A
relation arrow chain can embed conditional brackets at branch points. A
schema-then-values table can use inline scope in its schema definition.
Apply all that fit — the goal is the irreducible minimum that still delivers
full meaning.

---

### Step 1: Read the document and assess

Before compressing anything:
- How large is it? (rough token estimate)
- What type is it? (spec, architecture doc, workflow, knowledge base, notes)
- What is the apparent workflow phase? (start / mid / end)
- Is there explicit context pressure from the user?
- Does it already have YAML structure, or is it raw markdown?

### Step 2: Select compression level

Apply auto-selection logic. State the chosen level and reason in the output
header. If genuinely ambiguous, default to `core`.

### Step 3: Identify and extract Layer 1

Find or infer:
- Intent (explicit or inferred from document purpose)
- All constraints (search for: "must", "never", "required", "not allowed",
  SLAs, compliance, security mandates, architectural non-negotiables)
- Phase
- Critical state (decisions or facts that are immediately action-gating)

Write Layer 1 first. Do not proceed to other layers until it's complete.

### Step 4: Process Layer 2

Extract all active state items. Apply compression level rules:
- `scaffold`: keep full with supporting context
- `core`: compress each to one line
- `ultra`: keep top 3–5 by time-criticality, collapse rest to count

### Step 5: Process Layer 3

Apply compression level rules:
- `scaffold`: compress each section to 1–2 key/comment pairs, keep inline
- `core`: write one-line summary, defer to manifest
- `ultra`: drop entirely

If deferring, write manifest entries as you process each section.

### Step 6: Drop Layer 4

Identify and discard all archival content. Do not note what was dropped
(that itself costs tokens). Exception: if a significant section is dropped
that the user might expect to see, add a one-line comment:
`# dropped: revision history (archival, no forward signal)`

### Step 7: Check for chunking threshold

If the output is still larger than the situation warrants, or if the source
is so large that single-pass compression loses structural coherence, switch
to chunked manifest output. Restate the compression as a manifest + chunks.

### Step 8: Output

Label clearly. State level and auto-selection reason. Output Layer 1 first,
always. Manifest at the bottom if present.

---

## Output Format

### Standard (non-chunked)

```yaml
# ultrapack — {level} ({auto-selection reason})
---
_meta:
  mode: ultrapack_{level}
  source: {filename if known}
  phase: {start | mid | end}
  compressed: {date}
  confidence: {high | partial | stale}

intent: {single sentence}

constraints:
  - {hard limit}
  - {hard limit}

# --- active state ---
{Layer 2 content at appropriate compression}

# --- reference (scaffold only) ---
{Layer 3 compressed inline, scaffold level only}

_manifest:          # core level only
  - id: ...
    summary: ...
```

### Chunked manifest

```yaml
# ultrapack — chunked ({reason chunking was chosen})
---
_meta:
  mode: ultrapack_chunked
  ...

intent: ...
constraints: [...]

_chunks:
  - id: ...
    scope: ...
    phase_relevance: [...]
    tokens: ~{n}
    request: "load {id}"
```

---

## Relationship to Other Skills

| Need | Use |
|---|---|
| Large document, extreme compression needed | **ultrapack** (this skill) |
| Normal document, standard compression | yamlify or yamlify-advanced |
| Compressed YAML → human-readable doc | deyamlify |
| Audit quality of a compressed file | yamlify-review |
| Only what changed between versions | yamlify-advanced delta mode |

Ultrapack output is designed to be compatible with deyamlify: a `doc` or
`spec` can be generated from ultrapack output, but will be incomplete if
significant Layer 3 content was dropped or deferred. When deyamlifying
ultrapack output, always note the compression level so the reader knows
what may be missing.

---

## What Ultrapack Does Not Do

- **Summarize or paraphrase.** Content that is kept is encoded using explicit
  techniques (predicate strip, noun-chain, skeleton, etc.) that are lossless
  on meaning. Content that is dropped is dropped cleanly — not blended into
  vaguer abstractions. Encoding is not summarization.
- **Invent or infer content.** If a constraint is implied but not stated,
  flag it as inferred with a comment: `# inferred from context — verify`.
- **Guarantee round-trip fidelity.** Ultra and core levels intentionally drop
  content. A full deyamlify from ultrapack output will not reconstruct the
  source document. This is by design.
- **Replace RAG.** For truly massive corpora (hundreds of documents, full
  codebases), ultrapack is a stopgap that buys time. A proper retrieval system
  is the right long-term answer. Ultrapack handles the case where you don't
  have one yet.
