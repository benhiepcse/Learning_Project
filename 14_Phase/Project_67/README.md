# 🤖 Bài 67: Pick-and-Place Vision Candidate Planner — Bộ lập kế hoạch candidate cho chế độ pick-and-place vision của humanoid

> Mini Project số 67 trong **Đợt 14**  
> **Bài 67 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 65** nhưng theo nhánh **PICK_MODE** của Đợt 14.
>
> Nếu:
>
> - **Bài 65** đã chọn target theo `FOLLOW_MODE / PICK_MODE / TABLE_CLEAN_MODE`
> - **Bài 66** đã đào sâu riêng nhánh **FOLLOW_MODE**
>
> thì bước tiếp theo hợp lý nhất là đào sâu nhánh **PICK_MODE**:
>
> ```text
> pick-mode targets
> → pickability filtering
> → grasp-friendly zone inspection
> → approach / pickup suitability scoring
> → pick candidate ranking
> → pick-and-place candidate plan
> ```
>
> Đây là bước mà bạn không chỉ hỏi **target nào hợp lệ cho tương tác**, mà phải trả lời sâu hơn:
>
> - target nào thật sự **đáng để gắp**?
> - target nào nằm trong vùng thao tác thuận lợi của humanoid?
> - target nào có height / lateral offset / distance phù hợp hơn cho pick?
> - nếu có nhiều object trên bàn, robot nên ưu tiên object nào cho thao tác gắp trước?
>
> Tức là dữ liệu bắt đầu usable cho:
> - pick-and-place candidate filtering
> - grasp-friendly spatial reasoning
> - pickup priority planning
> - perception-to-manipulation handoff

---

# 📌 Mục lục

- [1. Vì sao Bài 67 xuất hiện sau Bài 65 và Bài 66](#1-vì-sao-bài-67-xuất-hiện-sau-bài-65-và-bài-66)
- [2. Bài 67 nâng từ Bài 65 lên chỗ nào](#2-bài-67-nâng-từ-bài-65-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Pick-and-place vision pipeline tổng thể](#5-pick-and-place-vision-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật pick candidate planner của project](#10-luật-pick-candidate-planner-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 67 xuất hiện sau Bài 65 và Bài 66

## Bài 65 đang làm gì?
Bài 65 đã giúp bạn:
- gom nhiều target trong robot frame
- áp dụng `FOLLOW_MODE / PICK_MODE / TABLE_CLEAN_MODE`
- chọn ra interaction target theo từng mode

## Bài 66 đã đào sâu gì?
Bài 66 đã tách riêng nhánh **FOLLOW_MODE**:
- temporal buffer
- tracking stability
- target loss handling
- following plan

## Còn nhánh nào cần đào sâu?
Nhánh còn lại rất quan trọng của Đợt 14 là **PICK_MODE**:
- target nào **pickable**
- target nào nằm trong **grasp-friendly zone**
- target nào đáng ưu tiên cho thao tác gắp

Đó là lý do **Bài 67** xuất hiện.

---

# 2. Bài 67 nâng từ Bài 65 lên chỗ nào

## Bài 65
- chọn target cho `PICK_MODE` ở mức tổng quát
- có task-aware selection nhưng chưa đào sâu logic gắp

## Bài 67
- chỉ tập trung vào **PICK_MODE**
- kiểm tra **pickability**
- chấm **grasp-friendly score**
- đánh giá:
  - target có nằm trong vùng trước robot không
  - target có ở dải height phù hợp để pick không
  - target có lệch ngang quá nhiều không
  - target có đủ gần / đủ ổn định để pick không
- tạo **pick candidate plan**

### Nói ngắn gọn:
- **Bài 65** hỏi: “Robot nên chọn target nào cho PICK_MODE?”
- **Bài 67** hỏi: “Trong các target của PICK_MODE, target nào là candidate tốt nhất cho thao tác pick-and-place?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Pick-and-Place Vision Candidate Planner**

System này nhận đầu vào là:
- một hoặc nhiều **pick scenes**
- mỗi scene chứa nhiều **pick-mode targets**
- một bộ **pickability rules**
- một bộ **grasp-zone thresholds**
- một bộ **pickup scoring weights**
- optional:
  - object label / type
  - confidence
  - pickable flag
  - surface tag
  - placement group tag

## Mỗi target sample tối thiểu chứa
- `scene_id`
- `target_id`
- `robot_x`
- `robot_y`
- `robot_z`
- `confidence`
- `pickable_flag`

Optional:
- `object_label`
- `object_type`
- `surface_tag`
- `placement_group`

## Nhiệm vụ của system
### Với mỗi pick scene:
1. load toàn bộ targets
2. với từng target:
   - classify zone
   - inspect reachability
   - inspect pickability
   - compute grasp-friendly score
   - compute pickup priority score
3. filter target không phù hợp để pick
4. rank các pick candidates
5. chọn **best pick candidate**
6. ghi **pick-and-place candidate plan report**

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build pick-scene config
→ build grasp thresholds / pickability rules / pickup weights
→ preview candidate ranking bằng NumPy
→ visualize pick candidate summary

C++
→ load pick-mode robot-frame targets
→ inspect pickability / reachability / grasp suitability
→ rank pick candidates
→ select best candidate
→ export pick planning reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **robot-frame target reasoning** với **pick-and-place vision**
- biết cách thiết kế **pickability filtering**
- biết cách tách:
  - spatial validity
  - pickability validity
  - grasp-friendly scoring
  - pickup priority scoring
- chuẩn bị nền trực tiếp cho:
  - manipulation candidate selection
  - grasp pre-check
  - pick-and-place perception planning

---

# 5. Pick-and-place vision pipeline tổng thể

```text
Load Pick Scene Config
Load Robot Zone Config
Load Reachability Threshold Config
Load Pickability Rule Config
Load Grasp Zone Config
Load Pickup Weight Config
Load Visualization Config
Load Runtime Config

Create TargetZoneClassifier
Create ReachabilityInspector
Create PickabilityInspector
Create GraspSuitabilityEngine
Create PickupPriorityEngine
Create PickCandidateRankingEngine
Create PickSummaryEngine
Create PickAndPlaceVisionCandidatePlanner

For each pick scene:
    1. load all pick targets
    2. inspect every target
    3. compute grasp suitability score
    4. compute pickup priority score
    5. filter invalid / unpickable targets
    6. rank candidates
    7. choose best pick candidate
    8. write report

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
- sorting / ranking / filtering logic
- project structure `.hpp / .cpp`

## Computer Vision / Robotics Vision
- robot-frame target localization
- pick-and-place vision
- pickability filtering
- grasp-friendly zone reasoning
- pickup candidate planning

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **pick candidate planning pipeline**:

```text
PickSceneTargets
→ ReachabilityInspection
→ PickabilityInspection
→ GraspSuitabilityScoring
→ PickupPriorityScoring
→ CandidateRanking
→ BestPickCandidate
→ Report
```

## 2. Python BST
Lưu:
- `scene_id`
- `target_id`
- `pick_result_id`

## 3. `std::vector<PickTarget>`
Danh sách target trong một pick scene.

## 4. `std::vector<PickCandidateResult>`
Danh sách kết quả inspect / score cho từng target.

## 5. `std::unordered_map<std::string, std::vector<PickTarget>>`
Group target theo `scene_id`.

## 6. `std::stack<PickDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Spatial inspection reuse
Bạn phải tái sử dụng logic từ Bài 63–65:
- zone classification
- reachability inspection
- validity checking

---

## Algorithm 2 — Pickability inspection
Một target có thể bị loại nếu:
- `pickable_flag = false`
- nằm sau robot
- quá xa / quá lệch
- confidence quá thấp
- object type bị cấm pick theo config

---

## Algorithm 3 — Grasp suitability scoring
Tạo điểm **grasp-friendly score** dựa trên:
- target ở `CENTER_ZONE` hay không
- lateral offset nhỏ
- height gần dải pick lý tưởng
- forward distance không quá xa
- confidence cao

Ví dụ ý tưởng:
```text
grasp_score =
    zone_bonus
  + confidence_bonus
  - lateral_penalty
  - forward_penalty
  - height_offset_penalty
```

---

## Algorithm 4 — Pickup priority scoring
Sau khi có grasp score, hãy tạo **pickup priority score**:
- cộng điểm cho object quan trọng
- cộng điểm cho target gần hơn
- cộng điểm nếu target nằm trên bề mặt hợp lệ
- phạt target khó với tới

---

## Algorithm 5 — Candidate ranking
Sort candidate theo `pickup_priority_score` giảm dần.

Tie-break gợi ý:
1. target có `grasp_score` cao hơn
2. target gần center hơn
3. target gần robot hơn
4. `target_id` nhỏ hơn

---

## Algorithm 6 — Pick summary
Report cuối phải trả lời được:
- target nào là **best pick candidate**
- vì sao nó được chọn
- candidate backup là ai

---

## Algorithm 7 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- pick candidate score chart
- grasp suitability plot
- scene-wise best pick summary

---

# 8. Cấu trúc folder

```text
mini_project_67_pick_and_place_vision_candidate_planner/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ pick_scene_inputs/
│  │  ├─ pick_scene_01.txt
│  │  ├─ pick_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ pick_candidate_report.txt
│     ├─ pick_ranking_report.txt
│     ├─ pick_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ pick_candidate_score_plot.png
│     └─ grasp_suitability_plot.png
│
├─ config/
│  ├─ pick_scene_config.txt
│  ├─ robot_zone_config.txt
│  ├─ reachability_threshold_config.txt
│  ├─ pickability_rule_config.txt
│  ├─ grasp_zone_config.txt
│  ├─ pickup_weight_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ pick_graph_preview.py
│     ├─ pick_bst.py
│     ├─ numpy_pick_preview.py
│     ├─ matplotlib_pick_plotter.py
│     └─ synthetic_pick_scene_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ PickTarget.hpp
   │  ├─ RobotZoneConfig.hpp
   │  ├─ ReachabilityThresholdConfig.hpp
   │  ├─ PickabilityRuleConfig.hpp
   │  ├─ GraspZoneConfig.hpp
   │  ├─ PickupWeightConfig.hpp
   │  ├─ TargetZone.hpp
   │  ├─ PickCandidateResult.hpp
   │  ├─ PickDebugRecord.hpp
   │  ├─ BaseTargetZoneClassifier.hpp
   │  ├─ TargetZoneClassifier.hpp
   │  ├─ BaseReachabilityInspector.hpp
   │  ├─ ReachabilityInspector.hpp
   │  ├─ BasePickabilityInspector.hpp
   │  ├─ PickabilityInspector.hpp
   │  ├─ BaseGraspSuitabilityEngine.hpp
   │  ├─ GraspSuitabilityEngine.hpp
   │  ├─ BasePickupPriorityEngine.hpp
   │  ├─ PickupPriorityEngine.hpp
   │  ├─ BasePickCandidateRankingEngine.hpp
   │  ├─ PickCandidateRankingEngine.hpp
   │  ├─ BasePickSummaryEngine.hpp
   │  ├─ PickSummaryEngine.hpp
   │  ├─ PickAndPlaceVisionCandidatePlanner.hpp
   │  └─ PickReportWriter.hpp
   │
   └─ src/
      ├─ TargetZoneClassifier.cpp
      ├─ ReachabilityInspector.cpp
      ├─ PickabilityInspector.cpp
      ├─ GraspSuitabilityEngine.cpp
      ├─ PickupPriorityEngine.cpp
      ├─ PickCandidateRankingEngine.cpp
      ├─ PickSummaryEngine.cpp
      ├─ PickAndPlaceVisionCandidatePlanner.cpp
      └─ PickReportWriter.cpp
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
pick_scene_config_path
robot_zone_config_path
reachability_threshold_config_path
pickability_rule_config_path
grasp_zone_config_path
pickup_weight_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `PickPlannerConfigBuilder`

Tạo class con:

```python
class PickPlannerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_target(
    scene_id,
    target_id,
    robot_x,
    robot_y,
    robot_z,
    confidence,
    pickable_flag,
    object_label=None,
    object_type=None,
    surface_tag=None,
    placement_group=None
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

### `set_pickability_rules(
    min_pick_confidence,
    allow_non_pickable,
    forbidden_object_types
)`

### `set_grasp_zone_config(
    ideal_pick_height,
    max_height_offset,
    center_zone_bonus
)`

### `set_pickup_weights(
    grasp_weight,
    confidence_weight,
    forward_penalty_weight,
    lateral_penalty_weight,
    object_priority_weight
)`

### `set_visualization_options(
    enable_pick_score_plot,
    enable_grasp_plot
)`

### `write_pick_scene_config()`
### `write_robot_zone_config()`
### `write_reachability_threshold_config()`
### `write_pickability_rule_config()`
### `write_grasp_zone_config()`
### `write_pickup_weight_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `pick_graph_preview.py`

Tạo class:

```python
class PickGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
PickSceneTargets
→ ReachabilityInspection
→ PickabilityInspection
→ GraspSuitabilityScoring
→ PickupPriorityScoring
→ CandidateRanking
→ BestPickCandidate
→ Report
```

---

# 9.4 Python — `pick_bst.py`

Tạo BST cho:
- `scene_id`
- `target_id`
- `pick_result_id`

---

# 9.5 Python — `numpy_pick_preview.py`

Tạo class:

```python
class NumPyPickPreview:
```

## Hàm cần có

### `load_pick_scene(path)`
### `inspect_pickability(targets, pickability_rule_config)`
### `compute_grasp_scores(targets, grasp_zone_config, threshold_config)`
### `compute_pick_priority(targets, pickup_weight_config)`
### `build_pick_ranking(results)`

---

# 9.6 Python — `matplotlib_pick_plotter.py`

Tạo class:

```python
class MatplotlibPickPlotter:
```

## Hàm cần có

### `plot_pick_candidate_scores(results, save_path)`
### `plot_grasp_suitability(results, save_path)`

---

# 9.7 C++ — `PickTarget`

```cpp
struct PickTarget
{
    std::string scene_id;
    std::string target_id;

    double robot_x;
    double robot_y;
    double robot_z;

    double confidence;
    bool pickable_flag;

    std::string object_label;
    std::string object_type;
    std::string surface_tag;
    std::string placement_group;
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

# 9.10 C++ — `PickabilityRuleConfig`

```cpp
struct PickabilityRuleConfig
{
    double min_pick_confidence;
    bool allow_non_pickable;
    std::vector<std::string> forbidden_object_types;
};
```

---

# 9.11 C++ — `GraspZoneConfig`

```cpp
struct GraspZoneConfig
{
    double ideal_pick_height;
    double max_height_offset;
    double center_zone_bonus;
};
```

---

# 9.12 C++ — `PickupWeightConfig`

```cpp
struct PickupWeightConfig
{
    double grasp_weight;
    double confidence_weight;
    double forward_penalty_weight;
    double lateral_penalty_weight;
    double object_priority_weight;
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

# 9.14 C++ — `PickCandidateResult`

```cpp
struct PickCandidateResult
{
    std::string scene_id;
    std::string target_id;

    TargetZone zone;

    bool is_reachable;
    bool is_pickable;
    bool is_valid_candidate;

    double grasp_score;
    double pickup_priority_score;
    int rank_position;

    std::string summary;
};
```

---

# 9.15 C++ — `PickDebugRecord`

```cpp
struct PickDebugRecord
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
        const PickTarget& target,
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
        const PickTarget& target,
        const ReachabilityThresholdConfig& config
    ) const = 0;

    virtual ~BaseReachabilityInspector() = default;
};
```

---

# 9.18 C++ — `BasePickabilityInspector`

```cpp
class BasePickabilityInspector
{
public:
    virtual bool inspect(
        const PickTarget& target,
        const PickabilityRuleConfig& config
    ) const = 0;

    virtual ~BasePickabilityInspector() = default;
};
```

---

# 9.19 C++ — `BaseGraspSuitabilityEngine`

```cpp
class BaseGraspSuitabilityEngine
{
public:
    virtual double compute(
        const PickTarget& target,
        TargetZone zone,
        bool is_reachable,
        bool is_pickable,
        const GraspZoneConfig& grasp_config
    ) const = 0;

    virtual ~BaseGraspSuitabilityEngine() = default;
};
```

---

# 9.20 C++ — `BasePickupPriorityEngine`

```cpp
class BasePickupPriorityEngine
{
public:
    virtual double compute(
        const PickTarget& target,
        double grasp_score,
        const PickupWeightConfig& weights
    ) const = 0;

    virtual ~BasePickupPriorityEngine() = default;
};
```

---

# 9.21 C++ — `BasePickCandidateRankingEngine`

```cpp
class BasePickCandidateRankingEngine
{
public:
    virtual std::vector<PickCandidateResult> rank(
        const std::vector<PickCandidateResult>& results
    ) const = 0;

    virtual ~BasePickCandidateRankingEngine() = default;
};
```

---

# 9.22 C++ — `BasePickSummaryEngine`

```cpp
class BasePickSummaryEngine
{
public:
    virtual std::string summarize(
        const std::vector<PickCandidateResult>& ranked_results
    ) const = 0;

    virtual ~BasePickSummaryEngine() = default;
};
```

---

# 9.23 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `TargetZoneClassifier`
- `ReachabilityInspector`
- `PickabilityInspector`
- `GraspSuitabilityEngine`
- `PickupPriorityEngine`
- `PickCandidateRankingEngine`
- `PickSummaryEngine`
- `PickAndPlaceVisionCandidatePlanner`
- `PickReportWriter`

---

# 9.24 C++ — `PickAndPlaceVisionCandidatePlanner`

Tạo class trung tâm:

```cpp
class PickAndPlaceVisionCandidatePlanner
```

## Thuộc tính

```cpp
private:
    std::vector<PickTarget> targets;

    RobotZoneConfig zone_config;
    ReachabilityThresholdConfig threshold_config;
    PickabilityRuleConfig pickability_rule_config;
    GraspZoneConfig grasp_zone_config;
    PickupWeightConfig pickup_weight_config;

    std::shared_ptr<BaseTargetZoneClassifier> zone_classifier;
    std::shared_ptr<BaseReachabilityInspector> reachability_inspector;
    std::shared_ptr<BasePickabilityInspector> pickability_inspector;
    std::shared_ptr<BaseGraspSuitabilityEngine> grasp_engine;
    std::shared_ptr<BasePickupPriorityEngine> pickup_engine;
    std::shared_ptr<BasePickCandidateRankingEngine> ranking_engine;
    std::shared_ptr<BasePickSummaryEngine> summary_engine;

    std::vector<PickCandidateResult> results;
    std::stack<PickDebugRecord> debug_history;
```

## Hàm cần có

### `load_pick_scene_config(const std::string& path)`
### `load_robot_zone_config(const std::string& path)`
### `load_reachability_threshold_config(const std::string& path)`
### `load_pickability_rule_config(const std::string& path)`
### `load_grasp_zone_config(const std::string& path)`
### `load_pickup_weight_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
group targets by scene_id

for each scene:
    inspect every target
    compute grasp score
    compute pickup priority score
    build PickCandidateResult
    rank scene results
    append ranked results
    push debug history
```

### `const std::vector<PickCandidateResult>& get_results() const`
### `std::vector<PickDebugRecord> get_debug_history_reverse()`

---

# 9.25 C++ — `PickReportWriter`

Tạo class:

```cpp
class PickReportWriter
```

## Hàm cần có

### `write_pick_candidate_report(...)`
Ví dụ:

```text
[Pick Candidate]
Scene: pick_scene_01
Target: mug_02
Zone: CENTER_ZONE
Reachable: YES
Pickable: YES
Grasp Score: 8.42
Pickup Priority Score: 11.05
Rank: 1
```

### `write_pick_ranking_report(...)`
Ví dụ:

```text
[Pick Ranking]
Scene: pick_scene_01
1. mug_02 — priority 11.05
2. bottle_01 — priority 9.44
3. spoon_03 — priority 7.90
```

### `write_pick_summary_report(...)`
Ví dụ:

```text
[Pick Summary]
Scene: pick_scene_01
Target mug_02 was selected as the best pick candidate because it is reachable,
pickable, close to the center grasp corridor, and has the strongest pickup priority score.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.26 C++ — `main.cpp`

## Yêu cầu

```text
Load pick scene config
Load robot zone config
Load reachability threshold config
Load pickability rule config
Load grasp zone config
Load pickup weight config
Load visualization config
Load runtime config

Create:
    TargetZoneClassifier
    ReachabilityInspector
    PickabilityInspector
    GraspSuitabilityEngine
    PickupPriorityEngine
    PickCandidateRankingEngine
    PickSummaryEngine

Create PickAndPlaceVisionCandidatePlanner
Run
Write reports
```

---

# 10. Luật pick candidate planner của project

## Luật 1 — Bài này chỉ tập trung vào PICK_MODE
Không trộn follow / clean vào logic chính.

## Luật 2 — Phải tách `pickable` và `reachable`
Một target có thể reachable nhưng không pickable.

## Luật 3 — Grasp score và pickup priority score phải là 2 lớp khác nhau
Không gộp tất cả vào một score duy nhất ngay từ đầu.

## Luật 4 — Candidate cuối phải usable cho pick planner / manipulation planner
Tức là sau bài này bạn phải trả lời được:
- target nào nên pick trước
- target nào là backup candidate
- vì sao target đó phù hợp cho thao tác gắp

## Luật 5 — Report phải mang tinh thần “pick-and-place vision”
Không chỉ là ranking target chung chung.

---

# 11. Output mong muốn

## Config
```text
config/pick_scene_config.txt
config/robot_zone_config.txt
config/reachability_threshold_config.txt
config/pickability_rule_config.txt
config/grasp_zone_config.txt
config/pickup_weight_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/pick_candidate_report.txt
assets/outputs/pick_ranking_report.txt
assets/outputs/pick_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/pick_candidate_score_plot.png
assets/outputs/grasp_suitability_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build pick-scene configs
- preview pickability / grasp suitability / candidate ranking
- visualize best-pick candidate summary

## C++
- đóng vai **pick candidate planning runtime**
- inspect pickability
- score grasp suitability
- rank pick candidates
- chọn best pick target

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Trong chế độ pick-and-place, object nào là candidate tốt nhất để humanoid gắp trước?**

Pipeline lúc này sẽ là:

```text
task-aware target selection
→ pick-mode target filtering
→ grasp suitability scoring
→ pickup priority ranking
→ best pick candidate plan
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho pick scene / zone / threshold / pickability / grasp / pickup weights / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / target ids / pick result ids
- [ ] Python có NumPy pick preview
- [ ] C++ có `PickTarget`
- [ ] C++ có `RobotZoneConfig`
- [ ] C++ có `ReachabilityThresholdConfig`
- [ ] C++ có `PickabilityRuleConfig`
- [ ] C++ có `GraspZoneConfig`
- [ ] C++ có `PickupWeightConfig`
- [ ] C++ có `PickCandidateResult`
- [ ] C++ có `TargetZoneClassifier`
- [ ] C++ có `ReachabilityInspector`
- [ ] C++ có `PickabilityInspector`
- [ ] C++ có `GraspSuitabilityEngine`
- [ ] C++ có `PickupPriorityEngine`
- [ ] C++ có `PickCandidateRankingEngine`
- [ ] C++ có `PickSummaryEngine`
- [ ] C++ có `PickAndPlaceVisionCandidatePlanner`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 67**, bước hợp lý nhất để tiếp tục Đợt 14 là:

# **Bài 68: Table-Cleaning Perception Task Scheduler**

Ý tưởng:
```text
clean-mode targets
→ table-zone clutter filtering
→ cleaning priority grouping
→ action-ready cleaning task order
→ cleaning schedule report
```

Tức là mạch sẽ đi:

```text
Bài 65: Humanoid Interaction Target Selector
→ Bài 66: Humanoid Object Following Target Tracker Planner
→ Bài 67: Pick-and-Place Vision Candidate Planner
→ Bài 68: Table-Cleaning Perception Task Scheduler
```
