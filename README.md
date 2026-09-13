# Hi, I'm <span>Sairaj Sandip Kashid</span>

**Systems engineer who builds the layer where software meets silicon.**
Final-year B.Tech, writing GPU kernels, physics engines, robot planners and the
test tooling that proves they work.

I don't have a GPU on my desk. So I built the four things below on a 2-core
laptop, made every number reproducible, and wrote down every place where the
hardware I *don't* have would change the answer.

---

## The work

| repo | what it is | the number I'd point at |
|---|---|---|
| **[miniflex-gpu](https://github.com/sairajkashid-prog/miniflex-gpu)** | Particle physics engine (PBD / position-based fluids) with a stable C ABI, a multithreaded CPU backend and a CUDA backend | **18.1 M neighbour-pairs/s** on 2 cores; CPU and CUDA backends agree to **8.6 × 10⁻⁷** relative |
| **[gpu-verify-lab](https://github.com/sairajkashid-prog/gpu-verify-lab)** | Differential + property-based verification for numerical kernels — the test harness, not the kernels | catches **5 of 6** seeded bug classes; the 6th is written up as a known blind spot |
| **[mppi-fleet](https://github.com/sairajkashid-prog/mppi-fleet)** | Sampling-based model-predictive control for warehouse robot fleets, written so the CUDA port is mechanical | **0.92** path efficiency at K=2048 vs **0.69** at K=128; **10–25× fewer** shelf contacts than the reactive controller teams actually ship |
| **[nvhealth](https://github.com/sairajkashid-prog/nvhealth)** | GPU health monitoring: drift detection, ECC tracking, burn-in | **114 tests** across the four repos; zero false positives on a healthy device across 5 seeds |

Every repo has CI, a benchmark with real numbers, and a README that says what
is *not* done yet.

---

## What I'd say in the interview

**On the CUDA backend in miniflex-gpu.** There is no nvcc on this machine, so I
wrote a host-side shim that emulates the CUDA slice I actually use — kernel
launch, thread indices, device malloc/memcpy/atomics. The same `.cu` file
compiles under `g++` and runs on the CPU in CI, which means the parity test
between backends runs on every commit instead of only on the days I have a GPU.
The shim is ~200 lines and it is not pretending to be a GPU; it is pretending to
be a *second implementation*, which is what makes the parity test meaningful.

**On testing numerics.** Comparing two floating-point kernels with
`np.allclose` tells you nothing you can act on. `gpu-verify-lab` reports ULP
distance, magnitude correlation, NaN/Inf counts and run-to-run determinism, and
then *guesses the cause* — "this is a reassociated reduction, expected on a GPU"
versus "index `k` is reading out of bounds". The framework's own tests are
written by injecting bugs and asserting they are caught, and one bug class
deliberately is not: left-to-right versus tree accumulation, which no
input/output comparison can ever distinguish. That's in the README, prominently.

**On the bug I'm proudest of.** The MPPI benchmark first showed my planner was
*worse* than a 30-line reactive controller on every metric. It wasn't the
planner. The path-efficiency metric was `straight_line / distance_driven`, and
robots stop within a tolerance of the goal, so they legitimately drive less than
the straight-line distance and the ratio came out above 1.0. The fix was in the
metric, not the controller — and the test `test_path_efficiency_never_exceeds_one`
now guards it, because a metric that can exceed its own bound will fool you twice.

**On false positives.** In nvhealth I shipped CUSUM with the textbook constants,
k = 0.5σ, h = 5σ. Then I measured it: **34% of clean devices raised an alarm.**
A monitor that cries wolf gets muted on day one, and a muted monitor is worse
than none. I measured the rate across eight (k, h) pairs and shipped k = 0.75σ,
h = 8σ — 1% false positives, and a 1.5σ shift still caught inside 15 samples.
The measurement is a table in the README and a test in the suite.

---

## Skills, mapped to what NVIDIA asks for

| | where I've actually done it |
|---|---|
| **C / C++** | `miniflex-gpu`: C++17 engine, C ABI (`dlopen`-able `.so`), ctypes bindings, header-only shared numerics so CPU and CUDA kernels cannot diverge |
| **CUDA** | `miniflex-gpu/src/cuda/engine.cu` + the emulation shim; `mppi-fleet/PORTING.md` is an op-by-op CPU→CUDA port spec (thread mapping, texture-object SDF sampling, deterministic block reductions) |
| **Python** | All four repos; numpy used as a stand-in for batched array ops, with the batch axis shaped the way a kernel grid would be |
| **Test & automation** | `gpu-verify-lab` (property-based + differential testing, JUnit/HTML reporting), CI on every repo, `nvhealth` burn-in with a silent-data-corruption check |
| **3D / simulation** | PBD/PBF fluids, granular Coulomb friction, mass-spring cloth; dam-break and sand-column benchmarks with measured compression ratios |
| **Robotics** | MPPI trajectory optimisation, unicycle dynamics, chamfer signed-distance fields, prioritised multi-robot coordination |
| **Linux / cloud** | SQLite-backed daemon with retention and rollups, stdlib-only so it runs on a node with no network, self-contained HTML reports for air-gapped clusters |

---

## Reading order if you only open one

1. **[mppi-fleet/PORTING.md](https://github.com/sairajkashid-prog/mppi-fleet/blob/main/PORTING.md)** —
   shows I know what happens *after* the numpy version works.
2. **[gpu-verify-lab/README.md](https://github.com/sairajkashid-prog/gpu-verify-lab#readme)** —
   shows I know what a test framework is for, including what it cannot do.
3. **[miniflex-gpu/results/BENCHMARK.md](https://github.com/sairajkashid-prog/miniflex-gpu/blob/main/results/BENCHMARK.md)** —
   the actual numbers, and the machine that produced them.

---

📍 Pune, India · 🎓 Final-year B.Tech · ✉️ sairajkashid.prog@gmail.com
<br>🔗 [LinkedIn](https://linkedin.com/in/sairajkashid-prog) · [GitHub](https://github.com/sairajkashid-prog)

<sub>Everything here was built on a 2-core laptop with no GPU. Benchmark numbers
state the hardware they came from, and each README lists what I would do next
with real hardware.</sub>
