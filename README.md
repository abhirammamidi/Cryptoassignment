# Cryptography Assignment 1 — Discrete Logarithms and Hash Functions

**B.Tech Semester 4 · Cryptography · Group 9**

| Roll No. | Name |
|---:|---|
| 8 | Arathi Vinod M |
| 11 | Ashish Rajesh |
| 17 | Jeswin C Joji |
| 26 | Nandith T Anil |
| 46 | Abhiram Mamidi |

---

## 1. Overview

**Part 1** implements six algorithms for the discrete logarithm problem g^x ≡ h (mod p):

- brute force
- baby-step giant-step (BSGS)
- Pollard's rho
- Pohlig–Hellman
- index calculus
- the linear sieve

Each one is run on the assignment instances A–H, and every recovered x is independently checked with `pow(g, x, p) == h`.

**Part 2** implements SHA-256, SHA-512 and SHA3-256 from their specifications (FIPS 180-4 and FIPS 202) without using `hashlib`. It also includes:

- validation against the supplied test vectors
- an avalanche experiment
- a SHA-256 length-extension forgery

Only the Python standard library is used for every algorithm. `matplotlib` is optional and is used only to draw the experiment plots.

---

## 2. Project structure

```
cryptography_assignment_1/
├── README.md
├── CHANGES.md                 fix log and verification record
├── assignment_data.py         instances A–H, SHA test vectors
├── run_all.py                 launcher (forwards to part1/run_all.py)
├── part1/
│   ├── bruteforce.py          Task 1.0
│   ├── bsgs.py                Task 1.1
│   ├── pollard_rho.py         Task 1.2
│   ├── pohlig_hellman.py      Task 1.3
│   ├── index_calculus.py      Task 1.4
│   ├── linear_sieve.py        Task 1.5
│   ├── run_all.py             Task 1.6 runner: solves, verifies, prints both tables
│   ├── common.py              shared helpers (gcd, inverse, CRT, primality, factoring, timing)
│   ├── data_loader.py         reads assignment_data.py in any common layout
│   ├── params.py              single source of all algorithm parameters
│   ├── test_part1.py          29 unit and end-to-end tests
│   └── experiments/           one script per required experiment (CSV + plots)
└── part2/
    ├── sha2.py                SHA-256 and SHA-512 (one parameterised implementation)
    ├── sha3.py                Keccak-f[1600], SHA3-256 and SHAKE128
    ├── test_hashes.py         Task 2.4 validation (PASS/FAIL, exit status)
    ├── avalanche.py           Task 2.5
    └── length_extension.py    Task 2.6
```

---

## 3. Requirements

- Python 3.9 or newer (developed and tested with Python 3.13.5 and 3.12)
- No third-party packages for any algorithm
- Optional, for plots only: `pip install matplotlib`

---

## 4. How to run

All commands are run from the project folder `cryptography_assignment_1/`.

### Part 1

```bash
python3 run_all.py                    # required tasks + full A–H comparison table (about 2 minutes)
python3 run_all.py --required-only    # only the tasks the assignment asks to be solved (under 30 s)
python3 part1/test_part1.py           # 29 unit and end-to-end tests
```

Useful options for `run_all.py`:

| Option | Meaning |
|---|---|
| `--data PATH` | use a different `assignment_data.py` |
| `--only bsgs,rho` | run selected algorithms only |
| `--instances C,C2` | run selected instances only |
| `--cell-seconds 600` | measure more of the optional table cells (default budget 120 s per cell) |
| `--out DIR` | write result files to another folder |
| `--use-instance-params` | use B/M values stored in the data file instead of the assignment values |

### Part 2

```bash
python3 part2/test_hashes.py          # validation; exit status 0 only if every test passes
python3 part2/avalanche.py            # writes part2/avalanche_histogram.png
python3 part2/length_extension.py     # prints the forged message and tag, verified with hashlib
```

### Experiments (Part 1)

```bash
cd part1/experiments
python3 exp_bruteforce_extrapolate.py   # 1.0  timings of A and B, extrapolation to D
python3 exp_bsgs_memory.py              # 1.1  peak table size vs sqrt(n), 8 GB estimate
python3 exp_rho_runs.py                 # 1.2  10 seeds on C and C2: mean, SD, comparison with 1.25*sqrt(n)
python3 exp_ph_scaling.py               # 1.3  cost vs largest prime factor of the group order
python3 exp_ic_B.py                     # 1.4  stage times for B = 100 ... 1500 on F
python3 exp_ls_vs_ic.py                 # 1.5  linear sieve vs index calculus on G
python3 exp_previously_failed.py        # re-runs every case that failed in earlier versions
```

Results (CSV, Markdown, JSON and PNG) are written to `part1/experiments/results/`.

---

## 5. Using a different `assignment_data.py`

Replace `assignment_data.py` in the project root, or pass `--data PATH`. No other change is needed.

**Layouts.** `part1/data_loader.py` recognises any of these:
- a dictionary of instances (any variable name)
- tuples `(p, g, h[, order])`
- one variable per instance, or one variable per field
- a list of records carrying a label
- a `get_instances()` function

Labels are case-insensitive, and common field names (`p`/`prime`, `g`/`generator`, `h`/`target`, `order`/`n`) are accepted.

**Required fields.** Only `p`, `g` and `h` are required. The order of g is checked and corrected, and the factorisation is computed when it is not supplied.

**Errors.** If the file cannot be read, a clear error message is printed (exit status 2) instead of a traceback. Missing instances are reported and skipped.

---

## 6. Output of `run_all.py`

The runner prints three things:
1. a progress line per run
2. **Table 1**, every run with instance, algorithm, status, recovered x, time, group operations, memory and parameters
3. **Table 2**, the Task 1.6 matrix: a measured time per algorithm and instance, or a numbered reason

| Status | Meaning |
|---|---|
| PASS | solved, and x verified with `pow(g, x, p) == h` |
| NOT RUN | feasible, but above the per-cell time or memory budget; the estimate is printed |
| INFEASIBLE | estimated above 1 year, or more memory than the machine has |
| EXTRAPOLATED | brute force on D: timed on D-sized numbers, never run to completion |
| N/A | the algorithm does not apply (e.g. index calculus needs a safe prime p = 2q + 1) |
| FAIL / TIMEOUT | a real problem; the reason is printed |

Exit status 0 means every required task passed.

---

## 7. Results on our machine

Measured on a MacBook Air (Apple silicon), Python 3.13.5, with the supplied `assignment_data.py`. Times vary between runs and machines.

| Task | Instance | Algorithm | Time (s) | Group operations | Result |
|---|---|---|---:|---:|---|
| 1.0 | A | Brute force | 0.0091 | 149,722 | x = 149722 |
| 1.0 | B | Brute force | 0.5402 | 9,336,324 | x = 9336324 |
| 1.0 | D | Brute force | — | — | extrapolated: about 9.3 × 10^19 years (average) |
| 1.1 | A | BSGS | 0.0001 | 1,053 | x = 149722 |
| 1.1 | B | BSGS | 0.0005 | 6,848 | x = 9336324 |
| 1.1 | C | BSGS | 0.2347 | 1,266,031 | x = 349412852845 |
| 1.2 | C | Pollard rho | 1.0598 | 3,253,661 | x = 349412852845 |
| 1.2 | C2 | Pollard rho | 0.8812 | 2,768,137 | x = 3623507350375 |
| 1.3 | D | Pohlig–Hellman | 0.0014 | 9,790 | x = 3048631699222626176292590840522770 |
| 1.3 | E | Pohlig–Hellman | 0.0002 | 2,910 | x = 38043505020760957 |
| 1.4 | F | Index calculus (B = 500) | 0.0496 | 46,381 | x = 772419623157 |
| 1.4 | G | Index calculus (B = 2000) | 0.9387 | 312,280 | x = 565608321738391 |
| 1.5 | G | Linear sieve (B = 700, M = 900) | 0.4532 | 25,099 | x = 565608321738391 |
| bonus | H | Index calculus (B = 2076) | 18.84 | 6,552,436 | x = 4733727204259496922 |
| bonus | H | Linear sieve | 2.788 | 41,861 | x = 4733727204259496922 |

The full A–H comparison table (Task 1.6), the experiment plots and their discussion are in the report.

---

## 8. Design notes and assumptions

**Operation counting.** `group_ops` counts modular multiplications. A call `pow(a, e, p)` is charged its square-and-multiply cost. Pohlig–Hellman reports its outer operations, its inner (BSGS) operations and their sum.

**Pollard rho.**
- The walk uses the assignment's three rules (multiply by g, square, multiply by h).
- Which rule applies is decided by a keyed hash of the current value, with a fresh key per seed and per restart. Each run is therefore an independent random walk.
- When the collision equation has a non-invertible coefficient, d = gcd(b₂ − b₁, n), all d candidate solutions are tested. If d = n, the walk restarts.
- Group operations are about 3 × iterations, because Floyd's method uses three multiplications per round.

**Pohlig–Hellman.** Prime powers are handled by digit lifting (one base-q digit at a time), then the results are combined with the Chinese Remainder Theorem.

**Index calculus and linear sieve.**
- Both work for a safe prime p = 2q + 1 with a generator g. The linear system is solved modulo the prime q using sparse Gaussian elimination, and the missing parity bit is recovered with Euler's criterion.
- Index calculus tests both g^k and p − g^k = g^(k+q) for smoothness at each step, and steps through exponents with a random stride.
- The linear sieve adds the anchor equation log_g(g) = 1. If a parameter choice gives too few relations, it enlarges M (and, if needed, B), and the values actually used are printed.

**Linear sieve rank on G.** The system has rank 1017 of 1026 columns, so it is not full rank. Every small-prime logarithm needed for the final step was nevertheless determined and verified.

**Parameters.** All parameters come from `part1/params.py`, which uses the assignment's starting values: index calculus B = 500 (F) and B = 2000 (G); linear sieve B = 700, M = 900.

**Primality testing.** Miller–Rabin with the first 13 prime bases. This is a proof of primality for every number below 3.3 × 10^24 (~81 bits); larger numbers get extra random bases.

**Part 2.** `hashlib` is used only in `test_hashes.py` and in the verification step of `length_extension.py`, never inside `sha2.py` or `sha3.py`. All SHA-2 constants and all Keccak round constants and rotation offsets are generated programmatically.

---

## 9. Academic integrity and AI assistance

This project was developed with the assistance of **Claude (Anthropic)**, an AI assistant. It was used for:
- explanations of the algorithms and their mathematics
- drafting and debugging implementations
- designing tests and experiments

All code was reviewed, run and tested by the group. Every reported result was produced by running the programs ourselves. Each group member can explain the parts of the implementation they worked on.

---

## 10. References

1. NIST FIPS 180-4, *Secure Hash Standard (SHS)*, 2015.
2. NIST FIPS 202, *SHA-3 Standard: Permutation-Based Hash and Extendable-Output Functions*, 2015.
3. A. J. Menezes, P. C. van Oorschot, S. A. Vanstone, *Handbook of Applied Cryptography*, CRC Press, 1996 (Chapters 3 and 9).
4. J. M. Pollard, "Monte Carlo methods for index computation (mod p)", *Mathematics of Computation*, 32 (1978).
5. S. C. Pohlig and M. E. Hellman, "An improved algorithm for computing logarithms over GF(p) and its cryptographic significance", *IEEE Transactions on Information Theory*, 24 (1978).
6. D. Coppersmith, A. M. Odlyzko, R. Schroeppel, "Discrete logarithms in GF(p)", *Algorithmica*, 1 (1986).
7. J. Sorenson and J. Webster, "Strong pseudoprimes to twelve prime bases", *Mathematics of Computation*, 86 (2017).
