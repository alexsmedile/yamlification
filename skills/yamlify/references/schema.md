# Key Schema Design

## Common failure modes

**Too flat** — prefix repetition instead of grouping:
```yaml
# bad
auth_mechanism: JWT
auth_token_expires_after: 24h
auth_public_endpoints: [/health, /login]

# good
auth:
  mechanism: JWT
  token_expires_after: 24h
  public_endpoints: [/health, /login]
```

```yaml
# bad
brand_name: Acme
brand_tagline: Built to last
brand_colors: [navy, gold]

# good
brand:
  name: Acme
  tagline: Built to last
  colors: [navy, gold]
```

---

**Too nested** — unnecessary parent wrapping:
```yaml
# bad
auth:
  config:
    tokens:
      mechanism: JWT

# good
auth:
  mechanism: JWT
```

```yaml
# bad
brand:
  identity:
    core:
      name: Acme

# good
brand:
  name: Acme
```

---

**Prose keys** — keys that describe instead of name:
```yaml
# bad
our_system_uses: JWT tokens for authentication
tokens_are_issued: at login and expire after 24 hours

# good
auth:
  mechanism: JWT
  token_expires_after: 24h
```

```yaml
# bad
the_brand_is_targeting: young professionals in urban areas
what_makes_us_different: we focus on simplicity over features

# good
target:
  audience: young professionals, urban
differentiation: simplicity over features
```

---

## Schema rules

- **Preserve source structure by default.** If the document has clear sections and headings, map them directly to keys — don't reorganize. The job is to mechanify the structure, not redesign it.
- **Propose before restructuring.** If a significantly better schema is apparent, tell the user what you'd change and why before applying it. Never silently reorganize.
- Use the document's own terminology, shortened (`authentication` → `auth`, `differentiation` → `diff` only if unambiguous)
- One level of nesting per logical grouping; two levels max
- Key names are concepts, not sentences — but must be self-documenting without a glossary (see Key naming rule in SKILL.md)
- Lists of parallel items → sequences; related attributes → mappings
- Decisions → values; rationale and caveats → comments

## Machine-readability rules

Apply to every output regardless of mode:

- Keys: `snake_case`, no spaces, no special characters
- Values: scalars, sequences, or mappings — never prose disguised as a value; explanation goes in a comment
- Short flat lists: inline `[a, b, c]`; sub-structured or 5+ items: block sequence
- Dates: `YYYY-MM-DD`; durations: `24h`, `30d`, `500ms` — never prose ("24 hours")
- Numbers: always attach unit (`500ms` not `500`)
- Enums over booleans: `status: draft` not `status: true`
