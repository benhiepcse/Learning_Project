# 🤖 Bài 21: Feature-Based Humanoid Target Tracker — Bộ theo dõi target bằng đặc trưng ảnh cho Humanoid Robot AI Perception

> Mini Project số 21 trong **Đợt 5**  
> **Bài 21 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5**.  
> Đây là bài mở đầu của **Đợt 5 (Ngày 9–10)**, nơi bạn bắt đầu đi từ **stereo/depth/robot-frame perception** sang một nhánh rất quan trọng trong AI Perception:
>
> **Feature-Based Vision** — dùng **corner / keypoint / descriptor / matching** để theo dõi target qua nhiều frame, thay vì chỉ nhìn theo ROI tĩnh.
>
> Nói ngắn gọn:
>
> ```text
> left/right image or frame sequence
> → detect keypoints
> → compute descriptors
> → match target features
> → estimate target motion / visibility
> → update humanoid tracking decision
> ```

---

# 📌 Mục lục

- [1. Bài 21 lấy gì từ Đợt 5](#1-bài-21-lấy-gì-từ-đợt-5)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 21 nằm ở đâu trong roadmap](#3-bài-21-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 21 là bước tiếp theo hợp lý sau Đợt 4](#4-vì-sao-bài-21-là-bước-tiếp-theo-hợp-lý-sau-đợt-4)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào](#8-kiến-thức-đợt-1--2--3--4--5-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
  - [11.1 Python — BaseConfigBuilder](#111-python--baseconfigbuilder)
  - [11.2 Python — FeatureTrackingConfigBuilder](#112-python--featuretrackingconfigbuilder)
  - [11.3 Python — main_config_builder.py](#113-python--main_config_builderpy)
  - [11.4 C++ — BaseSensor](#114-c--basesensor)
  - [11.5 C++ — StereoCameraSensor](#115-c--stereocamerasensor)
  - [11.6 C++ — StereoFrameRecord](#116-c--stereoframerecord)
  - [11.7 C++ — ROIConfig](#117-c--roiconfig)
  - [11.8 C++ — CameraIntrinsics](#118-c--cameraintrinsics)
  - [11.9 C++ — Transform3D](#119-c--transform3d)
  - [11.10 C++ — Point3D / RobotFramePoint3D](#1110-c--point3d--robotframepoint3d)
  - [11.11 C++ — FeatureTrackingConfig](#1111-c--featuretrackingconfig)
  - [11.12 C++ — TargetTemplateFeatureSet](#1112-c--targettemplatefeatureset)
  - [11.13 C++ — FrameFeatureSet](#1113-c--framefeatureset)
  - [11.14 C++ — FeatureMatchResult](#1114-c--featurematchresult)
  - [11.15 C++ — TargetTrackingState](#1115-c--targettrackingstate)
  - [11.16 C++ — BaseFeatureTracker](#1116-c--basefeaturetracker)
  - [11.17 C++ — StereoFeatureTargetTracker](#1117-c--stereofeaturetargettracker)
  - [11.18 C++ — FeatureTrackingReportWriter](#1118-c--featuretrackingreportwriter)
  - [11.19 C++ — main.cpp](#1119-c--maincpp)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 21 lấy gì từ Đợt 5

Theo roadmap 4 tuần, **Đợt 5 (Ngày 9–10)** có 3 phần chính:

## Python — Phase 6: OOP rõ ràng hơn
- attributes / methods
- encapsulation cơ bản
- inheritance với `super()`
- method overriding
- class methods / static methods

## C++ — Phase 6: OOP rõ ràng hơn
- destructor
- `this`
- tách `HPP / CPP`
- `#pragma once`
- `static`

## Computer Vision — Phase 3: Feature Detection & Matching
- feature là gì
- corner detection
- ORB / SIFT
- descriptor

Bài 21 sẽ lấy đúng 3 phần đó và gắn chúng vào **vai trò thực tế trong humanoid AI perception**:

> robot phải **giữ target ổn định qua nhiều frame**, kể cả khi target di chuyển hoặc ROI thay đổi nhẹ, nên không thể chỉ dựa vào box cố định — phải dùng **feature-based tracking**.

---

# 2. Mô tả

Ở **Đợt 4**, bạn đã có perception pipeline cho:

- object following
- pick candidate evaluation
- table cleaning decision
- multi-task scene manager

Nhưng các bài đó vẫn đang thiên về:

- ROI / object region có sẵn
- quyết định trên từng scene riêng lẻ
- chưa đi sâu vào **theo dõi target qua nhiều frame bằng đặc trưng ảnh**

Bài 21 yêu cầu bạn xây một mini-project để robot:

- đọc **frame sequence** hoặc nhiều stereo pairs theo thời gian
- chọn một **target ROI template**
- detect **keypoints / descriptors** trên target template
- detect keypoints / descriptors trên từng frame
- match đặc trưng giữa template và frame hiện tại
- ước lượng:
  - target còn thấy không
  - target đang lệch trái / phải
  - target đang lớn dần / nhỏ dần
  - tracking có ổn định không
- xuất **tracking report** cho humanoid robot

---

# 3. Bài 21 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**

Vì vậy:

## **Bài 21 = bài đầu tiên của Đợt 5**
và phải **kết hợp lại kiến thức của Đợt 1 + 2 + 3 + 4 + 5**.

---

# 4. Vì sao Bài 21 là bước tiếp theo hợp lý sau Đợt 4

## Đợt 4 cho bạn:
- stereo / depth / point cloud
- robot-frame localization
- follow / pick / cleaning / multi-task manager

## Đợt 5 bắt đầu thêm:
- **feature-based perception**
- **template matching bằng keypoint + descriptor**
- **tracking target qua nhiều frame**

Tức là chuyển từ:

```text
“robot hiểu một scene”
```

sang:

```text
“robot theo dõi cùng một target qua nhiều scene / nhiều frame”
```

Đây là bước rất đúng vì humanoid robot perception ngoài **scene understanding** còn cần **temporal consistency**.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Target Template ROI + Frame Sequence / Stereo Pair Sequence + Feature Tracking Config
→ Load Target Template
→ Detect Keypoints on Target Template
→ Compute Template Descriptors
→ For Each New Frame:
    → Detect Keypoints
    → Compute Descriptors
    → Match Template ↔ Current Frame
    → Filter Good Matches
    → Estimate Target Center / Motion Trend / Visibility Score
    → Decide Tracking State
    → Save Feature Visualization + Tracking Report
```

Bài này giúp bạn hiểu một ý rất quan trọng trong humanoid perception:

> **Perception không chỉ là phát hiện object ở một frame, mà còn là duy trì identity của target theo thời gian.**

---

# 6. Pipeline perception của bài

```text
Tracking Config
→ Read Frame / Stereo Sequence Manifest
→ Read ROI / Target Template Config
→ Read Feature Tracking Config
→ Create Stereo Camera Sensor Object
→ Extract Target Template ROI from first reference frame
→ Detect Template Keypoints + Descriptors
→ For Each Input Frame:
    → Load Left / Right Image (or left image only for tracking visualization)
    → Convert to grayscale
    → Detect keypoints
    → Compute descriptors
    → Match template descriptors with frame descriptors
    → Sort / filter matches
    → Estimate target center from matched keypoints
    → Estimate target tracking confidence
    → Decide tracking label
    → Save overlay image + match visualization + tracking report
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
- tách `hpp/cpp`
- vector / struct / enum-like string label

## Computer Vision
- corner / keypoint / descriptor
- ORB hoặc SIFT
- feature matching
- target tracking by matching
- overlay visualization

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào

# 8.1 Đợt 1
- class, function, loop, if/else
- đọc ảnh cơ bản

# 8.2 Đợt 2
- list / dict / vector
- ROI, crop, grayscale, resize

# 8.3 Đợt 3
- brightness / threshold / edge / contour
- xử lý ảnh trước khi detect feature nếu cần

# 8.4 Đợt 4
- perception pipeline mindset
- object following logic
- scene report / decision report

# 8.5 Đợt 5
- OOP rõ ràng hơn
- `super()`, static/class methods
- `hpp/cpp`, destructor, `this`
- feature detection / descriptor / matching

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 21, bạn phải nắm được 12 ý quan trọng:

## 1. Feature-based tracking khác ROI tĩnh
ROI tĩnh chỉ là vùng cắt sẵn; feature tracking giúp bám target khi target dịch chuyển.

## 2. Template target có thể được biểu diễn bằng keypoints + descriptors
Ví dụ ORB descriptor của một object hoặc một patch trên người.

## 3. Matching quality quyết định tracking confidence
Ít match tốt → tracking yếu; nhiều match tốt → tracking ổn hơn.

## 4. Không phải keypoint nào cũng hữu ích
Cần filter match hoặc chọn descriptor phù hợp.

## 5. Corner / feature là nền cho registration, tracking, relocalization
Đây là mảng rất quan trọng trong robot vision.

## 6. Feature tracker là bước đệm cho visual odometry / SLAM perception
Vì tư duy match giữa frame này và frame khác là nền tảng.

## 7. Trong humanoid following, tracking ổn định quan trọng hơn detect một lần
Robot cần biết target có còn ở đó hay đã biến mất.

## 8. Tracking state nên được tách thành một lớp riêng
Ví dụ:
- `TRACKING_STABLE`
- `TRACKING_WEAK`
- `TARGET_LOST`

## 9. OOP của Đợt 5 nên bắt đầu rõ hơn
Project này phải có class manager, class config, class report writer, class tracker.

## 10. `static` và class method có chỗ dùng thực tế
Ví dụ factory tạo detector hoặc helper validate config.

## 11. `hpp/cpp` tách file làm project giống codebase robot hơn
Không còn kiểu nhét hết vào một file.

## 12. Đây là bước đẹp trước các bài registration / multi-frame geometry
Vì bạn đã bắt đầu có temporal vision.

---

# 10. Cấu trúc folder

```text
mini_project_21_feature_based_humanoid_target_tracker/
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
│     ├─ frame_01_tracking_overlay.jpg
│     ├─ frame_01_match_visualization.jpg
│     ├─ frame_02_tracking_overlay.jpg
│     ├─ frame_02_match_visualization.jpg
│     └─ feature_tracking_report.txt
│
├─ config/
│  ├─ frame_sequence_manifest.txt
│  ├─ roi_config.txt
│  ├─ camera_intrinsics_config.txt
│  ├─ camera_to_robot_transform.txt
│  └─ feature_tracking_config.txt
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
   │  ├─ CameraIntrinsics.hpp
   │  ├─ Transform3D.hpp
   │  ├─ Point3D.hpp
   │  ├─ RobotFramePoint3D.hpp
   │  ├─ FeatureTrackingConfig.hpp
   │  ├─ TargetTemplateFeatureSet.hpp
   │  ├─ FrameFeatureSet.hpp
   │  ├─ FeatureMatchResult.hpp
   │  ├─ TargetTrackingState.hpp
   │  ├─ BaseFeatureTracker.hpp
   │  ├─ StereoFeatureTargetTracker.hpp
   │  └─ FeatureTrackingReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoFeatureTargetTracker.cpp
      └─ FeatureTrackingReportWriter.cpp
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
camera_intrinsics_config_path
camera_to_robot_transform_path
feature_tracking_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in toàn bộ đường dẫn config

### `@classmethod create_default_paths(cls, project_root)`
- trả về các path mặc định cho project

### `@staticmethod validate_image_extension(path)`
- kiểm tra `.jpg`, `.png`, `.jpeg`

---

# 11.2 Python — `FeatureTrackingConfigBuilder`

Tạo class con:

```python
class FeatureTrackingConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
frame_records
roi_regions
camera_intrinsics_config
camera_to_robot_transform
feature_tracking_config
```

## `feature_tracking_config`

Ví dụ:

```python
{
    "target_template_path": "assets/templates/target_template.jpg",
    "target_roi_name": "person_region",
    "feature_type": "ORB",
    "max_features": 800,
    "good_match_distance_threshold": 40.0,
    "min_good_matches": 20,
    "tracking_confidence_threshold": 0.55
}
```

## Hàm cần có

### `add_frame_record(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- thêm frame / stereo pair record
- dùng `validate_image_extension()`

### `add_roi_region(roi_name, x, y, width, height)`
- thêm ROI

### `set_camera_intrinsics_config(fx, fy, cx, cy)`
- lưu intrinsics

### `set_camera_to_robot_transform(tx, ty, tz, roll_deg, pitch_deg, yaw_deg)`
- lưu transform

### `set_feature_tracking_config(...)`
**Hành vi**
- lưu config feature tracking
- kiểm tra:
  - `feature_type` chỉ nhận `"ORB"` hoặc `"SIFT"`
  - `max_features > 0`
  - `min_good_matches > 0`
  - `0 < tracking_confidence_threshold <= 1`

### `write_feature_tracking_config()`
Format gợi ý:

```text
target_template_path=assets/templates/target_template.jpg
target_roi_name=person_region
feature_type=ORB
max_features=800
good_match_distance_threshold=40.0
min_good_matches=20
tracking_confidence_threshold=0.55
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **4 frame records**
- tạo ít nhất **2 ROI**
- set:
  - camera intrinsics
  - camera-to-robot transform
  - feature tracking config
- ghi đủ file config của project

---

# 11.4 C++ — `BaseSensor`
- giống các bài trước

# 11.5 C++ — `StereoCameraSensor`
- giống các bài trước

# 11.6 C++ — `StereoFrameRecord`
- giống các bài trước

# 11.7 C++ — `ROIConfig`
- giống các bài trước

# 11.8 C++ — `CameraIntrinsics`
- giống các bài trước

# 11.9 C++ — `Transform3D`
- giống các bài trước

# 11.10 C++ — `Point3D / RobotFramePoint3D`
- có thể tái sử dụng từ các bài trước nếu bạn muốn lưu target center trong robot frame

---

# 11.11 C++ — `FeatureTrackingConfig`

**File:**

```text
cpp/include/FeatureTrackingConfig.hpp
```

Tạo struct:

```cpp
struct FeatureTrackingConfig
{
    std::string target_template_path;
    std::string target_roi_name;
    std::string feature_type; // ORB hoặc SIFT

    int max_features;
    double good_match_distance_threshold;
    int min_good_matches;
    double tracking_confidence_threshold;
};
```

---

# 11.12 C++ — `TargetTemplateFeatureSet`

**File:**

```text
cpp/include/TargetTemplateFeatureSet.hpp
```

Tạo struct:

```cpp
struct TargetTemplateFeatureSet
{
    std::string target_name;
    std::vector<cv::KeyPoint> keypoints;
    cv::Mat descriptors;
    cv::Rect roi;
    bool is_valid;
};
```

---

# 11.13 C++ — `FrameFeatureSet`

**File:**

```text
cpp/include/FrameFeatureSet.hpp
```

Tạo struct:

```cpp
struct FrameFeatureSet
{
    std::string pair_name;
    std::vector<cv::KeyPoint> keypoints;
    cv::Mat descriptors;
    bool is_valid;
};
```

---

# 11.14 C++ — `FeatureMatchResult`

**File:**

```text
cpp/include/FeatureMatchResult.hpp
```

Tạo struct:

```cpp
struct FeatureMatchResult
{
    std::string pair_name;
    int total_matches;
    int good_matches;
    double tracking_confidence;

    cv::Point2f estimated_target_center;
    bool target_visible;
    bool is_tracking_stable;
};
```

---

# 11.15 C++ — `TargetTrackingState`

**File:**

```text
cpp/include/TargetTrackingState.hpp
```

Tạo struct:

```cpp
struct TargetTrackingState
{
    std::string pair_name;
    std::string tracking_label;   // TRACKING_STABLE / TRACKING_WEAK / TARGET_LOST
    double tracking_confidence;
    int good_matches;
    cv::Point2f target_center_px;
};
```

---

# 11.16 C++ — `BaseFeatureTracker`

**File:**

```text
cpp/include/BaseFeatureTracker.hpp
```

Tạo abstract class:

```cpp
class BaseFeatureTracker
{
public:
    virtual void load_frame_sequence_manifest(const std::string& path) = 0;
    virtual void load_roi_config(const std::string& path) = 0;
    virtual void load_camera_intrinsics_config(const std::string& path) = 0;
    virtual void load_camera_to_robot_transform(const std::string& path) = 0;
    virtual void load_feature_tracking_config(const std::string& path) = 0;
    virtual void run_tracking() = 0;
    virtual ~BaseFeatureTracker() = default;
};
```

---

# 11.17 C++ — `StereoFeatureTargetTracker`

**File:**

```text
cpp/include/StereoFeatureTargetTracker.hpp
cpp/src/StereoFeatureTargetTracker.cpp
```

Class kế thừa:

```cpp
class StereoFeatureTargetTracker : public BaseFeatureTracker
```

## Thuộc tính cần có

```cpp
private:
    std::vector<StereoFrameRecord> frame_records;
    std::vector<ROIConfig> roi_configs;

    CameraIntrinsics camera_intrinsics;
    Transform3D camera_to_robot_transform;
    FeatureTrackingConfig tracking_config;

    TargetTemplateFeatureSet template_feature_set;
    std::vector<FeatureMatchResult> match_results;
    std::vector<TargetTrackingState> tracking_states;

    static int tracker_instance_count;
```

---

## Destructor
Class này **phải có destructor** để bạn luyện đúng phần Đợt 5 C++ OOP.

Ví dụ:

```cpp
~StereoFeatureTargetTracker();
```

Bạn có thể dùng destructor để:
- log số frame đã xử lý
- giải phóng tài nguyên debug nếu bạn tự tạo thêm

---

## Nhóm hàm load config
- `load_frame_sequence_manifest(...)`
- `load_roi_config(...)`
- `load_camera_intrinsics_config(...)`
- `load_camera_to_robot_transform(...)`
- `load_feature_tracking_config(...)`

---

## Nhóm hàm feature extraction

### `cv::Ptr<cv::Feature2D> create_feature_detector() const;`
- dùng `this->tracking_config.feature_type`
- nếu `"ORB"` → tạo ORB
- nếu `"SIFT"` → tạo SIFT

### `TargetTemplateFeatureSet build_template_feature_set();`
**Hành vi**
1. đọc ảnh template
2. detect keypoints
3. compute descriptors
4. build `TargetTemplateFeatureSet`

### `FrameFeatureSet build_frame_feature_set(
    const std::string& pair_name,
    const cv::Mat& frame
) const;`

---

## Nhóm hàm matching

### `std::vector<cv::DMatch> match_descriptors(
    const cv::Mat& template_desc,
    const cv::Mat& frame_desc
) const;`

### `std::vector<cv::DMatch> filter_good_matches(
    const std::vector<cv::DMatch>& matches
) const;`
- giữ các match có `distance <= good_match_distance_threshold`

### `cv::Point2f estimate_target_center_from_matches(
    const std::vector<cv::KeyPoint>& frame_keypoints,
    const std::vector<cv::DMatch>& good_matches
) const;`
- lấy trung bình các điểm match trên frame

### `double compute_tracking_confidence(
    int good_matches,
    int total_template_keypoints
) const;`

---

## Nhóm hàm tracking decision

### `FeatureMatchResult build_match_result(
    const std::string& pair_name,
    const TargetTemplateFeatureSet& template_set,
    const FrameFeatureSet& frame_set
) const;`

### `TargetTrackingState build_tracking_state(
    const FeatureMatchResult& result
) const;`

## Rule gợi ý
- nếu `good_matches == 0` → `TARGET_LOST`
- nếu `tracking_confidence >= tracking_confidence_threshold` và `good_matches >= min_good_matches`
  → `TRACKING_STABLE`
- ngược lại → `TRACKING_WEAK`

---

## Overlay / visualization

### `void draw_tracking_overlay(
    cv::Mat& left_image,
    const FeatureMatchResult& match_result,
    const TargetTrackingState& tracking_state
) const;`

**Hành vi**
- vẽ target center ước lượng
- ghi:
  - `tracking_label`
  - `confidence`
  - `good_matches`

### `cv::Mat build_match_visualization(
    const cv::Mat& template_image,
    const cv::Mat& frame_image,
    const TargetTemplateFeatureSet& template_set,
    const FrameFeatureSet& frame_set,
    const std::vector<cv::DMatch>& good_matches
) const;`

---

## Hàm chính của pipeline

### `void process_single_frame_record(const StereoFrameRecord& record);`
**Hành vi tổng quát**
1. đọc frame trái / phải
2. build frame feature set
3. match với template
4. filter good matches
5. build `FeatureMatchResult`
6. build `TargetTrackingState`
7. vẽ overlay
8. tạo ảnh visualization match
9. lưu output
10. push result vào vector

### `void run_tracking() override;`
- build template feature set trước
- loop qua toàn bộ frame records

### Getter
```cpp
const std::vector<TargetTrackingState>& get_tracking_states() const;
```

---

# 11.18 C++ — `FeatureTrackingReportWriter`

**File:**

```text
cpp/include/FeatureTrackingReportWriter.hpp
cpp/src/FeatureTrackingReportWriter.cpp
```

Tạo class:

```cpp
class FeatureTrackingReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<TargetTrackingState>& tracking_states
);`

## Format gợi ý

```text
[Tracking Frame]
Pair Name: frame_01
Tracking Label: TRACKING_STABLE
Tracking Confidence: 0.71
Good Matches: 38
Estimated Target Center: (412.5, 233.0)

----------------------------------------
```

---

# 11.19 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- tạo `StereoFeatureTargetTracker`
- load toàn bộ config
- chạy `run_tracking()`
- tạo `FeatureTrackingReportWriter`
- ghi report ra:
  - `assets/outputs/feature_tracking_report.txt`

## Pipeline `main.cpp`

```text
Create StereoCameraSensor
→ Load Frame Sequence Manifest
→ Load ROI Config
→ Load Camera Intrinsics Config
→ Load Camera-to-Robot Transform Config
→ Load Feature Tracking Config
→ Run Feature Tracking Pipeline
→ Save Tracking Overlay + Match Visualization
→ Write Feature Tracking Report
```

---

# 12. Điều kiện bắt buộc

Project bắt buộc phải có:

- OOP trong Python
- OOP trong C++
- inheritance trong Python
- inheritance trong C++
- `super()` trong Python
- `@classmethod` hoặc `@staticmethod` trong Python
- destructor trong C++
- `this` trong C++ ở ít nhất một chỗ hợp lý
- `#pragma once`
- tách `hpp/cpp`
- function tách rõ
- module Python
- `loop`
- `if / else`
- `list` / `dict`
- `std::vector`
- detect feature
- compute descriptor
- match feature
- filter good matches
- build tracking state
- report scene/frame-level

---

# 13. Output mong muốn

## File config
```text
config/frame_sequence_manifest.txt
config/roi_config.txt
config/camera_intrinsics_config.txt
config/camera_to_robot_transform.txt
config/feature_tracking_config.txt
```

## Ảnh output
```text
assets/outputs/frame_01_tracking_overlay.jpg
assets/outputs/frame_01_match_visualization.jpg
assets/outputs/frame_02_tracking_overlay.jpg
assets/outputs/frame_02_match_visualization.jpg
```

## File report
```text
assets/outputs/feature_tracking_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo frame sequence manifest
- tạo ROI config
- tạo feature tracking config
- tạo intrinsics / transform config
- hỗ trợ report

Tức là Python làm phần:

```text
Feature Tracking Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- detect keypoints
- compute descriptors
- match target template với frame hiện tại
- tính confidence
- quyết định tracking state
- vẽ tracking overlay
- ghi report

Tức là C++ làm phần:

```text
Feature-Based Humanoid Target Tracking Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- tìm **keypoints / corners / descriptors**
- match đặc trưng giữa target và frame
- ước lượng target center
- đánh giá target còn được theo dõi ổn định hay không

Tức là CV làm phần:

```text
Template Features → Frame Features → Matching → Tracking Confidence → Tracking State
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ file config
- [ ] Python có class cha / class con
- [ ] Python có `classmethod` hoặc `staticmethod`
- [ ] C++ có `BaseFeatureTracker`
- [ ] C++ có `StereoFeatureTargetTracker`
- [ ] C++ có destructor
- [ ] C++ load đủ config
- [ ] C++ detect feature
- [ ] C++ compute descriptor
- [ ] C++ match feature
- [ ] C++ filter good matches
- [ ] C++ build tracking state
- [ ] C++ vẽ tracking overlay
- [ ] C++ ghi feature tracking report

---

# 16. Gợi ý mở rộng

## 1. Thêm ROI re-centering
Sau khi ước lượng target center, cập nhật ROI cho frame tiếp theo.

## 2. Thêm left-right stereo consistency
Kiểm tra target center ở cả ảnh trái và phải.

## 3. Thêm target motion direction
Ví dụ:
- `MOVING_LEFT`
- `MOVING_RIGHT`
- `APPROACHING`
- `LEAVING`

## 4. Chuẩn bị cho Bài 22
Sau Bài 21, bước hợp lý cho **Bài 22** là:

```text
Feature Matching + Homography Object Re-Localization
```

Tức là không chỉ biết target còn hay mất, mà còn bắt đầu **ước lượng vùng object mới bằng homography từ các feature matches**.
