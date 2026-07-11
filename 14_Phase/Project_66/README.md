# 🤖 Bài 66: Humanoid Object Following Target Tracker Planner — Bộ lập kế hoạch theo dõi mục tiêu cho chế độ object following của humanoid

> Mini Project số 66 trong **Đợt 14**  
> **Bài 66 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 65**.
>
> Nếu:
>
> - **Bài 65** đã chọn ra **interaction target cuối cùng** theo task mode
> - một trong các mode của Đợt 14 là **Object Following**
>
> thì bước tiếp theo hợp lý nhất là:
>
> ```text
> selected follow target
> → temporal target state buffer
> → target update / target loss handling
> → follow stability checking
> → following-ready tracking plan
> ```
>
> Đây là bước mà bạn không chỉ chọn target “nên follow”, mà còn phải xử lý bài toán:
>
> - target có được nhìn thấy ổn định qua nhiều frame không?
> - target có đang drift trái/phải quá mạnh không?
> - nếu target bị mất tạm thời thì robot nên giữ target cũ, tìm lại hay bỏ target?
> - robot cần một **tracking plan** như thế nào để tiếp tục following?
>
> Tức là dữ liệu bắt đầu usable cho:
> - temporal target tracking
> - follow-state management
> - target loss recovery logic
> - humanoid following perception planning

---

# 📌 Mục lục

- [1. Vì sao Bài 66 xuất hiện sau Bài 65](#1-vì-sao-bài-66-xuất-hiện-sau-bài-65)
- [2. Bài 66 nâng từ Bài 65 lên chỗ nào](#2-bài-66-nâng-từ-bài-65-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Following pipeline tổng thể](#5-following-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật following tracker planner của project](#10-luật-following-tracker-planner-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 66 xuất hiện sau Bài 65

## Bài 65 đang làm gì?
Bài 65 đã giúp bạn:
- nhận nhiều target trong robot frame
- áp dụng **task-aware rules**
- chọn **interaction target cuối cùng**
- hỗ trợ các mode:
  - `FOLLOW_MODE`
  - `PICK_MODE`
  - `TABLE_CLEAN_MODE`

## Nhưng còn thiếu gì?
Nếu robot đang ở **FOLLOW_MODE**, chỉ chọn target thôi chưa đủ.  
Robot còn phải **theo dõi target đó theo thời gian**:

- target frame hiện tại có giống target frame trước không?
- target có đang di chuyển ra ngoài vùng center không?
- target có bị mất detection ở một vài frame không?
- target có còn đáng để tiếp tục follow không?

## Bài 66 lấp đúng chỗ đó
Bài 66 sẽ làm:

```text
selected follow target
+ historical target states
+ tracking thresholds
+ loss tolerance rules
→ follow-state update
→ tracking stability evaluation
→ follow-ready tracking plan
```

Đây là bước chuyển từ **target selection** sang **temporal target following planning**.

---

# 2. Bài 66 nâng từ Bài 65 lên chỗ nào

## Bài 65
- chọn target cuối cùng cho mode hiện tại
- quyết định target nào robot nên tương tác

## Bài 66
- chỉ tập trung vào **FOLLOW_MODE**
- quản lý **lịch sử target theo thời gian**
- đánh giá:
  - target có ổn định không
  - target có bị mất không
  - target có drift quá mạnh không
- tạo **tracking plan** cho robot tiếp tục following

### Nói ngắn gọn:
- **Bài 65** hỏi: “Robot nên follow target nào?”
- **Bài 66** hỏi: “Sau khi đã chọn target follow, robot phải theo dõi target đó như thế nào qua thời gian?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Humanoid Object Following Target Tracker Planner**

System này nhận đầu vào là:
- một hoặc nhiều **follow scenes**
- mỗi scene chứa chuỗi **frame observations** của target đang được follow
- một bộ **tracking thresholds**
- một bộ **target loss handling rules**
- một bộ **follow plan rules**
- optional:
  - previous selected target id
  - confidence history
  - velocity estimate

## Mỗi frame observation tối thiểu chứa
- `scene_id`
- `frame_id`
- `target_id`
- `robot_x`
- `robot_y`
- `robot_z`
- `confidence`
- optional:
  - `is_detected`
  - `estimated_velocity`
  - `source_pair_id`

## Nhiệm vụ của system
### Với mỗi scene:
1. load toàn bộ frame observations của target
2. xây **temporal target state buffer**
3. cập nhật trạng thái target theo frame
4. kiểm tra:
   - target visible ratio
   - confidence trend
   - lateral drift
   - forward distance stability
   - lost-frame streak
5. quyết định:
   - `KEEP_FOLLOWING`
   - `SLOW_FOLLOW`
   - `SEARCH_TARGET`
   - `REACQUIRE_TARGET`
   - `DROP_TARGET`
6. ghi **following tracking plan report**

<p align="center">
  <img src="../../images/project_66.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build follow-scene sequence config
→ build tracking thresholds / loss rules
→ preview target trajectory & confidence trend bằng NumPy
→ visualize tracking stability summary

C++
→ load temporal target observations
→ maintain tracking buffer
→ inspect target stability / drift / visibility
→ build following-ready plan
→ export tracking reports
```

Mục tiêu cốt lõi:
- hiểu cách chuyển từ **single-frame target selection** sang **multi-frame target following**
- biết cách thiết kế **temporal tracking buffer**
- biết cách xử lý **target loss / reacquire / keep following**
- chuẩn bị nền trực tiếp cho:
  - object following logic
  - tracking state machine
  - perception-to-navigation follow control handoff

---

# 5. Following pipeline tổng thể

```text
Load Follow Scene Config
Load Tracking Threshold Config
Load Target Loss Rule Config
Load Follow Plan Rule Config
Load Visualization Config
Load Runtime Config

Create TemporalTargetBuffer
Create TrackingStabilityInspector
Create TargetLossHandler
Create FollowDecisionEngine
Create FollowingSummaryEngine
Create HumanoidObjectFollowingTargetTrackerPlanner

For each follow scene:
    1. load ordered frame observations
    2. update temporal buffer
    3. inspect tracking stability
    4. inspect target loss state
    5. build follow decision
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
- list / dict / deque
- Graph / BST
- NumPy
- Matplotlib

## C++
- OOP / inheritance / abstract class
- vector / deque / enum / smart pointer
- temporal state update logic
- report writing

## Computer Vision / Robotics Vision
- object following
- temporal tracking
- target confidence trend
- target loss handling
- robot-centric follow planning

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **following tracker planning pipeline**:

```text
FollowSceneFrames
→ TemporalBufferUpdate
→ TrackingStabilityInspection
→ LossStateCheck
→ FollowDecision
→ TrackingPlanReport
```

## 2. Python BST
Lưu:
- `scene_id`
- `target_id`
- `tracking_result_id`

## 3. `std::vector<FollowFrameObservation>`
Danh sách frame observations.

## 4. `std::deque<FollowFrameObservation>`
Temporal buffer cho target gần nhất.

## 5. `std::vector<TrackingPlanResult>`
Kết quả tracking plan cho từng scene.

## 6. `std::stack<TrackingDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Temporal buffer update
Với mỗi frame:
- push observation vào buffer
- nếu buffer vượt quá `max_history_length` thì pop front

---

## Algorithm 2 — Tracking stability inspection
Tính ít nhất:
- `visible_ratio`
- `average_confidence`
- `max_lateral_drift`
- `forward_distance_variation`

Ví dụ:
- visible ratio = số frame detected / tổng frame
- lateral drift = max(y) - min(y)
- forward variation = max(x) - min(x)

---

## Algorithm 3 — Target loss handling
Theo dõi:
- số frame liên tiếp không detect được target
- số frame confidence quá thấp
- số lần target nhảy zone bất thường

Gợi ý action:
- mất 1–2 frame → `SEARCH_TARGET`
- mất vài frame nhưng trước đó target ổn định → `REACQUIRE_TARGET`
- mất quá lâu → `DROP_TARGET`

---

## Algorithm 4 — Follow decision engine
Dựa trên:
- stability metrics
- loss metrics
- current target position

Quyết định một trong:
- `KEEP_FOLLOWING`
- `SLOW_FOLLOW`
- `SEARCH_TARGET`
- `REACQUIRE_TARGET`
- `DROP_TARGET`

---

## Algorithm 5 — Following summary
Tạo summary kiểu:
```text
The follow target remained visible in 85% of recent frames and stayed near the center corridor,
so the planner recommends KEEP_FOLLOWING.
```

---

## Algorithm 6 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- confidence trend plot
- visible / lost frame summary
- lateral drift plot

---

# 8. Cấu trúc folder

```text
mini_project_66_humanoid_object_following_target_tracker_planner/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ follow_scene_inputs/
│  │  ├─ follow_scene_01.txt
│  │  ├─ follow_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ tracking_plan_report.txt
│     ├─ tracking_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ confidence_trend_plot.png
│     └─ visibility_summary_plot.png
│
├─ config/
│  ├─ follow_scene_config.txt
│  ├─ tracking_threshold_config.txt
│  ├─ target_loss_rule_config.txt
│  ├─ follow_plan_rule_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ following_graph_preview.py
│     ├─ following_bst.py
│     ├─ numpy_following_preview.py
│     ├─ matplotlib_following_plotter.py
│     └─ synthetic_follow_scene_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ FollowFrameObservation.hpp
   │  ├─ TrackingThresholdConfig.hpp
   │  ├─ TargetLossRuleConfig.hpp
   │  ├─ FollowPlanRuleConfig.hpp
   │  ├─ FollowDecision.hpp
   │  ├─ TrackingPlanResult.hpp
   │  ├─ TrackingDebugRecord.hpp
   │  ├─ BaseTemporalTargetBuffer.hpp
   │  ├─ TemporalTargetBuffer.hpp
   │  ├─ BaseTrackingStabilityInspector.hpp
   │  ├─ TrackingStabilityInspector.hpp
   │  ├─ BaseTargetLossHandler.hpp
   │  ├─ TargetLossHandler.hpp
   │  ├─ BaseFollowDecisionEngine.hpp
   │  ├─ FollowDecisionEngine.hpp
   │  ├─ BaseFollowingSummaryEngine.hpp
   │  ├─ FollowingSummaryEngine.hpp
   │  ├─ HumanoidObjectFollowingTargetTrackerPlanner.hpp
   │  └─ TrackingReportWriter.hpp
   │
   └─ src/
      ├─ TemporalTargetBuffer.cpp
      ├─ TrackingStabilityInspector.cpp
      ├─ TargetLossHandler.cpp
      ├─ FollowDecisionEngine.cpp
      ├─ FollowingSummaryEngine.cpp
      ├─ HumanoidObjectFollowingTargetTrackerPlanner.cpp
      └─ TrackingReportWriter.cpp
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
follow_scene_config_path
tracking_threshold_config_path
target_loss_rule_config_path
follow_plan_rule_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `HumanoidFollowingConfigBuilder`

Tạo class con:

```python
class HumanoidFollowingConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_follow_observation(
    scene_id,
    frame_id,
    target_id,
    robot_x,
    robot_y,
    robot_z,
    confidence,
    is_detected=True,
    estimated_velocity=None,
    source_pair_id=None
)`

### `set_tracking_thresholds(
    max_history_length,
    min_visible_ratio,
    min_average_confidence,
    max_lateral_drift,
    max_forward_variation
)`

### `set_target_loss_rules(
    short_loss_tolerance,
    long_loss_tolerance,
    reacquire_confidence_threshold
)`

### `set_follow_plan_rules(
    keep_follow_confidence_bonus,
    slow_follow_penalty,
    search_target_penalty,
    drop_target_penalty
)`

### `set_visualization_options(
    enable_confidence_plot,
    enable_visibility_plot
)`

### `write_follow_scene_config()`
### `write_tracking_threshold_config()`
### `write_target_loss_rule_config()`
### `write_follow_plan_rule_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `following_graph_preview.py`

Tạo class:

```python
class FollowingGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
FollowSceneFrames
→ TemporalBufferUpdate
→ TrackingStabilityInspection
→ LossStateCheck
→ FollowDecision
→ TrackingPlanReport
```

---

# 9.4 Python — `following_bst.py`

Tạo BST cho:
- `scene_id`
- `target_id`
- `tracking_result_id`

---

# 9.5 Python — `numpy_following_preview.py`

Tạo class:

```python
class NumPyFollowingPreview:
```

## Hàm cần có

### `load_follow_scene(path)`
### `build_temporal_buffer(scene_frames, max_history_length)`
### `compute_tracking_metrics(scene_frames, threshold_config)`
### `build_follow_decision_preview(metrics, loss_rules)`

---

# 9.6 Python — `matplotlib_following_plotter.py`

Tạo class:

```python
class MatplotlibFollowingPlotter:
```

## Hàm cần có

### `plot_confidence_trend(scene_frames, save_path)`
### `plot_visibility_summary(scene_frames, save_path)`

---

# 9.7 C++ — `FollowFrameObservation`

```cpp
struct FollowFrameObservation
{
    std::string scene_id;
    int frame_id;
    std::string target_id;

    double robot_x;
    double robot_y;
    double robot_z;

    double confidence;
    bool is_detected;

    double estimated_velocity;
    std::string source_pair_id;
};
```

---

# 9.8 C++ — `TrackingThresholdConfig`

```cpp
struct TrackingThresholdConfig
{
    int max_history_length;

    double min_visible_ratio;
    double min_average_confidence;
    double max_lateral_drift;
    double max_forward_variation;
};
```

---

# 9.9 C++ — `TargetLossRuleConfig`

```cpp
struct TargetLossRuleConfig
{
    int short_loss_tolerance;
    int long_loss_tolerance;
    double reacquire_confidence_threshold;
};
```

---

# 9.10 C++ — `FollowPlanRuleConfig`

```cpp
struct FollowPlanRuleConfig
{
    double keep_follow_confidence_bonus;
    double slow_follow_penalty;
    double search_target_penalty;
    double drop_target_penalty;
};
```

---

# 9.11 C++ — `FollowDecision`

```cpp
enum class FollowDecision
{
    KEEP_FOLLOWING,
    SLOW_FOLLOW,
    SEARCH_TARGET,
    REACQUIRE_TARGET,
    DROP_TARGET
};
```

---

# 9.12 C++ — `TrackingPlanResult`

```cpp
struct TrackingPlanResult
{
    std::string scene_id;
    std::string target_id;

    double visible_ratio;
    double average_confidence;
    double max_lateral_drift;
    double forward_variation;

    int lost_frame_streak;
    FollowDecision decision;

    std::string summary;
};
```

---

# 9.13 C++ — `TrackingDebugRecord`

```cpp
struct TrackingDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.14 C++ — `BaseTemporalTargetBuffer`

```cpp
class BaseTemporalTargetBuffer
{
public:
    virtual void push(const FollowFrameObservation& observation) = 0;
    virtual std::vector<FollowFrameObservation> get_history() const = 0;
    virtual ~BaseTemporalTargetBuffer() = default;
};
```

---

# 9.15 C++ — `BaseTrackingStabilityInspector`

```cpp
class BaseTrackingStabilityInspector
{
public:
    virtual TrackingPlanResult inspect(
        const std::vector<FollowFrameObservation>& history,
        const TrackingThresholdConfig& config
    ) const = 0;

    virtual ~BaseTrackingStabilityInspector() = default;
};
```

---

# 9.16 C++ — `BaseTargetLossHandler`

```cpp
class BaseTargetLossHandler
{
public:
    virtual int count_lost_frame_streak(
        const std::vector<FollowFrameObservation>& history
    ) const = 0;

    virtual ~BaseTargetLossHandler() = default;
};
```

---

# 9.17 C++ — `BaseFollowDecisionEngine`

```cpp
class BaseFollowDecisionEngine
{
public:
    virtual FollowDecision decide(
        const TrackingPlanResult& result,
        const TargetLossRuleConfig& loss_rules,
        const FollowPlanRuleConfig& plan_rules
    ) const = 0;

    virtual ~BaseFollowDecisionEngine() = default;
};
```

---

# 9.18 C++ — `BaseFollowingSummaryEngine`

```cpp
class BaseFollowingSummaryEngine
{
public:
    virtual std::string summarize(
        const TrackingPlanResult& result
    ) const = 0;

    virtual ~BaseFollowingSummaryEngine() = default;
};
```

---

# 9.19 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `TemporalTargetBuffer`
- `TrackingStabilityInspector`
- `TargetLossHandler`
- `FollowDecisionEngine`
- `FollowingSummaryEngine`
- `HumanoidObjectFollowingTargetTrackerPlanner`
- `TrackingReportWriter`

---

# 9.20 C++ — `HumanoidObjectFollowingTargetTrackerPlanner`

Tạo class trung tâm:

```cpp
class HumanoidObjectFollowingTargetTrackerPlanner
```

## Thuộc tính

```cpp
private:
    std::vector<FollowFrameObservation> observations;

    TrackingThresholdConfig threshold_config;
    TargetLossRuleConfig loss_rule_config;
    FollowPlanRuleConfig plan_rule_config;

    std::shared_ptr<BaseTemporalTargetBuffer> temporal_buffer;
    std::shared_ptr<BaseTrackingStabilityInspector> stability_inspector;
    std::shared_ptr<BaseTargetLossHandler> loss_handler;
    std::shared_ptr<BaseFollowDecisionEngine> decision_engine;
    std::shared_ptr<BaseFollowingSummaryEngine> summary_engine;

    std::vector<TrackingPlanResult> results;
    std::stack<TrackingDebugRecord> debug_history;
```

## Hàm cần có

### `load_follow_scene_config(const std::string& path)`
### `load_tracking_threshold_config(const std::string& path)`
### `load_target_loss_rule_config(const std::string& path)`
### `load_follow_plan_rule_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
group observations by scene_id

for each scene:
    sort by frame_id
    push frames into temporal buffer
    history = temporal_buffer.get_history()

    result = stability_inspector.inspect(history, threshold_config)
    result.lost_frame_streak = loss_handler.count_lost_frame_streak(history)
    result.decision = decision_engine.decide(result, loss_rule_config, plan_rule_config)
    result.summary = summary_engine.summarize(result)

    save result
    push debug history
```

### `const std::vector<TrackingPlanResult>& get_results() const`
### `std::vector<TrackingDebugRecord> get_debug_history_reverse()`

---

# 9.21 C++ — `TrackingReportWriter`

Tạo class:

```cpp
class TrackingReportWriter
```

## Hàm cần có

### `write_tracking_plan_report(...)`
Ví dụ:

```text
[Tracking Plan]
Scene: follow_scene_01
Target: person_01
Visible Ratio: 0.86
Average Confidence: 0.81
Lost Frame Streak: 1
Decision: KEEP_FOLLOWING
```

### `write_tracking_summary_report(...)`
Ví dụ:

```text
[Tracking Summary]
Scene: follow_scene_01
The target remained visible in most recent frames and stayed stable enough for continued following.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.22 C++ — `main.cpp`

## Yêu cầu

```text
Load follow scene config
Load tracking threshold config
Load target loss rule config
Load follow plan rule config
Load visualization config
Load runtime config

Create:
    TemporalTargetBuffer
    TrackingStabilityInspector
    TargetLossHandler
    FollowDecisionEngine
    FollowingSummaryEngine

Create HumanoidObjectFollowingTargetTrackerPlanner
Run
Write reports
```

---

# 10. Luật following tracker planner của project

## Luật 1 — Bài này chỉ tập trung vào FOLLOW_MODE
Không trộn pick / clean vào logic chính.

## Luật 2 — Phải có yếu tố thời gian
Nếu không có chuỗi frame / temporal buffer thì bài này mất ý nghĩa.

## Luật 3 — Decision phải dựa trên stability + loss state
Không được gán `KEEP_FOLLOWING` hay `DROP_TARGET` ngẫu nhiên.

## Luật 4 — Phải xử lý target loss
Ít nhất phải phân biệt:
- mất ngắn hạn
- mất dài hạn
- có thể reacquire hay không

## Luật 5 — Report phải usable cho follow planner
Tức là sau bài này bạn phải trả lời được:
- target hiện có đủ ổn định để tiếp tục follow không?
- nếu target bị mất thì robot nên search, reacquire hay drop?

---

# 11. Output mong muốn

## Config
```text
config/follow_scene_config.txt
config/tracking_threshold_config.txt
config/target_loss_rule_config.txt
config/follow_plan_rule_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/tracking_plan_report.txt
assets/outputs/tracking_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/confidence_trend_plot.png
assets/outputs/visibility_summary_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build follow-scene temporal configs
- preview confidence / visibility / drift trends
- visualize follow stability

## C++
- đóng vai **following-target temporal planner**
- quản lý target history
- quyết định keep / slow / search / reacquire / drop

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Sau khi chọn target để follow, robot phải theo dõi target đó ổn định qua thời gian như thế nào?**

Pipeline lúc này sẽ là:

```text
target selection
→ temporal target observation history
→ stability + loss analysis
→ follow decision
→ following-ready tracking plan
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho follow scene / thresholds / loss rules / plan rules / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / target ids / tracking result ids
- [ ] Python có NumPy following preview
- [ ] C++ có `FollowFrameObservation`
- [ ] C++ có `TrackingThresholdConfig`
- [ ] C++ có `TargetLossRuleConfig`
- [ ] C++ có `FollowPlanRuleConfig`
- [ ] C++ có `FollowDecision`
- [ ] C++ có `TrackingPlanResult`
- [ ] C++ có `TemporalTargetBuffer`
- [ ] C++ có `TrackingStabilityInspector`
- [ ] C++ có `TargetLossHandler`
- [ ] C++ có `FollowDecisionEngine`
- [ ] C++ có `FollowingSummaryEngine`
- [ ] C++ có `HumanoidObjectFollowingTargetTrackerPlanner`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 66**, bước hợp lý nhất để tiếp tục Đợt 14 là:

# **Bài 67: Pick-and-Place Vision Candidate Planner**

Ý tưởng:
```text
pick-mode targets
→ pickability filtering
→ grasp-friendly zone inspection
→ placement-ready candidate ranking
→ pick candidate plan
```

Tức là mạch sẽ đi:

```text
Bài 65: Humanoid Interaction Target Selector
→ Bài 66: Humanoid Object Following Target Tracker Planner
→ Bài 67: Pick-and-Place Vision Candidate Planner
→ Bài 68: Table-Cleaning Perception Task Scheduler
```
