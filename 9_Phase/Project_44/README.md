# 🤖 Bài 44: Stereo Correspondence Cost Explorer — Bộ khám phá cost correspondence và chọn disparity cho Humanoid Robot AI Perception

> Mini Project số 44 trong **Đợt 9**  
> **Bài 44 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9** và đi tiếp trực tiếp từ **Bài 43**.
>
> Nếu:
>
> - **Bài 41** tập trung vào **camera calibration + back-projection**
> - **Bài 42** mở rộng sang **multi-camera calibration graph**
> - **Bài 43** đi vào **stereo rectification + epipolar geometry**
>
> thì **Bài 44** sẽ đi vào đúng “trái tim” của stereo matching:
>
> ```text
> rectified stereo pair
> → candidate disparity search
> → matching cost computation
> → cost table / cost curve
> → chọn disparity tốt nhất
> ```
>
> Đây là bước rất quan trọng vì từ chỗ bạn mới chỉ “chuẩn bị hình học cho stereo”, giờ bạn bắt đầu **thực sự chọn correspondence** giữa ảnh trái và ảnh phải — chính là nền của block matching, stereo depth, và perception 3D từ stereo rig.

---

# 📌 Mục lục

- [1. Vì sao Bài 44 xuất hiện sau Bài 43](#1-vì-sao-bài-44-xuất-hiện-sau-bài-43)
- [2. Đợt 9 đang học gì](#2-đợt-9-đang-học-gì)
- [3. Bài 44 nâng từ Bài 43 lên chỗ nào](#3-bài-44-nâng-từ-bài-43-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật correspondence của project](#11-luật-correspondence-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 44 xuất hiện sau Bài 43

## Bài 43 đang làm gì?
Bài 43 đã giúp bạn đi được đến:

```text
left/right camera
→ undistortion
→ rectification
→ epipolar relation
→ row alignment
```

Tức là ảnh stereo đã được đưa về trạng thái **“matching-ready”**.

## Bài 44 nâng lên ở đâu?
Bài 44 trả lời câu hỏi tiếp theo:

> Sau khi ảnh trái/phải đã rectify xong, **làm sao tìm điểm tương ứng giữa hai ảnh?**

Đó chính là bài toán **correspondence search**.

Với một pixel / patch ở ảnh trái, bạn sẽ:
1. quét qua nhiều disparity candidate
2. lấy patch bên phải tương ứng
3. tính **matching cost**
4. so sánh các cost
5. chọn disparity tốt nhất

Tức là Bài 44 chuyển từ:

```text
stereo geometry chuẩn bị cho matching
```

sang:

```text
thực sự tính cost và chọn disparity
```

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

## Vì sao Bài 44 vẫn bám Đợt 9?
Dù Bài 44 đã chạm sang matching, nó vẫn đứng vững trên nền Đợt 9:
- ảnh đầu vào là **rectified stereo geometry** từ Bài 43
- bạn vẫn dùng **Graph / BST / NumPy** để tổ chức sample, cost pipeline, disparity preview
- vẫn dùng **C++ polymorphism** để thiết kế nhiều cost strategy
- vẫn bám perception core của stereo

---

# 3. Bài 44 nâng từ Bài 43 lên chỗ nào

## Bài 43
- undistort
- rectify
- epipolar / row alignment
- disparity preview từ **điểm cặp đã biết sẵn**

## Bài 44
- chưa biết disparity trước
- phải **tự quét nhiều disparity candidate**
- phải **tự tính matching cost**
- phải **tự chọn disparity tốt nhất**

Đây là khác biệt rất lớn:

### Bài 43
> “Nếu đã có left point và right point thì disparity là bao nhiêu?”

### Bài 44
> “Từ left patch này, right patch nào là ứng viên tốt nhất?”

Bài 44 vì thế là bước đầu tiên bạn thật sự bước vào **stereo matching runtime**.

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Correspondence Cost Explorer**

System này nhận đầu vào là một tập **rectified stereo samples** hoặc **synthetic row samples**.  
Mỗi sample mô tả:

- một **left row signal** hoặc **left patch**
- một **right row signal** hoặc **right patch**
- một **vị trí anchor ở ảnh trái**
- một **disparity search range**

Mục tiêu là:

## Với mỗi sample:
1. lấy patch / window ở ảnh trái
2. quét qua nhiều disparity candidate `d`
3. ở mỗi `d`, lấy patch bên phải tương ứng
4. tính **matching cost**
5. tạo **cost table**
6. chọn disparity có cost tốt nhất
7. optional: so với ground-truth disparity nếu có

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build rectified stereo sample config
→ build disparity search config
→ preview disparity candidates
→ preview cost table bằng NumPy

C++
→ load stereo correspondence samples
→ extract left/right windows
→ evaluate cost for many disparity candidates
→ choose best disparity
→ export cost curve / ranking / matching report
```

Mục tiêu cốt lõi:
- hiểu **correspondence search** là gì
- hiểu **vì sao disparity được chọn bằng cost tối ưu**
- hiểu **SAD / SSD / NCC khác nhau ra sao ở mức runtime**
- chuẩn bị nền cho:
  - block matching
  - SGM
  - disparity map generation

---

# 6. Pipeline tổng thể

```text
Load Stereo Correspondence Sample Config
Load Disparity Search Config
Load Cost Strategy Config

Create StereoSampleWindowExtractor
Create Cost Engine
Create DisparitySearchRunner

For each stereo sample:
    1. extract left reference window

    2. for disparity d in [d_min, d_max]:
           extract right candidate window
           compute cost(left_window, right_window)
           store cost record

    3. sort / scan cost records
    4. choose best disparity
    5. compare with GT if available
    6. write result

Export:
    cost table report
    disparity selection report
    matching summary report
    reverse debug history
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
- cost exploration runtime design

## Computer Vision / Geometry
- rectified stereo pair
- disparity search
- correspondence
- matching cost
- SAD / SSD / NCC intuition
- disparity selection

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **cost exploration pipeline**:

```text
LeftPatch
→ CandidateDisparities
→ RightCandidateWindows
→ CostComputation
→ BestDisparitySelection
```

## 2. Python BST
Lưu `sample_id` hoặc `frame_id` của stereo sample.

## 3. `std::vector<StereoCorrespondenceSample>`
Danh sách sample cần matching.

## 4. `std::vector<DisparityCostRecord>`
Danh sách cost record cho từng disparity candidate.

## 5. `std::vector<StereoCorrespondenceResult>`
Kết quả cuối cho từng sample.

## 6. `std::stack<CorrespondenceDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Window extraction
Với sample đang xét:
- lấy **left reference window**
- với mỗi disparity `d`, lấy **right candidate window**

### Ví dụ
Nếu sample nằm ở cột `u_left = 20`, window size = 5, disparity = 3 thì:
- cửa sổ trái quanh `u=20`
- cửa sổ phải quanh `u=17`

---

## Algorithm 2 — Matching cost
Bạn phải hỗ trợ **ít nhất 2 cost strategy** trong 3 cái sau:

### `SAD`
```text
sum(|L_i - R_i|)
```

### `SSD`
```text
sum((L_i - R_i)^2)
```

### `NCC` hoặc normalized similarity kiểu đơn giản
Bạn có thể đơn giản hóa nếu muốn.

---

## Algorithm 3 — Cost table generation
Với mỗi disparity candidate:
- lưu `disparity`
- lưu `cost`
- lưu `is_valid_window`

---

## Algorithm 4 — Best disparity selection
Chọn disparity tốt nhất:
- với SAD / SSD → cost nhỏ nhất
- với NCC nếu dùng score similarity → score lớn nhất

---

## Algorithm 5 — Error evaluation
Nếu có `gt_disparity`:
- tính `disparity_error = |pred - gt|`

---

# 9. Cấu trúc folder

```text
mini_project_44_stereo_correspondence_cost_explorer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ cost_table_report.txt
│     ├─ disparity_selection_report.txt
│     ├─ matching_summary_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_correspondence_sample_config.txt
│  ├─ disparity_search_config.txt
│  ├─ cost_strategy_config.txt
│  └─ correspondence_runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ correspondence_graph_preview.py
│     ├─ stereo_sample_bst.py
│     ├─ numpy_cost_preview.py
│     └─ synthetic_row_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoCorrespondenceSample.hpp
   │  ├─ DisparityCostRecord.hpp
   │  ├─ StereoCorrespondenceResult.hpp
   │  ├─ CorrespondenceDebugRecord.hpp
   │  ├─ WindowRange.hpp
   │  ├─ BaseWindowExtractor.hpp
   │  ├─ StereoWindowExtractor.hpp
   │  ├─ BaseCostStrategy.hpp
   │  ├─ SADCostStrategy.hpp
   │  ├─ SSDCostStrategy.hpp
   │  ├─ NCCCostStrategy.hpp
   │  ├─ DisparitySearchRunner.hpp
   │  └─ CorrespondenceReportWriter.hpp
   │
   └─ src/
      ├─ StereoWindowExtractor.cpp
      ├─ SADCostStrategy.cpp
      ├─ SSDCostStrategy.cpp
      ├─ NCCCostStrategy.cpp
      ├─ DisparitySearchRunner.cpp
      └─ CorrespondenceReportWriter.cpp
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

# 10.2 Python — `StereoCorrespondenceConfigBuilder`

Tạo class con:

```python
class StereoCorrespondenceConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_correspondence_sample(
    sample_id,
    left_row_values,
    right_row_values,
    left_anchor_index,
    window_radius,
    disparity_min,
    disparity_max,
    gt_disparity=None
)`

### `set_cost_strategy(cost_name)`
- `SAD`
- `SSD`
- `NCC`

### `set_runtime_options(
    allow_partial_window,
    choose_min_cost,
    save_all_cost_records
)`

### `write_sample_config()`
### `write_search_config()`
### `write_cost_strategy_config()`
### `write_runtime_config()`

---

# 10.3 Python — `correspondence_graph_preview.py`

Tạo class:

```python
class CorrespondenceGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
LeftWindow
→ CandidateDisparities
→ RightWindows
→ CostRecords
→ BestDisparity
```

---

# 10.4 Python — `stereo_sample_bst.py`

Tạo BST cho `sample_id` / `frame_id`.

---

# 10.5 Python — `numpy_cost_preview.py`

Tạo class:

```python
class NumPyCostPreview:
```

## Hàm cần có

### `extract_window(row_values, center_index, radius)`
### `compute_sad(left_window, right_window)`
### `compute_ssd(left_window, right_window)`
### `scan_disparities(left_row, right_row, left_anchor, radius, d_min, d_max)`

---

# 10.6 C++ — `StereoCorrespondenceSample`

```cpp
struct StereoCorrespondenceSample
{
    std::string sample_id;

    std::vector<int> left_row_values;
    std::vector<int> right_row_values;

    int left_anchor_index;
    int window_radius;

    int disparity_min;
    int disparity_max;

    bool has_gt_disparity;
    int gt_disparity;
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

# 10.8 C++ — `StereoCorrespondenceResult`

```cpp
struct StereoCorrespondenceResult
{
    std::string sample_id;

    int predicted_disparity;
    double best_cost;
    bool success;

    bool has_gt_disparity;
    int gt_disparity;
    int disparity_error;

    std::vector<DisparityCostRecord> cost_records;
};
```

---

# 10.9 C++ — `CorrespondenceDebugRecord`

```cpp
struct CorrespondenceDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.10 C++ — `WindowRange`

```cpp
struct WindowRange
{
    int left_index;
    int right_index;
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

# 10.12 C++ — `StereoWindowExtractor`

Kế thừa `BaseWindowExtractor`.

## Nhiệm vụ
- cắt cửa sổ từ một hàng pixel/signal
- có thể fail nếu window vượt biên và runtime không cho phép partial window

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

Kế thừa `BaseCostStrategy`.

## Nhiệm vụ
Tính:

```text
sum(|L_i - R_i|)
```

---

# 10.15 C++ — `SSDCostStrategy`

Kế thừa `BaseCostStrategy`.

## Nhiệm vụ
Tính:

```text
sum((L_i - R_i)^2)
```

---

# 10.16 C++ — `NCCCostStrategy` (khuyến khích)
Kế thừa `BaseCostStrategy`.

## Nhiệm vụ
Tính score tương đồng kiểu normalized correlation đơn giản.

> Nếu chưa muốn làm NCC ngay, bạn vẫn phải làm ít nhất **SAD + SSD**.

---

# 10.17 C++ — `DisparitySearchRunner`

Tạo class trung tâm:

```cpp
class DisparitySearchRunner
```

## Thuộc tính

```cpp
private:
    std::vector<StereoCorrespondenceSample> samples;

    std::shared_ptr<BaseWindowExtractor> window_extractor;
    std::shared_ptr<BaseCostStrategy> cost_strategy;

    std::vector<StereoCorrespondenceResult> results;
    std::stack<CorrespondenceDebugRecord> debug_history;
```

## Hàm cần có

### `load_samples(const std::string& path)`

### `run()`
Pseudo:

```text
for each sample:
    1. extract left reference window
    2. for d from disparity_min to disparity_max:
           right_center = left_anchor_index - d
           extract right window
           if valid:
               cost = cost_strategy.compute_cost(...)
           save DisparityCostRecord
    3. choose best disparity
    4. compare with GT if available
    5. push result + debug history
```

### `const std::vector<StereoCorrespondenceResult>& get_results() const`
### `std::vector<CorrespondenceDebugRecord> get_debug_history_reverse()`

---

# 10.18 C++ — `CorrespondenceReportWriter`

Tạo class:

```cpp
class CorrespondenceReportWriter
```

## Hàm cần có

### `write_cost_table_report(...)`
Ví dụ:

```text
[Cost Table]
Sample: pair_01
d=0  cost=42
d=1  cost=31
d=2  cost=18
d=3  cost=7
d=4  cost=13
```

### `write_disparity_selection_report(...)`
Ví dụ:

```text
[Best Disparity]
Sample: pair_01
Predicted disparity: 3
Best cost: 7
```

### `write_matching_summary_report(...)`
Ví dụ:

```text
[Matching Summary]
Sample: pair_01
GT disparity: 3
Pred disparity: 3
Disparity error: 0
```

### `write_reverse_debug_history(...)`

---

# 10.19 C++ — `main.cpp`

## Yêu cầu

```text
Load stereo correspondence samples
Load search config
Load cost strategy config

Create:
    StereoWindowExtractor
    SADCostStrategy / SSDCostStrategy / NCCCostStrategy

Create DisparitySearchRunner
Run
Write reports
```

---

# 11. Luật correspondence của project

## Luật 1 — Chỉ quét disparity trong search range đã cho
Không được match toàn hàng nếu config chỉ cho `d_min → d_max`.

## Luật 2 — `right_center = left_anchor_index - disparity`
Vì trong stereo chuẩn, điểm ở ảnh phải thường lệch về bên trái so với ảnh trái khi disparity dương.

## Luật 3 — Chỉ tính cost khi cả left/right window hợp lệ
Nếu runtime không cho partial window thì window vượt biên phải bị đánh dấu invalid.

## Luật 4 — Với SAD / SSD, disparity tốt nhất là disparity có cost nhỏ nhất

## Luật 5 — Nếu có ground-truth disparity
Phải tính `disparity_error`

---

# 12. Output mong muốn

## Config
```text
config/stereo_correspondence_sample_config.txt
config/disparity_search_config.txt
config/cost_strategy_config.txt
config/correspondence_runtime_config.txt
```

## Reports
```text
assets/outputs/cost_table_report.txt
assets/outputs/disparity_selection_report.txt
assets/outputs/matching_summary_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build sample / disparity range / cost strategy config
- preview cost scan bằng NumPy
- preview candidate disparity logic

## C++
- chạy correspondence search runtime
- tính cost cho từng disparity candidate
- chọn disparity tốt nhất
- chuẩn bị nền cho disparity map runtime

## Computer Vision / Robot Perception
Đây là project cực kỳ quan trọng vì nó là lần đầu bạn thật sự chạm vào:
- **matching cost**
- **correspondence search**
- **disparity selection**

Nếu thiếu Bài 44, bạn sẽ rất khó nối tiếp sang:
- block matching
- SGM
- stereo depth pipeline hoàn chỉnh

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho sample / search / cost
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho sample ids
- [ ] Python có NumPy preview cho SAD / SSD / disparity scan
- [ ] C++ có `StereoCorrespondenceSample`
- [ ] C++ có `StereoWindowExtractor`
- [ ] C++ có `BaseCostStrategy`
- [ ] C++ có ít nhất `SADCostStrategy` và `SSDCostStrategy`
- [ ] C++ có `DisparitySearchRunner`
- [ ] C++ quét được nhiều disparity candidate
- [ ] C++ chọn được best disparity
- [ ] C++ ghi đủ 4 report

---

# 15. Gợi ý mở rộng

## 1. Thêm patch 2D thay vì chỉ row 1D
Hiện tại có thể làm theo **row signal / 1D window** để tập trung vào bản chất disparity. Sau đó có thể nâng lên patch 2D.

## 2. Thêm sub-pixel disparity refinement
Sau khi có best disparity nguyên, nội suy thêm để ước lượng tốt hơn.

## 3. So sánh nhiều cost strategy trên cùng sample
Ví dụ:
- SAD chọn `d=3`
- SSD chọn `d=3`
- NCC chọn `d=2`

## 4. Chuẩn bị cho Bài 45
Sau Bài 44, hướng đi rất đẹp cho Đợt 9 là:

# **Bài 45: Stereo Block Matching Mini Runtime**

Ý tưởng:
```text
rectified left/right image
→ trượt window trên nhiều pixel
→ disparity cho từng pixel
→ tạo disparity strip / mini disparity map
```

Lúc đó chuỗi Đợt 9 sẽ rất liền:
- **Bài 41**: calibration + back-projection
- **Bài 42**: multi-camera graph
- **Bài 43**: rectification + epipolar
- **Bài 44**: cost exploration + disparity search
- **Bài 45**: block matching runtime mini
