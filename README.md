[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# tiny-chess-01 — a 4 KB chess engine in Python

A complete chess engine in a single `main.py` of at most 4,096 bytes. The
readable source is `engine.py`; the shipped file is that source packed with
the kit's own `compress` command (an lzma payload with its unpacker).

## How it plays

The board is a 0x88 array of 128 squares. Each side's pieces also live in a
piece list with a reverse index, so move generation walks only the squares
that hold a piece of the side to move, and make/unmake stay cheap.

Evaluation is incremental and tapered. Two accumulators — one middlegame,
one endgame — track material plus piece-square terms from White's side and
are updated square by square as pieces are placed or lifted; a phase counter
over the non-pawn material blends the two scores. The piece-square tables
are not stored as data: they are built at startup from small formulas over
rank, file and centrality (pawns gain as they advance, knights and bishops
gain toward the centre, the king hides on the back rank early and walks to
the centre late). A Zobrist key with a fixed seed is maintained the same
incremental way and drives the transposition table and repetition detection.

The search is iterative deepening over a negamax alpha-beta core with a
principal-variation window at the root. It stops on a fixed budget of
80,000 nodes per move — never on a clock — so the same position and game
history always produce the same move. Inside the search: a transposition
table with mate-score adjustment, null-move pruning, late-move reductions,
check extensions, killer-move ordering, and a quiescence search over
captures and promotions that resolves every capture sequence and every
check evasion before a leaf is evaluated.

## Talking to the env

The program reads `MAKEFUN_ENV_URL` and plays one game at a time:
`POST /game` for the position in play, then `POST /move` with the chosen
UCI move until the answer stops saying `your_move`. For every position it
rebuilds the game from `start_fen` and the move list, so the search sees
the full repetition history; a single legal move is played at once, and
any move the search returns is checked against `legal_moves` first.

## Living in 4 KB

The size budget shaped everything: formula-built tables instead of stored
ones, one packed move integer (from-square, to-square, promotion) instead
of move objects, an undo record that restores the hash outright instead of
recomputing it, and a node budget in place of any clock logic. What is
left out matters too — no opening book, no endgame tablebase, no bishop
pair term; the bytes go to the search instead.
