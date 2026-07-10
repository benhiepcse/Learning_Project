# 🤖 Bài 60: Obstacle-Aware Local Navigation Decision System — Hệ ra quyết định điều hướng cục bộ có nhận thức vật cản cho Humanoid Robot AI Perception

> Mini Project số 60 trong **Đợt 12**  
> **Bài 60 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12** và đi tiếp trực tiếp từ **Bài 59**.
>
> Nếu:
>
> - **Bài 57** giúp bạn phân tích **robot-frame obstacle distance**
> - **Bài 58** giúp bạn chọn **best free-space corridor**
> - **Bài 59** giúp bạn tạo **walking direction decision**
>
> thì **Bài 60** là bước tiếp theo rất tự nhiên:
>
> ```text
> walking direction decision
> + corridor safety signals
> + obstacle danger signals
> → action priority selection
> → local navigation command suggestion
> → final local navigation decision report
> ```
>
> Đây là bước mà perception + decision không chỉ trả lời:
>
> - “robot nên đi trái / phải / thẳng?”
>
> mà còn tiến thêm một bước:
>
> - **“robot nên thực hiện lệnh điều hướng cục bộ nào ngay bây giờ?”**
> - **“có nên tiến tiếp, giảm tốc, đổi corridor, hay dừng lại?”**
> - **“quyết định cuối cùng có bị override bởi danger zone hay không?”**
>
> Tức là dữ liệu bắt đầu usable cho:
> - local navigation command suggestion
> - safety-aware locomotion decision
> - obstacle-aware movement arbitration
> - navigation-layer mockup

---

# 📌 Mục lục

- [1. Đợt 12 đang đi tiếp từ Bài 59 ra sao](#1-đợt-12-đang-đi-tiếp-từ-bài-59-ra-sao)
- [2. Vì sao Bài 60 xuất hiện sau Bài 59](#2-vì-sao-bài-60-xuất-hiện-sau-bài-59)
- [3. Bài 60 nâng từ Bài 59 lên chỗ nào](#3-bài-60-nâng-từ-bài-59-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật local navigation decision của project](#11-luật-local-navigation-decision-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 12 đang đi tiếp từ Bài 59 ra sao

Ở **Bài 59**, bạn đã có:

```text
corridor result
→ heading suggestion
→ steering action
→ local walking direction report
```

Tức là bạn đã biết:
- robot nên **đi thẳng / lệch trái / lệch phải / chậm lại / dừng**
- heading đề xuất là bao nhiêu độ
- độ tự tin của hướng đi đó là bao nhiêu

Nhưng robot vẫn còn thiếu một bước rất quan trọng:

> **Nếu hướng đi được đề xuất mâu thuẫn với danger signals hiện tại, lệnh cuối cùng nên là gì?**

Đó chính là mục tiêu của **Bài 60**.

---

# 2. Vì sao Bài 60 xuất hiện sau Bài 59

## Bài 59 đang làm gì?
Bài 59 giúp bạn:
- chọn hướng đi cục bộ
- gợi ý steering action
- sinh movement summary

## Nhưng còn thiếu gì?
Trong robot thật, quyết định cuối cùng không thể chỉ dựa trên **best corridor**. Nó còn phải xét:
- danger score hiện tại
- front obstacle distance
- stop condition
- action priority (ví dụ danger quá cao thì phải override mọi steering recommendation)

## Bài 60 lấp đúng chỗ đó
Bài 60 sẽ làm:

```text
direction decision
+ danger / clearance / corridor confidence
→ priority arbitration
→ local navigation command
→ final decision summary
```

Đây là bước chuyển từ **movement suggestion** sang **navigation decision arbitration**.

---

# 3. Bài 60 nâng từ Bài 59 lên chỗ nào

## Bài 59
- chọn heading
- chọn steering action
- tạo walking direction report

## Bài 60
- nhận nhiều tín hiệu perception / safety cùng lúc
- áp dụng **priority rules**
- gán **final navigation command**
- hỗ trợ các lệnh như:
  - `MOVE_FORWARD`
  - `MOVE_FORWARD_SLOW`
  - `TURN_LEFT_IN_PLACE`
  - `TURN_RIGHT_IN_PLACE`
  - `STOP_AND_WAIT`
  - `STOP_AND_REPLAN`

### Nói ngắn gọn:
- **Bài 59** hỏi: “Robot nên nghiêng hướng đi về đâu?”
- **Bài 60** hỏi: “Robot nên thực hiện lệnh điều hướng cục bộ nào ngay bây giờ?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Obstacle-Aware Local Navigation Decision System**

System này nhận đầu vào là:
- một hoặc nhiều **walking direction results**
- optional **obstacle distance / danger summaries**
- optional **corridor confidence summaries**
- một bộ **action priority rules**
- một bộ **navigation decision thresholds**

## Mỗi sample tối thiểu chứa
- `pair_id`
- `algorithm_type`
- `direction_result_path`
- optional:
  - `danger_result_path`
  - `corridor_result_path`
- optional `scene_label`

## Nhiệm vụ của system
### Với mỗi sample:
1. load walking direction result
2. load danger / corridor summaries nếu có
3. trích xuất:
   - steering action
   - target heading
   - direction confidence
   - danger score / stop signal / corridor confidence
4. áp dụng **priority arbitration**
5. sinh **final local navigation command**
6. ghi **navigation decision report**

<p align="center">
  <img src="images/project_60.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build direction-result config
→ build action-priority config
→ preview command arbitration logic
→ visualize final navigation commands

C++
→ load direction / danger / corridor summaries
→ fuse signals
→ apply navigation priority rules
→ output final local navigation command
→ export navigation decision reports
```

Mục tiêu cốt lõi:
- hiểu cách **fuse nhiều tín hiệu perception** vào một quyết định cục bộ
- biết cách thiết kế **priority rules**
- biết cách override heading suggestion khi danger cao
- chuẩn bị nền trực tiếp cho:
  - simple navigation layer
  - locomotion command arbitration
  - behavior-level local decision

---

# 6. Pipeline tổng thể

```text
Load Direction Result Config
Load Navigation Priority Config
Load Navigation Threshold Config
Load Visualization Config
Load Runtime Config

Create ResultLoader
Create SignalFusionEngine
Create NavigationPriorityEngine
Create FinalNavigationDecisionEngine
Create NavigationDecisionSummaryEngine
Create ObstacleAwareLocalNavigationDecisionSystem

For each sample:
    1. load direction result
    2. load optional danger / corridor results
    3. fuse all decision signals
    4. apply priority arbitration
    5. build final navigation command
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
- basic NumPy / Python data processing
- Matplotlib
- dict / list / BST / Graph

## C++
- class / inheritance / polymorphism / virtual function
- vector
- enum
- file parsing / export
- decision-engine architecture

## Computer Vision / Robot Perception / Decision Layer
- corridor result interpretation
- danger signal integration
- action arbitration
- navigation command selection
- safety override logic

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **local navigation decision pipeline**:

```text
DirectionResult
→ SignalFusion
→ PriorityArbitration
→ FinalCommandDecision
→ DecisionSummary
→ Report
```

## 2. Python BST
Lưu `pair_id`, `direction_result_id`, `navigation_decision_id`.

## 3. `std::vector<NavigationDecisionSample>`
Danh sách sample đầu vào.

## 4. `std::vector<NavigationDecisionResult>`
Danh sách kết quả quyết định điều hướng.

## 5. `std::stack<NavigationDecisionDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Signal fusion
Kết hợp ít nhất các tín hiệu:
- steering action
- direction confidence
- optional danger score
- optional corridor confidence

---

## Algorithm 2 — Priority arbitration
Thiết kế luật ưu tiên ví dụ:
- nếu `danger_score >= hard_stop_threshold` → `STOP_AND_WAIT`
- nếu `direction_confidence` thấp nhưng danger vừa → `MOVE_FORWARD_SLOW`
- nếu steering trái/phải được đề xuất và không có hard stop → turn tương ứng
- nếu corridor center tốt và danger thấp → `MOVE_FORWARD`

---

## Algorithm 3 — Final navigation command decision
Sinh lệnh cuối cùng từ tập:
- `MOVE_FORWARD`
- `MOVE_FORWARD_SLOW`
- `TURN_LEFT_IN_PLACE`
- `TURN_RIGHT_IN_PLACE`
- `STOP_AND_WAIT`
- `STOP_AND_REPLAN`

---

## Algorithm 4 — Decision confidence / status
Tạo:
- final decision confidence
- status message
- reason summary

---

## Algorithm 5 — Visualization
Bắt buộc có ít nhất 1 dạng plot:
- navigation command frequency chart
- decision confidence chart
- command-vs-danger plot

---

# 9. Cấu trúc folder

```text
mini_project_60_obstacle_aware_local_navigation_decision_system/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ decision_inputs/
│  │  ├─ direction_result_pair_01_bm.txt
│  │  ├─ direction_result_pair_01_sgm.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ fused_signal_report.txt
│     ├─ navigation_decision_report.txt
│     ├─ navigation_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ navigation_command_plot.png
│     └─ decision_confidence_plot.png
│
├─ config/
│  ├─ navigation_decision_sample_config.txt
│  ├─ navigation_priority_config.txt
│  ├─ navigation_threshold_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ navigation_graph_preview.py
│     ├─ navigation_bst.py
│     ├─ numpy_navigation_preview.py
│     ├─ matplotlib_navigation_plotter.py
│     └─ synthetic_navigation_input_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ NavigationDecisionSample.hpp
   │  ├─ NavigationPriorityConfig.hpp
   │  ├─ NavigationThresholdConfig.hpp
   │  ├─ FusedDecisionSignal.hpp
   │  ├─ NavigationDecisionResult.hpp
   │  ├─ NavigationDecisionDebugRecord.hpp
   │  ├─ BaseSignalFusionEngine.hpp
   │  ├─ SignalFusionEngine.hpp
   │  ├─ BaseNavigationPriorityEngine.hpp
   │  ├─ NavigationPriorityEngine.hpp
   │  ├─ BaseFinalNavigationDecisionEngine.hpp
   │  ├─ FinalNavigationDecisionEngine.hpp
   │  ├─ BaseNavigationDecisionSummaryEngine.hpp
   │  ├─ NavigationDecisionSummaryEngine.hpp
   │  ├─ ObstacleAwareLocalNavigationDecisionSystem.hpp
   │  └─ NavigationDecisionReportWriter.hpp
   │
   └─ src/
      ├─ SignalFusionEngine.cpp
      ├─ NavigationPriorityEngine.cpp
      ├─ FinalNavigationDecisionEngine.cpp
      ├─ NavigationDecisionSummaryEngine.cpp
      ├─ ObstacleAwareLocalNavigationDecisionSystem.cpp
      └─ NavigationDecisionReportWriter.cpp
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
navigation_sample_config_path
navigation_priority_config_path
navigation_threshold_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `NavigationDecisionConfigBuilder`

Tạo class con:

```python
class NavigationDecisionConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_navigation_sample(
    pair_id,
    algorithm_type,
    direction_result_path,
    danger_result_path=None,
    corridor_result_path=None,
    scene_label=None
)`

### `set_navigation_priority(
    hard_stop_danger_threshold,
    slow_forward_confidence_threshold,
    turn_confidence_threshold
)`

### `set_navigation_thresholds(
    min_direction_confidence,
    min_corridor_confidence,
    stop_score_threshold
)`

### `set_visualization_options(
    enable_command_plot,
    enable_confidence_plot
)`

### `write_navigation_sample_config()`
### `write_navigation_priority_config()`
### `write_navigation_threshold_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `navigation_graph_preview.py`

Tạo class:

```python
class NavigationGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
DirectionResult
→ SignalFusion
→ PriorityArbitration
→ FinalCommandDecision
→ DecisionSummary
→ Report
```

---

# 10.4 Python — `navigation_bst.py`

Tạo BST cho `pair_id` / `navigation_decision_id`.

---

# 10.5 Python — `numpy_navigation_preview.py`

Tạo class:

```python
class NumPyNavigationPreview:
```

## Hàm cần có

### `load_direction_result(path)`
### `fuse_signals(direction_data, danger_data=None, corridor_data=None)`
### `decide_navigation_command(fused_data, priority_config, threshold_config)`
### `compute_decision_confidence(fused_data)`

---

# 10.6 Python — `matplotlib_navigation_plotter.py`

Tạo class:

```python
class MatplotlibNavigationPlotter:
```

## Hàm cần có

### `plot_navigation_commands(command_summary, save_path)`
### `plot_decision_confidence(confidence_summary, save_path)`

---

# 10.7 C++ — `NavigationDecisionSample`

```cpp
enum class NavigationSourceType
{
    BM,
    SGM
};

struct NavigationDecisionSample
{
    std::string pair_id;
    NavigationSourceType algorithm_type;

    std::string direction_result_path;
    std::string danger_result_path;
    std::string corridor_result_path;

    std::string scene_label;
};
```

---

# 10.8 C++ — `NavigationPriorityConfig`

```cpp
struct NavigationPriorityConfig
{
    double hard_stop_danger_threshold;
    double slow_forward_confidence_threshold;
    double turn_confidence_threshold;
};
```

---

# 10.9 C++ — `NavigationThresholdConfig`

```cpp
struct NavigationThresholdConfig
{
    double min_direction_confidence;
    double min_corridor_confidence;
    double stop_score_threshold;
};
```

---

# 10.10 C++ — `FusedDecisionSignal`

```cpp
struct FusedDecisionSignal
{
    std::string pair_id;

    std::string steering_action;
    double target_heading_deg;
    double direction_confidence;

    double danger_score;
    double corridor_confidence;
};
```

---

# 10.11 C++ — `NavigationDecisionResult`

```cpp
enum class LocalNavigationCommand
{
    MOVE_FORWARD,
    MOVE_FORWARD_SLOW,
    TURN_LEFT_IN_PLACE,
    TURN_RIGHT_IN_PLACE,
    STOP_AND_WAIT,
    STOP_AND_REPLAN
};

struct NavigationDecisionResult
{
    std::string pair_id;
    NavigationSourceType algorithm_type;

    LocalNavigationCommand command;
    double final_confidence;
    std::string decision_reason;
};
```

---

# 10.12 C++ — `NavigationDecisionDebugRecord`

```cpp
struct NavigationDecisionDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.13 C++ — `BaseSignalFusionEngine`

Tạo abstract class:

```cpp
class BaseSignalFusionEngine
{
public:
    virtual FusedDecisionSignal fuse(
        const NavigationDecisionSample& sample
    ) const = 0;

    virtual ~BaseSignalFusionEngine() = default;
};
```

---

# 10.14 C++ — `SignalFusionEngine`

Kế thừa `BaseSignalFusionEngine`.

## Nhiệm vụ
- load direction / danger / corridor summaries
- tạo `FusedDecisionSignal`

---

# 10.15 C++ — `BaseNavigationPriorityEngine`

Tạo abstract class:

```cpp
class BaseNavigationPriorityEngine
{
public:
    virtual LocalNavigationCommand decide_command(
        const FusedDecisionSignal& signal,
        const NavigationPriorityConfig& priority_config,
        const NavigationThresholdConfig& threshold_config
    ) const = 0;

    virtual ~BaseNavigationPriorityEngine() = default;
};
```

---

# 10.16 C++ — `NavigationPriorityEngine`

Kế thừa `BaseNavigationPriorityEngine`.

## Nhiệm vụ
- áp dụng priority rules để chọn lệnh cuối

---

# 10.17 C++ — `BaseFinalNavigationDecisionEngine`

Tạo abstract class:

```cpp
class BaseFinalNavigationDecisionEngine
{
public:
    virtual NavigationDecisionResult build_result(
        const NavigationDecisionSample& sample,
        const FusedDecisionSignal& signal,
        LocalNavigationCommand command
    ) const = 0;

    virtual ~BaseFinalNavigationDecisionEngine() = default;
};
```

---

# 10.18 C++ — `FinalNavigationDecisionEngine`

Kế thừa `BaseFinalNavigationDecisionEngine`.

## Nhiệm vụ
- build `NavigationDecisionResult`

---

# 10.19 C++ — `BaseNavigationDecisionSummaryEngine`

Tạo abstract class:

```cpp
class BaseNavigationDecisionSummaryEngine
{
public:
    virtual std::string summarize(
        const FusedDecisionSignal& signal,
        const NavigationDecisionResult& result
    ) const = 0;

    virtual ~BaseNavigationDecisionSummaryEngine() = default;
};
```

---

# 10.20 C++ — `NavigationDecisionSummaryEngine`

Kế thừa `BaseNavigationDecisionSummaryEngine`.

## Nhiệm vụ
- tạo summary kiểu:
```text
The robot initially preferred steering left, but the danger score exceeded the hard-stop threshold.
Final command: STOP_AND_WAIT.
```

---

# 10.21 C++ — `ObstacleAwareLocalNavigationDecisionSystem`

Tạo class trung tâm:

```cpp
class ObstacleAwareLocalNavigationDecisionSystem
```

## Thuộc tính

```cpp
private:
    std::vector<NavigationDecisionSample> samples;
    NavigationPriorityConfig priority_config;
    NavigationThresholdConfig threshold_config;

    std::shared_ptr<BaseSignalFusionEngine> fusion_engine;
    std::shared_ptr<BaseNavigationPriorityEngine> priority_engine;
    std::shared_ptr<BaseFinalNavigationDecisionEngine> final_engine;
    std::shared_ptr<BaseNavigationDecisionSummaryEngine> summary_engine;

    std::vector<NavigationDecisionResult> results;
    std::stack<NavigationDecisionDebugRecord> debug_history;
```

## Hàm cần có

### `load_navigation_sample_config(const std::string& path)`
### `load_navigation_priority_config(const std::string& path)`
### `load_navigation_threshold_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each sample:
    fused_signal = fusion_engine.fuse(sample)
    command = priority_engine.decide_command(
        fused_signal,
        priority_config,
        threshold_config
    )
    result = final_engine.build_result(sample, fused_signal, command)
    result.decision_reason = summary_engine.summarize(fused_signal, result)
    save result
    push debug history
```

### `const std::vector<NavigationDecisionResult>& get_results() const`
### `std::vector<NavigationDecisionDebugRecord> get_debug_history_reverse()`

---

# 10.22 C++ — `NavigationDecisionReportWriter`

Tạo class:

```cpp
class NavigationDecisionReportWriter
```

## Hàm cần có

### `write_fused_signal_report(...)`
Ví dụ:

```text
[Fused Signal]
Pair: hallway_01
Steering Action: STEER_LEFT
Direction Confidence: 0.74
Danger Score: 81
Corridor Confidence: 0.68
```

### `write_navigation_decision_report(...)`
Ví dụ:

```text
[Navigation Decision]
Pair: hallway_01
Final Command: STOP_AND_WAIT
Final Confidence: 0.91
```

### `write_navigation_summary_report(...)`
Ví dụ:

```text
[Navigation Summary]
Pair: hallway_01
The robot initially preferred steering left, but the danger score exceeded the hard-stop threshold.
Final command: STOP_AND_WAIT.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.23 C++ — `main.cpp`

## Yêu cầu

```text
Load navigation sample config
Load navigation priority config
Load navigation threshold config
Load visualization config
Load runtime config

Create:
    SignalFusionEngine
    NavigationPriorityEngine
    FinalNavigationDecisionEngine
    NavigationDecisionSummaryEngine

Create ObstacleAwareLocalNavigationDecisionSystem
Run
Write reports
```

---

# 11. Luật local navigation decision của project

## Luật 1 — Final command phải xét danger override
Không được chỉ lấy thẳng kết quả từ Bài 59.

## Luật 2 — Phải có priority arbitration rõ ràng
Ví dụ:
- hard stop
- slow forward
- turn
- move forward

## Luật 3 — Nếu thiếu danger / corridor file thì phải có fallback logic
Không được crash vì input không đầy đủ.

## Luật 4 — Report phải giải thích vì sao lệnh cuối được chọn
Không chỉ ghi mỗi command.

## Luật 5 — Output phải usable cho navigation / locomotion layer
Tức là phải trả lời được:
- lệnh cuối cùng là gì?
- nó đến từ steering hay từ danger override?
- mức tự tin của quyết định cuối là bao nhiêu?

---

# 12. Output mong muốn

## Config
```text
config/navigation_decision_sample_config.txt
config/navigation_priority_config.txt
config/navigation_threshold_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/fused_signal_report.txt
assets/outputs/navigation_decision_report.txt
assets/outputs/navigation_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/navigation_command_plot.png
assets/outputs/decision_confidence_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho direction / danger / corridor inputs
- preview arbitration logic
- visualize final navigation commands

## C++
- fuse nhiều tín hiệu perception
- áp dụng priority rules
- tạo final local navigation command

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là bước perception + decision trả lời câu hỏi:

> **Robot nên thực hiện lệnh điều hướng cục bộ nào ngay bây giờ?**

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
→ local walking direction planning
→ obstacle-aware local navigation decision
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho navigation sample / priority / threshold / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / navigation result ids
- [ ] Python có NumPy navigation preview
- [ ] Python có Matplotlib navigation command / confidence plots
- [ ] C++ có `NavigationDecisionSample`
- [ ] C++ có `NavigationPriorityConfig`
- [ ] C++ có `NavigationThresholdConfig`
- [ ] C++ có `FusedDecisionSignal`
- [ ] C++ có `NavigationDecisionResult`
- [ ] C++ có `SignalFusionEngine`
- [ ] C++ có `NavigationPriorityEngine`
- [ ] C++ có `FinalNavigationDecisionEngine`
- [ ] C++ có `NavigationDecisionSummaryEngine`
- [ ] C++ có `ObstacleAwareLocalNavigationDecisionSystem`
- [ ] C++ ghi đủ report + plot outputs

---
