# S03: Documentation

**Goal:** `providers.md` ships with GSD and covers everything a user needs to configure any custom/local model provider from scratch.
**Demo:** A user who has never configured a custom provider can follow `docs/providers.md` to set up Ollama, LM Studio, vLLM, OpenRouter-as-proxy, ZAI-as-proxy, or any generic OpenAI-compatible endpoint — and the doc ships in the npm package.

## Must-Haves

- `docs/providers.md` exists with complete examples for all 6 provider types (Ollama, LM Studio, vLLM, OpenRouter proxy, ZAI proxy, generic OpenAI-compatible)
- Schema reference covering all `models.json` fields with types and defaults
- API key resolution section documenting the 3-tier chain (`!command` → env var → literal)
- `modelOverrides` section showing how to tweak built-in models
- Troubleshooting section with common errors and the startup warning format
- `package.json` `files` array includes `docs/providers.md` so it ships with npm
- README "Use Any Model" section links to `docs/providers.md`

## Verification

- `test -f docs/providers.md && echo "exists"` — file exists
- `grep -q 'docs/providers.md' package.json && echo "ships"` — included in npm package
- `grep -q 'providers.md' README.md && echo "linked"` — linked from README
- Content checks: `grep -c '```json' docs/providers.md` returns ≥6 (one per provider example + schema examples)
- Section checks: `grep -cE '^##' docs/providers.md` returns ≥7 (quick start, manual setup, examples, schema, auth, overrides, troubleshooting)

## Tasks

- [x] **T01: Write providers.md, wire into package.json and README** `est:45m`
  - Why: Single task covers the entire slice — creating the doc, ensuring it ships, and linking it from README. No runtime code or tests needed; splitting further would be artificial.
  - Files: `docs/providers.md`, `package.json`, `README.md`
  - Do: Create `docs/providers.md` with sections: Quick Start (wizard path), Manual Setup (create models.json, explain schema), Provider Examples (Ollama, LM Studio, vLLM, OpenRouter-as-proxy, ZAI-as-proxy, generic — each with complete models.json JSON block), Schema Reference (all fields from TypeBox schema with types/defaults/optionality), API Key Resolution (3-tier chain with examples), modelOverrides (tweak built-in models without redefining), Advanced Compatibility Flags (compat sub-fields), Troubleshooting (startup warning format, common errors, restart requirement). Use `CUSTOM_PROVIDER_PRESETS` values as source of truth for preset examples. Add `"docs/providers.md"` to package.json `files` array. Add a "Custom & Local Providers" subsection under README's "Use Any Model" with a brief summary and link to the full doc.
  - Verify: Run all 5 verification commands from the slice verification section above
  - Done when: All verification checks pass, doc has ≥6 JSON code blocks and ≥7 H2 sections, and content accurately reflects the codebase (preset values, schema fields, file paths)

## Files Likely Touched

- `docs/providers.md`
- `package.json`
- `README.md`
