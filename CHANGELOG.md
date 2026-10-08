# Changelog

All notable changes to Battle Builder are documented here.

---

## [2.4.1]

### Fixed

- **Typed Hidden Power on GSC** — a build's "Hidden Power <Type>" was taught even when its DVs weren't set (Allow Hidden Power Changes off, or a DV change that isn't legal), leaving whatever type the Pokémon's DVs happened to give. It's now only taught when the DVs are set, as in FRLG; otherwise the slot keeps its previous move, or another move the Pokémon knew, with a warning naming the type.

### Changed

- **Internal refactor (no behavior change)** — Gen 3 code moved into its own files (`FallbackMoveBuilder.Gen3.cs`, `BattleApplier.Gen3.cs`), with all Gen 3 per-species data in `Gen3Species.cs` and shared helpers replacing repeated logic. Auto Build output and apply results are identical for every game (both regression harnesses).
- Added an apply regression harness (`tools/apply-regress.ps1`): applies build data to local save fixtures with several option profiles and compares the resulting Pokémon and apply log to a baseline.

## [2.4.0]

### Added

- **Gen 3 (FireRed/LeafGreen) support** — Battle Builder now detects FRLG saves, with a full Auto Build for all 386 species. Sources: **Pikalytics**, **Smogon**, and Auto Build.
  - **Learnset discovery** — FRLG level-up, TM/HM, egg moves, and every FRLG and Emerald move tutor (Emerald tutor moves are legal on FRLG Pokémon by trade).
  - **Gen 3 physical/special split by type** — Normal, Fighting, Flying, Poison, Ground, Rock, Bug, Ghost and Steel are physical; Fire, Water, Grass, Electric, Psychic, Ice, Dragon and Dark are special. Roles, natures, EVs and move choices all follow it: sweepers take the category that hits harder, walls attack in their stronger category with a nature that lowers the stat they don't use, and Mixed Attackers split EVs by their moves and never lower an attacking stat or Speed.
  - **Gen 3 move ranking** — Gen 3 base powers, with drawbacks priced in (two-turn moves, recoil on walls, Overheat, delayed attacks, low accuracy); weak STAB (Fury Cutter, Twister) and weak filler aren't picked.
  - **Set quality** — Rest is always covered (Sleep Talk, a sleep move or Substitute) and Rest + Sleep Talk sets attack with their other slots; at most two HP-recovery sources; no boost for a stat the set doesn't attack with; walls carry Rapid Spin, take their second STAB or coverage over Roar, and skip Water Spout/Eruption; egg-move fallbacks never duplicate an attack type.
  - **Typed Hidden Power** — chosen to hit what walls the set's STAB (Zapdos: Ice, Charizard: Grass), and only taught when its IVs can be set; the IVs are set for the type at 70 power.
  - **Species sets** — the defining Gen 3 sets where the general rules can't assemble them: Kyogre Water Spout/Thunder, Groudon Eruption, Cloyster/Forretress/Skarmory Spikes, Snorlax Curse/Rest/Sleep Talk, Breloom Spore/Swords Dance, Celebi Calm Mind/Recover, Wobbuffet Counter/Mirror Coat, Chansey/Blissey Seismic Toss, a sketched Smeargle set, and more.
- **Nature and ability on FRLG** — Gen 3 stores both in the PID, so changing either finds a new PID that keeps gender, shininess and form. Bred Pokémon change directly; wild and static Pokémon are re-rolled at their original encounter for the best IVs it allows. Wild-caught and static shinies, in-game trades and event Pokémon are left as-is to preserve legality.
- **Egg Conversion (FRLG)** — optionally rebuilds a wild-caught Pokémon as egg-hatched, allowing egg moves and full 31 IVs. Off by default, behind a confirmation.
- **Allow Illegal on FRLG** — also changes nature and ability on Pokémon blocked for legality (trades, events, shinies).
- **Allow Hidden Power Changes** extends to FRLG, and **Check All Competitive** now includes it (FRLG shows one combined confirmation for Nature, Ability and Egg Conversion).
- **Max friendship for Return** (and none for Frustration), Gen 2 and up.

### Fixed

- Per-game options weren't restored when the plugin reopened (they were read before the window was shown).
- The FRLG options row pushed the Apply button out of the scroll area.
- Sketch no longer gets PP Ups.
- A typed Hidden Power that can't be taught (in-game trade IVs are fixed) falls back to another of the Pokémon's original moves instead of leaving the slot empty.

### Changed

- FRLG joins the Auto Build regression harness (`tools/regress.ps1`).
- Confirmation text notes that nothing is permanent until the save is exported.

## [2.3.0]

### Added

- **Gen 2 (Gold/Silver/Crystal) support** — Battle Builder now detects GSC saves, with a full Auto Build for all 251 species. Sources: **Pikalytics**, **Smogon**, and Auto Build.
  - **Learnset discovery** — union of Gold/Silver and Crystal level-up moves, TM/HM compatibility, Crystal's move tutor (Flamethrower/Thunderbolt/Ice Beam), and egg moves. A pre-evolution's level-up moves normally carry forward on evolution, except for Gen 2's baby Pokémon (Pichu, Cleffa, Igglybuff, Tyrogue, Smoochum, Elekid, Magby) — most specimens of their evolved forms were caught directly rather than bred through the baby stage, so those moves are treated as egg moves instead of assumed freely learnable. Hyper Beam (always recharges in Gen 2) and Hidden Power (type depends on DVs) are excluded from the learnable pool entirely.
  - **Move-selection engine** shares the same offense/defense/last-resort pass structure introduced for RBY, with Gen 2-specific staple pools: Rest + Sleep Talk, Curse, Spikes, and Roar for defensive roles; Return, Earthquake, and the elemental beams for attackers.
  - **Fixed movesets** for Magikarp (Flail/Tackle/Splash), Ditto (Transform), and Unown (Hidden Power only, excluded from every other species' pool). Smeargle gets a dedicated Spikes/Substitute/Baton Pass/Thunder Wave support set — Sketch technically opens its whole movepool, but its stats (Atk 20, SpA 20) rule out being an actual attacker.
  - **Purpose-matched egg-move substitutes** — an attacking egg move substitutes for the best learnable attack of the same category in a type the build doesn't already cover, instead of the first STAB-pool entry (which often just duplicated a move already in the set). A status egg move (Swords Dance, Synthesis, Leech Seed, Spikes, ...) substitutes for the closest-purpose learnable move instead of an unrelated attack.
- **Allow Hidden Power Changes** — for a build that specifies a Hidden Power type (Smogon/Pikalytics commonly do), optionally sets the DVs needed to hit that exact type at maximum power (70), for the specific Pokémon Hidden Power is actually being set on. Overrides Max IVs' own DV choice for that Pokémon only. Off by default, behind a confirmation dialog: Gen 2 derives gender and shininess directly from DVs with no separate stored flag, so this can change either as a side effect.
- **Allow Egg Encounter Edit** — optionally clears Met Level/Location/Catch Data so an egg-only move can be legal on an individual whose actual encounter says it wasn't bred. GSC only, off by default, behind a confirmation dialog. Works for a move from any source (Auto Build, Smogon, Pikalytics) that turns out to need it, not just ones Auto Build's own generation pass already flagged as egg-only.

### Fixed

- **Max IVs on Gen 1/2** compared DVs against the modern 31 cap instead of their real 15 cap, and tried to write IV_HP directly even though it's derived from the other DVs in these games — it never recognized a DV-game Pokémon as already maxed, so reapplying kept reporting changes forever. Now caps at 15, skips the derived HP DV, and settles after one apply.
- **"All moves skipped (illegal)" warning** fired whenever nothing changed on a given pass, even when most slots already held the intended move from a previous run. Now only fires when none of the four slots hold an intended (primary or substitute) move.
- **Mapping List textbox** was invisible at every window size — a chain of GroupBox AutoSize collapse, a wrapping format-hint label bloating its toolbar row, and `Dock=Fill` being fundamentally incompatible with `AutoScroll`'s overflow detection. Resolved with fixed-pixel row heights inside a proper scroll host.
- **Per-species build source selection** was keyed by raw list position — a Refresh rebuilds each species' build list from whichever sources currently have data for it, so a saved index could end up silently pointing at a different source after the list's composition or order changed. Now keyed by the build's own source name (`BuildSourceBySpecies`), so a Refresh can't misattribute a saved pick; existing index-based selections reset once on upgrade.
- **A duplicate recovery move** (e.g. both Rest and Moonlight on the same build) could end up in an Auto Build set on any game, since the role pools deliberately list several recovery options together and nothing capped the pick at one. `TryAdd` and the final "any learnable move" fallback pass are both capped now.
- **Rest + Sleep Talk quality** — an ailment move (Sleep Powder, Toxic, Will-O-Wisp, ...) or zero real attacks alongside Rest + Sleep Talk defeated the pairing's whole point (Sleep Talk calls a random move from the set while asleep). Now capped to one recovery move plus at least one real attack, and the "pair Sleep Talk with Rest when learnable" rule — previously Gen 2-only — applies to every game.
- **Swords Dance/Amnesia's "does this species have real own-type damage to boost" check** lived only in `TryAdd`; the final fallback pass built its move list directly and could in principle bypass it. Closed for consistency, though no case of it actually happening was found in current data.
- **Max IVs on Gen 2 Unown** always maxed every DV to 15, which always produces the 'Z' letter form (Unown's form is computed live from its DVs, not stored separately). Now brute-forces the DV combination that maximizes DVs while keeping the individual's actual letter.
- **Reapplying moves to an already-correct-but-still-illegal Gen 2 Pokémon** (e.g. an egg move present but the encounter never got converted to bred) reported no diff, since the moves already matched. Rerunning with Allow Egg Encounter Edit now catches this case and fixes the encounter even when no move slot changes.
- **Allow Illegal let a naive move write "succeed" and be reported as fixed** even when the written moveset was still genuinely illegal, because the pre-existing baseline-tolerance check only compares generic error text and `AllowIllegal` short-circuited the legality check entirely. Both the primary and substitute move-write passes now additionally require every move slot to independently check out under a real (non-baseline, non-`AllowIllegal`) legality analysis before accepting a write; the final per-slot fallback is unchanged and remains the genuine last resort.
- **Legality checks used whichever save was last loaded, not the one actually being applied to** — Battle Builder never told PKHeX.Core which save's era (real Gen 1/2 cartridge vs. Virtual Console) was in play, so a check that depends on it (e.g. Stadium's move relearner, cartridge-only) could silently use the wrong answer. Now synced from the save being applied to on every Apply run.

### Changed

- GSC joins the automated Auto Build regression harness (`tools/regress.ps1`), alongside BDSP/LGPE/PLA/PLZA/RBY.
- Added `BuildsJson.RegenerateAutoBuildEntries` — refreshes only a game's Auto Build entries in place, leaving Pikalytics/Smogon/other sources' entries untouched. Used to roll an Auto Build fix into already-merged live data without a full network Refresh (which re-fetches and reshuffles every source, not just Auto Build).

## [2.2.0]

### Added

- **Persisted settings** - the Parsed Sources checklist (checked state and priority order), the apply-option checkboxes, the box range, each species' selected build (for species with more than one source), and each species' toggled form (e.g. a Mega Evolution) are now saved to `builds/settings.json` and restored automatically when the plugin reopens. Saved per game family (Red/Blue/Green/Yellow share one set of settings, Gold/Silver/Crystal share another) rather than per individual cartridge.

## [2.1.2]

### Added

- **Max PP** - applying Competitive Moves now also gives every move 3 PP Ups and refills PP to the maximum. Skipped in Legends: Arceus (no PP Ups); in other games the change is reverted with a warning if PKHeX rejects it.

## [2.1.1]

Maintenance release. 2.1.0 was never published, so 2.1.1 ships everything listed under 2.1.0 below.

### Changed

- **Internal refactor (no behavior change)** - Auto Build output is identical to 2.1.0 for every game (verified with a regression harness under `tools/`). The move-selection engine moved out of `BuildsJson` into `FallbackMoveBuilder` (split into offense/defense/last-resort/post-check passes), Gen 1 data and rules moved into `Gen1Rules`, and `BuildsJson` was split into partial files.

## [2.1.0]

### Added

- **Gen 1 (RBY) support** — Pokémon Red / Blue / Yellow saves are detected and supported, with a full Auto Build for all 151 species. Sources: **Pikalytics**, **Smogon** (gen1ou), and Auto Build.
  - **Stat Exp** replaces EVs (all six stats set to 65535, no total cap); the UI relabels the option "Competitive Stat Exp" and hides Nature, Stat Nature, and Hyper Train (none exist in Gen 1).
  - **Mixed Attacker** role — sweeper and bulky roles whose Attack and Special are within 10% (Charizard, Pikachu, Weepinbell…) get a blended physical/special build. Role labels now show in the moves row, e.g. `Moves (Special Wall)`, for every game.
  - **Gen 1 learnable pool** — union of Red/Blue and Yellow level-up + TM data, plus every pre-evolution's moves (Gen 1 evolution never removes moves), each species' starting moves from PKHeX's base-stat data (e.g. Tentacool's Acid), fixed movesets for Caterpie/Metapod/Weedle/Kakuna/Ditto, and Surf on Pikachu/Raichu.
  - **Gen 1 move-selection rules** — see the README's *Gen 1 (RBY) Auto Build* section: per-type physical/special split, real Gen 1 move types, charge-move and accuracy ranking adjustments, one ailment per build (sleep > paralysis > poison for walls), Dream Eater only with a sleep move, Rest skipped when Recover/Soft-Boiled/a drain move exists and only kept with a sleep or paralysis source, one drain move, one self-KO move (not on high-HP tanks), Body Slam > Double-Edge > Hyper Beam, Thunder Wave/Seismic Toss/Agility/Reflect support, and limited special coverage for Normal-only physical sweepers.

### Improved

- **Applying to already-illegal Pokémon** — a Pokémon that was illegal before Battle Builder touched it (e.g. a Mew with no matching encounter) no longer rejects every change. A change is now accepted as long as it introduces no *new* invalid checks beyond the Pokémon's baseline; a genuinely illegal move is still rejected. Fully legal Pokémon behave as before. Applies to all games.
- **Post-check for defensive roles** — the "guarantee at least one attack" fallback no longer mistakes status moves that appear in the type-coverage table (e.g. Rest) for attacks, fixing all-status walls in every game.
- **Moves label** — the Parsed Mappings grid now actually paints the `Moves (…)` label; a custom cell painter was previously overwriting it with plain "Moves".

### Documentation

- README updated for Gen 1, Stat Exp, the already-illegal apply behavior, and the PLZA merged datasource, which is fetched from a Gist on every Refresh rather than shipped as a local CSV.

---

## [2.0.0]

### Added

- **PLZA support** — full Pokémon Legends: Z-A support. All species and alternate forms covered via Game8, Deltias Gaming, YouTube, Reddit, and Auto Build.
  - Sources: **Game8**, **Deltias Gaming**, **YouTube**, **Reddit** (in configurable priority order)
  - **Merged datasource** — builds from all sources are consolidated into a single curated CSV, hosted externally as a Gist (`gist.github.com/TheDuke3x2/81fea803a427383d2a1fed27fb4e65d9`) and fetched fresh on every Refresh, just like any other scraped source. It acts as a first-class source just like any website or JSON feed, containing reviewed and curated builds from multiple origins, then merged with Auto Build fallbacks into `plza_builds.json`
  - **Plus Move flags** — moves mastered through level-up in PLZA (using the PA9 `SetMovePlusFlag` API) are automatically set when applying at level 100, using `PersonalInfo9ZA.PlusMoveIndexes` cross-referenced against the PLZA learnset

- **Multi-build system** — species can now have multiple builds per form from different sources. The Parsed Mappings grid cycles through all available builds with the **⟳ build cycle button** (`*` prefix on species names indicates multiple builds exist). Build index is tracked per-row and persists through form toggles.

- **Build source display** — the source DB is shown in parentheses next to each species name in the Parsed Mappings grid (e.g. `Garchomp (Deltias)`, `*Greninja (Game8)`), making it clear which source is active for each row.

- **Battle Form Toggle improvements** — the BF column toggle system was overhauled to support multiple independent toggle groups per species (e.g. Meowstic has separate M→Mega-M and F→Mega-F groups). The column header click expands/collapses all toggleable rows simultaneously.

- **Species search filter** — a real-time search box above the Parsed Mappings grid filters rows by species name. When Competitive Moves is on, the associated moves sub-row stays visible alongside its matched species row.

- **PLZA-specific move filtering** — invalid moves are filtered at both merged datasource load time and auto-build generation time:
  - `PlzaLearnsetExcludes` — moves not available in PLZA (Twineedle, Eerie Spell, Poison Tail)
  - `PlzaStarterUltimateMoves` — starter ultimate tutor moves restricted to final evolutions
  - `PlzaFormExclusions` — forms excluded from auto-build (Ash-Greninja, Zygarde mid-battle forms, Magearna cosmetic duplicates)

- **`FormNameOverrides`** — per-game canonical name overrides for forms whose PKHeX auto-name differs from the merged datasource (e.g. Eternal Flower Floette).

- **`NormalizeSpeciesName`** — battle-form names are normalized to `Species (Form)` convention on Refresh write (e.g. `Mega Greninja` → `Greninja (Mega)`), ensuring UI display names match JSON names consistently.

- **Regional dex ordering** — PLZA builds are written to JSON in regional dex order (Lumiose dex 1–232, then Hyperspace dex). The `national` field is embedded per-entry so the correct species key is always recovered on load regardless of JSON key encoding.

### Improved

- **Move pool completeness** — added missing STAB and coverage moves across all type pools: Sacred Fire, Heat Crash, Bitter Blade, Crabhammer, Origin Pulse, Volt Tackle, Supercell Slam, Petal Dance, Grass Knot, Low Kick, Dire Claw, Precipice Blades, Headlong Rush, Mud Bomb, Esper Wing, Psyshield Bash, Gigaton Hammer, and more. Fixed Wave Crash type classification (Normal → Water).

- **Window default size** — increased default window dimensions (1300×750) and minimum size (1000×600) for better usability on standard displays.

- **`MovePower` registry** — expanded with accurate base powers for all newly added moves.

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
