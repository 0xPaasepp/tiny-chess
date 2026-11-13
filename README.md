# tiny-chess-01 — compact alpha-beta engine (≤4096 bytes)

A strong chess engine in a single `main.py` (2,983 bytes as committed).
It is an LZMA-compressed payload plus a two-line unpacker, which the rules
explicitly allow (payload + unpacker count toward the 4,096-byte cap).

## Interface

Talks to the house environment alone over HTTP at `$MAKEFUN_ENV_URL`:

- `POST /game {}` → game in play
- `POST /move {game, move}` (UCI, e.g. `e2e4`, `e7e8q`) → house answers at once
- `POST /resign {game}` (unused — the engine never resigns)

## Design (decompressed source ≈ 5,214 bytes)

- **Board:** 0x88 (128 squares, off-board test `sq & 136`), pieces
  1=P 2=N 3=B 4=R 5=Q 6=K, sign = colour.
- **Move gen:** pseudo-legal generation (sliders / leapers / pawns with double
  push, en passant, promotion, castling), filtered to legal by king safety.
  Perft-verified: startpos 20/400/8902, Kiwipete 48/2039/97862, plus
  en-passant/promotion and castling positions.
- **Search:** negamax + alpha-beta, iterative deepening to a **fixed node
  budget of 80,000 nodes/move** (never a clock — deterministic, so the live
  and replay runs match move for move), quiescence search on captures only.
  At ~20–35k nodes/sec this is ~2–4 s/move, safely under the 10 s/move
  limit and the 3,600 s trial wall.
- **Evaluation:** material (100/320/330/500/900) + piece-square tables
  (pawn advancement, knight/bishop centralisation, king safety).
- **Move ordering:** MVV-LVA captures first, plus a killer-move heuristic
  (the move that caused a beta cutoff at a ply is tried early there).

## Strength

Measured locally against sunfish (the house baseline, UCI `go nodes 50000`,
the same 50,000-node cap the baseline uses) in matches from book openings
— see `dev/match.py`:

- vs sunfish 10,000 nodes: 1 win, 1 draw (2 games).
- vs sunfish 50,000 nodes, final version (80k-node budget + killer moves):
  **2 wins, 3 draws, 0 losses (3.5/5 = 70%), undefeated** (the 6th game
  was cut short by a VM restart). Earlier 60k-node versions split 1-1 in
  2-game matches each.
- Combined vs the 50k baseline: 4 wins, 2 losses, 3 draws (5.5/9 = 61%).

The engine is therefore roughly at or above the sunfish baseline (~1,360
Elo on the practice trials), which is the prize gate. End-to-end, it was
also run against the kit's real env server (practice trial-0): it
connected, received a game, and made 9 consecutive legal moves, all
accepted (HTTP 200).

## Files

- `main.py` — the engine (LZMA payload + unpacker, 2,983 bytes)
- `LICENSE` — MIT
- `.makefun/owner.json` — payout wallet

Build: the readable source lives in `../dev/main_v3.py`; `main.py` is built
by LZMA-compressing it and base85-encoding the payload
(see `dev/` for the packing step and the perft tests).
