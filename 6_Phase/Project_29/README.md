# 🤖 Bài 29: Scene Cluster Builder from Visual Memory Graph — Gom cụm scene từ visual memory graph cho Humanoid Robot AI Perception

> Mini Project số 29 trong **Đợt 6**  
> **Bài 29 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5 + Đợt 6**.  
> Nếu **Bài 27** giúp robot xây **visual memory graph**, **Bài 28** giúp robot **query frame hiện tại để phát hiện revisit**, thì **Bài 29** sẽ đi tiếp một bước rất tự nhiên:
>
> **robot bắt đầu gom các frame trong memory thành từng “scene cluster” — ví dụ cluster bàn làm việc, cluster bàn ăn, cluster hành lang, cluster cửa ra vào.**

---

# 📌 Mục lục

- [1. Bài 29 lấy gì từ Đợt 6](#1-bài-29-lấy-gì-từ-đợt-6)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 29 nằm ở đâu trong roadmap](#3-bài-29-nằm-ở-đâu-trong-roadmap)
- [4. Từ Bài 28 sang Bài 29 đã nâng cấp gì](#4-từ-bài-28-sang-bài-29-đã-nâng-cấp-gì)
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

# 1. Bài 29 lấy gì từ Đợt 6

Theo roadmap, **Đợt 6** vẫn nằm trong mạch **Feature extractor project** và tổ chức project perception theo kiểu:

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

Bài 29 không thêm CV “mới” theo kiểu segmentation hay stereo depth, mà dùng chính **feature matching + graph + revisit reasoning** để tạo ra một tầng mới:

```text
scene clustering / visual memory organization
```

Nói cách khác, Đợt 6 vẫn là “feature extractor project”, nhưng giờ feature đã được dùng để **tổ chức trí nhớ scene** chứ không chỉ match ảnh.

---

# 2. Mô tả

Sau **Bài 28**, robot đã có thể:

- lưu một tập memory frames
- so query frame với memory frames
- quyết định robot có đang quay lại một scene cũ hay không

Nhưng memory của robot vẫn đang ở dạng:

```text
frame rời rạc
+ graph edge
+ revisit result
```

Đó vẫn chưa phải là “scene memory” thật sự.  
Một tầng cao hơn là robot cần gom các frame **gần nhau về mặt thị giác** thành các **cluster scene**.

Ví dụ:

- `memory_01, memory_02, memory_03` → cùng cluster **desk_scene**
- `memory_04, memory_05` → cùng cluster **corridor_scene**
- `memory_06, memory_07, memory_08` → cùng cluster **tabletop_scene**

**Bài 29** sẽ xây một **Scene Cluster Builder** dựa trên:

- visual memory graph
- edge score giữa các frame
- revisit similarity / feature similarity

<p align="center">
  <img src="../../images/project_29.png" width="800">
</p>

---

# 3. Bài 29 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**
- **Đợt 6 = Bài 26 → 30**

Vì vậy:

## **Bài 29 = bài thứ tư của Đợt 6**

và nó vẫn phải dùng lại toàn bộ nền từ **Đợt 1 → Đợt 6**.

---

# 4. Từ Bài 28 sang Bài 29 đã nâng cấp gì

## Bài 28
Bài 28 thiên về:

```text
query → memory retrieval
```

- frame query
- tìm memory frame gần nhất
- quyết định revisit

## Bài 29
Bài 29 thiên về:

```text
memory organization
```

- nhìn toàn bộ memory graph
- tìm các nhóm frame kết nối mạnh
- gán chúng vào cluster
- tạo cluster report
- trả lời:
  - robot hiện có bao nhiêu vùng scene khác nhau trong memory?
  - mỗi cluster gồm các frame nào?
  - frame nào là đại diện cho cluster?

Tức là:

```text
Bài 28 = robot hỏi "mình đã thấy cảnh này chưa?"
Bài 29 = robot hỏi "toàn bộ memory của mình đang chia thành những scene nào?"
```

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu luồng:

```text
Memory Frames + Visual Memory Graph + Scene Cluster Config
→ extract features for all frames
→ build pair edges
→ create adjacency structure
→ group strongly connected frames into scene clusters
→ choose cluster representative frame
→ export cluster summary
→ optionally map revisit results into cluster IDs
```

---

# 6. Pipeline perception của bài

```text
Python Config Builder
→ build memory frame manifest
→ build cluster config

C++ Runtime
→ load memory frames
→ load cluster config
→ initialize extractor
→ extract features for all frames

for each candidate frame pair:
    → match descriptors
    → filter good matches
    → estimate homography
    → compute inlier ratio
    → compute edge score
    → accept / reject edge

build adjacency graph from accepted edges

run scene clustering:
    → find connected components / strong groups
    → assign cluster_id to each frame
    → compute cluster representative frame
    → compute cluster size
    → compute average cluster score

write cluster report
write frame-to-cluster mapping
save selected edge visualizations
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- dict / list / set
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
- edge scoring
- graph-based grouping

---

# 8. Đợt 1 → Đợt 6 được dùng như thế nào

## Đợt 1
- class / function / if else / loop
- đọc ảnh cơ bản

## Đợt 2
- list / dict / vector
- quản lý memory frame list

## Đợt 3
- nested loop cho frame pairs
- if/else để accept / reject edge

## Đợt 4
- file handling
- exception
- report export

## Đợt 5
- feature matching / homography / registration quality mindset

## Đợt 6
- module Python
- pointer / virtual / override
- workbench architecture
- visual memory graph
- revisit reasoning

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 29, bạn phải nắm được 13 ý:

## 1. Visual memory graph có thể được dùng để tổ chức scene memory
Không chỉ để match hay revisit.

## 2. Scene cluster là tầng giữa “frame memory” và “semantic map”
Nó chưa phải bản đồ semantic hoàn chỉnh, nhưng đã là nhóm scene.

## 3. Một cluster có thể được hiểu là “một vùng quan sát tương đối đồng nhất”
Ví dụ cluster bàn làm việc hoặc hành lang.

## 4. Graph edge mạnh thường gợi ý rằng hai frame thuộc cùng một scene cluster
Nhưng không phải lúc nào cũng tuyệt đối.

## 5. Connected components là cách nhập môn rất hợp để cluster frame
Bạn chưa cần KMeans hay spectral clustering ngay từ đầu.

## 6. Cluster representative frame rất quan trọng
Nó có thể dùng làm frame đại diện để relocalize nhanh hơn.

## 7. Revisit detection và clustering liên quan chặt chẽ
Một query frame có thể match không chỉ tới 1 frame, mà tới cả một cluster.

## 8. Đây là bước tiến tự nhiên từ “query một frame” sang “reason theo scene”
Robot bắt đầu nghĩ theo scene thay vì chỉ frame.

## 9. Python ở đây vẫn nên đóng vai trò config builder
C++ xử lý runtime clustering.

## 10. `virtual` / `override` giúp sau này thay đổi cluster strategy
Ví dụ connected components → score-threshold clustering → hierarchical clustering.

## 11. Một cluster report tốt nên có:
- cluster id
- member frames
- representative frame
- cluster size
- average score

## 12. Đây là bước chuẩn bị đẹp cho memory-based navigation / relocalization
Vì robot không cần nhớ từng frame rời rạc, mà nhớ theo vùng scene.

## 13. Bài 29 là bước nối rất hợp lý sau Bài 28
Từ **revisit detector** sang **scene memory organization**.

---

# 10. Cấu trúc folder

```text
mini_project_29_scene_cluster_builder_from_visual_memory_graph/
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
│  └─ outputs/
│     ├─ edge_memory_01_memory_02.jpg
│     ├─ edge_memory_02_memory_03.jpg
│     ├─ edge_memory_04_memory_05.jpg
│     ├─ scene_cluster_report.txt
│     ├─ frame_cluster_mapping.txt
│     └─ cluster_summary.txt
│
├─ config/
│  ├─ memory_frame_manifest.txt
│  └─ scene_cluster_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ cluster_report_template.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ MemoryFrameRecord.hpp
   │  ├─ SceneClusterConfig.hpp
   │  ├─ FrameFeatureRecord.hpp
   │  ├─ MatchEdgeRecord.hpp
   │  ├─ ClusterRecord.hpp
   │  ├─ BaseFeatureExtractor.hpp
   │  ├─ ORBFeatureExtractor.hpp
   │  ├─ BaseSceneClusterBuilder.hpp
   │  ├─ SceneClusterBuilder.hpp
   │  └─ SceneClusterReportWriter.hpp
   │
   └─ src/
      ├─ ORBFeatureExtractor.cpp
      ├─ SceneClusterBuilder.cpp
      └─ SceneClusterReportWriter.cpp
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
scene_cluster_config_path
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

# 11.2 Python — `SceneClusterConfigBuilder`

Tạo class con:

```python
class SceneClusterConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
memory_frames
scene_cluster_config
```

## `scene_cluster_config` ví dụ

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
    "save_edge_visualization": True
}
```

## Hàm cần có

### `add_memory_frame(frame_name, image_path)`
- thêm memory frame

### `set_scene_cluster_config(...)`
**Hành vi**
- `feature_type` thuộc `"ORB"` hoặc `"SIFT"`
- `max_features > 0`
- `good_match_distance_threshold > 0`
- `min_good_matches_for_edge >= 4`
- `ransac_reproj_threshold > 0`
- `0 <= min_inlier_ratio_for_edge <= 1`
- `0 <= min_edge_score_for_cluster <= 1`

### `write_memory_frame_manifest()`
Format gợi ý:

```text
frame_name=memory_01
image_path=assets/memory_frames/memory_01.jpg
```

### `write_scene_cluster_config()`

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **8 memory frames**
- set scene cluster config
- ghi:
  - `config/memory_frame_manifest.txt`
  - `config/scene_cluster_config.txt`

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

# 11.5 C++ — `SceneClusterConfig`

**File:**
```text
cpp/include/SceneClusterConfig.hpp
```

```cpp
struct SceneClusterConfig
{
    std::string feature_type;
    int max_features;
    std::string match_mode;
    double good_match_distance_threshold;
    int min_good_matches_for_edge;
    double ransac_reproj_threshold;
    double min_inlier_ratio_for_edge;
    double min_edge_score_for_cluster;
    bool save_edge_visualization;
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

# 11.8 C++ — `ClusterRecord`

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

---

# 11.11 C++ — `BaseSceneClusterBuilder`

**File:**
```text
cpp/include/BaseSceneClusterBuilder.hpp
```

```cpp
class BaseSceneClusterBuilder
{
public:
    virtual void load_memory_frame_manifest(const std::string& path) = 0;
    virtual void load_scene_cluster_config(const std::string& path) = 0;
    virtual void build_scene_clusters() = 0;
    virtual ~BaseSceneClusterBuilder() = default;
};
```

---

# 11.12 C++ — `SceneClusterBuilder`

**File:**
```text
cpp/include/SceneClusterBuilder.hpp
cpp/src/SceneClusterBuilder.cpp
```

Class kế thừa:

```cpp
class SceneClusterBuilder : public BaseSceneClusterBuilder
```

## Thuộc tính cần có

```cpp
private:
    std::vector<MemoryFrameRecord> memory_frames;
    SceneClusterConfig config;

    BaseFeatureExtractor* extractor;

    std::vector<FrameFeatureRecord> frame_features;
    std::vector<MatchEdgeRecord> edges;
    std::vector<ClusterRecord> clusters;

    static int builder_instance_count;
```

## Constructor / Destructor
```cpp
SceneClusterBuilder();
~SceneClusterBuilder();
```

### Destructor phải làm gì?
- `delete extractor` nếu khác `nullptr`
- log số cluster đã tạo

---

## Hàm load
- `load_memory_frame_manifest(...)`
- `load_scene_cluster_config(...)`

---

## Hàm khởi tạo extractor

### `initialize_extractor()`
- ORB → tạo `ORBFeatureExtractor`
- nếu chưa hỗ trợ SIFT thì log rõ

---

## Hàm extract

### `extract_all_features()`
- load từng memory frame
- extract feature
- lưu `FrameFeatureRecord`

---

## Hàm matching / edge

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
2. filter good matches
3. nếu good matches < `min_good_matches_for_edge`
   - edge fail
4. nếu đủ:
   - `findHomography(..., RANSAC, ransac_reproj_threshold, mask)`
   - tính `inlier_matches`
   - tính `inlier_ratio`
   - tính `edge_score`

### Gợi ý công thức `edge_score`
Ví dụ:

```text
edge_score = 0.5 * normalized_good_matches + 0.5 * inlier_ratio
```

### Edge accepted khi:
- homography valid
- `good_matches >= min_good_matches_for_edge`
- `inlier_ratio >= min_inlier_ratio_for_edge`
- `edge_score >= min_edge_score_for_cluster`

---

## Hàm build edges

### `build_all_edges()`
- loop qua mọi cặp frame `(i, j)` với `j > i`
- build edge
- push vào `edges`

---

## Hàm clustering

### `std::map<std::string, std::vector<std::string>> build_adjacency_list() const;`
- chỉ lấy accepted edges

### `std::vector<std::vector<std::string>> find_connected_components(
    const std::map<std::string, std::vector<std::string>>& adjacency
) const;`

**Yêu cầu**
- dùng BFS hoặc DFS
- mỗi connected component = 1 cluster ứng viên

### `ClusterRecord build_cluster_record(
    int cluster_id,
    const std::vector<std::string>& member_frames
) const;`

## Rule gợi ý cho `representative_frame`
Bạn có thể chọn:
- frame có nhiều neighbor accepted nhất trong cluster
- hoặc frame có tổng edge score cao nhất trong cluster

---

## Visualization

### `save_selected_edge_visualizations()`
- lưu ảnh match của các accepted edge

---

## Hàm chính

### `void build_scene_clusters() override;`
1. `initialize_extractor()`
2. `extract_all_features()`
3. `build_all_edges()`
4. build adjacency
5. find connected components
6. build `ClusterRecord`
7. save selected edge visualizations

### Getter

```cpp
const std::vector<ClusterRecord>& get_clusters() const;
const std::vector<MatchEdgeRecord>& get_edges() const;
```

---

# 11.13 C++ — `SceneClusterReportWriter`

**File:**
```text
cpp/include/SceneClusterReportWriter.hpp
cpp/src/SceneClusterReportWriter.cpp
```

Tạo class:

```cpp
class SceneClusterReportWriter
```

## Hàm cần có

### `write_scene_cluster_report(
    const std::string& path,
    const std::vector<ClusterRecord>& clusters
);`

Format gợi ý:

```text
[Scene Cluster]
Cluster ID: 0
Representative Frame: memory_03
Cluster Size: 3
Average Internal Edge Score: 0.72
Members:
- memory_01
- memory_02
- memory_03
----------------------------------------
```

### `write_frame_cluster_mapping(
    const std::string& path,
    const std::vector<ClusterRecord>& clusters
);`

Ví dụ:

```text
memory_01 -> cluster_0
memory_02 -> cluster_0
memory_03 -> cluster_0
memory_04 -> cluster_1
```

### `write_cluster_summary(
    const std::string& path,
    const std::vector<ClusterRecord>& clusters
);`

Ví dụ:

```text
Total Clusters: 3
Largest Cluster Size: 4
Cluster 0 Representative: memory_03
Cluster 1 Representative: memory_05
Cluster 2 Representative: memory_08
```

---

# 11.14 C++ — `main.cpp`

## Yêu cầu
- tạo `SceneClusterBuilder`
- load:
  - `config/memory_frame_manifest.txt`
  - `config/scene_cluster_config.txt`
- gọi `build_scene_clusters()`
- tạo `SceneClusterReportWriter`
- ghi:
  - `assets/outputs/scene_cluster_report.txt`
  - `assets/outputs/frame_cluster_mapping.txt`
  - `assets/outputs/cluster_summary.txt`

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
- graph edge building
- connected components / scene clustering
- cluster report

---

# 13. Output mong muốn

## File config

```text
config/memory_frame_manifest.txt
config/scene_cluster_config.txt
```

## Ảnh output

```text
assets/outputs/edge_memory_01_memory_02.jpg
assets/outputs/edge_memory_02_memory_03.jpg
assets/outputs/edge_memory_04_memory_05.jpg
```

## File report

```text
assets/outputs/scene_cluster_report.txt
assets/outputs/frame_cluster_mapping.txt
assets/outputs/cluster_summary.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Scene Cluster Config Builder
```

Dùng để tạo:
- memory frame manifest
- scene cluster config

## C++ đóng vai trò gì?

C++ là:

```text
Visual Memory Scene Organizer Runtime
```

Dùng để:
- build edges giữa memory frames
- gom cluster scene
- chọn representative frame
- xuất cluster report

## Computer Vision đóng vai trò gì?

CV là:

```text
memory frames
→ feature extraction
→ pair matching
→ homography validation
→ scene clustering
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo `memory_frame_manifest.txt`
- [ ] Python tạo `scene_cluster_config.txt`
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có module + `if __name__ == "__main__"`
- [ ] C++ có `BaseFeatureExtractor`
- [ ] C++ có `ORBFeatureExtractor`
- [ ] C++ có `BaseSceneClusterBuilder`
- [ ] C++ có `SceneClusterBuilder`
- [ ] C++ có pointer extractor
- [ ] C++ extract toàn bộ memory features
- [ ] C++ build accepted edges
- [ ] C++ build adjacency list
- [ ] C++ find connected components
- [ ] C++ build cluster record
- [ ] C++ ghi 3 file report

---

# 16. Gợi ý mở rộng

## 1. Thêm query-to-cluster relocalization
Thay vì query tới từng frame, query trực tiếp tới representative frame của cluster.

## 2. Thêm cluster merge / split rule
Nếu cluster quá lớn hoặc quá yếu, cho phép tách hoặc gộp.

## 3. Thêm scene labels thủ công
Ví dụ:
- `desk_scene`
- `kitchen_scene`
- `corridor_scene`

## 4. Chuẩn bị cho Bài 30
Sau Bài 29, bước rất hợp lý là:

```text
Cluster-Aware Visual Relocalization Query Engine
```

Tức là query frame mới sẽ:
- không so với toàn bộ memory frames
- mà so theo **scene cluster trước**
- rồi mới đi sâu vào frame đại diện / frame trong cluster
