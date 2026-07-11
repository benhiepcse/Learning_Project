# 🤖 Bài 71: Humanoid Perception Mission Progress Monitor — Bộ giám sát tiến trình nhiệm vụ perception theo thời gian cho humanoid

> Mini Project số 71 trong **Đợt 14**  
> **Bài 71 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 70**.
>
> Nếu:
>
> - **Bài 69** đã chọn **mission mode**
> - **Bài 70** đã quản lý **mission state / retry / recovery**
>
> thì bước tiếp theo hợp lý nhất là:
>
> ```text
> current mission state
> + frame-by-frame progress signals
> + completion / stall / anomaly indicators
> → mission progress tracking
> → mission health monitoring
> → progress health report
> ```
>
> Đây là bước mà bạn không chỉ biết **mission đang ở state nào**, mà còn phải theo dõi:
>
> - mission có đang **tiến triển tốt** hay đang bị **stall**?
> - target follow có đang tiến gần hơn không hay robot bị đứng yên?
> - pick mission có đang đi từ `approach → align → grasp → lift` đúng nhịp không?
> - cleaning mission có đang xử lý từng clutter target đều đặn hay bị kẹt?
> - mission có dấu hiệu **anomaly / timeout / no-progress** không?
>
> Tức là dữ liệu bắt đầu usable cho:
> - mission progress monitoring
> - progress health scoring
> - stall / anomaly detection
> - perception-to-execution supervision handoff

---

# 📌 Mục lục

- [1. Vì sao Bài 71 xuất hiện sau Bài 70](#1-vì-sao-bài-71-xuất-hiện-sau-bài-70)
- [2. Bài 71 nâng từ Bài 70 lên chỗ nào](#2-bài-71-nâng-từ-bài-70-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Mission progress monitoring pipeline tổng thể](#5-mission-progress-monitoring-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật mission progress monitor của project](#10-luật-mission-progress-monitor-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 71 xuất hiện sau Bài 70

## Bài 70 đang làm gì?
Bài 70 đã giúp bạn:
- nhận `selected mission mode`
- nhận `previous mission state`
- nhận `mission event`
- quyết định **next mission state**
- xử lý **retry / recovery / failure**

## Nhưng còn thiếu gì?
Bài 70 mới trả lời:
- mission đang ở state nào
- khi có event thì state chuyển ra sao

Nó **chưa** trả lời:
- mission có đang **thực sự tiến triển** hay không
- mission bị **kẹt** ở state hiện tại bao lâu
- mission đang **tiến gần mục tiêu** hay chỉ lặp vô ích
- có dấu hiệu **anomaly** hay **no-progress** không

Ví dụ:
- follow mission ở `FOLLOWING_ACTIVE` nhưng khoảng cách tới target không giảm trong 40 frame
- pick mission ở `PICKING_ACTIVE` nhưng grasp phase không tiến sang lift phase
- cleaning mission ở `CLEANING_ACTIVE` nhưng số clutter còn lại không đổi

## Bài 71 lấp đúng chỗ đó
Bài 71 sẽ làm:

```text
current mission state
+ temporal progress signals
→ progress evaluation
→ stall / anomaly detection
→ mission health summary
```

Đây là bước chuyển từ **state machine** sang **mission progress supervision**.

---

# 2. Bài 71 nâng từ Bài 70 lên chỗ nào

## Bài 70
- quản lý state transition
- retry / recovery / failure handling

## Bài 71
- theo dõi **mission progress qua thời gian**
- tính **progress score / health score**
- phát hiện:
  - no-progress
  - stall
  - anomaly
  - repeated recovery loops
- sinh **mission progress health report**

### Nói ngắn gọn:
- **Bài 70** hỏi: “Mission đang ở state nào?”
- **Bài 71** hỏi: “Mission có đang tiến triển tốt trong state đó hay đang bị chậm / kẹt / bất thường?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Humanoid Perception Mission Progress Monitor**

System này nhận đầu vào là:
- một hoặc nhiều **mission progress scenes**
- mỗi scene chứa:
  - `selected_mode`
  - `current_state`
  - một chuỗi **frame-by-frame progress samples**
- một bộ **progress evaluation rules**
- một bộ **stall detection rules**
- một bộ **anomaly detection rules**
- optional:
  - expected progress phase
  - current retry count
  - current recovery count
  - mission urgency

## Mỗi progress sample tối thiểu chứa
- `scene_id`
- `selected_mode`
- `current_state`
- `frame_index`
- `progress_value`

Optional:
- `target_distance`
- `completed_steps`
- `remaining_targets`
- `retry_count`
- `recovery_count`
- `phase_name`

## Nhiệm vụ của system
### Với mỗi scene:
1. load progress sequence
2. inspect progress trend theo thời gian
3. compute **mission progress score**
4. compute **mission health status**
5. detect **stall / anomaly / repeated failure patterns**
6. ghi **mission progress health report**

<p align="center">
  <img src="../../images/project_71.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build mission-progress scene config
→ build progress / stall / anomaly configs
→ preview temporal progress trend bằng NumPy
→ visualize mission health distribution

C++
→ load mission progress sequences
→ compute progress trend / progress score
→ detect stall / anomaly
→ classify mission health
→ export progress monitoring reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **mission state machine** với **temporal supervision**
- biết cách thiết kế:
  - progress trend evaluation
  - stall detection
  - anomaly detection
  - mission health scoring
- chuẩn bị nền trực tiếp cho:
  - execution watchdog
  - task supervision
  - recovery trigger logic

---

# 5. Mission progress monitoring pipeline tổng thể

```text
Load Mission Progress Scene Config
Load Progress Rule Config
Load Stall Rule Config
Load Anomaly Rule Config
Load Visualization Config
Load Runtime Config

Create MissionProgressTrendEngine
Create MissionStallDetector
Create MissionAnomalyDetector
Create MissionHealthScoringEngine
Create MissionProgressSummaryEngine
Create HumanoidPerceptionMissionProgressMonitor

For each progress scene:
    1. load progress sequence
    2. compute trend features
    3. detect stall
    4. detect anomaly
    5. compute mission health
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
- vector / deque / smart pointer
- temporal trend analysis
- rolling window logic
- monitoring / watchdog style reasoning

## Computer Vision / Robotics Vision
- follow / pick / clean mission states
- temporal progress supervision
- mission health monitoring
- stall / anomaly detection
- execution monitoring for humanoid tasks

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **mission progress monitoring pipeline**:

```text
MissionProgressInput
→ TrendAnalysis
→ StallDetection
→ AnomalyDetection
→ HealthScoring
→ MissionProgressReport
```

## 2. Python BST
Lưu:
- `scene_id`
- `health_status`
- `mission_progress_result_id`

## 3. `std::vector<MissionProgressSample>`
Danh sách progress samples theo frame.

## 4. `std::vector<MissionProgressResult>`
Danh sách kết quả monitoring cho từng scene.

## 5. `std::unordered_map<std::string, std::vector<MissionProgressSample>>`
Group progress samples theo `scene_id`.

## 6. `std::deque<double>`
Dùng làm rolling window để kiểm tra progress gần đây.

## 7. `std::stack<MissionProgressDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Progress trend extraction
Từ chuỗi `progress_value` hoặc `target_distance`, tính:
- mean progress delta
- last-N frame progress delta
- monotonic improvement ratio
- stagnation frame count

Ví dụ:
- follow mission: khoảng cách target giảm dần → tốt
- pick mission: completed_steps tăng → tốt
- clean mission: remaining_targets giảm → tốt

---

## Algorithm 2 — Stall detection
Một mission bị stall nếu:
- trong `stall_window_size` frame gần nhất, progress gần như không tăng
- hoặc target distance không giảm đủ
- hoặc remaining_targets không đổi quá lâu

Ví dụ:
```text
if average_progress_delta < min_progress_delta
and stagnation_frames >= threshold:
    stall = true
```

---

## Algorithm 3 — Anomaly detection
Phát hiện anomaly nếu:
- progress_value giảm mạnh bất thường
- recovery_count tăng liên tục
- retry_count vượt ngưỡng
- target distance dao động bất thường
- state active kéo dài quá lâu nhưng không có completed_steps mới

---

## Algorithm 4 — Mission health scoring
Tạo **mission health score**:
```text
health_score =
    progress_score
  - stall_penalty
  - anomaly_penalty
  - retry_penalty
  - recovery_penalty
```

Sau đó classify:
- `HEALTHY`
- `WARNING`
- `CRITICAL`

---

## Algorithm 5 — Mission progress summary
Report cuối phải trả lời được:
- mission có tiến triển không
- có stall không
- có anomaly không
- health status là gì
- vì sao mission bị warning / critical

---

## Algorithm 6 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- progress trend plot
- health status distribution
- stall / anomaly summary chart

---

# 8. Cấu trúc folder

```text
mini_project_71_humanoid_perception_mission_progress_monitor/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ mission_progress_inputs/
│  │  ├─ progress_scene_01.txt
│  │  ├─ progress_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ mission_progress_report.txt
│     ├─ mission_progress_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ progress_trend_plot.png
│     └─ mission_health_distribution_plot.png
│
├─ config/
│  ├─ mission_progress_scene_config.txt
│  ├─ progress_rule_config.txt
│  ├─ stall_rule_config.txt
│  ├─ anomaly_rule_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ mission_progress_graph_preview.py
│     ├─ mission_progress_bst.py
│     ├─ numpy_mission_progress_preview.py
│     ├─ matplotlib_mission_progress_plotter.py
│     └─ synthetic_mission_progress_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ SelectedMissionMode.hpp
   │  ├─ MissionState.hpp
   │  ├─ MissionHealthStatus.hpp
   │  ├─ MissionProgressSample.hpp
   │  ├─ MissionProgressScene.hpp
   │  ├─ ProgressRuleConfig.hpp
   │  ├─ StallRuleConfig.hpp
   │  ├─ AnomalyRuleConfig.hpp
   │  ├─ MissionProgressResult.hpp
   │  ├─ MissionProgressDebugRecord.hpp
   │  ├─ BaseMissionProgressTrendEngine.hpp
   │  ├─ MissionProgressTrendEngine.hpp
   │  ├─ BaseMissionStallDetector.hpp
   │  ├─ MissionStallDetector.hpp
   │  ├─ BaseMissionAnomalyDetector.hpp
   │  ├─ MissionAnomalyDetector.hpp
   │  ├─ BaseMissionHealthScoringEngine.hpp
   │  ├─ MissionHealthScoringEngine.hpp
   │  ├─ BaseMissionProgressSummaryEngine.hpp
   │  ├─ MissionProgressSummaryEngine.hpp
   │  ├─ HumanoidPerceptionMissionProgressMonitor.hpp
   │  └─ MissionProgressReportWriter.hpp
   │
   └─ src/
      ├─ MissionProgressTrendEngine.cpp
      ├─ MissionStallDetector.cpp
      ├─ MissionAnomalyDetector.cpp
      ├─ MissionHealthScoringEngine.cpp
      ├─ MissionProgressSummaryEngine.cpp
      ├─ HumanoidPerceptionMissionProgressMonitor.cpp
      └─ MissionProgressReportWriter.cpp
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
mission_progress_scene_config_path
progress_rule_config_path
stall_rule_config_path
anomaly_rule_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `MissionProgressConfigBuilder`

Tạo class con:

```python
class MissionProgressConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_progress_sample(
    scene_id,
    selected_mode,
    current_state,
    frame_index,
    progress_value,
    target_distance=None,
    completed_steps=None,
    remaining_targets=None,
    retry_count=0,
    recovery_count=0,
    phase_name=None
)`

### `set_progress_rules(
    min_progress_delta,
    good_progress_threshold,
    monotonic_ratio_threshold
)`

### `set_stall_rules(
    stall_window_size,
    stagnation_frame_threshold,
    min_required_progress_delta
)`

### `set_anomaly_rules(
    max_retry_count,
    max_recovery_count,
    max_active_without_progress_frames
)`

### `set_visualization_options(
    enable_progress_trend_plot,
    enable_health_distribution_plot
)`

### `write_mission_progress_scene_config()`
### `write_progress_rule_config()`
### `write_stall_rule_config()`
### `write_anomaly_rule_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `mission_progress_graph_preview.py`

Tạo class:

```python
class MissionProgressGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
MissionProgressInput
→ TrendAnalysis
→ StallDetection
→ AnomalyDetection
→ HealthScoring
→ MissionProgressReport
```

---

# 9.4 Python — `mission_progress_bst.py`

Tạo BST cho:
- `scene_id`
- `health_status`
- `mission_progress_result_id`

---

# 9.5 Python — `numpy_mission_progress_preview.py`

Tạo class:

```python
class NumPyMissionProgressPreview:
```

## Hàm cần có

### `load_progress_scene(path)`
### `compute_progress_trend(scene, progress_rule_config)`
### `detect_stall(scene, stall_rule_config)`
### `detect_anomaly(scene, anomaly_rule_config)`
### `compute_health(scene, trend_result, stall_result, anomaly_result)`

---

# 9.6 Python — `matplotlib_mission_progress_plotter.py`

Tạo class:

```python
class MatplotlibMissionProgressPlotter:
```

## Hàm cần có

### `plot_progress_trend(scene_results, save_path)`
### `plot_health_distribution(results, save_path)`

---

# 9.7 C++ — `SelectedMissionMode`

```cpp
enum class SelectedMissionMode
{
    FOLLOW_MODE,
    PICK_MODE,
    CLEAN_MODE,
    IDLE_MODE
};
```

---

# 9.8 C++ — `MissionState`

```cpp
enum class MissionState
{
    WAITING,
    FOLLOWING_ACTIVE,
    PICKING_ACTIVE,
    CLEANING_ACTIVE,
    RECOVERY,
    RETRYING,
    PAUSED,
    FAILED,
    DONE,
    IDLE
};
```

---

# 9.9 C++ — `MissionHealthStatus`

```cpp
enum class MissionHealthStatus
{
    HEALTHY,
    WARNING,
    CRITICAL
};
```

---

# 9.10 C++ — `MissionProgressSample`

```cpp
struct MissionProgressSample
{
    std::string scene_id;
    SelectedMissionMode selected_mode;
    MissionState current_state;

    int frame_index;
    double progress_value;

    double target_distance;
    int completed_steps;
    int remaining_targets;

    int retry_count;
    int recovery_count;

    std::string phase_name;
};
```

---

# 9.11 C++ — `MissionProgressScene`

```cpp
struct MissionProgressScene
{
    std::string scene_id;
    std::vector<MissionProgressSample> samples;
};
```

---

# 9.12 C++ — `ProgressRuleConfig`

```cpp
struct ProgressRuleConfig
{
    double min_progress_delta;
    double good_progress_threshold;
    double monotonic_ratio_threshold;
};
```

---

# 9.13 C++ — `StallRuleConfig`

```cpp
struct StallRuleConfig
{
    int stall_window_size;
    int stagnation_frame_threshold;
    double min_required_progress_delta;
};
```

---

# 9.14 C++ — `AnomalyRuleConfig`

```cpp
struct AnomalyRuleConfig
{
    int max_retry_count;
    int max_recovery_count;
    int max_active_without_progress_frames;
};
```

---

# 9.15 C++ — `MissionProgressResult`

```cpp
struct MissionProgressResult
{
    std::string scene_id;

    double progress_score;
    bool stall_detected;
    bool anomaly_detected;

    MissionHealthStatus health_status;

    std::string summary;
};
```

---

# 9.16 C++ — `MissionProgressDebugRecord`

```cpp
struct MissionProgressDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.17 C++ — `BaseMissionProgressTrendEngine`

```cpp
class BaseMissionProgressTrendEngine
{
public:
    virtual double compute_progress_score(
        const std::vector<MissionProgressSample>& samples,
        const ProgressRuleConfig& config
    ) const = 0;

    virtual ~BaseMissionProgressTrendEngine() = default;
};
```

---

# 9.18 C++ — `BaseMissionStallDetector`

```cpp
class BaseMissionStallDetector
{
public:
    virtual bool detect(
        const std::vector<MissionProgressSample>& samples,
        const StallRuleConfig& config
    ) const = 0;

    virtual ~BaseMissionStallDetector() = default;
};
```

---

# 9.19 C++ — `BaseMissionAnomalyDetector`

```cpp
class BaseMissionAnomalyDetector
{
public:
    virtual bool detect(
        const std::vector<MissionProgressSample>& samples,
        const AnomalyRuleConfig& config
    ) const = 0;

    virtual ~BaseMissionAnomalyDetector() = default;
};
```

---

# 9.20 C++ — `BaseMissionHealthScoringEngine`

```cpp
class BaseMissionHealthScoringEngine
{
public:
    virtual MissionHealthStatus classify(
        double progress_score,
        bool stall_detected,
        bool anomaly_detected
    ) const = 0;

    virtual ~BaseMissionHealthScoringEngine() = default;
};
```

---

# 9.21 C++ — `BaseMissionProgressSummaryEngine`

```cpp
class BaseMissionProgressSummaryEngine
{
public:
    virtual std::string summarize(
        const MissionProgressResult& result
    ) const = 0;

    virtual ~BaseMissionProgressSummaryEngine() = default;
};
```

---

# 9.22 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `MissionProgressTrendEngine`
- `MissionStallDetector`
- `MissionAnomalyDetector`
- `MissionHealthScoringEngine`
- `MissionProgressSummaryEngine`
- `HumanoidPerceptionMissionProgressMonitor`
- `MissionProgressReportWriter`

---

# 9.23 C++ — `HumanoidPerceptionMissionProgressMonitor`

Tạo class trung tâm:

```cpp
class HumanoidPerceptionMissionProgressMonitor
```

## Thuộc tính

```cpp
private:
    std::vector<MissionProgressScene> scenes;

    ProgressRuleConfig progress_config;
    StallRuleConfig stall_config;
    AnomalyRuleConfig anomaly_config;

    std::shared_ptr<BaseMissionProgressTrendEngine> trend_engine;
    std::shared_ptr<BaseMissionStallDetector> stall_detector;
    std::shared_ptr<BaseMissionAnomalyDetector> anomaly_detector;
    std::shared_ptr<BaseMissionHealthScoringEngine> health_engine;
    std::shared_ptr<BaseMissionProgressSummaryEngine> summary_engine;

    std::vector<MissionProgressResult> results;
    std::stack<MissionProgressDebugRecord> debug_history;
```

## Hàm cần có

### `load_mission_progress_scene_config(const std::string& path)`
### `load_progress_rule_config(const std::string& path)`
### `load_stall_rule_config(const std::string& path)`
### `load_anomaly_rule_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each scene:
    progress_score = trend_engine.compute_progress_score(scene.samples, progress_config)
    stall = stall_detector.detect(scene.samples, stall_config)
    anomaly = anomaly_detector.detect(scene.samples, anomaly_config)
    health = health_engine.classify(progress_score, stall, anomaly)

    build MissionProgressResult
    summary = summary_engine.summarize(result)
    save result
    push debug history
```

### `const std::vector<MissionProgressResult>& get_results() const`
### `std::vector<MissionProgressDebugRecord> get_debug_history_reverse()`

---

# 9.24 C++ — `MissionProgressReportWriter`

Tạo class:

```cpp
class MissionProgressReportWriter
```

## Hàm cần có

### `write_mission_progress_report(...)`
Ví dụ:

```text
[Mission Progress]
Scene: progress_scene_01
Progress Score: 8.15
Stall Detected: NO
Anomaly Detected: NO
Health Status: HEALTHY
```

### `write_mission_progress_summary_report(...)`
Ví dụ:

```text
[Mission Progress Summary]
Scene: progress_scene_01
The mission is progressing steadily with no major stalls or anomalies.
Its recent progress trend remains healthy and the overall mission health is HEALTHY.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.25 C++ — `main.cpp`

## Yêu cầu

```text
Load mission progress scene config
Load progress rule config
Load stall rule config
Load anomaly rule config
Load visualization config
Load runtime config

Create:
    MissionProgressTrendEngine
    MissionStallDetector
    MissionAnomalyDetector
    MissionHealthScoringEngine
    MissionProgressSummaryEngine

Create HumanoidPerceptionMissionProgressMonitor
Run
Write reports
```

---

# 10. Luật mission progress monitor của project

## Luật 1 — Bài này phải thật sự là temporal monitoring layer
Không được chỉ đọc 1 sample rồi kết luận health.

## Luật 2 — Phải tách `trend`, `stall`, `anomaly`, `health`
Đây là 4 lớp logic khác nhau.

## Luật 3 — Phải có ít nhất một scene bị stall hoặc anomaly
Nếu tất cả đều HEALTHY thì bài sẽ quá yếu.

## Luật 4 — Kết quả cuối phải usable cho execution watchdog / supervisor
Tức là sau bài này bạn phải trả lời được:
- mission có tiến triển không?
- có stall / anomaly không?
- health status hiện tại là gì?

## Luật 5 — Report phải mang tinh thần “mission progress supervision”
Không chỉ là bảng điểm đơn giản.

---

# 11. Output mong muốn

## Config
```text
config/mission_progress_scene_config.txt
config/progress_rule_config.txt
config/stall_rule_config.txt
config/anomaly_rule_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/mission_progress_report.txt
assets/outputs/mission_progress_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/progress_trend_plot.png
assets/outputs/mission_health_distribution_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build mission-progress configs
- preview temporal progress behavior
- visualize health distribution

## C++
- đóng vai **mission progress watchdog**
- theo dõi tiến trình follow / pick / clean missions
- phát hiện stall / anomaly
- xuất progress-health reports

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Mission hiện tại có đang tiến triển tốt hay đang bị kẹt / bất thường theo thời gian?**

Pipeline lúc này sẽ là:

```text
mission state
→ temporal progress tracking
→ stall / anomaly detection
→ mission health scoring
→ supervisor-ready progress report
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho mission-progress scenes / progress / stall / anomaly / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / health status / mission result ids
- [ ] Python có NumPy mission-progress preview
- [ ] C++ có `SelectedMissionMode`
- [ ] C++ có `MissionState`
- [ ] C++ có `MissionHealthStatus`
- [ ] C++ có `MissionProgressSample`
- [ ] C++ có `MissionProgressScene`
- [ ] C++ có `ProgressRuleConfig`
- [ ] C++ có `StallRuleConfig`
- [ ] C++ có `AnomalyRuleConfig`
- [ ] C++ có `MissionProgressResult`
- [ ] C++ có `MissionProgressTrendEngine`
- [ ] C++ có `MissionStallDetector`
- [ ] C++ có `MissionAnomalyDetector`
- [ ] C++ có `MissionHealthScoringEngine`
- [ ] C++ có `MissionProgressSummaryEngine`
- [ ] C++ có `HumanoidPerceptionMissionProgressMonitor`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 71**, nếu bạn muốn vẫn bám Đợt 14 nhưng nâng từ “progress monitoring” sang “quyết định khi nào phải trigger recovery / replan”, bài hợp lý tiếp theo sẽ là:

# **Bài 72: Humanoid Perception Recovery Trigger & Replan Manager**

Ý tưởng:
```text
mission health result
+ stall / anomaly flags
+ mission context
→ recovery trigger decision
→ replan / retry / abort action
→ recovery action report
```

Tức là mạch sẽ đi:

```text
Bài 69: Humanoid Multi-Mode Perception Mission Orchestrator
→ Bài 70: Humanoid Perception Mission State Manager
→ Bài 71: Humanoid Perception Mission Progress Monitor
→ Bài 72: Humanoid Perception Recovery Trigger & Replan Manager
```
