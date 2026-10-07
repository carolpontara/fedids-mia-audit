# Results

All numbers below are read from `results/runs.csv` (round 100). Each cell averages three alpha values and three seeds (9 runs per dataset and aggregator).

## cicids2017

| Aggregator | Macro F1 | AUC loss | AUC refgap | AUC gradnorm | AUC gradalign | AUC trajectory | max TPR@1% FPR | max TPR@5% FPR |
|---|---|---|---|---|---|---|---|---|
| fedadam | 0.967 | 0.5038 | 0.5001 | 0.5038 | 0.4970 | 0.5040 | 1.17% | 5.59% |
| fedavg | 0.964 | 0.5039 | 0.4969 | 0.5040 | 0.4962 | 0.5041 | 1.14% | 5.58% |
| fedprox | 0.964 | 0.5039 | 0.4969 | 0.5041 | 0.4962 | 0.5042 | 1.19% | 5.61% |
| scaffold | 0.974 | 0.5042 | 0.4977 | 0.5042 | 0.4977 | 0.5048 | 1.24% | 5.59% |

| alpha | AUC loss (mean over aggregators and seeds) |
|---|---|
| 0.1 | 0.5034 |
| 0.5 | 0.5043 |
| 1.0 | 0.5042 |

## ciciot2023

| Aggregator | Macro F1 | AUC loss | AUC refgap | AUC gradnorm | AUC gradalign | AUC trajectory | max TPR@1% FPR | max TPR@5% FPR |
|---|---|---|---|---|---|---|---|---|
| fedadam | 0.683 | 0.5116 | 0.5094 | 0.5103 | 0.5006 | 0.5084 | 1.43% | 6.25% |
| fedavg | 0.679 | 0.5097 | 0.5044 | 0.5089 | 0.5002 | 0.5078 | 1.31% | 5.74% |
| fedprox | 0.678 | 0.5097 | 0.5042 | 0.5088 | 0.5002 | 0.5078 | 1.32% | 5.80% |
| scaffold | 0.661 | 0.5095 | 0.5040 | 0.5085 | 0.4997 | 0.5077 | 1.35% | 5.54% |

| alpha | AUC loss (mean over aggregators and seeds) |
|---|---|
| 0.1 | 0.5098 |
| 0.5 | 0.5101 |
| 1.0 | 0.5105 |

