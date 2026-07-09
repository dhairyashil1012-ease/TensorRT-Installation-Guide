# TensorRT Docker Workflow (Ubuntu)
## Reference

- TensorRT Container Installation Guide:
  https://docs.nvidia.com/deeplearning/tensorrt/latest/installing-tensorrt/install-container.html

- TensorRT Quick Start Guide:
  https://docs.nvidia.com/deeplearning/tensorrt/latest/getting-started/quick-start-guide.html

This guide explains how to use the NVIDIA TensorRT Docker container for converting ONNX models into TensorRT engines and running inference inside Jupyter Notebook.

---

# Prerequisites

Before starting, ensure the following are installed on your system:

- Ubuntu
- NVIDIA GPU
- NVIDIA Driver
- Docker
- NVIDIA Container Toolkit

Verify that Docker can access your GPU:

```bash
docker run --rm --gpus all nvidia/cuda:13.0.0-base-ubuntu24.04 nvidia-smi
```

If your GPU information is displayed, your setup is ready.

---

# Step 1: Pull the TensorRT Docker Image

Download the latest TensorRT Docker image from NVIDIA NGC.

```bash
docker pull nvcr.io/nvidia/tensorrt:26.06-py3
```

Verify the downloaded image:

```bash
docker images
```

---

# Step 2: Launch the TensorRT Container

Run the TensorRT container with GPU support, mount your current working directory, and expose Jupyter Notebook on port **8888**.

```bash
docker run -it --rm \
    --gpus all \
    -p 8888:8888 \
    -v $(pwd):/workspace \
    -w /workspace \
    nvcr.io/nvidia/tensorrt:26.06-py3 \
    bash
```

### Command Explanation

| Option | Description |
|---------|-------------|
| `-it` | Interactive terminal |
| `--rm` | Remove container after exiting |
| `--gpus all` | Enable GPU support |
| `-p 8888:8888` | Map Jupyter Notebook port |
| `-v $(pwd):/workspace` | Mount current directory inside container |
| `-w /workspace` | Set working directory |

---

# Step 3: Verify GPU Inside the Container

```bash
nvidia-smi
```

This should display your GPU details.

---

# Step 4: Convert Classification ONNX Model to TensorRT Engine

Navigate to the classification model directory.

```bash
cd Classification_Model_Evaluation
cd "Classification _Models"
```

Create the TensorRT engine.

```bash
trtexec \
    --onnx=yolo26n-cls.onnx \
    --minShapes=images:1x3x224x224 \
    --optShapes=images:4x3x224x224 \
    --maxShapes=images:16x3x224x224 \
    --saveEngine=yolo26n-cls.engine
```

Verify the generated engine:

```bash
ls -lh yolo26n-cls.engine
```

---

# Step 5: Convert Detection ONNX Model to TensorRT Engine

Navigate to the detection model directory.

```bash
cd ../../Detection_Model_Evaluation/Detection_Models
```

Run:

```bash
trtexec \
    --onnx=yolo26n.onnx \
    --minShapes=images:1x3x640x640 \
    --optShapes=images:4x3x640x640 \
    --maxShapes=images:16x3x640x640 \
    --saveEngine=yolo26n.engine
```

Verify the engine:

```bash
ls -lh yolo26n.engine
```

---

# Step 6: Convert Semantic Segmentation ONNX Model to TensorRT Engine

Navigate to the segmentation model directory.

```bash
cd ../../Semantic_Segmentation_Model_Evaluation/Sem_Models
```

Run:

```bash
trtexec \
    --onnx=yolo26n-sem.onnx \
    --minShapes=images:1x3x1024x2048 \
    --optShapes=images:4x3x1024x2048 \
    --maxShapes=images:16x3x1024x2048 \
    --saveEngine=yolo26n-sem.engine
```

Verify the generated engine.

```bash
ls -lh yolo26n-sem.engine
```

---

# Step 7: Install Python Dependencies

Install all required Python packages.

```bash
pip install -r requirements.txt
```

---

# Step 8: Install Required System Libraries

Some packages (such as OpenCV) require additional system libraries.

```bash
apt-get update && apt-get install -y \
    libgl1 \
    libglib2.0-0 \
    libsm6 \
    libxext6 \
    libxrender1 \
    libxcb1
```

---

# Step 9: Launch Jupyter Notebook

Start Jupyter Notebook inside the container.

```bash
jupyter notebook \
    --ip=0.0.0.0 \
    --port=8888 \
    --allow-root \
    --no-browser
```

Open your browser and navigate to:

```
http://localhost:8888
```

Copy the authentication token from the terminal and paste it into your browser.

---

# Docker Container Management

## List Running Containers

```bash
docker ps
```

---

## List All Containers

```bash
docker ps -a
```

---

## Attach to a Running Container

```bash
docker attach <container_id>
```

Example:

```bash
docker attach a66e96f37106
```

---

## Open a New Shell Inside a Running Container

```bash
docker exec -it <container_id> bash
```

Example:

```bash
docker exec -it a66e96f37106 bash
```

---

## Start a Stopped Container

```bash
docker start <container_id>
```

Example:

```bash
docker start a66e96f37106
```

---

## Remove a Container

```bash
docker rm <container_id>
```

Example:

```bash
docker rm 92c6bd6c2ed6
```

---

# Troubleshooting

## Check Which Process Is Using Port 8888

```bash
sudo lsof -i :8888
```

---

## Kill the Process

```bash
kill <PID>
```

Force kill if necessary.

```bash
kill -9 <PID>
```

---

## Kill All Running Jupyter Notebook Processes

```bash
pkill -f jupyter-notebook
```

---

# Useful Commands

Display command history.

```bash
history
```

Clear the terminal.

```bash
clear
```

List files.

```bash
ls
```

Check GPU status.

```bash
nvidia-smi
```

Exit the container.

```bash
exit
```

---

# Workflow Summary

```text
Host Machine
│
├── Pull TensorRT Docker Image
│
├── Launch TensorRT Container
│
├── Verify GPU (nvidia-smi)
│
├── Install Python Requirements
│
├── Install System Libraries
│
├── Convert Classification ONNX → TensorRT Engine
│
├── Convert Detection ONNX → TensorRT Engine
│
├── Convert Segmentation ONNX → TensorRT Engine
│
├── Verify Generated Engine Files
│
├── Launch Jupyter Notebook
│
└── Run TensorRT Inference
```