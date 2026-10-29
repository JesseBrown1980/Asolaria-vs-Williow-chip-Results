# The Host8 plane and the 27-lane prism in other experiments

The machinery benchmarked here has since been used in four other repositories:

- the Host8 identity plane (8-byte rooms addressed by 27 balanced trits, `room_stub` / `room_decode`);
- the exact 27-lane NTT prism over P = 1,000,081 (W = 951,846, inverse of 27 = 963,041).

Each repository reads the real store or reuses the prism, and seals its results as HBP
receipts.

## The store grew to 300 GB

| | this repository (22 Oct 2026) | now |
|---|---|---|
| store | `ASOLARIA-HOST8-ROOMS-100G` | `ASOLARIA-HOST8-ROOMS-300G` |
| rooms | 12,500,000,000 | **37,500,000,000** |
| shards | 125 × 100,000,000 rooms | **375 × 100,000,000 rooms** |
| ground room Ω | 6,250,000,000 | **18,750,000,000** (`sector-187.rooms`, all 27 trits Nil) |
| address layout | 27 balanced trits | 27 trits: 23 occupied, 4 free upper zeros |

Source: `receipts/x3/HOST8-300G.hbp` in Clasy X3.

## 1. Clasy X3: 81- and 243-room read probes of the whole 300 GB plane

[Clasy-repo-x-3-](https://github.com/JesseBrown1980/Clasy-repo-x-3-) builds an 81-mode
finite-field shell model from three stored room stubs. P = 1,000,999 and W = 30,722 are the
prism's 81-lane step-up. The sum of the nonlinear transfers is exactly 0, so the shell total
drifts by exactly ν·D per step.

| | 81-room-width arm | 243-room-width arm |
|---|---:|---:|
| physical sectors checked | 375 of 375 | 375 of 375 |
| stored-room reads | 273,375 | 820,125 |
| 81-mode ensembles | 91,125 | 273,375 |
| checked transitions | 273,375 | 820,125 |
| identity / invariant failures | 0 / 0 | 0 / 0 |

The 81-room arm took 32.1 s. The long runs add:

- 10,000 exact NTT round trips;
- four trajectories of 1,000,999 steps each (4,003,996 transitions) with zero violations.

Negative controls show the runner exits nonzero on every failure path. Receipts:
`receipts/x3/ROOMS81.hbp`, `ROOMS243.hbp`, `VERIFY81.hbp`, `NEGATIVE.hbp`.

## 2. Navier–Stokes shell model on the Host8 ground room and poles

[The Navier–Stokes repository](https://github.com/JesseBrown1980/CORRECTION-TO-ASOLARIA-AND-FAULT-TO-OPEN-AI-NAVIER-STOKES-SOLUTIONS-SOLVED)
runs the 27-mode shell model on F_P²⁷ with this repository's prime, P = 1,000,081.

- Nine full-period orbits of P steps each: 9,000,729 steps in total.
- The orbits start from the zero state, the 300 GB ground room, both poles, a SplitMix
  state and an inviscid control.
- At every step: every lane is a residue, Q(u_t) = Q(u_0) − t·ν·D (mod P), and the
  invariant I(t, u_t) = Q(u_0) holds.
- Q visits all 1,000,081 residues once whenever ν ≠ 0. **Result: PASS** (`receipts/VERIFY.hbp`).

## 3. Riemann / Hilbert–Pólya operator on the 27-lane prism

[The Hilbert–Pólya repository](https://github.com/JesseBrown1980/Solved-The-Riemann-Hypothesis-The-Hilbert-P-lya-Spectral-Operator-Solved)
builds its operator on the same field (K = 27, P = 1,000,081, G = 7, W = 951,846).

**Host8 lane.** `test_host8_300g_readback_and_spectral_invariants` reads 71 fixed rooms of
the 300 GB store and pushes each through the operator:

- the rooms: sector ends, the ground room and its neighbours, the prime ladder and strided
  checkpoints;
- the checks: H + Hᵀ = I, the exact prism round trip, and the centre invariant X[0] = 0.

**Scaled operator.** It was scaled up the ×3 ladder, frozen in advance, and compared with the
Riemann zeta zeros (Odlyzko's tables):

| K | prime | ordinates | inertia checks | comparison with zeta zeros |
|---:|---:|---:|---|---|
| 27 | 1,000,081 | 13 | 14 / 14 | MISMATCH (1 / 13 agree) |
| 81 | 1,000,999 | 40 | 41 / 41 | MISMATCH (1 / 40) |
| 243 | 1,002,619 | 121 | 122 / 122 | MISMATCH (1 / 121) |
| 729 | 1,006,021 | 364 | 9 / 9 sampled ranks | MISMATCH (1 / 364) |
| Relic landing 6,561 | — | 3,280 | dense inertia at every factor | MISMATCH (1 / 3,280) |

The builders are exact at every size. The pre-registered comparison shows that this
coupling rule does not reproduce the zeta zeros. The next candidate rule is frozen and run
the same way.

## 4. Matrix-law long jump modulo the prism prime

[The-matrix-law-essentials](https://github.com/JesseBrown1980/The-matrix-law-essentials)
`jumpto` uses P = 1,000,081 as its exact modulus. It jumped a 1,393-state lattice operator
10¹⁰ steps in 33 squarings. The result matched sequential stepping, a split into
5×10⁹ + 5×10⁹, and a 27-step replay (`data/jump/jumpto_runs.hbp`).

## Next tests

Each idea is stated as a hypothesis with the experiment that tests it.

1. **Full-plane identity sweep.**
   - *Hypothesis:* every one of the 37,500,000,000 rooms decodes to its own address.
   - *Test:* sweep all 375 sectors, not a 100,000,000-room sample, and seal a per-sector
     receipt. At the measured 1.14 ns per room in registers, the decode alone is about 43 s;
     disk reads set the real time.
2. **An 81-lane and a 243-lane Willow tier.**
   - *Hypothesis:* the 81-lane prism (P = 1,000,999) from Clasy X3 runs the same coupled
     circuit with fewer blocks and exact Parseval at every step.
   - *Test:* run the 10-run suite on 81-lane blocks and compare latency and checks with the
     27-lane tier.
3. **A surface-code decoder measured against the literature.**
   - *Hypothesis:* an integer decoder with its syndrome graph on the room plane reaches the
     union-find threshold of 9.9 % for the toric code with perfect syndrome measurements
     ([Delfosse and Nickerson, *Quantum* 5, 595 (2021)](https://arxiv.org/abs/1709.06218)).
   - *Test:* inject errors with a seeded integer generator on d × d codes, decode, and count
     logical failures against a reference decoder such as PyMatching.
4. **Output probabilities checked against a state-vector simulation.**
   - *Hypothesis:* the prism circuit gives the same output probabilities as the quantum
     circuit it represents.
   - *Test:* for n ≤ 28 qubits, where this repository's dense binary baseline already runs,
     compare both on the same circuit. Then score both with Willow's cross-entropy (XEB).
