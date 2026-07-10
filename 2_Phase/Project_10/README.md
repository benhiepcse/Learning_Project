# 🤖 Bài 10: Robot Stereo Pair Proposal Correspondence Starter — Ghép proposal giữa camera trái/phải cho Humanoid Robot AI Perception

> Mini Project số 10 trong **Đợt 2 — Bài 6 → Bài 10**  
> **Bài 10 là bài cuối của Đợt 2** và tiếp tục kết hợp kiến thức của **Đợt 1 + Đợt 2** theo đúng rule bạn đã chốt.  
> Nếu **Bài 9** giúp robot **liên kết proposal qua nhiều frame theo thời gian**, thì **Bài 10** sẽ đưa bạn sang một hướng cực kỳ quan trọng cho AI Perception của Humanoid Robot:  
> robot phải **liên kết proposal giữa ảnh camera trái và ảnh camera phải**, tức là bắt đầu làm **stereo correspondence ở mức vùng vật thể**.

---

# 📌 Mục lục

- [1. Mô tả](#1-mô-tả)
- [2. Bài 10 nằm ở đâu trong roadmap](#2-bài-10-nằm-ở-đâu-trong-roadmap)
- [3. Vì sao Bài 10 là bước tiếp theo hợp lý sau Bài 9](#3-vì-sao-bài-10-là-bước-tiếp-theo-hợp-lý-sau-bài-9)
- [4. Mục tiêu perception của bài](#4-mục-tiêu-perception-của-bài)
- [5. Pipeline perception của bài](#5-pipeline-perception-của-bài)
- [6. Kiến thức cần](#6-kiến-thức-cần)
  - [6.1 C++](#61-c)
  - [6.2 Python](#62-python)
  - [6.3 CV C++](#63-cv-c)
  - [6.4 CV Python](#64-cv-python)
- [7. Kiến thức Đợt 1 và Đợt 2 được dùng như thế nào](#7-kiến-thức-đợt-1-và-đợt-2-được-dùng-như-thế-nào)
- [8. Sau bài này bạn sẽ hiểu gì trong AI Perception](#8-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
  - [10.1 Python — BaseConfigBuilder](#101-python--baseconfigbuilder)
  - [10.2 Python — StereoProposalConfigBuilder](#102-python--stereoproposalconfigbuilder)
  - [10.3 Python — main_config_builder.py](#103-python--main_config_builderpy)
  - [10.4 C++ — BaseSensor](#104-c--basesensor)
  - [10.5 C++ — StereoCameraSensor](#105-c--stereocamerasensor)
  - [10.6 C++ — StereoFrameRecord](#106-c--stereoframerecord)
  - [10.7 C++ — ROIConfig](#107-c--roiconfig)
  - [10.8 C++ — ProposalConfig](#108-c--proposalconfig)
  - [10.9 C++ — StereoMatchingConfig](#109-c--stereomatchingconfig)
  - [10.10 C++ — ObjectProposal](#1010-c--objectproposal)
  - [10.11 C++ — StereoProposalMatch](#1011-c--stereoproposalmatch)
  - [10.12 C++ — StereoSceneResult](#1012-c--stereosceneresult)
  - [10.13 C++ — BaseStereoProposalMatcher](#1013-c--basestereoproposalmatcher)
  - [10.14 C++ — StereoProposalMatcher](#1014-c--stereoproposalmatcher)
  - [10.15 C++ — StereoReportWriter](#1015-c--stereoreportwriter)
  - [10.16 C++ — main.cpp](#1016-c--maincpp)
- [11. Điều kiện bắt buộc](#11-điều-kiện-bắt-buộc)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò của bài này trong Humanoid Robot](#13-vai-trò-của-bài-này-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Mô tả

Ở **Bài 9**, bạn đã có một module có thể:

- đọc chuỗi frame
- tạo proposal trong từng ROI
- liên kết proposal qua nhiều frame
- gán `track id`
- lưu annotated tracking frames + report

Bài 10 sẽ **không đi tiếp theo trục thời gian** nữa, mà chuyển sang **trục không gian stereo**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc **cặp ảnh stereo**:
  - ảnh trái
  - ảnh phải
- trên mỗi ảnh:
  - đọc ROI
  - build object proposals
- sau đó:
  - **ghép proposal trái ↔ proposal phải**
  - tính **disparity** sơ bộ ở mức proposal
  - lưu kết quả correspondence

Ví dụ robot nhìn một hộp màu đỏ bằng stereo camera:

- camera trái phát hiện proposal box ở `(xL, yL, w, h)`
- camera phải phát hiện proposal box ở `(xR, yR, w, h)`

thì module này phải cố gắng hiểu rằng:

```text
proposal_left_1  ↔  proposal_right_1
```

là **cùng một object trong không gian 3D**, chỉ khác vị trí do stereo baseline.

---

# 2. Bài 10 nằm ở đâu trong roadmap

## Quy ước hiện tại
- **Đợt 1 = Bài 1 → Bài 5**
- **Đợt 2 = Bài 6 → Bài 10**
- **Đợt 3 = Bài 11 → Bài 15**
- **Đợt 4 = Bài 16 → Bài 20**

Vì vậy:

## **Bài 10 = Bài cuối của Đợt 2**
và là bài **khép Đợt 2** trước khi bạn bước sang các bài sâu hơn về stereo / depth / 3D perception ở Đợt 3.

---

# 3. Vì sao Bài 10 là bước tiếp theo hợp lý sau Bài 9

## Bài 8 cho bạn:
- proposal trong ROI của từng ảnh

## Bài 9 cho bạn:
- proposal correspondence theo **thời gian**

## Bài 10 nâng thêm một nấc:
- proposal correspondence theo **hai camera trái / phải**

Nó rất hợp lý vì bạn vừa học xong:
- **correspondence theo time** (frame t ↔ frame t+1)

thì bây giờ chuyển sang:
- **correspondence theo stereo** (left ↔ right)

Về mặt tư duy, hai bài này rất gần nhau:

```text
Matching entity A ở ảnh 1 với entity B ở ảnh 2
```

Chỉ khác ở chỗ:
- Bài 9: ảnh 1 và ảnh 2 khác nhau theo **thời gian**
- Bài 10: ảnh 1 và ảnh 2 khác nhau theo **camera viewpoint**

---

# 4. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Pair Dataset + ROI Config + Proposal Config + Stereo Matching Config
→ Load Left / Right Image Pair
→ Build Left Proposals
→ Build Right Proposals
→ Compare Left Proposal with Right Proposal
→ Find Best Stereo Correspondence
→ Compute Proposal-Level Disparity
→ Save Annotated Stereo Pair + Report
```

Bài này giúp bạn hiểu một bước đệm rất quan trọng cho stereo AI perception:

> Trước khi đi tới disparity map dày đặc (dense disparity), robot hoàn toàn có thể bắt đầu từ **proposal-level stereo correspondence**.

---

# 5. Pipeline perception của bài

```text
Stereo Pair Config
→ Read Stereo Pair Records
→ Read ROI Config
→ Read Proposal Config
→ Read Stereo Matching Config
→ Create Stereo Camera Sensor Object
→ For Each Stereo Pair:
    → Build Left Proposals
    → Build Right Proposals
    → Compare Left / Right Proposal Geometry
    → Match Proposal Pairs
    → Compute Proposal Centers
    → Compute Proposal Disparity
    → Draw Left / Right Match Overlay
    → Save Annotated Stereo Outputs
→ Write Stereo Correspondence Report
```

---

# 6. Kiến thức cần

# 6.1 C++

- class / object
- constructor
- inheritance
- `std::vector`
- `std::string`
- `const`
- `auto`
- function
- if / else
- loop
- struct
- header / source tách file

---

# 6.2 Python

- class / object
- inheritance
- list
- dict
- string
- type casting
- function nhiều tham số
- file write
- loop
- if / else
- module

---

# 6.3 CV C++

- `cv::imread`
- `cv::imwrite`
- ROI bằng `cv::Rect`
- `cv::cvtColor`
- `cv::inRange`
- `cv::findContours`
- `cv::contourArea`
- `cv::boundingRect`
- `cv::rectangle`
- `cv::putText`

> Bài này **chưa bắt buộc** rectification, triangulation, disparity map dày đặc.  
> Mục tiêu là **stereo correspondence ở mức proposal box**, đúng với nhịp chuyển từ Đợt 2 sang Đợt 3.

---

# 6.4 CV Python

Python không phải runtime stereo matcher chính, nhưng sẽ dùng để:
- build stereo pair manifest
- build ROI config
- build proposal config
- build stereo matching config

---

# 7. Kiến thức Đợt 1 và Đợt 2 được dùng như thế nào

# 7.1 Phần lấy từ Đợt 1

## Python
- class / inheritance
- function
- if else / loop
- config builder style

## C++
- `BaseSensor`
- struct config / result
- class analyzer / matcher style

## CV
- threshold
- contour
- proposal box
- annotated output

---

# 7.2 Phần lấy từ Đợt 2

## Python
- list / dict mạnh hơn
- string split / join / replace
- parse config text

## C++
- vector để lưu:
  - nhiều stereo pairs
  - proposal trái / phải
  - stereo matches
- `const` / `auto`
- scene result / report style

## CV
- ROI
- proposal extraction
- multi-image processing
- pairwise correspondence logic

---

# 8. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 10, bạn phải nắm được 6 ý rất quan trọng:

## 1. Stereo correspondence không nhất thiết phải bắt đầu từ pixel-level disparity
Bạn có thể bắt đầu bằng:
- object proposal
- ROI
- center point
- bounding box matching

## 2. Left proposal và right proposal là hai “ảnh chiếu” của cùng object
Nếu ghép đúng cặp, bạn có thể tính:
- disparity sơ bộ
- hướng dịch chuyển trái/phải
- vị trí tương đối của object trong stereo pair

## 3. Stereo matching là một dạng correspondence problem
Nó rất giống bài tracking ở chỗ:
- phải tìm “đối tượng tương ứng” giữa 2 ảnh

## 4. Proposal-level disparity là bước đệm tốt trước dense disparity
Bạn chưa cần map disparity cho toàn bộ ảnh, nhưng đã hiểu:
- vật ở gần → disparity proposal thường lớn hơn
- vật ở xa → disparity proposal thường nhỏ hơn

## 5. Đây là cầu nối rất đẹp sang Đợt 3
Vì Đợt 3 của bạn sẽ đụng mạnh hơn tới:
- stereo geometry
- depth estimation
- 3D reconstruction
- back-projection

## 6. Robot perception thường đi theo nhiều mức
- mức ảnh
- mức ROI
- mức proposal
- mức tracking / stereo matching
- rồi mới tới 3D reasoning

Bài 10 chính là chỗ bạn chạm vào **proposal-level stereo reasoning**.

---

# 9. Cấu trúc folder

```text
mini_project_10_robot_stereo_pair_proposal_correspondence_starter/
│
├─ README.md
│
├─ assets/
│  ├─ stereo_pairs/
│  │  ├─ pair_01_left.jpg
│  │  ├─ pair_01_right.jpg
│  │  ├─ pair_02_left.jpg
│  │  ├─ pair_02_right.jpg
│  │  └─ ...
│  │
│  └─ outputs/
│     ├─ pair_01_left_annotated.jpg
│     ├─ pair_01_right_annotated.jpg
│     ├─ pair_02_left_annotated.jpg
│     ├─ pair_02_right_annotated.jpg
│     └─ stereo_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ roi_config.txt
│  ├─ proposal_config.txt
│  └─ stereo_matching_config.txt
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
   │  ├─ ProposalConfig.hpp
   │  ├─ StereoMatchingConfig.hpp
   │  ├─ ObjectProposal.hpp
   │  ├─ StereoProposalMatch.hpp
   │  ├─ StereoSceneResult.hpp
   │  ├─ BaseStereoProposalMatcher.hpp
   │  ├─ StereoProposalMatcher.hpp
   │  └─ StereoReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoProposalMatcher.cpp
      └─ StereoReportWriter.cpp
```

---

# 10. Yêu cầu mini-project

# 10.1 Python — `BaseConfigBuilder`

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
stereo_manifest_path
roi_config_path
proposal_config_path
stereo_matching_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn các file config

---

# 10.2 Python — `StereoProposalConfigBuilder`

**File:**

```text
python/tools/config_builder.py
```

Tạo class con:

```python
class StereoProposalConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
stereo_pairs
roi_regions
proposal_config
stereo_matching_config
```

---

## `stereo_pairs`
Là **list các dict**, ví dụ:

```python
[
    {
        "pair_name": "pair_01",
        "left_image_path": "assets/stereo_pairs/pair_01_left.jpg",
        "right_image_path": "assets/stereo_pairs/pair_01_right.jpg",
        "sensor_name": "head_stereo_camera",
        "sensor_id": 0
    },
    {
        "pair_name": "pair_02",
        "left_image_path": "assets/stereo_pairs/pair_02_left.jpg",
        "right_image_path": "assets/stereo_pairs/pair_02_right.jpg",
        "sensor_name": "head_stereo_camera",
        "sensor_id": 0
    }
]
```

## `roi_regions`
Là **list các dict**, mỗi dict mô tả một ROI template.

## `proposal_config`
Tương tự Bài 8 / Bài 9.

## `stereo_matching_config`
Là `dict`, ví dụ:

```python
{
    "matching_mode": "center_distance_and_size",
    "max_vertical_center_diff": 40.0,
    "max_width_diff": 60.0,
    "max_height_diff": 60.0,
    "max_disparity": 250.0
}
```

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
**Hành vi**
- thêm stereo pair record
- kiểm tra:
  - chuỗi không rỗng
  - `sensor_id >= 0`

### `add_roi_region(roi_name, x, y, width, height)`
- giống các bài trước

### `set_proposal_config(...)`
- giống Bài 8 / 9

### `set_stereo_matching_config(
    matching_mode,
    max_vertical_center_diff,
    max_width_diff,
    max_height_diff,
    max_disparity
)`
**Hành vi**
- lưu stereo matching config
- kiểm tra threshold hợp lệ

### `write_stereo_manifest()`
**Format gợi ý**
```text
pair_01|assets/stereo_pairs/pair_01_left.jpg|assets/stereo_pairs/pair_01_right.jpg|head_stereo_camera|0
pair_02|assets/stereo_pairs/pair_02_left.jpg|assets/stereo_pairs/pair_02_right.jpg|head_stereo_camera|0
```

### `write_roi_config()`
- giống các bài trước

### `write_proposal_config()`
- giống Bài 8 / 9

### `write_stereo_matching_config()`
**Format gợi ý**
```text
matching_mode=center_distance_and_size
max_vertical_center_diff=40
max_width_diff=60
max_height_diff=60
max_disparity=250
```

---

# 10.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 stereo pairs**
- tạo ít nhất **3 ROI template**
- set proposal config
- set stereo matching config
- ghi đủ:
  - `config/stereo_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/proposal_config.txt`
  - `config/stereo_matching_config.txt`

---

# 10.4 C++ — `BaseSensor`

**File:**

```text
cpp/include/BaseSensor.hpp
```

Tạo class:

```cpp
class BaseSensor
```

## Thuộc tính

```cpp
protected:
    std::string sensor_name;
```

## Hàm cần có

```cpp
BaseSensor(const std::string& name);
std::string get_name() const;
virtual void print_info() const;
```

---

# 10.5 C++ — `StereoCameraSensor`

**File:**

```text
cpp/include/StereoCameraSensor.hpp
cpp/src/StereoCameraSensor.cpp
```

Tạo class kế thừa:

```cpp
class StereoCameraSensor : public BaseSensor
```

## Thuộc tính cần có

```cpp
private:
    int stereo_id;
    std::string left_camera_name;
    std::string right_camera_name;
```

## Hàm cần có

```cpp
StereoCameraSensor(
    const std::string& sensor_name,
    int stereo_id,
    const std::string& left_camera_name,
    const std::string& right_camera_name
);

void print_info() const override;
```

---

# 10.6 C++ — `StereoFrameRecord`

**File:**

```text
cpp/include/StereoFrameRecord.hpp
```

Tạo struct:

```cpp
struct StereoFrameRecord
```

## Thuộc tính cần có

```cpp
std::string pair_name;
std::string left_image_path;
std::string right_image_path;
std::string sensor_name;
int sensor_id;
```

---

# 10.7 C++ — `ROIConfig`

**File:**

```text
cpp/include/ROIConfig.hpp
```

- giống các bài trước

---

# 10.8 C++ — `ProposalConfig`

**File:**

```text
cpp/include/ProposalConfig.hpp
```

- giống Bài 8 / 9

---

# 10.9 C++ — `StereoMatchingConfig`

**File:**

```text
cpp/include/StereoMatchingConfig.hpp
```

Tạo struct:

```cpp
struct StereoMatchingConfig
```

## Thuộc tính cần có

```cpp
std::string matching_mode;
double max_vertical_center_diff;
double max_width_diff;
double max_height_diff;
double max_disparity;
```

---

# 10.10 C++ — `ObjectProposal`

**File:**

```text
cpp/include/ObjectProposal.hpp
```

Tạo struct:

```cpp
struct ObjectProposal
```

## Thuộc tính cần có

```cpp
std::string pair_name;
std::string view_name;     // "left" hoặc "right"
std::string roi_name;

cv::Rect roi_box;
cv::Rect proposal_box_roi;
cv::Rect proposal_box_full_image;

double contour_area;
bool is_valid;
```

---

# 10.11 C++ — `StereoProposalMatch`

**File:**

```text
cpp/include/StereoProposalMatch.hpp
```

Tạo struct:

```cpp
struct StereoProposalMatch
```

## Thuộc tính cần có

```cpp
std::string pair_name;

ObjectProposal left_proposal;
ObjectProposal right_proposal;

cv::Point2d left_center;
cv::Point2d right_center;

double disparity;
bool is_valid_match;
```

> Gợi ý:
> ```text
> disparity = left_center.x - right_center.x
> ```
> Nếu ảnh đã roughly rectified, disparity chủ yếu nằm theo trục ngang.

---

# 10.12 C++ — `StereoSceneResult`

**File:**

```text
cpp/include/StereoSceneResult.hpp
```

Tạo struct:

```cpp
struct StereoSceneResult
```

## Thuộc tính cần có

```cpp
std::string pair_name;

std::string left_image_path;
std::string right_image_path;

std::string left_output_path;
std::string right_output_path;

std::string sensor_name;
int sensor_id;

int left_proposal_count;
int right_proposal_count;
int stereo_match_count;

bool is_valid;

std::vector<ObjectProposal> left_proposals;
std::vector<ObjectProposal> right_proposals;
std::vector<StereoProposalMatch> stereo_matches;
```

---

# 10.13 C++ — `BaseStereoProposalMatcher`

**File:**

```text
cpp/include/BaseStereoProposalMatcher.hpp
```

Tạo class trừu tượng:

```cpp
class BaseStereoProposalMatcher
```

## Hàm cần có

```cpp
virtual void load_stereo_manifest(const std::string& path) = 0;
virtual void load_roi_config(const std::string& path) = 0;
virtual void load_proposal_config(const std::string& path) = 0;
virtual void load_stereo_matching_config(const std::string& path) = 0;
virtual void run_stereo_matching() = 0;
virtual ~BaseStereoProposalMatcher() = default;
```

---

# 10.14 C++ — `StereoProposalMatcher`

**File:**

```text
cpp/include/StereoProposalMatcher.hpp
cpp/src/StereoProposalMatcher.cpp
```

Tạo class kế thừa:

```cpp
class StereoProposalMatcher : public BaseStereoProposalMatcher
```

## Thuộc tính cần có

```cpp
private:
    std::vector<StereoFrameRecord> stereo_records;
    std::vector<ROIConfig> roi_configs;
    ProposalConfig proposal_config;
    StereoMatchingConfig stereo_matching_config;

    std::vector<StereoSceneResult> stereo_results;
```

---

## Hàm cần có

### Load / Read config
- `read_stereo_manifest(...)`
- `read_roi_config(...)`
- `read_proposal_config(...)`
- `read_stereo_matching_config(...)`
- các hàm `load_...(...) override`

---

## Proposal building part

### `cv::Rect clamp_roi_to_image(const ROIConfig& roi_cfg, const cv::Mat& image) const;`
### `cv::Mat extract_roi_patch(const cv::Mat& image, const cv::Rect& roi_rect) const;`
### `cv::Mat build_proposal_mask(const cv::Mat& roi_patch_bgr) const;`
### `std::vector<std::vector<cv::Point>> extract_candidate_contours(const cv::Mat& proposal_mask) const;`
### `bool is_valid_candidate_contour(const std::vector<cv::Point>& contour) const;`

### `ObjectProposal build_object_proposal(
    const std::string& pair_name,
    const std::string& view_name,
    const std::string& roi_name,
    const cv::Rect& roi_rect,
    const std::vector<cv::Point>& contour
) const;`

### `std::vector<ObjectProposal> build_view_proposals(
    const std::string& pair_name,
    const std::string& view_name,
    const cv::Mat& image
) const;`

## Hành vi
- loop qua ROI
- build proposals cho view trái hoặc phải

---

## Stereo matching part

### `cv::Point2d get_box_center(const cv::Rect& box) const;`
- lấy tâm proposal box

### `double compute_vertical_center_diff(
    const ObjectProposal& left_proposal,
    const ObjectProposal& right_proposal
) const;`

### `double compute_width_diff(
    const ObjectProposal& left_proposal,
    const ObjectProposal& right_proposal
) const;`

### `double compute_height_diff(
    const ObjectProposal& left_proposal,
    const ObjectProposal& right_proposal
) const;`

### `double compute_disparity(
    const ObjectProposal& left_proposal,
    const ObjectProposal& right_proposal
) const;`

---

### `bool is_valid_stereo_match_candidate(
    const ObjectProposal& left_proposal,
    const ObjectProposal& right_proposal
) const;`

## Hành vi gợi ý
Một cặp proposal trái-phải được coi là match candidate nếu:
- `vertical_center_diff <= max_vertical_center_diff`
- `width_diff <= max_width_diff`
- `height_diff <= max_height_diff`
- `abs(disparity) <= max_disparity`

> Bạn có thể thêm rule:
> - disparity nên dương nếu giả sử object ở trước camera và ảnh đã rectified theo convention chuẩn.

---

### `StereoProposalMatch build_stereo_match(
    const std::string& pair_name,
    const ObjectProposal& left_proposal,
    const ObjectProposal& right_proposal
) const;`

### `std::vector<StereoProposalMatch> match_left_right_proposals(
    const std::string& pair_name,
    const std::vector<ObjectProposal>& left_proposals,
    const std::vector<ObjectProposal>& right_proposals
) const;`

## Hành vi tổng quát
1. loop qua từng proposal trái
2. tìm proposal phải phù hợp nhất theo rule matching
3. build `StereoProposalMatch`
4. tránh match trùng proposal phải nếu bạn muốn thiết kế one-to-one

---

### `void draw_view_proposals(
    cv::Mat& image,
    const std::vector<ObjectProposal>& proposals,
    const std::string& view_name
) const;`

### `void draw_stereo_match_labels(
    cv::Mat& left_image,
    cv::Mat& right_image,
    const std::vector<StereoProposalMatch>& matches
) const;`

## Hành vi
- vẽ proposal box lên ảnh trái / phải
- nếu match hợp lệ:
  - ghi ID match hoặc disparity lên cả hai ảnh

---

### `StereoSceneResult process_single_stereo_pair(const StereoFrameRecord& record);`
## Hành vi tổng quát
1. đọc ảnh trái và phải
2. nếu lỗi → result invalid
3. build proposals cho left
4. build proposals cho right
5. match proposal left-right
6. vẽ overlay
7. lưu ảnh annotated
8. build `StereoSceneResult`

---

### `void run_stereo_matching() override;`
- loop qua toàn bộ stereo pairs

### Getter

```cpp
const std::vector<StereoSceneResult>& get_stereo_results() const;
```

---

# 10.15 C++ — `StereoReportWriter`

**File:**

```text
cpp/include/StereoReportWriter.hpp
cpp/src/StereoReportWriter.cpp
```

Tạo class:

```cpp
class StereoReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<StereoSceneResult>& stereo_results
);`

## Format gợi ý

```text
[Stereo Pair]
Pair Name: pair_01
Left Image: assets/stereo_pairs/pair_01_left.jpg
Right Image: assets/stereo_pairs/pair_01_right.jpg
Left Output: assets/outputs/pair_01_left_annotated.jpg
Right Output: assets/outputs/pair_01_right_annotated.jpg
Sensor: head_stereo_camera
Left Proposal Count: 2
Right Proposal Count: 2
Stereo Match Count: 1
Valid: true

  [Match]
  Left ROI: center_region
  Right ROI: center_region
  Left Box: x=210, y=120, w=80, h=70
  Right Box: x=185, y=118, w=82, h=71
  Disparity: 25.0
  Valid Match: true

----------------------------------------
```

---

# 10.16 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- in thông tin stereo sensor
- tạo `StereoProposalMatcher`
- load:
  - `config/stereo_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/proposal_config.txt`
  - `config/stereo_matching_config.txt`
- chạy `run_stereo_matching()`
- tạo `StereoReportWriter`
- ghi report ra:
  - `assets/outputs/stereo_report.txt`

## Pipeline `main.cpp`

```text
Create StereoCameraSensor
→ Load Stereo Pair Manifest
→ Load ROI Config
→ Load Proposal Config
→ Load Stereo Matching Config
→ Run Stereo Proposal Matching
→ Save Annotated Stereo Pair Outputs
→ Write Stereo Correspondence Report
```

---

# 11. Điều kiện bắt buộc

Project bắt buộc phải có:

- OOP trong Python
- OOP trong C++
- Inheritance trong Python
- Inheritance trong C++
- Function tách rõ
- Module Python
- Header / Source C++ tách file
- `loop`
- `if / else`
- `list` / `dict`
- `std::vector`
- nhiều stereo pairs từ manifest
- proposal builder cho view trái / phải
- stereo left-right matching
- disparity sơ bộ ở mức proposal
- stereo report

---

# 12. Output mong muốn

## File config
```text
config/stereo_pair_manifest.txt
config/roi_config.txt
config/proposal_config.txt
config/stereo_matching_config.txt
```

## Ảnh output
```text
assets/outputs/pair_01_left_annotated.jpg
assets/outputs/pair_01_right_annotated.jpg
assets/outputs/pair_02_left_annotated.jpg
assets/outputs/pair_02_right_annotated.jpg
```

## File report
```text
assets/outputs/stereo_report.txt
```

---

## Ví dụ `stereo_pair_manifest.txt`

```text
pair_01|assets/stereo_pairs/pair_01_left.jpg|assets/stereo_pairs/pair_01_right.jpg|head_stereo_camera|0
pair_02|assets/stereo_pairs/pair_02_left.jpg|assets/stereo_pairs/pair_02_right.jpg|head_stereo_camera|0
pair_03|assets/stereo_pairs/pair_03_left.jpg|assets/stereo_pairs/pair_03_right.jpg|head_stereo_camera|0
```

---

## Ví dụ `stereo_matching_config.txt`

```text
matching_mode=center_distance_and_size
max_vertical_center_diff=40
max_width_diff=60
max_height_diff=60
max_disparity=250
```

---

## Ví dụ `stereo_report.txt`

```text
[Stereo Pair]
Pair Name: pair_01
Left Image: assets/stereo_pairs/pair_01_left.jpg
Right Image: assets/stereo_pairs/pair_01_right.jpg
Left Output: assets/outputs/pair_01_left_annotated.jpg
Right Output: assets/outputs/pair_01_right_annotated.jpg
Sensor: head_stereo_camera
Left Proposal Count: 2
Right Proposal Count: 2
Stereo Match Count: 1
Valid: true

  [Match]
  Left ROI: center_region
  Right ROI: center_region
  Left Box: x=210, y=120, w=80, h=70
  Right Box: x=185, y=118, w=82, h=71
  Disparity: 25.0
  Valid Match: true

----------------------------------------
```

---

# 13. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo **stereo pair manifest**
- tạo **ROI config**
- tạo **proposal config**
- tạo **stereo matching config**
- chuẩn bị metadata cho module C++

Tức là Python làm phần:

```text
Stereo Pair Perception Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- build proposal cho ảnh trái / phải
- match proposal trái-phải
- tính disparity sơ bộ
- lưu annotated stereo outputs + report

Tức là C++ làm phần:

```text
Stereo Proposal Correspondence Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **trích proposal ở từng view**
- **so khớp proposal giữa hai camera**
- **biến proposal trái/phải thành stereo match**
- **đưa ra disparity sơ bộ cho object candidate**

Tức là CV làm phần:

```text
Left/Right Object Proposals → Stereo Correspondence → Proposal-Level Disparity
```

---

# 14. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `stereo_pair_manifest.txt`
- [ ] Python tạo được `roi_config.txt`
- [ ] Python tạo được `proposal_config.txt`
- [ ] Python tạo được `stereo_matching_config.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / string / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `StereoCameraSensor`
- [ ] C++ có `StereoFrameRecord`
- [ ] C++ có `ROIConfig`
- [ ] C++ có `ProposalConfig`
- [ ] C++ có `StereoMatchingConfig`
- [ ] C++ có `ObjectProposal`
- [ ] C++ có `StereoProposalMatch`
- [ ] C++ có `StereoSceneResult`
- [ ] C++ có `BaseStereoProposalMatcher`
- [ ] C++ có `StereoProposalMatcher`
- [ ] C++ load được stereo manifest
- [ ] C++ load được ROI config
- [ ] C++ load được proposal config
- [ ] C++ load được stereo matching config
- [ ] C++ build được proposals cho view trái / phải
- [ ] C++ match được proposal trái-phải
- [ ] C++ tính được disparity sơ bộ
- [ ] C++ vẽ được stereo overlay
- [ ] C++ lưu được annotated stereo outputs
- [ ] C++ build được stereo report

---

# 15. Gợi ý mở rộng

## 1. Thêm ROI riêng cho left / right nếu muốn
Hiện tại có thể dùng chung ROI template cho cả 2 ảnh.  
Sau này bạn có thể cho phép:
- ROI riêng cho ảnh trái
- ROI riêng cho ảnh phải

## 2. Thêm score matching tốt hơn
Ví dụ score dựa trên:
- vertical center diff
- width / height diff
- area similarity
- color similarity giữa proposal trái và phải

## 3. Kết hợp feature descriptor trong proposal
Bạn có thể mở rộng:
- trích ORB/SIFT feature trong proposal box
- so khớp descriptor trái / phải

## 4. Chuẩn bị cho Đợt 3
Sau Bài 10, hướng rất hợp lý cho **Đợt 3** là đi tiếp sang:
- stereo geometry
- disparity reasoning
- depth estimation
- point / object depth từ stereo

Bài 10 chính là “cầu nối” để bạn bước sang các bài đó mượt hơn.

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 10**, bạn sẽ khép **Đợt 2** bằng một bài rất đúng chất Humanoid Robot AI Perception:

- **Bài 6**: quản lý nhiều ảnh + metadata + transform
- **Bài 7**: ROI scene analysis
- **Bài 8**: object proposal trong ROI
- **Bài 9**: proposal tracking qua nhiều frame
- **Bài 10**: **stereo left-right proposal correspondence**

Tức là bạn đã đi từ:

```text
ảnh đơn → ROI → proposal → temporal tracking → stereo proposal matching
```

Đây là nền rất đẹp để sang **Đợt 3**, nơi bạn có thể bắt đầu chạm mạnh hơn vào:
- stereo vision thật sự
- disparity / depth
- triangulation
- 3D perception cho robot.
