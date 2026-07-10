# 🤖 Bài 9: Robot Multi-Frame Proposal Tracker Starter — Theo dõi object proposal qua nhiều frame cho Humanoid Robot AI Perception

> Mini Project số 9 trong **Đợt 2 — Bài 6 → Bài 10**  
> **Bài 9 tiếp tục kết hợp kiến thức của Đợt 1 + Đợt 2** theo đúng rule bạn đã chốt.  
> Nếu **Bài 8** giúp robot tạo **candidate object proposals trong từng ROI của từng ảnh**, thì **Bài 9** sẽ tiến thêm một bước rất quan trọng trong AI Perception:  
> robot phải **liên kết proposal giữa nhiều frame liên tiếp**, tức là từ các proposal rời rạc ở từng ảnh, hệ thống phải bắt đầu hiểu **proposal nào ở frame trước tương ứng với proposal nào ở frame sau**.

---

# 📌 Mục lục

- [1. Mô tả](#1-mô-tả)
- [2. Bài 9 nằm ở đâu trong roadmap](#2-bài-9-nằm-ở-đâu-trong-roadmap)
- [3. Vì sao Bài 9 là bước tiếp theo hợp lý sau Bài 8](#3-vì-sao-bài-9-là-bước-tiếp-theo-hợp-lý-sau-bài-8)
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
  - [10.2 Python — ProposalTrackingConfigBuilder](#102-python--proposaltrackingconfigbuilder)
  - [10.3 Python — main_config_builder.py](#103-python--main_config_builderpy)
  - [10.4 C++ — BaseSensor](#104-c--basesensor)
  - [10.5 C++ — CameraSensor](#105-c--camerasensor)
  - [10.6 C++ — FrameRecord](#106-c--framerecord)
  - [10.7 C++ — ROIConfig](#107-c--roiconfig)
  - [10.8 C++ — ProposalConfig](#108-c--proposalconfig)
  - [10.9 C++ — TrackingConfig](#109-c--trackingconfig)
  - [10.10 C++ — ObjectProposal](#1010-c--objectproposal)
  - [10.11 C++ — ProposalTrack](#1011-c--proposaltrack)
  - [10.12 C++ — FrameTrackingResult](#1012-c--frametrackingresult)
  - [10.13 C++ — BaseProposalTracker](#1013-c--baseproposaltracker)
  - [10.14 C++ — MultiFrameProposalTracker](#1014-c--multiframeproposaltracker)
  - [10.15 C++ — TrackingReportWriter](#1015-c--trackingreportwriter)
  - [10.16 C++ — main.cpp](#1016-c--maincpp)
- [11. Điều kiện bắt buộc](#11-điều-kiện-bắt-buộc)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò của bài này trong Humanoid Robot](#13-vai-trò-của-bài-này-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Mô tả

Ở **Bài 8**, bạn đã có một module có thể:

- đọc nhiều scene images từ manifest
- đọc ROI config
- trong mỗi ROI:
  - tạo mask
  - tìm contour
  - lọc contour
  - tạo **object proposal bounding box**
- map proposal box từ ROI sang full image
- lưu annotated scene và report proposal

Bài 9 sẽ **không dừng ở mức “mỗi frame có các proposal nào”** nữa.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc **một chuỗi frame liên tiếp**
- với mỗi frame:
  - tạo proposal trong ROI như Bài 8
- sau đó:
  - so khớp proposal giữa **frame t** và **frame t+1**
  - gán **track id**
  - cập nhật track qua nhiều frame
- lưu ảnh annotated có:
  - proposal box
  - track id
- ghi report tracking

Ví dụ robot nhìn một vật màu đỏ di chuyển qua 4 frame:

- frame 1 có proposal box A
- frame 2 có proposal box B gần vị trí A
- frame 3 có proposal box C tiếp tục di chuyển
- frame 4 có proposal box D

thì module này phải hiểu rằng:

```text
A → B → C → D
```

là **cùng một track** chứ không phải 4 object độc lập.

<p align="center">
  <img src="images/project_9.png" width="800">
</p>

---

# 2. Bài 9 nằm ở đâu trong roadmap

## Quy ước hiện tại
- **Đợt 1 = Bài 1 → Bài 5**
- **Đợt 2 = Bài 6 → Bài 10**
- **Đợt 3 = Bài 11 → Bài 15**
- **Đợt 4 = Bài 16 → Bài 20**

Vì vậy:

## **Bài 9 = Bài thứ tư của Đợt 2**
và vẫn **kết hợp kiến thức Đợt 1 + Đợt 2**.

---

# 3. Vì sao Bài 9 là bước tiếp theo hợp lý sau Bài 8

## Bài 6 cho bạn:
- quản lý nhiều ảnh
- transform ảnh đầu vào
- metadata sensor

## Bài 7 cho bạn:
- phân tích ROI
- thống kê màu / brightness
- chọn vùng interesting

## Bài 8 cho bạn:
- tạo object proposal trong ROI

## Bài 9 nâng thêm một nấc:
- không chỉ biết proposal của **từng frame riêng lẻ**
- mà còn bắt đầu biết **proposal correspondence qua thời gian**

Đây là một bước rất quan trọng vì perception của robot không sống trên ảnh tĩnh, mà thường là:

```text
video / stream / frame sequence
```

---

# 4. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Frame Sequence + ROI Config + Proposal Config + Tracking Config
→ Load Ordered Frames
→ Build Proposals for Each Frame
→ Compare Proposals Between Consecutive Frames
→ Associate Proposal to Existing Track or Create New Track
→ Update Track History
→ Draw Proposal Boxes + Track IDs
→ Save Annotated Tracking Frames + Tracking Report
```

Bài này giúp bạn hiểu một module rất quan trọng trong AI Perception:

> Từ **candidate object proposals theo từng frame**, robot bắt đầu xây **temporal identity** cho vật thể.

---

# 5. Pipeline perception của bài

```text
Sensor Frame Sequence Config
→ Read Frame Records
→ Read ROI Config
→ Read Proposal Config
→ Read Tracking Config
→ Create Camera Sensor Object
→ For Each Frame:
    → Extract ROI Patch
    → Build Proposal Mask
    → Find Candidate Contours
    → Build Object Proposals
→ For Consecutive Frames:
    → Compare Proposal Boxes
    → Compute IoU / Center Distance
    → Assign Track ID
    → Update Proposal Track
→ Draw Proposal + Track Overlay
→ Save Annotated Frames
→ Write Tracking Report
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

> Bài này chưa bắt buộc optical flow hay Kalman filter.  
> Tracking sẽ đi theo hướng **proposal matching đơn giản bằng box geometry** để đúng với kiến thức Đợt 1 + Đợt 2.

---

# 6.4 CV Python

Python không phải runtime tracker chính, nhưng sẽ dùng để:
- build frame manifest
- build ROI config
- build proposal config
- build tracking config

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
- `CameraSensor`
- struct kết quả
- class detector / analyzer / tracker style

## CV
- color threshold
- contour extraction
- bounding box
- annotated output

---

# 7.2 Phần lấy từ Đợt 2

## Python
- list / dict mạnh hơn
- string split / join / replace
- type casting khi parse config text

## C++
- vector để lưu:
  - nhiều frame
  - nhiều proposal
  - nhiều track
- `const` / `auto`
- struct metadata / config

## CV
- ROI
- scene-level processing
- nhiều frame liên tiếp
- overlay tracking ID

---

# 8. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 9, bạn phải nắm được 6 ý rất quan trọng:

## 1. Proposal của từng frame chưa đủ
Robot còn cần biết proposal nào là **cùng một vật thể theo thời gian**.

## 2. Tracking có thể bắt đầu rất đơn giản
Không cần deep learning hay Kalman ngay từ đầu.  
Bạn có thể bắt đầu bằng:
- IoU giữa 2 bounding box
- khoảng cách tâm box
- rule matching đơn giản

## 3. Track ID là lớp thông tin mới
Proposal box chỉ nói:
- “có một candidate object ở đây”

Track ID nói thêm:
- “candidate này là continuation của object trước đó”

## 4. Tracking giúp perception ổn định hơn
Nếu một object xuất hiện trong nhiều frame, robot có thể:
- theo dõi chuyển động
- ước lượng hướng di chuyển
- giảm nhiễu từ frame đơn lẻ

## 5. Đây là bước đệm tốt cho visual tracking / MOT / stereo temporal reasoning
Sau này bạn học:
- optical flow
- feature tracking
- multi-object tracking
- stereo sequence processing

thì tư duy của Bài 9 sẽ rất hữu ích.

## 6. Bài 9 là cầu nối rất đẹp giữa ROI proposal và stereo / depth
Bởi vì nó bắt đầu dạy robot:
- so sánh vùng ảnh giữa **hai thời điểm**
thay vì chỉ nhìn **một ảnh độc lập**.

---

# 9. Cấu trúc folder

```text
mini_project_09_robot_multi_frame_proposal_tracker_starter/
│
├─ README.md
│
├─ assets/
│  ├─ raw_frames/
│  │  ├─ frame_0001.jpg
│  │  ├─ frame_0002.jpg
│  │  ├─ frame_0003.jpg
│  │  └─ frame_0004.jpg
│  │
│  └─ outputs/
│     ├─ tracked_frame_0001.jpg
│     ├─ tracked_frame_0002.jpg
│     ├─ tracked_frame_0003.jpg
│     ├─ tracked_frame_0004.jpg
│     └─ tracking_report.txt
│
├─ config/
│  ├─ frame_sequence_manifest.txt
│  ├─ roi_config.txt
│  ├─ proposal_config.txt
│  └─ tracking_config.txt
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
   │  ├─ CameraSensor.hpp
   │  ├─ FrameRecord.hpp
   │  ├─ ROIConfig.hpp
   │  ├─ ProposalConfig.hpp
   │  ├─ TrackingConfig.hpp
   │  ├─ ObjectProposal.hpp
   │  ├─ ProposalTrack.hpp
   │  ├─ FrameTrackingResult.hpp
   │  ├─ BaseProposalTracker.hpp
   │  ├─ MultiFrameProposalTracker.hpp
   │  └─ TrackingReportWriter.hpp
   │
   └─ src/
      ├─ CameraSensor.cpp
      ├─ MultiFrameProposalTracker.cpp
      └─ TrackingReportWriter.cpp
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
frame_manifest_path
roi_config_path
proposal_config_path
tracking_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn các file config

---

# 10.2 Python — `ProposalTrackingConfigBuilder`

**File:**

```text
python/tools/config_builder.py
```

Tạo class con:

```python
class ProposalTrackingConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
frame_records
roi_regions
proposal_config
tracking_config
```

---

## `frame_records`
Là **list các dict**, ví dụ:

```python
[
    {
        "frame_index": 1,
        "frame_name": "frame_0001",
        "image_path": "assets/raw_frames/frame_0001.jpg",
        "sensor_name": "head_rgb_camera",
        "sensor_id": 0
    },
    {
        "frame_index": 2,
        "frame_name": "frame_0002",
        "image_path": "assets/raw_frames/frame_0002.jpg",
        "sensor_name": "head_rgb_camera",
        "sensor_id": 0
    }
]
```

## `roi_regions`
Là **list các dict**, mỗi dict mô tả một ROI template.

## `proposal_config`
Tương tự Bài 8, ví dụ:

```python
{
    "analysis_color_space": "HSV",
    "proposal_mode": "target_color_mask",
    "target_color_label": "red",
    "lower_h": 0,
    "lower_s": 120,
    "lower_v": 70,
    "upper_h": 10,
    "upper_s": 255,
    "upper_v": 255,
    "min_contour_area": 400,
    "min_box_width": 20,
    "min_box_height": 20
}
```

## `tracking_config`
Là `dict`, ví dụ:

```python
{
    "association_mode": "iou_center_distance",
    "min_iou_threshold": 0.15,
    "max_center_distance": 80.0,
    "max_missing_frames": 1
}
```

---

## Hàm cần có

### `add_frame_record(frame_index, frame_name, image_path, sensor_name, sensor_id)`
**Hành vi**
- thêm frame record
- kiểm tra:
  - `frame_index > 0`
  - `sensor_id >= 0`
  - chuỗi không rỗng

### `add_roi_region(roi_name, x, y, width, height)`
- giống Bài 8

### `set_proposal_config(...)`
- giống Bài 8

### `set_tracking_config(
    association_mode,
    min_iou_threshold,
    max_center_distance,
    max_missing_frames
)`
**Hành vi**
- lưu tracking config
- kiểm tra:
  - `association_mode` hợp lệ
  - `min_iou_threshold >= 0`
  - `max_center_distance >= 0`
  - `max_missing_frames >= 0`

### `write_frame_manifest()`
**Format gợi ý**
```text
1|frame_0001|assets/raw_frames/frame_0001.jpg|head_rgb_camera|0
2|frame_0002|assets/raw_frames/frame_0002.jpg|head_rgb_camera|0
3|frame_0003|assets/raw_frames/frame_0003.jpg|head_rgb_camera|0
```

### `write_roi_config()`
- giống Bài 8

### `write_proposal_config()`
- giống Bài 8

### `write_tracking_config()`
**Format gợi ý**
```text
association_mode=iou_center_distance
min_iou_threshold=0.15
max_center_distance=80
max_missing_frames=1
```

---

# 10.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **4 frame records**
- tạo ít nhất **3 ROI template**
- set proposal config
- set tracking config
- ghi đủ:
  - `config/frame_sequence_manifest.txt`
  - `config/roi_config.txt`
  - `config/proposal_config.txt`
  - `config/tracking_config.txt`

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

# 10.5 C++ — `CameraSensor`

**File:**

```text
cpp/include/CameraSensor.hpp
cpp/src/CameraSensor.cpp
```

Tạo class kế thừa:

```cpp
class CameraSensor : public BaseSensor
```

## Thuộc tính cần có

```cpp
private:
    int camera_id;
    std::string camera_role;
```

## Hàm cần có

```cpp
CameraSensor(const std::string& name, int id, const std::string& role);
void print_info() const override;
```

---

# 10.6 C++ — `FrameRecord`

**File:**

```text
cpp/include/FrameRecord.hpp
```

Tạo struct:

```cpp
struct FrameRecord
```

## Thuộc tính cần có

```cpp
int frame_index;
std::string frame_name;
std::string image_path;
std::string sensor_name;
int sensor_id;
```

---

# 10.7 C++ — `ROIConfig`

**File:**

```text
cpp/include/ROIConfig.hpp
```

- giống Bài 8

---

# 10.8 C++ — `ProposalConfig`

**File:**

```text
cpp/include/ProposalConfig.hpp
```

- giống Bài 8

---

# 10.9 C++ — `TrackingConfig`

**File:**

```text
cpp/include/TrackingConfig.hpp
```

Tạo struct:

```cpp
struct TrackingConfig
```

## Thuộc tính cần có

```cpp
std::string association_mode;
double min_iou_threshold;
double max_center_distance;
int max_missing_frames;
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
int frame_index;
std::string frame_name;
std::string roi_name;

cv::Rect roi_box;
cv::Rect proposal_box_roi;
cv::Rect proposal_box_full_image;

double contour_area;
int assigned_track_id;
bool is_valid;
```

> `assigned_track_id = -1` nếu chưa được gán track.

---

# 10.11 C++ — `ProposalTrack`

**File:**

```text
cpp/include/ProposalTrack.hpp
```

Tạo struct:

```cpp
struct ProposalTrack
```

## Thuộc tính cần có

```cpp
int track_id;
int last_seen_frame_index;
int missing_frame_count;
std::vector<ObjectProposal> history;
```

---

# 10.12 C++ — `FrameTrackingResult`

**File:**

```text
cpp/include/FrameTrackingResult.hpp
```

Tạo struct:

```cpp
struct FrameTrackingResult
```

## Thuộc tính cần có

```cpp
int frame_index;
std::string frame_name;
std::string image_path;
std::string output_image_path;
std::string sensor_name;
int sensor_id;

int image_width;
int image_height;

int proposal_count;
int active_track_count;

bool is_valid;

std::vector<ObjectProposal> frame_proposals;
```

---

# 10.13 C++ — `BaseProposalTracker`

**File:**

```text
cpp/include/BaseProposalTracker.hpp
```

Tạo class trừu tượng:

```cpp
class BaseProposalTracker
```

## Hàm cần có

```cpp
virtual void load_frame_manifest(const std::string& path) = 0;
virtual void load_roi_config(const std::string& path) = 0;
virtual void load_proposal_config(const std::string& path) = 0;
virtual void load_tracking_config(const std::string& path) = 0;
virtual void run_tracking() = 0;
virtual ~BaseProposalTracker() = default;
```

---

# 10.14 C++ — `MultiFrameProposalTracker`

**File:**

```text
cpp/include/MultiFrameProposalTracker.hpp
cpp/src/MultiFrameProposalTracker.cpp
```

Tạo class kế thừa:

```cpp
class MultiFrameProposalTracker : public BaseProposalTracker
```

## Thuộc tính cần có

```cpp
private:
    std::vector<FrameRecord> frame_records;
    std::vector<ROIConfig> roi_configs;
    ProposalConfig proposal_config;
    TrackingConfig tracking_config;

    std::vector<ProposalTrack> tracks;
    std::vector<FrameTrackingResult> frame_results;

    int next_track_id;
```

---

## Hàm cần có

### Load / Read config
- `read_frame_manifest(...)`
- `read_roi_config(...)`
- `read_proposal_config(...)`
- `read_tracking_config(...)`
- các hàm `load_...(...) override`

---

### Proposal building part (tái dùng tinh thần Bài 8)

### `cv::Rect clamp_roi_to_image(const ROIConfig& roi_cfg, const cv::Mat& image) const;`
### `cv::Mat extract_roi_patch(const cv::Mat& image, const cv::Rect& roi_rect) const;`
### `cv::Mat build_proposal_mask(const cv::Mat& roi_patch_bgr) const;`
### `std::vector<std::vector<cv::Point>> extract_candidate_contours(const cv::Mat& proposal_mask) const;`
### `bool is_valid_candidate_contour(const std::vector<cv::Point>& contour) const;`

### `ObjectProposal build_object_proposal(
    int frame_index,
    const std::string& frame_name,
    const std::string& roi_name,
    const cv::Rect& roi_rect,
    const std::vector<cv::Point>& contour
) const;`

---

## Tracking part

### `double compute_iou(const cv::Rect& a, const cv::Rect& b) const;`
**Hành vi**
- tính IoU giữa 2 bounding box

### `cv::Point2d get_box_center(const cv::Rect& box) const;`
- lấy tâm bounding box

### `double compute_center_distance(const cv::Rect& a, const cv::Rect& b) const;`
- khoảng cách Euclidean giữa 2 tâm box

---

### `int find_best_track_for_proposal(const ObjectProposal& proposal) const;`
## Hành vi gợi ý
Loop qua các track đang tồn tại, lấy proposal cuối cùng của mỗi track, rồi chọn track phù hợp nhất nếu:
- IoU >= `min_iou_threshold`
- center distance <= `max_center_distance`

Nếu không có track phù hợp → trả `-1`.

---

### `void update_tracks_with_frame_proposals(
    std::vector<ObjectProposal>& frame_proposals,
    int frame_index
);`

## Hành vi tổng quát
1. Với từng proposal của frame hiện tại:
   - tìm best track
   - nếu có → gán vào track đó
   - nếu không → tạo track mới
2. Với các track không được match ở frame hiện tại:
   - tăng `missing_frame_count`
3. Nếu `missing_frame_count > max_missing_frames`
   - có thể coi track là inactive
   - hoặc giữ lại nhưng không match nữa tùy thiết kế

---

### `std::vector<ObjectProposal> build_frame_proposals(
    const FrameRecord& frame_record,
    const cv::Mat& image
) const;`

## Hành vi
- loop qua các ROI
- build proposal trong từng ROI
- gom tất cả proposal của frame

---

### `void draw_tracking_overlay(
    cv::Mat& image,
    const std::vector<ObjectProposal>& frame_proposals
) const;`

## Hành vi
- vẽ proposal box
- ghi:
  - `Track #id`
  - tên ROI nếu muốn

---

### `FrameTrackingResult process_single_frame(const FrameRecord& frame_record);`
## Hành vi tổng quát
1. đọc frame image
2. nếu lỗi → tạo result invalid
3. build proposals cho frame
4. update tracks bằng proposals frame hiện tại
5. vẽ overlay proposal + track id
6. lưu annotated frame
7. build `FrameTrackingResult`

---

### `void run_tracking() override;`
- loop qua frame theo đúng thứ tự `frame_index`
- gọi `process_single_frame(...)`

### Getter

```cpp
const std::vector<FrameTrackingResult>& get_frame_results() const;
const std::vector<ProposalTrack>& get_tracks() const;
```

---

# 10.15 C++ — `TrackingReportWriter`

**File:**

```text
cpp/include/TrackingReportWriter.hpp
cpp/src/TrackingReportWriter.cpp
```

Tạo class:

```cpp
class TrackingReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<FrameTrackingResult>& frame_results,
    const std::vector<ProposalTrack>& tracks
);`

## Format gợi ý

```text
[Frame Tracking]
Frame Index: 1
Frame Name: frame_0001
Input Image: assets/raw_frames/frame_0001.jpg
Output Image: assets/outputs/tracked_frame_0001.jpg
Sensor: head_rgb_camera
Proposal Count: 2
Active Track Count: 2
Valid: true

  [Proposal]
  ROI: left_region
  Full Box: x=40, y=60, w=70, h=55
  Area: 2510
  Assigned Track ID: 0

----------------------------------------

[Track Summary]
Track ID: 0
Last Seen Frame: 4
Missing Frame Count: 0
History Length: 4
Frames: 1, 2, 3, 4
```

---

# 10.16 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 CameraSensor**
- in thông tin camera
- tạo `MultiFrameProposalTracker`
- load:
  - `config/frame_sequence_manifest.txt`
  - `config/roi_config.txt`
  - `config/proposal_config.txt`
  - `config/tracking_config.txt`
- chạy `run_tracking()`
- tạo `TrackingReportWriter`
- ghi report ra:
  - `assets/outputs/tracking_report.txt`

## Pipeline `main.cpp`

```text
Create CameraSensor
→ Load Frame Manifest
→ Load ROI Config
→ Load Proposal Config
→ Load Tracking Config
→ Run Multi-Frame Proposal Tracking
→ Save Annotated Tracking Frames
→ Write Tracking Report
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
- nhiều frame từ manifest
- proposal builder trong ROI
- proposal association giữa các frame
- track ID
- tracking report

---

# 12. Output mong muốn

## File config
```text
config/frame_sequence_manifest.txt
config/roi_config.txt
config/proposal_config.txt
config/tracking_config.txt
```

## Ảnh output
```text
assets/outputs/tracked_frame_0001.jpg
assets/outputs/tracked_frame_0002.jpg
assets/outputs/tracked_frame_0003.jpg
assets/outputs/tracked_frame_0004.jpg
```

## File report
```text
assets/outputs/tracking_report.txt
```

---

## Ví dụ `frame_sequence_manifest.txt`

```text
1|frame_0001|assets/raw_frames/frame_0001.jpg|head_rgb_camera|0
2|frame_0002|assets/raw_frames/frame_0002.jpg|head_rgb_camera|0
3|frame_0003|assets/raw_frames/frame_0003.jpg|head_rgb_camera|0
4|frame_0004|assets/raw_frames/frame_0004.jpg|head_rgb_camera|0
```

---

## Ví dụ `tracking_config.txt`

```text
association_mode=iou_center_distance
min_iou_threshold=0.15
max_center_distance=80
max_missing_frames=1
```

---

## Ví dụ `tracking_report.txt`

```text
[Frame Tracking]
Frame Index: 1
Frame Name: frame_0001
Input Image: assets/raw_frames/frame_0001.jpg
Output Image: assets/outputs/tracked_frame_0001.jpg
Sensor: head_rgb_camera
Proposal Count: 2
Active Track Count: 2
Valid: true

  [Proposal]
  ROI: left_region
  Full Box: x=40, y=60, w=70, h=55
  Area: 2510
  Assigned Track ID: 0

----------------------------------------

[Track Summary]
Track ID: 0
Last Seen Frame: 4
Missing Frame Count: 0
History Length: 4
Frames: 1, 2, 3, 4
```

---

# 13. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo **frame sequence manifest**
- tạo **ROI config**
- tạo **proposal config**
- tạo **tracking config**
- chuẩn bị metadata cho module C++

Tức là Python làm phần:

```text
Temporal Perception Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- build proposal cho từng frame
- association proposal qua các frame
- quản lý track id
- lưu annotated frames + tracking report

Tức là C++ làm phần:

```text
Multi-Frame Proposal Tracking Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **trích proposal từ từng frame**
- **so sánh proposal boxes qua thời gian**
- **biến proposal rời rạc thành temporal track**

Tức là CV làm phần:

```text
Frame-wise Object Proposals → Temporal Object Tracks
```

---

# 14. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `frame_sequence_manifest.txt`
- [ ] Python tạo được `roi_config.txt`
- [ ] Python tạo được `proposal_config.txt`
- [ ] Python tạo được `tracking_config.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / string / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `CameraSensor`
- [ ] C++ có `FrameRecord`
- [ ] C++ có `ROIConfig`
- [ ] C++ có `ProposalConfig`
- [ ] C++ có `TrackingConfig`
- [ ] C++ có `ObjectProposal`
- [ ] C++ có `ProposalTrack`
- [ ] C++ có `FrameTrackingResult`
- [ ] C++ có `BaseProposalTracker`
- [ ] C++ có `MultiFrameProposalTracker`
- [ ] C++ load được frame manifest
- [ ] C++ load được ROI config
- [ ] C++ load được proposal config
- [ ] C++ load được tracking config
- [ ] C++ build được proposals cho từng frame
- [ ] C++ tính được IoU / center distance
- [ ] C++ gán được track id
- [ ] C++ cập nhật được track history
- [ ] C++ vẽ được tracking overlay
- [ ] C++ lưu được annotated tracking frames
- [ ] C++ build được tracking report

---

# 15. Gợi ý mở rộng

## 1. Track score tốt hơn
Bạn có thể kết hợp:
- IoU
- center distance
- area similarity
- color similarity

## 2. Track smoothing
Lưu thêm:
- center history
- average velocity sơ bộ

## 3. Kết hợp feature matching của Bài 5
Nếu muốn “đúng chất perception” hơn, bạn có thể:
- trong proposal box, trích ORB feature
- match proposal giữa frame t và t+1

## 4. Chuẩn bị cho Bài 10
Sau Bài 9, bước rất hợp lý cho **Bài 10** là đi tiếp theo hướng:

```text
Robot Stereo Pair Proposal Correspondence Starter
```

để bắt đầu chuyển từ:
- **tracking theo thời gian**
sang
- **correspondence giữa camera trái / phải**

Đây sẽ là cây cầu rất đẹp để khép Đợt 2 và chuẩn bị sang các đợt stereo / depth sau đó.

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 9**, bạn sẽ đi tiếp trong **Đợt 2** theo một hướng rất đúng nhịp:

- **Bài 6**: quản lý nhiều ảnh + transform + metadata sensor
- **Bài 7**: phân tích scene theo ROI
- **Bài 8**: tạo object proposal trong ROI
- **Bài 9**: **liên kết proposal qua nhiều frame**

Tức là bạn đi từ:

```text
“frame này có candidate object boxes nào”
```

sang

```text
“candidate object box nào đang tồn tại liên tục qua chuỗi frame”
```

Đây là bước rất tốt trước khi **Bài 10** bắt đầu đưa bạn sang hướng:
- **proposal correspondence giữa 2 ảnh / 2 camera**
- chuẩn bị cho stereo vision / depth / temporal perception nâng cao hơn.
