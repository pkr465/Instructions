Here is a comprehensive `.md` file tailored for your DGX Spark environment. This documentation assumes you are working with the **NVIDIA Grace Blackwell** architecture (ARM64) and leveraging the **Unified Memory** architecture.

```markdown
# NVIDIA DGX Spark: AI Development & Fine-Tuning Guide

This guide provides the technical steps to initialize the NVIDIA DGX Spark, install the necessary AI software stack, and identify compatible models for local fine-tuning.

---

## 1. System Initialization

DGX Spark runs **DGX OS** (Ubuntu-based). Use one of the following methods for initial setup:

### A. Local Setup
1. Connect a monitor (mini-DP), keyboard, and mouse.
2. Power on and follow the GUI wizard to set up credentials and networking.

### B. Headless Setup (Hotspot)
1. Power on the unit.
2. On another device, connect to the Wi-Fi SSID found on your **Quick Start Card**.
3. Navigate to the local IP address (usually `192.168.x.x`) in your browser to complete the setup.

> [!NOTE]
> During first boot, the system synchronizes with **NVIDIA AI Enterprise**. This requires a high-speed internet connection to download approximately 10GB–20GB of drivers and container manifests.

---

## 2. Software Stack Installation

The DGX Spark uses the **Grace Blackwell** chip. All software must be compatible with the **ARM64 (aarch64)** architecture.

### Step 1: System Verification
Ensure the Blackwell drivers and CUDA 12.x+ are active:
```bash
nvidia-smi
nvcc --version

```

### Step 2: Install Environment Manager

We recommend **Miniforge** for ARM64 to manage dependencies without bloat.

```bash
wget [https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh](https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-aarch64.sh)
bash Miniforge3-Linux-aarch64.sh
source ~/.bashrc

```

### Step 3: Create Fine-Tuning Environment

```bash
mamba create --name spark_funtune python=3.11 -y
mamba activate spark_funtune

```

### Step 4: Install Core Libraries

Install the specialized versions of PyTorch and Unsloth (optimized for Blackwell FP4/Unified Memory):

```bash
pip install torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/cu124](https://download.pytorch.org/whl/cu124)
pip install "unsloth[colab-new] @ git+[https://github.com/unslothai/unsloth.git](https://github.com/unslothai/unsloth.git)"
pip install ninja triton xformers

```

---

## 3. Supported Models for Fine-Tuning

The DGX Spark features **128GB of Unified Memory**, allowing for massive context windows and larger parameter counts than standard workstations.

| Model Series | Parameters | Tuning Method | Memory Footprint |
| --- | --- | --- | --- |
| **Llama 3.1 / 3.2** | 8B - 70B | Full / LoRA / QLoRA | 70B fits easily with QLoRA |
| **Mistral / Mixtral** | 7B - 8x7B | LoRA / QLoRA | Handles MoE efficiently |
| **DeepSeek R1** | 7B - 70B | QLoRA / Distilled | High reasoning throughput |
| **Gemma 2 / 3** | 2B - 27B | Full Fine-Tuning | Optimized for Blackwell |
| **Nemotron-3** | 8B - 22B | Full Fine-Tuning | Native FP4 Support |

---

## 4. Hardware Optimization Tips

* **Unified Memory:** Unlike PCIe GPUs, the Grace Blackwell Superchip shares memory. You can allocate nearly the entire 128GB to a single training process using `--max_memory`.
* **NVFP4 Support:** Use 4-bit floating point (FP4) during training to double your effective throughput on supported models.
* **Cooling:** Ensure the Spark has at least 6 inches of clearance on all sides. Fine-tuning 70B models will push the Grace Blackwell chip to its thermal limits.

---

## 5. Quick Test Script

Create a file named `test_gpu.py` to ensure your stack is communicating with the Blackwell cores:

```python
import torch
print(f"CUDA Available: {torch.cuda.is_available()}")
print(f"Device Name: {torch.cuda.get_device_name(0)}")
print(f"Total Memory: {torch.cuda.get_device_properties(0).total_memory / 1e9:.2f} GB")

```

```bash
python test_gpu.py

```
