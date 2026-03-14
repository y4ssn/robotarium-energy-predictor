# Robotarium GTernal Robots Energy Predictor

A data-driven power consumption predictor for GTernal mobile robots deployed on the Georgia Tech Robotarium. The model exploits strong temporal autocorrelation in power dynamics to achieve R² = 0.90 on held-out motion patterns and transfers zero-shot to unseen robots with mean R² = 0.87.

This repository accompanies the paper "Data-Driven Power Prediction for GTernal Mobile Robots: An Autoregressive Model for Energy-Aware Multi-Robot Simulation" [].

## Repository Contents
```
robotarium-energy-predictor/
├── data/
│   ├── energy_data_trial1_structured.npy
│   ├── energy_data_trial2_goal_navigation.npy
│   ├── energy_data_trial3_acceleration.npy
│   ├── energy_data_trial4_extremes.npy
│   ├── energy_data_trial5_random_walk.npy
│   └── energy_data_trial6_steady_state.npy
├── model/
│   ├── energy_predictor_full.npy
│   └── energy_predictor_full.pt
└── README.md
```


## Data Format

Each trial file contains a NumPy array with shape (8000, 11) at approximately 30 Hz. The columns are:

| Index | Field | Unit |
|-------|-------|------|
| 0 | robot_id | - |
| 1 | timestamp | seconds |
| 2 | phase | - |
| 3 | v_cmd | m/s |
| 4 | omega_cmd | rad/s |
| 5 | x | meters |
| 6 | y | meters |
| 7 | theta | radians |
| 8 | voltage | mV |
| 9 | current | mA |
| 10 | power_mW | mW |

## Model Details

The predictor is an autoregressive multi-layer perceptron with 7,041 parameters.

**Input features (11 dimensions):**
- Linear velocity, angular velocity, and their time derivatives
- Absolute values of linear and angular velocity
- Five lags of normalized power history

**Architecture:**
- Hidden layers with dimensions 64, 64, and 32
- Gaussian Error Linear Unit (GELU) activations
- Dropout of 0.1 during training

**Normalization parameters:**
- v_mean: 0.0279, v_std: 0.0736
- w_mean: 0.0479, w_std: 1.0187
- power_mean: 3651.7 mW, power_std: 374.7 mW

**Performance:**
- Single-robot validation: R² = 0.90, MAE = 37.3 mW
- Multi-robot transfer (7 robots): R² = 0.87, MAE = 70.7 mW
- Inference time: 224 μs on GPU

## Simulation Deployment

For simulation environments where ground-truth power is unavailable, the predictor operates autoregressively by feeding its own predictions back as input. This recursive structure accurately models energy accumulation over extended episodes.
```python
predictor.reset(initial_power=3500.0)
total_energy_mWh = 0

for t in range(episode_length):
    power_mW = predictor.step(v_cmd, w_cmd)
    total_energy_mWh += power_mW * dt / 3600
```

## Physical Deployment

When ground-truth power readings are available from the INA260 sensor, the predictor can be corrected at each timestep to prevent error accumulation. Use `predictor.step_with_correction(v_cmd, w_cmd, actual_power_mW)` to update the internal power history buffer with measured values.

## Citation

If you use this predictor or dataset in your research, please cite:
```bibtex
@article{abdelmeguid2025gternal,
  title={Data-Driven Power Prediction for GTernal Mobile Robots: An Autoregressive Model for Energy-Aware Multi-Robot Simulation},
  author={Abdelmeguid, Yassin and Hasan, Ammar},
  journal={arXiv preprint arXiv:XXXX.XXXXX},
  year={2026}
}
```
## Authors

- Dr. Ammar Hasan, American University of Sharjah
- Yassin Abdelmeguid, American University of Sharjah



## Acknowledgments

We thank Sean Wilson and Nathan Wert of the Georgia Tech Robotarium for providing access to the GTernal platform. The real-time power telemetry API used in this work was implemented by the Robotarium team to enable this research.

## License

MIT License
