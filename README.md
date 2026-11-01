# A3-PS018-TC130 | Team FLUX

**AIORI Hackathon 3** · **PS-018: HAPS Link-Quality Prediction Using a Documented Channel Model** · 6G & Future Networks

Introduction video: https://youtu.be/hWjJDWScCs0

## Team

| Role | Name | Email |
|---|---|---|
| Team Leader (RF and channel lead) | Srinidhi Shetty | iamshettysrinidhi@gmail.com |
| Team Member (data and ML lead) | Kaushik | acharyakaushik9535@gmail.com |

**Faculty mentor:** Dr. Dinesha P (drdinesh-ece@dayanandasagar.edu), Dayananda Sagar College of Engineering, Bengaluru

## Problem

High-Altitude Platform Stations (HAPS) fly at about 20 km and offer wide-area broadband, but received power and SNR vary by tens of dB with elevation, slant range, atmospheric gases and rain. We build a reproducible, model-based dataset for a fixed HAPS geometry, predict received power / SNR, and compare the predictor against an analytical baseline on held-out trajectories.

**Scope:** one band (28 GHz), one altitude (20 km), two scenarios (clear sky, monsoon rain). All data is synthetic and model-based; there is no real-network validation.

## Approach

1. **Generate (ground truth):** layered ITU-R model (free-space loss, gases, rain, cloud, scintillation, pointing jitter) with seeded stochastic rain.
2. **Predict:** deterministic analytical link budget as the baseline; a gradient-boosted residual model (SNR = baseline + learned correction) as the proposed predictor.
3. **Trust:** quantile regression with split-conformal calibration for 90% prediction bands.
4. **Evaluate:** whole held-out trajectories, including an unseen low-elevation band; MAE against the baseline, interval coverage, and a sensitivity ranking of key assumptions.

## Design assumptions (link budget)

| Parameter | Value |
|---|---|
| Carrier frequency | 28 GHz |
| Bandwidth | 100 MHz |
| HAPS altitude | 20 km |
| Tx power | 10 W (40 dBm) |
| HAPS antenna gain | 30 dBi |
| User antenna gain | 35 dBi |
| Noise figure | 4 dB |
| Noise floor | -90 dBm |

These are our documented starting values and may be refined with mentor feedback.

## Architecture

![Architecture](docs/architecture.png)

![Data flow](docs/dfd.png)

## Repository layout

```
README.md
A3-PS018-TC130-Team-FLUX-Presentation.pdf   Level 1 presentation
config/       scenario settings and seeds (YAML)
src/          geometry, channel, link budget, dataset, baseline, model, evaluation, sensitivity
app/          dashboard
results/      metrics and figures
docs/         report, pseudocode, diagrams
```

## Run

```
pip install -r requirements.txt
bash run_all.sh        # generates data, trains, evaluates, writes results/
```

Commands are finalised as the code lands. Every run is seeded and reproducible.

## Status

| Item | Status |
|---|---|
| Level 1 (repo, presentation, intro video) | Done |
| Channel engine, link budget, baseline | In progress |
| Dataset generator and trajectory split | In progress |
| Hybrid model, prediction bands, sensitivity analysis | Planned |
| Dashboard and live demo | Planned (due 31 Oct) |
| Technical report | Planned (due 31 Oct) |

## Demo

Link to be added.

## References

ITU-R P.528, P.525, P.676, P.838, P.618, P.840; 3GPP TR 38.811; Koenker and Bassett (1978); Angelopoulos and Bates (2021); Ke et al. (2017); ITU-Rpy.
