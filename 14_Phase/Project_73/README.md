# 🤖 Bài 73: Humanoid Recovery Outcome Evaluator — Bộ đánh giá hiệu quả recovery cho nhiệm vụ perception của humanoid

> Mini Project số 73 trong **Đợt 14**  
> **Bài 73 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 72**.
>
> Nếu:
>
> - **Bài 69** đã chọn **mission mode**
> - **Bài 70** đã quản lý **mission state / retry / recovery**
> - **Bài 71** đã giám sát **progress / stall / anomaly / health**
> - **Bài 72** đã quyết định **retry / replan / fallback / abort**
>
> thì bước tiếp theo hợp lý nhất là:
>
> ```text
> recovery action
> + post-recovery progress / health
> + recovery history
> → recovery effectiveness evaluation
> → keep / escalate / switch strategy decision
> → recovery outcome report
> ```
>
> Đây là bước mà bạn không chỉ quyết định **phải recovery như thế nào**, mà còn phải đánh giá:
>
> - recovery vừa rồi có **thực sự cải thiện mission** không?
> - sau `RETRY_ACTION` progress có tốt hơn không?
> - sau `REPLAN_ACTION` stall / anomaly có giảm không?
> - nếu recovery không hiệu quả thì có nên **escalate**, **switch strategy**, hay **abort** không?
>
> Tức là dữ liệu bắt đầu usable cho:
> - recovery effectiveness evaluation
> - adaptive recovery strategy switching
> - escalation policy
> - long-horizon mission supervision

---

# 📌 Mục lục

- [1. Vì sao Bài 73 xuất hiện sau Bài 72](#1-vì-sao-bài-73-xuất-hiện-sau-bài-72)
- [2. Bài 73 nâng từ Bài 72 lên chỗ nào](#2-bài-73-nâng-từ-bài-72-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Recovery outcome evaluation pipeline tổng thể](#5-recovery-outcome-evaluation-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật recovery outcome evaluator của project](#10-luật-recovery-outcome-evaluator-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 73 xuất hiện sau Bài 72

## Bài 72 đang làm gì?
Bài 72 đã giúp bạn:
- đọc `health_status`, `stall_detected`, `anomaly_detected`
- trigger recovery khi cần
- chọn action:
  - `RETRY_ACTION`
  - `REPLAN_ACTION`
  - `FALLBACK_ACTION`
  - `ABORT_ACTION`
  - `CONTINUE_ACTION`

## Nhưng còn thiếu gì?
Bài 72 mới chỉ quyết định **phải recovery như thế nào**.  
Nó chưa trả lời:
- recovery vừa rồi **có hiệu quả hay không**
- recovery nào **thực sự cứu được mission**
- recovery nào **không còn đáng thử nữa**
- có cần **leo thang chiến lược** sang action mạnh hơn không

Ví dụ:
- follow mission đã retry 2 lần nhưng target distance vẫn không cải thiện
- pick mission đã replan candidate nhưng anomaly vẫn còn
- clean mission fallback sang idle nhưng health vẫn critical ở lần sau

## Bài 73 lấp đúng chỗ đó
Bài 73 sẽ làm:

```text
recovery action + post-recovery mission signals
→ recovery effectiveness scoring
→ keep / escalate / switch / abort decision
→ recovery outcome summary
```

Đây là bước chuyển từ **recovery action selection** sang **recovery effectiveness evaluation**.

---

# 2. Bài 73 nâng từ Bài 72 lên chỗ nào

## Bài 72
- trigger recovery
- chọn retry / replan / fallback / abort

## Bài 73
- nhìn **hậu quả sau recovery**
- đo **mức cải thiện của mission**
- đánh giá **recovery action có hiệu quả không**
- quyết định:
  - giữ nguyên chiến lược
  - escalate recovery
  - switch strategy
  - abort

### Nói ngắn gọn:
- **Bài 72** hỏi: “Robot nên recovery bằng cách nào?”
- **Bài 73** hỏi: “Recovery vừa làm có hiệu quả không, và bước tiếp theo nên giữ / đổi / leo thang chiến lược gì?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Humanoid Recovery Outcome Evaluator**

System này nhận đầu vào là:
- một hoặc nhiều **recovery outcome scenes**
- mỗi scene chứa:
  - `recovery_action`
  - `pre_recovery_health`
  - `post_recovery_health`
  - `pre_recovery_progress_score`
  - `post_recovery_progress_score`
- một bộ **effectiveness scoring rules**
- một bộ **strategy escalation rules**
- optional:
  - recovery history length
  - retry count
  - replan count
  - fallback count
  - anomaly count after recovery
  - stall count after recovery

## Mỗi recovery outcome sample tối thiểu chứa
- `scene_id`
- `recovery_action`
- `pre_recovery_health`
- `post_recovery_health`
- `pre_progress_score`
- `post_progress_score`

Optional:
- `recovery_count`
- `retry_count`
- `replan_count`
- `fallback_count`
- `post_anomaly_count`
- `post_stall_count`

## Nhiệm vụ của system
### Với mỗi scene:
1. load recovery outcome scene
2. compute **health improvement**
3. compute **progress improvement**
4. compute **recovery effectiveness score**
5. decide next strategy:
   - keep current strategy
   - escalate recovery
   - switch recovery strategy
   - abort mission
6. ghi **recovery outcome report**

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build recovery-outcome config
→ build effectiveness / escalation configs
→ preview recovery outcome evaluation bằng NumPy
→ visualize recovery effectiveness distribution

C++
→ load recovery outcome scenes
→ compute improvement metrics
→ score recovery effectiveness
→ decide keep / escalate / switch / abort
→ export recovery outcome reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **recovery decision** với **hậu kiểm recovery**
- biết cách tách:
  - improvement measurement
  - effectiveness scoring
  - escalation decision
  - strategy switch policy
- chuẩn bị nền trực tiếp cho:
  - adaptive recovery loops
  - autonomous recovery policy tuning
  - long-term mission robustness supervision

---

# 5. Recovery outcome evaluation pipeline tổng thể

```text
Load Recovery Outcome Scene Config
Load Effectiveness Rule Config
Load Escalation Rule Config
Load Visualization Config
Load Runtime Config

Create RecoveryImprovementEngine
Create RecoveryEffectivenessScoringEngine
Create RecoveryStrategyDecisionEngine
Create RecoveryOutcomeSummaryEngine
Create HumanoidRecoveryOutcomeEvaluator

For each recovery outcome scene:
    1. load pre/post recovery metrics
    2. compute improvement metrics
    3. score recovery effectiveness
    4. decide keep / escalate / switch / abort
    5. write report

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
- scoring engine design
- escalation / strategy-switch logic

## Computer Vision / Robotics Vision
- recovery evaluation
- mission robustness analysis
- adaptive recovery policy
- failure handling loop
- post-recovery mission supervision

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **recovery outcome pipeline**:

```text
RecoveryOutcomeInput
→ ImprovementMeasurement
→ EffectivenessScoring
→ StrategyDecision
→ RecoveryOutcomeReport
```

## 2. Python BST
Lưu:
- `scene_id`
- `strategy_decision`
- `recovery_outcome_result_id`

## 3. `std::vector<RecoveryOutcomeScene>`
Danh sách recovery outcome scenes.

## 4. `std::vector<RecoveryOutcomeResult>`
Danh sách kết quả evaluation.

## 5. `std::unordered_map<std::string, std::vector<RecoveryOutcomeResult>>`
Group theo `scene_id` hoặc `strategy_decision`.

## 6. `std::stack<RecoveryOutcomeDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Improvement measurement
Tính:
- `health_improvement`
- `progress_improvement`
- `anomaly_reduction`
- `stall_reduction`

Ví dụ:
```text
health_improvement = post_health_score - pre_health_score
progress_improvement = post_progress_score - pre_progress_score
```

Trong đó có thể map:
- `HEALTHY = 2`
- `WARNING = 1`
- `CRITICAL = 0`

---

## Algorithm 2 — Recovery effectiveness scoring
Tạo **effectiveness score**:

```text
effectiveness_score =
    progress_improvement * progress_weight
  + health_improvement * health_weight
  - post_anomaly_penalty
  - post_stall_penalty
  - repeated_recovery_penalty
```

Sau đó classify:
- `SUCCESSFUL_RECOVERY`
- `PARTIAL_RECOVERY`
- `FAILED_RECOVERY`

---

## Algorithm 3 — Strategy decision
Dựa trên effectiveness:
- nếu recovery thành công → `KEEP_CURRENT_STRATEGY`
- nếu recovery chỉ cải thiện ít → `SWITCH_STRATEGY`
- nếu recovery thất bại nhiều lần → `ESCALATE_RECOVERY`
- nếu quá nhiều lần thất bại → `ABORT_MISSION`

---

## Algorithm 4 — Escalation policy
Ví dụ:
- `RETRY_ACTION` thất bại nhiều lần → escalate lên `REPLAN_ACTION`
- `REPLAN_ACTION` vẫn thất bại → escalate lên `FALLBACK_ACTION`
- `FALLBACK_ACTION` vẫn không cứu được → `ABORT_MISSION`

---

## Algorithm 5 — Recovery outcome summary
Report cuối phải trả lời được:
- recovery có hiệu quả không
- progress / health cải thiện bao nhiêu
- strategy tiếp theo là gì
- vì sao phải keep / switch / escalate / abort

---

## Algorithm 6 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- effectiveness distribution
- strategy decision distribution
- pre/post recovery comparison plot

---

# 8. Cấu trúc folder

```text
mini_project_73_humanoid_recovery_outcome_evaluator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ recovery_outcome_inputs/
│  │  ├─ recovery_outcome_scene_01.txt
│  │  ├─ recovery_outcome_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ recovery_outcome_report.txt
│     ├─ recovery_strategy_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ recovery_effectiveness_distribution_plot.png
│     └─ strategy_decision_plot.png
│
├─ config/
│  ├─ recovery_outcome_scene_config.txt
│  ├─ effectiveness_rule_config.txt
│  ├─ escalation_rule_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ recovery_outcome_graph_preview.py
│     ├─ recovery_outcome_bst.py
│     ├─ numpy_recovery_outcome_preview.py
│     ├─ matplotlib_recovery_outcome_plotter.py
│     └─ synthetic_recovery_outcome_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ RecoveryAction.hpp
   │  ├─ MissionHealthStatus.hpp
   │  ├─ RecoveryOutcomeLabel.hpp
   │  ├─ StrategyDecision.hpp
   │  ├─ RecoveryOutcomeScene.hpp
   │  ├─ EffectivenessRuleConfig.hpp
   │  ├─ EscalationRuleConfig.hpp
   │  ├─ RecoveryOutcomeResult.hpp
   │  ├─ RecoveryOutcomeDebugRecord.hpp
   │  ├─ BaseRecoveryImprovementEngine.hpp
   │  ├─ RecoveryImprovementEngine.hpp
   │  ├─ BaseRecoveryEffectivenessScoringEngine.hpp
   │  ├─ RecoveryEffectivenessScoringEngine.hpp
   │  ├─ BaseRecoveryStrategyDecisionEngine.hpp
   │  ├─ RecoveryStrategyDecisionEngine.hpp
   │  ├─ BaseRecoveryOutcomeSummaryEngine.hpp
   │  ├─ RecoveryOutcomeSummaryEngine.hpp
   │  ├─ HumanoidRecoveryOutcomeEvaluator.hpp
   │  └─ RecoveryOutcomeReportWriter.hpp
   │
   └─ src/
      ├─ RecoveryImprovementEngine.cpp
      ├─ RecoveryEffectivenessScoringEngine.cpp
      ├─ RecoveryStrategyDecisionEngine.cpp
      ├─ RecoveryOutcomeSummaryEngine.cpp
      ├─ HumanoidRecoveryOutcomeEvaluator.cpp
      └─ RecoveryOutcomeReportWriter.cpp
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
recovery_outcome_scene_config_path
effectiveness_rule_config_path
escalation_rule_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `RecoveryOutcomeConfigBuilder`

Tạo class con:

```python
class RecoveryOutcomeConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_recovery_outcome_scene(
    scene_id,
    recovery_action,
    pre_recovery_health,
    post_recovery_health,
    pre_progress_score,
    post_progress_score,
    recovery_count=0,
    retry_count=0,
    replan_count=0,
    fallback_count=0,
    post_anomaly_count=0,
    post_stall_count=0
)`

### `set_effectiveness_rules(
    progress_weight,
    health_weight,
    anomaly_penalty,
    stall_penalty,
    repeated_recovery_penalty
)`

### `set_escalation_rules(
    partial_recovery_threshold,
    failed_recovery_threshold,
    max_failed_recovery_before_abort
)`

### `set_visualization_options(
    enable_effectiveness_distribution_plot,
    enable_strategy_decision_plot
)`

### `write_recovery_outcome_scene_config()`
### `write_effectiveness_rule_config()`
### `write_escalation_rule_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `recovery_outcome_graph_preview.py`

Tạo class:

```python
class RecoveryOutcomeGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
RecoveryOutcomeInput
→ ImprovementMeasurement
→ EffectivenessScoring
→ StrategyDecision
→ RecoveryOutcomeReport
```

---

# 9.4 Python — `recovery_outcome_bst.py`

Tạo BST cho:
- `scene_id`
- `strategy_decision`
- `recovery_outcome_result_id`

---

# 9.5 Python — `numpy_recovery_outcome_preview.py`

Tạo class:

```python
class NumPyRecoveryOutcomePreview:
```

## Hàm cần có

### `load_recovery_outcome_scene(path)`
### `compute_improvement(scene)`
### `score_effectiveness(scene, effectiveness_rule_config)`
### `decide_strategy(effectiveness_score, escalation_rule_config)`
### `build_outcome_summary(results)`

---

# 9.6 Python — `matplotlib_recovery_outcome_plotter.py`

Tạo class:

```python
class MatplotlibRecoveryOutcomePlotter:
```

## Hàm cần có

### `plot_effectiveness_distribution(results, save_path)`
### `plot_strategy_decision_distribution(results, save_path)`

---

# 9.7 C++ — `RecoveryAction`

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

# 9.8 C++ — `MissionHealthStatus`

```cpp
enum class MissionHealthStatus
{
    HEALTHY,
    WARNING,
    CRITICAL
};
```

---

# 9.9 C++ — `RecoveryOutcomeLabel`

```cpp
enum class RecoveryOutcomeLabel
{
    SUCCESSFUL_RECOVERY,
    PARTIAL_RECOVERY,
    FAILED_RECOVERY
};
```

---

# 9.10 C++ — `StrategyDecision`

```cpp
enum class StrategyDecision
{
    KEEP_CURRENT_STRATEGY,
    SWITCH_STRATEGY,
    ESCALATE_RECOVERY,
    ABORT_MISSION
};
```

---

# 9.11 C++ — `RecoveryOutcomeScene`

```cpp
struct RecoveryOutcomeScene
{
    std::string scene_id;

    RecoveryAction recovery_action;

    MissionHealthStatus pre_recovery_health;
    MissionHealthStatus post_recovery_health;

    double pre_progress_score;
    double post_progress_score;

    int recovery_count;
    int retry_count;
    int replan_count;
    int fallback_count;

    int post_anomaly_count;
    int post_stall_count;
};
```

---

# 9.12 C++ — `EffectivenessRuleConfig`

```cpp
struct EffectivenessRuleConfig
{
    double progress_weight;
    double health_weight;
    double anomaly_penalty;
    double stall_penalty;
    double repeated_recovery_penalty;
};
```

---

# 9.13 C++ — `EscalationRuleConfig`

```cpp
struct EscalationRuleConfig
{
    double partial_recovery_threshold;
    double failed_recovery_threshold;
    int max_failed_recovery_before_abort;
};
```

---

# 9.14 C++ — `RecoveryOutcomeResult`

```cpp
struct RecoveryOutcomeResult
{
    std::string scene_id;

    double effectiveness_score;
    RecoveryOutcomeLabel outcome_label;
    StrategyDecision strategy_decision;

    std::string summary;
};
```

---

# 9.15 C++ — `RecoveryOutcomeDebugRecord`

```cpp
struct RecoveryOutcomeDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.16 C++ — `BaseRecoveryImprovementEngine`

```cpp
class BaseRecoveryImprovementEngine
{
public:
    virtual std::pair<double, double> compute_improvement(
        const RecoveryOutcomeScene& scene
    ) const = 0;

    virtual ~BaseRecoveryImprovementEngine() = default;
};
```

---

# 9.17 C++ — `BaseRecoveryEffectivenessScoringEngine`

```cpp
class BaseRecoveryEffectivenessScoringEngine
{
public:
    virtual std::pair<double, RecoveryOutcomeLabel> score(
        const RecoveryOutcomeScene& scene,
        const EffectivenessRuleConfig& config
    ) const = 0;

    virtual ~BaseRecoveryEffectivenessScoringEngine() = default;
};
```

---

# 9.18 C++ — `BaseRecoveryStrategyDecisionEngine`

```cpp
class BaseRecoveryStrategyDecisionEngine
{
public:
    virtual StrategyDecision decide(
        double effectiveness_score,
        RecoveryOutcomeLabel label,
        const RecoveryOutcomeScene& scene,
        const EscalationRuleConfig& config
    ) const = 0;

    virtual ~BaseRecoveryStrategyDecisionEngine() = default;
};
```

---

# 9.19 C++ — `BaseRecoveryOutcomeSummaryEngine`

```cpp
class BaseRecoveryOutcomeSummaryEngine
{
public:
    virtual std::string summarize(
        const RecoveryOutcomeResult& result
    ) const = 0;

    virtual ~BaseRecoveryOutcomeSummaryEngine() = default;
};
```

---

# 9.20 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `RecoveryImprovementEngine`
- `RecoveryEffectivenessScoringEngine`
- `RecoveryStrategyDecisionEngine`
- `RecoveryOutcomeSummaryEngine`
- `HumanoidRecoveryOutcomeEvaluator`
- `RecoveryOutcomeReportWriter`

---

# 9.21 C++ — `HumanoidRecoveryOutcomeEvaluator`

Tạo class trung tâm:

```cpp
class HumanoidRecoveryOutcomeEvaluator
```

## Thuộc tính

```cpp
private:
    std::vector<RecoveryOutcomeScene> scenes;

    EffectivenessRuleConfig effectiveness_config;
    EscalationRuleConfig escalation_config;

    std::shared_ptr<BaseRecoveryImprovementEngine> improvement_engine;
    std::shared_ptr<BaseRecoveryEffectivenessScoringEngine> effectiveness_engine;
    std::shared_ptr<BaseRecoveryStrategyDecisionEngine> strategy_engine;
    std::shared_ptr<BaseRecoveryOutcomeSummaryEngine> summary_engine;

    std::vector<RecoveryOutcomeResult> results;
    std::stack<RecoveryOutcomeDebugRecord> debug_history;
```

## Hàm cần có

### `load_recovery_outcome_scene_config(const std::string& path)`
### `load_effectiveness_rule_config(const std::string& path)`
### `load_escalation_rule_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each scene:
    (effectiveness_score, outcome_label) =
        effectiveness_engine.score(scene, effectiveness_config)

    strategy_decision =
        strategy_engine.decide(
            effectiveness_score,
            outcome_label,
            scene,
            escalation_config
        )

    build RecoveryOutcomeResult
    summary = summary_engine.summarize(result)
    save result
    push debug history
```

### `const std::vector<RecoveryOutcomeResult>& get_results() const`
### `std::vector<RecoveryOutcomeDebugRecord> get_debug_history_reverse()`

---

# 9.22 C++ — `RecoveryOutcomeReportWriter`

Tạo class:

```cpp
class RecoveryOutcomeReportWriter
```

## Hàm cần có

### `write_recovery_outcome_report(...)`
Ví dụ:

```text
[Recovery Outcome]
Scene: recovery_outcome_scene_01
Effectiveness Score: 7.20
Outcome Label: SUCCESSFUL_RECOVERY
Strategy Decision: KEEP_CURRENT_STRATEGY
```

### `write_recovery_strategy_report(...)`
Ví dụ:

```text
[Recovery Strategy]
Scene: recovery_outcome_scene_01
The previous recovery improved both mission health and progress,
so the evaluator kept the current strategy instead of escalating or switching.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.23 C++ — `main.cpp`

## Yêu cầu

```text
Load recovery outcome scene config
Load effectiveness rule config
Load escalation rule config
Load visualization config
Load runtime config

Create:
    RecoveryImprovementEngine
    RecoveryEffectivenessScoringEngine
    RecoveryStrategyDecisionEngine
    RecoveryOutcomeSummaryEngine

Create HumanoidRecoveryOutcomeEvaluator
Run
Write reports
```

---

# 10. Luật recovery outcome evaluator của project

## Luật 1 — Bài này phải thật sự là post-recovery evaluation layer
Không được chỉ so `pre` và `post` một cách sơ sài.

## Luật 2 — Phải tách `improvement`, `effectiveness`, `strategy decision`
Đây là 3 lớp logic khác nhau.

## Luật 3 — Phải có ít nhất một scene đi tới `SWITCH_STRATEGY` hoặc `ESCALATE_RECOVERY`
Nếu tất cả đều KEEP thì bài sẽ quá yếu.

## Luật 4 — Kết quả cuối phải usable cho adaptive recovery supervisor
Tức là sau bài này bạn phải trả lời được:
- recovery có hiệu quả không?
- strategy tiếp theo là keep / switch / escalate / abort?
- vì sao evaluator lại đưa ra quyết định đó?

## Luật 5 — Report phải mang tinh thần “recovery effectiveness evaluation”
Không chỉ là bảng điểm đơn giản.

---

# 11. Output mong muốn

## Config
```text
config/recovery_outcome_scene_config.txt
config/effectiveness_rule_config.txt
config/escalation_rule_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/recovery_outcome_report.txt
assets/outputs/recovery_strategy_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/recovery_effectiveness_distribution_plot.png
assets/outputs/strategy_decision_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build recovery-outcome configs
- preview post-recovery evaluation
- visualize effectiveness / strategy decisions

## C++
- đóng vai **post-recovery evaluator**
- đánh giá recovery vừa rồi có cứu được mission hay không
- quyết định keep / switch / escalate / abort
- xuất recovery outcome reports

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Recovery vừa rồi có hiệu quả không, và robot có nên giữ chiến lược hiện tại, đổi chiến lược, leo thang recovery hay dừng mission?**

Pipeline lúc này sẽ là:

```text
recovery action
→ post-recovery improvement measurement
→ recovery effectiveness scoring
→ keep / switch / escalate / abort decision
→ recovery outcome report
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho recovery-outcome scenes / effectiveness / escalation / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / strategy decisions / result ids
- [ ] Python có NumPy recovery-outcome preview
- [ ] C++ có `RecoveryAction`
- [ ] C++ có `MissionHealthStatus`
- [ ] C++ có `RecoveryOutcomeLabel`
- [ ] C++ có `StrategyDecision`
- [ ] C++ có `RecoveryOutcomeScene`
- [ ] C++ có `EffectivenessRuleConfig`
- [ ] C++ có `EscalationRuleConfig`
- [ ] C++ có `RecoveryOutcomeResult`
- [ ] C++ có `RecoveryImprovementEngine`
- [ ] C++ có `RecoveryEffectivenessScoringEngine`
- [ ] C++ có `RecoveryStrategyDecisionEngine`
- [ ] C++ có `RecoveryOutcomeSummaryEngine`
- [ ] C++ có `HumanoidRecoveryOutcomeEvaluator`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 73**, nếu bạn muốn vẫn bám Đợt 14 nhưng nâng từ “đánh giá recovery outcome” sang “tích lũy lịch sử recovery để tối ưu policy về sau”, bài hợp lý tiếp theo sẽ là:

# **Bài 74: Humanoid Recovery Memory & Policy Adapter**

Ý tưởng:
```text
recovery history
+ recovery outcomes
+ scene context
→ policy adaptation
→ preferred recovery strategy update
→ recovery memory report
```

Tức là mạch sẽ đi:

```text
Bài 71: Humanoid Perception Mission Progress Monitor
→ Bài 72: Humanoid Perception Recovery Trigger & Replan Manager
→ Bài 73: Humanoid Recovery Outcome Evaluator
→ Bài 74: Humanoid Recovery Memory & Policy Adapter
```
