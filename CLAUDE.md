# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A skill pack for Claude/Codex AI workflows. No build system, no tests, no runtime code. Everything here is prompt-based — SKILL.md files are instructions consumed by AI agents, not executed code.

## Repository Layout

```
skills/
├── yamlify/SKILL.md          # Markdown → compact YAML (lean/minified/max)
├── yamlify-advanced/SKILL.md # Same + agent-ready conventions (_meta, intent, constraints, delta)
├── deyamlify/SKILL.md        # YAML → human-readable Markdown (doc/brief/spec)
├── yamlify-review/SKILL.md      # Audit/compare/optimize markdown and YAML files
└── ultrapack/SKILL.md        # Extreme compression for oversized documents (scaffold/core/ultra)
.claude-plugin/plugin.json    # Plugin metadata for Claude hosts
.codex-plugin/plugin.json     # Plugin metadata for Codex hosts
.agents/plugins/marketplace.json
```

## Skill Architecture

Five skills, three roles:

**Markdown → YAML** (compression for agent context):
- `yamlify` — general-purpose compression with three modes
- `yamlify-advanced` — adds `_meta`/`intent`/`constraints` header block and a `delta` mode for cross-session diffs
- `ultrapack` — extreme compression (80–95%) using a four-layer information priority model and 13 encoding techniques; for documents too large for standard yamlify

**YAML → Markdown** (expansion for human consumption):
- `deyamlify` — three output modes (doc, brief, spec); writes fresh prose, not mechanical YAML dumps

**Quality / advisory** (observe, do not transform):
- `yamlify-review` — four modes (audit, compare, optimize, recommend); reports findings and next actions; never rewrites files directly

**Modes across skills:**

| Mode | Token target | Skill | Use case |
|------|-------------|-------|----------|
| `lean` | ~40–65% reduction | yamlify / advanced | Default; session start |
| `minified` | ~80% reduction | yamlify / advanced | Mid-session reminder |
| `max` | ~20% reduction | yamlify / advanced | Complex specs, nuance-critical |
| `delta` | varies | yamlify-advanced | Diff between YAML versions |
| `doc` | n/a | deyamlify | Human-readable full doc |
| `brief` | n/a | deyamlify | Handoff/status update |
| `spec` | n/a | deyamlify | Formal ADR/API doc |
| `audit` | n/a | yamlify-review | Single-file health check |
| `compare` | n/a | yamlify-review | Source vs. YAML fidelity/drift |
| `optimize` | n/a | yamlify-review | Token efficiency audit |
| `recommend` | n/a | yamlify-review | Workflow-level next-action advice |
| `scaffold` | ~50–65% reduction | ultrapack | Session start; full orientation |
| `core` | ~70–82% reduction | ultrapack | Mid-session execution |
| `ultra` | ~85–95% reduction | ultrapack | Critical context pressure |

## Core Design Rule

Transformation is **restructuring, not summarization** — all meaning preserved, format discarded. YAML comments are first-class (carry nuance that doesn't fit in key/value pairs). `ultrapack` extends this with explicit encoding techniques (predicate strip, noun-chain collapse, skeleton encoding, etc.) for extreme reduction.

## Skill Relationships

- `ultrapack` output is compatible with `deyamlify` (doc/spec modes), but will be incomplete if Layer 3 content was dropped — note the compression level to the reader
- `yamlify-review` reads output from any compression skill; `compare` mode requires both the source markdown and the derived YAML
- `yamlify-advanced` conventions (`_meta`, `intent`, `constraints`) must remain consistent with `deyamlify`'s mapping table

## Editing Skills

When modifying a SKILL.md:
- Frontmatter (`name`, `description`) is the trigger definition — changes here affect when the skill fires in host environments
- The description field drives auto-trigger matching; keep it exhaustive for the cases it should catch
- Mode tables and step-by-step process sections are the behavioral contract — changes affect output structure
- `ultrapack` encoding techniques (Section "Encoding Techniques") define the compression vocabulary — changes affect all compression levels
- `yamlify-review` severity definitions must remain consistent with the kinds of errors the other skills can produce

## Plugin Metadata

Two separate plugin manifests exist for different host ecosystems. When bumping version or changing metadata, update both `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json`.
