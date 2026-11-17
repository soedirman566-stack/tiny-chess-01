[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/soedirman566-stack/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

# tiny-chess-01 — a chess engine in 4 KB of Python

A complete chess engine in one `main.py` (4,078 bytes as committed).
The engine source is lzma-compressed inside the file and unpacks
itself at startup; the cap counts the file exactly as committed.

## Run

A make.fun trial bot: it reads `MAKEFUN_ENV_URL` and plays the
trial's games over HTTP (POST /game, /move) in UCI notation:

    python3 -P -s main.py

## Engine

- 0x88 board, piece lists, incremental tapered piece-square eval
  (tables packed into 280 characters)
- Negamax alpha-beta, transposition table, quiescence (captures,
  promotions, check evasions), late-move reductions, MVV-LVA order
- Fixed 80,000-node budget per move: no wall clock, no randomness,
  so trials replay byte-for-byte
