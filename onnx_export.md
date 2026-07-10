# Semantic Segmentation ONNX Export Investigation Checklist

## 1. Verify Environment Versions

Run on both **Local** and **Docker**.

```python
import torch
import ultralytics
import onnx
import onnxruntime as ort

print("Torch:", torch.__version__)
print("Ultralytics:", ultralytics.__version__)
print("ONNX:", onnx.__version__)
print("ONNX Runtime:", ort.__version__)
print("CUDA:", torch.version.cuda)
```

---

## 2. Export Using the Same Parameters

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    batch=16,
    dynamic=True,
    device="cuda"
)
```

Also test

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    batch=16,
    dynamic=True,
    device="cuda",
    simplify=False,
    opset=17
)
```

---

## 3. Compare ONNX File Size

```bash
ls -lh model.onnx
```

---

## 4. Compare SHA256 Hash

```bash
sha256sum model.onnx
```

---

## 5. Compare ONNX Metadata

```python
import onnx

model = onnx.load("model.onnx")

print("IR Version:", model.ir_version)

for op in model.opset_import:
    print("Opset:", op.version)
```

---

## 6. Compare Number of Graph Nodes

```python
import onnx

model = onnx.load("model.onnx")

print(len(model.graph.node))
```

---

## 7. Verify Model Inputs

```python
for inp in model.graph.input:
    print(inp)
```

---

## 8. Verify Model Outputs

```python
for out in model.graph.output:
    print(out)
```

---

## 9. Validate the ONNX Model

```python
import onnx

model = onnx.load("model.onnx")

onnx.checker.check_model(model)

print("Model is valid.")
```

---

## 10. Infer Shapes

```python
import onnx

model = onnx.load("model.onnx")

model = onnx.shape_inference.infer_shapes(model)

print("Shape inference completed.")
```

---

## 11. Compare ONNX Runtime Output Statistics

```python
outputs = session.run(None, {input_name: input_tensor})

for i, output in enumerate(outputs):
    print(f"\nOutput {i}")
    print("Shape :", output.shape)
    print("Dtype :", output.dtype)
    print("Min   :", output.min())
    print("Max   :", output.max())
    print("Mean  :", output.mean())
```

---

## 12. Verify PT Model Prediction

Run inference using the original `.pt` model in both environments.

```python
results = model.predict(
    source=image_path,
    imgsz=(1024, 2048)
)
```

---

## 13. Test Local ONNX Inside Docker

1. Export ONNX locally.
2. Copy the ONNX file into the Docker container.
3. Run inference using ONNX Runtime.

If prediction is correct, the Docker export process is the issue.

---

## 14. Compare ONNX Graphs in Netron

Inspect both models and compare:

* Input tensor
* Output tensor
* Graph nodes
* Last layers
* Segmentation head
* Dynamic dimensions

---

## 15. Test Different Opsets

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    opset=16
)
```

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    opset=17
)
```

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    opset=18
)
```

---

## 16. Test Without Dynamic Shapes

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    dynamic=False
)
```

---

## 17. Test Batch Size

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    batch=1
)
```

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    batch=16
)
```

---

## 18. Test Without Simplification

```python
model.export(
    format="onnx",
    imgsz=(1024, 2048),
    simplify=False
)
```

---

## 19. Verify ONNX Runtime Providers

```python
import onnxruntime as ort

print(ort.get_available_providers())
```

---

## 20. Compare Preprocessing Tensor

Before inference, print the input tensor.

```python
print(input_tensor.shape)
print(input_tensor.dtype)
print(input_tensor.min())
print(input_tensor.max())
```

---

## 21. Compare Final Output Mask

Before visualization, print the mask statistics.

```python
print(mask.shape)
print(mask.dtype)
print(mask.min())
print(mask.max())
print(mask.mean())
```

---

## 22. Compare the Export Log

Save the complete export log from both Local and Docker and compare:

* Ultralytics version
* PyTorch version
* ONNX version
* Opset
* Dynamic shapes
* Simplification
* Export warnings
* Export success messages

---

## Goal

Determine whether the difference is introduced by:

* Environment versions
* ONNX export process
* Export parameters
* Generated ONNX graph
* ONNX Runtime execution
* Dynamic shapes
* Opset
* Graph simplification
