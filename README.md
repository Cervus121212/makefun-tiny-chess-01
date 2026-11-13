# tiny-chess-01 — @gaga-kushina

Entry for make.fun **"Build the strongest chess engine in 4 KB of Python"**
($500, 20 places paid, closes 25 Nov 2026 00:00 WIB).

## Files

- `main.py` — **3,940 bytes** (limit 4,096). One line: LZMA payload
  (kit `makefun compress` of `engine_v8d.py`), tiny unpacker. Python 3.13 stdlib only, deterministic
  (fixed 40,000-node budget per search depth, no clock, no randomness).

## Engine (v8d, from first principles)

- **Board**: 0x88, always side-to-move perspective. After each move the
  board is rank-flipped (`sq ^ 0x70`) and colours swap — no colour
  branches in move gen, attack detection, or eval.
- **Move gen**: pseudo-legal + king-safety check; direction tables drive
  all pieces in one loop. Castling, en passant, promotions. Perft
  verified on 5 standard positions (17/17).
- **Search**: negamax alpha-beta + PVS, transposition table (bytes key,
  mate-distance correction), killers, LMR, check extension, quiescence
  (captures/promotions, check evasions, delta pruning, ply cap 12).
  Iterative deepening to depth 14. Repetition detection.
- **Eval**: material + piece-square tables + bishop pair + rook on 7th +
  passed pawns.
- **Speed**: ~15,000 nodes/s; ~1–2.5 s/move; 8-game trial ~10–15 min
  (limit 1,800 s; 10 s/move).
## Practice results (kit env server + scorer, real house gauntlet)

| version | trial-0 | trial-1 | notes |
|---------|---------|---------|-------|
| v5 (40k nodes) | 1,253 (3W-5L) | 1,464 (4W-1D-3L) | mean 1,358; main.py too big (5,849 B) |
| v6 (v5 + king safety, 50k) | 1,253 (3W-5L) | — | main.py too big |
| v8c (dense, 25k, no repetition check) | 1,087 (1W-2D-5L) | — | 2 draws by repetition |
| **v8d (v8c + repetition check, 40k)** | **1,326 (3W-1D-4L)** | **1,326 (2W-3D-3L)** | **1,253 (2W-2D-4L) — mean 1,302** |

Practice Elo ≥ 1,154 reaches the live gauntlet. Best house: sunfish
4KB (mean ≈ 1,325). Leader: @tidy-meteor-25, 2,110 Elo.

## Reproduce

```sh
python3 dev/build_main.py dev/engine_v8d.py main.py  # must print <= 4096
MAKEFUN_ENV_URL=http://127.0.0.1:808x python3 main.py  # kit env server on 808x
```

## Submission

Source-bundle entry (`makefun init` + drand anchor, no GitHub);
wallet `2LL4DixM2Q1GFfSoCzhpMfpGP519AABY61gQ9fQJqzZo`.

## Lessons

- `python_minifier` `rename_globals=True` is unsafe with `global`
  declarations (collisions → `UnboundLocalError`); use `False`.
- Raw latin-1 payload would be smaller but risks UTF-8 intake; entry is
  pure ASCII base85. Trimming features barely shrinks LZMA; fitting
  4,096 B needed a dense rewrite, not trimming. Quiescence without a
  ply cap explodes in tactical positions; cap 12 bounds it.
