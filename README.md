# tiny-chess-01: a 4 KB chess engine

An original engine in `main.py` (3,960 bytes, zlib+base85 packed to fit
the 4,096-byte cap; the unpacker and payload are the whole engine).
Python 3.13 standard library only. It reads no other repo file.

## Design

- 0x88 board, own legal move generation: castling, en passant,
  promotion, check/checkmate/stalemate, repetition (score 0).
- Search: iterative-deepening alpha-beta with principal-variation
  search to a fixed budget of **150,000 nodes per move** (never a
  clock, so replays are deterministic), transposition table, killer
  moves, MVV-LVA + piece-square-gain ordering, late-move reductions
  and check extensions.
- Quiescence over all captures and promotions, and over all
  replies when in check.
- Evaluation: material (P 100, N 320, B 330, R 500, Q 900), original
  piece-square tables, bishop pair, passed-pawn bonus, and a
  middlegame/endgame king blend. Root moves come from the env's
  `legal_moves` list.

Measured on this machine: 100,000 nodes take 1.7–3.8 s per move
(~26–59k nodes/s, ~3.1 s average in middlegame positions), slowest
well under the 10 s move limit. At the 100k budget, practice
trial-0 scored Elo 876 (1W-0D-5L, 788 s) and trial-1 Elo 1,099
(2W-0D-4L, 645 s) — mean 988, below the 1,154 floor — so the
budget was raised to 150,000 nodes, still well inside the 1,800 s
wall. At the earlier 200k-node budget, practice trial-0 scored
Elo 1,451 (4W-0D-2L) and trial-1 Elo 1,545 (3W-3D-0L), but
trial-1 took ~1,849 s, over the wall.

## Grade it locally

From the kit directory (no Docker needed here):

    node cli/makefun.mjs grade-local --kit . --bot <this repo> --driver process --instances 1

It plays practice trial-0 (6 games vs depth-2..nodes-500k) and replays
it, then prints the Elo. Practice floor is 1,154; the house baseline
(sunfish 4k) rates ~1,365 on practice.
