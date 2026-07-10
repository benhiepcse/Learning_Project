# 🤖 Bài 53: Depth Map Quality Analyzer — Bộ phân tích chất lượng depth map cho Humanoid Robot AI Perception

> Mini Project số 53 trong **Đợt 11**  
> **Bài 53 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11** và đi tiếp trực tiếp từ **Bài 52**.
>
> Nếu:
>
> - **Bài 51** giúp bạn tạo và so sánh **disparity maps**
> - **Bài 52** giúp bạn chuyển **disparity → depth**
>
> thì **Bài 53** sẽ là bước tiếp theo rất quan trọng:
>
> ```text
> depth map
> → phân tích vùng invalid / hole / outlier
> → phân tích vùng near / mid / far
> → thống kê theo hàng / vùng / ROI
> → chấm quality score
> → sinh depth reliability report
> ```
>
> Đây là bước biến **depth map thô** thành **depth map đã được đánh giá chất lượng**, trước khi bạn dùng nó cho:
>
> - point cloud reconstruction
> - obstacle perception
> - robot-frame geometry
> - walking safety / navigation reasoning

---

# 📌 Mục lục

- [1. Vì sao Bài 53 xuất hiện sau Bài 52](#1-vì-sao-bài-53-xuất-hiện-sau-bài-52)
- [2. Đợt 11 đang đẩy pipeline đi đâu](#2-đợt-11-đang-đẩy-pipeline-đi-đâu)
- [3. Bài 53 nâng từ Bài 52 lên chỗ nào](#3-bài-53-nâng-từ-bài-52-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật depth quality analysis của project](#11-luật-depth-quality-analysis-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 53 xuất hiện sau Bài 52

## Bài 52 đang làm gì?
Bài 52 giúp bạn có:

```text
disparity map
→ depth map
→ invalid depth cleanup
→ depth statistics
→ BM vs SGM depth comparison
```

Tức là bạn đã có **depth map**.

## Nhưng còn thiếu gì?
Không phải depth map nào cũng đủ tốt để đi tiếp sang point cloud và robot reasoning.  
Robot cần biết:

- vùng nào depth bị rỗng / invalid nhiều?
- vùng nào depth quá nhiễu?
- phần gần camera có ổn không?
- center corridor có đáng tin không?
- depth map này đủ ổn để back-project thành point cloud chưa?

## Bài 53 lấp đúng chỗ đó
Bài 53 sẽ làm:

```text
depth map
→ invalid/hole analysis
→ near / mid / far region analysis
→ row / ROI statistics
→ quality scoring
→ reliability report
```

Đây là bước chuyển từ **“đã có depth”** sang **“biết depth đó đáng tin đến mức nào”**.

---

# 2. Đợt 11 đang đẩy pipeline đi đâu

Nếu gom Bài 51–53 lại, mạch Đợt 11 đang rất rõ:

```text
Stereo pair
→ disparity map
→ depth map
→ depth quality analysis
```

Nghĩa là Đợt 11 không chỉ dừng ở việc “tạo output stereo”, mà còn bắt đầu trả lời:

> **Output đó có đủ tốt để robot dùng tiếp không?**

Đây là mindset rất đúng của perception engineer:
- không chỉ sinh kết quả
- mà còn phải **đánh giá chất lượng kết quả**

---

# 3. Bài 53 nâng từ Bài 52 lên chỗ nào

## Bài 52
- disparity → depth
- cleanup invalid depth
- depth statistics cơ bản

## Bài 53
- depth quality analysis theo vùng
- invalid / hole density
- near / far depth stability
- center ROI reliability
- quality scoring / reliability report

### Nói ngắn gọn:
- **Bài 52** hỏi: “Depth map có giá trị bao nhiêu?”
- **Bài 53** hỏi: “Depth map này có đáng tin để dùng cho 3D perception không?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Depth Map Quality Analyzer**

System này nhận đầu vào là:
- một hoặc nhiều **depth map samples**
- một bộ **quality thresholds**
- một bộ **ROI / region rules**
- optional **source info** (BM hay SGM)

## Mỗi depth sample tối thiểu chứa
- `pair_id`
- `algorithm_type`
- `depth_map_path`
- optional `scene_label`

## Hệ thống phải phân tích ít nhất các khía cạnh sau

## A. Invalid / hole analysis
- số pixel invalid
- tỉ lệ invalid
- vùng invalid tập trung ở đâu
- hàng nào / ROI nào có nhiều invalid nhất

## B. Near / Mid / Far depth analysis
Ví dụ chia depth thành:
- `NEAR`
- `MID`
- `FAR`

rồi thống kê:
- số pixel
- average depth
- min/max depth
- tỉ lệ invalid trong từng vùng

## C. Center ROI analysis
Ví dụ lấy vùng giữa ảnh làm:
- **walking corridor ROI**
- **forward perception ROI**

rồi tính:
- valid ratio
- average depth
- min depth
- quality label

## D. Quality scoring
Cho mỗi depth map, gán:
- `GOOD`
- `USABLE`
- `POOR`

hoặc thang điểm 0–100.

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build depth sample config
→ build ROI / threshold config
→ preview invalid masks bằng NumPy
→ preview region statistics
→ visualize quality bằng Matplotlib

C++
→ load depth maps
→ analyze invalid / hole density
→ analyze near / far / ROI statistics
→ compute quality score
→ export reliability report
```

Mục tiêu cốt lõi:
- hiểu depth map không chỉ có “giá trị depth” mà còn có **mức độ tin cậy**
- biết cách đánh giá **ROI quan trọng** cho robot
- biết cách tìm **vùng hỏng / vùng yếu** trong depth map
- chuẩn bị nền cho:
  - point cloud reconstruction đáng tin
  - robot obstacle reasoning
  - free-space estimation

---

# 6. Pipeline tổng thể

```text
Load Depth Sample Config
Load Quality Threshold Config
Load ROI Config
Load Visualization Config
Load Runtime Config

Create InvalidDepthAnalysisEngine
Create RegionDepthAnalysisEngine
Create ROIQualityAnalysisEngine
Create DepthQualityScoringEngine
Create DepthMapQualityAnalyzer

For each depth sample:
    1. load depth map
    2. analyze invalid / hole pixels
    3. analyze near / mid / far regions
    4. analyze center ROI quality
    5. compute overall quality score
    6. save results

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
- NumPy masking / slicing
- Matplotlib
- dict / list / BST / Graph

## C++
- class / inheritance / virtual function
- vector
- enum
- OpenCV matrix handling
- report-oriented runtime design

## Computer Vision / Stereo / Depth
- depth map
- invalid depth
- ROI analysis
- depth range analysis
- quality evaluation mindset

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **depth quality pipeline**:

```text
DepthMap
→ InvalidAnalysis
→ RegionAnalysis
→ ROIAnalysis
→ QualityScoring
→ ReliabilityReport
```

## 2. Python BST
Lưu `pair_id`, `depth_result_id`, `quality_profile_id`.

## 3. `std::vector<DepthMapSample>`
Danh sách depth map samples.

## 4. `std::vector<DepthQualityResult>`
Kết quả quality cho từng depth map.

## 5. `std::vector<ROIQualityStatistic>`
Thống kê ROI.

## 6. `std::vector<RegionDepthStatistic>`
Thống kê near / mid / far.

## 7. `std::stack<DepthQualityDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Invalid depth analysis
Xác định pixel invalid theo marker, ví dụ:
- `depth <= 0`
- hoặc `depth == invalid_marker`

Tính ít nhất:
- invalid count
- invalid ratio
- row invalid histogram
- center ROI invalid ratio

---

## Algorithm 2 — Region depth analysis
Chia depth thành ít nhất 3 vùng:
- `NEAR`
- `MID`
- `FAR`

Ví dụ theo ngưỡng:
- near: `0 < d <= 1.0`
- mid: `1.0 < d <= 3.0`
- far: `d > 3.0`

Tính:
- count
- min / max / average depth
- valid ratio

---

## Algorithm 3 — ROI quality analysis
Phân tích ít nhất 1 ROI trung tâm, ví dụ:
- center corridor ROI

Tính:
- valid ratio
- average depth
- min depth
- quality label cho ROI

---

## Algorithm 4 — Overall quality scoring
Cho mỗi depth map, tính quality score dựa trên:
- invalid ratio toàn ảnh
- invalid ratio ở center ROI
- độ phủ near / mid / far
- min depth / average depth hợp lý hay không

---

## Algorithm 5 — Visualization
Bắt buộc có ít nhất 1 dạng visualize bằng Matplotlib, ví dụ:
- invalid ratio bar chart
- near/mid/far distribution chart
- row invalid histogram
- ROI quality comparison plot

---

# 9. Cấu trúc folder

```text
mini_project_53_depth_map_quality_analyzer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ depth_inputs/
│  │  ├─ depth_bm_pair_01.png
│  │  ├─ depth_sgm_pair_01.png
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ depth_quality_report.txt
│     ├─ invalid_depth_report.txt
│     ├─ roi_quality_report.txt
│     ├─ region_depth_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ invalid_ratio_plot.png
│     ├─ region_distribution_plot.png
│     └─ roi_quality_plot.png
│
├─ config/
│  ├─ depth_sample_config.txt
│  ├─ quality_threshold_config.txt
│  ├─ roi_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ depth_quality_graph_preview.py
│     ├─ depth_quality_bst.py
│     ├─ numpy_depth_quality_preview.py
│     ├─ matplotlib_depth_quality_plotter.py
│     └─ synthetic_depth_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ DepthMapSample.hpp
   │  ├─ DepthQualityThresholdConfig.hpp
   │  ├─ ROIConfig.hpp
   │  ├─ RegionDepthStatistic.hpp
   │  ├─ ROIQualityStatistic.hpp
   │  ├─ DepthQualityResult.hpp
   │  ├─ DepthQualityDebugRecord.hpp
   │  ├─ BaseInvalidDepthAnalysisEngine.hpp
   │  ├─ InvalidDepthAnalysisEngine.hpp
   │  ├─ BaseRegionDepthAnalysisEngine.hpp
   │  ├─ RegionDepthAnalysisEngine.hpp
   │  ├─ BaseROIQualityAnalysisEngine.hpp
   │  ├─ ROIQualityAnalysisEngine.hpp
   │  ├─ BaseDepthQualityScoringEngine.hpp
   │  ├─ DepthQualityScoringEngine.hpp
   │  ├─ DepthMapQualityAnalyzer.hpp
   │  └─ DepthQualityReportWriter.hpp
   │
   └─ src/
      ├─ InvalidDepthAnalysisEngine.cpp
      ├─ RegionDepthAnalysisEngine.cpp
      ├─ ROIQualityAnalysisEngine.cpp
      ├─ DepthQualityScoringEngine.cpp
      ├─ DepthMapQualityAnalyzer.cpp
      └─ DepthQualityReportWriter.cpp
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
depth_sample_config_path
threshold_config_path
roi_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `DepthQualityAnalyzerConfigBuilder`

Tạo class con:

```python
class DepthQualityAnalyzerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_depth_sample(
    pair_id,
    algorithm_type,
    depth_map_path,
    scene_label=None
)`

### `set_quality_thresholds(
    max_invalid_ratio,
    max_center_invalid_ratio,
    near_depth_max,
    mid_depth_max
)`

### `set_center_roi(
    x_min,
    y_min,
    x_max,
    y_max
)`

### `set_visualization_options(
    enable_invalid_plot,
    enable_region_plot,
    enable_roi_plot
)`

### `write_depth_sample_config()`
### `write_threshold_config()`
### `write_roi_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `depth_quality_graph_preview.py`

Tạo class:

```python
class DepthQualityGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
DepthMap
→ InvalidAnalysis
→ RegionAnalysis
→ ROIAnalysis
→ QualityScoring
→ Report
```

---

# 10.4 Python — `depth_quality_bst.py`

Tạo BST cho `pair_id` / `quality_id`.

---

# 10.5 Python — `numpy_depth_quality_preview.py`

Tạo class:

```python
class NumPyDepthQualityPreview:
```

## Hàm cần có

### `compute_invalid_mask(depth_map, invalid_marker=-1.0)`
### `compute_invalid_statistics(depth_map, invalid_marker=-1.0)`
### `analyze_depth_regions(depth_map, near_max, mid_max, invalid_marker=-1.0)`
### `analyze_center_roi(depth_map, roi, invalid_marker=-1.0)`
### `compute_quality_score(...)`

---

# 10.6 Python — `matplotlib_depth_quality_plotter.py`

Tạo class:

```python
class MatplotlibDepthQualityPlotter:
```

## Hàm cần có

### `plot_invalid_ratio(summary_data, save_path)`
### `plot_region_distribution(region_stats, save_path)`
### `plot_roi_quality(roi_stats, save_path)`

---

# 10.7 C++ — `DepthMapSample`

```cpp
enum class DepthSourceType
{
    BM,
    SGM
};

struct DepthMapSample
{
    std::string pair_id;
    DepthSourceType algorithm_type;
    std::string depth_map_path;
    std::string scene_label;
};
```

---

# 10.8 C++ — `DepthQualityThresholdConfig`

```cpp
struct DepthQualityThresholdConfig
{
    double max_invalid_ratio;
    double max_center_invalid_ratio;
    double near_depth_max;
    double mid_depth_max;
};
```

---

# 10.9 C++ — `ROIConfig`

```cpp
struct ROIConfig
{
    int x_min;
    int y_min;
    int x_max;
    int y_max;
};
```

---

# 10.10 C++ — `RegionDepthStatistic`

```cpp
enum class DepthRegionType
{
    NEAR,
    MID,
    FAR
};

struct RegionDepthStatistic
{
    std::string pair_id;
    DepthRegionType region_type;

    int valid_count;
    int invalid_count;
    double min_depth;
    double max_depth;
    double average_depth;
};
```

---

# 10.11 C++ — `ROIQualityStatistic`

```cpp
struct ROIQualityStatistic
{
    std::string pair_id;

    int valid_count;
    int invalid_count;
    double valid_ratio;
    double average_depth;
    double min_depth;
};
```

---

# 10.12 C++ — `DepthQualityResult`

```cpp
enum class DepthQualityLabel
{
    GOOD,
    USABLE,
    POOR
};

struct DepthQualityResult
{
    std::string pair_id;
    DepthSourceType algorithm_type;

    double invalid_ratio;
    double center_invalid_ratio;
    double quality_score;
    DepthQualityLabel label;
};
```

---

# 10.13 C++ — `DepthQualityDebugRecord`

```cpp
struct DepthQualityDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.14 C++ — `BaseInvalidDepthAnalysisEngine`

Tạo abstract class:

```cpp
class BaseInvalidDepthAnalysisEngine
{
public:
    virtual std::pair<int, int> analyze_invalid_pixels(
        const cv::Mat& depth_map,
        double invalid_marker
    ) const = 0;

    virtual ~BaseInvalidDepthAnalysisEngine() = default;
};
```

---

# 10.15 C++ — `InvalidDepthAnalysisEngine`

Kế thừa `BaseInvalidDepthAnalysisEngine`.

## Nhiệm vụ
- đếm valid / invalid pixels
- optional row invalid statistics

---

# 10.16 C++ — `BaseRegionDepthAnalysisEngine`

Tạo abstract class:

```cpp
class BaseRegionDepthAnalysisEngine
{
public:
    virtual std::vector<RegionDepthStatistic> analyze_regions(
        const std::string& pair_id,
        const cv::Mat& depth_map,
        const DepthQualityThresholdConfig& config,
        double invalid_marker
    ) const = 0;

    virtual ~BaseRegionDepthAnalysisEngine() = default;
};
```

---

# 10.17 C++ — `RegionDepthAnalysisEngine`

Kế thừa `BaseRegionDepthAnalysisEngine`.

## Nhiệm vụ
- chia depth thành NEAR / MID / FAR
- tính thống kê từng vùng

---

# 10.18 C++ — `BaseROIQualityAnalysisEngine`

Tạo abstract class:

```cpp
class BaseROIQualityAnalysisEngine
{
public:
    virtual ROIQualityStatistic analyze_roi(
        const std::string& pair_id,
        const cv::Mat& depth_map,
        const ROIConfig& roi,
        double invalid_marker
    ) const = 0;

    virtual ~BaseROIQualityAnalysisEngine() = default;
};
```

---

# 10.19 C++ — `ROIQualityAnalysisEngine`

Kế thừa `BaseROIQualityAnalysisEngine`.

## Nhiệm vụ
- phân tích center ROI
- tính valid ratio / average depth / min depth

---

# 10.20 C++ — `BaseDepthQualityScoringEngine`

Tạo abstract class:

```cpp
class BaseDepthQualityScoringEngine
{
public:
    virtual DepthQualityResult score(
        const DepthMapSample& sample,
        int valid_count,
        int invalid_count,
        const ROIQualityStatistic& roi_stat,
        const DepthQualityThresholdConfig& config
    ) const = 0;

    virtual ~BaseDepthQualityScoringEngine() = default;
};
```

---

# 10.21 C++ — `DepthQualityScoringEngine`

Kế thừa `BaseDepthQualityScoringEngine`.

## Nhiệm vụ
- tính quality score
- gán GOOD / USABLE / POOR

---

# 10.22 C++ — `DepthMapQualityAnalyzer`

Tạo class trung tâm:

```cpp
class DepthMapQualityAnalyzer
```

## Thuộc tính

```cpp
private:
    std::vector<DepthMapSample> samples;
    DepthQualityThresholdConfig threshold_config;
    ROIConfig roi_config;

    std::shared_ptr<BaseInvalidDepthAnalysisEngine> invalid_engine;
    std::shared_ptr<BaseRegionDepthAnalysisEngine> region_engine;
    std::shared_ptr<BaseROIQualityAnalysisEngine> roi_engine;
    std::shared_ptr<BaseDepthQualityScoringEngine> scoring_engine;

    std::vector<DepthQualityResult> quality_results;
    std::vector<ROIQualityStatistic> roi_statistics;
    std::vector<RegionDepthStatistic> region_statistics;
    std::stack<DepthQualityDebugRecord> debug_history;
```

## Hàm cần có

### `load_depth_sample_config(const std::string& path)`
### `load_threshold_config(const std::string& path)`
### `load_roi_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each depth sample:
    load depth map
    analyze valid / invalid
    analyze near / mid / far
    analyze ROI
    score quality
    save results
    push debug history
```

### `const std::vector<DepthQualityResult>& get_quality_results() const`
### `const std::vector<ROIQualityStatistic>& get_roi_statistics() const`
### `const std::vector<RegionDepthStatistic>& get_region_statistics() const`
### `std::vector<DepthQualityDebugRecord> get_debug_history_reverse()`

---

# 10.23 C++ — `DepthQualityReportWriter`

Tạo class:

```cpp
class DepthQualityReportWriter
```

## Hàm cần có

### `write_depth_quality_report(...)`
Ví dụ:

```text
[Depth Quality]
Pair: hallway_01
Algorithm: BM
Invalid Ratio: 0.12
Center Invalid Ratio: 0.18
Quality Score: 78
Label: USABLE
```

### `write_invalid_depth_report(...)`
Ví dụ:

```text
[Invalid Depth]
Pair: hallway_01
Valid Pixels: 14200
Invalid Pixels: 2210
```

### `write_roi_quality_report(...)`
Ví dụ:

```text
[ROI Quality]
Pair: hallway_01
Center ROI Valid Ratio: 0.86
Average Depth: 2.12
Min Depth: 0.74
```

### `write_region_depth_report(...)`
Ví dụ:

```text
[Region Depth]
Pair: hallway_01
Region: NEAR
Valid Count: 3120
Average Depth: 0.82
```

### `write_reverse_debug_history(...)`

---

# 10.24 C++ — `main.cpp`

## Yêu cầu

```text
Load depth sample config
Load quality threshold config
Load ROI config
Load visualization config
Load runtime config

Create:
    InvalidDepthAnalysisEngine
    RegionDepthAnalysisEngine
    ROIQualityAnalysisEngine
    DepthQualityScoringEngine

Create DepthMapQualityAnalyzer
Run
Write reports
```

---

# 11. Luật depth quality analysis của project

## Luật 1 — Không được đánh giá quality chỉ bằng 1 con số average depth
Depth quality phải xét ít nhất:
- invalid ratio
- center ROI
- region distribution

## Luật 2 — Center ROI phải được ưu tiên
Vì trong perception robot, vùng trung tâm phía trước thường quan trọng nhất.

## Luật 3 — Depth invalid phải được tách rõ khỏi valid depth
Không được trộn invalid vào average depth.

## Luật 4 — Visualization là bắt buộc
Vì Đợt 11 đã thêm Matplotlib, bài này phải có ít nhất 1 plot quality.

## Luật 5 — Report phải trả lời được “depth map có usable hay không”
Không chỉ ghi số liệu, mà phải có quality label / score.

---

# 12. Output mong muốn

## Config
```text
config/depth_sample_config.txt
config/quality_threshold_config.txt
config/roi_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/depth_quality_report.txt
assets/outputs/invalid_depth_report.txt
assets/outputs/roi_quality_report.txt
assets/outputs/region_depth_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/invalid_ratio_plot.png
assets/outputs/region_distribution_plot.png
assets/outputs/roi_quality_plot.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho depth samples / ROI / thresholds
- preview invalid masks và region stats bằng NumPy
- visualize quality bằng Matplotlib

## C++
- chạy quality analyzer cho depth map
- chấm mức usable của depth output
- chuẩn bị depth map đáng tin để đi sang point cloud

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là lớp **“QA cho depth perception”**:

```text
Stereo pair
→ disparity
→ depth
→ depth quality check
→ point cloud
→ robot-frame geometry
→ obstacle reasoning
```

Nếu depth map chất lượng kém mà bạn vẫn back-project sang point cloud, mọi thứ phía sau sẽ nhiễu theo.

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho depth samples / thresholds / ROI / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / quality ids
- [ ] Python có NumPy depth quality preview
- [ ] Python có Matplotlib quality plots
- [ ] C++ có `DepthMapSample`
- [ ] C++ có `DepthQualityThresholdConfig`
- [ ] C++ có `ROIConfig`
- [ ] C++ có `RegionDepthStatistic`
- [ ] C++ có `ROIQualityStatistic`
- [ ] C++ có `DepthQualityResult`
- [ ] C++ có `InvalidDepthAnalysisEngine`
- [ ] C++ có `RegionDepthAnalysisEngine`
- [ ] C++ có `ROIQualityAnalysisEngine`
- [ ] C++ có `DepthQualityScoringEngine`
- [ ] C++ có `DepthMapQualityAnalyzer`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 53**, bài tiếp theo hợp lý nhất là:

# **Bài 54: Stereo Point Cloud Reconstruction Lab**

Ý tưởng:
```text
depth map
→ back-projection
→ point cloud generation
→ point filtering
→ point statistics
→ point cloud quality report
```

Tức là mạch Đợt 11 sẽ đi trọn như sau:

```text
Bài 51: Stereo disparity
→ Bài 52: Depth estimation
→ Bài 53: Depth quality analysis
→ Bài 54: Point cloud reconstruction
```
