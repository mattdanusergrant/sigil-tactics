# Sigil Tactics — Design

Mobile-web tactical card-battler. Two sides, three heroes each, on a small grid. Play proceeds in **rounds**; within a round, every living unit gets one turn in **Speed order**. Cards add sidekicks, spells, and battlefield effects. Win by eliminating all three enemy heroes.

Current build: **v0.9** (`index.html`).

---

## Board

- 5 columns × 4 rows. No neutral middle row — each side owns two rows.
- Player A (blue, human) owns rows 2–3; Player B (red, AI) owns rows 0–1.
- Deploy zone for cards: each side's own two rows.
- Tiles can hold one unit. Tiles can also carry a **High Ground** terrain marker (+1 ATK to the unit standing on it).

## Sides

Each side starts with **3 distinct heroes** on the board, placed in their back row at columns 1, 2, 3:

| Hero | Sym | HP | ATK | Move | Range | Speed | Notes |
|---|---|---|---|---|---|---|---|
| Knight | K | 10 | 1 | 1 | 1 | 3 | Tankiest hero, slowest in the bunch |
| Mage   | M | 6  | 1 | 1 | 2 | 4 | Heals adjacent ally for +3 as an action; ranged |
| Ranger | R | 8  | 1 | 1 | 2 | 5 | Fastest, range 2 |

Heroes never appear in the deck. They are gold-bordered on the board and tracked in mini HP bars above the grid.

## Sidekicks

Currently **not in the deck.** The Skirmisher and Scout classes remain defined in the engine for future use but no card summons them in v0.9.

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
- **Play one card** from hand — spends aether and the unit's entire activation. No move, no attack on this turn.
- **Pass** — end the turn with no effect (the END TURN button).

Movement updates facing toward the destination; attacks update facing toward the target. A Mage may attack instead of healing.

## Cards

Each side runs a **30-card deck** built from three 10-card hero suites — Knight, Ranger, Mage. Each hero contributes **5 equipment + 5 actions**. Both sides currently use identical decks. Hand cap 8; draw 1 at start of each round; starting hand 3.

**Categories:**
- **Equipment** — one each of: Weapon, Armor, Helmet, Boots, Artifact.
- **Action** — moves, attacks, and effects performed by a hero.
- **Spell** — a *subtype of action*. Mechanically identical to a non-spell action; distinguished by flavor (and shown with a purple stripe and `SPL` tag in-game).

Every card is a **one-shot consumable.** Mechanically a card resolves as one of these **kinds**:

| Kind | Target | Effect |
|---|---|---|
| `dmg` | enemy unit | Deal N damage (subject to defender's shield) |
| `heal` | friendly unit | Restore N HP (caps at maxHp) |
| `shield` | friendly unit | Add N shield (absorbs incoming damage before HP) |
| `shield-all` | all friendly units | Add N shield to each |
| `buff-atk` | friendly unit | +N ATK for the rest of this round |
| `buff-atk-all` | all friendly units | +N ATK each for the rest of this round |
| `charge` | friendly unit | +N move on its next turn (this round) |

Shield persists across rounds until consumed. ATK buffs and move buffs both clear (atk-buff at round start, move-buff at end of the buffed unit's turn).

### Knight suite (10) — 5 equip, 5 actions (1 spell)

| # | Slot/Type | Name | Cost | Effect |
|---|---|---|---|---|
| 1 | Weapon | Greatsword | 2 | 2 dmg to enemy |
| 2 | Armor | Plate Mail | 2 | +2 shield to ally |
| 3 | Helmet | Iron Helm | 1 | +1 shield to ally |
| 4 | Boots | Steel Greaves | 1 | Ally +1 move this turn |
| 5 | Artifact | War Banner | 3 | +1 shield to all allies |
| 6 | Action | Charge | 1 | Ally +2 move this turn |
| 7 | Action | Bulwark | 1 | Heal ally +2 |
| 8 | Action | Rally | 3 | All allies +1 ATK this round |
| 9 | Action | Sundering Blow | 3 | 3 dmg to enemy |
| 10 | Action · Spell | Battle Cry | 2 | Ally +2 ATK this round |

### Ranger suite (10) — 5 equip, 5 actions (2 spells)

| # | Slot/Type | Name | Cost | Effect |
|---|---|---|---|---|
| 1 | Weapon | Longbow | 2 | 2 dmg to enemy |
| 2 | Armor | Leather Cloak | 1 | +1 shield to ally |
| 3 | Helmet | Hawk Hood | 1 | Ally +1 move this turn |
| 4 | Boots | Swift Boots | 2 | Ally +2 move this turn |
| 5 | Artifact | Hunter's Mark | 2 | 2 dmg to enemy |
| 6 | Action | Quick Shot | 1 | 1 dmg to enemy |
| 7 | Action | Volley | 3 | 2 dmg to enemy |
| 8 | Action | Reposition | 1 | Ally +1 move this turn |
| 9 | Action · Spell | Piercing Arrow | 3 | 3 dmg to enemy |
| 10 | Action · Spell | Eagle's Eye | 1 | Ally +1 ATK this round |

### Mage suite (10) — 5 equip, 5 actions (all spells)

| # | Slot/Type | Name | Cost | Effect |
|---|---|---|---|---|
| 1 | Weapon | Staff | 1 | 1 dmg to enemy |
| 2 | Armor | Robe of Mending | 2 | Heal ally +2 |
| 3 | Helmet | Crown of Light | 2 | +2 shield to ally |
| 4 | Boots | Cleric Sandals | 1 | Heal ally +1 |
| 5 | Artifact | Sacred Chalice | 4 | Heal ally +6 (caps at maxHp) |
| 6 | Action · Spell | Mend | 2 | Heal ally +3 |
| 7 | Action · Spell | Restore | 4 | Heal ally +5 |
| 8 | Action · Spell | Shield Wall | 3 | +1 shield to all allies |
| 9 | Action · Spell | Bless | 2 | Ally +1 ATK this round |
| 10 | Action · Spell | Smite | 2 | 2 dmg to enemy |

## HP & dice tracking

**Rule:** every unit's max HP equals the face count of a classic polyhedral die — one of `{4, 6, 8, 10, 12, 20}`. New unit classes must pick from this set.

For tabletop play, place the matching die next to each minifig showing its current HP; rotate it down as damage is taken; the unit dies when the die would go below 1.

| Unit | Max HP | Die |
|---|---|---|
| Knight | 10 | d10 |
| Ranger | 8 | d8 |
| Mage | 6 | d6 |
| Skirmisher | 4 | d4 |
| Scout | 4 | d4 |

Healing rotates the die back up (capped at max).

## Combat

- **Facing**: each unit has a cardinal facing. Moving or attacking re-faces toward the destination/target.
- **Arc multipliers**: front ×1, side ×1.5, rear ×2. Skirmisher: side ×2, rear ×2.5.
- **High ground**: attacker on `terrain="high"` gets +1 ATK before the arc multiplier.
- **Damage**: `round((ATK + highground) × arc)`.
- **Range**: Manhattan distance. No line-of-sight blocking currently.

## Aether

- Both sides gain +1 max aether per round (cap 10), refilling to full each round. Standard ramp curve, now keyed to rounds instead of side-turns.

## Win condition

A side loses when all three of its heroes are at 0 HP. Sidekicks dying does not end the game.

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
