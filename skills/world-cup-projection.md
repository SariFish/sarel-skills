# World Cup Projection Skill

Rules for developing, debugging, and extending the [world-cup-model](https://github.com/SariFish/World-Cup-model) projection model. Use alongside the **workflow**, **debug**, and **zoom-out** skills.

---

## Model Mental Model

The stack is: **FIFA ratings → Dixon-Coles MLE → score matrix → outcome probabilities → Monte Carlo bracket**.

| Layer | File | What can go wrong |
|-------|------|-------------------|
| Data | `data/fixtures_2026.csv` | Stale scores; team name mismatch with API |
| Data | `data/teams_2026.csv` | Outdated FIFA points; wrong confederation |
| Model fit | `src/dixon_coles.py` | Optimiser non-convergence; identifiability |
| Projection | `src/projections.py` | Score matrix not summing to 1; rho out of range |
| ELO fallback | `src/elo.py` | Draw probability formula mis-calibrated |
| Tournament | `src/knockout_sim.py` | Bracket pairing; same-group rematch; sampling bias |

---

## Updating Scores

```bash
python data/fetch_data.py --api-key YOUR_KEY
python run_projections.py
```

The key is sent in the `X-Auth-Token` header. The fetcher self-throttles on the
`X-Requests-Available-Minute` header and retries once on HTTP 429.

If `fetch_data.py` returns no matches, check the diagnostic it prints:
1. **`blocked by network egress policy`** — the host `api.football-data.org` is not on
   this environment's allowlist. Add it in the environment's egress settings, or run
   the fetcher on a machine with open network and commit the updated CSV.
2. **`403 — check your API key`** — key invalid or unverified at football-data.org.
3. **`404 — competition not found`** — re-run with `--competition <CODE>`.
4. **Team name mismatch** — add entries to `api_name_to_csv_name()` in `fetch_data.py`.

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

### Knockout-round simulation (IMPLEMENTED)
`src/knockout_sim.py` + `run_projections.py --tournament`:
1. Fixes already-played group results, simulates the remaining group games.
2. Ranks each group (points → GD → GF); takes top 2 + 8 best third-placed teams.
3. Builds a strength-seeded Round of 32 that avoids same-group rematches.
4. Plays the bracket, sampling scorelines from the fitted model (shootout = 50/50 on draws).
5. Aggregates over N sims → P(reach R16/QF/SF/Final/Win).

Known simplification: the exact official 2026 bracket slotting (which group each
best-third feeds into) is approximated by a seeded pairing. Refine `build_round_of_32()`
once the official bracket map is confirmed.

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
