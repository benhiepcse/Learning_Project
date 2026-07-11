# 🤖 Bài 47: Stereo Depth Map Evaluator — Bộ đánh giá depth map từ disparity cho Humanoid Robot AI Perception

> Mini Project số 47 trong **Đợt 10**  
> **Bài 47 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10** và đi tiếp trực tiếp từ **Bài 46**.
>
> Nếu:
>
> - **Bài 41 → 45** đi từ calibration, rectification, correspondence, block matching
> - **Bài 46** chuyển sang **stereo calibration analyzer + baseline / focal / disparity sensitivity**
>
> thì **Bài 47** sẽ nối tiếp theo hướng rất tự nhiên:
>
> ```text
> disparity strip / disparity grid
> → convert sang depth
> → kiểm tra invalid depth
> → thống kê depth theo vùng
> → đánh giá chất lượng depth output
> ```
>
> Tức là thay vì chỉ phân tích công thức `Z = fx * B / d` ở mức sample hoặc rig, giờ bạn bắt đầu coi **depth output** là một thực thể perception cần được:
>
> - chuyển đổi
> - làm sạch
> - thống kê
> - đánh giá chất lượng
>
> Đây là bước rất sát với pipeline của **Robot Perception Engineer** vì robot không chỉ cần “có disparity”, mà cần **depth usable** cho navigation, obstacle awareness, manipulation, và 3D reasoning.

---

# 📌 Mục lục

- [1. Vì sao Bài 47 xuất hiện sau Bài 46](#1-vì-sao-bài-47-xuất-hiện-sau-bài-46)
- [2. Đợt 10 đang học gì](#2-đợt-10-đang-học-gì)
- [3. Bài 47 nâng từ Bài 46 lên chỗ nào](#3-bài-47-nâng-từ-bài-46-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật depth evaluation của project](#11-luật-depth-evaluation-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 47 xuất hiện sau Bài 46

## Bài 46 đang làm gì?
Bài 46 giúp bạn phân tích:

```text
stereo rig
+ disparity sample
→ depth
→ baseline sensitivity
→ focal sensitivity
→ rig comparison
```

Tức là bạn đã hiểu **depth phụ thuộc rig và disparity như thế nào**.

## Nhưng còn thiếu gì?
Trong perception thật, đầu ra thường không phải chỉ là **1 disparity sample** mà là:
- một **disparity strip**
- hoặc một **mini disparity map**
- hoặc một **depth map** nhỏ cần đánh giá

Bạn cần trả lời các câu hỏi như:
- chỗ nào depth bị invalid?
- vùng nào depth quá gần / quá xa?
- depth trung bình của một vùng là bao nhiêu?
- output depth có usable cho robot hay chưa?

## Bài 47 lấp đúng chỗ đó
Bài 47 nâng từ **sample-level depth reasoning** lên **map-level / strip-level depth evaluation**:

```text
disparity output
→ convert thành depth output
→ cleanup invalid values
→ compute statistics
→ evaluate quality
```

---

# 2. Đợt 10 đang học gì

Theo roadmap bạn gửi, **Đợt 10** xoay quanh:

## Python
### **Phase 9 — NumPy**
- reshape
- matrix multiplication
- robotics vector basics

## C++
- class
- inheritance
- virtual function
- vector
- pointer / reference

## Computer Vision
### **Phase 5 — Stereo Vision & Depth**
- Stereo Camera
- Disparity

## Vì sao Bài 47 vẫn bám đúng Đợt 10?
Vì Bài 47 dùng đúng các ý đó:

### Python / NumPy
- reshape disparity strip thành grid
- tạo depth matrix
- mask invalid depth
- tính thống kê theo hàng / vùng

### C++
- tổ chức depth evaluation runtime bằng OOP
- dùng vector / polymorphism cho nhiều rule xử lý depth

### CV / Stereo
- depth từ disparity
- đánh giá output depth
- chuẩn bị nền cho point cloud / 3D robot perception

---

# 3. Bài 47 nâng từ Bài 46 lên chỗ nào

## Bài 46
- tập trung vào **rig analysis**
- depth từ **sample disparity**
- sensitivity baseline / focal / disparity

## Bài 47
- tập trung vào **depth output**
- đầu vào là **disparity strip / mini disparity map**
- convert toàn bộ sang depth
- đánh giá chất lượng output

### Nói ngắn gọn:
- **Bài 46** hỏi: “Rig này đo depth thế nào?”
- **Bài 47** hỏi: “Depth output mà rig sinh ra có usable không?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Depth Map Evaluator**

System này nhận đầu vào là:
- một hoặc nhiều **stereo rig config**
- một hoặc nhiều **disparity strips / mini disparity maps**
- một số **depth evaluation rules**

## Mỗi disparity input có thể là:
### Dạng 1 — 1D strip
Ví dụ:
```text
[12, 12, 11, 10, 0, 0, 8, 7, 7]
```

### Dạng 2 — 2D mini disparity map
Ví dụ:
```text
[
  [12, 12, 11, 10],
  [11, 10, 10,  9],
  [ 0,  0,  8,  7]
]
```

Trong đó:
- `0` hoặc số âm có thể coi là invalid disparity
- disparity nhỏ → vật xa
- disparity lớn → vật gần

## Nhiệm vụ của system
### Với mỗi rig + disparity input:
1. convert disparity → depth
2. đánh dấu invalid depth
3. tính thống kê:
   - số pixel valid
   - số pixel invalid
   - depth min / max / average
4. tính thống kê theo vùng / theo hàng
5. optional: lọc / cleanup depth đơn giản
6. ghi report đánh giá chất lượng depth output

<p align="center">
  <img src="../../images/project_47.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build stereo rig config
→ build disparity strip / map config
→ reshape disparity thành grid
→ convert depth bằng NumPy
→ preview valid / invalid mask
→ preview depth statistics

C++
→ load stereo rig
→ load disparity strips / maps
→ convert disparity to depth
→ evaluate depth quality
→ summarize by row / region
→ export reports
```

Mục tiêu cốt lõi:
- hiểu **depth output** là thứ perception runtime thực sự dùng
- hiểu **invalid disparity / invalid depth** cần được xử lý
- hiểu **depth statistics** giúp robot ra quyết định tốt hơn
- chuẩn bị nền cho:
  - point cloud
  - obstacle reasoning
  - robot-centric distance analysis

---

# 6. Pipeline tổng thể

```text
Load Stereo Rig Config
Load Disparity Map Config
Load Depth Evaluation Config
Load Runtime Config

Create DepthConversionEngine
Create DepthValidationEngine
Create DepthStatisticsEngine
Create StereoDepthMapEvaluator

For each stereo rig:
    for each disparity strip / map:
        1. convert disparity to depth
        2. build valid / invalid mask
        3. compute depth statistics
        4. compute row / region summaries
        5. optional cleanup
        6. save result

After all inputs:
    write reports
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- BST / Graph / NumPy từ các đợt trước
- reshape
- broadcasting
- masking với NumPy

## C++
- class / inheritance
- virtual functions / polymorphism
- vector
- reference / pointer
- report-oriented runtime design

## Computer Vision / Stereo
- disparity
- depth from disparity
- invalid disparity
- depth map / disparity map
- local statistics cho perception

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **depth evaluation pipeline**:

```text
DisparityMap
→ DepthConversion
→ ValidityMask
→ DepthStatistics
→ RegionSummary
→ DepthQualityReport
```

## 2. Python BST
Lưu `rig_id`, `map_id`, `sample_id`.

## 3. `std::vector<StereoRigConfig>`
Danh sách rig stereo.

## 4. `std::vector<DisparityMapSample>`
Danh sách disparity strip / map.

## 5. `std::vector<DepthMapResult>`
Kết quả depth sau convert.

## 6. `std::vector<DepthRegionStatistic>`
Kết quả thống kê theo hàng / vùng.

## 7. `std::stack<DepthDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Disparity to depth conversion
Với mỗi disparity hợp lệ:

```text
Z = fx * baseline / disparity
```

Nếu disparity không hợp lệ (`<= 0`) thì depth phải bị đánh dấu invalid.

---

## Algorithm 2 — Validity mask
Tạo mask:
- `1` nếu depth hợp lệ
- `0` nếu depth invalid

Mask này phải dùng cho thống kê sau đó.

---

## Algorithm 3 — Depth statistics
Tính ít nhất:
- `valid_count`
- `invalid_count`
- `min_depth`
- `max_depth`
- `average_depth`

---

## Algorithm 4 — Row / region summary
Nếu disparity input là 2D mini map, hãy tính thống kê theo:
- từng hàng
- hoặc từng block nhỏ

Nếu disparity input là 1D strip, hãy tính thống kê theo:
- toàn strip
- nửa trái / nửa phải
- hoặc window regions

---

## Algorithm 5 — Optional cleanup
Bạn phải hỗ trợ **ít nhất 1 cleanup rule đơn giản**, ví dụ:
- thay invalid depth bằng `-1`
- clip depth vượt ngưỡng
- median smoothing trên strip

---

# 9. Cấu trúc folder

```text
mini_project_47_stereo_depth_map_evaluator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ depth_map_report.txt
│     ├─ depth_statistics_report.txt
│     ├─ depth_region_summary_report.txt
│     ├─ depth_cleanup_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_rig_config.txt
│  ├─ disparity_map_config.txt
│  ├─ depth_evaluation_config.txt
│  └─ depth_runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ depth_graph_preview.py
│     ├─ rig_bst.py
│     ├─ numpy_depth_map_preview.py
│     └─ synthetic_disparity_map_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoRigConfig.hpp
   │  ├─ DisparityMapSample.hpp
   │  ├─ DepthPixel.hpp
   │  ├─ DepthMapResult.hpp
   │  ├─ DepthRegionStatistic.hpp
   │  ├─ DepthDebugRecord.hpp
   │  ├─ BaseDepthConversionEngine.hpp
   │  ├─ StereoDepthConversionEngine.hpp
   │  ├─ BaseDepthValidationEngine.hpp
   │  ├─ DepthValidationEngine.hpp
   │  ├─ BaseDepthStatisticsEngine.hpp
   │  ├─ DepthStatisticsEngine.hpp
   │  ├─ StereoDepthMapEvaluator.hpp
   │  └─ StereoDepthReportWriter.hpp
   │
   └─ src/
      ├─ StereoDepthConversionEngine.cpp
      ├─ DepthValidationEngine.cpp
      ├─ DepthStatisticsEngine.cpp
      ├─ StereoDepthMapEvaluator.cpp
      └─ StereoDepthReportWriter.cpp
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
rig_config_path
map_config_path
evaluation_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoDepthMapEvaluatorConfigBuilder`

Tạo class con:

```python
class StereoDepthMapEvaluatorConfigBuilder(BaseConfigBuilder):
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

### `add_disparity_strip(
    map_id,
    disparity_values,
    target_label=None
)`

### `add_disparity_grid(
    map_id,
    disparity_grid,
    target_label=None
)`

### `set_depth_evaluation_options(
    invalid_disparity_value,
    replace_invalid_depth_with,
    enable_region_summary,
    enable_cleanup
)`

### `write_rig_config()`
### `write_map_config()`
### `write_evaluation_config()`
### `write_runtime_config()`

---

# 10.3 Python — `depth_graph_preview.py`

Tạo class:

```python
class DepthEvaluationGraph:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
DisparityMap
→ DepthConversion
→ ValidityMask
→ DepthStatistics
→ RegionSummary
→ Cleanup
→ Report
```

---

# 10.4 Python — `rig_bst.py`

Tạo BST cho `rig_id` / `map_id`.

---

# 10.5 Python — `numpy_depth_map_preview.py`

Tạo class:

```python
class NumPyDepthMapPreview:
```

## Hàm cần có

### `disparity_to_depth(disparity_array, fx, baseline, invalid_value=0)`
### `build_valid_mask(disparity_array, invalid_value=0)`
### `compute_depth_statistics(depth_array, valid_mask)`
### `compute_row_statistics(depth_array, valid_mask)`
### `cleanup_invalid_depth(depth_array, valid_mask, replacement=-1.0)`

---

# 10.6 C++ — `DisparityMapSample`

```cpp
struct DisparityMapSample
{
    std::string map_id;
    std::string target_label;

    bool is_grid;

    std::vector<double> disparity_strip;
    std::vector<std::vector<double>> disparity_grid;
};
```

---

# 10.7 C++ — `DepthPixel`

```cpp
struct DepthPixel
{
    double disparity_value;
    double depth_value;
    bool valid;
};
```

---

# 10.8 C++ — `DepthMapResult`

```cpp
struct DepthMapResult
{
    std::string rig_id;
    std::string map_id;

    bool is_grid;

    std::vector<DepthPixel> strip_depths;
    std::vector<std::vector<DepthPixel>> grid_depths;

    int valid_count;
    int invalid_count;
    double min_depth;
    double max_depth;
    double average_depth;
};
```

---

# 10.9 C++ — `DepthRegionStatistic`

```cpp
struct DepthRegionStatistic
{
    std::string rig_id;
    std::string map_id;
    std::string region_name;

    int valid_count;
    int invalid_count;
    double average_depth;
    double min_depth;
    double max_depth;
};
```

---

# 10.10 C++ — `DepthDebugRecord`

```cpp
struct DepthDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.11 C++ — `BaseDepthConversionEngine`

Tạo abstract class:

```cpp
class BaseDepthConversionEngine
{
public:
    virtual DepthMapResult convert(
        const StereoRigConfig& rig,
        const DisparityMapSample& sample
    ) const = 0;

    virtual ~BaseDepthConversionEngine() = default;
};
```

---

# 10.12 C++ — `StereoDepthConversionEngine`

Kế thừa `BaseDepthConversionEngine`.

## Nhiệm vụ
- convert disparity strip / grid sang depth
- gắn cờ valid / invalid cho từng pixel

---

# 10.13 C++ — `BaseDepthValidationEngine`

Tạo abstract class:

```cpp
class BaseDepthValidationEngine
{
public:
    virtual void cleanup_invalid_depth(DepthMapResult& result, double replacement) const = 0;
    virtual ~BaseDepthValidationEngine() = default;
};
```

---

# 10.14 C++ — `DepthValidationEngine`

Kế thừa `BaseDepthValidationEngine`.

## Nhiệm vụ
- xử lý invalid depth
- optional clip depth vượt ngưỡng
- optional cleanup depth strip / grid

---

# 10.15 C++ — `BaseDepthStatisticsEngine`

Tạo abstract class:

```cpp
class BaseDepthStatisticsEngine
{
public:
    virtual void compute_global_statistics(DepthMapResult& result) const = 0;
    virtual std::vector<DepthRegionStatistic> compute_region_statistics(
        const DepthMapResult& result
    ) const = 0;

    virtual ~BaseDepthStatisticsEngine() = default;
};
```

---

# 10.16 C++ — `DepthStatisticsEngine`

Kế thừa `BaseDepthStatisticsEngine`.

## Nhiệm vụ
- tính global statistics
- tính row / region statistics

---

# 10.17 C++ — `StereoDepthMapEvaluator`

Tạo class trung tâm:

```cpp
class StereoDepthMapEvaluator
```

## Thuộc tính

```cpp
private:
    std::vector<StereoRigConfig> rigs;
    std::vector<DisparityMapSample> maps;

    std::shared_ptr<BaseDepthConversionEngine> conversion_engine;
    std::shared_ptr<BaseDepthValidationEngine> validation_engine;
    std::shared_ptr<BaseDepthStatisticsEngine> statistics_engine;

    std::vector<DepthMapResult> results;
    std::vector<DepthRegionStatistic> region_statistics;
    std::stack<DepthDebugRecord> debug_history;
```

## Hàm cần có

### `load_rig_config(const std::string& path)`
### `load_map_config(const std::string& path)`
### `load_evaluation_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each rig:
    for each disparity map:
        result = conversion_engine.convert(rig, map)
        validation_engine.cleanup_invalid_depth(result, replacement)
        statistics_engine.compute_global_statistics(result)
        regions = statistics_engine.compute_region_statistics(result)

        save result
        save regions
        push debug history
```

### `const std::vector<DepthMapResult>& get_results() const`
### `const std::vector<DepthRegionStatistic>& get_region_statistics() const`
### `std::vector<DepthDebugRecord> get_debug_history_reverse()`

---

# 10.18 C++ — `StereoDepthReportWriter`

Tạo class:

```cpp
class StereoDepthReportWriter
```

## Hàm cần có

### `write_depth_map_report(...)`
Ví dụ:

```text
[Depth Map]
Rig: stereo_head_v1
Map: strip_01
Valid Pixels: 12
Invalid Pixels: 2
Average Depth: 2.83
```

### `write_depth_statistics_report(...)`
Ví dụ:

```text
[Depth Statistics]
Rig: stereo_head_v1
Map: grid_01
Min Depth: 0.82
Max Depth: 4.11
Average Depth: 2.25
```

### `write_depth_region_summary_report(...)`
Ví dụ:

```text
[Depth Region Summary]
Rig: stereo_head_v1
Map: grid_01
Region: row_0
Average Depth: 1.82
```

### `write_depth_cleanup_report(...)`
Ví dụ:

```text
[Depth Cleanup]
Rig: stereo_head_v1
Map: strip_01
Invalid depth replaced with -1
```

### `write_reverse_debug_history(...)`

---

# 10.19 C++ — `main.cpp`

## Yêu cầu

```text
Load stereo rig config
Load disparity map config
Load depth evaluation config
Load runtime config

Create:
    StereoDepthConversionEngine
    DepthValidationEngine
    DepthStatisticsEngine

Create StereoDepthMapEvaluator
Run
Write reports
```

---

# 11. Luật depth evaluation của project

## Luật 1 — Không được tính depth cho disparity không hợp lệ
Nếu disparity `<= 0` hoặc bằng invalid marker thì depth phải bị đánh dấu invalid.

## Luật 2 — Depth statistics chỉ được tính trên pixel valid
Không được lấy depth invalid vào min / max / average.

## Luật 3 — Cleanup phải diễn ra sau conversion và trước report cuối
Vì report phải phản ánh output depth cuối cùng mà runtime muốn sử dụng.

## Luật 4 — Phải hỗ trợ cả strip và grid
Vì project này là cầu nối từ mini runtime 1D sang perception 2D.

## Luật 5 — Region statistics phải có ít nhất một kiểu chia vùng rõ ràng
Ví dụ:
- theo hàng
- theo nửa trái / nửa phải
- theo block

---

# 12. Output mong muốn

## Config
```text
config/stereo_rig_config.txt
config/disparity_map_config.txt
config/depth_evaluation_config.txt
config/depth_runtime_config.txt
```

## Reports
```text
assets/outputs/depth_map_report.txt
assets/outputs/depth_statistics_report.txt
assets/outputs/depth_region_summary_report.txt
assets/outputs/depth_cleanup_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build disparity strip / grid config
- preview depth matrix bằng NumPy
- tạo valid mask và thống kê trước khi viết runtime C++

## C++
- convert disparity → depth
- cleanup invalid depth
- thống kê quality của depth output
- chuẩn bị đầu ra cho point cloud hoặc robot-centric distance reasoning

## Computer Vision / Robot Perception
Đây là project rất sát perception thật vì robot không dùng “disparity thô” trực tiếp. Nó cần:
- depth usable
- biết vùng nào thiếu depth
- biết vật thể đang gần hay xa
- biết chất lượng output đủ ổn cho bước tiếp theo hay chưa

Bài 47 vì vậy là bước đệm rất đẹp trước:
- **Bài 48: Stereo Point Cloud Mini Pipeline**
- **Bài 49: Camera-to-Robot Depth Projection**
- **Bài 50: Obstacle Distance Reasoning from Stereo Depth**

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho rig / disparity map / evaluation
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho rig ids / map ids
- [ ] Python có NumPy preview cho depth map / valid mask / row stats
- [ ] C++ có `DisparityMapSample`
- [ ] C++ có `DepthPixel`
- [ ] C++ có `DepthMapResult`
- [ ] C++ có `DepthRegionStatistic`
- [ ] C++ có `StereoDepthConversionEngine`
- [ ] C++ có `DepthValidationEngine`
- [ ] C++ có `DepthStatisticsEngine`
- [ ] C++ có `StereoDepthMapEvaluator`
- [ ] C++ ghi đủ 5 report

---

# 15. Gợi ý mở rộng

## 1. Thêm threshold theo “robot working range”
Ví dụ robot chỉ quan tâm depth từ `0.3m → 3.0m`, hãy đánh dấu pixel nào nằm ngoài range đó.

## 2. Thêm median filter hoặc hole filling đơn giản
Đây là bước đầu để làm disparity / depth post-processing.

## 3. Thêm so sánh nhiều rig trên cùng disparity map
Ví dụ cùng một disparity map nhưng với 2 rig khác nhau sẽ ra depth khác nhau.

## 4. Chuẩn bị cho Bài 48
Sau Bài 47, hướng đi rất đẹp là:

# **Bài 48: Stereo Point Cloud Mini Pipeline**

Ý tưởng:
```text
disparity / depth map
→ back-project sang 3D points
→ tạo point cloud mini
→ thống kê point ranges
→ point cloud quality report
```

Tức là:
- **Bài 46**: rig-level depth analysis
- **Bài 47**: depth map evaluation
- **Bài 48**: dựng 3D point cloud từ depth
