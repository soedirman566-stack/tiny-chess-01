[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# tiny-chess-01 — a full chess engine in a single small Python file

This repository holds a complete, legal-move chess engine in Python,
squeezed into a strict size budget. The readable source lives in
`engine.py`; the submitted entrypoint, `main.py`, is that source
packed with the make.fun kit's `compress` command: 3,511 bytes, under
the 4 KB entrypoint cap.

The board is the classic 0x88 representation: a 128-square array where
the off-board test is a single AND against 0x88, keeping move
generation branch-light. Pieces live in per-side lists with an index
array, so making and unmaking a move touches only the squares involved.
Evaluation is incremental — two
accumulators track middlegame and endgame scores blended by a tapered
phase weight, and the piece-square tables are decoded at startup from
a compact packed table rather than stored as literals.

Search is a negamax alpha-beta with iterative deepening over a fixed
budget of 80,000 nodes. Positions are hashed with Zobrist keys from a
seeded generator, and a 65,536-entry transposition table caches depth,
bound flags and best moves.
Move ordering tries the table move first, then captures by MVV-LVA,
then promotions. At the leaves, a capture-only quiescence search keeps
horizon effects from poisoning the static score, and when the side to
move is in check all evasions are searched.

## How it plays

The engine talks to the grading environment over HTTP. It reads the
`MAKEFUN_ENV_URL` environment variable on startup and, if one is set,
loops asking the server for the current game; whenever the server says
it is our turn it parses the FEN, runs the search, and posts the chosen
move back in coordinate notation. The board, hash key and evaluation
accumulators are rebuilt from scratch by the FEN loader for every
position, so nothing leaks from one game into the next.

## Designing under a size budget

Four kilobytes is not much room, so every feature had to pay rent:
names collapse to one or two letters, tables ride inside a packed
string, and anything that could not prove its strength in testing was
cut.

## What changed in v3

Version 3 keeps the v1/v2 skeleton — same board, same evaluation, same
80,000-node budget — and adds two search improvements that earn their
bytes. Null-move pruning lets the search skip a move when the static
score is already above beta and the side to move has real material:
if giving the opponent a free move still fails high, the subtree is
cut at reduced depth. Killer-move ordering remembers, per ply, two
quiet moves that recently caused beta cutoffs and tries them earlier
among the quiet moves. Earlier experiments with a bishop-pair bonus and
quiescence delta pruning were measured, found neutral to negative, and
removed; the shipped combination is the one that beat the previous
version in direct practice gauntlets. The packed entry remains fully
deterministic — the same position always produces the same move.
