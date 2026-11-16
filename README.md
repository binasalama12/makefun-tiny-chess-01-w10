# Noble Quasar Chess II

Noble Quasar Chess II is a self-contained chess engine in Python for the make.fun tiny-chess-01 competition. It plays a complete game through the competition interface, choosing every move on its own — no books, tablebases, or network help. Only the standard library is used. The entrypoint, `main.py`, is 4,032 bytes: a packed form produced with lzma and base85 from a longer readable source, decompressed and executed in memory at startup.

## Design

The board uses a 0x88 representation for cheap off-board detection and fast move generation in pure Python. Search is negamax with principal variation search, late-move reductions, null-move pruning, and check extensions. A transposition table reuses results across iterative deepening; killer moves and a history table order quiet moves behind captures and promotions. At the horizon, quiescence continues over captures, promotions, and check evasions so tactics are not cut off blindly.

Evaluation is incremental: material plus piece-square terms with strong centralization bonuses, a tapered king term (middlegame shelter against endgame centralization), and a king-shelter term counting friendly pawns around the king. Repetition and the fifty-move rule score as draws inside the search. The piece-square weights and node tiers were tuned in local head-to-head matches on the competition's trial openings, which exposed the previous entry's drift in long maneuvering games and drove the stronger central terms and richer endgame node budget in this version.

## Determinism

No clock anywhere: thinking is limited by fixed node budgets that depend only on game length — 50,000 nodes early, 34,000 in the middle phase, 22,000 in long games — sized to finish comfortably inside the 10-second per-move cap. Zobrist keys come from a fixed seed, so the same position always produces the same move.

## Rules compliance

`main.py` is 4,032 bytes, under the 4,096-byte limit, and the whole repository stays under the 8,192-byte limit. The engine runs on Python 3.13 using only the standard library.
