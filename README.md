# Battle Builder — PKHeX Plugin

A PKHeX plugin for bulk-applying competitive builds to Pokémon across save file boxes.
Applies abilities, natures, stat natures, EVs/AVs/Stat Exp, movesets, level, IVs, and hyper
training — individually or all at once — using a mapping list paired with a live competitive
build dictionary sourced from Game8, Deltias Gaming, YouTube, Reddit, Pikalytics, Smogon,
RankedBoost, and/or Auto Build.

Currently supports **RBY** (Pokémon Red / Blue / Yellow), **GSC** (Gold / Silver / Crystal),
**FRLG** (FireRed / LeafGreen), **BDSP** (Brilliant Diamond / Shining Pearl), **LGPE** (Let's Go
Pikachu / Eevee), **PLA** (Pokémon Legends: Arceus), and **PLZA** (Pokémon Legends: Z-A). The architecture is designed to support additional games
via per-game JSON build files.

> **Upgrading from 1.x:** Existing BDSP, LGPE, and PLA build JSONs load without changes.
> The new `national` field added for PLZA regional-dex keys is optional and ignored by
> earlier game configs.

---

## Requirements

- **PKHeX 26.03.06** or **26.05.05**
- **.NET 10 Windows Desktop Runtime** (included with PKHeX)
- **Windows**

---

## Installation

1. Locate your `PKHeX.exe` folder.
2. Create a `plugins` subfolder if it doesn't exist.
3. Copy `BattleBuilder.dll` into `plugins/`.
4. Launch PKHeX — the plugin appears under **Tools → Battle Builder**.

> **Note:** The `builds/` folder is created automatically on first launch. Open Battle
> Builder and click **🔄 Refresh** to populate the build dictionary before applying.

> **Note:** If PKHeX is in a cloud-synced folder (Dropbox, OneDrive), right-click the DLL
> → Properties → Unblock if the plugin doesn't appear.

---

## Usage

1. Open a save file in PKHeX.
2. Go to **Tools → Battle Builder**.
3. Click **🔄 Refresh** to populate the build dictionary.
4. **Paste** a mapping list into the text box, or click **Load from file…**.
5. Click **Preview / Validate ▶** to check for errors and preview resolved builds.
6. Set your **Box Range** and verify the detected game label is correct.
7. Tick the **Apply Options** checkboxes for the fields you want to change.
8. Click **Apply** and confirm. The log shows every change and any warnings.

**Quick tip:** For a full competitive build from the dictionary with no CSV overrides,
paste a species-only list and use **Check All Competitive**:

```
Garchomp
Togekiss
Lucario
Clefable
```

---

## Mapping List

Provide a CSV-style list mapping species to the values you want applied.

**Format:**
```
# Species[, Ability[, Nature[, Stat Nature]]]
Garchomp                         # species only — uses full dictionary build
Alakazam, Magic Guard            # override ability only
Gengar, Cursed Body, Timid       # override ability + nature
445,, Jolly, Jolly               # skip ability, set nature + stat nature
```

| Column | Notes |
|---|---|
| **Species** | English name or Dex number. Case-insensitive. |
| **Ability** | English ability name. Leave blank to fall back to dictionary. |
| **Nature** | Optional. English nature name. |
| **Stat Nature** | Optional. Sets mint/stat nature independently. Use `,,` to skip Nature. |

- Lines starting with `#` are comments.
- Blank lines are ignored.
- If a species appears more than once, the last entry wins.

### Priority — CSV vs Dictionary

CSV values always override the dictionary. Blank columns fall back to the dictionary
(if the relevant checkbox is ticked).

| CSV | Dictionary | Result |
|---|---|---|
| Specified | Anything | **CSV value applied** |
| Blank | Has entry | **Dictionary value used** |
| Blank | No entry | **No change** |

---

## Apply Options

All checkboxes default to **off**. Use **Check All Competitive** to enable all competitive
options at once (Stat Nature, EVs/AVs, GVs, Moves, Level 100, Hyper Train — Nature is
left to the user). Toggles to **Uncheck All Competitive** to clear.

| Option | What it does |
|---|---|
| **Ability** | Sets ability from CSV or dictionary. Reverted if PKHeX marks the result illegal. Hidden in LGPE and RBY (no ability mechanic). |
| **Nature** | Sets base Nature. Reverted if illegal — use Allow Illegal to force. Hidden in RBY (no natures in Gen 1). |
| **Stat Nature** | Sets Stat Nature (mint effect) independently of base Nature. Hidden in LGPE and RBY (not applicable). |
| **Competitive EVs / AVs / Stat Exp** | Applies EV spread (BDSP/PLA/PLZA), Awakening Values (LGPE), or Stat Exp (RBY, labelled "Competitive Stat Exp") from dictionary. |
| **Competitive GVs** | Applies Grit Values (PLA only). Max GVs per stat derived from IVs: IV 31 → 7, IV 26–30 → 8, IV 20–25 → 9, IV 0–19 → 10. Applied alongside EVs for Pokémon HOME compatibility. |
| **Competitive Moves** | Applies moveset from dictionary. Each move gets 3 PP Ups (max PP) with PP fully refilled (skipped in Legends: Arceus, which has no PP Ups; reverted with a warning if PKHeX rejects it). Tries the full 4-slot set atomically first; falls back to per-slot with warnings if the full set is rejected. |
| **Set Level 100** | Sets the Pokémon to level 100. Enable before Hyper Train. |
| **Max IVs** | Sets all IVs to 31, skipping stats already covered by Hyper Training. |
| **Hyper Train** | Applies Hyper Training flags directly. Requires level 100. Hidden in RBY (no Hyper Training in Gen 1). |
| **Allow Illegal** | Bypasses all PKHeX legality checks for every field. |

**Mutual exclusions:** Enabling Hyper Train disables Max IVs (and vice versa), since
both aim to maximise stats but via different mechanics.

---

## Legality

Every change is gated by **PKHeX's own `LegalityAnalysis`** — no custom legality logic
is added. If a change would make the Pokémon illegal, it is reverted and a warning is
logged. **Allow Illegal** bypasses all checks.

**Already-illegal Pokémon:** a Pokémon that was illegal *before* Battle Builder touched it
(for example a Mew whose origin matches no legal encounter) would otherwise reject every
change. The applier records the Pokémon's invalid checks up front and accepts a change as
long as it introduces **no new** invalid checks beyond that baseline — so a genuinely
illegal move (which adds its own "Invalid Move" line) is still rejected, but pre-existing
problems no longer block unrelated changes. Fully legal Pokémon behave exactly as before.

### Log output

Each Pokémon produces a single log line after apply:

| Colour | Meaning |
|---|---|
| 🟢 Green `✔` | All requested fields applied successfully |
| 🟡 Gold `✔ … ⚠ skipped: …` | Some fields applied, others skipped with reason |
| 🔴 Red `✘ … all skipped` | Nothing applied |
| 🔵 Blue `—` | Egg — skipped entirely |

Species not in the mapping are silently counted in the `No mapping: N` summary line
and do not produce individual log entries.

---

## Competitive Build Dictionary

The plugin maintains a per-game JSON file (e.g. `builds/plza/plza_builds.json`) loaded
at runtime. Populate it via the **🔄 Refresh** button.

### PLZA Merged Datasource

For PLZA, builds from Game8, Deltias Gaming, YouTube, and Reddit are consolidated into a
single curated CSV hosted as a GitHub Gist and fetched fresh on every Refresh (there is no
local CSV). It acts as a first-class source, containing reviewed and curated builds from
multiple origins. On Refresh, it is fetched and merged with Auto Build fallbacks into
`plza_builds.json`.

### Source Priority List

The source panel shows available sources in priority order (top = highest). Sources
vary by game:

| Source | Games | Coverage | Moves |
|---|---|---|---|
| **Game8** | PLZA | Competitive species | Top build per article |
| **Deltias** | PLZA | Competitive species | Ranked PVP build |
| **YouTube** | PLZA | Selected species | Manually curated builds |
| **Reddit** | PLZA | Selected species | Community builds |
| **Pikalytics** | BDSP, LGPE, RBY | All species (usage-based) | Top 4 by usage % |
| **Smogon** | BDSP, LGPE, RBY | Competitive species | First listed set |
| **RankedBoost** | PLA | All species | Top moves by ranking |
| **Auto Build** | All | All species and all obtainable alternate forms | Stat-derived (see below) |

Check or uncheck the available sources for the loaded game. **Auto Build is always active**
and always last — it fills any species or form not covered by the checked sources.

Use **▲ / ▼** to reorder sources. Click **🔄 Refresh** to fetch and merge all checked
sources in the displayed priority order.

### Multi-Build Species

Species with builds from multiple sources show a **`*` prefix** in the Parsed Mappings
grid (e.g. `*Greninja (Game8)`). Click the **⟳** button on any `*`-prefixed row to cycle
through all available builds. The source shown in parentheses reflects the currently
active build.

---

## Parsed Mappings Grid

The Parsed Mappings grid shows one row per species (or per form for species with
independent form builds). Additional columns (Nature, EVs, Moves) are shown when their
corresponding checkboxes are enabled.

### Species Search

Type in the **search box** above the grid to filter rows by species name in real time.
When **Competitive Moves** is on, the moves sub-row follows its parent species row in
the filter results.

### Battle Form Toggle (BF column)

The **BF** column is a **Battle Form Toggle** — click it on a toggleable row to cycle
between a Pokémon's base form and its battle form (Mega Evolution, Primal Reversion,
etc.). The column header click expands or collapses all toggleable rows simultaneously.

- Species with a toggleable form show **⟳** in the BF cell.
- When toggled to the battle form, the applied build uses the Mega/Primal build from
  the dictionary. The Pokémon in the box always stores the base form; the Mega build
  is applied as the selected moveset/EV build regardless of the stored form.

---

## Auto Build

Auto Build generates a complete entry for every species (and alternate form) not covered
by any checked source. All fields are derived from base stats:

- **Ability** — Hidden Ability if available, otherwise Ability 1. Empty string for LGPE (no ability mechanic).
- **Nature / EVs** — classified by stat profile into one of seven roles:

| Role | Nature | EV Spread (HP/Atk/Def/SpA/SpD/Spe) |
|---|---|---|
| Physical Sweeper | Jolly / Adamant | 6/252/0/0/0/252 |
| Special Sweeper  | Timid / Modest  | 6/0/0/252/0/252 |
| Physical Wall    | Impish          | 252/0/252/0/6/0 |
| Special Wall     | Calm            | 252/0/6/0/252/0 |
| Mixed Wall       | Impish          | 252/0/128/0/128/0 |
| Bulky Attacker   | Adamant         | 252/252/0/0/6/0 |
| Bulky Special    | Modest          | 252/0/0/252/6/0 |

For **LGPE**, EVs are replaced by **Awakening Values (AVs)**. All stat roles use a flat
200/200/200/200/200/200 spread (max AVs, no total cap). Stat Nature is not written.

- **Moves** — filled using a three-pass formula: role-specific pool → universal pool →
  raw learnset. Guarantees 4 moves for any species with 4+ learnable moves (level-up and
  TM only; egg moves are freely included in BDSP and Gen 9+ where they do not require
  breeding).
- **`_buildRole`** — the classified role is recorded in the JSON for reference.

For games where egg moves require breeding, any egg move that has no level-up/TM
equivalent is written in `"EggMove|*Substitute"` format — the egg move is tried first
at apply time and the substitute is used if the Pokémon's origin does not permit it.

### Gen 1 (RBY) Auto Build

Gen 1 plays very differently, so RBY has its own move-selection logic on top of the shared
role classification. Everything below applies only to RBY.

**Stats and fields.** There are no natures, abilities, or Hyper Training in Gen 1. Every
build sets all six **Stat Exp** values to 65535 (Gen 1 has no total cap), and the UI hides
the Nature, Stat Nature, and Hyper Train options. DVs are handled by the existing IV
options. Roles are classified from base stats as usual, then adjusted (below).

**Physical/special is per-type, not per-move.** Fire, Water, Grass, Electric, Psychic, Ice,
and Dragon are special; every other type is physical. PKHeX reports Gen 1 types in the
ROM's raw byte encoding, so they are normalized before use. Move types come from PKHeX's
Gen 1 data (e.g. Bite and Gust are Normal in Gen 1), and same-type duplicate checks use
those real types.

**Mixed Attacker.** A sweeper or bulky-attacker role whose Attack and Special are within
10% of each other (Charizard, Charmander, Pikachu, Weepinbell…) becomes **Mixed Attacker**.

**Learnable pool.** The union of Red/Blue *and* Yellow level-up and TM/HM data, plus:

- everything learnable by **every pre-evolution** (Gen 1 evolution never removes moves —
  e.g. Victreebel keeps Weepinbell's Acid);
- each species' **starting moves** from PKHeX's base-stat data, which the level-up tables
  omit (e.g. Tentacool/Tentacruel's Acid);
- fixed movesets for Caterpie, Metapod, Weedle, Kakuna, and Ditto;
- **Surf on Pikachu/Raichu**, which PKHeX's legality check accepts even though the TM
  compatibility table does not list it.

**Move order** (offensive roles): STAB (scanned from the real learnset) → sleep move +
Dream Eater → Explosion → Leech Seed/Toxic → Recover/Soft-Boiled → Counter → staple →
Coverage → attack fill → support (Thunder Wave, Agility, Seismic Toss, Reflect) →
universal pool. Walls take a sleep or paralysis move first, then one STAB attack matching
their category, then role pools. Bulky roles take Thunder Wave (and Seismic Toss for very
low-Attack species like Chansey) right after STAB.

**Ranking adjustments** (Gen 1 base power is a poor proxy in a few cases):

- Two-turn charge moves (Solar Beam, Skull Bash, Razor Wind, Sky Attack) are last resort.
- Thunder and Hydro Pump take a small penalty for accuracy; Body Slam ranks above
  Double-Edge (100 BP in Gen 1), which ranks above Hyper Beam.
- Hyper Beam is allowed for attackers (it only recharges if the target survives) but never
  for walls. Fixed-damage Dragon Rage and Sonic Boom are never chosen.
- Type dedup: at most one damaging move per real type, except Normal (Double-Edge beside
  Body Slam is allowed).
- A physical sweeper whose only physical damage is Normal-type (Tauros, Dragonite,
  Gyarados) may take up to two special coverage moves, preferring Ice, Electric, then Water.

**Status and setup rules.**

- One ailment per build (only one status can be active). Walls prefer sleep → paralysis →
  poison; attackers that learn a sleep move take one, and **Dream Eater** is only added
  alongside a sleep move.
- **Rest** is skipped when Recover, Soft-Boiled, or a drain move is learnable (no berries
  to wake early in Gen 1), is never given to pure sweepers, and is only added when the
  build already has a sleep or paralysis source (a sleep move, Thunder Wave, Stun Spore,
  Glare, or Body Slam/Lick/Thunderbolt/Thunder) so the two turns of downtime are affordable.
- At most one drain move per build (Mega Drain over Leech Life).
- **Swords Dance / Amnesia** require real own-type damage in the category they boost;
  Amnesia leads the special-sweeper staple pool.
- One self-KO move at most (Explosion, else Self-Destruct), never alongside Rest/Recover,
  and not on high-HP tanks (HP ≥ 100, e.g. Snorlax, Chansey).

### PLZA Auto Build

For PLZA, Auto Build additionally:

- Filters moves through `PlzaLearnsetExcludes` (moves removed in Gen 9) and
  `PlzaSpeciesMoveExcludes` (moves invalid for specific species in PLZA).
- Respects `PlzaFormExclusions` to skip forms not present in the game (e.g. Ash-Greninja,
  Zygarde mid-battle forms).
- Sets **Plus Move flags** (`PA9.SetMovePlusFlag`) for moves mastered through level-up
  when applied at level 100, using `PersonalInfo9ZA.PlusMoveIndexes`.

---

## Multi-Form Pokémon

The applier matches form automatically. Resolution order:

1. Exact species + form match in the mapping.
2. Pokémon is base form (0): use any mapping entry for the species.
3. Non-base form with no dedicated build in the game dict: fall back to form-0 mapping.
4. **PLZA only** — non-base form with a dedicated build but no mapping entry (e.g. base form of a toggled Mega): use the next higher-numbered form in the mapping.

Auto Build generates entries for all **obtainable** alternate forms independently, each
with their own stat-derived ability, nature, EVs/AVs, and moves.

---

## Building

1. Install the [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10).
2. Navigate to the project folder in a terminal.
3. Run:
   ```
   dotnet build -c Release
   ```
4. Output: `bin/Release/BattleBuilder.dll`

`PKHeX.Core` is pulled automatically from NuGet — no manual setup needed.

**Mac/Linux deploy script** — `deploy.sh` builds the project and copies the DLL (plus the
legality reference docs) to all configured PKHeX plugin directories automatically.

---

## File Layout

```
BattleBuilder/
├── BattleBuilder.csproj
├── BattleBuilderPlugin.cs      ← IPlugin entry point, menu registration
├── BattleBuilderForm.cs        ← WinForms UI
├── BattleApplier.cs            ← applies mappings; all legality via PKHeX
├── PkmHelper.cs                ← game-specific PKM preparation (CP sync, HT flags, etc.)
├── BuildsJson.cs               ← JSON engine + GameBuildsConfig + per-game configs
├── MappingParser.cs            ← parses CSV into AbilityMapping objects
├── CompetitiveBuilds.cs        ← PokemonBuild record, EVSpread, role pools, registry
├── GenDetector.cs              ← detects save generation and game
├── deploy.sh                   ← Mac build + multi-target deploy script
├── README.md
├── CHANGELOG.md
└── builds/                     ← per-game data (runtime, not compiled)
    ├── bdsp/
    │   └── bdsp_builds.json    ← populate via 🔄 Refresh
    ├── lgpe/
    │   └── lgpe_builds.json    ← populate via 🔄 Refresh
    ├── pla/
    │   └── pla_builds.json     ← populate via 🔄 Refresh
    ├── plza/
    │   └── plza_builds.json    ← populate via 🔄 Refresh
    ├── rby/
    │   └── rby_builds.json     ← populate via 🔄 Refresh
    ├── gsc/
    │   └── gsc_builds.json     ← populate via 🔄 Refresh
    ├── frlg/
    │   └── frlg_builds.json    ← populate via 🔄 Refresh
    ├── swsh/  ← shell
    ├── hgss/  ← shell
    ├── oras/  ← shell
    ├── usum/  ← shell
    ├── dppt/  ← shell
    ├── xy/    ← shell
    ├── sm/    ← shell
    └── rse/   ← shell
```

> The `builds/` folder is the plugin's runtime data directory, stored alongside the
> DLL in your PKHeX `plugins/` folder. It holds one JSON file per game containing the
> competitive build dictionary — abilities, natures, EV spreads, and movesets — loaded
> at startup and populated via Refresh.

---

## JSON Format Reference

**BDSP** — single-form species (Pikalytics source):
```json
"445": {
  "name": "Garchomp",
  "_buildSource": "Pikalytics",
  "ability": "Rough Skin",
  "nature": "Jolly",
  "statNature": "Jolly",
  "evs": "0/252/0/0/4/252",
  "moves": ["Earthquake", "Outrage", "Swords Dance", "Stealth Rock"]
}
```

**BDSP** — Auto Build entry with role and egg move delimiter:
```json
"27": {
  "name": "Sandshrew",
  "_buildSource": "Auto Build",
  "_buildRole": "Physical Wall",
  "ability": "Sand Rush",
  "nature": "Impish",
  "statNature": "Impish",
  "evs": "252/0/252/0/6/0",
  "moves": ["Earthquake", "Knock Off|*Rock Slide", "Swords Dance", "Facade"]
}
```

**LGPE** — `avs` replaces `evs`, `statNature` is omitted, `ability` is always `""`:
```json
"59": {
  "name": "Arcanine",
  "_buildSource": "Pikalytics",
  "ability": "",
  "nature": "Jolly",
  "avs": "200/200/200/200/200/200",
  "moves": ["Will-O-Wisp", "Crunch", "Flare Blitz", "Teleport"]
}
```

**PLZA** — multi-build array with regional dex key and embedded national dex:
```json
"39": [
  { "form": 0, "national": 670, "name": "Floette",                      "_buildSource": "Auto Build", ... },
  { "form": 5, "national": 670, "name": "Eternal Flower Floette",        "_buildSource": "Game8", ... },
  { "form": 6, "national": 670, "name": "Eternal Flower Floette (Mega)", "_buildSource": "Auto Build", ... }
]
```

Multi-form species (all games):
```json
"487": [
  { "form": 0, "name": "Giratina",          "_buildSource": "Auto Build", ... },
  { "form": 1, "name": "Giratina (Origin)", "_buildSource": "Auto Build", ... }
]
```
