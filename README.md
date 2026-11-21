# sokoban-solver-01-run-20261121-v1

[![make.fun](https://make.fun/badge/sokoban-solver-01-run-20261121.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)
[![Rank](https://make.fun/badge/sokoban-solver-01-run-20261121/Barkie303/sokoban-solver-01-run-20261121-v1/rank.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)
[![Reward](https://make.fun/badge/sokoban-solver-01-run-20261121/Barkie303/sokoban-solver-01-run-20261121-v1/reward.svg)](https://make.fun/bounty/sokoban-solver-01-run-20261121/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=sokoban-solver-01-run-20261121)

A push-optimal Sokoban solver for the make.fun competition
[Build the best Sokoban solver, 21 November 2026](https://make.fun/bounty/sokoban-solver-01-run-20261121).

## How it works

The levels are a one-square-wide hallway with 1–4 rooms off it, and a box pushed out of a
room can never come back, so the rooms are independent problems joined only by the player's
walk. The solver therefore:

1. **Splits a level into rooms** — the connected components of open floor once the hallway
   row (the row the player starts on) is taken away. Each component's door is the cell of it
   that touches the hallway, and the player enters through the hallway cell outside it. A box
   may already sit on the door itself, so the first step in can itself be a push.
2. **Solves each room push-optimally with A\* over pushes.** A state is the boxes plus the
   player's reachable region, canonically the region's smallest cell (regions are disjoint
   components, so the smallest cell names its region). A step costs one push; the estimate is
   the cheapest one-to-one matching of boxes to goals over static shortest paths, which never
   overestimates, so the first goal state A\* pops is push-optimal. Boxes on cells from which
   no goal is reachable are pruned at once. A goal state needs every box on a goal and the
   door reachable again, so the player can walk on to the next room.
3. **Stitches the rooms** — the player walks along the hallway from door to door (plain BFS),
   and each room's own moves follow. Only pushes are scored, so the room order does not matter
   for the score.

Everything is budgeted by work, never by the clock: each room search expands at most a fixed
number of states (and retries once with a larger fixed budget), and every tie is broken by
insertion order, so two runs of the program write the same bytes. No clock, no randomness.

Practice-set score: **1,000,000** — every instance solved in exactly P\* pushes (13, 16, 29,
47, 60), whole set in ~0.6 s. The same perfect score on six further sets freshly drawn with
the kit's own generator. The house baseline plain-astar 1.0.0 scores 200,000 on the practice
set (only i001 solved).

## Local run

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=practice/input MAKEFUN_OUTPUT_DIR=out node main.ts
```

`main.ts` is plain TypeScript with erasable syntax only (Node 22 runs it as-is) and uses only
the Node standard library. It reads `/input/<id>/level.txt` and writes
`/output/<id>/solution.txt` for every instance in one run.
