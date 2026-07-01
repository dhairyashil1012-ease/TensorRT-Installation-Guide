# TensorRT 11.1 Installation Guide (Ubuntu 24.04 + RTX A4000)

> **Target System**
>
> - Ubuntu 24.04 LTS
> - NVIDIA RTX A4000
> - TensorRT 11.1
> - CUDA 13.3
> - Goal: Convert `.pt` / `.onnx` → TensorRT `.engine`

---

# Overall Workflow

```text
Install NVIDIA Driver
        ↓
Verify GPU
        ↓
Install CUDA Toolkit
        ↓
Verify CUDA (nvcc)
        ↓
Install cuDNN (if required)
        ↓
Download TensorRT (.deb Local Repo)
        ↓
Install TensorRT
        ↓
Verify trtexec
        ↓
Verify Python API
        ↓
Export PyTorch → ONNX
        ↓
Build TensorRT Engine
        ↓
Benchmark
        ↓
Python Inference
```

---

# STEP 1 - Verify NVIDIA Driver

```bash
nvidia-smi
```

Expected:

- GPU Name
- Driver Version
- CUDA Version

If command fails:

```bash
ubuntu-drivers devices
sudo ubuntu-drivers autoinstall
sudo reboot
```

Verify again:

```bash
nvidia-smi
```

---

# STEP 2 - Verify Ubuntu Version

```bash
lsb_release -a
```

Expected:

```
Ubuntu 24.04 LTS
```

---

# STEP 3 - Verify CUDA Toolkit

```bash
nvcc --version
```

## Case A

Output:

```
Cuda compilation tools...
```

Go to TensorRT installation.

## Case B

Output:

```
command not found
```

Run:

```bash
which nvcc
find /usr -name nvcc 2>/dev/null
ls /usr/local/
echo $PATH
```

### If nvcc is found

Example:

```
/usr/local/cuda-13.3/bin/nvcc
```

Fix PATH

Temporary

```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

Permanent

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

Verify

```bash
nvcc --version
```

---

### If nvcc is NOT found

Install CUDA Toolkit.

Download the official CUDA 13.3 local installer.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-ubuntu2404.pin

sudo mv cuda-ubuntu2404.pin /etc/apt/preferences.d/cuda-repository-pin-600

wget https://developer.download.nvidia.com/compute/cuda/13.3.1/local_installers/cuda-repo-ubuntu2404-13-3-local_13.3.1-610.43.02-1_amd64.deb

sudo dpkg -i cuda-repo-ubuntu2404-13-3-local_13.3.1-610.43.02-1_amd64.deb

sudo cp /var/cuda-repo-ubuntu2404-13-3-local/cuda-*-keyring.gpg /usr/share/keyrings/

sudo apt update

sudo apt install -y cuda-toolkit-13-3
```

Verify

```bash
nvcc --version
```

---

# STEP 4 - Verify cuDNN

```bash
find /usr -name "libcudnn.so*" 2>/dev/null
```

If nothing is returned, install the cuDNN package compatible with your installed CUDA Toolkit before proceeding.

---

# STEP 5 - Download TensorRT

Download from NVIDIA Developer:

- TensorRT 11.1.0
- Ubuntu 24.04
- CUDA 13.0–13.3
- **DEB Local Repository Package**

Example filename:

```
nv-tensorrt-local-repo-ubuntu2404-11.1.0-cuda-13.x_*.deb
```

---

# STEP 6 - Install TensorRT Repository

```bash
cd ~/Downloads

sudo dpkg -i nv-tensorrt-local-repo-ubuntu2404-*.deb
```

Copy repository key

```bash
sudo cp /var/nv-tensorrt-local-repo-*/nv-tensorrt-local-*-keyring.gpg /usr/share/keyrings/
```

Update package list

```bash
sudo apt update
```

Install TensorRT

```bash
sudo apt install -y tensorrt
```

---

# STEP 7 - Verify TensorRT

CLI

```bash
trtexec --version
```

Python

```bash
python3 -c "import tensorrt as trt; print(trt.__version__)"
```

Installed packages

```bash
dpkg -l | grep nvinfer
```

---

# STEP 8 - Install Python Packages

```bash
pip install --upgrade pip

pip install torch torchvision

pip install onnx

pip install onnxruntime
```

---

# STEP 9 - Export PyTorch to ONNX

```python
torch.onnx.export(
    model,
    dummy_input,
    "model.onnx",
    opset_version=17
)
```

---

# STEP 10 - Build TensorRT Engine

FP16

```bash
trtexec \
--onnx=model.onnx \
--saveEngine=model.engine \
--fp16
```

FP32

```bash
trtexec \
--onnx=model.onnx \
--saveEngine=model_fp32.engine
```

Dynamic Shape

```bash
trtexec \
--onnx=model.onnx \
--minShapes=input:1x3x640x640 \
--optShapes=input:4x3x640x640 \
--maxShapes=input:8x3x640x640 \
--saveEngine=model.engine
```

---

# STEP 11 - Benchmark

```bash
trtexec --loadEngine=model.engine
```

---

# STEP 12 - Common Troubleshooting

## nvidia-smi not found

→ Install NVIDIA Driver.

## nvcc not found

```
which nvcc
find /usr -name nvcc
```

If found → Fix PATH.

If not found → Install CUDA Toolkit.

## trtexec not found

Verify TensorRT installation:

```bash
dpkg -l | grep nvinfer
find / -name trtexec 2>/dev/null
```

## Python cannot import TensorRT

```bash
python3 -c "import tensorrt"
```

If it fails, reinstall the TensorRT Python bindings.

---

# Recommended Learning Order

1. NVIDIA Driver
2. CUDA Toolkit
3. cuDNN
4. TensorRT Installation
5. trtexec
6. TensorRT Python API
7. ONNX
8. Engine Building
9. Dynamic Shapes
10. FP16 / INT8
11. Performance Optimization
12. Production Deployment
