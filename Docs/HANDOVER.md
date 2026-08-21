# Gunsmoke Mines — build handover

Context for continuing this build on another machine or in a new session.
Read this before touching anything.

## What this is

UCA Level 4 games design project (FGCT4017). Top-down wild-west dungeon
crawler. **Unreal Engine 5.6, Blueprints only, no C++.** Deliberately scoped
small — see `Gunsmoke Mines - GDD v2.2.docx` §15 for what is and isn't being
built, and `Gunsmoke Mines - 2-Day Build Plan.docx` for the step-by-step.

Repo: `github.com/BR44XX/gunsmokemines2` (private).
The `.uproject` sits at the repo root.

## Progress

**Blocks 1, 2 and 3 are complete and working.** Blocks are defined in the
Build Plan.

| Block | State |
|---|---|
| 1 — project, level, player, camera, dash | Done |
| 2 — health component, shooting, bandits | Done |
| 3 — keys, chests, powerups, checkpoints | Done |
| 4 — boss, win screen, HUD, build the mine | **Next: start at 4.1** |

The game currently plays: move, aim at cursor, dash with i-frames, shoot,
bandits chase and damage you, keys, chests that cost keys and spawn powerups,
checkpoints, death and respawn.

## Assets

All in `Content/` (flat, not in subfolders).

| Asset | Notes |
|---|---|
| `BP_Player` | Character. Spring arm + camera, dash, shooting, KeyCount, LastCheckpoint, OnDeath respawn |
| `BP_bandit` | Character. Timer-driven "Think" AI, no Behavior Tree |
| `BP_Checkpoint` | Stores its own location on the player |
| `BP_Key`, `BP_Chest`, `BP_Powerup` | Overlap pickups |
| `BPC_Health` | Actor Component. Shared by player, bandits, boss |
| `BP_GameMode` | Default Pawn = BP_Player |
| `IMC_gunsmoke` | Input mapping context (NOT IMC_Default — template has its own) |
| `IA_Move`, `IA_Dash`, `IA_Shoot` | IA_Move is Axis2D, the others Digital |
| `E_Powerup` | SpeedSurge, Deadeye |
| `Content/TopDown/L_Mine` | The level (still in the template folder) |

Placeholder meshes come from `Content/LevelPrototyping/Meshes` (`SM_Cylinder`,
`SM_Cube`). **Starter Content is not installed** — do not reference
`Shape_Cylinder` or `P_Explosion`.

## Conventions in use

- All damage goes through `BPC_Health → ApplyDamage`. **Never** Unreal's
  built-in Apply Damage / Apply Radial Damage — different system, fails silently.
- To touch anything on the player from another Blueprint: `Cast to BP_Player`
  → `As BP Player` → the variable. Never `self`.
- Pickups use a `bCollected` boolean guard; the checkpoint uses
  `Set Actor Enable Collision = false` instead.
- `Delay` is used rather than `Set Timer by Event` throughout, to avoid
  delegate pins.

## Traps already hit — do not re-diagnose these

1. **Pawn collision ignores the Visibility channel.** Line traces pass straight
   through characters. Fixed on `BP_bandit` by setting the capsule's Collision
   Presets to Custom with Visibility = Block. **The boss will need the same.**
2. **`Get Component by Class` defaults its Target to `self`.** In BP_bandit this
   made the bandit fetch its own health component and damage itself.
3. **Missing exec wires cause silent stops.** Caused three separate bugs. When a
   chain does part of its work and then nothing, look for a white wire that
   isn't there before suspecting anything cleverer.
4. **"Accessed None" aborts the rest of the chain.** A null Target doesn't just
   skip that node — everything downstream is abandoned. Check Output Log.
5. **Pure nodes re-evaluate on every read.** A `Get X + 1` feeding both a Set and
   a Print gives different answers to each. Read the Set node's own output pin
   instead.
6. **New Booleans default to false.** Cost time on `bCanDash`, `bCollected` and
   `bCanAttack`. Check Default Value after compiling.
7. **`Mesh (CharacterMesh0)`** on BP_Player is set to NoCollision — it's unused
   and was double-firing overlap events.
8. **Debug prints in `BPC_Health`** print the owner's display name, because the
   component is shared and an anonymous number tells you nothing.

## Known loose ends

- Level is at `Content/TopDown/L_Mine`, inside Epic's template folder. Fine, but
  untidy.
- Several `Print String` debug nodes still in `BPC_Health`, `BP_Key` and
  `BP_Checkpoint`. Remove them once the HUD exists in 4.3.
- `AttackDamage` on the bandit may still be raised to 40 from testing. GDD value
  is 12.
- Bandit `MaxHealth` is 40 and player `Damage` is 50, so bandits die in one shot.
  Raise MaxHealth to ~150 if you want Deadeye to be noticeable.
- The command line can't push to this repo — git is authenticated as a different
  account. **Push from GitHub Desktop.**

## Setting up on a new machine

1. Install **UE 5.6** via Epic Games Launcher. This is a very large download —
   start it before anything else.
2. Install **Git**, **Git LFS** and **GitHub Desktop**.
3. **Install Git LFS before cloning**, or you get 130-byte text stubs instead of
   real assets.
4. Clone `gunsmokemines2` and open the `.uproject`.

**Always close Unreal before pulling.** Committing and pushing with it open is
fine; pulling underneath it corrupts state.
