# Classification

| Type | Approach |
|---|---|
| Spec / ADR | High compression. Map sections directly to keys. |
| README | Medium. Keep purpose, install, constraints. Drop marketing prose. |
| Meeting notes | High. Map source sections to keys. Compress prose to dense values; drop discussion narrative. Preserve section order. If a thematic restructure (e.g. decisions/open_questions/actions) would significantly improve agent scannability, propose it — but default to source structure unless the user confirms. |
| Config / reference | Low. Preserve structure; collapse prose descriptions. |
| Narrative / essay | Do not convert silently — tell the user it's argument-first and ask if they want conclusions-only YAML. If yes, mark output as partial and reference the source. |