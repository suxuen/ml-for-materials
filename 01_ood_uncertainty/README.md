# Uncertainty Analysis on B-site Alloy Halide Perovskite dataset


## Motivation

Here we perform uncertainty analyses using regression model(s) on my halide perovskite dataset ([DOI: 10.1021/acs.chemmater.6c00830](https://doi.org/10.1021/acs.chemmater.6c00830)). The purpose is to evaluate model performance on data that differs significantly from the training data, i.e. to test the models' ability to extrapolate accurately (model generalizability).

## Results so far
1. We replicated our random and grouped CV MAE and stdev from our starting point. The resulting MAE and stdev for grouped CV differed slightly likely due to different grouped fold assignments but within one standard deviation of each other.

| HGB | Starting Point | This work |
| ----- | ----- | ----- |
| random CV | 0.074 +/- 0.002 eV | 0.074 +/- 0.004 eV |
| grouped CV | 0.144 +/- 0.023 eV | 0.154 +/- 0.020 eV |

2. We compared Histogram Gradient Boosted (HGB) and Extreme Gradient Boosted (XGB) tree models and found that HGB and XGB resulted in MAEs within one standard deviation from each other. XGB is about 2x faster than HGB, which could be useful for ensemble modeling and larger dataset sizes. We use XGB moving forward.

|  | HGB | HGB time | XGB | XGB time |
| ----- | ----- | -----  | -----  | ----- |
| random CV | 0.074 +/- 0.004 eV | ~4 s | 0.070 +/- 0.003 eV | ~2 s |
| grouped CV | 0.154 +/- 0.020 eV | ~4 s | 0.162 +/- 0.023 eV | ~2 s |

3. We found that changing the seed did not affect XGB's predictions. Because the model cannot be varied by assigning different seeds, ensemble members must be diversified another way. We use bootstrap aggregating (bagging).

| XGB | seed=5 | seed=105 |
| ----- | ----- | ----- |
| random CV | 0.070 +/- 0.003 eV | 0.070 +/- 0.003 eV |
| grouped CV | 0.162 +/- 0.023 eV | 0.162 +/- 0.023 eV |

4. We performed ensemble modeling via bagging. Four experiments were performed: grouped CV with grouped bagging, random CV with grouped bagging, grouped CV with random bagging, random CV with random bagging.

- Random bagging: bootstrapping resamples the training data by rows
- Group bagging: bootstrapping resamples on whole B-pairs

We did 5 ensemble members per CV (n_bootstrap = 5).


| XGB MAE | random bagging | grouped bagging |
| ----- | ----- | ----- |
| random CV | 0.069 +/- 0.004 eV | 0.092 +/- 0.005 eV |
| grouped CV | 0.157 +/- 0.019 eV | 0.184 +/- 0.022 eV |


| XGB spread | random bagging | grouped bagging |
| ----- | ----- | ----- |
| random CV | 0.052 eV | 0.100 eV |
| grouped CV | 0.073 eV | 0.160 eV |

**Grouped CV raises the model's error (MAE) and uncertainty (spread)**. In this experiment, we see that grouped CV increases model spread less than MAE. With grouped bagging, moving from random to grouped CV increased MAE by ~100% but spread by only ~60%. With random bagging, MAE increased by ~128% and spread by ~40%. (so what?)

**Grouped bagging gives a stronger uncertainty signal** at a moderate MAE cost. Moving from random to grouped bagging roughly doubles ensemble spread (~92% under random CV, ~119% under grouped CV), while increasing MAE moderately (~33% under random CV, ~17% under grouped CV).

**Grouped bagging raises MAE systematically while preserving fold standard deviation**. Moving from random to grouped bagging, for random CV, the MAE increased from 0.069 eV to 0.092 eV while the stdev between folds only increased from 0.004 eV to 0.005 eV; for grouped CV, the MAE increased from 0.157 eV to 0.184 eV while the stdev between folds only increased from 0.019 eV to 0.022 eV.


## To-do
1. Week 3: calibration curves
2. 
