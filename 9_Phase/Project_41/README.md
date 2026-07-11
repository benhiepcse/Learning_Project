# 🤖 Bài 41: Camera Calibration & Back-Projection Geometry Simulator — Bộ mô phỏng hiệu chỉnh camera và dựng hình 2D → 3D cho Humanoid Robot AI Perception

> Mini Project số 41 trong **Đợt 9**  
> **Bài 41 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9**.
>
> Sau khi chốt **Đợt 8** với cụm:
>
> - **Bài 36**: Stereo Disparity Track Manager  
> - **Bài 37**: Stereo Depth Command Graph Runner  
> - **Bài 38**: Stereo Landmark Motion Queue Simulator  
> - **Bài 39**: Stereo Multi-Landmark Track Table Analyzer  
> - **Bài 40**: Stereo Track Quality Dashboard Generator
>
> thì **Đợt 9** nên chuyển trọng tâm về **core vision geometry** thay vì tiếp tục dashboard/runtime.  
> Với định hướng **Robot Perception Engineer** và vẫn hữu ích cho **Reinforcement Learning Engineer** trong môi trường robot/simulation, hướng thuận lợi hơn là:
>
> # **Hướng A — Geometry / Calibration / Projection / 3D Reconstruction**
>
> vì nó chạm trực tiếp vào:
> - camera model
> - calibration
> - undistortion
> - back-projection
> - 2D ↔ 3D reasoning
> - perception foundation cho stereo / SLAM / pose / manipulation
>
> Còn **Hướng B** (benchmark / evaluation depth) vẫn tốt, nhưng hợp hơn như **tầng đánh giá sau khi bạn đã nắm chắc geometry**.
>
> Vì vậy, **Bài 41** sẽ mở Đợt 9 bằng một project rất sát lõi perception:
>
> ```text
> distorted pixel observation
> → undistort
> → back-project ray
> → intersect với mặt phẳng / độ sâu giả lập
> → recover 3D point
> → so sánh với ground truth geometry
> ```

---

# 📌 Mục lục

- [1. Vì sao chọn Hướng A cho Đợt 9](#1-vì-sao-chọn-hướng-a-cho-đợt-9)
- [2. Đợt 9 đang học gì](#2-đợt-9-đang-học-gì)
- [3. Bài 41 nâng từ Đợt 8 lên chỗ nào](#3-bài-41-nâng-từ-đợt-8-lên-chỗ-nào)
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

# 1. Vì sao chọn Hướng A cho Đợt 9

Bạn hỏi: **trong 2 hướng A và B, hướng nào thuận lợi hơn cho Robot Perception Engineer và Reinforcement Learning Engineer?**

## Kết luận ngắn
### **Mình chọn Hướng A**
> **Geometry / Calibration / Projection / 3D Reconstruction**  
> là hướng nên đi trước trong Đợt 9.

## Vì sao?
## Với **Robot Perception Engineer**
Đây gần như là lõi nghề:
- pinhole camera model
- intrinsic / extrinsic
- undistortion
- back projection
- stereo geometry
- point reconstruction
- camera-to-robot reasoning

Nếu thiếu phần này thì về sau rất khó học chắc:
- stereo depth
- visual odometry / SLAM
- 3D perception
- calibration giữa camera và robot
- grasping / manipulation perception

## Với **Reinforcement Learning Engineer** trong robot
RL không dùng calibration mỗi ngày như perception engineer, nhưng vẫn hưởng lợi mạnh từ geometry vì:
- cần hiểu observation space đến từ camera như thế nào
- cần dựng simulator / synthetic observations
- cần hiểu depth / point / egocentric view cho policy learning
- cần camera-aware reward / state design trong embodied RL

## Còn Hướng B thì sao?
**Hướng B (benchmark / error / robustness / post-processing)** rất đáng học, nhưng nên đặt **sau khi đã vững geometry**.  
Nói ngắn gọn:

- **Hướng A** = xây **nền móng**
- **Hướng B** = xây **tầng đánh giá / benchmark** trên nền đó

Vì vậy mở Đợt 9 bằng **Bài 41 theo Hướng A** là hợp lý nhất.

---

# 2. Đợt 9 đang học gì

Theo roadmap của bạn, **Đợt 9 (Ngày 17–18)** là:

## Chủ đề
# **Geometry simulator**

## Python
### **Phase 8 — Data Structures**
- `Python Binary Search Tree`
- `Python Graph`
  - graph representation
  - BFS / DFS

### **Phase 9 — Python Libraries**
- `Python NumPy`
  - array
  - shape
  - indexing
  - math operations

## C++
### **Phase 6 — OOP**
- `C++ Virtual Functions`
  - polymorphism
- `C++ Enums`

## Computer Vision
### **Phase 4 — Camera Geometry**
- `Back Projection 2D → 3D`
- `Camera Calibration`
- `Undistortion`

## Ý nghĩa của Đợt 9
Đây là đợt mà bạn bắt đầu bước từ:

```text
camera model / projection / stereo runtime
```

sang:

```text
calibration-aware geometry reasoning
+ pixel → ray → 3D point reconstruction
```

---

# 3. Bài 41 nâng từ Đợt 8 lên chỗ nào

## Đợt 8 đã cho bạn gì?
Từ Bài 36 → 40, bạn đã có:
- temporal track manager
- command graph runner
- landmark motion simulator
- multi-track analyzer
- dashboard evaluation

Tức là bạn đã có **runtime + data structures + multi-stage perception pipeline**.

## Nhưng còn thiếu gì?
Bạn vẫn còn thiếu một project tập trung sâu vào **lõi hình học camera**:
- pixel distortion
- undistortion
- normalized coordinate
- back-projection ray
- khôi phục 3D point từ pixel + depth / plane

## Bài 41 sẽ lấp đúng chỗ đó
Bài 41 không còn thiên về “track table / dashboard” nữa, mà quay về câu hỏi nền tảng:

> **Từ một pixel trên ảnh, sau khi biết calibration, làm sao suy ra tia nhìn trong không gian và dựng lại điểm 3D?**

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Camera Calibration & Back-Projection Geometry Simulator**

System này mô phỏng một camera có:
- **intrinsic matrix**
- **distortion coefficients**
- **extrinsic pose**
- tập các **2D observations** hoặc **distorted pixel measurements**

Mục tiêu là xây pipeline:

## Với mỗi observation:
1. nhận pixel bị méo (`u_distorted`, `v_distorted`)
2. **undistort** pixel về dạng gần lý tưởng
3. chuyển sang **normalized image coordinate**
4. dựng **back-projection ray**
5. giao tia với:
   - một **depth giả lập**
   - hoặc **một mặt phẳng 3D** (ví dụ `Z = constant`)
6. thu được **3D point estimate**
7. so sánh với **ground-truth point**

Tức là bạn sẽ học trọn vòng:

```text
distorted image point
→ undistorted point
→ normalized coordinate
→ camera ray
→ 3D reconstruction
```

<p align="center">
  <img src="../../images/project_41.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu được pipeline:

```text
Python
→ build camera calibration config
→ build distortion config
→ build 2D observation set / plane config
→ preview graph of observation relationships
→ preview NumPy geometry matrices

C++
→ load camera model
→ undistort pixel
→ back-project to camera ray
→ intersect ray with plane / use synthetic depth
→ reconstruct 3D point
→ compare with ground truth
→ export geometry report
```

Mục tiêu không chỉ là “code chạy được”, mà là:
- hiểu **pixel bị méo được sửa như thế nào**
- hiểu **từ pixel ra ray 3D**
- hiểu **back-projection phụ thuộc intrinsics / distortion ra sao**
- chuẩn bị nền cho **stereo depth / calibration / SLAM / manipulation perception**

---

# 6. Pipeline tổng thể

```text
Load Intrinsic Config
Load Distortion Config
Load Extrinsic / Plane Config
Load 2D Observation Config

Create CameraCalibrationModel
Create UndistortionEngine
Create BackProjectionEngine
Create GeometryEvaluationRunner

For each observation:
    Step 1: read distorted pixel (u_d, v_d)

    Step 2: undistort pixel
            -> (u_u, v_u)

    Step 3: convert to normalized coordinate
            x_n = (u_u - cx) / fx
            y_n = (v_u - cy) / fy

    Step 4: build camera ray
            ray = [x_n, y_n, 1]

    Step 5:
        Option A: if synthetic depth Z is known
            recover 3D point directly

        Option B: if plane is known
            intersect ray with plane

    Step 6:
        compare reconstructed point with ground truth
        compute reprojection / position error

Write:
    undistortion report
    back-projection report
    geometry evaluation report
    reverse debug history
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- **BST**
- **Graph + BFS / DFS**
- **NumPy arrays / matrix ops / indexing**
- config builder

## C++
- class / inheritance
- **virtual functions / polymorphism**
- **enum**
- `std::vector`
- `std::unordered_map`
- `std::queue`
- `std::stack`
- geometry runtime design

## Computer Vision / Geometry
- intrinsic matrix
- distortion / undistortion
- normalized image coordinate
- back projection 2D → 3D
- plane intersection / synthetic depth reconstruction
- calibration-aware geometry

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Dùng để biểu diễn **geometry processing graph** hoặc **observation dependency graph**.

Ví dụ:
```text
DistortedPixel
→ Undistort
→ Normalize
→ BackProject
→ IntersectPlane
→ Evaluate
```

Bạn phải có:
- graph representation
- BFS / DFS traversal preview

---

## 2. Python Binary Search Tree
Dùng để lưu các **observation frame id** hoặc **geometry sample id** theo thứ tự.

Ví dụ:
- insert `sample_001`, `sample_005`, `sample_003`
- duyệt inorder để ra thứ tự tăng dần

Mục đích là ép bạn dùng đúng kiến thức BST của Đợt 9 trong project.

---

## 3. `std::vector<GeometryObservation>`
Lưu danh sách các observation cần xử lý.

## 4. `std::vector<GeometryEvaluationResult>`
Lưu kết quả dựng 3D và sai số của từng observation.

## 5. `std::stack<GeometryDebugRecord>`
Reverse debug history.

## 6. `std::unordered_map<std::string, PlaneModel>`
Map tên mặt phẳng → mô hình mặt phẳng.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Undistortion
Bạn có thể làm **mô hình đơn giản hóa** cho radial distortion.

Ví dụ:
- observation đầu vào có `u_distorted`, `v_distorted`
- sau đó dùng một hàm `undistort_pixel(...)`

Bạn **không bắt buộc** phải implement full OpenCV calibration solver.  
Ở Bài 41, trọng tâm là **hiểu pipeline**, nên có thể:
- dùng distortion coefficients đã cho sẵn
- tự viết hàm undistort đơn giản
- hoặc gọi OpenCV `undistortPoints` nếu muốn

---

## Algorithm 2 — Normalize pixel
Sau khi undistort:

```text
x_n = (u - cx) / fx
y_n = (v - cy) / fy
```

---

## Algorithm 3 — Back-projection ray
Từ normalized coordinate:

```text
ray_camera = [x_n, y_n, 1]
```

Có thể chuẩn hóa ray nếu muốn.

---

## Algorithm 4 — 3D reconstruction from known depth
Nếu biết depth `Z`:

```text
X = x_n * Z
Y = y_n * Z
Z = Z
```

---

## Algorithm 5 — Ray-plane intersection
Nếu có mặt phẳng `ax + by + cz + d = 0`, hãy tìm giao điểm giữa ray và plane.

### Gợi ý
Ray:
```text
P(t) = t * [x_n, y_n, 1]
```

Thế vào phương trình mặt phẳng để tìm `t`.

---

## Algorithm 6 — Error evaluation
So sánh:
- `reconstructed_point`
- `ground_truth_point`

Tính tối thiểu:
- sai số Euclidean 3D
- sai số theo trục Z
- optional: reprojection error

---

# 9. Cấu trúc folder

```text
mini_project_41_camera_calibration_backprojection_geometry_simulator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ undistortion_report.txt
│     ├─ backprojection_report.txt
│     ├─ geometry_evaluation_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ camera_intrinsics_config.txt
│  ├─ camera_distortion_config.txt
│  ├─ geometry_plane_config.txt
│  ├─ geometry_observation_config.txt
│  └─ geometry_runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ geometry_graph_preview.py
│     ├─ observation_bst.py
│     ├─ numpy_geometry_preview.py
│     └─ sample_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ CameraIntrinsics.hpp
   │  ├─ DistortionCoefficients.hpp
   │  ├─ PlaneModel.hpp
   │  ├─ GeometryObservation.hpp
   │  ├─ GeometryEvaluationResult.hpp
   │  ├─ GeometryDebugRecord.hpp
   │  ├─ CameraCalibrationModel.hpp
   │  ├─ BaseUndistortionEngine.hpp
   │  ├─ RadialUndistortionEngine.hpp
   │  ├─ BaseBackProjectionSolver.hpp
   │  ├─ DepthBackProjectionSolver.hpp
   │  ├─ PlaneIntersectionBackProjectionSolver.hpp
   │  ├─ GeometryEvaluationRunner.hpp
   │  └─ GeometryReportWriter.hpp
   │
   └─ src/
      ├─ RadialUndistortionEngine.cpp
      ├─ DepthBackProjectionSolver.cpp
      ├─ PlaneIntersectionBackProjectionSolver.cpp
      ├─ GeometryEvaluationRunner.cpp
      └─ GeometryReportWriter.cpp
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
intrinsics_path
distortion_path
plane_path
observation_path
runtime_path
```

## Hàm cần có
### `show_project_info()`
- in tên project và các config path

### `@classmethod create_default_paths(cls, root_dir)`
- trả về dict path mặc định

### `@staticmethod validate_positive_number(value, field_name)`
- validate `fx`, `fy`, depth, threshold

---

# 10.2 Python — `CameraGeometryConfigBuilder`

Tạo class con:

```python
class CameraGeometryConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `set_intrinsics(fx, fy, cx, cy, image_width, image_height)`

### `set_distortion(k1, k2, p1, p2, k3=0.0)`
- cho phép radial + tangential cơ bản, dù project có thể chỉ dùng radial trước

### `add_plane(plane_id, a, b, c, d)`
Ví dụ:
```text
ground_plane: z = 2.0  ->  0x + 0y + 1z - 2 = 0
```

### `add_observation(
    sample_id,
    u_distorted,
    v_distorted,
    mode,
    known_depth=None,
    plane_id=None,
    gt_x=None, gt_y=None, gt_z=None
)`
- `mode` có thể là:
  - `DEPTH_MODE`
  - `PLANE_MODE`

### `set_runtime_options(
    normalize_ray,
    enable_error_report,
    use_plane_intersection_first
)`

### `write_intrinsics_config()`
### `write_distortion_config()`
### `write_plane_config()`
### `write_observation_config()`
### `write_runtime_config()`

---

# 10.3 Python — `geometry_graph_preview.py`

Tạo class:

```python
class GeometryProcessingGraph:
```

## Mục tiêu
Biểu diễn graph pipeline:

```text
DistortedPixel -> Undistort -> Normalize -> BackProject -> Evaluate
```

## Hàm cần có

### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

---

# 10.4 Python — `observation_bst.py`

Tạo:

```python
class ObservationBSTNode:
    ...
```

và

```python
class ObservationBST:
```

## Mục tiêu
Lưu `sample_id` hoặc `frame_index` theo BST.

## Hàm cần có

### `insert(key, payload)`
### `search(key)`
### `inorder()`
### `preorder()`

---

# 10.5 Python — `numpy_geometry_preview.py`

Tạo class:

```python
class NumPyGeometryPreview:
```

## Hàm cần có

### `build_intrinsic_matrix(fx, fy, cx, cy)`
- trả `numpy.ndarray`

### `normalize_pixel(u, v, fx, fy, cx, cy)`
- trả `[x_n, y_n]`

### `backproject_with_depth(u, v, depth, fx, fy, cx, cy)`
- trả `[X, Y, Z]`

### `ray_plane_intersection(ray, plane)`
- minh họa bằng NumPy

---

# 10.6 C++ — `CameraIntrinsics`

```cpp
struct CameraIntrinsics
{
    double fx;
    double fy;
    double cx;
    double cy;

    int image_width;
    int image_height;
};
```

---

# 10.7 C++ — `DistortionCoefficients`

```cpp
struct DistortionCoefficients
{
    double k1;
    double k2;
    double p1;
    double p2;
    double k3;
};
```

---

# 10.8 C++ — `PlaneModel`

```cpp
struct PlaneModel
{
    std::string plane_id;
    double a;
    double b;
    double c;
    double d;
};
```

---

# 10.9 C++ — `GeometryObservation`

```cpp
enum class BackProjectionMode
{
    DEPTH_MODE,
    PLANE_MODE
};

struct GeometryObservation
{
    std::string sample_id;

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

# 10.10 C++ — `GeometryEvaluationResult`

```cpp
struct GeometryEvaluationResult
{
    std::string sample_id;

    double u_undistorted;
    double v_undistorted;

    double x_normalized;
    double y_normalized;

    double reconstructed_x;
    double reconstructed_y;
    double reconstructed_z;

    bool success;

    double error_3d;
    double error_z;
};
```

---

# 10.11 C++ — `GeometryDebugRecord`

```cpp
struct GeometryDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.12 C++ — `CameraCalibrationModel`

Tạo class:

```cpp
class CameraCalibrationModel
{
private:
    CameraIntrinsics intrinsics;
    DistortionCoefficients distortion;

public:
    CameraCalibrationModel(
        const CameraIntrinsics& intrinsics,
        const DistortionCoefficients& distortion
    );

    const CameraIntrinsics& get_intrinsics() const;
    const DistortionCoefficients& get_distortion() const;
};
```

---

# 10.13 C++ — `BaseUndistortionEngine`

Tạo abstract class:

```cpp
class BaseUndistortionEngine
{
public:
    virtual std::pair<double, double> undistort_pixel(
        double u_distorted,
        double v_distorted,
        const CameraCalibrationModel& camera_model
    ) const = 0;

    virtual ~BaseUndistortionEngine() = default;
};
```

---

# 10.14 C++ — `RadialUndistortionEngine`

Kế thừa:

```cpp
class RadialUndistortionEngine : public BaseUndistortionEngine
```

## Nhiệm vụ
- nhận pixel distorted
- dùng distortion coefficients để ước lượng pixel undistorted
- có thể dùng mô hình radial đơn giản hóa

### Hàm override

```cpp
std::pair<double, double> undistort_pixel(
    double u_distorted,
    double v_distorted,
    const CameraCalibrationModel& camera_model
) const override;
```

---

# 10.15 C++ — `BaseBackProjectionSolver`

Tạo abstract class:

```cpp
class BaseBackProjectionSolver
{
public:
    virtual bool solve(
        const GeometryObservation& observation,
        const CameraCalibrationModel& camera_model,
        double u_undistorted,
        double v_undistorted,
        const std::unordered_map<std::string, PlaneModel>& plane_table,
        GeometryEvaluationResult& output
    ) const = 0;

    virtual ~BaseBackProjectionSolver() = default;
};
```

---

# 10.16 C++ — `DepthBackProjectionSolver`

Kế thừa:

```cpp
class DepthBackProjectionSolver : public BaseBackProjectionSolver
```

## Nhiệm vụ
Nếu observation ở `DEPTH_MODE`:
1. normalize pixel
2. dùng `known_depth`
3. reconstruct `(X, Y, Z)`

---

# 10.17 C++ — `PlaneIntersectionBackProjectionSolver`

Kế thừa:

```cpp
class PlaneIntersectionBackProjectionSolver : public BaseBackProjectionSolver
```

## Nhiệm vụ
Nếu observation ở `PLANE_MODE`:
1. normalize pixel
2. tạo ray `[x_n, y_n, 1]`
3. tìm giao với `PlaneModel`

---

# 10.18 C++ — `GeometryEvaluationRunner`

Tạo class trung tâm:

```cpp
class GeometryEvaluationRunner
```

## Thuộc tính

```cpp
private:
    CameraCalibrationModel camera_model;

    std::vector<GeometryObservation> observations;
    std::unordered_map<std::string, PlaneModel> plane_table;

    std::shared_ptr<BaseUndistortionEngine> undistortion_engine;
    std::shared_ptr<BaseBackProjectionSolver> depth_solver;
    std::shared_ptr<BaseBackProjectionSolver> plane_solver;

    std::vector<GeometryEvaluationResult> results;
    std::stack<GeometryDebugRecord> debug_history;
```

## Hàm cần có

### `load_plane_config(const std::string& path)`
### `load_observation_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each observation:
    1. undistort pixel
    2. if DEPTH_MODE:
           use depth_solver
       else if PLANE_MODE:
           use plane_solver
    3. if ground truth available:
           compute 3D error
    4. push result
    5. push debug record
```

### `const std::vector<GeometryEvaluationResult>& get_results() const`
### `std::vector<GeometryDebugRecord> get_debug_history_reverse()`

---

# 10.19 C++ — `GeometryReportWriter`

Tạo class:

```cpp
class GeometryReportWriter
```

## Hàm cần có

### `write_undistortion_report(...)`
Ví dụ:

```text
[Undistortion]
Sample: obs_01
Distorted: (622.4, 241.7)
Undistorted: (615.2, 240.9)
```

### `write_backprojection_report(...)`
Ví dụ:

```text
[BackProjection]
Sample: obs_01
Normalized: (0.153, -0.012)
Reconstructed 3D: (0.31, -0.02, 2.00)
Mode: DEPTH_MODE
```

### `write_geometry_evaluation_report(...)`
Ví dụ:

```text
[Geometry Evaluation]
Sample: obs_01
GT: (0.30, -0.02, 2.00)
Pred: (0.31, -0.02, 2.00)
3D Error: 0.01
Z Error: 0.00
```

### `write_reverse_debug_history(...)`

---

# 10.20 C++ — `main.cpp`

## Yêu cầu

```text
Load camera intrinsics + distortion
Create CameraCalibrationModel

Create:
    RadialUndistortionEngine
    DepthBackProjectionSolver
    PlaneIntersectionBackProjectionSolver

Create GeometryEvaluationRunner
Load planes
Load observations
Run
Write reports
```

---

# 11. Luật hình học của project

## Luật 1 — Nếu dùng `DEPTH_MODE`
Observation phải có `known_depth > 0`.

## Luật 2 — Nếu dùng `PLANE_MODE`
Observation phải có `plane_id` hợp lệ tồn tại trong `plane_table`.

## Luật 3 — Pixel phải được undistort trước khi normalize
Không được lấy pixel méo đi back-project trực tiếp nếu bạn đang bật pipeline calibration-aware.

## Luật 4 — Nếu plane song song với ray và không cắt nhau
Observation đó phải được đánh dấu `success = false`.

## Luật 5 — Nếu có ground truth
Phải tính ít nhất:
- `error_3d`
- `error_z`

---

# 12. Output mong muốn

## Config
```text
config/camera_intrinsics_config.txt
config/camera_distortion_config.txt
config/geometry_plane_config.txt
config/geometry_observation_config.txt
config/geometry_runtime_config.txt
```

## Reports
```text
assets/outputs/undistortion_report.txt
assets/outputs/backprojection_report.txt
assets/outputs/geometry_evaluation_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build camera / distortion / plane / observation config
- preview geometry graph
- preview BST ordering của samples
- dùng NumPy để kiểm tra nhanh công thức hình học

## C++
- chạy geometry runtime thật
- undistort pixel
- back-project thành ray / point 3D
- đánh giá sai số dựng hình

## Computer Vision / Robot Perception
Đây là một project rất sát lõi của **Robot Perception Engineer** vì nó dạy bạn:
- pixel trên ảnh không phải là 3D point
- phải đi qua **camera model**
- phải hiểu **intrinsics / distortion**
- phải biết **back-project** để đưa thông tin ảnh về không gian robot

Đây cũng là nền rất quan trọng cho:
- stereo depth
- hand-eye / camera-robot calibration
- 3D object localization
- manipulation perception
- embodied RL dùng camera observations

---

# 14. Checklist hoàn thành

- [ ] Python build đủ 5 config
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho sample ids
- [ ] Python có NumPy geometry preview
- [ ] C++ có `CameraCalibrationModel`
- [ ] C++ có `BaseUndistortionEngine` + `RadialUndistortionEngine`
- [ ] C++ có `BaseBackProjectionSolver`
- [ ] C++ có `DepthBackProjectionSolver`
- [ ] C++ có `PlaneIntersectionBackProjectionSolver`
- [ ] C++ load được plane table + observation list
- [ ] C++ reconstruct được 3D point
- [ ] C++ tính được geometry error
- [ ] C++ ghi đủ 4 report

---

# 15. Gợi ý mở rộng

## 1. Thêm reprojection check
Sau khi reconstruct 3D point, project ngược lại lên ảnh để kiểm tra reprojection error.

## 2. Thêm nhiều loại plane
Ví dụ:
- ground plane
- table plane
- wall plane

## 3. Thêm camera extrinsic
Cho point nằm ở **robot/world frame**, transform về **camera frame** trước khi project / back-project.

## 4. Chuẩn bị cho Bài 42
Sau Bài 41, hướng đi rất đẹp cho **Đợt 9** là:

# **Bài 42: Multi-Camera Calibration Graph Simulator**

Ý tưởng:
- nhiều camera / nhiều frame
- graph quan hệ giữa camera, plane, landmark
- BFS / DFS để truy vết transform / observation path
- dùng calibration + projection + back-projection trên nhiều camera

Hoặc nếu bạn muốn bám chặt stereo hơn:

# **Bài 42: Undistortion & Stereo Rectification Playground**

Ý tưởng:
- ảnh trái / phải bị distortion
- undistort từng bên
- rectification
- disparity-ready geometry pipeline

Cả hai hướng đều hợp Đợt 9, nhưng nếu bám đúng “Geometry simulator” thì **Multi-Camera Calibration Graph Simulator** là nước đi rất đẹp.
