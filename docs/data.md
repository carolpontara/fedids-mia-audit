# Data

Raw datasets are not redistributed. Download them from their sources and run `src/prepare_data.py`.

| Dataset | Source used | Pool | Features | Classes |
|---|---|---|---|---|
| CIC-IDS2017 (Sharafaldin et al., 2018) | Hugging Face `Ariasyah/cic-ids-2017`, the 8 daily parquet files | 48,091 records | 68 | 7 (Benign, DoS, DDoS, PortScan, BruteForce, Bot, WebAttack) |
| CICIoT2023 (Neto et al., 2023) | Hugging Face `casscloud/CIC-IoT-2023`, `train.parquet` | 62,000 records | 38 | 8 |

Preprocessing (`prepare_data.py`):

1. Strip column names, coerce to numeric, drop rows with NaN or infinite values, drop duplicates.
2. CIC-IDS2017 labels are grouped into 7 classes; Infiltration and Heartbleed (under 40 records) are dropped.
3. Benign is capped at 20,000 records and each attack class at 6,000.
4. A single stratified split (seed 42): members 40%, non-members 20%, reference 20%, test 20%.
5. Signed log1p transform, near-constant features removed, standardisation with reference-split
   statistics, clipping to [-10, 10].

The split of CICIoT2023 is random, not temporal. The client partitions used in the paper are in
`partitions/` (Dirichlet label skew of the member set, seed 42, 10 clients).
