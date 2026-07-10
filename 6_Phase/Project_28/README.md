# 🤖 Bài 28: Reference Scene Revisit Detector with Visual Memory Graph — Phát hiện robot đang quay lại cảnh đã từng thấy bằng đồ thị trí nhớ thị giác

> Mini Project số 28 trong **Đợt 6**  
> **Bài 28 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5 + Đợt 6**.  
> Nếu **Bài 27** đã xây được **Feature Match Graph for Multi-Frame Visual Memory**, thì **Bài 28** sẽ dùng chính graph đó để làm một bài toán “robotic” hơn:
>
> **robot quan sát một frame hiện tại và phải quyết định xem nó có đang quay lại một cảnh đã từng thấy trong memory hay không.**

---

# 📌 Mục lục

- [1. Bài 28 lấy gì từ Đợt 6](#1-bài-28-lấy-gì-từ-đợt-6)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 28 nằm ở đâu trong roadmap](#3-bài-28-nằm-ở-đâu-trong-roadmap)
- [4. Từ Bài 27 sang Bài 28 đã nâng cấp gì](#4-từ-bài-27-sang-bài-28-đã-nâng-cấp-gì)
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

# 1. Bài 28 lấy gì từ Đợt 6

Theo roadmap, **Đợt 6 (Ngày 11–12)** vẫn xoay quanh **Feature extractor project** với 3 khối chính:

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

Bài 28 vẫn giữ nguyên lõi này, nhưng thay vì chỉ **xây graph**, nó dùng graph đó để làm một bài toán có ý nghĩa hơn với robot:

```text
revisit detection / visual relocalization sơ cấp
```

Roadmap Đợt 6 nhấn mạnh **feature extractor project** và cách tổ chức thành project có cấu trúc fileciteturn9file0

---

# 2. Mô tả

Sau Bài 27, robot đã có một **visual memory graph**:

- mỗi frame cũ là một node
- giữa các frame có edge score dựa trên:
  - good matches
  - homography
  - inlier ratio

Nhưng graph đó mới chỉ là **memory thô**.  
Bài 28 sẽ thêm một tình huống perception thực tế hơn:

> robot vừa nhận một **current frame** mới, và muốn biết:
>
> - frame này có giống cảnh nào trong memory không?
> - nếu có, nó giống **frame nào nhất**?
> - mức độ tin cậy là bao nhiêu?
> - có đủ mạnh để nói robot **đang revisit một scene cũ** không?

Bài 28 vì vậy sẽ xây một **Reference Scene Revisit Detector**.

---

# 3. Bài 28 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**
- **Đợt 6 = Bài 26 → 30**

Vì vậy:

## **Bài 28 = bài thứ ba của Đợt 6**

và nó phải kết hợp lại toàn bộ nền từ **Đợt 1 → Đợt 6**.

---

# 4. Từ Bài 27 sang Bài 28 đã nâng cấp gì

## Bài 27
Bài 27 chủ yếu là:

```text
memory graph builder
```

- load nhiều frame
- build edge giữa các frame
- tạo graph report

## Bài 28
Bài 28 chuyển sang:

```text
graph-based query detector
```

- đã có memory frames
- có current query frame mới
- match query với memory frames
- chọn best memory frame
- kiểm tra score + inlier ratio + homography
- kết luận:
  - `REVISIT_DETECTED`
  - `WEAK_REVISIT`
  - `NO_REVISIT`

Nói ngắn gọn:

```text
Bài 27 = xây trí nhớ thị giác
Bài 28 = dùng trí nhớ đó để nhận ra robot đang quay lại đâu
```

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu luồng:

```text
Memory Frame Set + Query Frame Set + Revisit Config
→ extract features for all memory frames
→ extract features for each query frame
→ match query ↔ từng memory frame
→ estimate homography
→ compute revisit score
→ choose best memory candidate
→ classify revisit state
→ save visualization + revisit report
```

---

# 6. Pipeline perception của bài

```text
Python Config Builder
→ build memory frame manifest
→ build query frame manifest
→ build revisit detector config

C++ Runtime
→ load memory frame manifest
→ load query frame manifest
→ load revisit config
→ initialize extractor

for each memory frame:
    → load image
    → extract features
    → cache memory feature record

for each query frame:
    → load query image
    → extract query features
    → for each memory frame:
         → match descriptors
         → filter good matches
         → estimate homography
         → compute inlier ratio
         → compute revisit score
    → choose best memory candidate
    → classify revisit state
    → save best-match visualization
    → save query result record

after all queries:
    → write revisit report
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- dict / list
- config builder

## C++
- class / inheritance
- `virtual` / `override`
- pass by reference
- pointer cơ bản
- vector / struct / map
- destructor
- static member

## Computer Vision
- feature extraction
- descriptor matching
- good match filtering
- homography
- inlier ratio
- match scoring
- query-to-memory comparison

---

# 8. Đợt 1 → Đợt 6 được dùng như thế nào

## Đợt 1
- class / function / if else / loop
- đọc ảnh

## Đợt 2
- list / dict / vector
- quản lý memory frame list và query frame list

## Đợt 3
- nested loop query ↔ memory
- menu / điều kiện phân loại revisit

## Đợt 4
- file handling
- exception
- report

## Đợt 5
- feature detection / homography / registration mindset
- quality report

## Đợt 6
- module Python
- pointer / virtual / override
- feature workbench / visual memory graph mindset

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 28, bạn phải nắm được 12 ý:

## 1. Visual memory chỉ thật sự hữu ích khi robot có thể query lại nó
Bài 27 mới xây graph, Bài 28 mới bắt đầu “dùng” graph.

## 2. Query frame cần so với nhiều memory frames
Không chỉ so với 1 frame duy nhất.

## 3. “Best match” không nên chỉ dựa vào total matches
Cần kết hợp:
- good matches
- inlier ratio
- homography validity

## 4. Revisit detection là bước nhập môn của visual relocalization
Nó chưa phải SLAM hoàn chỉnh, nhưng tư duy rất gần.

## 5. Cần phân biệt:
- `REVISIT_DETECTED`
- `WEAK_REVISIT`
- `NO_REVISIT`

## 6. Caching feature của memory frames rất quan trọng
Nếu mỗi query lại extract toàn bộ memory thì runtime sẽ nặng.

## 7. Một memory system nên tách:
- **memory building**
- **memory querying**

## 8. Python rất hợp để tạo query manifest / memory manifest / detector config
C++ phù hợp hơn cho runtime matching.

## 9. Bài này rất gần với robot “đi lại hành lang / căn phòng cũ”
Robot nhìn một frame hiện tại và hỏi: “mình đã ở đây chưa?”

## 10. Đây là nền rất tốt cho loop closure / relocalization / scene recall
Đặc biệt nếu sau này bạn học SLAM hoặc navigation perception.

## 11. `virtual` / `override` giúp sau này thay detector bằng pipeline mạnh hơn
Ví dụ ORB → SIFT hoặc thêm geometric verification mạnh hơn.

## 12. Đây là bước tiến rất tự nhiên sau Bài 27
Từ **build memory graph** sang **query memory graph**.

---

# 10. Cấu trúc folder

```text
mini_project_28_reference_scene_revisit_detector_with_visual_memory_graph/
│
├─ README.md
│
├─ assets/
│  ├─ memory_frames/
│  │  ├─ memory_01.jpg
│  │ ├─ memory_02.jpg
│  │ ├─ memory_03.jpg
│  │ └─ ...
│  │
│  ├─ query_frames/
│  │ ├─ query_01.jpg
│  │ ├─ query_02.jpg
│  │ ├─ query_03.jpg
│  │ └─ ...
│  │
│  └─ outputs/
│     ├─ query_01_best_match.jpg
│     ├─ query_02_best_match.jpg
│     ├─ query_03_best_match.jpg
│     ├─ revisit_report.txt
│     └─ query_summary.txt
│
├─ config/
│  ├─ memory_frame_manifest.txt
│  ├─ query_frame_manifest.txt
│  └─ revisit_detector_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ revisit_report_template.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ MemoryFrameRecord.hpp
   │  ├─ QueryFrameRecord.hpp
   │  ├─ RevisitDetectorConfig.hpp
   │  ├─ FrameFeatureRecord.hpp
   │  ├─ RevisitCandidateRecord.hpp
   │  ├─ QueryRevisitResult.hpp
   │  ├─ BaseFeatureExtractor.hpp
   │  ├─ ORBFeatureExtractor.hpp
   │  ├─ BaseRevisitDetector.hpp
   │  ├─ VisualMemoryRevisitDetector.hpp
   │  └─ RevisitReportWriter.hpp
   │
   └─ src/
      ├─ ORBFeatureExtractor.cpp
      ├─ VisualMemoryRevisitDetector.cpp
      └─ RevisitReportWriter.cpp
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

## Thuộc tính
```python
project_name
memory_frame_manifest_path
query_frame_manifest_path
revisit_detector_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in path config

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

### `@staticmethod validate_image_extension(path)`
- chấp nhận `.jpg`, `.jpeg`, `.png`

---

# 11.2 Python — `RevisitDetectorConfigBuilder`

Tạo class con:

```python
class RevisitDetectorConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
memory_frames
query_frames
revisit_config
```

## `revisit_config` ví dụ

```python
{
    "feature_type": "ORB",
    "max_features": 1500,
    "match_mode": "BF_HAMMING",
    "good_match_distance_threshold": 45.0,
    "min_good_matches_for_revisit": 20,
    "ransac_reproj_threshold": 4.0,
    "min_inlier_ratio_for_revisit": 0.35,
    "strong_revisit_score_threshold": 0.72,
    "weak_revisit_score_threshold": 0.45,
    "save_best_match_visualization": True
}
```

## Hàm cần có

### `add_memory_frame(frame_name, image_path)`
- thêm memory frame

### `add_query_frame(frame_name, image_path)`
- thêm query frame

### `set_revisit_config(...)`
**Hành vi**
- `feature_type` thuộc `"ORB"` hoặc `"SIFT"`
- `max_features > 0`
- `good_match_distance_threshold > 0`
- `min_good_matches_for_revisit >= 4`
- `ransac_reproj_threshold > 0`
- `0 <= min_inlier_ratio_for_revisit <= 1`
- `strong_revisit_score_threshold >= weak_revisit_score_threshold`

### `write_memory_frame_manifest()`
Format gợi ý:

```text
frame_name=memory_01
image_path=assets/memory_frames/memory_01.jpg
```

### `write_query_frame_manifest()`
Format gợi ý:

```text
frame_name=query_01
image_path=assets/query_frames/query_01.jpg
```

### `write_revisit_detector_config()`

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **6 memory frames**
- tạo ít nhất **4 query frames**
- set revisit config
- ghi:
  - `config/memory_frame_manifest.txt`
  - `config/query_frame_manifest.txt`
  - `config/revisit_detector_config.txt`

---

# 11.4 C++ — `MemoryFrameRecord`

**File:**
```text
cpp/include/MemoryFrameRecord.hpp
```

```cpp
struct MemoryFrameRecord
{
    std::string frame_name;
    std::string image_path;
};
```

---

# 11.5 C++ — `QueryFrameRecord`

**File:**
```text
cpp/include/QueryFrameRecord.hpp
```

```cpp
struct QueryFrameRecord
{
    std::string frame_name;
    std::string image_path;
};
```

---

# 11.6 C++ — `RevisitDetectorConfig`

**File:**
```text
cpp/include/RevisitDetectorConfig.hpp
```

```cpp
struct RevisitDetectorConfig
{
    std::string feature_type;
    int max_features;
    std::string match_mode;
    double good_match_distance_threshold;
    int min_good_matches_for_revisit;
    double ransac_reproj_threshold;
    double min_inlier_ratio_for_revisit;
    double strong_revisit_score_threshold;
    double weak_revisit_score_threshold;
    bool save_best_match_visualization;
};
```

---

# 11.7 C++ — `FrameFeatureRecord`

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

# 11.8 C++ — `RevisitCandidateRecord`

**File:**
```text
cpp/include/RevisitCandidateRecord.hpp
```

```cpp
struct RevisitCandidateRecord
{
    std::string query_frame_name;
    std::string memory_frame_name;

    int total_matches;
    int good_matches;
    int inlier_matches;

    double inlier_ratio;
    double revisit_score;

    bool homography_valid;
    bool candidate_valid;
};
```

---

# 11.9 C++ — `QueryRevisitResult`

**File:**
```text
cpp/include/QueryRevisitResult.hpp
```

```cpp
struct QueryRevisitResult
{
    std::string query_frame_name;
    std::string best_memory_frame_name;

    double best_revisit_score;
    double best_inlier_ratio;
    int best_good_matches;

    std::string revisit_state; // REVISIT_DETECTED / WEAK_REVISIT / NO_REVISIT
    bool is_valid;
};
```

---

# 11.10 C++ — `BaseFeatureExtractor`

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

# 11.11 C++ — `ORBFeatureExtractor`

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

---

# 11.12 C++ — `BaseRevisitDetector`

**File:**
```text
cpp/include/BaseRevisitDetector.hpp
```

```cpp
class BaseRevisitDetector
{
public:
    virtual void load_memory_frame_manifest(const std::string& path) = 0;
    virtual void load_query_frame_manifest(const std::string& path) = 0;
    virtual void load_revisit_config(const std::string& path) = 0;
    virtual void run_revisit_detection() = 0;
    virtual ~BaseRevisitDetector() = default;
};
```

---

# 11.13 C++ — `VisualMemoryRevisitDetector`

**File:**
```text
cpp/include/VisualMemoryRevisitDetector.hpp
cpp/src/VisualMemoryRevisitDetector.cpp
```

Class kế thừa:

```cpp
class VisualMemoryRevisitDetector : public BaseRevisitDetector
```

## Thuộc tính cần có

```cpp
private:
    std::vector<MemoryFrameRecord> memory_frames;
    std::vector<QueryFrameRecord> query_frames;
    RevisitDetectorConfig config;

    BaseFeatureExtractor* extractor;

    std::vector<FrameFeatureRecord> memory_feature_cache;
    std::vector<QueryRevisitResult> query_results;

    static int detector_instance_count;
```

## Constructor / Destructor
```cpp
VisualMemoryRevisitDetector();
~VisualMemoryRevisitDetector();
```

### Destructor phải làm gì?
- `delete extractor` nếu khác `nullptr`
- log số query đã xử lý

---

## Hàm load
- `load_memory_frame_manifest(...)`
- `load_query_frame_manifest(...)`
- `load_revisit_config(...)`

---

## Hàm khởi tạo extractor

### `initialize_extractor()`
- nếu ORB → tạo ORB extractor
- nếu chưa hỗ trợ SIFT thì log rõ

---

## Hàm cache memory features

### `cache_memory_features()`
- load từng memory frame
- extract feature
- lưu `memory_feature_cache`

---

## Hàm matching / scoring

### `std::vector<cv::DMatch> match_descriptors(
    const cv::Mat& descriptors_a,
    const cv::Mat& descriptors_b
) const;`

### `std::vector<cv::DMatch> filter_good_matches(
    const std::vector<cv::DMatch>& matches
) const;`

### `RevisitCandidateRecord build_candidate_record(
    const FrameFeatureRecord& query_feature,
    const FrameFeatureRecord& memory_feature
) const;`

## Hành vi của `build_candidate_record(...)`
1. match descriptors
2. filter good matches
3. nếu good matches < `min_good_matches_for_revisit`
   - candidate fail
4. nếu đủ matches:
   - build point correspondences
   - `findHomography(..., RANSAC, ransac_reproj_threshold, mask)`
   - tính `inlier_matches`
   - tính `inlier_ratio`
   - tính `revisit_score`

### Gợi ý công thức `revisit_score`
Ví dụ:

```text
revisit_score = 0.5 * normalized_good_matches + 0.5 * inlier_ratio
```

### Candidate hợp lệ khi:
- `good_matches >= min_good_matches_for_revisit`
- homography valid

---

## Hàm chọn best candidate

### `QueryRevisitResult build_query_result(
    const std::string& query_frame_name,
    const std::vector<RevisitCandidateRecord>& candidates
) const;`

## Rule gợi ý
- nếu không có candidate hợp lệ → `NO_REVISIT`
- nếu `best_revisit_score >= strong_revisit_score_threshold`
  → `REVISIT_DETECTED`
- nếu `weak_revisit_score_threshold <= best_revisit_score < strong_revisit_score_threshold`
  → `WEAK_REVISIT`
- còn lại → `NO_REVISIT`

---

## Visualization

### `cv::Mat build_best_match_visualization(
    const FrameFeatureRecord& query_feature,
    const FrameFeatureRecord& memory_feature,
    const std::vector<cv::DMatch>& good_matches
) const;`

### `save_query_best_match_visualization(...)`
- lưu ảnh match của best candidate nếu config cho phép

---

## Process từng query

### `void process_single_query(const QueryFrameRecord& query);`
1. load query image
2. extract query features
3. loop qua toàn bộ memory_feature_cache
4. build candidate cho từng memory frame
5. build `QueryRevisitResult`
6. save best visualization nếu có
7. push query result

---

## Hàm chính

### `void run_revisit_detection() override;`
1. `initialize_extractor()`
2. `cache_memory_features()`
3. loop toàn bộ query frames

### Getter

```cpp
const std::vector<QueryRevisitResult>& get_query_results() const;
```

---

# 11.14 C++ — `RevisitReportWriter`

**File:**
```text
cpp/include/RevisitReportWriter.hpp
cpp/src/RevisitReportWriter.cpp
```

Tạo class:

```cpp
class RevisitReportWriter
```

## Hàm cần có

### `write_revisit_report(
    const std::string& path,
    const std::vector<QueryRevisitResult>& query_results
);`

Format gợi ý:

```text
[Query Revisit Result]
Query Frame: query_01
Best Memory Frame: memory_03
Best Revisit Score: 0.78
Best Inlier Ratio: 0.66
Best Good Matches: 118
Revisit State: REVISIT_DETECTED
Valid: true
----------------------------------------
```

### `write_query_summary(
    const std::string& path,
    const std::vector<QueryRevisitResult>& query_results
);`

Ví dụ:

```text
query_01 -> memory_03 -> REVISIT_DETECTED
query_02 -> memory_05 -> WEAK_REVISIT
query_03 -> none -> NO_REVISIT
```

---

# 11.15 C++ — `main.cpp`

## Yêu cầu
- tạo `VisualMemoryRevisitDetector`
- load:
  - `config/memory_frame_manifest.txt`
  - `config/query_frame_manifest.txt`
  - `config/revisit_detector_config.txt`
- chạy `run_revisit_detection()`
- tạo `RevisitReportWriter`
- ghi:
  - `assets/outputs/revisit_report.txt`
  - `assets/outputs/query_summary.txt`

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
- query-to-memory comparison
- revisit report

---

# 13. Output mong muốn

## File config

```text
config/memory_frame_manifest.txt
config/query_frame_manifest.txt
config/revisit_detector_config.txt
```

## Ảnh output

```text
assets/outputs/query_01_best_match.jpg
assets/outputs/query_02_best_match.jpg
assets/outputs/query_03_best_match.jpg
```

## File report

```text
assets/outputs/revisit_report.txt
assets/outputs/query_summary.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Revisit Detector Config Builder
```

Dùng để tạo:
- memory frame manifest
- query frame manifest
- revisit detector config

## C++ đóng vai trò gì?

C++ là:

```text
Visual Memory Query Runtime
```

Dùng để:
- cache memory features
- match query với memory
- tính revisit score
- quyết định robot có đang quay lại scene cũ không

## Computer Vision đóng vai trò gì?

CV là:

```text
query frame
→ feature extraction
→ compare with memory frames
→ homography validation
→ revisit decision
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo 3 file config
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có module + `if __name__ == "__main__"`
- [ ] C++ có `BaseFeatureExtractor`
- [ ] C++ có `ORBFeatureExtractor`
- [ ] C++ có `BaseRevisitDetector`
- [ ] C++ có `VisualMemoryRevisitDetector`
- [ ] C++ có pointer extractor
- [ ] C++ cache memory features
- [ ] C++ process từng query frame
- [ ] C++ build revisit candidate
- [ ] C++ chọn best memory frame
- [ ] C++ phân loại revisit state
- [ ] C++ ghi report

---

# 16. Gợi ý mở rộng

## 1. Thêm temporal filtering
Nếu query liên tiếp đều match về cùng một memory frame, tăng confidence.

## 2. Thêm top-k revisit candidates
Không chỉ giữ best 1, mà giữ top 3 memory frames.

## 3. Thêm pose / timestamp metadata
Để revisit detection có thêm ngữ cảnh robot.

## 4. Chuẩn bị cho Bài 29
Sau Bài 28, bước rất hợp lý là:

```text
Scene Cluster Builder from Visual Memory Graph
```

Tức là từ memory graph + revisit results, robot bắt đầu gom frame thành:
- cluster “bàn ăn”
- cluster “bàn làm việc”
- cluster “cửa ra vào”
- cluster “hành lang”
