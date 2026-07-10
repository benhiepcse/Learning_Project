# 🤖 Bài 56: Robot-Frame Point Cloud Transform & Depth Estimation Pipeline — Chuyển point cloud từ camera frame sang robot frame cho Humanoid Robot AI Perception

> Mini Project số 56 trong **Đợt 12**  
> **Bài 56 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12** và đi tiếp trực tiếp từ **Bài 55**.
>
> **Lưu ý nhỏ:** theo chuỗi bạn đang đi từ **Bài 51 → 55**, bài của **Đợt 12** hợp logic sẽ là **Bài 56** chứ không phải Bài 46 nữa. Mình sẽ tiếp tục theo mạch đó.
>
> Nếu:
>
> - **Bài 51** tạo và so sánh **disparity maps**
> - **Bài 52** chuyển **disparity → depth**
> - **Bài 53** đánh giá **depth quality**
> - **Bài 54** dựng **point cloud**
> - **Bài 55** phân tích **point cloud trong camera frame**
>
> thì **Bài 56** là bước rất quan trọng của **Đợt 12**:
>
> ```text
> depth / point cloud / camera geometry
> → camera-to-robot extrinsic transform
> → robot-frame point cloud
> → robot-centric distance analysis
> → forward obstacle summary
> → transform report
> ```
>
> Đây là bước perception chuyển từ:
>
> - **góc nhìn của camera**
>
> sang
>
> - **góc nhìn của robot**
>
> và đó là nơi dữ liệu bắt đầu usable cho:
> - obstacle reasoning
> - locomotion safety
> - navigation / local planning
> - manipulation workspace awareness

---

# 📌 Mục lục

- [1. Đợt 12 đang thêm gì vào pipeline](#1-đợt-12-đang-thêm-gì-vào-pipeline)
- [2. Vì sao Bài 56 xuất hiện sau Bài 55](#2-vì-sao-bài-56-xuất-hiện-sau-bài-55)
- [3. Bài 56 nâng từ Bài 55 lên chỗ nào](#3-bài-56-nâng-từ-bài-55-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật robot-frame transform của project](#11-luật-robot-frame-transform-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 12 đang thêm gì vào pipeline

Mình đã đọc **Đợt 12 (Ngày 23–24)** trong roadmap. Trọng tâm của đợt này là:

## Python
- ôn + áp dụng:
  - **NumPy**
  - **Matplotlib**
  - **OOP**
  - **file handling**

## C++
- ôn + áp dụng:
  - **class**
  - **inheritance**
  - **polymorphism**
  - **vector**

## Computer Vision — Phase 5: Stereo Vision & Depth
- **Depth Estimation**
- **Depth Map**
- **Point Cloud**

Điểm quan trọng là: **Đợt 12 không còn chỉ dừng ở disparity**, mà đi hẳn vào:

```text
disparity
→ depth estimation
→ depth map
→ point cloud
```

Và vì ở Đợt 11 bạn đã đi đến:
- disparity
- depth estimation
- depth quality
- point cloud
- camera-frame cloud analysis

nên bước hợp lý nhất để **khóa Đợt 12** là:

> **đưa point cloud từ camera frame sang robot frame**  
> để point cloud bắt đầu usable cho robot-centric reasoning.

---

# 2. Vì sao Bài 56 xuất hiện sau Bài 55

## Bài 55 đang làm gì?
Bài 55 giúp bạn có:

```text
point cloud camera-frame
→ left / center / right analysis
→ near / mid / far analysis
→ obstacle summary theo camera frame
```

Tức là bạn đã hiểu scene theo **camera coordinate system**.

## Nhưng còn thiếu gì?
Robot thật không ra quyết định dựa trên “điểm nằm trước camera” một cách cô lập.  
Robot cần biết:

- điểm này nằm **so với base robot** ở đâu?
- vật cản ở **phía trước robot** hay lệch sang trái robot?
- cloud cách robot bao xa theo trục tiến lên?
- vùng nào là vùng nguy hiểm cho chân / thân robot?

Nói cách khác, perception cần bước:

```text
camera-frame point cloud
→ robot-frame point cloud
```

## Bài 56 lấp đúng chỗ đó
Bài 56 sẽ làm:

```text
camera-frame point cloud
→ apply extrinsic transform
→ robot-frame point cloud
→ robot-centric region analysis
→ forward obstacle report
```

Đây là bước chuyển từ **camera-centric perception** sang **robot-centric perception**.

---

# 3. Bài 56 nâng từ Bài 55 lên chỗ nào

## Bài 55
- phân tích point cloud trong **camera frame**
- density / distance / center-region summary

## Bài 56
- thêm **extrinsic transform**
- chuyển toàn bộ point cloud sang **robot frame**
- so sánh camera-frame vs robot-frame coordinates
- phân tích obstacle theo **robot front / left / right / body-relative distances**
- ghi **robot-frame transform report**

### Nói ngắn gọn:
- **Bài 55** hỏi: “Camera nhìn thấy point cloud như thế nào?”
- **Bài 56** hỏi: “Robot nhìn point cloud đó như thế nào trong hệ tọa độ của chính nó?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Robot-Frame Point Cloud Transform & Depth Estimation Pipeline**

System này nhận đầu vào là:
- một hoặc nhiều **point cloud samples** (camera frame)
- một hoặc nhiều **camera-to-robot extrinsic configs**
- một bộ **robot-centric region rules**
- một bộ **analysis / reporting options**

## Mỗi point cloud sample tối thiểu chứa
- `pair_id`
- `algorithm_type`
- `point_cloud_path`
- optional `scene_label`

## Mỗi extrinsic config tối thiểu chứa
- `rig_id`
- translation: `tx, ty, tz`
- rotation:
  - có thể dùng **rotation matrix 3x3**
  - hoặc yaw / pitch / roll rồi tự build matrix
- optional `camera_name`

## Nhiệm vụ của system
### Với mỗi point cloud:
1. load camera-frame point cloud
2. load camera-to-robot transform
3. áp dụng rigid transform:
   - `P_robot = R * P_camera + t`
4. tạo robot-frame point cloud
5. phân tích:
   - nearest obstacle trước robot
   - left / center / right occupancy trong robot frame
   - average forward distance
6. xuất:
   - transformed cloud
   - transform report
   - robot-frame obstacle report

<p align="center">
  <img src="images/project_56.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build point cloud sample config
→ build camera-to-robot transform config
→ preview rigid transform bằng NumPy
→ visualize camera-vs-robot point statistics bằng Matplotlib

C++
→ load point cloud camera-frame
→ apply camera-to-robot transform
→ compute robot-frame obstacle statistics
→ export transformed cloud + transform reports
```

Mục tiêu cốt lõi:
- hiểu công thức **rigid transform 3D**
- hiểu khác nhau giữa **camera frame** và **robot frame**
- biết cách đưa point cloud về hệ tọa độ robot
- chuẩn bị nền trực tiếp cho:
  - robot obstacle reasoning
  - locomotion safety
  - corridor estimation
  - navigation / manipulation perception

---

# 6. Pipeline tổng thể

```text
Load Point Cloud Sample Config
Load CameraToRobotTransform Config
Load RobotRegionConfig
Load Visualization Config
Load Runtime Config

Create PointCloudLoader
Create CameraToRobotTransformEngine
Create RobotFramePartitionEngine
Create RobotDistanceAnalysisEngine
Create RobotFrameObstacleSummaryEngine
Create RobotFramePointCloudTransformPipeline

For each point cloud sample:
    1. load camera-frame cloud
    2. load transform
    3. transform all points to robot frame
    4. partition points in robot frame
    5. compute robot-centric distance statistics
    6. summarize forward obstacle layout
    7. save result

After all samples:
    write reports
    export transformed clouds
```

---

# 7. Kiến thức cần

## Python
- OOP
- module
- file handling
- NumPy matrix multiplication
- Matplotlib
- dict / list / BST / Graph

## C++
- class / inheritance / virtual function / polymorphism
- vector
- file parsing / export
- struct / enum
- report-oriented runtime design

## Computer Vision / 3D Geometry
- depth map
- point cloud
- rigid transform
- rotation matrix
- translation vector
- camera frame vs robot frame

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **robot-frame transform pipeline**:

```text
PointCloudCameraFrame
→ CameraToRobotTransform
→ RobotFramePartition
→ RobotDistanceAnalysis
→ ObstacleSummary
→ Report
```

## 2. Python BST
Lưu `pair_id`, `cloud_id`, `transform_result_id`.

## 3. `std::vector<PointCloudSample>`
Danh sách point cloud samples.

## 4. `std::vector<CameraToRobotTransform>`
Danh sách extrinsic transforms.

## 5. `std::vector<Point3D>`
Danh sách điểm 3D camera frame.

## 6. `std::vector<RobotFramePointCloudResult>`
Kết quả sau transform.

## 7. `std::stack<RobotTransformDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Rigid transform 3D
Với mỗi điểm camera-frame:

```text
P_camera = [x, y, z]^T
P_robot  = R * P_camera + t
```

Trong đó:
- `R` là rotation matrix `3x3`
- `t = [tx, ty, tz]^T`

---

## Algorithm 2 — Robot-frame partition
Sau khi transform, chia cloud theo vùng robot-centric, ví dụ:
- `LEFT`
- `CENTER`
- `RIGHT`
- `FRONT_NEAR`
- `FRONT_MID`
- `FRONT_FAR`

Bạn có thể chia theo:
- trục ngang robot
- trục tiến lên robot
- vùng trước mặt robot

---

## Algorithm 3 — Robot distance statistics
Tính ít nhất:
- nearest point phía trước robot
- average forward distance
- point count vùng center-front
- min distance vùng danger zone

---

## Algorithm 4 — Camera vs Robot comparison
Ít nhất so sánh:
- số điểm trước / sau transform
- nearest point camera frame vs robot frame
- center-region distance before/after transform (nếu có ý nghĩa)

---

## Algorithm 5 — Visualization / Export
Bắt buộc có ít nhất 1 đầu ra trực quan:
- export robot-frame `.xyz`
- plot center-front obstacle distance
- plot left/center/right robot-region density
- plot transform summary chart

---

# 9. Cấu trúc folder

```text
mini_project_56_robot_frame_point_cloud_transform_pipeline/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ point_cloud_inputs/
│  │  ├─ cloud_pair_01_bm.xyz
│  │  ├─ cloud_pair_01_sgm.xyz
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ robot_transform_report.txt
│     ├─ robot_region_report.txt
│     ├─ robot_obstacle_summary_report.txt
│     ├─ robot_runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ robot_cloud_pair_01_bm.xyz
│     ├─ robot_cloud_pair_01_sgm.xyz
│     └─ robot_region_density_plot.png
│
├─ config/
│  ├─ point_cloud_sample_config.txt
│  ├─ camera_to_robot_transform_config.txt
│  ├─ robot_region_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ robot_transform_graph_preview.py
│     ├─ robot_transform_bst.py
│     ├─ numpy_robot_transform_preview.py
│     ├─ matplotlib_robot_transform_plotter.py
│     └─ synthetic_transform_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ PointCloudSample.hpp
   │  ├─ CameraToRobotTransform.hpp
   │  ├─ RobotRegionConfig.hpp
   │  ├─ RobotRegionStatistic.hpp
   │  ├─ RobotFramePointCloudResult.hpp
   │  ├─ RobotTransformDebugRecord.hpp
   │  ├─ BaseCameraToRobotTransformEngine.hpp
   │  ├─ CameraToRobotTransformEngine.hpp
   │  ├─ BaseRobotFramePartitionEngine.hpp
   │  ├─ RobotFramePartitionEngine.hpp
   │  ├─ BaseRobotDistanceAnalysisEngine.hpp
   │  ├─ RobotDistanceAnalysisEngine.hpp
   │  ├─ BaseRobotFrameObstacleSummaryEngine.hpp
   │  ├─ RobotFrameObstacleSummaryEngine.hpp
   │  ├─ RobotFramePointCloudTransformPipeline.hpp
   │  └─ RobotFrameReportWriter.hpp
   │
   └─ src/
      ├─ CameraToRobotTransformEngine.cpp
      ├─ RobotFramePartitionEngine.cpp
      ├─ RobotDistanceAnalysisEngine.cpp
      ├─ RobotFrameObstacleSummaryEngine.cpp
      ├─ RobotFramePointCloudTransformPipeline.cpp
      └─ RobotFrameReportWriter.cpp
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
sample_config_path
transform_config_path
robot_region_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `RobotFrameTransformConfigBuilder`

Tạo class con:

```python
class RobotFrameTransformConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_point_cloud_sample(
    pair_id,
    algorithm_type,
    point_cloud_path,
    scene_label=None
)`

### `add_camera_to_robot_transform(
    rig_id,
    tx, ty, tz,
    r11, r12, r13,
    r21, r22, r23,
    r31, r32, r33
)`

### `set_robot_region_rules(
    x_center_threshold,
    front_near_threshold,
    front_mid_threshold
)`

### `set_visualization_options(
    enable_robot_region_plot,
    enable_transform_summary_plot
)`

### `write_sample_config()`
### `write_transform_config()`
### `write_robot_region_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `robot_transform_graph_preview.py`

Tạo class:

```python
class RobotTransformGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
PointCloudCameraFrame
→ CameraToRobotTransform
→ RobotFramePartition
→ RobotDistanceAnalysis
→ ObstacleSummary
→ Report
```

---

# 10.4 Python — `robot_transform_bst.py`

Tạo BST cho `pair_id` / `transform_result_id`.

---

# 10.5 Python — `numpy_robot_transform_preview.py`

Tạo class:

```python
class NumPyRobotTransformPreview:
```

## Hàm cần có

### `load_xyz_points(path)`
### `apply_rigid_transform(points_xyz, rotation_matrix, translation_vector)`
### `partition_robot_regions(points_xyz, x_threshold, near_z, mid_z)`
### `compute_robot_distance_statistics(points_xyz)`

---

# 10.6 Python — `matplotlib_robot_transform_plotter.py`

Tạo class:

```python
class MatplotlibRobotTransformPlotter:
```

## Hàm cần có

### `plot_robot_region_density(region_stats, save_path)`
### `plot_transform_summary(summary_data, save_path)`

---

# 10.7 C++ — `Point3D`

```cpp
struct Point3D
{
    double x;
    double y;
    double z;
};
```

---

# 10.8 C++ — `PointCloudSample`

```cpp
enum class PointCloudSourceType
{
    BM,
    SGM
};

struct PointCloudSample
{
    std::string pair_id;
    PointCloudSourceType algorithm_type;
    std::string point_cloud_path;
    std::string scene_label;
};
```

---

# 10.9 C++ — `CameraToRobotTransform`

```cpp
struct CameraToRobotTransform
{
    std::string rig_id;

    double r11, r12, r13;
    double r21, r22, r23;
    double r31, r32, r33;

    double tx, ty, tz;
};
```

---

# 10.10 C++ — `RobotRegionConfig`

```cpp
struct RobotRegionConfig
{
    double x_center_threshold;
    double front_near_threshold;
    double front_mid_threshold;
};
```

---

# 10.11 C++ — `RobotRegionStatistic`

```cpp
enum class RobotRegionType
{
    LEFT,
    CENTER,
    RIGHT,
    FRONT_NEAR,
    FRONT_MID,
    FRONT_FAR
};

struct RobotRegionStatistic
{
    std::string pair_id;
    RobotRegionType region_type;

    int point_count;
    double ratio;
    double min_distance;
    double max_distance;
    double average_distance;
};
```

---

# 10.12 C++ — `RobotFramePointCloudResult`

```cpp
struct RobotFramePointCloudResult
{
    std::string pair_id;
    PointCloudSourceType algorithm_type;

    std::vector<Point3D> robot_points;

    int total_point_count;
    double nearest_forward_distance;
    double average_forward_distance;
    double center_min_distance;

    std::string obstacle_summary;
};
```

---

# 10.13 C++ — `RobotTransformDebugRecord`

```cpp
struct RobotTransformDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.14 C++ — `BaseCameraToRobotTransformEngine`

Tạo abstract class:

```cpp
class BaseCameraToRobotTransformEngine
{
public:
    virtual std::vector<Point3D> transform(
        const std::vector<Point3D>& camera_points,
        const CameraToRobotTransform& transform
    ) const = 0;

    virtual ~BaseCameraToRobotTransformEngine() = default;
};
```

---

# 10.15 C++ — `CameraToRobotTransformEngine`

Kế thừa `BaseCameraToRobotTransformEngine`.

## Nhiệm vụ
- áp dụng rigid transform cho toàn bộ point cloud

---

# 10.16 C++ — `BaseRobotFramePartitionEngine`

Tạo abstract class:

```cpp
class BaseRobotFramePartitionEngine
{
public:
    virtual std::vector<RobotRegionStatistic> partition(
        const std::string& pair_id,
        const std::vector<Point3D>& robot_points,
        const RobotRegionConfig& config
    ) const = 0;

    virtual ~BaseRobotFramePartitionEngine() = default;
};
```

---

# 10.17 C++ — `RobotFramePartitionEngine`

Kế thừa `BaseRobotFramePartitionEngine`.

## Nhiệm vụ
- chia robot-frame cloud theo LEFT / CENTER / RIGHT
- chia theo FRONT_NEAR / FRONT_MID / FRONT_FAR
- tạo thống kê vùng

---

# 10.18 C++ — `BaseRobotDistanceAnalysisEngine`

Tạo abstract class:

```cpp
class BaseRobotDistanceAnalysisEngine
{
public:
    virtual void analyze(
        RobotFramePointCloudResult& result,
        const std::vector<Point3D>& robot_points,
        const std::vector<RobotRegionStatistic>& region_stats
    ) const = 0;

    virtual ~BaseRobotDistanceAnalysisEngine() = default;
};
```

---

# 10.19 C++ — `RobotDistanceAnalysisEngine`

Kế thừa `BaseRobotDistanceAnalysisEngine`.

## Nhiệm vụ
- tính nearest forward distance
- average forward distance
- center min distance

---

# 10.20 C++ — `BaseRobotFrameObstacleSummaryEngine`

Tạo abstract class:

```cpp
class BaseRobotFrameObstacleSummaryEngine
{
public:
    virtual std::string summarize(
        const RobotFramePointCloudResult& result,
        const std::vector<RobotRegionStatistic>& region_stats
    ) const = 0;

    virtual ~BaseRobotFrameObstacleSummaryEngine() = default;
};
```

---

# 10.21 C++ — `RobotFrameObstacleSummaryEngine`

Kế thừa `BaseRobotFrameObstacleSummaryEngine`.

## Nhiệm vụ
- tạo obstacle summary kiểu robot-centric, ví dụ:
```text
A close obstacle exists in the center-front region at 0.68m.
Left region is denser than right region.
Front corridor is partially blocked.
```

---

# 10.22 C++ — `RobotFramePointCloudTransformPipeline`

Tạo class trung tâm:

```cpp
class RobotFramePointCloudTransformPipeline
```

## Thuộc tính

```cpp
private:
    std::vector<PointCloudSample> samples;
    std::vector<CameraToRobotTransform> transforms;
    RobotRegionConfig region_config;

    std::shared_ptr<BaseCameraToRobotTransformEngine> transform_engine;
    std::shared_ptr<BaseRobotFramePartitionEngine> partition_engine;
    std::shared_ptr<BaseRobotDistanceAnalysisEngine> distance_engine;
    std::shared_ptr<BaseRobotFrameObstacleSummaryEngine> summary_engine;

    std::vector<RobotRegionStatistic> region_statistics;
    std::vector<RobotFramePointCloudResult> results;
    std::stack<RobotTransformDebugRecord> debug_history;
```

## Hàm cần có

### `load_sample_config(const std::string& path)`
### `load_transform_config(const std::string& path)`
### `load_robot_region_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each point cloud sample:
    load camera-frame cloud
    find matching transform
    robot_points = transform_engine.transform(camera_points, transform)

    region_stats = partition_engine.partition(...)
    build RobotFramePointCloudResult
    distance_engine.analyze(...)
    result.obstacle_summary = summary_engine.summarize(...)

    save result
    push debug history
```

### `const std::vector<RobotRegionStatistic>& get_region_statistics() const`
### `const std::vector<RobotFramePointCloudResult>& get_results() const`
### `std::vector<RobotTransformDebugRecord> get_debug_history_reverse()`

---

# 10.23 C++ — `RobotFrameReportWriter`

Tạo class:

```cpp
class RobotFrameReportWriter
```

## Hàm cần có

### `write_robot_transform_report(...)`
Ví dụ:

```text
[Robot Transform Report]
Pair: hallway_01
Camera cloud transformed to robot frame successfully.
Total points: 18234
```

### `write_robot_region_report(...)`
Ví dụ:

```text
[Robot Region Report]
Pair: hallway_01
Region: CENTER
Point Count: 4210
Average Distance: 1.92
```

### `write_robot_obstacle_summary_report(...)`
Ví dụ:

```text
[Robot Obstacle Summary]
Pair: hallway_01
A close obstacle exists in the center-front region at 0.68m.
Front corridor is partially blocked.
```

### `write_robot_runtime_report(...)`
### `write_xyz_file(const RobotFramePointCloudResult& result, const std::string& save_path)`
### `write_reverse_debug_history(...)`

---

# 10.24 C++ — `main.cpp`

## Yêu cầu

```text
Load point cloud sample config
Load camera-to-robot transform config
Load robot region config
Load visualization config
Load runtime config

Create:
    CameraToRobotTransformEngine
    RobotFramePartitionEngine
    RobotDistanceAnalysisEngine
    RobotFrameObstacleSummaryEngine

Create RobotFramePointCloudTransformPipeline
Run
Write reports
Export transformed clouds
```

---

# 11. Luật robot-frame transform của project

## Luật 1 — Không được bỏ qua extrinsic transform
Nếu vẫn phân tích point cloud ở camera frame thì chưa phải mục tiêu của Bài 56.

## Luật 2 — Robot-frame analysis phải dùng point sau transform
Mọi thống kê obstacle của bài này phải dựa trên **robot_points**.

## Luật 3 — Center-front region là bắt buộc
Vì đây là vùng quan trọng nhất cho robot tiến thẳng.

## Luật 4 — Output phải gồm transformed cloud + report
Không chỉ in console.

## Luật 5 — Bài này là cầu nối sang robot perception pipeline
Tức là output phải usable cho:
- obstacle danger scoring
- free-space corridor estimation
- locomotion safety analysis

---

# 12. Output mong muốn

## Config
```text
config/point_cloud_sample_config.txt
config/camera_to_robot_transform_config.txt
config/robot_region_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/robot_transform_report.txt
assets/outputs/robot_region_report.txt
assets/outputs/robot_obstacle_summary_report.txt
assets/outputs/robot_runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Robot-frame cloud exports
```text
assets/outputs/robot_cloud_pair_01_bm.xyz
assets/outputs/robot_cloud_pair_01_sgm.xyz
```

## Plots
```text
assets/outputs/robot_region_density_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho point cloud / extrinsic transform / robot region
- preview rigid transform bằng NumPy
- visualize robot-region density và transform summary

## C++
- chạy transform camera → robot
- phân tích point cloud theo robot-centric regions
- tạo obstacle summary usable cho pipeline phía sau

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là bước **“gắn perception vào cơ thể robot”**:

```text
Stereo pair
→ disparity
→ depth
→ point cloud
→ camera-frame analysis
→ camera-to-robot transform
→ robot-frame obstacle reasoning
```

Sau Bài 56, bạn đã tiến rất gần đến các project kiểu:
- front obstacle danger analyzer
- free-space corridor detector
- humanoid walking safety perception
- simple local navigation perception

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho point cloud / transform / robot region / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / transform result ids
- [ ] Python có NumPy rigid transform preview
- [ ] Python có Matplotlib robot-region plot
- [ ] C++ có `PointCloudSample`
- [ ] C++ có `CameraToRobotTransform`
- [ ] C++ có `RobotRegionConfig`
- [ ] C++ có `RobotRegionStatistic`
- [ ] C++ có `RobotFramePointCloudResult`
- [ ] C++ có `CameraToRobotTransformEngine`
- [ ] C++ có `RobotFramePartitionEngine`
- [ ] C++ có `RobotDistanceAnalysisEngine`
- [ ] C++ có `RobotFrameObstacleSummaryEngine`
- [ ] C++ có `RobotFramePointCloudTransformPipeline`
- [ ] C++ ghi đủ report + export transformed clouds

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 56**, bước hợp lý nhất để tiếp tục Đợt 12 là:

# **Bài 57: Robot-Frame Obstacle Distance Analyzer**

Ý tưởng:
```text
robot-frame point cloud
→ front danger zone analysis
→ center corridor distance analysis
→ left / right clearance estimation
→ obstacle danger scoring
→ locomotion safety report
```

Tức là mạch sẽ đi:

```text
Bài 54: Point cloud reconstruction
→ Bài 55: Camera-frame point cloud analysis
→ Bài 56: Camera-to-Robot transform
→ Bài 57: Robot-frame obstacle distance analysis
```
