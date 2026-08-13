# TEBBY.md - pickle_multiplayerjobs

## Overview

`pickle_multiplayerjobs` is a FiveM server resource described as "the best multiplayer jobs creator and system on FiveM" (per `README.md`). It is a Pickle Modifications product intended to let server administrators create and manage synchronised multiplayer jobs for players. As of the current commit the repository contains only a stub (README and `.gitattributes`) — no Lua source, manifest, or NUI assets have been committed yet.

## Code design

No source code is present in the repository at this time. Based on the product family context (sister resources `pickle_casinos` and `pickle_restaurants`) the expected architecture when source is added will be:

- **`fxmanifest.lua`** — FiveM resource manifest declaring shared/client/server scripts, NUI pages, and file references. This will be the primary entry point.
- **Client scripts** — handle world entities, interaction prompts (via target system: `ox_target`, `qb-target`, or `qtarget`), NUI messaging, and player state.
- **Server scripts** — authoritative game logic, database persistence (expected: `oxmysql` or `mysql-async`), economy transactions, and server callbacks.
- **Bridge layer (`bridge/`)** — framework abstraction for ESX, QBCore, and a custom stub, following the same pattern used in other Pickle resources.
- **Config (`config.lua` / `config_sv.lua`)** — shared/client settings and server-only settings (webhooks, society names).
- **Locales (`locales/translations/`)** — all player-facing strings; active locale selected by a `Config.Language` key.
- **NUI** — in-world or fullscreen UI built as a web app and communicated with via `SendNUIMessage` / `RegisterNUICallback`.

None of these files exist yet; the above is inferred from repository naming conventions and Pickle product patterns.

## Important files

| Path | Why it matters |
|------|----------------|
| `README.md` | Only extant documentation; states the project name and purpose. |
| `.gitattributes` | Enforces LF line endings on all text files — relevant for cross-platform contributors. |
| `fxmanifest.lua` *(not yet present)* | Will be the FiveM manifest — the authoritative list of every script and file the resource loads. |
| `config.lua` *(not yet present)* | Will hold all customer-editable shared/client settings (framework, target system, feature flags). |
| `config_sv.lua` *(not yet present)* | Will hold server-only settings such as Discord webhook URLs. |
| `bridge/` *(not yet present)* | Will contain framework and target-system adapters (ESX, QBCore, custom). |
| `locales/translations/en.lua` *(not yet present)* | Will hold all English player-facing strings loaded by the locale helper. |

## Intended behaviors

None found yet — no source code is present to inspect. When source is added, watch for:

- **Anti-tamper / escrow**: Pickle Modifications products are sold with Tebex escrow. Parts of the resource (core game-logic scripts) may be `.lua` files served as compiled bytecode that cannot be read or edited. Do not flag missing readable source as a bug.
- **Resource name sensitivity**: Locale loading and NUI registration in Pickle resources depend on the resource folder being named **exactly** `pickle_multiplayerjobs`. Renaming the folder (e.g. to `pickle_multiplayerjobs_v2`) breaks locale lookup and produces `ERR_TRANSLATE_<key>_404` errors. This is intentional design, not a bug.
- **`taxRecipients` must sum to 100**: If casino-style tax-recipient tables appear in config, percentages must total 100 — mismatches silently cause incorrect payouts, not a code defect.

## Common issues & fixes

Because no source is committed yet, the following are inferred from the Pickle Modifications product family:

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Locale strings show as `ERR_TRANSLATE_<key>_404` | Resource folder name doesn't match `pickle_multiplayerjobs` exactly, or `Config.Language` doesn't match the translation file name | Rename folder to `pickle_multiplayerjobs`; confirm `Config.Language = "en"` matches `locales/translations/en.lua` |
| Resource fails to start / manifest errors | `ox_lib` or database resource (`oxmysql` / `mysql-async`) not started before this resource | Reorder `server.cfg` so dependencies start first |
| Target interactions not appearing | `Config.UseTarget = true` but no supported target resource (`ox_target`, `qb-target`, `qtarget`) is running | Install a supported target resource or set `Config.UseTarget = false` |
| Framework bridge not detected | Server uses a renamed/forked framework | Implement `bridge/custom/client.lua` and `bridge/custom/server.lua` |
| Webhook requests not sent | `url` field left as placeholder `"DISCORD_WEBHOOK_URL"` | Replace placeholder with a real Discord webhook URL; leaving placeholder disables the webhook (by design) |
| Items not consuming correctly (ox_inventory) | Vessel items registered with `consume = 1` instead of `consume = 0` | Set `consume = 0` and point to the resource's `useItem` export |

## Conventions

Based on the Pickle Modifications product family, the following conventions are expected when source is added:

- **Lua style**: 4-space indentation; `Config.*` table for all settings; `ConfigSV.*` for server-only settings.
- **Naming**: `snake_case` for variables and functions; `PascalCase` for exported module tables.
- **Locale strings**: All player-facing text must go through the `_L(key, ...)` helper — never hardcode UI strings in logic files.
- **Bridge pattern**: All framework-specific calls (money, notifications, identifiers) must go through the `bridge/` layer, not called directly in game logic.
- **Server authority**: Economy mutations (add/remove money, chip balances) must be validated and executed server-side only; client scripts send requests and receive results.
- **Line endings**: LF only (enforced by `.gitattributes`).
- **No direct `print()` in production paths**: Debug output should be gated behind `Config.Debug == true`.
- **File organisation**: Client scripts in `client/`, server scripts in `server/`, shared scripts in `shared/` or `config.lua`; NUI source in `web/` or `nui/`.
