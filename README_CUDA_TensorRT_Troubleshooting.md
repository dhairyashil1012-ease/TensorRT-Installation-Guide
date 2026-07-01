# CUDA Toolkit & TensorRT Installation Troubleshooting Guide (Ubuntu)

# Goal

Convert a PyTorch (`.pt`) or ONNX (`.onnx`) model into a TensorRT (`.engine`) model.

Pipeline:

```text
PyTorch (.pt)
      │
      ▼
Export to ONNX
      │
      ▼
TensorRT (.engine)
```

---

# Step 1 - Check NVIDIA Driver

Run:

```bash
nvidia-smi
```

Expected:

- GPU is detected
- Driver version is shown
- CUDA Version is shown

Example:

```text
Driver Version : 570.xx
CUDA Version   : 13.x
GPU            : RTX A4000
```

✅ If this works:
- NVIDIA Driver is installed.

❌ If this command fails:
- Install the NVIDIA Driver first.
- Do not continue until `nvidia-smi` works.

---

# Important

The CUDA version shown by `nvidia-smi` is the maximum CUDA version supported by the driver.

It DOES NOT mean the CUDA Toolkit is installed.

Example:

```text
nvidia-smi
CUDA Version : 13.x
```

This is completely normal even if `nvcc` is missing.

---

# Step 2 - Check CUDA Toolkit

Run:

```bash
nvcc --version
```

## Case A

Output:

```text
Cuda compilation tools ...
```

✅ CUDA Toolkit is installed.

Proceed to TensorRT checks.

---

## Case B

Output:

```text
command not found
```

Need further diagnosis.

Run:

```bash
which nvcc
```

---

# Step 3 - Diagnose

## Check 1

```bash
which nvcc
```

### If output is

```text
/usr/local/cuda/bin/nvcc
```

CUDA exists.

Proceed to Step 4.

### If no output

Run:

```bash
find /usr -name nvcc 2>/dev/null
```

---

## Check 2

If output is

```text
/usr/local/cuda-12.9/bin/nvcc
```

or

```text
/usr/local/cuda/bin/nvcc
```

CUDA exists.

Only PATH is broken.

Proceed to Step 4.

---

## Check 3

If nothing is returned

CUDA Toolkit is NOT installed.

Proceed to Step 5.

---

# Step 4 - Fix PATH

Check current PATH

```bash
echo $PATH
```

If it does NOT contain

```text
/usr/local/cuda/bin
```

Add CUDA to PATH.

Open your shell configuration:

For Bash:

```bash
nano ~/.bashrc
```

For Zsh:

```bash
nano ~/.zshrc
```

Add:

```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

Save.

Reload:

```bash
source ~/.bashrc
```

or

```bash
source ~/.zshrc
```

Verify:

```bash
nvcc --version
```

If it works, the issue is solved.

---

# Step 5 - Check CUDA Installation Directory

Run:

```bash
ls /usr/local/
```

Expected examples:

```text
cuda
cuda-12.9
```

If no CUDA folder exists

CUDA Toolkit is not installed.

Proceed to Step 6.

---

# Step 6 - Install CUDA Toolkit

Install the CUDA Toolkit version that is compatible with the TensorRT release you plan to use.

After installation verify:

```bash
nvcc --version
```

Expected:

```text
Cuda compilation tools ...
```

---

# Step 7 - Verify cuDNN

Run:

```bash
find /usr -name "libcudnn.so*" 2>/dev/null
```

If libraries are found

✅ cuDNN is installed.

Otherwise install the matching cuDNN package.

---

# Step 8 - Verify TensorRT

Run:

```bash
trtexec --version
```

## If it works

TensorRT is installed.

---

## If

```text
command not found
```

Install TensorRT.

Verify again:

```bash
trtexec --version
```

---

# Step 9 - Verify Python TensorRT

Run:

```bash
python3 -c "import tensorrt as trt; print(trt.__version__)"
```

If version is printed

TensorRT Python API is installed.

Otherwise install the Python bindings.

---

# Step 10 - Convert ONNX to TensorRT

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
--saveEngine=model.engine
```

---

# Step 11 - Benchmark Engine

```bash
trtexec \
--loadEngine=model.engine
```

---

# Complete Decision Tree

```text
nvidia-smi
      │
      ▼
Works?
 ├── No → Install NVIDIA Driver
 └── Yes
        │
        ▼
nvcc --version
        │
        ├── Works
        │      │
        │      ▼
        │   Check TensorRT
        │
        └── command not found
               │
               ▼
         which nvcc
               │
     ├─────────┴─────────┐
     │                   │
Found                 Not Found
     │                   │
     ▼                   ▼
Fix PATH          find /usr -name nvcc
                          │
               ├──────────┴──────────┐
               │                     │
             Found              Not Found
               │                     │
               ▼                     ▼
            Fix PATH          Install CUDA Toolkit
```

