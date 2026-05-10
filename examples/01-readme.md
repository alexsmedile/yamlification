# YAMLification

![License](https://img.shields.io/badge/license-MIT-blue)
![Version](https://img.shields.io/badge/version-1.1.0-green)

> A small skill pack for AI workflows that need tighter context, lower token
> usage, and cleaner handoffs between agents and humans.

## The Problem

Markdown is easy for humans to read, but inefficient for agents to carry
through long sessions. Specs, READMEs, notes, and handoff docs often include
repetition, prose, and structure that is helpful to people but wasteful for
context windows.

When that context grows, agents lose track, token costs rise, and cross-session
continuity gets messy.

## What This Does

`yamlification` gives you five focused skills:

- `yamlify` converts Markdown into compact YAML for agent context.
- `yamlify-advanced` adds agent-ready structure such as `_meta`, `intent`,
  `constraints`, and `delta` updates.
- `deyamlify` turns YAML back into readable Markdown for humans.
- `yamlify-review` audits and compares Markdown and YAML files for quality,
  fidelity, and drift.
- `ultrapack` applies extreme compression (80–95%) to documents too large for
  standard YAML context.

This is not summarization. It is format conversion with structure preserved.

## Install

### Claude Code

```bash
/plugin marketplace add alexsmedile/yamlification
/plugin install yamlification@alexsmedile-yamlification
```

### Codex

```bash
npx codex-marketplace add alexsmedile/yamlification --plugin
```

### Skills CLI

```bash
npx skills add alexsmedile/yamlification
```

### Manual

```bash
git clone https://github.com/alexsmedile/yamlification
claude --plugin-dir ./yamlification
```

## License

MIT
