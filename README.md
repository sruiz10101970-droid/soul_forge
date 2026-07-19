# Soulforge

**A single-file browser-based RPG character builder and combat simulator, with a dark medieval theme.**

Soulforge is a self-contained playground for building custom heroes, tuning their stats, giving them clans, abilities, weapons and armor, and pitting them against each other in a proportional-wheel combat system. Everything runs locally in the browser — no server, no build step, no dependencies. All progress saves automatically to your browser's local storage.

---

## Table of Contents

- [Overview](#overview)
- [Features at a glance](#features-at-a-glance)
- [Getting started](#getting-started)
- [User guide](#user-guide)
  - [The roster](#the-roster)
  - [Characters and identity](#characters-and-identity)
  - [Stats](#stats)
  - [Vitals: HP and Energy](#vitals-hp-and-energy)
  - [The combat wheel](#the-combat-wheel)
  - [The manual dice roller](#the-manual-dice-roller)
  - [Inventory and weapons](#inventory-and-weapons)
  - [Armor and damage types](#armor-and-damage-types)
  - [Clans](#clans)
  - [Abilities](#abilities)
  - [Wallet and sheckles](#wallet-and-sheckles)
  - [The Wipe wheel](#the-wipe-wheel)
  - [Tiers](#tiers)
- [Math reference](#math-reference)
- [Data and storage](#data-and-storage)
- [Tech](#tech)
- [File structure](#file-structure)
- [Contributing](#contributing)
- [License](#license)
- [Roadmap](#roadmap)

---

## Overview

Soulforge is a hobbyist tool for anyone who wants to:

- Design characters for their own TTRPG homebrew.
- Playtest stat systems, class balance, and combat math without dice and paper.
- Build power-scaled rosters — the tier system lets a "House"-level rogue and a "Planetary"-level champion coexist in the same file.
- Have a fast, offline character sheet that isn't tied to any particular ruleset.

It is **not** a copy of D&D, Pathfinder, or any specific game. It's its own light system built around a six-stat point-buy, a proportional hit-chance wheel, and a compact modifier chain (score → clan → weapon → ability) that flows through every damage and combat calculation.

The app is one HTML file (currently about 3000 lines including everything). Open it in a browser and you're ready.

---

## Features at a glance

- **Point-buy stat system** — six stats (Willpower, Strength, Speed, Fortitude, Energy, Soul), one point per step, automatic multipliers.
- **Multi-character roster** — create, switch, hide, and delete characters; everything persists automatically.
- **HP and Energy tracking** — max scales with Fortitude and Energy; damage and healing controls; separate current vs. max.
- **Proportional combat wheel** — hit chance derived from effective stats, with attacker/defender/damage-stat/damage-type pickers.
- **Manual dice roller** — d4/d6/d8/d10/d12/d20, count and modifier, optional stat bonus, crit/fumble detection on single d20.
- **Weapons in inventory** — mark slots as weapons with bonus dice (+Nd?) and per-stat multipliers, one equipped at a time.
- **Armor system** — 4 slots (Head/Body/Arms/Legs) with independent weight and defense, Strength offsets Speed penalty, per-damage-type percent resistances that stack.
- **8 damage types** — Piercing, Slashing, Bludgeoning, Fire, Cold, Poison, Bleeding, Psychic.
- **Clans** — shared clan list with editable per-stat multipliers per clan.
- **Abilities** — per-character list with stat multiplier (applied to checked stats), damage multiplier (scales the whole roll), and one-at-a-time active toggle.
- **Total-stats damage scaling** — every 5 combined stat points adds +0.3 to the damage multiplier.
- **Wallet** — copper/silver/gold sheckles per character.
- **Wipe wheel** — spinning random-picker for Soul / Race / Clan with editable lists.
- **Power tiers** — House / Block / City / Mountain / Continental / Planetary badge next to each character's name.
- **Fully local** — all data is stored in your browser; no accounts, no cloud, no telemetry.

---

## Getting started

**Try it right now.** Download `soulforge.html`, double-click it, and it opens in your default browser. That's it.

**Running locally in a specific browser.** Right-click the file and pick your browser. All modern browsers (Chrome, Firefox, Safari, Edge) work fine.

**Serving over localhost (optional).** If you're testing something in an environment where `file://` URLs behave oddly, you can serve it locally:

```bash
cd path/to/soulforge
python3 -m http.server 8000
# then open http://localhost:8000/soulforge.html
```

**Persistence note.** All your data lives in the browser's `localStorage` for the origin the file is opened from. This means:

- Opening the file from `file:///Users/you/soulforge.html` uses a different storage bucket than `http://localhost:8000/soulforge.html`. If you switch between them, your rosters will look different.
- Clearing your browser's site data will erase everything. Back up important characters by copying the JSON out of storage (see [Data and storage](#data-and-storage)) if this matters to you.
- Different browsers (Chrome vs Firefox) never share storage.

---

## User guide

Below is a full walkthrough of every system in the app, in the order you're likely to touch them.

### The roster

The horizontal strip at the top of the page is the **roster bar**. Each character has a chip showing their name, a **◐** half-moon button to hide them, and a **×** button to delete them. Click a chip to switch to that character. Click **+ New Character** to add a fresh one.

You can never hide or delete your last visible character — the app enforces at least one active hero. If you hide the character you're currently viewing, the app automatically switches you to the next visible one.

When you have hidden characters, an underlined **"N hidden — manage"** link appears at the end of the roster bar. Clicking it opens a popup listing the hidden heroes with their basic info and **Show** buttons to bring them back.

### Characters and identity

Each character has:

- **Name** — free text.
- **Tier badge** — a colored badge next to the name (House through Planetary; see [Tiers](#tiers)).
- **Tier dropdown** — pick from the six tiers.
- **Soul** — free text (a name for their essence, technique, or magical school).
- **Race** — free text.
- **Clan** — free text; typing a clan name auto-adds it to the shared clan list, so it also appears in the Wipe wheel and Clans editor.
- **Level** and **per-level points** — how many stat points they get per level (default 10). Base is 10 at level 1.
- **+ Level Up** — increments level and grants points.

Below the name you'll find HP and Energy bars, then the six stat rows.

### Stats

Six stats sit in a fixed order:

| Stat        | Key   | Role                                                                                     |
| ----------- | ----- | ---------------------------------------------------------------------------------------- |
| Willpower   | `wil` | Mental resilience; magical presence.                                                     |
| Strength    | `str` | Physical power. Offsets armor Speed penalty. Contributes to total-stats damage scaling.  |
| Speed       | `spd` | Reflexes, initiative. Reduced by heavy armor.                                            |
| Fortitude   | `for` | Endurance; drives HP maximum.                                                            |
| Energy      | `enr` | Resource pool; drives Energy maximum.                                                    |
| Soul        | `sol` | Mystical force. Symmetric with Willpower on the "magic" side.                            |

Each stat has:

- A **score** (minimum 1, no upper cap — capped only by available points).
- A **− / +** pair to spend or refund points.
- A **live multiplier** display (auto-derived, read-only).
- An **effective value** (score × multiplier chain).
- A visual bar.

**Score multiplier** is derived automatically from the score:

```
scoreMult = 1 + floor((score - 1) / 5)
```

So scores 1–5 give ×1, 6–10 give ×2, 11–15 give ×3, and so on. This is the base multiplier before clan / weapon / ability modifiers stack in.

**Effective value** is the multiplier chain:

```
effective = score × scoreMult × clan × weapon × ability
```

Where clan, weapon, and ability are each ×1 by default. Effective values feed the combat wheel's hit-chance math and the dice roller's stat bonuses.

**Point pool** in the header shows remaining / total. Total = `10 + (level − 1) × perLevelPoints`.

### Vitals: HP and Energy

Right below the name field are the two vitals bars:

- **HP** — max is `20 + (Fortitude − 1) × 30`. So Fort 1 → 20, Fort 5 → 140, Fort 10 → 290, Fort 20 → 590.
- **EN** (Energy) — max is `10 + (Energy − 1) × 20`. So Energy 1 → 10, Energy 5 → 90, Energy 10 → 190.

Each bar has:

- **Current / max** readout.
- An **amount** input.
- **Damage** and **Heal** buttons (apply the amount to current).
- **Full** to snap current to max.

Current values persist per character, so you can wound a character, switch to someone else, and come back to the same wound level.

### The combat wheel

The heart of the app. Two roster characters face off in a proportional-hit-chance calculation.

**Setup fields:**

- **Attacker** — dropdown of visible characters.
- **Target** — dropdown of visible characters.
- **Attack stat** — the stat that decides the attacker's share of the wheel.
- **Defense stat** — the stat that decides the target's share.
- **Damage stat** — optional; adds a stat bonus to the damage roll.
- **Damage type** — one of the 8 types; determines which resistance applies.
- **Damage die** — the base die to roll on hit (d4–d20).

**The math:**

```
atkEff = attacker's effective(attack stat)
defEff = target's effective(defense stat)
atkDeg = atkEff / (atkEff + defEff) × 360
hitChance % = atkDeg / 3.6
```

The wheel visually shows the split: the green "HIT" arc is `atkDeg` degrees, the red "MISS" arc is `360 − atkDeg`. Edge cases: if one side has effective 0, they get the full wheel (all-HIT or all-MISS). If both are 0, it falls back to a fair 50/50.

**Spinning:** the button rolls a real random check against the exact odds and lands the pointer in the correct slice with a 3.7s spin animation.

**On HIT:** the game rolls damage using the manual dice + stat bonus + weapon dice + ability multiplier + total-stats multiplier, applies target armor mitigation, and subtracts from the target's current HP. The result readout shows every component: base dice, flat modifier, stat bonus, weapon contribution, ability × multiplier, total-stats × multiplier, damage type, resistance %, armor reduction, and final HP.

### The manual dice roller

Independent of the combat wheel, this rolls dice for the active character.

- **Die buttons** — d4 through d20.
- **Count** — how many dice to roll.
- **Modifier** — flat + or −.
- **Stat bonus** — dropdown; adds the character's `mod(score) × clanMult × weaponMult × abilityMult` for the picked stat. `mod(score) = floor((score − 1) / 5)`, so scores 1–5 give +0, 6–10 give +1, and so on.
- **Subtract from HP** — checkbox; if checked, the total damage is applied to the active character's current HP after their armor mitigates it (using the currently-selected damage type).

The equipped weapon's bonus dice roll on top automatically. Rolling a single d20 that shows 20 marks it as a critical hit; showing 1 marks it as a fumble.

### Inventory and weapons

Below the stats, the **Inventory** panel is a grid of slots (8 by default). Click any slot to add or edit an item.

**Making a weapon:**

1. Click an empty slot.
2. Enter a name.
3. When asked "Is this a weapon?", click OK.
4. Enter dice count (e.g. `1`) and die size (e.g. `6`) for the weapon's bonus dice (+1d6).

Weapon slots have a gold border, a **⚔** icon in the corner, and a small **+Nd?** badge showing their bonus dice.

**Equipping:** each weapon slot has an **Equip** button. Click it to equip that weapon — this unequips any others (one weapon at a time per character). The equipped weapon glows red. Its bonus dice roll on top of every damage total in both the manual roller and the combat wheel.

**Weapon multipliers:** when a weapon is equipped, a **Weapon multipliers** sub-panel appears below the inventory grid. It lists all six stats with editable × values that stack into that character's multiplier chain while the weapon is equipped. A **Reset to ×1** button clears them.

### Armor and damage types

The **Armor** panel sits below the Wallet, with four slots: **Head**, **Body**, **Arms**, **Legs**.

Each armor piece has:

- **Name** — free text; leave empty to remove the piece.
- **Weight** (0–20) — drives the Speed penalty. Strength offsets this: every 2 Strength cancels 1 weight of Speed drag.
- **Defense** (0–200) — flat damage reduction; independent from weight. New pieces default `defense = weight × 2`, but you can adjust either freely.
- **▸ Resistances** — a collapsible section with 8 percent inputs, one per damage type. Values 0–90 each.

**Aggregate effects across all four slots:**

- **Total Speed penalty** = `max(0, sum(weights) − floor(Strength / 2))`. This subtracts from raw Speed *before* the multiplier chain, so heavy armor lowers your effective Speed and shrinks your slice of the combat wheel.
- **Total flat defense** = `sum(defense values)`. Subtracted from every incoming damage total.
- **Total resistance %** for a given damage type = `sum of that type's resistance across all pieces`, capped at 90%.

**Mitigation order** on incoming damage:

```
afterPct  = raw × (1 − resistPct / 100)
finalTaken = max(0, afterPct − flatDefense)
```

So a 100 Fire hit against a suit with 40% fire resistance and 25 flat defense: `100 × 0.6 = 60`, then `60 − 25 = 35` HP lost. The full breakdown appears in the wheel's hit note.

**Damage type dropdown** in the combat panel picks the type for the current attack. This is the type used to look up the target's resistance, and it appears in the damage breakdown.

### Clans

Clans are a shared library across the roster (edit them once, use them everywhere). The **Clans** panel below the combat panel shows only the currently active character's clan card, so you're always editing what applies to the character in front of you.

**Adding a clan:** either type a new clan name into the Clan field on any character (auto-adds), or open the Wipe wheel and add options to the Clan wheel.

**Editing modifiers:** on the Clans panel, each stat has a × input (default ×1.0). A **Reset to ×1** button clears them.

Clan multipliers stack into every stat computation: displayed multiplier, effective value, dice stat bonus, combat wheel effective stats, and everything downstream.

### Abilities

Per-character list, saved with the character's data. Click **+ New ability** in the Abilities panel to add one.

Each ability card has:

- **Name** — free text.
- **STAT ×** — a stat multiplier applied only to stats you check below.
- **DMG ×** — a whole-damage-roll multiplier applied to every hit while this ability is active.
- **Stat checkboxes** — pick which stats the STAT × applies to.
- **Activate** button — toggles the ability on. Only one ability can be active at a time; activating a second deactivates the first.
- **×** — delete the ability.

Example: "Berserker" with STAT ×1.5 and DMG ×2, Strength checked. When active, this character's Strength is multiplied by ×1.5 (into the effective chain) and every damage roll doubled.

### Wallet and sheckles

Each character has their own **Wallet**, saved with their data. Denominations:

- **1 Silver (S)** = 100 Copper (C)
- **1 Gold (G)** = 10 Silver = 1000 Copper

The panel shows the running total in three colored badges. Inputs let you enter amounts in Gold, Silver, and Copper independently.

- **+ Add** — deposit the entered amount.
- **− Take** — withdraw. If you try to take more than you have, the wallet clamps to zero and flashes "not enough sheckles."
- **Quick add** — one-click buttons for +1G, +1S, +10C, +100C.
- **↺ Reset** — zero the wallet (with confirmation).

Internally, everything is stored as a single copper total to avoid rounding issues across denominations.

### The Wipe wheel

The **Wipe** button in the actions row opens a popup with three side-by-side spinning wheels: **Soul**, **Race**, and **Clan**. Each has:

- Its own editable list of options (chips with × to remove, plus an input + Add button).
- A wheel visualization.
- A result readout.

**Spin All** spins all three wheels at once and lands each on a random option. **Use Results** applies the picks to the active character's Soul, Race, and Clan fields.

Lists are shared across all characters (and persist between sessions). Handy for generating names quickly when starting a new hero.

### Tiers

Six manual power tiers, sorted by scale:

- **House** — small scale, personal power.
- **Block** — neighborhood-level threat.
- **City** — city-level, a major named villain or hero.
- **Mountain** — regional force of nature.
- **Continental** — continent-shaking power.
- **Planetary** — planet-tier apocalypse-class.

The tier is picked manually from a dropdown next to the name, and displays as a colored badge. Purely cosmetic and organizational — the app doesn't enforce any stat gates on tiers.

---

## Math reference

Everything in one place for reference. All values below assume MIN = 1 (the stat floor).

### Score multiplier

```
scoreMult(score) = 1 + floor((score - 1) / 5)
```

| Score | ×    |
| ----- | ---- |
| 1–5   | ×1   |
| 6–10  | ×2   |
| 11–15 | ×3   |
| 16–20 | ×4   |
| 21–25 | ×5   |

### Stat mod (for dice stat bonus)

```
mod(score) = floor((score - 1) / 5)
```

Same steps as scoreMult minus 1. Applies to the "Stat bonus" dropdown on the manual dice roller and to the combat wheel's damage-stat bonus.

### Effective value

```
effective(char, stat) = charScore × scoreMult × clan × weapon × ability
```

The `char.score` for Speed is first reduced by armor Speed penalty (see below), then fed into the multiplier chain.

### Hit chance (combat wheel)

```
atkEff = effective(attacker, attackStat)
defEff = effective(target,   defenseStat)
pool   = atkEff + defEff
atkDeg = pool > 0 ? atkEff / pool × 360 : 180
hit %  = atkDeg / 3.6
```

### HP and Energy

```
maxHP = 20 + (Fortitude - 1) × 30
maxEN = 10 + (Energy - 1)    × 20
```

### Damage total (before mitigation)

```
preTotal = diceSum + flatMod + statBonus + weaponBonusDice
statBonus = round(mod(scoreForDmgStat) × clanMult × weaponMult × abilityMult)
combinedMult = abilityDmgMult × totalStatsMult
total = round(preTotal × combinedMult)
```

Where:

```
totalStatsMult = 1 + floor(sumOfAll6Scores / 5) × 0.3
```

So Total 5 → ×1.3, Total 10 → ×1.6, Total 25 → ×2.5, Total 50 → ×4.0, Total 100 → ×7.0.

### Armor Speed penalty

```
totalWeight = sum of equipped armor weights
strOffset   = floor(Strength / 2)
speedPenalty = max(0, totalWeight − strOffset)
```

Applied to raw Speed *before* scoreMult, so it also drops the Speed multiplier tier at bigger penalties.

### Damage mitigation

```
resistPct = min(90, sum of target's resistances for this damage type)
flatDef   = sum of target's armor defense values
afterPct  = raw × (1 − resistPct / 100)
taken     = max(0, afterPct − flatDef)
```

---

## Data and storage

Everything is saved to your browser's `localStorage`. Two keys are used:

- `character-forge-roster-v1` — an object `{ roster: [...], activeId: "..." }` with all your characters, their stats, inventories, abilities, wallets, armor, and vitals. (The key is kept as `character-forge-` prefix for backward compatibility with existing saves from the app's earlier name.)
- `character-forge-wipe-lists-v1` — an object with the shared Soul, Race, and Clan lists, plus the map of per-clan stat modifiers.

**Backing up:** open your browser's DevTools (F12), go to the Application / Storage tab, find Local Storage for the file's origin, and copy the two values out to a text file. To restore, paste them back in the same place.

**Wiping:** clearing site data / cookies for the origin will erase everything. Doing this in one browser doesn't touch the other browsers' data.

**No cloud, no accounts, no telemetry.** Everything you do is entirely local. Close the tab and re-open it, and it comes right back.

---

## Tech

- **Vanilla HTML, CSS, JavaScript** — one file, no framework.
- **No dependencies, no build step.** Open and go.
- **Google Fonts** — Cinzel, Cormorant Garamond, IM Fell English SC (loaded via CDN; the page works offline once fonts are cached, but the first load needs internet).
- **SVG** for the combat wheel and Wipe wheels.
- **`localStorage`** for persistence.

The entire codebase lives in `soulforge.html`. The `<script>` block is roughly 2500 lines and organized top-to-bottom by feature: state & helpers → stats & vitals → inventory & weapons → abilities → clans → armor → wallet → combat wheel → manual dice → Wipe popup → roster & persistence → init.

---

## File structure

```
soulforge/
├── soulforge.html    # The entire app
├── README.md         # This file
└── .gitignore
```

That's it. Nothing else to install or configure.

---

## Contributing

Contributions are welcome. Since the whole app is a single file, most changes are straightforward find-and-edit jobs.

**Ways to help:**

- Report bugs by filing a GitHub Issue with steps to reproduce.
- Suggest features via Issues.
- Send pull requests for bug fixes, small improvements, or new features. Please keep the single-file structure — no build tooling.
- If you change stat / damage math, please update the [Math reference](#math-reference) section of this README too.

**Coding style:**

- Vanilla JS, no transpilation.
- Function names in `camelCase`, constants in `UPPER_SNAKE_CASE`.
- Prefer clear, boring code over clever tricks. This is a hobby tool; readability wins.

**Testing changes:** open the file in a browser, use it, verify existing characters still load correctly. Consider bumping the storage key version (e.g. `-v2`) if you make a breaking data change.

---

## License

No license has been chosen yet. Until one is added, all rights are reserved by the repository owner. If you'd like to use, fork, or redistribute this project, please open an Issue to discuss.

Suggested licenses if you want to open it up:

- **MIT** — permissive, easy sharing.
- **CC BY-SA 4.0** — permissive with share-alike (good for creative/tabletop-adjacent projects).
- **GPL-3.0** — copyleft.

Add a `LICENSE` file at the root with the full text once you pick one.

---

## Roadmap

Ideas for future features (unordered, unpromised):

- **Export / import** characters and rosters as JSON files.
- **Character sheet export** as PDF or printable HTML.
- **Battle log** — a scrollable history of all combat rolls.
- **Status effects** — poisoned, burning, stunned, etc., with duration.
- **Ability energy costs and cooldowns.**
- **Weapon and armor crafting** with sheckle costs.
- **Preset armor types** (Light / Medium / Heavy) as templates.
- **Multi-target attacks** (cleave, AOE).
- **Custom stats** — user-defined 7th and 8th stats.
- **Party mode** — a shared "party" screen showing all visible heroes at once.
- **Theme picker** — light / dark / parchment.
- **Mobile-friendly layout** — the app works on mobile but the layout is desktop-first.

---

**Built as a personal RPG tool.** If you find it useful, star the repo — it helps other people discover it.
