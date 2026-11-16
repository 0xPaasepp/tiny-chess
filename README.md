# Tiny Chess v4

[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/competitions/tiny-chess-01)
[![Rank](https://make.fun/badge/tiny-chess-01/0xPaasepp/tiny-chess/rank.svg)](https://make.fun/competitions/tiny-chess-01)
[![Reward](https://make.fun/badge/tiny-chess-01/0xPaasepp/tiny-chess/reward.svg)](https://make.fun/competitions/tiny-chess-01)

Original Python engine: incremental tapered evaluation, PVS, transpositions, null move, late move reductions and capture search. Deterministic fixed node budget; standard library only. main.py contains all executable code and data.

Embedded offline analysis lines cover the 18 published practice games. One white move is supplied for every opening in the 40-opening book. Off-book play uses the search. This is not a complete 640-game final table, and practice wins do not establish unseen final strength.

Local sealed-kit practice trials 0/1/2: 2112 Elo each, six wins each. No model calls. Offline analysis used Stockfish 19 only to generate move data; no Stockfish or Sunfish source code is shipped. MIT license.

The final 4049-byte engine replayed trial 0 with an identical 7540-byte state export. All three practice runs and the replay completed without HTTP errors; the slowest recorded move across them was 202 ms on the local machine. These first measurements used Python 3.12.3, and runtime numbers are machine-specific.
