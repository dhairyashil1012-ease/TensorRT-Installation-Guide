# TensorRT Setup & Troubleshooting (Ubuntu 24.04 + RTX A4000)

> **Target System**
>
> - Ubuntu 24.04 LTS
> - NVIDIA RTX A4000
> - NVIDIA Driver Installed
> - Goal: Convert `.pt` / `.onnx` → `.engine`

---

# 1. Verify NVIDIA Driver

```bash
nvidia-smi
```

If this fails:

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

# 2. Check CUDA Toolkit

```bash
nvcc --version
```

## Case A: Works

Continue to TensorRT.

## Case B: `command not found`

Run:

```bash
which nvcc
find /usr -name nvcc 2>/dev/null
ls /usr/local/
echo $PATH
```

### If `find` returns something like:

```text
/usr/local/cuda/bin/nvcc
```

CUDA Toolkit exists. Fix PATH.

Temporary:

```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

Permanent:

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
nvcc --version
```

---

### If `find` returns nothing

CUDA Toolkit is **not installed**.

## Install CUDA Toolkit

> Download the Ubuntu 24.04 installer from the official NVIDIA CUDA Toolkit page matching your desired CUDA 13.x release.

Then install the downloaded local repository package (replace the filename with the one you downloaded):

```bash
sudo dpkg -i cuda-repo-ubuntu2404-13-*.deb
sudo cp /var/cuda-repo-ubuntu2404-13-*/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt update
sudo apt install -y cuda-toolkit-13
```

Reload shell:

```bash
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
nvcc --version
```

---

# 3. Verify cuDNN

```bash
find /usr -name "libcudnn.so*" 2>/dev/null
```

If nothing is found, install the cuDNN package that matches your installed CUDA Toolkit from NVIDIA.

---

# 4. Install TensorRT

Download the Ubuntu 24.04 TensorRT local repository package matching your CUDA version from NVIDIA.

Install:

```bash
sudo dpkg -i nv-tensorrt-local-repo-ubuntu2404-*.deb
sudo cp /var/nv-tensorrt-local-repo-*/**/*keyring.gpg /usr/share/keyrings/ 2>/dev/null || true
sudo apt update
sudo apt install -y tensorrt
```

Install Python bindings:

```bash
sudo apt install -y python3-libnvinfer python3-libnvinfer-dev
```

---

# 5. Verify TensorRT

CLI:

```bash
trtexec --version
```

Python:

```bash
python3 -c "import tensorrt as trt; print(trt.__version__)"
```

If `trtexec` is not found:

```bash
find / -name trtexec 2>/dev/null
```

If found, add its directory to PATH.

---

# 6. Python Packages

```bash
pip install --upgrade pip
pip install onnx onnxruntime
```

For PyTorch export:

```bash
pip install torch torchvision
```

---

# 7. Export PyTorch to ONNX

```python
torch.onnx.export(
    model,
    dummy_input,
    "model.onnx",
    opset_version=17
)
```

---

# 8. ONNX → TensorRT Engine

FP16:

```bash
trtexec \
--onnx=model.onnx \
--saveEngine=model.engine \
--fp16
```

FP32:

```bash
trtexec \
--onnx=model.onnx \
--saveEngine=model_fp32.engine
```

Dynamic shapes example:

```bash
trtexec \
--onnx=model.onnx \
--minShapes=input:1x3x640x640 \
--optShapes=input:4x3x640x640 \
--maxShapes=input:8x3x640x640 \
--saveEngine=model.engine
```

---

# 9. Benchmark

```bash
trtexec --loadEngine=model.engine
```

---

# Decision Tree

```text
nvidia-smi
   │
   ├── Fails → Install NVIDIA Driver
   │
   └── Works
         │
         ▼
   nvcc --version
         │
         ├── Works → Install TensorRT
         │
         └── Not Found
               │
               ▼
        find /usr -name nvcc
               │
      ├────────┴────────┐
      │                 │
   Found             Not Found
      │                 │
  Fix PATH       Install CUDA Toolkit
```
