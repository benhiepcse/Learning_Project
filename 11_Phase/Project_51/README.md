# 🤖 Bài 51: Stereo Disparity Workbench — Bộ workbench phân tích disparity cho Humanoid Robot AI Perception

> Mini Project số 51 trong **Đợt 11**  
> **Bài 51 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10 + 11**.
>
> Mình hiểu câu “cho tôi bài 46” ở đây theo đúng mạch project hiện tại là **bài tiếp theo sau Bài 50**, tức **Bài 51**.  
> Vì bạn đang đi chuỗi:
>
> ```text
> Bài 46 → Bài 47 → Bài 48 → Bài 49 → Bài 50
> ```
>
> nên sau khi đọc **Đợt 11**, bài kế tiếp hợp logic sẽ là **Bài 51**.

---

# 📌 Mục lục

- [1. Đợt 11 có gì mới](#1-đợt-11-có-gì-mới)
- [2. Bài 51 nên đi theo hướng nào](#2-bài-51-nên-đi-theo-hướng-nào)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Pipeline tổng thể](#5-pipeline-tổng-thể)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Luật disparity workbench của project](#10-luật-disparity-workbench-của-project)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý bước tiếp theo](#14-gợi-ý-bước-tiếp-theo)

---

# 1. Đợt 11 có gì mới

## Chủ đề của Đợt 11
# **Stereo Disparity**

Trong roadmap bạn gửi, **Đợt 11 (Ngày 21–22)** có 3 phần chính:

## Python
### **Phase 9 — Python Libraries**
- `Python Matplotlib`

## C++
### Ôn + áp dụng
- OOP
- vector
- file structure
- enum

## Computer Vision
### **Phase 5 — Stereo Vision & Depth**
- `Block Matching`
- `SGM`

---

# 2. Bài 51 nên đi theo hướng nào

## Vì sao không nên đi tiếp free-space ngay?
Ở cuối Đợt 10 bạn vừa đi đến:

```text
Stereo → Disparity → Depth → Point Cloud → Robot Frame → Obstacle Reasoning
```

Nhưng **Đợt 11** lại quay về **một lõi rất quan trọng hơn ở giữa pipeline**:

```text
Stereo pair
→ disparity computation
→ block matching / SGM
→ disparity analysis
```

Tức là Đợt 11 không còn đứng ở tầng “robot reasoning” nữa, mà quay về **nâng chất lượng lõi stereo**.

## Vì vậy Bài 51 hợp lý nhất là:
# **Stereo Disparity Workbench**

Nó sẽ là một “bàn thí nghiệm disparity” để bạn:
- nạp stereo left/right pair
- chạy **Block Matching**
- chạy **SGM**
- so sánh disparity outputs
- thống kê disparity quality
- vẽ histogram / profile bằng **Matplotlib**
- xuất report để chuẩn bị cho **depth estimation** của Đợt 12

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Disparity Workbench**

System này nhận đầu vào là:
- một hoặc nhiều **stereo image pairs**
- một hoặc nhiều **disparity algorithm configs**
- một số **evaluation / visualization options**

## Mỗi stereo pair gồm:
- ảnh trái
- ảnh phải
- tên pair
- optional label / scene tag

Ví dụ:
```text
pair_01:
  left  = left_01.png
  right = right_01.png
  label = hallway_scene
```

## Các disparity algorithm tối thiểu phải có
### 1. **Block Matching**
- window size
- min disparity
- num disparities

### 2. **SGM / Semi-Global Matching**
Ở mức project mini, bạn có thể:
- dùng OpenCV StereoSGBM
- hoặc mô phỏng config SGM-style nếu chưa muốn đi quá sâu vào implementation nội bộ

## Nhiệm vụ của system
### Với mỗi stereo pair:
1. load ảnh trái / phải
2. optional preprocess:
   - grayscale
   - blur nhẹ
3. chạy **Block Matching**
4. chạy **SGM**
5. tạo disparity map cho từng thuật toán
6. tính disparity statistics
7. vẽ histogram / line profile / summary bằng Matplotlib
8. ghi report so sánh BM vs SGM

<p align="center">
  <img src="../../images/project_51.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build stereo pair config
→ build disparity algorithm config
→ build visualization config
→ preview disparity statistics bằng NumPy + Matplotlib

C++
→ load stereo pairs
→ run Block Matching / SGM
→ produce disparity maps
→ evaluate disparity outputs
→ export disparity reports
```

Mục tiêu cốt lõi:
- hiểu **disparity map được sinh ra như thế nào**
- hiểu **Block Matching và SGM khác nhau ở đâu**
- biết cách **so sánh 2 disparity outputs**
- chuẩn bị nền trực tiếp cho **Đợt 12: Depth Estimation / Depth Map / Point Cloud**

---

# 5. Pipeline tổng thể

```text
Load Stereo Pair Config
Load Disparity Algorithm Config
Load Visualization Config
Load Runtime Config

Create StereoPairLoader
Create DisparityPreprocessEngine
Create BlockMatchingEngine
Create SGMEngine
Create DisparityStatisticsEngine
Create StereoDisparityWorkbench

For each stereo pair:
    1. load left / right images
    2. preprocess if needed
    3. run block matching
    4. run SGM
    5. normalize / store disparity maps
    6. compute disparity statistics
    7. compute BM vs SGM comparison
    8. save results

After all pairs:
    write reports
    write plots
```

---

# 6. Kiến thức cần

## Python
- OOP
- module
- file handling
- NumPy
- **Matplotlib**
- dict / list / BST / Graph từ các đợt trước

## C++
- class / inheritance / virtual function
- vector
- enum
- file structure
- OpenCV image I/O
- OpenCV stereo matcher API

## Computer Vision
- stereo pair
- rectified images
- disparity
- block matching
- SGM / StereoSGBM
- disparity map visualization

---

# 7. DSA + Algorithm bắt buộc

# 7.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **stereo disparity pipeline**:

```text
StereoPair
→ Preprocess
→ BlockMatching
→ SGM
→ DisparityMap
→ Statistics
→ Visualization
→ Report
```

## 2. Python BST
Lưu `pair_id`, `config_id`, `scene_id`.

## 3. `std::vector<StereoPairSample>`
Danh sách stereo pairs.

## 4. `std::vector<DisparityAlgorithmConfig>`
Danh sách cấu hình thuật toán disparity.

## 5. `std::vector<DisparityMapResult>`
Kết quả disparity theo pair và theo thuật toán.

## 6. `std::vector<DisparityComparisonResult>`
Kết quả so sánh BM vs SGM.

## 7. `std::stack<DisparityDebugRecord>`
Reverse debug history.

---

# 7.2 Algorithm bắt buộc

## Algorithm 1 — Block Matching disparity
Bạn phải có một disparity engine theo hướng Block Matching:
- block window
- min disparity
- num disparities

Có thể dùng:
- `cv::StereoBM`
- hoặc implementation đơn giản kiểu SAD window nếu muốn làm tay ở mức mini

---

## Algorithm 2 — SGM / SGBM disparity
Bạn phải có engine thứ hai theo hướng:
- `StereoSGBM`
- hoặc SGM-style configuration wrapper

Mục tiêu là có **ít nhất 2 disparity outputs** để so sánh.

---

## Algorithm 3 — Disparity statistics
Tính tối thiểu cho mỗi disparity map:
- `valid_pixel_count`
- `invalid_pixel_count`
- `min_disparity`
- `max_disparity`
- `average_disparity`

---

## Algorithm 4 — BM vs SGM comparison
So sánh ít nhất:
- chênh lệch average disparity
- valid pixel ratio
- disparity range
- optional absolute difference map

---

## Algorithm 5 — Visualization
Bắt buộc có **ít nhất 1 dạng visualize bằng Matplotlib**, ví dụ:
- disparity histogram
- disparity line profile
- valid/invalid ratio bar chart
- BM vs SGM average disparity comparison

---

# 8. Cấu trúc folder

```text
mini_project_51_stereo_disparity_workbench/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  ├─ stereo_pairs/
│  │  ├─ left_01.png
│  │  ├─ right_01.png
│  │  ├─ left_02.png
│  │  └─ right_02.png
│  │
│  └─ outputs/
│     ├─ disparity_statistics_report.txt
│     ├─ disparity_comparison_report.txt
│     ├─ disparity_runtime_report.txt
│     ├─ reverse_debug_history.txt
│     ├─ disparity_histogram_pair_01.png
│     ├─ disparity_profile_pair_01.png
│     └─ bm_vs_sgm_comparison.png
│
├─ config/
│  ├─ stereo_pair_config.txt
│  ├─ disparity_algorithm_config.txt
│  ├─ visualization_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ disparity_graph_preview.py
│     ├─ disparity_bst.py
│     ├─ numpy_disparity_preview.py
│     ├─ matplotlib_disparity_plotter.py
│     └─ synthetic_stereo_pair_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoPairSample.hpp
   │  ├─ DisparityAlgorithmConfig.hpp
   │  ├─ DisparityMapResult.hpp
   │  ├─ DisparityComparisonResult.hpp
   │  ├─ DisparityDebugRecord.hpp
   │  ├─ BaseStereoMatcher.hpp
   │  ├─ BlockMatchingStereoMatcher.hpp
   │  ├─ SGMStereoMatcher.hpp
   │  ├─ BaseDisparityStatisticsEngine.hpp
   │  ├─ DisparityStatisticsEngine.hpp
   │  ├─ BaseDisparityComparisonEngine.hpp
   │  ├─ DisparityComparisonEngine.hpp
   │  ├─ StereoDisparityWorkbench.hpp
   │  └─ StereoDisparityReportWriter.hpp
   │
   └─ src/
      ├─ BlockMatchingStereoMatcher.cpp
      ├─ SGMStereoMatcher.cpp
      ├─ DisparityStatisticsEngine.cpp
      ├─ DisparityComparisonEngine.cpp
      ├─ StereoDisparityWorkbench.cpp
      └─ StereoDisparityReportWriter.cpp
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
pair_config_path
algorithm_config_path
visualization_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 9.2 Python — `StereoDisparityWorkbenchConfigBuilder`

Tạo class con:

```python
class StereoDisparityWorkbenchConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_stereo_pair(
    pair_id,
    left_image_path,
    right_image_path,
    scene_label=None
)`

### `add_block_matching_config(
    config_id,
    min_disparity,
    num_disparities,
    block_size
)`

### `add_sgm_config(
    config_id,
    min_disparity,
    num_disparities,
    block_size,
    p1,
    p2
)`

### `set_visualization_options(
    enable_histogram,
    enable_line_profile,
    enable_bm_vs_sgm_plot
)`

### `write_pair_config()`
### `write_algorithm_config()`
### `write_visualization_config()`
### `write_runtime_config()`

---

# 9.3 Python — `disparity_graph_preview.py`

Tạo class:

```python
class DisparityGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
StereoPair
→ Preprocess
→ BlockMatching
→ SGM
→ DisparityStatistics
→ Visualization
→ Report
```

---

# 9.4 Python — `disparity_bst.py`

Tạo BST cho `pair_id` / `config_id`.

---

# 9.5 Python — `numpy_disparity_preview.py`

Tạo class:

```python
class NumPyDisparityPreview:
```

## Hàm cần có

### `normalize_disparity(disparity_map)`
### `compute_disparity_statistics(disparity_map, invalid_threshold=0)`
### `compute_difference_map(disparity_a, disparity_b)`
### `extract_row_profile(disparity_map, row_index)`

---

# 9.6 Python — `matplotlib_disparity_plotter.py`

Tạo class:

```python
class MatplotlibDisparityPlotter:
```

## Hàm cần có

### `plot_histogram(disparity_map, save_path)`
### `plot_row_profile(disparity_map, row_index, save_path)`
### `plot_bm_vs_sgm_summary(bm_stats, sgm_stats, save_path)`

---

# 9.7 C++ — `StereoPairSample`

```cpp
struct StereoPairSample
{
    std::string pair_id;
    std::string left_image_path;
    std::string right_image_path;
    std::string scene_label;
};
```

---

# 9.8 C++ — `DisparityAlgorithmConfig`

```cpp
enum class DisparityAlgorithmType
{
    BLOCK_MATCHING,
    SGM
};

struct DisparityAlgorithmConfig
{
    std::string config_id;
    DisparityAlgorithmType algorithm_type;

    int min_disparity;
    int num_disparities;
    int block_size;

    int p1 = 0;
    int p2 = 0;
};
```

---

# 9.9 C++ — `DisparityMapResult`

```cpp
struct DisparityMapResult
{
    std::string pair_id;
    std::string config_id;
    DisparityAlgorithmType algorithm_type;

    cv::Mat disparity_map;

    int valid_pixel_count;
    int invalid_pixel_count;
    double min_disparity;
    double max_disparity;
    double average_disparity;
};
```

---

# 9.10 C++ — `DisparityComparisonResult`

```cpp
struct DisparityComparisonResult
{
    std::string pair_id;

    double bm_average_disparity;
    double sgm_average_disparity;

    int bm_valid_pixels;
    int sgm_valid_pixels;

    double average_absolute_difference;
};
```

---

# 9.11 C++ — `DisparityDebugRecord`

```cpp
struct DisparityDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.12 C++ — `BaseStereoMatcher`

Tạo abstract class:

```cpp
class BaseStereoMatcher
{
public:
    virtual DisparityMapResult compute(
        const StereoPairSample& pair,
        const DisparityAlgorithmConfig& config
    ) const = 0;

    virtual ~BaseStereoMatcher() = default;
};
```

---

# 9.13 C++ — `BlockMatchingStereoMatcher`

Kế thừa `BaseStereoMatcher`.

## Nhiệm vụ
- load stereo pair
- convert grayscale nếu cần
- chạy `cv::StereoBM` hoặc block matching mini
- trả về `DisparityMapResult`

---

# 9.14 C++ — `SGMStereoMatcher`

Kế thừa `BaseStereoMatcher`.

## Nhiệm vụ
- load stereo pair
- convert grayscale nếu cần
- chạy `cv::StereoSGBM`
- trả về `DisparityMapResult`

---

# 9.15 C++ — `BaseDisparityStatisticsEngine`

Tạo abstract class:

```cpp
class BaseDisparityStatisticsEngine
{
public:
    virtual void compute_statistics(DisparityMapResult& result) const = 0;
    virtual ~BaseDisparityStatisticsEngine() = default;
};
```

---

# 9.16 C++ — `DisparityStatisticsEngine`

Kế thừa `BaseDisparityStatisticsEngine`.

## Nhiệm vụ
- đếm valid / invalid pixels
- tính min / max / average disparity

---

# 9.17 C++ — `BaseDisparityComparisonEngine`

Tạo abstract class:

```cpp
class BaseDisparityComparisonEngine
{
public:
    virtual DisparityComparisonResult compare(
        const DisparityMapResult& bm_result,
        const DisparityMapResult& sgm_result
    ) const = 0;

    virtual ~BaseDisparityComparisonEngine() = default;
};
```

---

# 9.18 C++ — `DisparityComparisonEngine`

Kế thừa `BaseDisparityComparisonEngine`.

## Nhiệm vụ
- so sánh BM vs SGM
- tính average disparity difference
- tính valid pixel difference

---

# 9.19 C++ — `StereoDisparityWorkbench`

Tạo class trung tâm:

```cpp
class StereoDisparityWorkbench
```

## Thuộc tính

```cpp
private:
    std::vector<StereoPairSample> pairs;
    std::vector<DisparityAlgorithmConfig> configs;

    std::shared_ptr<BaseStereoMatcher> bm_matcher;
    std::shared_ptr<BaseStereoMatcher> sgm_matcher;
    std::shared_ptr<BaseDisparityStatisticsEngine> statistics_engine;
    std::shared_ptr<BaseDisparityComparisonEngine> comparison_engine;

    std::vector<DisparityMapResult> results;
    std::vector<DisparityComparisonResult> comparisons;
    std::stack<DisparityDebugRecord> debug_history;
```

## Hàm cần có

### `load_pair_config(const std::string& path)`
### `load_algorithm_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each stereo pair:
    run BM config
    compute BM statistics

    run SGM config
    compute SGM statistics

    compare BM vs SGM

    save results
    save comparison
    push debug history
```

### `const std::vector<DisparityMapResult>& get_results() const`
### `const std::vector<DisparityComparisonResult>& get_comparisons() const`
### `std::vector<DisparityDebugRecord> get_debug_history_reverse()`

---

# 9.20 C++ — `StereoDisparityReportWriter`

Tạo class:

```cpp
class StereoDisparityReportWriter
```

## Hàm cần có

### `write_disparity_statistics_report(...)`
Ví dụ:

```text
[Disparity Statistics]
Pair: hallway_01
Algorithm: BLOCK_MATCHING
Valid Pixels: 15230
Invalid Pixels: 1180
Average Disparity: 12.48
```

### `write_disparity_comparison_report(...)`
Ví dụ:

```text
[Disparity Comparison]
Pair: hallway_01
BM Average Disparity: 12.48
SGM Average Disparity: 13.06
Average Absolute Difference: 1.42
```

### `write_disparity_runtime_report(...)`
Ví dụ:

```text
[Runtime Summary]
Pair: hallway_01
Executed Algorithms: BM, SGM
```

### `write_reverse_debug_history(...)`

---

# 9.21 C++ — `main.cpp`

## Yêu cầu

```text
Load stereo pair config
Load disparity algorithm config
Load visualization config
Load runtime config

Create:
    BlockMatchingStereoMatcher
    SGMStereoMatcher
    DisparityStatisticsEngine
    DisparityComparisonEngine

Create StereoDisparityWorkbench
Run
Write reports
```

---

# 10. Luật disparity workbench của project

## Luật 1 — Chỉ so sánh BM và SGM trên cùng một stereo pair
Không được lấy disparity của pair A so với pair B.

## Luật 2 — Stereo pair phải được hiểu là ảnh đã cùng scene
Tức là ảnh trái và ảnh phải phải là một cặp tương ứng.

## Luật 3 — Mọi disparity map đều phải đi qua bước statistics
Không được chỉ “tạo ảnh disparity” rồi kết thúc.

## Luật 4 — Matplotlib là bắt buộc ở bài này
Vì Đợt 11 đã mở thêm **Python Matplotlib**, nên project phải dùng nó cho visualization.

## Luật 5 — Report phải chỉ ra BM vs SGM khác nhau ở đâu
Ít nhất qua:
- valid pixels
- average disparity
- disparity range
- hoặc histogram / profile

---

# 11. Output mong muốn

## Config
```text
config/stereo_pair_config.txt
config/disparity_algorithm_config.txt
config/visualization_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/disparity_statistics_report.txt
assets/outputs/disparity_comparison_report.txt
assets/outputs/disparity_runtime_report.txt
assets/outputs/reverse_debug_history.txt
```

## Plots
```text
assets/outputs/disparity_histogram_pair_01.png
assets/outputs/disparity_profile_pair_01.png
assets/outputs/bm_vs_sgm_comparison.png
```

---

# 12. Vai trò trong Humanoid Robot

## Python
- build stereo pair config
- thống kê disparity bằng NumPy
- vẽ histogram / profile bằng Matplotlib
- hỗ trợ debug quality của disparity

## C++
- chạy runtime disparity thật sự
- tạo disparity maps từ stereo pair
- so sánh BM vs SGM
- chuẩn bị đầu vào cho depth estimation

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó chạm đúng “lõi stereo”:
- disparity map chất lượng tốt → depth map tốt hơn
- disparity map tệ → depth, point cloud, robot reasoning phía sau đều tệ

Bài 51 vì vậy là **cầu nối trực tiếp** sang:
- **Bài 52: Stereo Depth Estimation Workbench**
- **Bài 53: Depth Map Quality Analyzer**
- **Bài 54: Stereo Point Cloud Builder v2**

---

# 13. Checklist hoàn thành

- [ ] Python build đủ config cho stereo pair / disparity algorithms / visualization
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho pair ids / config ids
- [ ] Python có NumPy disparity preview
- [ ] Python có Matplotlib disparity plots
- [ ] C++ có `StereoPairSample`
- [ ] C++ có `DisparityAlgorithmConfig`
- [ ] C++ có `DisparityMapResult`
- [ ] C++ có `DisparityComparisonResult`
- [ ] C++ có `BlockMatchingStereoMatcher`
- [ ] C++ có `SGMStereoMatcher`
- [ ] C++ có `DisparityStatisticsEngine`
- [ ] C++ có `DisparityComparisonEngine`
- [ ] C++ có `StereoDisparityWorkbench`
- [ ] C++ ghi đủ report + plot outputs

---

# 14. Gợi ý bước tiếp theo

Sau **Bài 51**, bài hợp lý nhất để bám đúng **Đợt 12** là:

# **Bài 52: Stereo Depth Estimation Workbench**

Ý tưởng:
```text
disparity map
→ depth conversion
→ invalid depth handling
→ depth statistics
→ depth visualization
→ depth quality report
```

Tức là mạch sẽ đi rất thẳng:

```text
Bài 51: Stereo disparity
→ Bài 52: Depth estimation
→ Bài 53: Depth map quality analysis
→ Bài 54: Point cloud reconstruction
```
