# tiny-chess-01 — a chess engine in 4,058 bytes

An original chess engine for make.fun's tiny-chess-01, in a single
`main.py` (4,058 bytes, limit 4,096). Like the house baseline, the file is
an LZMA + base85 packed payload that unpacks to a self-contained engine
using only the Python standard library. It plays over HTTP against the
trial environment (`MAKEFUN_ENV_URL`: POST `/game`, `/move`, `/resign`,
UCI moves).

This is v3 of the entry. The only change from v2 is one line: the tiered
node budget went from 170,000 / 120,000 / 70,000 to 260,000 / 180,000 /
110,000 by game phase (plies < 70 / < 130 / later). No search, eval, or
movegen change. The bigger budget converts two of v2's repetition draws
into wins on the kit's trial.

## Design

- 0x88 board, pseudo-legal move generation with king-safety filtering,
  make/unmake with full undo (perft-verified, see below).
- Iterative-deepening alpha-beta under a tiered node budget (260,000 /
  180,000 / 110,000 by game phase). No wall clock and no randomness
  anywhere, so play is fully deterministic and replays match
  move-for-move.
- Quiescence search (captures, promotions, full check evasions),
  transposition table, PVS at the root and inside the search, null-move
  pruning, killer moves, LMR, check extensions.
- Tapered midgame/endgame evaluation: material, file-symmetric quantized
  piece-square tables, bishop-pair bonus; repetition and 50-move scores
  inside the search.

## Local verification (kit practice trials)

- Perft: startpos 20/400/8902/197281, Kiwipete 48/2039/97862, en-passant
  pin 14/191/2812/43238, CPW positions 6/264/9467 and 44/1486/62379 — all
  match the known node counts. A python-chess oracle walk found no
  legal-move differences.
- Gauntlet with the exact submitted file: **trial-0: 4 wins, 0 draws,
  2 losses — Elo 1,451; trial-1: 4 wins, 2 draws, 0 losses — Elo 1,654**
  (mean 1,552). v2 scored 1,364 on both trials. That is above the 1,154
  practice floor by ~300 points and above the house baseline's ~1,365
  practice mean. Six-game samples carry wide error bars; the trial-0
  gain alone (+87) is inside the noise band, but the two trials together
  (+188 mean) and the 12-game record (8W 2D 2L vs v2's 5W 4D 3L) make the
  improvement clear.
- Determinism: identical moves across separate processes and across two
  independent trial runs.
- Speed: the tiered budget costs up to ~8.3 s per move on a shared VM
  core (limit 10 s/move; trial walls 1,361 s and 1,563 s vs the 1,800 s
  limit). No time losses in 12 games. The official trial pod is faster
  than this VM per the competition rules, but the margin is thin and
  stated here, not hidden.

## License

MIT — Copyright (c) 2026 Ava.
