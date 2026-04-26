# Battle Builder — PKHeX Plugin

A PKHeX plugin for bulk-applying competitive builds to Pokémon across save file boxes.
Applies abilities, natures, stat natures, EVs/AVs, movesets, level, IVs, and hyper training —
individually or all at once — using a mapping list paired with a live competitive build
dictionary sourced from Pikalytics, Smogon, RankedBoost, and/or Auto Build.

Currently supports **BDSP** (Brilliant Diamond / Shining Pearl), **LGPE** (Let's Go
Pikachu / Eevee), and **PLA** (Pokémon Legends: Arceus). The architecture is designed
to support additional games via per-game JSON build files.

---

## Requirements

- **PKHeX 26.03.06**
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
| **Ability** | Sets ability from CSV or dictionary. Reverted if PKHeX marks the result illegal. Hidden in LGPE (no ability mechanic). |
| **Nature** | Sets base Nature. Reverted if illegal — use Allow Illegal to force. |
| **Stat Nature** | Sets Stat Nature (mint effect) independently of base Nature. Hidden in LGPE (not applicable). |
| **Competitive EVs / AVs** | Applies EV spread (BDSP/PLA) or Awakening Values (LGPE) from dictionary. |
| **Competitive GVs** | Applies Grit Values (PLA only). Max GVs per stat derived from IVs: IV 31 → 7, IV 26–30 → 8, IV 20–25 → 9, IV 0–19 → 10. Applied alongside EVs for Pokémon HOME compatibility. |
| **Competitive Moves** | Applies moveset from dictionary. PP set automatically. Tries the full 4-slot set atomically first; falls back to per-slot with warnings if the full set is rejected. |
| **Set Level 100** | Sets the Pokémon to level 100. Enable before Hyper Train. |
| **Max IVs** | Sets all IVs to 31, skipping stats already covered by Hyper Training. |
| **Hyper Train** | Applies Hyper Training flags directly. Requires level 100. |
| **Allow Illegal** | Bypasses all PKHeX legality checks for every field. |

**Mutual exclusions:** Enabling Hyper Train disables Max IVs (and vice versa), since
both aim to maximise stats but via different mechanics.

---

## Legality

Every change is gated by **PKHeX's own `LegalityAnalysis`** — no custom legality logic
is added. If a change would make the Pokémon illegal, it is reverted and a warning is
logged. **Allow Illegal** bypasses all checks.

### Log output

Each Pokémon produces a single log line after apply:

| Colour | Meaning |
|---|---|
| 🟢 Green `✔` | All requested fields applied successfully |
| 🟡 Gold `✔ … ⚠ skipped: …` | Some fields applied, others skipped with reason |
| 🔴 Red `✘ … all skipped` | Nothing applied |
| 🔵 Blue `—` | Egg or no mapping — skipped entirely |

---

## Competitive Build Dictionary

The plugin maintains a per-game JSON file (e.g. `builds/bdsp/bdsp_builds.json`) loaded
at runtime. Populate it via the **🔄 Refresh** button.

### Source Priority List

The source panel shows three sources in priority order (top = highest):

Available sources depend on the game:

| Source | Games | Coverage | Moves |
|---|---|---|---|
| **Pikalytics** | BDSP, LGPE | All species (usage-based) | Top 4 by usage % |
| **Smogon** | BDSP, LGPE | Competitively relevant species only | First listed set |
| **RankedBoost** | PLA | All species | Top moves by ranking |
| **Auto Build** | All | All species and all obtainable alternate forms | Stat-derived (see below) |

Check or uncheck the available sources for the loaded game. **Auto Build is always active**
and always last — it fills any species or form not covered by the checked sources.

Use **▲ / ▼** to reorder sources. Click **🔄 Refresh** to fetch and merge all checked
sources in the displayed priority order.

**Merge logic per species:**
1. Highest-priority checked source with moves → use it
2. Only one checked source has moves → use it directly
3. Both checked sources have moves → highest priority wins
4. No checked source covers the species → Auto Build

Each JSON entry records its origin in `_buildSource`:
`"Pikalytics"`, `"Smogon"`, `"Pikalytics>Smogon"`, `"RankedBoost"`, or `"Auto Build"`.

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

---

## Multi-Form Pokémon

The applier matches form automatically and falls back to form 0 if no form-specific
entry exists. Multi-form species use a JSON array with a `"form"` field per element.
Auto Build generates entries for all **obtainable** alternate forms independently, each
with their own stat-derived ability, nature, EVs/AVs, and moves.

The **BF** column in the preview grid is a **Battle Form Toggle** — click it to cycle
through a species' available forms before applying. Hover the column header for a
tooltip description.

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

---

## File Layout

```
BattleBuilder/
├── BattleBuilder.csproj
├── BattleBuilderPlugin.cs      ← IPlugin entry point, menu registration
├── BattleBuilderForm.cs        ← WinForms UI
├── BattleApplier.cs            ← applies mappings; all legality via PKHeX
├── PkmHelper.cs                ← game-specific PKM preparation (CP sync, HT flags, etc.)
├── BuildsJson.cs               ← JSON engine + GameBuildsConfig + BDSPBuildsJson + LGPEBuildsJson
├── MappingParser.cs            ← parses CSV into AbilityMapping objects
├── CompetitiveBuilds.cs        ← PokemonBuild record, EVSpread, role pools, registry
├── GenDetector.cs              ← detects save generation and game
├── README.md
├── CHANGELOG.md
└── builds/                     ← per-game data (runtime, not compiled)
    ├── bdsp/
    │   └── bdsp_builds.json    ← populate via 🔄 Refresh
    ├── lgpe/
    │   └── lgpe_builds.json    ← populate via 🔄 Refresh
    ├── pla/
    │   └── pla_builds.json     ← populate via 🔄 Refresh
    ├── swsh/  ← shell
    ├── hgss/  ← shell
    ├── oras/  ← shell
    ├── usum/  ← shell
    ├── dppt/  ← shell
    ├── xy/    ← shell
    ├── sm/    ← shell
    ├── rse/   ← shell
    ├── frlg/  ← shell
    ├── gsc/   ← shell
    └── rby/   ← shell
```

> The `builds/` folder is the plugin's runtime data directory, stored alongside the
> DLL in your PKHeX `plugins/` folder. It holds one JSON file per game containing the
> competitive build dictionary — abilities, natures, EV spreads, and movesets — loaded
> at startup and populated via Refresh. The folder structure and empty shell JSON files
> for all supported games are created automatically on first launch.

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

Multi-form species:
```json
"487": [
  { "form": 0, "name": "Giratina",          "_buildSource": "Auto Build", ... },
  { "form": 1, "name": "Giratina (Origin)", "_buildSource": "Auto Build", ... }
]
```
