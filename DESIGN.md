# Sigil Tactics — Design

Mobile-web tactical card-battler. Two sides, three heroes each, on a small grid. Cards add sidekicks, spells, and battlefield effects. Win by eliminating all three enemy heroes.

Current build: **v0.2** (`index.html`).

---

## Board

- 5 columns × 4 rows. No neutral middle row — each side owns two rows.
- Player A (blue, human) owns rows 2–3; Player B (red, AI) owns rows 0–1.
- Deploy zone for cards: each side's own two rows.
- Tiles can hold one unit. Tiles can also carry a **High Ground** terrain marker (+1 ATK to the unit standing on it).

## Sides

Each side starts with **3 distinct heroes** on the board, placed in their back row at columns 1, 2, 3:

| Hero | Sym | HP | ATK | Move | Range | Notes |
|---|---|---|---|---|---|---|
| Vanguard | V | 7 | 3 | 2 | 1 | Frontline melee |
| Warden | W | 6 | 2 | 2 | 1 | Heals adjacent ally for +3 as an action |
| Archer | A | 5 | 3 | 2 | 2 | Ranged |

Heroes never appear in the deck. They are gold-bordered on the board and tracked in mini HP bars above the grid.

## Sidekicks

Cheaper units played from hand. They count toward unit density but **not** toward win condition.

| Sidekick | Sym | HP | ATK | Move | Range | Cost | Notes |
|---|---|---|---|---|---|---|---|
| Skirmisher | S | 4 | 2 | 3 | 1 | 2 | Flanker — side ×2, rear ×2.5 |
| Scout | s | 3 | 2 | 3 | 2 | 3 | Cheap ranged |

## Cards

Hand size cap 8; draw 1 at start of turn. Starting hand 3.

| Card | Sym | Cost | Kind | Effect |
|---|---|---|---|---|
| Skirmisher | S | 2 | unit | Deploy a Skirmisher into your half |
| Scout | s | 3 | unit | Deploy a Scout into your half |
| Potion | + | 2 | heal | Restore +4 HP to a wounded ally anywhere |
| Bolt | ϟ | 2 | dmg | 2 damage to any enemy on the board |
| High Ground | ▲ | 1 | terrain | Place high ground on an empty tile in your half |

Deck (12 cards): Skirmisher×3, Scout×2, Potion×3, Bolt×2, High Ground×2.

## Combat

- **Facing**: each unit has a cardinal facing. Moving or attacking re-faces toward the destination/target.
- **Arc multipliers**: front ×1, side ×1.5, rear ×2. Skirmisher: side ×2, rear ×2.5.
- **High ground**: attacker on `terrain="high"` gets +1 ATK before the arc multiplier.
- **Damage**: `round((ATK + highground) × arc)`.
- **Range**: Manhattan distance. No line-of-sight blocking currently.

## Action economy

- Each unit may move once and act once per turn (order flexible; movement updates facing).
- Acting = attack (any unit) or heal (Warden only, adjacent ally).
- Spells/items consume aether and an entire card; they are not bound to unit actions.

## Aether

- Both sides gain +1 max aether per turn (cap 10), refilling to full each turn. Standard ramp curve.

## Win condition

A side loses when all three of its heroes are at 0 HP. Sidekicks dying does not end the game.

## AI sketch

- Plays cards greedily, scoring Potion on wounded heroes, Bolt on low-HP or hero targets.
- Sidekicks deploy onto its own back rows, center-out.
- Units prioritize attacking heroes; move toward nearest enemy when no target in range. Wardens heal adjacent low-HP allies before attacking.

---

## Open questions / next up

- **Hero variety**: only one hero set right now (V/W/A) shared by both sides. Drafting or asymmetric rosters would add identity.
- **Items vs spells**: currently mechanically identical (one-shot). If equippable items return, need card category, slot model, and removal rules.
- **More spells**: Smoke (skip), Charge (+ATK this turn), Knockback, AoE — design space is wide open.
- **Line of sight / cover**: Archer is currently unblocked, which lets it sit in the back row safely.
- **Win-condition variants**: "kill any 2" as a faster mode; objective tiles as an alternative axis.
- **Mobile UX**: confirm the 3-hero ribbon is legible at iPhone widths; consider tap-and-hold for card details.
