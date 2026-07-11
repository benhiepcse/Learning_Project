# 🤖 Bài 30: Cluster-Aware Visual Relocalization Query Engine — Truy vấn relocalization theo scene cluster cho Humanoid Robot AI Perception

> Mini Project số 30 trong **Đợt 6**  
> **Bài 30 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5 + Đợt 6**.  
> Đây là **bài chốt của Đợt 6**. Nếu **Bài 29** đã giúp robot tổ chức visual memory thành **scene clusters**, thì **Bài 30** sẽ dùng chính các cluster đó để thực hiện một truy vấn perception thực tế hơn:
>
> **khi robot nhận một query frame mới, nó sẽ không còn so “phẳng” với toàn bộ memory frames nữa, mà sẽ relocalize theo 2 tầng:**
>
> ```text
> query frame
> → chọn cluster phù hợp nhất
> → chọn representative / best frame trong cluster
> → kết luận relocalization
> ```

---

# 📌 Mục lục

- [1. Bài 30 lấy gì từ Đợt 6](#1-bài-30-lấy-gì-từ-đợt-6)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 30 nằm ở đâu trong roadmap](#3-bài-30-nằm-ở-đâu-trong-roadmap)
- [4. Từ Bài 29 sang Bài 30 đã nâng cấp gì](#4-từ-bài-29-sang-bài-30-đã-nâng-cấp-gì)
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
- [16. Sau Bài 30 nên đi tiếp như thế nào](#16-sau-bài-30-nên-đi-tiếp-như-thế-nào)

---

# 1. Bài 30 lấy gì từ Đợt 6

Theo roadmap, **Đợt 6** vẫn là cụm **Feature extractor project** và cấu trúc project perception, với ba khối nền:

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

Bài 30 không thêm “môn mới”, mà **gom và nâng cấp** toàn bộ những gì đã học ở Bài 26 → 29:

- feature extraction
- feature matching
- homography verification
- visual memory graph
- revisit detection
- scene clustering

để tạo ra một **query engine relocalization có awareness về cluster**.

---

# 2. Mô tả

Sau **Bài 29**, robot đã có thể:

- build visual memory graph từ nhiều memory frames
- gom memory thành các **scene clusters**
- chọn representative frame cho mỗi cluster

Nhưng nếu robot nhận một **query frame mới**, và muốn biết:

- “Mình đang ở cluster scene nào?”
- “Trong cluster đó, frame nào gần nhất với query?”
- “Mức độ tin cậy relocalization là bao nhiêu?”
- “Có nên kết luận robot đã quay lại một vùng đã biết không?”

thì ta cần một tầng mới: **Cluster-Aware Visual Relocalization Query Engine**.

Điểm quan trọng là engine này **không so query với toàn bộ memory frames một cách phẳng**, mà đi theo 2 bước:

## Bước 1 — Cluster retrieval
- query frame so với **representative frames** hoặc **cluster prototypes**
- chọn cluster phù hợp nhất

## Bước 2 — In-cluster relocalization
- chỉ so query với các frame trong cluster tốt nhất
- chọn best frame cuối cùng
- kết luận relocalization

<p align="center">
  <img src="../../images/project_30.png" width="800">
</p>

---

# 3. Bài 30 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**
- **Đợt 6 = Bài 26 → 30**

Vì vậy:

## **Bài 30 = bài chốt của Đợt 6**

và phải tổng hợp lại toàn bộ nền từ **Đợt 1 → Đợt 6**.

---

# 4. Từ Bài 29 sang Bài 30 đã nâng cấp gì

## Bài 29
Bài 29 thiên về:

```text
memory organization
```

- build edges
- build clusters
- representative frame
- cluster report

## Bài 30
Bài 30 thiên về:

```text
memory-aware query engine
```

- có cluster memory rồi
- query frame mới đi vào engine
- engine chọn **cluster tốt nhất**
- rồi mới chọn **frame tốt nhất trong cluster**
- xuất **relocalization result**

Tức là:

```text
Bài 29 = robot sắp xếp trí nhớ
Bài 30 = robot dùng trí nhớ đã sắp xếp đó để định vị lại bằng thị giác
```

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu luồng:

```text
Memory Frames + Scene Cluster Memory + Query Frames + Relocalization Config
→ build / load cluster memory
→ extract query feature
→ compare query with cluster representatives
→ choose best cluster
→ compare query with frames inside chosen cluster
→ choose best frame
→ compute relocalization confidence
→ export relocalization report
```

---

# 6. Pipeline perception của bài

```text
Python Config Builder
→ build memory frame manifest
→ build query frame manifest
→ build cluster-aware relocalization config

C++ Runtime
→ load memory frames
→ load query frames
→ load relocalization config
→ initialize extractor
→ extract / cache memory features
→ build accepted edges
→ build scene clusters
→ choose representative frame for each cluster

for each query frame:
    → load query image
    → extract query features

    # Stage 1: cluster retrieval
    → compare query with representative frame of each cluster
    → compute cluster candidate scores
    → choose best cluster

    # Stage 2: in-cluster relocalization
    → compare query with all frames inside chosen cluster
    → compute frame candidate scores
    → choose best frame

    → classify relocalization state
    → save best visualization
    → store result

write final relocalization report
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
- vector / map / set / queue
- struct
- destructor
- static member

## Computer Vision
- feature extraction
- descriptor matching
- good match filtering
- homography
- inlier ratio
- graph-based memory
- scene clustering
- query relocalization

---

# 8. Đợt 1 → Đợt 6 được dùng như thế nào

## Đợt 1
- class / function / if else / loop
- đọc ảnh

## Đợt 2
- list / dict / vector
- quản lý memory frame list và query frame list

## Đợt 3
- nested loop cho frame pair và query-frame comparisons

## Đợt 4
- file handling
- exception
- report

## Đợt 5
- feature matching / homography / registration mindset

## Đợt 6
- feature workbench
- visual memory graph
- revisit detector
- scene clustering
- cluster-aware relocalization

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 30, bạn phải nắm được 14 ý:

## 1. Query-to-all-memory là baseline, nhưng query-to-cluster là cấu trúc tốt hơn
Nó giúp memory có tổ chức hơn.

## 2. Representative frame là cầu nối giữa “cluster memory” và “query engine”
Không có representative, cluster sẽ khó query hiệu quả.

## 3. Relocalization có thể tách thành 2 tầng:
- cluster retrieval
- frame retrieval

## 4. Đây là tư duy rất gần với hierarchical retrieval trong robot perception
Từ coarse → fine.

## 5. Cluster-aware query giảm số lượng frame cần so trực tiếp
Nó phù hợp hơn khi memory lớn dần.

## 6. Revisit detection ở Bài 28 là một dạng “flat memory query”
Bài 30 là bước nâng cấp tự nhiên.

## 7. Cluster memory không chỉ để đẹp report
Nó phải phục vụ truy vấn.

## 8. Một relocalization result tốt nên chứa:
- best cluster
- best frame
- cluster score
- frame score
- relocalization state

## 9. Homography vẫn là bước geometric verification quan trọng
Không chỉ descriptor matching.

## 10. Python tiếp tục phù hợp để build config / manifests
C++ tiếp tục phù hợp để chạy runtime.

## 11. Đây là bài chốt đẹp của Đợt 6
Vì nó gom:
- feature extraction
- matching
- homography
- graph memory
- revisit detection
- clustering
- relocalization query

## 12. Sau Bài 30, bạn đã có một nhánh visual memory khá tròn
Từ pair matching → graph → revisit → cluster → relocalization.

## 13. Đây là nền tốt cho visual place recognition / loop closure / robot memory
Dù mới ở mức mini-project.

## 14. Nếu sau này gắn thêm depth / pose / ROS2 thì nhánh này sẽ rất mạnh
Vì bạn đã có cấu trúc perception memory khá rõ.

---

# 10. Cấu trúc folder

```text
mini_project_30_cluster_aware_visual_relocalization_query_engine/
│
├─ README.md
│
├─ assets/
│  ├─ memory_frames/
│  │  ├─ memory_01.jpg
│  │  ├─ memory_02.jpg
│  │  ├─ memory_03.jpg
│  │  └─ ...
│  │
│  ├─ query_frames/
│  │  ├─ query_01.jpg
│  │  ├─ query_02.jpg
│  │  ├─ query_03.jpg
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ query_01_best_cluster_match.jpg
│     ├─ query_01_best_frame_match.jpg
│     ├─ query_02_best_cluster_match.jpg
│     ├─ query_02_best_frame_match.jpg
│     ├─ relocalization_report.txt
│     └─ relocalization_summary.txt
│
├─ config/
│  ├─ memory_frame_manifest.txt
│  ├─ query_frame_manifest.txt
│  └─ cluster_relocalization_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ relocalization_report_template.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ MemoryFrameRecord.hpp
   │  ├─ QueryFrameRecord.hpp
   │  ├─ ClusterRelocalizationConfig.hpp
   │  ├─ FrameFeatureRecord.hpp
   │  ├─ MatchEdgeRecord.hpp
   │  ├─ ClusterRecord.hpp
   │  ├─ ClusterCandidateRecord.hpp
   │  ├─ FrameRelocalizationCandidate.hpp
   │  ├─ QueryRelocalizationResult.hpp
   │  ├─ BaseFeatureExtractor.hpp
   │  ├─ ORBFeatureExtractor.hpp
   │  ├─ BaseClusterAwareRelocalizer.hpp
   │  ├─ ClusterAwareRelocalizer.hpp
   │  └─ RelocalizationReportWriter.hpp
   │
   └─ src/
      ├─ ORBFeatureExtractor.cpp
      ├─ ClusterAwareRelocalizer.cpp
      └─ RelocalizationReportWriter.cpp
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
cluster_relocalization_config_path
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

# 11.2 Python — `ClusterRelocalizationConfigBuilder`

Tạo class con:

```python
class ClusterRelocalizationConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
memory_frames
query_frames
cluster_relocalization_config
```

## `cluster_relocalization_config` ví dụ

```python
{
    "feature_type": "ORB",
    "max_features": 1500,
    "match_mode": "BF_HAMMING",
    "good_match_distance_threshold": 45.0,
    "min_good_matches_for_edge": 18,
    "ransac_reproj_threshold": 4.0,
    "min_inlier_ratio_for_edge": 0.30,
    "min_edge_score_for_cluster": 0.50,

    "min_good_matches_for_query": 20,
    "min_inlier_ratio_for_query": 0.35,
    "strong_relocalization_score_threshold": 0.72,
    "weak_relocalization_score_threshold": 0.45,

    "save_best_cluster_match_visualization": True,
    "save_best_frame_match_visualization": True
}
```

## Hàm cần có

### `add_memory_frame(frame_name, image_path)`
- thêm memory frame

### `add_query_frame(frame_name, image_path)`
- thêm query frame

### `set_cluster_relocalization_config(...)`
**Hành vi**
- validate toàn bộ threshold > 0 nếu là ngưỡng số lượng / reprojection
- các tỉ lệ nằm trong `[0, 1]`
- `strong_relocalization_score_threshold >= weak_relocalization_score_threshold`

### `write_memory_frame_manifest()`
### `write_query_frame_manifest()`
### `write_cluster_relocalization_config()`

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **8 memory frames**
- tạo ít nhất **4 query frames**
- set config relocalization
- ghi:
  - `config/memory_frame_manifest.txt`
  - `config/query_frame_manifest.txt`
  - `config/cluster_relocalization_config.txt`

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

# 11.6 C++ — `ClusterRelocalizationConfig`

**File:**
```text
cpp/include/ClusterRelocalizationConfig.hpp
```

```cpp
struct ClusterRelocalizationConfig
{
    std::string feature_type;
    int max_features;
    std::string match_mode;

    double good_match_distance_threshold;
    int min_good_matches_for_edge;
    double ransac_reproj_threshold;
    double min_inlier_ratio_for_edge;
    double min_edge_score_for_cluster;

    int min_good_matches_for_query;
    double min_inlier_ratio_for_query;
    double strong_relocalization_score_threshold;
    double weak_relocalization_score_threshold;

    bool save_best_cluster_match_visualization;
    bool save_best_frame_match_visualization;
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

# 11.8 C++ — `MatchEdgeRecord`

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

# 11.9 C++ — `ClusterRecord`

**File:**
```text
cpp/include/ClusterRecord.hpp
```

```cpp
struct ClusterRecord
{
    int cluster_id;
    std::vector<std::string> member_frames;
    std::string representative_frame;
    int cluster_size;
    double average_internal_edge_score;
};
```

---

# 11.10 C++ — `ClusterCandidateRecord`

**File:**
```text
cpp/include/ClusterCandidateRecord.hpp
```

```cpp
struct ClusterCandidateRecord
{
    std::string query_frame_name;
    int cluster_id;
    std::string representative_frame_name;

    int good_matches;
    int inlier_matches;
    double inlier_ratio;
    double cluster_score;

    bool homography_valid;
    bool candidate_valid;
};
```

---

# 11.11 C++ — `FrameRelocalizationCandidate`

**File:**
```text
cpp/include/FrameRelocalizationCandidate.hpp
```

```cpp
struct FrameRelocalizationCandidate
{
    std::string query_frame_name;
    int cluster_id;
    std::string memory_frame_name;

    int good_matches;
    int inlier_matches;
    double inlier_ratio;
    double frame_score;

    bool homography_valid;
    bool candidate_valid;
};
```

---

# 11.12 C++ — `QueryRelocalizationResult`

**File:**
```text
cpp/include/QueryRelocalizationResult.hpp
```

```cpp
struct QueryRelocalizationResult
{
    std::string query_frame_name;

    int best_cluster_id;
    std::string best_cluster_representative_frame;
    double best_cluster_score;

    std::string best_memory_frame;
    double best_frame_score;
    double best_frame_inlier_ratio;
    int best_frame_good_matches;

    std::string relocalization_state; // RELOCALIZED / WEAK_RELOCALIZATION / NO_RELOCALIZATION
    bool is_valid;
};
```

---

# 11.13 C++ — `BaseFeatureExtractor`

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

# 11.14 C++ — `ORBFeatureExtractor`

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

# 11.15 C++ — `BaseClusterAwareRelocalizer`

**File:**
```text
cpp/include/BaseClusterAwareRelocalizer.hpp
```

```cpp
class BaseClusterAwareRelocalizer
{
public:
    virtual void load_memory_frame_manifest(const std::string& path) = 0;
    virtual void load_query_frame_manifest(const std::string& path) = 0;
    virtual void load_cluster_relocalization_config(const std::string& path) = 0;
    virtual void run_cluster_aware_relocalization() = 0;
    virtual ~BaseClusterAwareRelocalizer() = default;
};
```

---

# 11.16 C++ — `ClusterAwareRelocalizer`

**File:**
```text
cpp/include/ClusterAwareRelocalizer.hpp
cpp/src/ClusterAwareRelocalizer.cpp
```

Class kế thừa:

```cpp
class ClusterAwareRelocalizer : public BaseClusterAwareRelocalizer
```

## Thuộc tính cần có

```cpp
private:
    std::vector<MemoryFrameRecord> memory_frames;
    std::vector<QueryFrameRecord> query_frames;
    ClusterRelocalizationConfig config;

    BaseFeatureExtractor* extractor;

    std::vector<FrameFeatureRecord> memory_feature_cache;
    std::vector<MatchEdgeRecord> edges;
    std::vector<ClusterRecord> clusters;
    std::vector<QueryRelocalizationResult> query_results;

    static int relocalizer_instance_count;
```

## Constructor / Destructor
```cpp
ClusterAwareRelocalizer();
~ClusterAwareRelocalizer();
```

### Destructor phải làm gì?
- `delete extractor` nếu khác `nullptr`
- log số query đã xử lý

---

## Nhóm hàm load
- `load_memory_frame_manifest(...)`
- `load_query_frame_manifest(...)`
- `load_cluster_relocalization_config(...)`

---

## Nhóm hàm feature / memory building

### `initialize_extractor()`
- ORB → tạo extractor

### `cache_memory_features()`
- load toàn bộ memory frames
- extract feature
- lưu cache

### `MatchEdgeRecord build_edge_record(
    const FrameFeatureRecord& frame_a,
    const FrameFeatureRecord& frame_b
) const;`

### `build_memory_edges()`
- loop qua các memory frames
- build accepted edges

### `build_scene_clusters()`
- tương tự tư duy Bài 29:
  - adjacency list
  - connected components
  - representative frame

---

## Nhóm hàm query matching

### `ClusterCandidateRecord build_cluster_candidate(
    const FrameFeatureRecord& query_feature,
    const ClusterRecord& cluster
) const;`

**Ý tưởng**
- query so với `representative_frame` của cluster
- tính:
  - good matches
  - inlier ratio
  - cluster_score

### `FrameRelocalizationCandidate build_frame_candidate(
    const FrameFeatureRecord& query_feature,
    int cluster_id,
    const FrameFeatureRecord& memory_feature
) const;`

**Ý tưởng**
- query so với một frame cụ thể trong cluster
- tính frame score

---

## Nhóm hàm chọn kết quả

### `int choose_best_cluster(
    const std::vector<ClusterCandidateRecord>& cluster_candidates
) const;`

### `QueryRelocalizationResult build_query_result(
    const std::string& query_frame_name,
    const ClusterCandidateRecord& best_cluster_candidate,
    const std::vector<FrameRelocalizationCandidate>& frame_candidates
) const;`

## Rule gợi ý cho `relocalization_state`
- nếu không có cluster candidate hợp lệ → `NO_RELOCALIZATION`
- nếu không có frame candidate hợp lệ → `NO_RELOCALIZATION`
- nếu `best_frame_score >= strong_relocalization_score_threshold`
  → `RELOCALIZED`
- nếu `weak_relocalization_score_threshold <= best_frame_score < strong_relocalization_score_threshold`
  → `WEAK_RELOCALIZATION`
- còn lại → `NO_RELOCALIZATION`

---

## Visualization

### `save_best_cluster_match_visualization(...)`
- query ↔ representative frame của cluster tốt nhất

### `save_best_frame_match_visualization(...)`
- query ↔ best frame cuối cùng

---

## Process từng query

### `void process_single_query(const QueryFrameRecord& query);`
1. load query image
2. extract query feature
3. build cluster candidates cho tất cả clusters
4. chọn best cluster
5. build frame candidates trong best cluster
6. build `QueryRelocalizationResult`
7. save visualizations
8. push query result

---

## Hàm chính

### `void run_cluster_aware_relocalization() override;`
1. `initialize_extractor()`
2. `cache_memory_features()`
3. `build_memory_edges()`
4. `build_scene_clusters()`
5. loop toàn bộ query frames

### Getter

```cpp
const std::vector<QueryRelocalizationResult>& get_query_results() const;
const std::vector<ClusterRecord>& get_clusters() const;
```

---

# 11.17 C++ — `RelocalizationReportWriter`

**File:**
```text
cpp/include/RelocalizationReportWriter.hpp
cpp/src/RelocalizationReportWriter.cpp
```

Tạo class:

```cpp
class RelocalizationReportWriter
```

## Hàm cần có

### `write_relocalization_report(
    const std::string& path,
    const std::vector<QueryRelocalizationResult>& results
);`

Format gợi ý:

```text
[Query Relocalization Result]
Query Frame: query_01
Best Cluster ID: 1
Best Cluster Representative: memory_05
Best Cluster Score: 0.74
Best Memory Frame: memory_06
Best Frame Score: 0.79
Best Frame Inlier Ratio: 0.68
Best Frame Good Matches: 121
Relocalization State: RELOCALIZED
Valid: true
----------------------------------------
```

### `write_relocalization_summary(
    const std::string& path,
    const std::vector<QueryRelocalizationResult>& results
);`

Ví dụ:

```text
query_01 -> cluster_1 -> memory_06 -> RELOCALIZED
query_02 -> cluster_0 -> memory_02 -> WEAK_RELOCALIZATION
query_03 -> none -> none -> NO_RELOCALIZATION
```

---

# 11.18 C++ — `main.cpp`

## Yêu cầu
- tạo `ClusterAwareRelocalizer`
- load:
  - `config/memory_frame_manifest.txt`
  - `config/query_frame_manifest.txt`
  - `config/cluster_relocalization_config.txt`
- chạy `run_cluster_aware_relocalization()`
- tạo `RelocalizationReportWriter`
- ghi:
  - `assets/outputs/relocalization_report.txt`
  - `assets/outputs/relocalization_summary.txt`

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
- scene clustering
- cluster-aware relocalization
- report

---

# 13. Output mong muốn

## File config

```text
config/memory_frame_manifest.txt
config/query_frame_manifest.txt
config/cluster_relocalization_config.txt
```

## Ảnh output

```text
assets/outputs/query_01_best_cluster_match.jpg
assets/outputs/query_01_best_frame_match.jpg
assets/outputs/query_02_best_cluster_match.jpg
assets/outputs/query_02_best_frame_match.jpg
```

## File report

```text
assets/outputs/relocalization_report.txt
assets/outputs/relocalization_summary.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Cluster-Aware Relocalization Config Builder
```

Dùng để tạo:
- memory frame manifest
- query frame manifest
- relocalization config

## C++ đóng vai trò gì?

C++ là:

```text
Cluster-Aware Visual Relocalization Runtime
```

Dùng để:
- build memory edges
- build scene clusters
- query cluster
- query frame trong cluster
- kết luận relocalization

## Computer Vision đóng vai trò gì?

CV là:

```text
memory frames + query frame
→ feature extraction
→ graph memory
→ cluster retrieval
→ in-cluster frame matching
→ relocalization
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo 3 file config
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có module + `if __name__ == "__main__"`
- [ ] C++ có `BaseFeatureExtractor`
- [ ] C++ có `ORBFeatureExtractor`
- [ ] C++ có `BaseClusterAwareRelocalizer`
- [ ] C++ có `ClusterAwareRelocalizer`
- [ ] C++ cache memory features
- [ ] C++ build memory edges
- [ ] C++ build scene clusters
- [ ] C++ build cluster candidates
- [ ] C++ build frame candidates
- [ ] C++ chọn best cluster
- [ ] C++ chọn best frame
- [ ] C++ ghi relocalization report

---

# 16. Sau Bài 30 nên đi tiếp như thế nào

Bài 30 là **điểm chốt rất đẹp của Đợt 6** vì bạn đã đi được trọn nhánh:

```text
Bài 26 → feature registration workbench
Bài 27 → visual memory graph
Bài 28 → revisit detector
Bài 29 → scene cluster builder
Bài 30 → cluster-aware visual relocalization
```

Sau đây là hướng đi hợp lý cho **Đợt 7** nếu bạn muốn tiếp tục bám sát **Humanoid Robot AI Perception**:

## Hướng 1 — Gắn depth vào visual memory
- stereo pair + visual memory
- 3D scene memory
- depth-aware relocalization

## Hướng 2 — Gắn ROS2 vào perception runtime
- biến visual memory / relocalization thành ROS2 nodes
- publish cluster / relocalization result

## Hướng 3 — Gắn scene understanding
- object-level cluster memory
- tabletop semantic memory
- object-aware relocalization

## Nếu bám sát robot perception nhất
Bài mở đầu hợp lý cho **Đợt 7** là:

```text
Stereo Landmark Database Builder for Humanoid Robot Perception
```

Tức là từ memory bằng **ảnh 2D + feature**, bạn nâng sang:
- landmark có **left-right correspondence**
- có **depth / 3D position estimate**
- dùng cho relocalization mạnh hơn
