# World Cup Projection Skill

Rules for developing, debugging, and extending the [world-cup-model](https://github.com/SariFish/World-Cup-model) projection model. Use alongside the **workflow**, **debug**, and **zoom-out** skills.

---

## Model Mental Model

The stack is: **FIFA ratings → Dixon-Coles MLE → score matrix → outcome probabilities**.

| Layer | File | What can go wrong |
|-------|------|-------------------|
| Data | `data/fixtures_2026.csv` | Stale scores; team name mismatch with API |
| Data | `data/teams_2026.csv` | Outdated FIFA points; wrong confederation |
| Model fit | `src/dixon_coles.py` | Optimiser non-convergence; identifiability |
| Projection | `src/projections.py` | Score matrix not summing to 1; rho out of range |
| ELO fallback | `src/elo.py` | Draw probability formula mis-calibrated |

---

## Updating Scores

```bash
python data/fetch_data.py [--api-key YOUR_KEY]
python run_projections.py
```

If `fetch_data.py` returns no matches, check:
1. API key valid? (free key at football-data.org)
2. Competition code correct? May need to update `COMPETITION` constant in `fetch_data.py`.
3. Team name mismatch? Add entries to `api_name_to_csv_name()` in `fetch_data.py`.

---

## Debugging the Dixon-Coles Fit

Apply the **debug** skill. Common hypotheses in ranked order:

1. **Not enough games** — DC needs ≥15 completed games; ELO fallback is used below that.
   - Prediction: `run_projections.py` prints "ELO fallback".
2. **Non-convergence** — optimiser hits iteration limit.
   - Prediction: `params['converged'] == False`.
   - Fix: increase `maxiter`; check for teams that appear in only 1 game (sparse data).
3. **rho out of range** — correction factor τ goes negative → log(negative) = NaN.
   - Prediction: `_neg_ll` returns 1e10 during optimisation.
   - Fix: verify rho bound in `minimize()` is `(-0.5, 0.1)`.
4. **Team name missing from teams_df** — `set_fifa_prior()` assigns 0 for unknown teams.
   - Prediction: one team has `att_init == 0.0` despite being a strong team.
   - Fix: add team to `teams_2026.csv` or fix spelling to match fixture CSV.

### Quick smoke test
```bash
python -m src.dixon_coles   # runs _test_fit() on synthetic data
```

---

## Extending the Model

### Add a new competition / data source
1. Fetch data into `data/fixtures_2026.csv` format (same columns).
2. `data_loader.py` reads whatever is in the CSV — no other changes needed.

### Add knockout-round simulation
1. After group stage, determine qualified teams (top 2 per group + best 8 third-place).
2. Run Monte Carlo simulation: for each bracket match, sample a score from the fitted score matrix.
3. Aggregate over N=10000 simulations to get P(team reaches QF/SF/Final/wins).
4. Add `src/knockout_sim.py` and wire it into `run_projections.py --tournament`.

### Add player-level features
1. Extend `teams_2026.csv` with `avg_age`, `key_player_unavailable` (0/1), etc.
2. In `dixon_coles.py`, add these as additional linear terms in the log-lambda equation.
3. Refit with the expanded feature vector.

---

## Validation Targets

| Metric | Acceptable | Good |
|--------|-----------|------|
| Mean Brier score | < 0.50 | < 0.40 |
| Log-loss | < 1.0 | < 0.9 |

Run: `python run_projections.py --validate`

---

## Zoom-Out Triggers

Use the **zoom-out** skill if:
- Projections look obviously wrong (e.g., 95% win for a weak team) — map the data pipeline before changing model code.
- Iterating on `dixon_coles.py` for >3 rounds without improvement — zoom out to check if the bug is actually in `data_loader.py`.
