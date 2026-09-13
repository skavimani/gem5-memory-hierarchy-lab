# Results Summary

All six runs used the same `hello` workload (`src/hello.c`) via `configs/deprecated/example/se.py` (or `configs/my_tlb.py` for the TLB run), with `--caches --l2cache` and gem5's default hierarchy (64KB 2-way L1D, 32KB 2-way L1I, 2MB 8-way L2, 64B block size, 64-entry TLBs) except for the one parameter changed per run.

## L1 data cache

| Configuration | Accesses | Hits | Misses | Miss rate |
|---|---|---|---|---|
| Baseline (defaults) | 406 | 370 | 36 | 8.87% |
| 8-way L1D associativity | 406 | 370 | 36 | 8.87% |
| 128B block size | 406 | 385 | 21 | 5.17% |
| 128KB L1D size | 406 | 370 | 36 | 8.87% |
| 2MB L2 | 406 | 370 | 36 | 8.87% |
| 128-entry TLB | 406 | 370 | 36 | 8.87% |

## L1 instruction cache

| Configuration | Accesses | Hits | Misses | Miss rate |
|---|---|---|---|---|
| Baseline (defaults) | 1269 | 1215 | 54 | 4.26% |
| 8-way L1D associativity | 1269 | 1215 | 54 | 4.26% |
| 128B block size | 1269 | 1240 | 29 | 2.29% |
| 128KB L1D size | 1269 | 1215 | 54 | 4.26% |
| 2MB L2 | 1269 | 1215 | 54 | 4.26% |
| 128-entry TLB | 1269 | 1215 | 54 | 4.26% |

## TLB (data / instruction)

Identical across all six runs, including the 128-entry TLB run:

| Metric | dTLB | iTLB |
|---|---|---|
| Read accesses | 218 | 0 |
| Write accesses | 188 | 0 |
| Execute accesses | 0 | 1269 |
| Read misses | 3 | 0 |
| Write misses | 2 | 0 |
| Execute misses | 0 | 2 |
| Total accesses | 406 | 1269 |
| Total misses | 5 | 2 |

## Takeaway

Only doubling the cache block size (64B to 128B) changed anything: fewer, wider lines cut both the L1D and L1I miss counts because `hello`'s few distinct cache lines each satisfy more consecutive accesses. Associativity, L1D size, and L2 size were already more than the workload needed, so widening them further had nothing left to fix. The TLB result is the same story: raw accesses stayed at 218 (dTLB reads), 188 (dTLB writes), and 1269 (iTLB) with only 5 dTLB and 2 iTLB misses even at 64 entries, so doubling the TLB to 128 entries could not reduce a miss count that was already close to zero.

