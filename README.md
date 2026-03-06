# malto-tracker
It's a simple app to track my character in my D&amp;D campaign. Think of it as an overly engineered and overly specific character sheet. Feel free to look around.
https://christiananggoro.github.io/malto-tracker/

Feel free to iterate based on your characters' need. Be aware that this is for a specific character. Therefore the following blocks might not be useful for you: Jack of All Trades, Bardic features, Entertainer features, werewolf features. Remove the unecessary and write your own blocks as per your DM's ruling. 
DISCLAIMER: I can not code. Evey line in this code is written using Claude Sonnet 4.6. Below is the documentation note, written by Claude Sonnet 4.6.

CLAUDE HANDOVER NOTE:
# Malto Dextrin — D&D 5e Stat Tracker

> A browser-based character tracker built for **Malto Dextrin**, Level 3 Human Bard (College of Eloquence), Myshem campaign.  
> Single-file React app. No server. No build tools. Runs entirely in the browser.

-----

## How to Run

### Option A — Claude.ai (quickest)

Upload `dnd_tracker.jsx` to a Claude.ai chat. It renders instantly.

### Option B — GitHub Pages (permanent URL, cross-device)

1. Upload `index.html` to a GitHub repository
1. Enable GitHub Pages (Settings → Pages → Deploy from branch: main / root)
1. Your URL: `https://yourusername.github.io/your-repo-name`

### Option C — Local file

Open `index.html` directly in any browser. No server needed.

-----

## Files

|File             |Purpose                                                      |
|-----------------|-------------------------------------------------------------|
|`dnd_tracker.jsx`|Source of truth — React JSX, upload to Claude to develop     |
|`index.html`     |Self-contained browser build — upload to GitHub Pages to host|
|`README.md`      |This file                                                    |


> **Rule:** Always develop in `dnd_tracker.jsx`. Regenerate `index.html` from it when ready to deploy.

-----

## Save System

The app uses **`localStorage`** under the key `malto_v3`. Saves are:

- ✅ Persistent across tab closes and browser restarts
- ❌ Per device, per browser (Chrome ≠ Firefox, phone ≠ laptop)

### Cross-device workflow

Use **REST → SAVE / LOAD** in the app:

- **Export** — copies your full save as a JSON text blob
- **Import** — paste a previously exported blob to load it

Store the exported text in OneDrive / Google Drive / Mega to sync between devices.

### localStorage key versioning

The key is `malto_v3`. If you make a **breaking change to the state shape** (removing or renaming fields), increment the version: `malto_v4`. This prevents old saves from corrupting the new schema.

-----

## Architecture

### Tech Stack

- **React 18** (via CDN in `index.html`, via import in `.jsx`)
- **Babel Standalone** — transpiles JSX in the browser, no build step
- **Google Fonts** — Orbitron (headings) + Share Tech Mono (body)
- **Zero dependencies** beyond the above

### State

All mutable game data lives in one object, persisted to localStorage on every change via the `st()` helper:

```js
const st = useCallback((upd) => {
  setRaw(prev => {
    const next = typeof upd === "function" ? upd(prev) : upd;
    saveState(next);   // writes to localStorage immediately
    return next;
  });
}, []);
```

#### Full State Shape

```js
{
  charLevel,          // int 1–20
  hp,                 // int 0..maxHp
  maxHp,              // int
  ac,                 // int (auto-adjusted by Beast Mode ±1)
  initMod,            // int
  speed,              // int (auto-adjusted by Beast Mode ±10)
  pasPerc,            // int
  abilities,          // array[8]: [STR, INT, DEX, WIS, CON, CHA, PB, HI]
  slotUsed,           // array[9]: used spell slots per level (index 0 = L1)
  bardicUsed,         // int 0–4
  encouragingSongUsed,// int 0–PB
  currency,           // array[4]: [PP, GP, SP, CP]
  conditions,         // object: { "Poisoned": true, "Exhaustion": 3, ... }
  inventory,          // array: [{ name: string, qty: number }]
  hitDiceUsed,        // int 0..charLevel+bonusHitDice
  bonusHitDice,       // int (1 for lycanthropy, adds to hit dice pool)
  beastMode,          // bool
}
```

**Ability array indices:** `0=STR, 1=INT, 2=DEX, 3=WIS, 4=CON, 5=CHA, 6=PB, 7=HI`

#### DEFAULT_STATE

Malto’s starting values. Edit this when levelling up or making permanent stat changes.

```js
const DEFAULT_STATE = {
  charLevel:3, hp:19, maxHp:24, ac:13, initMod:4, speed:30, pasPerc:12,
  abilities:[15,14,14,14,13,18,2,0],
  slotUsed:[0,0,0,0,0,0,0,0,0],
  bardicUsed:0, currency:[0,282,8,0],
  conditions:{}, inventory:[],
  hitDiceUsed:0, bonusHitDice:1,
  encouragingSongUsed:0, beastMode:false,
};
```

-----

## Pages

|#|Page        |Editable                                  |Notes                         |
|-|------------|------------------------------------------|------------------------------|
|0|INFO        |—                                         |Hardcoded character info      |
|1|ABILITIES   |STR,INT,DEX,WIS,CON,CHA,PB,HI             |HI is boolean toggle          |
|2|SKILLS      |—                                         |Auto-calculated, read-only    |
|3|COMBAT      |Level,HP,MaxHP,AC,Init,Speed,PassPerc     |HP capped at maxHp            |
|4|SPELLCASTING|—                                         |Derived from CHA+PB, read-only|
|5|SPELL SLOTS |slotUsed[0..n]                            |Level-gated                   |
|6|FEATURES    |bardicUsed, encouragingSongUsed, beastMode|Beast Mode is direct tap      |
|7|CONDITIONS  |All 15 conditions + Exhaustion 1–6        |Edit mode only                |
|8|INVENTORY   |items[{name,qty}]                         |Add via input, delete via ✕   |
|9|CURRENCY    |PP,GP,SP,CP                               |Max 99999 each                |

Read-only pages: `0, 2, 4` — defined in `const READ_ONLY = new Set([0,2,4])`.

-----

## Key Patterns & Gotchas

### Panel rendering — CRITICAL

`LeftPanel` and `RightPanel` are arrow functions defined inside `App`. They **must** be called as `{LeftPanel()}` and `{RightPanel()}` — **NOT as `<LeftPanel/>`**. Using JSX tags causes React to remount them on every render, destroying HoldBtn timers.

### HoldBtn

Fires `onFire` on `pointerdown`, then every 100ms after a 600ms hold. Uses native DOM `addEventListener` (not React synthetic events) to work inside iframes. The `onFire` callback is stored in a ref (`fireRef.current`) so intervals always call the latest version without stale closures.

### adjRef pattern

```js
adjRef.current = (delta) => { /* switch(page) ... */ };
const adjUp   = useCallback(() => adjRef.current(1),  []);
const adjDown = useCallback(() => adjRef.current(-1), []);
```

`adjUp`/`adjDown` are stable references. The actual logic lives in `adjRef.current` so HoldBtn intervals never go stale.

### fieldCount

```js
const fieldCount = (pg, lv, invLen) =>
  [0, 8, 0, 7, 0, unlockedSlots(lv), 2, 15, invLen||0, 4][pg] || 0;
```

Array index = page number. Value = number of selectable fields on that page.  
Update this if you add new editable fields to any page.

### Inventory input

Uses an **uncontrolled input with a ref** (`defaultValue=""`, read via `inputRef.current.value`). Controlled inputs are unreliable inside the Claude artifact iframe. The ADD button is a plain `<button onClick>`, not a `Btn` component.

-----

## Game Formulas (5e RAW)

|Formula              |Expression                   |
|---------------------|-----------------------------|
|Ability Modifier     |`floor((score - 10) / 2)`    |
|Jack of All Trades   |`floor(PB / 2)`              |
|Save (proficient)    |`abilMod + PB`               |
|Save (not proficient)|`abilMod`                    |
|Skill (expertise)    |`abilMod + PB * 2`           |
|Skill (proficient)   |`abilMod + PB`               |
|Skill (none)         |`abilMod + JoAT`             |
|Spell Save DC        |`8 + PB + CHAmod`            |
|Spellcasting Mod     |`CHAmod`                     |
|Spell Attack Bonus   |`PB + CHAmod`                |
|Passive Perception   |`10 + Perception skill value`|

-----

## Malto-Specific Features

### Lycanthropy (Beast Mode)

Malto is infected with lycanthropy (DM ruling). Beast Mode is a toggle on the FEATURES page.

**Permanent (always active):**

- STR = 15 (set in DEFAULT_STATE)
- Hit Dice pool = charLevel + bonusHitDice (bonusHitDice = 1)
- Max HP = 24 (set manually via COMBAT edit)

**Beast Mode ON only:**

- AC +1 (auto-applied on toggle, reverted on toggle off)
- Speed +10ft (same)
- Bite: +4 to hit, 1d8+2 piercing
- Claw: +4 to hit, 2d4+2 slashing

**Blood Lust Mode** (Beast Mode ON + HP < 50%):

- Full UI turns red — panels, screen, everything
- Titlebar throbs with a glow animation

### Beast Mode theming — how it works

```js
const beastActive = state.beastMode;
const bloodLust   = beastActive && (state.hp / state.maxHp) < 0.5;
const screenTheme = bloodLust ? "bloodlust-screen" : beastActive ? "beast-screen" : "";
const panelTheme  = bloodLust ? "bloodlust-panel" : "";
```

CSS classes override CSS variables. `bloodlust-panel` also forces `background: #130000 !important` directly because CSS variables on `.side` are set at `:root` level and don’t cascade from a sibling class.

-----

## Bard Spell Slots (BARD_SLOTS)

A 20×9 array. `BARD_SLOTS[level-1][spellLevel-1]` = max slots. Used by `slotMax()` and `unlockedSlots()`.  
Already fully populated for levels 1–20. No changes needed unless your DM grants bonus slots.

-----

## REST System

|Rest Type      |What it does                                                                                                  |
|---------------|--------------------------------------------------------------------------------------------------------------|
|Long Rest      |HP→max, all slots restored, bardicUsed→0, encouragingSongUsed→0, hitDiceUsed reduced by max(1, floor(hdMax/2))|
|Short Rest     |HP += hdGain, spends 1 hit die, bardic restored only at level 5+                                              |
|Restart Session|Long rest + clears conditions + turns off Beast Mode (reverts AC/speed) + shows intro splash                  |

-----

## Theming / CSS Variables

All CSS lives in `const CSS` (one inline template string). Variables on `:root`:

|Variable|Value    |Role                     |
|--------|---------|-------------------------|
|`--bg`  |`#141618`|Root background          |
|`--pan` |`#1c1f22`|Panel background         |
|`--scr` |`#111315`|Screen background        |
|`--bd`  |`#2e3338`|Primary border           |
|`--bd2` |`#222629`|Secondary border         |
|`--hi`  |`#dde1e5`|Primary text             |
|`--mid` |`#7a8490`|Muted / label text       |
|`--dim` |`#3a4148`|Inactive accents         |
|`--sel` |`#242830`|Selected field background|
|`--am`  |`#d4a017`|Amber — edit, selection  |
|`--rd`  |`#c94040`|Red — danger, dying      |
|`--gr`  |`#3a9060`|Green — save, ok         |

-----

## Adapting for a New Character / Class

Here’s what to change for a fresh character:

### 1. Character info

```js
const CHAR = {
  name: "Your Character",
  class: "Fighter",
  subclass: "Battle Master",
  race: "Elf",
  background: "Soldier",
  campaign: "Your Campaign",
};
```

### 2. DEFAULT_STATE

Update all starting values. Remove `bonusHitDice` if no special dice pool. Set `beastMode: false` and remove Beast Mode fields if not applicable.

### 3. Spell slots

`BARD_SLOTS` is Bard-specific. Replace with the correct table for your class, or remove spell-related pages if the character is non-magical.

### 4. SKILL_GROUPS

Update `prof` values to match the new character’s proficiencies:

- `prof: 0` = no proficiency (gets Jack of All Trades if applicable)
- `prof: 1` = proficient
- `prof: 2` = expertise

### 5. Features page

`BardicUsed` and `encouragingSongUsed` are Bard-specific. Replace with class features relevant to the new character (e.g. Superiority Dice for Battle Master, Ki points for Monk, Rage uses for Barbarian).

### 6. fieldCount

If you change the number of editable fields on any page, update `fieldCount`:

```js
const fieldCount = (pg, lv, invLen) =>
  [0, 8, 0, 7, 0, unlockedSlots(lv), 2, 15, invLen||0, 4][pg] || 0;
//   ↑  ↑  ↑  ↑  ↑        ↑          ↑   ↑       ↑      ↑
//   0  1  2  3  4         5          6   7        8      9  ← page index
```

### 7. localStorage key

Increment the version suffix to avoid stale saves from the old character:

```js
localStorage.getItem("malto_v3")  →  localStorage.getItem("newchar_v1")
```

-----

## Known Backlog

- No in-app data reset button (workaround: open browser console → `localStorage.removeItem('malto_v3')`)
- No escape hatch from dying mode (must roll 3 successes or 3 failures)
- No XP / level-up tracker
- No Notes / free text page
- Short Rest HP input is manual (player rolls physical dice and enters the result)

-----

## Handoff Notes for Claude

When resuming development in a new chat, provide:

1. `dnd_tracker.jsx` — current source
1. This `README.md`

Tell Claude to read both before writing any code. Key things to re-read before touching code:

- **Panel rendering** — never use `<LeftPanel/>`, always `{LeftPanel()}`
- **Inventory input** — uncontrolled ref, plain `<button onClick>` for ADD
- **Beast Mode AC/speed** — always revert on toggle off, check for sync issues
- **Brace balance** — run the Python brace checker after any structural edits:

```python
src = open('dnd_tracker.jsx').read()
lines = src.split('\n')
stack = []
for i, line in enumerate(lines, 1):
    for j, ch in enumerate(line):
        if ch == '{': stack.append((i, j))
        elif ch == '}':
            if stack: stack.pop()
print("Unmatched opens:", len(stack))
for item in stack:
    print("  Line", item[0], ":", lines[item[0]-1][:80])
```
