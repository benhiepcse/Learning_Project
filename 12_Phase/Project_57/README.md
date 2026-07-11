# 🤖 Bài 57: Robot-Frame Obstacle Distance Analyzer — Bộ phân tích khoảng cách vật cản trong robot frame cho Humanoid Robot AI Perception

> Mini Project số 57 trong **Đợt 12**  
> **Bài 57 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12** và đi tiếp trực tiếp từ **Bài 56**.
>
> Nếu:
>
> - **Bài 54** giúp bạn dựng **point cloud từ depth**
> - **Bài 55** giúp bạn phân tích **point cloud trong camera frame**
> - **Bài 56** giúp bạn **transform point cloud sang robot frame**
>
> thì **Bài 57** là bước tiếp theo rất quan trọng:
>
> ```text
> robot-frame point cloud
> → front danger zone analysis
> → center corridor distance analysis
> → left / right clearance estimation
> → obstacle danger scoring
> → locomotion safety report
> ```
>
> Đây là bước mà perception không chỉ “nhìn thấy point cloud trong robot frame” nữa, mà bắt đầu trả lời:
>
> - **robot có đang bị chặn phía trước không?**
> - **vùng đi thẳng phía trước còn trống bao nhiêu?**
> - **trái / phải bên nào thoáng hơn?**
> - **mức nguy hiểm của vật cản hiện tại là gì?**
>
> Tức là dữ liệu bắt đầu usable cho:
> - locomotion safety
> - free-space reasoning
> - local obstacle avoidance
> - robot-centric navigation perception

---

# 📌 Mục lục

- [1. Đợt 12 đang đi tiếp từ Bài 56 ra sao](#1-đợt-12-đang-đi-tiếp-từ-bài-56-ra-sao)
- [2. Vì sao Bài 57 xuất hiện sau Bài 56](#2-vì-sao-bài-57-xuất-hiện-sau-bài-56)
- [3. Bài 57 nâng từ Bài 56 lên chỗ nào](#3-bài-57-nâng-từ-bài-56-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật obstacle distance analysis của project](#11-luật-obstacle-distance-analysis-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 12 đang đi tiếp từ Bài 56 ra sao

Ở **Bài 56**, bạn đã có:

```text
camera-frame point cloud
→ camera-to-robot transform
→ robot-frame point cloud
→ robot-centric region analysis
```

Nghĩa là point cloud đã được gắn vào **hệ tọa độ của robot**.

Nhưng với robot humanoid, chỉ biết “có điểm ở robot frame” vẫn chưa đủ.  
Robot cần biết cụ thể hơn:

- khoảng cách gần nhất phía trước là bao nhiêu?
- corridor ở giữa có đang bị chặn không?
- bên trái và bên phải bên nào thoáng hơn để có hướng tránh?
- có vật cản lọt vào vùng nguy hiểm sát robot không?

Đó là lý do **Bài 57** xuất hiện.

---

# 2. Vì sao Bài 57 xuất hiện sau Bài 56

## Bài 56 đang làm gì?
Bài 56 giúp bạn có:
- **robot-frame point cloud**
- **robot-centric region statistics**
- **robot obstacle summary cơ bản**

## Nhưng còn thiếu gì?
Bạn vẫn chưa có một lớp phân tích chuyên sâu cho **distance safety** — tức là:

- **front danger zone** còn bao nhiêu khoảng trống?
- **center corridor** có đủ rộng để tiến thẳng không?
- **left clearance** và **right clearance** bên nào tốt hơn?
- cloud hiện tại thuộc mức **SAFE / WARNING / DANGER**?

## Bài 57 lấp đúng chỗ đó
Bài 57 sẽ làm:

```text
robot-frame point cloud
→ front-zone partition
→ center-corridor distance analysis
→ left/right clearance estimation
→ obstacle danger scoring
→ locomotion safety report
```

Đây là bước chuyển từ **robot-frame geometry** sang **robot-frame safety reasoning**.

---

# 3. Bài 57 nâng từ Bài 56 lên chỗ nào

## Bài 56
- transform cloud sang robot frame
- chia vùng cơ bản trong robot frame
- tóm tắt obstacle layout tổng quát

## Bài 57
- phân tích **front danger zone**
- phân tích **center corridor**
- ước lượng **clearance trái / phải**
- tính **obstacle danger score**
- gán **safety label**: `SAFE / WARNING / DANGER`

### Nói ngắn gọn:
- **Bài 56** hỏi: “Point cloud trong robot frame trông như thế nào?”
- **Bài 57** hỏi: “Robot có thể tiến lên an toàn hay không?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Robot-Frame Obstacle Distance Analyzer**

System này nhận đầu vào là:
- một hoặc nhiều **robot-frame point cloud samples**
- một bộ **danger zone / corridor configs**
- một bộ **clearance thresholds**
- một bộ **scoring / reporting rules**

## Mỗi point cloud sample tối thiểu chứa
- `pair_id`
- `algorithm_type`
- `robot_cloud_path`
- optional `scene_label`

## Nhiệm vụ của system
### Với mỗi robot-frame point cloud:
1. load point cloud
2. chia cloud thành:
   - **front danger zone**
   - **front warning zone**
   - **left corridor**
   - **center corridor**
   - **right corridor**
3. tính:
   - nearest point phía trước robot
   - average distance của center corridor
   - left clearance
   - right clearance
   - occupancy ratio vùng nguy hiểm
4. tính **danger score**
5. gán **safety label**
6. ghi **locomotion safety report**

<p align="center">
  <img src="../../images/project_57.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build robot-cloud config
→ build danger-zone / corridor config
→ preview corridor distance analysis bằng NumPy
→ visualize safety summary bằng Matplotlib

C++
→ load robot-frame point cloud
→ partition front / left / center / right safety zones
→ compute clearance + nearest obstacle distance
→ compute danger score + safety label
→ export locomotion safety reports
```

Mục tiêu cốt lõi:
- hiểu cách dùng **robot-frame point cloud** để suy luận **độ an toàn khi tiến lên**
- biết cách định nghĩa **danger zone / warning zone / corridor**
- biết cách tính **clearance** và **danger score**
- chuẩn bị nền trực tiếp cho:
  - free-space corridor detection
  - simple local navigation
  - walking safety perception
  - obstacle avoidance heuristics

---

# 6. Pipeline tổng thể

```text
Load Robot Cloud Sample Config
Load Danger Zone Config
Load Corridor Config
Load Safety Threshold Config
Load Visualization Config
Load Runtime Config

Create RobotCloudLoader
Create SafetyZonePartitionEngine
Create ClearanceAnalysisEngine
Create DangerScoringEngine
Create LocomotionSafetySummaryEngine
Create RobotFrameObstacleDistanceAnalyzer

For each robot cloud sample:
    1. load robot-frame point cloud
    2. partition points into safety zones
    3. compute front-nearest / center-distance / left-right clearance
    4. compute danger score
    5. assign safety label
    6. write result

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
- class / inheritance / polymorphism / virtual function
- vector
- enum
- file parsing / export
- report-oriented runtime design

## Computer Vision / 3D Geometry / Robot Perception
- point cloud
- robot frame
- danger zone / safety corridor
- clearance estimation
- obstacle danger scoring

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **obstacle distance safety pipeline**:

```text
RobotFramePointCloud
→ SafetyZonePartition
→ ClearanceAnalysis
→ DangerScoring
→ SafetySummary
→ Report
```

## 2. Python BST
Lưu `pair_id`, `robot_cloud_id`, `safety_result_id`.

## 3. `std::vector<RobotCloudSample>`
Danh sách robot-frame point cloud samples.

## 4. `std::vector<SafetyZoneStatistic>`
Thống kê từng vùng an toàn.

## 5. `std::vector<ObstacleDistanceResult>`
Kết quả phân tích khoảng cách vật cản.

## 6. `std::vector<Point3D>`
Danh sách điểm robot-frame.

## 7. `std::stack<ObstacleDistanceDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Safety zone partition
Chia point cloud thành ít nhất:
- `FRONT_DANGER`
- `FRONT_WARNING`
- `LEFT_CORRIDOR`
- `CENTER_CORRIDOR`
- `RIGHT_CORRIDOR`

Bạn có thể định nghĩa bằng:
- trục trước robot
- bề rộng corridor
- ngưỡng distance danger / warning

---

## Algorithm 2 — Clearance analysis
Tính ít nhất:
- nearest distance trong center corridor
- average distance center corridor
- left clearance
- right clearance

### Gợi ý:
- clearance có thể lấy bằng **min distance** hoặc **average safe distance** của từng corridor.

---

## Algorithm 3 — Danger occupancy analysis
Tính:
- số điểm trong `FRONT_DANGER`
- ratio điểm trong danger zone
- ratio điểm trong center corridor đang quá gần

---

## Algorithm 4 — Danger scoring
Tính `danger_score` dựa trên:
- nearest front obstacle
- center corridor min distance
- danger zone occupancy
- left/right clearance imbalance

Ví dụ gán nhãn:
- `SAFE`
- `WARNING`
- `DANGER`

---

## Algorithm 5 — Safety summary
Tạo summary kiểu:

```text
A close obstacle exists in the center corridor at 0.62m.
Left corridor is more open than right corridor.
Current locomotion state is WARNING.
```

---

## Algorithm 6 — Visualization
Bắt buộc có ít nhất 1 dạng plot:
- center / left / right clearance plot
- danger score bar chart
- zone occupancy chart

---

# 9. Cấu trúc folder

```text
mini_project_57_robot_frame_obstacle_distance_analyzer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ robot_cloud_inputs/
│  │  ├─ robot_cloud_pair_01_bm.xyz
│  │  ├─ robot_cloud_pair_01_sgm.xyz
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ safety_zone_report.txt
│     ├─ clearance_report.txt
│     ├─ danger_score_report.txt
│     ├─ locomotion_safety_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ clearance_plot.png
│     └─ danger_score_plot.png
│
├─ config/
│  ├─ robot_cloud_sample_config.txt
│  ├─ danger_zone_config.txt
│  ├─ corridor_config.txt
│  ├─ safety_threshold_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ obstacle_distance_graph_preview.py
│     ├─ obstacle_distance_bst.py
│     ├─ numpy_obstacle_distance_preview.py
│     ├─ matplotlib_obstacle_distance_plotter.py
│     └─ synthetic_robot_cloud_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ RobotCloudSample.hpp
   │  ├─ DangerZoneConfig.hpp
   │  ├─ CorridorConfig.hpp
   │  ├─ SafetyThresholdConfig.hpp
   │  ├─ SafetyZoneStatistic.hpp
   │  ├─ ObstacleDistanceResult.hpp
   │  ├─ ObstacleDistanceDebugRecord.hpp
   │  ├─ BaseSafetyZonePartitionEngine.hpp
   │  ├─ SafetyZonePartitionEngine.hpp
   │  ├─ BaseClearanceAnalysisEngine.hpp
   │  ├─ ClearanceAnalysisEngine.hpp
   │  ├─ BaseDangerScoringEngine.hpp
   │  ├─ DangerScoringEngine.hpp
   │  ├─ BaseLocomotionSafetySummaryEngine.hpp
   │  ├─ LocomotionSafetySummaryEngine.hpp
   │  ├─ RobotFrameObstacleDistanceAnalyzer.hpp
   │  └─ ObstacleDistanceReportWriter.hpp
   │
   └─ src/
      ├─ SafetyZonePartitionEngine.cpp
      ├─ ClearanceAnalysisEngine.cpp
      ├─ DangerScoringEngine.cpp
      ├─ LocomotionSafetySummaryEngine.cpp
      ├─ RobotFrameObstacleDistanceAnalyzer.cpp
      └─ ObstacleDistanceReportWriter.cpp
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
danger_zone_config_path
corridor_config_path
safety_threshold_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `RobotFrameObstacleDistanceConfigBuilder`

Tạo class con:

```python
class RobotFrameObstacleDistanceConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_robot_cloud_sample(
    pair_id,
    algorithm_type,
    robot_cloud_path,
    scene_label=None
)`

### `set_danger_zone(
    front_danger_distance,
    front_warning_distance
)`

### `set_corridor_config(
    corridor_half_width,
    left_right_split_x
)`

### `set_safety_thresholds(
    safe_distance_threshold,
    warning_distance_threshold,
    danger_score_threshold
)`

### `set_visualization_options(
    enable_clearance_plot,
    enable_danger_score_plot
)`

### `write_sample_config()`
### `write_danger_zone_config()`
### `write_corridor_config()`
### `write_safety_threshold_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `obstacle_distance_graph_preview.py`

Tạo class:

```python
class ObstacleDistanceGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
RobotFramePointCloud
→ SafetyZonePartition
→ ClearanceAnalysis
→ DangerScoring
→ SafetySummary
→ Report
```

---

# 10.4 Python — `obstacle_distance_bst.py`

Tạo BST cho `pair_id` / `safety_result_id`.

---

# 10.5 Python — `numpy_obstacle_distance_preview.py`

Tạo class:

```python
class NumPyObstacleDistancePreview:
```

## Hàm cần có

### `load_xyz_points(path)`
### `partition_safety_zones(points_xyz, danger_distance, warning_distance, corridor_half_width)`
### `compute_clearance_statistics(partitions)`
### `compute_danger_score(clearance_stats, thresholds)`

---

# 10.6 Python — `matplotlib_obstacle_distance_plotter.py`

Tạo class:

```python
class MatplotlibObstacleDistancePlotter:
```

## Hàm cần có

### `plot_clearance_summary(clearance_stats, save_path)`
### `plot_danger_scores(score_summary, save_path)`

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

# 10.8 C++ — `RobotCloudSample`

```cpp
enum class RobotCloudSourceType
{
    BM,
    SGM
};

struct RobotCloudSample
{
    std::string pair_id;
    RobotCloudSourceType algorithm_type;
    std::string robot_cloud_path;
    std::string scene_label;
};
```

---

# 10.9 C++ — `DangerZoneConfig`

```cpp
struct DangerZoneConfig
{
    double front_danger_distance;
    double front_warning_distance;
};
```

---

# 10.10 C++ — `CorridorConfig`

```cpp
struct CorridorConfig
{
    double corridor_half_width;
    double left_right_split_x;
};
```

---

# 10.11 C++ — `SafetyThresholdConfig`

```cpp
struct SafetyThresholdConfig
{
    double safe_distance_threshold;
    double warning_distance_threshold;
    double danger_score_threshold;
};
```

---

# 10.12 C++ — `SafetyZoneStatistic`

```cpp
enum class SafetyZoneType
{
    FRONT_DANGER,
    FRONT_WARNING,
    LEFT_CORRIDOR,
    CENTER_CORRIDOR,
    RIGHT_CORRIDOR
};

struct SafetyZoneStatistic
{
    std::string pair_id;
    SafetyZoneType zone_type;

    int point_count;
    double ratio;
    double nearest_distance;
    double average_distance;
};
```

---

# 10.13 C++ — `ObstacleDistanceResult`

```cpp
enum class LocomotionSafetyLabel
{
    SAFE,
    WARNING,
    DANGER
};

struct ObstacleDistanceResult
{
    std::string pair_id;
    RobotCloudSourceType algorithm_type;

    double front_nearest_distance;
    double center_average_distance;
    double left_clearance;
    double right_clearance;
    double danger_score;

    LocomotionSafetyLabel safety_label;
    std::string safety_summary;
};
```

---

# 10.14 C++ — `ObstacleDistanceDebugRecord`

```cpp
struct ObstacleDistanceDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.15 C++ — `BaseSafetyZonePartitionEngine`

Tạo abstract class:

```cpp
class BaseSafetyZonePartitionEngine
{
public:
    virtual std::vector<SafetyZoneStatistic> partition(
        const std::string& pair_id,
        const std::vector<Point3D>& robot_points,
        const DangerZoneConfig& danger_config,
        const CorridorConfig& corridor_config
    ) const = 0;

    virtual ~BaseSafetyZonePartitionEngine() = default;
};
```

---

# 10.16 C++ — `SafetyZonePartitionEngine`

Kế thừa `BaseSafetyZonePartitionEngine`.

## Nhiệm vụ
- chia cloud thành FRONT_DANGER / FRONT_WARNING / LEFT / CENTER / RIGHT corridor

---

# 10.17 C++ — `BaseClearanceAnalysisEngine`

Tạo abstract class:

```cpp
class BaseClearanceAnalysisEngine
{
public:
    virtual void analyze(
        ObstacleDistanceResult& result,
        const std::vector<SafetyZoneStatistic>& zone_stats
    ) const = 0;

    virtual ~BaseClearanceAnalysisEngine() = default;
};
```

---

# 10.18 C++ — `ClearanceAnalysisEngine`

Kế thừa `BaseClearanceAnalysisEngine`.

## Nhiệm vụ
- tính front nearest distance
- center average distance
- left / right clearance

---

# 10.19 C++ — `BaseDangerScoringEngine`

Tạo abstract class:

```cpp
class BaseDangerScoringEngine
{
public:
    virtual void score(
        ObstacleDistanceResult& result,
        const std::vector<SafetyZoneStatistic>& zone_stats,
        const SafetyThresholdConfig& thresholds
    ) const = 0;

    virtual ~BaseDangerScoringEngine() = default;
};
```

---

# 10.20 C++ — `DangerScoringEngine`

Kế thừa `BaseDangerScoringEngine`.

## Nhiệm vụ
- tính danger score
- gán SAFE / WARNING / DANGER

---

# 10.21 C++ — `BaseLocomotionSafetySummaryEngine`

Tạo abstract class:

```cpp
class BaseLocomotionSafetySummaryEngine
{
public:
    virtual std::string summarize(
        const ObstacleDistanceResult& result,
        const std::vector<SafetyZoneStatistic>& zone_stats
    ) const = 0;

    virtual ~BaseLocomotionSafetySummaryEngine() = default;
};
```

---

# 10.22 C++ — `LocomotionSafetySummaryEngine`

Kế thừa `BaseLocomotionSafetySummaryEngine`.

## Nhiệm vụ
- tạo summary kiểu:
```text
A close obstacle exists in the center corridor at 0.62m.
Left corridor is more open than right corridor.
Current locomotion state is WARNING.
```

---

# 10.23 C++ — `RobotFrameObstacleDistanceAnalyzer`

Tạo class trung tâm:

```cpp
class RobotFrameObstacleDistanceAnalyzer
```

## Thuộc tính

```cpp
private:
    std::vector<RobotCloudSample> samples;
    DangerZoneConfig danger_config;
    CorridorConfig corridor_config;
    SafetyThresholdConfig threshold_config;

    std::shared_ptr<BaseSafetyZonePartitionEngine> partition_engine;
    std::shared_ptr<BaseClearanceAnalysisEngine> clearance_engine;
    std::shared_ptr<BaseDangerScoringEngine> scoring_engine;
    std::shared_ptr<BaseLocomotionSafetySummaryEngine> summary_engine;

    std::vector<SafetyZoneStatistic> zone_statistics;
    std::vector<ObstacleDistanceResult> results;
    std::stack<ObstacleDistanceDebugRecord> debug_history;
```

## Hàm cần có

### `load_sample_config(const std::string& path)`
### `load_danger_zone_config(const std::string& path)`
### `load_corridor_config(const std::string& path)`
### `load_safety_threshold_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each robot cloud sample:
    load robot-frame points
    zone_stats = partition_engine.partition(...)
    build ObstacleDistanceResult
    clearance_engine.analyze(...)
    scoring_engine.score(...)
    result.safety_summary = summary_engine.summarize(...)
    save result
    push debug history
```

### `const std::vector<SafetyZoneStatistic>& get_zone_statistics() const`
### `const std::vector<ObstacleDistanceResult>& get_results() const`
### `std::vector<ObstacleDistanceDebugRecord> get_debug_history_reverse()`

---

# 10.24 C++ — `ObstacleDistanceReportWriter`

Tạo class:

```cpp
class ObstacleDistanceReportWriter
```

## Hàm cần có

### `write_safety_zone_report(...)`
Ví dụ:

```text
[Safety Zone Report]
Pair: hallway_01
Zone: CENTER_CORRIDOR
Point Count: 4210
Nearest Distance: 0.62
Average Distance: 1.94
```

### `write_clearance_report(...)`
Ví dụ:

```text
[Clearance Report]
Pair: hallway_01
Front Nearest Distance: 0.62
Center Average Distance: 1.94
Left Clearance: 2.31
Right Clearance: 1.44
```

### `write_danger_score_report(...)`
Ví dụ:

```text
[Danger Score]
Pair: hallway_01
Danger Score: 78
Safety Label: WARNING
```

### `write_locomotion_safety_report(...)`
Ví dụ:

```text
[Locomotion Safety]
Pair: hallway_01
A close obstacle exists in the center corridor at 0.62m.
Left corridor is more open than right corridor.
Current locomotion state is WARNING.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.25 C++ — `main.cpp`

## Yêu cầu

```text
Load robot cloud sample config
Load danger zone config
Load corridor config
Load safety threshold config
Load visualization config
Load runtime config

Create:
    SafetyZonePartitionEngine
    ClearanceAnalysisEngine
    DangerScoringEngine
    LocomotionSafetySummaryEngine

Create RobotFrameObstacleDistanceAnalyzer
Run
Write reports
```

---

# 11. Luật obstacle distance analysis của project

## Luật 1 — Phải phân tích theo robot frame, không quay lại camera frame
Đây là bài safety cho robot, nên mọi khoảng cách phải đo trong robot frame.

## Luật 2 — Center corridor là bắt buộc
Vì đây là vùng quan trọng nhất cho hành vi đi thẳng.

## Luật 3 — Danger score phải dựa trên số liệu thật
Ít nhất phải dùng:
- front nearest distance
- center corridor distance
- danger zone occupancy

## Luật 4 — Safety label phải được gán tự động
Không được ghi tay SAFE / WARNING / DANGER.

## Luật 5 — Report phải usable cho locomotion reasoning
Tức là report phải đủ để trả lời:
- robot có nên đi tiếp không?
- bên nào thoáng hơn?
- obstacle hiện tại đang nguy hiểm đến mức nào?

---

# 12. Output mong muốn

## Config
```text
config/robot_cloud_sample_config.txt
config/danger_zone_config.txt
config/corridor_config.txt
config/safety_threshold_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/safety_zone_report.txt
assets/outputs/clearance_report.txt
assets/outputs/danger_score_report.txt
assets/outputs/locomotion_safety_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/clearance_plot.png
assets/outputs/danger_score_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho robot cloud / danger zone / corridor / thresholds
- preview safety zones bằng NumPy
- visualize clearance và danger score bằng Matplotlib

## C++
- chạy robot-frame obstacle distance analysis
- tính front distance / corridor clearance / danger score
- tạo safety report usable cho locomotion

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là bước perception trả lời trực tiếp cho câu hỏi:

> **Robot có thể tiến lên an toàn hay không?**

Pipeline lúc này sẽ là:

```text
Stereo pair
→ disparity
→ depth
→ point cloud
→ camera-frame analysis
→ robot-frame transform
→ obstacle distance analysis
→ locomotion safety reasoning
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho robot cloud / danger zone / corridor / thresholds / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / safety result ids
- [ ] Python có NumPy obstacle distance preview
- [ ] Python có Matplotlib clearance / danger score plots
- [ ] C++ có `RobotCloudSample`
- [ ] C++ có `DangerZoneConfig`
- [ ] C++ có `CorridorConfig`
- [ ] C++ có `SafetyThresholdConfig`
- [ ] C++ có `SafetyZoneStatistic`
- [ ] C++ có `ObstacleDistanceResult`
- [ ] C++ có `SafetyZonePartitionEngine`
- [ ] C++ có `ClearanceAnalysisEngine`
- [ ] C++ có `DangerScoringEngine`
- [ ] C++ có `LocomotionSafetySummaryEngine`
- [ ] C++ có `RobotFrameObstacleDistanceAnalyzer`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 57**, bước hợp lý nhất để tiếp tục Đợt 12 là:

# **Bài 58: Free-Space Corridor Estimator**

Ý tưởng:
```text
robot-frame point cloud
→ center corridor occupancy
→ left/right free-space estimation
→ best walking corridor selection
→ corridor confidence score
→ walking corridor report
```

Tức là mạch sẽ đi:

```text
Bài 56: Camera-to-Robot transform
→ Bài 57: Robot-frame obstacle distance analysis
→ Bài 58: Free-space corridor estimation
```
