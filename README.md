# Pitch Quality Model (Stuff+ / Location+)

A reproducible pipeline that estimates the run value of a pitch from its **physical characteristics** (Stuff) and its **location in context** (Location), using only public Statcast data.

> This repository documents a *method*: how the problem is framed, how leakage is avoided, and how the output is validated. It is a baseline implementation, not a production model. Trained model artifacts are intentionally not versioned here.

## Motivation

Outcome stats (ERA, FIP, even xwOBA against) are noisy on small samples. Pitch-level models that use only what the pitcher controls (velocity, spin, movement, release) stabilize much faster and are useful for evaluating relievers, call-ups, and pitch-mix changes.

## Approach

| Step | Decision |
|---|---|
| **Data** | Public Statcast pitch-level data via [`pybaseball`](https://github.com/jldbc/pybaseball), regular season only |
| **Target** | Run value to the pitcher: `-delta_run_exp` (change in run expectancy on each pitch) |
| **Stuff features** | Release speed, spin rate, spin axis (sin/cos), movement (`pfx_x`, `pfx_z`), release point, extension, and differences vs. the pitcher's own fastball |
| **Location features** | Plate location (batter-relative), count, platoon matchup |
| **Handedness** | Horizontal quantities are mirrored so left- and right-handed pitchers share one frame of reference |
| **Model** | Gradient-boosted trees (LightGBM), one model per pitch group (fastball / breaking / offspeed) |
| **Validation** | **Temporal split**: train on earlier seasons, test on a later one. No random pitch-level splits |
| **Scaling** | Output scaled to 100 = average, +10 per standard deviation, higher is better |

### Why temporal validation?

Random splits let the model see the same pitcher, same game, and same season in both train and test, which inflates performance. Training on the past and testing on a future season mimics how the model would actually be used.

## Evaluation

Two questions matter more than raw error:

1. **Does it beat a naive baseline?** (RMSE vs. predicting the training mean)
2. **Is the pitcher-level score reliable?** Split-half reliability (with Spearman-Brown correction) tells you how quickly the metric stabilizes compared with outcome stats.

### Results

_Fill in after running the pipeline. Report numbers for a held-out season, not the training seasons._

| Metric | Value |
|---|---|
| Test season | TBD |
| RMSE (model) | TBD |
| RMSE (baseline) | TBD |
| Split-half reliability (Stuff, min. 100 pitches) | TBD |

## Repository structure

```
pitch-quality-model/
├── README.md
├── pyproject.toml
├── requirements.txt
├── .gitignore
├── src/pitchquality/
│   ├── __init__.py
│   ├── data.py        # download + cache Statcast seasons
│   ├── features.py    # cleaning, handedness mirroring, target, feature sets
│   ├── model.py       # per-pitch-group LightGBM + plus-scale conversion
│   ├── evaluate.py    # temporal split, RMSE, split-half reliability
│   └── pipeline.py    # end-to-end command line entry point
├── tests/
│   └── test_features.py
├── notebooks/         # exploratory analysis (add your own)
├── data/              # local only, git-ignored
├── models/            # local only, git-ignored
└── outputs/           # scores and figures
```

## Quickstart

```bash
git clone https://github.com/<your-user>/pitch-quality-model.git
cd pitch-quality-model
python -m venv .venv && source .venv/bin/activate
pip install -e .
pip install -r requirements.txt

# Download 2021-2024, train on 2021-2023, test on 2024
python -m pitchquality.pipeline --start-year 2021 --end-year 2024 --test-year 2024
```

Outputs are written to `outputs/pitcher_scores_<test_year>.csv`.

Run the tests:

```bash
pytest
```

## Limitations

- **Not a causal model.** It estimates the value associated with pitch characteristics, not what would happen if a pitcher changed them.
- **No sequencing or tunneling.** Each pitch is scored independently of the previous one.
- **Fastball reference is per pitcher-season.** Differences vs. a pitcher's own fastball use season averages, which is a mild form of look-ahead within a season.
- **Run expectancy is noisy.** The target is high-variance at the single-pitch level; value comes from aggregating many pitches.
- **Public data only.** Tracking-system updates across seasons can shift measurements slightly.

## Roadmap

- [ ] Add arm angle as a feature
- [ ] Calibrate scores per pitch type
- [ ] Add uncertainty intervals for pitcher-level scores
- [ ] Compare against published pitch-quality metrics
- [ ] Notebook walking through error analysis by pitch type

## Data and licensing

Pitch data comes from MLB Statcast through Baseball Savant. Review their terms of use before redistributing any data; this repository does not include raw data.

## Author

**Gibrán** · Mechatronics Engineering · Baseball analytics and performance science
