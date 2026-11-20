# Hosting checklist (Kaggle community competition)

Kaggle's menus change; verify wording in the current UI. What must be uploaded / configured:

1. **Competition data** (participants see these): everything in `dataset/public/` (5 CSVs). `dataset/public.zip` is the same.
2. **Solution file** (host only): `dataset/host_private/solution.csv` with columns `id, target, Usage`
   (`Usage` = `Public` / `Private`, constant per test town; 18 public / 42 private towns).
3. **Sample submission**: `dataset/public/sample_submission.csv` (`id,target`, constant 0.5).
4. **Evaluation metric**: Area Under ROC Curve (higher is better), applied to `target`.
5. **Never upload**: `host_private/*`, `kaggle_reference_solution_ORGANIZERS_ONLY.ipynb`, `tools/headroom.py`, `tools/bench_*.py` (they read the solution).
6. **Starter**: publish `kaggle_starter.ipynb` as a public notebook / attach to the competition. It auto-finds `/kaggle/input/<slug>/train_nodes.csv`.
7. **Before launch (do all of these):**
   - [ ] Run `kaggle_reference_solution_ORGANIZERS_ONLY.ipynb` once on Kaggle (it is the only place the PyTorch cells have never run in my tests) and upload its `submission.csv`.
         Expected: public ≈ 0.78–0.79, private ≈ 0.80, all ≈ 0.80 (my run used a numpy GCN stand-in: 0.7876 / 0.8038 / 0.7990).
   - [ ] Upload `sample_submission.csv` as a submission: must score exactly 0.5.
   - [ ] Upload a deliberately shuffled-id / missing-row file: Kaggle must reject it (mirrors `score_submission.py`).
   - [ ] Check the starter on a fresh session with the HP cell untouched: it must stop with the "Fill in these hyper-parameters" message.
   - [ ] Run `python tools/validate_kaggle_dataset.py dataset` (30 tests) after any change to the data.
8. **Suggested limits**: ≤ 5 submissions/day; final-submission count 2; treat leaderboard gaps < 0.005 as ties (see noise note).

## Rebuilding / auditing
```
cd tools
python make_kaggle_dataset.py --out ../dataset      # deterministic (seeds fixed); MANIFEST.json holds sha256 of every file
python validate_kaggle_dataset.py ../dataset         # 30 tests, exit code 1 on any failure
python score_submission.py my_submission.csv ../dataset/host_private/solution.csv
python benchmark.py ../dataset                       # leaderboard ladder (sklearn only)
```
