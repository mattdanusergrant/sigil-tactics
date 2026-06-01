# Sigil Tactics — Design

Mobile-web tactical card-battler. Two sides, three heroes each, on a small grid. Play proceeds in **rounds**; within a round, every living unit gets one turn in **Speed order**. Cards add sidekicks, spells, and battlefield effects. Win by eliminating all three enemy heroes.

Current build: **v0.15** (`index.html`).

## Modes

- **Campaign** (PvE) — the primary mode. Start with Knight, Ranger, Mage. Beat each of 6 boss heroes to add them to your roster (Paladin → Druid → Scout → Berserker → Warlock → Assassin, locked in that order). Progress persists in `localStorage` under key `sigil-progress`.
- **Skirmish** — single match against the random-AI; full 9-hero pool is available to both sides.

Mode is chosen from the title screen at app open. The "New" button on the play HUD returns to the title (skirmish) or the mission list (campaign).

### Campaign mission table

| # | Title | Boss unlock | AI priority pool |
|---|---|---|---|
| 1 | The Sworn Oath | 🛡️ Paladin | Paladin, Knight, Mage, Ranger |
| 2 | The Wild Heart | 🌿 Druid | Druid, Mage, Ranger, Paladin |
| 3 | The Quick Shadow | 👁️ Scout | Scout, Ranger, Knight, Druid |
| 4 | The Wild Rage | 🪓 Berserker | Berserker, Knight, Mage, Paladin |
| 5 | The Pact of Shadows | 🩸 Warlock | Warlock, Mage, Druid, Ranger |
| 6 | The Silent Blade | 🗡️ Assassin | Assassin, Scout, Ranger, Berserker |

Missions are linear: mission *N* unlocks once mission *N-1* is complete. Replaying a completed mission re-unlocks the boss (no-op if already unlocked) and is otherwise harmless.

### Campaign draft behavior

The snake-draft phase runs as in skirmish, but with two filters:
- The player's pickable roster is restricted to **unlocked heroes**.
- The AI walks the mission's `aiPool` list in order, picking the first hero not already drafted. If every entry on the list is taken (e.g., the player snake-picked something the AI wanted), the AI falls back to any random unpicked active hero.

This guarantees the boss always shows up on the AI's side in their unlock mission (since the player can't yet pick them), and gives missions a thematic enemy composition without sacrificing the draft phase's tension.

---

## Board

- 5 columns × 4 rows. No neutral middle row — each side owns two rows.
- Player A (blue, human) owns rows 2–3; Player B (red, AI) owns rows 0–1.
- Deploy zone for cards: each side's own two rows.
- Tiles can hold one unit. Tiles can also carry a **High Ground** terrain marker (+1 ATK to the unit standing on it).

## Sides

Each side fields **3 heroes** drafted from a **9-hero active roster**. Three additional classes (Sentinel, Necromancer, Crusader) remain defined in code under a `hidden:true` flag and are excluded from the draft for now. HP must come from the polyhedral dice set `{4, 6, 8, 10, 12, 20}`.

### Active roster (9)

| Glyph | Hero | HP | ATK | Move | Range | Speed | Tags | Notes |
|---|---|---|---|---|---|---|---|---|
| 🛡️ | Paladin   | 10 | 1 | 1 | 1 | 1 | heal  | Tank with adjacent-ally heal |
| ⚔️ | Knight    | 10 | 1 | 1 | 1 | 2 | —     | Tanky melee |
| 🩸 | Warlock   | 6  | 1 | 1 | 3 | 3 | —     | Longest range in the roster |
| 🌿 | Druid     | 8  | 1 | 1 | 2 | 4 | heal  | Ranged healer |
| 🔮 | Mage      | 6  | 1 | 1 | 2 | 5 | heal  | Ranged caster, adjacent-ally heal |
| 🪓 | Berserker | 8  | 2 | 1 | 1 | 6 | —     | High-damage melee |
| 🏹 | Ranger    | 8  | 1 | 1 | 2 | 7 | —     | Fast ranged |
| 👁️ | Scout     | 4  | 1 | 2 | 2 | 8 | —     | Mobile recon, fragile |
| 🗡️ | Assassin  | 6  | 2 | 2 | 1 | 9 | flank | Fastest, ×2 side / ×2.5 rear |

Active-roster speeds form a contiguous `{1, 2, 3, 4, 5, 6, 7, 8, 9}` — all unique. Combined with the snake draft (no class duplicates within a match), initiative ties between heroes never occur. Hidden-roster speeds (Sentinel 1, Necromancer 4, Crusader 8) currently collide with active speeds; if any of those are un-hidden later, the active roster will need to be re-numbered accordingly.

### Hidden roster (3, not in draft)

| Glyph | Hero | HP | ATK | Move | Range | Speed | Tags |
|---|---|---|---|---|---|---|---|
| 🏰 | Sentinel    | 12 | 1 | 1 | 1 | 1 | —    |
| 💀 | Necromancer | 6  | 1 | 1 | 2 | 4 | —    |
| ✝️ | Crusader    | 10 | 1 | 1 | 1 | 8 | —    |

Heroes never appear in the deck. They are gold-bordered on the board.

## Draft & deploy

Each match begins with a draft phase, followed by a deploy phase, then play.

### Draft (1-2-2-1 snake)

| Pick | Player |
|---|---|
| 1 | P1 |
| 2 | P2 |
| 3 | P2 |
| 4 | P1 |
| 5 | P1 |
| 6 | P2 |

Each side ends with 3 heroes from the 12-hero roster. A pick removes the hero from the pool — no duplicates across sides.

### Deploy (alternating, face-down)

Placement mimics tabletop play: players alternate placing one unit at a time, each token going down **face-down**. You see *where* the opponent placed their unit but not *which* hero is in each tile — identities are concealed until play begins, at which point all units flip face-up.

**Placement order is the reverse of the draft** to keep things fair — whoever picked first in the draft places last in the deploy phase:

| Place | Player |
|---|---|
| 1 | P2 |
| 2 | P1 |
| 3 | P1 |
| 4 | P2 |
| 5 | P2 |
| 6 | P1 |

Each turn, the active player picks one of their unplaced heroes from a tray, then taps a tile in their half of the board (rows 0–1 for P2, rows 2–3 for P1). The AI uses a simple heuristic (ranged units to the back row, melee to the front, centered).

An **Auto** button lets the human burn through their remaining placements in a default formation; the deploy sequence then skips any consecutive P1 slots and the AI resumes.

### Decks

For v0.10 the deck is the fixed 30-card Knight/Ranger/Mage suite, used by both sides regardless of who they drafted. Suites for the other 9 heroes are not yet authored — a future task.

## Round structure

Play proceeds in numbered rounds.

1. **Start of round**
   - Both sides gain +1 max aether (cap 10) and refill to full.
   - Both sides draw 1 card (hand cap 8).
   - All units lose their `hasMoved` / `hasActed` flags.
2. **Initiative pass**
   - Repeatedly pick the next unit to activate per the Initiative System (below) until no unacted units remain.
3. **Next round** — back to step 1.

The initiative ribbon above the board shows the current Initiative token holder (◆) plus current and upcoming activations (acted units are faded).

## Initiative System

Activation order is **strictly by Speed**, ties broken by the Initiative token.

**Game start.** The unit with the highest Speed acts first. If multiple units are tied for highest Speed, randomly pick which one. The player who is **not** taking that first action gains the **Initiative token**.

**Speed ties during play.** Whenever multiple pending units share the same top Speed and they belong to different sides, the units belonging to the token-holder go first (in id order), then the opponent's tied units. The token then transfers to the opponent.

**Within-side ties.** Units of the same side at the same Speed activate in id order (older units / heroes first).

**Tie-bracket semantics.** A "tie" is resolved per *bracket* (a single Speed value with both sides represented). All token-holder units in the bracket act, then all opponent units, and the token transfers once at the end of the bracket.

**Persistence.** Token state persists between rounds. Each cross-side tie consumed during a round flips it.

**Summons mid-round.** A summoned sidekick is inserted into the remaining initiative at its Speed using the live token state. If it joins an already-active cross-tie bracket, its side's order within the bracket follows the token-holder rule.

## A unit's turn

When a unit is the active one, its owner does **one** of:

- **Act normally** — move (up to `move`) and/or perform one action (attack any enemy in range; Mage may instead heal an adjacent ally).
- **Play one card** from hand — consumes the unit's entire turn. No move, no attack on this turn. There is no resource cost; the trade-off is opportunity cost.
- **Pass** — end the turn with no effect (the END TURN button).

Movement updates facing toward the destination; attacks update facing toward the target. A Mage may attack instead of healing.

## Cards

The card pool spans **9 hero suites × 10 cards = 90 cards total**. Each hero contributes **5 equipment + 5 actions** themed to their identity. After the draft, each player's **match deck is 30 cards** — the union of their 3 drafted heroes' suites — then shuffled. Both players draw from their own deck independently.

### Card economy

- **Starting hand:** 2 cards.
- **Max hand size:** none.
- **Draw timing:** at the end of every turn — yours or the AI's — the active unit's owner draws 1 card from their deck.
- **No aether / no costs.** Cards have no resource gate. A card's price is paid in **opportunity cost**: playing a card *consumes the active unit's entire turn* (no move, no attack — the card is what that turn does).
- **Design target:** a card should be tuned so that, in the moment, it is the *best play* the active unit could make this turn. The intended loop is: play 1 of your 2 cards → draw 1 → consider the new card next turn → repeat. If most cards are weaker than a normal attack, players will just attack and the system breaks down.

### Card-value baselines

A "normal turn" from a unit moves up to its `move` and deals roughly **1–2 damage** to one enemy (with the relevant arc multiplier on top). Cards must clear that bar. Working baselines:

| Effect shape | Threshold value to feel "worth a turn" |
|---|---|
| Single-target damage | ≥ 3 |
| Single-target heal | ≥ 4 |
| Single-target shield | ≥ 4 |
| Single-target buff (this round) | +2 ATK on a unit that will attack this round |
| Single-target charge (move) | +3 move, or +2 move that enables a kill/flank |
| Area / all-allies effect | ≥ ~2 per affected unit (so ≥ 6 total expected value) |
| Multi-target damage | total damage ≥ 4 across all hit enemies |

Below those thresholds the card is dead weight; well above them it warps the game.

**Categories:**
- **Equipment** — one each of: Weapon, Armor, Helmet, Boots, Artifact.
- **Action** — moves, attacks, and effects performed by a hero.
- **Spell** — a *subtype of action*. Mechanically identical to a non-spell action; distinguished by flavor (and shown with a purple stripe and `SPL` tag in-game).

Every card is a **one-shot consumable** with no resource cost — playing it consumes the active unit's turn. Mechanically a card resolves as one of these **kinds**:

| Kind | Target | Effect |
|---|---|---|
| `dmg` | enemy unit | Deal N damage (defender's shield absorbs first) |
| `dmg-adj` | enemy unit | Deal N to target + S splash to enemies orthogonally adjacent to it |
| `heal` | friendly unit | Restore N HP (caps at maxHp) |
| `heal-all` | all friendly units | Restore N HP to each |
| `shield` | friendly unit | Add N shield (absorbs incoming damage before HP) |
| `shield-all` | all friendly units | Add N shield to each |
| `buff-atk` | friendly unit | +N ATK for the rest of this round |
| `buff-atk-all` | all friendly units | +N ATK each for the rest of this round |
| `charge` | friendly unit | +N move on the buffed unit's next turn |

Shield persists across rounds until consumed. ATK buffs clear at round start. Move buffs clear at end of the buffed unit's turn.

### Knight suite (10) — 5 equip, 5 actions (1 spell)

Tank / team-shield identity. Big single hit, layered defense, group heals.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Greatsword | dmg 4 | 4 dmg to enemy |
| 2 | Armor | Plate Mail | shield 5 | +5 shield to ally |
| 3 | Helmet | Iron Helm | shield 3 | +3 shield to ally |
| 4 | Boots | Steel Greaves | charge 3 | Ally +3 move this turn |
| 5 | Artifact | War Banner | shield-all 2 | +2 shield to all allies |
| 6 | Action | Cleave | dmg-adj 3 (+1) | 3 dmg to enemy + 1 splash to adjacent enemies |
| 7 | Action | Bulwark | heal 4 | Heal ally +4 |
| 8 | Action | Rally | heal-all 2 | Heal all allies +2 |
| 9 | Action | Shield Wall | shield-all 3 | +3 shield to all allies |
| 10 | Action · Spell | Battle Cry | buff-atk-all 3 | All allies +3 ATK this round |

### Ranger suite (10) — 5 equip, 5 actions (2 spells)

Mobile sniper / combat tricks. Strong single-target damage, lots of movement.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Longbow | dmg 4 | 4 dmg to enemy |
| 2 | Armor | Leather Cloak | shield 3 | +3 shield to ally |
| 3 | Helmet | Hawk Hood | charge 2 | Ally +2 move this turn |
| 4 | Boots | Swift Boots | charge 4 | Ally +4 move this turn |
| 5 | Artifact | Hunter's Mark | buff-atk 3 | Ally +3 ATK this round |
| 6 | Action | Quick Shot | dmg 3 | 3 dmg to enemy |
| 7 | Action | Volley | dmg-adj 2 (+1) | 2 dmg to enemy + 1 splash to adjacent enemies |
| 8 | Action | Reposition | charge 3 | Ally +3 move this turn |
| 9 | Action · Spell | Piercing Arrow | dmg 5 | 5 dmg to enemy (biggest single hit in the deck) |
| 10 | Action · Spell | Eagle's Eye | buff-atk-all 2 | All allies +2 ATK this round |

### Mage suite (10) — 5 equip, 5 actions (all spells)

Caster / healer. Wide selection of heals, plus team shield/buff.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Staff | dmg 3 | 3 dmg to enemy |
| 2 | Armor | Robe of Mending | heal 3 | Heal ally +3 |
| 3 | Helmet | Crown of Light | shield 4 | +4 shield to ally |
| 4 | Boots | Cleric Sandals | charge 2 | Ally +2 move this turn |
| 5 | Artifact | Sacred Chalice | heal 8 | Heal ally +8 (caps at maxHp — effectively top-up for any hero) |
| 6 | Action · Spell | Mend | heal 5 | Heal ally +5 |
| 7 | Action · Spell | Heal | heal-all 3 | Heal all allies +3 |
| 8 | Action · Spell | Aegis | shield-all 2 | +2 shield to all allies |
| 9 | Action · Spell | Bless | buff-atk 3 | Ally +3 ATK this round |
| 10 | Action · Spell | Smite | dmg 4 | 4 dmg to enemy |

### Paladin suite (10) — 5 equip, 5 actions (3 spells)

Holy tank / heavy heals + heavy shields.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Warhammer | dmg 4 | 4 dmg to enemy |
| 2 | Armor | Holy Plate | shield 6 | +6 shield to ally (biggest single shield in pool) |
| 3 | Helmet | Helm of Faith | shield-all 2 | +2 shield to all allies |
| 4 | Boots | Sabatons | charge 2 | Ally +2 move this turn |
| 5 | Artifact | Holy Symbol | heal-all 3 | Heal all allies +3 |
| 6 | Action | Lay on Hands | heal 7 | Heal ally +7 |
| 7 | Action | Smite Evil | dmg 4 | 4 dmg to enemy |
| 8 | Action · Spell | Divine Shield | shield-all 3 | +3 shield to all allies |
| 9 | Action · Spell | Consecration | dmg-adj 3 (+1) | 3 dmg + 1 splash to adjacent enemies |
| 10 | Action · Spell | Sanctuary | buff-atk-all 2 | All allies +2 ATK this round |

### Warlock suite (10) — 5 equip, 5 actions (4 spells)

Dark caster — heavy single-target damage, AoE curses, life drain.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Athame | dmg 3 | 3 dmg to enemy |
| 2 | Armor | Shadow Cloak | shield 3 | +3 shield to ally |
| 3 | Helmet | Hood of Shadows | charge 2 | Ally +2 move this turn |
| 4 | Boots | Boots of Misdirection | charge 3 | Ally +3 move this turn |
| 5 | Artifact | Cursed Skull | buff-atk 3 | Ally +3 ATK this round |
| 6 | Action | Hex | dmg 4 | 4 dmg to enemy |
| 7 | Action · Spell | Eldritch Blast | dmg 5 | 5 dmg to enemy |
| 8 | Action · Spell | Pulse of Decay | dmg-adj 3 (+1) | 3 dmg + 1 splash to adjacent enemies |
| 9 | Action · Spell | Drain Soul | heal 4 | Heal ally +4 (life-drain flavor) |
| 10 | Action · Spell | Curse of Weakness | buff-atk-all 2 | All allies +2 ATK this round |

### Druid suite (10) — 5 equip, 5 actions (4 spells)

Nature magic — effects that spread (heal-all + dmg-adj heavy).

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Sickle | dmg-adj 2 (+1) | 2 dmg + 1 splash to adjacent enemies |
| 2 | Armor | Bark Skin | shield 4 | +4 shield to ally |
| 3 | Helmet | Antlered Crown | buff-atk-all 2 | All allies +2 ATK this round |
| 4 | Boots | Mossy Wraps | charge 3 | Ally +3 move this turn |
| 5 | Artifact | Living Wood | heal-all 4 | Heal all allies +4 (biggest group heal) |
| 6 | Action | Thorns | dmg 4 | 4 dmg to enemy |
| 7 | Action · Spell | Regrowth | heal 5 | Heal ally +5 |
| 8 | Action · Spell | Entangle | dmg-adj 3 (+1) | 3 dmg + 1 splash to adjacent enemies |
| 9 | Action · Spell | Lightning Bolt | dmg 5 | 5 dmg to enemy |
| 10 | Action · Spell | Sunbeam | heal-all 3 | Heal all allies +3 |

### Berserker suite (10) — 5 equip, 5 actions (1 spell)

Rage — overwhelming damage, weak defense.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Battle Axe | dmg 5 | 5 dmg to enemy |
| 2 | Armor | Leather Wraps | shield 2 | +2 shield to ally (weakest armor in pool) |
| 3 | Helmet | Horned Helm | buff-atk 3 | Ally +3 ATK this round |
| 4 | Boots | War Boots | charge 3 | Ally +3 move this turn |
| 5 | Artifact | Bloodthirster | dmg 6 | 6 dmg to enemy (largest dmg from equipment) |
| 6 | Action | Frenzy | buff-atk-all 3 | All allies +3 ATK this round |
| 7 | Action | Reckless Charge | charge 4 | Ally +4 move this turn |
| 8 | Action | Whirlwind | dmg-adj 3 (+1) | 3 dmg + 1 splash to adjacent enemies |
| 9 | Action | Rend | dmg-adj 4 (+1) | 4 dmg + 1 splash to adjacent enemies |
| 10 | Action · Spell | Battle Trance | buff-atk 4 | Ally +4 ATK this round |

### Scout suite (10) — 5 equip, 5 actions (1 spell)

Mobility — hit-and-run, evasion, recon.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Shortbow | dmg 3 | 3 dmg to enemy |
| 2 | Armor | Camo Cloak | shield 3 | +3 shield to ally |
| 3 | Helmet | Spyglass | buff-atk 3 | Ally +3 ATK this round |
| 4 | Boots | Cat's Boots | charge 3 | Ally +3 move this turn |
| 5 | Artifact | Trap | dmg-adj 3 (+1) | 3 dmg + 1 splash to adjacent enemies |
| 6 | Action | Snipe | dmg 4 | 4 dmg to enemy |
| 7 | Action | Dash | charge 5 | Ally +5 move this turn (biggest movement card) |
| 8 | Action | Vanish | shield 4 | +4 shield to ally |
| 9 | Action | Mark Target | buff-atk-all 2 | All allies +2 ATK this round |
| 10 | Action · Spell | Shadow Step | charge 4 | Ally +4 move this turn |

### Assassin suite (10) — 5 equip, 5 actions (2 spells)

Lethal precision — spike damage, single-target kills.

| # | Slot/Type | Name | Kind | Effect |
|---|---|---|---|---|
| 1 | Weapon | Dagger | dmg 4 | 4 dmg to enemy |
| 2 | Armor | Black Leathers | shield 2 | +2 shield to ally |
| 3 | Helmet | Mask | buff-atk 3 | Ally +3 ATK this round |
| 4 | Boots | Shadow Boots | charge 4 | Ally +4 move this turn |
| 5 | Artifact | Death Mark | dmg 5 | 5 dmg to enemy |
| 6 | Action | Backstab | dmg 5 | 5 dmg to enemy |
| 7 | Action | Garrote | dmg-adj 2 (+2) | 2 dmg + 2 splash to adjacent enemies |
| 8 | Action | Smoke Bomb | shield-all 2 | +2 shield to all allies |
| 9 | Action · Spell | Shadow Strike | dmg 7 | 7 dmg to enemy (biggest single hit in the pool) |
| 10 | Action · Spell | Veil | charge 5 | Ally +5 move this turn |

## HP & dice tracking

**Rule:** every unit's max HP equals the face count of a classic polyhedral die — one of `{4, 6, 8, 10, 12, 20}`. New unit classes must pick from this set.

For tabletop play, place the matching die next to each minifig showing its current HP; rotate it down as damage is taken; the unit dies when the die would go below 1.

| Die | Heroes |
|---|---|
| d4  | Scout |
| d6  | Mage, Assassin, Warlock, Necromancer |
| d8  | Ranger, Berserker, Druid |
| d10 | Knight, Paladin, Crusader |
| d12 | Sentinel |
| d20 | *(reserved for future heavyweights)* |

Healing rotates the die back up (capped at max).

## Combat

- **Facing**: each unit has a cardinal facing. Moving or attacking re-faces toward the destination/target.
- **Arc multipliers**: front ×1, side ×1.5, rear ×2. Assassin (flank tag): side ×2, rear ×2.5.
- **High ground**: attacker on `terrain="high"` gets +1 ATK before the arc multiplier.
- **Damage**: `round((ATK + highground) × arc)`. Shield absorbs before HP.
- **Range**: Manhattan distance. No line-of-sight blocking currently.

## Win condition

A side loses when all three of its heroes are at 0 HP.

## AI sketch

For each AI-owned unit on its turn:

1. Score the best possible card play *with* this unit's turn (heal a wounded hero, bolt a hero or finishable target, deploy a sidekick, place terrain).
2. Score the unit's best normal action (best attack arc-adjusted damage; small baseline for "advance toward enemy").
3. Choose whichever scores higher. If acting normally, Mage heals first if adjacent ally is wounded, then attacks/advances.
4. Targets prefer heroes, low-HP enemies, and finishing blows.

---

## Open questions / next up

- **Hero variety**: only one hero set right now (V/W/A) shared by both sides. Drafting or asymmetric rosters would add identity.
- **Speed counter-play**: nothing currently modifies Speed mid-match. Haste / Slow effects, initiative-swap spells, or wait-actions (delay your slot) would deepen the layer. A card that steals or pre-spends the Initiative token would also be interesting.
- **Card play during enemy turns**: currently disallowed (instant/reaction cards would be a separate system).
- **Items vs spells**: currently mechanically identical (one-shot). If equippable items return, need card category, slot model, and removal rules.
- **More spells**: Smoke (skip a unit's next turn), Charge (+ATK this turn), Knockback, AoE — design space is wide open.
- **Line of sight / cover**: Ranger is currently unblocked, which lets it sit in the back row safely.
- **Win-condition variants**: "kill any 2" as a faster mode; objective tiles as an alternative axis.
- **Mobile UX**: initiative ribbon may need horizontal scroll affordance; consider tap-and-hold for card details.
