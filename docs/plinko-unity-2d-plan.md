# Unity 2D Plinko — Game Plan

A physics-driven drop game that recreates the feel of physical Plinko: release a disc, watch it bounce through a peg field, and land in a scored slot.

---

## 1. Design goals

| Goal | Detail |
|------|--------|
| Feel | Soft, chaotic bounces — readable, not purely random |
| Clarity | Player always knows where they dropped, where they landed, and what they won |
| Scope | Vertical slice first: one board, one disc type, score slots, restart |
| Platform | Unity 2D (URP or Built-in), portrait or square playfield |

**Non-goals for v1:** multiplayer, gambling/real money, deep progression, procedural boards.

---

## 2. Core loop

```
Select drop lane (optional) → Release disc → Bounce through pegs → Land in slot → Award score → Next disc / round end
```

1. **Aim / release** — Player taps/clicks a top lane (or a continuous top rail). Disc spawns with near-zero horizontal velocity so small aim differences matter.
2. **Fall** — Rigidbody2D + CircleCollider2D interact with static pegs and walls.
3. **Resolve** — Trigger in a bottom slot awards points, FX, and optional multipliers.
4. **Continue** — Spend a disc from a limited pool, or endless casual mode.

---

## 3. Board layout

```
┌─────────────────────────────┐
│   Drop zones / aim rail     │
│                             │
│     ●   ●   ●   ●   ●       │  ← peg rows (staggered)
│       ●   ●   ●   ●         │
│     ●   ●   ●   ●   ●       │
│       ●   ●   ●   ●         │
│     …                       │
│                             │
│  [10][50][100][50][10]      │  ← score slots (skewed high in center)
└─────────────────────────────┘
```

**Pegs**
- Staggered hexagonal / brick layout so paths diverge.
- Static colliders; optional slight bounce via Physics Material 2D.
- Count/spacing tuned so a disc usually contacts several pegs before the bottom.

**Walls**
- Left/right rails keep the disc on-board.
- Optional soft “funnel” near the top so drops don’t clip corners.

**Slots**
- Adjacent BoxCollider2D triggers (or one composite with slot IDs).
- Point values: low on edges, high in center (classic Plinko skew), e.g. `10 · 25 · 50 · 100 · 50 · 25 · 10`.

---

## 4. Physics recipe (Unity)

| Piece | Recommendation |
|-------|----------------|
| Disc | `Rigidbody2D` (Dynamic), `CircleCollider2D`, gravity scale ~1–1.5 |
| Pegs | Static colliders; small radius circles |
| Material | Physics Material 2D: friction ~0.1–0.3, bounciness ~0.4–0.7 |
| Simulation | Fixed timestep; avoid scaling Time.timeScale for “slow-mo” until base feel is solid |
| Determinism | Not required for casual play; seed RNG only if you add fake “luck” later |

**Feel tuning knobs (expose in a ScriptableObject)**
- Gravity scale, disc mass, peg bounciness, peg spacing, drop height, max angular velocity.

**Anti-frustration**
- Cap extreme spin so the disc doesn’t look broken.
- Minimum fall time / “stuck” detection: if velocity stays near zero mid-board, nudge slightly or respawn.

---

## 5. Systems architecture

```
GameManager          Round state, disc budget, win/lose / continue
BoardGenerator       Builds peg grid + slots from config (or baked scene)
DropController       Input → spawn disc at aimed X
Disc                 Rigidbody wrapper; reports land / stuck
Slot                 Trigger; holds score value + FX hook
ScoreService         Running total, best score, combo (optional)
UIController         Disc count, score, result banner, replay
Audio / VFX          Peg hit, slot land, UI clicks
```

Prefer **data-driven board config** (ScriptableObject) so you can iterate layouts without rebuilding scenes.

---

## 6. Player input

| Platform | Aim | Drop |
|----------|-----|------|
| Desktop | Mouse X along top rail | Click / Space |
| Mobile | Drag finger on top rail | Release / Drop button |
| Gamepad | Left stick X | A / South |

One disc in flight at a time in v1 (simplifies scoring and camera).

---

## 7. Scoring & modes

**Casual (v1)**
- Fixed disc count (e.g. 5–10).
- Sum slot values → final score → high score (PlayerPrefs / local save).

**Optional later**
- Risk lanes (side slots = jackpot but rare).
- Multipliers on special pegs.
- Daily challenge board seed.
- Endless: earn discs from score thresholds.

Avoid framing as gambling; keep rewards cosmetic or score-only.

---

## 8. Presentation

**Visual**
- Clear silhouette: disc vs pegs vs slots.
- Soft background depth (gradient or subtle board texture) — not flat gray.
- Slot labels always readable after land.
- Impact squash/spark on peg hit; celebrate slot land (particles + brief camera punch).

**Audio**
- Peg tick (pitched variation by velocity).
- Slot chime scaled by value.
- UI confirm / round end sting.

**Motion (2–3 intentional)**
1. Disc drop anticipation (short hold / ease before gravity).
2. Peg hit pulses.
3. Slot fill / score count-up.

---

## 9. Scene / project structure

```
Assets/
  Scenes/          Boot, MainBoard
  Scripts/
    Core/          GameManager, ScoreService
    Board/         BoardGenerator, Peg, Slot
    Disc/          Disc, DropController
    UI/            HUD, Results
    Data/          BoardConfig (SO), GameSettings (SO)
  Art/             Sprites, materials
  Audio/
  Prefabs/         Disc, Peg, Slot, VFX
```

**Unity packages:** 2D Sprite, 2D Physics (built-in), TextMeshPro, Input System (recommended).

---

## 10. Implementation phases

### Phase A — Playable physics board
- Empty scene with walls, staggered pegs, score slots.
- Spawn disc from fixed top positions; land → log score.
- Tune Physics Material until drops feel “Plinko.”

### Phase B — Game loop + UI
- Disc budget, running score, round end, replay.
- Aim rail + drop input.
- Basic SFX stubs.

### Phase C — Polish
- VFX, camera settle, score popups, haptics (mobile).
- Stuck-disc recovery.
- Local high score.

### Phase D — Content (optional)
- Alternate boards via BoardConfig.
- Themes / skins.
- Challenge mode with seeded boards.

---

## 11. Acceptance criteria (vertical slice)

- [ ] Player can aim and drop a disc.
- [ ] Disc reliably interacts with peg field and stays on board.
- [ ] Landing in a slot awards the correct points exactly once.
- [ ] Round ends after N discs; score and replay work.
- [ ] 60 FPS on mid-tier mobile / desktop target with one disc + ~100 pegs.

---

## 12. Risks & mitigations

| Risk | Mitigation |
|------|------------|
| Feels too random / unfair | Tighter peg spacing, lower bounciness, wider high-value center |
| Disc tunnels through pegs | Continuous collision detection on disc; reasonable Fixed Timestep |
| Boring after 2 drops | Peg hit feedback + rising stakes (last disc, near-miss FX) |
| Over-scoped content | Ship one board before themes/modes |

---

## 13. Suggested first Unity tasks

1. Create 2D project; set portrait camera orthographic size to fit board.
2. Author `BoardConfig` (rows, cols, spacing, slot values).
3. Prefab peg + slot; generate or hand-place one board.
4. Prefab disc with Rigidbody2D + Physics Material 2D.
5. `DropController` + `Slot.OnTriggerEnter2D` → score event.
6. Thin `GameManager` state machine: Ready → InFlight → Scored → RoundEnd.

---

## 14. Out of scope (unless requested later)

- Networked multiplayer / spectating
- Real-money or prize redemption
- 3D Plinko board
- Full meta-game (shop, XP trees)

This plan is enough to stand up a faithful Plinko feel in Unity 2D and grow outward without rewriting the core physics loop.
