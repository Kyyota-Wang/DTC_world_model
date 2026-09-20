# A Patient World Model for Early Forecasting of Digital Health Campaign Outcomes

**Capabilities and Limits**

Yunlong Wang — Advanced Analytics, IQVIA (Wayne, PA, USA)

📄 Paper: [`world_model_arxiv.pdf`](world_model_arxiv.pdf) (arXiv preprint, September 2026)

---

## Overview

Digital direct-to-consumer (DTC) health campaigns are usually measured after the fact. In-flight forecasting typically trains a separate classifier for every (cutoff, horizon) pair, which produces incoherent cumulative curves and cannot simulate alternative exposure plans.

This paper treats in-flight campaign measurement as a **dynamic-system problem** and builds a compact, intervention-conditioned **patient world model**:

- a latent state per patient, advanced weekly by a sequence-state module (a two-layer GRU here, but swappable);
- a **survival hazard head** that outputs a weekly conversion hazard;
- an auxiliary **transition head** that predicts next week's exposure summary (dense supervision);
- a **recursive survival rollout** that multiplies weekly hazards into cumulative-incidence curves that are monotone and consistent across all horizons by construction.

The model is small (~2 × 10⁵ parameters) and trains on CPU inside the certified environment where patient data must stay.

## Headline results

Evaluated on a US digital DTC campaign for a prescription cardiovascular drug: 147,173 patients (stratified case-control cohort from a 10.9M eligible universe), 5.2M at-risk person-weeks, 70/15/15 split by patient hash.

**Task:** from an early cutoff week τ, forecast the remaining new-to-brand prescription (NBRx) volume through week 52, conditioned on recorded future exposures.

| Model | τ=4 | τ=8 | τ=13 | τ=26 | Coherent |
|---|---|---|---|---|---|
| Per-(τ,H) GBM (common practice) | 2582.0% | 3054.8% | 3754.7% | 8491.6% | 19–24% |
| Naive extrapolation | 64.4% | 82.6% | 97.4% | 166.1% | 100% |
| Pooled-hazard GBM + rollout | 13.6% | 17.3% | 19.2% | 33.1% | 100% |
| GRU forecaster (λ=0 ablation) | 5.2% | 7.4% | 8.2% | 11.1% | 100% |
| **GRU world model (proposed)** | **2.9%** | **2.3%** | **2.6%** | **0.8%** | 100% |

Relative error of the forecast final converter count on the test split. At these cutoffs the 1σ sampling floor is 3.7–7.0%, so the world model is statistically indistinguishable from a perfectly calibrated forecaster.

## Key findings

1. **Structure matters.** Coherence and additive (not multiplicative) error propagation come from the survival rollout, not the network. Any pooled-hazard model with the rollout beats per-horizon classifiers by orders of magnitude.
2. **Dense supervision helps in the rare-event regime.** A Fisher-information argument shows the outcome loss supplies only ~1% of the encoder's gradient curvature when events are rare. Removing the transition head (a one-flag ablation) increases NBRx volume error by roughly 2–14×, with no consistent loss on the ten-times-more-common specialist-visit outcome.
3. **Scenario simulation has limits.** Within observed dose support the simulator behaves sanely (monotone, saturating, small dose-response). Switching all future exposure *off* raises predicted conversion from 0.31 to 0.89, a selection artifact caused by outcome-dependent censoring. Exposure-conditioned rollouts must not be read as causal effects.

## Repository status

| Item | Status |
|---|---|
| Paper PDF | ✅ included |
| Model and evaluation code | 🔜 coming soon |
| Data | ❌ cannot be shared |

The underlying data are protected health information and stay on a secured analytics cluster. Only aggregate metrics are reported. Cohort construction rules, feature definitions, hyperparameters, and full as-run result tables are given in Appendices A–G of the paper at a level of detail sufficient to rebuild the pipeline on comparable data.

## Training details (summary)

- PyTorch, CPU training
- Tuned config: 2 GRU layers, hidden size 128, dropout 0.2; base config: 1 layer, hidden 64
- Loss: discrete-time survival NLL + λ · next-exposure MSE, λ = 0.3
- Adam, lr 1e-3, ReduceLROnPlateau (factor 0.5, patience 2), early stopping (patience 10), ≤50 epochs, batch size 512
- Baselines: LightGBM (per-cell and pooled-hazard), scikit-learn logistic regression and MLP, all at library defaults

## Responsible use

This work forecasts observed health-seeking behavior under recorded digital exposure. It does not estimate causal effects. Outputs are decision support for campaign measurement and planning, and are not intended for individual-level clinical or coverage decisions. Results come from one campaign, one therapeutic area, and one country; external validity is untested.

## Citation

```bibtex
@article{wang2026patientworldmodel,
  title   = {A Patient World Model for Early Forecasting of Digital Health Campaign Outcomes: Capabilities and Limits},
  author  = {Wang, Yunlong},
  journal = {arXiv preprint},
  year    = {2026}
}
```
