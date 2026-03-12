---
estimated_steps: 5
estimated_files: 3
---

# T01: Write providers.md, wire into package.json and README

**Slice:** S03 — Documentation
**Milestone:** M003

## Description

Create the complete `docs/providers.md` documentation file covering custom/local model provider setup for GSD. This is the full slice in one task — create the doc with all required sections and examples, add it to `package.json` `files` so it ships with npm, and link it from the README's existing "Use Any Model" section.

## Steps

1. Create `docs/providers.md` with the following sections:
   - **Quick Start** — run `gsd config` → pick "Custom/Local provider" → pick preset → restart
   - **Manual Setup** — create `~/.gsd/agent/models.json`, explain the file structure
   - **Provider Examples** — complete `models.json` JSON blocks for each of 6 provider types:
     - Ollama (baseUrl `http://localhost:11434/v1`, apiKey `"ollama"`, no real key needed)
     - LM Studio (baseUrl `http://localhost:1234/v1`, apiKey `"lm-studio"`, no real key needed)
     - vLLM (baseUrl `http://localhost:8000/v1`, apiKey from env var)
     - OpenRouter as proxy (baseUrl `https://openrouter.ai/api/v1`, apiKey env var, explain why you'd use this vs built-in)
     - ZAI/custom cloud provider (baseUrl pointing to provider's endpoint, apiKey env var)
     - Generic OpenAI-compatible (template for any compatible endpoint)
   - **Schema Reference** — all fields from TypeBox schema: provider-level (`baseUrl`, `apiKey`, `api`, `headers`, `authHeader`, `models`, `modelOverrides`) and model-level (`id`, `name`, `api`, `baseUrl`, `reasoning`, `input`, `cost`, `contextWindow`, `maxTokens`, `headers`, `compat`), with types, defaults, and optionality
   - **API Key Resolution** — document `resolveConfigValue()` behavior: `!command` → shell exec (cached), otherwise env var check → literal fallback. Show examples of each.
   - **modelOverrides** — how to tweak built-in model settings (maxTokens, compat flags) without redefining the model
   - **Advanced: Compatibility Flags** — brief reference of the 11 `compat` sub-fields for non-standard endpoints
   - **Troubleshooting** — startup warning format (`[gsd] Warning: models.json error — <detail>`), common errors (invalid JSON, missing required fields, wrong API format), restart requirement
2. Use `CUSTOM_PROVIDER_PRESETS` from `src/onboarding.ts` as source of truth for preset baseUrl, api, apiKey, and exampleModel values
3. Add `"docs/providers.md"` to the `files` array in `package.json`
4. Add a "Custom & Local Providers" subsection under the "Use Any Model" section in `README.md` with a brief summary and a relative link to `docs/providers.md`
5. Run verification commands to confirm structure and wiring

## Must-Haves

- [ ] `docs/providers.md` exists with ≥7 H2 sections covering all required topics
- [ ] ≥6 complete JSON code blocks (one per provider example minimum)
- [ ] All preset values match `CUSTOM_PROVIDER_PRESETS` exactly (baseUrl, api, apiKey, exampleModel)
- [ ] File path documented as `~/.gsd/agent/models.json` (not `~/.gsd/` or `~/.pi/agent/`)
- [ ] `apiKey` field clearly explained as env var name / command / placeholder — NOT the literal secret
- [ ] Dual auth path documented (wizard via auth.json preferred, manual via models.json apiKey field)
- [ ] Restart requirement mentioned
- [ ] `package.json` `files` array includes `docs/providers.md`
- [ ] README links to `docs/providers.md`

## Verification

- `test -f docs/providers.md && echo "exists"` — file exists
- `grep -q 'docs/providers.md' package.json && echo "ships"` — in npm files
- `grep -q 'providers.md' README.md && echo "linked"` — linked from README
- `grep -c '```json' docs/providers.md` — returns ≥6
- `grep -cE '^##' docs/providers.md` — returns ≥7

## Inputs

- `src/onboarding.ts` — `CUSTOM_PROVIDER_PRESETS` constant (lines 121-162) for exact preset values
- `node_modules/@mariozechner/pi-coding-agent/dist/core/model-registry.js` — TypeBox schema (lines 42-89) for field definitions and defaults
- `node_modules/@mariozechner/pi-coding-agent/dist/core/resolve-config-value.js` — `resolveConfigValue()` for auth resolution docs
- `src/cli.ts` — error surfacing format (lines 107-112)
- S01/S02 summaries for wizard flow and validation behavior

## Expected Output

- `docs/providers.md` — complete documentation file (~400-600 lines)
- `package.json` — `files` array updated with `docs/providers.md`
- `README.md` — "Custom & Local Providers" subsection added under "Use Any Model"
