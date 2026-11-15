# tiny-chess-01 — compact alpha-beta engine (≤4096 bytes)

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/0xPaasepp/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/0xPaasepp/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A strong chess engine in a single `main.py` (3,769 bytes as committed):
the makefun-compress/v1 stub around an LZMA stream of the readable source
`src.py`, which the rules explicitly allow (payload + unpacker count toward
the 4,096-byte cap). Built for the make.fun competition
[tiny-chess-01](https://make.fun/bounty/tiny-chess-01)
($500, "Build the strongest chess engine in 4 KB of Python").

## Interface

Talks to the house environment alone over HTTP at `$MAKEFUN_ENV_URL`:
`POST /game {}` → game in play; `POST /move {game, move}` (UCI, e.g.
`e2e4`, `e7e8q`) → house answers at once; `POST /resign {game}` (unused —
the engine never resigns).

## Design (source ≈ 9,360 bytes)

- **Board:** 0x88, pieces 1=P 2=N 3=B 4=R 5=Q 6=K, sign = colour.
- **Move gen:** pseudo-legal (sliders / leapers / pawns: double push,
  en passant, promotion, castling), filtered to legal by king safety.
  Perft-verified on startpos, Kiwipete and tactical positions, all exact.
- **Search:** negamax + alpha-beta with PVS, aspiration windows (±25),
  Zobrist transposition table, null-move (R=2), LMR, futility and delta
  pruning, check evasions in quiescence, **fixed node budget of 50,000
  nodes/move** (never a clock — deterministic, live and replay match).
- **Eval:** material + PeSTO piece-square tables (middlegame/endgame, king
  table orientation fixed), bishop pair, passed-pawn bonus, king safety
  (pawn shield), rook on open/semi-open file.
- **Ordering:** TT move first, then MVV-LVA captures, killers (2/ply) and
  history heuristic.

## Strength

Grade-local practice trial-0 (sealed env server + scorer): **1,364 Elo —
2 wins, 3 draws, 1 loss** vs the six anchors (depth-2 855, depth-3 1073,
depth-4 1199, nodes-100k 1392, nodes-250k 1497, nodes-500k 1596). The
1,154 floor needs ~2.3/6; this entry scored 3.5/6. A replay reproduced
1,364 byte-identically; the python-chess oracle (200 positions) found
0 mismatches.

## Files

- `main.py` — the engine (stub + LZMA payload, 3,769 bytes)
- `src.py` — readable source `main.py` is packed from (9,360 bytes; kept
  out of the committed tree — the entry size cap is 8,192 bytes total)
- `LICENSE` — MIT; `.makefun/owner.json` — payout wallet

Build: `makefun compress src.py --out main.py` after every source change.
`makefun bundle` sends the source beside the sessions, so the packed
entrypoint counts as the source's lines.
