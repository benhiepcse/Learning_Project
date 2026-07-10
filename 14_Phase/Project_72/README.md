# 🤖 Bài 72: Humanoid Perception Recovery Trigger & Replan Manager — Bộ kích hoạt recovery và tái lập kế hoạch cho nhiệm vụ perception của humanoid

> Mini Project số 72 trong **Đợt 14**  
> **Bài 72 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 71**.
>
> Nếu:
>
> - **Bài 69** đã chọn **mission mode**
> - **Bài 70** đã quản lý **mission state / retry / recovery**
> - **Bài 71** đã giám sát **progress / stall / anomaly / health**
>
> thì bước tiếp theo hợp lý nhất là:
>
> ```text
> mission health result
> + stall / anomaly flags
> + mission context
> → recovery trigger decision
> → retry / replan / fallback / abort action
> → recovery action report
> ```
>
> Đây là bước mà bạn không chỉ biết **mission đang khỏe hay không**, mà còn phải quyết định:
>
> - khi nào cần **trigger recovery** thật sự?
> - khi nào chỉ cần **retry nhẹ**?
> - khi nào phải **replan target / replan mission**?
> - khi nào phải **fallback sang mode khác**?
> - khi nào phải **abort mission**?
>
> Tức là dữ liệu bắt đầu usable cho:
> - recovery trigger logic
> - mission replan decision
> - fallback / abort policy
> - perception-to-execution recovery handoff

---

# 📌 Mục lục

- [1. Vì sao Bài 72 xuất hiện sau Bài 71](#1-vì-sao-bài-72-xuất-hiện-sau-bài-71)
- [2. Bài 72 nâng từ Bài 71 lên chỗ nào](#2-bài-72-nâng-từ-bài-71-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Recovery trigger & replan pipeline tổng thể](#5-recovery-trigger--replan-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật recovery trigger manager của project](#10-luật-recovery-trigger-manager-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 72 xuất hiện sau Bài 71

## Bài 71 đang làm gì?
Bài 71 đã giúp bạn:
- theo dõi progress của mission theo thời gian
- phát hiện **stall**
- phát hiện **anomaly**
- chấm **mission health**

Ví dụ output của Bài 71:
- `HEALTHY`
- `WARNING`
- `CRITICAL`
- `stall_detected = true`
- `anomaly_detected = true`

## Nhưng còn thiếu gì?
Bài 71 mới chỉ là **monitoring layer**.  
Nó cho bạn biết:
- mission đang khỏe hay không
- mission có bị kẹt hay không

Nhưng nó **chưa** quyết định:
- robot phải **làm gì tiếp theo**?

Ví dụ:
- nếu follow mission bị stall 30 frame thì có retry tracking không?
- nếu pick mission anomaly liên tục thì có replan pick candidate không?
- nếu cleaning mission critical quá lâu thì có abort không?
- nếu follow mode thất bại thì có fallback sang clean / pick / idle không?

## Bài 72 lấp đúng chỗ đó
Bài 72 sẽ làm:

```text
mission health + context
→ recovery trigger decision
→ retry / replan / fallback / abort
→ recovery action summary
```

Đây là bước chuyển từ **mission health monitoring** sang **actionable recovery decision**.

---

# 2. Bài 72 nâng từ Bài 71 lên chỗ nào

## Bài 71
- đánh giá progress
- phát hiện stall / anomaly
- classify health status

## Bài 72
- nhận health status + stall/anomaly signals
- quyết định **có trigger recovery hay không**
- chọn **loại hành động recovery**
- sinh **replan / fallback / abort action**
- ghi **recovery action report**

### Nói ngắn gọn:
- **Bài 71** hỏi: “Mission có khỏe không?”
- **Bài 72** hỏi: “Nếu mission không khỏe, robot phải recovery / retry / replan / abort như thế nào?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Humanoid Perception Recovery Trigger & Replan Manager**

System này nhận đầu vào là:
- một hoặc nhiều **recovery scenes**
- mỗi scene chứa:
  - `selected_mode`
  - `current_state`
  - `health_status`
  - `stall_detected`
  - `anomaly_detected`
- một bộ **recovery trigger rules**
- một bộ **replan rules**
- một bộ **fallback rules**
- optional:
  - retry count
  - recovery count
  - current mission priority
  - last selected mode
  - target lost count

## Mỗi recovery sample tối thiểu chứa
- `scene_id`
- `selected_mode`
- `current_state`
- `health_status`
- `stall_detected`
- `anomaly_detected`

Optional:
- `retry_count`
- `recovery_count`
- `mission_priority`
- `target_lost_count`
- `last_action`

## Nhiệm vụ của system
### Với mỗi scene:
1. load recovery scene input
2. inspect health / stall / anomaly / state context
3. decide whether to **trigger recovery**
4. choose recovery action:
   - retry
   - replan
   - fallback
   - abort
   - continue
5. build **recovery action result**
6. ghi **recovery decision report**

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build recovery-scene config
→ build trigger / replan / fallback configs
→ preview recovery decisions bằng NumPy
→ visualize recovery action distribution

C++
→ load recovery scene inputs
→ evaluate recovery trigger conditions
→ decide retry / replan / fallback / abort
→ export recovery decision reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **mission monitoring** với **recovery decision**
- biết cách tách:
  - trigger condition
  - recovery action selection
  - fallback selection
  - abort threshold
- chuẩn bị nền trực tiếp cho:
  - execution recovery policy
  - autonomous replanning
  - task supervisor escalation

---

# 5. Recovery trigger & replan pipeline tổng thể

```text
Load Recovery Scene Config
Load Recovery Trigger Rule Config
Load Replan Rule Config
Load Fallback Rule Config
Load Visualization Config
Load Runtime Config

Create RecoveryTriggerEngine
Create RecoveryActionSelectionEngine
Create ReplanDecisionEngine
Create FallbackDecisionEngine
Create RecoverySummaryEngine
Create HumanoidPerceptionRecoveryTriggerAndReplanManager

For each recovery scene:
    1. load recovery input
    2. decide if recovery should trigger
    3. choose retry / replan / fallback / abort / continue
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
- vector / smart pointer
- rule-based decision engine
- fallback / abort / retry reasoning

## Computer Vision / Robotics Vision
- mission monitoring
- recovery trigger logic
- mission replanning
- failure handling
- perception-to-execution recovery policies

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **recovery trigger pipeline**:

```text
RecoverySceneInput
→ TriggerEvaluation
→ RecoveryActionSelection
→ ReplanOrFallbackDecision
→ RecoveryReport
```

## 2. Python BST
Lưu:
- `scene_id`
- `recovery_action`
- `recovery_result_id`

## 3. `std::vector<RecoverySceneInput>`
Danh sách recovery scenes.

## 4. `std::vector<RecoveryDecisionResult>`
Danh sách kết quả recovery.

## 5. `std::unordered_map<std::string, std::vector<RecoveryDecisionResult>>`
Group theo `scene_id` hoặc `recovery_action`.

## 6. `std::stack<RecoveryDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Recovery trigger evaluation
Trigger recovery nếu:
- `health_status == CRITICAL`
- hoặc `stall_detected == true` trong state active
- hoặc `anomaly_detected == true` với severity cao
- hoặc retry/recovery count vượt ngưỡng warning

Ví dụ:
```text
if health == CRITICAL:
    trigger_recovery = true
elif stall and current_state in ACTIVE_STATES:
    trigger_recovery = true
elif anomaly and retry_count > threshold:
    trigger_recovery = true
```

---

## Algorithm 2 — Recovery action selection
Nếu recovery được trigger, chọn action:

- `RETRY_ACTION`
  - khi failure nhẹ, retry count còn thấp
- `REPLAN_ACTION`
  - khi target hiện tại không còn phù hợp
  - hoặc mission bị stall kéo dài
- `FALLBACK_ACTION`
  - khi mode hiện tại không cứu được, chuyển sang mode khác
- `ABORT_ACTION`
  - khi mission quá nhiều failure / recovery / anomaly
- `CONTINUE_ACTION`
  - nếu health vẫn đủ tốt

---

## Algorithm 3 — Replan decision
Ví dụ:
- follow mode stall lâu → replan target tracking
- pick mode anomaly → replan pick candidate
- clean mode cluster không giảm → replan cleaning schedule

---

## Algorithm 4 — Fallback decision
Ví dụ:
- FOLLOW_MODE fail → fallback PICK_MODE hoặc IDLE_MODE
- PICK_MODE fail → fallback CLEAN_MODE hoặc IDLE_MODE
- CLEAN_MODE fail → fallback IDLE_MODE

---

## Algorithm 5 — Recovery summary
Report cuối phải trả lời được:
- recovery có được trigger không
- action được chọn là gì
- vì sao chọn retry / replan / fallback / abort
- fallback mode là gì nếu có

---

## Algorithm 6 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- recovery action distribution
- trigger reason summary
- scene-wise recovery report

---

# 8. Cấu trúc folder

```text
mini_project_72_humanoid_perception_recovery_trigger_replan_manager/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ recovery_scene_inputs/
│  │  ├─ recovery_scene_01.txt
│  │  ├─ recovery_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ recovery_decision_report.txt
│     ├─ recovery_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ recovery_action_distribution_plot.png
│     └─ trigger_reason_plot.png
│
├─ config/
│  ├─ recovery_scene_config.txt
│  ├─ recovery_trigger_rule_config.txt
│  ├─ replan_rule_config.txt
│  ├─ fallback_rule_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ recovery_graph_preview.py
│     ├─ recovery_bst.py
│     ├─ numpy_recovery_preview.py
│     ├─ matplotlib_recovery_plotter.py
│     └─ synthetic_recovery_scene_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ SelectedMissionMode.hpp
   │  ├─ MissionState.hpp
   │  ├─ MissionHealthStatus.hpp
   │  ├─ RecoveryAction.hpp
   │  ├─ RecoverySceneInput.hpp
   │  ├─ RecoveryTriggerRuleConfig.hpp
   │  ├─ ReplanRuleConfig.hpp
   │  ├─ FallbackRuleConfig.hpp
   │  ├─ RecoveryDecisionResult.hpp
   │  ├─ RecoveryDebugRecord.hpp
   │  ├─ BaseRecoveryTriggerEngine.hpp
   │  ├─ RecoveryTriggerEngine.hpp
   │  ├─ BaseRecoveryActionSelectionEngine.hpp
   │  ├─ RecoveryActionSelectionEngine.hpp
   │  ├─ BaseReplanDecisionEngine.hpp
   │  ├─ ReplanDecisionEngine.hpp
   │  ├─ BaseFallbackDecisionEngine.hpp
   │  ├─ FallbackDecisionEngine.hpp
   │  ├─ BaseRecoverySummaryEngine.hpp
   │  ├─ RecoverySummaryEngine.hpp
   │  ├─ HumanoidPerceptionRecoveryTriggerAndReplanManager.hpp
   │  └─ RecoveryReportWriter.hpp
   │
   └─ src/
      ├─ RecoveryTriggerEngine.cpp
      ├─ RecoveryActionSelectionEngine.cpp
      ├─ ReplanDecisionEngine.cpp
      ├─ FallbackDecisionEngine.cpp
      ├─ RecoverySummaryEngine.cpp
      ├─ HumanoidPerceptionRecoveryTriggerAndReplanManager.cpp
      └─ RecoveryReportWriter.cpp
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
recovery_scene_config_path
recovery_trigger_rule_config_path
replan_rule_config_path
fallback_rule_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `RecoveryConfigBuilder`

Tạo class con:

```python
class RecoveryConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_recovery_scene(
    scene_id,
    selected_mode,
    current_state,
    health_status,
    stall_detected,
    anomaly_detected,
    retry_count=0,
    recovery_count=0,
    mission_priority=0,
    target_lost_count=0,
    last_action=None
)`

### `set_recovery_trigger_rules(
    critical_health_triggers_recovery,
    stall_active_triggers_recovery,
    anomaly_retry_threshold
)`

### `set_replan_rules(
    max_stall_before_replan,
    max_target_lost_before_replan,
    allow_mode_specific_replan
)`

### `set_fallback_rules(
    enable_fallback,
    max_recovery_before_abort,
    fallback_order
)`

### `set_visualization_options(
    enable_recovery_distribution_plot,
    enable_trigger_reason_plot
)`

### `write_recovery_scene_config()`
### `write_recovery_trigger_rule_config()`
### `write_replan_rule_config()`
### `write_fallback_rule_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `recovery_graph_preview.py`

Tạo class:

```python
class RecoveryGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
RecoverySceneInput
→ TriggerEvaluation
→ RecoveryActionSelection
→ ReplanOrFallbackDecision
→ RecoveryReport
```

---

# 9.4 Python — `recovery_bst.py`

Tạo BST cho:
- `scene_id`
- `recovery_action`
- `recovery_result_id`

---

# 9.5 Python — `numpy_recovery_preview.py`

Tạo class:

```python
class NumPyRecoveryPreview:
```

## Hàm cần có

### `load_recovery_scene(path)`
### `evaluate_trigger(scene, recovery_trigger_rule_config)`
### `select_recovery_action(scene, trigger_result, replan_rule_config, fallback_rule_config)`
### `build_recovery_summary(results)`

---

# 9.6 Python — `matplotlib_recovery_plotter.py`

Tạo class:

```python
class MatplotlibRecoveryPlotter:
```

## Hàm cần có

### `plot_recovery_action_distribution(results, save_path)`
### `plot_trigger_reason_distribution(results, save_path)`

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

# 9.10 C++ — `RecoveryAction`

```cpp
enum class RecoveryAction
{
    CONTINUE_ACTION,
    RETRY_ACTION,
    REPLAN_ACTION,
    FALLBACK_ACTION,
    ABORT_ACTION
};
```

---

# 9.11 C++ — `RecoverySceneInput`

```cpp
struct RecoverySceneInput
{
    std::string scene_id;

    SelectedMissionMode selected_mode;
    MissionState current_state;
    MissionHealthStatus health_status;

    bool stall_detected;
    bool anomaly_detected;

    int retry_count;
    int recovery_count;
    int mission_priority;
    int target_lost_count;

    std::string last_action;
};
```

---

# 9.12 C++ — `RecoveryTriggerRuleConfig`

```cpp
struct RecoveryTriggerRuleConfig
{
    bool critical_health_triggers_recovery;
    bool stall_active_triggers_recovery;
    int anomaly_retry_threshold;
};
```

---

# 9.13 C++ — `ReplanRuleConfig`

```cpp
struct ReplanRuleConfig
{
    int max_stall_before_replan;
    int max_target_lost_before_replan;
    bool allow_mode_specific_replan;
};
```

---

# 9.14 C++ — `FallbackRuleConfig`

```cpp
struct FallbackRuleConfig
{
    bool enable_fallback;
    int max_recovery_before_abort;
    std::vector<SelectedMissionMode> fallback_order;
};
```

---

# 9.15 C++ — `RecoveryDecisionResult`

```cpp
struct RecoveryDecisionResult
{
    std::string scene_id;

    bool recovery_triggered;
    RecoveryAction recovery_action;
    SelectedMissionMode fallback_mode;

    std::string summary;
};
```

---

# 9.16 C++ — `RecoveryDebugRecord`

```cpp
struct RecoveryDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.17 C++ — `BaseRecoveryTriggerEngine`

```cpp
class BaseRecoveryTriggerEngine
{
public:
    virtual bool should_trigger(
        const RecoverySceneInput& input,
        const RecoveryTriggerRuleConfig& config
    ) const = 0;

    virtual ~BaseRecoveryTriggerEngine() = default;
};
```

---

# 9.18 C++ — `BaseRecoveryActionSelectionEngine`

```cpp
class BaseRecoveryActionSelectionEngine
{
public:
    virtual RecoveryAction select_action(
        const RecoverySceneInput& input,
        bool recovery_triggered,
        const ReplanRuleConfig& replan_config,
        const FallbackRuleConfig& fallback_config
    ) const = 0;

    virtual ~BaseRecoveryActionSelectionEngine() = default;
};
```

---

# 9.19 C++ — `BaseReplanDecisionEngine`

```cpp
class BaseReplanDecisionEngine
{
public:
    virtual bool should_replan(
        const RecoverySceneInput& input,
        const ReplanRuleConfig& config
    ) const = 0;

    virtual ~BaseReplanDecisionEngine() = default;
};
```

---

# 9.20 C++ — `BaseFallbackDecisionEngine`

```cpp
class BaseFallbackDecisionEngine
{
public:
    virtual SelectedMissionMode choose_fallback(
        const RecoverySceneInput& input,
        const FallbackRuleConfig& config
    ) const = 0;

    virtual ~BaseFallbackDecisionEngine() = default;
};
```

---

# 9.21 C++ — `BaseRecoverySummaryEngine`

```cpp
class BaseRecoverySummaryEngine
{
public:
    virtual std::string summarize(
        const RecoveryDecisionResult& result
    ) const = 0;

    virtual ~BaseRecoverySummaryEngine() = default;
};
```

---

# 9.22 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `RecoveryTriggerEngine`
- `RecoveryActionSelectionEngine`
- `ReplanDecisionEngine`
- `FallbackDecisionEngine`
- `RecoverySummaryEngine`
- `HumanoidPerceptionRecoveryTriggerAndReplanManager`
- `RecoveryReportWriter`

---

# 9.23 C++ — `HumanoidPerceptionRecoveryTriggerAndReplanManager`

Tạo class trung tâm:

```cpp
class HumanoidPerceptionRecoveryTriggerAndReplanManager
```

## Thuộc tính

```cpp
private:
    std::vector<RecoverySceneInput> scenes;

    RecoveryTriggerRuleConfig trigger_config;
    ReplanRuleConfig replan_config;
    FallbackRuleConfig fallback_config;

    std::shared_ptr<BaseRecoveryTriggerEngine> trigger_engine;
    std::shared_ptr<BaseRecoveryActionSelectionEngine> action_engine;
    std::shared_ptr<BaseReplanDecisionEngine> replan_engine;
    std::shared_ptr<BaseFallbackDecisionEngine> fallback_engine;
    std::shared_ptr<BaseRecoverySummaryEngine> summary_engine;

    std::vector<RecoveryDecisionResult> results;
    std::stack<RecoveryDebugRecord> debug_history;
```

## Hàm cần có

### `load_recovery_scene_config(const std::string& path)`
### `load_recovery_trigger_rule_config(const std::string& path)`
### `load_replan_rule_config(const std::string& path)`
### `load_fallback_rule_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each scene:
    triggered = trigger_engine.should_trigger(scene, trigger_config)

    action = action_engine.select_action(
        scene,
        triggered,
        replan_config,
        fallback_config
    )

    if action == FALLBACK_ACTION:
        fallback_mode = fallback_engine.choose_fallback(scene, fallback_config)
    else:
        fallback_mode = scene.selected_mode

    build RecoveryDecisionResult
    summary = summary_engine.summarize(result)
    save result
    push debug history
```

### `const std::vector<RecoveryDecisionResult>& get_results() const`
### `std::vector<RecoveryDebugRecord> get_debug_history_reverse()`

---

# 9.24 C++ — `RecoveryReportWriter`

Tạo class:

```cpp
class RecoveryReportWriter
```

## Hàm cần có

### `write_recovery_decision_report(...)`
Ví dụ:

```text
[Recovery Decision]
Scene: recovery_scene_01
Recovery Triggered: YES
Action: REPLAN_ACTION
Fallback Mode: FOLLOW_MODE
```

### `write_recovery_summary_report(...)`
Ví dụ:

```text
[Recovery Summary]
Scene: recovery_scene_01
Recovery was triggered because the mission health was CRITICAL and the mission had stalled.
The manager selected REPLAN_ACTION instead of RETRY_ACTION because the target-lost count exceeded the replan threshold.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.25 C++ — `main.cpp`

## Yêu cầu

```text
Load recovery scene config
Load recovery trigger rule config
Load replan rule config
Load fallback rule config
Load visualization config
Load runtime config

Create:
    RecoveryTriggerEngine
    RecoveryActionSelectionEngine
    ReplanDecisionEngine
    FallbackDecisionEngine
    RecoverySummaryEngine

Create HumanoidPerceptionRecoveryTriggerAndReplanManager
Run
Write reports
```

---

# 10. Luật recovery trigger manager của project

## Luật 1 — Bài này phải thật sự là decision layer
Không được chỉ map health status → action một cách sơ sài.

## Luật 2 — Phải tách `trigger`, `action`, `replan`, `fallback`
Đây là 4 lớp logic khác nhau.

## Luật 3 — Phải có ít nhất một scene đi tới `REPLAN_ACTION` hoặc `FALLBACK_ACTION`
Nếu tất cả chỉ CONTINUE thì bài sẽ quá yếu.

## Luật 4 — Kết quả cuối phải usable cho execution recovery supervisor
Tức là sau bài này bạn phải trả lời được:
- có trigger recovery không?
- action tiếp theo là retry / replan / fallback / abort?
- fallback mode là gì nếu có?

## Luật 5 — Report phải mang tinh thần “recovery action decision”
Không chỉ là bảng health đơn giản.

---

# 11. Output mong muốn

## Config
```text
config/recovery_scene_config.txt
config/recovery_trigger_rule_config.txt
config/replan_rule_config.txt
config/fallback_rule_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/recovery_decision_report.txt
assets/outputs/recovery_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/recovery_action_distribution_plot.png
assets/outputs/trigger_reason_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build recovery-scene configs
- preview recovery / replan / fallback decisions
- visualize recovery action distribution

## C++
- đóng vai **recovery decision supervisor**
- quyết định retry / replan / fallback / abort
- xuất recovery action reports

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Nếu mission hiện tại bị stall / anomaly / critical health, robot phải recovery như thế nào: retry, replan, fallback hay abort?**

Pipeline lúc này sẽ là:

```text
mission health result
→ recovery trigger evaluation
→ retry / replan / fallback / abort selection
→ recovery action report
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho recovery scenes / trigger / replan / fallback / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / recovery actions / result ids
- [ ] Python có NumPy recovery preview
- [ ] C++ có `SelectedMissionMode`
- [ ] C++ có `MissionState`
- [ ] C++ có `MissionHealthStatus`
- [ ] C++ có `RecoveryAction`
- [ ] C++ có `RecoverySceneInput`
- [ ] C++ có `RecoveryTriggerRuleConfig`
- [ ] C++ có `ReplanRuleConfig`
- [ ] C++ có `FallbackRuleConfig`
- [ ] C++ có `RecoveryDecisionResult`
- [ ] C++ có `RecoveryTriggerEngine`
- [ ] C++ có `RecoveryActionSelectionEngine`
- [ ] C++ có `ReplanDecisionEngine`
- [ ] C++ có `FallbackDecisionEngine`
- [ ] C++ có `RecoverySummaryEngine`
- [ ] C++ có `HumanoidPerceptionRecoveryTriggerAndReplanManager`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 72**, nếu bạn muốn vẫn bám Đợt 14 nhưng nâng từ “chọn recovery action” sang “đánh giá sau recovery có hiệu quả không”, bài hợp lý tiếp theo sẽ là:

# **Bài 73: Humanoid Recovery Outcome Evaluator**

Ý tưởng:
```text
recovery action
+ post-recovery progress / health
+ recovery history
→ recovery effectiveness evaluation
→ keep / escalate / switch strategy decision
→ recovery outcome report
```

Tức là mạch sẽ đi:

```text
Bài 70: Humanoid Perception Mission State Manager
→ Bài 71: Humanoid Perception Mission Progress Monitor
→ Bài 72: Humanoid Perception Recovery Trigger & Replan Manager
→ Bài 73: Humanoid Recovery Outcome Evaluator
```
