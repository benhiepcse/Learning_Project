# 🤖 Bài 63: Robot-Frame Target Localization Inspector — Bộ kiểm tra định vị mục tiêu trong robot frame cho Humanoid Robot AI Perception

> Mini Project số 63 trong **Đợt 13**  
> **Bài 63 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11 + 12 + 13** và đi tiếp trực tiếp từ **Bài 62**.
>
> Nếu:
>
> - **Bài 62** giúp bạn lấy:
>   - `pixel + depth`
>   - `camera-frame point`
>   - `robot-frame point`
> - **Đợt 13** nhấn mạnh mạnh vào:
>   - `Robot Perception Pipeline`
>   - `Camera to Robot Coordinate`
>
> thì **Bài 63** là bước tiếp theo rất tự nhiên:
>
> ```text
> robot-frame point
> → target localization inspection
> → left / center / right zone assignment
> → forward reachability analysis
> → target validity report
> ```
>
> Đây là bước mà bạn không chỉ biết **một điểm nằm ở đâu trong robot frame**, mà còn tiến thêm một bước:
>
> - điểm đó có nằm ở **vùng trái / giữa / phải** không?
> - nó có ở **phía trước robot** hay nằm ngoài vùng thao tác?
> - nó có đủ gần / đủ hợp lý để được coi là **target hợp lệ** cho perception / navigation / manipulation không?
>
> Tức là dữ liệu bắt đầu usable cho:
> - robot-frame target localization
> - zone-based reasoning
> - reachable target checking
> - robot-centric perception validation

---

# 📌 Mục lục

- [1. Đợt 13 đang đi tiếp từ Bài 62 ra sao](#1-đợt-13-đang-đi-tiếp-từ-bài-62-ra-sao)
- [2. Vì sao Bài 63 xuất hiện sau Bài 62](#2-vì-sao-bài-63-xuất-hiện-sau-bài-62)
- [3. Bài 63 nâng từ Bài 62 lên chỗ nào](#3-bài-63-nâng-từ-bài-62-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật robot-frame localization inspection của project](#11-luật-robot-frame-localization-inspection-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 13 đang đi tiếp từ Bài 62 ra sao

Ở **Bài 62**, bạn đã có:

```text
pixel + depth
→ camera-frame point
→ camera-to-robot transform
→ robot-frame point
```

Tức là bạn đã đưa được một điểm từ camera coordinate sang robot coordinate.

Nhưng robot vẫn chưa thật sự “hiểu” điểm đó theo góc nhìn hành động.  
Nó còn cần trả lời các câu hỏi như:

- điểm này nằm **trái / giữa / phải** so với robot?
- điểm này nằm **trước mặt robot** bao nhiêu mét?
- điểm này có nằm trong **vùng quan tâm / vùng tiếp cận** không?
- có nên coi nó là **target hợp lệ** cho các bước perception / navigation tiếp theo không?

Đó là lý do **Bài 63** xuất hiện.

---

# 2. Vì sao Bài 63 xuất hiện sau Bài 62

## Bài 62 đang làm gì?
Bài 62 giúp bạn:
- back-project từ pixel + depth
- transform camera point sang robot point
- kiểm tra consistency của phép chiếu

## Nhưng còn thiếu gì?
Bạn vẫn mới dừng ở mức **geometry**:
- “điểm ở đâu?”

Robot perception thực tế cần thêm mức **semantic robot-centric interpretation**:
- “điểm đó nằm ở vùng nào quanh robot?”
- “có nằm trong vùng có thể tiến tới / quan sát / tương tác không?”
- “có phải target hợp lệ hay không?”

## Bài 63 lấp đúng chỗ đó
Bài 63 sẽ làm:

```text
robot-frame point
→ zone classification
→ forward / lateral / vertical inspection
→ target validity checking
→ localization inspection report
```

Đây là bước chuyển từ **camera-to-robot projection** sang **robot-frame target localization reasoning**.

---

# 3. Bài 63 nâng từ Bài 62 lên chỗ nào

## Bài 62
- sinh camera point
- sinh robot point
- kiểm tra projection consistency

## Bài 63
- đọc robot-frame point
- gán **zone label**:
  - LEFT_ZONE
  - CENTER_ZONE
  - RIGHT_ZONE
  - REAR_ZONE
  - OUT_OF_RANGE
- kiểm tra:
  - khoảng cách phía trước
  - độ lệch ngang
  - độ cao
  - tính reachable / valid
- sinh **target localization inspection report**

### Nói ngắn gọn:
- **Bài 62** hỏi: “Điểm ảnh này nằm ở đâu trong robot frame?”
- **Bài 63** hỏi: “Điểm đó có phải target hợp lệ trong robot frame không, và nó thuộc vùng nào?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Robot-Frame Target Localization Inspector**

System này nhận đầu vào là:
- một hoặc nhiều **target localization samples**
- một bộ **robot zone config**
- một bộ **reachability / validity thresholds**
- optional **camera projection result files**
- một bộ **reporting options**

## Mỗi sample tối thiểu chứa
- `pair_id`
- `robot_x`
- `robot_y`
- `robot_z`
- optional:
  - `label`
  - `source_projection_path`
  - `expected_zone`

## Nhiệm vụ của system
### Với mỗi sample:
1. load robot-frame target point
2. phân tích:
   - forward distance
   - lateral offset
   - vertical offset
3. gán zone cho target
4. kiểm tra:
   - target có ở phía trước robot không
   - target có nằm trong lateral range không
   - target có nằm trong reachable distance không
5. kết luận target:
   - valid / invalid
   - reachable / not reachable
6. ghi **target localization inspection report**

<p align="center">
  <img src="images/project_63.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build localization sample config
→ build zone config
→ preview target zoning bằng NumPy
→ visualize zone / reachability statistics

C++
→ load robot-frame points
→ assign localization zones
→ evaluate target validity
→ build localization summary
→ export inspection reports
```

Mục tiêu cốt lõi:
- hiểu cách robot **diễn giải một điểm trong robot frame**
- biết cách gán **trái / giữa / phải / sau / ngoài tầm**
- biết cách kiểm tra **reachable target**
- chuẩn bị nền trực tiếp cho:
  - robot-centric target selection
  - local interaction target filtering
  - navigation / manipulation target pre-check

---

# 6. Pipeline tổng thể

```text
Load Localization Sample Config
Load Robot Zone Config
Load Reachability Threshold Config
Load Visualization Config
Load Runtime Config

Create TargetZoneClassifier
Create ReachabilityInspector
Create TargetValidityEngine
Create LocalizationSummaryEngine
Create RobotFrameTargetLocalizationInspector

For each target sample:
    1. load robot-frame point
    2. classify zone
    3. inspect reachability
    4. evaluate validity
    5. summarize localization result
    6. write report

After all samples:
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

## Computer Vision / Robotics Vision
- robot-frame coordinates
- target localization
- zone classification
- reachability reasoning
- robot-centric spatial interpretation

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **localization inspection pipeline**:

```text
RobotFramePoint
→ ZoneClassification
→ ReachabilityInspection
→ ValidityDecision
→ LocalizationSummary
→ Report
```

## 2. Python BST
Lưu:
- `pair_id`
- `localization_result_id`

## 3. `std::vector<LocalizationTargetSample>`
Danh sách input targets.

## 4. `std::vector<LocalizationInspectionResult>`
Danh sách kết quả localization inspection.

## 5. `std::stack<LocalizationDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Zone classification
Từ robot-frame point `(x, y, z)`, gán zone.

Ví dụ rule gợi ý:
- nếu `x < 0` → `REAR_ZONE`
- nếu `x >= 0` và `|y| <= center_band` → `CENTER_ZONE`
- nếu `x >= 0` và `y < -center_band` → `RIGHT_ZONE` hoặc `LEFT_ZONE` tùy convention
- nếu `x >= 0` và `y > center_band` → zone còn lại

Bạn phải ghi rõ **quy ước trục trái/phải**.

---

## Algorithm 2 — Reachability inspection
Tính:
- `forward_distance = x`
- `lateral_distance = y`
- `vertical_distance = z`

Kiểm tra:
- `min_forward <= x <= max_forward`
- `abs(y) <= max_lateral`
- `min_height <= z <= max_height`

---

## Algorithm 3 — Target validity
Target có thể được coi là valid nếu:
- nằm trước robot
- nằm trong vùng lateral hợp lý
- nằm trong khoảng cách có thể tiếp cận
- không vượt height range

---

## Algorithm 4 — Localization summary
Tạo kết luận kiểu:
- target nằm ở left-front zone
- target reachable
- target invalid vì quá xa / lệch ngang quá nhiều / nằm sau robot

---

## Algorithm 5 — Visualization / Reporting
Bắt buộc có ít nhất 1 dạng:
- zone distribution chart
- reachable vs invalid count chart
- forward / lateral scatter summary

---

# 9. Cấu trúc folder

```text
mini_project_63_robot_frame_target_localization_inspector/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ localization_inputs/
│  │  ├─ target_sample_01.txt
│  │  ├─ target_sample_02.txt
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ localization_result_report.txt
│     ├─ localization_summary_report.txt
│     ├─ runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ zone_distribution_plot.png
│     └─ reachability_plot.png
│
├─ config/
│  ├─ localization_sample_config.txt
│  ├─ robot_zone_config.txt
│  ├─ reachability_threshold_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ localization_graph_preview.py
│     ├─ localization_bst.py
│     ├─ numpy_localization_preview.py
│     ├─ matplotlib_localization_plotter.py
│     └─ synthetic_localization_input_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ LocalizationTargetSample.hpp
   │  ├─ RobotZoneConfig.hpp
   │  ├─ ReachabilityThresholdConfig.hpp
   │  ├─ TargetZone.hpp
   │  ├─ LocalizationInspectionResult.hpp
   │  ├─ LocalizationDebugRecord.hpp
   │  ├─ BaseTargetZoneClassifier.hpp
   │  ├─ TargetZoneClassifier.hpp
   │  ├─ BaseReachabilityInspector.hpp
   │  ├─ ReachabilityInspector.hpp
   │  ├─ BaseTargetValidityEngine.hpp
   │  ├─ TargetValidityEngine.hpp
   │  ├─ BaseLocalizationSummaryEngine.hpp
   │  ├─ LocalizationSummaryEngine.hpp
   │  ├─ RobotFrameTargetLocalizationInspector.hpp
   │  └─ LocalizationReportWriter.hpp
   │
   └─ src/
      ├─ TargetZoneClassifier.cpp
      ├─ ReachabilityInspector.cpp
      ├─ TargetValidityEngine.cpp
      ├─ LocalizationSummaryEngine.cpp
      ├─ RobotFrameTargetLocalizationInspector.cpp
      └─ LocalizationReportWriter.cpp
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
localization_sample_config_path
robot_zone_config_path
reachability_threshold_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `LocalizationInspectorConfigBuilder`

Tạo class con:

```python
class LocalizationInspectorConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_target_sample(
    pair_id,
    robot_x,
    robot_y,
    robot_z,
    label=None,
    source_projection_path=None,
    expected_zone=None
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

### `set_visualization_options(
    enable_zone_plot,
    enable_reachability_plot
)`

### `write_localization_sample_config()`
### `write_robot_zone_config()`
### `write_reachability_threshold_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `localization_graph_preview.py`

Tạo class:

```python
class LocalizationGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
RobotFramePoint
→ ZoneClassification
→ ReachabilityInspection
→ ValidityDecision
→ LocalizationSummary
→ Report
```

---

# 10.4 Python — `localization_bst.py`

Tạo BST cho:
- `pair_id`
- `localization_result_id`

---

# 10.5 Python — `numpy_localization_preview.py`

Tạo class:

```python
class NumPyLocalizationPreview:
```

## Hàm cần có

### `load_target_samples(path)`
### `classify_zone(point, zone_config)`
### `inspect_reachability(point, threshold_config)`
### `build_localization_summary(results)`

---

# 10.6 Python — `matplotlib_localization_plotter.py`

Tạo class:

```python
class MatplotlibLocalizationPlotter:
```

## Hàm cần có

### `plot_zone_distribution(results, save_path)`
### `plot_reachability_summary(results, save_path)`

---

# 10.7 C++ — `LocalizationTargetSample`

```cpp
struct LocalizationTargetSample
{
    std::string pair_id;

    double robot_x;
    double robot_y;
    double robot_z;

    std::string label;
    std::string source_projection_path;
    std::string expected_zone;
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

# 10.10 C++ — `TargetZone`

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

# 10.11 C++ — `LocalizationInspectionResult`

```cpp
struct LocalizationInspectionResult
{
    std::string pair_id;

    TargetZone zone;
    bool is_reachable;
    bool is_valid;

    double forward_distance;
    double lateral_distance;
    double vertical_distance;

    std::string summary;
};
```

---

# 10.12 C++ — `LocalizationDebugRecord`

```cpp
struct LocalizationDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.13 C++ — `BaseTargetZoneClassifier`

Tạo abstract class:

```cpp
class BaseTargetZoneClassifier
{
public:
    virtual TargetZone classify(
        const LocalizationTargetSample& sample,
        const RobotZoneConfig& config
    ) const = 0;

    virtual ~BaseTargetZoneClassifier() = default;
};
```

---

# 10.14 C++ — `TargetZoneClassifier`

Kế thừa `BaseTargetZoneClassifier`.

## Nhiệm vụ
- gán `LEFT_ZONE / CENTER_ZONE / RIGHT_ZONE / REAR_ZONE / OUT_OF_RANGE`

---

# 10.15 C++ — `BaseReachabilityInspector`

Tạo abstract class:

```cpp
class BaseReachabilityInspector
{
public:
    virtual bool inspect(
        const LocalizationTargetSample& sample,
        const ReachabilityThresholdConfig& config
    ) const = 0;

    virtual ~BaseReachabilityInspector() = default;
};
```

---

# 10.16 C++ — `ReachabilityInspector`

Kế thừa `BaseReachabilityInspector`.

## Nhiệm vụ
- kiểm tra target có reachable không

---

# 10.17 C++ — `BaseTargetValidityEngine`

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

# 10.18 C++ — `TargetValidityEngine`

Kế thừa `BaseTargetValidityEngine`.

## Nhiệm vụ
- gán valid / invalid

---

# 10.19 C++ — `BaseLocalizationSummaryEngine`

Tạo abstract class:

```cpp
class BaseLocalizationSummaryEngine
{
public:
    virtual std::string summarize(
        const LocalizationInspectionResult& result
    ) const = 0;

    virtual ~BaseLocalizationSummaryEngine() = default;
};
```

---

# 10.20 C++ — `LocalizationSummaryEngine`

Kế thừa `BaseLocalizationSummaryEngine`.

## Nhiệm vụ
- tạo summary kiểu:
```text
The target lies in the left-front zone and is reachable within the allowed forward range.
```

---

# 10.21 C++ — `RobotFrameTargetLocalizationInspector`

Tạo class trung tâm:

```cpp
class RobotFrameTargetLocalizationInspector
```

## Thuộc tính

```cpp
private:
    std::vector<LocalizationTargetSample> samples;

    RobotZoneConfig zone_config;
    ReachabilityThresholdConfig threshold_config;

    std::shared_ptr<BaseTargetZoneClassifier> zone_classifier;
    std::shared_ptr<BaseReachabilityInspector> reachability_inspector;
    std::shared_ptr<BaseTargetValidityEngine> validity_engine;
    std::shared_ptr<BaseLocalizationSummaryEngine> summary_engine;

    std::vector<LocalizationInspectionResult> results;
    std::stack<LocalizationDebugRecord> debug_history;
```

## Hàm cần có

### `load_localization_sample_config(const std::string& path)`
### `load_robot_zone_config(const std::string& path)`
### `load_reachability_threshold_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each sample:
    zone = zone_classifier.classify(sample, zone_config)
    reachable = reachability_inspector.inspect(sample, threshold_config)
    valid = validity_engine.evaluate(zone, reachable)

    build LocalizationInspectionResult
    result.summary = summary_engine.summarize(result)

    save result
    push debug history
```

### `const std::vector<LocalizationInspectionResult>& get_results() const`
### `std::vector<LocalizationDebugRecord> get_debug_history_reverse()`

---

# 10.22 C++ — `LocalizationReportWriter`

Tạo class:

```cpp
class LocalizationReportWriter
```

## Hàm cần có

### `write_localization_result_report(...)`
Ví dụ:

```text
[Localization Result]
Pair: target_01
Zone: LEFT_ZONE
Reachable: YES
Valid: YES
Forward: 1.25
Lateral: 0.32
Vertical: 0.88
```

### `write_localization_summary_report(...)`
Ví dụ:

```text
[Localization Summary]
Pair: target_01
The target lies in the left-front zone and remains reachable within the configured spatial limits.
```

### `write_runtime_report(...)`
### `write_reverse_debug_history(...)`

---

# 10.23 C++ — `main.cpp`

## Yêu cầu

```text
Load localization sample config
Load robot zone config
Load reachability threshold config
Load visualization config
Load runtime config

Create:
    TargetZoneClassifier
    ReachabilityInspector
    TargetValidityEngine
    LocalizationSummaryEngine

Create RobotFrameTargetLocalizationInspector
Run
Write reports
```

---

# 11. Luật robot-frame localization inspection của project

## Luật 1 — Phải dùng robot-frame point làm đầu vào chính
Không quay lại camera frame ở bài này.

## Luật 2 — Zone convention trái / phải phải ghi rõ
Ví dụ:
- `y > 0` là trái, `y < 0` là phải
hoặc ngược lại, nhưng phải thống nhất.

## Luật 3 — Reachability phải dựa trên threshold thật
Không được random gán reachable / invalid.

## Luật 4 — Target validity phải dựa trên zone + reachability
Không chỉ dựa vào 1 yếu tố.

## Luật 5 — Report phải usable cho target selection
Tức là sau project này bạn phải trả lời được:
- target ở zone nào?
- target có nằm trong vùng hợp lệ không?
- target có thể dùng cho bước navigation / interaction tiếp theo không?

---

# 12. Output mong muốn

## Config
```text
config/localization_sample_config.txt
config/robot_zone_config.txt
config/reachability_threshold_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/localization_result_report.txt
assets/outputs/localization_summary_report.txt
assets/outputs/runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/zone_distribution_plot.png
assets/outputs/reachability_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build target localization samples / zone / threshold configs
- preview zone classification + reachability
- visualize localization distribution

## C++
- phân tích robot-frame target points
- gán zone
- kiểm tra reachable / valid
- tạo localization inspection reports

## Computer Vision / Robotics Vision
Đây là project rất quan trọng vì nó là bước perception trả lời câu hỏi:

> **Một target trong robot frame có nằm ở vùng hợp lệ và có thể được robot sử dụng hay không?**

Pipeline lúc này sẽ là:

```text
pixel + depth
→ camera-frame point
→ robot-frame point
→ target zone classification
→ reachability inspection
→ localization validity report
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho target samples / zone / threshold / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / localization result ids
- [ ] Python có NumPy localization preview
- [ ] C++ có `LocalizationTargetSample`
- [ ] C++ có `RobotZoneConfig`
- [ ] C++ có `ReachabilityThresholdConfig`
- [ ] C++ có `TargetZone`
- [ ] C++ có `LocalizationInspectionResult`
- [ ] C++ có `TargetZoneClassifier`
- [ ] C++ có `ReachabilityInspector`
- [ ] C++ có `TargetValidityEngine`
- [ ] C++ có `LocalizationSummaryEngine`
- [ ] C++ có `RobotFrameTargetLocalizationInspector`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 63**, bước hợp lý nhất để tiếp tục Đợt 13 là:

# **Bài 64: Multi-Target Robot-Frame Spatial Prioritizer**

Ý tưởng:
```text
multiple robot-frame targets
→ zone-aware target comparison
→ reachability ranking
→ priority target selection
→ robot-centric target priority report
```

Tức là mạch sẽ đi:

```text
Bài 62: Camera-to-Robot Point Projection Analyzer
→ Bài 63: Robot-Frame Target Localization Inspector
→ Bài 64: Multi-Target Robot-Frame Spatial Prioritizer
```
