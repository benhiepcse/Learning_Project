# 🤖 Bài 61: Robot Perception Pipeline Orchestrator — Bộ điều phối pipeline perception từ camera frame sang robot frame cho Humanoid Robot AI Perception

> Mini Project số 61 trong **Đợt 13**  
> **Bài 61 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13** và đi tiếp trực tiếp từ **Bài 60**.
>
> Vì bạn đang đi theo chuỗi cộng dồn:
>
> - **Bài 56–60**: depth → point cloud → obstacle analysis → corridor → local navigation decision
> - **Đợt 13** thêm:
>   - `Robot Perception Pipeline`
>   - `Camera to Robot Coordinate`
>
> nên bài đầu tiên hợp logic của Đợt 13 sẽ là **Bài 61** chứ không quay lại Bài 46.
>
> Nếu:
>
> - **Bài 56** đã transform point cloud sang robot frame
> - **Bài 57–60** đã dùng robot-frame data để phân tích obstacle / corridor / direction / navigation command
>
> thì **Bài 61** là bước tiếp theo rất đúng với Đợt 13:
>
> ```text
> stereo/depth/point-cloud intermediate results
> + camera-to-robot transform config
> + robot-frame perception modules
> → perception pipeline orchestration
> → stage-by-stage execution
> → unified robot perception report
> ```
>
> Đây là bước mà bạn không chỉ làm **một module perception đơn lẻ** nữa, mà bắt đầu làm **bộ điều phối toàn pipeline perception**:
>
> - stage nào chạy trước, stage nào chạy sau
> - input / output của từng stage là gì
> - robot-frame module nào nhận dữ liệu từ stage trước
> - toàn pipeline tạo ra báo cáo perception cuối cùng như thế nào
>
> Tức là dữ liệu bắt đầu usable cho:
> - robot perception pipeline orchestration
> - stage dependency management
> - multi-module perception runtime design
> - humanoid robot perception architecture mockup

---

# 📌 Mục lục

- [1. Đợt 13 thêm gì mới so với Đợt 12](#1-đợt-13-thêm-gì-mới-so-với-đợt-12)
- [2. Vì sao Bài 61 xuất hiện sau Bài 60](#2-vì-sao-bài-61-xuất-hiện-sau-bài-60)
- [3. Bài 61 nâng từ Bài 60 lên chỗ nào](#3-bài-61-nâng-từ-bài-60-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật robot perception orchestration của project](#11-luật-robot-perception-orchestration-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 13 thêm gì mới so với Đợt 12

## Đợt 12 tập trung vào gì?
Đợt 12 tập trung vào **Stereo Vision & Depth**:
- Depth Estimation
- Depth Map
- Point Cloud

và chuỗi project gần nhất của bạn đã đi tới:

```text
camera-frame point cloud
→ robot-frame point cloud
→ obstacle distance analysis
→ free-space corridor estimation
→ local walking direction
→ obstacle-aware local navigation decision
```

## Đợt 13 thêm gì?
Đợt 13 bổ sung 2 mảnh rất quan trọng:

### Computer Vision / Robotics Vision
- `Robot Perception Pipeline`
- `Camera to Robot Coordinate`

### Python
- Graph
- NumPy
- OOP
- dict / list / module

### C++
- OOP
- virtual class
- enum
- vector
- smart pointer

Tức là từ Đợt 13 trở đi, project không chỉ là **xử lý từng perception module**, mà phải bắt đầu **điều phối nhiều module thành một pipeline perception hoàn chỉnh**.

---

# 2. Vì sao Bài 61 xuất hiện sau Bài 60

## Bài 60 đang làm gì?
Bài 60 đã cho bạn một **local navigation decision layer** dựa trên:
- obstacle danger
- corridor confidence
- direction decision
- priority arbitration

## Nhưng còn thiếu gì?
Bạn vẫn đang làm project theo kiểu:
- mỗi bài là **một module perception / decision**
- chưa có một **pipeline orchestrator** đứng trên các module đó

Robot thật thì cần:
- biết **stage order**
- biết **module dependencies**
- biết **module nào nhận output từ module nào**
- biết **khi nào dùng camera-frame data, khi nào dùng robot-frame data**

## Bài 61 lấp đúng chỗ đó
Bài 61 sẽ làm:

```text
input sample
→ load stage graph
→ run perception stages in order
→ pass outputs giữa các stage
→ aggregate final robot perception result
→ write pipeline report
```

Đây là bước chuyển từ **single-module perception projects** sang **pipeline-level perception architecture**.

---

# 3. Bài 61 nâng từ Bài 60 lên chỗ nào

## Bài 60
- lấy direction result + danger result + corridor result
- ra final local navigation command

## Bài 61
- quản lý **nhiều stage perception**
- điều phối:
  - depth / point cloud stage
  - camera-to-robot transform stage
  - obstacle analysis stage
  - corridor stage
  - direction stage
  - navigation decision stage
- build **robot perception pipeline graph**
- xuất **unified perception report**

### Nói ngắn gọn:
- **Bài 60** hỏi: “Lệnh điều hướng cục bộ cuối cùng là gì?”
- **Bài 61** hỏi: “Toàn bộ robot perception pipeline được ghép và chạy như thế nào?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Robot Perception Pipeline Orchestrator**

System này nhận đầu vào là:
- một hoặc nhiều **pipeline samples**
- một bộ **stage configuration**
- một bộ **camera-to-robot transform configuration**
- một bộ **pipeline execution rules**
- một bộ **reporting options**

## Mỗi pipeline sample tối thiểu chứa
- `pair_id`
- `algorithm_type`
- optional các đường dẫn:
  - `depth_result_path`
  - `point_cloud_path`
  - `robot_cloud_path`
  - `obstacle_result_path`
  - `corridor_result_path`
  - `direction_result_path`
  - `navigation_result_path`
- optional `scene_label`

## Nhiệm vụ của system
### Với mỗi pipeline sample:
1. load pipeline sample config
2. load stage graph
3. xác định stage execution order
4. chạy từng stage theo thứ tự
5. truyền intermediate outputs giữa các stage
6. gom kết quả perception cuối cùng
7. ghi **robot perception pipeline report**

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build pipeline config
→ build stage graph
→ build camera-to-robot transform config
→ preview pipeline order bằng graph / NumPy
→ visualize stage dependency summary

C++
→ load pipeline sample
→ create stage executors
→ run pipeline stages theo dependency order
→ aggregate final perception outputs
→ export unified pipeline reports
```

Mục tiêu cốt lõi:
- hiểu cách tổ chức **robot perception pipeline** thay vì từng module rời rạc
- biết cách thiết kế **stage graph**
- biết cách nối **camera-frame → robot-frame → robot reasoning**
- chuẩn bị nền trực tiếp cho:
  - humanoid perception runtime architecture
  - ROS2 perception node graph thinking
  - multi-stage AI perception systems

---

# 6. Pipeline tổng thể

```text
Load Pipeline Sample Config
Load Stage Graph Config
Load Camera-to-Robot Transform Config
Load Pipeline Execution Config
Load Visualization Config
Load Runtime Config

Create PipelineStageRegistry
Create StageDependencyResolver
Create PipelineExecutionEngine
Create PerceptionResultAggregator
Create RobotPerceptionPipelineOrchestrator

For each pipeline sample:
    1. resolve stage order
    2. load / prepare stage inputs
    3. run depth / point cloud / transform / obstacle / corridor / direction / navigation stages
    4. collect stage outputs
    5. aggregate final perception result
    6. write pipeline report

After all samples:
    write reports
    write plots
```

---

# 7. Kiến thức cần

## Python
- Graph
- NumPy
- OOP
- dict / list / module
- file handling
- Matplotlib nếu muốn plot pipeline stats

## C++
- OOP
- inheritance
- abstract / virtual class
- enum
- vector
- smart pointer

## Computer Vision / Robotics Vision
- robot perception pipeline
- camera-to-robot coordinate
- stage dependency
- camera-frame vs robot-frame reasoning
- multi-stage perception architecture

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph — bắt buộc
Biểu diễn **robot perception pipeline graph**:

```text
StereoDepthStage
→ PointCloudStage
→ CameraToRobotTransformStage
→ ObstacleDistanceStage
→ FreeSpaceCorridorStage
→ DirectionPlanningStage
→ NavigationDecisionStage
```

Bạn phải có:
- adjacency list hoặc adjacency dict
- BFS / DFS
- topological-style execution order preview nếu bạn muốn làm thêm

## 2. Python BST
Lưu:
- `pair_id`
- `pipeline_result_id`
- `stage_result_id`

## 3. `std::vector<PipelineSample>`
Danh sách sample pipeline đầu vào.

## 4. `std::vector<StageExecutionRecord>`
Lưu lịch sử chạy từng stage.

## 5. `std::vector<PipelinePerceptionResult>`
Lưu kết quả perception cuối cùng.

## 6. `std::stack<PipelineDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Stage dependency resolution
Bạn phải mô tả rõ:
- stage nào phụ thuộc stage nào
- execution order được xác định ra sao

Ví dụ:

```text
DepthStage
→ PointCloudStage
→ CameraToRobotTransformStage
→ ObstacleDistanceStage
→ CorridorStage
→ DirectionStage
→ NavigationStage
```

---

## Algorithm 2 — Camera-to-robot transform stage routing
Bạn phải có một stage hoặc module đại diện cho:

```text
camera-frame data
→ camera-to-robot transform
→ robot-frame data
```

Đây là trọng tâm của Đợt 13.

---

## Algorithm 3 — Pipeline execution
Với mỗi sample:
- chạy các stage theo thứ tự
- stage sau nhận output stage trước
- nếu thiếu input thì ghi fallback / skipped status

---

## Algorithm 4 — Perception result aggregation
Sau khi chạy xong pipeline, tạo một result tổng hợp ít nhất chứa:
- `pair_id`
- `executed_stage_count`
- `final_navigation_command` nếu có
- `obstacle_status`
- `corridor_status`
- `pipeline_summary`

---

## Algorithm 5 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- pipeline stage graph preview
- executed stage count plot
- stage status summary chart

---

# 9. Cấu trúc folder

```text
mini_project_61_robot_perception_pipeline_orchestrator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ pipeline_inputs/
│  │  ├─ pipeline_sample_01_bm.txt
│  │  ├─ pipeline_sample_01_sgm.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ stage_execution_report.txt
│     ├─ pipeline_result_report.txt
│     ├─ pipeline_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ pipeline_stage_plot.png
│     └─ executed_stage_count_plot.png
│
├─ config/
│  ├─ pipeline_sample_config.txt
│  ├─ stage_graph_config.txt
│  ├─ camera_to_robot_transform_config.txt
│  ├─ pipeline_execution_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ pipeline_graph_preview.py
│     ├─ pipeline_bst.py
│     ├─ numpy_pipeline_preview.py
│     ├─ matplotlib_pipeline_plotter.py
│     └─ synthetic_pipeline_input_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ PipelineSample.hpp
   │  ├─ StageType.hpp
   │  ├─ StageGraphConfig.hpp
   │  ├─ CameraToRobotTransformConfig.hpp
   │  ├─ PipelineExecutionConfig.hpp
   │  ├─ StageExecutionRecord.hpp
   │  ├─ PipelinePerceptionResult.hpp
   │  ├─ PipelineDebugRecord.hpp
   │  ├─ BasePipelineStage.hpp
   │  ├─ DepthStage.hpp
   │  ├─ PointCloudStage.hpp
   │  ├─ CameraToRobotTransformStage.hpp
   │  ├─ ObstacleDistanceStage.hpp
   │  ├─ FreeSpaceCorridorStage.hpp
   │  ├─ DirectionPlanningStage.hpp
   │  ├─ NavigationDecisionStage.hpp
   │  ├─ StageDependencyResolver.hpp
   │  ├─ PipelineExecutionEngine.hpp
   │  ├─ PerceptionResultAggregator.hpp
   │  ├─ RobotPerceptionPipelineOrchestrator.hpp
   │  └─ PipelineReportWriter.hpp
   │
   └─ src/
      ├─ DepthStage.cpp
      ├─ PointCloudStage.cpp
      ├─ CameraToRobotTransformStage.cpp
      ├─ ObstacleDistanceStage.cpp
      ├─ FreeSpaceCorridorStage.cpp
      ├─ DirectionPlanningStage.cpp
      ├─ NavigationDecisionStage.cpp
      ├─ StageDependencyResolver.cpp
      ├─ PipelineExecutionEngine.cpp
      ├─ PerceptionResultAggregator.cpp
      ├─ RobotPerceptionPipelineOrchestrator.cpp
      └─ PipelineReportWriter.cpp
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
pipeline_sample_config_path
stage_graph_config_path
camera_to_robot_transform_config_path
pipeline_execution_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `RobotPerceptionPipelineConfigBuilder`

Tạo class con:

```python
class RobotPerceptionPipelineConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_pipeline_sample(
    pair_id,
    algorithm_type,
    depth_result_path=None,
    point_cloud_path=None,
    robot_cloud_path=None,
    obstacle_result_path=None,
    corridor_result_path=None,
    direction_result_path=None,
    navigation_result_path=None,
    scene_label=None
)`

### `set_camera_to_robot_transform(
    tx, ty, tz,
    roll_deg, pitch_deg, yaw_deg
)`

### `add_stage_dependency(stage_name, dependency_name)`

### `set_pipeline_execution_options(
    allow_skipped_stage,
    stop_on_critical_failure
)`

### `set_visualization_options(
    enable_stage_graph_plot,
    enable_stage_count_plot
)`

### `write_pipeline_sample_config()`
### `write_stage_graph_config()`
### `write_camera_to_robot_transform_config()`
### `write_pipeline_execution_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `pipeline_graph_preview.py`

Tạo class:

```python
class PipelineGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`
### `topological_preview()` *(khuyến khích)*

Graph ví dụ:

```text
StereoDepthStage
→ PointCloudStage
→ CameraToRobotTransformStage
→ ObstacleDistanceStage
→ FreeSpaceCorridorStage
→ DirectionPlanningStage
→ NavigationDecisionStage
```

---

# 10.4 Python — `pipeline_bst.py`

Tạo BST cho:
- `pair_id`
- `pipeline_result_id`
- `stage_result_id`

---

# 10.5 Python — `numpy_pipeline_preview.py`

Tạo class:

```python
class NumPyPipelinePreview:
```

## Hàm cần có

### `load_pipeline_sample(path)`
### `simulate_stage_execution(stage_order, sample_data)`
### `simulate_camera_to_robot_transform(metadata, transform_config)`
### `build_pipeline_summary(stage_results)`

---

# 10.6 Python — `matplotlib_pipeline_plotter.py`

Tạo class:

```python
class MatplotlibPipelinePlotter:
```

## Hàm cần có

### `plot_stage_execution_count(stage_records, save_path)`
### `plot_stage_status_summary(stage_records, save_path)`

---

# 10.7 C++ — `PipelineSample`

```cpp
enum class PipelineSourceType
{
    BM,
    SGM
};

struct PipelineSample
{
    std::string pair_id;
    PipelineSourceType algorithm_type;

    std::string depth_result_path;
    std::string point_cloud_path;
    std::string robot_cloud_path;
    std::string obstacle_result_path;
    std::string corridor_result_path;
    std::string direction_result_path;
    std::string navigation_result_path;

    std::string scene_label;
};
```

---

# 10.8 C++ — `StageType`

```cpp
enum class StageType
{
    DEPTH_STAGE,
    POINT_CLOUD_STAGE,
    CAMERA_TO_ROBOT_STAGE,
    OBSTACLE_DISTANCE_STAGE,
    FREE_SPACE_CORRIDOR_STAGE,
    DIRECTION_PLANNING_STAGE,
    NAVIGATION_DECISION_STAGE
};
```

---

# 10.9 C++ — `StageGraphConfig`

```cpp
struct StageGraphConfig
{
    std::vector<std::pair<StageType, StageType>> edges;
};
```

---

# 10.10 C++ — `CameraToRobotTransformConfig`

```cpp
struct CameraToRobotTransformConfig
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

# 10.11 C++ — `PipelineExecutionConfig`

```cpp
struct PipelineExecutionConfig
{
    bool allow_skipped_stage;
    bool stop_on_critical_failure;
};
```

---

# 10.12 C++ — `StageExecutionRecord`

```cpp
enum class StageExecutionStatus
{
    SUCCESS,
    SKIPPED,
    FAILED
};

struct StageExecutionRecord
{
    std::string pair_id;
    StageType stage_type;
    StageExecutionStatus status;
    std::string message;
};
```

---

# 10.13 C++ — `PipelinePerceptionResult`

```cpp
struct PipelinePerceptionResult
{
    std::string pair_id;
    PipelineSourceType algorithm_type;

    int executed_stage_count;
    std::string obstacle_status;
    std::string corridor_status;
    std::string final_navigation_command;
    std::string pipeline_summary;
};
```

---

# 10.14 C++ — `PipelineDebugRecord`

```cpp
struct PipelineDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.15 C++ — `BasePipelineStage`

Tạo abstract class:

```cpp
class BasePipelineStage
{
public:
    virtual StageType get_stage_type() const = 0;

    virtual StageExecutionRecord execute(
        const PipelineSample& sample
    ) const = 0;

    virtual ~BasePipelineStage() = default;
};
```

---

# 10.16 C++ — Các stage class cụ thể

Bạn phải tạo tối thiểu các class sau, mỗi class kế thừa `BasePipelineStage`:

- `DepthStage`
- `PointCloudStage`
- `CameraToRobotTransformStage`
- `ObstacleDistanceStage`
- `FreeSpaceCorridorStage`
- `DirectionPlanningStage`
- `NavigationDecisionStage`

## Ví dụ skeleton

```cpp
class CameraToRobotTransformStage : public BasePipelineStage
{
public:
    StageType get_stage_type() const override;
    StageExecutionRecord execute(const PipelineSample& sample) const override;
};
```

Mỗi stage có thể mock việc:
- kiểm tra input path
- ghi status
- trả stage execution record

---

# 10.17 C++ — `StageDependencyResolver`

Tạo class:

```cpp
class StageDependencyResolver
```

## Hàm cần có

### `load_graph(const StageGraphConfig& config)`
### `std::vector<StageType> resolve_execution_order() const`

Nếu muốn, bạn có thể mô phỏng topo-sort đơn giản.

---

# 10.18 C++ — `PipelineExecutionEngine`

Tạo class:

```cpp
class PipelineExecutionEngine
```

## Nhiệm vụ
- nhận danh sách stage executors
- chạy theo execution order
- thu `StageExecutionRecord`

## Hàm cần có

### `register_stage(std::shared_ptr<BasePipelineStage> stage)`
### `run_sample(const PipelineSample& sample, const std::vector<StageType>& order)`

---

# 10.19 C++ — `PerceptionResultAggregator`

Tạo class:

```cpp
class PerceptionResultAggregator
```

## Nhiệm vụ
- gom `StageExecutionRecord`
- build `PipelinePerceptionResult`

## Hàm cần có

### `PipelinePerceptionResult aggregate(
    const PipelineSample& sample,
    const std::vector<StageExecutionRecord>& records
) const`

---

# 10.20 C++ — `RobotPerceptionPipelineOrchestrator`

Tạo class trung tâm:

```cpp
class RobotPerceptionPipelineOrchestrator
```

## Thuộc tính

```cpp
private:
    std::vector<PipelineSample> samples;

    StageGraphConfig stage_graph_config;
    CameraToRobotTransformConfig transform_config;
    PipelineExecutionConfig execution_config;

    std::shared_ptr<StageDependencyResolver> dependency_resolver;
    std::shared_ptr<PipelineExecutionEngine> execution_engine;
    std::shared_ptr<PerceptionResultAggregator> aggregator;

    std::vector<StageExecutionRecord> stage_records;
    std::vector<PipelinePerceptionResult> results;
    std::stack<PipelineDebugRecord> debug_history;
```

## Hàm cần có

### `load_pipeline_sample_config(const std::string& path)`
### `load_stage_graph_config(const std::string& path)`
### `load_camera_to_robot_transform_config(const std::string& path)`
### `load_pipeline_execution_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
resolve execution order

for each sample:
    records = execution_engine.run_sample(sample, execution_order)
    result = aggregator.aggregate(sample, records)
    save stage records
    save result
    push debug history
```

### `const std::vector<StageExecutionRecord>& get_stage_records() const`
### `const std::vector<PipelinePerceptionResult>& get_results() const`
### `std::vector<PipelineDebugRecord> get_debug_history_reverse()`

---

# 10.21 C++ — `PipelineReportWriter`

Tạo class:

```cpp
class PipelineReportWriter
```

## Hàm cần có

### `write_stage_execution_report(...)`
Ví dụ:

```text
[Stage Execution]
Pair: room_01
Stage: CAMERA_TO_ROBOT_STAGE
Status: SUCCESS
Message: Robot-frame cloud generated successfully.
```

### `write_pipeline_result_report(...)`
Ví dụ:

```text
[Pipeline Result]
Pair: room_01
Executed Stages: 7
Obstacle Status: WARNING
Corridor Status: LEFT_FREE
Final Navigation Command: TURN_LEFT_IN_PLACE
```

### `write_pipeline_summary_report(...)`
Ví dụ:

```text
[Pipeline Summary]
Pair: room_01
The sample was processed from depth estimation to local navigation decision.
Robot-frame obstacle reasoning and corridor-based navigation suggestion were both produced successfully.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.22 C++ — `main.cpp`

## Yêu cầu

```text
Load pipeline sample config
Load stage graph config
Load camera-to-robot transform config
Load pipeline execution config
Load visualization config
Load runtime config

Create:
    StageDependencyResolver
    PipelineExecutionEngine
    PerceptionResultAggregator

Register stages:
    DepthStage
    PointCloudStage
    CameraToRobotTransformStage
    ObstacleDistanceStage
    FreeSpaceCorridorStage
    DirectionPlanningStage
    NavigationDecisionStage

Create RobotPerceptionPipelineOrchestrator
Run
Write reports
```

---

# 11. Luật robot perception orchestration của project

## Luật 1 — Camera-to-robot transform phải là stage riêng hoặc module riêng
Đây là trọng tâm của Đợt 13, không được bỏ.

## Luật 2 — Phải có stage dependency graph
Không được hard-code pipeline kiểu “gọi tay từng hàm” mà không mô tả quan hệ stage.

## Luật 3 — Phải có khả năng skipped / failed stage
Nếu thiếu input, report phải thể hiện rõ stage bị skip hay fail.

## Luật 4 — Pipeline report phải là report tổng hợp
Không chỉ ghi từng stage rời rạc, mà phải có summary cuối.

## Luật 5 — Project phải thể hiện đúng tinh thần “Robot Perception Pipeline”
Tức là phải thấy rõ:
- camera-frame data đi vào đâu
- robot-frame data được sinh ở stage nào
- obstacle / corridor / direction / navigation nằm ở các stage nào

---

# 12. Output mong muốn

## Config
```text
config/pipeline_sample_config.txt
config/stage_graph_config.txt
config/camera_to_robot_transform_config.txt
config/pipeline_execution_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/stage_execution_report.txt
assets/outputs/pipeline_result_report.txt
assets/outputs/pipeline_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/pipeline_stage_plot.png
assets/outputs/executed_stage_count_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build pipeline config
- build stage graph
- preview execution order
- build transform config
- hỗ trợ report / visualization

## C++
- đóng vai perception runtime orchestrator
- quản lý stage executors
- chạy pipeline theo dependency order
- tổng hợp perception result cuối

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Toàn bộ robot perception pipeline từ depth đến robot-frame reasoning được ghép và chạy như thế nào?**

Pipeline lúc này sẽ là:

```text
Stereo pair
→ depth
→ point cloud
→ camera-to-robot transform
→ robot-frame obstacle analysis
→ corridor estimation
→ direction planning
→ local navigation decision
→ unified robot perception report
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho pipeline sample / stage graph / transform / execution / visualization
- [ ] Python có graph preview bằng BFS / DFS / topological preview
- [ ] Python có BST cho pair ids / pipeline result ids / stage result ids
- [ ] Python có NumPy pipeline preview
- [ ] C++ có `PipelineSample`
- [ ] C++ có `StageType`
- [ ] C++ có `StageGraphConfig`
- [ ] C++ có `CameraToRobotTransformConfig`
- [ ] C++ có `PipelineExecutionConfig`
- [ ] C++ có `StageExecutionRecord`
- [ ] C++ có `PipelinePerceptionResult`
- [ ] C++ có đủ các stage classes
- [ ] C++ có `StageDependencyResolver`
- [ ] C++ có `PipelineExecutionEngine`
- [ ] C++ có `PerceptionResultAggregator`
- [ ] C++ có `RobotPerceptionPipelineOrchestrator`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 61**, bước hợp lý nhất để tiếp tục Đợt 13 là:

# **Bài 62: Camera-to-Robot Point Projection Analyzer**

Ý tưởng:
```text
pixel / depth / camera point
→ camera coordinate
→ robot coordinate
→ robot-frame target localization
→ projection consistency report
```

Tức là mạch sẽ đi:

```text
Bài 61: Robot perception pipeline orchestrator
→ Bài 62: Camera-to-robot point projection analyzer
→ Bài 63: Robot-frame target localization inspector
```
