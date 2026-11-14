Tiny Chess Engine — make.fun tiny-chess-01 entry (4th attempt)

A chess engine in 4 KB of Python: 0x88 board, alpha-beta with iterative
deepening, transposition table, quiescence search (captures, promotions,
check evasions), null-move pruning (R=2), late-move reductions,
check extensions, killer moves, MVV-LVA ordering,
material + piece-square tables + bishop pair + passed-pawn bonus.
Fixed 120,000-node budget per move (deterministic, stdlib only,
no clock, no model). The budget is 120k, not 160k: at 160k the mean
think time was 4.6 s/move and the 8-game practice trial (1,750 s wall
clock) timed out at move 333 of ~400; at 120k the mean is ~3.5 s/move
and the trial fits.

Speed is the whole point of this attempt. The 3rd attempt reached only
~24k nodes/s, so its node budget had to be cut to 80k to fit the harness
timeout and its trial-0 Elo fell to 1253. This engine is re-authored for
speed:

- Incremental evaluation: a running score is updated in make/unmake via a
  signed per-piece-per-square value (white +, black -); eval() is O(1)
  plus the bishop-pair term.
- Zobrist hashing: 64-bit keys from a fixed-seed LCG generated at import;
  the transposition table is keyed by the integer hash (no board-bytes
  copies), with castling, en-passant and side-to-move folded in.
- Repetition detection via a hash-count dictionary maintained in
  make/unmake (replaces O(n) history scans).
- Data-driven castling tables, one-line castling-rights mask, centrality
  via a lookup table instead of abs() in the hot path.

main.py is the entry: lzma preset 9e (FORMAT_ALONE) + base85 packed source
with a two-line unpacker (3932 bytes, limit 4096). Readable source is in
../makefun-tiny-chess-04-dev/min_src.py (packed by ../tc04-pack.py).

Verified locally 2026-11-15: perft 6/6 correct (startpos d4 197281,
kiwipete d3 97862, pos3 d4 43238, pos4 d3 9467, pos5 d3 62379,
pos6 d3 89890); make/unmake round-trip clean over 254 random plies;
incremental eval matches full recompute over 120 plies; Zobrist keys
agree across move orders; mate-in-1 (Qxf7#) found; deterministic
(same position, same move, same node count across fresh processes).
Speed: ~52k nodes/s from the start position (cold transposition table),
~43k nodes/s on the last iterative-deepening depth.
Practice-trial Elo vs the kit gauntlet (floor 1,154, must also beat
sunfish) to be measured before submission.
