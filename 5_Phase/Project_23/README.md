# 🤖 Bài 23: Image Registration & Scene Alignment Tool — Căn chỉnh 2 ảnh / 2 frame bằng Feature Matching + Homography cho Humanoid Robot AI Perception

> Mini Project số 23 trong **Đợt 5 — Bài 21 → Bài 25**  
> **Bài 23 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5**.  
> Nếu **Bài 22** đã giúp robot **re-localize object** bằng feature matching + homography, thì **Bài 23** sẽ nâng tư duy lên cấp **scene-level geometry**:
>
> **căn chỉnh 2 ảnh / 2 frame / 2 scene views bằng Image Registration**.

---

# 📌 Mục lục

- [1. Bài 23 lấy gì từ Đợt 5](#1-bài-23-lấy-gì-từ-đợt-5)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 23 nằm ở đâu trong roadmap](#3-bài-23-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 23 là bước tiếp theo hợp lý sau Bài 22](#4-vì-sao-bài-23-là-bước-tiếp-theo-hợp-lý-sau-bài-22)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-pipeline-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào](#8-kiến-thức-đợt-1--2--3--4--5-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 23 lấy gì từ Đợt 5

Đợt 5 đang đi theo nhánh:

- **Bài 21** → Feature-based target tracking
- **Bài 22** → Feature matching + homography object re-localization
- **Bài 23** → **Image Registration & Scene Alignment**

Bài 23 lấy trực tiếp các kiến thức của Đợt 5:

## Python
- OOP rõ ràng hơn
- `super()`
- `classmethod` / `staticmethod`
- config builder
- validation logic

## C++
- destructor
- `this`
- `static`
- `.hpp / .cpp`
- class manager kiểu rõ ràng hơn

## Computer Vision
- feature detection
- descriptor matching
- good match filtering
- homography
- perspective warping
- image registration
- scene alignment

---

# 2. Mô tả

Ở **Bài 22**, bạn đã có pipeline:

```text
template image + frame image
→ feature matching
→ homography
→ re-localize object polygon
```

Nhưng Bài 22 vẫn còn tập trung vào **một object / template**.

Bài 23 sẽ nâng lên:

```text
scene A + scene B
→ feature matching
→ estimate homography
→ warp scene B về scene A
→ tạo aligned image
→ đo registration quality
```

Mini-project này yêu cầu bạn xây một hệ thống để robot:

- đọc **cặp ảnh scene reference / scene current**
- detect features ở cả 2 ảnh
- match descriptors
- estimate homography giữa 2 scene
- warp ảnh current về hệ của ảnh reference
- tạo ảnh **aligned result**
- tạo ảnh **difference / overlap preview**
- tính **registration quality**
- ghi report

Bài này rất quan trọng vì nó dạy robot cách:

- so sánh scene trước / sau
- căn chỉnh góc nhìn khác nhau
- chuẩn bị cho change detection, mapping, visual localization

<p align="center">
  <img src="../../images/project_23.png" width="800">
</p>

---

# 3. Bài 23 nằm ở đâu trong roadmap

## Quy ước mini-project

- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**

Vì vậy:

## **Bài 23 = bài thứ ba của Đợt 5**

và phải kết hợp lại toàn bộ kiến thức của:

```text
Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5
```

---

# 4. Vì sao Bài 23 là bước tiếp theo hợp lý sau Bài 22

## Bài 22 cho bạn:
- homography từ template → frame
- object polygon re-localization

## Bài 23 thêm:
- homography giữa **2 scene**
- warp ảnh current về ảnh reference
- registration quality
- overlap / difference visualization

Tức là chuyển từ:

```text
“object nằm ở đâu trong frame mới?”
```

sang:

```text
“toàn bộ scene mới có thể căn về scene cũ như thế nào?”
```

Đây là bước rất đúng để đi từ **object-level feature perception** sang **scene-level geometric alignment**.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Reference Scene + Current Scene + Registration Config
→ Detect Keypoints on Both Images
→ Compute Descriptors
→ Match Features
→ Filter Good Matches
→ Estimate Homography
→ Warp Current Scene to Reference Scene
→ Build Difference / Overlap Visualization
→ Compute Registration Quality
→ Save Report
```

---

# 6. Pipeline perception của bài

```text
Registration Pair Manifest
→ Read Registration Config
→ For Each Scene Pair:
    → Load reference image
    → Load current image
    → Convert to grayscale
    → Detect keypoints on both images
    → Compute descriptors
    → Match descriptors
    → Filter good matches
    → If enough matches:
        → findHomography with RANSAC
        → warpPerspective(current → reference)
        → build aligned image
        → build overlap visualization
        → build absolute difference map
        → compute registration score
    → Else:
        → mark registration failed
    → save outputs
→ write registration report
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- classmethod / staticmethod
- list / dict
- file handling
- config builder

## C++
- class / inheritance
- destructor
- `this`
- `static`
- vector / struct
- `.hpp / .cpp`
- `#pragma once`

## Computer Vision
- ORB / SIFT
- descriptor matching
- good match filtering
- homography
- `warpPerspective`
- overlap / difference image
- registration quality estimation

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào

## Đợt 1
- class / function / loop / if else
- đọc ảnh / ghi ảnh cơ bản

## Đợt 2
- ROI / grayscale / resize / file config
- list / dict / vector

## Đợt 3
- threshold / contour / image preprocessing nếu muốn xử lý difference map

## Đợt 4
- perception pipeline thinking
- scene report / multi-stage output
- robot task modules có thể dùng scene alignment cho follow / pick / cleaning

## Đợt 5
- feature detection / matching / homography
- object relocalization
- image registration

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 23, bạn phải nắm được 12 ý quan trọng:

## 1. Homography không chỉ dùng cho object template
Nó còn dùng để căn chỉnh **2 scene** với nhau.

## 2. Registration là bước nền cho scene comparison
Nếu 2 ảnh chưa căn chỉnh thì rất khó so sánh “cái gì đã thay đổi”.

## 3. `warpPerspective` là bước quan trọng sau khi có homography
Có H rồi nhưng phải warp ảnh thì mới thấy alignment thực tế.

## 4. Good matches quyết định registration quality
Ít match hoặc match sai → scene alignment kém.

## 5. Difference image chỉ hữu ích khi ảnh đã align đủ tốt
Nếu chưa align mà lấy difference thì nhiễu rất nhiều.

## 6. Registration là cầu nối sang change detection
Sau khi căn scene, bạn mới tìm vật nào mới xuất hiện / biến mất / bị di chuyển.

## 7. Registration cũng liên quan đến visual localization và mapping
Robot có thể so scene hiện tại với scene đã biết trước.

## 8. Không phải mọi scene đều hợp với homography
Nếu scene có parallax mạnh / 3D mạnh / viewpoint khác quá lớn thì homography có thể không đủ.

## 9. Cần đánh giá chất lượng registration
Ví dụ bằng:
- inlier ratio
- good match count
- average reprojection error
- overlap quality

## 10. Đây là bước đệm cho change detection và map alignment
Bài 23 là nền rất tốt cho Bài 24.

## 11. Bài này làm bạn quen với tư duy “scene-to-scene” thay vì “template-to-frame”
Đó là bước trưởng thành quan trọng trong perception.

## 12. Đây là kỹ thuật rất hữu ích cho robot household / humanoid
Ví dụ robot quay lại bàn làm việc và muốn so scene hiện tại với scene trước đó.

---

# 10. Cấu trúc folder

```text
mini_project_23_image_registration_scene_alignment_tool/
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
│     ├─ pair_01_match_visualization.jpg
│     ├─ pair_01_aligned_current.jpg
│     ├─ pair_01_overlap_visualization.jpg
│     ├─ pair_01_difference_map.jpg
│     ├─ pair_02_match_visualization.jpg
│     ├─ pair_02_aligned_current.jpg
│     ├─ pair_02_overlap_visualization.jpg
│     ├─ pair_02_difference_map.jpg
│     └─ image_registration_report.txt
│
├─ config/
│  ├─ registration_pair_manifest.txt
│  └─ registration_config.txt
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
   │  ├─ RegistrationPairRecord.hpp
   │  ├─ RegistrationConfig.hpp
   │  ├─ SceneFeatureSet.hpp
   │  ├─ RegistrationMatchResult.hpp
   │  ├─ RegistrationSceneState.hpp
   │  ├─ BaseImageRegistrar.hpp
   │  ├─ FeatureHomographyImageRegistrar.hpp
   │  └─ RegistrationReportWriter.hpp
   │
   └─ src/
      ├─ FeatureHomographyImageRegistrar.cpp
      └─ RegistrationReportWriter.cpp
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
registration_pair_manifest_path
registration_config_path
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

# 11.2 Python — `RegistrationConfigBuilder`

Tạo class con:

```python
class RegistrationConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
registration_pairs
registration_config
```

## `registration_config` ví dụ

```python
{
    "feature_type": "ORB",
    "max_features": 1200,
    "good_match_distance_threshold": 45.0,
    "min_good_matches": 25,
    "ransac_reproj_threshold": 4.0,
    "min_inlier_ratio": 0.35,
    "difference_threshold": 25
}
```

## Hàm cần có

### `add_registration_pair(pair_name, reference_image_path, current_image_path)`
- thêm pair
- validate extension

### `set_registration_config(...)`
**Hành vi**
- `feature_type` thuộc `"ORB"` hoặc `"SIFT"`
- `max_features > 0`
- `min_good_matches >= 4`
- `ransac_reproj_threshold > 0`
- `0 < min_inlier_ratio <= 1`
- `difference_threshold >= 0`

### `write_registration_pair_manifest()`

Format gợi ý:

```text
pair_name=pair_01
reference_image=assets/scene_pairs/pair_01_reference.jpg
current_image=assets/scene_pairs/pair_01_current.jpg
```

### `write_registration_config()`

Format gợi ý:

```text
feature_type=ORB
max_features=1200
good_match_distance_threshold=45.0
min_good_matches=25
ransac_reproj_threshold=4.0
min_inlier_ratio=0.35
difference_threshold=25
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 scene registration pairs**
- set registration config
- ghi đủ:
  - `config/registration_pair_manifest.txt`
  - `config/registration_config.txt`

---

# 11.4 C++ — `RegistrationPairRecord`

**File:**

```text
cpp/include/RegistrationPairRecord.hpp
```

```cpp
struct RegistrationPairRecord
{
    std::string pair_name;
    std::string reference_image_path;
    std::string current_image_path;
};
```

---

# 11.5 C++ — `RegistrationConfig`

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
    int difference_threshold;
};
```

---

# 11.6 C++ — `SceneFeatureSet`

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

# 11.7 C++ — `RegistrationMatchResult`

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
    std::vector<cv::DMatch> good_match_list;
    std::vector<char> inlier_mask;
};
```

---

# 11.8 C++ — `RegistrationSceneState`

**File:**

```text
cpp/include/RegistrationSceneState.hpp
```

```cpp
struct RegistrationSceneState
{
    std::string pair_name;
    std::string state_label; // REGISTERED / WEAK_REGISTRATION / REGISTRATION_FAILED

    int good_matches;
    int inlier_matches;
    double inlier_ratio;
    double difference_score;

    bool is_valid;
};
```

---

# 11.9 C++ — `BaseImageRegistrar`

**File:**

```text
cpp/include/BaseImageRegistrar.hpp
```

```cpp
class BaseImageRegistrar
{
public:
    virtual void load_registration_pair_manifest(const std::string& path) = 0;
    virtual void load_registration_config(const std::string& path) = 0;
    virtual void run_registration() = 0;
    virtual ~BaseImageRegistrar() = default;
};
```

---

# 11.10 C++ — `FeatureHomographyImageRegistrar`

**File:**

```text
cpp/include/FeatureHomographyImageRegistrar.hpp
cpp/src/FeatureHomographyImageRegistrar.cpp
```

Class kế thừa:

```cpp
class FeatureHomographyImageRegistrar : public BaseImageRegistrar
```

## Thuộc tính cần có

```cpp
private:
    std::vector<RegistrationPairRecord> pair_records;
    RegistrationConfig registration_config;

    std::vector<RegistrationMatchResult> match_results;
    std::vector<RegistrationSceneState> registration_states;

    static int registrar_instance_count;
```

## Destructor bắt buộc

```cpp
~FeatureHomographyImageRegistrar();
```

---

## Nhóm hàm load config
- `load_registration_pair_manifest(...)`
- `load_registration_config(...)`

---

## Nhóm hàm feature

### `cv::Ptr<cv::Feature2D> create_feature_detector() const;`
- ORB hoặc SIFT

### `SceneFeatureSet build_scene_feature_set(
    const std::string& image_name,
    const cv::Mat& image
) const;`

---

## Nhóm hàm matching

### `std::vector<cv::DMatch> match_descriptors(...) const;`

### `std::vector<cv::DMatch> filter_good_matches(...) const;`

### `RegistrationMatchResult build_match_result(
    const RegistrationPairRecord& pair,
    const SceneFeatureSet& reference_features,
    const SceneFeatureSet& current_features
) const;`

## Hành vi tổng quát
1. match descriptors
2. filter good matches
3. nếu good matches < min_good_matches → invalid
4. lấy matched points
5. `cv::findHomography(..., cv::RANSAC, ransac_reproj_threshold, inlier_mask)`
6. tính inlier ratio
7. tạo `RegistrationMatchResult`

---

## Nhóm hàm alignment

### `cv::Mat warp_current_to_reference(
    const cv::Mat& current_image,
    const cv::Mat& homography,
    const cv::Size& output_size
) const;`

### `cv::Mat build_overlap_visualization(
    const cv::Mat& reference_image,
    const cv::Mat& aligned_current
) const;`

### `cv::Mat build_difference_map(
    const cv::Mat& reference_image,
    const cv::Mat& aligned_current
) const;`

### `double compute_difference_score(
    const cv::Mat& difference_map
) const;`

---

## Nhóm hàm state

### `RegistrationSceneState build_scene_state(
    const RegistrationMatchResult& result,
    double difference_score
) const;`

## Rule gợi ý
- nếu homography invalid → `REGISTRATION_FAILED`
- nếu inlier_ratio >= min_inlier_ratio → `REGISTERED`
- ngược lại → `WEAK_REGISTRATION`

---

## Hàm chính

### `void process_single_pair(const RegistrationPairRecord& pair);`

1. đọc reference image
2. đọc current image
3. build feature sets
4. build match result
5. nếu valid:
   - warp current
   - build overlap
   - build difference map
   - compute difference score
6. build scene state
7. save outputs
8. push state

### `void run_registration() override;`
- loop qua pair records

### Getter

```cpp
const std::vector<RegistrationSceneState>& get_registration_states() const;
```

---

# 11.11 C++ — `RegistrationReportWriter`

**File:**

```text
cpp/include/RegistrationReportWriter.hpp
cpp/src/RegistrationReportWriter.cpp
```

Tạo class:

```cpp
class RegistrationReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<RegistrationSceneState>& states
);`

Format gợi ý:

```text
[Registration Pair]
Pair Name: pair_01
State: REGISTERED
Good Matches: 42
Inlier Matches: 31
Inlier Ratio: 0.74
Difference Score: 18.2
Valid: true

----------------------------------------
```

---

# 11.12 C++ — `main.cpp`

## Yêu cầu
- tạo `FeatureHomographyImageRegistrar`
- load:
  - `config/registration_pair_manifest.txt`
  - `config/registration_config.txt`
- chạy `run_registration()`
- tạo `RegistrationReportWriter`
- ghi report:
  - `assets/outputs/image_registration_report.txt`

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
- detect features
- compute descriptors
- match descriptors
- estimate homography
- warp perspective
- overlap visualization
- difference map
- registration report

---

# 13. Output mong muốn

## File config

```text
config/registration_pair_manifest.txt
config/registration_config.txt
```

## Ảnh output

```text
assets/outputs/pair_01_match_visualization.jpg
assets/outputs/pair_01_aligned_current.jpg
assets/outputs/pair_01_overlap_visualization.jpg
assets/outputs/pair_01_difference_map.jpg
```

## File report

```text
assets/outputs/image_registration_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Image Registration Config Builder
```

Dùng để tạo:
- scene pair manifest
- registration config

## C++ đóng vai trò gì?

C++ là:

```text
Feature-Homography Scene Registration Runtime
```

Dùng để:
- detect features
- match scenes
- estimate homography
- warp current scene về reference scene
- đo registration quality

## Computer Vision đóng vai trò gì?

CV là:

```text
Reference Scene ↔ Current Scene
→ Feature Matching
→ Homography
→ Warp Perspective
→ Scene Alignment
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ config
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có `staticmethod` / `classmethod`
- [ ] C++ có `BaseImageRegistrar`
- [ ] C++ có `FeatureHomographyImageRegistrar`
- [ ] C++ có destructor
- [ ] C++ dùng `this`
- [ ] C++ dùng `static`
- [ ] C++ detect features
- [ ] C++ compute descriptors
- [ ] C++ match descriptors
- [ ] C++ filter good matches
- [ ] C++ estimate homography
- [ ] C++ warp perspective
- [ ] C++ build overlap image
- [ ] C++ build difference map
- [ ] C++ compute registration score
- [ ] C++ ghi report

---

# 16. Gợi ý mở rộng

## 1. Thêm ROI-only registration
Chỉ căn chỉnh vùng tabletop hoặc object zone.

## 2. Thêm reprojection error
Ngoài inlier ratio, đo thêm reprojection error.

## 3. Thêm binary change mask
Threshold difference map để tìm vùng thay đổi rõ.

## 4. Chuẩn bị cho Bài 24
Sau Bài 23, bước hợp lý là:

```text
Scene Change Detection after Registration
```

Tức là sau khi align 2 scene, bắt đầu xác định:
- vật nào mới xuất hiện
- vật nào biến mất
- vật nào bị di chuyển
