# 🤖 Bài 52: Stereo Depth Estimation Workbench — Bộ workbench chuyển disparity thành depth map cho Humanoid Robot AI Perception

> Mini Project số 52 trong **Đợt 11**  
> **Bài 52 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11** và đi tiếp trực tiếp từ **Bài 51**.
>
> Nếu:
>
> - **Bài 51** là nơi bạn tạo và so sánh **disparity maps** bằng **Block Matching** và **SGM**
>
> thì **Bài 52** sẽ là bước tiếp theo cực kỳ tự nhiên:
>
> ```text
> disparity map
> → convert sang depth map
> → xử lý invalid depth
> → thống kê chất lượng depth
> → visualize depth bằng Python
> → ghi depth estimation report
> ```
>
> Đây là bước nối trực tiếp giữa:
>
> - **stereo disparity**
>
> và
>
> - **3D perception / point cloud / robot reasoning**
>
> vì depth map là cầu nối chính để robot chuyển từ “pixel disparity” sang “khoảng cách thật trong không gian”.

---

# 📌 Mục lục

- [1. Vì sao Bài 52 xuất hiện sau Bài 51](#1-vì-sao-bài-52-xuất-hiện-sau-bài-51)
- [2. Đợt 11 đang bổ sung gì cho pipeline](#2-đợt-11-đang-bổ-sung-gì-cho-pipeline)
- [3. Bài 52 nâng từ Bài 51 lên chỗ nào](#3-bài-52-nâng-từ-bài-51-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật depth estimation của project](#11-luật-depth-estimation-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý bước tiếp theo](#15-gợi-ý-bước-tiếp-theo)

---

# 1. Vì sao Bài 52 xuất hiện sau Bài 51

## Bài 51 đang làm gì?
Bài 51 giúp bạn có:

```text
stereo pair
→ Block Matching / SGM
→ disparity maps
→ disparity statistics
→ BM vs SGM comparison
```

Tức là bạn đã có **đầu ra disparity**.

## Nhưng còn thiếu gì?
Disparity chưa phải thứ robot dùng trực tiếp cho 3D reasoning.  
Robot cần một lớp chuyển đổi quan trọng hơn:

> **Từ disparity map, hãy tính ra depth map.**

Lúc đó bạn mới trả lời được:
- pixel này cách camera bao xa?
- vật thể ở vùng này gần hay xa?
- vùng nào bị invalid depth?
- depth output có usable cho point cloud hay chưa?

## Bài 52 lấp đúng chỗ đó
Bài 52 sẽ làm:

```text
disparity map
→ depth conversion
→ invalid depth handling
→ depth statistics
→ depth visualization
→ depth report
```

Đây là bước chuyển từ **stereo matching output** sang **distance representation**.

---

# 2. Đợt 11 đang bổ sung gì cho pipeline

Đợt 11 có trọng tâm:
- **Python Matplotlib**
- **C++ OOP / enum / vector**
- **Block Matching**
- **SGM**

Điều đó có nghĩa là sau khi disparity đã được tạo ở Bài 51, bước hợp lý tiếp theo trong cùng cụm kiến thức là:

1. **đọc disparity**
2. **chuyển sang depth**
3. **so sánh depth của BM vs SGM**
4. **vẽ histogram / profile / summary bằng Matplotlib**

Tức là Đợt 11 không chỉ dừng ở “có disparity map”, mà còn phải bắt đầu **phân tích depth sinh ra từ disparity map đó**.

---

# 3. Bài 52 nâng từ Bài 51 lên chỗ nào

## Bài 51
- stereo pair
- disparity maps
- BM vs SGM disparity comparison

## Bài 52
- disparity → depth
- invalid depth cleanup
- BM vs SGM depth comparison
- depth visualization
- depth quality report

### Nói ngắn gọn:
- **Bài 51** hỏi: “Disparity map trông như thế nào?”
- **Bài 52** hỏi: “Disparity đó tương ứng khoảng cách thật bao nhiêu mét?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Depth Estimation Workbench**

System này nhận đầu vào là:
- một hoặc nhiều **stereo rig configs**
- một hoặc nhiều **disparity map results** (từ BM / SGM)
- một bộ **depth conversion settings**
- một bộ **visualization settings**

## Mỗi rig phải chứa tối thiểu
- `fx`
- `fy`
- `cx`
- `cy`
- `baseline`
- image size

## Mỗi disparity map sample phải chứa
- `pair_id`
- `algorithm_type` (`BM` hoặc `SGM`)
- disparity matrix / disparity image path
- optional scene label

## Nhiệm vụ của system
### Với mỗi disparity map:
1. load disparity map
2. đọc stereo rig config
3. tính depth theo công thức:

```text
depth = fx * baseline / disparity
```

4. đánh dấu invalid depth khi:
   - disparity <= 0
   - disparity thiếu dữ liệu
5. optional cleanup:
   - replace invalid depth
   - clip max depth
6. tính depth statistics
7. vẽ histogram / profile / BM vs SGM depth comparison
8. ghi report

<p align="center">
  <img src="../../images/project_52.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build rig config
→ build disparity sample config
→ build depth conversion config
→ preview disparity-to-depth bằng NumPy
→ visualize depth bằng Matplotlib

C++
→ load stereo rig
→ load disparity results
→ convert disparity to depth
→ compute depth statistics
→ compare BM depth vs SGM depth
→ export depth reports
```

Mục tiêu cốt lõi:
- hiểu công thức **depth = fx * baseline / disparity**
- hiểu **invalid disparity → invalid depth**
- biết cách đánh giá **depth quality**
- chuẩn bị nền trực tiếp cho:
  - point cloud reconstruction
  - robot obstacle perception
  - camera-to-robot 3D transform

---

# 6. Pipeline tổng thể

```text
Load Stereo Rig Config
Load Disparity Result Config
Load Depth Conversion Config
Load Visualization Config
Load Runtime Config

Create DisparityLoader
Create DepthConversionEngine
Create DepthCleanupEngine
Create DepthStatisticsEngine
Create DepthComparisonEngine
Create StereoDepthEstimationWorkbench

For each disparity map sample:
    1. load disparity map
    2. convert disparity -> depth
    3. cleanup invalid depth if enabled
    4. compute depth statistics
    5. store result

For each stereo pair:
    compare BM depth vs SGM depth

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
- NumPy
- Matplotlib
- list / dict / BST / Graph

## C++
- class / inheritance / virtual function
- vector
- enum
- file structure
- OpenCV matrix handling
- report-oriented runtime design

## Computer Vision / Stereo
- disparity
- depth estimation
- stereo rig intrinsics / baseline
- invalid disparity handling
- depth map visualization

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **depth estimation pipeline**:

```text
DisparityMap
→ DepthConversion
→ InvalidDepthCleanup
→ DepthStatistics
→ Visualization
→ Report
```

## 2. Python BST
Lưu `pair_id`, `rig_id`, `depth_result_id`.

## 3. `std::vector<StereoRigConfig>`
Danh sách stereo rigs.

## 4. `std::vector<DisparityMapSample>`
Danh sách disparity samples.

## 5. `std::vector<DepthMapResult>`
Kết quả depth sau conversion.

## 6. `std::vector<DepthComparisonResult>`
Kết quả so sánh depth BM vs SGM.

## 7. `std::stack<DepthDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Disparity to depth
Với mỗi disparity hợp lệ:

```text
depth = fx * baseline / disparity
```

Trong đó:
- disparity càng lớn → vật càng gần
- disparity càng nhỏ → vật càng xa

---

## Algorithm 2 — Invalid depth handling
Nếu disparity:
- `<= 0`
- hoặc nằm trong vùng invalid

thì depth phải:
- được gắn cờ invalid
- hoặc thay bằng marker như `-1`

---

## Algorithm 3 — Depth statistics
Tính ít nhất:
- `valid_pixel_count`
- `invalid_pixel_count`
- `min_depth`
- `max_depth`
- `average_depth`

---

## Algorithm 4 — BM vs SGM depth comparison
So sánh ít nhất:
- average depth
- valid depth ratio
- min/max depth
- optional average absolute depth difference

---

## Algorithm 5 — Visualization
Bắt buộc có ít nhất 1 dạng visualization bằng **Matplotlib**, ví dụ:
- depth histogram
- depth line profile
- BM vs SGM average depth bar chart
- valid/invalid ratio chart

---

# 9. Cấu trúc folder

```text
mini_project_52_stereo_depth_estimation_workbench/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ disparity_inputs/
│  │  ├─ disparity_bm_pair_01.png
│  │  ├─ disparity_sgm_pair_01.png
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ depth_statistics_report.txt
│     ├─ depth_comparison_report.txt
│     ├─ depth_runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ depth_histogram_pair_01.png
│     ├─ depth_profile_pair_01.png
│     └─ bm_vs_sgm_depth_comparison.png
│
├─ config/
│  ├─ stereo_rig_config.txt
│  ├─ disparity_result_config.txt
│  ├─ depth_conversion_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ depth_graph_preview.py
│     ├─ depth_bst.py
│     ├─ numpy_depth_preview.py
│     ├─ matplotlib_depth_plotter.py
│     └─ synthetic_disparity_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoRigConfig.hpp
   │  ├─ DisparityMapSample.hpp
   │  ├─ DepthMapResult.hpp
   │  ├─ DepthComparisonResult.hpp
   │  ├─ DepthDebugRecord.hpp
   │  ├─ BaseDepthConversionEngine.hpp
   │  ├─ StereoDepthConversionEngine.hpp
   │  ├─ BaseDepthCleanupEngine.hpp
   │  ├─ DepthCleanupEngine.hpp
   │  ├─ BaseDepthStatisticsEngine.hpp
   │  ├─ DepthStatisticsEngine.hpp
   │  ├─ BaseDepthComparisonEngine.hpp
   │  ├─ DepthComparisonEngine.hpp
   │  ├─ StereoDepthEstimationWorkbench.hpp
   │  └─ StereoDepthReportWriter.hpp
   │
   └─ src/
      ├─ StereoDepthConversionEngine.cpp
      ├─ DepthCleanupEngine.cpp
      ├─ DepthStatisticsEngine.cpp
      ├─ DepthComparisonEngine.cpp
      ├─ StereoDepthEstimationWorkbench.cpp
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
disparity_config_path
depth_conversion_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoDepthEstimationConfigBuilder`

Tạo class con:

```python
class StereoDepthEstimationConfigBuilder(BaseConfigBuilder):
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
    pair_id,
    algorithm_type,
    disparity_path,
    scene_label=None
)`

### `set_depth_conversion_options(
    invalid_disparity_threshold,
    replace_invalid_depth_with,
    max_depth_clip
)`

### `set_visualization_options(
    enable_histogram,
    enable_line_profile,
    enable_bm_vs_sgm_plot
)`

### `write_rig_config()`
### `write_disparity_config()`
### `write_depth_conversion_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 10.3 Python — `depth_graph_preview.py`

Tạo class:

```python
class DepthGraphPreview:
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
→ InvalidCleanup
→ DepthStatistics
→ Visualization
→ Report
```

---

# 10.4 Python — `depth_bst.py`

Tạo BST cho `pair_id` / `rig_id`.

---

# 10.5 Python — `numpy_depth_preview.py`

Tạo class:

```python
class NumPyDepthPreview:
```

## Hàm cần có

### `disparity_to_depth(disparity_map, fx, baseline, invalid_threshold=0)`
### `cleanup_invalid_depth(depth_map, invalid_mask, replacement=-1.0)`
### `compute_depth_statistics(depth_map, invalid_marker=-1.0)`
### `compute_difference_map(depth_a, depth_b)`
### `extract_row_profile(depth_map, row_index)`

---

# 10.6 Python — `matplotlib_depth_plotter.py`

Tạo class:

```python
class MatplotlibDepthPlotter:
```

## Hàm cần có

### `plot_depth_histogram(depth_map, save_path)`
### `plot_depth_profile(depth_map, row_index, save_path)`
### `plot_bm_vs_sgm_depth_summary(bm_stats, sgm_stats, save_path)`

---

# 10.7 C++ — `StereoRigConfig`

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

# 10.8 C++ — `DisparityMapSample`

```cpp
enum class DisparityAlgorithmType
{
    BM,
    SGM
};

struct DisparityMapSample
{
    std::string pair_id;
    DisparityAlgorithmType algorithm_type;
    std::string disparity_path;
    std::string scene_label;
};
```

---

# 10.9 C++ — `DepthMapResult`

```cpp
struct DepthMapResult
{
    std::string pair_id;
    DisparityAlgorithmType algorithm_type;
    cv::Mat depth_map;

    int valid_pixel_count;
    int invalid_pixel_count;
    double min_depth;
    double max_depth;
    double average_depth;
};
```

---

# 10.10 C++ — `DepthComparisonResult`

```cpp
struct DepthComparisonResult
{
    std::string pair_id;

    double bm_average_depth;
    double sgm_average_depth;

    int bm_valid_pixels;
    int sgm_valid_pixels;

    double average_absolute_depth_difference;
};
```

---

# 10.11 C++ — `DepthDebugRecord`

```cpp
struct DepthDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.12 C++ — `BaseDepthConversionEngine`

Tạo abstract class:

```cpp
class BaseDepthConversionEngine
{
public:
    virtual DepthMapResult convert(
        const StereoRigConfig& rig,
        const DisparityMapSample& disparity_sample
    ) const = 0;

    virtual ~BaseDepthConversionEngine() = default;
};
```

---

# 10.13 C++ — `StereoDepthConversionEngine`

Kế thừa `BaseDepthConversionEngine`.

## Nhiệm vụ
- load disparity map
- chuyển disparity → depth
- tạo `DepthMapResult`

---

# 10.14 C++ — `BaseDepthCleanupEngine`

Tạo abstract class:

```cpp
class BaseDepthCleanupEngine
{
public:
    virtual void cleanup(
        DepthMapResult& result,
        double invalid_replacement,
        double max_depth_clip
    ) const = 0;

    virtual ~BaseDepthCleanupEngine() = default;
};
```

---

# 10.15 C++ — `DepthCleanupEngine`

Kế thừa `BaseDepthCleanupEngine`.

## Nhiệm vụ
- thay invalid depth bằng marker
- clip depth nếu vượt ngưỡng

---

# 10.16 C++ — `BaseDepthStatisticsEngine`

Tạo abstract class:

```cpp
class BaseDepthStatisticsEngine
{
public:
    virtual void compute_statistics(DepthMapResult& result) const = 0;
    virtual ~BaseDepthStatisticsEngine() = default;
};
```

---

# 10.17 C++ — `DepthStatisticsEngine`

Kế thừa `BaseDepthStatisticsEngine`.

## Nhiệm vụ
- tính valid / invalid pixels
- min / max / average depth

---

# 10.18 C++ — `BaseDepthComparisonEngine`

Tạo abstract class:

```cpp
class BaseDepthComparisonEngine
{
public:
    virtual DepthComparisonResult compare(
        const DepthMapResult& bm_result,
        const DepthMapResult& sgm_result
    ) const = 0;

    virtual ~BaseDepthComparisonEngine() = default;
};
```

---

# 10.19 C++ — `DepthComparisonEngine`

Kế thừa `BaseDepthComparisonEngine`.

## Nhiệm vụ
- so sánh depth BM vs SGM
- tính average depth difference
- valid depth difference

---

# 10.20 C++ — `StereoDepthEstimationWorkbench`

Tạo class trung tâm:

```cpp
class StereoDepthEstimationWorkbench
```

## Thuộc tính

```cpp
private:
    std::vector<StereoRigConfig> rigs;
    std::vector<DisparityMapSample> disparity_samples;

    std::shared_ptr<BaseDepthConversionEngine> conversion_engine;
    std::shared_ptr<BaseDepthCleanupEngine> cleanup_engine;
    std::shared_ptr<BaseDepthStatisticsEngine> statistics_engine;
    std::shared_ptr<BaseDepthComparisonEngine> comparison_engine;

    std::vector<DepthMapResult> results;
    std::vector<DepthComparisonResult> comparisons;
    std::stack<DepthDebugRecord> debug_history;
```

## Hàm cần có

### `load_rig_config(const std::string& path)`
### `load_disparity_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each disparity sample:
    find matching rig
    result = conversion_engine.convert(rig, disparity_sample)
    cleanup_engine.cleanup(result, invalid_replacement, max_depth_clip)
    statistics_engine.compute_statistics(result)
    save result
    push debug history

for each pair_id:
    compare BM depth vs SGM depth
```

### `const std::vector<DepthMapResult>& get_results() const`
### `const std::vector<DepthComparisonResult>& get_comparisons() const`
### `std::vector<DepthDebugRecord> get_debug_history_reverse()`

---

# 10.21 C++ — `StereoDepthReportWriter`

Tạo class:

```cpp
class StereoDepthReportWriter
```

## Hàm cần có

### `write_depth_statistics_report(...)`
Ví dụ:

```text
[Depth Statistics]
Pair: hallway_01
Algorithm: BM
Valid Pixels: 14200
Invalid Pixels: 2210
Average Depth: 2.34
```

### `write_depth_comparison_report(...)`
Ví dụ:

```text
[Depth Comparison]
Pair: hallway_01
BM Average Depth: 2.34
SGM Average Depth: 2.18
Average Absolute Depth Difference: 0.41
```

### `write_depth_runtime_report(...)`
Ví dụ:

```text
[Depth Runtime]
Pair: hallway_01
Depth conversion completed for BM and SGM
```

### `write_reverse_debug_history(...)`

---

# 10.22 C++ — `main.cpp`

## Yêu cầu

```text
Load stereo rig config
Load disparity result config
Load depth conversion config
Load visualization config
Load runtime config

Create:
    StereoDepthConversionEngine
    DepthCleanupEngine
    DepthStatisticsEngine
    DepthComparisonEngine

Create StereoDepthEstimationWorkbench
Run
Write reports
```

---

# 11. Luật depth estimation của project

## Luật 1 — Mọi disparity map phải đi qua bước depth conversion
Không được chỉ giữ disparity rồi bỏ qua phần distance.

## Luật 2 — Invalid disparity phải sinh invalid depth
Không được tính depth bằng cách chia cho 0 hoặc giữ giá trị rác.

## Luật 3 — BM và SGM chỉ được so sánh trên cùng pair
Không được so chéo giữa các pair khác nhau.

## Luật 4 — Matplotlib là bắt buộc
Vì Đợt 11 đã thêm **Matplotlib**, nên bài này phải có ít nhất 1 visualization depth.

## Luật 5 — Depth report phải phản ánh chất lượng thật của output
Ít nhất qua:
- valid pixels
- invalid pixels
- min/max depth
- average depth
- BM vs SGM comparison

---

# 12. Output mong muốn

## Config
```text
config/stereo_rig_config.txt
config/disparity_result_config.txt
config/depth_conversion_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/depth_statistics_report.txt
assets/outputs/depth_comparison_report.txt
assets/outputs/depth_runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/depth_histogram_pair_01.png
assets/outputs/depth_profile_pair_01.png
assets/outputs/bm_vs_sgm_depth_comparison.png
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build rig + disparity config
- preview disparity-to-depth bằng NumPy
- vẽ depth histogram / profile bằng Matplotlib
- hỗ trợ debug depth quality

## C++
- convert disparity thành depth
- so sánh depth output giữa BM và SGM
- tạo depth report cho các bước 3D phía sau

## Computer Vision / Robot Perception
Đây là project cực quan trọng vì nó là **cầu nối trực tiếp từ stereo sang 3D**:

```text
Stereo pair
→ disparity
→ depth
→ point cloud
→ robot-frame geometry
→ obstacle reasoning
```

Nếu depth ở bước này không ổn, toàn bộ point cloud và obstacle reasoning phía sau sẽ yếu.

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho rig / disparity / depth conversion / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / rig ids
- [ ] Python có NumPy disparity-to-depth preview
- [ ] Python có Matplotlib depth plots
- [ ] C++ có `StereoRigConfig`
- [ ] C++ có `DisparityMapSample`
- [ ] C++ có `DepthMapResult`
- [ ] C++ có `DepthComparisonResult`
- [ ] C++ có `StereoDepthConversionEngine`
- [ ] C++ có `DepthCleanupEngine`
- [ ] C++ có `DepthStatisticsEngine`
- [ ] C++ có `DepthComparisonEngine`
- [ ] C++ có `StereoDepthEstimationWorkbench`
- [ ] C++ ghi đủ report + plot outputs

---

# 15. Gợi ý bước tiếp theo

Sau **Bài 52**, bài hợp lý tiếp theo là:

# **Bài 53: Depth Map Quality Analyzer**

Ý tưởng:
```text
depth map
→ invalid hole analysis
→ near / far depth region analysis
→ row / region statistics
→ quality scoring
→ depth reliability report
```

Tức là mạch sẽ đi:

```text
Bài 51: Stereo disparity
→ Bài 52: Depth estimation
→ Bài 53: Depth map quality analysis
→ Bài 54: Point cloud reconstruction
```
