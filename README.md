# gem5 Memory Hierarchy Lab

Cache and TLB configuration experiments run in gem5's syscall-emulation (SE) mode on an x86 target, for the "Memory Hierarchy Design" course lab (Parts 1 and 2). Screenshots, full analysis, and discussion are in the accompanying lab report; this repo holds the workload, the modified configuration script, and a summary of the results.

## Setup

- gem5 version 25.1.0.1, built for the X86 target on macOS via `scons build/X86/gem5.opt`.
- Workload: `src/hello.c`, cross-compiled with a musl-based x86_64 Linux cross-compiler (`x86_64-linux-musl-gcc -static hello.c -o hello`), since SE mode needs a statically linked Linux/x86 binary and the host is not native x86 Linux.

## Files

- `src/hello.c` - the Hello World workload used for every run.
- `configs/my_tlb.py` - a copy of gem5's deprecated `configs/deprecated/example/se.py` with `system.cpu[i].mmu.itb.size` and `system.cpu[i].mmu.dtb.size` overrides added after `createThreads()`. The stock script exposes cache size/associativity/block size as CLI flags but not TLB size, so this was the way to change it.
- `results/summary.md` - cache and TLB accesses/misses/miss rates extracted from each run's `stats.txt`.

## Running

```
./build/X86/gem5.opt --outdir=m5out_baseline configs/deprecated/example/se.py -c hello --caches --l2cache
./build/X86/gem5.opt --outdir=m5out_tlb128 configs/deprecated/example/my_tlb.py -c hello --caches --l2cache
```
The four single-parameter sweeps (8-way L1D associativity, 128B block size, 128KB L1D size, 2MB L2) add the matching `se.py` size/associativity/block-size flags on top of the baseline command.

## Results (summary)

| Configuration | L1D miss rate | L1I miss rate | dTLB misses / accesses | iTLB misses / accesses |
|---|---|---|---|---|
| Baseline (defaults) | 8.87% | 4.26% | 5 / 406 | 2 / 1269 |
| 8-way L1D associativity | 8.87% | 4.26% | 5 / 406 | 2 / 1269 |
| 128B block size | 5.17% | 2.29% | 5 / 406 | 2 / 1269 |
| 128KB L1D size | 8.87% | 4.26% | 5 / 406 | 2 / 1269 |
| 2MB L2 | 8.87% | 4.26% | 5 / 406 | 2 / 1269 |
| 128-entry TLB | 8.87% | 4.26% | 5 / 406 | 2 / 1269 |

Doubling the cache block size was the only single-parameter change that reduced misses; Hello World's working set already fits within the default cache sizes/associativity, and its TLB footprint (accesses in the low hundreds) never gets close to stressing a 64-entry TLB, so the other changes, including the larger TLB, made no measurable difference. Full discussion is in the lab report.

