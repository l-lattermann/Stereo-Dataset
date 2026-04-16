# Dataset Description

This dataset contains stereo image pairs captured with an RC Cube–based setup, along with derived disparity maps, filtered 3D point clouds, and camera parameters. Each sample represents a single scene with one or more objects under varying spatial configurations (e.g., occlusions, stacking, or contact situations).

The dataset is designed for:
- stereo depth estimation evaluation  
- point cloud reconstruction  
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
│   ├── <sample_name>_params.txt
│   ├── <sample_name>_pointcloud.ply
│   ├── <sample_name>.npz
```

---

## File Descriptions

### 1. Left / Right Images

```
<sample_name>_left_img.png
<sample_name>_right_img.png
```

- RGB stereo image pair  
- Resolution: depends on capture setup  
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

### 4. Point Cloud

```
<sample_name>_pointcloud.ply
```

- Binary little-endian PLY format  
- Contains filtered 3D points reconstructed from disparity  

Structure:
- `x, y, z` (float32, meters)
- `red, green, blue` (uint8)

---

### 5. Model-Ready Data (.npz)

```
<sample_name>.npz
```

This is the main file for downstream processing.

#### Contents:

```python
rgb            # (H, W, 3) uint8
disp           # (H, W) float32
valid_mask     # (H, W) uint8 (0/1)

points_xyz     # (N, 3) float32
points_rgb     # (N, 3) uint8

fx, fy         # float32
cx, cy         # float32
baseline       # float32
rho            # float32 (fx * baseline)
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

## Filtering

Points included satisfy:

- disparity:
```
MIN_DISP < disp < MAX_DISP_VALID
```

- depth:
```
MIN_DEPTH < z < MAX_DEPTH
```

---

## Naming Convention

All samples are translated and normalized into English.

Examples:
- `blaue box offen` → `open_blue_box`
- `kabel schwarz aufgerollt` → `coiled_black_cable`
- `flasche steht auf ladekabel` → `bottle_on_charging_cable`

---

## Notes

- `.npz` is the recommended format for ML pipelines  
- `.ply` is for visualization (Open3D, MeshLab)  
- disparity `.png` is only for inspection  

---

## Intended Use

- stereo matching benchmarking  
- point cloud–based perception  
- grasp generation (e.g., Contact-GraspNet)  
- multimodal reasoning (image + geometry)
