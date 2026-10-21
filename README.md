# cmix-wolf P13

A Hutter Prize entry: a fork of **cmix-neif-1.0 by Marcus Wölk**, on the line
cmix -> cmix-hp -> fast-cmix -> fx-cmix -> fx2-cmix -> cmix-lex -> cmix-neif -> cmix-wolf.
Licensed under GPL-3. The upstream LICENSE, NOTICE.md and third_party/ license
texts are retained unmodified; the author's attribution is in CHANGES.md and
NOTICE.md. The full upstream documentation — with the section numbering that
CHANGES.md and training/README.md refer to — is kept at `doc/README_upstream.md`.

## Result

| quantity | value |
|---|---:|
| `comp9`, the compressor | 3,991,874 B |
| `archive9`, self-extracting | 91,885,607 B |
| `comp9.args` (`-e enwik9 archive9`) | 19 B |
| **S = comp9 + archive9 + args** | **95,877,500 B** |
| vs upstream entry (95,914,051 B) | **-36,551 B (-0.038 %)** |
| sha256 `comp9` / `archive9` | 5443e50b1a5bedab135fbbe4b2b1ab2877d8a1c9ca8f3f112889835fbf170cbf / 4d44a185d913f2e1303685f81f82b71d83b156981da66b456ca16f8626336edd |
| md5 `comp9` | 3597e9b9ae2260cc0aa644ca145db9a1 |

## Changes against cmix-neif-1.0

| # | change |
|---|---|
| 1 | PRL: per-row decay for the readalike prior (`PORT_PRL=1`, shipped but off upstream) |
| 2 | E1 interpolation + E3b timestamp windows kN=9 (`NEIF_E1=1`, `NEIF_E3B=1`, shipped but off upstream) |
| 3 | IND restore subset (`PORT_RESTORE_IND=1`): the two doubly-indirect context models of the predecessor line are restored |
| 4 | PPMd logit-prior gate lowered: kGate 0.55 -> 0.35 (one scalar in `src/ppm_logit_prior.h`) |
| 5 | packed encoder builds without PGO profile use and links with lld-18 (author's documented 18.1.3 reference build) |

Item-by-item description and measurements: `CHANGES.md`.

## Measurements (enwik9, one core, local x86-64 host)

| item | value |
|---|---:|
| span16 payload (16 MiB slice at 381 MiB) | 2,024,891 B vs author 2,048,995 B (-24,060 B) |
| full-stream encode, wall clock | 54.0 h |
| full-stream decode | F10 reference verified bit-exact against enwik9 (sha256) in 41.6 h single core; the P13 archive9 re-verification (same decode chain) is running |
| memory | under the 10 GiB rule: the PPMd pool self-sizes via /proc/self/statm (upstream measured 9.75 GiB, same mechanism) |
| free disk | about 21 GB in the working directory (upstream measurement, layout unchanged) |

## Requirements

- Linux x86-64, **AVX2 + FMA (x86-64-v3)**; a build without AVX2 stops with an error
- clang / lld / llvm **18.1.3** (Ubuntu 24.04 snapshot); another major version produces a different binary
- one CPU core, no GPU, no network after `install.sh`
- input `enwik9`, 1,000,000,000 B, sha256 159b85351e5f76e60cbe32e04c677847a9ecba3adc79addab6f4c6c7aa3744bc

## Build

```
bash install.sh      # system packages; the only step with root + network
bash build.sh        # offline; writes comp9 into the working directory
```

The byte-exact packed reproduction of the submitted comp9 from this tree:
`bash build_and_construct_comp.sh` (clang/lld 18.1.3).

## Run and verify

```
./comp9 -e enwik9 archive9                      # compress; ~54 h on this host
mkdir out && cp archive9 out/ && cd out
./archive9                                      # decompress; writes enwik9_uncompressed
cmp /path/to/enwik9 ./enwik9_uncompressed       # must print nothing
grep Threads /proc/$(pgrep -f 'archive9')/status # shows 1
```

Both programs print a progress line in percent; runtime is not linear in it.

## Package

```
README.md              this file
CHANGES.md             changes against cmix-neif-1.0, with measurements
environment.txt        frozen toolchain, artefact sizes, sha256/md5, S
doc/README_upstream.md full upstream documentation (cmix-neif-1.0)
LICENSE / NOTICE.md / third_party/   upstream license + attribution, unchanged
install.sh build.sh build_and_construct_comp.sh makefile   build chain
src/ dictionary/ pgo_data/ prof_input/ weights-*.c4        coder + build inputs
```
