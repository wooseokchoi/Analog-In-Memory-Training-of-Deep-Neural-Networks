# Analog In-Memory Training Simulation of Deep Neural Networks

[![MATLAB](https://img.shields.io/badge/MATLAB-R2020b%2B-orange?logo=mathworks)](https://www.mathworks.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Book Chapter](https://img.shields.io/badge/Book%20Chapter-Wiley-blue)](https://onlinelibrary.wiley.com/doi/abs/10.1002/9783527847419.ch22)

> MATLAB simulator for analog in-memory training of deep neural networks, using a **nonlinear resistive memory device model** and a **fully parallel crossbar weight update scheme**.
>
> This code accompanies the book chapter:
> **"Nonvolatile Resistive Memory Technology for Deep Neural Network Hardware Applications"**
> in *Nonvolatile Memory and Selector Devices* (Wiley, 2024).
> [[Publisher Link]](https://onlinelibrary.wiley.com/doi/abs/10.1002/9783527847419.ch22)

---

## Overview

This simulator demonstrates how deep neural networks can be trained **directly on analog resistive memory (ReRAM/PCM) crossbar arrays**, capturing key nonidealities of real hardware devices:

- **Nonlinear and asymmetric weight updates** (softbound model)
- **Stochastic weight update** via parallel pulse bitstream coincidence
- **Device-to-device (D2D) and cycle-to-cycle (C2C) variability**
- **Differential crossbar configuration for negative weights** (weight = W − Wref)

The simulation targets the **MNIST handwritten digit classification** task using a 3-layer fully connected network.

---

## Network Architecture

```
Input (784) → Hidden Layer 1 (256) → Hidden Layer 2 (128) → Output (10)
```

- Activation function: Sigmoid (or ReLU)
- Output layer: Softmax
- Loss function: Mean Squared Error (or Cross-Entropy)
- Optimizer: Stochastic Gradient Descent (SGD, batch size = 1)

---

## Device Model

The weight update follows a **softbound nonlinear model**:

- **Potentiation:** `Δw⁺ = slope_p × (w_max − w)`
- **Depression:** `Δw⁻ = slope_n × (w_min − w)`

This naturally captures the **conductance saturation** behavior near the weight bounds observed in real analog memory devices.

### Key Device Parameters

| Parameter | Description | Default |
|---|---|---|
| `wmax` / `wmin` | Weight bounds | +1 / −1 |
| `dw_min` | Minimum weight step | 0.002 |
| `dw_min_std` | C2C variation (std) | 0 |
| `w_max_dtod` / `w_min_dtod` | D2D variation of bounds | 0 |
| `up_down` | Potentiation/depression asymmetry | 0 |
| `up_down_dtod` | D2D variation of asymmetry | 0 |
| `Nstates` | Number of discrete conductance states | 1000 |

---

## Stochastic Crossbar Update

Instead of computing exact gradient products, this simulator implements a **fully parallel stochastic weight update** scheme:

1. Convert neuron activations `x` and error signals `δ` into **stochastic bitstreams** of length `len_bit`
2. Detect **pulse coincidences** across the entire crossbar simultaneously
3. Apply weight updates only at coincidence events

The bitstream length `len_bit` is set by:
```
len_bit = round(LR / dw_min)
```
and is **halved each epoch** (learning rate decay).

---

## Requirements

- **MATLAB R2020b or later** (no additional toolboxes required)
- MNIST dataset files (included in this repository):
  - `train-images-idx3-ubyte`
  - `train-labels-idx1-ubyte`
  - `t10k-images-idx3-ubyte`
  - `t10k-labels-idx1-ubyte`

---

## Quick Start

1. Clone this repository:
   ```bash
   git clone https://github.com/wooseokchoi/Analog-In-Memory-Training-of-Deep-Neural-Networks.git
   cd Analog-In-Memory-Training-of-Deep-Neural-Networks
   ```

2. Open MATLAB and navigate to the repository folder.

3. Run the main script:
   ```matlab
   DNN_3Layer_Softbound_Stochastic_Weight_Update
   ```

4. Training progress will be printed to the console and a test accuracy plot will appear after training.

---

## Key Tunable Parameters

Open `DNN_3Layer_Softbound_Stochastic_Weight_Update.m` and adjust the following at the top of the file:

```matlab
%% network parameters
NumNeurons     = [784 256 128 10]; % Network topology
LR             = 0.04;             % Learning rate
epoch_number   = 30;               % Number of training epochs
trainset_length = 5000;            % Training set size (max 60000)

%% device parameters
device.dw_min     = 0.002;  % Minimum conductance step
device.dw_min_std = 0;      % C2C variation
device.w_max_dtod = 0;      % D2D variation (wmax)
device.up_down    = 0;      % Update asymmetry
```

---

## File Structure

```
├── DNN_3Layer_Softbound_Stochastic_Weight_Update.m   # Main simulation script
├── loadMNISTImages.m                                  # MNIST image loader
├── loadMNISTLabels.m                                  # MNIST label loader
├── t10k-images-idx3-ubyte                             # MNIST test images
├── t10k-labels-idx1-ubyte                             # MNIST test labels
├── LICENSE
└── README.md
```

> ⚠️ **Note:** The training set files (`train-images-idx3-ubyte`, `train-labels-idx1-ubyte`) are not included due to file size. Download them from the [official MNIST website](http://yann.lecun.com/exdb/mnist/) and place them in the same folder.

---

## Expected Results

With default settings (5,000 training samples, 30 epochs, ideal device), you can expect:

| Metric | Approx. Value |
|---|---|
| Test Accuracy | ~95% |
| Training Time | ~5–15 min (CPU) |

Accuracy will decrease when device nonidealities (C2C, D2D variation) are enabled — this is the intended behavior for hardware-aware simulation.

---

## Citation

If you use this code in your research, please cite:

```bibtex
@incollection{choi2024nonvolatile,
  author    = {Choi, Wooseok},
  title     = {Nonvolatile Resistive Memory Technology for Deep Neural Network Hardware Applications},
  booktitle = {Nonvolatile Memory and Selector Devices},
  publisher = {Wiley},
  year      = {2024},
  doi       = {10.1002/9783527847419.ch22}
}

@article{choi2026update,
  author    = {Choi, Wooseok and Stecconi, Tommaso and Falcone, Donato Francesco
               and Galetta, Matteo and Clerico, Victoria and Zaccaria, Elisa
               and Ram, Mamidala Saketh and La Porta, Antonio and Horst, Folkert
               and Jubin, Daniel and Senger, Matias and Sousa, Marilyne
               and Reidt, Steffen and Heller, Ralph and Linares-Barranco, Bernabe
               and Bragaglia, Valeria and Offrein, Bert Jan},
  title     = {Update Disturbance-Resilient Analog {ReRAM} Crossbar Arrays
               for In-Memory Deep Learning Accelerators},
  journal   = {Advanced Science},
  volume    = {13},
  number    = {4},
  year      = {2026},
  doi       = {10.1002/advs.202504578}
}

@misc{rasch2023fast,
  author    = {Rasch, Malte J. and Carta, Fabio and Fagbohungbe, Omebayode and Gokmen, Tayfun},
  title     = {Fast Offset Corrected In-Memory Training},
  year      = {2023},
  eprint    = {2303.04721},
  archivePrefix = {arXiv},
  primaryClass  = {cs.LG},
  url       = {https://arxiv.org/abs/2303.04721}
}
```

---

## License

This project is licensed under the [MIT License](LICENSE).
