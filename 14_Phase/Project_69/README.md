# 🤖 Bài 69: Humanoid Multi-Mode Perception Mission Orchestrator — Bộ điều phối nhiệm vụ perception đa chế độ cho humanoid

> Mini Project số 69 trong **Đợt 14**  
> **Bài 69 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13 + 14** và đi tiếp trực tiếp từ **Bài 65 → 68**.
>
> Nếu:
>
> - **Bài 65** đã làm **task-aware interaction target selection**
> - **Bài 66** đã đào sâu riêng nhánh **FOLLOW_MODE**
> - **Bài 67** đã đào sâu riêng nhánh **PICK_MODE**
> - **Bài 68** đã đào sâu riêng nhánh **TABLE_CLEAN_MODE**
>
> thì bước tiếp theo hợp lý nhất là **gom 3 nhánh này lại** thành một tầng điều phối chung:
>
> ```text
> follow planner outputs
> + pick planner outputs
> + cleaning scheduler outputs
> + mission context / mode request / scene urgency
> → mode-aware mission arbitration
> → selected humanoid perception mission plan
> → mission orchestration report
> ```
>
> Đây là bước mà bạn không chỉ xử lý **từng mode riêng lẻ**, mà phải trả lời sâu hơn:
>
> - nếu robot cùng lúc có thể **follow**, **pick**, hoặc **clean**, mode nào nên được ưu tiên?
> - nếu một follow target đang ổn định nhưng bàn lại có clutter ưu tiên cao, robot nên làm gì trước?
> - nếu pick candidate mạnh nhưng scene cleaning đang khẩn cấp hơn thì planner nên ra quyết định thế nào?
> - perception layer phải xuất ra **mission-level decision** usable cho behavior / planning layer
>
> Tức là dữ liệu bắt đầu usable cho:
> - multi-mode mission arbitration
> - perception-level task switching
> - humanoid mission scheduling
> - perception-to-behavior orchestration handoff

---

# 📌 Mục lục

- [1. Vì sao Bài 69 xuất hiện sau Bài 65–68](#1-vì-sao-bài-69-xuất-hiện-sau-bài-6568)
- [2. Bài 69 nâng từ Bài 65–68 lên chỗ nào](#2-bài-69-nâng-từ-bài-6568-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Multi-mode perception mission pipeline tổng thể](#5-multi-mode-perception-mission-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật mission orchestrator của project](#10-luật-mission-orchestrator-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 69 xuất hiện sau Bài 65–68

## Bài 65 đã làm gì?
Bài 65 tạo ra tầng:
- **task-aware target selection**
- chọn target cho `FOLLOW_MODE / PICK_MODE / TABLE_CLEAN_MODE`

## Bài 66–68 đã tách ra 3 nhánh gì?
- **Bài 66**: nhánh **FOLLOW_MODE**
  - temporal tracking
  - target loss handling
  - follow decision
- **Bài 67**: nhánh **PICK_MODE**
  - pickability filtering
  - grasp suitability
  - best pick candidate
- **Bài 68**: nhánh **TABLE_CLEAN_MODE**
  - cleanability filtering
  - cleaning priority
  - cleaning task ordering

## Vậy còn thiếu gì?
Từ góc nhìn **humanoid mission**, robot không chạy từng nhánh một cách cô lập.  
Nó cần một tầng cao hơn để quyết định:

- **lúc này nên follow hay pick hay clean?**
- nếu nhiều mode đều có candidate tốt thì mode nào thắng?
- nếu mode hiện tại thất bại, có nên fallback sang mode khác không?

Đó là lý do **Bài 69** xuất hiện.

---

# 2. Bài 69 nâng từ Bài 65–68 lên chỗ nào

## Bài 66–68
- mỗi bài giải một mode riêng:
  - follow
  - pick
  - clean

## Bài 69
- gom output của 3 mode lại
- thêm **mission context**
- thêm **mode arbitration rules**
- thêm **fallback / override / urgency logic**
- chọn **final mission action mode**
- sinh **mission orchestration report**

### Nói ngắn gọn:
- **Bài 66–68** hỏi: “Trong từng mode riêng, robot nên làm gì?”
- **Bài 69** hỏi: “Khi có nhiều mode cạnh tranh nhau, robot nên chọn mode nào làm nhiệm vụ hiện tại?”

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Humanoid Multi-Mode Perception Mission Orchestrator**

System này nhận đầu vào là:
- một hoặc nhiều **mission scenes**
- output rút gọn từ 3 planner:
  - **follow planner results**
  - **pick planner results**
  - **cleaning scheduler results**
- một bộ **mission arbitration rules**
- một bộ **mode priority weights**
- một bộ **fallback rules**
- optional:
  - current requested mode
  - urgency tag
  - operator preference
  - scene risk level

## Mỗi mission scene tối thiểu chứa
- `scene_id`
- `follow_score`
- `pick_score`
- `clean_score`
- `follow_available`
- `pick_available`
- `clean_available`

Optional:
- `requested_mode`
- `urgency_level`
- `risk_level`
- `operator_priority_mode`

## Nhiệm vụ của system
### Với mỗi mission scene:
1. load mission scene inputs
2. đọc kết quả rút gọn của follow / pick / clean
3. evaluate availability của từng mode
4. apply mission arbitration rules
5. compute **mode-level mission score**
6. chọn **selected mission mode**
7. nếu mode chính không khả thi, apply fallback
8. ghi **mission orchestration report**

<p align="center">
  <img src="../../images/project_69.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build mission-scene config
→ build arbitration / priority / fallback configs
→ preview mode competition bằng NumPy
→ visualize selected mission modes

C++
→ load follow/pick/clean summarized results
→ evaluate multi-mode availability
→ arbitrate mission mode
→ select final humanoid perception mission
→ export orchestration reports
```

Mục tiêu cốt lõi:
- hiểu cách nối **3 perception branches** thành **1 mission-level decision layer**
- biết cách thiết kế **mode arbitration**
- biết cách tách:
  - mode availability
  - mode score
  - urgency override
  - fallback mode selection
- chuẩn bị nền trực tiếp cho:
  - behavior arbitration
  - humanoid mission scheduling
  - perception-to-high-level control handoff

---

# 5. Multi-mode perception mission pipeline tổng thể

```text
Load Mission Scene Config
Load Arbitration Rule Config
Load Mode Priority Weight Config
Load Fallback Rule Config
Load Visualization Config
Load Runtime Config

Create MissionAvailabilityInspector
Create ModeMissionScoreEngine
Create MissionArbitrationEngine
Create MissionFallbackEngine
Create MissionSummaryEngine
Create HumanoidMultiModePerceptionMissionOrchestrator

For each mission scene:
    1. load summarized mode results
    2. inspect availability of follow / pick / clean
    3. compute mission score for each mode
    4. arbitrate best mission mode
    5. apply fallback if needed
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
- arbitration / ranking / fallback logic
- project structure `.hpp / .cpp`

## Computer Vision / Robotics Vision
- object following
- pick-and-place vision
- table-cleaning perception
- mission-level arbitration
- humanoid perception orchestration

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **mission orchestration pipeline**:

```text
MissionSceneInputs
→ ModeAvailabilityInspection
→ ModeMissionScoring
→ MissionArbitration
→ FallbackSelection
→ MissionPlanReport
```

## 2. Python BST
Lưu:
- `scene_id`
- `selected_mode`
- `mission_result_id`

## 3. `std::vector<MissionSceneInput>`
Danh sách mission scenes.

## 4. `std::vector<MissionModeResult>`
Danh sách kết quả arbitration.

## 5. `std::unordered_map<std::string, std::vector<MissionModeResult>>`
Group kết quả theo `scene_id` hoặc `selected_mode`.

## 6. `std::stack<MissionDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Mode availability inspection
Một mode được xem là available nếu:
- score của mode > ngưỡng tối thiểu
- flag availability = true
- không bị cấm bởi rule hiện tại

Ví dụ:
- follow available nếu target follow ổn định và decision không phải `DROP_TARGET`
- pick available nếu có pick candidate hợp lệ
- clean available nếu có cleaning schedule hợp lệ

---

## Algorithm 2 — Mode mission scoring
Tạo **mission score** cho từng mode:

```text
mission_score(mode) =
    base_mode_score
  + priority_weight(mode)
  + urgency_bonus
  + operator_preference_bonus
  - risk_penalty
```

Trong đó:
- `base_mode_score` có thể đến từ `follow_score / pick_score / clean_score`
- `priority_weight(mode)` lấy từ config
- `urgency_bonus` phụ thuộc scene
- `risk_penalty` phụ thuộc mode + risk level

---

## Algorithm 3 — Mission arbitration
Chọn mode có mission score cao nhất **trong số các mode available**.

Tie-break gợi ý:
1. mode trùng với `requested_mode`
2. mode có priority weight cao hơn
3. mode có base score cao hơn
4. ưu tiên `PICK_MODE` hoặc `CLEAN_MODE` nếu scene có urgency cao

---

## Algorithm 4 — Fallback selection
Nếu mode được chọn không khả thi ở bước cuối:
- thử mode thứ 2 nếu available
- nếu không có mode nào usable thì trả về `IDLE_MODE`

Ví dụ:
```text
FOLLOW_MODE failed → fallback PICK_MODE
PICK_MODE failed → fallback CLEAN_MODE
CLEAN_MODE failed → fallback IDLE_MODE
```

---

## Algorithm 5 — Mission summary
Report cuối phải trả lời được:
- mode nào được chọn
- vì sao mode đó thắng
- fallback mode là gì
- mode nào bị loại và vì sao

---

## Algorithm 6 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- mode mission score chart
- selected mode distribution
- scene-wise mission orchestration summary

---

# 8. Cấu trúc folder

```text
mini_project_69_humanoid_multi_mode_perception_mission_orchestrator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ mission_scene_inputs/
│  │  ├─ mission_scene_01.txt
│  │  ├─ mission_scene_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ mission_mode_report.txt
│     ├─ mission_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ mode_mission_score_plot.png
│     └─ selected_mode_distribution_plot.png
│
├─ config/
│  ├─ mission_scene_config.txt
│  ├─ arbitration_rule_config.txt
│  ├─ mode_priority_weight_config.txt
│  ├─ fallback_rule_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ mission_graph_preview.py
│     ├─ mission_bst.py
│     ├─ numpy_mission_preview.py
│     ├─ matplotlib_mission_plotter.py
│     └─ synthetic_mission_scene_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ MissionSceneInput.hpp
   │  ├─ RequestedMode.hpp
   │  ├─ SelectedMissionMode.hpp
   │  ├─ ArbitrationRuleConfig.hpp
   │  ├─ ModePriorityWeightConfig.hpp
   │  ├─ FallbackRuleConfig.hpp
   │  ├─ MissionModeResult.hpp
   │  ├─ MissionDebugRecord.hpp
   │  ├─ BaseMissionAvailabilityInspector.hpp
   │  ├─ MissionAvailabilityInspector.hpp
   │  ├─ BaseModeMissionScoreEngine.hpp
   │  ├─ ModeMissionScoreEngine.hpp
   │  ├─ BaseMissionArbitrationEngine.hpp
   │  ├─ MissionArbitrationEngine.hpp
   │  ├─ BaseMissionFallbackEngine.hpp
   │  ├─ MissionFallbackEngine.hpp
   │  ├─ BaseMissionSummaryEngine.hpp
   │  ├─ MissionSummaryEngine.hpp
   │  ├─ HumanoidMultiModePerceptionMissionOrchestrator.hpp
   │  └─ MissionReportWriter.hpp
   │
   └─ src/
      ├─ MissionAvailabilityInspector.cpp
      ├─ ModeMissionScoreEngine.cpp
      ├─ MissionArbitrationEngine.cpp
      ├─ MissionFallbackEngine.cpp
      ├─ MissionSummaryEngine.cpp
      ├─ HumanoidMultiModePerceptionMissionOrchestrator.cpp
      └─ MissionReportWriter.cpp
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
mission_scene_config_path
arbitration_rule_config_path
mode_priority_weight_config_path
fallback_rule_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `MissionPlannerConfigBuilder`

Tạo class con:

```python
class MissionPlannerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_mission_scene(
    scene_id,
    follow_score,
    pick_score,
    clean_score,
    follow_available,
    pick_available,
    clean_available,
    requested_mode=None,
    urgency_level=None,
    risk_level=None,
    operator_priority_mode=None
)`

### `set_arbitration_rules(
    requested_mode_bonus,
    urgency_override_threshold,
    risk_penalty_scale
)`

### `set_mode_priority_weights(
    follow_weight,
    pick_weight,
    clean_weight
)`

### `set_fallback_rules(
    enable_fallback,
    allow_idle_mode,
    fallback_order
)`

### `set_visualization_options(
    enable_mode_score_plot,
    enable_mode_distribution_plot
)`

### `write_mission_scene_config()`
### `write_arbitration_rule_config()`
### `write_mode_priority_weight_config()`
### `write_fallback_rule_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `mission_graph_preview.py`

Tạo class:

```python
class MissionGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
MissionSceneInputs
→ ModeAvailabilityInspection
→ ModeMissionScoring
→ MissionArbitration
→ FallbackSelection
→ MissionPlanReport
```

---

# 9.4 Python — `mission_bst.py`

Tạo BST cho:
- `scene_id`
- `selected_mode`
- `mission_result_id`

---

# 9.5 Python — `numpy_mission_preview.py`

Tạo class:

```python
class NumPyMissionPreview:
```

## Hàm cần có

### `load_mission_scene(path)`
### `inspect_mode_availability(scene, arbitration_rule_config)`
### `compute_mode_scores(scene, weight_config, arbitration_rule_config)`
### `select_mode(scene_scores, fallback_rule_config)`

---

# 9.6 Python — `matplotlib_mission_plotter.py`

Tạo class:

```python
class MatplotlibMissionPlotter:
```

## Hàm cần có

### `plot_mode_scores(results, save_path)`
### `plot_selected_mode_distribution(results, save_path)`

---

# 9.7 C++ — `RequestedMode`

```cpp
enum class RequestedMode
{
    NONE,
    FOLLOW_MODE,
    PICK_MODE,
    CLEAN_MODE
};
```

---

# 9.8 C++ — `SelectedMissionMode`

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

# 9.9 C++ — `MissionSceneInput`

```cpp
struct MissionSceneInput
{
    std::string scene_id;

    double follow_score;
    double pick_score;
    double clean_score;

    bool follow_available;
    bool pick_available;
    bool clean_available;

    RequestedMode requested_mode;

    int urgency_level;
    int risk_level;

    RequestedMode operator_priority_mode;
};
```

---

# 9.10 C++ — `ArbitrationRuleConfig`

```cpp
struct ArbitrationRuleConfig
{
    double requested_mode_bonus;
    int urgency_override_threshold;
    double risk_penalty_scale;
};
```

---

# 9.11 C++ — `ModePriorityWeightConfig`

```cpp
struct ModePriorityWeightConfig
{
    double follow_weight;
    double pick_weight;
    double clean_weight;
};
```

---

# 9.12 C++ — `FallbackRuleConfig`

```cpp
struct FallbackRuleConfig
{
    bool enable_fallback;
    bool allow_idle_mode;
    std::vector<SelectedMissionMode> fallback_order;
};
```

---

# 9.13 C++ — `MissionModeResult`

```cpp
struct MissionModeResult
{
    std::string scene_id;

    bool follow_available;
    bool pick_available;
    bool clean_available;

    double follow_mission_score;
    double pick_mission_score;
    double clean_mission_score;

    SelectedMissionMode selected_mode;
    SelectedMissionMode fallback_mode;

    std::string summary;
};
```

---

# 9.14 C++ — `MissionDebugRecord`

```cpp
struct MissionDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.15 C++ — `BaseMissionAvailabilityInspector`

```cpp
class BaseMissionAvailabilityInspector
{
public:
    virtual MissionModeResult inspect(
        const MissionSceneInput& input
    ) const = 0;

    virtual ~BaseMissionAvailabilityInspector() = default;
};
```

---

# 9.16 C++ — `BaseModeMissionScoreEngine`

```cpp
class BaseModeMissionScoreEngine
{
public:
    virtual void compute_scores(
        MissionModeResult& result,
        const MissionSceneInput& input,
        const ArbitrationRuleConfig& arbitration_config,
        const ModePriorityWeightConfig& weight_config
    ) const = 0;

    virtual ~BaseModeMissionScoreEngine() = default;
};
```

---

# 9.17 C++ — `BaseMissionArbitrationEngine`

```cpp
class BaseMissionArbitrationEngine
{
public:
    virtual SelectedMissionMode arbitrate(
        const MissionModeResult& result,
        const MissionSceneInput& input
    ) const = 0;

    virtual ~BaseMissionArbitrationEngine() = default;
};
```

---

# 9.18 C++ — `BaseMissionFallbackEngine`

```cpp
class BaseMissionFallbackEngine
{
public:
    virtual SelectedMissionMode choose_fallback(
        const MissionModeResult& result,
        const FallbackRuleConfig& fallback_config
    ) const = 0;

    virtual ~BaseMissionFallbackEngine() = default;
};
```

---

# 9.19 C++ — `BaseMissionSummaryEngine`

```cpp
class BaseMissionSummaryEngine
{
public:
    virtual std::string summarize(
        const MissionModeResult& result
    ) const = 0;

    virtual ~BaseMissionSummaryEngine() = default;
};
```

---

# 9.20 C++ — Các class cụ thể bạn phải tạo

Tối thiểu phải có các class sau:

- `MissionAvailabilityInspector`
- `ModeMissionScoreEngine`
- `MissionArbitrationEngine`
- `MissionFallbackEngine`
- `MissionSummaryEngine`
- `HumanoidMultiModePerceptionMissionOrchestrator`
- `MissionReportWriter`

---

# 9.21 C++ — `HumanoidMultiModePerceptionMissionOrchestrator`

Tạo class trung tâm:

```cpp
class HumanoidMultiModePerceptionMissionOrchestrator
```

## Thuộc tính

```cpp
private:
    std::vector<MissionSceneInput> scenes;

    ArbitrationRuleConfig arbitration_config;
    ModePriorityWeightConfig weight_config;
    FallbackRuleConfig fallback_config;

    std::shared_ptr<BaseMissionAvailabilityInspector> availability_inspector;
    std::shared_ptr<BaseModeMissionScoreEngine> score_engine;
    std::shared_ptr<BaseMissionArbitrationEngine> arbitration_engine;
    std::shared_ptr<BaseMissionFallbackEngine> fallback_engine;
    std::shared_ptr<BaseMissionSummaryEngine> summary_engine;

    std::vector<MissionModeResult> results;
    std::stack<MissionDebugRecord> debug_history;
```

## Hàm cần có

### `load_mission_scene_config(const std::string& path)`
### `load_arbitration_rule_config(const std::string& path)`
### `load_mode_priority_weight_config(const std::string& path)`
### `load_fallback_rule_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each scene:
    result = availability_inspector.inspect(scene)
    score_engine.compute_scores(result, scene, arbitration_config, weight_config)

    selected = arbitration_engine.arbitrate(result, scene)
    result.selected_mode = selected

    if selected is not usable:
        result.fallback_mode = fallback_engine.choose_fallback(result, fallback_config)
    else:
        result.fallback_mode = selected

    result.summary = summary_engine.summarize(result)

    save result
    push debug history
```

### `const std::vector<MissionModeResult>& get_results() const`
### `std::vector<MissionDebugRecord> get_debug_history_reverse()`

---

# 9.22 C++ — `MissionReportWriter`

Tạo class:

```cpp
class MissionReportWriter
```

## Hàm cần có

### `write_mission_mode_report(...)`
Ví dụ:

```text
[Mission Mode]
Scene: mission_scene_01
Follow Score: 8.20
Pick Score: 11.40
Clean Score: 7.80
Selected Mode: PICK_MODE
Fallback Mode: PICK_MODE
```

### `write_mission_summary_report(...)`
Ví dụ:

```text
[Mission Summary]
Scene: mission_scene_01
PICK_MODE was selected because it is available, has the highest mission score,
and matches the current urgency-weighted mission context better than follow or clean.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 9.23 C++ — `main.cpp`

## Yêu cầu

```text
Load mission scene config
Load arbitration rule config
Load mode priority weight config
Load fallback rule config
Load visualization config
Load runtime config

Create:
    MissionAvailabilityInspector
    ModeMissionScoreEngine
    MissionArbitrationEngine
    MissionFallbackEngine
    MissionSummaryEngine

Create HumanoidMultiModePerceptionMissionOrchestrator
Run
Write reports
```

---

# 10. Luật mission orchestrator của project

## Luật 1 — Bài này phải thật sự là tầng “đa mode”
Không được chỉ rank follow/pick/clean đơn giản rồi chọn max một cách sơ sài.

## Luật 2 — Phải tách `availability`, `mission score`, `arbitration`, `fallback`
Đây là 4 lớp logic khác nhau.

## Luật 3 — Phải có ít nhất một yếu tố context
Ví dụ:
- requested mode
- urgency level
- risk level
- operator priority mode

## Luật 4 — Kết quả cuối phải usable cho behavior planner
Tức là sau bài này bạn phải trả lời được:
- robot nên follow / pick / clean / idle?
- nếu mode hiện tại fail thì fallback là gì?
- vì sao mode được chọn lại thắng các mode khác?

## Luật 5 — Report phải mang tinh thần “mission orchestration”
Không chỉ là bảng điểm của 3 mode.

---

# 11. Output mong muốn

## Config
```text
config/mission_scene_config.txt
config/arbitration_rule_config.txt
config/mode_priority_weight_config.txt
config/fallback_rule_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/mission_mode_report.txt
assets/outputs/mission_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/mode_mission_score_plot.png
assets/outputs/selected_mode_distribution_plot.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build mission-scene configs
- preview competition giữa follow / pick / clean
- visualize selected mission modes

## C++
- đóng vai **mission-level perception orchestrator**
- arbitrate follow/pick/clean
- chọn mode nhiệm vụ cuối cùng
- xuất mission-level decision report

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Khi humanoid có nhiều nhiệm vụ perception cạnh tranh nhau, robot nên follow, pick, clean hay idle ở thời điểm hiện tại?**

Pipeline lúc này sẽ là:

```text
follow planner output
+ pick planner output
+ clean scheduler output
→ multi-mode mission arbitration
→ selected humanoid perception mission
→ behavior-ready mission report
```

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho mission scenes / arbitration / priority / fallback / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho scene ids / selected modes / mission result ids
- [ ] Python có NumPy mission preview
- [ ] C++ có `MissionSceneInput`
- [ ] C++ có `RequestedMode`
- [ ] C++ có `SelectedMissionMode`
- [ ] C++ có `ArbitrationRuleConfig`
- [ ] C++ có `ModePriorityWeightConfig`
- [ ] C++ có `FallbackRuleConfig`
- [ ] C++ có `MissionModeResult`
- [ ] C++ có `MissionAvailabilityInspector`
- [ ] C++ có `ModeMissionScoreEngine`
- [ ] C++ có `MissionArbitrationEngine`
- [ ] C++ có `MissionFallbackEngine`
- [ ] C++ có `MissionSummaryEngine`
- [ ] C++ có `HumanoidMultiModePerceptionMissionOrchestrator`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 69**, nếu bạn muốn vẫn bám chặt Đợt 14 nhưng nâng từ “chọn mode” sang “quản lý vòng đời nhiệm vụ”, bài hợp lý tiếp theo sẽ là:

# **Bài 70: Humanoid Perception Mission State Manager**

Ý tưởng:
```text
selected mission mode
+ previous mission state
+ success / failure / interruption events
→ mission state transition
→ mission lifecycle report
```

Tức là mạch sẽ đi:

```text
Bài 65: Humanoid Interaction Target Selector
→ Bài 66: Humanoid Object Following Target Tracker Planner
→ Bài 67: Pick-and-Place Vision Candidate Planner
→ Bài 68: Table-Cleaning Perception Task Scheduler
→ Bài 69: Humanoid Multi-Mode Perception Mission Orchestrator
→ Bài 70: Humanoid Perception Mission State Manager
```
