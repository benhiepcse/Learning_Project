# 🤖 Bài 22: Feature Matching Homography Object Re-Localization — Ước lượng lại vùng object bằng Homography cho Humanoid Robot AI Perception

> Mini Project số 22 trong **Đợt 5 — Bài 21 → Bài 25**  
> **Bài 22 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5**.  
> Nếu **Bài 21** giúp robot biết target còn được tracking ổn hay không bằng **feature matching**, thì **Bài 22** sẽ nâng thêm một bước quan trọng:
>
> **dùng feature matches + homography để ước lượng lại vùng object trong frame mới.**

---

# 📌 Mục lục

- [1. Bài 22 lấy gì từ Đợt 5](#1-bài-22-lấy-gì-từ-đợt-5)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 22 nằm ở đâu trong roadmap](#3-bài-22-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 22 là bước tiếp theo hợp lý sau Bài 21](#4-vì-sao-bài-22-là-bước-tiếp-theo-hợp-lý-sau-bài-21)
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
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 22 lấy gì từ Đợt 5

Đợt 5 tập trung vào:

## Python
- OOP rõ ràng hơn
- `super()`
- method overriding
- classmethod / staticmethod
- config builder sạch hơn

## C++
- destructor
- `this`
- `static`
- tách file `.hpp / .cpp`
- `#pragma once`

## Computer Vision
- feature detection
- corner / keypoint
- ORB / SIFT
- descriptor
- feature matching
- **homography**
- object re-localization

Bài 21 mới dừng ở:

```text
template features ↔ frame features
→ good matches
→ tracking confidence
```

Bài 22 sẽ thêm:

```text
good matches
→ homography
→ transform template corners
→ estimated object polygon / bounding box
```

---

# 2. Mô tả

Ở **Bài 21**, robot đã có thể:

- đọc target template
- detect keypoints / descriptors
- match template với frame hiện tại
- tính confidence
- quyết định:
  - `TRACKING_STABLE`
  - `TRACKING_WEAK`
  - `TARGET_LOST`

Nhưng Bài 21 chủ yếu mới biết:

```text
target còn thấy hay không
target center ước lượng ở đâu
```

Bài 22 sẽ nâng thêm:

```text
target object nằm trong vùng nào của frame hiện tại
```

Mini-project này yêu cầu bạn xây một hệ thống để robot:

- đọc target template
- đọc frame sequence
- extract ORB / SIFT features
- match descriptors
- lọc good matches
- dùng `cv::findHomography`
- transform 4 góc template sang frame hiện tại
- tạo estimated object polygon
- tạo bounding box mới cho target
- đánh giá homography có ổn không
- lưu overlay + report

<p align="center">
  <img src="../../images/project_22.png" width="800">
</p>

---

# 3. Bài 22 nằm ở đâu trong roadmap

## Quy ước mini-project

- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**

Vì vậy:

## **Bài 22 = bài thứ hai của Đợt 5**

và phải kết hợp lại kiến thức của:

```text
Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5
```

---

# 4. Vì sao Bài 22 là bước tiếp theo hợp lý sau Bài 21

## Bài 21 cho bạn:
- feature detection
- descriptor matching
- tracking confidence

## Bài 22 thêm:
- homography estimation
- object polygon re-localization
- target region update

Tức là chuyển từ:

```text
“target có match tốt không?”
```

sang:

```text
“target đang nằm chính xác ở vùng nào trong frame mới?”
```

Đây là bước rất quan trọng trong humanoid robot perception vì robot cần biết vùng object hiện tại để:

- update ROI
- crop object tốt hơn
- đo depth tại object region
- đưa object vào pipeline pick / follow / cleaning

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Target Template + Frame Sequence + Feature Config + Homography Config
→ Detect Template Keypoints
→ Compute Template Descriptors
→ Detect Frame Keypoints
→ Compute Frame Descriptors
→ Match Descriptors
→ Filter Good Matches
→ Estimate Homography
→ Transform Template Corners
→ Estimate Object Polygon / Bounding Box
→ Save Overlay + Report
```

---

# 6. Pipeline perception của bài

```text
Config Builder
→ Read Frame Sequence Manifest
→ Read ROI / Template Config
→ Read Feature-Homography Config
→ Load Target Template
→ Build Template Feature Set
→ For Each Frame:
    → Load Left Image
    → Build Frame Feature Set
    → Match Template ↔ Frame
    → Filter Good Matches
    → If Enough Matches:
        → findHomography with RANSAC
        → perspectiveTransform template corners
        → build estimated polygon
        → build bounding box
        → compute homography quality
    → Else:
        → mark target lost
    → Draw polygon / bbox / status
    → Save output image
→ Write Homography Tracking Report
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- classmethod / staticmethod
- list / dict
- file handling
- config builder
- report helper

## C++
- class / inheritance
- destructor
- `this`
- `static`
- `std::vector`
- struct
- header / source tách file
- `#pragma once`

## Computer Vision
- ORB / SIFT
- keypoints
- descriptors
- descriptor matching
- good match filtering
- `cv::findHomography`
- `cv::perspectiveTransform`
- polygon / bounding box overlay

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào

## Đợt 1
- class / function / loop / if else
- đọc ảnh cơ bản

## Đợt 2
- ROI / crop / vector / list / dict
- config parsing

## Đợt 3
- image preprocessing
- threshold / contour nếu muốn hỗ trợ kiểm tra vùng object

## Đợt 4
- perception pipeline mindset
- object following / pick / scene decision có thể dùng output re-localized ROI

## Đợt 5
- OOP rõ hơn
- feature detection
- descriptor matching
- homography

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 22, bạn phải nắm được 10 ý quan trọng:

## 1. Feature matching không chỉ để biết giống nhau
Nó còn có thể dùng để ước lượng biến đổi hình học giữa template và frame.

## 2. Homography giúp re-localize object dạng mặt phẳng / vùng gần phẳng
Ví dụ poster, bảng, marker, mặt hộp nhìn tương đối phẳng.

## 3. `findHomography` cần đủ match tốt
Nếu match ít hoặc sai nhiều, homography sẽ không ổn.

## 4. RANSAC giúp giảm ảnh hưởng của match sai
Nó chọn tập match hợp lý hơn để ước lượng transform.

## 5. Template corners có thể được transform sang frame mới
Từ đó robot biết object polygon hiện tại.

## 6. Polygon tốt hơn center point
Center chỉ cho vị trí trung tâm, polygon cho cả vùng object.

## 7. Homography output có thể cấp lại ROI cho pipeline khác
Ví dụ:
- crop object mới
- đo depth trong object box
- pick candidate evaluation

## 8. Homography không phải lúc nào cũng đúng
Nếu object không phẳng, bị che khuất nhiều, hoặc viewpoint thay đổi mạnh thì kết quả có thể nhiễu.

## 9. Đây là bước đệm cho image registration
Bài 22 là cầu nối trực tiếp sang registration project.

## 10. Đây là kỹ thuật rất hữu ích cho humanoid target relocalization
Khi target bị lệch frame, robot vẫn có thể tìm lại vùng target.

---

# 10. Cấu trúc folder

```text
mini_project_22_feature_matching_homography_object_relocalization/
│
├─ README.md
│
├─ assets/
│  ├─ frames/
│  │  ├─ frame_01_left.jpg
│  │  ├─ frame_01_right.jpg
│  │  ├─ frame_02_left.jpg
│  │  ├─ frame_02_right.jpg
│  │  └─ ...
│  │
│  ├─ templates/
│  │  └─ target_template.jpg
│  │
│  └─ outputs/
│     ├─ frame_01_homography_overlay.jpg
│     ├─ frame_01_match_visualization.jpg
│     ├─ frame_02_homography_overlay.jpg
│     ├─ frame_02_match_visualization.jpg
│     └─ homography_tracking_report.txt
│
├─ config/
│  ├─ frame_sequence_manifest.txt
│  ├─ roi_config.txt
│  └─ feature_homography_config.txt
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
   │  ├─ BaseSensor.hpp
   │  ├─ StereoCameraSensor.hpp
   │  ├─ StereoFrameRecord.hpp
   │  ├─ ROIConfig.hpp
   │  ├─ FeatureHomographyConfig.hpp
   │  ├─ TemplateFeatureSet.hpp
   │  ├─ FrameFeatureSet.hpp
   │  ├─ HomographyMatchResult.hpp
   │  ├─ ObjectRelocalizationState.hpp
   │  ├─ BaseHomographyRelocalizer.hpp
   │  ├─ StereoHomographyObjectRelocalizer.hpp
   │  └─ HomographyReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoHomographyObjectRelocalizer.cpp
      └─ HomographyReportWriter.cpp
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
frame_sequence_manifest_path
roi_config_path
feature_homography_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in toàn bộ đường dẫn config

### `@classmethod create_default_paths(cls, project_root)`
- trả về path mặc định

### `@staticmethod validate_image_extension(path)`
- kiểm tra `.jpg`, `.jpeg`, `.png`

---

# 11.2 Python — `FeatureHomographyConfigBuilder`

Tạo class con:

```python
class FeatureHomographyConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
frame_records
roi_regions
feature_homography_config
```

## Config ví dụ

```python
{
    "target_template_path": "assets/templates/target_template.jpg",
    "target_roi_name": "object_region",
    "feature_type": "ORB",
    "max_features": 1000,
    "good_match_distance_threshold": 45.0,
    "min_good_matches": 20,
    "ransac_reproj_threshold": 4.0,
    "min_inlier_ratio": 0.35,
    "min_polygon_area": 500.0
}
```

## Hàm cần có

### `add_frame_record(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- thêm frame record

### `add_roi_region(roi_name, x, y, width, height)`
- thêm ROI

### `set_feature_homography_config(...)`
**Hành vi**
- kiểm tra `feature_type` thuộc `"ORB"` hoặc `"SIFT"`
- `max_features > 0`
- `min_good_matches >= 4`
- `ransac_reproj_threshold > 0`
- `0 < min_inlier_ratio <= 1`
- `min_polygon_area > 0`

### `write_feature_homography_config()`

Format gợi ý:

```text
target_template_path=assets/templates/target_template.jpg
target_roi_name=object_region
feature_type=ORB
max_features=1000
good_match_distance_threshold=45.0
min_good_matches=20
ransac_reproj_threshold=4.0
min_inlier_ratio=0.35
min_polygon_area=500.0
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **4 frame records**
- tạo ít nhất **2 ROI**
- set feature homography config
- ghi đủ:
  - `config/frame_sequence_manifest.txt`
  - `config/roi_config.txt`
  - `config/feature_homography_config.txt`

---

# 11.4 C++ — `BaseSensor`
- giống các bài trước

# 11.5 C++ — `StereoCameraSensor`
- giống các bài trước

# 11.6 C++ — `StereoFrameRecord`
- giống các bài trước

# 11.7 C++ — `ROIConfig`
- giống các bài trước

---

# 11.8 C++ — `FeatureHomographyConfig`

**File:**

```text
cpp/include/FeatureHomographyConfig.hpp
```

```cpp
struct FeatureHomographyConfig
{
    std::string target_template_path;
    std::string target_roi_name;
    std::string feature_type;

    int max_features;
    double good_match_distance_threshold;
    int min_good_matches;
    double ransac_reproj_threshold;
    double min_inlier_ratio;
    double min_polygon_area;
};
```

---

# 11.9 C++ — `TemplateFeatureSet`

**File:**

```text
cpp/include/TemplateFeatureSet.hpp
```

```cpp
struct TemplateFeatureSet
{
    std::string target_name;
    cv::Mat template_image;
    std::vector<cv::KeyPoint> keypoints;
    cv::Mat descriptors;
    std::vector<cv::Point2f> template_corners;
    bool is_valid;
};
```

---

# 11.10 C++ — `FrameFeatureSet`

**File:**

```text
cpp/include/FrameFeatureSet.hpp
```

```cpp
struct FrameFeatureSet
{
    std::string pair_name;
    cv::Mat frame_image;
    std::vector<cv::KeyPoint> keypoints;
    cv::Mat descriptors;
    bool is_valid;
};
```

---

# 11.11 C++ — `HomographyMatchResult`

**File:**

```text
cpp/include/HomographyMatchResult.hpp
```

```cpp
struct HomographyMatchResult
{
    std::string pair_name;

    int total_matches;
    int good_matches;
    int inlier_matches;

    double inlier_ratio;
    double polygon_area;

    std::vector<cv::DMatch> good_match_list;
    std::vector<char> inlier_mask;

    cv::Mat homography;
    std::vector<cv::Point2f> projected_corners;
    cv::Rect estimated_bbox;

    bool homography_valid;
};
```

---

# 11.12 C++ — `ObjectRelocalizationState`

**File:**

```text
cpp/include/ObjectRelocalizationState.hpp
```

```cpp
struct ObjectRelocalizationState
{
    std::string pair_name;
    std::string state_label;  // RELOCALIZED / WEAK_RELOCALIZATION / TARGET_LOST

    cv::Rect estimated_bbox;
    std::vector<cv::Point2f> object_polygon;

    double inlier_ratio;
    double polygon_area;

    bool is_valid;
};
```

---

# 11.13 C++ — `BaseHomographyRelocalizer`

**File:**

```text
cpp/include/BaseHomographyRelocalizer.hpp
```

```cpp
class BaseHomographyRelocalizer
{
public:
    virtual void load_frame_sequence_manifest(const std::string& path) = 0;
    virtual void load_roi_config(const std::string& path) = 0;
    virtual void load_feature_homography_config(const std::string& path) = 0;
    virtual void run_relocalization() = 0;
    virtual ~BaseHomographyRelocalizer() = default;
};
```

---

# 11.14 C++ — `StereoHomographyObjectRelocalizer`

**File:**

```text
cpp/include/StereoHomographyObjectRelocalizer.hpp
cpp/src/StereoHomographyObjectRelocalizer.cpp
```

Class kế thừa:

```cpp
class StereoHomographyObjectRelocalizer : public BaseHomographyRelocalizer
```

## Thuộc tính cần có

```cpp
private:
    std::vector<StereoFrameRecord> frame_records;
    std::vector<ROIConfig> roi_configs;

    FeatureHomographyConfig homography_config;

    TemplateFeatureSet template_feature_set;
    std::vector<HomographyMatchResult> homography_results;
    std::vector<ObjectRelocalizationState> relocalization_states;

    static int relocalizer_instance_count;
```

## Destructor bắt buộc

```cpp
~StereoHomographyObjectRelocalizer();
```

Dùng để log:
- số frame đã xử lý
- số state đã tạo

---

## Nhóm hàm load config
- `load_frame_sequence_manifest(...)`
- `load_roi_config(...)`
- `load_feature_homography_config(...)`

---

## Nhóm hàm feature

### `cv::Ptr<cv::Feature2D> create_feature_detector() const;`
- ORB hoặc SIFT

### `TemplateFeatureSet build_template_feature_set();`
- load template
- detect + compute descriptors
- tạo 4 góc template:
```text
(0,0), (w,0), (w,h), (0,h)
```

### `FrameFeatureSet build_frame_feature_set(const StereoFrameRecord& record) const;`
- đọc ảnh trái
- detect + compute descriptors

---

## Nhóm hàm matching + homography

### `std::vector<cv::DMatch> match_descriptors(...) const;`

### `std::vector<cv::DMatch> filter_good_matches(...) const;`

### `HomographyMatchResult estimate_homography_result(
    const std::string& pair_name,
    const TemplateFeatureSet& template_set,
    const FrameFeatureSet& frame_set
) const;`

## Hành vi tổng quát
1. match descriptors
2. filter good matches
3. nếu good matches < min_good_matches → invalid
4. lấy điểm template và điểm frame từ matches
5. `cv::findHomography(..., cv::RANSAC, ransac_reproj_threshold, inlier_mask)`
6. `cv::perspectiveTransform(template_corners, projected_corners, H)`
7. tính polygon area
8. tính inlier ratio
9. tạo estimated bbox
10. gán `homography_valid`

---

## Nhóm hàm relocalization state

### `double compute_polygon_area(const std::vector<cv::Point2f>& polygon) const;`

### `cv::Rect build_bbox_from_polygon(const std::vector<cv::Point2f>& polygon) const;`

### `ObjectRelocalizationState build_relocalization_state(
    const HomographyMatchResult& result
) const;`

## Rule gợi ý
- nếu homography invalid → `TARGET_LOST`
- nếu inlier ratio >= min_inlier_ratio và polygon_area >= min_polygon_area → `RELOCALIZED`
- ngược lại → `WEAK_RELOCALIZATION`

---

## Visualization

### `void draw_relocalization_overlay(
    cv::Mat& frame,
    const ObjectRelocalizationState& state
) const;`

- vẽ polygon
- vẽ bbox
- ghi label

### `cv::Mat build_match_visualization(...) const;`

- dùng `cv::drawMatches`

---

## Hàm chính

### `void process_single_frame_record(const StereoFrameRecord& record);`
1. build frame feature set
2. estimate homography
3. build relocalization state
4. draw overlay
5. save overlay
6. save match visualization
7. push result

### `void run_relocalization() override;`
- build template feature set
- loop frame records

### Getter

```cpp
const std::vector<ObjectRelocalizationState>& get_relocalization_states() const;
```

---

# 11.15 C++ — `HomographyReportWriter`

**File:**

```text
cpp/include/HomographyReportWriter.hpp
cpp/src/HomographyReportWriter.cpp
```

Tạo class:

```cpp
class HomographyReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<ObjectRelocalizationState>& states
);`

Format gợi ý:

```text
[Homography Re-Localization Frame]
Pair Name: frame_01
State: RELOCALIZED
BBox: x=220, y=130, w=140, h=90
Inlier Ratio: 0.62
Polygon Area: 12450.5
Valid: true

----------------------------------------
```

---

# 11.16 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- tạo `StereoHomographyObjectRelocalizer`
- load:
  - `config/frame_sequence_manifest.txt`
  - `config/roi_config.txt`
  - `config/feature_homography_config.txt`
- chạy `run_relocalization()`
- tạo `HomographyReportWriter`
- ghi report ra:
  - `assets/outputs/homography_tracking_report.txt`

---

# 12. Điều kiện bắt buộc

Project bắt buộc phải có:

- OOP Python
- OOP C++
- inheritance Python
- inheritance C++
- `super()` Python
- `classmethod` hoặc `staticmethod`
- destructor C++
- `this` C++
- `static` C++
- `#pragma once`
- tách `.hpp / .cpp`
- function rõ ràng
- module Python
- `loop`
- `if / else`
- `list / dict`
- `std::vector`
- detect keypoints
- compute descriptors
- match descriptors
- filter good matches
- estimate homography
- perspective transform corners
- object bbox / polygon
- report

---

# 13. Output mong muốn

## File config

```text
config/frame_sequence_manifest.txt
config/roi_config.txt
config/feature_homography_config.txt
```

## Ảnh output

```text
assets/outputs/frame_01_homography_overlay.jpg
assets/outputs/frame_01_match_visualization.jpg
assets/outputs/frame_02_homography_overlay.jpg
assets/outputs/frame_02_match_visualization.jpg
```

## File report

```text
assets/outputs/homography_tracking_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Feature Homography Config Builder
```

Dùng để tạo:
- frame sequence manifest
- ROI config
- feature homography config

## C++ đóng vai trò gì?

C++ là:

```text
Feature Matching + Homography Re-Localization Runtime
```

Dùng để:
- detect features
- match descriptors
- estimate homography
- re-localize target object

## Computer Vision đóng vai trò gì?

CV là:

```text
Template Features → Frame Features → Matches → Homography → Object Region
```

Dùng để robot không chỉ biết target còn thấy hay không, mà còn biết target nằm ở vùng nào.

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ config
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có `staticmethod` hoặc `classmethod`
- [ ] C++ có `BaseHomographyRelocalizer`
- [ ] C++ có `StereoHomographyObjectRelocalizer`
- [ ] C++ có destructor
- [ ] C++ dùng `this`
- [ ] C++ dùng `static`
- [ ] C++ detect keypoints
- [ ] C++ compute descriptors
- [ ] C++ match descriptors
- [ ] C++ filter good matches
- [ ] C++ estimate homography
- [ ] C++ perspective transform template corners
- [ ] C++ tạo estimated bbox
- [ ] C++ vẽ overlay
- [ ] C++ ghi report

---

# 16. Gợi ý mở rộng

## 1. Thêm homography quality score
Kết hợp:
- inlier ratio
- polygon area
- bbox size
- good match count

## 2. Dùng bbox mới làm ROI cho Bài 18 / 19
Sau khi re-localize object, bạn có thể đưa bbox vào pick candidate hoặc table cleaning.

## 3. Thêm fallback khi homography yếu
Nếu homography yếu, quay lại center tracking của Bài 21.

## 4. Chuẩn bị cho Bài 23
Sau Bài 22, bước hợp lý là:

```text
Image Registration & Scene Alignment Tool
```

Tức là dùng feature matching + homography để căn chỉnh 2 ảnh / 2 frame / 2 view.
