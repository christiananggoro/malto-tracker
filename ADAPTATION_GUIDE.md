# Adapting This Tracker for Your Own Character
## A Guide for Players & Developers

> This tracker was built for **Malto Dextrin**, a Level 3 Human Bard with lycanthropy in the Myshem campaign. It is fully open for adaptation to any D&D 5e character.  
>
> As noted in the README: *"Feel free to iterate based on your characters' need. Be aware that this is for a specific character. Therefore the following blocks might not be useful for you: Jack of All Trades, Bardic features, Entertainer features, werewolf features. Remove the unnecessary and write your own blocks as per your DM's ruling."*

This guide walks you through how to do exactly that — with or without coding experience, using an AI assistant like Claude.

---

## Table of Contents
1. [What You Need](#what-you-need)
2. [The Golden Rule](#the-golden-rule)
3. [The Easy Stuff — No Code Required](#the-easy-stuff--no-code-required)
4. [Malto-Specific Blocks to Remove or Replace](#malto-specific-blocks-to-remove-or-replace)
5. [Using Claude to Make Changes](#using-claude-to-make-changes)
6. [Common Adaptations by Class](#common-adaptations-by-class)
7. [Prompting Tips for AI Assistance](#prompting-tips-for-ai-assistance)
8. [Deploying Your Version](#deploying-your-version)
9. [Keeping Your Changes Safe](#keeping-your-changes-safe)

---

## What You Need

| Item | Purpose |
|------|---------|
| `dnd_tracker.jsx` | The source file — this is what you edit |
| `HANDOVER.md` | Full technical architecture reference for developers |
| `USER_MANUAL.md` | End-user reference for how the app works |
| A Claude.ai account (free tier works) | To make code changes without writing code yourself |

You do **not** need to know how to code. You do **not** need to install anything. Claude can read the files and make changes for you based on plain English instructions.

---

## The Golden Rule

> **Always work from `dnd_tracker.jsx`, not `index.html`.**

`dnd_tracker.jsx` is the readable source file. `index.html` is the deployable build — it's the same code but packed into a single HTML file for GitHub Pages.

**Workflow:**
1. Make all your changes in `dnd_tracker.jsx` via Claude
2. Test it by uploading the `.jsx` to Claude.ai and previewing it
3. Once happy, ask Claude to generate a fresh `index.html` from it
4. Upload the new `index.html` to your GitHub repo

---

## The Easy Stuff — No Code Required

These changes don't require touching code logic — just find and replace values.

### Change the Character Name & Info
In `dnd_tracker.jsx`, find this block near the top:

```js
const CHAR = {
  name:"Malto Dextrin", class:"Bard", subclass:"College of Eloquence",
  race:"Human", background:"Entertainer", campaign:"Myshem",
};
```

Replace every value with your character's details. This populates the **INFO page**.

---

### Change Starting Stats
Find `DEFAULT_STATE` near the top of the file:

```js
const DEFAULT_STATE = {
  charLevel:3, hp:19, maxHp:24, ac:13, initMod:4, speed:30, pasPerc:12,
  abilities:[15,14,14,14,13,18,2,0],
  ...
};
```

Update every value to match your character sheet:

| Field | What it is |
|-------|-----------|
| `charLevel` | Character level |
| `hp` / `maxHp` | Current and max hit points |
| `ac` | Armour Class |
| `initMod` | Initiative modifier (DEX mod, or DEX+PB for Alert feat, etc.) |
| `speed` | Base movement speed in feet |
| `pasPerc` | Passive Perception |
| `abilities` | `[STR, INT, DEX, WIS, CON, CHA, PB, HI]` — your 6 ability scores, then Proficiency Bonus, then Half-caster flag (0 or 1) |
| `currency` | `[PP, GP, SP, CP]` — starting gold |

---

### Change Skill Proficiencies
Find `SKILL_GROUPS` near the top of the file. Each skill has a `prof` value:

```js
{name:"Perception", prof:0, isSave:false, abil:3},
```

| `prof` value | Meaning |
|-------------|---------|
| `0` | No proficiency |
| `1` | Proficient |
| `2` | Expertise (double proficiency) |

Go through each skill and update `prof` to match your character sheet. This automatically updates the **SKILLS page** calculations.

---

### Change Spell Slots
Malto uses the **Bard spell slot table**. If your character uses a different class's spell progression, find `BARD_SLOTS` and replace it with the correct table.

For **non-spellcasters** (Fighter, Barbarian, etc.), you can remove the SPELL SLOTS page entirely — ask Claude to do this.

For **half-casters** (Paladin, Ranger), use the appropriate table and set `HI: 1` in the abilities array.

---

## Malto-Specific Blocks to Remove or Replace

The following features exist specifically for Malto. Remove or replace anything that doesn't apply to your character.

### 1. Jack of All Trades
Malto's Bard feature. All skills without proficiency get `floor(PB/2)` added.

**Located in:** `calcSkill()` function.

```js
const calcSkill = (sk, ab) => {
  const mod = abilMod(ab[sk.abil]), pb = ab[6], joat = Math.floor(pb/2);
  if(sk.isSave) return mod + (sk.prof >= 1 ? pb : 0);
  if(sk.prof === 2) return mod + pb * 2;
  if(sk.prof === 1) return mod + pb;
  return mod + joat;  // ← this line is Jack of All Trades
};
```

**If your class doesn't have Jack of All Trades**, change the last line to:
```js
return mod;  // no proficiency = just the ability modifier
```

---

### 2. Bardic Features (FEATURES page — fields 1 & 2)
- **Bardic Inspiration** — Bard-specific
- **Encouraging Song** — Malto-specific Bard feature

Replace these with your class features (see [Common Adaptations by Class](#common-adaptations-by-class) below).

---

### 3. Werewolf / Beast Mode
Malto's lycanthropy is entirely custom. This includes:
- The Beast Mode toggle on the FEATURES page
- AC +1 / Speed +10 auto-adjustment on toggle
- Blood Lust red theme when HP < 50%
- Bite and Claw attack reference cards
- `beastMode` and `bonusHitDice` in state

**If your character is not a lycanthrope**, remove all of this. Ask Claude to strip Beast Mode and restore the normal grey theme.

---

### 4. Entertainer Background
The background listed in `CHAR` is Entertainer. This is display-only — just change the string value.

---

## Using Claude to Make Changes

You don't need to edit code manually. Claude can do it for you.

### How to Start a Session
1. Go to [claude.ai](https://claude.ai)
2. Start a new chat
3. Upload **both** `dnd_tracker.jsx` **and** `HANDOVER.md`
4. Paste this opening message:

> *"Please read both files carefully before doing anything. The .jsx is the source of truth for the current code. HANDOVER.md is the source of truth for architecture decisions and known gotchas. Once you've read them, tell me you're ready and wait for my instructions."*

5. Wait for Claude to confirm it has read both files
6. Then describe your changes in plain English

---

### Example Prompts

**Changing character info:**
> *"Change the character to a Level 5 Wood Elf Ranger named Syla Dawntracker, background Outlander, campaign The Thornwood. Update the default stats: STR 12, INT 10, DEX 18, WIS 14, CON 13, CHA 8, PB 3."*

**Replacing Bard features:**
> *"Remove Bardic Inspiration and Encouraging Song from the FEATURES page. Replace them with Hunter's Mark — a toggle that tracks whether it's active or not, just a simple on/off, no edit mode needed."*

**Removing lycanthropy:**
> *"Remove Beast Mode entirely. Remove the Blood Lust red theme. Remove beastMode and bonusHitDice from state. The app should always use the normal grey theme."*

**Adding a new resource:**
> *"Add a Ki Points tracker to the FEATURES page. Max Ki Points = character level. INC/DEC in edit mode. Restored on short rest and long rest."*

**Changing spell slots:**
> *"My character is a Paladin (half-caster). Replace the Bard spell slot table with the Paladin spell slot table from D&D 5e."*

**Removing spellcasting entirely:**
> *"My character is a Barbarian with no spellcasting. Remove pages 5 (SPELLCASTING) and 6 (SPELL SLOTS) entirely. Renumber the remaining pages."*

---

### Tips for Working with Claude

- **Always upload both files** at the start of a session. Claude has no memory between chats.
- **One change at a time** is safer than asking for everything at once — easier to catch mistakes.
- **Ask Claude to preview the artifact** after each change so you can test it before downloading.
- **If something breaks**, tell Claude exactly what's wrong. Copy the error message if there is one. Claude can fix it.
- **After all changes are done**, ask Claude to regenerate `index.html` for GitHub Pages deployment.

---

### Saving Your Progress Between Sessions

Claude doesn't remember previous chats. After each session, download the updated `dnd_tracker.jsx` and keep it somewhere safe (your cloud drive works great). Next session, upload the latest version.

If you're doing a lot of development, consider making a **HANDOFF note** like the one in this repo — a plain text document summarising what's been built, what decisions were made, and any gotchas. Paste it at the start of each new Claude session to bring it up to speed instantly.

---

## Common Adaptations by Class

Here are starting-point suggestions for replacing Malto's Bard features with features common to other classes. Use these as prompts for Claude.

### Barbarian
- Remove spellcasting pages entirely
- Remove Jack of All Trades from skill calculations
- Replace FEATURES page with:
  - **Rage** — uses per day (2 at L1, scaling with level), toggle for active/inactive
  - **Reckless Attack** — simple on/off toggle (no edit mode)

### Fighter
- Remove spellcasting pages entirely (unless Eldritch Knight)
- Replace FEATURES page with:
  - **Action Surge** — 1 use per short rest (2 at Level 17)
  - **Second Wind** — 1 use per short rest, recovers HP

### Monk
- Remove spellcasting pages entirely (unless Way of the Four Elements)
- Replace FEATURES page with:
  - **Ki Points** — max = character level, restored on short rest
  - **Stunning Strike** — reference note only (no tracking needed)

### Paladin
- Replace Bard spell slots with Paladin half-caster table
- Remove Jack of All Trades
- Replace FEATURES page with:
  - **Lay on Hands** — pool of HP (5 × level), decrements by amount spent
  - **Divine Smite** — reference note only
  - **Channel Divinity** — uses per short rest

### Ranger
- Replace Bard spell slots with Ranger half-caster table
- Remove Jack of All Trades
- Replace FEATURES page with:
  - **Hunter's Mark** — active/inactive toggle
  - **Favoured Enemy** — display text only

### Rogue
- Remove spellcasting pages entirely (unless Arcane Trickster)
- Remove Jack of All Trades (Rogues have Expertise and their own mechanic)
- Replace FEATURES page with:
  - **Sneak Attack** — reference display (damage dice by level)
  - **Cunning Action** — reference note only

### Sorcerer
- Replace Bard spell slots with Sorcerer full-caster table
- Replace FEATURES page with:
  - **Sorcery Points** — max = character level, INC/DEC
  - **Metamagic** — display active options as reference text

### Warlock
- Replace Bard spell slots with Warlock pact slot table (very different — all slots are the same level, fully restored on short rest)
- Replace FEATURES page with:
  - **Pact Magic Slots** — simpler tracking than standard slots
  - **Eldritch Invocations** — reference display

### Wizard
- Replace Bard spell slots with Wizard full-caster table
- Remove Jack of All Trades
- Replace FEATURES page with:
  - **Arcane Recovery** — 1 use per long rest
  - **Spell Mastery** (higher levels) — toggle

---

## Prompting Tips for AI Assistance

Getting good results from Claude (or any LLM) comes down to being specific and giving context. Here are patterns that work well.

### Be specific about what you want to keep
> ❌ *"Change the features page"*  
> ✅ *"Remove Bardic Inspiration and Encouraging Song. Keep Beast Mode exactly as it is. Add a Rage tracker in their place."*

### Reference the HANDOVER for technical terms
> ✅ *"As described in HANDOVER.md under fieldCount, update the fieldCount array for page 6 to reflect the new number of editable fields."*

### Describe behaviour, not implementation
You don't need to know how the code works. Describe what you want the app to do:
> ✅ *"When I toggle Rage on, I want the border of the tracker to turn orange. When HP drops below 25%, I want it to turn red. When Rage is toggled off, it goes back to normal."*

### Ask Claude to explain before it codes
> ✅ *"Before you write any code, tell me your plan for how you'll implement this."*  
This catches misunderstandings early and saves you from having to fix broken code.

### Test incrementally
After each change, ask Claude to show you the updated artifact. Catch problems early rather than stacking many changes and then debugging.

### If Claude makes a mistake
Tell it exactly what's wrong. The more specific you are, the faster it fixes it:
> ✅ *"The Rage toggle turns on but doesn't turn off when I tap it again. Also the counter goes below zero."*  
> ❌ *"It's broken"*

---

## Deploying Your Version

Once you're happy with your customised `dnd_tracker.jsx`, get a deployable build:

1. Upload your final `dnd_tracker.jsx` to Claude
2. Ask:  
   > *"Generate a standalone `index.html` from this JSX file, suitable for GitHub Pages. It should use React and Babel from CDN so no build step is needed."*
3. Download the `index.html` Claude produces
4. Upload it to your GitHub repository (replace the existing one)
5. GitHub Pages will automatically redeploy — your live URL updates within a minute

---

## Keeping Your Changes Safe

| What | Where to store it |
|------|------------------|
| `dnd_tracker.jsx` | GitHub repo + your cloud drive |
| `index.html` | GitHub repo |
| `HANDOVER.md` | GitHub repo |
| Your save data (exported JSON) | OneDrive / Google Drive / Mega |
| Any session notes | GitHub repo or cloud drive |

> 💡 Treat `dnd_tracker.jsx` like your character sheet — keep it backed up somewhere you won't lose it.

---

*Good luck, adventurer. May your rolls be nat 20s and your DM be merciful.* 🎲
