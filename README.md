# Diffusion-MTS-AD: Diffusion-Based Multivariate Time Series Anomaly Detection for Automotive Ethernet Traffic

A comprehensive evaluation of **7 diffusion model frameworks** for multivariate time series anomaly detection, applied to the **TOW-IDS Automotive Ethernet Intrusion Detection Dataset**. Each framework is implemented from its original research paper with full training, evaluation, and deployment profiling.

## Overview

This notebook implements and benchmarks diffusion-based anomaly detection models for identifying network intrusions in Automotive Ethernet traffic. All models follow the same pipeline: extract windowed features from PCAPs, build sliding-window sequences, train on normal traffic only, and score anomalies via reconstruction/imputation error from the diffusion process.

## Models Implemented

| # | Model | Paper | Paradigm | Key Architecture |
|---|-------|-------|----------|-----------------|
| 1 | **DDPM Diffusion** | Pintilie et al., IEEE ICDM-W 2023 | Reconstruction | 1D U-Net with sinusoidal timestep embeddings |
| 2 | **DiffusionAE** | Pintilie et al., IEEE ICDM-W 2023 | AE + Diffusion | Bottleneck Transformer AE + DDPM on residuals |
| 3 | **ImDiffusion** | Chen et al., Microsoft 2023 | Imputation | Conditional U-Net with CSDI-style masking |
| 4 | **DiffAD** | Xiao et al., KDD 2023 | Imp. + Weight-Inc. | Multi-scale S4-based U-Net + density-ratio selection |
| 5 | **DiffADT** | Zuo et al., Applied Intelligence 2024 | Recon. + S4 | Cascaded S4 layers + cosine schedule + conv scoring |
| 6 | **TimeGrad** | Rasul et al., ICML 2021 | AR Forecasting | Autoregressive GRU + per-step conditional DDPM |
| 7 | **Diffusion-TS** | Yuan & Qiao, ICLR 2024 | Gen. + Decomp. | Transformer with trend/seasonal decomposition |

## Dataset

This project uses the **TOW-IDS Automotive Ethernet Intrusion Detection Dataset**, which contains:
- Training PCAP (normal + attack traffic)
- Test PCAP (normal + attack traffic)
- Per-packet labels (`y_train.csv`, `y_test.csv`) with attack type annotations
- Attack types: CAN DoS (C_D), CAN Replay (C_R), PTP sync (P_I), Injection (F_I), MAC Flooding (M_F)
- **Dataset Download Link**: [TOW-IDS Automotive Ethernet Intrusion Dataset](https://ieee-dataport.org/documents/tow-ids-automotive-ethernet-intrusion-dataset)

## Configuration Defaults

| Parameter | Value | Description |
|-----------|-------|-------------|
| `WINDOW_SEC` | 1.0 | Window aggregation interval (seconds) |
| `T_SEQ` | 40 | Sequence length (number of windows) |
| `STRIDE` | 1 | Sliding window stride |
| `N_DIFF_STEPS` | 100 | Diffusion timesteps T |
| `BETA_START` | 1e-4 | Linear noise schedule start |
| `BETA_END` | 0.02 | Linear noise schedule end |
| `THR_PCT` | 99 | Anomaly threshold percentile (on train-normal scores) |
| `BATCH_SIZE` | 256 | Training batch size |
| `EPOCHS` | 100 | Training epochs |
| `LR` | 1e-4 | Learning rate (Adam optimizer) |
| `SEED` | 42 | Random seed |

## Windowed Features

Each 1-second time window is represented by 13 statistical features:

| Feature | Description |
|---------|-------------|
| `pkt_count` | Number of packets in window |
| `bytes_sum` | Total bytes |
| `pkt_len_mean/std` | Packet length statistics |
| `dt_mean/std` | Inter-arrival time statistics |
| `uniq_src_mac/dst_mac` | Unique MAC address counts |
| `uniq_ip_src/dst` | Unique IP address counts |
| `uniq_eth_type` | Unique Ethernet type count |
| `multicast_ratio` | Fraction of multicast packets |
| `vlan_ratio` | Fraction of VLAN-tagged packets |

## Evaluation

### Per-Framework Evaluation
Each model is evaluated with:
- **Per-attack one-vs-normal breakdown**: attack_type, n_attack_seqs, TN, FP, FN, TP, Precision, Recall, F1, ROC-AUC, PR-AUC
- **Score distribution histogram**: Normal vs Attack density with threshold line
- **Confusion matrix**: Heatmap with printed TN/FP/FN/TP and P/R/F1 annotation

### Unified Comparison
- **All confusion matrices grid**: 2x4 subplot of all 7 frameworks
- **Radar chart**: Multi-metric comparison (ROC-AUC, PR-AUC, F1, Precision, Recall)
- **F1 vs Latency bar chart**: Detection quality vs inference speed tradeoff

### Deployment Metrics
- Model size (Params, Size in MB)
- Inference latency (Latency in ms)
- Throughput (calls/s)
- Energy consumption (Energy in J/call, measured via NVML)
- GPU Power Avg/Peak (W), GPU Utilization (%), GPU Memory (MB)
- CPU Utilization (%), RSS Peak (MB)

## Notebook Structure

| Cell(s) | Section | Description |
|---------|---------|-------------|
| 0 | Title | Framework comparison table |
| 1 | Imports | PyTorch, sklearn, pynvml, psutil, etc. |
| 2 | Configuration | File paths, hyperparameters |
| 3 | Utilities | Seed, metrics, profiling functions |
| 4 | Diffusion Schedule | Linear/cosine beta schedules, q_sample, p_sample |
| 5-7 | Data Ingestion | PCAP extraction, label loading |
| 8-9 | Feature Engineering | Window building with attack-type labels |
| 10 | Sequence Building | Sliding windows, StandardScaler, train/test split |
| 11-12 | EDA | Label distribution, feature correlation heatmap |
| 13-16 | DDPM Diffusion | Paper reference, 1D U-Net, training, evaluation |
| 17-19 | DiffusionAE | Bottleneck Transformer AE + diffusion, training, eval |
| 20-22 | ImDiffusion | Conditional imputation denoiser, training, eval |
| 23-25 | DiffAD | S4-based U-Net + weight-incremental sampling, training, eval |
| 26-28 | DiffADT | Cascaded S4 + cosine schedule, training, eval |
| 29-31 | TimeGrad | Autoregressive GRU + DDPM, training, eval |
| 32-34 | Diffusion-TS | Transformer + decomposition, training, eval |
| 35-36 | All Confusion Matrices | 2x4 grid of all frameworks |
| 37-38 | Profiling | Model size, latency, energy measurement |
| 39-40 | Final Comparison Table | Deployment comparison with best-model highlights |
| 41 | Radar & Bar Charts | Multi-metric radar + F1 vs latency |

## Requirements

- Python 3.8+
- PyTorch (with CUDA recommended)
- scikit-learn
- pandas, numpy, matplotlib, seaborn
- pynvml (for GPU energy measurement)
- psutil (for CPU profiling)
- tshark / Wireshark (for PCAP extraction)

Install dependencies:
```bash
pip install torch scikit-learn pandas numpy matplotlib seaborn pynvml psutil
```

## Usage

1. Ensure the TOW-IDS dataset is available locally
2. Update file paths in **Cell 2** to point to your dataset
3. Run all cells sequentially in Jupyter Notebook or JupyterLab
4. Training all 7 models takes approximately 1-2 hours on a GPU

## References

1. Pintilie, S., Bhatt, M., & Gao, J. (2023). *Time Series Anomaly Detection using Diffusion-based Models.* IEEE ICDM Workshop.
2. Chen, Y., et al. (2023). *ImDiffusion: Diffusion Models for Multivariate Time Series Anomaly Detection.* Microsoft.
3. Xiao, Z., et al. (2023). *Imputation-based Time-Series Anomaly Detection with Conditional Weight-Incremental Diffusion Models.* KDD.
4. Zuo, J., et al. (2024). *Unsupervised Diffusion Based Anomaly Detection for Time Series.* Applied Intelligence.
5. Rasul, K., Seward, C., Schuster, I., & Vollgraf, R. (2021). *Autoregressive Denoising Diffusion Models for Multivariate Probabilistic Time Series Forecasting.* ICML.
6. Yuan, X., & Qiao, Y. (2024). *Diffusion-TS: Interpretable Diffusion for General Time Series Generation.* ICLR.

## License

This project is for research and educational purposes.
