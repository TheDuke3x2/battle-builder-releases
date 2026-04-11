# Battle Builder — PKHeX Plugin

A PKHeX plugin for bulk-applying competitive builds to Pokémon across save file boxes.
Applies abilities, natures, stat natures, EVs, movesets, level, IVs, and hyper training —
individually or all at once — using a mapping list paired with a live competitive build
dictionary sourced from Pikalytics, Smogon, and/or Auto Build.

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
paste a species-only list and use **Check All**:

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

All checkboxes default to **off**. Use **Check All** to enable all competitive options
at once, **Uncheck All** to clear.

| Option | What it does |
|---|---|
| **Ability** | Sets ability from CSV or dictionary. Reverted if PKHeX marks the result illegal. |
| **Nature** | Sets base Nature. Reverted if illegal — use Allow Illegal to force. |
| **Stat Nature** | Sets Stat Nature (mint effect) independently of base Nature. |
| **Competitive EVs** | Applies EV spread from dictionary. |
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
at runtime. Populate it via the **🔄 Refresh** button.

### Source Priority List

The source panel shows three sources in priority order (top = highest):

| Source | Coverage | Moves |
|---|---|---|
| **Pikalytics** | All 493 BDSP species (usage-based) | Top 4 by usage % |
| **Smogon** | Competitively relevant species only | First listed set |
| **Auto Build** | All species and all alternate forms | Stat-derived (see below) |

Check or uncheck Pikalytics and Smogon to include them. **Auto Build is always active**
and always last — it fills any species or form not covered by the checked sources.

Use **▲ / ▼** to reorder Pikalytics and Smogon. Click **🔄 Refresh** to fetch and merge
all checked sources in the displayed priority order.

**Merge logic per species:**
1. Highest-priority checked source with moves → use it
2. Only one checked source has moves → use it directly
3. Both checked sources have moves → highest priority wins
4. No checked source covers the species → Auto Build

Each JSON entry records its origin in `_buildSource`:
`"Pikalytics"`, `"Smogon"`, `"Pikalytics>Smogon"`, or `"Auto Build"`.

---

## Auto Build

Auto Build generates a complete entry for every species (and alternate form) not covered
by Pikalytics or Smogon. All fields are derived from base stats:

- **Ability** — Hidden Ability if available, otherwise Ability 1
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
Auto Build generates entries for all alternate forms independently, each with their own
stat-derived ability, nature, EVs, and moves.

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
├── CompetitiveBuilds.cs        ← PokemonBuild record, EVSpread, role pools, registry
├── GenDetector.cs              ← detects save generation and game
├── README.md
├── CHANGELOG.md
└── builds/                     ← per-game data (runtime, not compiled)
    ├── bdsp/
    │   └── bdsp_builds.json    ← populate via 🔄 Refresh
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
> at startup and populated via Refresh. The folder structure and empty shell JSON files
> for all supported games are created automatically on first launch.

---

## JSON Format Reference

Single-form species (Pikalytics source):
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

Auto Build entry with role and egg move delimiter:
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

Multi-form species:
```json
"487": [
  { "form": 0, "name": "Giratina",          "_buildSource": "Auto Build", ... },
  { "form": 1, "name": "Giratina (Origin)", "_buildSource": "Auto Build", ... }
]
```
