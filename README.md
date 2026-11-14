# Origins bots — milestone 1: bots that walk

Bots that walk around the island using **our own navigation**, because 19.10
builds have no navigation data. Includes everything from the bot test v4
(resizable window, crash fix).

## Install

1. Copy the `Erbium` folder from this zip over your Erbium folder (replace files).
2. Build **Release x64** in VS 2022, inject `Erbium.dll` like before.

## Test (10 minutes)

Land somewhere fairly **open** first (a field, not inside a town). Then open
the new **Bots** tab:

1. **Spawn 1 walker**: a bot appears near you, standing still.
2. Run about **40 m away**, then press **Come to me**.
   The status shows `scanning ground` for a moment, then `walking`.
   The bot should run to you.
3. Press **Wander**: the bot walks off to a random spot 50–150 m away.
4. **Spawn 5 walkers**, then **Come to me** again. Watch whether the server
   stays smooth.
5. Try a harder route: stand on the other side of a hill, a river or a house.

**Send me:** a screenshot of the status box, and a few words on what the bots
did (walked straight to you? got stuck? jumped? went around things?).

## Reading the status box

| Line | Means |
| --- | --- |
| `scanning ground` | Checking the terrain ahead (each spot is checked once, then remembered) |
| `walking`, `X m to go` | Following its route |
| `stuck: jumping` / `stepping aside` / `rerouting` | Its 3 ways of getting unstuck, in that order |
| `looking for a way around` | No route straight ahead; trying other directions |
| `no route found` | Gave up on this goal (after 6 tries) |
| `NO AI CONTROLLER` | This bot can't move; tell me if you see it |
| `Ground checks: N per second` | Terrain checks; capped at 120 per tick so the server stays smooth |

## How it works

- **Seeing the ground:** the server traces a line straight down every 4 m
  around a bot's route. That gives the ground height and whether it's
  walkable (not too steep). Each spot is checked once and remembered, shared
  by all bots.
- **Finding a route:** an A* path search over those spots, up to 100 m ahead,
  never stepping up more than about a 33° slope, never cutting corners past
  blocked spots. Far goals are walked in 100 m legs.
- **Moving:** the bot is steered with movement input, like a player's stick.
  It uses Phoebe's character and AI controller, but **not** Epic's brain: on a
  dedicated server only an AI-controlled character can be moved by the server.
- **Getting unstuck:** no progress for 1.5 s → jump; still stuck → step
  sideways for 1 s; still stuck → mark that spot blocked and reroute.

## Not in milestone 1 (on purpose)

- Bots don't loot, shoot or build yet. Walking comes first.
- Bots aren't registered as match players yet (they don't count in "players
  left", have no team).
- 4 m spots are too coarse for doorways and building interiors: bots walk
  around buildings, not through them.

## Verified here

- **The navigation core passed 17 tests** on synthetic terrain (flat ground,
  a wall with one gap, a cliff, a blocked goal, an unreachable goal, the
  checks-per-tick budget). A 100 × 100 m route takes about 0.3 ms.
  The test is in `tests/OriginsNavTest.cpp`.
- **Compiled against Erbium's real headers:** the bot code with zero errors
  and zero warnings, `GUI.cpp` with zero errors, and no new errors elsewhere.

**Not verified:** a Visual Studio build and running it in game. This is the
first time the server moves bots itself, so the walk test is the real test.
