# 🤖 Bài 42: Multi-Camera Calibration Graph Simulator — Bộ mô phỏng graph hiệu chuẩn nhiều camera cho Humanoid Robot AI Perception

> Mini Project số 42 trong **Đợt 9**  
> **Bài 42 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9** và đi tiếp trực tiếp từ **Bài 41**.
>
> Nếu **Bài 41** tập trung vào:
>
> ```text
> distorted pixel
> → undistort
> → normalize
> → back-project
> → reconstruct 3D point
> ```
>
> thì **Bài 42** sẽ mở rộng từ **một camera** sang **nhiều camera** và thêm đúng tinh thần “Geometry simulator” của Đợt 9:
>
> ```text
> nhiều camera
> → calibration graph giữa các camera / frame / plane
> → observation từ nhiều góc nhìn
> → BFS / DFS để truy vết quan hệ
> → back-projection / plane intersection trên từng camera
> → so sánh kết quả hình học đa camera
> ```
>
> Đây là bước rất quan trọng vì perception thật trên robot không chỉ có **một camera cô lập**, mà thường là:
> - stereo pair
> - RGB + depth
> - head camera + wrist camera
> - nhiều frame quy chiếu cần nối với nhau

---

# 📌 Mục lục

- [1. Vì sao Bài 42 xuất hiện sau Bài 41](#1-vì-sao-bài-42-xuất-hiện-sau-bài-41)
- [2. Đợt 9 đang học gì](#2-đợt-9-đang-học-gì)
- [3. Bài 42 nâng từ Bài 41 lên chỗ nào](#3-bài-42-nâng-từ-bài-41-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật hình học của project](#11-luật-hình-học-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 42 xuất hiện sau Bài 41

## Bài 41 đang làm gì?
Bài 41 đã cho bạn pipeline:

```text
1 camera
→ distorted pixel
→ undistort
→ normalize
→ back-project ray
→ reconstruct 3D point
→ compare with ground truth
```

Nó giúp bạn hiểu **một camera** hoạt động ra sao.

## Bài 42 nâng lên ở đâu?
Bài 42 sẽ trả lời câu hỏi:

> Nếu robot có **nhiều camera / nhiều frame camera**, thì làm sao quản lý quan hệ calibration giữa chúng và dùng chúng để xử lý cùng một landmark / plane?

Thay vì chỉ có:
- `camera_0`

bạn sẽ có ví dụ:
- `left_camera`
- `right_camera`
- `head_camera`
- `wrist_camera`

và giữa chúng sẽ có các quan hệ:
- extrinsic transform
- shared plane observations
- cùng quan sát một landmark

Bài 42 vì vậy là bước chuyển từ:

```text
single-camera geometry
```

sang:

```text
multi-camera calibration graph
```

---

# 2. Đợt 9 đang học gì

Theo roadmap của bạn, **Đợt 9** gồm:

## Python
### Phase 8 — Data Structures
- `Binary Search Tree`
- `Graph`
  - graph representation
  - BFS / DFS

### Phase 9 — Python Libraries
- `NumPy`
  - array
  - shape
  - indexing
  - math operations

## C++
### Phase 6 — OOP
- `Virtual Functions`
  - polymorphism
- `Enums`

## Computer Vision
### Phase 4 — Camera Geometry
- `Back Projection 2D → 3D`
- `Camera Calibration`
- `Undistortion`

## Vì sao Bài 42 bám rất sát Đợt 9?
Vì project này dùng gần như trọn bộ:
- **Graph + BFS / DFS** → graph calibration giữa nhiều camera / frame
- **BST** → sắp xếp sample / frame / observation id
- **NumPy** → preview matrix extrinsic / intrinsic / projection
- **polymorphism + enum** → solver cho nhiều loại observation / nhiều camera mode
- **camera calibration + back projection + undistortion** → lõi perception

---

# 3. Bài 42 nâng từ Bài 41 lên chỗ nào

## Bài 41
- 1 camera
- 1 calibration model
- undistort
- back-project
- plane/depth reconstruction

## Bài 42
- nhiều camera
- mỗi camera có:
  - intrinsic riêng
  - distortion riêng
  - extrinsic riêng
- build **camera calibration graph**
- mỗi observation biết nó đến từ camera nào
- có thể so sánh:
  - cùng một landmark được reconstruct từ camera A và camera B
  - cùng một plane được nhìn bởi nhiều camera

Tức là Bài 42 thêm:
1. **multi-camera structure**
2. **graph reasoning**
3. **cross-camera geometry evaluation**

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Multi-Camera Calibration Graph Simulator**

System này sẽ có:

## Một tập camera
Ví dụ:
- `left_camera`
- `right_camera`
- `head_camera`

Mỗi camera có:
- `intrinsic`
- `distortion`
- `extrinsic` (pose của camera trong robot/world frame)

## Một tập plane / landmark
Ví dụ:
- `ground_plane`
- `table_plane`
- `marker_01`

## Một tập observations
Mỗi observation sẽ chứa:
- `camera_id`
- pixel distorted `(u_d, v_d)`
- `mode`
  - `DEPTH_MODE`
  - `PLANE_MODE`
- `plane_id` hoặc `known_depth`
- optional ground truth 3D

## Nhiệm vụ của system
1. load toàn bộ camera models
2. build **camera graph**
3. với mỗi observation:
   - chọn camera tương ứng
   - undistort pixel
   - normalize
   - back-project
   - reconstruct 3D
4. gom kết quả theo camera / landmark / plane
5. so sánh chất lượng giữa các camera

<p align="center">
  <img src="images/project_42.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build camera graph config
→ build camera intrinsics / distortion / extrinsic config
→ build plane / landmark observation set
→ preview graph bằng BFS / DFS
→ preview matrix bằng NumPy

C++
→ load multi-camera calibration table
→ build calibration graph
→ choose camera model per observation
→ undistort
→ back-project
→ reconstruct 3D
→ compare cross-camera results
→ export geometry reports
```

Mục tiêu không chỉ là “multi-camera cho vui”, mà là:
- hiểu cách robot perception quản lý **nhiều camera**
- hiểu observation phải gắn với **frame / camera cụ thể**
- chuẩn bị nền cho:
  - stereo rig
  - head + wrist camera
  - camera fusion
  - multi-view perception

---

# 6. Pipeline tổng thể

```text
Load Multi-Camera Intrinsics Config
Load Multi-Camera Distortion Config
Load Multi-Camera Extrinsic Config
Load Plane / Landmark Config
Load Observation Config

Build Camera Calibration Graph
Create MultiCameraGeometryRunner

For each observation:
    Step 1:
        identify source camera_id

    Step 2:
        fetch corresponding camera model

    Step 3:
        undistort pixel

    Step 4:
        normalize pixel

    Step 5:
        if DEPTH_MODE:
            back-project using known depth
        else if PLANE_MODE:
            intersect ray with plane

    Step 6:
        store reconstructed 3D result

After all observations:
    compare results grouped by camera
    compare results grouped by landmark / plane
    export reports
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- **BST**
- **Graph + BFS / DFS**
- **NumPy**
- config builder

## C++
- class / inheritance
- virtual functions / polymorphism
- enum class
- `std::vector`
- `std::unordered_map`
- `std::queue`
- `std::stack`
- multi-camera geometry runtime design

## Computer Vision / Geometry
- camera intrinsics / distortion / extrinsic
- undistortion
- normalized coordinate
- back projection
- plane intersection
- camera-to-world / world-to-camera thinking

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Graph phải biểu diễn **calibration graph** hoặc **camera relation graph**.

Ví dụ:
```text
left_camera  --(baseline transform)--> right_camera
head_camera  --(mount transform)-----> robot_head_frame
wrist_camera --(mount transform)-----> robot_wrist_frame
```

Bạn phải có:
- adjacency representation
- BFS
- DFS

---

## 2. Python BST
Dùng để lưu `observation_id`, `frame_id`, hoặc `camera sample id`.

---

## 3. `std::unordered_map<std::string, CameraCalibrationModel>`
Map `camera_id -> calibration model`.

## 4. `std::unordered_map<std::string, CameraExtrinsic>`
Map `camera_id -> extrinsic`.

## 5. `std::vector<MultiCameraObservation>`
Danh sách observation đa camera.

## 6. `std::vector<MultiCameraGeometryResult>`
Danh sách kết quả dựng hình.

## 7. `std::stack<MultiCameraDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Camera selection
Với mỗi observation:
- đọc `camera_id`
- lấy đúng calibration model tương ứng

---

## Algorithm 2 — Undistortion per camera
Pixel phải được undistort theo distortion coefficients của **đúng camera đó**.

---

## Algorithm 3 — Back projection per camera
Sau khi normalize:
- nếu `DEPTH_MODE` → reconstruct từ depth
- nếu `PLANE_MODE` → ray-plane intersection

---

## Algorithm 4 — Camera extrinsic conversion
Sau khi reconstruct point ở **camera frame**, bạn phải cho phép đổi sang:
- **world frame**
hoặc
- **robot base frame**

Ví dụ:
```text
P_world = T_world_camera * P_camera
```

---

## Algorithm 5 — Cross-camera comparison
Nếu nhiều observation thuộc cùng một `landmark_id` hoặc `target_id`, hãy so sánh:
- vị trí 3D reconstructed giữa các camera
- sai số từng camera so với ground truth

---

# 9. Cấu trúc folder

```text
mini_project_42_multi_camera_calibration_graph_simulator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ camera_graph_report.txt
│     ├─ undistortion_report.txt
│     ├─ multicamera_backprojection_report.txt
│     ├─ cross_camera_comparison_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ camera_intrinsics_table.txt
│  ├─ camera_distortion_table.txt
│  ├─ camera_extrinsic_table.txt
│  ├─ geometry_plane_config.txt
│  ├─ multicamera_observation_config.txt
│  └─ geometry_runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ camera_graph_preview.py
│     ├─ observation_bst.py
│     ├─ numpy_multicamera_preview.py
│     └─ sample_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ CameraIntrinsics.hpp
   │  ├─ DistortionCoefficients.hpp
   │  ├─ CameraExtrinsic.hpp
   │  ├─ PlaneModel.hpp
   │  ├─ CameraCalibrationModel.hpp
   │  ├─ MultiCameraObservation.hpp
   │  ├─ MultiCameraGeometryResult.hpp
   │  ├─ MultiCameraDebugRecord.hpp
   │  ├─ BaseUndistortionEngine.hpp
   │  ├─ RadialUndistortionEngine.hpp
   │  ├─ BaseBackProjectionSolver.hpp
   │  ├─ DepthBackProjectionSolver.hpp
   │  ├─ PlaneIntersectionBackProjectionSolver.hpp
   │  ├─ MultiCameraCalibrationGraph.hpp
   │  ├─ MultiCameraGeometryRunner.hpp
   │  └─ MultiCameraReportWriter.hpp
   │
   └─ src/
      ├─ RadialUndistortionEngine.cpp
      ├─ DepthBackProjectionSolver.cpp
      ├─ PlaneIntersectionBackProjectionSolver.cpp
      ├─ MultiCameraCalibrationGraph.cpp
      ├─ MultiCameraGeometryRunner.cpp
      └─ MultiCameraReportWriter.cpp
```

---

# 10. Yêu cầu mini-project

# 10.1 Python — `BaseConfigBuilder`

Tạo class:

```python
class BaseConfigBuilder:
```

## Thuộc tính
```python
project_name
intrinsics_table_path
distortion_table_path
extrinsic_table_path
plane_path
observation_path
runtime_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `MultiCameraGeometryConfigBuilder`

Tạo class con:

```python
class MultiCameraGeometryConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_camera_intrinsics(camera_id, fx, fy, cx, cy, image_width, image_height)`

### `add_camera_distortion(camera_id, k1, k2, p1, p2, k3=0.0)`

### `add_camera_extrinsic(
    camera_id,
    tx, ty, tz,
    roll_deg, pitch_deg, yaw_deg
)`
- extrinsic của camera trong robot/world frame

### `add_plane(plane_id, a, b, c, d)`

### `add_observation(
    sample_id,
    camera_id,
    u_distorted,
    v_distorted,
    mode,
    target_id=None,
    known_depth=None,
    plane_id=None,
    gt_x=None, gt_y=None, gt_z=None
)`

### `set_runtime_options(
    output_world_frame,
    normalize_ray,
    enable_cross_camera_compare
)`

### `write_intrinsics_table()`
### `write_distortion_table()`
### `write_extrinsic_table()`
### `write_plane_config()`
### `write_observation_config()`
### `write_runtime_config()`

---

# 10.3 Python — `camera_graph_preview.py`

Tạo class:

```python
class CameraGraphPreview:
```

## Mục tiêu
Biểu diễn graph quan hệ giữa các camera / frame.

## Hàm cần có
### `add_edge(src_camera, dst_camera, relation_name)`
### `bfs(start_camera)`
### `dfs(start_camera)`
### `show_graph()`

---

# 10.4 Python — `observation_bst.py`

Giống Bài 41 nhưng dùng cho `sample_id` hoặc `frame_id` đa camera.

---

# 10.5 Python — `numpy_multicamera_preview.py`

Tạo class:

```python
class NumPyMultiCameraPreview:
```

## Hàm cần có

### `build_intrinsic_matrix(fx, fy, cx, cy)`
### `build_extrinsic_matrix(tx, ty, tz, roll_deg, pitch_deg, yaw_deg)`
### `transform_point_camera_to_world(point_camera, T_world_camera)`
### `normalize_pixel(u, v, fx, fy, cx, cy)`

---

# 10.6 C++ — `CameraExtrinsic`

```cpp
struct CameraExtrinsic
{
    std::string camera_id;

    double tx;
    double ty;
    double tz;

    double roll_deg;
    double pitch_deg;
    double yaw_deg;
};
```

---

# 10.7 C++ — `MultiCameraObservation`

```cpp
enum class BackProjectionMode
{
    DEPTH_MODE,
    PLANE_MODE
};

struct MultiCameraObservation
{
    std::string sample_id;
    std::string camera_id;
    std::string target_id;

    double u_distorted;
    double v_distorted;

    BackProjectionMode mode;

    bool has_known_depth;
    double known_depth;

    std::string plane_id;

    bool has_ground_truth;
    double gt_x;
    double gt_y;
    double gt_z;
};
```

---

# 10.8 C++ — `MultiCameraGeometryResult`

```cpp
struct MultiCameraGeometryResult
{
    std::string sample_id;
    std::string camera_id;
    std::string target_id;

    double u_undistorted;
    double v_undistorted;

    double x_normalized;
    double y_normalized;

    double reconstructed_x_camera;
    double reconstructed_y_camera;
    double reconstructed_z_camera;

    double reconstructed_x_world;
    double reconstructed_y_world;
    double reconstructed_z_world;

    bool success;

    double error_3d;
    double error_z;
};
```

---

# 10.9 C++ — `MultiCameraDebugRecord`

```cpp
struct MultiCameraDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.10 C++ — `MultiCameraCalibrationGraph`

Tạo class:

```cpp
class MultiCameraCalibrationGraph
```

## Thuộc tính

```cpp
private:
    std::unordered_map<std::string, CameraCalibrationModel> camera_models;
    std::unordered_map<std::string, CameraExtrinsic> camera_extrinsics;
```

## Hàm cần có

### `void add_camera_model(const std::string& camera_id, const CameraCalibrationModel& model)`
### `void add_camera_extrinsic(const CameraExtrinsic& extrinsic)`

### `const CameraCalibrationModel* get_camera_model(const std::string& camera_id) const`
### `const CameraExtrinsic* get_camera_extrinsic(const std::string& camera_id) const`

### `std::vector<std::string> get_camera_ids() const`

---

# 10.11 C++ — `BaseUndistortionEngine`
Giống Bài 41.

---

# 10.12 C++ — `BaseBackProjectionSolver`
Đổi `GeometryObservation` thành `MultiCameraObservation`, còn tinh thần giữ nguyên.

---

# 10.13 C++ — `MultiCameraGeometryRunner`

Tạo class trung tâm:

```cpp
class MultiCameraGeometryRunner
```

## Thuộc tính

```cpp
private:
    MultiCameraCalibrationGraph calibration_graph;
    std::unordered_map<std::string, PlaneModel> plane_table;

    std::vector<MultiCameraObservation> observations;

    std::shared_ptr<BaseUndistortionEngine> undistortion_engine;
    std::shared_ptr<BaseBackProjectionSolver> depth_solver;
    std::shared_ptr<BaseBackProjectionSolver> plane_solver;

    std::vector<MultiCameraGeometryResult> results;
    std::stack<MultiCameraDebugRecord> debug_history;
```

## Hàm cần có

### `load_plane_config(const std::string& path)`
### `load_observation_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each observation:
    1. find camera model by camera_id
    2. undistort pixel using that camera
    3. if DEPTH_MODE:
           solve with depth_solver
       else:
           solve with plane_solver
    4. transform reconstructed point from camera frame -> world frame
    5. compute error if GT exists
    6. save result
    7. push debug record
```

### `std::unordered_map<std::string, std::vector<MultiCameraGeometryResult>> group_results_by_camera() const`
### `std::unordered_map<std::string, std::vector<MultiCameraGeometryResult>> group_results_by_target() const`
### `const std::vector<MultiCameraGeometryResult>& get_results() const`
### `std::vector<MultiCameraDebugRecord> get_debug_history_reverse()`

---

# 10.14 C++ — `MultiCameraReportWriter`

Tạo class:

```cpp
class MultiCameraReportWriter
```

## Hàm cần có

### `write_camera_graph_report(...)`
Ví dụ:

```text
[Camera Graph]
left_camera
right_camera
head_camera
```

### `write_undistortion_report(...)`

### `write_multicamera_backprojection_report(...)`
Ví dụ:

```text
[BackProjection]
Sample: obs_07
Camera: head_camera
Target: marker_01
World Point: (0.42, 0.10, 1.95)
```

### `write_cross_camera_comparison_report(...)`
Ví dụ:

```text
[Cross Camera Comparison]
Target: marker_01

left_camera  -> (0.40, 0.11, 1.96)
right_camera -> (0.41, 0.10, 1.95)
head_camera  -> (0.39, 0.12, 1.97)
```

### `write_reverse_debug_history(...)`

---

# 10.15 C++ — `main.cpp`

## Yêu cầu

```text
Load camera tables:
    intrinsics
    distortion
    extrinsic

Create calibration graph
Load planes
Load observations

Create:
    RadialUndistortionEngine
    DepthBackProjectionSolver
    PlaneIntersectionBackProjectionSolver

Create MultiCameraGeometryRunner
Run
Write reports
```

---

# 11. Luật hình học của project

## Luật 1 — Mỗi observation phải có `camera_id` hợp lệ
Nếu camera không tồn tại trong calibration graph, observation phải fail.

## Luật 2 — Mỗi camera phải dùng đúng intrinsic + distortion + extrinsic của nó
Không được dùng chung bừa một calibration model cho mọi camera.

## Luật 3 — Nếu output world frame được bật
Phải transform point từ camera frame sang world/robot frame.

## Luật 4 — Nếu nhiều camera cùng nhìn một `target_id`
Phải có ít nhất một report so sánh kết quả giữa các camera.

## Luật 5 — Nếu `PLANE_MODE`
Observation phải có `plane_id` hợp lệ.

---

# 12. Output mong muốn

## Config
```text
config/camera_intrinsics_table.txt
config/camera_distortion_table.txt
config/camera_extrinsic_table.txt
config/geometry_plane_config.txt
config/multicamera_observation_config.txt
config/geometry_runtime_config.txt
```

## Reports
```text
assets/outputs/camera_graph_report.txt
assets/outputs/undistortion_report.txt
assets/outputs/multicamera_backprojection_report.txt
assets/outputs/cross_camera_comparison_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build multi-camera configs
- preview camera relation graph
- dùng NumPy để kiểm tra intrinsic / extrinsic / transform

## C++
- load và quản lý nhiều camera models
- chạy back-projection cho từng camera
- đưa kết quả về world frame
- so sánh nhiều góc nhìn

## Computer Vision / Robot Perception
Đây là một project rất sát thực tế vì humanoid robot thường có nhiều camera:
- stereo head cameras
- wrist camera ở tay
- torso camera

Perception engineer phải biết:
- camera nào đang sinh observation
- point 3D đang ở frame nào
- làm sao so sánh / hợp nhất observation từ nhiều camera

Bài 42 chính là bước chuyển từ **single-camera geometry** sang **multi-camera perception geometry**.

---

# 14. Checklist hoàn thành

- [ ] Python build đủ bảng intrinsics / distortion / extrinsic / observation
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho sample ids
- [ ] Python có NumPy preview cho extrinsic transform
- [ ] C++ có `MultiCameraCalibrationGraph`
- [ ] C++ load được nhiều camera models
- [ ] C++ load được extrinsic table
- [ ] C++ xử lý được observation theo từng camera
- [ ] C++ reconstruct được point ở camera frame
- [ ] C++ transform được sang world frame
- [ ] C++ có cross-camera comparison report
- [ ] C++ ghi đủ 5 report

---

# 15. Gợi ý mở rộng

## 1. Thêm stereo pair mode
Nếu `left_camera` và `right_camera` cùng nhìn một target, cho phép dùng disparity để reconstruct depth thay vì depth giả lập / plane mode.

## 2. Thêm camera-to-camera transform query
Cho phép hỏi:
- từ `left_camera` sang `head_camera` đi qua chain nào?

## 3. Thêm reprojection consistency check
Reproject world point từ camera A sang camera B để kiểm tra consistency.

## 4. Chuẩn bị cho Bài 43
Sau Bài 42, hướng đi rất đẹp cho Đợt 9 là:

# **Bài 43: Stereo Rectification & Epipolar Geometry Playground**

Ý tưởng:
```text
left/right camera calibration
→ undistortion
→ rectification
→ epipolar lines
→ correspondence-ready stereo geometry
```

Lúc đó chuỗi Đợt 9 sẽ rất đẹp:
- **Bài 41**: single-camera calibration + back-projection
- **Bài 42**: multi-camera calibration graph
- **Bài 43**: stereo rectification + epipolar geometry
