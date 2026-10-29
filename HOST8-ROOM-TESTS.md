# Host8 rooms at work: exact Yang–Mills computations

The Host8 room store (8-byte rooms on local disk, the same store this repository benchmarks)
is now used as working memory for exact, integer-only spectral computations in
[Finite-universe-Yang-mills-existence-and-mass-gap-using-HEB-C](https://github.com/JesseBrown1980/Finite-universe-Yang-mills-existence-and-mass-gap-using-HEB-C)
(branch `ym-3x4-room-backed`, run log `native/ym/results/3x4_room_backed/RUN-LOG.md`).

The problem: a finite S₃ lattice gauge Hamiltonian whose matrix is too large for runner RAM
(3×4 lattice, about 40 GB at 12 bytes per entry). The rooms hold the matrix, one 8-byte room
per entry, written once and streamed in index order on every Gauss–Seidel sweep. Every
result is an integer record with a SHA-256, and every run can be reproduced.

## Tools

| tool | what it does |
|---|---|
| `ymroom passa` | scans the whole gauge-fixed configuration space and keeps one representative per gauge orbit, with its weight and magnetic number |
| `ymroom build` | writes each representative's row of the Hamiltonian as rooms: a header room `w:u32 \| mag:u16 \| cnt:u16`, then one room `j:u32 \| z:i32` per nonzero entry |
| `roomschur` | the `shellschur` inertia count (Haynsworth Schur complement, interval Gauss–Seidel, BigInt LDLᵀ) with the matrix streamed from the rooms instead of held in RAM |

Both tools: Rust 1.81, `clippy -D warnings`, no floating point.

## Results on Host8

Machine: Intel i5-8300H (4 cores, 8 threads), 16 GiB RAM.

| # | test | result | time |
|---|---|---|---|
| 1 | room builder vs reference builder `ymsec`, 2×3 lattice, byte compare | **EXACT**: 2,646 states, 74,956 nonzeros | 0.12 s |
| 2 | same, 2×4 | **EXACT**: 58,040 states, 2,606,690 nonzeros | 5.9 s |
| 3 | same, 3×3 | **EXACT**: 288,645 states, 15,408,669 nonzeros | 37.6 s |
| 4 | 3×3 matrix written to rooms | 15,408,669 entries; symmetry check (W-self-adjoint) PASS; header read back EXACT | 93.8 s |
| 5 | 3×3 eigenvalue count from rooms, threshold t = 60,000,000 | core 101, negative 7, PASS | 669 s |
| 6 | 3×3 eigenvalue count from rooms, t = 90,000,000, segmented run | 2,931 of 2,940 core directions resolved, 9 unresolved: AMBIGUOUS | — |
| 7 | **3×4 pass A**: every orbit of the 3×4 sector | **45,508,080 states, 7 flat**; matches the independent GitHub run 40391283682 | **23.3 min** on 8 threads |
| 8 | 3×4 pass B: the 3×4 matrix written to rooms | in progress (about 3.35 × 10⁹ rooms, about 27 GB) | — |

Test 7 is the 3×4 lattice's whole gauge-fixed configuration space, 6¹³ = 13,060,694,016
configurations, reduced to 45,508,080 sector states on a 2018 laptop in 23 minutes.

## Storage measured before any write

| device | sequential read | 4 KiB random read |
|---|---|---|
| USB stick, 2 TB | 10 MB/s | 61–65 reads/s |
| HDD `WD10SPZX` | 55–196 MB/s | 72–78 reads/s |

Random reads run at about 70 per second, so the room runner never reads rooms at random. It
streams them in order and keeps only the active columns in RAM.

## Cross-checks on GitHub

The same room tools are being run on GitHub Actions, so that Host8 and GitHub cross-check
each other (branch `ym-room-ci-checks` in the Yang–Mills repository):

- **P2:** the room builder must reproduce the reference matrix byte for byte on 2×3, 2×4
  and 3×3.
- **P3:** the room runner and the in-RAM runner must give record-identical output on 3×3 at
  t = 60, 72 and 90 million. The work is split into 116 parallel column jobs, all run
  through the HEB-C HTTP-0 backend.

On 2×3 the two runners already agree on all 5,476 column records.

## Next tests

Each idea below states what it claims and the experiment that would test it.

1. **Rooms make the 3×4 count possible on a laptop.**
   - *Hypothesis:* with the matrix in rooms, one Gauss–Seidel sweep over 3×4 takes the
     streaming time of the room file (about 45 min at 10 MB/s, less on the HDD), and the
     count fits in 12 GiB of RAM.
   - *Test:* the frozen memory pilot (`MEMORY-PILOT-SPEC.md`) measures RESERVE, COL_BYTES
     and sweep time, then the production count runs.
2. **Sweep time is set by sequential bandwidth.**
   - *Hypothesis:* sweep time ≈ room-file bytes ÷ sequential read rate.
   - *Test:* time sweeps of the same room file on the USB stick, the HDD and an SSD, and
     compare with the prediction.
3. **A real surface-code decoder on rooms.**
   - *Hypothesis:* an integer decoder (for example union-find) with its syndrome graph in
     rooms reaches the published union-find threshold of 9.9 % for the toric code with perfect
     syndrome measurements ([Delfosse and Nickerson, *Quantum* 5, 595 (2021)](https://arxiv.org/abs/1709.06218)).
   - *Test:* inject errors with a seeded integer generator on d × d surface codes, decode,
     and count logical failures against a reference decoder such as PyMatching.
4. **Exact amplitudes for small circuits.**
   - *Hypothesis:* the integer circuit path reproduces a quantum circuit's output
     probabilities exactly.
   - *Test:* for n ≤ 28 qubits, compare its output against a dense state-vector run of the
     same circuit, then against Willow-style cross-entropy (XEB) scoring.
