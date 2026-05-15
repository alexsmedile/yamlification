# Changelog

All notable changes are documented here.
Format: [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)

---

## [1.2.1] — 2026-05-16

### Fixed
- `.codex-plugin/plugin.json` — restored the required `hooks` field; trimmed non-functional metadata
- `.agents/plugins/marketplace.json` — fixed broken `local`/`../../` source → GitHub URL source
- README and examples — corrected invalid `codex plugin install` commands to `npx codex-marketplace add ... --plugin`
- Added `hooks/hooks.json` + `hooks/hooks-codex.json` and a git-guard pre-commit hook for version-drift protection

---

## [1.2.0] — 2026-05-11

### Breaking
- `ultrapack` renamed to `yamlify-ultra` — update any direct skill references or install paths

### Added
- `yamlify` key naming guide: mode-aware rules — self-documenting names in `lean`/`full`, abbreviations acceptable in `minified`
- Version fields added to `deyamlify` (1.0.0), `yamlify-review` (1.0.0), `yamlify-ultra` (1.1.0); `versions/` snapshots created for all three

### Changed
- `yamlify` lean mode: added everyday (non-technical) transformation example alongside the JWT example
- `yamlify` comments guide: four explicit use cases now documented inline (clarify value, preserve nuance, flag uncertainty, capture intent)
- `yamlify` minified description: removed "Mid-session reminder" label; prose cleaned up
- `yamlify` `full` mode: "(richest)" subtitle dropped; mode alias `max` retained in trigger table only

### Removed
- `yamlify-advanced` skill removed from repository

---

## [1.1.0] — 2026-05-10

### Added
- `yamlify-review` skill — four modes: `audit` (single-file health check), `compare` (source vs. YAML fidelity), `optimize` (token efficiency), `recommend` (workflow-level advice); observe-only, never rewrites files directly
- `ultrapack` skill — extreme compression (80–95%) using a four-layer information priority model and 13 encoding techniques; for documents too large for standard yamlify

### Changed
- `deyamlify` updated for ultrapack compatibility: `doc` and `spec` modes now handle ultrapack output with a completeness note when Layer 3 content was dropped
- Plugin manifests (`.claude-plugin/`, `.codex-plugin/`, `.agents/plugins/`) bumped to `1.1.0`
