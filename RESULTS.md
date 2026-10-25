# HAPS Link-Quality Prediction (Team FLUX, A3-PS018-TC130)

Hybrid physics + machine-learning prediction of SNR for a High Altitude
Platform Station (HAPS) link at 28 GHz, 20 km altitude, with two scenarios:
clear sky and monsoon rain.

## What it does
1. **Generate** a seeded synthetic dataset: free-space loss, gas, cloud,
   rain, scintillation and pointing jitter, then the link budget gives SNR.
2. **Predict** with two models:
   - Baseline: free-space loss plus one fixed average atmospheric loss
     (4.04 dB, learned from training data).
   - Hybrid: baseline plus a LightGBM-learned correction.
3. **Trust**: 90% prediction band from quantile regression plus
   split-conformal calibration, computed separately for each scenario
   and elevation group.

## Link budget assumptions
| Parameter | Value |
|---|---|
| Carrier frequency | 28 GHz |
| Bandwidth | 100 MHz |
| Tx power | 10 W (40 dBm) |
| HAPS antenna gain | 30 dBi |
| User antenna gain | 35 dBi |
| Noise floor | -90 dBm |
| HAPS altitude | 20 km |
| Footprint radius | 50 km |

## Dataset (seed = 42)
- Total rows: **18,000** (300 trajectories x 60 time steps)
- 150 trajectories per scenario (9,000 rows each)
- Elevation range: **21.8 to 87.1 degrees**

| Scenario | Mean SNR (dB) | Std (dB) | Min (dB) | Median (dB) | Max (dB) |
|---|---|---|---|---|---|
| clear_sky | 43.0 | 2.8 | 36.7 | 43.2 | 48.2 |
| monsoon_rain | 33.2 | 12.7 | -67.0 | 36.1 | 46.5 |

Rain lowers the average SNR by about 10 dB and widens the spread from
2.8 dB to 12.7 dB. The extreme minimum (-67 dB) comes from rare, very
heavy rain bursts in the simulated rain-rate series.

## Validation
- Split by whole trajectory (never by row): 60% train, 20% calibration,
  20% test.
- The **model** is trained only on elevations of 30 degrees and above, so
  the band below 30 degrees is unseen by the model.
- The **calibration** set includes all elevations (different trajectories
  from the test set), and the conformal adjustment is computed separately
  for each scenario and elevation group.

## Results (held-out trajectories)

### All held-out trajectories
| Scenario | Rows | Baseline MAE (dB) | Hybrid MAE (dB) | MAE reduction | 90% band coverage | Avg band width (dB) |
|---|---|---|---|---|---|---|
| clear_sky | 2,220 | 3.45 | 0.35 | 90% | 91.2% | 1.63 |
| monsoon_rain | 1,380 | 5.61 | 1.58 | 72% | 87.6% | 6.30 |

### Unseen low-elevation band (below 30 degrees)
| Scenario | Rows | Baseline MAE (dB) | Hybrid MAE (dB) | MAE reduction | 90% band coverage | Avg band width (dB) |
|---|---|---|---|---|---|---|
| clear_sky | 757 | 3.30 | 0.41 | 88% | 92.1% | 2.08 |
| monsoon_rain | 515 | 8.48 | 2.61 | 69% | 85.6% | 10.46 |

### What these numbers say
- **MAE target met:** the hybrid model beats the analytical baseline in
  both scenarios, on all held-out trajectories and in the unseen band.
- **Coverage target met in clear sky:** 91.2% overall and 92.1% in the
  unseen band.
- **Coverage target nearly met in monsoon rain:** 87.6% overall and 85.6%
  in the unseen band, just below 90%. The bands are wider in rain
  (6.30 dB, and 10.46 dB at low elevation) because rain loss is
  heavy-tailed and hard to predict.
- **Earlier version:** using one shared adjustment and a calibration set
  without low elevations gave monsoon coverage of 75.9% (58.1% in the
  unseen band). Grouped calibration fixed most of that gap.
- **Next step:** more trajectories per scenario, or a finer elevation
  grouping, to close the last 2 to 4 points.

## How to rerun
1. Open `kaushik_hybrid_model.ipynb` in Google Colab (File > Upload notebook).
2. Click Runtime > Run all.
3. The seed is fixed (42), so results repeat exactly.

## Files
- `kaushik_hybrid_model.ipynb`: dataset, models, charts
- `metrics.csv`: MAE and coverage table
- `chart1_trajectory.png`: true vs baseline vs hybrid SNR with 90% band
- `chart2_elevation.png`: SNR vs elevation, showing the unseen band
- `chart3_mae.png`: baseline vs hybrid error

## Limitations (honest scope)
- Synthetic, model-based data only. No real-network validation is claimed.
- Rain, gas and cloud losses use simplified ITU-R-style formulas,
  not the full ITU-Rpy implementation.
- Monsoon-rain interval coverage is slightly below the 90% target.
- Sensitivity analysis (tornado and Sobol) is done separately by Srinidhi.

## Team
Srinidhi Shetty (Leader): RF and channel lead.
Kaushik: data and ML lead.
