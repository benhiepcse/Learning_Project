# 🤖 Bài 4: Robot Shape Detector — Nhận diện hình dạng vật thể cho Humanoid Robot AI Perception

> Mini Project số 4 trong **Đợt 1 — Bài 1 → Bài 5**  
> Trọng tâm: dùng **Python + C++ + Computer Vision cơ bản** để xây một module perception giúp robot nhận diện **hình dạng vật thể** trong ảnh như **triangle / rectangle / square / circle**.  
> Nếu **Bài 3** là bước robot hiểu **biên và contour**, thì **Bài 4** là bước robot bắt đầu **gán nhãn hình dạng** cho vật thể dựa trên contour đó.

---

# 📌 Mục lục

- [1. Mô tả](#1-mô-tả)
- [2. Bài 4 nằm ở đâu trong Đợt 1](#2-bài-4-nằm-ở-đâu-trong-đợt-1)
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
  - [8.1 Python — Shape Config Builder](#81-python--shape-config-builder)
  - [8.2 Python — Shape Dataset Manifest Builder](#82-python--shape-dataset-manifest-builder)
  - [8.3 Python — main_config_builder.py](#83-python--main_config_builderpy)
  - [8.4 C++ — BaseSensor](#84-c--basesensor)
  - [8.5 C++ — CameraSensor](#85-c--camerasensor)
  - [8.6 C++ — ShapeConfig](#86-c--shapeconfig)
  - [8.7 C++ — ShapeDetectionResult](#87-c--shapedetectionresult)
  - [8.8 C++ — BaseShapeDetector](#88-c--baseshapedetector)
  - [8.9 C++ — ShapeDetector](#89-c--shapedetector)
  - [8.10 C++ — ShapeReportWriter](#810-c--shapereportwriter)
  - [8.11 C++ — main.cpp](#811-c--maincpp)
- [9. Điều kiện bắt buộc](#9-điều-kiện-bắt-buộc)
- [10. Output mong muốn](#10-output-mong-muốn)
- [11. Vai trò của bài này trong Humanoid Robot](#11-vai-trò-của-bài-này-trong-humanoid-robot)
- [12. Checklist hoàn thành](#12-checklist-hoàn-thành)
- [13. Gợi ý mở rộng](#13-gợi-ý-mở-rộng)

---

# 1. Mô tả

Ở **Bài 3**, bạn đã có một module có thể:

- đọc ảnh
- grayscale + blur
- detect edge
- tìm contour
- tính area / perimeter / bounding box

Bài 4 nâng perception lên một bước rất quan trọng:

> Robot không chỉ cần biết **“biên của vật thể nằm ở đâu”**, mà còn cần biết **“vật thể này có hình gì”**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc ảnh
- tìm contour
- xấp xỉ contour thành polygon
- đếm số đỉnh
- phân loại shape
- vẽ nhãn shape lên ảnh
- lưu report phát hiện shape

Ví dụ robot nhìn thấy:
- một biển báo hình **tam giác**
- một khối **vuông**
- một hộp **chữ nhật**
- một nắp chai gần **hình tròn**

thì module này phải gán nhãn đúng shape cho từng vật thể.

---

# 2. Bài 4 nằm ở đâu trong Đợt 1

## Đợt 1 = Bài 1 → Bài 5
Đây là cụm nền tảng perception cơ bản:

- **Bài 1** — Robot Image Inspector
- **Bài 2** — Robot Color Object Detector
- **Bài 3** — Robot Edge & Contour Inspector
- **Bài 4** — Robot Shape Detector
- **Bài 5** — Robot Feature Matcher Starter

Tức là **Bài 4** là bước robot chuyển từ **“biên / contour”** sang **“nhận diện hình dạng”**.

---

# 3. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng perception:

```text
Image
→ Grayscale / Preprocess
→ Edge / Binary Shape Regions
→ Contour Extraction
→ Polygon Approximation
→ Vertex Counting / Geometric Rules
→ Shape Labeling
→ Save Detection Results
```

Đây là nền cho:
- object shape classification cổ điển
- tabletop object sorting
- marker recognition
- pre-processing cho manipulation perception
- classical geometric reasoning trước khi sang feature/depth

---

# 4. Pipeline perception của bài

```text
Image Input
→ Read Image
→ Convert to Grayscale
→ Blur / Threshold or Edge
→ Find Contours
→ Filter Small Contours
→ Approximate Polygon
→ Classify Shape
→ Draw Bounding Box + Label
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
- `cv::GaussianBlur`
- `cv::threshold` hoặc `cv::Canny`
- `cv::findContours`
- `cv::approxPolyDP`
- `cv::boundingRect`
- `cv::contourArea`
- `cv::arcLength`
- `cv::drawContours`
- `cv::rectangle`
- `cv::putText`
- `cv::imwrite`

---

# 5.4 CV Python

Python vẫn không phải runtime detector chính, nhưng dùng để:
- build config shape detection
- build manifest ảnh
- quản lý dataset test

---

# 6. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 4, bạn phải nắm được 3 ý quan trọng:

## 1. Contour có thể được chuyển thành “shape”
Robot không chỉ thấy đường biên, mà còn suy luận:
- 3 đỉnh → triangle
- 4 đỉnh → rectangle / square
- nhiều đỉnh + tròn → circle-like

## 2. Shape detection là perception hình học cổ điển
Nó rất hữu ích khi:
- scene đơn giản
- object có silhouette rõ
- cần logic nhanh, dễ debug

## 3. Detection result phải chứa cả “geometry” và “semantic label”
Không chỉ lưu contour, mà còn lưu:
- shape label
- bounding box
- center
- area
- số đỉnh polygon

---

# 7. Cấu trúc folder

```text
mini_project_04_robot_shape_detector/
│
├─ README.md
│
├─ assets/
│  ├─ test_images/
│  │  ├─ shape_scene_01.jpg
│  │  ├─ shape_scene_02.jpg
│  │  └─ shape_scene_03.jpg
│  └─ outputs/
│     ├─ annotated_shape_scene_01.jpg
│     ├─ annotated_shape_scene_02.jpg
│     ├─ annotated_shape_scene_03.jpg
│     └─ shape_report.txt
│
├─ config/
│  ├─ shape_config.txt
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
   │  ├─ ShapeConfig.hpp
   │  ├─ ShapeDetectionResult.hpp
   │  ├─ BaseShapeDetector.hpp
   │  ├─ ShapeDetector.hpp
   │  └─ ShapeReportWriter.hpp
   │
   └─ src/
      ├─ CameraSensor.cpp
      ├─ ShapeDetector.cpp
      └─ ShapeReportWriter.cpp
```

---

# 8. Yêu cầu mini-project

# 8.1 Python — Shape Config Builder

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
shape_config_path
image_manifest_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn config

---

Tạo class con:

```python
class ShapeConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
shape_config
```

`shape_config` là một `dict`, ví dụ:

```python
{
    "blur_kernel_size": 5,
    "binary_threshold": 120,
    "min_contour_area": 300,
    "polygon_epsilon_ratio": 0.02
}
```

## Hàm cần có

### `set_shape_config(blur_kernel_size, binary_threshold, min_contour_area, polygon_epsilon_ratio)`
**Hành vi:**
- lưu config vào dict
- kiểm tra:
  - `blur_kernel_size` là số lẻ dương
  - `0 <= binary_threshold <= 255`
  - `min_contour_area > 0`
  - `polygon_epsilon_ratio > 0`

### `write_shape_config()`
**Format gợi ý**
```text
blur_kernel_size=5
binary_threshold=120
min_contour_area=300
polygon_epsilon_ratio=0.02
```

---

# 8.2 Python — Shape Dataset Manifest Builder

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
- thêm ảnh test

### `write_image_manifest()`
**Format**
```text
assets/test_images/shape_scene_01.jpg
assets/test_images/shape_scene_02.jpg
assets/test_images/shape_scene_03.jpg
```

---

# 8.3 Python — `main_config_builder.py`

## Yêu cầu
- set shape config
- thêm ít nhất 3 ảnh test
- ghi đủ:
  - `config/shape_config.txt`
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

## Thuộc tính

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

# 8.6 C++ — `ShapeConfig`

**File:**

```text
cpp/include/ShapeConfig.hpp
```

Tạo struct:

```cpp
struct ShapeConfig
```

## Thuộc tính cần có

```cpp
int blur_kernel_size;
int binary_threshold;
double min_contour_area;
double polygon_epsilon_ratio;
```

---

# 8.7 C++ — `ShapeDetectionResult`

**File:**

```text
cpp/include/ShapeDetectionResult.hpp
```

Tạo struct:

```cpp
struct ShapeDetectionResult
```

## Thuộc tính cần có

```cpp
std::string image_path;
std::string shape_label;
cv::Rect bounding_box;
cv::Point center;
double area;
double perimeter;
int polygon_vertex_count;
bool is_valid;
```

---

# 8.8 C++ — `BaseShapeDetector`

**File:**

```text
cpp/include/BaseShapeDetector.hpp
```

Tạo class trừu tượng:

```cpp
class BaseShapeDetector
```

## Hàm cần có

```cpp
virtual void load_shape_config(const std::string& path) = 0;
virtual void load_image_manifest(const std::string& path) = 0;
virtual void run_detection() = 0;
virtual ~BaseShapeDetector() = default;
```

---

# 8.9 C++ — `ShapeDetector`

**File:**

```text
cpp/include/ShapeDetector.hpp
cpp/src/ShapeDetector.cpp
```

Tạo class kế thừa:

```cpp
class ShapeDetector : public BaseShapeDetector
```

## Thuộc tính cần có

```cpp
private:
    ShapeConfig shape_config;
    std::vector<std::string> image_paths;
    std::vector<ShapeDetectionResult> detections;
```

---

## Hàm cần có

### `ShapeConfig read_shape_config(const std::string& path);`
- đọc `config/shape_config.txt`

### `std::vector<std::string> read_image_manifest(const std::string& path);`
- đọc `config/image_manifest.txt`

### `void load_shape_config(const std::string& path) override;`
### `void load_image_manifest(const std::string& path) override;`

---

### `cv::Mat preprocess_image(const cv::Mat& image) const;`
**Hành vi:**
1. BGR → grayscale
2. Gaussian blur
3. threshold nhị phân hoặc tạo edge image tùy bạn chọn

---

### `std::vector<std::vector<cv::Point>> extract_contours(const cv::Mat& processed_image) const;`
- dùng `cv::findContours()`

---

### `bool is_valid_contour(const std::vector<cv::Point>& contour) const;`
**Hành vi:**
- contour area phải >= `min_contour_area`

---

### `std::vector<cv::Point> approximate_polygon(const std::vector<cv::Point>& contour) const;`
**Hành vi:**
- dùng `cv::approxPolyDP()`
- epsilon = `polygon_epsilon_ratio * perimeter`

---

### `std::string classify_shape(
    const std::vector<cv::Point>& polygon,
    const cv::Rect& bounding_box
) const;`

## Gợi ý logic
- **3 đỉnh** → `"triangle"`
- **4 đỉnh**:
  - nếu tỉ lệ width / height gần 1 → `"square"`
  - ngược lại → `"rectangle"`
- **5 đỉnh** → `"pentagon"`
- **nhiều đỉnh**:
  - nếu contour đủ tròn → `"circle_like"`
  - không thì `"polygon"`

---

### `ShapeDetectionResult build_detection_result(
    const std::string& image_path,
    const std::vector<cv::Point>& contour
) const;`

**Hành vi:**
- tính:
  - bounding box
  - center
  - area
  - perimeter
  - polygon approximation
  - shape label
  - số đỉnh polygon

---

### `void draw_detection(
    cv::Mat& image,
    const std::vector<cv::Point>& contour,
    const ShapeDetectionResult& result
) const;`

**Hành vi:**
- vẽ contour
- vẽ bounding box
- vẽ center
- ghi shape label + area

---

### `void process_single_image(const std::string& image_path);`
## Hành vi tổng quát
1. đọc ảnh
2. preprocess ảnh
3. find contours
4. lọc contour
5. build `ShapeDetectionResult`
6. draw detection
7. lưu annotated image

---

### `void run_detection() override;`
- loop qua toàn bộ ảnh
- gọi `process_single_image()`

### Getter

```cpp
const std::vector<ShapeDetectionResult>& get_detections() const;
```

---

# 8.10 C++ — `ShapeReportWriter`

**File:**

```text
cpp/include/ShapeReportWriter.hpp
cpp/src/ShapeReportWriter.cpp
```

Tạo class:

```cpp
class ShapeReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<ShapeDetectionResult>& detections
);`

## Format gợi ý

```text
[Shape Detection]
Image: assets/test_images/shape_scene_01.jpg
Shape: rectangle
Bounding Box: x=120, y=80, w=64, h=70
Center: (152, 115)
Area: 3480
Perimeter: 255.3
Polygon Vertices: 4
Valid: true
----------------------------------------
```

---

# 8.11 C++ — `main.cpp`

## Yêu cầu
- tạo `CameraSensor`
- in thông tin camera
- tạo `ShapeDetector`
- load:
  - `config/shape_config.txt`
  - `config/image_manifest.txt`
- chạy detection
- tạo `ShapeReportWriter`
- ghi report

## Pipeline `main.cpp`

```text
Create CameraSensor
→ Load Shape Config
→ Load Image Manifest
→ Run Shape Detection
→ Save Annotated Images
→ Write Shape Report
```

---

# 9. Điều kiện bắt buộc

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
- `std::vector`
- `list` / `dict`
- contour extraction
- polygon approximation bằng **approxPolyDP**
- shape classification rule-based
- detection result đóng gói bằng `struct` hoặc class

---

# 10. Output mong muốn

## File config
```text
config/shape_config.txt
config/image_manifest.txt
```

## Ảnh output
```text
assets/outputs/annotated_shape_scene_01.jpg
assets/outputs/annotated_shape_scene_02.jpg
assets/outputs/annotated_shape_scene_03.jpg
```

## File report
```text
assets/outputs/shape_report.txt
```

---

## Ví dụ nội dung `shape_config.txt`

```text
blur_kernel_size=5
binary_threshold=120
min_contour_area=300
polygon_epsilon_ratio=0.02
```

---

## Ví dụ nội dung `shape_report.txt`

```text
[Shape Detection]
Image: assets/test_images/shape_scene_01.jpg
Shape: square
Bounding Box: x=120, y=80, w=70, h=70
Center: (155, 115)
Area: 4900
Perimeter: 280.0
Polygon Vertices: 4
Valid: true
----------------------------------------

[Shape Detection]
Image: assets/test_images/shape_scene_01.jpg
Shape: triangle
Bounding Box: x=240, y=90, w=80, h=75
Center: (280, 127)
Area: 2650
Perimeter: 220.4
Polygon Vertices: 3
Valid: true
----------------------------------------
```

---

# 11. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:
- tạo config shape detection
- tạo manifest dataset
- quản lý input cho perception module

Tức là Python làm phần:
```text
Perception Config / Dataset Preparation
```

## C++ đóng vai trò gì?
C++ là runtime perception chính:
- đọc ảnh
- preprocess
- contour extraction
- polygon approximation
- shape classification
- annotated output + report

Tức là C++ làm phần:
```text
Geometric Object Understanding Module
```

## Computer Vision đóng vai trò gì?
CV ở đây giúp robot đi từ contour sang hình dạng có nghĩa:
- contour → polygon
- polygon → shape label
- shape label → object geometry understanding

Tức là CV làm phần:
```text
Image → Geometric Shape Evidence
```

---

# 12. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `shape_config.txt`
- [ ] Python tạo được `image_manifest.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `CameraSensor`
- [ ] C++ có `BaseShapeDetector`
- [ ] C++ có `ShapeDetector`
- [ ] C++ có `ShapeConfig`
- [ ] C++ có `ShapeDetectionResult`
- [ ] C++ load được shape config
- [ ] C++ load được manifest ảnh
- [ ] C++ preprocess được ảnh
- [ ] C++ find được contour
- [ ] C++ approximate polygon được bằng `approxPolyDP`
- [ ] C++ classify được triangle / rectangle / square / circle_like
- [ ] C++ build được detection result
- [ ] C++ vẽ được contour + box + label
- [ ] C++ lưu được annotated image
- [ ] C++ ghi được shape report

---

# 13. Gợi ý mở rộng

## 1. Thêm circularity score
Bạn có thể tính:

```text
circularity = 4πA / P²
```

để phân biệt **circle_like** tốt hơn.

## 2. Tách rectangle và square tốt hơn
Ngoài `width / height`, bạn có thể kiểm tra:
- góc gần vuông
- contour ổn định hơn

## 3. Shape + color kết hợp
Bạn có thể kết hợp **Bài 2 + Bài 4**:
- red_triangle
- blue_square
- yellow_circle

## 4. Chuẩn bị cho Bài 5
Sau Bài 4, bước hợp lý tiếp theo là:

```text
Bài 5 → Robot Feature Matcher Starter
```

để đi tiếp sang:
- keypoint
- descriptor
- matching
- nền cho tracking / stereo / SLAM sau này

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 4**, bạn sẽ có bước chuyển rất rõ trong Đợt 1:

```text
Bài 1 → Robot Image Inspector
Bài 2 → Robot Color Object Detector
Bài 3 → Robot Edge & Contour Inspector
Bài 4 → Robot Shape Detector
```

Tức là bạn đi từ:

```text
“hiểu được biên và contour”
```

sang

```text
“gán được nhãn hình dạng cho vật thể”
```

Đó là bước rất hợp lý trước khi sang **Bài 5 — Feature Matcher Starter**.
