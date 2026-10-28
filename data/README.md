# Data

Raw CSVs are **not** committed (see `.gitignore`); each notebook downloads its
dataset from a public mirror into this folder on first run.

| File | Source (public mirror) | Protected attr(s) | Target |
|---|---|---|---|
| `credit_german.csv` | UCI Statlog *German Credit* | `personal_status_sex`, `age` | `credit_risk` |
| `health_medcost.csv` | *Medical Cost Personal* | `sex`, `region`, (`smoker`) | `charges` |
| `fraud_autoclaims.csv` | *Auto Insurance Claims* (fraud-labeled) | `insured_sex`, `age` | `fraud_reported` |

Dataset licenses vary and some are unstated; verify terms before redistribution.
