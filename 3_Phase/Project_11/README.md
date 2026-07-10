# 🤖 Bài 11: Robot Stereo Disparity Region Analyzer — Phân tích disparity theo ROI cho Humanoid Robot AI Perception

> Mini Project số 11 trong **Đợt 3 — Bài 11 → Bài 15**  
> **Bài 11 là bài mở đầu của Đợt 3** và được thiết kế theo đúng rule bạn đã chốt:
>
> - **Bài 1 → 5** = kiến thức **Đợt 1**
> - **Bài 6 → 10** = kiến thức **Đợt 2**
> - **Bài 11 → 15** = kiến thức **Đợt 3**
>
> Vì vậy **Bài 11 phải kết hợp lại kiến thức của Đợt 1 + Đợt 2 + Đợt 3**.  
> Nếu **Bài 10** đã đưa bạn tới **stereo proposal correspondence giữa camera trái/phải**, thì **Bài 11** sẽ đẩy tiếp sang bước rất quan trọng trong stereo perception:
>
> **từ cặp proposal trái-phải → tạo disparity map / disparity ROI analysis / disparity statistics cho robot.**

---

# 📌 Mục lục

- [1. Đợt 3 có gì mới và Bài 11 sẽ lấy gì từ đó](#1-đợt-3-có-gì-mới-và-bài-11-sẽ-lấy-gì-từ-đó)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 11 nằm ở đâu trong roadmap](#3-bài-11-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 11 là bước tiếp theo hợp lý sau Bài 10](#4-vì-sao-bài-11-là-bước-tiếp-theo-hợp-lý-sau-bài-10)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
  - [7.1 C++](#71-c)
  - [7.2 Python](#72-python)
  - [7.3 CV C++](#73-cv-c)
  - [7.4 CV Python](#74-cv-python)
- [8. Kiến thức Đợt 1 + Đợt 2 + Đợt 3 được dùng như thế nào](#8-kiến-thức-đợt-1--đợt-2--đợt-3-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
  - [11.1 Python — BaseConfigBuilder](#111-python--baseconfigbuilder)
  - [11.2 Python — StereoDisparityConfigBuilder](#112-python--stereodisparityconfigbuilder)
  - [11.3 Python — main_config_builder.py](#113-python--main_config_builderpy)
  - [11.4 C++ — BaseSensor](#114-c--basesensor)
  - [11.5 C++ — StereoCameraSensor](#115-c--stereocamerasensor)
  - [11.6 C++ — StereoFrameRecord](#116-c--stereoframerecord)
  - [11.7 C++ — ROIConfig](#117-c--roiconfig)
  - [11.8 C++ — StereoBMConfig](#118-c--stereobmconfig)
  - [11.9 C++ — DisparityROIStats](#119-c--disparityroistats)
  - [11.10 C++ — StereoDisparitySceneResult](#1110-c--stereodisparitysceneresult)
  - [11.11 C++ — BaseDisparityAnalyzer](#1111-c--basedisparityanalyzer)
  - [11.12 C++ — StereoDisparityAnalyzer](#1112-c--stereodisparityanalyzer)
  - [11.13 C++ — DisparityReportWriter](#1113-c--disparityreportwriter)
  - [11.14 C++ — main.cpp](#1114-c--maincpp)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Đợt 3 có gì mới và Bài 11 sẽ lấy gì từ đó

Theo roadmap 4 tuần của bạn, **Đợt 3 = Đợt 11 → 14 trong lịch 28 ngày**, còn trong chuỗi mini-project thì **Đợt 3 = Bài 11 → 15**.

## Kiến thức mới ở đầu Đợt 3 mà Bài 11 sẽ lấy
Từ roadmap, phần mở đầu Đợt 3 của bạn tập trung vào:

### Python
- **Phase 9 — Python Libraries**
  - `Matplotlib`

### Computer Vision
- **Phase 5 — Stereo Vision & Depth**
  - `Stereo Camera`
  - `Disparity`
  - sau đó là `Block Matching`, `SGM`

### C++
- không mở phase cú pháp mới nặng
- chủ yếu **dùng lại OOP + vector + file structure + enum / config / module runtime**

## Vì vậy Bài 11 sẽ lấy đúng phần sau:
- dùng **Python** để build config + viết report template + chuẩn bị manifest
- dùng **C++** để làm runtime stereo disparity analyzer
- dùng **OpenCV StereoBM hoặc StereoSGBM ở mức starter** để tính disparity map
- dùng **ROI** để đo disparity theo vùng
- dùng **Matplotlib (Python)** ở mức phụ trợ nếu bạn muốn vẽ histogram disparity sau khi project chạy xong

> Nghĩa là Bài 11 chưa nhảy thẳng sang **depth map / point cloud / triangulation**.  
> Nó là bước **trung gian cực hợp lý**:
>
> ```text
> stereo left-right proposal matching
> → stereo disparity computation
> → disparity analysis theo ROI / object region
> ```

---

# 2. Mô tả

Ở **Bài 10**, bạn đã có một module có thể:

- đọc **stereo pair**
- build proposal ở ảnh trái / phải
- match proposal trái-phải
- tính **proposal-level disparity** sơ bộ
- lưu stereo report

Bài 11 sẽ **đi sâu hơn vào disparity map**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc **cặp ảnh stereo trái / phải**
- tính **disparity map**
- chọn **nhiều ROI** trên ảnh trái
- cắt ROI tương ứng trên disparity map
- tính các thống kê như:
  - mean disparity
  - min disparity
  - max disparity
  - valid pixel ratio
- vẽ ROI lên ảnh trái
- lưu ảnh disparity visualization
- ghi report cho từng stereo pair

Ví dụ robot nhìn một chiếc hộp trên bàn:

- ROI trung tâm chứa chiếc hộp → disparity trung bình lớn hơn
- ROI nền phía sau → disparity trung bình nhỏ hơn

Từ đó robot bắt đầu hiểu được:

```text
vùng nào đang gần camera hơn
vùng nào đang xa hơn
```

---

# 3. Bài 11 nằm ở đâu trong roadmap

## Quy ước hiện tại
- **Đợt 1 = Bài 1 → Bài 5**
- **Đợt 2 = Bài 6 → Bài 10**
- **Đợt 3 = Bài 11 → Bài 15**
- **Đợt 4 = Bài 16 → Bài 20**

Vì vậy:

## **Bài 11 = bài mở đầu của Đợt 3**
và phải **kết hợp lại kiến thức của Đợt 1 + Đợt 2 + Đợt 3**.

---

# 4. Vì sao Bài 11 là bước tiếp theo hợp lý sau Bài 10

## Bài 8 cho bạn:
- object proposal trong ROI

## Bài 9 cho bạn:
- proposal tracking qua nhiều frame

## Bài 10 cho bạn:
- proposal correspondence giữa ảnh trái / phải
- proposal-level disparity sơ bộ

## Bài 11 nâng thêm một nấc:
- không chỉ nhìn **proposal-level disparity**
- mà bắt đầu tính **disparity map thật sự trên ảnh stereo**
- rồi **đo disparity theo ROI**

Đây là bước rất hợp lý vì trong stereo perception thật, luồng suy nghĩ thường là:

```text
left-right correspondence
→ disparity
→ depth reasoning
```

Bài 10 mới chạm vào correspondence, còn Bài 11 mới bắt đầu chạm vào **disparity field**.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Pair Dataset + ROI Config + StereoBM/SGBM Config
→ Load Left / Right Image Pair
→ Convert to Grayscale
→ Compute Disparity Map
→ Normalize / Visualize Disparity
→ Extract ROI Regions on Disparity Map
→ Compute ROI Disparity Statistics
→ Save Left Overlay + Disparity Visualization + Report
```

Bài này giúp bạn hiểu một module cực quan trọng trong stereo AI perception:

> **Disparity map** là bước trung gian giữa **stereo image pair** và **depth estimation / 3D perception**.

---

# 6. Pipeline perception của bài

```text
Stereo Pair Config
→ Read Stereo Pair Records
→ Read ROI Config
→ Read StereoBM Config
→ Create Stereo Camera Sensor Object
→ For Each Stereo Pair:
    → Load Left / Right Image
    → Convert to Grayscale
    → Compute Disparity Map
    → Normalize Disparity for Visualization
    → For Each ROI:
        → Clamp ROI
        → Extract Disparity ROI
        → Compute Disparity Statistics
    → Draw ROI Overlay on Left Image
    → Save Annotated Left Image
    → Save Disparity Visualization
→ Write Disparity Report
```

---

# 7. Kiến thức cần

# 7.1 C++

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

# 7.2 Python

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

# 7.3 CV C++

Ngoài những thứ của Đợt 1–2, Bài 11 bắt đầu chạm vào:

- `cv::StereoBM` **hoặc** `cv::StereoSGBM` *(khuyên dùng starter với StereoBM trước)*
- `cv::cvtColor`
- `cv::normalize`
- `cv::Rect`
- thao tác với disparity map kiểu `CV_16S` / `CV_32F`
- thống kê trên ROI của disparity map

---

# 7.4 CV Python

Python không phải runtime disparity chính, nhưng sẽ dùng để:
- build stereo pair manifest
- build ROI config
- build disparity config
- có thể hỗ trợ **vẽ histogram disparity** bằng `matplotlib`

---

# 8. Kiến thức Đợt 1 + Đợt 2 + Đợt 3 được dùng như thế nào

# 8.1 Phần lấy từ Đợt 1

## Python
- class / inheritance
- function
- loop / if else
- config builder style

## C++
- class sensor
- struct config / result
- file chia header / source

## CV
- đọc ảnh
- grayscale
- ROI
- save image

---

# 8.2 Phần lấy từ Đợt 2

## Python
- list / dict / string parsing
- ghi config cho nhiều stereo pairs

## C++
- vector để lưu nhiều scene result
- config parsing
- runtime nhiều ảnh

## CV
- stereo pair handling
- ROI scene analysis
- left-right workflow

---

# 8.3 Phần mới của Đợt 3

## Python
- có thể dùng `matplotlib` để hỗ trợ vẽ histogram disparity

## C++
- chưa mở phase cú pháp mới lớn, nhưng phải dùng lại OOP chặt hơn cho stereo module

## CV
- **StereoBM / StereoSGBM**
- **disparity map**
- **disparity ROI statistics**

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 11, bạn phải nắm được 7 ý rất quan trọng:

## 1. Disparity map là “ảnh trung gian” cực quan trọng của stereo
Nó không phải depth map, nhưng nó là nền để đi tới depth.

## 2. Disparity lớn thường tương ứng vật gần hơn
Nếu stereo pair đã rectified hợp lý, disparity thường phản ánh độ gần / xa tương đối.

## 3. Không phải toàn ảnh đều đáng quan tâm như nhau
Robot thường chỉ cần disparity ở:
- ROI chứa object
- vùng trước mặt robot
- vùng mặt bàn / vùng bước đi

## 4. Proposal-level disparity và dense disparity là hai mức khác nhau
- Bài 10: disparity ở **mức proposal box**
- Bài 11: disparity ở **mức pixel / ROI disparity map**

## 5. StereoBM / SGBM là công cụ thực chiến rất quan trọng
Dù sau này có deep learning stereo, bạn vẫn nên hiểu stereo cổ điển.

## 6. ROI disparity statistics rất hữu ích cho robot
Ví dụ:
- mean disparity vùng object
- valid disparity ratio
- so sánh foreground / background

## 7. Đây là cầu nối trực tiếp sang depth estimation
Sau Bài 11, việc sang Bài 12 làm **depth estimation** sẽ tự nhiên hơn rất nhiều.

---

# 10. Cấu trúc folder

```text
mini_project_11_robot_stereo_disparity_region_analyzer/
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
│     ├─ pair_01_left_roi_overlay.jpg
│     ├─ pair_01_disparity_visualization.jpg
│     ├─ pair_02_left_roi_overlay.jpg
│     ├─ pair_02_disparity_visualization.jpg
│     └─ disparity_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ roi_config.txt
│  └─ stereo_bm_config.txt
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
   │  ├─ StereoBMConfig.hpp
   │  ├─ DisparityROIStats.hpp
   │  ├─ StereoDisparitySceneResult.hpp
   │  ├─ BaseDisparityAnalyzer.hpp
   │  ├─ StereoDisparityAnalyzer.hpp
   │  └─ DisparityReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoDisparityAnalyzer.cpp
      └─ DisparityReportWriter.cpp
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
stereo_manifest_path
roi_config_path
stereo_bm_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn các file config

---

# 11.2 Python — `StereoDisparityConfigBuilder`

**File:**

```text
python/tools/config_builder.py
```

Tạo class con:

```python
class StereoDisparityConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
stereo_pairs
roi_regions
stereo_bm_config
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
    }
]
```

## `roi_regions`
Là list các dict:

```python
[
    {
        "roi_name": "center_region",
        "x": 160,
        "y": 80,
        "width": 220,
        "height": 180
    }
]
```

## `stereo_bm_config`
Là dict, ví dụ:

```python
{
    "matcher_type": "StereoBM",
    "num_disparities": 64,
    "block_size": 15,
    "min_disparity": 0,
    "texture_threshold": 10,
    "uniqueness_ratio": 12
}
```

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
**Hành vi**
- thêm stereo pair
- kiểm tra:
  - chuỗi không rỗng
  - `sensor_id >= 0`

### `add_roi_region(roi_name, x, y, width, height)`
**Hành vi**
- thêm ROI
- kiểm tra:
  - `width > 0`, `height > 0`
  - `x >= 0`, `y >= 0`

### `set_stereo_bm_config(
    matcher_type,
    num_disparities,
    block_size,
    min_disparity,
    texture_threshold,
    uniqueness_ratio
)`
**Hành vi**
- lưu config disparity
- kiểm tra:
  - `matcher_type` thuộc `{"StereoBM", "StereoSGBM"}`
  - `num_disparities > 0`
  - `num_disparities % 16 == 0`
  - `block_size` là số lẻ và > 0

### `write_stereo_manifest()`
**Format gợi ý**
```text
pair_01|assets/stereo_pairs/pair_01_left.jpg|assets/stereo_pairs/pair_01_right.jpg|head_stereo_camera|0
pair_02|assets/stereo_pairs/pair_02_left.jpg|assets/stereo_pairs/pair_02_right.jpg|head_stereo_camera|0
```

### `write_roi_config()`
**Format gợi ý**
```text
center_region|160|80|220|180
left_region|0|80|180|180
right_region|360|80|180|180
```

### `write_stereo_bm_config()`
**Format gợi ý**
```text
matcher_type=StereoBM
num_disparities=64
block_size=15
min_disparity=0
texture_threshold=10
uniqueness_ratio=12
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 stereo pairs**
- tạo ít nhất **3 ROI**
- set stereo disparity config
- ghi đủ:
  - `config/stereo_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/stereo_bm_config.txt`

---

# 11.4 C++ — `BaseSensor`

**File:**

```text
cpp/include/BaseSensor.hpp
```

- giống các bài trước

---

# 11.5 C++ — `StereoCameraSensor`

**File:**

```text
cpp/include/StereoCameraSensor.hpp
cpp/src/StereoCameraSensor.cpp
```

## Thuộc tính cần có

```cpp
private:
    int stereo_id;
    std::string left_camera_name;
    std::string right_camera_name;
```

## Hàm cần có
- constructor
- `print_info()`

---

# 11.6 C++ — `StereoFrameRecord`

**File:**

```text
cpp/include/StereoFrameRecord.hpp
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

# 11.7 C++ — `ROIConfig`

**File:**

```text
cpp/include/ROIConfig.hpp
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

# 11.8 C++ — `StereoBMConfig`

**File:**

```text
cpp/include/StereoBMConfig.hpp
```

Tạo struct:

```cpp
struct StereoBMConfig
```

## Thuộc tính cần có

```cpp
std::string matcher_type;   // StereoBM hoặc StereoSGBM
int num_disparities;
int block_size;
int min_disparity;
int texture_threshold;
int uniqueness_ratio;
```

> Nếu bạn dùng `StereoSGBM`, có thể mở rộng thêm `P1`, `P2`, `speckleWindowSize`, ...  
> Nhưng trong bài starter này, ưu tiên **StereoBM** cho dễ nắm.

---

# 11.9 C++ — `DisparityROIStats`

**File:**

```text
cpp/include/DisparityROIStats.hpp
```

Tạo struct:

```cpp
struct DisparityROIStats
```

## Thuộc tính cần có

```cpp
std::string pair_name;
std::string roi_name;
cv::Rect roi_rect;

double min_disparity;
double max_disparity;
double mean_disparity;
double valid_ratio;

bool is_valid;
```

### Giải thích
- `valid_ratio` = tỉ lệ pixel disparity hợp lệ trong ROI  
- bạn có thể định nghĩa pixel hợp lệ là pixel có disparity > 0

---

# 11.10 C++ — `StereoDisparitySceneResult`

**File:**

```text
cpp/include/StereoDisparitySceneResult.hpp
```

Tạo struct:

```cpp
struct StereoDisparitySceneResult
```

## Thuộc tính cần có

```cpp
std::string pair_name;

std::string left_image_path;
std::string right_image_path;

std::string left_overlay_output_path;
std::string disparity_output_path;

std::string sensor_name;
int sensor_id;

int image_width;
int image_height;

int roi_count;
bool is_valid;

std::vector<DisparityROIStats> roi_stats;
```

---

# 11.11 C++ — `BaseDisparityAnalyzer`

**File:**

```text
cpp/include/BaseDisparityAnalyzer.hpp
```

Tạo class trừu tượng:

```cpp
class BaseDisparityAnalyzer
```

## Hàm cần có

```cpp
virtual void load_stereo_manifest(const std::string& path) = 0;
virtual void load_roi_config(const std::string& path) = 0;
virtual void load_stereo_bm_config(const std::string& path) = 0;
virtual void run_disparity_analysis() = 0;
virtual ~BaseDisparityAnalyzer() = default;
```

---

# 11.12 C++ — `StereoDisparityAnalyzer`

**File:**

```text
cpp/include/StereoDisparityAnalyzer.hpp
cpp/src/StereoDisparityAnalyzer.cpp
```

Tạo class kế thừa:

```cpp
class StereoDisparityAnalyzer : public BaseDisparityAnalyzer
```

## Thuộc tính cần có

```cpp
private:
    std::vector<StereoFrameRecord> stereo_records;
    std::vector<ROIConfig> roi_configs;
    StereoBMConfig stereo_bm_config;

    std::vector<StereoDisparitySceneResult> scene_results;
```

---

## Hàm cần có

### Load / Read config
- `read_stereo_manifest(...)`
- `read_roi_config(...)`
- `read_stereo_bm_config(...)`
- các hàm `load_...(...) override`

---

## Stereo disparity part

### `cv::Rect clamp_roi_to_image(const ROIConfig& roi_cfg, const cv::Mat& image) const;`
- đảm bảo ROI không vượt biên ảnh

### `cv::Mat compute_disparity_map(
    const cv::Mat& left_bgr,
    const cv::Mat& right_bgr
) const;`

## Hành vi gợi ý
1. convert trái / phải sang grayscale
2. nếu `matcher_type == "StereoBM"`:
   - tạo `cv::StereoBM`
   - set `numDisparities`
   - set `blockSize`
   - set `minDisparity`
   - set `textureThreshold`
   - set `uniquenessRatio`
3. chạy `compute()`
4. chuyển disparity sang dạng dễ xử lý hơn (`CV_32F` nếu muốn)

---

### `cv::Mat build_disparity_visualization(const cv::Mat& disparity_map) const;`
## Hành vi
- normalize disparity về `0..255`
- convert sang `CV_8U`
- có thể apply colormap nếu muốn

---

### `DisparityROIStats analyze_single_roi(
    const std::string& pair_name,
    const ROIConfig& roi_cfg,
    const cv::Mat& disparity_map
) const;`

## Hành vi tổng quát
1. clamp ROI
2. crop disparity ROI
3. duyệt từng pixel disparity
4. chỉ lấy pixel hợp lệ (ví dụ disparity > 0)
5. tính:
   - min
   - max
   - mean
   - valid_ratio
6. build `DisparityROIStats`

---

### `void draw_roi_overlay(
    cv::Mat& left_image,
    const std::vector<DisparityROIStats>& roi_stats
) const;`

## Hành vi
- vẽ ROI lên ảnh trái
- ghi text:
  - tên ROI
  - mean disparity
  - valid ratio

---

### `StereoDisparitySceneResult process_single_stereo_pair(
    const StereoFrameRecord& record
);`

## Hành vi tổng quát
1. đọc ảnh trái / phải
2. nếu lỗi → result invalid
3. compute disparity map
4. build disparity visualization
5. loop qua toàn bộ ROI
6. `analyze_single_roi(...)`
7. vẽ ROI overlay lên ảnh trái
8. lưu:
   - `pair_xx_left_roi_overlay.jpg`
   - `pair_xx_disparity_visualization.jpg`
9. build `StereoDisparitySceneResult`

---

### `void run_disparity_analysis() override;`
- loop qua toàn bộ stereo pairs

### Getter

```cpp
const std::vector<StereoDisparitySceneResult>& get_scene_results() const;
```

---

# 11.13 C++ — `DisparityReportWriter`

**File:**

```text
cpp/include/DisparityReportWriter.hpp
cpp/src/DisparityReportWriter.cpp
```

Tạo class:

```cpp
class DisparityReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<StereoDisparitySceneResult>& scene_results
);`

## Format gợi ý

```text
[Stereo Disparity Scene]
Pair Name: pair_01
Left Image: assets/stereo_pairs/pair_01_left.jpg
Right Image: assets/stereo_pairs/pair_01_right.jpg
Left Overlay Output: assets/outputs/pair_01_left_roi_overlay.jpg
Disparity Output: assets/outputs/pair_01_disparity_visualization.jpg
Sensor: head_stereo_camera
ROI Count: 3
Valid: true

  [ROI Disparity]
  ROI Name: center_region
  Rect: x=160, y=80, w=220, h=180
  Min Disparity: 8.5
  Max Disparity: 42.0
  Mean Disparity: 24.7
  Valid Ratio: 0.81
  Valid: true

----------------------------------------
```

---

# 11.14 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- in thông tin sensor
- tạo `StereoDisparityAnalyzer`
- load:
  - `config/stereo_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/stereo_bm_config.txt`
- chạy `run_disparity_analysis()`
- tạo `DisparityReportWriter`
- ghi report ra:
  - `assets/outputs/disparity_report.txt`

## Pipeline `main.cpp`

```text
Create StereoCameraSensor
→ Load Stereo Pair Manifest
→ Load ROI Config
→ Load StereoBM Config
→ Run Stereo Disparity Analysis
→ Save Left Overlay + Disparity Visualization
→ Write Disparity Report
```

---

# 12. Điều kiện bắt buộc

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
- disparity map computation
- ROI disparity statistics
- disparity visualization
- report scene-level

---

# 13. Output mong muốn

## File config
```text
config/stereo_pair_manifest.txt
config/roi_config.txt
config/stereo_bm_config.txt
```

## Ảnh output
```text
assets/outputs/pair_01_left_roi_overlay.jpg
assets/outputs/pair_01_disparity_visualization.jpg
assets/outputs/pair_02_left_roi_overlay.jpg
assets/outputs/pair_02_disparity_visualization.jpg
```

## File report
```text
assets/outputs/disparity_report.txt
```

---

## Ví dụ `stereo_pair_manifest.txt`

```text
pair_01|assets/stereo_pairs/pair_01_left.jpg|assets/stereo_pairs/pair_01_right.jpg|head_stereo_camera|0
pair_02|assets/stereo_pairs/pair_02_left.jpg|assets/stereo_pairs/pair_02_right.jpg|head_stereo_camera|0
pair_03|assets/stereo_pairs/pair_03_left.jpg|assets/stereo_pairs/pair_03_right.jpg|head_stereo_camera|0
```

---

## Ví dụ `stereo_bm_config.txt`

```text
matcher_type=StereoBM
num_disparities=64
block_size=15
min_disparity=0
texture_threshold=10
uniqueness_ratio=12
```

---

## Ví dụ `disparity_report.txt`

```text
[Stereo Disparity Scene]
Pair Name: pair_01
Left Image: assets/stereo_pairs/pair_01_left.jpg
Right Image: assets/stereo_pairs/pair_01_right.jpg
Left Overlay Output: assets/outputs/pair_01_left_roi_overlay.jpg
Disparity Output: assets/outputs/pair_01_disparity_visualization.jpg
Sensor: head_stereo_camera
ROI Count: 3
Valid: true

  [ROI Disparity]
  ROI Name: center_region
  Rect: x=160, y=80, w=220, h=180
  Min Disparity: 8.5
  Max Disparity: 42.0
  Mean Disparity: 24.7
  Valid Ratio: 0.81
  Valid: true

----------------------------------------
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo **stereo pair manifest**
- tạo **ROI config**
- tạo **disparity config**
- có thể tạo tool phụ để vẽ histogram disparity sau khi chạy xong

Tức là Python làm phần:

```text
Stereo Disparity Config Builder + Report Helper
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- đọc stereo pair
- compute disparity map
- phân tích disparity theo ROI
- lưu overlay + disparity visualization + report

Tức là C++ làm phần:

```text
Stereo Disparity Runtime Analyzer
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **biến cặp ảnh trái/phải thành disparity map**
- **đo disparity trong từng vùng quan tâm**
- **bắt đầu suy luận vùng gần / xa trong ảnh**

Tức là CV làm phần:

```text
Left/Right Stereo Pair → Disparity Map → ROI Disparity Analysis
```

---

# 15. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `stereo_pair_manifest.txt`
- [ ] Python tạo được `roi_config.txt`
- [ ] Python tạo được `stereo_bm_config.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / string / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `StereoCameraSensor`
- [ ] C++ có `StereoFrameRecord`
- [ ] C++ có `ROIConfig`
- [ ] C++ có `StereoBMConfig`
- [ ] C++ có `DisparityROIStats`
- [ ] C++ có `StereoDisparitySceneResult`
- [ ] C++ có `BaseDisparityAnalyzer`
- [ ] C++ có `StereoDisparityAnalyzer`
- [ ] C++ load được stereo manifest
- [ ] C++ load được ROI config
- [ ] C++ load được stereo BM config
- [ ] C++ compute được disparity map
- [ ] C++ build được disparity visualization
- [ ] C++ tính được ROI disparity statistics
- [ ] C++ vẽ được ROI overlay
- [ ] C++ lưu được output images
- [ ] C++ build được disparity report

---

# 16. Gợi ý mở rộng

## 1. Hỗ trợ cả StereoBM và StereoSGBM
Bài starter có thể dùng StereoBM, sau đó mở rộng thêm StereoSGBM để so sánh chất lượng.

## 2. Thêm histogram disparity cho từng ROI
Python có thể đọc report hoặc file CSV rồi vẽ histogram bằng `matplotlib`.

## 3. Kết hợp proposal từ Bài 10
Thay vì ROI cố định, bạn có thể:
- lấy proposal box từ Bài 10
- dùng proposal box đó làm ROI disparity region

## 4. Chuẩn bị cho Bài 12
Sau Bài 11, bước hợp lý nhất cho **Bài 12** là:

```text
Robot Depth Estimation Workbench
```

tức là:
- nhận disparity
- dùng baseline + focal length
- tính depth
- so sánh depth giữa nhiều ROI / object

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 11**, bạn sẽ mở đầu **Đợt 3** theo đúng trục stereo-depth:

- **Bài 10**: proposal correspondence giữa ảnh trái / phải
- **Bài 11**: **disparity map + ROI disparity analysis**

Tức là bạn đã đi từ:

```text
left-right proposal matching
```

sang

```text
left-right disparity reasoning ở mức pixel / ROI
```

Đây là nền rất đẹp để sang **Bài 12** làm **depth estimation** thật sự.  
Nếu bạn muốn, bước tiếp theo mình sẽ làm luôn **Bài 12** theo hướng:

> **Robot Depth Estimation Workbench**  
> *(lấy disparity → tính depth → so sánh depth theo ROI / object / scene)*.
