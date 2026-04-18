---
name: yamlify-advanced
description: >
  Convert markdown (or any prose document) into compact, AI-optimized YAML
  for use as agent context, skill input, workflow state, or session memory.
  Trigger this skill whenever a user wants to "yamlify", compress context,
  reduce token usage, convert a document for AI consumption, or produce a
  lean/minified/max/delta YAML version of any file. Also trigger when the
  user says a document is "too bloated", "too long for context", or wants
  a version "an agent can use". Default output is lean unless specified.
  Always use this skill over ad-hoc compression — it enforces conventions
  that make YAML reliably parseable by downstream agents and skills.
---

# Yamlify Advanced

Convert documents into structured, AI-optimized YAML with agent-ready
conventions. Four output modes: `lean` (default), `minified`, `max`, `delta`.

---

## Core Philosophy

Markdown is human-optimized: prose, headers, redundancy, narrative flow.
Agent YAML is machine-optimized: structured keys, zero redundancy, high signal.

The transformation is **restructuring, not summarization.** All meaning is
preserved — format is discarded. Keys are semantic labels. Values are dense
payloads. Comments carry nuance that doesn't compress into a value cleanly.

Every yamlified document follows the **agent-ready convention**: a `_meta`
block at the top, `intent` and `constraints` surfaced at the root level,
and `stale` markers on anything that may no longer be current.

---

## Agent-Ready Conventions (apply to ALL modes except minified)

These are non-negotiable structural conventions. They make every output
immediately parseable by any downstream agent without needing to read
the full document first.

### 1. `_meta` header block

Always the first key:

```yaml
_meta:
  phase: start          # start | mid | end
  mode: lean            # lean | max | delta
  source: spec.md       # original filename if known
  last_updated: 2025-04-14
  confidence: high      # high | partial | stale
```

- `phase`: where in a workflow this context applies
- `confidence`: `high` = fully current; `partial` = some sections uncertain;
  `stale` = known to be outdated, use with caution

### 2. `intent` at root level

Always the second key. One line: what is this document/project trying to
accomplish? Even if it repeats something deeper, surface it here.

```yaml
intent: build a centralized notification service replacing per-service email logic
```

### 3. `constraints` at root level

Always the third key. A flat list of hard limits, non-negotiables, and
must-not-violates. Agents check this first before taking any action.

```yaml
constraints:
  - no PII in notification body stored beyond 30 days
  - all endpoints must be idempotent
  - max latency 500ms p99
```

### 4. `stale` markers

When a field is known to be outdated or superseded, mark it:

```yaml
owner: TBD  # STALE — decision pending since 2025-03-01
sla: sub-30s  # STALE — revised up to 2min after load testing
```

Or group stale fields under a `_stale` key at the bottom:

```yaml
_stale:
  - sla: original sub-30s target, superseded
  - owner: was "platform", now under review
```

---

## The Four Output Modes

### 1. `lean` (DEFAULT)

The everyday workhorse. All meaningful information, ~50–70% fewer tokens
than source. Comments carry nuance. Agent-ready conventions applied.

**Rules:**
- Apply `_meta`, `intent`, `constraints` header.
- Map each concept to a short `snake_case` key.
- Values: concise phrases or short sentences, never full prose.
- Comments for: ambiguity, dropped nuance, decision rationale, stale flags.
- Drop: filler, restatements, transitional prose, decorative headers.

**Example:**

```yaml
# lean
---
_meta:
  phase: start
  mode: lean
  source: auth-spec.md
  confidence: high

intent: implement JWT-based auth for all internal APIs

constraints:
  - tokens must expire — no infinite sessions
  - /health and /login must remain public

auth:
  mechanism: JWT bearer token
  token_ttl: 24h
  refresh_ttl: 30d
  # Refresh token rotation not yet decided — see open_questions
  public_endpoints: [/health, /login]

open_questions:
  refresh_rotation: undecided  # security vs UX tradeoff unresolved
```

---

### 2. `minified`

The smallest possible YAML. Mid-session agent reminder: "what am I doing
and what are my hard limits?" Not for project start. No `_meta` block needed.

**Rules:**
- Keep only: intent, active constraints, current state, next action.
- Collapse nested structures to inline flow style.
- Abbreviate aggressively — nouns and verbs only, drop adjectives/examples.
- No comments unless load-bearing.
- Target: 15–30% of original token count.

**Example:**

```yaml
# minified
---
intent: JWT auth for internal APIs
constraints: [tokens expire, /health+/login public]
status: in_progress
next: implement refresh endpoint
```

---

### 3. `max`

Richest YAML form. Full sentences permitted in values. Best for complex
specs or architecture docs where nuance is critical. Agent-ready conventions
applied with generous comments.

**Rules:**
- Full `_meta`, `intent`, `constraints` header — fully detailed.
- Full sentence values permitted.
- Nested objects and lists encouraged to preserve full hierarchy.
- Comments used generously: intent behind decisions, edge cases, trade-offs.
- Still no markdown prose — no headers-as-text, no transitional sentences.
- Target: 75–85% of original token count.

---

### 4. `delta`

A diff between two YAML versions. Used when context evolves across sessions
and you don't want to re-inject the full document — only what changed.

**Rules:**
- Requires a reference version (specify as `_base`).
- Three sections: `changed`, `added`, `removed`.
- `changed`: show `old_value → new_value` inline.
- `added`: new keys/sections not in base.
- `removed`: keys that no longer exist or were resolved.
- Include updated `_meta.last_updated` and `_meta.confidence`.

**Example:**

```yaml
# delta
---
_meta:
  mode: delta
  _base: lean_v1  # diffing against this version
  last_updated: 2025-04-14
  confidence: high

changed:
  status: in_progress → complete
  owner: TBD → platform-team
  sla: sub-30s → 2min  # revised after load testing

added:
  retry_logic:
    max_attempts: 3
    backoff: [1m, 5m, 30m]
    on_exhaustion: page on-call

removed:
  - open_question: sla_undefined  # resolved
  - open_question: refresh_rotation  # resolved — rotation not required v1
```

---

## Step-by-Step Process

### Step 1: Read and classify the document

Before writing any YAML:
- What type is it? (spec, README, architecture doc, meeting notes, config,
  workflow definition, skill description, agent prompt)
- What phase does it represent? (start / mid / end)
- Are there sections that are clearly stale or superseded?
- Is there an explicit intent stated, or must it be inferred?

### Step 2: Extract the three header fields

Before mapping anything else, determine:
1. `intent` — the single-sentence purpose. Infer if not explicit.
2. `constraints` — hard limits. Look for: "must", "never", "required",
   "not allowed", SLAs, security requirements, architectural mandates.
3. `_meta.confidence` — are all sections current and complete?

### Step 3: Design the key schema

Map the document's semantic structure to YAML keys:
- Use the document's own terminology, shortened (`authentication` → `auth`).
- Group related concepts under parent keys.
- Lists of items → YAML sequences.
- Key/value pairs expressed in prose → YAML mappings.
- Decisions/conclusions → values. Rationale → comments.

### Step 4: Fill values and comments

- Write the densest accurate value possible.
- Ask for every dropped paragraph: "Is there anything here that can't be
  inferred from the key/value alone?" If yes → add a comment.
- Flag genuine ambiguity: `field: TBD  # two options, undecided`
- Flag stale content explicitly.

### Step 5: Output

Label mode at top as a comment. Apply conventions. For delta, verify the
base version is available or ask the user to provide it.

---

## Mode Selection Reference

| User says | Produce |
|---|---|
| "yamlify this" | lean |
| "lean" / "compress this" | lean |
| "minified" / "reminder" / "smallest" | minified |
| "max" / "full" / "rich" | max |
| "delta" / "what changed" / "diff" | delta |
| "all three" / "all versions" | lean + minified + max, labeled |
| "update the yaml" + existing YAML present | delta |

---

## Comment Usage Guide

Comments are first-class. Use them for:

```yaml
# 1. Clarify key meaning
deadline: 2025-06-01  # hard cutoff — no extensions approved

# 2. Preserve dropped nuance
cache_ttl: 5m
# Short TTL prioritizes freshness; revisit if p99 latency degrades

# 3. Flag uncertainty
owner: engineering  # unclear if platform or product eng

# 4. Flag stale data
sla: sub-30s  # STALE — revised to 2min post load-test

# 5. Preserve decision rationale
auth: JWT  # evaluated OAuth2 and sessions; JWT chosen for statelessness
```

---

## Output Format

```yaml
# {mode}
---
_meta:
  ...

intent: ...

constraints:
  - ...

{document content}
```

Multiple versions: separate with `---`, label each with `# {mode}`.