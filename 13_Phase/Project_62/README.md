# 🤖 Bài 62: Camera-to-Robot Point Projection Analyzer — Bộ phân tích chiếu điểm từ camera sang robot frame cho Humanoid Robot AI Perception

> Mini Project số 62 trong **Đợt 13**  
> **Bài 62 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13** và đi tiếp trực tiếp từ **Bài 61**.
>
> Nếu:
>
> - **Bài 61** giúp bạn xây **Robot Perception Pipeline Orchestrator**
> - **Đợt 13** nhấn mạnh mạnh vào:
>   - `Robot Perception Pipeline`
>   - `Camera to Robot Coordinate`
>
> thì **Bài 62** là bước tiếp theo rất tự nhiên:
>
> ```text
> pixel + depth
> → camera coordinate
> → camera-frame 3D point
> → camera-to-robot transform
> → robot-frame 3D point
> → projection consistency report
> ```
>
> Đây là bước mà bạn đi sâu vào **một stage cực quan trọng của perception pipeline**:
>
> - từ pixel / depth / intrinsics
> - khôi phục 3D point trong camera frame
> - transform point đó sang robot frame
> - kiểm tra tính nhất quán của phép chiếu / phép biến đổi
>
> Tức là dữ liệu bắt đầu usable cho:
> - camera-to-robot projection analysis
> - target point localization
> - frame consistency checking
> - perception-to-robot geometry validation

---

# 📌 Mục lục

- [1. Đợt 13 đang đi tiếp từ Bài 61 ra sao](#1-đợt-13-đang-đi-tiếp-từ-bài-61-ra-sao)
- [2. Vì sao Bài 62 xuất hiện sau Bài 61](#2-vì-sao-bài-62-xuất-hiện-sau-bài-61)
- [3. Bài 62 nâng từ Bài 61 lên chỗ nào](#3-bài-62-nâng-từ-bài-61-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật camera-to-robot projection của project](#11-luật-camera-to-robot-projection-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 13 đang đi tiếp từ Bài 61 ra sao

Ở **Bài 61**, bạn đã có:

```text
depth / point cloud / transform / obstacle / corridor / direction / navigation stages
→ stage dependency graph
→ pipeline execution orchestration
```

Tức là bạn đã có **khung pipeline perception tổng thể**.

Nhưng một pipeline perception tốt vẫn cần từng stage đủ chắc.  
Trong Đợt 13, stage quan trọng nhất được nhấn mạnh là:

```text
camera frame → robot frame
```

vì robot không thể reasoning đúng nếu không đưa được điểm / object / cloud từ camera coordinate sang robot coordinate.

Đó là lý do **Bài 62** xuất hiện.

---

# 2. Vì sao Bài 62 xuất hiện sau Bài 61

## Bài 61 đang làm gì?
Bài 61 giúp bạn xây:
- stage graph
- pipeline execution order
- orchestrator cho perception modules

## Nhưng còn thiếu gì?
Bạn vẫn cần một project đào sâu vào **camera-to-robot geometry stage**:
- từ pixel + depth suy ra camera 3D point
- transform sang robot frame
- kiểm tra kết quả transform có hợp lý không

Nếu không có bước này, pipeline chỉ mới “ghép stage” chứ chưa thật sự nắm chắc **hình học chuyển hệ tọa độ**.

## Bài 62 lấp đúng chỗ đó
Bài 62 sẽ làm:

```text
pixel / depth / intrinsics
→ back-project camera point
→ apply camera-to-robot transform
→ analyze robot-frame point
→ write projection report
```

Đây là bước chuyển từ **pipeline orchestration** sang **camera-to-robot projection analysis**.

---

# 3. Bài 62 nâng từ Bài 61 lên chỗ nào

## Bài 61
- tổ chức nhiều stage perception thành một pipeline
- chạy stage theo dependency order

## Bài 62
- đi sâu vào **camera-to-robot transform stage**
- làm rõ:
  - pixel → camera point
  - camera point → robot point
  - consistency của projection
- sinh **projection analysis report**

### Nói ngắn gọn:
- **Bài 61** hỏi: “Perception pipeline được ghép và chạy như thế nào?”
- **Bài 62** hỏi: “Một điểm ảnh + depth được chiếu sang robot frame như thế nào?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Camera-to-Robot Point Projection Analyzer**

System này nhận đầu vào là:
- một hoặc nhiều **projection samples**
- một bộ **camera intrinsic config**
- một bộ **camera-to-robot extrinsic transform config**
- một bộ **projection checking thresholds**
- một bộ **reporting options**

## Mỗi projection sample tối thiểu chứa
- `pair_id`
- `pixel_u`
- `pixel_v`
- `depth_value`
- optional:
  - `label`
  - `expected_robot_zone`

## Nhiệm vụ của system
### Với mỗi sample:
1. load pixel + depth sample
2. dùng intrinsics để back-project ra **camera-frame 3D point**
3. dùng extrinsic transform để chuyển sang **robot-frame 3D point**
4. tính:
   - camera distance
   - robot distance
   - forward / lateral / vertical components
5. kiểm tra consistency
6. ghi **projection analysis report**

<p align="center">
  <img src="images/project_62.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build projection sample config
→ build intrinsic / extrinsic config
→ preview back-projection bằng NumPy
→ visualize camera-point và robot-point summary

C++
→ load projection samples
→ compute camera-frame 3D point
→ transform to robot-frame point
→ compute projection statistics
→ export projection consistency reports
```

Mục tiêu cốt lõi:
- hiểu cách đi từ **pixel + depth** sang **camera-frame 3D point**
- hiểu cách đi từ **camera-frame point** sang **robot-frame point**
- biết cách kiểm tra projection có hợp lý không
- chuẩn bị nền trực tiếp cho:
  - robot-frame target localization
  - point-based object localization
  - robot-centric geometric reasoning

---

# 6. Pipeline tổng thể

```text
Load Projection Sample Config
Load Camera Intrinsic Config
Load Camera-to-Robot Extrinsic Config
Load Projection Threshold Config
Load Visualization Config
Load Runtime Config

Create BackProjectionEngine
Create CameraToRobotTransformEngine
Create ProjectionConsistencyEngine
Create ProjectionSummaryEngine
Create CameraToRobotPointProjectionAnalyzer

For each projection sample:
    1. load pixel / depth
    2. back-project to camera point
    3. transform to robot point
    4. compute projection metrics
    5. check consistency
    6. write report

After all samples:
    write reports
    write plots
```

---

# 7. Kiến thức cần

## Python
- NumPy
- OOP
- dict / list / module
- Graph / BST
- file handling
- Matplotlib nếu muốn visualize

## C++
- OOP
- inheritance / abstract class
- vector
- enum
- smart pointer

## Computer Vision / Robotics Vision
- pixel coordinate
- depth back-projection
- camera intrinsics
- camera-to-robot extrinsic transform
- robot-frame localization

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **projection analysis pipeline**:

```text
PixelDepthInput
→ BackProjection
→ CameraPoint
→ CameraToRobotTransform
→ RobotPoint
→ ProjectionConsistencyCheck
→ Report
```

## 2. Python BST
Lưu:
- `pair_id`
- `projection_result_id`

## 3. `std::vector<ProjectionSample>`
Danh sách input samples.

## 4. `std::vector<ProjectionAnalysisResult>`
Danh sách kết quả projection.

## 5. `std::stack<ProjectionDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Back-projection
Từ:
- `u, v, depth`
- `fx, fy, cx, cy`

tính camera-frame point:

```text
Xc = (u - cx) * Z / fx
Yc = (v - cy) * Z / fy
Zc = depth
```

Bạn phải ghi rõ convention của trục camera.

---

## Algorithm 2 — Camera-to-robot transform
Từ camera-frame point:

```text
Pc = [Xc, Yc, Zc, 1]^T
Pr = T_robot_camera * Pc
```

hoặc dùng:
- rotation + translation riêng
- homogeneous transform

Nhưng phải ghi rõ convention.

---

## Algorithm 3 — Projection statistics
Tính ít nhất:
- `camera_distance`
- `robot_distance`
- `robot_forward`
- `robot_lateral`
- `robot_vertical`

---

## Algorithm 4 — Consistency checking
Ví dụ:
- depth phải > 0
- robot forward distance phải nằm trong ngưỡng hợp lý
- điểm không được NaN / inf
- optional: zone check với `expected_robot_zone`

---

## Algorithm 5 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- histogram / bar chart cho robot forward distances
- camera vs robot distance summary
- zone distribution chart

---

# 9. Cấu trúc folder

```text
mini_project_62_camera_to_robot_point_projection_analyzer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ projection_inputs/
│  │  ├─ projection_sample_01.txt
│  │  ├─ projection_sample_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ projection_result_report.txt
│     ├─ projection_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ robot_forward_plot.png
│     └─ distance_summary_plot.png
│
├─ config/
│  ├─ projection_sample_config.txt
│  ├─ camera_intrinsic_config.txt
│  ├─ camera_to_robot_extrinsic_config.txt
│  ├─ projection_threshold_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ projection_graph_preview.py
│     ├─ projection_bst.py
│     ├─ numpy_projection_preview.py
│     ├─ matplotlib_projection_plotter.py
│     └─ synthetic_projection_input_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ ProjectionSample.hpp
   │  ├─ CameraIntrinsicConfig.hpp
   │  ├─ CameraToRobotExtrinsicConfig.hpp
   │  ├─ ProjectionThresholdConfig.hpp
   │  ├─ Point3D.hpp
   │  ├─ ProjectionAnalysisResult.hpp
   │  ├─ ProjectionDebugRecord.hpp
   │  ├─ BaseBackProjectionEngine.hpp
   │  ├─ BackProjectionEngine.hpp
   │  ├─ BaseCameraToRobotTransformEngine.hpp
   │  ├─ CameraToRobotTransformEngine.hpp
   │  ├─ BaseProjectionConsistencyEngine.hpp
   │  ├─ ProjectionConsistencyEngine.hpp
   │  ├─ BaseProjectionSummaryEngine.hpp
   │  ├─ ProjectionSummaryEngine.hpp
   │  ├─ CameraToRobotPointProjectionAnalyzer.hpp
   │  └─ ProjectionReportWriter.hpp
   │
   └─ src/
      ├─ BackProjectionEngine.cpp
      ├─ CameraToRobotTransformEngine.cpp
      ├─ ProjectionConsistencyEngine.cpp
      ├─ ProjectionSummaryEngine.cpp
      ├─ CameraToRobotPointProjectionAnalyzer.cpp
      └─ ProjectionReportWriter.cpp
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
projection_sample_config_path
camera_intrinsic_config_path
camera_to_robot_extrinsic_config_path
projection_threshold_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `CameraToRobotProjectionConfigBuilder`

Tạo class con:

```python
class CameraToRobotProjectionConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_projection_sample(
    pair_id,
    pixel_u,
    pixel_v,
    depth_value,
    label=None,
    expected_robot_zone=None
)`

### `set_camera_intrinsics(
    fx, fy, cx, cy
)`

### `set_camera_to_robot_extrinsic(
    tx, ty, tz,
    roll_deg, pitch_deg, yaw_deg
)`

### `set_projection_thresholds(
    min_depth,
    max_forward_distance,
    lateral_limit
)`

### `set_visualization_options(
    enable_forward_plot,
    enable_distance_plot
)`

### `write_projection_sample_config()`
### `write_camera_intrinsic_config()`
### `write_camera_to_robot_extrinsic_config()`
### `write_projection_threshold_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `projection_graph_preview.py`

Tạo class:

```python
class ProjectionGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
PixelDepthInput
→ BackProjection
→ CameraPoint
→ CameraToRobotTransform
→ RobotPoint
→ ProjectionConsistencyCheck
→ Report
```

---

# 10.4 Python — `projection_bst.py`

Tạo BST cho:
- `pair_id`
- `projection_result_id`

---

# 10.5 Python — `numpy_projection_preview.py`

Tạo class:

```python
class NumPyProjectionPreview:
```

## Hàm cần có

### `load_projection_samples(path)`
### `back_project(u, v, depth, intrinsic_config)`
### `transform_camera_to_robot(camera_point, extrinsic_config)`
### `build_projection_summary(results)`

---

# 10.6 Python — `matplotlib_projection_plotter.py`

Tạo class:

```python
class MatplotlibProjectionPlotter:
```

## Hàm cần có

### `plot_robot_forward_distances(results, save_path)`
### `plot_camera_robot_distance_summary(results, save_path)`

---

# 10.7 C++ — `ProjectionSample`

```cpp
struct ProjectionSample
{
    std::string pair_id;

    double pixel_u;
    double pixel_v;
    double depth_value;

    std::string label;
    std::string expected_robot_zone;
};
```

---

# 10.8 C++ — `CameraIntrinsicConfig`

```cpp
struct CameraIntrinsicConfig
{
    double fx;
    double fy;
    double cx;
    double cy;
};
```

---

# 10.9 C++ — `CameraToRobotExtrinsicConfig`

```cpp
struct CameraToRobotExtrinsicConfig
{
    double tx;
    double ty;
    double tz;

    double roll_deg;
    double pitch_deg;
    double yaw_deg;
};
```

---

# 10.10 C++ — `ProjectionThresholdConfig`

```cpp
struct ProjectionThresholdConfig
{
    double min_depth;
    double max_forward_distance;
    double lateral_limit;
};
```

---

# 10.11 C++ — `Point3D`

```cpp
struct Point3D
{
    double x;
    double y;
    double z;
};
```

---

# 10.12 C++ — `ProjectionAnalysisResult`

```cpp
struct ProjectionAnalysisResult
{
    std::string pair_id;

    Point3D camera_point;
    Point3D robot_point;

    double camera_distance;
    double robot_distance;

    double robot_forward;
    double robot_lateral;
    double robot_vertical;

    bool is_consistent;
    std::string summary;
};
```

---

# 10.13 C++ — `ProjectionDebugRecord`

```cpp
struct ProjectionDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.14 C++ — `BaseBackProjectionEngine`

Tạo abstract class:

```cpp
class BaseBackProjectionEngine
{
public:
    virtual Point3D back_project(
        const ProjectionSample& sample,
        const CameraIntrinsicConfig& intrinsic
    ) const = 0;

    virtual ~BaseBackProjectionEngine() = default;
};
```

---

# 10.15 C++ — `BackProjectionEngine`

Kế thừa `BaseBackProjectionEngine`.

## Nhiệm vụ
- từ `u, v, depth` tính camera-frame point

---

# 10.16 C++ — `BaseCameraToRobotTransformEngine`

Tạo abstract class:

```cpp
class BaseCameraToRobotTransformEngine
{
public:
    virtual Point3D transform(
        const Point3D& camera_point,
        const CameraToRobotExtrinsicConfig& extrinsic
    ) const = 0;

    virtual ~BaseCameraToRobotTransformEngine() = default;
};
```

---

# 10.17 C++ — `CameraToRobotTransformEngine`

Kế thừa `BaseCameraToRobotTransformEngine`.

## Nhiệm vụ
- transform camera point → robot point

---

# 10.18 C++ — `BaseProjectionConsistencyEngine`

Tạo abstract class:

```cpp
class BaseProjectionConsistencyEngine
{
public:
    virtual bool check(
        const ProjectionAnalysisResult& result,
        const ProjectionThresholdConfig& thresholds
    ) const = 0;

    virtual ~BaseProjectionConsistencyEngine() = default;
};
```

---

# 10.19 C++ — `ProjectionConsistencyEngine`

Kế thừa `BaseProjectionConsistencyEngine`.

## Nhiệm vụ
- kiểm tra consistency của kết quả projection

---

# 10.20 C++ — `BaseProjectionSummaryEngine`

Tạo abstract class:

```cpp
class BaseProjectionSummaryEngine
{
public:
    virtual std::string summarize(
        const ProjectionAnalysisResult& result
    ) const = 0;

    virtual ~BaseProjectionSummaryEngine() = default;
};
```

---

# 10.21 C++ — `ProjectionSummaryEngine`

Kế thừa `BaseProjectionSummaryEngine`.

## Nhiệm vụ
- tạo summary kiểu:
```text
The pixel-depth sample was back-projected successfully.
Its robot-frame point lies 1.84m ahead and 0.22m to the left of the robot base.
```

---

# 10.22 C++ — `CameraToRobotPointProjectionAnalyzer`

Tạo class trung tâm:

```cpp
class CameraToRobotPointProjectionAnalyzer
```

## Thuộc tính

```cpp
private:
    std::vector<ProjectionSample> samples;

    CameraIntrinsicConfig intrinsic_config;
    CameraToRobotExtrinsicConfig extrinsic_config;
    ProjectionThresholdConfig threshold_config;

    std::shared_ptr<BaseBackProjectionEngine> back_projection_engine;
    std::shared_ptr<BaseCameraToRobotTransformEngine> transform_engine;
    std::shared_ptr<BaseProjectionConsistencyEngine> consistency_engine;
    std::shared_ptr<BaseProjectionSummaryEngine> summary_engine;

    std::vector<ProjectionAnalysisResult> results;
    std::stack<ProjectionDebugRecord> debug_history;
```

## Hàm cần có

### `load_projection_sample_config(const std::string& path)`
### `load_camera_intrinsic_config(const std::string& path)`
### `load_camera_to_robot_extrinsic_config(const std::string& path)`
### `load_projection_threshold_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each sample:
    camera_point = back_projection_engine.back_project(sample, intrinsic_config)
    robot_point = transform_engine.transform(camera_point, extrinsic_config)

    build ProjectionAnalysisResult
    result.is_consistent = consistency_engine.check(result, threshold_config)
    result.summary = summary_engine.summarize(result)

    save result
    push debug history
```

### `const std::vector<ProjectionAnalysisResult>& get_results() const`
### `std::vector<ProjectionDebugRecord> get_debug_history_reverse()`

---

# 10.23 C++ — `ProjectionReportWriter`

Tạo class:

```cpp
class ProjectionReportWriter
```

## Hàm cần có

### `write_projection_result_report(...)`
Ví dụ:

```text
[Projection Result]
Pair: shelf_01
Camera Point: (0.18, 0.04, 1.25)
Robot Point: (1.14, -0.12, 0.96)
Consistent: YES
```

### `write_projection_summary_report(...)`
Ví dụ:

```text
[Projection Summary]
Pair: shelf_01
The sample was projected successfully from camera frame to robot frame.
The robot-frame point lies 1.14m ahead of the base and slightly to the right.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.24 C++ — `main.cpp`

## Yêu cầu

```text
Load projection sample config
Load camera intrinsic config
Load camera-to-robot extrinsic config
Load projection threshold config
Load visualization config
Load runtime config

Create:
    BackProjectionEngine
    CameraToRobotTransformEngine
    ProjectionConsistencyEngine
    ProjectionSummaryEngine

Create CameraToRobotPointProjectionAnalyzer
Run
Write reports
```

---

# 11. Luật camera-to-robot projection của project

## Luật 1 — Phải tách rõ camera point và robot point
Không được gộp mơ hồ.

## Luật 2 — Công thức back-projection phải ghi rõ
Ít nhất phải thể hiện:
- `u, v, depth`
- `fx, fy, cx, cy`
- công thức suy ra `Xc, Yc, Zc`

## Luật 3 — Camera-to-robot transform phải là module riêng
Đây là trọng tâm của Đợt 13.

## Luật 4 — Report phải giải thích vị trí robot-frame của điểm
Không chỉ ghi số thô.

## Luật 5 — Project phải usable cho target localization
Tức là sau project này, bạn phải hình dung được:
- một điểm ảnh trong ảnh nằm ở đâu trong robot frame
- nó nằm trước / trái / phải / cao / thấp bao nhiêu so với robot

---

# 12. Output mong muốn

## Config
```text
config/projection_sample_config.txt
config/camera_intrinsic_config.txt
config/camera_to_robot_extrinsic_config.txt
config/projection_threshold_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/projection_result_report.txt
assets/outputs/projection_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/robot_forward_plot.png
assets/outputs/distance_summary_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build sample / intrinsic / extrinsic / threshold configs
- preview back-projection + transform
- visualize forward / lateral / vertical distributions

## C++
- chạy camera-to-robot projection analysis
- sinh camera-frame point + robot-frame point
- tạo report phục vụ localization reasoning

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Một pixel + depth cụ thể trong camera sẽ nằm ở đâu trong robot frame?**

Pipeline lúc này sẽ là:

```text
pixel + depth
→ camera-frame point
→ camera-to-robot transform
→ robot-frame point
→ projection consistency analysis
→ target localization reasoning
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho samples / intrinsic / extrinsic / threshold / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / projection result ids
- [ ] Python có NumPy back-projection preview
- [ ] C++ có `ProjectionSample`
- [ ] C++ có `CameraIntrinsicConfig`
- [ ] C++ có `CameraToRobotExtrinsicConfig`
- [ ] C++ có `ProjectionThresholdConfig`
- [ ] C++ có `Point3D`
- [ ] C++ có `ProjectionAnalysisResult`
- [ ] C++ có `BackProjectionEngine`
- [ ] C++ có `CameraToRobotTransformEngine`
- [ ] C++ có `ProjectionConsistencyEngine`
- [ ] C++ có `ProjectionSummaryEngine`
- [ ] C++ có `CameraToRobotPointProjectionAnalyzer`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 62**, bước hợp lý nhất để tiếp tục Đợt 13 là:

# **Bài 63: Robot-Frame Target Localization Inspector**

Ý tưởng:
```text
camera-frame point / robot-frame point
→ target position validation
→ left / center / right zone assignment
→ reachable target summary
→ localization inspection report
```

Tức là mạch sẽ đi:

```text
Bài 61: Robot Perception Pipeline Orchestrator
→ Bài 62: Camera-to-Robot Point Projection Analyzer
→ Bài 63: Robot-Frame Target Localization Inspector
```
