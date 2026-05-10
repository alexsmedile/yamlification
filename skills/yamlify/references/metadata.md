# Metadata Reference

`meta:` is optional. Include only the fields that apply. Never fill a template.
All values must come from the source document or be explicitly supplied by the
user — never invent or infer. If a value is unknown, omit the field.

**Source provenance rule:** use the actual filename the user provides or that
appears in the document. If no filename is known, omit `source:` entirely —
do not construct or guess a name.

## Fields

| Field | Include when |
|---|---|
| `source:` | Filename is known and the YAML may outlive the session. Use `source_url:` instead when the source lives online. |
| `source_url:` | Source lives online — use this instead of `source:`, not alongside it, unless both a local path and a remote URL are genuinely needed |
| `source_date:` | Source was written/updated at a different time than the conversion |
| `date:` | Freshness of the YAML itself matters to the consumer |
| `version:` | Source document explicitly states a version |
| `status:` | Work is in progress (`draft`, `review`) — omit for static/final files |
| `mode:` | Full traceability needed — otherwise keep as top comment (`# lean`) |

## Examples

```yaml
# in-session — source known, nothing else needed
meta:
  source: auth-spec.md

# cross-session or shared with another agent
meta:
  source: auth-spec.md
  date: 2025-05-10

# full traceability — versioned, in-progress artifact
meta:
  source: auth-spec.md
  source_url: https://github.com/org/repo/docs/auth-spec.md
  source_date: 2025-04-28
  date: 2025-05-10
  version: 1.2      # stated in source
  status: draft     # in-progress — omit when final
  mode: lean        # omit when already present as top comment
```

## `intent` and `constraints`

Optional root-level fields — include when a downstream agent needs to orient
quickly without reading the full document:

```yaml
intent: one-line statement of what this document is about

constraints:
  - hard limit or non-negotiable
# Omit entirely if none exist — an empty list adds noise.
```
