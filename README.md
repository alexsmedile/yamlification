# yaml-skills

A set of Claude skills for converting documents into AI-optimized YAML context — and back into human-readable markdown. Built for AI agent workflows where token efficiency, context management, and session continuity matter.

---

## Why this exists

Markdown is written for humans. It has narrative flow, transitional sentences, redundant headers, and decorative prose. All of that is noise when the reader is an AI agent working through a long workflow.

This pack introduces **yamlification**: a structured approach to converting any document into a compact, semantically organized YAML format that agents can parse, reference, and update efficiently. The transformation is not summarization — it is restructuring. Meaning is preserved; format is discarded.

The pack also includes **deyamlification**: expanding agent YAML back into clean, well-written markdown for humans.

---

## Skills in this pack

### `yamlify` — basic

> Convert any markdown file into AI-optimized YAML.

Three output modes, triggered by natural language:

| Mode | Command | Token reduction | Use when |
|---|---|---|---|
| `lean` | "yamlify this" *(default)* | ~60% | Starting a project, loading context |
| `minified` | "minified" / "smallest" | ~80% | Mid-session reminder, keeping an agent on track |
| `max` | "max" / "full yaml" | ~20% | Complex specs where nuance is critical |

**Core mechanic:** Keys are semantic labels. Values are dense payloads. YAML comments carry nuance that doesn't compress cleanly into a value — they are first-class, not an afterthought.

```markdown
## Authentication

Our system uses JWT tokens for authentication. Tokens are issued at login
and expire after 24 hours. Refresh tokens are valid for 30 days. All
endpoints except /health and /login require a valid Bearer token.
```

becomes:

```yaml
# lean
---
auth:
  mechanism: JWT bearer token
  token_ttl: 24h
  refresh_ttl: 30d
  # All endpoints require Bearer token except:
  public_endpoints: [/health, /login]
```

---

### `yamlify-advanced` — agent-ready

> Everything in `yamlify`, plus conventions that make YAML reliably parseable by downstream agents.

Adds a fourth mode and enforces a structural contract on all output.

**Agent-ready conventions** (applied to all modes except `minified`):

Every output begins with three root-level keys an agent can find without reading the full document:

```yaml
_meta:
  phase: start       # start | mid | end
  mode: lean
  source: spec.md
  confidence: high   # high | partial | stale

intent: one-line purpose of this document/project

constraints:
  - hard limit one
  - hard limit two
```

**Stale markers** let agents know which fields are outdated without rebuilding the whole context:

```yaml
sla: sub-30s  # STALE — revised to 2min after load testing
```

**Fourth mode: `delta`**

When context evolves across sessions, you don't need to re-inject the full document. A delta only contains what changed:

```yaml
# delta
---
_meta:
  mode: delta
  _base: lean_v1
  last_updated: 2025-04-14

changed:
  status: in_progress → complete
  owner: TBD → platform-team

added:
  retry_logic:
    max_attempts: 3
    backoff: [1m, 5m, 30m]

removed:
  - open_question: sla_undefined  # resolved
```

| Mode | Command | Token reduction | Use when |
|---|---|---|---|
| `lean` | "yamlify this" *(default)* | ~60% | Starting a project |
| `minified` | "minified" | ~80% | Mid-session reminder |
| `max` | "max" | ~20% | Rich specs, architecture docs |
| `delta` | "delta" / "what changed" | varies | Updating context across sessions |

---

### `deyamlify` — expand back to markdown

> Convert agent YAML back into clean, human-readable markdown.

The output is a freshly written document — not a mechanical YAML dump. YAML is the source of truth; the skill authors a real document from it.

Three output modes:

| Mode | Command | Use when |
|---|---|---|
| `doc` | "deyamlify this" *(default)* | General working document, README, spec |
| `brief` | "brief" / "summary" | Status update, handoff, ~100 words |
| `spec` | "spec" / "formal doc" | Technical specification with standard structure |

**Key behaviour:** YAML comments are expansion cues. When the skill encounters:

```yaml
cache_ttl: 5m
# Short TTL prioritizes freshness; revisit if p99 latency degrades
```

It expands to:

```markdown
The cache TTL is set to 5 minutes. This was chosen to prioritize data
freshness over cache efficiency — if p99 latency becomes a concern,
this value should be revisited.
```

`TBD` values become named open questions at the end of the document. `STALE` markers become a "Known Gaps" section.

---

## The full round-trip

```
Source document (.md)
       │
       ▼
   [ yamlify ]
       │
   ┌───┴──────────────┐
   │                  │
lean / max        minified / delta
   │                  │
   │            agent context / memory
   │
   ▼
[ deyamlify ]
       │
       ▼
 Human document (spec / brief / doc)
```

A typical workflow:

1. **Project start** — yamlify the spec into `lean`. Load as agent context.
2. **Mid-session** — inject a `minified` reminder when the conversation grows long.
3. **Something changes** — generate a `delta` instead of re-injecting the full context.
4. **Handoff to a human** — deyamlify the current `lean` into a `doc` or `brief`.

---

## Installation

This repo can be used either as a standalone skill pack or as a Codex plugin.

### Claude / manual skill install

Install each skill by uploading the corresponding `SKILL.md` file through Claude's skill interface, or by adding the skill folder to your local skills directory.

```
yaml-skills/
└── skills/
    ├── yamlify/
    │   └── SKILL.md
    ├── yamlify-advanced/
    │   └── SKILL.md
    └── deyamlify/
        └── SKILL.md
```

**Which skill to install?**

- If you're using Claude for general writing or document work: install `yamlify` only.
- If you're building AI agent workflows, skills, or multi-session projects: install `yamlify-advanced` and `deyamlify`.
- You can install all three — they have distinct names and won't conflict.

### Codex plugin install

This repository is also structured as a repo-root Codex plugin:

```
yaml-skills/
├── .codex-plugin/
│   └── plugin.json
├── .agents/plugins/
│   └── marketplace.json
└── skills/
```

If you use this repo directly in Codex, the repo marketplace entry points to the
plugin root (`./`), so the root of the repository is the plugin folder.

---

## Quick reference

### yamlify / yamlify-advanced

| Say this | Get this |
|---|---|
| "yamlify this" | lean YAML (default) |
| "lean yaml" | lean YAML |
| "minified" / "reminder version" | minified YAML |
| "max" / "full yaml" | max YAML |
| "delta" / "what changed" | delta YAML (advanced only) |
| "all three" / "all versions" | lean + minified + max |
| "update the yaml" + existing YAML | delta (advanced only) |

### deyamlify

| Say this | Get this |
|---|---|
| "deyamlify this" | doc (default) |
| "make this readable" / "expand this" | doc |
| "brief" / "summary" / "tl;dr" | brief (~100 words) |
| "spec" / "formal doc" / "ADR" | formal specification |
| "status update" / "handoff" | brief |

---

## Token reduction reference

Approximate reduction from source markdown, measured across typical technical documents:

| Mode | Reduction | Best for |
|---|---|---|
| lean | 50–70% | General context loading |
| minified | 70–85% | Agent reminders, long sessions |
| max | 15–25% | Nuance-critical documents |
| delta | varies | Incremental context updates |

---

## Design principles

**Restructuring, not summarization.** All meaning from the source is preserved — only the format changes. Nothing is dropped unless it is genuinely redundant.

**Comments are first-class.** When nuance doesn't fit cleanly into a key/value pair, it goes into a YAML comment. Comments are expansion cues for `deyamlify` and context cues for agents.

**Agent-readability is a contract.** The `yamlify-advanced` skill enforces `_meta`, `intent`, and `constraints` at the root of every document so that any downstream agent can orient itself in three reads.

**The round-trip is lossless in intent, if not in form.** A document yamlified and then deyamlified will not be identical to the original — but it will be accurate to the original's meaning, and often better organized.

---

## Contributing

Issues and pull requests welcome. If you build a workflow on top of these skills or find edge cases the current prompts don't handle well, please open an issue with an example.

---

## License

MIT
