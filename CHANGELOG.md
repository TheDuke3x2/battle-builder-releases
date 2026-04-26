# Changelog

All notable changes to Battle Builder are documented here.

---

## [1.3.1]

### Fixed

- **Check All Competitive skipped Level 100** — Set Level 100 was not included in the Check All Competitive selection, so it had to be ticked manually every time.

### Improved

- **Check All Competitive** — the button now selects all competitive options (Stat Nature, EVs/AVs, GVs, Moves, Level 100, Hyper Train) without touching Nature, which is left to the user since it can affect legality more aggressively.

---

## [1.3.0]

### Added

- **PLA support** — full Pokémon Legends: Arceus support. All species covered via RankedBoost and Auto Build refresh. Grit Values (GVs) are applied per-stat using the IV-based formula (IV 31 → 7 GVs, IV 26–30 → 8, IV 20–25 → 9, IV 0–19 → 10).

### Fixed

- **Move application at level 100** — moves for level-100 Pokémon with encounter-restricted or partially-locked movesets no longer fail with "all moves skipped (illegal)". Root cause: intermediate per-slot legality checks during set assembly (with fewer than 4 moves set) caused PKHeX to report spurious "Empty Move" violations that cascaded to every remaining slot. The applier now assembles the full 4-slot set from the current moveset baseline before running a single legality check, falling back to per-slot validation only if the complete set is rejected.
- **Subsequent-run false "moves updated"** — Pokémon whose moves were already correct no longer report "moves updated" on every subsequent run. Root cause: `TryApplySlot`'s "already in place" early return was being counted as a write, triggering a false change record. The applier now compares slot state directly against the original to detect real writes.

### Improved

- **Move apply skip** — when the current moveset already matches the build's primary moves exactly, the entire move apply step is skipped with no legality checks performed.
- **Move substitution** — the substitute move set is only tried when it differs from the primary set, avoiding a redundant legality check on species with no egg-move substitutes defined.

---

## [1.2.0]

### Added

- **LGPE support** — full Let's Go Pikachu / Eevee support covering all 151 Kanto species + Meltan.
  - Awakening Values (AVs) replace EVs in the UI and JSON (0–200 per stat, no total cap)
  - Stat Nature checkbox hidden for LGPE (no stat nature in game)

### Fixed

- **Move slot swapping** — when a desired move already exists in a different slot, the applier now swaps slots instead of writing a duplicate, which PKHeX flags as illegal. Affected any Pokémon whose current moveset contained one of the target moves out of order (e.g. Will-O-Wisp in slot 2 when the build wants it in slot 1).
- **Check All** now correctly detects Max IVs as an active option when deciding whether to uncheck all.

### Improved

- **PkmHelper** — extracted LGPE-specific PKM preparation (CP sync, Stat_Level init, hyper train flags) into a shared helper class to keep the apply engine game-agnostic.

---

## [1.1.0]

### Added

- **Auto Build** — introduced as a fallback for Pokémon with no competitive data from Pikalytics or Smogon. Derives ability, nature, and EVs from base stats, and classifies each Pokémon into a role (Physical Sweeper, Special Wall, etc.) stored as `_buildRole` in the JSON.
  - Move generation uses a three-pass formula: role-specific pool → universal pool → raw learnset, guaranteeing 4 moves for any species with 4+ learnable moves. Egg moves are paired with a non-egg substitute using the `EggMove|*Substitute` format for games where egg moves require breeding; in BDSP and later games where egg moves are freely transferable, they are treated as normal learnable moves.

### Improved

- **Move application** — improved reliability across all build sources, correctly handling cases where intermediate move states caused valid sets to be rejected.

---

## [1.0.2]

### Fixed
- Stat cache is now recalculated and written back for all processed Pokémon, including
  those where no fields changed. Previously, Pokémon that were skipped or already
  up-to-date would not have their stat block updated, requiring a battle to refresh stats.

---

## [1.0.1]

### Fixed
- EV, IV, and nature changes now reflect immediately in-game without needing to enter a battle.
  PKHeX's `ResetPartyStats()` is called after each apply to recalculate the cached stat block.

---

## [1.0.0] — Initial release

Full BDSP support. All 493 species covered via Pikalytics and/or Smogon refresh.

### Added

**Core apply engine**
- Bulk application of ability, nature, stat nature, EVs, moves, level, max IVs, and hyper training across save file boxes and party
- All legality gated exclusively by PKHeX's `LegalityAnalysis` — no custom legality logic
- Per-field revert on failure: if a change would make the Pokémon illegal, it is reverted and a warning is logged
- Allow Illegal mode bypasses all PKHeX legality checks
- CSV mapping list with per-species overrides; CSV values take priority over dictionary

**UI**
- All checkboxes default off; Check All / Uncheck All toggle
- Mutual exclusion between Max IVs and Hyper Train
- Box range selector (From / To / All) with optional party inclusion
- Preview / Validate grid showing resolved builds before applying
- Single-line log output per Pokémon: green (all applied), gold (partial), red (none), blue (egg/no mapping)
- Warnings inline on the log line showing the PKHeX reason for each skipped field

**Build dictionary**
- Per-game JSON files loaded at runtime from `builds/{game}/` alongside the DLL
- Pikalytics refresh for BDSP — all 493 species, top 4 moves by usage
- Smogon refresh for BDSP — competitively relevant species, first listed set
- Merge Refresh combining both sources with configurable priority order
- Per-entry `_buildSource` field recording origin (`Pikalytics`, `Smogon`, `Pikalytics>Smogon`, `Auto Build`)
- Stat-derived base entries for species not covered by any source
- Multi-form support with per-form dictionary entries
- Auto-bootstrap: creates `builds/` folder structure and shell JSONs on first launch
- Shell state detection with gold warning prompting Refresh if no data is loaded
- `GameBuildsConfig` record — adding a new game requires only a new config instance
- Shell JSON for 14 future games: swsh, pla, hgss, oras, usum, dppt, xy, lgpe, sm, rse, frlg, gsc, rby
