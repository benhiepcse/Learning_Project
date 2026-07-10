# 🤖 Bài 2: Robot Color Object Detector — Phát hiện vật thể theo màu cho Humanoid Robot AI Perception

> Mini Project số 2 trong **Đợt 1 — Bài 1 → Bài 5**  
> Trọng tâm: dùng **Python + C++ + Computer Vision cơ bản** để xây một module perception nhỏ giúp robot nhận biết vật thể theo **màu sắc** trong ảnh.  
> Đây là bài đầu tiên trong chuỗi mini-project đi đúng vai trò của **AI Perception**: từ việc chỉ đọc ảnh ở **Bài 1**, sang **phân tích ảnh và tìm object** ở **Bài 2**.

---

# 📌 Mục lục

- [1. Mô tả](#1-mô-tả)
- [2. Bài 2 nằm ở đâu trong Đợt 1](#2-bài-2-nằm-ở-đâu-trong-đợt-1)
- [3. Mục tiêu perception của bài](#3-mục-tiêu-perception-của-bài)
- [4. Pipeline perception của bài](#4-pipeline-perception-của-bài)
- [5. Kiến thức cần](#5-kiến-thức-cần)
  - [5.1 C++](#51-c)
  - [5.2 Python](#52-python)
  - [5.3 CV C++](#53-cv-c)
  - [5.4 CV Python](#54-cv-python)
- [6. Sau bài này bạn sẽ hiểu gì trong AI Perception](#6-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [7. Cấu trúc folder](#7-cấu-trúc-folder)
- [8. Yêu cầu mini-project](#8-yêu-cầu-mini-project)
  - [8.1 Python — Color Config Builder](#81-python--color-config-builder)
  - [8.2 Python — Color Dataset Manifest Builder](#82-python--color-dataset-manifest-builder)
  - [8.3 Python — main_config_builder.py](#83-python--main_config_builderpy)
  - [8.4 C++ — BaseSensor](#84-c--basesensor)
  - [8.5 C++ — CameraSensor](#85-c--camerasensor)
  - [8.6 C++ — ColorRange](#86-c--colorrange)
  - [8.7 C++ — DetectionResult](#87-c--detectionresult)
  - [8.8 C++ — BaseDetector](#88-c--basedetector)
  - [8.9 C++ — ColorObjectDetector](#89-c--colorobjectdetector)
  - [8.10 C++ — DetectionReportWriter](#810-c--detectionreportwriter)
  - [8.11 C++ — main.cpp](#811-c--maincpp)
- [9. Điều kiện bắt buộc](#9-điều-kiện-bắt-buộc)
- [10. Output mong muốn](#10-output-mong-muốn)
- [11. Vai trò của bài này trong Humanoid Robot](#11-vai-trò-của-bài-này-trong-humanoid-robot)
- [12. Checklist hoàn thành](#12-checklist-hoàn-thành)
- [13. Gợi ý mở rộng](#13-gợi-ý-mở-rộng)

---

# 1. Mô tả

Ở **Bài 1**, bạn mới dừng ở mức:

- đọc ảnh
- kiểm tra kích thước ảnh
- lấy thông tin cơ bản của sensor / image

Bài 2 nâng cấp lên một bước rất đúng chất **AI Perception**:

> Robot không chỉ “nhìn thấy ảnh”, mà phải **tách được vùng vật thể quan trọng trong ảnh**.

Ở mini-project này, bạn sẽ xây một hệ thống nhỏ để robot:

- đọc ảnh từ camera
- đổi ảnh sang không gian màu phù hợp
- tạo **mask màu**
- tìm **contour / bounding box**
- trả ra **danh sách object theo màu**
- ghi **report phát hiện vật thể**

Ví dụ robot nhìn thấy:
- quả bóng **màu đỏ**
- khối hộp **màu xanh**
- vật đánh dấu **màu vàng**

thì module này phải phát hiện được các vùng đó trong ảnh.

---

# 2. Bài 2 nằm ở đâu trong Đợt 1

## Đợt 1 = Bài 1 → Bài 5
Đây là cụm **nền tảng perception cơ bản**:

- **Bài 1** — đọc ảnh, quản lý image input
- **Bài 2** — color object detection
- **Bài 3** — edge / contour
- **Bài 4** — shape detection
- **Bài 5** — feature matching starter

Tức là **Bài 2** là bước đầu tiên mà robot bắt đầu:

```text
Image → Mask → Object Region → Detection Result
```

---

# 3. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng xử lý rất cơ bản nhưng cực quan trọng trong robot perception:

```text
Camera Image
→ Chuyển không gian màu
→ Lọc theo khoảng màu
→ Tạo binary mask
→ Tìm vùng vật thể
→ Lấy bounding box / center / area
→ Xuất kết quả detection
```

Đây chính là kiểu logic nền để sau này bạn học:
- object segmentation đơn giản
- ROI proposal
- pre-processing cho tracking
- pre-processing cho grasping
- color marker detection trên robot

---

# 4. Pipeline perception của bài

```text
Image Input
→ Read Image
→ Convert BGR to HSV
→ Apply Color Threshold
→ Build Binary Mask
→ Morphology / Noise Cleanup
→ Find Contours
→ Filter Small Regions
→ Build Detection Results
→ Save Annotated Image + Report
```

---

# 5. Kiến thức cần

# 5.1 C++

- class / object
- constructor
- inheritance
- `std::vector`
- `std::string`
- function
- if / else
- loop
- struct
- header / source tách file

---

# 5.2 Python

- class / object
- inheritance
- list
- dict
- string
- function
- file write
- loop
- if / else
- module

---

# 5.3 CV C++

- `cv::imread`
- `cv::cvtColor`
- `cv::inRange`
- `cv::findContours`
- `cv::boundingRect`
- `cv::rectangle`
- `cv::circle`
- `cv::putText`
- `cv::imwrite`

---

# 5.4 CV Python

Ở bài này Python **không phải runtime detector chính**, nhưng vẫn cần hiểu để build config và manifest:
- đường dẫn ảnh
- danh sách màu cần detect
- ngưỡng màu HSV
- metadata cho ảnh test

---

# 6. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 2, bạn phải nắm được 3 ý rất quan trọng:

## 1. Robot không xử lý “toàn bộ ảnh” một cách mơ hồ
Nó cần **quy tắc perception cụ thể** để tìm object.

## 2. Color threshold là một cách perception cơ bản
Không phải AI/deep learning, nhưng rất hữu ích khi:
- object có marker màu rõ
- môi trường có ít nhiễu
- muốn debug pipeline nhanh

## 3. Detection result phải được đóng gói thành dữ liệu
Không chỉ vẽ box lên ảnh, mà còn phải trả ra:
- tên object / màu
- bounding box
- center
- area
- ảnh nguồn

---

# 7. Cấu trúc folder

```text
mini_project_02_robot_color_object_detector/
│
├─ README.md
│
├─ assets/
│  ├─ test_images/
│  │  ├─ color_scene_01.jpg
│  │  ├─ color_scene_02.jpg
│  │  └─ color_scene_03.jpg
│  └─ outputs/
│     ├─ annotated_color_scene_01.jpg
│     ├─ annotated_color_scene_02.jpg
│     └─ detection_report.txt
│
├─ config/
│  ├─ color_ranges.txt
│  └─ image_manifest.txt
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
   │  ├─ ColorRange.hpp
   │  ├─ DetectionResult.hpp
   │  ├─ BaseDetector.hpp
   │  ├─ ColorObjectDetector.hpp
   │  └─ DetectionReportWriter.hpp
   │
   └─ src/
      ├─ CameraSensor.cpp
      ├─ ColorObjectDetector.cpp
      └─ DetectionReportWriter.cpp
```

---

# 8. Yêu cầu mini-project

# 8.1 Python — Color Config Builder

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
color_config_path
image_manifest_path
```

## Hàm cần có

### `show_project_info()`
**Hành vi:**
- in tên project
- in đường dẫn file config

---

Tạo class con:

```python
class ColorConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
color_ranges
```

Trong đó `color_ranges` là một **list các dict**, ví dụ:

```python
[
    {
        "label": "red",
        "lower_h": 0,
        "lower_s": 120,
        "lower_v": 70,
        "upper_h": 10,
        "upper_s": 255,
        "upper_v": 255
    },
    {
        "label": "blue",
        "lower_h": 100,
        "lower_s": 120,
        "lower_v": 70,
        "upper_h": 130,
        "upper_s": 255,
        "upper_v": 255
    }
]
```

## Hàm cần có

### `add_color_range(label, lh, ls, lv, uh, us, uv)`
**Hành vi:**
- thêm một màu mới vào danh sách
- kiểm tra:
  - label không rỗng
  - giá trị HSV nằm trong khoảng hợp lệ

### `write_color_config()`
**Hành vi:**
- ghi ra file `config/color_ranges.txt`

**Format gợi ý:**
```text
red|0|120|70|10|255|255
blue|100|120|70|130|255|255
yellow|20|100|100|35|255|255
```

---

# 8.2 Python — Color Dataset Manifest Builder

Tiếp tục trong file:

```text
python/tools/config_builder.py
```

Tạo class:

```python
class ImageManifestBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
image_paths
```

## Hàm cần có

### `add_image_path(image_path)`
- thêm đường dẫn ảnh test

### `write_image_manifest()`
**Hành vi:**
- ghi ra file `config/image_manifest.txt`

**Format gợi ý:**
```text
assets/test_images/color_scene_01.jpg
assets/test_images/color_scene_02.jpg
assets/test_images/color_scene_03.jpg
```

---

# 8.3 Python — `main_config_builder.py`

**File:**

```text
python/main_config_builder.py
```

## Yêu cầu
- tạo ít nhất **3 màu** để detect
- tạo manifest ít nhất **3 ảnh**
- ghi đủ:
  - `config/color_ranges.txt`
  - `config/image_manifest.txt`

---

# 8.4 C++ — `BaseSensor`

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

# 8.5 C++ — `CameraSensor`

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

**Ví dụ camera_role:**
- `"head_rgb_camera"`
- `"left_eye_camera"`
- `"debug_color_camera"`

---

# 8.6 C++ — `ColorRange`

**File:**

```text
cpp/include/ColorRange.hpp
```

Tạo struct:

```cpp
struct ColorRange
```

## Thuộc tính cần có

```cpp
std::string label;
int lower_h;
int lower_s;
int lower_v;
int upper_h;
int upper_s;
int upper_v;
```

---

# 8.7 C++ — `DetectionResult`

**File:**

```text
cpp/include/DetectionResult.hpp
```

Tạo struct:

```cpp
struct DetectionResult
```

## Thuộc tính cần có

```cpp
std::string image_path;
std::string color_label;
cv::Rect bounding_box;
cv::Point center;
double area;
bool is_valid;
```

> `area` là diện tích contour hoặc bounding box tùy cách bạn chọn, nhưng nên thống nhất.

---

# 8.8 C++ — `BaseDetector`

**File:**

```text
cpp/include/BaseDetector.hpp
```

Tạo class trừu tượng:

```cpp
class BaseDetector
```

## Hàm cần có

```cpp
virtual void load_color_config(const std::string& path) = 0;
virtual void load_image_manifest(const std::string& path) = 0;
virtual void run_detection() = 0;
virtual ~BaseDetector() = default;
```

---

# 8.9 C++ — `ColorObjectDetector`

**File:**

```text
cpp/include/ColorObjectDetector.hpp
cpp/src/ColorObjectDetector.cpp
```

Tạo class kế thừa:

```cpp
class ColorObjectDetector : public BaseDetector
```

## Thuộc tính cần có

```cpp
private:
    std::vector<ColorRange> color_ranges;
    std::vector<std::string> image_paths;
    std::vector<DetectionResult> detections;
```

---

## Hàm cần có

### `std::vector<ColorRange> read_color_config(const std::string& path);`
**Hành vi:**
- đọc file `color_ranges.txt`
- parse từng dòng thành `ColorRange`

### `std::vector<std::string> read_image_manifest(const std::string& path);`
**Hành vi:**
- đọc file `image_manifest.txt`
- lấy danh sách đường dẫn ảnh

### `void load_color_config(const std::string& path) override;`
### `void load_image_manifest(const std::string& path) override;`

---

### `cv::Mat build_color_mask(const cv::Mat& hsv_image, const ColorRange& color_range) const;`
**Hành vi:**
- dùng `cv::inRange()` để tạo binary mask theo ngưỡng HSV

---

### `cv::Mat clean_mask(const cv::Mat& mask) const;`
**Hành vi:**
- dùng morphology hoặc blur đơn giản để giảm nhiễu
- ví dụ:
  - erode / dilate
  - opening / closing

---

### `std::vector<std::vector<cv::Point>> extract_contours(const cv::Mat& mask) const;`
**Hành vi:**
- dùng `cv::findContours()`

---

### `bool is_valid_contour(const std::vector<cv::Point>& contour) const;`
**Hành vi:**
- lọc contour quá nhỏ
- ví dụ area phải lớn hơn một ngưỡng nào đó

---

### `DetectionResult build_detection_result(
    const std::string& image_path,
    const std::string& color_label,
    const std::vector<cv::Point>& contour
) const;`

**Hành vi:**
- tạo bounding box
- tính center
- tính area
- trả `DetectionResult`

---

### `void draw_detection(cv::Mat& image, const DetectionResult& detection) const;`
**Hành vi:**
- vẽ:
  - bounding box
  - center point
  - text label + area

---

### `void process_single_image(const std::string& image_path);`
## Hành vi tổng quát
1. đọc ảnh BGR
2. chuyển sang HSV
3. loop qua từng `ColorRange`
4. tạo mask
5. clean mask
6. tìm contours
7. lọc contour hợp lệ
8. tạo `DetectionResult`
9. vẽ detection lên ảnh
10. lưu ảnh annotated

---

### `void run_detection() override;`
**Hành vi:**
- loop qua toàn bộ ảnh trong manifest
- gọi `process_single_image()`

---

### Getter cần có

```cpp
const std::vector<DetectionResult>& get_detections() const;
```

---

# 8.10 C++ — `DetectionReportWriter`

**File:**

```text
cpp/include/DetectionReportWriter.hpp
cpp/src/DetectionReportWriter.cpp
```

Tạo class:

```cpp
class DetectionReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<DetectionResult>& detections
);`

**Hành vi:**
- ghi report ra file `assets/outputs/detection_report.txt`

## Report nên có format kiểu:

```text
[Detection]
Image: assets/test_images/color_scene_01.jpg
Color: red
Bounding Box: x=120, y=80, w=64, h=70
Center: (152, 115)
Area: 3480
Valid: true
----------------------------------------
```

---

# 8.11 C++ — `main.cpp`

**File:**

```text
cpp/main.cpp
```

## Yêu cầu
- tạo một `CameraSensor`
- in thông tin camera
- tạo `ColorObjectDetector`
- load:
  - `config/color_ranges.txt`
  - `config/image_manifest.txt`
- chạy detection
- tạo `DetectionReportWriter`
- ghi report

## Pipeline `main.cpp`

```text
Create CameraSensor
→ Load Color Config
→ Load Image Manifest
→ Run Color Detection
→ Save Annotated Images
→ Write Detection Report
```

---

# 9. Điều kiện bắt buộc

Project **bắt buộc phải có**:

- OOP trong Python
- OOP trong C++
- Inheritance trong Python
- Inheritance trong C++
- Function tách rõ ràng
- Module Python
- Header / Source C++ tách file
- `loop`
- `if / else`
- `std::vector`
- `list` / `dict`
- color threshold bằng **HSV**
- detection result đóng gói bằng `struct` hoặc class

---

# 10. Output mong muốn

Sau khi chạy project, bạn phải tạo được:

## File config
```text
config/color_ranges.txt
config/image_manifest.txt
```

## Ảnh output
```text
assets/outputs/annotated_color_scene_01.jpg
assets/outputs/annotated_color_scene_02.jpg
assets/outputs/annotated_color_scene_03.jpg
```

## File report
```text
assets/outputs/detection_report.txt
```

---

## Ví dụ nội dung `color_ranges.txt`

```text
red|0|120|70|10|255|255
blue|100|120|70|130|255|255
yellow|20|100|100|35|255|255
```

---

## Ví dụ nội dung `detection_report.txt`

```text
[Detection]
Image: assets/test_images/color_scene_01.jpg
Color: red
Bounding Box: x=120, y=80, w=64, h=70
Center: (152, 115)
Area: 3480
Valid: true
----------------------------------------

[Detection]
Image: assets/test_images/color_scene_01.jpg
Color: blue
Bounding Box: x=250, y=100, w=80, h=92
Center: (290, 146)
Area: 5120
Valid: true
----------------------------------------
```

---

# 11. Vai trò của bài này trong Humanoid Robot

Bài này tuy là mini-project cơ bản, nhưng rất đúng vai trò của **AI Perception** trong humanoid robot.

## Python đóng vai trò gì?
Python ở đây đóng vai trò:
- tạo config màu
- tạo manifest dataset
- quản lý input để perception module C++ chạy

Tức là Python làm phần:
```text
Perception Config / Dataset Preparation
```

## C++ đóng vai trò gì?
C++ là runtime perception chính:
- đọc ảnh
- convert màu
- threshold
- detect contour
- tạo detection result
- vẽ annotated output

Tức là C++ làm phần:
```text
Realtime-ish Vision Processing Module
```

## Computer Vision đóng vai trò gì?
CV ở đây là lõi perception:
- biến ảnh thành mask
- biến mask thành contour
- biến contour thành object result

Tức là CV làm phần:
```text
Image → Visual Object Evidence
```

---

# 12. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `color_ranges.txt`
- [ ] Python tạo được `image_manifest.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `CameraSensor`
- [ ] C++ có `BaseDetector`
- [ ] C++ có `ColorObjectDetector`
- [ ] C++ có `ColorRange`
- [ ] C++ có `DetectionResult`
- [ ] C++ load được config màu
- [ ] C++ load được manifest ảnh
- [ ] C++ convert được BGR → HSV
- [ ] C++ tạo được color mask
- [ ] C++ clean được mask
- [ ] C++ tìm được contour
- [ ] C++ tạo được detection result
- [ ] C++ vẽ được bounding box
- [ ] C++ lưu được annotated image
- [ ] C++ ghi được detection report

---

# 13. Gợi ý mở rộng

## 1. Detect nhiều dải màu cho cùng một object
Ví dụ màu đỏ trong HSV thường phải tách thành 2 khoảng:
- đỏ thấp
- đỏ cao

Bạn có thể nâng cấp detector để hỗ trợ điều đó.

## 2. Lưu confidence kiểu đơn giản
Ví dụ:
- confidence dựa trên area
- confidence dựa trên độ đầy của contour trong bounding box

## 3. Gộp object theo loại
Ví dụ:
- red_marker
- blue_box
- yellow_ball

thay vì chỉ lưu nhãn màu.

## 4. Chuẩn bị cho Bài 3
Sau Bài 2, bước hợp lý tiếp theo là:

```text
Bài 3 → Robot Edge & Contour Inspector
```

để đi tiếp sang:
- edge detection
- contour analysis
- shape boundary understanding

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 2**, bạn sẽ có bước chuyển rất rõ trong Đợt 1:

```text
Bài 1 → Robot Image Inspector
Bài 2 → Robot Color Object Detector
```

Tức là bạn đi từ:

```text
“đọc được ảnh”
```

sang

```text
“tách được object theo màu và đóng gói thành detection result”
```

Đó là đúng kiểu bước đầu của một **Humanoid Robot AI Perception pipeline**.
