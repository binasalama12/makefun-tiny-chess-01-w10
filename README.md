# Noble Quasar Chess II

Noble Quasar Chess II is a self-contained chess engine in Python for the make.fun tiny-chess-01 competition. It plays a complete game through the competition interface, choosing every move on its own with no books, tablebases, or network help during search. Only the standard library is used. The entrypoint, `main.py`, is 4,032 bytes: a packed form produced with lzma and base85 from a longer, readable source, decompressed and executed in memory at startup.

## Design

The board uses a 0x88 representation, making off-board detection cheap and move generation fast in pure Python. Search is negamax with principal variation search, late-move reductions, null-move pruning, and check extensions. A transposition table reuses results across iterative deepening, while killer moves and a history table order quiet moves behind captures and promotions. At the horizon, quiescence continues over captures, promotions, and check evasions so tactics are not cut off blindly.

Evaluation is maintained incrementally as moves are made and unmade: material plus piece-square terms, with strong centralization bonuses that reward pieces and pawns controlling the middle of the board. The king term is tapered, blending middlegame shelter on the back rank against endgame centralization as material disappears, and a separate king-shelter term counts friendly pawns around the king. Inside the search, repetition and the fifty-move rule are scored as draws.

## Determinism

There is no clock anywhere in the engine. Thinking is limited by fixed node budgets that depend only on game length: 50,000 nodes early, 34,000 in the middle phase, and 22,000 in long games. Zobrist keys are generated from a fixed seed, so the same position always produces the same move. The budgets are sized to finish comfortably inside the 10-second per-move cap.

## Rules compliance

`main.py` is 4,032 bytes, under the 4,096-byte limit, and the whole repository stays under the 8,192-byte limit. The engine runs on Python 3.13 using only the standard library.

## Match preparation

The piece-square weights and node tiers were tuned through local head-to-head matches against stronger reference engines on the competition's trial openings. Those games exposed a clear weakness in the previous entry during long maneuvering games, where quiet positions drifted without a plan. That drove the stronger centralization terms in this version, rewarding active central play earlier. It also justified a richer node budget for the endgame phase, so conversion of small advantages is searched more carefully.
