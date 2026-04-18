---
name: yamlify
description: >
  Convert markdown files into compact, AI-optimized YAML representations.
  Use this skill whenever a user wants to "yamlify" a markdown file, compress
  context for an AI, convert .md to YAML for use as AI context, or asks for a
  "lean", "minified", or "max" YAML version of any document. Also trigger when
  the user says a markdown file is "too bloated", "too many words", or wants to
  reduce token usage of a context document. Default output is the lean version
  unless otherwise specified.
---

# Yamlify Skill

Convert markdown files into clean, AI-optimized YAML. Three output modes exist:
lean (default), minified, and max. Most of the time, produce lean only.

---

## Core Philosophy

Markdown is human-optimized: full sentences, prose, headers, redundancy.
YAML context is AI-optimized: structured keys, minimal repetition, high signal.

The transformation is not summarization — it is **restructuring**. Meaning is
preserved; format is discarded. Use keys as semantic labels. Use values as
dense payloads. Use YAML comments (`#`) to preserve nuance that doesn't fit
cleanly into a value.

---

## The Three Output Modes

### 1. `lean` (DEFAULT — use unless told otherwise)

The everyday workhorse. Produces the leanest YAML that preserves all meaningful
information from the markdown. Typically 40–65% fewer tokens than the source.

**Rules:**
- Analyze the markdown first: identify its semantic structure (what are the
  real "properties" it's describing?)
- Map each concept to a YAML key. Use short, lowercase, snake_case keys.
- Write values as concise phrases or short sentences — not full prose.
- When a value needs clarification that can't fit in the value itself, add a
  YAML comment on the line above or inline: `key: value  # clarifying note`
- Preserve all factual content, decisions, constraints, and intent.
- Drop: filler phrases, restatements, transitional sentences, decorative prose,
  redundant headers.
- Output as a single YAML document.

**Example transformation:**

```markdown
## Authentication

Our system uses JWT tokens for authentication. Tokens are issued at login and
expire after 24 hours. Refresh tokens are valid for 30 days. All endpoints
except /health and /login require a valid Bearer token in the Authorization
header.
```

```yaml
auth:
  mechanism: JWT bearer token
  token_ttl: 24h
  refresh_ttl: 30d
  # All endpoints require Bearer token except:
  public_endpoints: [/health, /login]
```

---

### 2. `minified`

The smallest possible YAML. Used as a mid-conversation reminder: "what are we
doing and why?" Not suitable for project start, but excellent for keeping a
long-running AI session on track.

**Rules:**
- Keep only: goals, current state, active constraints, next actions.
- Collapse nested structures to single lines where possible.
- Abbreviate values aggressively — drop adjectives, drop examples, keep nouns
  and verbs.
- No comments unless they carry load-bearing information.
- Target: 20–35% of original token count.

**Example:**

```yaml
project: auth_service  |  goal: JWT auth  |  status: in_progress
constraints: [24h tokens, public=[/health,/login]]
next: implement refresh endpoint
```

---

### 3. `max`

The richest YAML form. Structural discipline of YAML, but values are allowed
to be fuller sentences. Best for complex specs, architecture docs, or documents
where nuance is critical and token budget is secondary.

**Rules:**
- Full sentence values are permitted.
- Nested objects and lists encouraged to preserve hierarchy.
- Comments used generously to preserve authorial intent and edge cases.
- Still no markdown prose — no headers-as-text, no transitional sentences.
- Target: 70–85% of original token count.

---

## Step-by-Step Process

### Step 1: Read and analyze the markdown

Before writing any YAML:
- Identify the document type (spec, README, guide, notes, config doc, etc.)
- Identify the real semantic structure: what are the top-level *things* being
  described?
- Note any sections that are pure filler vs. load-bearing content.

### Step 2: Design the key schema

Map the document's concepts to YAML keys:
- Use the document's own terminology, shortened. (`authentication` → `auth`)
- Group related concepts under parent keys.
- Lists of items → YAML sequences.
- Key/value pairs in prose → YAML mappings.

### Step 3: Fill values

- Write the densest accurate value possible.
- When a value needs context that can't fit: add a `# comment`.
- If something is genuinely ambiguous in the source, preserve the ambiguity
  with a comment: `field: TBD  # source unclear, two options mentioned`

### Step 4: Output

- Wrap in a YAML document (`---` at top).
- For lean: single document, clean output.
- For minified: optionally use inline flow style (`{key: val, key2: val2}`).
- For max: full block style, generous comments.

---

## When the User Specifies a Mode

| User says | Produce |
|---|---|
| "yamlify this" | lean only |
| "lean yaml" | lean only |
| "minified" / "reminder version" | minified only |
| "max" / "full yaml" | max only |
| "all three" / "all versions" | lean + minified + max, labeled |

---

## Comment Usage Guide

Comments are a first-class tool — not an afterthought. Use them to:

1. **Clarify ambiguous keys**: `deadline: 2025-06-01  # hard cutoff, not soft`
2. **Preserve discarded nuance**: When dropping a paragraph, ask: is there
   anything in here that can't be inferred from the key/value? If yes → comment.
3. **Flag uncertainty**: `owner: engineering  # unclear if platform or product`
4. **Preserve intent behind a decision**: 
   ```yaml
   cache_ttl: 5m
   # Short TTL chosen to prioritize freshness over performance — revisit if load increases
   ```

---

## Output Format

Always label the output with the mode name as a comment at the top:

```yaml
# lean
---
key: value
...
```

If producing multiple versions, separate with `---` and label each.