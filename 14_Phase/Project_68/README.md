# 🤖 Bài 68: Table-Cleaning Perception Task Scheduler — Bộ lập lịch tác vụ perception cho chế độ dọn bàn của humanoid

> Mini Project số 68 trong **Đợt 14**  
> **Bài 68 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 65** nhưng theo nhánh **TABLE_CLEAN_MODE** của Đợt 14.
>
> Nếu:
>
> - **Bài 65** đã chọn target theo `FOLLOW_MODE / PICK_MODE / TABLE_CLEAN_MODE`
> - **Bài 66** đã đào sâu riêng nhánh **FOLLOW_MODE**
> - **Bài 67** đã đào sâu riêng nhánh **PICK_MODE**
>
> thì bước tiếp theo hợp lý nhất là đào sâu nhánh **TABLE_CLEAN_MODE**:
>
> ```text
> clean-mode targets
> → table-zone clutter filtering
> → cleaning priority grouping
> → action-ready cleaning task ordering
> → cleaning schedule report
> ```
>
> Đây là bước mà bạn không chỉ hỏi **target nào hợp lệ**, mà phải trả lời sâu hơn:
>
> - vật / vùng nào trên bàn thật sự cần dọn trước?
> - object nào nên được gắp / đẩy / gom trước theo logic dọn bàn?
> - nếu có nhiều clutter targets trên mặt bàn, robot nên tạo **thứ tự xử lý** như thế nào?
> - perception phải xuất ra **cleaning task schedule** usable cho planner / manipulation layer
>
> Tức là dữ liệu bắt đầu usable cho:
> - table-zone clutter reasoning
> - cleaning priority grouping
> - task ordering for table cleaning
> - perception-to-cleaning handoff

---

# 📌 Mục lục

- [1. Vì sao Bài 68 xuất hiện sau Bài 65, 66, 67](#1-vì-sao-bài-68-xuất-hiện-sau-bài-65-66-67)
- [2. Bài 68 nâng từ Bài 65 lên chỗ nào](#2-bài-68-nâng-từ-bài-65-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Table-cleaning perception pipeline tổng thể](#5-table-cleaning-perception-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật cleaning task scheduler của project](#10-luật-cleaning-task-scheduler-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 68 xuất hiện sau Bài 65, 66, 67

## Bài 65 đang làm gì?
Bài 65 đã giúp bạn:
- gom nhiều target trong robot frame
- áp dụng `FOLLOW_MODE / PICK_MODE / TABLE_CLEAN_MODE`
- chọn ra interaction target theo từng mode

## Bài 66 và 67 đã đào sâu gì?
- **Bài 66** đào sâu riêng nhánh **FOLLOW_MODE**
- **Bài 67** đào sâu riêng nhánh **PICK_MODE**

## Còn nhánh nào cần đào sâu?
Nhánh còn lại rất quan trọng của Đợt 14 là **TABLE_CLEAN_MODE**:
- object nào nằm trên bàn và cần dọn
- object nào nên xử lý trước
- object nào là clutter chính, object nào là phụ
- nếu có nhiều object trên bàn, robot nên sinh **task schedule** thế nào

Đó là lý do **Bài 68** xuất hiện.

---

# 2. Bài 68 nâng từ Bài 65 lên chỗ nào

## Bài 65
- chọn target cho `TABLE_CLEAN_MODE` ở mức tổng quát
- có task-aware selection nhưng chưa đào sâu logic dọn bàn

## Bài 68
- chỉ tập trung vào **TABLE_CLEAN_MODE**
- kiểm tra **table-zone validity**
- nhóm clutter theo mức ưu tiên
- chấm **cleaning priority score**
- tạo **task order** cho các object cần dọn
- sinh **cleaning schedule report**

### Nói ngắn gọn:
- **Bài 65** hỏi: “Robot nên chọn target nào cho TABLE_CLEAN_MODE?”
- **Bài 68** hỏi: “Trong các object trên bàn, robot nên dọn object nào trước, object nào sau, và lịch dọn nên sắp thế nào?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Table-Cleaning Perception Task Scheduler**

System này nhận đầu vào là:
- một hoặc nhiều **clean scenes**
- mỗi scene chứa nhiều **clean-mode targets** trên bàn
- một bộ **table-zone rules**
- một bộ **clutter grouping rules**
- một bộ **cleaning priority weights**
- optional:
  - object label / type
  - confidence
  - cleanable flag
  - table sector / cluster id
  - estimated clutter size

## Mỗi target sample tối thiểu chứa
- `scene_id`
- `target_id`
- `robot_x`
- `robot_y`
- `robot_z`
- `confidence`
- `cleanable_flag`

Optional:
- `object_label`
- `object_type`
- `table_sector`
- `cluster_id`
- `clutter_size`

## Nhiệm vụ của system
### Với mỗi clean scene:
1. load toàn bộ clean targets
2. với từng target:
   - classify zone
   - inspect reachability
   - inspect table-zone validity
   - inspect cleanability
   - compute cleaning priority score
3. group clutter targets theo rule
4. sort target theo cleaning priority
5. build **action-ready cleaning task order**
6. ghi **table-cleaning schedule report**

<p align="center">
  <img src="images/project_68.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build clean-scene config
→ build table-zone / cleanability / priority configs
→ preview cleaning order bằng NumPy
→ visualize cleaning task schedule summary

C++
→ load clean-mode robot-frame targets
→ inspect table-zone validity / cleanability / reachability
→ compute cleaning priority
→ build cleaning task ordering
→ export cleaning schedule reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **robot-frame target reasoning** với **table-cleaning perception**
- biết cách thiết kế **cleanability filtering**
- biết cách tách:
  - table-zone validity
  - cleanability validity
  - cleaning priority scoring
  - clutter grouping / task ordering
- chuẩn bị nền trực tiếp cho:
  - cleaning manipulation planning
  - clutter scheduling
  - table-cleaning task orchestration

---

# 5. Table-cleaning perception pipeline tổng thể

```text
Load Clean Scene Config
Load Robot Zone Config
Load Reachability Threshold Config
Load TableZoneRuleConfig
Load CleanabilityRuleConfig
Load CleaningPriorityWeightConfig
Load Visualization Config
Load Runtime Config

Create TargetZoneClassifier
Create ReachabilityInspector
Create TableZoneInspector
Create CleanabilityInspector
Create CleaningPriorityEngine
Create ClutterGroupingEngine
Create CleaningTaskOrderingEngine
Create CleaningSummaryEngine
Create TableCleaningPerceptionTaskScheduler

For each clean scene:
    1. load all clean targets
    2. inspect every target
    3. compute cleaning priority score
    4. group clutter targets
    5. build task ordering
    6. write report

After all scenes:
    write reports
    write plots
```

---

# 6. Kiến thức cần

## Python
- OOP
- file handling
- modules
- list / dict / set / tuple
- Graph / BST
- NumPy
- Matplotlib

## C++
- OOP / inheritance / abstract class
- vector / enum / smart pointer
- sorting / ranking / grouping logic
- project structure `.hpp / .cpp`

## Computer Vision / Robotics Vision
- robot-frame target localization
- table-cleaning perception
- table-zone reasoning
- cleanability filtering
- clutter grouping
- cleaning task scheduling

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **table-cleaning task scheduling pipeline**:

```text
CleanSceneTargets
→ ReachabilityInspection
→ TableZoneInspection
→ CleanabilityInspection
→ CleaningPriorityScoring
→ ClutterGrouping
→ TaskOrdering
→ CleaningScheduleReport
```

## 2. Python BST
Lưu:
- `scene_id`
- `target_id`
- `cleaning_result_id`

## 3. `std::vector<CleanTarget>`
Danh sách target trong một clean scene.

## 4. `std::vector<CleaningTaskResult>`
Danh sách kết quả inspect / score cho từng target.

## 5. `std::unordered_map<std::string, std::vector<CleanTarget>>`
Group target theo `scene_id`.

## 6. `std::stack<CleaningDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Spatial inspection reuse
Bạn phải tái sử dụng logic từ Bài 63–65:
- zone classification
- reachability inspection
- validity checking

---

## Algorithm 2 — Table-zone inspection
Một target có thể bị loại nếu:
- không nằm trong vùng mặt bàn hợp lệ
- quá thấp / quá cao so với mặt bàn
- nằm sau robot hoặc quá xa
- confidence quá thấp

---

## Algorithm 3 — Cleanability inspection
Một target có thể bị loại nếu:
- `cleanable_flag = false`
- object type không thuộc nhóm cần dọn
- object đang được đánh dấu là “do not clean”
- target nằm ngoài clutter zone

---

## Algorithm 4 — Cleaning priority scoring
Tạo **cleaning priority score** dựa trên:
- target ở sector ưu tiên nào của bàn
- target gần robot hay không
- clutter size lớn / nhỏ
- confidence
- center proximity / lateral offset
- object type priority

Ví dụ ý tưởng:
```text
cleaning_score =
    sector_bonus
  + cleanable_bonus
  + clutter_size_bonus
  + confidence_bonus
  - forward_penalty
  - lateral_penalty
```

---

## Algorithm 5 — Clutter grouping
Group target theo:
- `table_sector`
- `cluster_id`
- hoặc `object_type`

Mục tiêu:
- tạo các cụm task dọn thay vì xử lý object hoàn toàn rời rạc

---

## Algorithm 6 — Cleaning task ordering
Sort target theo `cleaning_priority_score` giảm dần, hoặc theo nhóm:
1. sector ưu tiên cao
2. cluster gần robot
3. object dễ dọn trước
4. object còn lại sau

---

## Algorithm 7 — Cleaning summary
Report cuối phải trả lời được:
- target / cluster nào cần dọn trước
- thứ tự dọn bàn là gì
- task backup / next-best order là gì

---

## Algorithm 8 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- cleaning priority chart
- clutter group distribution
- scene-wise cleaning schedule summary

---

# 8. Cấu trúc folder

```text
mini_project_68_table_cleaning_perception_task_scheduler/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ clean_scene_inputs/
│  │  ├─ clean_scene_01.txt
│  │  ├─ clean_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ cleaning_task_report.txt
│     ├─ cleaning_order_report.txt
│     ├─ cleaning_schedule_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ cleaning_priority_plot.png
│     └─ clutter_group_plot.png
│
├─ config/
│  ├─ clean_scene_config.txt
│  ├─ robot_zone_config.txt
│  ├─ reachability_threshold_config.txt
│  ├─ table_zone_rule_config.txt
│  ├─ cleanability_rule_config.txt
│  ├─ cleaning_priority_weight_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ cleaning_graph_preview.py
│     ├─ cleaning_bst.py
│     ├─ numpy_cleaning_preview.py
│     ├─ matplotlib_cleaning_plotter.py
│     └─ synthetic_clean_scene_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ CleanTarget.hpp
   │  ├─ RobotZoneConfig.hpp
   │  ├─ ReachabilityThresholdConfig.hpp
   │  ├─ TableZoneRuleConfig.hpp
   │  ├─ CleanabilityRuleConfig.hpp
   │  ├─ CleaningPriorityWeightConfig.hpp
   │  ├─ TargetZone.hpp
   │  ├─ CleaningTaskResult.hpp
   │  ├─ CleaningDebugRecord.hpp
   │  ├─ BaseTargetZoneClassifier.hpp
   │  ├─ TargetZoneClassifier.hpp
   │  ├─ BaseReachabilityInspector.hpp
   │  ├─ ReachabilityInspector.hpp
   │  ├─ BaseTableZoneInspector.hpp
   │  ├─ TableZoneInspector.hpp
   │  ├─ BaseCleanabilityInspector.hpp
   │  ├─ CleanabilityInspector.hpp
   │  ├─ BaseCleaningPriorityEngine.hpp
   │  ├─ CleaningPriorityEngine.hpp
   │  ├─ BaseClutterGroupingEngine.hpp
   │  ├─ ClutterGroupingEngine.hpp
   │  ├─ BaseCleaningTaskOrderingEngine.hpp
   │  ├─ CleaningTaskOrderingEngine.hpp
   │  ├─ BaseCleaningSummaryEngine.hpp
   │  ├─ CleaningSummaryEngine.hpp
   │  ├─ TableCleaningPerceptionTaskScheduler.hpp
   │  └─ CleaningReportWriter.hpp
   │
   └─ src/
      ├─ TargetZoneClassifier.cpp
      ├─ ReachabilityInspector.cpp
      ├─ TableZoneInspector.cpp
      ├─ CleanabilityInspector.cpp
      ├─ CleaningPriorityEngine.cpp
      ├─ ClutterGroupingEngine.cpp
      ├─ CleaningTaskOrderingEngine.cpp
      ├─ CleaningSummaryEngine.cpp
      ├─ TableCleaningPerceptionTaskScheduler.cpp
      └─ CleaningReportWriter.cpp
```

---

# 9. Yêu cầu mini-project

# 9.1 Python — `BaseConfigBuilder`

Tạo class:

```python
class BaseConfigBuilder:
```

## Thuộc tính
```python
project_name
clean_scene_config_path
robot_zone_config_path
reachability_threshold_config_path
table_zone_rule_config_path
cleanability_rule_config_path
cleaning_priority_weight_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `CleaningPlannerConfigBuilder`

Tạo class con:

```python
class CleaningPlannerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_target(
    scene_id,
    target_id,
    robot_x,
    robot_y,
    robot_z,
    confidence,
    cleanable_flag,
    object_label=None,
    object_type=None,
    table_sector=None,
    cluster_id=None,
    clutter_size=None
)`

### `set_robot_zone_config(
    center_band,
    left_right_boundary
)`

### `set_reachability_thresholds(
    min_forward,
    max_forward,
    max_lateral,
    min_height,
    max_height
)`

### `set_table_zone_rules(
    min_table_height,
    max_table_height,
    allowed_table_sectors
)`

### `set_cleanability_rules(
    min_clean_confidence,
    forbidden_object_types,
    allow_non_cleanable
)`

### `set_cleaning_priority_weights(
    sector_weight,
    confidence_weight,
    clutter_size_weight,
    forward_penalty_weight,
    lateral_penalty_weight
)`

### `set_visualization_options(
    enable_cleaning_priority_plot,
    enable_clutter_group_plot
)`

### `write_clean_scene_config()`
### `write_robot_zone_config()`
### `write_reachability_threshold_config()`
### `write_table_zone_rule_config()`
### `write_cleanability_rule_config()`
### `write_cleaning_priority_weight_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `cleaning_graph_preview.py`

Tạo class:

```python
class CleaningGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
CleanSceneTargets
→ ReachabilityInspection
→ TableZoneInspection
→ CleanabilityInspection
→ CleaningPriorityScoring
→ ClutterGrouping
→ TaskOrdering
→ CleaningScheduleReport
```

---

# 9.4 Python — `cleaning_bst.py`

Tạo BST cho:
- `scene_id`
- `target_id`
- `cleaning_result_id`

---

# 9.5 Python — `numpy_cleaning_preview.py`

Tạo class:

```python
class NumPyCleaningPreview:
```

## Hàm cần có

### `load_clean_scene(path)`
### `inspect_cleanability(targets, cleanability_rule_config)`
### `inspect_table_zone(targets, table_zone_rule_config)`
### `compute_cleaning_priority(targets, cleaning_priority_weight_config)`
### `build_cleaning_schedule(results)`

---

# 9.6 Python — `matplotlib_cleaning_plotter.py`

Tạo class:

```python
class MatplotlibCleaningPlotter:
```

## Hàm cần có

### `plot_cleaning_priority(results, save_path)`
### `plot_clutter_groups(results, save_path)`

---

# 9.7 C++ — `CleanTarget`

```cpp
struct CleanTarget
{
    std::string scene_id;
    std::string target_id;

    double robot_x;
    double robot_y;
    double robot_z;

    double confidence;
    bool cleanable_flag;

    std::string object_label;
    std::string object_type;
    std::string table_sector;
    std::string cluster_id;
    double clutter_size;
};
```

---

# 9.8 C++ — `RobotZoneConfig`

```cpp
struct RobotZoneConfig
{
    double center_band;
    double left_right_boundary;
};
```

---

# 9.9 C++ — `ReachabilityThresholdConfig`

```cpp
struct ReachabilityThresholdConfig
{
    double min_forward;
    double max_forward;
    double max_lateral;
    double min_height;
    double max_height;
};
```

---

# 9.10 C++ — `TableZoneRuleConfig`

```cpp
struct TableZoneRuleConfig
{
    double min_table_height;
    double max_table_height;
    std::vector<std::string> allowed_table_sectors;
};
```

---

# 9.11 C++ — `CleanabilityRuleConfig`

```cpp
struct CleanabilityRuleConfig
{
    double min_clean_confidence;
    std::vector<std::string> forbidden_object_types;
    bool allow_non_cleanable;
};
```

---

# 9.12 C++ — `CleaningPriorityWeightConfig`

```cpp
struct CleaningPriorityWeightConfig
{
    double sector_weight;
    double confidence_weight;
    double clutter_size_weight;
    double forward_penalty_weight;
    double lateral_penalty_weight;
};
```

---

# 9.13 C++ — `TargetZone`

```cpp
enum class TargetZone
{
    LEFT_ZONE,
    CENTER_ZONE,
    RIGHT_ZONE,
    REAR_ZONE,
    OUT_OF_RANGE
};
```

---

# 9.14 C++ — `CleaningTaskResult`

```cpp
struct CleaningTaskResult
{
    std::string scene_id;
    std::string target_id;

    TargetZone zone;

    bool is_reachable;
    bool in_table_zone;
    bool is_cleanable;
    bool is_valid_task;

    double cleaning_priority_score;
    int task_order;

    std::string cluster_id;
    std::string summary;
};
```

---

# 9.15 C++ — `CleaningDebugRecord`

```cpp
struct CleaningDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.16 C++ — `BaseTargetZoneClassifier`

```cpp
class BaseTargetZoneClassifier
{
public:
    virtual TargetZone classify(
        const CleanTarget& target,
        const RobotZoneConfig& config
    ) const = 0;

    virtual ~BaseTargetZoneClassifier() = default;
};
```

---

# 9.17 C++ — `BaseReachabilityInspector`

```cpp
class BaseReachabilityInspector
{
public:
    virtual bool inspect(
        const CleanTarget& target,
        const ReachabilityThresholdConfig& config
    ) const = 0;

    virtual ~BaseReachabilityInspector() = default;
};
```

---

# 9.18 C++ — `BaseTableZoneInspector`

```cpp
class BaseTableZoneInspector
{
public:
    virtual bool inspect(
        const CleanTarget& target,
        const TableZoneRuleConfig& config
    ) const = 0;

    virtual ~BaseTableZoneInspector() = default;
};
```

---

# 9.19 C++ — `BaseCleanabilityInspector`

```cpp
class BaseCleanabilityInspector
{
public:
    virtual bool inspect(
        const CleanTarget& target,
        const CleanabilityRuleConfig& config
    ) const = 0;

    virtual ~BaseCleanabilityInspector() = default;
};
```

---

# 9.20 C++ — `BaseCleaningPriorityEngine`

```cpp
class BaseCleaningPriorityEngine
{
public:
    virtual double compute(
        const CleanTarget& target,
        bool in_table_zone,
        bool is_cleanable,
        const CleaningPriorityWeightConfig& weights
    ) const = 0;

    virtual ~BaseCleaningPriorityEngine() = default;
};
```

---

# 9.21 C++ — `BaseClutterGroupingEngine`

```cpp
class BaseClutterGroupingEngine
{
public:
    virtual std::unordered_map<std::string, std::vector<CleaningTaskResult>> group(
        const std::vector<CleaningTaskResult>& results
    ) const = 0;

    virtual ~BaseClutterGroupingEngine() = default;
};
```

---

# 9.22 C++ — `BaseCleaningTaskOrderingEngine`

```cpp
class BaseCleaningTaskOrderingEngine
{
public:
    virtual std::vector<CleaningTaskResult> order(
        const std::vector<CleaningTaskResult>& results
    ) const = 0;

    virtual ~BaseCleaningTaskOrderingEngine() = default;
};
```

---

# 9.23 C++ — `BaseCleaningSummaryEngine`

```cpp
class BaseCleaningSummaryEngine
{
public:
    virtual std::string summarize(
        const std::vector<CleaningTaskResult>& ordered_results
    ) const = 0;

    virtual ~BaseCleaningSummaryEngine() = default;
};
```

---

# 9.24 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `TargetZoneClassifier`
- `ReachabilityInspector`
- `TableZoneInspector`
- `CleanabilityInspector`
- `CleaningPriorityEngine`
- `ClutterGroupingEngine`
- `CleaningTaskOrderingEngine`
- `CleaningSummaryEngine`
- `TableCleaningPerceptionTaskScheduler`
- `CleaningReportWriter`

---

# 9.25 C++ — `TableCleaningPerceptionTaskScheduler`

Tạo class trung tâm:

```cpp
class TableCleaningPerceptionTaskScheduler
```

## Thuộc tính

```cpp
private:
    std::vector<CleanTarget> targets;

    RobotZoneConfig zone_config;
    ReachabilityThresholdConfig threshold_config;
    TableZoneRuleConfig table_zone_rule_config;
    CleanabilityRuleConfig cleanability_rule_config;
    CleaningPriorityWeightConfig cleaning_priority_weight_config;

    std::shared_ptr<BaseTargetZoneClassifier> zone_classifier;
    std::shared_ptr<BaseReachabilityInspector> reachability_inspector;
    std::shared_ptr<BaseTableZoneInspector> table_zone_inspector;
    std::shared_ptr<BaseCleanabilityInspector> cleanability_inspector;
    std::shared_ptr<BaseCleaningPriorityEngine> priority_engine;
    std::shared_ptr<BaseClutterGroupingEngine> grouping_engine;
    std::shared_ptr<BaseCleaningTaskOrderingEngine> ordering_engine;
    std::shared_ptr<BaseCleaningSummaryEngine> summary_engine;

    std::vector<CleaningTaskResult> results;
    std::stack<CleaningDebugRecord> debug_history;
```

## Hàm cần có

### `load_clean_scene_config(const std::string& path)`
### `load_robot_zone_config(const std::string& path)`
### `load_reachability_threshold_config(const std::string& path)`
### `load_table_zone_rule_config(const std::string& path)`
### `load_cleanability_rule_config(const std::string& path)`
### `load_cleaning_priority_weight_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
group targets by scene_id

for each scene:
    inspect every target
    compute cleaning priority
    build CleaningTaskResult
    group clutter
    order tasks
    append ordered results
    push debug history
```

### `const std::vector<CleaningTaskResult>& get_results() const`
### `std::vector<CleaningDebugRecord> get_debug_history_reverse()`

---

# 9.26 C++ — `CleaningReportWriter`

Tạo class:

```cpp
class CleaningReportWriter
```

## Hàm cần có

### `write_cleaning_task_report(...)`
Ví dụ:

```text
[Cleaning Task]
Scene: clean_scene_01
Target: wrapper_03
Zone: CENTER_ZONE
Reachable: YES
Table Zone: YES
Cleanable: YES
Priority Score: 10.42
Task Order: 1
```

### `write_cleaning_order_report(...)`
Ví dụ:

```text
[Cleaning Order]
Scene: clean_scene_01
1. wrapper_03 — priority 10.42
2. tissue_01 — priority 9.31
3. cup_04 — priority 7.80
```

### `write_cleaning_schedule_summary_report(...)`
Ví dụ:

```text
[Cleaning Summary]
Scene: clean_scene_01
Target wrapper_03 is scheduled first because it is reachable, lies in a valid table sector,
belongs to a high-priority clutter cluster, and has the strongest cleaning priority score.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.27 C++ — `main.cpp`

## Yêu cầu

```text
Load clean scene config
Load robot zone config
Load reachability threshold config
Load table zone rule config
Load cleanability rule config
Load cleaning priority weight config
Load visualization config
Load runtime config

Create:
    TargetZoneClassifier
    ReachabilityInspector
    TableZoneInspector
    CleanabilityInspector
    CleaningPriorityEngine
    ClutterGroupingEngine
    CleaningTaskOrderingEngine
    CleaningSummaryEngine

Create TableCleaningPerceptionTaskScheduler
Run
Write reports
```

---

# 10. Luật cleaning task scheduler của project

## Luật 1 — Bài này chỉ tập trung vào TABLE_CLEAN_MODE
Không trộn follow / pick vào logic chính.

## Luật 2 — Phải tách `in_table_zone`, `is_cleanable`, `is_reachable`
Một target có thể nằm trên bàn nhưng không cleanable, hoặc cleanable nhưng quá xa.

## Luật 3 — Cleaning priority và task ordering phải là 2 lớp khác nhau
Không gộp thành một bước duy nhất.

## Luật 4 — Schedule cuối phải usable cho cleaning planner / manipulation planner
Tức là sau bài này bạn phải trả lời được:
- object nào dọn trước
- object nào dọn sau
- cluster / sector nào cần ưu tiên
- task backup order là gì

## Luật 5 — Report phải mang tinh thần “table-cleaning perception”
Không chỉ là ranking target chung chung.

---

# 11. Output mong muốn

## Config
```text
config/clean_scene_config.txt
config/robot_zone_config.txt
config/reachability_threshold_config.txt
config/table_zone_rule_config.txt
config/cleanability_rule_config.txt
config/cleaning_priority_weight_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/cleaning_task_report.txt
assets/outputs/cleaning_order_report.txt
assets/outputs/cleaning_schedule_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/cleaning_priority_plot.png
assets/outputs/clutter_group_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build clean-scene configs
- preview table-zone validity / cleanability / cleaning order
- visualize cleaning schedule

## C++
- đóng vai **table-cleaning perception scheduler**
- inspect clean targets
- group clutter
- rank cleaning tasks
- xuất action-ready cleaning order

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Trong chế độ dọn bàn, robot nên dọn object nào trước, object nào sau, và lịch xử lý nên sắp thế nào?**

Pipeline lúc này sẽ là:

```text
task-aware target selection
→ clean-mode target filtering
→ cleaning priority scoring
→ clutter grouping
→ cleaning task ordering
→ cleaning schedule plan
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho clean scene / zone / threshold / table rules / cleanability / cleaning weights / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / target ids / cleaning result ids
- [ ] Python có NumPy cleaning preview
- [ ] C++ có `CleanTarget`
- [ ] C++ có `RobotZoneConfig`
- [ ] C++ có `ReachabilityThresholdConfig`
- [ ] C++ có `TableZoneRuleConfig`
- [ ] C++ có `CleanabilityRuleConfig`
- [ ] C++ có `CleaningPriorityWeightConfig`
- [ ] C++ có `CleaningTaskResult`
- [ ] C++ có `TargetZoneClassifier`
- [ ] C++ có `ReachabilityInspector`
- [ ] C++ có `TableZoneInspector`
- [ ] C++ có `CleanabilityInspector`
- [ ] C++ có `CleaningPriorityEngine`
- [ ] C++ có `ClutterGroupingEngine`
- [ ] C++ có `CleaningTaskOrderingEngine`
- [ ] C++ có `CleaningSummaryEngine`
- [ ] C++ có `TableCleaningPerceptionTaskScheduler`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 68**, nếu bạn muốn giữ đúng tinh thần Đợt 14 và bắt đầu gom lại 3 nhánh follow / pick / clean thành một hệ lớn hơn, bài hợp lý tiếp theo sẽ là:

# **Bài 69: Humanoid Multi-Mode Perception Mission Orchestrator**

Ý tưởng:
```text
follow-mode planner outputs
+ pick-mode planner outputs
+ clean-mode planner outputs
→ mode-aware mission arbitration
→ selected humanoid perception mission plan
→ mission orchestration report
```

Tức là mạch sẽ đi:

```text
Bài 65: Humanoid Interaction Target Selector
→ Bài 66: Humanoid Object Following Target Tracker Planner
→ Bài 67: Pick-and-Place Vision Candidate Planner
→ Bài 68: Table-Cleaning Perception Task Scheduler
→ Bài 69: Humanoid Multi-Mode Perception Mission Orchestrator
```
