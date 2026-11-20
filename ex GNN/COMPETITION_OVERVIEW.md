# Hidden Carriers: Who Should Be Quarantined?

## Overview
An infectious disease spreads through towns made of households, schools and workplaces. On the day contact tracers
look, about **40 % of infected people have no symptoms**: they look healthy but are infectious.
For every person who *looks healthy*, predict the probability that they are **already infected or will be infected within the next 3 days**.
Ranking people well = building a good quarantine priority list.

The lesson of the workshop: a model that only looks at a person (0 hops) is blind; a rule that checks for sick neighbours (1 hop) is
decent; a model that reasons through **hidden carriers** (2+ hops: visibly sick → hidden carrier → you) does best.

## Evaluation
Submissions are scored by **ROC-AUC** over all rows of `sample_submission.csv` (pooled over towns).
Only people with `observed == 0` appear in the submission. Any real-valued score is accepted (it is only ranked).
Public leaderboard = 18 test towns, private leaderboard = the other 42 (split by town, fixed in advance).
Expect ±0.005–0.01 noise between close solutions: validate with graph-grouped cross-validation, not with the leaderboard.

### Submission format
```
id,target
109_0,0.231
109_1,0.007
```
`id = "<graph_id>_<node_id>"`, `target` = your score. Exactly one row per scored person, no duplicates, no missing or non-finite values.

## Data
Each `graph_id` is one independent town (500 people) with its own outbreak. Split is **by town**: train towns and test towns never overlap.

| file | content |
|---|---|
| `train_nodes.csv` | `graph_id, node_id, observed, n_contacts, contact_intensity, community, target` |
| `test_nodes.csv` | same without `target` |
| `train_edges.csv`, `test_edges.csv` | `graph_id, src, dst, weight` — undirected contacts, one row per pair (`src < dst`), `weight` in [0.2, 1] |
| `sample_submission.csv` | `id, target` (constant 0.5 → AUC 0.5) |

* `observed`: 0 = looks healthy (**scored**: susceptible or hidden carrier), 1 = visibly infected, 2 = recovered.
* `n_contacts`: number of contacts. `contact_intensity`: sum of the edge weights of the person. **Missing for ~5 %** (incomplete tracing).
* `community`: household/school/work block id 0–4 (arbitrary labels per town). **Missing for ~5 %.**
* `target` (train only, only where `observed == 0`): 1 if the person is a hidden carrier or becomes infected within 3 days, else 0 (~12 % positives).
* Node ids are shuffled inside every town: ids carry no information.

## Rules / tips
* Never split validation by node; use `GroupKFold(groups=graph_id)`.
* The test graphs may be used for unsupervised feature building (they are given), but test labels do not exist in any public file.
* Reference ladder (pooled AUC): constant 0.50 · own features only ≈ 0.59 · 1-hop heuristic ≈ 0.68 · graph features / GCN ≈ 0.79–0.80.
