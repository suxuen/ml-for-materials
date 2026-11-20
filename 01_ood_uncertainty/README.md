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

5. Calibration: we evaluated the calibration of our ensemble model from **4.** Here's a summary of the 2x2 models.

| ensemble name | random bagging | grouped bagging |
| ----- | ----- | ----- |
| random CV | rcv_rbag | rcv_gbag |
| grouped CV | gcv_rbag | gcv_gbag |

In a model that ranks predictions well, the error (RMSE) should increase with uncertainty (spread) monotonically. In a perfectly calibrated model, RMSE and spread should follow a y=x parity.

We compared RMSE and spread from our four ensemble models using Spearman ρ. Spearman ρ indicates that all four models have RMSEs and spreads that are moderately correlated. Grouped bagging ranks errors better under both CV splits, improving ρ from ~0.5 to ~0.6 

| Spearman ρ (RMSE vs spread, per composition) | random bagging | grouped bagging |
| ----- | ----- | ----- |
| random CV | 0.54 | 0.65 |
| grouped CV | 0.51 | 0.61 |

We plotted RMSE v.s. spread, binned by spread values (Binning: 65 bins of about 100 points each). We found that random bagging tends to make models overconfident; the resulting spreads are compressed and have small values wrt RMSE.
Grouped bagging results in a better calibrated model, but the model is still a overconfident in some places, such as the region of 0.05-0.10 eV error even at near 0 eV spread for gcv_gbag (Figures/2_calibration_spread_bin_gcv_gbag.png).

Evaluating prediction distribution, for all four models, 1σ spread contains the target band gap less often than the 68% expected for well-calibrated, normally distributed (gaussian) errors. This result agrees with the visual inspection above that our models' spread is likely too small. An aside: 68%/95.45% reference assumes normal errors, which may not hold, so treat this evaluation as an approximation. rcv_gbag (Figures/2_calibration_spread_bin_rcv_gbag.png) produces the closest match; grouped CVs result in reduced coverage for both random and grouped ensemble, but grouped ensemble increases coverage for both random and grouped CV.
| prediction distribution | 1σ | 2σ |
| ----- | ----- | ----- |
| gaussian (ideal) | 68.27 % | 95.45 % |
| gcv_gbag | 47.83 % | 77.40 % |
| rcv_gbag | 59.36 % | 85.01 % |
| gcv_rbag | 34.02 % | 58.55 % |
| rcv_rbag | 48.14 % | 76.41 % |


We also compared RMSE vs spread, binned by B-pair. A quick Spearman ρ shows that the ranking improved overall compared to the individual ρ above. This is likely due to the reduced scatter in data associated with grouping cation pairs into their average errors/spreads.

| Spearman ρ (RMSE vs spread, per cation pair) | random bagging | grouped bagging |
| ----- | ----- | ----- |
| random CV | 0.87 | 0.91 |
| grouped CV | 0.75 | 0.76 |

Plotting the cation-pair grouped RMSE v.s. spread shows that, similar to above, random bagging results overconfident models, where the spread turns out smaller than their respective RMSEs. This indicates the model does not fully recognizing test/validation data could follow different chemistries compared to the training data. Grouped bagging does imporve calibration and results in points that are more equally distributed above and below the perfect calibration line. While the points in grouped bagging are more well distributed around perfect calibration, many points still lie far from the line, indicating significant disparity in RMSE and spread values in specific cation-pair groups that would need a closer look.

Overall, grouped bagging is the better uncertainty model. It results in better calibrated spread vs RMSE, and ensemble spread ranks errors more usefully. Note, model calibration varies with spread size, so a single scale factor won't fully correct model calibration.

Ranking B-site cation pair groups from largest to smallest RMSE/spread deviation, we see a variety of B-site species pop up. Both the top (high to low deviation) and bottom (low to high deviation) of the list are rich in transition metals. V, Mn, Fe, Ni from the top and Cu, Pt, Pd from the bottom (recall +2 oxidation state on halide perovskite B-site may result in open shell noble metals). Additionally, closed shell cations such as Cd, Zn, Sn, Pb appear in both the top and bottom of the list. As such, no single class or combination of B-site cation(s) explain(s) the miscalibration from the ratio ranking alone. A closer look, using additional features, descriptors, or methods, is needed to determine whether specific elements or combination of elements lead to poorly calibrated predictions.


## To-do
1. Chemistry specific analysis. Per cation-pair or per element
2. 
