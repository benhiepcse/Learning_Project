# 🤖 Bài 8: Robot ROI Object Proposal Builder — Xây vùng đề xuất vật thể từ ROI cho Humanoid Robot AI Perception

> Mini Project số 8 trong **Đợt 2 — Bài 6 → Bài 10**  
> **Bài 8 tiếp tục kết hợp kiến thức của Đợt 1 + Đợt 2** theo đúng rule bạn đã chốt.  
> Nếu **Bài 6** là **quản lý dataset ảnh + sensor metadata**, **Bài 7** là **phân tích scene theo ROI và thống kê màu**, thì **Bài 8** sẽ tiến thêm một nấc rất quan trọng trong AI Perception:  
> robot sẽ **tạo các object proposal trong từng ROI**, tức là từ một vùng ảnh lớn, hệ thống phải tìm ra **những candidate object region** có khả năng là vật thể đáng quan tâm.

---

# 📌 Mục lục

- [1. Mô tả](#1-mô-tả)
- [2. Bài 8 nằm ở đâu trong roadmap](#2-bài-8-nằm-ở-đâu-trong-roadmap)
- [3. Vì sao Bài 8 là bước tiếp theo hợp lý sau Bài 7](#3-vì-sao-bài-8-là-bước-tiếp-theo-hợp-lý-sau-bài-7)
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
  - [10.2 Python — ROIProposalConfigBuilder](#102-python--roiproposalconfigbuilder)
  - [10.3 Python — main_config_builder.py](#103-python--main_config_builderpy)
  - [10.4 C++ — BaseSensor](#104-c--basesensor)
  - [10.5 C++ — CameraSensor](#105-c--camerasensor)
  - [10.6 C++ — ImageRecord](#106-c--imagerecord)
  - [10.7 C++ — ROIConfig](#107-c--roiconfig)
  - [10.8 C++ — ProposalConfig](#108-c--proposalconfig)
  - [10.9 C++ — ObjectProposal](#109-c--objectproposal)
  - [10.10 C++ — ROIProposalResult](#1010-c--roiproposalresult)
  - [10.11 C++ — SceneProposalResult](#1011-c--sceneproposalresult)
  - [10.12 C++ — BaseProposalBuilder](#1012-c--baseproposalbuilder)
  - [10.13 C++ — ROIObjectProposalBuilder](#1013-c--roiobjectproposalbuilder)
  - [10.14 C++ — ProposalReportWriter](#1014-c--proposalreportwriter)
  - [10.15 C++ — main.cpp](#1015-c--maincpp)
- [11. Điều kiện bắt buộc](#11-điều-kiện-bắt-buộc)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò của bài này trong Humanoid Robot](#13-vai-trò-của-bài-này-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Mô tả

Ở **Bài 7**, bạn đã có một module có thể:

- đọc nhiều ảnh scene từ manifest
- định nghĩa nhiều ROI trên ảnh
- cắt ROI
- tính mean color / brightness
- đo `target_mask_ratio`
- đánh dấu ROI “interesting”
- vẽ overlay ROI và ghi report scene

Bài 8 sẽ **không dừng ở mức “ROI nào đáng chú ý”** nữa.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc nhiều scene images
- đọc ROI config
- chọn / duyệt từng ROI
- trong mỗi ROI:
  - tạo **mask màu mục tiêu** hoặc binary mask
  - tìm **contour candidate**
  - lọc contour theo diện tích / kích thước
  - tạo **object proposal bounding box**
- chuyển proposal từ tọa độ ROI sang tọa độ toàn ảnh
- vẽ proposal lên scene
- lưu report cho từng ROI và từng scene

Ví dụ robot nhìn một mặt bàn hoặc một vùng trước mặt:

- ROI trái có một vùng đỏ lớn → có thể là object proposal số 1
- ROI giữa có 2 contour rõ → sinh ra 2 object proposals
- ROI phải không có contour hợp lệ → không tạo proposal

Tức là bài này giúp robot đi từ:

```text
“ROI này có vẻ đáng chú ý”
```

sang

```text
“ROI này có 1 hoặc nhiều candidate object regions”
```
<p align="center">
  <img src="images/project_8.png" width="800">
</p>

---

# 2. Bài 8 nằm ở đâu trong roadmap

## Quy ước hiện tại
- **Đợt 1 = Bài 1 → Bài 5**
- **Đợt 2 = Bài 6 → Bài 10**
- **Đợt 3 = Bài 11 → Bài 15**
- **Đợt 4 = Bài 16 → Bài 20**

Vì vậy:

## **Bài 8 = Bài thứ ba của Đợt 2**
và vẫn **kết hợp kiến thức Đợt 1 + Đợt 2**.

---

# 3. Vì sao Bài 8 là bước tiếp theo hợp lý sau Bài 7

## Bài 6 cho bạn:
- quản lý nhiều ảnh
- transform ảnh đầu vào
- metadata sensor

## Bài 7 cho bạn:
- phân tích ROI
- thống kê màu / brightness
- chọn vùng interesting

## Bài 8 nâng thêm một nấc:
- không chỉ **đánh giá ROI**
- mà còn **tìm candidate object box bên trong ROI**

Đây là một bước rất hợp lý vì trong perception thật, sau khi có vùng đáng chú ý, robot thường phải làm tiếp:

```text
ROI → candidate object regions → detector / classifier / tracker
```

---

# 4. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Image Dataset + ROI Config + Proposal Config
→ Load Scene Images
→ Extract ROI Patches
→ Build ROI Masks
→ Find Candidate Contours
→ Filter Candidate Regions
→ Build Object Proposals
→ Map ROI Coordinates back to Full Image
→ Save Annotated Scene + Proposal Report
```

Bài này giúp bạn hiểu một module rất quan trọng trong AI Perception:

> **Object proposal** là bước “giữa” giữa scene-level analysis và object detection/classification.

---

# 5. Pipeline perception của bài

```text
Sensor Dataset Config
→ Read Image Records
→ Read ROI Config
→ Read Proposal Config
→ Create Camera Sensor Object
→ Load Each Scene
→ Extract ROI Patch
→ Convert ROI to Analysis Space
→ Threshold / Build Mask
→ Find Contours
→ Filter Candidate Regions
→ Build Proposal Boxes
→ Convert Proposal Coordinates to Full Scene Coordinates
→ Draw Proposals on Scene
→ Save Annotated Scene
→ Write Proposal Report
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
- morphology cơ bản *(nếu muốn clean mask)*

---

# 6.4 CV Python

Python không phải runtime proposal builder chính, nhưng sẽ dùng để:
- build manifest dataset
- build ROI config
- build proposal config
- chuẩn bị metadata đầu vào

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
- class detector/analyzer style

## CV
- color threshold
- contour extraction
- bounding box
- annotated output

> Đây chính là phần nối lại với:
> - **Bài 2** (Color Object Detector)
> - **Bài 3** (Edge & Contour Inspector)

---

# 7.2 Phần lấy từ Đợt 2

## Python
- list / dict mạnh hơn
- string split / join / replace
- type casting khi parse config text

## C++
- vector để lưu nhiều scene, nhiều ROI, nhiều proposals
- `const` / `auto`
- struct metadata / config

## CV
- ROI
- scene-level processing
- nhiều vùng trên một ảnh
- scene overlay / report

---

# 8. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 8, bạn phải nắm được 5 ý rất quan trọng:

## 1. Object proposal không phải detector hoàn chỉnh
Nó chỉ tạo ra **candidate vùng vật thể**, chưa chắc đã biết vật đó là gì.

## 2. ROI giúp giảm phạm vi tìm kiếm
Thay vì scan cả ảnh, robot có thể:
- chọn ROI
- build proposal trong ROI
- tiết kiệm xử lý hơn

## 3. Contour + threshold vẫn rất hữu ích trong pipeline cổ điển
Không phải lúc nào cũng cần deep learning mới có proposal.

## 4. Proposal cần được map về tọa độ full image
Vì detector / tracker / robot logic sau này thường cần tọa độ theo **scene gốc**.

## 5. Đây là bước đệm rất tốt trước các bài stereo / depth
Khi bạn sang các đợt sau, việc “tìm candidate region” sẽ cực hữu ích cho:
- matching
- disparity focus region
- object localization

---

# 9. Cấu trúc folder

```text
mini_project_08_robot_roi_object_proposal_builder/
│
├─ README.md
│
├─ assets/
│  ├─ raw_images/
│  │  ├─ scene_01.jpg
│  │  ├─ scene_02.jpg
│  │  ├─ scene_03.jpg
│  │  └─ scene_04.jpg
│  │
│  └─ outputs/
│     ├─ proposal_scene_01.jpg
│     ├─ proposal_scene_02.jpg
│     ├─ proposal_scene_03.jpg
│     ├─ proposal_scene_04.jpg
│     └─ proposal_report.txt
│
├─ config/
│  ├─ sensor_dataset_manifest.txt
│  ├─ roi_config.txt
│  └─ proposal_config.txt
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
   │  ├─ ImageRecord.hpp
   │  ├─ ROIConfig.hpp
   │  ├─ ProposalConfig.hpp
   │  ├─ ObjectProposal.hpp
   │  ├─ ROIProposalResult.hpp
   │  ├─ SceneProposalResult.hpp
   │  ├─ BaseProposalBuilder.hpp
   │  ├─ ROIObjectProposalBuilder.hpp
   │  └─ ProposalReportWriter.hpp
   │
   └─ src/
      ├─ CameraSensor.cpp
      ├─ ROIObjectProposalBuilder.cpp
      └─ ProposalReportWriter.cpp
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
dataset_manifest_path
roi_config_path
proposal_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn các file config

---

# 10.2 Python — `ROIProposalConfigBuilder`

**File:**

```text
python/tools/config_builder.py
```

Tạo class con:

```python
class ROIProposalConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
image_records
roi_regions
proposal_config
```

---

## `image_records`
Là **list các dict**, ví dụ:

```python
[
    {
        "frame_name": "scene_01",
        "image_path": "assets/raw_images/scene_01.jpg",
        "sensor_name": "head_rgb_camera",
        "sensor_id": 0
    },
    {
        "frame_name": "scene_02",
        "image_path": "assets/raw_images/scene_02.jpg",
        "sensor_name": "head_rgb_camera",
        "sensor_id": 0
    }
]
```

## `roi_regions`
Là **list các dict**, mỗi dict mô tả một ROI template, ví dụ:

```python
[
    {
        "roi_name": "left_region",
        "x": 0,
        "y": 0,
        "width": 200,
        "height": 240
    },
    {
        "roi_name": "center_region",
        "x": 200,
        "y": 0,
        "width": 200,
        "height": 240
    }
]
```

## `proposal_config`
Là `dict`, ví dụ:

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

---

## Hàm cần có

### `add_image_record(frame_name, image_path, sensor_name, sensor_id)`
**Hành vi**
- thêm một record ảnh
- kiểm tra:
  - chuỗi không rỗng
  - `sensor_id >= 0`

### `add_roi_region(roi_name, x, y, width, height)`
**Hành vi**
- thêm một ROI template
- kiểm tra:
  - `width > 0`, `height > 0`
  - `x >= 0`, `y >= 0`

### `set_proposal_config(
    analysis_color_space,
    proposal_mode,
    target_color_label,
    lh, ls, lv,
    uh, us, uv,
    min_contour_area,
    min_box_width,
    min_box_height
)`
**Hành vi**
- lưu config proposal
- kiểm tra:
  - `analysis_color_space` thuộc `{"BGR", "GRAY", "HSV"}`
  - `proposal_mode` thuộc `{"target_color_mask", "gray_threshold"}` *(bạn có thể chỉ bắt buộc 1 mode nếu muốn)*
  - ngưỡng hợp lệ
  - `min_contour_area > 0`
  - `min_box_width > 0`
  - `min_box_height > 0`

### `write_dataset_manifest()`
**Format gợi ý**
```text
scene_01|assets/raw_images/scene_01.jpg|head_rgb_camera|0
scene_02|assets/raw_images/scene_02.jpg|head_rgb_camera|0
scene_03|assets/raw_images/scene_03.jpg|head_rgb_camera|0
```

### `write_roi_config()`
**Format gợi ý**
```text
left_region|0|0|200|240
center_region|200|0|200|240
right_region|400|0|200|240
```

### `write_proposal_config()`
**Format gợi ý**
```text
analysis_color_space=HSV
proposal_mode=target_color_mask
target_color_label=red
lower_h=0
lower_s=120
lower_v=70
upper_h=10
upper_s=255
upper_v=255
min_contour_area=400
min_box_width=20
min_box_height=20
```

---

# 10.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **4 image records**
- tạo ít nhất **3 ROI template**
- set proposal config
- ghi đủ:
  - `config/sensor_dataset_manifest.txt`
  - `config/roi_config.txt`
  - `config/proposal_config.txt`

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

# 10.6 C++ — `ImageRecord`

**File:**

```text
cpp/include/ImageRecord.hpp
```

Tạo struct:

```cpp
struct ImageRecord
```

## Thuộc tính cần có

```cpp
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

Tạo struct:

```cpp
struct ROIConfig
```

## Thuộc tính cần có

```cpp
std::string roi_name;
int x;
int y;
int width;
int height;
```

---

# 10.8 C++ — `ProposalConfig`

**File:**

```text
cpp/include/ProposalConfig.hpp
```

Tạo struct:

```cpp
struct ProposalConfig
```

## Thuộc tính cần có

```cpp
std::string analysis_color_space;
std::string proposal_mode;
std::string target_color_label;

int lower_h;
int lower_s;
int lower_v;

int upper_h;
int upper_s;
int upper_v;

double min_contour_area;
int min_box_width;
int min_box_height;
```

---

# 10.9 C++ — `ObjectProposal`

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
std::string frame_name;
std::string roi_name;

cv::Rect roi_box;
cv::Rect proposal_box_roi;
cv::Rect proposal_box_full_image;

double contour_area;
bool is_valid;
```

> `proposal_box_roi` = bounding box theo hệ tọa độ ROI  
> `proposal_box_full_image` = bounding box theo hệ tọa độ ảnh gốc

---

# 10.10 C++ — `ROIProposalResult`

**File:**

```text
cpp/include/ROIProposalResult.hpp
```

Tạo struct:

```cpp
struct ROIProposalResult
```

## Thuộc tính cần có

```cpp
std::string frame_name;
std::string roi_name;
cv::Rect roi_rect;
int proposal_count;
bool is_interesting;
std::vector<ObjectProposal> proposals;
```

> `is_interesting` có thể gán bằng rule:
> - `proposal_count > 0`

---

# 10.11 C++ — `SceneProposalResult`

**File:**

```text
cpp/include/SceneProposalResult.hpp
```

Tạo struct:

```cpp
struct SceneProposalResult
```

## Thuộc tính cần có

```cpp
std::string frame_name;
std::string image_path;
std::string output_image_path;
std::string sensor_name;
int sensor_id;

int image_width;
int image_height;

int roi_count;
int total_proposal_count;

bool is_valid;

std::vector<ROIProposalResult> roi_results;
```

---

# 10.12 C++ — `BaseProposalBuilder`

**File:**

```text
cpp/include/BaseProposalBuilder.hpp
```

Tạo class trừu tượng:

```cpp
class BaseProposalBuilder
```

## Hàm cần có

```cpp
virtual void load_dataset_manifest(const std::string& path) = 0;
virtual void load_roi_config(const std::string& path) = 0;
virtual void load_proposal_config(const std::string& path) = 0;
virtual void build_proposals() = 0;
virtual ~BaseProposalBuilder() = default;
```

---

# 10.13 C++ — `ROIObjectProposalBuilder`

**File:**

```text
cpp/include/ROIObjectProposalBuilder.hpp
cpp/src/ROIObjectProposalBuilder.cpp
```

Tạo class kế thừa:

```cpp
class ROIObjectProposalBuilder : public BaseProposalBuilder
```

## Thuộc tính cần có

```cpp
private:
    std::vector<ImageRecord> image_records;
    std::vector<ROIConfig> roi_configs;
    ProposalConfig proposal_config;
    std::vector<SceneProposalResult> scene_results;
```

---

## Hàm cần có

### `std::vector<ImageRecord> read_dataset_manifest(const std::string& path);`
- đọc `sensor_dataset_manifest.txt`

### `std::vector<ROIConfig> read_roi_config(const std::string& path);`
- đọc `roi_config.txt`

### `ProposalConfig read_proposal_config(const std::string& path);`
- đọc `proposal_config.txt`

### `void load_dataset_manifest(const std::string& path) override;`
### `void load_roi_config(const std::string& path) override;`
### `void load_proposal_config(const std::string& path) override;`

---

### `cv::Rect clamp_roi_to_image(const ROIConfig& roi_cfg, const cv::Mat& image) const;`
**Hành vi**
- đảm bảo ROI không vượt biên ảnh

---

### `cv::Mat extract_roi_patch(const cv::Mat& image, const cv::Rect& roi_rect) const;`
- crop ROI patch

### `cv::Mat build_proposal_mask(const cv::Mat& roi_patch_bgr) const;`
## Hành vi
Tùy theo `proposal_mode`:

### Nếu `target_color_mask`
1. convert ROI sang HSV
2. `cv::inRange(...)`
3. trả mask

### Nếu `gray_threshold`
1. convert ROI sang grayscale
2. threshold ảnh để lấy vùng foreground
3. trả mask

> Bạn có thể thêm bước morphology nếu muốn clean mask.

---

### `std::vector<std::vector<cv::Point>> extract_candidate_contours(
    const cv::Mat& proposal_mask
) const;`
- dùng `cv::findContours()`

---

### `bool is_valid_candidate_contour(
    const std::vector<cv::Point>& contour
) const;`

## Hành vi
- contour area >= `min_contour_area`
- bounding box width >= `min_box_width`
- bounding box height >= `min_box_height`

---

### `ObjectProposal build_object_proposal(
    const std::string& frame_name,
    const std::string& roi_name,
    const cv::Rect& roi_rect,
    const std::vector<cv::Point>& contour
) const;`

## Hành vi
1. tạo `proposal_box_roi` từ `boundingRect(contour)`
2. map sang `proposal_box_full_image`
3. lưu `contour_area`
4. gán `is_valid`

---

### `ROIProposalResult analyze_single_roi(
    const std::string& frame_name,
    const ROIConfig& roi_cfg,
    const cv::Mat& full_image
) const;`

## Hành vi tổng quát
1. clamp ROI
2. crop ROI
3. build proposal mask
4. find contours
5. lọc contour hợp lệ
6. build `ObjectProposal`
7. gom vào `ROIProposalResult`

---

### `void draw_roi_and_proposals(
    cv::Mat& image,
    const ROIProposalResult& roi_result
) const;`

## Hành vi
- vẽ ROI rectangle
- vẽ proposal boxes trên ảnh full scene
- ghi text:
  - tên ROI
  - số proposals

---

### `SceneProposalResult analyze_single_scene(const ImageRecord& record);`
## Hành vi tổng quát
1. đọc ảnh scene
2. nếu lỗi → tạo result invalid
3. loop qua toàn bộ ROI
4. gọi `analyze_single_roi(...)`
5. cộng tổng proposal
6. vẽ overlay ROI + proposal lên scene
7. lưu annotated image
8. build `SceneProposalResult`

---

### `void build_proposals() override;`
- loop qua toàn bộ `image_records`
- gọi `analyze_single_scene(...)`

### Getter

```cpp
const std::vector<SceneProposalResult>& get_scene_results() const;
```

---

# 10.14 C++ — `ProposalReportWriter`

**File:**

```text
cpp/include/ProposalReportWriter.hpp
cpp/src/ProposalReportWriter.cpp
```

Tạo class:

```cpp
class ProposalReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<SceneProposalResult>& scene_results
);`

## Format gợi ý

```text
[Scene Proposal]
Frame: scene_01
Input Image: assets/raw_images/scene_01.jpg
Output Image: assets/outputs/proposal_scene_01.jpg
Sensor: head_rgb_camera
Sensor ID: 0
Image Size: 640x480
ROI Count: 3
Total Proposal Count: 4
Valid: true

  [ROI]
  Name: left_region
  Rect: x=0, y=0, w=200, h=240
  Proposal Count: 1
  Interesting: true

    [Proposal]
    ROI Box: x=30, y=40, w=70, h=60
    Full Image Box: x=30, y=40, w=70, h=60
    Area: 2650
    Valid: true

  [ROI]
  Name: center_region
  Rect: x=200, y=0, w=200, h=240
  Proposal Count: 2
  Interesting: true

----------------------------------------
```

---

# 10.15 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 CameraSensor**
- in thông tin camera
- tạo `ROIObjectProposalBuilder`
- load:
  - `config/sensor_dataset_manifest.txt`
  - `config/roi_config.txt`
  - `config/proposal_config.txt`
- chạy `build_proposals()`
- tạo `ProposalReportWriter`
- ghi report ra:
  - `assets/outputs/proposal_report.txt`

## Pipeline `main.cpp`

```text
Create CameraSensor
→ Load Dataset Manifest
→ Load ROI Config
→ Load Proposal Config
→ Build ROI Object Proposals
→ Save Annotated Proposal Scenes
→ Write Proposal Report
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
- nhiều ảnh từ manifest
- nhiều ROI trên mỗi ảnh
- threshold / contour / proposal boxes
- proposal map từ ROI sang full image
- scene-level proposal report

---

# 12. Output mong muốn

## File config
```text
config/sensor_dataset_manifest.txt
config/roi_config.txt
config/proposal_config.txt
```

## Ảnh output
```text
assets/outputs/proposal_scene_01.jpg
assets/outputs/proposal_scene_02.jpg
assets/outputs/proposal_scene_03.jpg
assets/outputs/proposal_scene_04.jpg
```

## File report
```text
assets/outputs/proposal_report.txt
```

---

## Ví dụ `sensor_dataset_manifest.txt`

```text
scene_01|assets/raw_images/scene_01.jpg|head_rgb_camera|0
scene_02|assets/raw_images/scene_02.jpg|head_rgb_camera|0
scene_03|assets/raw_images/scene_03.jpg|head_rgb_camera|0
scene_04|assets/raw_images/scene_04.jpg|head_rgb_camera|0
```

---

## Ví dụ `roi_config.txt`

```text
left_region|0|0|200|240
center_region|200|0|200|240
right_region|400|0|200|240
```

---

## Ví dụ `proposal_config.txt`

```text
analysis_color_space=HSV
proposal_mode=target_color_mask
target_color_label=red
lower_h=0
lower_s=120
lower_v=70
upper_h=10
upper_s=255
upper_v=255
min_contour_area=400
min_box_width=20
min_box_height=20
```

---

## Ví dụ `proposal_report.txt`

```text
[Scene Proposal]
Frame: scene_01
Input Image: assets/raw_images/scene_01.jpg
Output Image: assets/outputs/proposal_scene_01.jpg
Sensor: head_rgb_camera
Sensor ID: 0
Image Size: 640x480
ROI Count: 3
Total Proposal Count: 4
Valid: true

  [ROI]
  Name: left_region
  Rect: x=0, y=0, w=200, h=240
  Proposal Count: 1
  Interesting: true

    [Proposal]
    ROI Box: x=30, y=40, w=70, h=60
    Full Image Box: x=30, y=40, w=70, h=60
    Area: 2650
    Valid: true

----------------------------------------
```

---

# 13. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo **manifest scene dataset**
- tạo **ROI config**
- tạo **proposal config**
- chuẩn bị metadata cho module C++

Tức là Python làm phần:

```text
Scene Dataset / ROI Proposal Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- đọc scene images
- loop qua nhiều ROI
- build mask / find contour / tạo proposal
- map proposal từ ROI sang full image
- vẽ overlay proposal lên scene
- ghi report

Tức là C++ làm phần:

```text
ROI-Level Candidate Object Proposal Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **threshold / mask hóa ROI**
- **tìm contour candidate**
- **tạo bounding box vật thể tiềm năng**
- **biến scene thành tập candidate object regions**

Tức là CV làm phần:

```text
Scene ROI → Candidate Object Regions
```

---

# 14. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `sensor_dataset_manifest.txt`
- [ ] Python tạo được `roi_config.txt`
- [ ] Python tạo được `proposal_config.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / string / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `CameraSensor`
- [ ] C++ có `ImageRecord`
- [ ] C++ có `ROIConfig`
- [ ] C++ có `ProposalConfig`
- [ ] C++ có `ObjectProposal`
- [ ] C++ có `ROIProposalResult`
- [ ] C++ có `SceneProposalResult`
- [ ] C++ có `BaseProposalBuilder`
- [ ] C++ có `ROIObjectProposalBuilder`
- [ ] C++ load được dataset manifest
- [ ] C++ load được ROI config
- [ ] C++ load được proposal config
- [ ] C++ clamp / crop ROI được
- [ ] C++ build được proposal mask
- [ ] C++ find được contour candidate
- [ ] C++ lọc được contour candidate hợp lệ
- [ ] C++ build được object proposals
- [ ] C++ map được proposal box từ ROI sang full image
- [ ] C++ vẽ được proposal overlay
- [ ] C++ lưu được annotated proposal scene
- [ ] C++ build được proposal report

---

# 15. Gợi ý mở rộng

## 1. Proposal nhiều mode
Ngoài `target_color_mask`, bạn có thể thêm:
- `gray_threshold`
- `edge_contour`
- `saliency_like`

## 2. Kết hợp Bài 4 / Bài 2 trong proposal
Ví dụ:
- proposal xong thì chạy **shape detector** hoặc **color detector** trên từng proposal box

## 3. Thêm score cho proposal
Ví dụ score dựa trên:
- contour area
- mask fill ratio
- aspect ratio hợp lý

## 4. Chuẩn bị cho Bài 9
Sau Bài 8, bước rất hợp lý cho **Bài 9** là đi tiếp theo hướng:

```text
Robot Multi-Frame Proposal Tracker Starter
```

hoặc

```text
Robot Stereo Pair ROI Proposal Organizer
```

để bắt đầu kết hợp:
- proposal giữa nhiều frame
- proposal correspondence
- và chuẩn bị tư duy cho stereo / depth / tracking.

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 8**, bạn sẽ đi tiếp trong **Đợt 2** theo một hướng rất đúng nhịp:

- **Bài 6**: quản lý nhiều ảnh + transform + metadata sensor
- **Bài 7**: phân tích scene theo ROI
- **Bài 8**: tạo **candidate object proposals** trong ROI

Tức là bạn đi từ:

```text
“scene nào / ROI nào đáng chú ý”
```

sang

```text
“trong ROI này có những candidate object boxes nào”
```

Đây là bước rất tốt trước khi các bài tiếp theo trong Đợt 2 chạm sâu hơn vào:
- proposal tracking nhiều frame
- proposal matching
- stereo-ready region management
- và perception logic gần với robot hơn.
