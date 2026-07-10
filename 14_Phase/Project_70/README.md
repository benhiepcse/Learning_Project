# 🤖 Bài 70: Humanoid Perception Mission State Manager — Bộ quản lý trạng thái nhiệm vụ perception cho humanoid

> Mini Project số 70 trong **Đợt 14**  
> **Bài 70 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 69**.
>
> Nếu:
>
> - **Bài 69** đã chọn ra **mission mode cuối cùng** giữa `FOLLOW_MODE / PICK_MODE / CLEAN_MODE / IDLE_MODE`
>
> thì bước tiếp theo hợp lý nhất là:
>
> ```text
> selected mission mode
> + previous mission state
> + success / failure / interruption events
> → mission state transition
> → mission lifecycle management
> → mission state report
> ```
>
> Đây là bước mà bạn không chỉ biết **robot nên làm mode nào**, mà còn phải quản lý:
>
> - robot đang ở **trạng thái nhiệm vụ nào**?
> - khi mission thành công / thất bại / bị gián đoạn thì trạng thái chuyển ra sao?
> - khi FOLLOW_MODE đang chạy mà bị mất target thì có quay về `WAITING`, sang `RECOVERY`, hay `IDLE`?
> - khi PICK_MODE hoàn tất thì mission có chuyển sang `CLEANING_PREP`, `MISSION_DONE`, hay chờ mode mới?
>
> Tức là dữ liệu bắt đầu usable cho:
> - mission lifecycle management
> - state transition logic
> - recovery / interruption handling
> - perception-to-behavior state handoff

---

# 📌 Mục lục

- [1. Vì sao Bài 70 xuất hiện sau Bài 69](#1-vì-sao-bài-70-xuất-hiện-sau-bài-69)
- [2. Bài 70 nâng từ Bài 69 lên chỗ nào](#2-bài-70-nâng-từ-bài-69-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Mission state management pipeline tổng thể](#5-mission-state-management-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật mission state manager của project](#10-luật-mission-state-manager-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 70 xuất hiện sau Bài 69

## Bài 69 đang làm gì?
Bài 69 đã giúp bạn:
- gom 3 nhánh perception:
  - follow
  - pick
  - clean
- arbitrate mission mode
- chọn ra **selected mission mode**

Ví dụ output của Bài 69 có thể là:
- `FOLLOW_MODE`
- `PICK_MODE`
- `CLEAN_MODE`
- `IDLE_MODE`

## Nhưng còn thiếu gì?
Chọn mode xong **chưa đủ**. Robot còn cần biết:
- mode đó đang ở **giai đoạn nào**?
- mission có đang chạy, đang chờ, đã thành công, hay đang recovery?
- nếu mission fail thì có retry / fallback / abort không?

Nói cách khác, bạn đã có **“chọn nhiệm vụ”** nhưng chưa có **“quản lý vòng đời nhiệm vụ”**.

## Bài 70 lấp đúng chỗ đó
Bài 70 sẽ làm:

```text
selected mission mode
+ previous mission state
+ mission event
→ next mission state
→ state lifecycle summary
```

Đây là bước chuyển từ **mode selection** sang **mission lifecycle state management**.

---

# 2. Bài 70 nâng từ Bài 69 lên chỗ nào

## Bài 69
- chọn mode nhiệm vụ cuối cùng
- làm arbitration giữa follow / pick / clean / idle

## Bài 70
- nhận mode đã chọn
- theo dõi **mission state**
- cập nhật **state transition**
- xử lý:
  - success
  - failure
  - interruption
  - timeout
  - target lost
- sinh **mission lifecycle report**

### Nói ngắn gọn:
- **Bài 69** hỏi: “Robot nên chạy mode nào?”
- **Bài 70** hỏi: “Sau khi đã chọn mode, mission đang ở state nào và phải chuyển state ra sao khi sự kiện xảy ra?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Humanoid Perception Mission State Manager**

System này nhận đầu vào là:
- một hoặc nhiều **mission state scenes**
- mỗi scene chứa:
  - **selected mission mode**
  - **previous mission state**
  - **mission event**
- một bộ **state transition rules**
- một bộ **retry / recovery rules**
- optional:
  - failure count
  - timeout flag
  - interruption reason
  - mission priority

## Mỗi mission state sample tối thiểu chứa
- `scene_id`
- `selected_mode`
- `previous_state`
- `event_type`

Optional:
- `failure_count`
- `timeout_flag`
- `interruption_reason`
- `mission_priority`

## Nhiệm vụ của system
### Với mỗi scene:
1. load mission state input
2. inspect current mode + previous state + event
3. áp dụng **state transition rules**
4. áp dụng **retry / recovery rules** nếu cần
5. sinh **next mission state**
6. ghi **mission lifecycle report**

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build mission-state scene config
→ build transition / retry / recovery configs
→ preview state transitions bằng NumPy
→ visualize mission lifecycle distribution

C++
→ load selected mission mode + previous state + events
→ evaluate state transition
→ apply retry / recovery handling
→ produce next mission state
→ export mission lifecycle reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **mission selection** với **state machine quản lý vòng đời**
- biết cách thiết kế **mission state transitions**
- biết cách xử lý:
  - success
  - failure
  - interruption
  - recovery
- chuẩn bị nền trực tiếp cho:
  - behavior state machine
  - task execution monitoring
  - mission-level recovery logic

---

# 5. Mission state management pipeline tổng thể

```text
Load Mission State Scene Config
Load Transition Rule Config
Load Retry Rule Config
Load Recovery Rule Config
Load Visualization Config
Load Runtime Config

Create MissionStateTransitionEngine
Create MissionRetryDecisionEngine
Create MissionRecoveryEngine
Create MissionStateSummaryEngine
Create HumanoidPerceptionMissionStateManager

For each mission-state scene:
    1. load selected mission mode + previous state + event
    2. compute next mission state
    3. evaluate retry / recovery if needed
    4. write report

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
- state machine logic
- transition / retry / recovery handling

## Computer Vision / Robotics Vision
- follow / pick / clean mission outputs
- mission orchestration
- state machine for perception missions
- mission lifecycle management

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **mission state management pipeline**:

```text
MissionStateInput
→ TransitionRuleCheck
→ RetryDecision
→ RecoveryDecision
→ NextMissionState
→ MissionLifecycleReport
```

## 2. Python BST
Lưu:
- `scene_id`
- `mission_state`
- `mission_result_id`

## 3. `std::vector<MissionStateInput>`
Danh sách scene đầu vào.

## 4. `std::vector<MissionStateResult>`
Danh sách kết quả transition.

## 5. `std::unordered_map<std::string, std::vector<MissionStateResult>>`
Group theo `scene_id` hoặc `next_state`.

## 6. `std::stack<MissionStateDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — State transition
Tạo bảng / logic transition:

Ví dụ:
- `WAITING + FOLLOW_MODE_SELECTED` → `FOLLOWING_ACTIVE`
- `FOLLOWING_ACTIVE + TARGET_LOST` → `FOLLOWING_RECOVERY`
- `PICKING_ACTIVE + PICK_SUCCESS` → `MISSION_DONE`
- `CLEANING_ACTIVE + CLEAN_INTERRUPTED` → `MISSION_PAUSED`

---

## Algorithm 2 — Retry decision
Nếu event là failure:
- kiểm tra `failure_count`
- nếu chưa vượt ngưỡng retry thì vào `RETRYING`
- nếu vượt ngưỡng thì vào `FAILED`

---

## Algorithm 3 — Recovery decision
Nếu event là:
- `TARGET_LOST`
- `INTERRUPTED`
- `TIMEOUT`

thì có thể vào:
- `RECOVERY`
- `PAUSED`
- `ABORTED`
tùy rule config.

---

## Algorithm 4 — State summary
Report cuối phải trả lời được:
- previous state là gì
- event là gì
- next state là gì
- có retry / recovery hay không
- vì sao state chuyển như vậy

---

## Algorithm 5 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- state transition distribution
- event-to-next-state summary
- retry / recovery summary plot

---

# 8. Cấu trúc folder

```text
mini_project_70_humanoid_perception_mission_state_manager/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ mission_state_inputs/
│  │  ├─ mission_state_scene_01.txt
│  │  ├─ mission_state_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ mission_state_report.txt
│     ├─ mission_lifecycle_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ state_transition_plot.png
│     └─ retry_recovery_plot.png
│
├─ config/
│  ├─ mission_state_scene_config.txt
│  ├─ transition_rule_config.txt
│  ├─ retry_rule_config.txt
│  ├─ recovery_rule_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ mission_state_graph_preview.py
│     ├─ mission_state_bst.py
│     ├─ numpy_mission_state_preview.py
│     ├─ matplotlib_mission_state_plotter.py
│     └─ synthetic_mission_state_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ SelectedMissionMode.hpp
   │  ├─ MissionState.hpp
   │  ├─ MissionEventType.hpp
   │  ├─ MissionStateInput.hpp
   │  ├─ TransitionRuleConfig.hpp
   │  ├─ RetryRuleConfig.hpp
   │  ├─ RecoveryRuleConfig.hpp
   │  ├─ MissionStateResult.hpp
   │  ├─ MissionStateDebugRecord.hpp
   │  ├─ BaseMissionStateTransitionEngine.hpp
   │  ├─ MissionStateTransitionEngine.hpp
   │  ├─ BaseMissionRetryDecisionEngine.hpp
   │  ├─ MissionRetryDecisionEngine.hpp
   │  ├─ BaseMissionRecoveryEngine.hpp
   │  ├─ MissionRecoveryEngine.hpp
   │  ├─ BaseMissionStateSummaryEngine.hpp
   │  ├─ MissionStateSummaryEngine.hpp
   │  ├─ HumanoidPerceptionMissionStateManager.hpp
   │  └─ MissionStateReportWriter.hpp
   │
   └─ src/
      ├─ MissionStateTransitionEngine.cpp
      ├─ MissionRetryDecisionEngine.cpp
      ├─ MissionRecoveryEngine.cpp
      ├─ MissionStateSummaryEngine.cpp
      ├─ HumanoidPerceptionMissionStateManager.cpp
      └─ MissionStateReportWriter.cpp
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
mission_state_scene_config_path
transition_rule_config_path
retry_rule_config_path
recovery_rule_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `MissionStateConfigBuilder`

Tạo class con:

```python
class MissionStateConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_mission_state_scene(
    scene_id,
    selected_mode,
    previous_state,
    event_type,
    failure_count=0,
    timeout_flag=False,
    interruption_reason=None,
    mission_priority=None
)`

### `set_transition_rules(
    waiting_to_follow_state,
    waiting_to_pick_state,
    waiting_to_clean_state
)`

### `set_retry_rules(
    max_retry_count,
    retry_failure_events
)`

### `set_recovery_rules(
    timeout_to_state,
    target_lost_to_state,
    interruption_to_state
)`

### `set_visualization_options(
    enable_state_transition_plot,
    enable_retry_recovery_plot
)`

### `write_mission_state_scene_config()`
### `write_transition_rule_config()`
### `write_retry_rule_config()`
### `write_recovery_rule_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `mission_state_graph_preview.py`

Tạo class:

```python
class MissionStateGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
MissionStateInput
→ TransitionRuleCheck
→ RetryDecision
→ RecoveryDecision
→ NextMissionState
→ MissionLifecycleReport
```

---

# 9.4 Python — `mission_state_bst.py`

Tạo BST cho:
- `scene_id`
- `mission_state`
- `mission_result_id`

---

# 9.5 Python — `numpy_mission_state_preview.py`

Tạo class:

```python
class NumPyMissionStatePreview:
```

## Hàm cần có

### `load_mission_state_scene(path)`
### `apply_transition_rules(scene, transition_rule_config)`
### `apply_retry_rules(scene, retry_rule_config)`
### `apply_recovery_rules(scene, recovery_rule_config)`
### `build_state_transition_summary(results)`

---

# 9.6 Python — `matplotlib_mission_state_plotter.py`

Tạo class:

```python
class MatplotlibMissionStatePlotter:
```

## Hàm cần có

### `plot_state_transition_distribution(results, save_path)`
### `plot_retry_recovery_summary(results, save_path)`

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

# 9.9 C++ — `MissionEventType`

```cpp
enum class MissionEventType
{
    MODE_SELECTED,
    TARGET_LOST,
    PICK_SUCCESS,
    CLEAN_SUCCESS,
    FAILURE,
    INTERRUPTED,
    TIMEOUT_EVENT,
    NO_EVENT
};
```

---

# 9.10 C++ — `MissionStateInput`

```cpp
struct MissionStateInput
{
    std::string scene_id;
    SelectedMissionMode selected_mode;
    MissionState previous_state;
    MissionEventType event_type;

    int failure_count;
    bool timeout_flag;
    std::string interruption_reason;
    int mission_priority;
};
```

---

# 9.11 C++ — `TransitionRuleConfig`

```cpp
struct TransitionRuleConfig
{
    MissionState waiting_to_follow_state;
    MissionState waiting_to_pick_state;
    MissionState waiting_to_clean_state;
};
```

---

# 9.12 C++ — `RetryRuleConfig`

```cpp
struct RetryRuleConfig
{
    int max_retry_count;
    std::vector<MissionEventType> retry_failure_events;
};
```

---

# 9.13 C++ — `RecoveryRuleConfig`

```cpp
struct RecoveryRuleConfig
{
    MissionState timeout_to_state;
    MissionState target_lost_to_state;
    MissionState interruption_to_state;
};
```

---

# 9.14 C++ — `MissionStateResult`

```cpp
struct MissionStateResult
{
    std::string scene_id;

    SelectedMissionMode selected_mode;
    MissionState previous_state;
    MissionEventType event_type;

    MissionState next_state;

    bool used_retry;
    bool used_recovery;

    std::string summary;
};
```

---

# 9.15 C++ — `MissionStateDebugRecord`

```cpp
struct MissionStateDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.16 C++ — `BaseMissionStateTransitionEngine`

```cpp
class BaseMissionStateTransitionEngine
{
public:
    virtual MissionState compute_next_state(
        const MissionStateInput& input,
        const TransitionRuleConfig& config
    ) const = 0;

    virtual ~BaseMissionStateTransitionEngine() = default;
};
```

---

# 9.17 C++ — `BaseMissionRetryDecisionEngine`

```cpp
class BaseMissionRetryDecisionEngine
{
public:
    virtual bool should_retry(
        const MissionStateInput& input,
        const RetryRuleConfig& config
    ) const = 0;

    virtual ~BaseMissionRetryDecisionEngine() = default;
};
```

---

# 9.18 C++ — `BaseMissionRecoveryEngine`

```cpp
class BaseMissionRecoveryEngine
{
public:
    virtual bool should_recover(
        const MissionStateInput& input
    ) const = 0;

    virtual MissionState recovery_state(
        const MissionStateInput& input,
        const RecoveryRuleConfig& config
    ) const = 0;

    virtual ~BaseMissionRecoveryEngine() = default;
};
```

---

# 9.19 C++ — `BaseMissionStateSummaryEngine`

```cpp
class BaseMissionStateSummaryEngine
{
public:
    virtual std::string summarize(
        const MissionStateResult& result
    ) const = 0;

    virtual ~BaseMissionStateSummaryEngine() = default;
};
```

---

# 9.20 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `MissionStateTransitionEngine`
- `MissionRetryDecisionEngine`
- `MissionRecoveryEngine`
- `MissionStateSummaryEngine`
- `HumanoidPerceptionMissionStateManager`
- `MissionStateReportWriter`

---

# 9.21 C++ — `HumanoidPerceptionMissionStateManager`

Tạo class trung tâm:

```cpp
class HumanoidPerceptionMissionStateManager
```

## Thuộc tính

```cpp
private:
    std::vector<MissionStateInput> scenes;

    TransitionRuleConfig transition_config;
    RetryRuleConfig retry_config;
    RecoveryRuleConfig recovery_config;

    std::shared_ptr<BaseMissionStateTransitionEngine> transition_engine;
    std::shared_ptr<BaseMissionRetryDecisionEngine> retry_engine;
    std::shared_ptr<BaseMissionRecoveryEngine> recovery_engine;
    std::shared_ptr<BaseMissionStateSummaryEngine> summary_engine;

    std::vector<MissionStateResult> results;
    std::stack<MissionStateDebugRecord> debug_history;
```

## Hàm cần có

### `load_mission_state_scene_config(const std::string& path)`
### `load_transition_rule_config(const std::string& path)`
### `load_retry_rule_config(const std::string& path)`
### `load_recovery_rule_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each scene:
    next_state = transition_engine.compute_next_state(scene, transition_config)

    used_retry = retry_engine.should_retry(scene, retry_config)
    used_recovery = recovery_engine.should_recover(scene)

    if used_retry:
        next_state = RETRYING
    else if used_recovery:
        next_state = recovery_engine.recovery_state(scene, recovery_config)

    build MissionStateResult
    summary = summary_engine.summarize(result)
    save result
    push debug history
```

### `const std::vector<MissionStateResult>& get_results() const`
### `std::vector<MissionStateDebugRecord> get_debug_history_reverse()`

---

# 9.22 C++ — `MissionStateReportWriter`

Tạo class:

```cpp
class MissionStateReportWriter
```

## Hàm cần có

### `write_mission_state_report(...)`
Ví dụ:

```text
[Mission State]
Scene: mission_state_scene_01
Selected Mode: FOLLOW_MODE
Previous State: FOLLOWING_ACTIVE
Event: TARGET_LOST
Next State: RECOVERY
Retry Used: NO
Recovery Used: YES
```

### `write_mission_lifecycle_summary_report(...)`
Ví dụ:

```text
[Mission Lifecycle Summary]
Scene: mission_state_scene_01
The mission was previously in FOLLOWING_ACTIVE. Because the target was lost,
the state manager moved the mission into RECOVERY instead of DONE or FAILED.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.23 C++ — `main.cpp`

## Yêu cầu

```text
Load mission state scene config
Load transition rule config
Load retry rule config
Load recovery rule config
Load visualization config
Load runtime config

Create:
    MissionStateTransitionEngine
    MissionRetryDecisionEngine
    MissionRecoveryEngine
    MissionStateSummaryEngine

Create HumanoidPerceptionMissionStateManager
Run
Write reports
```

---

# 10. Luật mission state manager của project

## Luật 1 — Bài này phải thật sự là state-machine layer
Không được chỉ map event → output string một cách sơ sài.

## Luật 2 — Phải tách `transition`, `retry`, `recovery`
Đây là 3 lớp logic khác nhau.

## Luật 3 — Phải có ít nhất một event gây recovery / retry / failure
Nếu tất cả scene chỉ là success path thì bài sẽ quá yếu.

## Luật 4 — Kết quả cuối phải usable cho behavior / execution manager
Tức là sau bài này bạn phải trả lời được:
- mission đang ở state nào?
- vì sao nó chuyển state?
- có retry / recovery hay không?

## Luật 5 — Report phải mang tinh thần “mission lifecycle management”
Không chỉ là bảng chuyển trạng thái đơn giản.

---

# 11. Output mong muốn

## Config
```text
config/mission_state_scene_config.txt
config/transition_rule_config.txt
config/retry_rule_config.txt
config/recovery_rule_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/mission_state_report.txt
assets/outputs/mission_lifecycle_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/state_transition_plot.png
assets/outputs/retry_recovery_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build mission-state configs
- preview transition / retry / recovery behavior
- visualize mission lifecycle distribution

## C++
- đóng vai **mission lifecycle manager**
- cập nhật state của follow / pick / clean missions
- xử lý retry / recovery / failure
- xuất mission-state reports

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Sau khi humanoid chọn mission mode, nhiệm vụ đó đang ở trạng thái nào, và phải chuyển trạng thái ra sao khi có success / failure / interruption?**

Pipeline lúc này sẽ là:

```text
selected mission mode
→ mission event monitoring
→ state transition / retry / recovery
→ mission lifecycle state
→ behavior-ready state report
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho mission-state scenes / transition / retry / recovery / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / mission states / mission result ids
- [ ] Python có NumPy mission-state preview
- [ ] C++ có `SelectedMissionMode`
- [ ] C++ có `MissionState`
- [ ] C++ có `MissionEventType`
- [ ] C++ có `MissionStateInput`
- [ ] C++ có `TransitionRuleConfig`
- [ ] C++ có `RetryRuleConfig`
- [ ] C++ có `RecoveryRuleConfig`
- [ ] C++ có `MissionStateResult`
- [ ] C++ có `MissionStateTransitionEngine`
- [ ] C++ có `MissionRetryDecisionEngine`
- [ ] C++ có `MissionRecoveryEngine`
- [ ] C++ có `MissionStateSummaryEngine`
- [ ] C++ có `HumanoidPerceptionMissionStateManager`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 70**, nếu bạn muốn vẫn bám chặt Đợt 14 nhưng nâng từ “state machine” sang “giám sát tiến trình nhiệm vụ theo thời gian”, bài hợp lý tiếp theo sẽ là:

# **Bài 71: Humanoid Perception Mission Progress Monitor**

Ý tưởng:
```text
current mission state
+ frame-by-frame progress signals
+ completion / stall / anomaly indicators
→ mission progress tracking
→ progress health report
```

Tức là mạch sẽ đi:

```text
Bài 69: Humanoid Multi-Mode Perception Mission Orchestrator
→ Bài 70: Humanoid Perception Mission State Manager
→ Bài 71: Humanoid Perception Mission Progress Monitor
```
