# Battle Builder — PKHeX Plugin

A PKHeX plugin for bulk-applying competitive builds to Pokémon across save file boxes.
Applies abilities, natures, stat natures, EVs, movesets, level, IVs, and hyper training —
individually or all at once — using a mapping list paired with a live competitive build
dictionary sourced from Pikalytics and/or Smogon.

Currently supports **BDSP** (Brilliant Diamond / Shining Pearl). The architecture is
designed to support additional games via per-game JSON build files.

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
> Builder and click **Refresh Builds** or **Merge Refresh** to populate the build
> dictionary before applying.

> **Note:** If PKHeX is in a cloud-synced folder (Dropbox, OneDrive), right-click the DLL
> → Properties → Unblock if the plugin doesn't appear.

---

## Usage

1. Open a save file in PKHeX.
2. Go to **Tools → Battle Builder**.
3. Click **🔄 Refresh Builds** (or **🔀 Merge Refresh**) to populate the build dictionary.
4. **Paste** a mapping list into the text box, or click **Load from file…**.
5. Click **Preview / Validate ▶** to check for errors and preview resolved builds.
6. Set your **Box Range** and verify the detected game label is correct.
7. Tick the **Apply Options** checkboxes for the fields you want to change.
8. Click **Apply** and confirm. The log shows every change and any warnings.

**Quick tip:** For a full competitive build from the dictionary with no CSV overrides,
paste a species-only list and use **Check All**:

```
Garchomp
Togekiss
Lucario
Garchomp
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
445,, Jolly,, Jolly              # skip ability, set nature + stat nature
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

All checkboxes default to **off**. Use **Check All** to enable all competitive options
at once, **Uncheck All** to clear.

| Option | What it does |
|---|---|
| **Ability** | Sets ability from CSV or dictionary. Reverted if PKHeX marks the result illegal. |
| **Nature** | Sets base Nature. Reverted if illegal — use Allow Illegal to force. |
| **Stat Nature** | Sets Stat Nature (mint effect) independently of base Nature. |
| **Competitive EVs** | Applies EV spread from dictionary, or a stat-profile fallback if no entry exists. |
| **Competitive Moves** | Applies moveset from dictionary. PP set automatically. |
| **Set Level 100** | Sets the Pokémon to level 100. Enable before Hyper Train. |
| **Max IVs** | Sets all IVs to 31, skipping stats already covered by Hyper Training. |
| **Hyper Train** | Applies Hyper Training via PKHeX's suggested data. Requires level 100. |
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
at runtime. Populate it via the refresh controls in the toolbar.

### Refresh Builds

Select a source and click **🔄 Refresh Builds**:

| Source | Coverage | Moves |
|---|---|---|
| **Pikalytics** | All 493 BDSP species (usage-based) | Top 4 by usage % |
| **Smogon** | Competitively relevant species only | First listed set |

Species not covered by the selected source receive a stat-derived base entry
(best ability, sensible nature, physical/special EV split — no moves).

### Merge Refresh

Click **🔀 Merge Refresh** to fetch both sources and combine them. Priority order is
shown in the toolbar — click **⇄** to swap.

**Merge logic per species:**
1. Highest-priority source with moves → use it
2. Only one source has moves → use it directly
3. Both have moves → highest priority wins
4. Neither has moves → highest-priority source that exists
5. No source covers the species → stat-derived base entry

Each JSON entry records its origin in `_buildSource`:
`"Pikalytics"`, `"Smogon"`, `"Pikalytics>Smogon"`, or `"Base"`.

---

## Multi-Form Pokémon

The applier matches form automatically and falls back to form 0 if no form-specific
entry exists. Multi-form species use a JSON array with a `"form"` field per element.

---

## EV Spread Fallback

Used when no dictionary entry exists. The heuristic classifies each species by stat
profile and picks the matching preset.

| Preset | HP | Atk | Def | SpAtk | SpDef | Spd |
|---|---|---|---|---|---|---|
| PhysicalSweeper | 6 | 252 | 0 | 0 | 0 | 252 |
| SpecialSweeper | 6 | 0 | 0 | 252 | 0 | 252 |
| PhysicalWall | 252 | 0 | 252 | 0 | 6 | 0 |
| SpecialWall | 252 | 0 | 6 | 0 | 252 | 0 |
| MixedWall | 252 | 0 | 128 | 0 | 128 | 0 |
| BulkyAttacker | 252 | 252 | 0 | 0 | 6 | 0 |
| BulkySpecial | 252 | 0 | 0 | 252 | 6 | 0 |

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
├── BuildsJson.cs               ← JSON engine + GameBuildsConfig + BDSPBuildsJson
├── MappingParser.cs            ← parses CSV into AbilityMapping objects
├── CompetitiveBuilds.cs        ← PokemonBuild record, EVSpread, master registry
├── GenDetector.cs              ← detects save generation and game
├── README.md
├── CHANGELOG.md
└── builds/                     ← per-game data (runtime, not compiled)
    ├── bdsp/
    │   └── bdsp_builds.json    ← populate via Refresh Builds / Merge Refresh
    ├── swsh/  ← shell
    ├── pla/   ← shell
    ├── hgss/  ← shell
    ├── oras/  ← shell
    ├── usum/  ← shell
    ├── dppt/  ← shell
    ├── xy/    ← shell
    ├── lgpe/  ← shell
    ├── sm/    ← shell
    ├── rse/   ← shell
    ├── frlg/  ← shell
    ├── gsc/   ← shell
    └── rby/   ← shell
```

> The `builds/` folder is the plugin's runtime data directory, stored alongside the
> DLL in your PKHeX `plugins/` folder. It holds one JSON file per game containing the
> competitive build dictionary — abilities, natures, EV spreads, and movesets — loaded
> at startup and populated via Refresh Builds or Merge Refresh. The folder structure
> and empty shell JSON files for all supported games are created automatically on first
> launch. Shell game folders become active once Refresh Builds support is added for
> that game.

---

## JSON Format Reference

Single-form species:
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

Multi-form species:
```json
"479": [
  { "form": 0, "name": "Rotom",   "ability": "Levitate", "nature": "Timid", ... },
  { "form": 2, "name": "Rotom-W", "ability": "Levitate", "nature": "Timid", ... }
]
```
