# 🤖 Bài 3: Robot Edge & Contour Inspector — Phân tích biên và contour cho Humanoid Robot AI Perception

> Mini Project số 3 trong **Đợt 1 — Bài 1 → Bài 5**  
> Trọng tâm: dùng **Python + C++ + Computer Vision cơ bản** để xây một module perception giúp robot phân tích **biên (edge)** và **contour** của vật thể trong ảnh.  
> Nếu **Bài 2** là “tìm vật thể theo màu”, thì **Bài 3** là bước tiếp theo để robot hiểu **ranh giới hình dạng của vật thể**.

---

# 📌 Mục lục

- [1. Mô tả](#1-mô-tả)
- [2. Bài 3 nằm ở đâu trong Đợt 1](#2-bài-3-nằm-ở-đâu-trong-đợt-1)
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
  - [8.1 Python — Edge Config Builder](#81-python--edge-config-builder)
  - [8.2 Python — Contour Dataset Manifest Builder](#82-python--contour-dataset-manifest-builder)
  - [8.3 Python — main_config_builder.py](#83-python--main_config_builderpy)
  - [8.4 C++ — BaseSensor](#84-c--basesensor)
  - [8.5 C++ — CameraSensor](#85-c--camerasensor)
  - [8.6 C++ — EdgeConfig](#86-c--edgeconfig)
  - [8.7 C++ — ContourResult](#87-c--contourresult)
  - [8.8 C++ — BaseContourInspector](#88-c--basecontourinspector)
  - [8.9 C++ — EdgeContourInspector](#89-c--edgecontourinspector)
  - [8.10 C++ — ContourReportWriter](#810-c--contourreportwriter)
  - [8.11 C++ — main.cpp](#811-c--maincpp)
- [9. Điều kiện bắt buộc](#9-điều-kiện-bắt-buộc)
- [10. Output mong muốn](#10-output-mong-muốn)
- [11. Vai trò của bài này trong Humanoid Robot](#11-vai-trò-của-bài-này-trong-humanoid-robot)
- [12. Checklist hoàn thành](#12-checklist-hoàn-thành)
- [13. Gợi ý mở rộng](#13-gợi-ý-mở-rộng)

---

# 1. Mô tả

Ở **Bài 2**, bạn đã có một module phát hiện object theo màu:

- threshold màu
- tạo mask
- tìm contour từ vùng màu
- tạo detection result

Bài 3 nâng perception lên thêm một nấc:

> Robot không chỉ cần biết **“vùng nào có màu gì”**, mà còn cần biết **“biên của vật thể nằm ở đâu, contour của nó ra sao”**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc ảnh
- chuyển ảnh sang grayscale
- làm mượt / tiền xử lý
- phát hiện **edge**
- tìm **contour**
- lọc contour theo kích thước
- tính một số thuộc tính cơ bản của contour
- lưu ảnh minh họa + report

Ví dụ robot nhìn thấy:
- một chai nước
- một cái hộp
- một vật hình tròn
- một vùng có biên rõ

thì module này phải phát hiện được **đường biên** và **ranh giới contour** của các vật đó.

---

# 2. Bài 3 nằm ở đâu trong Đợt 1

## Đợt 1 = Bài 1 → Bài 5
Đây là cụm nền tảng perception cơ bản:

- **Bài 1** — Robot Image Inspector
- **Bài 2** — Robot Color Object Detector
- **Bài 3** — Robot Edge & Contour Inspector
- **Bài 4** — Robot Shape Detector
- **Bài 5** — Robot Feature Matcher Starter

Tức là **Bài 3** là bước robot học cách nhìn **cấu trúc biên** của scene, thay vì chỉ nhìn theo màu.

---

# 3. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng perception rất cơ bản nhưng cực kỳ quan trọng:

```text
Image
→ Grayscale
→ Blur / Preprocess
→ Edge Detection
→ Contour Extraction
→ Contour Filtering
→ Geometric Measurements
→ Save Contour Results
```

Đây là nền cho:
- shape detection
- segmentation cổ điển
- object boundary analysis
- pre-processing cho feature extraction
- tiền xử lý cho depth / stereo / object proposal

---

# 4. Pipeline perception của bài

```text
Image Input
→ Read Image
→ Convert to Grayscale
→ Blur / Denoise
→ Detect Edges
→ Find Contours
→ Filter Small Contours
→ Compute Contour Features
→ Draw Contours + Bounding Boxes
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
- `cv::Canny`
- `cv::findContours`
- `cv::contourArea`
- `cv::arcLength`
- `cv::boundingRect`
- `cv::drawContours`
- `cv::rectangle`
- `cv::putText`
- `cv::imwrite`

---

# 5.4 CV Python

Ở bài này Python vẫn không phải runtime CV chính, nhưng dùng để:
- build config cho Canny / blur
- build manifest ảnh
- quản lý dataset test

---

# 6. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 3, bạn phải nắm được 3 ý rất quan trọng:

## 1. Robot có thể dùng edge để hiểu ranh giới vật thể
Không cần màu, chỉ cần thay đổi cường độ ảnh cũng có thể tìm biên.

## 2. Contour là một biểu diễn hình học quan trọng
Từ contour, robot có thể tính:
- area
- perimeter
- bounding box
- center
- shape hint cơ bản

## 3. Một perception module tốt phải tách:
- **config**
- **processing**
- **result**
- **report**

---

# 7. Cấu trúc folder

```text
mini_project_03_robot_edge_contour_inspector/
│
├─ README.md
│
├─ assets/
│  ├─ test_images/
│  │  ├─ contour_scene_01.jpg
│  │  ├─ contour_scene_02.jpg
│  │  └─ contour_scene_03.jpg
│  └─ outputs/
│     ├─ annotated_contour_scene_01.jpg
│     ├─ annotated_contour_scene_02.jpg
│     ├─ edge_contour_scene_01.jpg
│     └─ contour_report.txt
│
├─ config/
│  ├─ edge_config.txt
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
   │  ├─ EdgeConfig.hpp
   │  ├─ ContourResult.hpp
   │  ├─ BaseContourInspector.hpp
   │  ├─ EdgeContourInspector.hpp
   │  └─ ContourReportWriter.hpp
   │
   └─ src/
      ├─ CameraSensor.cpp
      ├─ EdgeContourInspector.cpp
      └─ ContourReportWriter.cpp
```

---

# 8. Yêu cầu mini-project

# 8.1 Python — Edge Config Builder

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
edge_config_path
image_manifest_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn config

---

Tạo class con:

```python
class EdgeConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
edge_config
```

`edge_config` là một `dict`, ví dụ:

```python
{
    "blur_kernel_size": 5,
    "canny_threshold_1": 50,
    "canny_threshold_2": 150,
    "min_contour_area": 300
}
```

## Hàm cần có

### `set_edge_config(blur_kernel_size, canny_t1, canny_t2, min_contour_area)`
**Hành vi:**
- lưu config vào dict
- kiểm tra:
  - `blur_kernel_size` là số lẻ dương
  - threshold > 0
  - `canny_threshold_2 >= canny_threshold_1`
  - `min_contour_area > 0`

### `write_edge_config()`
**Format gợi ý**
```text
blur_kernel_size=5
canny_threshold_1=50
canny_threshold_2=150
min_contour_area=300
```

---

# 8.2 Python — Contour Dataset Manifest Builder

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
assets/test_images/contour_scene_01.jpg
assets/test_images/contour_scene_02.jpg
assets/test_images/contour_scene_03.jpg
```

---

# 8.3 Python — `main_config_builder.py`

## Yêu cầu
- set edge config
- thêm ít nhất 3 ảnh test
- ghi đủ:
  - `config/edge_config.txt`
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

# 8.6 C++ — `EdgeConfig`

**File:**

```text
cpp/include/EdgeConfig.hpp
```

Tạo struct:

```cpp
struct EdgeConfig
```

## Thuộc tính cần có

```cpp
int blur_kernel_size;
int canny_threshold_1;
int canny_threshold_2;
double min_contour_area;
```

---

# 8.7 C++ — `ContourResult`

**File:**

```text
cpp/include/ContourResult.hpp
```

Tạo struct:

```cpp
struct ContourResult
```

## Thuộc tính cần có

```cpp
std::string image_path;
cv::Rect bounding_box;
cv::Point center;
double area;
double perimeter;
bool is_valid;
```

---

# 8.8 C++ — `BaseContourInspector`

**File:**

```text
cpp/include/BaseContourInspector.hpp
```

Tạo class trừu tượng:

```cpp
class BaseContourInspector
```

## Hàm cần có

```cpp
virtual void load_edge_config(const std::string& path) = 0;
virtual void load_image_manifest(const std::string& path) = 0;
virtual void run_inspection() = 0;
virtual ~BaseContourInspector() = default;
```

---

# 8.9 C++ — `EdgeContourInspector`

**File:**

```text
cpp/include/EdgeContourInspector.hpp
cpp/src/EdgeContourInspector.cpp
```

Tạo class kế thừa:

```cpp
class EdgeContourInspector : public BaseContourInspector
```

## Thuộc tính cần có

```cpp
private:
    EdgeConfig edge_config;
    std::vector<std::string> image_paths;
    std::vector<ContourResult> contour_results;
```

---

## Hàm cần có

### `EdgeConfig read_edge_config(const std::string& path);`
- đọc `config/edge_config.txt`

### `std::vector<std::string> read_image_manifest(const std::string& path);`
- đọc `config/image_manifest.txt`

### `void load_edge_config(const std::string& path) override;`
### `void load_image_manifest(const std::string& path) override;`

---

### `cv::Mat preprocess_grayscale(const cv::Mat& image) const;`
**Hành vi:**
1. chuyển BGR → grayscale
2. Gaussian blur

---

### `cv::Mat detect_edges(const cv::Mat& gray_image) const;`
**Hành vi:**
- dùng `cv::Canny()` với threshold trong config

---

### `std::vector<std::vector<cv::Point>> extract_contours(const cv::Mat& edge_image) const;`
- dùng `cv::findContours()`

---

### `bool is_valid_contour(const std::vector<cv::Point>& contour) const;`
**Hành vi:**
- contour area phải >= `min_contour_area`

---

### `ContourResult build_contour_result(
    const std::string& image_path,
    const std::vector<cv::Point>& contour
) const;`

**Hành vi:**
- tính:
  - bounding box
  - center
  - area
  - perimeter

---

### `void draw_contour_result(
    cv::Mat& image,
    const std::vector<cv::Point>& contour,
    const ContourResult& result
) const;`

**Hành vi:**
- vẽ contour
- vẽ bounding box
- vẽ center
- ghi area / perimeter

---

### `void process_single_image(const std::string& image_path);`
## Hành vi tổng quát
1. đọc ảnh
2. preprocess grayscale
3. detect edges
4. find contours
5. lọc contour
6. build contour result
7. draw result
8. lưu:
   - edge image
   - annotated image

---

### `void run_inspection() override;`
- loop qua toàn bộ ảnh
- gọi `process_single_image()`

### Getter

```cpp
const std::vector<ContourResult>& get_contour_results() const;
```

---

# 8.10 C++ — `ContourReportWriter`

**File:**

```text
cpp/include/ContourReportWriter.hpp
cpp/src/ContourReportWriter.cpp
```

Tạo class:

```cpp
class ContourReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<ContourResult>& contour_results
);`

## Format gợi ý

```text
[Contour]
Image: assets/test_images/contour_scene_01.jpg
Bounding Box: x=120, y=80, w=64, h=70
Center: (152, 115)
Area: 3480
Perimeter: 255.3
Valid: true
----------------------------------------
```

---

# 8.11 C++ — `main.cpp`

## Yêu cầu
- tạo `CameraSensor`
- in thông tin camera
- tạo `EdgeContourInspector`
- load:
  - `config/edge_config.txt`
  - `config/image_manifest.txt`
- chạy inspection
- tạo `ContourReportWriter`
- ghi report

## Pipeline `main.cpp`

```text
Create CameraSensor
→ Load Edge Config
→ Load Image Manifest
→ Run Edge + Contour Inspection
→ Save Edge Images
→ Save Annotated Images
→ Write Contour Report
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
- edge detection bằng **Canny**
- contour extraction bằng **findContours**
- contour result đóng gói bằng `struct` hoặc class

---

# 10. Output mong muốn

## File config
```text
config/edge_config.txt
config/image_manifest.txt
```

## Ảnh output
```text
assets/outputs/edge_contour_scene_01.jpg
assets/outputs/edge_contour_scene_02.jpg
assets/outputs/edge_contour_scene_03.jpg

assets/outputs/annotated_contour_scene_01.jpg
assets/outputs/annotated_contour_scene_02.jpg
assets/outputs/annotated_contour_scene_03.jpg
```

## File report
```text
assets/outputs/contour_report.txt
```

---

## Ví dụ nội dung `edge_config.txt`

```text
blur_kernel_size=5
canny_threshold_1=50
canny_threshold_2=150
min_contour_area=300
```

---

## Ví dụ nội dung `contour_report.txt`

```text
[Contour]
Image: assets/test_images/contour_scene_01.jpg
Bounding Box: x=120, y=80, w=64, h=70
Center: (152, 115)
Area: 3480
Perimeter: 255.3
Valid: true
----------------------------------------

[Contour]
Image: assets/test_images/contour_scene_01.jpg
Bounding Box: x=250, y=100, w=80, h=92
Center: (290, 146)
Area: 5120
Perimeter: 301.8
Valid: true
----------------------------------------
```

---

# 11. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:
- tạo config cho edge detection
- tạo manifest dataset
- quản lý input cho perception module

Tức là Python làm phần:
```text
Perception Config / Dataset Preparation
```

## C++ đóng vai trò gì?
C++ là runtime perception chính:
- đọc ảnh
- grayscale + blur
- Canny edge
- contour extraction
- contour measurement
- annotated output + report

Tức là C++ làm phần:
```text
Vision Geometry Extraction Module
```

## Computer Vision đóng vai trò gì?
CV ở đây giúp robot chuyển từ “ảnh pixel” sang “hình học biên”:
- edge → ranh giới cục bộ
- contour → biên khép kín / vùng vật thể
- bounding box / area / perimeter → mô tả hình học

Tức là CV làm phần:
```text
Image → Shape Boundary Evidence
```

---

# 12. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `edge_config.txt`
- [ ] Python tạo được `image_manifest.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `CameraSensor`
- [ ] C++ có `BaseContourInspector`
- [ ] C++ có `EdgeContourInspector`
- [ ] C++ có `EdgeConfig`
- [ ] C++ có `ContourResult`
- [ ] C++ load được edge config
- [ ] C++ load được manifest ảnh
- [ ] C++ grayscale + blur được ảnh
- [ ] C++ detect được edge bằng Canny
- [ ] C++ find được contour
- [ ] C++ lọc được contour nhỏ
- [ ] C++ build được contour result
- [ ] C++ vẽ được contour + box + center
- [ ] C++ lưu được edge image
- [ ] C++ lưu được annotated image
- [ ] C++ ghi được contour report

---

# 13. Gợi ý mở rộng

## 1. So sánh nhiều cấu hình Canny
Ví dụ:
- threshold thấp → nhiều edge hơn
- threshold cao → ít edge hơn

## 2. Thêm contour hierarchy
Nếu muốn nâng cấp, bạn có thể lưu luôn:
- contour cha / con
- lỗ bên trong object

## 3. Thêm polygon approximation
Chuẩn bị cho **Bài 4 — Shape Detector** bằng cách:
- dùng `cv::approxPolyDP()`
- đếm số đỉnh contour

## 4. Chuẩn bị cho Bài 4
Sau Bài 3, bước hợp lý tiếp theo là:

```text
Bài 4 → Robot Shape Detector
```

để đi tiếp sang:
- triangle / rectangle / circle detection
- polygon approximation
- shape labeling

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 3**, bạn sẽ có bước chuyển rất rõ trong Đợt 1:

```text
Bài 1 → Robot Image Inspector
Bài 2 → Robot Color Object Detector
Bài 3 → Robot Edge & Contour Inspector
```

Tức là bạn đi từ:

```text
“tách được object theo màu”
```

sang

```text
“hiểu được biên và contour của vật thể trong ảnh”
```

Đó là nền rất quan trọng để bước sang **shape detection** ở Bài 4.
