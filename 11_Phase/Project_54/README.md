# 🤖 Bài 54: Stereo Point Cloud Reconstruction Lab — Bộ dựng point cloud từ depth map cho Humanoid Robot AI Perception

> Mini Project số 54 trong **Đợt 11**  
> **Bài 54 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11** và đi tiếp trực tiếp từ **Bài 53**.
>
> Nếu:
>
> - **Bài 51** giúp bạn tạo và so sánh **disparity maps**
> - **Bài 52** giúp bạn chuyển **disparity → depth**
> - **Bài 53** giúp bạn **đánh giá chất lượng depth map**
>
> thì **Bài 54** là bước tiếp theo rất tự nhiên và cực kỳ quan trọng:
>
> ```text
> depth map
> → back-projection
> → point cloud generation
> → point filtering
> → point statistics
> → point cloud quality report
> ```
>
> Đây là bước mà perception bắt đầu thật sự bước vào **3D geometry**:
>
> - từ ảnh depth 2D
> - sang tập điểm 3D trong camera frame
>
> Và đó là nền trực tiếp cho:
> - camera-to-robot transform
> - obstacle perception
> - free-space reasoning
> - robot-centric scene understanding

---

# 📌 Mục lục

- [1. Vì sao Bài 54 xuất hiện sau Bài 53](#1-vì-sao-bài-54-xuất-hiện-sau-bài-53)
- [2. Đợt 11 đang hoàn thiện chuỗi stereo-depth ra sao](#2-đợt-11-đang-hoàn-thiện-chuỗi-stereo-depth-ra-sao)
- [3. Bài 54 nâng từ Bài 53 lên chỗ nào](#3-bài-54-nâng-từ-bài-53-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật point cloud reconstruction của project](#11-luật-point-cloud-reconstruction-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 54 xuất hiện sau Bài 53

## Bài 53 đang làm gì?
Bài 53 giúp bạn có:

```text
depth map
→ invalid / hole analysis
→ ROI quality analysis
→ quality score / reliability report
```

Tức là bạn đã có:
- **depth map**
- và một mức độ hiểu biết về **chất lượng của depth map đó**

## Nhưng còn thiếu gì?
Robot không dùng depth map chỉ để “xem màu sắc gần xa”.  
Robot cần một biểu diễn 3D thật sự hơn:

> **Từ depth map, hãy dựng ra point cloud trong không gian 3D.**

Lúc đó bạn mới có thể:
- biết mỗi pixel nằm ở đâu trong camera frame
- gom thành bề mặt / vật cản / vùng không gian
- transform sang robot frame
- làm obstacle reasoning hoặc free-space estimation

## Bài 54 lấp đúng chỗ đó
Bài 54 sẽ làm:

```text
depth map
→ back-projection
→ XYZ points
→ point cloud filtering
→ point statistics
→ point cloud report
```

Đây là bước chuyển từ **2D depth representation** sang **3D point representation**.

---

# 2. Đợt 11 đang hoàn thiện chuỗi stereo-depth ra sao

Nếu gom Bài 51–54 lại, chuỗi Đợt 11 rất rõ:

```text
Stereo pair
→ disparity map
→ depth map
→ depth quality analysis
→ point cloud reconstruction
```

Tức là Đợt 11 đang hoàn thiện **toàn bộ đoạn từ stereo pair đến 3D point cloud đầu tiên**.

Đây là một khối cực quan trọng cho AI Perception vì sau đợt này bạn đã có:
- stereo matching
- depth estimation
- depth QA
- point cloud reconstruction

---

# 3. Bài 54 nâng từ Bài 53 lên chỗ nào

## Bài 53
- phân tích chất lượng depth
- đánh giá usable / poor
- thống kê invalid / ROI / near-mid-far

## Bài 54
- back-project depth thành điểm 3D
- dựng point cloud camera-frame
- lọc điểm invalid / out-of-range
- thống kê point cloud
- chuẩn bị cho transform sang robot frame

### Nói ngắn gọn:
- **Bài 53** hỏi: “Depth map này có đáng tin không?”
- **Bài 54** hỏi: “Từ depth map đó, point cloud 3D trông như thế nào?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Point Cloud Reconstruction Lab**

System này nhận đầu vào là:
- một hoặc nhiều **depth map samples**
- một hoặc nhiều **camera intrinsic / stereo rig configs**
- một bộ **point filtering rules**
- một bộ **visualization / reporting options**

## Mỗi depth sample tối thiểu chứa
- `pair_id`
- `algorithm_type` (`BM` hoặc `SGM`)
- `depth_map_path`
- optional `scene_label`

## Mỗi rig config tối thiểu chứa
- `fx`
- `fy`
- `cx`
- `cy`
- image width / height

## Nhiệm vụ của system
### Với mỗi depth map:
1. load depth map
2. đọc intrinsic config
3. back-project từng pixel depth hợp lệ thành điểm 3D:
   - `(X, Y, Z)` trong **camera frame**
4. loại bỏ điểm:
   - invalid depth
   - depth quá xa / quá gần
5. thống kê point cloud:
   - point count
   - min / max / average depth
   - XYZ ranges
6. optional xuất point cloud ra:
   - `.txt`
   - `.xyz`
   - hoặc định dạng đơn giản tự thiết kế
7. ghi point cloud report

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build rig config
→ build depth sample config
→ preview back-projection bằng NumPy
→ preview point cloud statistics
→ visualize point distribution summary

C++
→ load depth maps
→ back-project to 3D points
→ filter invalid / out-of-range points
→ compute point cloud statistics
→ export point cloud reports
```

Mục tiêu cốt lõi:
- hiểu công thức **back-projection từ depth sang 3D**
- hiểu **camera-frame point cloud** là gì
- biết cách xử lý invalid / noisy points
- chuẩn bị nền trực tiếp cho:
  - camera-to-robot transform
  - obstacle reasoning
  - robot-centric perception
  - free-space corridor estimation

---

# 6. Pipeline tổng thể

```text
Load Camera Rig Config
Load Depth Sample Config
Load Point Filter Config
Load Visualization Config
Load Runtime Config

Create DepthMapLoader
Create BackProjectionEngine
Create PointFilterEngine
Create PointCloudStatisticsEngine
Create StereoPointCloudReconstructionLab

For each depth sample:
    1. load depth map
    2. back-project valid pixels to 3D points
    3. filter invalid / out-of-range points
    4. compute point cloud statistics
    5. save point cloud result

After all samples:
    write reports
    write exported point cloud files
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
- OpenCV matrix handling
- file export
- report-oriented runtime design

## Computer Vision / 3D Geometry
- depth map
- intrinsic matrix
- normalized image coordinates
- back-projection
- point cloud basics

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **point cloud reconstruction pipeline**:

```text
DepthMap
→ BackProjection
→ PointFiltering
→ PointStatistics
→ PointCloudExport
→ Report
```

## 2. Python BST
Lưu `pair_id`, `rig_id`, `cloud_id`.

## 3. `std::vector<DepthMapSample>`
Danh sách depth samples.

## 4. `std::vector<CameraRigConfig>`
Danh sách camera rigs.

## 5. `std::vector<Point3D>`
Danh sách điểm 3D.

## 6. `std::vector<PointCloudResult>`
Kết quả point cloud theo từng depth sample.

## 7. `std::stack<PointCloudDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Back-projection
Với pixel `(u, v)` và depth `Z`, dùng:

```text
X = (u - cx) * Z / fx
Y = (v - cy) * Z / fy
Z = depth(u, v)
```

để dựng điểm 3D trong **camera frame**.

---

## Algorithm 2 — Invalid point rejection
Không dựng point nếu:
- depth invalid
- depth <= 0
- depth vượt max depth clip
- hoặc point không hợp lệ

---

## Algorithm 3 — Point filtering
Lọc ít nhất theo:
- min depth
- max depth
- optional X/Y range

---

## Algorithm 4 — Point cloud statistics
Tính ít nhất:
- point count
- min / max / average Z
- min / max X
- min / max Y

---

## Algorithm 5 — Visualization / Export
Bắt buộc có ít nhất 1 dạng đầu ra dễ quan sát, ví dụ:
- export `.xyz`
- export `.txt`
- plot depth-to-point-count summary
- plot histogram của Z

---

# 9. Cấu trúc folder

```text
mini_project_54_stereo_point_cloud_reconstruction_lab/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ depth_inputs/
│  │  ├─ depth_bm_pair_01.png
│  │  ├─ depth_sgm_pair_01.png
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ point_cloud_statistics_report.txt
│     ├─ point_cloud_runtime_report.txt
│     ├─ point_cloud_quality_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ cloud_pair_01_bm.xyz
│     ├─ cloud_pair_01_sgm.xyz
│     └─ z_distribution_plot.png
│
├─ config/
│  ├─ camera_rig_config.txt
│  ├─ depth_sample_config.txt
│  ├─ point_filter_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ point_cloud_graph_preview.py
│     ├─ point_cloud_bst.py
│     ├─ numpy_back_projection_preview.py
│     ├─ matplotlib_point_cloud_plotter.py
│     └─ synthetic_depth_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ DepthMapSample.hpp
   │  ├─ CameraRigConfig.hpp
   │  ├─ PointFilterConfig.hpp
   │  ├─ Point3D.hpp
   │  ├─ PointCloudResult.hpp
   │  ├─ PointCloudDebugRecord.hpp
   │  ├─ BaseBackProjectionEngine.hpp
   │  ├─ BackProjectionEngine.hpp
   │  ├─ BasePointFilterEngine.hpp
   │  ├─ PointFilterEngine.hpp
   │  ├─ BasePointCloudStatisticsEngine.hpp
   │  ├─ PointCloudStatisticsEngine.hpp
   │  ├─ StereoPointCloudReconstructionLab.hpp
   │  └─ PointCloudReportWriter.hpp
   │
   └─ src/
      ├─ BackProjectionEngine.cpp
      ├─ PointFilterEngine.cpp
      ├─ PointCloudStatisticsEngine.cpp
      ├─ StereoPointCloudReconstructionLab.cpp
      └─ PointCloudReportWriter.cpp
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
rig_config_path
depth_sample_config_path
filter_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoPointCloudReconstructionConfigBuilder`

Tạo class con:

```python
class StereoPointCloudReconstructionConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_camera_rig(
    rig_id,
    fx,
    fy,
    cx,
    cy,
    image_width,
    image_height
)`

### `add_depth_sample(
    pair_id,
    algorithm_type,
    depth_map_path,
    scene_label=None
)`

### `set_point_filter_rules(
    min_depth,
    max_depth,
    min_x=None,
    max_x=None,
    min_y=None,
    max_y=None
)`

### `set_visualization_options(
    enable_z_histogram,
    enable_cloud_export_summary
)`

### `write_rig_config()`
### `write_depth_sample_config()`
### `write_filter_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `point_cloud_graph_preview.py`

Tạo class:

```python
class PointCloudGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
DepthMap
→ BackProjection
→ PointFiltering
→ PointStatistics
→ PointCloudExport
→ Report
```

---

# 10.4 Python — `point_cloud_bst.py`

Tạo BST cho `pair_id` / `cloud_id`.

---

# 10.5 Python — `numpy_back_projection_preview.py`

Tạo class:

```python
class NumPyBackProjectionPreview:
```

## Hàm cần có

### `depth_to_point_cloud(depth_map, fx, fy, cx, cy, invalid_marker=-1.0)`
### `filter_points(points_xyz, min_depth, max_depth)`
### `compute_point_statistics(points_xyz)`
### `extract_z_values(points_xyz)`

---

# 10.6 Python — `matplotlib_point_cloud_plotter.py`

Tạo class:

```python
class MatplotlibPointCloudPlotter:
```

## Hàm cần có

### `plot_z_histogram(z_values, save_path)`
### `plot_point_count_summary(summary_data, save_path)`

---

# 10.7 C++ — `DepthMapSample`

```cpp
enum class DepthSourceType
{
    BM,
    SGM
};

struct DepthMapSample
{
    std::string pair_id;
    DepthSourceType algorithm_type;
    std::string depth_map_path;
    std::string scene_label;
};
```

---

# 10.8 C++ — `CameraRigConfig`

```cpp
struct CameraRigConfig
{
    std::string rig_id;
    double fx;
    double fy;
    double cx;
    double cy;
    int image_width;
    int image_height;
};
```

---

# 10.9 C++ — `PointFilterConfig`

```cpp
struct PointFilterConfig
{
    double min_depth;
    double max_depth;

    bool use_x_filter = false;
    double min_x = 0.0;
    double max_x = 0.0;

    bool use_y_filter = false;
    double min_y = 0.0;
    double max_y = 0.0;
};
```

---

# 10.10 C++ — `Point3D`

```cpp
struct Point3D
{
    double x;
    double y;
    double z;
};
```

---

# 10.11 C++ — `PointCloudResult`

```cpp
struct PointCloudResult
{
    std::string pair_id;
    DepthSourceType algorithm_type;

    std::vector<Point3D> points;

    int point_count;
    double min_x;
    double max_x;
    double min_y;
    double max_y;
    double min_z;
    double max_z;
    double average_z;
};
```

---

# 10.12 C++ — `PointCloudDebugRecord`

```cpp
struct PointCloudDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.13 C++ — `BaseBackProjectionEngine`

Tạo abstract class:

```cpp
class BaseBackProjectionEngine
{
public:
    virtual std::vector<Point3D> reconstruct(
        const cv::Mat& depth_map,
        const CameraRigConfig& rig
    ) const = 0;

    virtual ~BaseBackProjectionEngine() = default;
};
```

---

# 10.14 C++ — `BackProjectionEngine`

Kế thừa `BaseBackProjectionEngine`.

## Nhiệm vụ
- back-project depth map thành vector điểm 3D

---

# 10.15 C++ — `BasePointFilterEngine`

Tạo abstract class:

```cpp
class BasePointFilterEngine
{
public:
    virtual std::vector<Point3D> filter(
        const std::vector<Point3D>& points,
        const PointFilterConfig& filter_config
    ) const = 0;

    virtual ~BasePointFilterEngine() = default;
};
```

---

# 10.16 C++ — `PointFilterEngine`

Kế thừa `BasePointFilterEngine`.

## Nhiệm vụ
- lọc điểm theo min/max depth
- optional lọc theo X/Y range

---

# 10.17 C++ — `BasePointCloudStatisticsEngine`

Tạo abstract class:

```cpp
class BasePointCloudStatisticsEngine
{
public:
    virtual void compute_statistics(PointCloudResult& result) const = 0;
    virtual ~BasePointCloudStatisticsEngine() = default;
};
```

---

# 10.18 C++ — `PointCloudStatisticsEngine`

Kế thừa `BasePointCloudStatisticsEngine`.

## Nhiệm vụ
- tính point count
- min/max/average XYZ

---

# 10.19 C++ — `StereoPointCloudReconstructionLab`

Tạo class trung tâm:

```cpp
class StereoPointCloudReconstructionLab
```

## Thuộc tính

```cpp
private:
    std::vector<CameraRigConfig> rigs;
    std::vector<DepthMapSample> depth_samples;
    PointFilterConfig filter_config;

    std::shared_ptr<BaseBackProjectionEngine> back_projection_engine;
    std::shared_ptr<BasePointFilterEngine> filter_engine;
    std::shared_ptr<BasePointCloudStatisticsEngine> statistics_engine;

    std::vector<PointCloudResult> results;
    std::stack<PointCloudDebugRecord> debug_history;
```

## Hàm cần có

### `load_rig_config(const std::string& path)`
### `load_depth_sample_config(const std::string& path)`
### `load_filter_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each depth sample:
    load depth map
    find matching rig
    points = back_projection_engine.reconstruct(depth_map, rig)
    filtered_points = filter_engine.filter(points, filter_config)

    build PointCloudResult
    statistics_engine.compute_statistics(result)

    save result
    push debug history
```

### `const std::vector<PointCloudResult>& get_results() const`
### `std::vector<PointCloudDebugRecord> get_debug_history_reverse()`

---

# 10.20 C++ — `PointCloudReportWriter`

Tạo class:

```cpp
class PointCloudReportWriter
```

## Hàm cần có

### `write_point_cloud_statistics_report(...)`
Ví dụ:

```text
[Point Cloud Statistics]
Pair: hallway_01
Algorithm: BM
Point Count: 18234
X Range: [-1.2, 1.1]
Y Range: [-0.7, 0.9]
Z Range: [0.5, 4.8]
Average Z: 2.31
```

### `write_point_cloud_runtime_report(...)`
Ví dụ:

```text
[Point Cloud Runtime]
Pair: hallway_01
Point cloud reconstruction completed
```

### `write_point_cloud_quality_report(...)`
Ví dụ:

```text
[Point Cloud Quality]
Pair: hallway_01
Valid Reconstructed Points: 18234
Depth Range Used: [0.3, 5.0]
```

### `write_xyz_file(const PointCloudResult& result, const std::string& save_path)`

### `write_reverse_debug_history(...)`

---

# 10.21 C++ — `main.cpp`

## Yêu cầu

```text
Load camera rig config
Load depth sample config
Load point filter config
Load visualization config
Load runtime config

Create:
    BackProjectionEngine
    PointFilterEngine
    PointCloudStatisticsEngine

Create StereoPointCloudReconstructionLab
Run
Write reports
Export point clouds
```

---

# 11. Luật point cloud reconstruction của project

## Luật 1 — Không được back-project pixel invalid
Depth invalid phải bị loại trước hoặc trong lúc reconstruct.

## Luật 2 — Point cloud phải nằm trong camera frame
Ở bài này chưa transform sang robot frame.

## Luật 3 — Point filtering là bắt buộc
Ít nhất phải lọc theo min/max depth để tránh cloud quá bẩn.

## Luật 4 — Report phải có XYZ statistics
Không được chỉ export point cloud rồi dừng.

## Luật 5 — Output phải đủ để dùng cho bài sau
Tức là point cloud result cần sẵn sàng cho:
- camera-to-robot transform
- obstacle reasoning
- corridor analysis

---

# 12. Output mong muốn

## Config
```text
config/camera_rig_config.txt
config/depth_sample_config.txt
config/point_filter_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/point_cloud_statistics_report.txt
assets/outputs/point_cloud_runtime_report.txt
assets/outputs/point_cloud_quality_report.txt
assets/outputs/reverse_debug_history.txt
```

## Point cloud exports
```text
assets/outputs/cloud_pair_01_bm.xyz
assets/outputs/cloud_pair_01_sgm.xyz
```

## Plots
```text
assets/outputs/z_distribution_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho rig / depth sample / point filter
- preview back-projection bằng NumPy
- visualize Z distribution / point statistics

## C++
- chạy point cloud reconstruction runtime
- lọc cloud
- tính statistics
- export cloud cho các bước 3D phía sau

## Computer Vision / Robot Perception
Đây là project cực quan trọng vì nó là **bước vào 3D point-based perception**:

```text
Stereo pair
→ disparity
→ depth
→ depth quality
→ point cloud
→ camera-to-robot transform
→ obstacle reasoning
```

Nếu bạn nắm chắc Bài 54, thì các bài như:
- camera-frame point cloud analysis
- robot-frame point cloud transform
- obstacle reasoning
- free-space corridor estimation

sẽ nối vào rất tự nhiên.

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho rig / depth samples / filter / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / cloud ids
- [ ] Python có NumPy back-projection preview
- [ ] Python có Matplotlib Z distribution plot
- [ ] C++ có `DepthMapSample`
- [ ] C++ có `CameraRigConfig`
- [ ] C++ có `PointFilterConfig`
- [ ] C++ có `Point3D`
- [ ] C++ có `PointCloudResult`
- [ ] C++ có `BackProjectionEngine`
- [ ] C++ có `PointFilterEngine`
- [ ] C++ có `PointCloudStatisticsEngine`
- [ ] C++ có `StereoPointCloudReconstructionLab`
- [ ] C++ ghi đủ report + export point cloud files

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 54**, bước hợp lý nhất nếu bám tiếp mạch 3D perception là:

# **Bài 55: Camera-Frame Point Cloud Analyzer**

Ý tưởng:
```text
point cloud
→ region partition
→ density analysis
→ nearest-point / farthest-point stats
→ local cluster summary
→ camera-frame obstacle report
```

Tức là mạch sẽ đi:

```text
Bài 51: Stereo disparity
→ Bài 52: Depth estimation
→ Bài 53: Depth quality analysis
→ Bài 54: Point cloud reconstruction
→ Bài 55: Camera-frame point cloud analysis
```
