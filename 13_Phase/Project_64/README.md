# 🤖 Bài 64: Multi-Target Robot-Frame Spatial Prioritizer — Bộ ưu tiên mục tiêu không gian trong robot frame cho Humanoid Robot AI Perception

> Mini Project số 64 trong **Đợt 13**  
> **Bài 64 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13** và đi tiếp trực tiếp từ **Bài 63**.
>
> Nếu:
>
> - **Bài 62** giúp bạn biến `pixel + depth` thành `robot-frame point`
> - **Bài 63** giúp bạn kiểm tra **một target** trong robot frame:
>   - target nằm zone nào
>   - target có reachable / valid không
>
> thì **Bài 64** là bước tiếp theo rất tự nhiên:
>
> ```text
> multiple robot-frame targets
> → zone-aware target comparison
> → reachability ranking
> → priority target selection
> → robot-centric target priority report
> ```
>
> Đây là bước mà bạn không còn nhìn **từng target riêng lẻ** nữa, mà bắt đầu xử lý **nhiều target cùng lúc**:
>
> - target nào gần hơn?
> - target nào nằm ở center zone và thuận lợi hơn cho robot?
> - target nào reachable nhưng ưu tiên thấp?
> - nếu có nhiều candidate cùng hợp lệ, robot nên ưu tiên target nào trước?
>
> Tức là dữ liệu bắt đầu usable cho:
> - multi-target spatial reasoning
> - robot-centric target ranking
> - navigation / interaction target prioritization
> - perception-driven target selection

---

# 📌 Mục lục

- [1. Đợt 13 đang đi tiếp từ Bài 63 ra sao](#1-đợt-13-đang-đi-tiếp-từ-bài-63-ra-sao)
- [2. Vì sao Bài 64 xuất hiện sau Bài 63](#2-vì-sao-bài-64-xuất-hiện-sau-bài-63)
- [3. Bài 64 nâng từ Bài 63 lên chỗ nào](#3-bài-64-nâng-từ-bài-63-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật multi-target prioritization của project](#11-luật-multi-target-prioritization-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 13 đang đi tiếp từ Bài 63 ra sao

Ở **Bài 63**, bạn đã có khả năng đánh giá **một target** trong robot frame:

```text
robot-frame point
→ zone classification
→ reachability inspection
→ validity decision
```

Tức là với từng target riêng lẻ, bạn đã biết:
- nó nằm trái / giữa / phải / sau robot
- nó có reachable hay không
- nó có valid cho bước tiếp theo hay không

Nhưng robot ngoài đời hiếm khi chỉ có **1 target**.  
Nó thường phải chọn giữa **nhiều candidate** cùng lúc:

- nhiều obstacle points đáng chú ý
- nhiều candidate object / target
- nhiều điểm có thể đi tới hoặc quan sát

Lúc này robot cần một tầng mới:

> **Nếu có nhiều target hợp lệ trong robot frame, target nào nên được ưu tiên trước?**

Đó là lý do **Bài 64** xuất hiện.

---

# 2. Vì sao Bài 64 xuất hiện sau Bài 63

## Bài 63 đang làm gì?
Bài 63 giúp bạn:
- kiểm tra validity của **một target**
- gán zone cho target
- đánh giá reachability

## Nhưng còn thiếu gì?
Bạn vẫn chưa có cơ chế để:
- so sánh **nhiều target**
- xếp hạng target
- chọn target ưu tiên nhất

Trong perception thực tế, đây là bước rất quan trọng vì robot phải quyết định:
- focus vào target nào
- target nào tốt hơn cho navigation
- target nào tốt hơn cho manipulation / tracking

## Bài 64 lấp đúng chỗ đó
Bài 64 sẽ làm:

```text
multiple robot-frame targets
→ filter invalid targets
→ compare reachable targets
→ rank by zone + distance + priority score
→ choose top-priority target
→ write prioritization report
```

Đây là bước chuyển từ **single-target localization inspection** sang **multi-target spatial prioritization**.

---

# 3. Bài 64 nâng từ Bài 63 lên chỗ nào

## Bài 63
- nhận **1 target**
- gán zone
- đánh giá reachable / valid

## Bài 64
- nhận **nhiều target**
- đánh giá từng target
- so sánh giữa các target
- tính **priority score**
- sắp xếp target theo độ ưu tiên
- chọn **best target candidate**

### Nói ngắn gọn:
- **Bài 63** hỏi: “Target này có hợp lệ không?”
- **Bài 64** hỏi: “Trong tất cả target hợp lệ, target nào đáng ưu tiên nhất?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Multi-Target Robot-Frame Spatial Prioritizer**

System này nhận đầu vào là:
- một hoặc nhiều **target sets**
- mỗi target set chứa **nhiều robot-frame targets**
- một bộ **zone priority config**
- một bộ **distance / reachability thresholds**
- một bộ **scoring weights**
- một bộ **reporting options**

## Mỗi target sample tối thiểu chứa
- `pair_id`
- `target_id`
- `robot_x`
- `robot_y`
- `robot_z`
- optional:
  - `label`
  - `expected_priority`

## Nhiệm vụ của system
### Với mỗi target set:
1. load toàn bộ targets của cùng `pair_id`
2. với từng target:
   - classify zone
   - inspect reachability
   - evaluate validity
   - compute priority score
3. loại bỏ target invalid nếu cần
4. sort / rank các target còn lại
5. chọn **top-priority target**
6. ghi **multi-target prioritization report**

<p align="center">
  <img src="../../images/project_64.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build multi-target sample config
→ build zone priority / scoring config
→ preview ranking bằng NumPy
→ visualize priority distribution

C++
→ load multiple robot-frame targets
→ classify / inspect / score each target
→ rank targets
→ select best target candidate
→ export prioritization reports
```

Mục tiêu cốt lõi:
- hiểu cách **xếp hạng nhiều target** trong robot frame
- biết cách kết hợp:
  - zone
  - reachability
  - distance
  - validity
  - scoring weight
- chuẩn bị nền trực tiếp cho:
  - target selection
  - navigation focus selection
  - interaction candidate ranking

---

# 6. Pipeline tổng thể

```text
Load Multi-Target Sample Config
Load Robot Zone Config
Load Reachability Threshold Config
Load Priority Weight Config
Load Visualization Config
Load Runtime Config

Create TargetZoneClassifier
Create ReachabilityInspector
Create TargetValidityEngine
Create PriorityScoreEngine
Create MultiTargetRankingEngine
Create MultiTargetSummaryEngine
Create MultiTargetRobotFrameSpatialPrioritizer

For each target set:
    1. load all robot-frame targets
    2. inspect every target
    3. compute priority scores
    4. rank targets
    5. choose best target
    6. write report

After all target sets:
    write reports
    write plots
```

---

# 7. Kiến thức cần

## Python
- NumPy
- OOP
- dict / list / module
- Graph / BST
- file handling
- Matplotlib nếu muốn visualize

## C++
- OOP
- inheritance / abstract class
- vector
- enum
- smart pointer
- sorting / ranking logic

## Computer Vision / Robotics Vision
- robot-frame target localization
- zone classification
- reachability reasoning
- target prioritization
- robot-centric spatial ranking

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **multi-target prioritization pipeline**:

```text
TargetSet
→ TargetInspection
→ ValidTargetFiltering
→ PriorityScoring
→ TargetRanking
→ BestTargetSelection
→ Report
```

## 2. Python BST
Lưu:
- `pair_id`
- `target_id`
- `priority_result_id`

## 3. `std::vector<RobotFrameTarget>`
Danh sách targets trong một sample / scene.

## 4. `std::vector<TargetPriorityResult>`
Danh sách kết quả priority của từng target.

## 5. `std::vector<TargetPriorityResult>`
sau khi sort để thành ranking.

## 6. `std::stack<PriorityDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Single-target inspection reuse
Bạn phải tái sử dụng logic từ Bài 63:
- zone classification
- reachability inspection
- validity checking

---

## Algorithm 2 — Priority scoring
Mỗi target phải có **priority score**.  
Ví dụ có thể dựa trên:
- zone bonus
- forward distance
- lateral penalty
- validity bonus
- reachability bonus

Ví dụ ý tưởng:

```text
priority_score =
    zone_weight * zone_bonus
  + reachability_weight * reachable_bonus
  + validity_weight * valid_bonus
  - forward_weight * normalized_forward_distance
  - lateral_weight * normalized_lateral_distance
```

Bạn có thể tự thiết kế công thức, nhưng phải giải thích rõ.

---

## Algorithm 3 — Filtering
Tùy config:
- bỏ target invalid trước khi ranking
- hoặc vẫn giữ nhưng cho điểm rất thấp

---

## Algorithm 4 — Ranking
- sort targets theo priority score giảm dần
- nếu bằng điểm, dùng tie-break:
  - target gần center hơn
  - target gần robot hơn
  - target có target_id nhỏ hơn

---

## Algorithm 5 — Best target selection
Chọn target đứng đầu ranking và ghi rõ:
- vì sao nó được ưu tiên
- target nào đứng thứ 2 / 3 nếu cần

---

## Algorithm 6 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- top target score chart
- zone distribution chart
- per-target priority ranking chart

---

# 9. Cấu trúc folder

```text
mini_project_64_multi_target_robot_frame_spatial_prioritizer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ target_inputs/
│  │  ├─ target_set_01.txt
│  │  ├─ target_set_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ target_priority_report.txt
│     ├─ target_ranking_report.txt
│     ├─ prioritization_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ target_priority_plot.png
│     └─ zone_distribution_plot.png
│
├─ config/
│  ├─ multi_target_sample_config.txt
│  ├─ robot_zone_config.txt
│  ├─ reachability_threshold_config.txt
│  ├─ priority_weight_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ prioritization_graph_preview.py
│     ├─ prioritization_bst.py
│     ├─ numpy_prioritization_preview.py
│     ├─ matplotlib_prioritization_plotter.py
│     └─ synthetic_multi_target_input_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ RobotFrameTarget.hpp
   │  ├─ RobotZoneConfig.hpp
   │  ├─ ReachabilityThresholdConfig.hpp
   │  ├─ PriorityWeightConfig.hpp
   │  ├─ TargetZone.hpp
   │  ├─ TargetPriorityResult.hpp
   │  ├─ PriorityDebugRecord.hpp
   │  ├─ BaseTargetZoneClassifier.hpp
   │  ├─ TargetZoneClassifier.hpp
   │  ├─ BaseReachabilityInspector.hpp
   │  ├─ ReachabilityInspector.hpp
   │  ├─ BaseTargetValidityEngine.hpp
   │  ├─ TargetValidityEngine.hpp
   │  ├─ BasePriorityScoreEngine.hpp
   │  ├─ PriorityScoreEngine.hpp
   │  ├─ BaseMultiTargetRankingEngine.hpp
   │  ├─ MultiTargetRankingEngine.hpp
   │  ├─ BaseMultiTargetSummaryEngine.hpp
   │  ├─ MultiTargetSummaryEngine.hpp
   │  ├─ MultiTargetRobotFrameSpatialPrioritizer.hpp
   │  └─ PriorityReportWriter.hpp
   │
   └─ src/
      ├─ TargetZoneClassifier.cpp
      ├─ ReachabilityInspector.cpp
      ├─ TargetValidityEngine.cpp
      ├─ PriorityScoreEngine.cpp
      ├─ MultiTargetRankingEngine.cpp
      ├─ MultiTargetSummaryEngine.cpp
      ├─ MultiTargetRobotFrameSpatialPrioritizer.cpp
      └─ PriorityReportWriter.cpp
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
multi_target_sample_config_path
robot_zone_config_path
reachability_threshold_config_path
priority_weight_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `MultiTargetPrioritizerConfigBuilder`

Tạo class con:

```python
class MultiTargetPrioritizerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_target(
    pair_id,
    target_id,
    robot_x,
    robot_y,
    robot_z,
    label=None,
    expected_priority=None
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

### `set_priority_weights(
    zone_weight,
    reachability_weight,
    validity_weight,
    forward_weight,
    lateral_weight
)`

### `set_visualization_options(
    enable_priority_plot,
    enable_zone_distribution_plot
)`

### `write_multi_target_sample_config()`
### `write_robot_zone_config()`
### `write_reachability_threshold_config()`
### `write_priority_weight_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `prioritization_graph_preview.py`

Tạo class:

```python
class PrioritizationGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
TargetSet
→ TargetInspection
→ ValidTargetFiltering
→ PriorityScoring
→ TargetRanking
→ BestTargetSelection
→ Report
```

---

# 10.4 Python — `prioritization_bst.py`

Tạo BST cho:
- `pair_id`
- `target_id`
- `priority_result_id`

---

# 10.5 Python — `numpy_prioritization_preview.py`

Tạo class:

```python
class NumPyPrioritizationPreview:
```

## Hàm cần có

### `load_target_set(path)`
### `classify_targets(targets, zone_config)`
### `compute_priority_scores(targets, weight_config, threshold_config)`
### `build_priority_ranking(results)`

---

# 10.6 Python — `matplotlib_prioritization_plotter.py`

Tạo class:

```python
class MatplotlibPrioritizationPlotter:
```

## Hàm cần có

### `plot_target_priority(results, save_path)`
### `plot_zone_distribution(results, save_path)`

---

# 10.7 C++ — `RobotFrameTarget`

```cpp
struct RobotFrameTarget
{
    std::string pair_id;
    std::string target_id;

    double robot_x;
    double robot_y;
    double robot_z;

    std::string label;
    int expected_priority;
};
```

---

# 10.8 C++ — `RobotZoneConfig`

```cpp
struct RobotZoneConfig
{
    double center_band;
    double left_right_boundary;
};
```

---

# 10.9 C++ — `ReachabilityThresholdConfig`

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

# 10.10 C++ — `PriorityWeightConfig`

```cpp
struct PriorityWeightConfig
{
    double zone_weight;
    double reachability_weight;
    double validity_weight;
    double forward_weight;
    double lateral_weight;
};
```

---

# 10.11 C++ — `TargetZone`

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

# 10.12 C++ — `TargetPriorityResult`

```cpp
struct TargetPriorityResult
{
    std::string pair_id;
    std::string target_id;

    TargetZone zone;
    bool is_reachable;
    bool is_valid;

    double forward_distance;
    double lateral_distance;
    double vertical_distance;

    double priority_score;
    int rank_position;

    std::string summary;
};
```

---

# 10.13 C++ — `PriorityDebugRecord`

```cpp
struct PriorityDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.14 C++ — `BaseTargetZoneClassifier`

Tạo abstract class:

```cpp
class BaseTargetZoneClassifier
{
public:
    virtual TargetZone classify(
        const RobotFrameTarget& target,
        const RobotZoneConfig& config
    ) const = 0;

    virtual ~BaseTargetZoneClassifier() = default;
};
```

---

# 10.15 C++ — `TargetZoneClassifier`

Kế thừa `BaseTargetZoneClassifier`.

## Nhiệm vụ
- gán zone cho từng target

---

# 10.16 C++ — `BaseReachabilityInspector`

Tạo abstract class:

```cpp
class BaseReachabilityInspector
{
public:
    virtual bool inspect(
        const RobotFrameTarget& target,
        const ReachabilityThresholdConfig& config
    ) const = 0;

    virtual ~BaseReachabilityInspector() = default;
};
```

---

# 10.17 C++ — `ReachabilityInspector`

Kế thừa `BaseReachabilityInspector`.

## Nhiệm vụ
- kiểm tra reachability

---

# 10.18 C++ — `BaseTargetValidityEngine`

Tạo abstract class:

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

# 10.19 C++ — `TargetValidityEngine`

Kế thừa `BaseTargetValidityEngine`.

## Nhiệm vụ
- gán valid / invalid

---

# 10.20 C++ — `BasePriorityScoreEngine`

Tạo abstract class:

```cpp
class BasePriorityScoreEngine
{
public:
    virtual double compute(
        const RobotFrameTarget& target,
        TargetZone zone,
        bool is_reachable,
        bool is_valid,
        const PriorityWeightConfig& weights
    ) const = 0;

    virtual ~BasePriorityScoreEngine() = default;
};
```

---

# 10.21 C++ — `PriorityScoreEngine`

Kế thừa `BasePriorityScoreEngine`.

## Nhiệm vụ
- tính priority score cho từng target

---

# 10.22 C++ — `BaseMultiTargetRankingEngine`

Tạo abstract class:

```cpp
class BaseMultiTargetRankingEngine
{
public:
    virtual std::vector<TargetPriorityResult> rank(
        const std::vector<TargetPriorityResult>& results
    ) const = 0;

    virtual ~BaseMultiTargetRankingEngine() = default;
};
```

---

# 10.23 C++ — `MultiTargetRankingEngine`

Kế thừa `BaseMultiTargetRankingEngine`.

## Nhiệm vụ
- sort / rank targets theo priority score

---

# 10.24 C++ — `BaseMultiTargetSummaryEngine`

Tạo abstract class:

```cpp
class BaseMultiTargetSummaryEngine
{
public:
    virtual std::string summarize(
        const std::vector<TargetPriorityResult>& ranked_results
    ) const = 0;

    virtual ~BaseMultiTargetSummaryEngine() = default;
};
```

---

# 10.25 C++ — `MultiTargetSummaryEngine`

Kế thừa `BaseMultiTargetSummaryEngine`.

## Nhiệm vụ
- tạo summary kiểu:
```text
Target T03 was selected as the highest-priority candidate because it lies in the center zone,
is reachable, and has the strongest overall spatial score among valid targets.
```

---

# 10.26 C++ — `MultiTargetRobotFrameSpatialPrioritizer`

Tạo class trung tâm:

```cpp
class MultiTargetRobotFrameSpatialPrioritizer
```

## Thuộc tính

```cpp
private:
    std::vector<RobotFrameTarget> targets;

    RobotZoneConfig zone_config;
    ReachabilityThresholdConfig threshold_config;
    PriorityWeightConfig weight_config;

    std::shared_ptr<BaseTargetZoneClassifier> zone_classifier;
    std::shared_ptr<BaseReachabilityInspector> reachability_inspector;
    std::shared_ptr<BaseTargetValidityEngine> validity_engine;
    std::shared_ptr<BasePriorityScoreEngine> score_engine;
    std::shared_ptr<BaseMultiTargetRankingEngine> ranking_engine;
    std::shared_ptr<BaseMultiTargetSummaryEngine> summary_engine;

    std::vector<TargetPriorityResult> results;
    std::stack<PriorityDebugRecord> debug_history;
```

## Hàm cần có

### `load_multi_target_sample_config(const std::string& path)`
### `load_robot_zone_config(const std::string& path)`
### `load_reachability_threshold_config(const std::string& path)`
### `load_priority_weight_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
group targets by pair_id

for each target set:
    inspect every target
    compute priority score
    rank targets
    append ranked results
    push debug history
```

### `const std::vector<TargetPriorityResult>& get_results() const`
### `std::vector<PriorityDebugRecord> get_debug_history_reverse()`

---

# 10.27 C++ — `PriorityReportWriter`

Tạo class:

```cpp
class PriorityReportWriter
```

## Hàm cần có

### `write_target_priority_report(...)`
Ví dụ:

```text
[Target Priority]
Pair: scene_01
Target: T03
Zone: CENTER_ZONE
Reachable: YES
Valid: YES
Priority Score: 9.24
Rank: 1
```

### `write_target_ranking_report(...)`
Ví dụ:

```text
[Target Ranking]
Pair: scene_01
1. T03 — score 9.24
2. T01 — score 8.71
3. T05 — score 7.90
```

### `write_prioritization_summary_report(...)`
Ví dụ:

```text
[Prioritization Summary]
Pair: scene_01
Target T03 was selected as the best candidate because it lies in the center zone,
is reachable, and has the strongest combined spatial score.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.28 C++ — `main.cpp`

## Yêu cầu

```text
Load multi-target sample config
Load robot zone config
Load reachability threshold config
Load priority weight config
Load visualization config
Load runtime config

Create:
    TargetZoneClassifier
    ReachabilityInspector
    TargetValidityEngine
    PriorityScoreEngine
    MultiTargetRankingEngine
    MultiTargetSummaryEngine

Create MultiTargetRobotFrameSpatialPrioritizer
Run
Write reports
```

---

# 11. Luật multi-target prioritization của project

## Luật 1 — Phải có nhiều target trong cùng một pair / scene
Nếu mỗi pair chỉ có 1 target thì không còn ý nghĩa ranking.

## Luật 2 — Priority score phải giải thích được
Không chỉ “cho số đại”. Phải mô tả target nào được thưởng / phạt vì zone, reachability, distance.

## Luật 3 — Ranking phải dựa trên score và tie-break rõ ràng
Không được random.

## Luật 4 — Target invalid không được ưu tiên vô lý
Bạn có thể:
- loại bỏ target invalid
- hoặc giữ lại nhưng cho điểm rất thấp

## Luật 5 — Report phải usable cho target selection layer
Tức là sau project này bạn phải trả lời được:
- target nào đứng hạng 1?
- vì sao nó đứng hạng 1?
- target nào là backup candidate?

---

# 12. Output mong muốn

## Config
```text
config/multi_target_sample_config.txt
config/robot_zone_config.txt
config/reachability_threshold_config.txt
config/priority_weight_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/target_priority_report.txt
assets/outputs/target_ranking_report.txt
assets/outputs/prioritization_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/target_priority_plot.png
assets/outputs/zone_distribution_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build multi-target configs
- preview scoring / ranking
- visualize target priority distribution

## C++
- inspect nhiều robot-frame targets
- score / rank / select target
- tạo prioritization report

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Nếu robot có nhiều target hợp lệ trong robot frame, target nào nên được ưu tiên trước?**

Pipeline lúc này sẽ là:

```text
pixel + depth
→ camera-frame point
→ robot-frame point
→ target localization inspection
→ multi-target spatial prioritization
→ best target selection
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho multi-target / zone / threshold / weight / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / target ids / priority result ids
- [ ] Python có NumPy prioritization preview
- [ ] C++ có `RobotFrameTarget`
- [ ] C++ có `RobotZoneConfig`
- [ ] C++ có `ReachabilityThresholdConfig`
- [ ] C++ có `PriorityWeightConfig`
- [ ] C++ có `TargetZone`
- [ ] C++ có `TargetPriorityResult`
- [ ] C++ có `TargetZoneClassifier`
- [ ] C++ có `ReachabilityInspector`
- [ ] C++ có `TargetValidityEngine`
- [ ] C++ có `PriorityScoreEngine`
- [ ] C++ có `MultiTargetRankingEngine`
- [ ] C++ có `MultiTargetSummaryEngine`
- [ ] C++ có `MultiTargetRobotFrameSpatialPrioritizer`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 64**, bước hợp lý nhất để tiếp tục Đợt 13 là:

# **Bài 65: Robot-Frame Interaction Target Selector**

Ý tưởng:
```text
ranked robot-frame targets
→ task-specific filtering
→ interaction-ready candidate selection
→ final interaction target choice
→ target selection report
```

Tức là mạch sẽ đi:

```text
Bài 63: Robot-Frame Target Localization Inspector
→ Bài 64: Multi-Target Robot-Frame Spatial Prioritizer
→ Bài 65: Robot-Frame Interaction Target Selector
```
