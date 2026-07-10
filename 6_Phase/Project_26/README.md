# 🤖 Bài 26: Stereo Feature Registration Workbench — Bàn thực hành ghép cặp feature + homography + stereo verification cho Humanoid Robot AI Perception

> Mini Project số 26 trong **Đợt 6**  
> **Bài 26 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5 + Đợt 6**.  
> Đây là bài mở đầu cho **Đợt 6 — Feature Extractor Project**. Sau khi Đợt 5 đã đi đến mức **object-level change analysis**, Đợt 6 quay lại siết chặt một khối cực quan trọng của perception runtime:
>
> **feature extraction + feature matching + homography + image registration**, nhưng lần này được đóng gói theo kiểu **stereo-aware workbench** để phục vụ robot perception thực tế.

---

# 📌 Mục lục

- [1. Bài 26 lấy gì từ Đợt 6](#1-bài-26-lấy-gì-từ-đợt-6)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 26 nằm ở đâu trong roadmap](#3-bài-26-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 26 là bước mở đầu hợp lý cho Đợt 6](#4-vì-sao-bài-26-là-bước-mở-đầu-hợp-lý-cho-đợt-6)
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

# 1. Bài 26 lấy gì từ Đợt 6

Theo roadmap, **Đợt 6 (Ngày 11–12)** có chủ đề **Feature extractor project** và giới hạn kiến thức như sau:

## Python — Phase 7: Project Structure
- `Python Virtual Environments`
- `Python Modules`
- `if __name__ == "__main__"`

## C++ — Phase 5 + Phase 6
- `C++ Pass By Reference`
- `C++ Pointers`
  - pointer cơ bản
  - dereference
  - `nullptr`
- `C++ Inheritance`
  - constructor trong inheritance
- `C++ Virtual Functions`
  - `virtual`
  - `override`

## Computer Vision — Phase 3
- `Feature Matching`
- `Homography`
- `Image Registration`

Nguồn roadmap đợt 6 nằm ở phần “Feature extractor project” của file roadmap 4 tuần fileciteturn9file0

Bài 26 vì vậy sẽ bám đúng tinh thần đó:  
**không đi sang camera geometry hay stereo depth nặng**, mà tập trung xây một **feature registration workbench** đủ “robotic” để dùng lại cho các bài sau.

---

# 2. Mô tả

Ở **Đợt 5**, bạn đã làm những project thiên về:

- feature tracking
- homography relocalization
- image registration
- scene change detection
- object-level change analysis

Nhưng các bài đó đi theo **mục tiêu perception task**.  
Còn **Bài 26** sẽ quay về xây một **workbench lõi** cho perception runtime, chuyên để:

- trích feature từ ảnh trái / phải / reference / current
- match feature theo nhiều chế độ
- estimate homography giữa các ảnh
- kiểm tra chất lượng registration
- lưu lại kết quả để dùng cho các pipeline robot khác

Nói ngắn gọn:

```text
Bài 25 = perception reasoning theo scene / object
Bài 26 = dựng lại "engine" feature registration sạch, tái sử dụng được
```

<p align="center">
  <img src="images/project_26.png" width="800">
</p>

---

# 3. Bài 26 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**
- **Đợt 5 = Bài 21 → 25**
- **Đợt 6 = Bài 26 → 30**

Vì vậy:

## **Bài 26 = bài đầu tiên của Đợt 6**

và phải kết hợp lại toàn bộ kiến thức của:

```text
Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4 + Đợt 5 + Đợt 6
```

---

# 4. Vì sao Bài 26 là bước mở đầu hợp lý cho Đợt 6

Đợt 6 không còn chỉ học “dùng feature” nữa, mà bắt đầu học cách **tổ chức project feature** đúng kiểu runtime:

- Python quản lý project config / manifest / output profile
- C++ quản lý feature extractor / matcher / registrar bằng OOP + virtual functions
- CV xử lý:
  - feature extraction
  - feature matching
  - homography
  - registration quality

Tức là chuyển từ kiểu:

```text
"có một bài toán perception, mình dùng feature để giải"
```

sang:

```text
"mình xây một module feature registration sạch, có thể tái sử dụng cho nhiều bài toán perception"
```

Đây là bước rất đúng sau Đợt 5.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Frame Pair + Reference Pair + Feature Config + Registration Config
→ Build Left / Right / Reference / Current Feature Sets
→ Match Features in Selected Mode
→ Estimate Homography
→ Warp / Register if needed
→ Score Match Quality
→ Save Match Visualization + Registration Visualization + Feature Report
```

---

# 6. Pipeline perception của bài

```text
Python Config Builder
→ build stereo frame manifest
→ build registration pair manifest
→ build feature workbench config

C++ Runtime
→ load manifests
→ load feature config
→ create feature extractor
→ extract features from:
    - left image
    - right image
    - reference image
    - current image
→ run matching in 2 modes:
    1. stereo_pair_match
    2. registration_pair_match
→ filter good matches
→ if registration mode:
    → estimate homography
    → warp current to reference
    → compute registration score
→ build workbench record
→ save outputs
→ write final report
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module structure
- `if __name__ == "__main__"`
- file handling
- dict / list
- config builder

## C++
- class / inheritance
- constructor trong inheritance
- `virtual`, `override`
- pointers cơ bản
- pass by reference
- `nullptr`
- destructor
- `this`
- `static`
- `.hpp / .cpp`

## Computer Vision
- ORB / SIFT feature extraction
- descriptor matching
- good match filtering
- homography
- image registration
- match visualization

---

# 8. Đợt 1 → Đợt 6 được dùng như thế nào

## Đợt 1
- class / function / loop / if else
- đọc ảnh và in thông tin ảnh

## Đợt 2
- list / dict / vector
- ROI, resize, crop, color conversion

## Đợt 3
- menu logic, threshold / filter nếu muốn debug ảnh trước khi extract feature

## Đợt 4
- file handling, exception, contour / edge mindset cho debug quality

## Đợt 5
- OOP rõ hơn
- feature detection
- homography
- image registration
- report / overlay / workbench state

## Đợt 6
- project structure
- module Python
- virtual functions trong C++
- pointer / reference
- feature extractor project đúng nghĩa

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 26, bạn phải nắm được 12 ý quan trọng:

## 1. Feature extractor và feature matcher nên được tách thành module rõ ràng
Không nên nhét tất cả logic vào `main.cpp`.

## 2. Một perception workbench tốt cần hỗ trợ nhiều mode
Ví dụ:
- stereo left ↔ right
- reference ↔ current
- template ↔ frame

## 3. `virtual` / `override` trong C++ rất hợp để xây perception runtime
Bạn có thể thay extractor / matcher / registrar về sau.

## 4. Python ở đây không phải để chạy perception chính
Nó đóng vai trò:
- config builder
- manifest builder
- report helper

## 5. Homography không phải lúc nào cũng cần cho stereo pair
Nhưng rất hữu ích cho registration pair.

## 6. Feature matching là một “công cụ nền”
Nó có thể được dùng lại cho:
- relocalization
- registration
- change detection
- visual odometry sau này

## 7. Một workbench nên lưu cả intermediate outputs
Ví dụ:
- keypoint count
- total matches
- good matches
- inlier ratio
- registration score

## 8. Pointer / reference trong C++ giúp runtime gọn hơn
Đặc biệt khi truyền config, feature sets, result records.

## 9. Cần phân biệt “extract feature” và “reason about scene”
Bài 26 thiên về **feature engine**, chưa thiên về scene reasoning như Bài 25.

## 10. Đây là bước nối rất đẹp trước khi đi sâu hơn vào camera geometry / stereo / calibration
Vì nó siết lại khối feature trước khi bạn dùng nó ở bài toán lớn hơn.

## 11. Workbench kiểu này rất hợp để dùng lại trong ROS2 node sau này
Bạn có thể biến nó thành:
- feature matching node
- registration node
- visual relocalization node

## 12. Đây là bài mở đầu rất đúng cho Đợt 6
Vì nó vừa học **project structure**, vừa học **virtual OOP C++**, vừa dùng đúng **Feature Matching + Homography + Registration** của roadmap đợt 6 fileciteturn9file0

---

# 10. Cấu trúc folder

```text
mini_project_26_stereo_feature_registration_workbench/
│
├─ README.md
│
├─ assets/
│  ├─ stereo_pairs/
│  │  ├─ stereo_01_left.jpg
│  │  ├─ stereo_01_right.jpg
│  │  ├─ stereo_02_left.jpg
│  │  ├─ stereo_02_right.jpg
│  │  └─ ...
│  │
│  ├─ registration_pairs/
│  │  ├─ reg_01_reference.jpg
│  │  ├─ reg_01_current.jpg
│  │  ├─ reg_02_reference.jpg
│  │  ├─ reg_02_current.jpg
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ stereo_01_match.jpg
│     ├─ stereo_02_match.jpg
│     ├─ reg_01_match.jpg
│     ├─ reg_01_registered.jpg
│     ├─ reg_01_overlap.jpg
│     ├─ reg_02_match.jpg
│     ├─ reg_02_registered.jpg
│     ├─ reg_02_overlap.jpg
│     └─ feature_workbench_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ registration_pair_manifest.txt
│  └─ feature_workbench_config.txt
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
   │  ├─ StereoPairRecord.hpp
   │  ├─ RegistrationPairRecord.hpp
   │  ├─ FeatureWorkbenchConfig.hpp
   │  ├─ FeatureSet.hpp
   │  ├─ MatchResult.hpp
   │  ├─ RegistrationResult.hpp
   │  ├─ FeatureWorkbenchRecord.hpp
   │  ├─ BaseFeatureExtractor.hpp
   │  ├─ ORBFeatureExtractor.hpp
   │  ├─ BaseFeatureWorkbench.hpp
   │  ├─ StereoFeatureRegistrationWorkbench.hpp
   │  └─ FeatureWorkbenchReportWriter.hpp
   │
   └─ src/
      ├─ ORBFeatureExtractor.cpp
      ├─ StereoFeatureRegistrationWorkbench.cpp
      └─ FeatureWorkbenchReportWriter.cpp
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
stereo_pair_manifest_path
registration_pair_manifest_path
feature_workbench_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in các path config

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

### `@staticmethod validate_image_extension(path)`
- kiểm tra `.jpg`, `.jpeg`, `.png`

---

# 11.2 Python — `FeatureWorkbenchConfigBuilder`

Tạo class con:

```python
class FeatureWorkbenchConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
stereo_pairs
registration_pairs
feature_workbench_config
```

## `feature_workbench_config` ví dụ

```python
{
    "feature_type": "ORB",
    "max_features": 1200,
    "match_mode": "BF_HAMMING",
    "good_match_distance_threshold": 45.0,
    "min_good_matches_for_registration": 20,
    "ransac_reproj_threshold": 4.0,
    "save_match_visualization": True,
    "save_registration_visualization": True
}
```

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path)`
- thêm stereo pair
- validate extension

### `add_registration_pair(pair_name, reference_image_path, current_image_path)`
- thêm registration pair
- validate extension

### `set_feature_workbench_config(...)`
**Hành vi**
- `feature_type` thuộc `"ORB"` hoặc `"SIFT"`
- `max_features > 0`
- `good_match_distance_threshold > 0`
- `min_good_matches_for_registration >= 4`
- `ransac_reproj_threshold > 0`

### `write_stereo_pair_manifest()`
Format gợi ý:

```text
pair_name=stereo_01
left_image=assets/stereo_pairs/stereo_01_left.jpg
right_image=assets/stereo_pairs/stereo_01_right.jpg
```

### `write_registration_pair_manifest()`
Format gợi ý:

```text
pair_name=reg_01
reference_image=assets/registration_pairs/reg_01_reference.jpg
current_image=assets/registration_pairs/reg_01_current.jpg
```

### `write_feature_workbench_config()`
Format gợi ý:

```text
feature_type=ORB
max_features=1200
match_mode=BF_HAMMING
good_match_distance_threshold=45.0
min_good_matches_for_registration=20
ransac_reproj_threshold=4.0
save_match_visualization=true
save_registration_visualization=true
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 stereo pairs**
- tạo ít nhất **3 registration pairs**
- set feature workbench config
- ghi đủ:
  - `config/stereo_pair_manifest.txt`
  - `config/registration_pair_manifest.txt`
  - `config/feature_workbench_config.txt`

---

# 11.4 C++ — `StereoPairRecord`

**File:**

```text
cpp/include/StereoPairRecord.hpp
```

```cpp
struct StereoPairRecord
{
    std::string pair_name;
    std::string left_image_path;
    std::string right_image_path;
};
```

---

# 11.5 C++ — `RegistrationPairRecord`

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

# 11.6 C++ — `FeatureWorkbenchConfig`

**File:**

```text
cpp/include/FeatureWorkbenchConfig.hpp
```

```cpp
struct FeatureWorkbenchConfig
{
    std::string feature_type;
    int max_features;
    std::string match_mode;
    double good_match_distance_threshold;
    int min_good_matches_for_registration;
    double ransac_reproj_threshold;
    bool save_match_visualization;
    bool save_registration_visualization;
};
```

---

# 11.7 C++ — `FeatureSet`

**File:**

```text
cpp/include/FeatureSet.hpp
```

```cpp
struct FeatureSet
{
    std::string image_name;
    cv::Mat image;
    std::vector<cv::KeyPoint> keypoints;
    cv::Mat descriptors;
    bool is_valid;
};
```

---

# 11.8 C++ — `MatchResult`

**File:**

```text
cpp/include/MatchResult.hpp
```

```cpp
struct MatchResult
{
    std::string pair_name;
    std::string pair_type; // STEREO / REGISTRATION

    int total_matches;
    int good_matches;

    std::vector<cv::DMatch> good_match_list;
    bool is_valid;
};
```

---

# 11.9 C++ — `RegistrationResult`

**File:**

```text
cpp/include/RegistrationResult.hpp
```

```cpp
struct RegistrationResult
{
    std::string pair_name;

    int good_matches;
    int inlier_matches;
    double inlier_ratio;

    cv::Mat homography;
    cv::Mat registered_image;
    bool registration_valid;
};
```

---

# 11.10 C++ — `FeatureWorkbenchRecord`

**File:**

```text
cpp/include/FeatureWorkbenchRecord.hpp
```

```cpp
struct FeatureWorkbenchRecord
{
    std::string pair_name;
    std::string pair_type; // STEREO / REGISTRATION

    int left_or_reference_keypoints;
    int right_or_current_keypoints;
    int total_matches;
    int good_matches;

    bool registration_attempted;
    bool registration_valid;
    double inlier_ratio;
};
```

---

# 11.11 C++ — `BaseFeatureExtractor`

**File:**

```text
cpp/include/BaseFeatureExtractor.hpp
```

```cpp
class BaseFeatureExtractor
{
public:
    virtual FeatureSet extract(
        const std::string& image_name,
        const cv::Mat& image
    ) = 0;

    virtual ~BaseFeatureExtractor() = default;
};
```

---

# 11.12 C++ — `ORBFeatureExtractor`

**File:**

```text
cpp/include/ORBFeatureExtractor.hpp
cpp/src/ORBFeatureExtractor.cpp
```

Class kế thừa:

```cpp
class ORBFeatureExtractor : public BaseFeatureExtractor
```

## Thuộc tính cần có

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
FeatureSet extract(
    const std::string& image_name,
    const cv::Mat& image
) override;
```

## Yêu cầu kỹ thuật
- dùng `cv::ORB::create(max_features)`
- convert ảnh sang grayscale nếu cần
- detect + compute descriptors
- trả `FeatureSet`

---

# 11.13 C++ — `BaseFeatureWorkbench`

**File:**

```text
cpp/include/BaseFeatureWorkbench.hpp
```

```cpp
class BaseFeatureWorkbench
{
public:
    virtual void load_stereo_pair_manifest(const std::string& path) = 0;
    virtual void load_registration_pair_manifest(const std::string& path) = 0;
    virtual void load_feature_workbench_config(const std::string& path) = 0;
    virtual void run_workbench() = 0;
    virtual ~BaseFeatureWorkbench() = default;
};
```

---

# 11.14 C++ — `StereoFeatureRegistrationWorkbench`

**File:**

```text
cpp/include/StereoFeatureRegistrationWorkbench.hpp
cpp/src/StereoFeatureRegistrationWorkbench.cpp
```

Class kế thừa:

```cpp
class StereoFeatureRegistrationWorkbench : public BaseFeatureWorkbench
```

## Thuộc tính cần có

```cpp
private:
    std::vector<StereoPairRecord> stereo_pairs;
    std::vector<RegistrationPairRecord> registration_pairs;
    FeatureWorkbenchConfig config;

    BaseFeatureExtractor* extractor;

    std::vector<FeatureWorkbenchRecord> records;

    static int workbench_instance_count;
```

## Constructor / Destructor

```cpp
StereoFeatureRegistrationWorkbench();
~StereoFeatureRegistrationWorkbench();
```

### Destructor phải làm gì?
- nếu `extractor != nullptr` thì `delete extractor`
- log số records đã xử lý

---

## Nhóm hàm load config
- `load_stereo_pair_manifest(...)`
- `load_registration_pair_manifest(...)`
- `load_feature_workbench_config(...)`

---

## Nhóm hàm khởi tạo extractor

### `void initialize_extractor();`
**Hành vi**
- nếu `feature_type == "ORB"` → tạo `new ORBFeatureExtractor(...)`
- nếu chưa hỗ trợ `SIFT`, bạn vẫn phải có nhánh kiểm tra và log rõ

---

## Nhóm hàm matching

### `std::vector<cv::DMatch> match_descriptors(
    const cv::Mat& descriptors_a,
    const cv::Mat& descriptors_b
) const;`

### `std::vector<cv::DMatch> filter_good_matches(
    const std::vector<cv::DMatch>& matches
) const;`

### `MatchResult build_match_result(
    const std::string& pair_name,
    const std::string& pair_type,
    const FeatureSet& first_features,
    const FeatureSet& second_features
) const;`

---

## Nhóm hàm registration

### `RegistrationResult build_registration_result(
    const RegistrationPairRecord& pair,
    const FeatureSet& reference_features,
    const FeatureSet& current_features,
    const MatchResult& match_result
) const;`

## Hành vi tổng quát
1. nếu good matches < `min_good_matches_for_registration` → fail
2. tạo vector points từ good matches
3. `cv::findHomography(..., cv::RANSAC, ransac_reproj_threshold, mask)`
4. tính inlier ratio
5. nếu homography hợp lệ:
   - `cv::warpPerspective(current, registered_image, H, reference.size())`
6. trả `RegistrationResult`

---

## Visualization

### `cv::Mat build_match_visualization(
    const FeatureSet& first_features,
    const FeatureSet& second_features,
    const std::vector<cv::DMatch>& good_matches
) const;`

### `cv::Mat build_registration_overlap(
    const cv::Mat& reference_image,
    const cv::Mat& registered_image
) const;`

---

## Process từng pair

### `void process_stereo_pair(const StereoPairRecord& pair);`
1. load left/right image
2. extract left/right features
3. build match result
4. save match image nếu config cho phép
5. push `FeatureWorkbenchRecord`

### `void process_registration_pair(const RegistrationPairRecord& pair);`
1. load reference/current image
2. extract features
3. build match result
4. build registration result
5. save:
   - match image
   - registered image
   - overlap image
6. push `FeatureWorkbenchRecord`

---

## Hàm chính

### `void run_workbench() override;`
1. `initialize_extractor()`
2. loop stereo pairs
3. loop registration pairs

### Getter

```cpp
const std::vector<FeatureWorkbenchRecord>& get_records() const;
```

---

# 11.15 C++ — `FeatureWorkbenchReportWriter`

**File:**

```text
cpp/include/FeatureWorkbenchReportWriter.hpp
cpp/src/FeatureWorkbenchReportWriter.cpp
```

Tạo class:

```cpp
class FeatureWorkbenchReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<FeatureWorkbenchRecord>& records
);`

Format gợi ý:

```text
[Feature Workbench Pair]
Pair Name: stereo_01
Pair Type: STEREO
Left/Reference Keypoints: 812
Right/Current Keypoints: 776
Total Matches: 812
Good Matches: 164
Registration Attempted: false
Registration Valid: false
Inlier Ratio: 0.0

----------------------------------------
```

---

# 11.16 C++ — `main.cpp`

## Yêu cầu
- tạo `StereoFeatureRegistrationWorkbench`
- load:
  - `config/stereo_pair_manifest.txt`
  - `config/registration_pair_manifest.txt`
  - `config/feature_workbench_config.txt`
- chạy `run_workbench()`
- tạo `FeatureWorkbenchReportWriter`
- ghi report ra:
  - `assets/outputs/feature_workbench_report.txt`

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
- destructor C++
- `virtual` / `override`
- pointer cơ bản trong C++
- `nullptr`
- pass by reference
- `this`
- `static`
- `#pragma once`
- tách `.hpp / .cpp`
- feature extraction
- feature matching
- homography
- registration
- report workbench

---

# 13. Output mong muốn

## File config

```text
config/stereo_pair_manifest.txt
config/registration_pair_manifest.txt
config/feature_workbench_config.txt
```

## Ảnh output

```text
assets/outputs/stereo_01_match.jpg
assets/outputs/reg_01_match.jpg
assets/outputs/reg_01_registered.jpg
assets/outputs/reg_01_overlap.jpg
```

## File report

```text
assets/outputs/feature_workbench_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?

Python là:

```text
Feature Workbench Config Builder
```

Dùng để tạo:
- stereo pair manifest
- registration pair manifest
- feature workbench config

## C++ đóng vai trò gì?

C++ là:

```text
Feature Extraction + Matching + Registration Runtime
```

Dùng để:
- extract feature
- match feature
- estimate homography
- warp current image
- ghi report runtime

## Computer Vision đóng vai trò gì?

CV là:

```text
Stereo / Registration Images
→ Feature Extraction
→ Descriptor Matching
→ Homography
→ Registration
→ Quality Report
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ 3 file config
- [ ] Python có class cha / class con
- [ ] Python có `super()`
- [ ] Python có module + `if __name__ == "__main__"`
- [ ] C++ có `BaseFeatureExtractor`
- [ ] C++ có `ORBFeatureExtractor`
- [ ] C++ có `BaseFeatureWorkbench`
- [ ] C++ có `StereoFeatureRegistrationWorkbench`
- [ ] C++ có destructor
- [ ] C++ có pointer `extractor`
- [ ] C++ có `virtual` / `override`
- [ ] C++ extract feature
- [ ] C++ match descriptors
- [ ] C++ estimate homography
- [ ] C++ warp registration pair
- [ ] C++ save match / overlap image
- [ ] C++ ghi report

---

# 16. Gợi ý mở rộng

## 1. Thêm `SIFTFeatureExtractor`
Giữ nguyên interface `BaseFeatureExtractor`, chỉ thay extractor.

## 2. Thêm `KNN + ratio test`
Thay vì threshold distance đơn giản.

## 3. Tách `BaseMatcher`
Để sau này có:
- BFMatcher
- FLANN matcher

## 4. Chuẩn bị cho Bài 27
Sau Bài 26, bước rất hợp lý là:

```text
Feature Match Graph for Multi-Frame Visual Memory
```

Tức là từ chỗ match từng cặp ảnh, bạn nâng lên:
- nhiều frame
- lưu quan hệ match giữa các frame
- tạo graph trí nhớ thị giác cho robot
