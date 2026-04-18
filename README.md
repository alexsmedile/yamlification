# YAMLification

*Turn Markdown into token-efficient YAML, then turn it back into clean Markdown.*

![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-1.0.0-green)
![Platform](https://img.shields.io/badge/platform-Claude%20%7C%20Codex%20%7C%20compatible-lightgrey)
![Category](https://img.shields.io/badge/category-productivity-2F6B5F)

> A small skill pack for AI workflows that need tighter context, lower token usage, and cleaner handoffs between agents and humans.

---

## The Problem

Markdown is easy for humans to read, but inefficient for agents to carry through long sessions. Specs, READMEs, notes, and handoff docs often include repetition, prose, and structure that is helpful to people but wasteful for context windows.

When that context grows, agents lose track, token costs rise, and cross-session continuity gets messy.

## What This Does

`yamlification` gives you five focused skills:

- `yamlify` converts Markdown into compact YAML for agent context.
- `yamlify-advanced` adds agent-ready structure such as `_meta`, `intent`, `constraints`, and `delta` updates.
- `deyamlify` turns YAML back into readable Markdown for humans.
- `yamlify-review` audits and compares Markdown and YAML files for quality, fidelity, and drift.
- `ultrapack` applies extreme compression (80–95%) to documents too large for standard YAML context.

This is not summarization. It is format conversion with structure preserved.

---

## Quick Start

### Claude Code — install from marketplace

Add this repo as a marketplace, then install the plugin:

```bash
/plugin marketplace add alexsmedile/yamlification
/plugin install yamlification@alexsmedile-yamlification
```

Or open the interactive plugin manager and browse from there:

```bash
/plugin
```

### Claude Code — install from local clone

```bash
git clone https://github.com/alexsmedile/yamlification
claude --plugin-dir ./yamlification
```

### Codex — install from GitHub

```bash
codex plugin install https://github.com/alexsmedile/yamlification
```

### Codex — install from local clone

```bash
git clone https://github.com/alexsmedile/yamlification
codex plugin install ./yamlification
```

### Use it

Ask your agent:

```text
yamlify this README into lean YAML
```

Expected result:

- A compact YAML version of the source document
- Lower-token context that is easier for agents to parse and reuse
- Optional `minified`, `max`, or `delta` variants depending on the skill and prompt

---

## Skills

| Skill | What it does | Best for |
|---|---|---|
| `yamlify` | Converts Markdown into compact YAML in `lean`, `minified`, or `max` mode | General document compression |
| `yamlify-advanced` | Adds agent-ready conventions and a `delta` mode for changes over time | Multi-session workflows and downstream agent use |
| `deyamlify` | Expands YAML back into readable Markdown in `doc`, `brief`, or `spec` mode | Handoffs, reports, READMEs, and formal docs |
| `yamlify-review` | Audits single files or compares markdown/YAML pairs for quality and drift | QA before using context in a workflow |
| `ultrapack` | Extreme compression using a layer priority model and 13 encoding techniques | Documents too large for standard YAML context |

---

## Modes At A Glance

Three YAML output modes are triggered by natural language. The default is `lean`.

| Mode | Command | Approx. token reduction | Typical use |
|---|---|---|---|
| `lean` | `"yamlify this"` | ~60% | Starting a project, loading context |
| `minified` | `"minified"` / `"smallest"` | ~80% | Mid-session reminder, keeping an agent on track |
| `max` | `"max"` / `"full yaml"` | ~20% | Complex specs where nuance is critical |
| `delta` | `"delta"` / `"what changed"` | varies | Updating context across sessions |
| `doc` | `"deyamlify this"` | n/a | Human-readable output |
| `brief` | `"brief"` / `"summary"` | n/a | Handoffs and status updates |
| `spec` | `"spec"` / `"formal doc"` | n/a | Structured documentation |
| `audit` | `"review this"` / `"is this good?"` | n/a | Single-file health check (yamlify-review) |
| `compare` | `"compare these"` / `"did anything get lost?"` | n/a | Fidelity and drift check (yamlify-review) |
| `optimize` | `"make this leaner"` / `"trim this"` | n/a | Token efficiency audit (yamlify-review) |
| `scaffold` | `"ultrapack this"` | ~50–65% | Session start, full orientation (ultrapack) |
| `core` | `"ultrapack this"` | ~70–82% | Mid-session execution (ultrapack) |
| `ultra` | `"maximum compression"` | ~85–95% | Critical context pressure (ultrapack) |

**Core mechanic:** keys are semantic labels. Values are dense payloads. YAML comments carry nuance that does not compress cleanly into a value, so they remain first-class instead of being treated as throwaway notes.

---

## How It Works

```text
Markdown document
      |
      v
  yamlify-review (audit)         ← optional QA step
      |
      v
  yamlify / yamlify-advanced / ultrapack
      |
      +--> lean / minified / max / delta / scaffold / core / ultra YAML
      |
      +--  yamlify-review (compare / optimize)  ← quality check on output
      |
      v
    deyamlify
      |
      v
Human-readable Markdown
```

Typical workflow:

1. Start with a README, spec, or notes file.
2. Optionally run `yamlify-review audit` to assess yamlification readiness.
3. Convert to `lean` YAML with `yamlify`; use `ultrapack` if the document is very large.
4. Use `minified` or `ultrapack core` when the session gets long, or `delta` when only part of the context changed.
5. Run `yamlify-review compare` to check fidelity between source and compressed output.
6. Convert the current YAML back into a `doc`, `brief`, or `spec` when a human needs to review it.

---

## Example

Source Markdown:

```markdown
## Authentication

Our system uses JWT tokens for authentication. Tokens are issued at login
and expire after 24 hours. Refresh tokens are valid for 30 days. All
endpoints except /health and /login require a valid Bearer token.
```

YAML output:

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

That same YAML can then be expanded back into readable Markdown with `deyamlify`.

---

## Why This Repo Is Different

| Without YAMLification | With YAMLification |
|---|---|
| Agents read full prose documents every time | Agents work from compact structured context |
| Session reminders are ad hoc and inconsistent | `minified` output gives a stable reminder format |
| Context updates require re-injecting whole docs | `delta` captures only what changed |
| YAML handoffs stay agent-centric | `deyamlify` turns them back into human docs |

---

## Installation Options

### Skill folders

```text
skills/
├── yamlify/
├── yamlify-advanced/
├── deyamlify/
├── yamlify-review/
└── ultrapack/
```

Install only what you need:

- Use `yamlify` for general document compression.
- Use `yamlify-advanced` when another agent or workflow will consume the YAML.
- Use `deyamlify` when YAML needs to become a document again.
- Use `yamlify-review` to audit YAML quality or check for drift between source and output.
- Use `ultrapack` when documents are too large for standard YAML context (80–95% reduction).

### Plugin metadata

This repo includes plugin manifests for Claude Code and Codex:

```text
.claude-plugin/plugin.json   ← Claude Code host
.codex-plugin/plugin.json    ← Codex host
.agents/plugins/marketplace.json
```

That makes the repository usable both as a skill library and as an installable plugin package via `/plugin` (Claude Code) or `codex plugin install` (Codex).

---

## Who This Is For

- People building long-running AI workflows with Markdown-heavy context
- Users who want lower-token, more structured agent memory
- Teams that need cleaner handoffs between agent output and human documentation
- Claude, Codex, and compatible-host users working from local skills

## Who It Is Not For

- Users looking for a YAML parser library or SDK
- Teams expecting a hosted service, API, or telemetry dashboard
- Cases where the original human-friendly Markdown should remain the working format end to end

---

## Design Principles

- **Restructuring, not summarization.** Meaning stays; format changes.
- **Comments carry nuance.** YAML comments preserve context that does not fit cleanly into a key/value pair.
- **Agent readability is a contract.** The advanced skill surfaces `_meta`, `intent`, and `constraints` so downstream agents can orient quickly.
- **Round-trip matters.** YAML is for agent efficiency; Markdown is for human review and sharing.

---

## Repository Layout

```text
yamlification/
├── skills/
│   ├── yamlify/
│   ├── yamlify-advanced/
│   ├── deyamlify/
│   ├── yamlify-review/
│   └── ultrapack/
├── assets/
├── .claude-plugin/
├── .codex-plugin/
├── PRIVACY.md
├── TERMS.md
└── LICENSE
```

---

## Privacy And Terms

`yamlification` is a local prompt-based package. It does not include its own remote service, telemetry pipeline, or account system. Data handling depends on the host product where you install and run the skills.

See [PRIVACY.md](PRIVACY.md) and [TERMS.md](TERMS.md).

---

## License

MIT
