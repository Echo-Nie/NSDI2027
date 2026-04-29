### 📁 Project Structure
```
├── controller/          # All Controller
├── env/                # Custom IIoT-MEC cyber-physical simulator
├── train/              # Training scripts
│   ├── agent.py                 # Base agent class & core training utilities
│   ├── run_arer_ddpg.py         # ARER-enhanced DDPG training script
│   ├── run_std_ddpg.py          # Standard DDPG baseline training script
│   ├── agent_other_algo.py      # Extended agent classes
│   └── ...                      # Training scripts (to be added)
├── utils/              # Core utilities (ReplayBuffer, training loops, etc.)
├── evaluation/         # Evaluation scripts, plotting tools, and pre-trained models
│   └── all_models/     # Pre-trained network weights for all experiments
│   └── all_data/       # Raw experimental data
├── README.md
└── requirements.txt
```

### 📂 Repository Structure
The repository is logically divided into three main components:

- [`/env`](./env): Contains the custom IIoT-MEC cyber-physical simulator (`env.py`). It explicitly models the discrete DVFS P-states, FCFS queueing dynamics, context switch penalties, and hardware thermal steady-state behaviors.

- [`/controller`](./controller): Contains the source code for the standard RL baselines and their corresponding ARER-enhanced variants. Each algorithm has its own directory (e.g., `/DDPG`, `/CrossQ`).

- [`/utils`](./utils): Provides reusable reinforcement learning utility functions and components. Includes a generic `ReplayBuffer` class for off-line training, a `moving_average` function for data smoothing, and standardized training loops such as `train_on_policy_agent` and `train_off_policy_agent` to ensure consistent and reproducible experiment execution.

- [`/train`](./train): Contains the end-to-end model training scripts used to learn all policies in the paper. These scripts initialize the environment, instantiate RL agents, manage training pipelines, and periodically save optimized network weights to the model directory.

- [`/evaluation`](./evaluation): Contains all pre-trained network weights (`/all_models`) and out-of-the-box plotting scripts to regenerate the paper's results.

### ⚙️ Requirements & Installation

Please refer to our [`requirements.txt`](./requirements.txt) for the full list of Python dependencies. We recommend using Python 3.9+.

```bash
# Clone the repository
git clone xxx.git

# Install dependencies
pip install -r requirements.txt
```

**Hardware Requirements:**
* **Evaluation & Plotting:** The artifact is entirely software-based. All zero-shot evaluation scripts utilizing the pre-trained weights can be executed seamlessly on **standard multi-core CPUs**.
* **Training from Scratch:** While the simulator is CPU-compatible, the highly parallelized Bellman updates and state-action representation learning (particularly for CrossQ and TD7) are computationally intensive. We utilized an **NVIDIA GeForce RTX 4090 GPU (24 GB VRAM)** with **CUDA 13.0** for all reported training sessions. 

### 📊 System Parameters & Configuration

The simulation parameters strictly align with physical hardware thermal specifications and standard industrial URLLC requirements. The default parameters defined in [`env/env.py`](./env/env.py) are summarized below:

| Parameter Category       | Parameter Name                    | Default Value         |
| :----------------------- | :-------------------------------- | :-------------------- |
| **Network & Traffic**    | Number of Devices (N)             | 10, 20, 30, 40, 50    |
|                          | Time slot duration (tau)          | 0.1 s                 |
|                          | Task arrival probability (lambda) | 0.6                   |
|                          | Required CPU cycles               | [200, 700] Megacycles |
|                          | System bandwidth                  | 20 MHz                |
| **Edge Server (MEC)**    | MEC CPU capacity                  | 20 GHz                |
| **Hardware Constraints** | Context switch overhead           | 20 ms                 |
|                          | Thermal resistance                | 22 °C/W               |
|                          | Thermal safety cap                | 65 °C                 |
| **URLLC & Penalty**      | Delay deadline (D_max)            | 0.5 s                 |
|                          | Violation penalty (beta)          | 20.0                  |

> **Note on Customization:** You can easily modify these parameters to test different cyber-physical limits by passing arguments when initializing the environment (e.g., `env = IIoT_DVFS_MEC_Env(num_devices=20, prob_arrival=0.8, D_max=0.4)`).

### 🚀 How to Reproduce the Paper's Claims

We provide ready-to-run scripts in the `/evaluation` directory to perfectly reproduce the tables and figures presented in **Section 5: Performance Evaluation** of our paper.

##### 1. DVFS Thrashing & Policy Stability Analysis (Experiment 5)
This script evaluates the context switch overheads and pathological policy thrashing behavior across controller.
```bash
python evaluation/exp5_thrashing.py
```
* **Output:** Generates `Fig_Thrashing_Analysis.pdf`, confirming ARER-TD7's remarkable stability (reducing switches to ~20 per episode).

##### 2. Traffic Density Impact (Experiment 4)
This script performs a zero-shot stress test across varying task arrival probabilities (lambda from 0.2 to 1.0).
```bash
python evaluation/exp4_lambda_density.py
```
* **Output:** Generates the 2x3 Facet Grid `Fig_Traffic_Density_FacetGrid.pdf`, demonstrating the "Performance Gap" and how ARER suppresses URLLC violations below 5% under heavy load.

##### 3. Extreme URLLC Deadline Sensitivity (Experiment 6)
This script tightens the hardware deadline progressively from 0.50s down to 0.40s in precise 20ms steps (reflecting the exact physical context switch penalty).
```bash
python evaluation/exp6_dmax_sensitivity.py
```
* **Output:** Generates `Fig_Dmax_Sensitivity.pdf`, validating the graceful degradation of ARER-enhanced agents at the micro-architectural frontier.
