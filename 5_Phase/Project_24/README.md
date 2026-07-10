# 🤖 Bài 24: Scene Change Detection after Registration — Phát hiện thay đổi của scene sau khi đã căn chỉnh bằng Image Registration cho Humanoid Robot AI Perception

> Mini Project số 24 trong **Đợt 5 — Bài 21 → Bài 25**  
> **Bài 24 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5**.  
> Nếu **Bài 23** giúp robot **căn chỉnh current scene về reference scene** bằng feature matching + homography, thì **Bài 24** sẽ đi tiếp đúng bước kế tiếp trong perception:
>
> **so sánh scene đã align để phát hiện vùng thay đổi, vật mới xuất hiện, vật biến mất hoặc vật bị di chuyển.**

---

# 📌 Mục lục

- [1. Bài 24 lấy gì từ Đợt 5](#1-bài-24-lấy-gì-từ-đợt-5)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 24 nằm ở đâu trong roadmap](#3-bài-24-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 24 là bước tiếp theo hợp lý sau Bài 23](#4-vì-sao-bài-24-là-bước-tiếp-theo-hợp-lý-sau-bài-23)
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

# 1. Bài 24 lấy gì từ Đợt 5

Bài 24 tiếp tục đúng chuỗi của Đợt 5:

- **Bài 21** → Feature-based target tracking  
- **Bài 22** → Feature matching + homography object re-localization  
- **Bài 23** → Image registration & scene alignment  
- **Bài 24** → **Scene change detection after registration**

Bài 24 lấy trực tiếp các kiến thức của Đợt 5:

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
- feature matching
- homography
- image registration
- difference map
- threshold
- morphology
- contour / connected component
- scene change region analysis

---

# 2. Mô tả

Ở **Bài 23**, robot đã có thể:

- đọc **reference scene** và **current scene**
- detect features ở cả 2 ảnh
- match descriptors
- estimate homography
- warp ảnh current về hệ của ảnh reference
- tạo overlap / difference preview
- tính registration quality

Nhưng Bài 23 mới dừng ở:

```text
scene đã align tốt hay chưa
```

Bài 24 sẽ đi tiếp một bước rất quan trọng:

```text
scene sau khi align đã thay đổi ở đâu
```

Mini-project này yêu cầu bạn xây một hệ thống để robot:

- đọc scene pair
- registration current → reference
- tạo **difference map**
- threshold difference map để lấy **change mask**
- lọc nhiễu bằng morphology
- tìm **change regions** bằng contour / bounding box
- phân loại scene:
  - gần như không đổi
  - thay đổi nhỏ
  - thay đổi đáng kể
- ghi report + lưu ảnh visualization

Bài này cực kỳ sát với các bài toán thực tế của humanoid / household robot, ví dụ:

- robot quay lại bàn ăn và muốn biết **có vật nào mới xuất hiện**
- robot so sánh trước / sau khi dọn bàn
- robot kiểm tra **scene có khác với map tham chiếu không**

---

# 3. Bài 24 nằm ở đâu trong roadmap

## Quy ước mini-project

- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**

Vì vậy:

## **Bài 24 = bài thứ tư của Đợt 5**

và phải kết hợp lại toàn bộ kiến thức của:

```text
Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5
```

---

# 4. Vì sao Bài 24 là bước tiếp theo hợp lý sau Bài 23

## Bài 23 cho bạn:
- scene registration
- overlap visualization
- difference map cơ bản
- registration score

## Bài 24 thêm:
- threshold difference
- change mask
- morphology cleanup
- contour / region extraction
- scene change summary

Tức là chuyển từ:

```text
“2 scene đã căn chỉnh được chưa?”
```

sang:

```text
“sau khi căn chỉnh, scene đã thay đổi cái gì?”
```

Đây là đúng nhịp phát triển perception vì sau khi align xong, bước hợp lý nhất là **change detection**.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Reference Scene + Current Scene + Registration Config + Change Detection Config
→ Register Current Scene to Reference Scene
→ Build Difference Map
→ Threshold Difference Map
→ Morphological Cleanup
→ Extract Change Regions
→ Compute Change Statistics
→ Build Scene Change State
→ Save Visualization + Report
```

---

# 6. Pipeline perception của bài

```text
Scene Pair Manifest
→ Read Registration Config
→ Read Change Detection Config
→ For Each Scene Pair:
    → Load reference image
    → Load current image
    → Detect keypoints
    → Compute descriptors
    → Match descriptors
    → Estimate homography
    → Warp current scene to reference frame
    → Build absolute difference map
    → Threshold difference map to binary mask
    → Apply morphology open/close
    → Find contours / connected regions
    → Filter small regions
    → Build change bounding boxes
    → Compute total changed area / region count
    → Decide scene change state
    → Save aligned image + difference map + binary mask + overlay
→ Write scene change report
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
- feature matching
- homography
- warp perspective
- absolute difference
- thresholding
- morphology open / close
- contour / bounding box
- scene change statistics

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 + 5 được dùng như thế nào

## Đợt 1
- class / function / loop / if else
- đọc / ghi ảnh cơ bản

## Đợt 2
- ROI / grayscale / resize / file config
- list / dict / vector

## Đợt 3
- threshold
- morphology
- contour / connected component
- segmentation kiểu cơ bản

## Đợt 4
- scene report / multi-stage pipeline
- output scene summary để phục vụ follow / pick / cleaning

## Đợt 5
- feature matching
- homography
- registration
- change detection after registration

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 24, bạn phải nắm được 12 ý quan trọng:

## 1. Registration và change detection thường đi cùng nhau
Nếu scene chưa align tốt thì change detection sẽ rất nhiễu.

## 2. Difference map chưa đủ để dùng trực tiếp
Cần threshold và lọc nhiễu.

## 3. Morphology rất quan trọng trong change mask
Open / close giúp bỏ vùng lẻ tẻ và nối vùng thay đổi hợp lý hơn.

## 4. Bounding box change region là output rất hữu ích
Robot có thể biết “vùng nào của bàn đã thay đổi”.

## 5. Changed area là một tín hiệu scene-level quan trọng
Ví dụ bàn thay đổi 2% khác hẳn với 35%.

## 6. Scene change detection có thể dùng cho household robotics
Ví dụ:
- vật mới xuất hiện trên bàn
- vật bị lấy đi
- vị trí vật thay đổi

## 7. Không phải mọi khác biệt đều là object change
Có thể do ánh sáng, registration chưa tốt, blur.

## 8. Nên tách registration quality và change quality
Nếu registration fail thì change result cũng đáng nghi.

## 9. Đây là bước đệm tốt cho memory-based scene understanding
Robot có thể so current scene với scene “đã biết trước”.

## 10. Đây cũng là bước đệm cho anomaly detection / inspection
Ví dụ robot kiểm tra khu vực có gì khác thường.

## 11. Change region có thể được đưa sang ROI cho pick / clean
Sau khi biết vùng thay đổi, robot có thể zoom vào đó.

## 12. Đây là một bài rất thực dụng cho humanoid perception
Vì robot trong nhà thường phải so scene trước / sau.

---

# 10. Cấu trúc folder

```text
mini_project_24_scene_change_detection_after_registration/
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
│     ├─ pair_01_change_overlay.jpg
│     ├─ pair_02_aligned_current.jpg
│     ├─ pair_02_difference_map.jpg
│     ├─ pair_02_change_mask.jpg
│     ├─ pair_02_change_overlay.jpg
│     └─ scene_change_report.txt
│
├─ config/
│  ├─ scene_pair_manifest.txt
│  ├─ registration_config.txt
│  └─ change_detection_config.txt
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
   │  ├─ RegistrationConfig.hpp
   │  ├─ ChangeDetectionConfig.hpp
   │  ├─ SceneFeatureSet.hpp
   │  ├─ RegistrationMatchResult.hpp
   │  ├─ ChangeRegion.hpp
   │  ├─ SceneChangeState.hpp
   │  ├─ BaseSceneChangeDetector.hpp
   │  ├─ RegistrationSceneChangeDetector.hpp
   │  └─ SceneChangeReportWriter.hpp
   │
   └─ src/
      ├─ RegistrationSceneChangeDetector.cpp
      └─ SceneChangeReportWriter.cpp
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
registration_config_path
change_detection_config_path
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

# 11.2 Python — `SceneChangeConfigBuilder`

Tạo class con:

```python
class SceneChangeConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
scene_pairs
registration_config
change_detection_config
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
    "min_change_area": 300,
    "small_change_area_threshold": 2000,
    "large_change_area_threshold": 8000
}
```

## Hàm cần có

### `add_scene_pair(pair_name, reference_image_path, current_image_path)`
- thêm pair

### `set_registration_config(...)`
- validate như Bài 23

### `set_change_detection_config(...)`
**Hành vi**
- `difference_threshold >= 0`
- `morph_open_kernel > 0`
- `morph_close_kernel > 0`
- `min_change_area > 0`
- `small_change_area_threshold >= min_change_area`
- `large_change_area_threshold >= small_change_area_threshold`

### `write_scene_pair_manifest()`
Format gợi ý:

```text
pair_name=pair_01
reference_image=assets/scene_pairs/pair_01_reference.jpg
current_image=assets/scene_pairs/pair_01_current.jpg
```

### `write_registration_config()`

### `write_change_detection_config()`
Format gợi ý:

```text
difference_threshold=25
morph_open_kernel=3
morph_close_kernel=5
min_change_area=300
small_change_area_threshold=2000
large_change_area_threshold=8000
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 scene pairs**
- set registration config
- set change detection config
- ghi đủ:
  - `config/scene_pair_manifest.txt`
  - `config/registration_config.txt`
  - `config/change_detection_config.txt`

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
};
```

---

# 11.6 C++ — `ChangeDetectionConfig`

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
    int small_change_area_threshold;
    int large_change_area_threshold;
};
```

---

# 11.7 C++ — `SceneFeatureSet`

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

# 11.8 C++ — `RegistrationMatchResult`

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

# 11.9 C++ — `ChangeRegion`

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

# 11.10 C++ — `SceneChangeState`

**File:**

```text
cpp/include/SceneChangeState.hpp
```

```cpp
struct SceneChangeState
{
    std::string pair_name;
    std::string state_label; // NO_SIGNIFICANT_CHANGE / SMALL_CHANGE / LARGE_CHANGE / REGISTRATION_FAILED

    int change_region_count;
    double total_changed_area;
    double inlier_ratio;
    bool is_valid;
};
```

---

# 11.11 C++ — `BaseSceneChangeDetector`

**File:**

```text
cpp/include/BaseSceneChangeDetector.hpp
```

```cpp
class BaseSceneChangeDetector
{
public:
    virtual void load_scene_pair_manifest(const std::string& path) = 0;
    virtual void load_registration_config(const std::string& path) = 0;
    virtual void load_change_detection_config(const std::string& path) = 0;
    virtual void run_change_detection() = 0;
    virtual ~BaseSceneChangeDetector() = default;
};
```

---

# 11.12 C++ — `RegistrationSceneChangeDetector`

**File:**

```text
cpp/include/RegistrationSceneChangeDetector.hpp
cpp/src/RegistrationSceneChangeDetector.cpp
```

Class kế thừa:

```cpp
class RegistrationSceneChangeDetector : public BaseSceneChangeDetector
```

## Thuộc tính cần có

```cpp
private:
    std::vector<ScenePairRecord> scene_pairs;
    RegistrationConfig registration_config;
    ChangeDetectionConfig change_config;

    std::vector<RegistrationMatchResult> registration_results;
    std::vector<SceneChangeState> change_states;

    static int detector_instance_count;
```

## Destructor bắt buộc

```cpp
~RegistrationSceneChangeDetector();
```

---

## Nhóm hàm load config
- `load_scene_pair_manifest(...)`
- `load_registration_config(...)`
- `load_change_detection_config(...)`

---

## Nhóm hàm registration

### `cv::Ptr<cv::Feature2D> create_feature_detector() const;`
- ORB hoặc SIFT

### `SceneFeatureSet build_scene_feature_set(...) const;`

### `std::vector<cv::DMatch> match_descriptors(...) const;`

### `std::vector<cv::DMatch> filter_good_matches(...) const;`

### `RegistrationMatchResult build_registration_result(
    const ScenePairRecord& pair,
    const SceneFeatureSet& ref_features,
    const SceneFeatureSet& cur_features
) const;`

### `cv::Mat warp_current_to_reference(
    const cv::Mat& current_image,
    const cv::Mat& homography,
    const cv::Size& output_size
) const;`

---

## Nhóm hàm change detection

### `cv::Mat build_difference_map(
    const cv::Mat& reference_image,
    const cv::Mat& aligned_current
) const;`

### `cv::Mat build_binary_change_mask(
    const cv::Mat& difference_map
) const;`
- threshold difference map

### `cv::Mat apply_morphology_cleanup(
    const cv::Mat& binary_mask
) const;`
- morphology open / close

### `std::vector<ChangeRegion> extract_change_regions(
    const cv::Mat& cleaned_mask
) const;`
- contour hoặc connected component
- bỏ vùng nhỏ hơn `min_change_area`

### `double compute_total_changed_area(
    const std::vector<ChangeRegion>& regions
) const;`

### `SceneChangeState build_scene_change_state(
    const std::string& pair_name,
    const RegistrationMatchResult& registration_result,
    const std::vector<ChangeRegion>& regions
) const;`

## Rule gợi ý
- nếu registration fail → `REGISTRATION_FAILED`
- nếu `total_changed_area < small_change_area_threshold`
  → `NO_SIGNIFICANT_CHANGE`
- nếu `small_change_area_threshold <= total_changed_area < large_change_area_threshold`
  → `SMALL_CHANGE`
- nếu `total_changed_area >= large_change_area_threshold`
  → `LARGE_CHANGE`

---

## Visualization

### `cv::Mat draw_change_overlay(
    const cv::Mat& aligned_current,
    const std::vector<ChangeRegion>& regions,
    const SceneChangeState& state
) const;`

**Hành vi**
- vẽ bbox cho từng change region
- ghi:
  - `state_label`
  - `change_region_count`
  - `total_changed_area`

---

## Hàm chính

### `void process_single_scene_pair(const ScenePairRecord& pair);`

1. đọc reference image
2. đọc current image
3. build registration result
4. nếu registration valid:
   - warp current
   - difference map
   - binary mask
   - morphology cleanup
   - extract change regions
   - build state
   - save aligned / diff / mask / overlay
5. nếu registration fail:
   - build state fail
6. push state

### `void run_change_detection() override;`
- loop qua toàn bộ scene pairs

### Getter

```cpp
const std::vector<SceneChangeState>& get_change_states() const;
```

---

# 11.13 C++ — `SceneChangeReportWriter`

**File:**

```text
cpp/include/SceneChangeReportWriter.hpp
cpp/src/SceneChangeReportWriter.cpp
```

Tạo class:

```cpp
class SceneChangeReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<SceneChangeState>& states
);`

Format gợi ý:

```text
[Scene Change Pair]
Pair Name: pair_01
State: SMALL_CHANGE
Change Region Count: 3
Total Changed Area: 2540
Inlier Ratio: 0.61
Valid: true

----------------------------------------
```

---

# 11.14 C++ — `main.cpp`

## Yêu cầu
- tạo `RegistrationSceneChangeDetector`
- load:
  - `config/scene_pair_manifest.txt`
  - `config/registration_config.txt`
  - `config/change_detection_config.txt`
- chạy `run_change_detection()`
- tạo `SceneChangeReportWriter`
- ghi report:
  - `assets/outputs/scene_change_report.txt`

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
- homography / registration
- difference map
- threshold
- morphology
- contour / change region extraction
- change report

---

# 13. Output mong muốn

## File config

```text
config/scene_pair_manifest.txt
config/registration_config.txt
config/change_detection_config.txt
```

## Ảnh output

```text
assets/outputs/pair_01_aligned_current.jpg
assets/outputs/pair_01_difference_map.jpg
assets/outputs/pair_01_change_mask.jpg
assets/outputs/pair_01_change_overlay.jpg
```

## File report

```text
assets/outputs/scene_change_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Scene Change Detection Config Builder
```

Dùng để tạo:
- scene pair manifest
- registration config
- change detection config

## C++ đóng vai trò gì?

C++ là:

```text
Registration + Scene Change Detection Runtime
```

Dùng để:
- register current scene về reference scene
- build difference map
- detect change regions
- build scene change summary

## Computer Vision đóng vai trò gì?

CV là:

```text
Scene Registration
→ Difference
→ Threshold
→ Morphology
→ Change Regions
→ Scene Change State
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ config
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có `staticmethod` / `classmethod`
- [ ] C++ có `BaseSceneChangeDetector`
- [ ] C++ có `RegistrationSceneChangeDetector`
- [ ] C++ có destructor
- [ ] C++ dùng `this`
- [ ] C++ dùng `static`
- [ ] C++ build registration
- [ ] C++ warp current scene
- [ ] C++ build difference map
- [ ] C++ threshold difference
- [ ] C++ morphology cleanup
- [ ] C++ extract change regions
- [ ] C++ build scene change state
- [ ] C++ vẽ overlay
- [ ] C++ ghi report

---

# 16. Gợi ý mở rộng

## 1. Thêm phân loại “new object / removed object / moved object”
Thay vì chỉ biết “có thay đổi”, cố gắng tách loại thay đổi.

## 2. Thêm ROI-only change detection
Chỉ kiểm tra vùng tabletop hoặc shelf.

## 3. Thêm change confidence
Kết hợp registration quality + changed area + number of regions.

## 4. Chuẩn bị cho Bài 25
Sau Bài 24, bước rất hợp lý là:

```text
Object-Level Change Analysis for Humanoid Tabletop Scene
```

Tức là từ change region của scene, bắt đầu gắn chúng với object-level logic:
- object nào mới xuất hiện
- object nào bị lấy đi
- object nào bị dời vị trí
- object nào nên pick / clean / inspect
