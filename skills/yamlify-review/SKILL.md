---
name: yamlify-review
description: >
  Audit, compare, and advise on markdown and YAML files used in yamlification
  workflows. Trigger this skill whenever a user wants to "review", "audit",
  "check", or "optimize" a YAML context file or markdown source; asks if a
  file is "well yamlified", "ready to yamlify", or "worth compressing"; wants
  to compare a markdown source against its YAML version; asks "what's wrong
  with this", "is this good YAML", "how can I improve this context", or "what
  should I do next". Also trigger when the user shares both a markdown and a
  YAML file together without a clear conversion request — they likely want a
  comparison, not a conversion. This skill observes and advises; it does not
  transform documents. Transformations are handled by yamlify, yamlify-advanced,
  and deyamlify.
---

# Yamlify-Review

Audit markdown and YAML files in yamlification workflows. Identify what's
working, what's broken, and what to do next. Output is a structured review
report — not a transformed document.

---

## Core Philosophy

The other skills in this system transform documents. This skill observes them.

A yamlification workflow has failure modes that no transformation skill can
catch: context that silently drifted from its source, YAML that is technically
valid but semantically weak, markdown that gets yamlified at the wrong moment
or into the wrong mode, compression that wastes tokens on noise while losing
signal. These failures show up in downstream agent behavior — wrong assumptions,
missed constraints, hallucinated state — but are hard to trace back to the
context document.

Yaml-review makes these problems visible before they propagate. It reads
what you have, compares it to what the workflow needs, and tells you exactly
what to fix and how.

**What this skill does:**
- Audits a single file (markdown or YAML) for structural health and quality
- Compares a markdown source against its YAML version for fidelity and drift
- Identifies optimization opportunities in existing YAML
- Recommends the right next action: which skill, which mode, which output

**What this skill does not do:**
- Produce transformed output (use yamlify / yamlify-advanced / deyamlify)
- Rewrite files on its own — it surfaces findings for the user to act on
- Judge document content (whether the *decisions* are good) — only the
  *representation* of those decisions

---

## Four Operating Modes

### 1. `audit` (DEFAULT when one file provided)

Full diagnostic on a single file. Works on markdown or YAML.

**Use when:** User shares a file and asks if it's good, ready, or well-formed.

**What it checks:**

For YAML files:
- **Convention compliance**: Does it have `_meta`, `intent`, `constraints`?
  (Required for advanced-mode YAML; flag if missing when the file looks
  like it should be agent-ready)
- **Schema coherence**: Are keys semantic (`auth_token_ttl`) or generic
  (`item_1`, `field_a`)? Are sibling keys at consistent levels of abstraction?
- **Value density**: Are values genuinely dense, or do they repeat the key
  name, pad with adjectives, or carry prose that should be a comment?
- **Comment quality**: Are comments earning their place (nuance, rationale,
  ambiguity) or are they noise (restating the key, describing obvious values)?
- **Compression efficiency**: Are there nested structures that could be
  collapsed to inline flow? Values that could be shortened without loss?
  Sections that add tokens but carry no signal?
- **Stale markers**: Are any values that appear outdated or superseded missing
  a `# STALE` flag? Are existing stale flags blocking agent action?
- **Mode fit**: Does the YAML's density match its declared mode? A `lean` file
  with full-sentence values is misclassified. A `minified` file with comments
  is defeating its own purpose.

For Markdown files:
- **Yamlification readiness**: Is this document structured enough to compress
  well? Or is it primarily narrative — better left as markdown?
- **Redundancy load**: How much of the content is filler, restatement, or
  transitional prose? High redundancy = strong candidate for yamlify.
- **Structural clarity**: Are the semantic concepts clearly delineated, or is
  the document a flat wall of prose that will produce generic keys?
- **Mode recommendation**: Given the document's type, length, and downstream
  use, which mode (lean / minified / max) is appropriate?
- **Skill recommendation**: Does this document need `yamlify` or
  `yamlify-advanced`? (Use advanced if: it will be consumed by another agent,
  it's a spec with hard constraints, or it's part of a multi-session workflow)

---

### 2. `compare` (DEFAULT when two files provided)

Round-trip fidelity check and drift analysis. Requires both the markdown
source and its derived YAML.

**Use when:** User shares both files, or asks "did anything get lost?",
"is this still accurate?", "have these drifted?"

**What it checks:**

**Fidelity (markdown → YAML):**
- Is every factual claim in the markdown represented in the YAML?
  Map each markdown section to a YAML key and flag gaps.
- Did any constraints or hard limits get dropped during compression?
  These are the highest-risk losses — flag them prominently.
- Did any ambiguity or `TBD` from the markdown go missing?
  Lost uncertainty is worse than lost facts — it creates false confidence.
- Were any comments in the YAML used correctly to preserve nuance that
  couldn't fit in a value? Or was nuance silently dropped?
- Are YAML values faithful to the markdown's intent, or did compression
  introduce subtle distortions? (e.g., "should not" compressed to "cannot")

**Drift (YAML vs. current markdown):**
- Does the markdown look newer than the YAML? Check for sections, values,
  or decisions in the markdown that don't appear in the YAML at all.
- Does the YAML contain keys that no longer correspond to anything in the
  markdown? (May indicate markdown was edited after yamlification)
- Are `_meta.last_updated` or `confidence` values consistent with the
  apparent freshness of both files?
- Should a `delta` be run, or does drift warrant a full re-yamlify?

**Output format for compare:**
Three sections — **Lost** (in YAML but not in markdown), **Drifted**
(present in both but inconsistent), **Missing** (in markdown, absent from
YAML) — each with specific findings and severity ratings.

---

### 3. `optimize`

Targeted efficiency audit. Assumes structure and fidelity are acceptable —
focuses purely on compression and signal quality.

**Use when:** User asks "can this be leaner?", "I want to reduce tokens",
"trim this without losing anything", "is there any fat here?"

**What it checks:**
- **Inline flow opportunities**: Nested objects with 2–3 simple scalar values
  can often collapse to `{key: val, key2: val2}` — saving 2–3 lines each.
- **Value verbosity**: Values that use 8 words when 3 would do. Adverbs,
  adjectives, and hedging language that add tokens but not meaning.
- **Redundant nesting**: Parent keys whose only purpose is grouping, but whose
  name repeats in every child key (`auth.auth_token`, `auth.auth_refresh`).
- **Comment bloat**: Comments that restate the key or value. One-word comment
  that could be the value. Multi-line comment that should be a nested key.
- **Dead keys**: Keys with `null`, empty string, or placeholder values
  (`TBD`, `n/a`) that add structure with no signal — consolidate into
  `open_questions` or drop.
- **Duplicate signal**: Two keys that express the same constraint in different
  words. Two sections with overlapping content.
- **Mode mismatch**: If the file is labeled `lean` but reads like `max`, flag it.
  The mode label is a contract — downstream agents calibrate expectations from it.

**Output format for optimize:**
A prioritized list of specific changes, each with: location (key path),
what to change, and estimated token savings. End with a rough total reduction
estimate. Do not rewrite the file — output the change list only.

---

### 4. `recommend`

Workflow-level advisory. Given the current situation, what should the user
do next?

**Use when:** User asks "what should I do?", "which skill should I use?",
"what mode makes sense here?", "how should I handle this?", or describes a
situation without a specific file.

**What it considers:**
- **Session phase**: Start of a new session → lean. Mid-session reminder →
  minified. Session ending, output needed → delta or deyamlify.
- **Downstream consumer**: Human reading it → deyamlify. Another agent using
  it as context → yamlify-advanced with agent-ready conventions. Same agent,
  later in the session → minified.
- **Document type**: Spec with hard constraints → advanced (surfaces them in
  `constraints` block). Notes or scratch → yamlify basic. Architecture doc
  with nuance → max mode.
- **Existing state**: YAML exists but source changed → delta, not re-yamlify.
  YAML exists and source hasn't changed → minified for mid-session reminder.
  No YAML exists → full yamlify at appropriate mode.
- **Token pressure**: If context window is getting long, the answer is almost
  always minified or delta — not re-injecting a full lean document.

**Output format for recommend:**
A direct recommendation: which skill, which mode, why. Then one paragraph
of reasoning. Then any conditions or caveats. Short — this is a decision
aid, not a report.

---

## Mode Selection

| User says / situation | Mode |
|---|---|
| "review this yaml" / "is this good?" | audit |
| "check this markdown" / "is this ready to yamlify?" | audit |
| "compare these" / both files provided | compare |
| "did anything get lost?" / "have these drifted?" | compare |
| "make this leaner" / "reduce tokens" / "trim this" | optimize |
| "what should I do?" / "which skill/mode?" | recommend |
| One file, no clear instruction | audit |
| Two files, no clear instruction | compare |

---

## Report Structure

Every audit or compare output follows this structure. Optimize and recommend
have abbreviated formats described in their sections.

```
## Yamlify-Review: {filename or "Document"}
Mode: {audit | compare | optimize | recommend}
Date: {today}

### Summary
{2–4 sentence overview: what's the overall state, what's the biggest issue,
what's the recommended immediate action}

### Health Signals
{Table or list of dimensions with status: ✓ / ⚠ / ✗}

### Findings
{Numbered list of specific issues, each with:
  - Location: key path, section name, or line reference
  - Severity: critical | moderate | minor
  - Issue: what's wrong
  - Fix: what to do about it (concrete, not vague)}

### Recommendations
{Ordered list of actions. First action = highest value / lowest effort.
Each action names the specific skill and mode to use if applicable.}
```

---

## Severity Definitions

**Critical** — Likely to cause downstream agent errors or false behavior:
- Dropped constraint from source markdown
- Missing `intent` in agent-ready YAML
- `confidence: high` on a clearly stale document
- TBD in source compressed away entirely (false certainty injected)
- Key value silently distorted during compression ("should not" → "cannot not")

**Moderate** — Reduces quality without causing outright failures:
- Missing `_meta` block when document is clearly agent-facing
- Generic key names (`data`, `info`, `details`) instead of semantic ones
- Comment restating the key rather than adding nuance
- YAML mode labeled `lean` but values are full prose sentences
- Drift between source and YAML that doesn't yet break anything

**Minor** — Cosmetic or efficiency opportunities:
- Inline flow possible but not used (no semantic impact, saves a few tokens)
- Redundant nesting that could collapse one level
- Dead keys that add noise without adding signal
- Inconsistent key naming style within the same file

---

## Special Cases

### Reviewing a `minified` YAML

Minified is intentionally lossy. Don't flag missing nuance as a finding —
that's the point. Instead, check:
- Is it being used at the right phase? (mid-session only — not start)
- Does it still carry intent, constraints, and next action?
- Is it actually short, or has it grown back toward lean size?

### Reviewing a `delta` YAML

Delta is a diff, not a full state. Check:
- Are `changed`, `added`, `removed` sections all present?
- Is `_base` identified?
- Is there a base document to merge against, or is this delta orphaned?

### When only one file is provided and it's unclear which direction

Ask: is this the source or the output? A markdown file is almost always a
source. A YAML file with `_meta` is almost always output. If genuinely
unclear, state the assumption and run audit on it.

### When the markdown is not a good yamlification candidate

Some documents shouldn't be yamlified at all — narrative essays, creative
writing, heavily contextual prose where structure would destroy meaning.
Say so directly. Recommend keeping as markdown. Don't suggest yamlify as
a courtesy.

---

## Tone and Output Style

- Lead with the most important finding. Don't bury the critical issue at item 7.
- Be specific: "key `auth.mechanism` repeats the parent key name — rename to
  `method` or `type`" beats "some keys could be named better".
- Cite exact key paths, section names, or quoted values when referencing findings.
- Don't hedge findings: "this appears to possibly be" → "this is".
- Recommendations must name a concrete next action. "Consider optimizing" is
  not a recommendation. "Run yamlify-advanced in lean mode on this file" is.
- Never pad the report. A three-finding audit should be short. A fifteen-finding
  audit should be longer. Length tracks findings, not effort.
