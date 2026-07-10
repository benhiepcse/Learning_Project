# 🤖 Bài 48: Stereo Point Cloud Mini Pipeline — Bộ dựng point cloud mini từ depth map cho Humanoid Robot AI Perception

> Mini Project số 48 trong **Đợt 10**  
> **Bài 48 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10** và đi tiếp trực tiếp từ **Bài 47**.
>
> Nếu:
>
> - **Bài 41 → 45** đi từ calibration, rectification, correspondence, block matching
> - **Bài 46** chuyển sang stereo calibration analyzer
> - **Bài 47** đánh giá disparity / depth output ở mức strip / mini map
>
> thì **Bài 48** sẽ là bước nối cực quan trọng từ **depth** sang **3D perception**:
>
> ```text
> disparity / depth map
> → back-project sang 3D points
> → tạo mini point cloud
> → thống kê vùng không gian
> → đánh giá chất lượng point cloud
> ```
>
> Đây là một bước rất sát với công việc **Robot Perception Engineer**, vì point cloud là dạng dữ liệu mà robot thường dùng để:
>
> - hiểu khoảng cách vật thể trong không gian
> - ước lượng mặt phẳng sàn / vật cản
> - dựng local 3D geometry
> - chuẩn bị cho obstacle reasoning, mapping, manipulation, navigation

---

# 📌 Mục lục

- [1. Vì sao Bài 48 xuất hiện sau Bài 47](#1-vì-sao-bài-48-xuất-hiện-sau-bài-47)
- [2. Đợt 10 đang học gì](#2-đợt-10-đang-học-gì)
- [3. Bài 48 nâng từ Bài 47 lên chỗ nào](#3-bài-48-nâng-từ-bài-47-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật point cloud của project](#11-luật-point-cloud-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 48 xuất hiện sau Bài 47

## Bài 47 đang làm gì?
Bài 47 đã giúp bạn có:

```text
disparity strip / disparity map
→ depth map
→ valid / invalid mask
→ depth statistics
```

Tức là bạn đã có **depth usable** ở mức 1D / 2D mini.

## Nhưng còn thiếu gì?
Trong perception thật, depth map thường không phải đích cuối.  
Robot thường cần đi thêm một bước:

> Từ depth map, **hãy dựng ra các điểm 3D trong camera frame**.

Lúc đó robot mới bắt đầu trả lời được:
- vật thể đang nằm ở đâu trong không gian?
- mặt đất / vật cản nằm ở vùng nào?
- khoảng cách 3D thực sự tới vật là bao nhiêu?

## Bài 48 lấp đúng chỗ đó
Bài 48 sẽ làm:

```text
depth map
→ back-projection
→ point cloud mini
→ thống kê point ranges / bounds / density
→ quality report
```

Đây là bước chuyển từ **2D depth representation** sang **3D geometric representation**.

---

# 2. Đợt 10 đang học gì

Theo roadmap bạn gửi, **Đợt 10** xoay quanh:

## Python
### **Phase 9 — NumPy**
- reshape
- matrix multiplication
- robotics vector basics

## C++
- class
- inheritance
- virtual function
- vector
- pointer / reference

## Computer Vision
### **Phase 5 — Stereo Vision & Depth**
- Stereo Camera
- Disparity

## Vì sao Bài 48 vẫn bám đúng Đợt 10?
Vì Bài 48 dùng đúng tinh thần:
- stereo → disparity → depth
- depth → 3D geometry
- NumPy để dựng / reshape point arrays
- C++ OOP để xây point cloud pipeline
- robot vector basics cho `(X, Y, Z)`

---

# 3. Bài 48 nâng từ Bài 47 lên chỗ nào

## Bài 47
- disparity → depth
- valid mask
- depth statistics
- depth quality evaluation

## Bài 48
- depth → **3D point**
- nhiều điểm → **point cloud**
- point cloud → **không gian 3D có thể phân tích được**

### Nói ngắn gọn:
- **Bài 47** hỏi: “Depth output có ổn không?”
- **Bài 48** hỏi: “Từ depth đó, robot nhìn thấy gì trong không gian 3D?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Point Cloud Mini Pipeline**

System này nhận đầu vào là:
- một hoặc nhiều **stereo rig config**
- một hoặc nhiều **depth strip / depth map**
- một số **point cloud filtering / analysis options**

## Mỗi depth input có thể là:
### Dạng 1 — 1D depth strip
Ví dụ:
```text
[2.4, 2.3, 2.1, -1, -1, 1.8, 1.7]
```

### Dạng 2 — 2D depth map mini
Ví dụ:
```text
[
  [2.4, 2.3, 2.1, 2.0],
  [2.5, 2.4, 2.2, 2.1],
  [-1,  -1,  1.8, 1.7]
]
```

Trong đó:
- `-1` hoặc một marker khác có thể coi là invalid depth

## Nhiệm vụ của system
### Với mỗi rig + depth input:
1. duyệt qua các pixel depth hợp lệ
2. dùng intrinsics để **back-project** thành điểm 3D:
   - `(u, v, Z) → (X, Y, Z)`
3. tạo **mini point cloud**
4. tính thống kê:
   - số điểm hợp lệ
   - min/max theo X, Y, Z
   - average distance
5. optional:
   - lọc điểm theo range
   - loại bỏ điểm quá gần / quá xa
6. ghi point cloud report

<p align="center">
  <img src="images/project_48.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build stereo rig config
→ build depth strip / map config
→ preview back-projection bằng NumPy
→ tạo point array
→ preview XYZ ranges

C++
→ load stereo rig
→ load depth strips / maps
→ back-project depth thành 3D points
→ build point cloud result
→ compute point statistics
→ export point cloud reports
```

Mục tiêu cốt lõi:
- hiểu **depth map không phải đích cuối**, mà là đầu vào để dựng **3D points**
- hiểu công thức back-projection bằng intrinsics
- hiểu point cloud là cầu nối giữa stereo perception và robot 3D reasoning
- chuẩn bị nền cho:
  - obstacle distance reasoning
  - ground plane analysis
  - robot-centric mapping
  - manipulation perception

---

# 6. Pipeline tổng thể

```text
Load Stereo Rig Config
Load Depth Map Config
Load Point Cloud Runtime Config
Load Filtering Config

Create BackProjectionEngine
Create PointCloudFilteringEngine
Create PointCloudStatisticsEngine
Create StereoPointCloudPipeline

For each stereo rig:
    for each depth strip / map:
        1. convert valid depth pixels to 3D points
        2. build point cloud
        3. filter points by rule if needed
        4. compute point statistics
        5. save result

After all inputs:
    write reports
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- BST / Graph / NumPy từ các đợt trước
- reshape
- vectorized coordinate generation
- robotics vector basics `(X, Y, Z)`

## C++
- class / inheritance
- virtual functions / polymorphism
- vector
- pointer / reference
- report-oriented runtime design

## Computer Vision / Geometry
- stereo depth
- pinhole camera intrinsics
- back-projection
- camera-frame 3D points
- point cloud basics

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **point cloud pipeline**:

```text
DepthMap
→ ValidDepthPixels
→ BackProjection
→ PointCloud
→ PointFiltering
→ PointStatistics
→ Report
```

## 2. Python BST
Lưu `rig_id`, `map_id`, `cloud_id`.

## 3. `std::vector<StereoRigConfig>`
Danh sách rig stereo.

## 4. `std::vector<DepthMapSample>`
Danh sách depth strip / map.

## 5. `std::vector<Point3D>`
Danh sách điểm 3D.

## 6. `std::vector<PointCloudResult>`
Kết quả point cloud theo từng rig + map.

## 7. `std::stack<PointCloudDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Back-projection
Với pixel `(u, v)` và depth `Z`, tính:

```text
X = (u - cx) * Z / fx
Y = (v - cy) * Z / fy
Z = depth
```

Nếu input là strip 1D, bạn có thể quy ước:
- `v = row_index cố định`
- hoặc map strip thành một hàng ảnh nhỏ

---

## Algorithm 2 — Valid depth filtering
Chỉ back-project những pixel có depth hợp lệ:
- depth > 0
- depth khác invalid marker

---

## Algorithm 3 — Point cloud statistics
Tính ít nhất:
- `valid_point_count`
- `min_x`, `max_x`
- `min_y`, `max_y`
- `min_z`, `max_z`
- `average_distance`

---

## Algorithm 4 — Point filtering
Hỗ trợ ít nhất **1 rule lọc**:
- bỏ điểm có `Z > max_depth`
- bỏ điểm có `Z < min_depth`
- hoặc bỏ điểm có `|X|` vượt ngưỡng

---

## Algorithm 5 — Region / slice analysis
Nếu input là 2D map, hãy thống kê:
- số điểm theo từng hàng
- hoặc theo từng depth slice / vùng trái-phải

---

# 9. Cấu trúc folder

```text
mini_project_48_stereo_point_cloud_mini_pipeline/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ point_cloud_report.txt
│     ├─ point_statistics_report.txt
│     ├─ point_filter_report.txt
│     ├─ point_region_summary_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_rig_config.txt
│  ├─ depth_map_config.txt
│  ├─ point_cloud_runtime_config.txt
│  └─ point_filter_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ point_cloud_graph_preview.py
│     ├─ rig_bst.py
│     ├─ numpy_point_cloud_preview.py
│     └─ synthetic_depth_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoRigConfig.hpp
   │  ├─ DepthMapSample.hpp
   │  ├─ Point3D.hpp
   │  ├─ PointCloudResult.hpp
   │  ├─ PointRegionStatistic.hpp
   │  ├─ PointCloudDebugRecord.hpp
   │  ├─ BaseBackProjectionEngine.hpp
   │  ├─ StereoBackProjectionEngine.hpp
   │  ├─ BasePointCloudFilterEngine.hpp
   │  ├─ PointCloudFilterEngine.hpp
   │  ├─ BasePointStatisticsEngine.hpp
   │  ├─ PointStatisticsEngine.hpp
   │  ├─ StereoPointCloudPipeline.hpp
   │  └─ StereoPointCloudReportWriter.hpp
   │
   └─ src/
      ├─ StereoBackProjectionEngine.cpp
      ├─ PointCloudFilterEngine.cpp
      ├─ PointStatisticsEngine.cpp
      ├─ StereoPointCloudPipeline.cpp
      └─ StereoPointCloudReportWriter.cpp
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
depth_map_path
runtime_config_path
filter_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoPointCloudConfigBuilder`

Tạo class con:

```python
class StereoPointCloudConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_stereo_rig(
    rig_id,
    fx,
    fy,
    cx,
    cy,
    baseline,
    image_width,
    image_height
)`

### `add_depth_strip(
    map_id,
    depth_values,
    target_label=None
)`

### `add_depth_grid(
    map_id,
    depth_grid,
    target_label=None
)`

### `set_point_cloud_options(
    invalid_depth_value,
    min_depth_keep,
    max_depth_keep,
    enable_region_summary
)`

### `write_rig_config()`
### `write_depth_map_config()`
### `write_runtime_config()`
### `write_filter_config()`

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
→ ValidDepthPixels
→ BackProjection
→ PointCloud
→ PointFiltering
→ PointStatistics
→ Report
```

---

# 10.4 Python — `rig_bst.py`

Tạo BST cho `rig_id` / `map_id`.

---

# 10.5 Python — `numpy_point_cloud_preview.py`

Tạo class:

```python
class NumPyPointCloudPreview:
```

## Hàm cần có

### `back_project_strip(depth_values, fx, fy, cx, cy, row_index)`
### `back_project_grid(depth_grid, fx, fy, cx, cy)`
### `compute_xyz_statistics(points_xyz)`
### `filter_points_by_depth(points_xyz, min_depth, max_depth)`

---

# 10.6 C++ — `DepthMapSample`

```cpp
struct DepthMapSample
{
    std::string map_id;
    std::string target_label;

    bool is_grid;

    std::vector<double> depth_strip;
    std::vector<std::vector<double>> depth_grid;
};
```

---

# 10.7 C++ — `Point3D`

```cpp
struct Point3D
{
    double x;
    double y;
    double z;
    int u;
    int v;
};
```

---

# 10.8 C++ — `PointCloudResult`

```cpp
struct PointCloudResult
{
    std::string rig_id;
    std::string map_id;

    std::vector<Point3D> points;

    int valid_point_count;
    double min_x;
    double max_x;
    double min_y;
    double max_y;
    double min_z;
    double max_z;
    double average_distance;
};
```

---

# 10.9 C++ — `PointRegionStatistic`

```cpp
struct PointRegionStatistic
{
    std::string rig_id;
    std::string map_id;
    std::string region_name;

    int point_count;
    double average_z;
    double min_z;
    double max_z;
};
```

---

# 10.10 C++ — `PointCloudDebugRecord`

```cpp
struct PointCloudDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.11 C++ — `BaseBackProjectionEngine`

Tạo abstract class:

```cpp
class BaseBackProjectionEngine
{
public:
    virtual std::vector<Point3D> back_project(
        const StereoRigConfig& rig,
        const DepthMapSample& sample
    ) const = 0;

    virtual ~BaseBackProjectionEngine() = default;
};
```

---

# 10.12 C++ — `StereoBackProjectionEngine`

Kế thừa `BaseBackProjectionEngine`.

## Nhiệm vụ
- duyệt toàn bộ depth strip / grid
- back-project các depth hợp lệ thành `Point3D`

---

# 10.13 C++ — `BasePointCloudFilterEngine`

Tạo abstract class:

```cpp
class BasePointCloudFilterEngine
{
public:
    virtual void filter_points(
        std::vector<Point3D>& points,
        double min_depth_keep,
        double max_depth_keep
    ) const = 0;

    virtual ~BasePointCloudFilterEngine() = default;
};
```

---

# 10.14 C++ — `PointCloudFilterEngine`

Kế thừa `BasePointCloudFilterEngine`.

## Nhiệm vụ
- bỏ điểm quá gần / quá xa theo `z`
- có thể mở rộng thêm rule theo `x`, `y`

---

# 10.15 C++ — `BasePointStatisticsEngine`

Tạo abstract class:

```cpp
class BasePointStatisticsEngine
{
public:
    virtual PointCloudResult compute_statistics(
        const std::string& rig_id,
        const std::string& map_id,
        const std::vector<Point3D>& points
    ) const = 0;

    virtual std::vector<PointRegionStatistic> compute_region_statistics(
        const std::string& rig_id,
        const std::string& map_id,
        const std::vector<Point3D>& points
    ) const = 0;

    virtual ~BasePointStatisticsEngine() = default;
};
```

---

# 10.16 C++ — `PointStatisticsEngine`

Kế thừa `BasePointStatisticsEngine`.

## Nhiệm vụ
- tính global point statistics
- tính region / row summary

---

# 10.17 C++ — `StereoPointCloudPipeline`

Tạo class trung tâm:

```cpp
class StereoPointCloudPipeline
```

## Thuộc tính

```cpp
private:
    std::vector<StereoRigConfig> rigs;
    std::vector<DepthMapSample> maps;

    std::shared_ptr<BaseBackProjectionEngine> back_projection_engine;
    std::shared_ptr<BasePointCloudFilterEngine> filter_engine;
    std::shared_ptr<BasePointStatisticsEngine> statistics_engine;

    std::vector<PointCloudResult> results;
    std::vector<PointRegionStatistic> region_statistics;
    std::stack<PointCloudDebugRecord> debug_history;
```

## Hàm cần có

### `load_rig_config(const std::string& path)`
### `load_depth_map_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each rig:
    for each depth map:
        points = back_projection_engine.back_project(rig, map)
        filter_engine.filter_points(points, min_depth_keep, max_depth_keep)

        result = statistics_engine.compute_statistics(rig.rig_id, map.map_id, points)
        regions = statistics_engine.compute_region_statistics(rig.rig_id, map.map_id, points)

        save result
        save regions
        push debug history
```

### `const std::vector<PointCloudResult>& get_results() const`
### `const std::vector<PointRegionStatistic>& get_region_statistics() const`
### `std::vector<PointCloudDebugRecord> get_debug_history_reverse()`

---

# 10.18 C++ — `StereoPointCloudReportWriter`

Tạo class:

```cpp
class StereoPointCloudReportWriter
```

## Hàm cần có

### `write_point_cloud_report(...)`
Ví dụ:

```text
[Point Cloud]
Rig: stereo_head_v1
Map: grid_01
Valid Points: 42
```

### `write_point_statistics_report(...)`
Ví dụ:

```text
[Point Statistics]
Rig: stereo_head_v1
Map: grid_01
X range: -0.42 -> 0.37
Y range: -0.18 -> 0.22
Z range: 0.80 -> 3.10
Average distance: 1.92
```

### `write_point_filter_report(...)`
Ví dụ:

```text
[Point Filter]
Rig: stereo_head_v1
Map: grid_01
Keep depth range: 0.30 -> 3.00
```

### `write_point_region_summary_report(...)`
Ví dụ:

```text
[Point Region Summary]
Rig: stereo_head_v1
Map: grid_01
Region: row_0
Point Count: 12
Average Z: 1.82
```

### `write_reverse_debug_history(...)`

---

# 10.19 C++ — `main.cpp`

## Yêu cầu

```text
Load stereo rig config
Load depth map config
Load point cloud runtime config
Load point filter config

Create:
    StereoBackProjectionEngine
    PointCloudFilterEngine
    PointStatisticsEngine

Create StereoPointCloudPipeline
Run
Write reports
```

---

# 11. Luật point cloud của project

## Luật 1 — Chỉ back-project depth hợp lệ
Nếu depth `<= 0` hoặc bằng invalid marker thì không được tạo `Point3D`.

## Luật 2 — Công thức back-projection phải dùng đúng intrinsics
Phải bám theo:
```text
X = (u - cx) * Z / fx
Y = (v - cy) * Z / fy
Z = depth
```

## Luật 3 — Point statistics chỉ tính trên điểm sau filtering
Nếu đã lọc điểm quá xa / quá gần thì report phải phản ánh cloud sau lọc.

## Luật 4 — Phải hỗ trợ cả strip và grid
Để nối mạch từ disparity strip / depth strip sang mini point cloud 2D.

## Luật 5 — Region statistics phải có ít nhất một kiểu chia vùng rõ ràng
Ví dụ:
- theo hàng
- theo nửa trái / nửa phải
- theo depth slices

---

# 12. Output mong muốn

## Config
```text
config/stereo_rig_config.txt
config/depth_map_config.txt
config/point_cloud_runtime_config.txt
config/point_filter_config.txt
```

## Reports
```text
assets/outputs/point_cloud_report.txt
assets/outputs/point_statistics_report.txt
assets/outputs/point_filter_report.txt
assets/outputs/point_region_summary_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build depth strip / map config
- preview back-projection bằng NumPy
- kiểm tra XYZ ranges trước khi viết runtime C++

## C++
- back-project depth thành point cloud
- lọc điểm
- tính thống kê cloud
- chuẩn bị đầu ra cho robot-centric reasoning

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là lần đầu bạn thực sự chuyển từ:
- **depth**
sang
- **3D geometry**

Bài 48 là cầu nối cực đẹp trước các bài:
- ground / obstacle analysis
- camera-to-robot transform
- local 3D scene understanding

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho rig / depth map / filter
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho rig ids / map ids
- [ ] Python có NumPy preview cho back-projection / XYZ ranges
- [ ] C++ có `DepthMapSample`
- [ ] C++ có `Point3D`
- [ ] C++ có `PointCloudResult`
- [ ] C++ có `PointRegionStatistic`
- [ ] C++ có `StereoBackProjectionEngine`
- [ ] C++ có `PointCloudFilterEngine`
- [ ] C++ có `PointStatisticsEngine`
- [ ] C++ có `StereoPointCloudPipeline`
- [ ] C++ ghi đủ 5 report

---

# 15. Gợi ý mở rộng

## 1. Thêm camera-frame → robot-frame transform
Đây là bước rất tự nhiên sau point cloud camera frame.

## 2. Thêm ground filtering đơn giản
Ví dụ loại bỏ các điểm gần mặt sàn theo ngưỡng `Y` hoặc `Z` tùy frame quy ước.

## 3. Thêm obstacle distance summary
Ví dụ tìm điểm gần robot nhất trong vùng trung tâm.

## 4. Chuẩn bị cho Bài 49
Sau Bài 48, hướng đi rất đẹp là:

# **Bài 49: Camera-to-Robot Point Cloud Transformer**

Ý tưởng:
```text
camera-frame point cloud
→ extrinsic transform
→ robot-frame point cloud
→ robot-centric region analysis
→ obstacle distance report
```

Tức là:
- **Bài 47**: depth map evaluation
- **Bài 48**: point cloud trong camera frame
- **Bài 49**: đưa point cloud sang robot frame để robot thực sự dùng
