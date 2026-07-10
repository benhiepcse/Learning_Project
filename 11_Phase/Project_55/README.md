# 🤖 Bài 55: Camera-Frame Point Cloud Analyzer — Bộ phân tích point cloud trong camera frame cho Humanoid Robot AI Perception

> Mini Project số 55 trong **Đợt 11**  
> **Bài 55 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11** và đi tiếp trực tiếp từ **Bài 54**.
>
> Nếu:
>
> - **Bài 51** giúp bạn tạo **disparity maps**
> - **Bài 52** giúp bạn chuyển **disparity → depth**
> - **Bài 53** giúp bạn **đánh giá chất lượng depth**
> - **Bài 54** giúp bạn dựng **point cloud từ depth**
>
> thì **Bài 55** là bước tiếp theo rất tự nhiên:
>
> ```text
> point cloud trong camera frame
> → chia vùng / chia sector
> → phân tích density / distance / spread
> → tìm nearest / farthest points
> → tóm tắt obstacle structure theo camera frame
> → camera-frame obstacle report
> ```
>
> Đây là bước perception bắt đầu **đọc hiểu point cloud**, chứ không chỉ dựng point cloud rồi dừng lại.
>
> Từ đây bạn sẽ chuẩn bị rất tốt cho:
> - camera-to-robot transform
> - robot-frame obstacle reasoning
> - free-space corridor estimation
> - walking safety analysis

---

# 📌 Mục lục

- [1. Vì sao Bài 55 xuất hiện sau Bài 54](#1-vì-sao-bài-55-xuất-hiện-sau-bài-54)
- [2. Đợt 11 đang đẩy chuỗi stereo-depth-3D đi đâu](#2-đợt-11-đang-đẩy-chuỗi-stereo-depth-3d-đi-đâu)
- [3. Bài 55 nâng từ Bài 54 lên chỗ nào](#3-bài-55-nâng-từ-bài-54-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật camera-frame point cloud analysis của project](#11-luật-camera-frame-point-cloud-analysis-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 55 xuất hiện sau Bài 54

## Bài 54 đang làm gì?
Bài 54 giúp bạn có:

```text
depth map
→ back-projection
→ point cloud camera-frame
→ point filtering
→ point statistics
```

Tức là bạn đã có:
- point cloud 3D
- các điểm `(x, y, z)` trong **camera frame**

## Nhưng còn thiếu gì?
Robot không dùng point cloud chỉ để “có nhiều điểm 3D”.  
Robot cần hiểu point cloud đó nói gì về scene phía trước:

- vật cản nằm bên trái, giữa hay bên phải?
- điểm gần nhất ở đâu?
- vùng trung tâm có dày đặc vật cản không?
- cloud có đang tập trung ở một dải depth nhất định không?
- vùng trước mặt robot có trống hay bị chắn?

## Bài 55 lấp đúng chỗ đó
Bài 55 sẽ làm:

```text
point cloud
→ chia vùng / chia sector
→ phân tích density
→ nearest / farthest point analysis
→ local obstacle summary
→ camera-frame obstacle report
```

Đây là bước chuyển từ **“có point cloud”** sang **“đọc hiểu point cloud trong camera frame”**.

---

# 2. Đợt 11 đang đẩy chuỗi stereo-depth-3D đi đâu

Nếu gom Bài 51–55 lại, chuỗi Đợt 11 bây giờ là:

```text
Stereo pair
→ disparity
→ depth
→ depth quality
→ point cloud
→ camera-frame point cloud analysis
```

Tức là Đợt 11 không chỉ dựng 3D, mà còn bắt đầu **suy luận hình học cơ bản trên point cloud**.

Đây là đúng hướng cho AI Perception:
- không dừng ở output trung gian
- mà tiến dần sang **scene understanding trong hệ tọa độ của robot / camera**

---

# 3. Bài 55 nâng từ Bài 54 lên chỗ nào

## Bài 54
- depth → point cloud
- point filtering
- point statistics tổng quát

## Bài 55
- chia point cloud theo vùng / sector
- density analysis
- nearest / farthest point analysis
- center corridor / center sector analysis
- obstacle summary theo camera frame

### Nói ngắn gọn:
- **Bài 54** hỏi: “Point cloud 3D được dựng như thế nào?”
- **Bài 55** hỏi: “Point cloud đó cho biết gì về vật cản phía trước camera?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Camera-Frame Point Cloud Analyzer**

System này nhận đầu vào là:
- một hoặc nhiều **point cloud samples** (đã dựng từ Bài 54)
- một bộ **region / sector partition rules**
- một bộ **distance thresholds**
- một bộ **analysis / reporting options**

## Mỗi point cloud sample tối thiểu chứa
- `pair_id`
- `algorithm_type` (`BM` hoặc `SGM`)
- `point_cloud_path`
- optional `scene_label`

## Nhiệm vụ của system
### Với mỗi point cloud:
1. load point cloud
2. chia điểm theo vùng:
   - left / center / right
   - near / mid / far
   - optional top / middle / bottom theo `y`
3. tính density / point count của từng vùng
4. tìm:
   - nearest point
   - farthest point
   - average Z
   - min Z của center region
5. tạo obstacle summary cho camera frame
6. ghi report

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build point cloud sample config
→ build region / threshold config
→ preview point partition bằng NumPy
→ preview density / nearest-point statistics
→ visualize point cloud summary bằng Matplotlib

C++
→ load point clouds
→ partition points into camera-frame regions
→ compute density / nearest / farthest / center stats
→ summarize obstacle layout
→ export camera-frame analysis reports
```

Mục tiêu cốt lõi:
- hiểu cách **phân tích point cloud theo camera frame**
- hiểu ý nghĩa của **center region** đối với forward perception
- biết cách rút ra obstacle statistics từ point cloud
- chuẩn bị nền trực tiếp cho:
  - camera-to-robot transform
  - robot-frame obstacle reasoning
  - free-space corridor estimation

---

# 6. Pipeline tổng thể

```text
Load Point Cloud Sample Config
Load Region Partition Config
Load Distance Threshold Config
Load Visualization Config
Load Runtime Config

Create PointCloudLoader
Create PointPartitionEngine
Create PointDensityAnalysisEngine
Create PointDistanceAnalysisEngine
Create CameraFrameObstacleSummaryEngine
Create CameraFramePointCloudAnalyzer

For each point cloud sample:
    1. load point cloud
    2. partition points into regions
    3. compute density statistics
    4. compute nearest / farthest / average depth statistics
    5. summarize obstacle layout
    6. save result

After all samples:
    write reports
    write plots
```

---

# 7. Kiến thức cần

## Python
- OOP
- module
- file handling
- NumPy
- Matplotlib
- dict / list / BST / Graph

## C++
- class / inheritance / virtual function
- vector
- enum
- file parsing / export
- report-oriented runtime design

## Computer Vision / 3D Geometry
- point cloud basics
- camera frame
- point partitioning
- distance statistics
- obstacle region analysis

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **camera-frame point cloud analysis pipeline**:

```text
PointCloud
→ Partition
→ DensityAnalysis
→ DistanceAnalysis
→ ObstacleSummary
→ Report
```

## 2. Python BST
Lưu `pair_id`, `cloud_id`, `analysis_id`.

## 3. `std::vector<PointCloudSample>`
Danh sách point cloud samples.

## 4. `std::vector<CameraRegionStatistic>`
Thống kê theo vùng.

## 5. `std::vector<PointCloudAnalysisResult>`
Kết quả phân tích point cloud.

## 6. `std::vector<Point3D>`
Danh sách điểm 3D đã nạp.

## 7. `std::stack<PointCloudAnalysisDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Point partition theo vùng trái / giữa / phải
Chia theo trục `X` trong camera frame, ví dụ:
- `LEFT`: `x < -x_threshold`
- `CENTER`: `-x_threshold <= x <= x_threshold`
- `RIGHT`: `x > x_threshold`

---

## Algorithm 2 — Near / Mid / Far partition
Chia theo `Z`, ví dụ:
- `NEAR`: `z <= near_z`
- `MID`: `near_z < z <= mid_z`
- `FAR`: `z > mid_z`

---

## Algorithm 3 — Density analysis
Tính ít nhất:
- point count từng vùng
- ratio từng vùng
- center region density

---

## Algorithm 4 — Distance analysis
Tính ít nhất:
- nearest point distance
- farthest point distance
- average Z
- min Z của center region

---

## Algorithm 5 — Obstacle summary
Tạo summary kiểu:
- center region crowded / sparse
- left region closer than right region
- near obstacle exists in center corridor
- front space relatively open / partially blocked

---

## Algorithm 6 — Visualization
Bắt buộc có ít nhất 1 dạng visualize bằng Matplotlib, ví dụ:
- region density bar chart
- nearest/farthest/average distance chart
- center vs left vs right comparison chart

---

# 9. Cấu trúc folder

```text
mini_project_55_camera_frame_point_cloud_analyzer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ point_cloud_inputs/
│  │  ├─ cloud_pair_01_bm.xyz
│  │ ├─ cloud_pair_01_sgm.xyz
│  │ └─ ...
│  │
│  └─ outputs/
│     ├─ camera_region_report.txt
│     ├─ point_distance_report.txt
│     ├─ obstacle_summary_report.txt
│     ├─ point_cloud_runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ region_density_plot.png
│     └─ distance_summary_plot.png
│
├─ config/
│  ├─ point_cloud_sample_config.txt
│  ├─ region_partition_config.txt
│  ├─ distance_threshold_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ camera_cloud_graph_preview.py
│     ├─ camera_cloud_bst.py
│     ├─ numpy_camera_cloud_preview.py
│     ├─ matplotlib_camera_cloud_plotter.py
│     └─ synthetic_point_cloud_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ PointCloudSample.hpp
   │  ├─ RegionPartitionConfig.hpp
   │  ├─ DistanceThresholdConfig.hpp
   │  ├─ CameraRegionStatistic.hpp
   │  ├─ PointCloudAnalysisResult.hpp
   │  ├─ PointCloudAnalysisDebugRecord.hpp
   │  ├─ BasePointPartitionEngine.hpp
   │  ├─ PointPartitionEngine.hpp
   │  ├─ BasePointDensityAnalysisEngine.hpp
   │  ├─ PointDensityAnalysisEngine.hpp
   │  ├─ BasePointDistanceAnalysisEngine.hpp
   │  ├─ PointDistanceAnalysisEngine.hpp
   │  ├─ BaseCameraFrameObstacleSummaryEngine.hpp
   │  ├─ CameraFrameObstacleSummaryEngine.hpp
   │  ├─ CameraFramePointCloudAnalyzer.hpp
   │  └─ CameraFramePointCloudReportWriter.hpp
   │
   └─ src/
      ├─ PointPartitionEngine.cpp
      ├─ PointDensityAnalysisEngine.cpp
      ├─ PointDistanceAnalysisEngine.cpp
      ├─ CameraFrameObstacleSummaryEngine.cpp
      ├─ CameraFramePointCloudAnalyzer.cpp
      └─ CameraFramePointCloudReportWriter.cpp
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
region_config_path
distance_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `CameraFramePointCloudAnalyzerConfigBuilder`

Tạo class con:

```python
class CameraFramePointCloudAnalyzerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_point_cloud_sample(
    pair_id,
    algorithm_type,
    point_cloud_path,
    scene_label=None
)`

### `set_region_partition(
    x_center_threshold,
    near_z_threshold,
    mid_z_threshold
)`

### `set_distance_thresholds(
    obstacle_warning_z,
    obstacle_danger_z
)`

### `set_visualization_options(
    enable_region_density_plot,
    enable_distance_plot
)`

### `write_sample_config()`
### `write_region_config()`
### `write_distance_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `camera_cloud_graph_preview.py`

Tạo class:

```python
class CameraCloudGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
PointCloud
→ Partition
→ DensityAnalysis
→ DistanceAnalysis
→ ObstacleSummary
→ Report
```

---

# 10.4 Python — `camera_cloud_bst.py`

Tạo BST cho `pair_id` / `analysis_id`.

---

# 10.5 Python — `numpy_camera_cloud_preview.py`

Tạo class:

```python
class NumPyCameraCloudPreview:
```

## Hàm cần có

### `load_xyz_points(path)`
### `partition_left_center_right(points_xyz, x_threshold)`
### `partition_near_mid_far(points_xyz, near_z, mid_z)`
### `compute_density_statistics(partitions)`
### `compute_distance_statistics(points_xyz)`

---

# 10.6 Python — `matplotlib_camera_cloud_plotter.py`

Tạo class:

```python
class MatplotlibCameraCloudPlotter:
```

## Hàm cần có

### `plot_region_density(region_stats, save_path)`
### `plot_distance_summary(distance_stats, save_path)`

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

# 10.9 C++ — `RegionPartitionConfig`

```cpp
struct RegionPartitionConfig
{
    double x_center_threshold;
    double near_z_threshold;
    double mid_z_threshold;
};
```

---

# 10.10 C++ — `DistanceThresholdConfig`

```cpp
struct DistanceThresholdConfig
{
    double obstacle_warning_z;
    double obstacle_danger_z;
};
```

---

# 10.11 C++ — `CameraRegionStatistic`

```cpp
enum class CameraRegionType
{
    LEFT,
    CENTER,
    RIGHT,
    NEAR,
    MID,
    FAR
};

struct CameraRegionStatistic
{
    std::string pair_id;
    CameraRegionType region_type;

    int point_count;
    double ratio;
    double min_z;
    double max_z;
    double average_z;
};
```

---

# 10.12 C++ — `PointCloudAnalysisResult`

```cpp
struct PointCloudAnalysisResult
{
    std::string pair_id;
    PointCloudSourceType algorithm_type;

    int total_point_count;

    double nearest_distance;
    double farthest_distance;
    double average_z;
    double center_min_z;

    std::string obstacle_summary;
};
```

---

# 10.13 C++ — `PointCloudAnalysisDebugRecord`

```cpp
struct PointCloudAnalysisDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.14 C++ — `BasePointPartitionEngine`

Tạo abstract class:

```cpp
class BasePointPartitionEngine
{
public:
    virtual std::vector<CameraRegionStatistic> partition(
        const std::string& pair_id,
        const std::vector<Point3D>& points,
        const RegionPartitionConfig& config
    ) const = 0;

    virtual ~BasePointPartitionEngine() = default;
};
```

---

# 10.15 C++ — `PointPartitionEngine`

Kế thừa `BasePointPartitionEngine`.

## Nhiệm vụ
- chia point cloud theo left / center / right
- chia point cloud theo near / mid / far
- tính thống kê sơ bộ từng vùng

---

# 10.16 C++ — `BasePointDensityAnalysisEngine`

Tạo abstract class:

```cpp
class BasePointDensityAnalysisEngine
{
public:
    virtual void analyze_density(
        std::vector<CameraRegionStatistic>& region_stats,
        int total_points
    ) const = 0;

    virtual ~BasePointDensityAnalysisEngine() = default;
};
```

---

# 10.17 C++ — `PointDensityAnalysisEngine`

Kế thừa `BasePointDensityAnalysisEngine`.

## Nhiệm vụ
- tính ratio của từng vùng
- làm rõ center density / side density

---

# 10.18 C++ — `BasePointDistanceAnalysisEngine`

Tạo abstract class:

```cpp
class BasePointDistanceAnalysisEngine
{
public:
    virtual void analyze_distances(
        PointCloudAnalysisResult& result,
        const std::vector<Point3D>& points,
        const std::vector<CameraRegionStatistic>& region_stats
    ) const = 0;

    virtual ~BasePointDistanceAnalysisEngine() = default;
};
```

---

# 10.19 C++ — `PointDistanceAnalysisEngine`

Kế thừa `BasePointDistanceAnalysisEngine`.

## Nhiệm vụ
- tính nearest / farthest / average distance
- tính center min Z

---

# 10.20 C++ — `BaseCameraFrameObstacleSummaryEngine`

Tạo abstract class:

```cpp
class BaseCameraFrameObstacleSummaryEngine
{
public:
    virtual std::string summarize(
        const PointCloudAnalysisResult& result,
        const std::vector<CameraRegionStatistic>& region_stats,
        const DistanceThresholdConfig& thresholds
    ) const = 0;

    virtual ~BaseCameraFrameObstacleSummaryEngine() = default;
};
```

---

# 10.21 C++ — `CameraFrameObstacleSummaryEngine`

Kế thừa `BaseCameraFrameObstacleSummaryEngine`.

## Nhiệm vụ
- tạo obstacle summary dựa trên:
  - center density
  - center min Z
  - near-region density
  - warning / danger thresholds

Ví dụ:
```text
Center region is moderately crowded. A near obstacle exists at 0.72m.
Left side is denser than right side.
```

---

# 10.22 C++ — `CameraFramePointCloudAnalyzer`

Tạo class trung tâm:

```cpp
class CameraFramePointCloudAnalyzer
```

## Thuộc tính

```cpp
private:
    std::vector<PointCloudSample> samples;
    RegionPartitionConfig region_config;
    DistanceThresholdConfig distance_config;

    std::shared_ptr<BasePointPartitionEngine> partition_engine;
    std::shared_ptr<BasePointDensityAnalysisEngine> density_engine;
    std::shared_ptr<BasePointDistanceAnalysisEngine> distance_engine;
    std::shared_ptr<BaseCameraFrameObstacleSummaryEngine> summary_engine;

    std::vector<CameraRegionStatistic> region_statistics;
    std::vector<PointCloudAnalysisResult> analysis_results;
    std::stack<PointCloudAnalysisDebugRecord> debug_history;
```

## Hàm cần có

### `load_sample_config(const std::string& path)`
### `load_region_config(const std::string& path)`
### `load_distance_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each point cloud sample:
    load points
    partition points into camera regions
    density_engine.analyze_density(...)
    build analysis result
    distance_engine.analyze_distances(...)
    summary = summary_engine.summarize(...)
    save result
    push debug history
```

### `const std::vector<CameraRegionStatistic>& get_region_statistics() const`
### `const std::vector<PointCloudAnalysisResult>& get_analysis_results() const`
### `std::vector<PointCloudAnalysisDebugRecord> get_debug_history_reverse()`

---

# 10.23 C++ — `CameraFramePointCloudReportWriter`

Tạo class:

```cpp
class CameraFramePointCloudReportWriter
```

## Hàm cần có

### `write_camera_region_report(...)`
Ví dụ:

```text
[Camera Region Report]
Pair: hallway_01
Region: CENTER
Point Count: 4210
Ratio: 0.28
Average Z: 1.94
```

### `write_point_distance_report(...)`
Ví dụ:

```text
[Point Distance Report]
Pair: hallway_01
Nearest Distance: 0.72
Farthest Distance: 4.82
Average Z: 2.11
Center Min Z: 0.72
```

### `write_obstacle_summary_report(...)`
Ví dụ:

```text
[Obstacle Summary]
Pair: hallway_01
Center region is moderately crowded. A near obstacle exists at 0.72m.
```

### `write_point_cloud_runtime_report(...)`

### `write_reverse_debug_history(...)`

---

# 10.24 C++ — `main.cpp`

## Yêu cầu

```text
Load point cloud sample config
Load region partition config
Load distance threshold config
Load visualization config
Load runtime config

Create:
    PointPartitionEngine
    PointDensityAnalysisEngine
    PointDistanceAnalysisEngine
    CameraFrameObstacleSummaryEngine

Create CameraFramePointCloudAnalyzer
Run
Write reports
```

---

# 11. Luật camera-frame point cloud analysis của project

## Luật 1 — Không được chỉ nhìn tổng point count
Point cloud analysis phải có ít nhất:
- region partition
- density stats
- distance stats

## Luật 2 — Center region là bắt buộc
Vì forward perception của robot phụ thuộc rất nhiều vào vùng giữa phía trước.

## Luật 3 — Obstacle summary phải dựa trên số liệu
Không được viết summary cảm tính; nó phải dựa trên:
- center min Z
- density
- near-region occupancy

## Luật 4 — Visualization là bắt buộc
Ít nhất phải có 1 plot density hoặc distance.

## Luật 5 — Output phải sẵn sàng cho bước robot-frame
Tức là report phải đủ để bạn quyết định:
- vùng nào đáng quan tâm nhất
- vật cản gần nằm ở đâu
- cloud có đang chặn hướng tiến lên hay không

---

# 12. Output mong muốn

## Config
```text
config/point_cloud_sample_config.txt
config/region_partition_config.txt
config/distance_threshold_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/camera_region_report.txt
assets/outputs/point_distance_report.txt
assets/outputs/obstacle_summary_report.txt
assets/outputs/point_cloud_runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/region_density_plot.png
assets/outputs/distance_summary_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho point cloud samples / region partition / thresholds
- preview region density và distance stats bằng NumPy
- visualize density / distance summary bằng Matplotlib

## C++
- chạy camera-frame point cloud analysis
- tóm tắt vật cản theo left / center / right và near / mid / far
- chuẩn bị dữ liệu cho robot-frame reasoning

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là bước **đọc hiểu point cloud ngay trong camera frame**:

```text
Stereo pair
→ disparity
→ depth
→ point cloud
→ camera-frame cloud analysis
→ robot-frame transform
→ obstacle reasoning
```

Sau Bài 55, việc đi tiếp sang:
- camera-to-robot transform
- robot-centric region analysis
- free-space corridor estimation
- walking safety report

sẽ mượt hơn rất nhiều.

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho point cloud samples / region / distance / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / analysis ids
- [ ] Python có NumPy point cloud partition preview
- [ ] Python có Matplotlib region density / distance plots
- [ ] C++ có `PointCloudSample`
- [ ] C++ có `RegionPartitionConfig`
- [ ] C++ có `DistanceThresholdConfig`
- [ ] C++ có `CameraRegionStatistic`
- [ ] C++ có `PointCloudAnalysisResult`
- [ ] C++ có `PointPartitionEngine`
- [ ] C++ có `PointDensityAnalysisEngine`
- [ ] C++ có `PointDistanceAnalysisEngine`
- [ ] C++ có `CameraFrameObstacleSummaryEngine`
- [ ] C++ có `CameraFramePointCloudAnalyzer`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 55**, bước hợp lý nhất là:

# **Bài 56: Camera-to-Robot Point Cloud Transform Analyzer**

Ý tưởng:
```text
camera-frame point cloud
→ apply extrinsic transform
→ robot-frame point cloud
→ compare camera vs robot coordinates
→ robot-centric spatial summary
→ transform report
```

Tức là mạch sẽ đi:

```text
Bài 54: Point cloud reconstruction
→ Bài 55: Camera-frame point cloud analysis
→ Bài 56: Camera-to-Robot point cloud transform
```
