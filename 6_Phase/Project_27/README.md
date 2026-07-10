# 🤖 Bài 27: Feature Match Graph for Multi-Frame Visual Memory — Đồ thị liên kết feature giữa nhiều frame cho Humanoid Robot AI Perception

> Mini Project số 27 trong **Đợt 6**  
> **Bài 27 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5 + Đợt 6**.  
> Nếu **Bài 26** là một **Stereo Feature Registration Workbench** cho từng cặp ảnh, thì **Bài 27** nâng cấp lên một mức “robotic” hơn:
>
> **không chỉ match từng cặp ảnh nữa, mà lưu quan hệ match giữa nhiều frame thành một graph trí nhớ thị giác**.

---

# 📌 Mục lục

- [1. Bài 27 lấy gì từ Đợt 6](#1-bài-27-lấy-gì-từ-đợt-6)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 27 nằm ở đâu trong roadmap](#3-bài-27-nằm-ở-đâu-trong-roadmap)
- [4. Từ Bài 26 sang Bài 27 đã nâng cấp gì](#4-từ-bài-26-sang-bài-27-đã-nâng-cấp-gì)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. Đợt 1 → Đợt 6 được dùng như thế nào](#8-đợt-1--đợt-6-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 27 lấy gì từ Đợt 6

Theo roadmap, **Đợt 6 (Ngày 11–12)** có chủ đề **Feature extractor project** với ba nhóm kiến thức chính:

## Python — Phase 7: Project Structure
- `Python Virtual Environments`
- `Python Modules`
- `if __name__ == "__main__"`

## C++ — Phase 5 + Phase 6
- `C++ Pass By Reference`
- `C++ Pointers`
- constructor trong inheritance
- `virtual`
- `override`

## Computer Vision — Phase 3
- `Feature Matching`
- `Homography`
- `Image Registration`

Roadmap này xác định Đợt 6 là giai đoạn chuyển từ “dùng feature trong một bài toán” sang “xây project feature extractor / feature workbench có cấu trúc” fileciteturn9file0

**Bài 27** bám đúng tinh thần đó, nhưng đẩy lên thêm một bước:  
thay vì chỉ xử lý **1 pair tại một thời điểm**, ta bắt đầu xây **multi-frame visual memory graph**.

---

# 2. Mô tả

Một humanoid robot khi quan sát môi trường thường không chỉ có:

```text
ảnh A ↔ ảnh B
```

mà là cả một chuỗi:

```text
frame_01
frame_02
frame_03
frame_04
...
frame_N
```

Trong chuỗi đó, robot cần biết:

- frame nào nhìn giống frame nào
- frame nào match mạnh với frame nào
- giữa 2 frame có đủ good matches không
- homography giữa chúng có ổn không
- frame nào có thể coi là “gần” nhau trong trí nhớ thị giác

**Bài 27** vì vậy xây một **Feature Match Graph**:

- **mỗi frame = 1 node**
- **mỗi quan hệ match = 1 edge**
- edge lưu:
  - total matches
  - good matches
  - inlier ratio
  - homography valid / invalid
  - match score

Từ đó robot có thể tạo một **visual memory graph** cơ bản.

<p align="center">
  <img src="images/project_27.png" width="800">
</p>

---

# 3. Bài 27 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**
- **Đợt 6 = Bài 26 → 30**

Vì vậy:

## **Bài 27 = bài thứ hai của Đợt 6**

Nó vẫn phải dùng toàn bộ nền từ **Đợt 1 → Đợt 6**, nhưng trọng tâm mới là:

```text
multi-frame matching + graph memory
```

---

# 4. Từ Bài 26 sang Bài 27 đã nâng cấp gì

## Bài 26
Bài 26 chủ yếu là:

```text
single pair workbench
```

- stereo pair
- registration pair
- extract feature
- match
- homography
- registration

## Bài 27
Bài 27 chuyển sang:

```text
multi-frame workbench
```

- load nhiều frame
- xét match giữa các frame theo một chiến lược
- build graph quan hệ giữa frame
- query “frame gần nhất”
- query “frame nào kết nối mạnh”
- export visual memory report

Nói ngắn gọn:

```text
Bài 26 = xử lý cặp
Bài 27 = quản lý mạng lưới match giữa nhiều frame
```

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu luồng:

```text
Frame Sequence + Match Graph Config
→ load frames
→ extract features for each frame
→ choose candidate frame pairs
→ run descriptor matching
→ filter good matches
→ estimate homography for each candidate pair
→ compute edge score
→ build graph
→ query graph for strongest neighbors / reusable visual memory links
→ save graph report + pair visualizations
```

---

# 6. Pipeline perception của bài

```text
Python Config Builder
→ build frame manifest
→ build graph config
→ build candidate-pair strategy config

C++ Runtime
→ load frame manifest
→ load graph config
→ initialize extractor
→ initialize graph builder

for each frame:
    → load image
    → extract features
    → store FrameFeatureRecord

for each candidate frame pair:
    → match descriptors
    → filter good matches
    → estimate homography
    → compute inlier ratio
    → compute edge score
    → if edge strong enough:
         add graph edge

after graph built:
    → compute adjacency report
    → find top-k neighbors for each frame
    → find strongest connected frame
    → export graph summary
    → save selected match visualization images
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- dict / list / set
- graph-like config representation

## C++
- class / inheritance
- `virtual` / `override`
- pass by reference
- pointer cơ bản
- vector
- map
- struct
- destructor
- static member

## Computer Vision
- feature extraction
- descriptor matching
- good match filtering
- homography
- inlier ratio
- registration quality score

---

# 8. Đợt 1 → Đợt 6 được dùng như thế nào

## Đợt 1
- class / function / if else / loop
- đọc ảnh cơ bản

## Đợt 2
- list / dict / vector
- xử lý dữ liệu frame manifest

## Đợt 3
- nested loop để xét candidate frame pairs
- menu logic / debug loop

## Đợt 4
- file handling
- exception
- ghi report
- kiểm tra lỗi file ảnh

## Đợt 5
- OOP rõ ràng hơn
- feature detection / descriptor / homography
- registration mindset

## Đợt 6
- project structure
- module Python
- `virtual` / `override`
- pointer / reference
- feature workbench architecture

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 27, bạn cần nắm được 12 ý:

## 1. Một robot không chỉ làm việc với từng pair ảnh rời rạc
Nó thường phải quản lý cả chuỗi frame.

## 2. Feature matching có thể trở thành “trí nhớ liên kết”
Nếu frame A match mạnh với frame B, ta có thể coi đó là một liên kết trong memory graph.

## 3. Graph là cấu trúc rất hợp cho visual memory
Node = frame, edge = mức độ liên quan thị giác.

## 4. Không phải cặp frame nào cũng cần match
Cần có **candidate pair strategy**.

## 5. Good matches và inlier ratio đều quan trọng
Good matches nhiều nhưng inlier ratio thấp thì edge có thể vẫn yếu.

## 6. Homography giúp xác minh rằng match có tính hình học
Không chỉ là descriptor trùng ngẫu nhiên.

## 7. Một workbench tốt nên lưu feature của từng frame trước
Thay vì extract đi extract lại.

## 8. Python rất hợp để tạo manifest và graph config
Còn C++ xử lý runtime matching.

## 9. Bài này là bước chuẩn bị đẹp cho visual relocalization / loop closure
Vì bản chất chúng đều cần “frame nào giống frame nào”.

## 10. `virtual` / `override` giúp thay extractor hoặc graph builder dễ hơn
Ví dụ sau này đổi ORB → SIFT.

## 11. Graph memory là bước trung gian giữa “feature matching” và “scene memory”
Nó chưa phải semantic memory, nhưng đã là cấu trúc perception có trí nhớ.

## 12. Đây là một project rất đúng với Đợt 6
Vì nó dùng trực tiếp **Feature Matching + Homography + cấu trúc project** của roadmap Đợt 6 fileciteturn9file0

---

# 10. Cấu trúc folder

```text
mini_project_27_feature_match_graph_for_multi_frame_visual_memory/
│
├─ README.md
│
├─ assets/
│  ├─ frames/
│  │  ├─ frame_01.jpg
│  │  ├─ frame_02.jpg
│  │  ├─ frame_03.jpg
│  │  ├─ frame_04.jpg
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ pair_frame_01_frame_02_match.jpg
│     ├─ pair_frame_01_frame_03_match.jpg
│     ├─ pair_frame_02_frame_04_match.jpg
│     ├─ graph_summary.txt
│     ├─ frame_neighbor_report.txt
│     └─ adjacency_matrix.txt
│
├─ config/
│  ├─ frame_manifest.txt
│  └─ feature_graph_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ graph_report_template.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ FrameRecord.hpp
   │  ├─ FeatureGraphConfig.hpp
   │  ├─ FrameFeatureRecord.hpp
   │  ├─ MatchEdgeRecord.hpp
   │  ├─ VisualMemoryGraphRecord.hpp
   │  ├─ BaseFeatureExtractor.hpp
   │  ├─ ORBFeatureExtractor.hpp
   │  ├─ BaseGraphBuilder.hpp
   │  ├─ FeatureMatchGraphBuilder.hpp
   │  └─ GraphReportWriter.hpp
   │
   └─ src/
      ├─ ORBFeatureExtractor.cpp
      ├─ FeatureMatchGraphBuilder.cpp
      └─ GraphReportWriter.cpp
```

---

# 11. Yêu cầu mini-project

# 11.1 Python — `BaseConfigBuilder`

**File:**
```text
python/tools/config_builder.py
```

Tạo class cha:

```python
class BaseConfigBuilder:
```

## Thuộc tính cần có
```python
project_name
frame_manifest_path
feature_graph_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn config

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

### `@staticmethod validate_image_extension(path)`
- chấp nhận `.jpg`, `.jpeg`, `.png`

---

# 11.2 Python — `FeatureGraphConfigBuilder`

Tạo class con:

```python
class FeatureGraphConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
frames
graph_config
```

## `graph_config` ví dụ

```python
{
    "feature_type": "ORB",
    "max_features": 1500,
    "match_mode": "BF_HAMMING",
    "good_match_distance_threshold": 45.0,
    "min_good_matches_for_edge": 18,
    "ransac_reproj_threshold": 4.0,
    "min_inlier_ratio_for_edge": 0.30,
    "max_frame_gap": 3,
    "save_match_visualization": True
}
```

## Hàm cần có

### `add_frame(frame_name, image_path)`
- thêm frame vào danh sách
- validate extension

### `set_graph_config(...)`
**Hành vi**
- `feature_type` thuộc `"ORB"` hoặc `"SIFT"`
- `max_features > 0`
- `good_match_distance_threshold > 0`
- `min_good_matches_for_edge >= 4`
- `ransac_reproj_threshold > 0`
- `0.0 <= min_inlier_ratio_for_edge <= 1.0`
- `max_frame_gap >= 1`

### `write_frame_manifest()`
Format gợi ý:

```text
frame_name=frame_01
image_path=assets/frames/frame_01.jpg
```

### `write_feature_graph_config()`
Format gợi ý:

```text
feature_type=ORB
max_features=1500
match_mode=BF_HAMMING
good_match_distance_threshold=45.0
min_good_matches_for_edge=18
ransac_reproj_threshold=4.0
min_inlier_ratio_for_edge=0.30
max_frame_gap=3
save_match_visualization=true
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **6 frame**
- set graph config
- ghi:
  - `config/frame_manifest.txt`
  - `config/feature_graph_config.txt`

---

# 11.4 C++ — `FrameRecord`

**File:**
```text
cpp/include/FrameRecord.hpp
```

```cpp
struct FrameRecord
{
    std::string frame_name;
    std::string image_path;
};
```

---

# 11.5 C++ — `FeatureGraphConfig`

**File:**
```text
cpp/include/FeatureGraphConfig.hpp
```

```cpp
struct FeatureGraphConfig
{
    std::string feature_type;
    int max_features;
    std::string match_mode;
    double good_match_distance_threshold;
    int min_good_matches_for_edge;
    double ransac_reproj_threshold;
    double min_inlier_ratio_for_edge;
    int max_frame_gap;
    bool save_match_visualization;
};
```

---

# 11.6 C++ — `FrameFeatureRecord`

**File:**
```text
cpp/include/FrameFeatureRecord.hpp
```

```cpp
struct FrameFeatureRecord
{
    std::string frame_name;
    cv::Mat image;
    std::vector<cv::KeyPoint> keypoints;
    cv::Mat descriptors;
    bool is_valid;
};
```

---

# 11.7 C++ — `MatchEdgeRecord`

**File:**
```text
cpp/include/MatchEdgeRecord.hpp
```

```cpp
struct MatchEdgeRecord
{
    std::string frame_a;
    std::string frame_b;

    int total_matches;
    int good_matches;
    int inlier_matches;
    double inlier_ratio;
    double edge_score;

    bool homography_valid;
    bool edge_accepted;
};
```

---

# 11.8 C++ — `VisualMemoryGraphRecord`

**File:**
```text
cpp/include/VisualMemoryGraphRecord.hpp
```

```cpp
struct VisualMemoryGraphRecord
{
    std::string frame_name;
    std::vector<std::string> neighbor_frames;
    std::vector<double> neighbor_scores;
};
```

---

# 11.9 C++ — `BaseFeatureExtractor`

**File:**
```text
cpp/include/BaseFeatureExtractor.hpp
```

```cpp
class BaseFeatureExtractor
{
public:
    virtual FrameFeatureRecord extract(
        const std::string& frame_name,
        const cv::Mat& image
    ) = 0;

    virtual ~BaseFeatureExtractor() = default;
};
```

---

# 11.10 C++ — `ORBFeatureExtractor`

**File:**
```text
cpp/include/ORBFeatureExtractor.hpp
cpp/src/ORBFeatureExtractor.cpp
```

Class kế thừa:

```cpp
class ORBFeatureExtractor : public BaseFeatureExtractor
```

## Thuộc tính
```cpp
private:
    int max_features;
```

## Constructor
```cpp
ORBFeatureExtractor(int max_features);
```

## Hàm override
```cpp
FrameFeatureRecord extract(
    const std::string& frame_name,
    const cv::Mat& image
) override;
```

## Yêu cầu
- dùng `cv::ORB::create(max_features)`
- grayscale nếu cần
- detect + compute descriptors

---

# 11.11 C++ — `BaseGraphBuilder`

**File:**
```text
cpp/include/BaseGraphBuilder.hpp
```

```cpp
class BaseGraphBuilder
{
public:
    virtual void load_frame_manifest(const std::string& path) = 0;
    virtual void load_graph_config(const std::string& path) = 0;
    virtual void build_graph() = 0;
    virtual ~BaseGraphBuilder() = default;
};
```

---

# 11.12 C++ — `FeatureMatchGraphBuilder`

**File:**
```text
cpp/include/FeatureMatchGraphBuilder.hpp
cpp/src/FeatureMatchGraphBuilder.cpp
```

Class kế thừa:

```cpp
class FeatureMatchGraphBuilder : public BaseGraphBuilder
```

## Thuộc tính cần có

```cpp
private:
    std::vector<FrameRecord> frames;
    FeatureGraphConfig config;

    BaseFeatureExtractor* extractor;

    std::vector<FrameFeatureRecord> frame_features;
    std::vector<MatchEdgeRecord> edges;
    std::vector<VisualMemoryGraphRecord> graph_records;

    static int graph_builder_instance_count;
```

## Constructor / Destructor
```cpp
FeatureMatchGraphBuilder();
~FeatureMatchGraphBuilder();
```

### Destructor phải làm gì?
- `delete extractor` nếu khác `nullptr`
- log số edge đã tạo

---

## Hàm load

### `load_frame_manifest(const std::string& path) override;`
- đọc danh sách frame

### `load_graph_config(const std::string& path) override;`
- đọc config graph

---

## Hàm khởi tạo extractor

### `initialize_extractor()`
- nếu `feature_type == "ORB"` → tạo `new ORBFeatureExtractor(max_features)`
- nếu chưa hỗ trợ `SIFT` thì log rõ

---

## Hàm extract

### `extract_all_frame_features()`
- loop toàn bộ frame
- load ảnh
- gọi extractor
- lưu `FrameFeatureRecord`

---

## Hàm tạo candidate pairs

### `std::vector<std::pair<int, int>> build_candidate_pairs() const;`

**Yêu cầu**
- chỉ xét cặp `(i, j)` với `j > i`
- chỉ xét nếu `(j - i) <= max_frame_gap`

Ví dụ:
- nếu `max_frame_gap = 2`
- frame_01 sẽ chỉ xét với frame_02, frame_03

---

## Hàm matching

### `std::vector<cv::DMatch> match_descriptors(
    const cv::Mat& descriptors_a,
    const cv::Mat& descriptors_b
) const;`

### `std::vector<cv::DMatch> filter_good_matches(
    const std::vector<cv::DMatch>& matches
) const;`

### `MatchEdgeRecord build_edge_record(
    const FrameFeatureRecord& frame_a,
    const FrameFeatureRecord& frame_b
) const;`

## Hành vi của `build_edge_record(...)`
1. match descriptors
2. lọc good matches
3. nếu good matches < `min_good_matches_for_edge`
   - edge fail
4. nếu đủ matches:
   - build point correspondences
   - `findHomography(..., RANSAC, ransac_reproj_threshold, mask)`
   - tính `inlier_matches`
   - tính `inlier_ratio`
   - tính `edge_score`

### Công thức gợi ý cho `edge_score`
Bạn có thể tự chọn, ví dụ:

```text
edge_score = 0.6 * normalized_good_matches + 0.4 * inlier_ratio
```

### Điều kiện accept edge
- `good_matches >= min_good_matches_for_edge`
- `inlier_ratio >= min_inlier_ratio_for_edge`
- homography valid

---

## Hàm build graph

### `build_graph_records()`
- với mỗi frame, gom toàn bộ edge accepted có liên quan tới frame đó
- lưu neighbor name + neighbor score

### `build_adjacency_matrix_text() const`
- tạo text ma trận kề đơn giản để ghi file

---

## Visualization

### `cv::Mat build_match_visualization(
    const FrameFeatureRecord& frame_a,
    const FrameFeatureRecord& frame_b,
    const std::vector<cv::DMatch>& good_matches
) const;`

### `save_selected_match_visualizations()`
- chỉ lưu các edge accepted

---

## Hàm query graph

### `std::string find_best_neighbor(const std::string& frame_name) const;`
- trả frame neighbor có score cao nhất

### `std::vector<std::string> find_all_neighbors(const std::string& frame_name) const;`

---

## Hàm chính

### `void build_graph() override;`
1. `initialize_extractor()`
2. `extract_all_frame_features()`
3. tạo candidate pairs
4. loop từng pair → build edge
5. build graph records
6. save selected match visualizations

### Getter

```cpp
const std::vector<MatchEdgeRecord>& get_edges() const;
const std::vector<VisualMemoryGraphRecord>& get_graph_records() const;
```

---

# 11.13 C++ — `GraphReportWriter`

**File:**
```text
cpp/include/GraphReportWriter.hpp
cpp/src/GraphReportWriter.cpp
```

Tạo class:

```cpp
class GraphReportWriter
```

## Hàm cần có

### `write_graph_summary(
    const std::string& path,
    const std::vector<MatchEdgeRecord>& edges
);`

Format gợi ý:

```text
[Graph Edge]
Frame A: frame_01
Frame B: frame_02
Total Matches: 622
Good Matches: 141
Inlier Matches: 118
Inlier Ratio: 0.836
Edge Score: 0.782
Homography Valid: true
Edge Accepted: true
----------------------------------------
```

### `write_neighbor_report(
    const std::string& path,
    const std::vector<VisualMemoryGraphRecord>& graph_records
);`

Format gợi ý:

```text
[Frame Node]
Frame: frame_03
Neighbors:
- frame_02 (score=0.74)
- frame_04 (score=0.69)
- frame_05 (score=0.48)
----------------------------------------
```

### `write_adjacency_matrix(
    const std::string& path,
    const std::string& adjacency_text
);`

---

# 11.14 C++ — `main.cpp`

## Yêu cầu
- tạo `FeatureMatchGraphBuilder`
- load:
  - `config/frame_manifest.txt`
  - `config/feature_graph_config.txt`
- gọi `build_graph()`
- tạo `GraphReportWriter`
- ghi:
  - `assets/outputs/graph_summary.txt`
  - `assets/outputs/frame_neighbor_report.txt`
  - `assets/outputs/adjacency_matrix.txt`

---

# 12. Điều kiện bắt buộc

Project bắt buộc phải có:

- OOP Python
- OOP C++
- inheritance Python
- inheritance C++
- `super()`
- module Python
- `if __name__ == "__main__"`
- `virtual` / `override`
- pointer cơ bản
- `nullptr`
- pass by reference
- `static`
- `.hpp / .cpp`
- feature extraction
- feature matching
- homography
- graph memory
- graph report

---

# 13. Output mong muốn

## File config

```text
config/frame_manifest.txt
config/feature_graph_config.txt
```

## Ảnh output

```text
assets/outputs/pair_frame_01_frame_02_match.jpg
assets/outputs/pair_frame_02_frame_03_match.jpg
assets/outputs/pair_frame_03_frame_05_match.jpg
```

## File report

```text
assets/outputs/graph_summary.txt
assets/outputs/frame_neighbor_report.txt
assets/outputs/adjacency_matrix.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Visual Memory Graph Config Builder
```

Dùng để tạo:
- frame manifest
- graph config

## C++ đóng vai trò gì?

C++ là:

```text
Multi-Frame Feature Matching Runtime
```

Dùng để:
- extract features cho từng frame
- match candidate pairs
- build edge records
- tạo graph memory
- query neighbor frames
- ghi report

## Computer Vision đóng vai trò gì?

CV là:

```text
frame sequence
→ feature extraction
→ descriptor matching
→ homography validation
→ graph-based visual memory
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo `frame_manifest.txt`
- [ ] Python tạo `feature_graph_config.txt`
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có module + `if __name__ == "__main__"`
- [ ] C++ có `BaseFeatureExtractor`
- [ ] C++ có `ORBFeatureExtractor`
- [ ] C++ có `BaseGraphBuilder`
- [ ] C++ có `FeatureMatchGraphBuilder`
- [ ] C++ có pointer extractor
- [ ] C++ có `virtual` / `override`
- [ ] C++ extract features cho toàn bộ frame
- [ ] C++ build candidate pairs
- [ ] C++ match descriptors
- [ ] C++ estimate homography
- [ ] C++ build edge score
- [ ] C++ build graph record
- [ ] C++ ghi 3 file report

---

# 16. Gợi ý mở rộng

## 1. Thêm loop-closure style matching
Không chỉ xét frame gần nhau, mà còn xét frame xa để tìm cảnh lặp lại.

## 2. Tách `BaseMatcher`
Để sau này có:
- BFMatcher
- FLANN matcher
- KNN ratio test matcher

## 3. Thêm frame timestamp / robot pose metadata
Để graph không chỉ có thị giác mà còn có ngữ cảnh robot.

## 4. Chuẩn bị cho Bài 28
Sau Bài 27, bước rất hợp lý là:

```text
Reference Scene Revisit Detector with Visual Memory Graph
```

Tức là dùng graph này để phát hiện:
- robot đang quay lại cảnh cũ nào
- frame hiện tại gần nhất với vùng nào trong memory
- có thể relocalize bằng visual memory hay không
