# Machine learning from DFT for halide perovskites

## Motivation

This project uses my halide perovskite dataset ([DOI: 10.1021/acs.chemmater.6c00830](https://doi.org/10.1021/acs.chemmater.6c00830)) to train and evaluate machine learning models:
1. Regression models on DFT band gap and stability targets: Uncertainty and out-of-distribution evaluation (`01_uncertainty_ood/`)
2. Neural Network and Graph Neural Network Methods on DFT band gap and stability targets (`02_nn_gnn/`)
3. Machine-learned Interatomic Potentials and exploratory molecular dynamics (`03_mlip/`)
`00_foundations/` folder contains history of self-directed ML coursework, moved from https://github.com/suxuen/ml_foundations (archived)

## Results so far
`00_foundations/` findings summary:
Composition-based band gap models (Magpie features, histogram gradient boosting) reach 0.074 +/- 0.002 eV MAE on random splits, but error roughly doubles (0.144 +/- 0.023 eV) when B-site cation pairs are held out, so grouped split CV more accurately evaluates regression model performance on unseen chemistries.


## Repository Structure

```
ml-for-materials/
├── README.md
├── .gitignore                            # items not publicly tracked yet
├── configs/                              # default configurations for ML
├── csv/                                  # DFT data in csv format          (not tracked)
├── parquet/                              # DFT data in parquet format      (not tracked)
├── 00_foundations/                       # self-directed ML coursework
├── 01_uncertainty_ood/                   # regression models on DFT data
├── 02_nn_gnn/                            # nn and gnn on DFT data
├── 03_mlip/                              # mlip on DFT data
└── environment.yml                       # Python environment information  (to be added)
```


## Python packages:

**Environment:**
```
Python >=3.8
numpy pandas scipy matplotlib scikit-learn xgboost torch matminer pymatgen
```
to be added: `environment.yml`
*You may use pip or conda to create the environment needed to replicate my results*


## References
- Yew et al. (2026); Thermodynamic and Electronic Effects of B-Site Alloying in Halide Perovskites. *Chemistry of Materials* https://doi.org/10.1021/acs.chemmater.6c00830
- Ward et al. (2018); Matminer: An open source toolkit for materials data mining. *Computational Materials Science* https://doi.org/10.1016/j.commatsci.2018.05.018
- Ong et al. (2013); Python Materials Genomics (pymatgen): A robust, open-source python library for materials analysis. *Computational Materials Science* https://doi.org/10.1016/j.commatsci.2012.10.028
