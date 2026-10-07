# Methodology

| Element | Choice |
|---|---|
| Data | Deduplicated pool per dataset, Benign capped at 20,000 records and each attack class at 6,000, split once, stratified, into member (40%), non-member (20%), reference (20%) and test (20%) sets. Features: signed log1p, standardised with reference statistics, clipped to [-10, 10]. CIC-IDS2017 labels grouped into 7 classes (Infiltration and Heartbleed dropped, under 40 records). |
| Federation | 10 clients, full participation, 100 rounds, 1 local epoch, SGD lr 0.05, batch 32, MLP 128-64. |
| Partition | Dirichlet label skew of the member set, partition seed 42, identical for every aggregator and every training seed (`partitions/*.npz`). |
| Aggregators | FedAvg; FedProx (mu = 0.01); SCAFFOLD option II (custom strategy on Flower FedAvg, client control variates kept in the Flower node state); FedAdam (server lr 0.01, tau 1e-3). Only the aggregator changes between cells. |
| Seeds | 0, 1, 2 (model initialisation and client batch order). |
| Stored artefacts | Global model every 10 rounds; broadcast model and every client's local model at rounds 10, 50 and 100 (`updates_r*.npz`). |
| Attacks | `loss`, `refgap` (reference model trained on the reference split), `gradnorm`, `gradalign` (server-side cosine between the per-sample descent direction and each client update), `trajectory` (mean standardised score over rounds 10 to 100). See `attacks.py`. |
| Calibration | Thresholds from non-member scores only; a record is exposed at rate a if its score is at least the threshold giving FPR <= a. |
| Outputs | AUC, TPR at 1% and 5% FPR, identities of exposed records, fraction of the training data exposed, union of attacks (descriptive and Bonferroni-recalibrated to a 1% global FPR), Jaccard stability across seeds against its chance level, per-class TPR with FPR, per-client exposure against the class-matched FPR. |
| Positive control | 200 member and 200 non-member records with flipped labels (`make_canary.py`), CIC-IDS2017, alpha = 0.5, seed 0, the four aggregators, same pipeline. |

## Membership score

All attacks are built on the log-odds of the true class,

    phi(x) = z_y - log sum_{j != y} exp(z_j)

where z are the logits of the model under audit. Every score is oriented so that a larger value
means "more likely a member".

| Attack | Score | Observer |
|---|---|---|
| `loss` | phi under the global model at the audited round | anyone with the global model |
| `refgap` | phi(global) - phi(reference model trained on the reference split) | anyone with the global model and population data |
| `gradnorm` | minus log of the per-sample gradient norm under the global model | anyone with the global model |
| `gradalign` | maximum cosine between the per-sample descent direction and each client update | honest-but-curious server |
| `trajectory` | mean of the standardised phi over the global models of rounds 10, 20, ..., 100 | anyone observing the model sequence |

## Threshold calibration

For a target false positive rate a, the threshold is the smallest h such that the fraction of
non-members with score >= h is at most a. Members at or above h are "exposed". The union of the
five attacks is reported twice: descriptively (each attack at a, with its effective FPR) and
recalibrated so that the union has a global FPR of 1% (Bonferroni, a / 5 per attack).

## Stability

For each cell, the sets of members exposed at 5% FPR under seeds 0, 1 and 2 are compared with the
mean pairwise Jaccard index. The same statistic is computed for non-members flagged at the same
threshold and for random sets of the same size (chance level). Member stability is evidence of
record-level memorisation only if it exceeds the non-member stability.
