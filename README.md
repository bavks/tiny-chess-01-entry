# tiny-chess-01 entry

A chess engine in a single `main.py` of 3,821 bytes (cap: 4,096), written
for make.fun's tiny-chess-01. `main.py` is an lzma-packed payload with a
two-line unpacker — the packing the rules expressly allow; the unpacked
source is plain Python 3.13, standard library only, and makes no model
calls.

## Design

- 10x12 mailbox board, Zobrist hashing, make/unmake with full state
  (castling, en passant, halfmove clock) restored exactly.
- Negamax alpha-beta with a transposition table, iterative deepening
  under a fixed budget of 110,000 nodes per move (quiescence nodes
  count double, which bounds the cost of capture-heavy positions) —
  never a clock, so a replayed trial plays the identical moves.
  Quiescence over captures and promotions, with all evasions when in
  check; check extensions; late-move reductions; MVV-LVA, killer and
  TT move ordering.
- Evaluation: material, piece-square tables (king blended between
  middlegame and endgame tables by phase), bishop pair, passed-pawn
  bonuses. Repetitions and the 100-ply rule score 0 inside the search.
- The driver plays the env's games over HTTP, rebuilding the position
  history from each game's move list so repetition state is exact, and
  verifies every chosen move against the env's legal-move list with a
  capture-value fallback, so it can never stall on an illegal move.

## Local measurement (kit practice trials, real house gauntlet)

Graded locally with the kit's own env and scorer: practice trial-1,
Elo 1451 (4W-0D-2L, every game decided by checkmate), 747 s of the
1,800 s trial wall. The house baseline (sunfish 4k) rates 1,279-1,451
across the three practice instances (mean 1,365); the final
qualification floor is 1,154. Perft suites (start position, Kiwipete,
and four further standard positions, up to depth 4) all match the
known counts, and a 300-game random-play fuzz keeps the incremental
Zobrist key exact through every make/unmake.
