# Data

This project uses the IoTID20 network intrusion dataset.

The raw dataset is intentionally excluded from this repository because of file size and redistribution considerations.

## Experimental Safeguards

- Direct identifiers are excluded from model predictors.
- Feature-identical groups with conflicting labels are excluded.
- Train, validation, and test partitions are feature-group aware.
- Preprocessing statistics are fitted using training data only.
- The final test set remains locked until model development is complete.
