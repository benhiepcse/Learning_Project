# 🤖 Bài 46: Stereo Calibration Analyzer & Disparity Baseline Lab — Bộ phân tích hiệu chuẩn stereo và tác động baseline–disparity cho Humanoid Robot AI Perception

> Mini Project số 46 trong **Đợt 10**  
> **Bài 46 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10**.
>
> Sau chuỗi Đợt 9:
>
> - **Bài 41**: Camera Calibration & Back-Projection Geometry Simulator  
> - **Bài 42**: Multi-Camera Calibration Graph Simulator  
> - **Bài 43**: Stereo Rectification & Epipolar Geometry Playground  
> - **Bài 44**: Stereo Correspondence Cost Explorer  
> - **Bài 45**: Stereo Block Matching Mini Runtime
>
> thì **Đợt 10** chuyển trọng tâm sang:
>
> # **Calibration Analyzer**
>
> Tức là không chỉ “match disparity” nữa, mà bắt đầu trả lời các câu hỏi rất đúng kiểu **Robot Perception Engineer**:
>
> - stereo camera gồm những tham số gì?
> - baseline đổi thì disparity / depth nhạy ra sao?
> - focal length đổi thì depth thay đổi thế nào?
> - calibration table nào đang tốt / xấu?
> - một cặp điểm stereo có hợp lý với cấu hình camera hiện tại không?
>
> Vì vậy **Bài 46** sẽ là một project nối giữa **stereo geometry** và **depth reasoning**, nhưng vẫn bám đúng giới hạn của **Đợt 10**:
>
> ```text
> stereo calibration config
> → disparity samples
> → baseline / focal / disparity analysis
> → depth sensitivity report
> → stereo calibration analyzer
> ```

---

# 📌 Mục lục

- [1. Đợt 10 đang học gì](#1-đợt-10-đang-học-gì)
- [2. Bài 46 nâng từ Đợt 9 lên chỗ nào](#2-bài-46-nâng-từ-đợt-9-lên-chỗ-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Pipeline tổng thể](#5-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật stereo calibration của project](#10-luật-stereo-calibration-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý mở rộng](#14-gợi-ý-mở-rộng)

---

# 1. Đợt 10 đang học gì

Theo roadmap bạn gửi, **Đợt 10 (Ngày 19–20)** là:

# **Chủ đề: Calibration analyzer**

## Python
### **Phase 9 — Python Libraries**
- `Python NumPy`
  - reshape
  - matrix multiplication
  - robotics vector basics

## C++
### Ôn + áp dụng
- class
- inheritance
- virtual function
- vector
- pointer / reference

## Computer Vision
### **Phase 5 — Stereo Vision & Depth**
- `Stereo Camera`
- `Disparity`

## Ý nghĩa của Đợt 10
Đợt 10 là lúc bạn chuyển từ:
- “biết stereo matching / disparity là gì”

sang:
- “biết phân tích một **cấu hình stereo camera**”
- “biết đọc baseline, focal, disparity để suy ra **độ nhạy depth**”
- “biết một calibration setup sẽ ảnh hưởng perception ra sao”

Tức là bắt đầu chạm vào kiểu tư duy:
> **Calibration-aware perception analysis**

---

# 2. Bài 46 nâng từ Đợt 9 lên chỗ nào

## Đợt 9 đã cho bạn gì?
Bạn đã có:
- single-camera calibration
- multi-camera graph
- rectification
- epipolar geometry
- disparity cost
- block matching mini runtime

## Nhưng còn thiếu gì?
Bạn vẫn còn thiếu một bài chuyên về **phân tích stereo calibration như một hệ đo depth**:

- baseline lớn hơn thì disparity thay đổi ra sao?
- focal length lớn hơn thì depth estimation nhạy thế nào?
- cùng một disparity nhưng với 2 stereo rigs khác nhau thì depth có giống nhau không?
- cặp stereo nào “hợp” hơn cho khoảng cách robot đang quan tâm?

## Bài 46 lấp đúng chỗ đó
Bài 46 không đi sâu thêm vào SGM hay full disparity map.  
Thay vào đó, nó xây một **stereo calibration analyzer**:

```text
stereo config + disparity samples
→ compute depth
→ compare rigs
→ analyze baseline / focal / disparity sensitivity
→ write calibration report
```

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Calibration Analyzer & Disparity Baseline Lab**

System này quản lý **nhiều cấu hình stereo camera**.  
Mỗi stereo rig sẽ có:

- `rig_id`
- `fx`
- `fy`
- `cx`
- `cy`
- `baseline`
- optional image size / distortion metadata

Ngoài ra system còn có:
- một tập **disparity samples**
- một tập **depth query samples**
- một tập **baseline sensitivity scenarios**

## Nhiệm vụ của system
### Với mỗi stereo rig:
1. đọc calibration config
2. nhận các sample disparity
3. tính depth từ disparity
4. phân tích độ nhạy khi:
   - disparity thay đổi
   - baseline thay đổi
   - focal length thay đổi
5. so sánh nhiều rig với nhau
6. ghi report đánh giá rig

<p align="center">
  <img src="../../images/project_46.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build stereo rig config
→ build disparity sample config
→ build sensitivity scenarios
→ preview depth table bằng NumPy
→ preview baseline / focal analysis

C++
→ load stereo rig table
→ load disparity samples
→ compute depth per rig
→ compare multiple stereo rigs
→ analyze disparity-baseline-focal sensitivity
→ export stereo calibration reports
```

Mục tiêu cốt lõi:
- hiểu **depth = f * B / d** không chỉ là công thức, mà là **một hệ phân tích**
- hiểu **baseline** ảnh hưởng trực tiếp tới khả năng đo depth
- hiểu **cùng một disparity** trên rig khác nhau cho ra **depth khác nhau**
- chuẩn bị nền cho:
  - depth estimation
  - depth map interpretation
  - point cloud reconstruction
  - camera selection trong robot perception

---

# 5. Pipeline tổng thể

```text
Load Stereo Rig Config
Load Disparity Sample Config
Load Sensitivity Scenario Config
Load Runtime Config

Create StereoRigTable
Create DepthComputationEngine
Create SensitivityAnalysisEngine
Create StereoCalibrationAnalyzer

For each stereo rig:
    1. compute depth for each disparity sample
    2. compute min / max / average depth
    3. run baseline sensitivity analysis
    4. run focal sensitivity analysis
    5. run disparity sensitivity analysis

After all rigs:
    6. compare rigs on same disparity sample
    7. rank rigs by target depth range behavior
    8. write reports
```

---

# 6. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- BST / Graph / NumPy từ các đợt trước
- **NumPy reshape**
- **matrix multiplication**
- vector basics cho robotics / geometry

## C++
- class / inheritance
- virtual functions / polymorphism
- enum class
- vector
- pointer / reference
- report-oriented runtime design

## Computer Vision / Stereo
- stereo camera
- disparity
- depth from disparity
- baseline
- focal length
- calibration-aware analysis

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **calibration analysis pipeline**:

```text
StereoRig
→ DisparitySamples
→ DepthComputation
→ BaselineSensitivity
→ FocalSensitivity
→ RigComparison
→ CalibrationReport
```

## 2. Python BST
Lưu `rig_id`, `sample_id`, hoặc `scenario_id`.

## 3. `std::vector<StereoRigConfig>`
Danh sách stereo rigs.

## 4. `std::vector<DisparitySample>`
Danh sách disparity samples.

## 5. `std::vector<RigDepthResult>`
Kết quả depth theo từng rig.

## 6. `std::vector<SensitivityRecord>`
Kết quả phân tích baseline / focal / disparity sensitivity.

## 7. `std::stack<CalibrationDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Depth from disparity
Với mỗi rig và sample disparity:

```text
Z = fx * baseline / disparity
```

Trong project, phải kiểm tra:
- disparity > 0
- baseline > 0
- fx > 0

---

## Algorithm 2 — Baseline sensitivity analysis
Với một sample disparity cố định, thử nhiều baseline khác nhau:

```text
baseline = 0.06, 0.10, 0.14, ...
```

và xem depth thay đổi ra sao.

---

## Algorithm 3 — Focal sensitivity analysis
Giữ disparity cố định, thay đổi `fx` và quan sát depth.

---

## Algorithm 4 — Disparity sensitivity analysis
Giữ rig cố định, thay đổi disparity:
- disparity nhỏ → vật xa
- disparity lớn → vật gần

Bạn phải cho thấy điều này trong report.

---

## Algorithm 5 — Rig comparison
Cho cùng một disparity sample, so sánh:
- `rig_A depth`
- `rig_B depth`
- `rig_C depth`

để xem rig nào nhạy hơn / phù hợp hơn cho một khoảng đo nào đó.

---

# 8. Cấu trúc folder

```text
mini_project_46_stereo_calibration_analyzer_disparity_baseline_lab/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ stereo_rig_depth_report.txt
│     ├─ baseline_sensitivity_report.txt
│     ├─ focal_sensitivity_report.txt
│     ├─ rig_comparison_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_rig_config.txt
│  ├─ disparity_sample_config.txt
│  ├─ sensitivity_scenario_config.txt
│  └─ calibration_runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ calibration_graph_preview.py
│     ├─ rig_bst.py
│     ├─ numpy_depth_preview.py
│     └─ synthetic_disparity_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoRigConfig.hpp
   │  ├─ DisparitySample.hpp
   │  ├─ RigDepthResult.hpp
   │  ├─ SensitivityRecord.hpp
   │  ├─ CalibrationDebugRecord.hpp
   │  ├─ BaseDepthComputationEngine.hpp
   │  ├─ StereoDepthComputationEngine.hpp
   │  ├─ BaseSensitivityAnalysisEngine.hpp
   │  ├─ BaselineSensitivityEngine.hpp
   │  ├─ FocalSensitivityEngine.hpp
   │  ├─ DisparitySensitivityEngine.hpp
   │  ├─ StereoCalibrationAnalyzer.hpp
   │  └─ StereoCalibrationReportWriter.hpp
   │
   └─ src/
      ├─ StereoDepthComputationEngine.cpp
      ├─ BaselineSensitivityEngine.cpp
      ├─ FocalSensitivityEngine.cpp
      ├─ DisparitySensitivityEngine.cpp
      ├─ StereoCalibrationAnalyzer.cpp
      └─ StereoCalibrationReportWriter.cpp
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
rig_config_path
disparity_config_path
scenario_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `StereoCalibrationAnalyzerConfigBuilder`

Tạo class con:

```python
class StereoCalibrationAnalyzerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_stereo_rig(
    rig_id,
    fx,
    fy,
    cx,
    cy,
    baseline,
    image_width,
    image_height
)`

### `add_disparity_sample(
    sample_id,
    disparity_value,
    target_label=None
)`

### `add_sensitivity_scenario(
    scenario_id,
    rig_id,
    disparity_value,
    baseline_candidates,
    focal_candidates
)`

### `set_runtime_options(
    enable_rig_comparison,
    save_all_depth_records,
    target_depth_min=None,
    target_depth_max=None
)`

### `write_rig_config()`
### `write_disparity_config()`
### `write_scenario_config()`
### `write_runtime_config()`

---

# 9.3 Python — `calibration_graph_preview.py`

Tạo class:

```python
class CalibrationAnalysisGraph:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
StereoRig
→ DisparitySamples
→ DepthComputation
→ BaselineSensitivity
→ FocalSensitivity
→ RigComparison
→ CalibrationReport
```

---

# 9.4 Python — `rig_bst.py`

Tạo BST cho `rig_id` / `sample_id`.

---

# 9.5 Python — `numpy_depth_preview.py`

Tạo class:

```python
class NumPyDepthPreview:
```

## Hàm cần có

### `compute_depth(fx, baseline, disparity)`
### `compute_depth_series(fx, baseline, disparity_values)`
### `baseline_sensitivity(fx, baseline_values, disparity)`
### `focal_sensitivity(focal_values, baseline, disparity)`
### `build_depth_matrix(focal_values, baseline_values, disparity)`

> `build_depth_matrix(...)` nên dùng đúng tinh thần **Đợt 10**:
- `reshape`
- matrix multiplication hoặc broadcasting
- robotics vector basics

---

# 9.6 C++ — `StereoRigConfig`

```cpp
struct StereoRigConfig
{
    std::string rig_id;

    double fx;
    double fy;
    double cx;
    double cy;

    double baseline;

    int image_width;
    int image_height;
};
```

---

# 9.7 C++ — `DisparitySample`

```cpp
struct DisparitySample
{
    std::string sample_id;
    double disparity_value;
    std::string target_label;
};
```

---

# 9.8 C++ — `RigDepthResult`

```cpp
struct RigDepthResult
{
    std::string rig_id;
    std::string sample_id;

    double disparity_value;
    double depth_value;

    bool valid;
};
```

---

# 9.9 C++ — `SensitivityRecord`

```cpp
enum class SensitivityType
{
    BASELINE,
    FOCAL,
    DISPARITY
};

struct SensitivityRecord
{
    std::string scenario_id;
    std::string rig_id;

    SensitivityType type;

    double input_value;
    double output_depth;
};
```

---

# 9.10 C++ — `CalibrationDebugRecord`

```cpp
struct CalibrationDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.11 C++ — `BaseDepthComputationEngine`

Tạo abstract class:

```cpp
class BaseDepthComputationEngine
{
public:
    virtual bool compute_depth(
        const StereoRigConfig& rig,
        const DisparitySample& sample,
        RigDepthResult& output
    ) const = 0;

    virtual ~BaseDepthComputationEngine() = default;
};
```

---

# 9.12 C++ — `StereoDepthComputationEngine`

Kế thừa `BaseDepthComputationEngine`.

## Nhiệm vụ
Tính depth theo công thức:

```text
Z = fx * baseline / disparity
```

và validate dữ liệu.

---

# 9.13 C++ — `BaseSensitivityAnalysisEngine`

Tạo abstract class:

```cpp
class BaseSensitivityAnalysisEngine
{
public:
    virtual std::vector<SensitivityRecord> analyze(
        const StereoRigConfig& rig,
        double disparity_value,
        const std::vector<double>& candidates,
        const std::string& scenario_id
    ) const = 0;

    virtual ~BaseSensitivityAnalysisEngine() = default;
};
```

---

# 9.14 C++ — `BaselineSensitivityEngine`

Kế thừa `BaseSensitivityAnalysisEngine`.

## Nhiệm vụ
Với mỗi baseline candidate:
- tạo `SensitivityRecord`
- tính depth khi thay baseline

---

# 9.15 C++ — `FocalSensitivityEngine`

Kế thừa `BaseSensitivityAnalysisEngine`.

## Nhiệm vụ
Với mỗi focal candidate:
- tính depth khi thay `fx`

---

# 9.16 C++ — `DisparitySensitivityEngine`

Tạo class riêng hoặc dùng lại base engine tùy bạn, nhưng phải có phân tích:
- disparity tăng / giảm
- depth thay đổi ra sao

---

# 9.17 C++ — `StereoCalibrationAnalyzer`

Tạo class trung tâm:

```cpp
class StereoCalibrationAnalyzer
```

## Thuộc tính

```cpp
private:
    std::vector<StereoRigConfig> rigs;
    std::vector<DisparitySample> samples;

    std::shared_ptr<BaseDepthComputationEngine> depth_engine;
    std::shared_ptr<BaseSensitivityAnalysisEngine> baseline_engine;
    std::shared_ptr<BaseSensitivityAnalysisEngine> focal_engine;

    std::vector<RigDepthResult> depth_results;
    std::vector<SensitivityRecord> sensitivity_records;
    std::stack<CalibrationDebugRecord> debug_history;
```

## Hàm cần có

### `load_rig_config(const std::string& path)`
### `load_disparity_samples(const std::string& path)`
### `load_scenarios(const std::string& path)`

### `run()`
Pseudo:

```text
for each rig:
    for each disparity sample:
        compute depth
        save RigDepthResult

    run baseline sensitivity scenarios of this rig
    run focal sensitivity scenarios of this rig
    run disparity sensitivity summary of this rig

push debug history
```

### `group_depth_results_by_rig()`
### `group_depth_results_by_sample()`
### `const std::vector<RigDepthResult>& get_depth_results() const`
### `const std::vector<SensitivityRecord>& get_sensitivity_records() const`
### `std::vector<CalibrationDebugRecord> get_debug_history_reverse()`

---

# 9.18 C++ — `StereoCalibrationReportWriter`

Tạo class:

```cpp
class StereoCalibrationReportWriter
```

## Hàm cần có

### `write_stereo_rig_depth_report(...)`
Ví dụ:

```text
[Depth Report]
Rig: stereo_head_v1
Sample: disp_10
Disparity: 10
Depth: 4.20
```

### `write_baseline_sensitivity_report(...)`
Ví dụ:

```text
[Baseline Sensitivity]
Scenario: baseline_test_01
baseline=0.06 -> depth=2.40
baseline=0.10 -> depth=4.00
baseline=0.14 -> depth=5.60
```

### `write_focal_sensitivity_report(...)`
Ví dụ:

```text
[Focal Sensitivity]
Scenario: focal_test_01
fx=400 -> depth=2.00
fx=700 -> depth=3.50
fx=900 -> depth=4.50
```

### `write_rig_comparison_report(...)`
Ví dụ:

```text
[Rig Comparison]
Sample: disp_20

stereo_head_v1 -> depth=2.10
stereo_head_v2 -> depth=2.60
stereo_wrist_v1 -> depth=1.75
```

### `write_reverse_debug_history(...)`

---

# 9.19 C++ — `main.cpp`

## Yêu cầu

```text
Load stereo rig config
Load disparity samples
Load sensitivity scenarios
Load runtime config

Create:
    StereoDepthComputationEngine
    BaselineSensitivityEngine
    FocalSensitivityEngine
    DisparitySensitivityEngine

Create StereoCalibrationAnalyzer
Run
Write reports
```

---

# 10. Luật stereo calibration của project

## Luật 1 — Không được tính depth nếu disparity <= 0
Vì công thức `Z = fx * B / d` sẽ vô nghĩa hoặc sai ngữ nghĩa khi `d <= 0`.

## Luật 2 — Baseline phải > 0
Nếu baseline bằng 0 thì đó không còn là stereo rig có ý nghĩa đo depth.

## Luật 3 — `fx` phải > 0
Không cho phép focal length không hợp lệ.

## Luật 4 — Cùng một disparity sample có thể cho depth khác nhau trên các rig khác nhau
Đây là điều bắt buộc project phải thể hiện được.

## Luật 5 — Sensitivity report phải tách rõ theo loại
Ít nhất phải có:
- baseline sensitivity
- focal sensitivity
- disparity sensitivity

---

# 11. Output mong muốn

## Config
```text
config/stereo_rig_config.txt
config/disparity_sample_config.txt
config/sensitivity_scenario_config.txt
config/calibration_runtime_config.txt
```

## Reports
```text
assets/outputs/stereo_rig_depth_report.txt
assets/outputs/baseline_sensitivity_report.txt
assets/outputs/focal_sensitivity_report.txt
assets/outputs/rig_comparison_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build stereo rig configs
- build disparity samples / scenarios
- preview depth tables bằng NumPy
- phân tích baseline / focal trước khi code runtime

## C++
- load nhiều stereo rigs
- tính depth cho từng disparity sample
- phân tích sensitivity
- so sánh rigs
- tạo calibration report

## Computer Vision / Robot Perception
Đây là một project rất sát với công việc perception thật vì bạn không chỉ chạy thuật toán, mà còn phải hiểu:
- rig stereo nào phù hợp cho robot của mình
- khoảng depth nào đang nhạy / kém nhạy
- thay đổi baseline sẽ ảnh hưởng tới độ đo thế nào
- disparity nhỏ / lớn phản ánh khoảng cách ra sao

Đây là nền rất tốt trước khi sang:
- depth map analysis
- point cloud
- camera-to-robot perception pipeline

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho stereo rigs / disparity samples / scenarios
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho rig ids / sample ids
- [ ] Python có NumPy preview cho depth / baseline / focal matrix
- [ ] C++ có `StereoRigConfig`
- [ ] C++ có `DisparitySample`
- [ ] C++ có `RigDepthResult`
- [ ] C++ có `SensitivityRecord`
- [ ] C++ có `StereoDepthComputationEngine`
- [ ] C++ có `BaselineSensitivityEngine`
- [ ] C++ có `FocalSensitivityEngine`
- [ ] C++ có `StereoCalibrationAnalyzer`
- [ ] C++ ghi đủ 5 report

---

# 14. Gợi ý mở rộng

## 1. Thêm disparity noise simulation
Ví dụ disparity bị nhiễu `±1`, `±2`, rồi xem depth nhảy bao nhiêu.

## 2. Thêm target depth range scoring
Ví dụ robot chủ yếu làm việc ở `0.5m → 2.0m`, hãy chấm rig nào phù hợp nhất cho dải này.

## 3. Thêm left-right disparity pair log
Nếu muốn nối lại với Bài 43–45, bạn có thể lưu luôn:
- `u_left`
- `u_right`
- `disparity`
- `depth`

## 4. Chuẩn bị cho Bài 47
Sau Bài 46, nếu bám đúng mạch Đợt 10 → 12 thì bước đi rất đẹp là:

# **Bài 47: Stereo Depth Map Evaluator**

Ý tưởng:
```text
disparity strip / disparity samples
→ depth conversion
→ invalid depth cleanup
→ depth statistics
→ depth quality report
```

Hoặc nếu muốn bám sát robotics hơn:

# **Bài 47: Stereo Depth to Robot Distance Probe**

Ý tưởng:
```text
stereo depth samples
→ estimate 3D forward distance
→ compare target ranges
→ robot-centric depth analysis
```
