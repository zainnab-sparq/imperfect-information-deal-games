# How Much Due Diligence Before You Bid?<br>Learning in Intractable Takeover Auctions

[![arXiv](https://img.shields.io/badge/arXiv-pending-b31b1b.svg)](paper/main.pdf) [![paper](https://img.shields.io/badge/paper-PDF-blue)](paper/main.pdf) [![Python](https://img.shields.io/badge/python-3.10%2B-blue)](requirements.txt) [![OpenSpiel](https://img.shields.io/badge/built%20on-OpenSpiel-green)](https://github.com/google-deepmind/open_spiel) [![compute](https://img.shields.io/badge/compute-laptop%20CPU-lightgrey)](#reproducing-the-paper)

## TL;DR

Code, games, and experiments for the paper *How Much Due Diligence Before You Bid? Learning in Intractable Takeover Auctions*. We model M&A and private-equity takeover bidding as a small family of two-player zero-sum imperfect-information auction games on top of [OpenSpiel](https://github.com/google-deepmind/open_spiel), benchmark nine solvers (exact, tabular, and deep) under exact exploitability, and separately solve the auction's own-profit Bayes-Nash equilibrium to read off the value of due diligence and the profit-maximizing amount to buy. Everything runs on a commodity laptop CPU with no GPU and no frontier-model spend. The compiled paper is in [`paper/main.pdf`](paper/main.pdf).

> **New to imperfect-information games or takeover auctions?** In a takeover contest, two buyers bid for a target company whose true worth nobody observes. Each buyer pays for private *due diligence*, noisy homework that sharpens its own estimate, then submits a sealed bid without seeing the other's. "Imperfect information" just means each side knows things the other does not. This repo turns that situation into a game a computer can solve, then asks two questions: what is the smartest way to bid, and how much diligence is actually worth paying for?

<details>
<summary><b>Under the hood: the research behind this repo</b></summary>

The paper makes five contributions: (1) a small family of zero-sum deal games grounded in the takeover-auction literature (common-value with a winner's curse, common-value with a toehold, and independent-private-value), sharing a reusable `DealGame` abstraction; (2) a like-for-like benchmark of nine solvers under exact exploitability, on both iterations and wall-clock, spanning the exact (CFR, MMD, PSRO), tabular-learning (REINFORCE), and deep-learning (PPO, PPG, a generic deep policy gradient, Deep CFR, NFSP) families; (3) evidence that the generic policy-gradient methods PPO and PPG are the strongest *learning* solvers, beating deep CFR and deep fictitious play at equal episode budgets, while the exact solvers stay best wherever the game is small enough to enumerate; (4) a scaling study showing the learning methods' per-target wall-clock stays roughly flat while exact CFR's grows steeply with the game tree, together with a first step into the genuinely intractable regime, a multi-signal auction too large to enumerate, where PPO and PPG drive a *calibrated* learned-best-response exploitability estimate to its resolution floor, below a naive unshaded bidder (reported as a lower bound, not a Nash certificate); and (5) economic results read off the genuine own-profit Bayes-Nash equilibrium: the value of diligence is positive on net, and the profit-maximizing amount is finite and falls as the per-signal cost rises, including when both bidders choose diligence in a symmetric acquisition equilibrium.

Full tables, figures, and caveats are in the [paper](paper/main.pdf).

</details>

## What's in here

| Path | Contents |
|------|----------|
| [`src/dealgame/`](src/dealgame) | The reusable `DealGame` abstraction, the auction games, the solver wrappers, and the exact general-sum equilibrium solver. |
| [`experiments/`](experiments) | The studies that produce every figure and table in the paper. |
| [`results/`](results) | Generated CSVs and figures. |
| [`tests/`](tests) | Game-validity and solver-convergence tests. |
| [`paper/`](paper) | LaTeX source and the compiled PDF. |

## Quick start

OpenSpiel has no Windows wheels, so everything runs in a pinned Docker image. Build it once:

```bash
docker build -t imperfect-info:latest .
```

Run the tests to confirm the games and solvers behave:

```bash
docker run --rm -v "$(pwd):/work" -w /work imperfect-info:latest python -m pytest tests/ -q
```

> **On Windows in Git Bash:** prefix the `docker run` commands with `MSYS_NO_PATHCONV=1`, and if `$(pwd)` does not expand cleanly, substitute the absolute repository path for the volume mount.

## Reproducing the paper

The two main scripts run the full suites and write their CSVs and figures into `results/`:

```bash
# solver benchmark, scaling study, intractable regime, calibration
docker run --rm -v "$(pwd):/work" -w /work imperfect-info:latest python experiments/angle_a_benchmark.py

# economic equilibria: value of information, diligence cutoff, toehold
docker run --rm -v "$(pwd):/work" -w /work imperfect-info:latest python experiments/general_sum_equilibrium.py
```

Each script also accepts a single study name, so you can regenerate one figure without rerunning everything:

```bash
docker run --rm -v "$(pwd):/work" -w /work imperfect-info:latest \
  python experiments/general_sum_equilibrium.py acquisition_symmetric_experiment
```

### Figure and table map

| Paper artifact | Script | Single-study name |
|----------------|--------|-------------------|
| Deep self-play (headline) | `angle_a_benchmark.py` | `headline_deep_experiment` |
| Cross-game exploitability table | `angle_a_benchmark.py` | `cross_game_table` |
| Scaling table | `angle_a_benchmark.py` | `scaling_study` |
| Intractable regime | `angle_a_benchmark.py` | `intractable_experiment` |
| Estimator calibration | `angle_a_benchmark.py` | `calibration_study` |
| Multi-signal calibration | `angle_a_benchmark.py` | `multisignal_calibration_study` |
| Tabular convergence | `angle_c_toy.py` | (full script) |
| Value of information | `general_sum_equilibrium.py` | `asymmetry_gs_experiment` |
| Diligence cutoff | `general_sum_equilibrium.py` | `diligence_experiment` |
| Diligence robustness | `general_sum_equilibrium.py` | `diligence_robustness_experiment` |
| Bid-grid refinement | `general_sum_equilibrium.py` | `grid_refinement_experiment` |
| Symmetric acquisition equilibrium | `general_sum_equilibrium.py` | `acquisition_symmetric_experiment` |
| Toehold at own-profit equilibrium | `general_sum_equilibrium.py` | `toehold_gs_experiment` |

## Code layout

| File | Role |
|------|------|
| [`src/dealgame/base.py`](src/dealgame/base.py) | Reusable deal-game primitives: information-set construction and the zero-sum payoff contract. |
| [`src/dealgame/takeover.py`](src/dealgame/takeover.py) | The common-value takeover auction and its toehold variant as OpenSpiel games. |
| [`src/dealgame/private_value.py`](src/dealgame/private_value.py) | The independent-private-value auction (control for common-value artifacts). |
| [`src/dealgame/solving.py`](src/dealgame/solving.py) | Solver wrappers and the from-scratch tabular policy gradient. |
| [`src/dealgame/deep_solving.py`](src/dealgame/deep_solving.py) | The deep solvers (PPO, PPG, generic deep policy gradient, Deep CFR, NFSP). |
| [`src/dealgame/ppo_solving.py`](src/dealgame/ppo_solving.py) | The PPO/PPG self-play implementation. |
| [`src/dealgame/intractable.py`](src/dealgame/intractable.py) | The multi-signal auction and the learned-best-response exploitability estimator. |
| [`src/dealgame/general_sum.py`](src/dealgame/general_sum.py) | The exact own-profit Bayes-Nash equilibrium solver (fictitious play on factorized payoff tensors). |

## Citation

```bibtex
@misc{naboulsi2026diligence,
  title  = {How Much Due Diligence Before You Bid? Learning in Intractable Takeover Auctions},
  author = {Naboulsi, Zain},
  year   = {2026},
  note   = {arXiv preprint; identifier to be added upon posting}
}
```
