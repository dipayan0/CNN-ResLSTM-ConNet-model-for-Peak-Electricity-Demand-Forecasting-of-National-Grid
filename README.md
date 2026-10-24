# CNN-ResLSTM-ConNet: Peak Electricity Demand Forecasting for the Bangladesh National Grid

[![Code DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23586081.svg)](https://doi.org/10.5281/zenodo.23586081)
[![Data DOI](https://img.shields.io/badge/Data%20DOI-10.17632%2F4c5jwjhpns.1-blue)](https://doi.org/10.17632/4c5jwjhpns.1)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

This repository contains the code, data and trained models for the paper:

> Bhadra, D. & Rois, R. **CNN-ResLSTM-ConNet: A Novel Hybrid Deep Learning Model for Peak Electricity Demand Forecasting of National Grid.** Submitted to *Scientific Reports*. Preprint: [Research Square, doi:10.21203/rs.3.rs-8706187/v1](https://doi.org/10.21203/rs.3.rs-8706187/v1)

CNN-ResLSTM-ConNet is a hybrid deep learning model that forecasts the next day's **evening peak electricity demand** (MW) of the Bangladesh national power grid from the previous 30 days of demand. It combines a 1D convolutional block that extracts local temporal features, stacked LSTM layers with a residual connection, and a final concatenation network (LSTM + MLP) that fuses the learned features with the raw input.

## Contents

| File | Description |
| --- | --- |
| `Daily_Electricity_peak_demand_frecasting_v6.2.ipynb` | The complete experiment: exploratory analysis, statistical tests, the statistical models, all deep learning models, evaluation and figures |
| `Daily_Peak_Electricity_Demand_Bangladesh.csv` | Cleaned daily dataset: date, day peak demand (MW), evening peak demand (MW) |
| `Monthly_data_preprocessed_ver_Final.csv` | The same data with calendar columns; this is the file the notebook reads |
| `*.keras` | Trained Keras models (recommended format) |
| `*.pkl` | The same trained models saved with `pickle` |
| `LICENSE` | MIT License |

The number in each model file name is that run's test-set RMSE in MW.

| Model | Saved runs (test RMSE, MW) | Run reported in the paper |
| --- | --- | --- |
| CNN-ResLSTM-ConNet (proposed) | 933.75, 967.29, 986.83 | 967.29 |
| LSTM-mTrans-MLP | 1032.54, 1042.75 | 1042.75 |
| CNN-ResLSTM (ablation) | 1167.47 | 1167.47 |
| CNN-LSTM (ablation; CNN-Concat-LSTM in the paper) | 1118.47 | 1118.47 |
| LSTM | 1058.55 | 1058.55 |
| MLP | 1066.11 | 1066.11 |

## Data

- **Source:** monthly operational reports published by [Power Grid Bangladesh PLC (PGCB)](https://www.pgcb.gov.bd), compiled into a single daily series.
- **Period:** 1 January 2016 to 30 September 2024 (3,196 days).
- **Target:** daily evening peak demand (MW). The evening peak exceeds the day peak on 98.4% of the days on which both values are recorded.
- **Split:** chronological. Training: 1 January 2016 to 31 December 2022 (2,557 days). Test: 1 January 2023 to 30 September 2024 (639 days).
- **Archived dataset:** [Mendeley Data, doi:10.17632/4c5jwjhpns.1](https://doi.org/10.17632/4c5jwjhpns.1), also on [GitHub](https://github.com/dipayan0/Daily-Peak-Demand-Dataset-of-Power-Grid-Bangladesh-PLC) (CC0 1.0).

## Model architecture

Input: a window of 30 daily values, shape `(30, 1)`, min–max scaled to [0, 1].

```
Input (30, 1)
├─ CNN block:        Conv1D(15 filters, kernel 15, ReLU) → Conv1D(15 filters, kernel 15, ReLU) → Reshape(30, 1)
├─ Concatenate 1:    [CNN output, raw input]                          → (30, 2)
├─ LSTM block 1:     LSTM(30, return_sequences) → LSTM(30) → Reshape(30, 1)
├─ Residual add:     LSTM block 1 output + Concatenate 1 (broadcast)   → (30, 2)
├─ LSTM block 2:     LSTM(30, return_sequences) → LSTM(30) → Reshape(30, 1)
├─ Concatenate 2:    [LSTM block 2 output, raw input]                  → (30, 2)
└─ Output block:     LSTM(30) → Dense(20, ReLU) → Dropout(0.1) → Dense(1)
```

- **Trainable parameters:** 30,791
- **Training:** Adam (learning rate 0.001), mean squared error loss, batch size 1, 12–30 epochs, dropout 0.1 on the first LSTM layer of each block

## Benchmarks

| Group | Models |
| --- | --- |
| Statistical | Classical decomposition + ARIMA(15,1,12), classical decomposition + ETS, STL + ARIMA(15,1,12), STL + ETS(A,A,N), Prophet |
| Base deep learning | MLP, CNN, LSTM |
| Hybrid | LSTM-mTrans-MLP ([Kabir et al., 2025](https://doi.org/10.3390/sci7010007)) |
| Ablation | CNN-Concat-LSTM (no residual connection), CNN-ResLSTM (no final concatenation network) |

## Results

Test-set results from the paper (Table 2):

| Model | RMSE (MW) | MAE (MW) | MAPE (%) |
| --- | --- | --- | --- |
| **CNN-ResLSTM-ConNet** | **967.30** | **657.08** | **5.45** |
| LSTM-mTrans-MLP | 1042.76 | 749.46 | 6.05 |
| LSTM | 1058.55 | 779.34 | 6.35 |
| MLP | 1066.11 | 784.38 | 6.31 |
| CNN | 1171.80 | 807.74 | 6.79 |
| Prophet | 1455.65 | 1154.64 | 9.65 |
| STL + ETS(A,A,N) | 1505.94 | 1109.76 | 9.11 |

The full table, including MSE, MPE, MASE and RMSLE, is in the paper.

## Getting started

### Requirements

The experiments were run with Python 3.8–3.9, TensorFlow 2.13.0 and Keras 2.13.1.

```bash
git clone https://github.com/dipayan0/CNN-ResLSTM-ConNet-model-for-Peak-Electricity-Demand-Forecasting-of-National-Grid.git
cd CNN-ResLSTM-ConNet-model-for-Peak-Electricity-Demand-Forecasting-of-National-Grid

pip install tensorflow==2.13.0 keras==2.13.1 numpy pandas scipy scikit-learn statsmodels \
            pmdarima sktime prophet tbats seaborn matplotlib dill jupyter
```

`xlwings`, imported in the first cell, was only used to compile the original Excel reports. It needs Microsoft Excel and is not required to run the experiments; you can remove that import.

### Running the experiments

Open the notebook and run the cells in order:

```bash
jupyter notebook Daily_Electricity_peak_demand_frecasting_v6.2.ipynb
```

1. **Data loading and EDA:** descriptive statistics, KPSS, ADF and Shapiro–Wilk tests, STL decomposition.
2. **Statistical models:** classical and STL seasonal adjustment with ARIMA and ETS, and Prophet.
3. **Deep learning data preparation:** "Scale the data" and "Create the training dataset" (30-day windows).
4. **Deep learning models:** MLP, CNN, LSTM, LSTM-mTrans-MLP, the ablation variants and CNN-ResLSTM-ConNet.
5. **Evaluation and figures:** test-set metrics and the comparison plots.

The notebook also contains exploratory models that are not reported in the paper, such as TBATS, SARIMAX and transformer variants.

### Using a trained model

```python
import keras

model = keras.models.load_model("CNN_ResLSTM_ConNet_967.29.keras")
y_scaled = model.predict(x)   # x: array of shape (n, 30, 1), scaled to [0, 1]
# Convert back to MW with the same MinMaxScaler used for training (see the notebook)
```

Prefer the `.keras` files. Loading a `.pkl` file runs code embedded in the file, so only load pickle files you trust.

## Evaluation metrics

MSE, RMSE, MAE, MAPE, MPE, MASE and RMSLE. MASE scales each model's test MAE by the in-sample MAE of the one-step naive forecast on the training set, both in MW.

## Citation

If you use this code or the trained models, please cite the software and the paper:

```bibtex
@software{bhadra_cnn_reslstm_connet_2026,
  author    = {Bhadra, Dipayan},
  title     = {CNN-ResLSTM-ConNet model for Peak Electricity Demand Forecasting of National Grid},
  year      = {2026},
  publisher = {Zenodo},
  version   = {v1.0.0},
  doi       = {10.5281/zenodo.23586081},
  url       = {https://doi.org/10.5281/zenodo.23586081}
}

@article{bhadra_rois_cnn_reslstm_connet,
  author  = {Bhadra, Dipayan and Rois, Rumana},
  title   = {CNN-ResLSTM-ConNet: A Novel Hybrid Deep Learning Model for Peak Electricity Demand Forecasting of National Grid},
  journal = {Research Square (preprint)},
  year    = {2026},
  doi     = {10.21203/rs.3.rs-8706187/v1}
}
```

If you use the dataset, please also cite: Bhadra, D. (2026). *Bangladesh National Grid Daily Peak Electricity Demand Dataset (2016–2024)*. Mendeley Data, V1. https://doi.org/10.17632/4c5jwjhpns.1

## License

The code and trained models are released under the [MIT License](LICENSE). The dataset is released under CC0 1.0.

## Acknowledgements

The authors thank the Information Management Division and the System Planning Division of Power Grid Bangladesh PLC for providing the data and the national grid map used in this study.

## Contact
Dipayan Bhadra, Department of Electrical Engineering, University of Colorado Denver, Denver, Colorado, USA and Department of Statistics and Data Science, Jahangirnagar University, Savar, Dhaka
dipu_dipayan@outlook.com · ORCID [0009-0004-9844-3607](https://orcid.org/0009-0004-9844-3607)
Dr. Rumana Rois, Department of Statistics and Data Science, Jahangirnagar University
Email: rois@juniv.edu · ORCID ORCID ID: [0000-0002-0751-7104](https://orcid.org/0000-0002-0751-7104)
