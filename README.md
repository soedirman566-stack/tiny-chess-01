[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# tiny-chess-01 — a chess engine in 4 KB of Python (v3)

This is a complete, self-contained chess engine written in Python and squeezed
into a single file. The board uses the classic 0x88 representation: one list of
128 slots where the off-board half of the array makes edge detection a single
bitwise test, so move generation stays branch-light even in an interpreted
language.

Evaluation is incremental and tapered. Every piece carries two piece-square
tables, one tuned for the middlegame and one for the endgame, and the engine
keeps running sums for both phases plus a material counter as pieces are
placed, moved and captured. The final score blends the two table sums by how
much material remains on the board. The tables themselves are not stored as
Python literals — they are decoded at startup from one packed byte table,
which saves well over a kilobyte of source.

Search is a negamax alpha-beta with iterative deepening up to an 80,000-node
budget per move. Positions are hashed with Zobrist keys (seeded from a fixed
PRNG so runs are reproducible) and cached in a 65,536-entry transposition
table that stores depth, bound type and the best move. Move ordering tries
the table move first, then captures by MVV-LVA, then promotions, then quiet
moves that caused cutoffs before. Late quiet moves are reduced, and at the
horizon a capture-only quiescence search settles tactics before the static
evaluation is trusted.

## How it plays

The engine talks to the make.fun environment over HTTP. When the
`MAKEFUN_ENV_URL` variable is set, it polls the server for a game, receives
the position as FEN together with the list of legal moves, answers with its
chosen move in coordinate notation, and repeats until the server reports the
game is done. If the search ever produced a move outside the legal list, it
falls back to the first legal move instead of failing.

## Living inside the size budget

The entry is capped at 4,096 bytes for the main file and 8,192 bytes for the
whole repository, so the readable source is packed with the make.fun kit's
own `compress` command into a 3,511-byte `main.py`. Every design choice —
the packed evaluation tables, one-letter hot-path names, the 0x88 board —
exists to buy playing strength per byte.

## What changed in v3

Two search improvements were added without growing the node budget.
Null-move pruning: when the static evaluation already exceeds beta and the
side to move has non-pawn material, the engine first checks whether giving
the opponent a free move still fails high, and skips the full search when it
does. Killer-move ordering: the two quiet moves that most recently caused a
beta cutoff at each ply are tried early among the quiet moves, which makes
the alpha-beta tree noticeably narrower in practice.
