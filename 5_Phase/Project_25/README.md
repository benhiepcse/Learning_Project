# 🤖 Bài 25: Object-Level Change Analysis for Humanoid Tabletop Scene — Phân tích thay đổi ở mức object cho tabletop scene trong Humanoid Robot AI Perception

> Mini Project số 25 trong **Đợt 5 — Bài 21 → Bài 25**  
> **Bài 25 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5**.  
> Đây là **bài chốt của Đợt 5**. Nếu **Bài 24** giúp robot phát hiện **vùng nào của scene đã thay đổi** sau registration, thì **Bài 25** sẽ nâng perception lên mức **object-level reasoning**:
>
> **robot không chỉ biết “có thay đổi”, mà còn cố gắng diễn giải thay đổi đó thành “object nào mới xuất hiện / biến mất / bị dời vị trí / cần pick-clean-inspect”.**

---

# 📌 Mục lục

- [1. Bài 25 lấy gì từ Đợt 5](#1-bài-25-lấy-gì-từ-đợt-5)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 25 nằm ở đâu trong roadmap](#3-bài-25-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 25 là bài chốt hợp lý cho Đợt 5](#4-vì-sao-bài-25-là-bài-chốt-hợp-lý-cho-đợt-5)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào](#8-kiến-thức-đợt-1--2--3--4--5-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Sau Bài 25 nên đi tiếp như thế nào](#16-sau-bài-25-nên-đi-tiếp-như-thế-nào)

---

# 1. Bài 25 lấy gì từ Đợt 5

Đợt 5 hiện tại của bạn đã đi theo chuỗi:

- **Bài 21** → Feature-based target tracking  
- **Bài 22** → Homography object re-localization  
- **Bài 23** → Image registration & scene alignment  
- **Bài 24** → Scene change detection after registration  
- **Bài 25** → **Object-level change analysis**

Bài 25 lấy trực tiếp phần cuối của nhánh này:

## Python
- OOP rõ ràng hơn
- `super()`
- `classmethod` / `staticmethod`
- config builder
- scene memory config / tabletop config

## C++
- destructor
- `this`
- `static`
- `.hpp / .cpp`
- manager kiểu perception reasoning rõ ràng hơn

## Computer Vision
- registration
- change mask
- contour / region extraction
- ROI/object association
- object-level change reasoning
- tabletop scene summary

---

# 2. Mô tả

Ở **Bài 24**, robot đã có thể:

- align current scene với reference scene
- tạo difference map
- threshold + morphology
- tìm change regions
- tính tổng diện tích thay đổi
- phân loại scene:
  - `NO_SIGNIFICANT_CHANGE`
  - `SMALL_CHANGE`
  - `LARGE_CHANGE`

Nhưng Bài 24 vẫn còn ở mức **scene-level**:

```text
scene này thay đổi ít hay nhiều
```

Bài 25 sẽ nâng thêm một bước rất quan trọng:

```text
change region nào tương ứng với object nào
object đó là object mới / object bị mất / object bị dời
object đó nên được pick / clean / inspect hay không
```

Mini-project này yêu cầu bạn xây một hệ thống để robot:

- đọc **reference tabletop scene** và **current tabletop scene**
- registration current → reference
- tạo change mask và change regions
- đọc **ROI/object zones** của bàn
- gán change regions vào từng object zone / tabletop zone
- tạo **object-level change record**
- phân loại object change:
  - `NEW_OBJECT`
  - `REMOVED_OBJECT`
  - `MOVED_OBJECT`
  - `UNCHANGED_OBJECT`
  - `UNKNOWN_CHANGE`
- gán **recommended action**:
  - `PICK`
  - `CLEAN`
  - `INSPECT`
  - `IGNORE`
- ghi **tabletop scene memory report**

<p align="center">
  <img src="../../images/project_25.png" width="800">
</p>

---

# 3. Bài 25 nằm ở đâu trong roadmap

## Quy ước mini-project

- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**

Vì vậy:

## **Bài 25 = bài chốt của Đợt 5**

và phải kết hợp lại toàn bộ kiến thức của:

```text
Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5
```

---

# 4. Vì sao Bài 25 là bài chốt hợp lý cho Đợt 5

## Bài 24 cho bạn:
- registration
- change mask
- change regions
- scene-level change summary

## Bài 25 thêm:
- association giữa **change region** và **object/table zone**
- object-level change label
- recommended robot action
- tabletop scene memory report

Tức là chuyển từ:

```text
“scene có thay đổi ở vài vùng”
```

sang:

```text
“cái cốc ở vùng trái bàn là vật mới xuất hiện”
“vùng giữa bàn có object bị dời”
“robot nên inspect hoặc pick vùng này”
```

Đây là đúng tinh thần **Humanoid Robot AI Perception** hơn nhiều, vì perception cuối cùng phải hỗ trợ **actionable reasoning** chứ không chỉ dừng ở mask.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Reference Tabletop Scene + Current Tabletop Scene + ROI/Object Zone Config + Registration Config + Change Detection Config + Object Change Reasoning Config
→ Register Current Scene to Reference Scene
→ Build Difference Map
→ Threshold + Morphology
→ Extract Change Regions
→ Associate Change Regions with Object Zones / Tabletop Zones
→ Build Object Change Records
→ Classify NEW / REMOVED / MOVED / UNCHANGED / UNKNOWN
→ Recommend PICK / CLEAN / INSPECT / IGNORE
→ Save Overlay + Tabletop Scene Memory Report
```

---

# 6. Pipeline perception của bài

```text
Scene Pair Manifest
→ Read ROI / Tabletop Object Zone Config
→ Read Registration Config
→ Read Change Detection Config
→ Read Object Change Reasoning Config
→ For Each Tabletop Scene Pair:
    → Load reference image
    → Load current image
    → Registration current → reference
    → Build difference map
    → Threshold + morphology
    → Extract change regions
    → For each ROI / tabletop zone:
        → measure overlap with change regions
        → build object change record
        → classify object-level change
        → assign recommended action
    → build tabletop scene summary
    → draw overlay
    → save aligned / diff / mask / overlay / report
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- classmethod / staticmethod
- list / dict
- file handling
- config builder
- scene memory config builder

## C++
- class / inheritance
- destructor
- `this`
- `static`
- vector / struct
- `.hpp / .cpp`
- `#pragma once`

## Computer Vision
- feature matching
- homography
- registration
- difference map
- threshold
- morphology
- contour / region extraction
- ROI overlap
- object-level scene reasoning

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào

## Đợt 1
- class / function / loop / if else
- đọc / ghi file / ảnh

## Đợt 2
- ROI / crop / config / list / dict / vector

## Đợt 3
- threshold / morphology / contour / bounding box

## Đợt 4
- object following / pick / clean / scene summary mindset
- decision label / action label / report style

## Đợt 5
- feature matching
- homography
- registration
- scene change detection
- object-level change reasoning

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 25, bạn phải nắm được 14 ý quan trọng:

## 1. Change detection hữu ích nhất khi gắn với object
Mask tự nó chưa đủ giúp robot hành động.

## 2. Một change region cần được gắn vào tabletop zone / object zone
Đó là bước đầu của scene memory.

## 3. Scene-level change và object-level change là hai tầng khác nhau
- scene-level: “scene đổi nhiều hay ít”
- object-level: “cái gì đã đổi”

## 4. `NEW_OBJECT`, `REMOVED_OBJECT`, `MOVED_OBJECT` là tư duy perception rất thực dụng
Vì robot trong nhà rất hay gặp các tình huống này.

## 5. Recommended action là cầu nối perception → behavior
Ví dụ:
- `PICK`
- `CLEAN`
- `INSPECT`
- `IGNORE`

## 6. Không phải mọi vùng change đều nên pick
Có thể chỉ cần inspect hoặc bỏ qua.

## 7. ROI overlap là cách đơn giản nhưng rất hữu ích để gắn change region vào object zone
Đây là bước nhập môn tốt trước khi làm tracking / segmentation sâu hơn.

## 8. Tabletop scene memory là tư duy rất đúng cho household humanoid
Robot nhớ scene trước đó và so với scene hiện tại.

## 9. Đây là bài tập rất tốt để luyện “perception summary”
Không chỉ xuất mask mà còn xuất report kiểu robot-friendly.

## 10. Registration quality vẫn phải được theo dõi
Nếu registration fail thì object change reasoning cũng đáng nghi.

## 11. Một object change record nên chứa cả geometry lẫn semantic label
Ví dụ:
- bbox
- overlap area
- change label
- action label

## 12. Bài 25 là điểm nối đẹp từ CV cơ bản sang robot scene reasoning
Bạn bắt đầu đi từ xử lý ảnh → reasoning cho robot.

## 13. Đây là nền cho ROS2 perception node / world-state node
Sau này bạn có thể publish object change records thành message.

## 14. Đây là bài chốt đẹp cho Đợt 5
Vì nó gom:
- feature matching
- homography
- registration
- change detection
- object-level reasoning

---

# 10. Cấu trúc folder

```text
mini_project_25_object_level_change_analysis_for_humanoid_tabletop_scene/
│
├─ README.md
│
├─ assets/
│  ├─ scene_pairs/
│  │  ├─ pair_01_reference.jpg
│  │  ├─ pair_01_current.jpg
│  │  ├─ pair_02_reference.jpg
│  │  ├─ pair_02_current.jpg
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ pair_01_aligned_current.jpg
│     ├─ pair_01_difference_map.jpg
│     ├─ pair_01_change_mask.jpg
│     ├─ pair_01_object_change_overlay.jpg
│     ├─ pair_02_aligned_current.jpg
│     ├─ pair_02_difference_map.jpg
│     ├─ pair_02_change_mask.jpg
│     ├─ pair_02_object_change_overlay.jpg
│     └─ tabletop_scene_memory_report.txt
│
├─ config/
│  ├─ scene_pair_manifest.txt
│  ├─ roi_config.txt
│  ├─ registration_config.txt
│  ├─ change_detection_config.txt
│  └─ object_change_reasoning_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ report_template.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ ScenePairRecord.hpp
   │  ├─ ROIConfig.hpp
   │  ├─ RegistrationConfig.hpp
   │  ├─ ChangeDetectionConfig.hpp
   │  ├─ ObjectChangeReasoningConfig.hpp
   │  ├─ SceneFeatureSet.hpp
   │  ├─ RegistrationMatchResult.hpp
   │  ├─ ChangeRegion.hpp
   │  ├─ ObjectChangeRecord.hpp
   │  ├─ TabletopSceneSummary.hpp
   │  ├─ BaseObjectChangeAnalyzer.hpp
   │  ├─ TabletopObjectChangeAnalyzer.hpp
   │  └─ TabletopSceneMemoryReportWriter.hpp
   │
   └─ src/
      ├─ TabletopObjectChangeAnalyzer.cpp
      └─ TabletopSceneMemoryReportWriter.cpp
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
scene_pair_manifest_path
roi_config_path
registration_config_path
change_detection_config_path
object_change_reasoning_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in path config

### `@classmethod create_default_paths(cls, project_root)`
- trả về path mặc định

### `@staticmethod validate_image_extension(path)`
- kiểm tra `.jpg`, `.jpeg`, `.png`

---

# 11.2 Python — `ObjectChangeSceneConfigBuilder`

Tạo class con:

```python
class ObjectChangeSceneConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
scene_pairs
roi_regions
registration_config
change_detection_config
object_change_reasoning_config
```

## `registration_config` ví dụ

```python
{
    "feature_type": "ORB",
    "max_features": 1200,
    "good_match_distance_threshold": 45.0,
    "min_good_matches": 25,
    "ransac_reproj_threshold": 4.0,
    "min_inlier_ratio": 0.35
}
```

## `change_detection_config` ví dụ

```python
{
    "difference_threshold": 25,
    "morph_open_kernel": 3,
    "morph_close_kernel": 5,
    "min_change_area": 300
}
```

## `object_change_reasoning_config` ví dụ

```python
{
    "min_overlap_ratio_for_object_change": 0.20,
    "moved_change_area_threshold": 1200,
    "new_object_change_area_threshold": 2000,
    "removed_object_change_area_threshold": 2000,
    "inspect_area_threshold": 1000,
    "pick_area_threshold": 2500
}
```

## Hàm cần có

### `add_scene_pair(pair_name, reference_image_path, current_image_path)`
- thêm scene pair

### `add_roi_region(roi_name, x, y, width, height)`
- thêm tabletop zone / object zone

### `set_registration_config(...)`
- validate như Bài 24

### `set_change_detection_config(...)`
- validate như Bài 24

### `set_object_change_reasoning_config(...)`
**Hành vi**
- `0 < min_overlap_ratio_for_object_change <= 1`
- các threshold area phải > 0

### `write_scene_pair_manifest()`

### `write_roi_config()`

### `write_registration_config()`

### `write_change_detection_config()`

### `write_object_change_reasoning_config()`

Format gợi ý:

```text
min_overlap_ratio_for_object_change=0.20
moved_change_area_threshold=1200
new_object_change_area_threshold=2000
removed_object_change_area_threshold=2000
inspect_area_threshold=1000
pick_area_threshold=2500
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 scene pairs**
- tạo ít nhất **4 ROI/table zones**
- set:
  - registration config
  - change detection config
  - object change reasoning config
- ghi đủ:
  - `config/scene_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/registration_config.txt`
  - `config/change_detection_config.txt`
  - `config/object_change_reasoning_config.txt`

---

# 11.4 C++ — `ScenePairRecord`

**File:**

```text
cpp/include/ScenePairRecord.hpp
```

```cpp
struct ScenePairRecord
{
    std::string pair_name;
    std::string reference_image_path;
    std::string current_image_path;
};
```

---

# 11.5 C++ — `ROIConfig`

**File:**

```text
cpp/include/ROIConfig.hpp
```

```cpp
struct ROIConfig
{
    std::string roi_name;
    int x;
    int y;
    int width;
    int height;
};
```

---

# 11.6 C++ — `RegistrationConfig`

**File:**

```text
cpp/include/RegistrationConfig.hpp
```

```cpp
struct RegistrationConfig
{
    std::string feature_type;
    int max_features;
    double good_match_distance_threshold;
    int min_good_matches;
    double ransac_reproj_threshold;
    double min_inlier_ratio;
};
```

---

# 11.7 C++ — `ChangeDetectionConfig`

**File:**

```text
cpp/include/ChangeDetectionConfig.hpp
```

```cpp
struct ChangeDetectionConfig
{
    int difference_threshold;
    int morph_open_kernel;
    int morph_close_kernel;
    int min_change_area;
};
```

---

# 11.8 C++ — `ObjectChangeReasoningConfig`

**File:**

```text
cpp/include/ObjectChangeReasoningConfig.hpp
```

```cpp
struct ObjectChangeReasoningConfig
{
    double min_overlap_ratio_for_object_change;
    double moved_change_area_threshold;
    double new_object_change_area_threshold;
    double removed_object_change_area_threshold;
    double inspect_area_threshold;
    double pick_area_threshold;
};
```

---

# 11.9 C++ — `SceneFeatureSet`

**File:**

```text
cpp/include/SceneFeatureSet.hpp
```

```cpp
struct SceneFeatureSet
{
    std::string image_name;
    cv::Mat image;
    std::vector<cv::KeyPoint> keypoints;
    cv::Mat descriptors;
    bool is_valid;
};
```

---

# 11.10 C++ — `RegistrationMatchResult`

**File:**

```text
cpp/include/RegistrationMatchResult.hpp
```

```cpp
struct RegistrationMatchResult
{
    std::string pair_name;
    int total_matches;
    int good_matches;
    int inlier_matches;
    double inlier_ratio;
    bool homography_valid;
    cv::Mat homography;
};
```

---

# 11.11 C++ — `ChangeRegion`

**File:**

```text
cpp/include/ChangeRegion.hpp
```

```cpp
struct ChangeRegion
{
    cv::Rect bbox;
    double area;
};
```

---

# 11.12 C++ — `ObjectChangeRecord`

**File:**

```text
cpp/include/ObjectChangeRecord.hpp
```

```cpp
struct ObjectChangeRecord
{
    std::string pair_name;
    std::string roi_name;

    double overlap_ratio;
    double changed_area_in_roi;

    std::string change_label;
    std::string recommended_action;
};
```

## `change_label` gợi ý
- `NEW_OBJECT`
- `REMOVED_OBJECT`
- `MOVED_OBJECT`
- `UNCHANGED_OBJECT`
- `UNKNOWN_CHANGE`

## `recommended_action` gợi ý
- `PICK`
- `CLEAN`
- `INSPECT`
- `IGNORE`

---

# 11.13 C++ — `TabletopSceneSummary`

**File:**

```text
cpp/include/TabletopSceneSummary.hpp
```

```cpp
struct TabletopSceneSummary
{
    std::string pair_name;

    int new_object_count;
    int removed_object_count;
    int moved_object_count;
    int unchanged_object_count;
    int unknown_change_count;

    std::string dominant_scene_action;
};
```

## `dominant_scene_action` gợi ý
- `PICK`
- `CLEAN`
- `INSPECT`
- `IGNORE`

---

# 11.14 C++ — `BaseObjectChangeAnalyzer`

**File:**

```text
cpp/include/BaseObjectChangeAnalyzer.hpp
```

```cpp
class BaseObjectChangeAnalyzer
{
public:
    virtual void load_scene_pair_manifest(const std::string& path) = 0;
    virtual void load_roi_config(const std::string& path) = 0;
    virtual void load_registration_config(const std::string& path) = 0;
    virtual void load_change_detection_config(const std::string& path) = 0;
    virtual void load_object_change_reasoning_config(const std::string& path) = 0;
    virtual void run_object_change_analysis() = 0;
    virtual ~BaseObjectChangeAnalyzer() = default;
};
```

---

# 11.15 C++ — `TabletopObjectChangeAnalyzer`

**File:**

```text
cpp/include/TabletopObjectChangeAnalyzer.hpp
cpp/src/TabletopObjectChangeAnalyzer.cpp
```

Class kế thừa:

```cpp
class TabletopObjectChangeAnalyzer : public BaseObjectChangeAnalyzer
```

## Thuộc tính cần có

```cpp
private:
    std::vector<ScenePairRecord> scene_pairs;
    std::vector<ROIConfig> roi_configs;

    RegistrationConfig registration_config;
    ChangeDetectionConfig change_config;
    ObjectChangeReasoningConfig reasoning_config;

    std::vector<RegistrationMatchResult> registration_results;
    std::vector<ObjectChangeRecord> object_change_records;
    std::vector<TabletopSceneSummary> scene_summaries;

    static int analyzer_instance_count;
```

## Destructor bắt buộc

```cpp
~TabletopObjectChangeAnalyzer();
```

---

## Nhóm hàm registration
- tương tự Bài 24:
  - detect features
  - match
  - find homography
  - warp current → reference

## Nhóm hàm change detection
- tương tự Bài 24:
  - difference map
  - threshold
  - morphology
  - extract change regions

---

## Nhóm hàm object-level reasoning

### `double compute_overlap_ratio(
    const cv::Rect& roi_bbox,
    const cv::Rect& change_bbox
) const;`

### `double compute_changed_area_inside_roi(
    const ROIConfig& roi,
    const std::vector<ChangeRegion>& regions
) const;`

### `ObjectChangeRecord build_object_change_record(
    const std::string& pair_name,
    const ROIConfig& roi,
    const std::vector<ChangeRegion>& regions
) const;`

## Rule gợi ý cho `change_label`
- nếu overlap quá nhỏ → `UNCHANGED_OBJECT`
- nếu changed area lớn và vượt `new_object_change_area_threshold`
  → `NEW_OBJECT`
- nếu changed area vừa và vượt `moved_change_area_threshold`
  → `MOVED_OBJECT`
- nếu changed area lớn theo kiểu “vùng trước có, giờ mất” thì bạn có thể gán `REMOVED_OBJECT`
- nếu khó quyết định → `UNKNOWN_CHANGE`

## Rule gợi ý cho `recommended_action`
- `NEW_OBJECT` + area lớn → `INSPECT` hoặc `PICK`
- `MOVED_OBJECT` → `INSPECT`
- `REMOVED_OBJECT` → `IGNORE`
- `UNCHANGED_OBJECT` → `IGNORE`
- nếu changed area rất lớn ở tabletop clutter zone → `CLEAN`

---

## Scene summary

### `TabletopSceneSummary build_scene_summary(
    const std::string& pair_name,
    const std::vector<ObjectChangeRecord>& pair_records
) const;`

## Rule gợi ý cho `dominant_scene_action`
- nếu nhiều record cần `CLEAN` → `CLEAN`
- nếu có object mới lớn → `INSPECT`
- nếu có object pickable / cần nhặt → `PICK`
- ngược lại → `IGNORE`

---

## Visualization

### `cv::Mat draw_object_change_overlay(
    const cv::Mat& aligned_current,
    const std::vector<ROIConfig>& rois,
    const std::vector<ObjectChangeRecord>& pair_records,
    const TabletopSceneSummary& summary
) const;`

**Hành vi**
- vẽ ROI/table zones
- ghi:
  - `roi_name`
  - `change_label`
  - `recommended_action`
- ghi `dominant_scene_action` của cả scene

---

## Hàm chính

### `void process_single_scene_pair(const ScenePairRecord& pair);`

1. load reference/current
2. registration
3. difference map + mask + change regions
4. loop qua ROI để build `ObjectChangeRecord`
5. build `TabletopSceneSummary`
6. draw overlay
7. save:
   - aligned current
   - difference map
   - change mask
   - object change overlay
8. push records và summary

### `void run_object_change_analysis() override;`
- loop qua scene pairs

### Getter

```cpp
const std::vector<TabletopSceneSummary>& get_scene_summaries() const;
const std::vector<ObjectChangeRecord>& get_object_change_records() const;
```

---

# 11.16 C++ — `TabletopSceneMemoryReportWriter`

**File:**

```text
cpp/include/TabletopSceneMemoryReportWriter.hpp
cpp/src/TabletopSceneMemoryReportWriter.cpp
```

Tạo class:

```cpp
class TabletopSceneMemoryReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<TabletopSceneSummary>& scene_summaries,
    const std::vector<ObjectChangeRecord>& object_records
);`

Format gợi ý:

```text
[Tabletop Scene Pair]
Pair Name: pair_01
Dominant Scene Action: INSPECT
New Objects: 1
Removed Objects: 0
Moved Objects: 2
Unchanged Objects: 3
Unknown Changes: 1

[Object Change]
ROI Name: cup_zone
Change Label: NEW_OBJECT
Recommended Action: PICK
Changed Area In ROI: 2840
Overlap Ratio: 0.46

----------------------------------------
```

---

# 11.17 C++ — `main.cpp`

## Yêu cầu
- tạo `TabletopObjectChangeAnalyzer`
- load:
  - `config/scene_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/registration_config.txt`
  - `config/change_detection_config.txt`
  - `config/object_change_reasoning_config.txt`
- chạy `run_object_change_analysis()`
- tạo `TabletopSceneMemoryReportWriter`
- ghi report:
  - `assets/outputs/tabletop_scene_memory_report.txt`

---

# 12. Điều kiện bắt buộc

Project bắt buộc phải có:

- OOP Python
- OOP C++
- inheritance Python
- inheritance C++
- `super()`
- `classmethod` hoặc `staticmethod`
- destructor C++
- `this`
- `static`
- `#pragma once`
- tách `.hpp / .cpp`
- `loop`
- `if / else`
- `list / dict`
- `std::vector`
- feature matching
- registration
- change detection
- ROI overlap analysis
- object-level change record
- scene summary
- memory report

---

# 13. Output mong muốn

## File config

```text
config/scene_pair_manifest.txt
config/roi_config.txt
config/registration_config.txt
config/change_detection_config.txt
config/object_change_reasoning_config.txt
```

## Ảnh output

```text
assets/outputs/pair_01_aligned_current.jpg
assets/outputs/pair_01_difference_map.jpg
assets/outputs/pair_01_change_mask.jpg
assets/outputs/pair_01_object_change_overlay.jpg
```

## File report

```text
assets/outputs/tabletop_scene_memory_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Object-Level Tabletop Scene Config Builder
```

Dùng để tạo:
- scene pair manifest
- ROI config
- registration config
- change detection config
- object change reasoning config

## C++ đóng vai trò gì?

C++ là:

```text
Tabletop Object Change Analysis Runtime
```

Dùng để:
- registration
- change detection
- object-zone reasoning
- action recommendation
- tabletop memory report

## Computer Vision đóng vai trò gì?

CV là:

```text
Scene Registration
→ Change Regions
→ ROI/Object Association
→ Object-Level Change Labels
→ Scene Memory Reasoning
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ config
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có `staticmethod` / `classmethod`
- [ ] C++ có `BaseObjectChangeAnalyzer`
- [ ] C++ có `TabletopObjectChangeAnalyzer`
- [ ] C++ có destructor
- [ ] C++ dùng `this`
- [ ] C++ dùng `static`
- [ ] C++ registration scene pair
- [ ] C++ build difference map / mask
- [ ] C++ extract change regions
- [ ] C++ compute ROI overlap
- [ ] C++ build object change records
- [ ] C++ build tabletop scene summary
- [ ] C++ vẽ overlay
- [ ] C++ ghi tabletop scene memory report

---

# 16. Sau Bài 25 nên đi tiếp như thế nào

Bài 25 là **điểm chốt rất đẹp cho Đợt 5**. Sau đây là hướng đi hợp lý nhất cho **Đợt 6** nếu bạn muốn giữ đúng tinh thần **Humanoid Robot AI Perception**:

## Hướng 1 — Đi tiếp theo nhánh scene memory / reasoning
- Multi-frame tabletop memory
- object permanence
- scene graph đơn giản cho tabletop
- temporal change history

## Hướng 2 — Đi tiếp theo nhánh stereo + feature fusion
- feature tracking + depth fusion
- 3D object re-localization
- stereo scene change with depth consistency

## Hướng 3 — Đi tiếp theo nhánh robot behavior integration
- dùng output `ObjectChangeRecord` để:
  - tạo pick task queue
  - tạo clean task queue
  - publish task decision cho ROS2 node

## Nếu bám sát humanoid robot nhất
Bước hợp lý nhất cho **Bài 26** là:

```text
Stereo + Tabletop Object Memory Fusion
```

Tức là nâng Bài 25 từ **change analysis trên ảnh 2D** sang:

- change analysis + **depth**
- object zone + **3D position estimate**
- scene memory + **robot-frame object memory**
