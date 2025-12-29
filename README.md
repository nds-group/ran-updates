# RAN Updates Traffic Impact Dataset

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

This repository contains the dataset accompanying the paper:

> **"A First Look at Operational RAN Updates and Their Impact on Carrier Traffic Demands and Prediction"**

This dataset provides **1,931 traffic time series** measured at carriers affected by RAN modifications in a nationwide operational infrastructure.


## Dataset Structure

The dataset is organized into **two related tables** linked by a unique identifier (`UUID`), following a normalized structure to reduce redundancy:

```
dataset/
├── df_traffic.parquet    # Normalized Hourly traffic measurements
├── df_info.parquet       # General carrier information
└── README.md
```


## Data Dictionary

### `df_info.parquet` — Carrier Information Table

Contains one record per impacted carrier with static metadata.

| Column | Type | Description |
|--------|------|-------------|
| `UUID` | string | Unique identifier to use to correlate with the traffic table. |
| `antennaID` | string | Anonymized antenna identifier |
| `urbanization` | string | Area classification: `rural`, `suburban`, `urban`, `metropolitan`, or `city_center` |
| `date` | string | Event date — first day of the week after the RAN update occurred (YYYY-MM-DD format) |
| `technology` | string | Radio Access Technology of the **impacted** carrier (`4G` or `5G`) |
| `event_type` | string | Type of RAN update event (e.g., `4G_open`, `5G_open` — opening of new antenna at the same base station location) |

### `df_traffic.parquet` — Traffic Measurements Table

Contains hourly traffic time series for each carrier within a 120-day window centered on the RAN update event.

| Column | Type | Description |
|--------|------|-------------|
| `UUID` | string | UUID key linking to `df_info.parquet` |
| `uxtime` | int | Absolute timestamp in Unix time |
| `deltaT` | int | Time offset in seconds relative to the event date as reported in the Carrier Information Table (negative = before event, positive = after event) |
| `DL` | float | Downlink traffic volume (standardized, z-score normalized) |
| `UL` | float | Uplink traffic volume (standardized, z-score normalized) |

---

## Example Usage
We will see how to load and plot a single timeseries
```python
import pandas as pd
import matplotlib.pyplot as plt

# Load data
df_traffic = pd.read_parquet('df_traffic.parquet')
df_info = pd.read_parquet('df_info.parquet')

# Get a single carrier's time series
sample_uuid = df_info['UUID'].iloc[0]
carrier_traffic = df_traffic[df_traffic['UUID'] == sample_uuid].sort_values('uxtime')

# Plot traffic around the event
plt.figure(figsize=(12, 4))
plt.plot(carrier_traffic['deltaT'] / 3600 / 24, carrier_traffic['DL'], alpha=0.7)
plt.xlabel('Days from Event')
plt.ylabel('Normalized DL Traffic (σ)')
plt.title('Traffic Impact of RAN Update')
plt.legend()
plt.show()
```


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
