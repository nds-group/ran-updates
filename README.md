# RAN Updates Traffic Impact Dataset

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

This repository contains the dataset accompanying the [paper](https://hdl.handle.net/20.500.12761/2006):

> **"A First Look at Operational RAN Updates and Their Impact on Carrier Traffic Demands and Prediction"**

This dataset provides **1,931 traffic time series** measured at carriers affected by RAN modifications in a nationwide operational infrastructure.


## Dataset Structure

The dataset is organized into **two related tables** linked by a unique identifier (`UUID`), following a normalized structure to reduce redundancy:

```
dataset/
├── traffic_dataset.parquet    # Normalized Hourly traffic measurements
├── info_dataset.parquet       # General carrier information
README.md
create_dataset.ipynb
```


## Data Dictionary

### `info_dataset.parquet` — Carrier Information Table

Contains one record per impacted carrier with static metadata.

| Column | Type | Description |
|--------|------|-------------|
| `UUID` | string | Unique identifier to use to correlate with the traffic table |
| `antennaID` | string | Anonymized antenna identifier |
| `urbanization` | string | Area classification: `Rural`, `Suburban`, `Urban`, `Metropolitan`, or `Metropolitan Center` |
| `technology` | string | Radio Access Technology of the **impacted** carrier (e.g., `4G`) |
| `event_type` | string | Type of RAN update event (e.g., `4G addition`, `5G addition` — opening of new antenna at the same base station location) |

### `traffic_dataset.parquet` — Traffic Measurements Table

Contains hourly traffic time series for each carrier within a 120-day window centered on the RAN update event.

| Column | Type | Description |
|--------|------|-------------|
| `UUID` | string | UUID key linking to `info_dataset.parquet` |
| `uxtime` | int | Absolute timestamp in Unix time |
| `deltaT` | int | Time offset in seconds relative to the event period (negative = before event, positive = after event) |
| `DL` | float | Downlink traffic volume (standardized, z-score normalized) |
| `UL` | float | Uplink traffic volume (standardized, z-score normalized) |

---

## Example Usage
We will see how to load and plot a single timeseries
```python
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.tsa.seasonal import seasonal_decompose
import matplotlib.pyplot as plt

# Load data
df_traffic = pd.read_parquet('traffic_dataset.parquet')
df_info = pd.read_parquet('info_dataset.parquet')

# Get a single carrier's time series
sample_uuid = df_info['UUID'].iloc[12]
carrier_traffic = df_traffic[df_traffic['UUID'] == sample_uuid].copy()
carrier_traffic = carrier_traffic.sort_values('deltaT').reset_index(drop=True)

# Convert deltaT to days and aggregate to daily level
carrier_traffic['days'] = carrier_traffic['deltaT'] / 3600 / 24
daily_traffic = carrier_traffic.groupby(carrier_traffic['days'].astype(int))['DL'].mean()

# Perform seasonal decomposition (period=7 for weekly seasonality)
decomposition = seasonal_decompose(daily_traffic, model='additive', period=7)

# Plot the decomposition
fig, axes = plt.subplots(4, 1, figsize=(14, 8), sharex=True)

axes[0].plot(daily_traffic.index, decomposition.observed, color='blue')
axes[0].axvline(x=0, color='red', linestyle='--', alpha=0.7, label='Event')
axes[0].set_ylabel('Observed')
axes[0].set_title(f'Time Series Decomposition (UUID: {sample_uuid[:8]}...)')
axes[0].legend()

axes[1].plot(daily_traffic.index, decomposition.trend, color='orange')
axes[1].axvline(x=0, color='red', linestyle='--', alpha=0.7)
axes[1].set_ylabel('Trend')

axes[2].plot(daily_traffic.index, decomposition.seasonal, color='green')
axes[2].axvline(x=0, color='red', linestyle='--', alpha=0.7)
axes[2].set_ylabel('Seasonal')

axes[3].plot(daily_traffic.index, decomposition.resid, color='red')
axes[3].axvline(x=0, color='red', linestyle='--', alpha=0.7)
axes[3].set_ylabel('Residual')
axes[3].set_xlabel('Days from Event')

plt.tight_layout()
plt.savefig('time_series_decomposition.png', dpi=300)
plt.show()




```
<img width="4200" height="2400" alt="time_series_decomposition" src="https://github.com/user-attachments/assets/06416f80-a9b7-4153-9736-9ebac454c410" />


## Data Processing Notes

### Standardization

Traffic values (`DL`, `UL`) are **z-score normalized** using statistics from a 30-day reference period located two months before the RAN update:

Each time series spans a **120-day window** centered on the RAN update event:
- **60 days before** the update (including a 30-day reference period for normalization)
- **60 days after** the update

### Selection Criteria

Carriers in this dataset were selected based on:
1. **Isolated updates**: Only RAN updates occurring without concurrent modifications at the same site and azimuth
2. **Statistical significance**: Carriers passing a Wilcoxon signed-rank test (α = 0.01) confirming significant traffic change
3. **Impact type**: Carriers classified in "Cluster A" — those experiencing a significant drop in served traffic following the update

---

## Citation

If you use this dataset in your research, please cite:

```bibtex
@inproceedings{ran_updates_2025,
  title     = {A First Look at Operational RAN Updates and Their Impact on Carrier Traffic Demands and Prediction},
  author    = {[Antonio Boiano, Nadezhda Chukhno, Zbigniew Smoreda, Alessandro E. C. Redondi, Marco Fiore]},
  booktitle = {[INFOCOM 2026]},
  year      = {2026}
}
```

---
## License

This dataset is released under the [Creative Commons Attribution 4.0 International License (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/).

You are free to:
- **Share** — copy and redistribute the material in any medium or format
- **Adapt** — remix, transform, and build upon the material for any purpose

Under the following terms:
- **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made.

---

## Contact

For questions about the dataset, please contact: antonio.boiano@polimi.it, nadezda.chukhno@networks.imdea.org, marco.fiore@networks.imdea.org

For issues or contributions to this repository, please open a GitHub issue.

---

## Acknowledge
The work involving the data collection, processing, and analysis was funded by CoCo5G (ANR-22-CE25-0016) of the French National Research Agency (ANR) as well as by the 6G-IRONWARE (CNS2023-143870) funded by MICIU/AEI /10.13039/501100011033 and by the European Union NextGenerationEU/PRTR, and by the 6G-AI-TANGO (GA 101206327) funded by the European Union.

<img alt="coco5g-logo" src="https://github.com/user-attachments/assets/fd8af8e0-c3a2-4f3f-aabf-9dfbb8740b56" height="60" />
<img alt="media-file-6g-ironware-project-logo-300x162" src="https://github.com/user-attachments/assets/14144f6e-e247-4d98-a0ce-cd8f09b46656" height="60" />
<img alt="logo_6G_AI_TANGO" src="https://github.com/user-attachments/assets/b741846b-c608-4bc6-b9b6-7939121f99d2" height="60" />
<img alt="EU logo transparent" src="https://github.com/user-attachments/assets/680149c2-8bf0-4ba0-9688-017796b96886" height="60" />
