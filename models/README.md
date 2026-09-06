# Models

Large trained model binaries are excluded from version control.

The final selected model is a Top-6 XGBoost classifier using:

1. `Flow_Duration`
2. `Dst_Port`
3. `Src_Port`
4. `Protocol`
5. `Flow_Pkts/s`
6. `Init_Bwd_Win_Byts`

This directory contains lightweight metadata documenting:

- the full 69-feature XGBoost baseline;
- feature-selection experiments;
- the selected Top-6 model;
- probability calibration;
- the frozen final experiment.
