# Dataset Description

This dataset contains stereo image pairs captured with an RC Cube–based setup, along with derived disparity maps, **organized 3D point clouds**, and **triangle meshes** reconstructed from stereo depth.

Each sample represents a single scene with one or more objects under varying spatial configurations (e.g., occlusions, stacking, or contact situations).

The dataset is designed for:
- stereo depth estimation evaluation  
- point cloud reconstruction  
- surface reconstruction (mesh generation)  
- robotic perception and grasping research  
- downstream tasks such as segmentation or grasp selection  

All samples are preprocessed into a unified, model-friendly format.

---

## Directory Structure

```
dataset/
├── <sample_name>/
│   ├── <sample_name>_left_img.png
│   ├── <sample_name>_right_img.png
│   ├── <sample_name>_disparity_img.png
│   ├── <sample_name>_pointcloud.ply
│   ├── <sample_name>.npz
├── camera_parameters.txt
├── README.md
```

---

## File Descriptions

### 1. Left / Right Images

```
<sample_name>_left_img.png
<sample_name>_right_img.png
```

- RGB stereo image pair  
- Resolution: typically ~1280×960  
- Format: 8-bit PNG  

---

### 2. Disparity Image

```
<sample_name>_disparity_img.png
```

- Normalized visualization of disparity  
- Used for debugging and qualitative inspection  
- Not intended for quantitative use (use `.npz` instead)

---

### 3. Camera Parameters

```
<sample_name>_params.txt
```

Contains relevant stereo calibration parameters (subset of RC Cube output):

- `camera.A` → intrinsic matrix  
- `t` → baseline  
- optional:
  - disparity scale / offset  
  - invalid value markers  

---

### 4. Point Cloud + Mesh

```
<sample_name>_pointcloud.ply
```

Binary little-endian PLY format.

#### Vertex attributes:
- `x, y, z` (float32, meters)
- `scan_size` (float32)
- `diffuse_red, diffuse_green, diffuse_blue` (uint8)

#### Mesh:
- Triangle faces connecting neighboring pixels  
- Built from organized stereo grid  
- Invalid or discontinuous regions are filtered using:
  - depth threshold
  - edge length constraint
  - depth discontinuity constraint

---

### 5. Model-Ready Data (.npz)

```
<sample_name>.npz
```

This is the main file for downstream processing.

#### Contents:

```python
rgb        # (H, W, 3) uint8 — left RGB image

xyz        # (H, W, 3) float32 — per-pixel 3D coordinates in meters (camera frame)
           # [X, Y, Z] where Z = depth

label      # (H, W) int32 — validity / segmentation mask (1 = valid point, 0 = invalid)

fx, fy     # float32 (scalar) — focal lengths
cx, cy     # float32 (scalar) — principal point

width      # int32 (scalar) — image width
height     # int32 (scalar) — image height

baseline   # float32 (scalar) — stereo baseline (meters)
rho        # float32 (scalar) — disparity-to-depth factor (fx * baseline)
```

---

## Coordinate System

Depth is computed as:

```
z = (fx * baseline) / disparity
```

3D projection:

```
x = (u - cx) * z / fx
y = -(v - cy) * z / fy
```

Units:
- meters

---

## Mesh Construction

The mesh is generated from the organized stereo grid:

- Each pixel cell → 2 triangles
- Triangles are **discarded if**:
  - depth difference too large 
  - edge length too large 

This prevents:
- connections between foreground/background
- artifacts across occlusions

---


## Notes

- `.npz` is the recommended format for ML pipelines  
- `.ply` contains both **geometry and mesh** for visualization  
- disparity `.png` is only for inspection  

---

## Intended Use

- stereo matching benchmarking  
- point cloud–based perception  
- mesh reconstruction from stereo  
- grasp generation (e.g., Contact-GraspNet)  
- multimodal reasoning (image + geometry)
