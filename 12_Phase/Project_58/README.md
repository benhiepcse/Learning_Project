# 🤖 Bài 58: Free-Space Corridor Estimator — Bộ ước lượng hành lang trống cho Humanoid Robot AI Perception

> Mini Project số 58 trong **Đợt 12**  
> **Bài 58 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12** và đi tiếp trực tiếp từ **Bài 57**.
>
> Nếu:
>
> - **Bài 56** giúp bạn đưa **point cloud sang robot frame**
> - **Bài 57** giúp bạn phân tích **obstacle distance + danger score**
>
> thì **Bài 58** là bước tiếp theo rất tự nhiên:
>
> ```text
> robot-frame point cloud
> → center corridor occupancy analysis
> → left / center / right free-space estimation
> → best walking corridor selection
> → corridor confidence score
> → walking corridor report
> ```
>
> Đây là bước mà perception không chỉ trả lời:
>
> - “phía trước có vật cản hay không?”
>
> mà còn tiến thêm một bước:
>
> - **“robot nên đi qua corridor nào?”**
> - **“corridor giữa có đủ thoáng để đi thẳng không?”**
> - **“nếu corridor giữa bị chặn, corridor trái hay phải tốt hơn?”**
>
> Tức là dữ liệu bắt đầu usable cho:
> - walking corridor selection
> - local navigation heuristics
> - obstacle avoidance direction hint
> - locomotion safety planning

---

# 📌 Mục lục

- [1. Đợt 12 đang đi tiếp từ Bài 57 ra sao](#1-đợt-12-đang-đi-tiếp-từ-bài-57-ra-sao)
- [2. Vì sao Bài 58 xuất hiện sau Bài 57](#2-vì-sao-bài-58-xuất-hiện-sau-bài-57)
- [3. Bài 58 nâng từ Bài 57 lên chỗ nào](#3-bài-58-nâng-từ-bài-57-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật free-space corridor estimation của project](#11-luật-free-space-corridor-estimation-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 12 đang đi tiếp từ Bài 57 ra sao

Ở **Bài 57**, bạn đã có:

```text
robot-frame point cloud
→ front danger zone analysis
→ center corridor distance analysis
→ left / right clearance estimation
→ danger score
→ locomotion safety report
```

Tức là bạn đã biết:
- obstacle phía trước gần hay xa
- vùng center có nguy hiểm không
- trái / phải bên nào thoáng hơn

Nhưng robot vẫn còn thiếu một bước rất quan trọng:

> **Từ các khoảng trống hiện tại, corridor nào là corridor nên đi?**

Đó chính là mục tiêu của **Bài 58**.

---

# 2. Vì sao Bài 58 xuất hiện sau Bài 57

## Bài 57 đang làm gì?
Bài 57 giúp bạn đánh giá:
- **độ nguy hiểm của obstacle**
- **clearance trái / giữa / phải**
- **khả năng tiến lên an toàn hay không**

## Nhưng còn thiếu gì?
Robot không chỉ cần biết “nguy hiểm hay không”, mà còn cần một **gợi ý hành động đơn giản**:

- corridor giữa có đi được không?
- nếu không, trái hay phải là lựa chọn tốt hơn?
- mức tự tin của corridor đó là bao nhiêu?

## Bài 58 lấp đúng chỗ đó
Bài 58 sẽ làm:

```text
robot-frame point cloud
→ partition free-space corridors
→ estimate corridor occupancy
→ score left / center / right corridors
→ choose best walking corridor
→ corridor confidence report
```

Đây là bước chuyển từ **obstacle danger analysis** sang **free-space decision support**.

---

# 3. Bài 58 nâng từ Bài 57 lên chỗ nào

## Bài 57
- front danger analysis
- center corridor distance analysis
- left / right clearance
- safety label

## Bài 58
- chia **walking corridors**
- ước lượng **mức trống / occupancy** của từng corridor
- chấm **corridor score**
- chọn **best walking corridor**
- xuất **corridor confidence report**

### Nói ngắn gọn:
- **Bài 57** hỏi: “Robot có đang gặp vật cản nguy hiểm không?”
- **Bài 58** hỏi: “Nếu robot muốn đi tiếp, nên đi corridor nào?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Free-Space Corridor Estimator**

System này nhận đầu vào là:
- một hoặc nhiều **robot-frame point cloud samples**
- một bộ **corridor geometry configs**
- một bộ **occupancy / scoring thresholds**
- một bộ **reporting options**

## Mỗi robot cloud sample tối thiểu chứa
- `pair_id`
- `algorithm_type`
- `robot_cloud_path`
- optional `scene_label`

## Nhiệm vụ của system
### Với mỗi robot-frame point cloud:
1. load point cloud
2. chia không gian đi bộ thành ít nhất:
   - `LEFT_CORRIDOR`
   - `CENTER_CORRIDOR`
   - `RIGHT_CORRIDOR`
3. với từng corridor, tính:
   - số điểm vật cản
   - occupancy ratio
   - nearest obstacle distance
   - average free distance
4. tính **corridor score**
5. chọn **best corridor**
6. tính **corridor confidence**
7. ghi **walking corridor report**

<p align="center">
  <img src="images/project_58.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build robot-cloud config
→ build corridor geometry config
→ preview corridor occupancy bằng NumPy
→ visualize corridor scores bằng Matplotlib

C++
→ load robot-frame point cloud
→ partition walking corridors
→ compute corridor occupancy / nearest obstacle / free-distance stats
→ choose best corridor
→ export corridor reports
```

Mục tiêu cốt lõi:
- hiểu cách biểu diễn **free space** trong robot frame
- biết cách chấm điểm **corridor đi bộ**
- biết cách chọn **left / center / right corridor tốt nhất**
- chuẩn bị nền trực tiếp cho:
  - local navigation heuristics
  - walking direction suggestion
  - obstacle avoidance direction hint
  - corridor-based locomotion planning

---

# 6. Pipeline tổng thể

```text
Load Robot Cloud Sample Config
Load Corridor Geometry Config
Load Corridor Scoring Config
Load Visualization Config
Load Runtime Config

Create RobotCloudLoader
Create CorridorPartitionEngine
Create CorridorOccupancyAnalysisEngine
Create CorridorScoringEngine
Create BestCorridorSelectionEngine
Create FreeSpaceCorridorEstimator

For each robot cloud sample:
    1. load robot-frame point cloud
    2. partition points into walking corridors
    3. compute occupancy / nearest-distance / free-distance stats
    4. compute corridor scores
    5. select best corridor
    6. compute confidence
    7. write result

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
- robot-frame point cloud
- free space
- corridor occupancy
- corridor scoring
- direction selection

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **free-space corridor estimation pipeline**:

```text
RobotFramePointCloud
→ CorridorPartition
→ OccupancyAnalysis
→ CorridorScoring
→ BestCorridorSelection
→ Report
```

## 2. Python BST
Lưu `pair_id`, `robot_cloud_id`, `corridor_result_id`.

## 3. `std::vector<RobotCloudSample>`
Danh sách robot-frame point cloud samples.

## 4. `std::vector<CorridorStatistic>`
Thống kê từng corridor.

## 5. `std::vector<CorridorEstimationResult>`
Kết quả ước lượng corridor.

## 6. `std::vector<Point3D>`
Danh sách điểm robot-frame.

## 7. `std::stack<CorridorDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Corridor partition
Chia point cloud thành ít nhất:
- `LEFT_CORRIDOR`
- `CENTER_CORRIDOR`
- `RIGHT_CORRIDOR`

Bạn có thể định nghĩa bằng:
- trục ngang robot `x`
- bề rộng corridor
- chiều dài vùng xét phía trước robot

---

## Algorithm 2 — Corridor occupancy analysis
Với từng corridor, tính ít nhất:
- point count
- occupancy ratio
- nearest obstacle distance
- average forward distance

---

## Algorithm 3 — Corridor scoring
Tính `corridor_score` cho từng corridor dựa trên:
- occupancy thấp thì tốt
- nearest obstacle xa thì tốt
- average free distance lớn thì tốt

---

## Algorithm 4 — Best corridor selection
Chọn corridor tốt nhất:
- `LEFT`
- `CENTER`
- `RIGHT`

Nếu cần, ưu tiên `CENTER` khi điểm số tương đương để giữ hành vi đi thẳng ổn định.

---

## Algorithm 5 — Corridor confidence
Tính `confidence_score`, ví dụ dựa trên:
- khoảng cách giữa top-1 và top-2 corridor scores
- mức occupancy tuyệt đối
- độ ổn định của corridor center

---

## Algorithm 6 — Visualization
Bắt buộc có ít nhất 1 dạng plot:
- corridor occupancy plot
- corridor score bar chart
- corridor confidence plot

---

# 9. Cấu trúc folder

```text
mini_project_58_free_space_corridor_estimator/
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
│     ├─ corridor_statistics_report.txt
│     ├─ corridor_score_report.txt
│     ├─ best_corridor_report.txt
│     ├─ corridor_runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ corridor_occupancy_plot.png
│     └─ corridor_score_plot.png
│
├─ config/
│  ├─ robot_cloud_sample_config.txt
│  ├─ corridor_geometry_config.txt
│  ├─ corridor_scoring_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ corridor_graph_preview.py
│     ├─ corridor_bst.py
│     ├─ numpy_corridor_preview.py
│     ├─ matplotlib_corridor_plotter.py
│     └─ synthetic_corridor_cloud_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ RobotCloudSample.hpp
   │  ├─ CorridorGeometryConfig.hpp
   │  ├─ CorridorScoringConfig.hpp
   │  ├─ CorridorStatistic.hpp
   │  ├─ CorridorEstimationResult.hpp
   │  ├─ CorridorDebugRecord.hpp
   │  ├─ BaseCorridorPartitionEngine.hpp
   │  ├─ CorridorPartitionEngine.hpp
   │  ├─ BaseCorridorOccupancyAnalysisEngine.hpp
   │  ├─ CorridorOccupancyAnalysisEngine.hpp
   │  ├─ BaseCorridorScoringEngine.hpp
   │  ├─ CorridorScoringEngine.hpp
   │  ├─ BaseBestCorridorSelectionEngine.hpp
   │  ├─ BestCorridorSelectionEngine.hpp
   │  ├─ FreeSpaceCorridorEstimator.hpp
   │  └─ CorridorReportWriter.hpp
   │
   └─ src/
      ├─ CorridorPartitionEngine.cpp
      ├─ CorridorOccupancyAnalysisEngine.cpp
      ├─ CorridorScoringEngine.cpp
      ├─ BestCorridorSelectionEngine.cpp
      ├─ FreeSpaceCorridorEstimator.cpp
      └─ CorridorReportWriter.cpp
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
corridor_geometry_config_path
corridor_scoring_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `FreeSpaceCorridorConfigBuilder`

Tạo class con:

```python
class FreeSpaceCorridorConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_robot_cloud_sample(
    pair_id,
    algorithm_type,
    robot_cloud_path,
    scene_label=None
)`

### `set_corridor_geometry(
    corridor_half_width,
    forward_analysis_distance,
    left_center_split_x,
    right_center_split_x
)`

### `set_corridor_scoring(
    occupancy_weight,
    nearest_distance_weight,
    average_distance_weight
)`

### `set_visualization_options(
    enable_corridor_occupancy_plot,
    enable_corridor_score_plot
)`

### `write_sample_config()`
### `write_corridor_geometry_config()`
### `write_corridor_scoring_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `corridor_graph_preview.py`

Tạo class:

```python
class CorridorGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
RobotFramePointCloud
→ CorridorPartition
→ OccupancyAnalysis
→ CorridorScoring
→ BestCorridorSelection
→ Report
```

---

# 10.4 Python — `corridor_bst.py`

Tạo BST cho `pair_id` / `corridor_result_id`.

---

# 10.5 Python — `numpy_corridor_preview.py`

Tạo class:

```python
class NumPyCorridorPreview:
```

## Hàm cần có

### `load_xyz_points(path)`
### `partition_corridors(points_xyz, geometry_config)`
### `compute_corridor_statistics(partitions)`
### `compute_corridor_scores(corridor_stats, scoring_config)`

---

# 10.6 Python — `matplotlib_corridor_plotter.py`

Tạo class:

```python
class MatplotlibCorridorPlotter:
```

## Hàm cần có

### `plot_corridor_occupancy(corridor_stats, save_path)`
### `plot_corridor_scores(score_summary, save_path)`

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

# 10.9 C++ — `CorridorGeometryConfig`

```cpp
struct CorridorGeometryConfig
{
    double corridor_half_width;
    double forward_analysis_distance;
    double left_center_split_x;
    double right_center_split_x;
};
```

---

# 10.10 C++ — `CorridorScoringConfig`

```cpp
struct CorridorScoringConfig
{
    double occupancy_weight;
    double nearest_distance_weight;
    double average_distance_weight;
};
```

---

# 10.11 C++ — `CorridorStatistic`

```cpp
enum class CorridorType
{
    LEFT_CORRIDOR,
    CENTER_CORRIDOR,
    RIGHT_CORRIDOR
};

struct CorridorStatistic
{
    std::string pair_id;
    CorridorType corridor_type;

    int point_count;
    double occupancy_ratio;
    double nearest_obstacle_distance;
    double average_forward_distance;
    double corridor_score;
};
```

---

# 10.12 C++ — `CorridorEstimationResult`

```cpp
struct CorridorEstimationResult
{
    std::string pair_id;
    RobotCloudSourceType algorithm_type;

    CorridorType best_corridor;
    double best_corridor_score;
    double confidence_score;

    std::string corridor_summary;
};
```

---

# 10.13 C++ — `CorridorDebugRecord`

```cpp
struct CorridorDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.14 C++ — `BaseCorridorPartitionEngine`

Tạo abstract class:

```cpp
class BaseCorridorPartitionEngine
{
public:
    virtual std::vector<CorridorStatistic> partition(
        const std::string& pair_id,
        const std::vector<Point3D>& robot_points,
        const CorridorGeometryConfig& geometry
    ) const = 0;

    virtual ~BaseCorridorPartitionEngine() = default;
};
```

---

# 10.15 C++ — `CorridorPartitionEngine`

Kế thừa `BaseCorridorPartitionEngine`.

## Nhiệm vụ
- chia cloud thành LEFT / CENTER / RIGHT corridor
- tạo thống kê point count sơ bộ

---

# 10.16 C++ — `BaseCorridorOccupancyAnalysisEngine`

Tạo abstract class:

```cpp
class BaseCorridorOccupancyAnalysisEngine
{
public:
    virtual void analyze(std::vector<CorridorStatistic>& stats) const = 0;
    virtual ~BaseCorridorOccupancyAnalysisEngine() = default;
};
```

---

# 10.17 C++ — `CorridorOccupancyAnalysisEngine`

Kế thừa `BaseCorridorOccupancyAnalysisEngine`.

## Nhiệm vụ
- tính occupancy ratio
- nearest obstacle distance
- average forward distance

---

# 10.18 C++ — `BaseCorridorScoringEngine`

Tạo abstract class:

```cpp
class BaseCorridorScoringEngine
{
public:
    virtual void score(
        std::vector<CorridorStatistic>& stats,
        const CorridorScoringConfig& config
    ) const = 0;

    virtual ~BaseCorridorScoringEngine() = default;
};
```

---

# 10.19 C++ — `CorridorScoringEngine`

Kế thừa `BaseCorridorScoringEngine`.

## Nhiệm vụ
- tính corridor_score cho từng corridor

---

# 10.20 C++ — `BaseBestCorridorSelectionEngine`

Tạo abstract class:

```cpp
class BaseBestCorridorSelectionEngine
{
public:
    virtual CorridorEstimationResult select_best(
        const std::string& pair_id,
        RobotCloudSourceType source_type,
        const std::vector<CorridorStatistic>& stats
    ) const = 0;

    virtual ~BaseBestCorridorSelectionEngine() = default;
};
```

---

# 10.21 C++ — `BestCorridorSelectionEngine`

Kế thừa `BaseBestCorridorSelectionEngine`.

## Nhiệm vụ
- chọn corridor tốt nhất
- tính confidence score
- tạo corridor summary

Ví dụ:
```text
Center corridor is partially occupied.
Left corridor offers the largest free space and best forward clearance.
Confidence: 0.81
```

---

# 10.22 C++ — `FreeSpaceCorridorEstimator`

Tạo class trung tâm:

```cpp
class FreeSpaceCorridorEstimator
```

## Thuộc tính

```cpp
private:
    std::vector<RobotCloudSample> samples;
    CorridorGeometryConfig geometry_config;
    CorridorScoringConfig scoring_config;

    std::shared_ptr<BaseCorridorPartitionEngine> partition_engine;
    std::shared_ptr<BaseCorridorOccupancyAnalysisEngine> occupancy_engine;
    std::shared_ptr<BaseCorridorScoringEngine> scoring_engine;
    std::shared_ptr<BaseBestCorridorSelectionEngine> selection_engine;

    std::vector<CorridorStatistic> corridor_statistics;
    std::vector<CorridorEstimationResult> results;
    std::stack<CorridorDebugRecord> debug_history;
```

## Hàm cần có

### `load_sample_config(const std::string& path)`
### `load_corridor_geometry_config(const std::string& path)`
### `load_corridor_scoring_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each robot cloud sample:
    load robot-frame points
    corridor_stats = partition_engine.partition(...)
    occupancy_engine.analyze(corridor_stats)
    scoring_engine.score(corridor_stats, scoring_config)
    result = selection_engine.select_best(...)
    save result
    save corridor stats
    push debug history
```

### `const std::vector<CorridorStatistic>& get_corridor_statistics() const`
### `const std::vector<CorridorEstimationResult>& get_results() const`
### `std::vector<CorridorDebugRecord> get_debug_history_reverse()`

---

# 10.23 C++ — `CorridorReportWriter`

Tạo class:

```cpp
class CorridorReportWriter
```

## Hàm cần có

### `write_corridor_statistics_report(...)`
Ví dụ:

```text
[Corridor Statistics]
Pair: hallway_01
Corridor: CENTER
Occupancy Ratio: 0.32
Nearest Obstacle Distance: 0.84
Average Forward Distance: 1.91
Corridor Score: 0.58
```

### `write_corridor_score_report(...)`
Ví dụ:

```text
[Corridor Score]
Pair: hallway_01
LEFT: 0.81
CENTER: 0.58
RIGHT: 0.47
```

### `write_best_corridor_report(...)`
Ví dụ:

```text
[Best Corridor]
Pair: hallway_01
Selected Corridor: LEFT
Confidence Score: 0.81
Left corridor offers the largest free space and best forward clearance.
```

### `write_corridor_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.24 C++ — `main.cpp`

## Yêu cầu

```text
Load robot cloud sample config
Load corridor geometry config
Load corridor scoring config
Load visualization config
Load runtime config

Create:
    CorridorPartitionEngine
    CorridorOccupancyAnalysisEngine
    CorridorScoringEngine
    BestCorridorSelectionEngine

Create FreeSpaceCorridorEstimator
Run
Write reports
```

---

# 11. Luật free-space corridor estimation của project

## Luật 1 — Phải làm trên robot-frame point cloud
Vì corridor walking là khái niệm robot-centric.

## Luật 2 — Không được chỉ chọn corridor bằng point count
Ít nhất phải xét:
- occupancy
- nearest obstacle distance
- average forward distance

## Luật 3 — Center corridor phải luôn được phân tích
Vì đây là corridor quan trọng nhất cho hành vi đi thẳng.

## Luật 4 — Best corridor phải có confidence score
Không chỉ chọn corridor, mà phải nói robot “tự tin đến đâu”.

## Luật 5 — Report phải usable cho bước navigation / locomotion
Tức là phải trả lời được:
- corridor nào nên đi?
- corridor giữa có đủ thoáng không?
- nếu phải tránh, nên lệch trái hay phải?

---

# 12. Output mong muốn

## Config
```text
config/robot_cloud_sample_config.txt
config/corridor_geometry_config.txt
config/corridor_scoring_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/corridor_statistics_report.txt
assets/outputs/corridor_score_report.txt
assets/outputs/best_corridor_report.txt
assets/outputs/corridor_runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/corridor_occupancy_plot.png
assets/outputs/corridor_score_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho robot cloud / corridor geometry / scoring
- preview corridor occupancy bằng NumPy
- visualize corridor scores bằng Matplotlib

## C++
- chạy free-space corridor estimation
- chấm điểm left / center / right corridor
- chọn corridor tốt nhất cho robot

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Nếu robot muốn tiếp tục di chuyển, nó nên đi hướng nào?**

Pipeline lúc này sẽ là:

```text
Stereo pair
→ disparity
→ depth
→ point cloud
→ camera-frame analysis
→ robot-frame transform
→ obstacle distance analysis
→ free-space corridor estimation
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho robot cloud / corridor geometry / scoring / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / corridor result ids
- [ ] Python có NumPy corridor preview
- [ ] Python có Matplotlib corridor occupancy / score plots
- [ ] C++ có `RobotCloudSample`
- [ ] C++ có `CorridorGeometryConfig`
- [ ] C++ có `CorridorScoringConfig`
- [ ] C++ có `CorridorStatistic`
- [ ] C++ có `CorridorEstimationResult`
- [ ] C++ có `CorridorPartitionEngine`
- [ ] C++ có `CorridorOccupancyAnalysisEngine`
- [ ] C++ có `CorridorScoringEngine`
- [ ] C++ có `BestCorridorSelectionEngine`
- [ ] C++ có `FreeSpaceCorridorEstimator`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 58**, bước hợp lý nhất để tiếp tục Đợt 12 là:

# **Bài 59: Local Walking Direction Planner**

Ý tưởng:
```text
best walking corridor
→ target heading suggestion
→ left / center / right steering recommendation
→ obstacle-aware local movement decision
→ walking direction report
```

Tức là mạch sẽ đi:

```text
Bài 57: Robot-frame obstacle distance analysis
→ Bài 58: Free-space corridor estimation
→ Bài 59: Local walking direction planner
```
