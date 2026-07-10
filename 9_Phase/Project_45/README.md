# 🤖 Bài 45: Stereo Block Matching Mini Runtime — Bộ runtime block matching mini để sinh disparity strip cho Humanoid Robot AI Perception

> Mini Project số 45 trong **Đợt 9**  
> **Bài 45 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9** và đi tiếp trực tiếp từ **Bài 44**.
>
> Nếu:
>
> - **Bài 41** tập trung vào **camera calibration + back-projection**
> - **Bài 42** mở rộng sang **multi-camera calibration graph**
> - **Bài 43** đi vào **stereo rectification + epipolar geometry**
> - **Bài 44** đi vào **correspondence cost cho một sample**
>
> thì **Bài 45** sẽ nâng tiếp lên thành:
>
> ```text
> rectified stereo row / stereo patch strip
> → trượt window qua nhiều pixel anchor
> → với mỗi anchor quét disparity candidates
> → chọn disparity tốt nhất
> → ghép lại thành disparity strip / mini disparity map
> ```
>
> Đây là bước rất quan trọng vì nó biến tư duy “một sample correspondence” thành **runtime block matching lặp trên nhiều vị trí ảnh** — đúng kiểu cầu nối từ kiến thức mini sang stereo depth pipeline thực tế.

---

# 📌 Mục lục

- [1. Vì sao Bài 45 xuất hiện sau Bài 44](#1-vì-sao-bài-45-xuất-hiện-sau-bài-44)
- [2. Đợt 9 đang học gì](#2-đợt-9-đang-học-gì)
- [3. Bài 45 nâng từ Bài 44 lên chỗ nào](#3-bài-45-nâng-từ-bài-44-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật block matching của project](#11-luật-block-matching-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 45 xuất hiện sau Bài 44

## Bài 44 đang làm gì?
Bài 44 đã giúp bạn giải bài toán:

```text
1 left anchor
→ nhiều disparity candidates
→ cost table
→ chọn best disparity
```

Tức là bạn đã biết cách matching **một vị trí ảnh**.

## Bài 45 nâng lên ở đâu?
Bài 45 trả lời câu hỏi tiếp theo:

> Nếu không phải chỉ một anchor, mà là **nhiều pixel trên một hàng ảnh hoặc một vùng ảnh nhỏ**, thì làm sao chạy block matching cho tất cả chúng?

Đó chính là bước chuyển từ:

```text
single-sample correspondence search
```

sang:

```text
multi-anchor stereo block matching runtime
```

Thay vì xử lý duy nhất `left_anchor = 20`, bạn sẽ xử lý:

```text
left_anchor = 10, 11, 12, 13, ... , 40
```

Với mỗi anchor:
1. lấy left window
2. quét disparity range
3. tính cost
4. chọn disparity tốt nhất

Sau đó ghép các disparity lại để tạo:
- **disparity strip** (1D)
- hoặc **mini disparity map** (nếu bạn mở rộng sang patch 2D)

---

# 2. Đợt 9 đang học gì

Theo roadmap của bạn, **Đợt 9** gồm:

## Python
### Phase 8 — Data Structures
- `Binary Search Tree`
- `Graph`
  - graph representation
  - BFS / DFS

### Phase 9 — Python Libraries
- `NumPy`
  - array
  - shape
  - indexing
  - math operations

## C++
### Phase 6 — OOP
- `Virtual Functions`
  - polymorphism
- `Enums`

## Computer Vision
### Phase 4 — Camera Geometry
- `Back Projection 2D → 3D`
- `Camera Calibration`
- `Undistortion`

## Vì sao Bài 45 vẫn hợp với Đợt 9?
Vì dù Bài 45 đã bắt đầu giống một mini stereo runtime, nó vẫn là phần nối tiếp rất tự nhiên của Đợt 9:
- đầu vào vẫn là **rectified stereo geometry**
- bạn vẫn cần **NumPy** để preview disparity strip
- bạn vẫn cần **Graph / BST** để tổ chức processing pipeline / sample ordering
- bạn vẫn dùng **C++ polymorphism** cho cost strategy / runtime components

---

# 3. Bài 45 nâng từ Bài 44 lên chỗ nào

## Bài 44
- xử lý **một stereo sample**
- một `left_anchor`
- nhiều disparity candidates
- chọn best disparity cho sample đó

## Bài 45
- xử lý **một dải anchor**
- mỗi anchor có **một disparity search riêng**
- kết quả cuối là **một dải disparity**

Nói cách khác:

### Bài 44
> “Disparity tốt nhất cho pixel này là gì?”

### Bài 45
> “Hãy làm điều đó cho cả một đoạn ảnh, rồi ghép thành disparity strip.”

Đây là bước cực kỳ quan trọng để tiến tới:
- block matching trên cả ảnh
- disparity map
- depth map

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Block Matching Mini Runtime**

System này nhận đầu vào là một tập **rectified stereo row signals** hoặc **synthetic stereo strips**.  
Mỗi sample mô tả một cặp hàng ảnh trái/phải, cùng với:
- khoảng anchor cần quét
- kích thước window
- disparity search range
- optional ground-truth disparity strip

## Nhiệm vụ của system
### Với từng sample strip:
1. duyệt qua các anchor pixel trong khoảng `[anchor_start, anchor_end]`
2. với mỗi anchor:
   - lấy left reference window
   - quét disparity từ `d_min` đến `d_max`
   - tính cost cho từng disparity
   - chọn disparity tốt nhất
3. lưu disparity tốt nhất cho từng anchor
4. ghép thành **disparity strip**
5. thống kê chất lượng matching của cả strip

<p align="center">
  <img src="images/project_45.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build stereo strip config
→ build disparity search config
→ preview disparity strip bằng NumPy
→ preview cost scan tại nhiều anchor

C++
→ load stereo strip samples
→ chạy block matching trên nhiều anchor
→ tạo disparity strip
→ tính thống kê strip quality
→ export reports
```

Mục tiêu cốt lõi:
- hiểu block matching không chỉ là “1 cost table”
- hiểu disparity map được hình thành từ **rất nhiều local matching**
- chuẩn bị nền cho:
  - mini disparity map
  - SAD/SSD runtime thực tế hơn
  - post-processing disparity

---

# 6. Pipeline tổng thể

```text
Load Stereo Strip Sample Config
Load Block Matching Runtime Config
Load Cost Strategy Config

Create StereoStripWindowExtractor
Create Cost Strategy
Create BlockMatchingRunner

For each stereo strip sample:
    For each anchor in [anchor_start, anchor_end]:
        1. extract left reference window

        2. for disparity d in [d_min, d_max]:
               compute right center = anchor - d
               extract right candidate window
               compute cost
               store candidate record

        3. choose best disparity for this anchor
        4. append disparity to strip result

    compute strip summary
    export reports
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- BST
- Graph + BFS / DFS
- NumPy
- config builder

## C++
- class / inheritance
- virtual functions / polymorphism
- enum class
- `std::vector`
- `std::unordered_map`
- `std::stack`
- `std::deque`
- mini stereo runtime design

## Computer Vision / Geometry
- rectified stereo pair
- disparity search
- local block matching
- disparity strip / mini disparity map
- SAD / SSD / NCC intuition

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **block matching pipeline**:

```text
StereoStrip
→ AnchorLoop
→ LeftWindow
→ CandidateDisparities
→ RightWindows
→ CostRecords
→ BestDisparityPerAnchor
→ DisparityStrip
```

## 2. Python BST
Lưu `sample_id`, `frame_id`, hoặc `strip_id`.

## 3. `std::vector<StereoStripSample>`
Danh sách strip samples.

## 4. `std::vector<AnchorDisparityResult>`
Kết quả disparity cho từng anchor.

## 5. `std::vector<StereoStripResult>`
Kết quả disparity strip cho từng sample.

## 6. `std::stack<BlockMatchingDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Anchor loop
Với mỗi sample:
- lặp anchor từ `anchor_start` đến `anchor_end`

---

## Algorithm 2 — Per-anchor disparity search
Cho từng anchor:
1. cắt left window
2. quét `d_min → d_max`
3. lấy right window tương ứng
4. tính cost
5. chọn disparity tốt nhất

---

## Algorithm 3 — Build disparity strip
Sau khi xử lý hết anchor:
- lưu các predicted disparity theo thứ tự anchor
- tạo `disparity_strip`

---

## Algorithm 4 — Strip quality summary
Tính ít nhất:
- số anchor match thành công
- disparity trung bình
- disparity min / max
- optional: sai số trung bình nếu có GT strip

---

## Algorithm 5 — Cost strategy abstraction
Bạn phải hỗ trợ ít nhất **2 cost strategy**:
- `SAD`
- `SSD`

NCC là phần mở rộng tốt nếu muốn.

---

# 9. Cấu trúc folder

```text
mini_project_45_stereo_block_matching_mini_runtime/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ disparity_strip_report.txt
│     ├─ anchor_matching_report.txt
│     ├─ strip_summary_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_strip_sample_config.txt
│  ├─ disparity_search_config.txt
│  ├─ cost_strategy_config.txt
│  └─ block_matching_runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ block_matching_graph_preview.py
│     ├─ stereo_strip_bst.py
│     ├─ numpy_disparity_strip_preview.py
│     └─ synthetic_strip_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoStripSample.hpp
   │  ├─ DisparityCostRecord.hpp
   │  ├─ AnchorDisparityResult.hpp
   │  ├─ StereoStripResult.hpp
   │  ├─ BlockMatchingDebugRecord.hpp
   │  ├─ BaseWindowExtractor.hpp
   │  ├─ StereoStripWindowExtractor.hpp
   │  ├─ BaseCostStrategy.hpp
   │  ├─ SADCostStrategy.hpp
   │  ├─ SSDCostStrategy.hpp
   │  ├─ NCCCostStrategy.hpp
   │  ├─ BlockMatchingRunner.hpp
   │  └─ BlockMatchingReportWriter.hpp
   │
   └─ src/
      ├─ StereoStripWindowExtractor.cpp
      ├─ SADCostStrategy.cpp
      ├─ SSDCostStrategy.cpp
      ├─ NCCCostStrategy.cpp
      ├─ BlockMatchingRunner.cpp
      └─ BlockMatchingReportWriter.cpp
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
sample_config_path
search_config_path
cost_strategy_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoBlockMatchingConfigBuilder`

Tạo class con:

```python
class StereoBlockMatchingConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_strip_sample(
    sample_id,
    left_row_values,
    right_row_values,
    anchor_start,
    anchor_end,
    window_radius,
    disparity_min,
    disparity_max,
    gt_disparity_strip=None
)`

### `set_cost_strategy(cost_name)`
- `SAD`
- `SSD`
- `NCC`

### `set_runtime_options(
    allow_partial_window,
    save_anchor_cost_details,
    compute_strip_summary
)`

### `write_sample_config()`
### `write_search_config()`
### `write_cost_strategy_config()`
### `write_runtime_config()`

---

# 10.3 Python — `block_matching_graph_preview.py`

Tạo class:

```python
class BlockMatchingGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
StereoStrip
→ AnchorLoop
→ LeftWindow
→ CandidateDisparities
→ RightWindowCandidates
→ CostRecords
→ BestDisparityPerAnchor
→ DisparityStrip
```

---

# 10.4 Python — `stereo_strip_bst.py`

Tạo BST cho `sample_id` / `strip_id`.

---

# 10.5 Python — `numpy_disparity_strip_preview.py`

Tạo class:

```python
class NumPyDisparityStripPreview:
```

## Hàm cần có

### `extract_window(row_values, center_index, radius)`
### `compute_sad(left_window, right_window)`
### `compute_ssd(left_window, right_window)`
### `scan_anchor_disparities(left_row, right_row, anchor, radius, d_min, d_max)`
### `build_disparity_strip(left_row, right_row, anchor_start, anchor_end, radius, d_min, d_max)`

---

# 10.6 C++ — `StereoStripSample`

```cpp
struct StereoStripSample
{
    std::string sample_id;

    std::vector<int> left_row_values;
    std::vector<int> right_row_values;

    int anchor_start;
    int anchor_end;

    int window_radius;

    int disparity_min;
    int disparity_max;

    bool has_gt_disparity_strip;
    std::vector<int> gt_disparity_strip;
};
```

---

# 10.7 C++ — `DisparityCostRecord`

```cpp
struct DisparityCostRecord
{
    int disparity;
    double cost;
    bool valid_window;
};
```

---

# 10.8 C++ — `AnchorDisparityResult`

```cpp
struct AnchorDisparityResult
{
    int anchor_index;
    int predicted_disparity;
    double best_cost;
    bool success;

    std::vector<DisparityCostRecord> cost_records;
};
```

---

# 10.9 C++ — `StereoStripResult`

```cpp
struct StereoStripResult
{
    std::string sample_id;

    std::vector<AnchorDisparityResult> anchor_results;
    std::vector<int> disparity_strip;

    int success_count;
    double average_disparity;
    int min_disparity;
    int max_disparity;
};
```

---

# 10.10 C++ — `BlockMatchingDebugRecord`

```cpp
struct BlockMatchingDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.11 C++ — `BaseWindowExtractor`

Tạo abstract class:

```cpp
class BaseWindowExtractor
{
public:
    virtual bool extract_window(
        const std::vector<int>& row_values,
        int center_index,
        int radius,
        std::vector<int>& output_window
    ) const = 0;

    virtual ~BaseWindowExtractor() = default;
};
```

---

# 10.12 C++ — `StereoStripWindowExtractor`

Kế thừa `BaseWindowExtractor`.

## Nhiệm vụ
- cắt window từ row signal
- xử lý rule partial / invalid window

---

# 10.13 C++ — `BaseCostStrategy`

Tạo abstract class:

```cpp
class BaseCostStrategy
{
public:
    virtual std::string name() const = 0;

    virtual double compute_cost(
        const std::vector<int>& left_window,
        const std::vector<int>& right_window
    ) const = 0;

    virtual ~BaseCostStrategy() = default;
};
```

---

# 10.14 C++ — `SADCostStrategy`

Tính:

```text
sum(|L_i - R_i|)
```

---

# 10.15 C++ — `SSDCostStrategy`

Tính:

```text
sum((L_i - R_i)^2)
```

---

# 10.16 C++ — `NCCCostStrategy` (khuyến khích)
Có thể thêm nếu muốn so sánh nhiều cost strategy.

---

# 10.17 C++ — `BlockMatchingRunner`

Tạo class trung tâm:

```cpp
class BlockMatchingRunner
```

## Thuộc tính

```cpp
private:
    std::vector<StereoStripSample> samples;

    std::shared_ptr<BaseWindowExtractor> window_extractor;
    std::shared_ptr<BaseCostStrategy> cost_strategy;

    std::vector<StereoStripResult> results;
    std::stack<BlockMatchingDebugRecord> debug_history;
```

## Hàm cần có

### `load_samples(const std::string& path)`

### `run()`
Pseudo:

```text
for each strip sample:
    create StereoStripResult

    for anchor from anchor_start to anchor_end:
        extract left window

        for d in [d_min, d_max]:
            right_center = anchor - d
            extract right window
            if valid:
                cost = cost_strategy.compute_cost(...)
            save cost record

        choose best disparity for anchor
        append AnchorDisparityResult
        append disparity to disparity_strip

    compute strip summary
    save result
    push debug history
```

### `const std::vector<StereoStripResult>& get_results() const`
### `std::vector<BlockMatchingDebugRecord> get_debug_history_reverse()`

---

# 10.18 C++ — `BlockMatchingReportWriter`

Tạo class:

```cpp
class BlockMatchingReportWriter
```

## Hàm cần có

### `write_disparity_strip_report(...)`
Ví dụ:

```text
[Disparity Strip]
Sample: strip_01
Disparities: 3 3 3 4 4 4 5 5
```

### `write_anchor_matching_report(...)`
Ví dụ:

```text
[Anchor Matching]
Sample: strip_01
Anchor 12 -> disparity=3 cost=7
Anchor 13 -> disparity=3 cost=8
Anchor 14 -> disparity=4 cost=6
```

### `write_strip_summary_report(...)`
Ví dụ:

```text
[Strip Summary]
Sample: strip_01
Success Count: 18
Average Disparity: 3.72
Min Disparity: 2
Max Disparity: 5
```

### `write_reverse_debug_history(...)`

---

# 10.19 C++ — `main.cpp`

## Yêu cầu

```text
Load stereo strip sample config
Load disparity search config
Load cost strategy config

Create:
    StereoStripWindowExtractor
    SADCostStrategy / SSDCostStrategy / NCCCostStrategy

Create BlockMatchingRunner
Run
Write reports
```

---

# 11. Luật block matching của project

## Luật 1 — Mỗi anchor phải tự có disparity search riêng
Không được dùng một disparity duy nhất cho toàn strip nếu chưa matching từng anchor.

## Luật 2 — `right_center = anchor - disparity`
Giữ đúng hướng stereo disparity dương.

## Luật 3 — Chỉ tính cost khi left/right window hợp lệ
Nếu runtime không cho partial window, cửa sổ vượt biên phải bị loại.

## Luật 4 — Với SAD / SSD, disparity tốt nhất là disparity có cost nhỏ nhất

## Luật 5 — Strip summary phải được tính từ các anchor match thành công
Không được lấy anchor fail đưa vào average một cách bừa.

---

# 12. Output mong muốn

## Config
```text
config/stereo_strip_sample_config.txt
config/disparity_search_config.txt
config/cost_strategy_config.txt
config/block_matching_runtime_config.txt
```

## Reports
```text
assets/outputs/disparity_strip_report.txt
assets/outputs/anchor_matching_report.txt
assets/outputs/strip_summary_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build stereo strip config
- preview disparity strip bằng NumPy
- test cost scan tại nhiều anchor

## C++
- chạy mini block matching runtime
- sinh disparity strip từ nhiều anchor
- chuẩn bị nền cho disparity map và depth map

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó biến:
- **1 sample correspondence**
thành
- **nhiều anchor local matching liên tiếp**

Đó chính là tư duy nền để sau này đi tiếp:
- block matching trên ảnh thật
- disparity map
- depth estimation từ stereo

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho strip / search / cost
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho strip sample ids
- [ ] Python có NumPy preview cho disparity strip
- [ ] C++ có `StereoStripSample`
- [ ] C++ có `AnchorDisparityResult`
- [ ] C++ có `StereoStripResult`
- [ ] C++ có `StereoStripWindowExtractor`
- [ ] C++ có ít nhất `SADCostStrategy` và `SSDCostStrategy`
- [ ] C++ chạy được anchor loop + disparity loop
- [ ] C++ sinh được disparity strip
- [ ] C++ ghi đủ 4 report

---

# 15. Gợi ý mở rộng

## 1. Nâng từ strip 1D lên patch/image 2D
Thay vì một hàng ảnh, hãy xử lý một ảnh nhỏ hoặc một patch grid.

## 2. Thêm left-right consistency check
Sau khi có disparity strip, kiểm tra ngược lại từ phải sang trái.

## 3. Thêm median filter cho disparity strip
Đây sẽ là bước đầu của post-processing.

## 4. Chuẩn bị cho Bài 46
Sau Bài 45, hướng đi rất đẹp cho chặng tiếp theo là:

# **Bài 46: Stereo Disparity Map Post-Processing Lab**

Ý tưởng:
```text
disparity strip / mini disparity map
→ invalid disparity cleanup
→ median filter / smoothing
→ left-right consistency style validation
→ refined disparity output
```

Nếu bạn muốn giữ mạch perception thật mượt, thì sau Bài 45 nên đi tiếp theo hướng:
- **Bài 46**: disparity post-processing
- **Bài 47**: depth reconstruction from disparity
- **Bài 48**: stereo point cloud mini pipeline
