# YAML
<!-- source: https://en.wikipedia.org/wiki/YAML -->

YAML (/ˈjæməl/ YAM-əl) is a human-readable data serialization language. Commonly used for
configuration files and in applications where data is being stored or transmitted. Targets many
of the same communications applications as XML but has a minimal syntax that intentionally
differs from SGML. Uses Python-style indentation for nesting; supports JSON-style {} and []
mixed in the same file.

Custom data types are allowed, but YAML natively encodes scalars (strings, integers, floats),
lists, and associative arrays (maps/dictionaries/hashmaps). Colon-centered key-value syntax
inspired by RFC 822 email headers; document separator --- borrowed from MIME (RFC 2046).

## History and Name

First proposed by Clark Evans in 2001, designed with Ingy döt Net and Oren Ben-Kiki.
Originally stood for "Yet Another Markup Language" (tongue-in-cheek reference to the HTML/XML/SGML
era). Repurposed in 2002 as "YAML Ain't Markup Language" (recursive acronym) to distinguish its
data-oriented purpose from document markup.

Versions: 1.0 (2004-01-29), 1.1 (2005-01-18), 1.2.0 (2009-07-21), 1.2.2 (2021-10-01, current)

## Design / Syntax

- Whitespace indentation denotes structure; tabs not allowed
- Comments: # character, separated from tokens by whitespace
- Lists: leading hyphen per item, or inline [a, b, c]
- Key-value: colon+space, or inline {key: val}
- Strings: unquoted by default; double-quotes allow C-style escapes; single-quotes allow '' only
- Block scalars: | (preserve newlines) or > (fold newlines)
- Multiple documents in one stream: separated by ---; optionally ended with ...
- Anchors: & to define, * to reference (works for all data types)
- Type tags: !! prefix (e.g. !!float, !!str, !!binary)
- Directives: % prefix — %YAML for version, %TAG for URI shortcuts

## Data Types

Core: floats, ints, strings, lists, maps. Extended (spec-defined, not universally implemented):
binary data, timestamps, sets, ordered maps, hexadecimal.

Type autodetection: YAML infers types automatically. Explicit disambiguation via quotes or !!tags.
Examples: `Yes` → Boolean True (YAML 1.1) or string "Yes" (YAML 1.2); `123` → int; `123.0` → float.

User-defined types: supported via single ! tag in application-specific parsers.

Composite keys: multiple values as a single key — useful for coordinate transforms, multi-field
identifiers, compound test conditions.

## Comparison with Other Formats

**vs JSON:** YAML 1.2 made JSON an official subset. YAML adds: comments, extensible types,
relational anchors, unquoted strings, key-order-preserving maps. JSON serialization/deserialization
is significantly faster.

**vs TOML:** TOML designed as .ini advancement. YAML uses significant indentation; TOML uses dot
notation for structure. Both target config files; readability opinions differ.

**vs XML:** YAML lacks tag attributes; uses extensible type declarations instead. No built-in
schema validation (external: Doctrine, Kwalify, Rx). YAXML allows XSLT and XML schema tools on
YAML data.

## Security

Purely a data-representation language — no executable commands. However: language-specific tags
allow arbitrary object instantiation. Parsers that support these enable injection attacks (e.g.
!!python/object/apply:os.system). Use yaml.safe_load in Python; avoid unrestricted loaders for
untrusted input.

## Criticism

- Dynamic language parsers can execute commands silently via !! tags
- Large files: indentation errors go unnoticed
- Type autodetection causes surprises (Yes → bool, version numbers → floats)
- Truncated files parse as valid (no mandatory terminator)
- Standard complexity led to inconsistent implementations and portability issues

Alternatives driven by YAML criticism: StrictYAML, NestedText.

## File Format

Extension: .yaml (recommended since 2006), .yml (common alias)
MIME type: application/yaml (finalized 2024, RFC by Polli/Wilde/Aro)
Encoding: UTF-8, UTF-16, UTF-32 (UTF-32 required only for JSON compatibility)
