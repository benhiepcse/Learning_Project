# 🤖 Bài 65: Humanoid Interaction Target Selector — Bộ chọn mục tiêu tương tác cho mini humanoid perception pipeline

> Mini Project số 65 trong **Đợt 14**  
> **Bài 65 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 64**.
>
> Nếu:
>
> - **Bài 62** đã đưa điểm từ **camera frame → robot frame**
> - **Bài 63** đã kiểm tra **một target** có hợp lệ / reachable không
> - **Bài 64** đã **xếp hạng nhiều target** trong robot frame
>
> thì **Đợt 14** phải nâng từ “robot nhìn và chấm điểm target” lên **“robot chọn target để hành động trong một mini pipeline humanoid”**.
>
> Vì Đợt 14 thêm các mảnh:
>
> - `Object Following`
> - `Pick and Place Vision`
> - `Table Cleaning Robot`
> - `Humanoid Robot Perception`
>
> nên **Bài 65** hợp logic nhất sẽ là:
>
> ```text
> ranked robot-frame targets
> + task mode (follow / pick / clean)
> + task-specific spatial rules
> + target type / confidence / zone / reachability
> → interaction-ready target selection
> → humanoid mini perception decision report
> ```
>
> Đây là bước mà bạn không chỉ biết **target nào tốt nhất về mặt hình học**, mà còn phải trả lời:
>
> - nếu robot đang ở mode **object following** thì nên chọn target nào?
> - nếu robot đang ở mode **pick and place** thì target nào nằm trong vùng gắp thuận lợi nhất?
> - nếu robot đang ở mode **table cleaning** thì target rác / vật cản nào cần xử lý trước?
>
> Tức là dữ liệu bắt đầu usable cho:
> - task-aware target selection
> - humanoid mini perception decision
> - interaction-ready candidate filtering
> - bridge từ perception sang hành động cấp cao

---

# 📌 Mục lục

- [1. Đợt 14 thêm gì mới so với Đợt 13](#1-đợt-14-thêm-gì-mới-so-với-đợt-13)
- [2. Vì sao Bài 65 xuất hiện sau Bài 64](#2-vì-sao-bài-65-xuất-hiện-sau-bài-64)
- [3. Bài 65 nâng từ Bài 64 lên chỗ nào](#3-bài-65-nâng-từ-bài-64-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Mini humanoid pipeline tổng thể](#6-mini-humanoid-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật interaction target selection của project](#11-luật-interaction-target-selection-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 14 thêm gì mới so với Đợt 13

## Đợt 13 tập trung vào gì?
Đợt 13 tập trung vào **robot-frame perception**:
- `Robot Perception Pipeline`
- `Camera to Robot Coordinate`

và chuỗi project gần nhất của bạn đang đi tới:

```text
camera/depth → robot-frame point → target localization → multi-target ranking
```

## Đợt 14 thêm gì?
Đợt 14 là bước **gom toàn bộ** và đưa perception sang ngữ cảnh **mini humanoid task pipeline**.

### Computer Vision / Robotics Vision
- `Object Following`
- `Pick and Place Vision`
- `Table Cleaning Robot`
- `Humanoid Robot Perception`

### Python
- ôn + áp dụng toàn bộ:
  - OOP
  - file handling
  - list / dict / graph / BST
  - NumPy / Matplotlib
  - modules / config builder

### C++
- ôn + áp dụng toàn bộ:
  - OOP / inheritance / virtual
  - enum / vector / smart pointer
  - project architecture
  - ranking / filtering / runtime modules

Tức là từ Đợt 14 trở đi, project không chỉ hỏi **“target nào tốt?”** mà hỏi **“target nào phù hợp với nhiệm vụ humanoid hiện tại?”**

---

# 2. Vì sao Bài 65 xuất hiện sau Bài 64

## Bài 64 đang làm gì?
Bài 64 đã cho bạn một **multi-target spatial prioritizer**:
- nhiều target trong robot frame
- zone classification
- reachability checking
- priority scoring
- ranking
- best target candidate

## Nhưng còn thiếu gì?
Bài 64 vẫn thiên về **xếp hạng hình học chung**.  
Nó chưa phân biệt rõ **robot đang làm nhiệm vụ gì**.

Ví dụ:
- Với **object following**, robot nên ưu tiên target nằm **trước mặt**, ổn định, dễ theo dõi.
- Với **pick and place**, robot nên ưu tiên target nằm **trong vùng với tay/gắp**, không quá thấp hay quá xa.
- Với **table cleaning**, robot nên ưu tiên target nằm **trên mặt bàn**, reachable, và có thể dọn theo thứ tự hợp lý.

## Bài 65 lấp đúng chỗ đó
Bài 65 sẽ làm:

```text
ranked targets
+ task mode
+ task-specific spatial rules
+ optional target label/type
→ interaction-ready filtering
→ final target selection
→ humanoid decision summary
```

Đây là bước chuyển từ **generic target ranking** sang **task-aware humanoid interaction target selection**.

---

# 3. Bài 65 nâng từ Bài 64 lên chỗ nào

## Bài 64
- so sánh nhiều target
- tính priority score chung
- chọn top target theo score

## Bài 65
- thêm **task mode**
  - `FOLLOW_MODE`
  - `PICK_MODE`
  - `TABLE_CLEAN_MODE`
- thêm **task-specific constraints**
  - follow target nên nằm trước robot, không lệch ngang quá nhiều
  - pick target nên nằm trong vùng thao tác / height phù hợp
  - cleaning target nên nằm trong table zone
- lọc / re-score target theo **ngữ cảnh hành động**
- chọn **interaction target cuối cùng**

### Nói ngắn gọn:
- **Bài 64** hỏi: “Target nào tốt nhất theo xếp hạng hình học tổng quát?”
- **Bài 65** hỏi: “Với nhiệm vụ humanoid hiện tại, robot nên tương tác với target nào?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Humanoid Interaction Target Selector**

System này nhận đầu vào là:
- một hoặc nhiều **scene target sets**
- mỗi scene có nhiều **robot-frame targets**
- một **task mode** cho từng scene:
  - `FOLLOW_MODE`
  - `PICK_MODE`
  - `TABLE_CLEAN_MODE`
- một bộ **zone / reachability / priority config**
- một bộ **task-specific interaction rules**
- optional:
  - target label / target type
  - confidence
  - table-zone tag
  - pickability tag

## Mỗi target sample tối thiểu chứa
- `pair_id`
- `target_id`
- `robot_x`
- `robot_y`
- `robot_z`
- `task_mode`
- optional:
  - `target_label`
  - `target_type`
  - `confidence`
  - `table_zone`
  - `pickable_flag`

## Nhiệm vụ của system
### Với mỗi scene:
1. load toàn bộ targets
2. đọc `task_mode`
3. với từng target:
   - classify zone
   - inspect reachability
   - evaluate validity
   - compute generic spatial score
   - compute task-specific interaction score
4. filter targets không phù hợp với mode hiện tại
5. rank các target còn lại
6. chọn **final interaction target**
7. ghi **humanoid interaction target selection report**

<p align="center">
  <img src="../../images/project_65.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build humanoid scene config
→ build task-mode config
→ build interaction rule config
→ preview task-aware target ranking bằng NumPy
→ visualize target selection summary

C++
→ load multi-target humanoid scenes
→ inspect / score targets theo task mode
→ select interaction-ready target
→ export humanoid perception decision reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **robot-frame target reasoning** với **nhiệm vụ humanoid cụ thể**
- biết cách thiết kế **task-aware target selection**
- biết cách tách:
  - generic spatial priority
  - task-specific interaction priority
- chuẩn bị nền trực tiếp cho:
  - object following target selection
  - pick-and-place candidate selection
  - table-cleaning target scheduling

---

# 6. Mini humanoid pipeline tổng thể

```text
Load Scene Target Config
Load Robot Zone Config
Load Reachability Threshold Config
Load Generic Priority Weight Config
Load Interaction Rule Config
Load Visualization Config
Load Runtime Config

Create TargetZoneClassifier
Create ReachabilityInspector
Create TargetValidityEngine
Create SpatialPriorityEngine
Create TaskAwareInteractionScoreEngine
Create InteractionTargetRankingEngine
Create HumanoidInteractionSummaryEngine
Create HumanoidInteractionTargetSelector

For each scene:
    1. load all robot-frame targets
    2. read task mode
    3. inspect every target
    4. compute spatial score
    5. compute task-aware interaction score
    6. filter unsuitable targets
    7. rank targets
    8. choose final interaction target
    9. write report

After all scenes:
    write reports
    write plots
```

---

# 7. Kiến thức cần

## Python
- OOP
- file handling
- modules / requirements
- list / dict / set / tuple
- Graph / BST
- NumPy
- Matplotlib

## C++
- OOP / inheritance / abstract class
- vector / enum / smart pointer
- ranking / filtering logic
- project structure `.hpp / .cpp`

## Computer Vision / Robotics Vision
- robot-frame target localization
- multi-target prioritization
- object following target selection
- pick-and-place spatial reasoning
- table-cleaning perception logic
- humanoid mini perception pipeline

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **humanoid interaction target selection pipeline**:

```text
SceneTargets
→ TargetInspection
→ TaskModeRuleCheck
→ SpatialPriorityScoring
→ InteractionScoreScoring
→ TargetRanking
→ FinalInteractionTarget
→ Report
```

## 2. Python BST
Lưu:
- `pair_id`
- `target_id`
- `interaction_result_id`

## 3. `std::vector<HumanoidTarget>`
Danh sách target trong một scene.

## 4. `std::vector<InteractionTargetResult>`
Danh sách kết quả target sau khi inspect / score.

## 5. `std::unordered_map<std::string, std::vector<HumanoidTarget>>`
Group target theo `pair_id` hoặc `scene_id`.

## 6. `std::stack<InteractionDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Generic spatial inspection
Bạn phải tái sử dụng logic từ Bài 63–64:
- zone classification
- reachability inspection
- validity checking
- spatial priority score

---

## Algorithm 2 — Task mode dispatch
Mỗi scene phải có một mode:
- `FOLLOW_MODE`
- `PICK_MODE`
- `TABLE_CLEAN_MODE`

Engine phải rẽ nhánh theo mode để chấm điểm target khác nhau.

---

## Algorithm 3 — Task-aware interaction scoring

### A. `FOLLOW_MODE`
Gợi ý rule:
- thưởng target ở **CENTER_ZONE**
- thưởng target có `forward_distance` vừa phải
- phạt target lệch ngang quá nhiều
- thưởng target có confidence cao

### B. `PICK_MODE`
Gợi ý rule:
- target phải reachable
- target nên ở vùng trước robot
- target nên có `robot_z` nằm trong dải height cho thao tác gắp
- nếu có `pickable_flag = false` thì phạt mạnh hoặc loại

### C. `TABLE_CLEAN_MODE`
Gợi ý rule:
- target nên nằm trong `table_zone`
- target phải reachable
- có thể ưu tiên target gần trước, rồi target gần center
- optional: ưu tiên target “rác” hơn target “đồ vật không cần dọn”

---

## Algorithm 4 — Filtering
Tùy mode, một target có thể bị loại nếu:
- nằm sau robot
- không reachable
- sai zone
- sai height range
- không thuộc table zone
- không pickable

---

## Algorithm 5 — Final interaction ranking
Sau khi có:
- spatial score
- interaction score
- validity / reachability

hãy tạo **final score** và rank toàn bộ target.

Ví dụ:
```text
final_score = spatial_score_weight * spatial_score
            + interaction_score_weight * interaction_score
            + confidence_bonus
```

---

## Algorithm 6 — Humanoid decision summary
Report cuối phải trả lời được:
- scene đang ở mode nào?
- target nào được chọn?
- vì sao nó được chọn?
- target backup là ai?

---

## Algorithm 7 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- final target score chart
- task-mode selection summary
- scene-wise chosen target report

---

# 9. Cấu trúc folder

```text
mini_project_65_humanoid_interaction_target_selector/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ humanoid_scene_inputs/
│  │  ├─ scene_01_targets.txt
│  │  ├─ scene_02_targets.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ interaction_target_report.txt
│     ├─ interaction_ranking_report.txt
│     ├─ humanoid_decision_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ final_target_score_plot.png
│     └─ task_mode_summary_plot.png
│
├─ config/
│  ├─ humanoid_scene_target_config.txt
│  ├─ robot_zone_config.txt
│  ├─ reachability_threshold_config.txt
│  ├─ spatial_priority_weight_config.txt
│  ├─ interaction_rule_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ humanoid_selection_graph_preview.py
│     ├─ humanoid_selection_bst.py
│     ├─ numpy_humanoid_selection_preview.py
│     ├─ matplotlib_humanoid_selection_plotter.py
│     └─ synthetic_humanoid_scene_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ HumanoidTarget.hpp
   │  ├─ TaskMode.hpp
   │  ├─ RobotZoneConfig.hpp
   │  ├─ ReachabilityThresholdConfig.hpp
   │  ├─ SpatialPriorityWeightConfig.hpp
   │  ├─ InteractionRuleConfig.hpp
   │  ├─ TargetZone.hpp
   │  ├─ InteractionTargetResult.hpp
   │  ├─ InteractionDebugRecord.hpp
   │  ├─ BaseTargetZoneClassifier.hpp
   │  ├─ TargetZoneClassifier.hpp
   │  ├─ BaseReachabilityInspector.hpp
   │  ├─ ReachabilityInspector.hpp
   │  ├─ BaseTargetValidityEngine.hpp
   │  ├─ TargetValidityEngine.hpp
   │  ├─ BaseSpatialPriorityEngine.hpp
   │  ├─ SpatialPriorityEngine.hpp
   │  ├─ BaseTaskAwareInteractionScoreEngine.hpp
   │  ├─ TaskAwareInteractionScoreEngine.hpp
   │  ├─ BaseInteractionTargetRankingEngine.hpp
   │  ├─ InteractionTargetRankingEngine.hpp
   │  ├─ BaseHumanoidInteractionSummaryEngine.hpp
   │  ├─ HumanoidInteractionSummaryEngine.hpp
   │  ├─ HumanoidInteractionTargetSelector.hpp
   │  └─ HumanoidInteractionReportWriter.hpp
   │
   └─ src/
      ├─ TargetZoneClassifier.cpp
      ├─ ReachabilityInspector.cpp
      ├─ TargetValidityEngine.cpp
      ├─ SpatialPriorityEngine.cpp
      ├─ TaskAwareInteractionScoreEngine.cpp
      ├─ InteractionTargetRankingEngine.cpp
      ├─ HumanoidInteractionSummaryEngine.cpp
      ├─ HumanoidInteractionTargetSelector.cpp
      └─ HumanoidInteractionReportWriter.cpp
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
humanoid_scene_target_config_path
robot_zone_config_path
reachability_threshold_config_path
spatial_priority_weight_config_path
interaction_rule_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `HumanoidInteractionConfigBuilder`

Tạo class con:

```python
class HumanoidInteractionConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_target(
    pair_id,
    target_id,
    robot_x,
    robot_y,
    robot_z,
    task_mode,
    target_label=None,
    target_type=None,
    confidence=None,
    table_zone=None,
    pickable_flag=True
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

### `set_spatial_priority_weights(
    zone_weight,
    reachability_weight,
    validity_weight,
    forward_weight,
    lateral_weight
)`

### `set_interaction_rules(
    follow_center_bonus,
    follow_lateral_penalty,
    pick_height_bonus,
    pick_non_pickable_penalty,
    clean_table_zone_bonus
)`

### `set_visualization_options(
    enable_final_target_score_plot,
    enable_task_mode_summary_plot
)`

### `write_humanoid_scene_target_config()`
### `write_robot_zone_config()`
### `write_reachability_threshold_config()`
### `write_spatial_priority_weight_config()`
### `write_interaction_rule_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `humanoid_selection_graph_preview.py`

Tạo class:

```python
class HumanoidSelectionGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
SceneTargets
→ TargetInspection
→ TaskModeRuleCheck
→ SpatialPriorityScoring
→ InteractionScoreScoring
→ TargetRanking
→ FinalInteractionTarget
→ Report
```

---

# 10.4 Python — `humanoid_selection_bst.py`

Tạo BST cho:
- `pair_id`
- `target_id`
- `interaction_result_id`

---

# 10.5 Python — `numpy_humanoid_selection_preview.py`

Tạo class:

```python
class NumPyHumanoidSelectionPreview:
```

## Hàm cần có

### `load_scene_targets(path)`
### `classify_targets(targets, zone_config)`
### `compute_spatial_scores(targets, spatial_weight_config, threshold_config)`
### `compute_task_scores(targets, task_mode, interaction_rule_config)`
### `build_final_ranking(results)`

---

# 10.6 Python — `matplotlib_humanoid_selection_plotter.py`

Tạo class:

```python
class MatplotlibHumanoidSelectionPlotter:
```

## Hàm cần có

### `plot_final_target_scores(results, save_path)`
### `plot_task_mode_summary(results, save_path)`

---

# 10.7 C++ — `TaskMode`

```cpp
enum class TaskMode
{
    FOLLOW_MODE,
    PICK_MODE,
    TABLE_CLEAN_MODE
};
```

---

# 10.8 C++ — `HumanoidTarget`

```cpp
struct HumanoidTarget
{
    std::string pair_id;
    std::string target_id;

    double robot_x;
    double robot_y;
    double robot_z;

    TaskMode task_mode;

    std::string target_label;
    std::string target_type;
    double confidence;
    bool table_zone;
    bool pickable_flag;
};
```

---

# 10.9 C++ — `RobotZoneConfig`

```cpp
struct RobotZoneConfig
{
    double center_band;
    double left_right_boundary;
};
```

---

# 10.10 C++ — `ReachabilityThresholdConfig`

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

# 10.11 C++ — `SpatialPriorityWeightConfig`

```cpp
struct SpatialPriorityWeightConfig
{
    double zone_weight;
    double reachability_weight;
    double validity_weight;
    double forward_weight;
    double lateral_weight;
};
```

---

# 10.12 C++ — `InteractionRuleConfig`

```cpp
struct InteractionRuleConfig
{
    double follow_center_bonus;
    double follow_lateral_penalty;

    double pick_height_bonus;
    double pick_non_pickable_penalty;

    double clean_table_zone_bonus;
};
```

---

# 10.13 C++ — `TargetZone`

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

# 10.14 C++ — `InteractionTargetResult`

```cpp
struct InteractionTargetResult
{
    std::string pair_id;
    std::string target_id;

    TaskMode task_mode;
    TargetZone zone;

    bool is_reachable;
    bool is_valid;
    bool passes_task_rules;

    double forward_distance;
    double lateral_distance;
    double vertical_distance;

    double spatial_score;
    double interaction_score;
    double final_score;

    int rank_position;
    std::string summary;
};
```

---

# 10.15 C++ — `InteractionDebugRecord`

```cpp
struct InteractionDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.16 C++ — `BaseTargetZoneClassifier`

```cpp
class BaseTargetZoneClassifier
{
public:
    virtual TargetZone classify(
        const HumanoidTarget& target,
        const RobotZoneConfig& config
    ) const = 0;

    virtual ~BaseTargetZoneClassifier() = default;
};
```

---

# 10.17 C++ — `BaseReachabilityInspector`

```cpp
class BaseReachabilityInspector
{
public:
    virtual bool inspect(
        const HumanoidTarget& target,
        const ReachabilityThresholdConfig& config
    ) const = 0;

    virtual ~BaseReachabilityInspector() = default;
};
```

---

# 10.18 C++ — `BaseTargetValidityEngine`

```cpp
class BaseTargetValidityEngine
{
public:
    virtual bool evaluate(
        TargetZone zone,
        bool is_reachable
    ) const = 0;

    virtual ~BaseTargetValidityEngine() = default;
};
```

---

# 10.19 C++ — `BaseSpatialPriorityEngine`

```cpp
class BaseSpatialPriorityEngine
{
public:
    virtual double compute(
        const HumanoidTarget& target,
        TargetZone zone,
        bool is_reachable,
        bool is_valid,
        const SpatialPriorityWeightConfig& weights
    ) const = 0;

    virtual ~BaseSpatialPriorityEngine() = default;
};
```

---

# 10.20 C++ — `BaseTaskAwareInteractionScoreEngine`

```cpp
class BaseTaskAwareInteractionScoreEngine
{
public:
    virtual std::pair<bool, double> compute(
        const HumanoidTarget& target,
        const InteractionRuleConfig& rules
    ) const = 0;

    virtual ~BaseTaskAwareInteractionScoreEngine() = default;
};
```

Giá trị trả về:
- `first`  → target có qua được task rule hay không
- `second` → interaction score

---

# 10.21 C++ — `BaseInteractionTargetRankingEngine`

```cpp
class BaseInteractionTargetRankingEngine
{
public:
    virtual std::vector<InteractionTargetResult> rank(
        const std::vector<InteractionTargetResult>& results
    ) const = 0;

    virtual ~BaseInteractionTargetRankingEngine() = default;
};
```

---

# 10.22 C++ — `BaseHumanoidInteractionSummaryEngine`

```cpp
class BaseHumanoidInteractionSummaryEngine
{
public:
    virtual std::string summarize(
        const std::vector<InteractionTargetResult>& ranked_results
    ) const = 0;

    virtual ~BaseHumanoidInteractionSummaryEngine() = default;
};
```

---

# 10.23 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `TargetZoneClassifier`
- `ReachabilityInspector`
- `TargetValidityEngine`
- `SpatialPriorityEngine`
- `TaskAwareInteractionScoreEngine`
- `InteractionTargetRankingEngine`
- `HumanoidInteractionSummaryEngine`
- `HumanoidInteractionTargetSelector`
- `HumanoidInteractionReportWriter`

---

# 10.24 C++ — `HumanoidInteractionTargetSelector`

Tạo class trung tâm:

```cpp
class HumanoidInteractionTargetSelector
```

## Thuộc tính

```cpp
private:
    std::vector<HumanoidTarget> targets;

    RobotZoneConfig zone_config;
    ReachabilityThresholdConfig threshold_config;
    SpatialPriorityWeightConfig spatial_weight_config;
    InteractionRuleConfig interaction_rule_config;

    std::shared_ptr<BaseTargetZoneClassifier> zone_classifier;
    std::shared_ptr<BaseReachabilityInspector> reachability_inspector;
    std::shared_ptr<BaseTargetValidityEngine> validity_engine;
    std::shared_ptr<BaseSpatialPriorityEngine> spatial_priority_engine;
    std::shared_ptr<BaseTaskAwareInteractionScoreEngine> interaction_score_engine;
    std::shared_ptr<BaseInteractionTargetRankingEngine> ranking_engine;
    std::shared_ptr<BaseHumanoidInteractionSummaryEngine> summary_engine;

    std::vector<InteractionTargetResult> results;
    std::stack<InteractionDebugRecord> debug_history;
```

## Hàm cần có

### `load_humanoid_scene_target_config(const std::string& path)`
### `load_robot_zone_config(const std::string& path)`
### `load_reachability_threshold_config(const std::string& path)`
### `load_spatial_priority_weight_config(const std::string& path)`
### `load_interaction_rule_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
group targets by pair_id

for each scene:
    inspect every target
    compute spatial score
    compute task-aware interaction score
    build InteractionTargetResult
    rank results in scene
    append ranked results
    push debug history
```

### `const std::vector<InteractionTargetResult>& get_results() const`
### `std::vector<InteractionDebugRecord> get_debug_history_reverse()`

---

# 10.25 C++ — `HumanoidInteractionReportWriter`

Tạo class:

```cpp
class HumanoidInteractionReportWriter
```

## Hàm cần có

### `write_interaction_target_report(...)`
Ví dụ:

```text
[Interaction Target]
Pair: scene_01
Task Mode: PICK_MODE
Target: mug_02
Zone: CENTER_ZONE
Reachable: YES
Valid: YES
Task Rule Pass: YES
Final Score: 12.48
Rank: 1
```

### `write_interaction_ranking_report(...)`
Ví dụ:

```text
[Interaction Ranking]
Pair: scene_01
1. mug_02 — final score 12.48
2. bottle_01 — final score 9.82
3. spoon_03 — final score 7.10
```

### `write_humanoid_decision_summary_report(...)`
Ví dụ:

```text
[Humanoid Decision Summary]
Pair: scene_01
The robot is operating in PICK_MODE.
Target mug_02 was selected because it is reachable, lies near the center zone,
passes pickability rules, and achieves the strongest combined spatial + interaction score.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.26 C++ — `main.cpp`

## Yêu cầu

```text
Load humanoid scene target config
Load robot zone config
Load reachability threshold config
Load spatial priority weight config
Load interaction rule config
Load visualization config
Load runtime config

Create:
    TargetZoneClassifier
    ReachabilityInspector
    TargetValidityEngine
    SpatialPriorityEngine
    TaskAwareInteractionScoreEngine
    InteractionTargetRankingEngine
    HumanoidInteractionSummaryEngine

Create HumanoidInteractionTargetSelector
Run
Write reports
```

---

# 11. Luật interaction target selection của project

## Luật 1 — Bài này phải có **task mode thật**
Không được chỉ rank target chung chung như Bài 64.

## Luật 2 — Mỗi task mode phải có logic riêng
Ít nhất phải phân biệt:
- follow
- pick
- table clean

## Luật 3 — Final target phải vừa qua spatial rule vừa qua task rule
Không được chỉ vì điểm hình học cao mà bỏ qua logic task.

## Luật 4 — Report phải nói rõ robot đang làm nhiệm vụ gì
Và target được chọn vì lý do nào.

## Luật 5 — Project phải mang đúng tinh thần “Humanoid mini pipeline”
Tức là sau bài này bạn phải thấy rõ cầu nối:

```text
robot-frame perception
→ multi-target prioritization
→ task-aware target selection
→ humanoid mini decision
```

---

# 12. Output mong muốn

## Config
```text
config/humanoid_scene_target_config.txt
config/robot_zone_config.txt
config/reachability_threshold_config.txt
config/spatial_priority_weight_config.txt
config/interaction_rule_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/interaction_target_report.txt
assets/outputs/interaction_ranking_report.txt
assets/outputs/humanoid_decision_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/final_target_score_plot.png
assets/outputs/task_mode_summary_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build humanoid scene configs
- build task-specific interaction rules
- preview ranking / selection
- visualize scene-wise chosen targets

## C++
- đóng vai **mini humanoid perception decision runtime**
- inspect targets
- score targets theo mode
- chọn interaction-ready target cuối cùng

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Với nhiệm vụ humanoid hiện tại (follow / pick / clean), robot nên tương tác với target nào?**

Pipeline lúc này sẽ là:

```text
depth / robot-frame points / localized targets
→ multi-target prioritization
→ task-aware interaction selection
→ humanoid mini perception decision
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho humanoid scenes / zone / threshold / weights / interaction rules / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / target ids / interaction result ids
- [ ] Python có NumPy humanoid selection preview
- [ ] C++ có `TaskMode`
- [ ] C++ có `HumanoidTarget`
- [ ] C++ có `RobotZoneConfig`
- [ ] C++ có `ReachabilityThresholdConfig`
- [ ] C++ có `SpatialPriorityWeightConfig`
- [ ] C++ có `InteractionRuleConfig`
- [ ] C++ có `InteractionTargetResult`
- [ ] C++ có `TargetZoneClassifier`
- [ ] C++ có `ReachabilityInspector`
- [ ] C++ có `TargetValidityEngine`
- [ ] C++ có `SpatialPriorityEngine`
- [ ] C++ có `TaskAwareInteractionScoreEngine`
- [ ] C++ có `InteractionTargetRankingEngine`
- [ ] C++ có `HumanoidInteractionSummaryEngine`
- [ ] C++ có `HumanoidInteractionTargetSelector`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 65**, bước hợp lý nhất để tiếp tục Đợt 14 là:

# **Bài 66: Humanoid Object Following Target Tracker Planner**

Ý tưởng:
```text
selected follow target
→ temporal target state buffer
→ follow stability check
→ target update / loss handling
→ following-ready tracking plan
```

Tức là mạch sẽ đi:

```text
Bài 65: Humanoid Interaction Target Selector
→ Bài 66: Humanoid Object Following Target Tracker Planner
→ Bài 67: Pick-and-Place Vision Candidate Planner
→ Bài 68: Table-Cleaning Perception Task Scheduler
```
