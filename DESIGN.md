# Sigil Tactics — Design

Mobile-web tactical card-battler. Two sides, three heroes each, on a small grid. Play proceeds in **rounds**; within a round, every living unit gets one turn in **Speed order**. Cards add sidekicks, spells, and battlefield effects. Win by eliminating all three enemy heroes.

Current build: **v0.6** (`index.html`).

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
| Vanguard | V | 7 | 3 | 1 | 1 | 2 | Frontline bruiser |
| Warden | W | 6 | 2 | 1 | 1 | 3 | Heals adjacent ally for +3 as an action |
| Archer | A | 5 | 2 | 1 | 2 | 3 | Ranged — pays in ATK for the reach |

Heroes never appear in the deck. They are gold-bordered on the board and tracked in mini HP bars above the grid.

## Sidekicks

Cheaper units played from hand. They count toward unit density but **not** toward win condition.

| Sidekick | Sym | HP | ATK | Move | Range | Speed | Cost | Notes |
|---|---|---|---|---|---|---|---|---|
| Skirmisher | S | 4 | 2 | 1 | 1 | 4 | 2 | Flanker — side ×2, rear ×2.5 |
| Scout | s | 4 | 2 | 1 | 2 | 4 | 2 | Same cost as Skirmisher — pays HP-and-flank for range |

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

- **Act normally** — move (up to `move`) and/or perform one action (attack any enemy in range; Warden may instead heal an adjacent ally).
- **Play one card** from hand — spends aether and the unit's entire activation. No move, no attack on this turn.
- **Pass** — end the turn with no effect (the END TURN button).

Movement updates facing toward the destination; attacks update facing toward the target. A Warden may attack instead of healing.

## Cards

Hand size cap 8; draw 1 at start of each round. Starting hand 3.

| Card | Sym | Cost | Kind | Effect |
|---|---|---|---|---|
| Skirmisher | S | 2 | unit | Deploy a Skirmisher into your half |
| Scout | s | 2 | unit | Deploy a Scout into your half |
| Potion | + | 2 | heal | Restore +4 HP to a wounded ally anywhere |
| Bolt | ϟ | 2 | dmg | 2 damage to any enemy on the board |
| High Ground | ▲ | 2 | terrain | Place high ground on an empty tile in your half |

Deck (12 cards): Skirmisher×3, Scout×2, Potion×3, Bolt×2, High Ground×2.

**Summoning sickness — none.** A sidekick summoned this round is inserted into initiative at its Speed and acts this round if its slot hasn't passed (i.e. if any pending unit has equal-or-lower Speed). A Skirmisher (Speed 5) summoned by your Vanguard (Speed 2) effectively jumps the queue and acts the same round.

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
3. Choose whichever scores higher. If acting normally, Warden heals first if adjacent ally is wounded, then attacks/advances.
4. Targets prefer heroes, low-HP enemies, and finishing blows.

---

## Open questions / next up

- **Hero variety**: only one hero set right now (V/W/A) shared by both sides. Drafting or asymmetric rosters would add identity.
- **Speed counter-play**: nothing currently modifies Speed mid-match. Haste / Slow effects, initiative-swap spells, or wait-actions (delay your slot) would deepen the layer. A card that steals or pre-spends the Initiative token would also be interesting.
- **Card play during enemy turns**: currently disallowed (instant/reaction cards would be a separate system).
- **Items vs spells**: currently mechanically identical (one-shot). If equippable items return, need card category, slot model, and removal rules.
- **More spells**: Smoke (skip a unit's next turn), Charge (+ATK this turn), Knockback, AoE — design space is wide open.
- **Line of sight / cover**: Archer is currently unblocked, which lets it sit in the back row safely.
- **Win-condition variants**: "kill any 2" as a faster mode; objective tiles as an alternative axis.
- **Mobile UX**: initiative ribbon may need horizontal scroll affordance; consider tap-and-hold for card details.
