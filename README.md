# fedids-mia-audit

Membership inference audit of federated intrusion detectors, built with Flower and PyTorch.

This repository has the code, the fixed client partitions and the derived results of the executed
audit reported in:

> A. C. S. Pontara, F. M. Cruz, K. A. P. da Costa, T. J. Lucas, J. P. Papa, D. Colombo,
> A. R. Avila. *Membership Inference Attacks in Federated Intrusion Detection: A Systematic Review
> and an Auditing Framework*. Submitted to Computers & Security, 2026.

The audit asks whether the aggregation algorithm and the heterogeneity of the clients change how
vulnerable a federated intrusion detector is to membership inference: can an observer tell whether
a given network flow was part of some client's training data?

## What was run

| Factor | Levels |
|---|---|
| Datasets | CIC-IDS2017, CICIoT2023 |
| Clients | 10, simulated on one machine with the Flower simulation engine, full participation |
| Non-IID level | Dirichlet label skew, alpha = 0.1, 0.5, 1.0 |
| Aggregators | FedAvg, FedProx, SCAFFOLD, FedAdam |
| Seeds | 0, 1, 2 |
| Training | 100 rounds, MLP 128-64, SGD lr 0.05, batch 32, 1 local epoch |
| Attacks | Loss, Reference gap, Gradient norm, Gradient alignment, Trajectory |
| Audited rounds | 10, 50, 100 |

2 datasets x 3 alphas x 4 aggregators x 3 seeds = **72 federated training runs**, plus a positive
control with 200 member and 200 non-member canaries (flipped labels) on CIC-IDS2017, alpha 0.5,
seed 0, four aggregators.

Data, partitions and seeds are identical for all aggregators; only the aggregator changes
between comparable cells.

## Main results (round 100)

| | CIC-IDS2017 | CICIoT2023 |
|---|---|---|
| Mean macro F1 per cell | 0.948 to 0.979 | 0.619 to 0.688 |
| AUC, all attacks and runs | 0.494 to 0.506 | 0.498 to 0.513 |
| Max TPR at 1% FPR | 1.24% | 1.43% |
| Max TPR at 5% FPR | 5.61% | 6.25% |
| FedAdam minus FedAvg, loss AUC | -0.0001 (sign changes) | +0.0019 (positive in 9 of 9 pairs) |

- All five attacks stay close to chance in every configuration.
- On CICIoT2023, FedAdam is slightly more exposed than FedAvg and FedProx in all nine paired
  alpha/seed comparisons. The margin is small.
- High exposure in some classes (DoS, DDoS, Mirai) is matched by the false positive rate of
  non-members of the same class, so it is not membership leakage.
- Members flagged across seeds are reproducible, but flagged non-members are equally reproducible,
  so stability alone is not evidence of memorisation.
- Canary members are ranked above canary non-members (loss AUC 0.678 to 0.741), yet thresholds
  calibrated on the regular population flag none of them. The control confirms ranking
  sensitivity, not threshold-level detection.

Full tables: [docs/results.md](docs/results.md). Every number comes from the CSV files in
[`results/`](results).

## Repository layout

```
src/
  prepare_data.py    build the fixed pools and the member / non-member / reference / test split
  common.py          data loading, Dirichlet partition, MLP
  fl_app.py          Flower ClientApp and ServerApp, FedAvg, FedProx, SCAFFOLD, FedAdam, checkpointing
  run_experiment.py  one federated training run
  attacks.py         the five membership scores for one run
  run_grid.sh        the full 72-run grid (skips runs already done)
  make_canary.py     positive-control pool with flipped-label canaries
  analyze.py         AUC, TPR at 1%/5% FPR, exposed records, union, stability, per class, per client
  canary_report.py   positive-control summary
  report.py          LaTeX tables and figures used in the paper
partitions/          the exact client partitions used in the paper (seed 42)
results/             derived results (CSV, LaTeX tables, figures)
docs/                methodology, data preparation and result tables
```

## Reproduce

Tested with Python 3.10 on CPU (Ubuntu). One run takes about one minute on 2 CPU cores; the full
grid takes a little over one hour.

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

export WORK=$HOME/fedids_work
mkdir -p $WORK/logs && cp -r partitions $WORK/

cd src
# 1. Pools (see docs/data.md for the sources)
python prepare_data.py --dataset cicids2017 --raw <dir with the 8 CIC-IDS2017 parquet files> --out $WORK/data/cicids2017.npz
python prepare_data.py --dataset ciciot2023 --raw <dir with train.parquet> --out $WORK/data/ciciot2023.npz

# 2. Pilot: CIC-IDS2017, 10 clients, FedAvg, loss attack
python run_experiment.py --work $WORK --dataset cicids2017 --alpha 0.5 --agg fedavg --seed 0
python attacks.py --work $WORK --dataset cicids2017 --alpha 0.5 --agg fedavg --seed 0

# 3. Full grid (72 runs)
./run_grid.sh

# 4. Positive control
python make_canary.py --work $WORK
DATASETS=cicids2017_canary SEEDS=0 ALPHAS=0.5 ./run_grid.sh

# 5. Metrics, tables and figures
python analyze.py --work $WORK --out ../results
python canary_report.py --work $WORK --out ../results/canary.csv
python report.py --work $WORK --res ../results
```

`run_grid.sh` accepts `DATASETS`, `ALPHAS`, `AGGS` and `SEEDS` to run a subset. Model
checkpoints (every 10 rounds) and client updates (rounds 10, 50, 100) are written to `$WORK/runs`
and are not versioned (about 250 MB).

## Result files

| File | Content |
|---|---|
| `runs.csv` | one row per run, attack and round: accuracy, macro F1, AUC, TPR and FPR at 1% and 5% |
| `union.csv` | union of the five attacks, descriptive and recalibrated to 1% global FPR |
| `stability.csv` | Jaccard of exposed members across seeds, non-member Jaccard and chance level |
| `per_class.csv` | AUC, TPR and FPR per class |
| `per_client.csv` | exposed fraction per client against the class-matched FPR |
| `identified_samples.csv.gz` | pool index of every member exposed at 5% FPR, flag for 1% FPR |
| `canary.csv` | positive-control metrics |
| `tab_fed_*.tex`, `fig_fed_*.pdf` | tables and figures of the paper |

## Limitations

Class-capped subsets of both datasets; random (not temporal) split of CICIoT2023; a small MLP;
three seeds over one partition per alpha; passive attacks without shadow models; no differential
privacy. Conclusions apply to the configurations executed.

See [docs/methodology.md](docs/methodology.md) for the attack definitions and threshold
calibration.

## Citation

See [CITATION.cff](CITATION.cff).

## License

MIT, see [LICENSE](LICENSE). CIC-IDS2017 and CICIoT2023 are distributed by the Canadian Institute
for Cybersecurity under their own terms and are not included here.
