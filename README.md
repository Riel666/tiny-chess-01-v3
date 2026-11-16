# tiny-chess-01-v3

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Riel666/tiny-chess-01-v3/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Riel666/tiny-chess-01-v3/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

An original chess engine for make.fun's tiny-chess-01, in a single `main.py` (4,058 bytes, limit 4,096). The file is an LZMA + base85 packed payload that unpacks to a self-contained engine using only the Python standard library. It plays over HTTP against the trial environment (`MAKEFUN_ENV_URL`: POST `/game`, `/move`, `/resign`, UCI moves).

v3 changes one line from v2: the tiered node budget went from 170,000 / 120,000 / 70,000 to 260,000 / 180,000 / 110,000 by game phase (plies < 70 / < 130 / later). No search, eval, or movegen change.

## Design

- 0x88 board, pseudo-legal move generation with king-safety filtering, make/unmake with full undo (perft-verified).
- Iterative-deepening alpha-beta under a tiered node budget, quiescence search, transposition table, PVS, null-move pruning, killer moves, LMR, check extensions. Fully deterministic, no randomness, no wall clock.
- Tapered midgame/endgame evaluation: material, piece-square tables, bishop-pair bonus; repetition and 50-move scores inside the search.

## Local verification (kit practice trials)

- Perft: startpos, Kiwipete, en-passant pin and CPW positions all match the known node counts; a python-chess oracle walk found no legal-move differences.
- Gauntlet with the exact submitted file: **trial-0 Elo 1,451 (4W 0D 2L); trial-1 Elo 1,654 (4W 2D 0L)** — mean 1,552 vs v2's 1,364. Above the 1,154 practice floor and the ~1,365 house baseline mean.
- Speed: up to ~8.3 s per move on a shared VM core (limit 10 s/move); trial walls 1,361 s and 1,563 s vs the 1,800 s limit. No time losses in 12 games.

## Run it

```sh
MAKEFUN_ENV_URL=<trial environment URL> python3 main.py
```

## License

MIT — Copyright (c) 2026 Ava.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
