# 🤖 Bài 15: Robot 3D Object Localization Starter — Ước lượng tâm 3D và kích thước 3D của Object trong Robot Frame cho Humanoid Robot AI Perception

> Mini Project số 15 trong **Đợt 3 — Bài 11 → Bài 15**  
> **Bài 15 là bài cuối của Đợt 3** và tiếp tục kết hợp kiến thức của **Đợt 1 + Đợt 2 + Đợt 3** theo đúng rule bạn đã chốt.  
> Nếu **Bài 14** đã giúp robot đi từ **camera-frame point cloud → robot-frame point cloud**, thì **Bài 15** sẽ chốt Đợt 3 bằng một bước cực kỳ đúng chất humanoid AI perception:
>
> **gom point cloud theo ROI / object region → ước lượng vị trí 3D, tâm 3D và kích thước 3D của object trong robot frame.**

---

# 📌 Mục lục

- [1. Bài 15 lấy gì từ Đợt 3](#1-bài-15-lấy-gì-từ-đợt-3)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 15 nằm ở đâu trong roadmap](#3-bài-15-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 15 là bước tiếp theo hợp lý sau Bài 14](#4-vì-sao-bài-15-là-bước-tiếp-theo-hợp-lý-sau-bài-14)
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
  - [11.2 Python — ObjectLocalizationConfigBuilder](#112-python--objectlocalizationconfigbuilder)
  - [11.3 Python — main_config_builder.py](#113-python--main_config_builderpy)
  - [11.4 C++ — BaseSensor](#114-c--basesensor)
  - [11.5 C++ — StereoCameraSensor](#115-c--stereocamerasensor)
  - [11.6 C++ — StereoFrameRecord](#116-c--stereoframerecord)
  - [11.7 C++ — ROIConfig](#117-c--roiconfig)
  - [11.8 C++ — StereoBMConfig](#118-c--stereobmconfig)
  - [11.9 C++ — StereoCalibrationConfig](#119-c--stereocalibrationconfig)
  - [11.10 C++ — CameraIntrinsics](#1110-c--cameraintrinsics)
  - [11.11 C++ — Transform3D](#1111-c--transform3d)
  - [11.12 C++ — Point3D](#1112-c--point3d)
  - [11.13 C++ — RobotFramePoint3D](#1113-c--robotframepoint3d)
  - [11.14 C++ — LocalizedObject3D](#1114-c--localizedobject3d)
  - [11.15 C++ — ObjectLocalizationSceneResult](#1115-c--objectlocalizationsceneresult)
  - [11.16 C++ — BaseObjectLocalizer3D](#1116-c--baseobjectlocalizer3d)
  - [11.17 C++ — StereoObjectLocalizer3D](#1117-c--stereoobjectlocalizer3d)
  - [11.18 C++ — ObjectLocalizationReportWriter](#1118-c--objectlocalizationreportwriter)
  - [11.19 C++ — main.cpp](#1119-c--maincpp)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 15 lấy gì từ Đợt 3

Sau Bài 14, trục Đợt 3 của bạn đang là:

- **Stereo Camera**
- **Disparity**
- **Depth**
- **Point Cloud**
- **Camera Frame → Robot Frame**
- và bước rất hợp lý tiếp theo là:
  - **3D Object Localization**
  - **ước lượng object center**
  - **ước lượng object extent / bounding stats**
  - chuẩn bị cho object-centric manipulation reasoning

Vì vậy **Bài 15** lấy đúng phần tiếp theo:

## Phần mới của Đợt 3 mà Bài 15 dùng
### Computer Vision / 3D Geometry / Perception
- **robot-frame point cloud**
- **gom point theo ROI / object region**
- **ước lượng 3D center**
- **ước lượng min/max extents**
- **3D object statistics**

### Python
- tiếp tục làm:
  - config builder
  - ROI/object label config
  - report helper

### C++
- vẫn dùng:
  - OOP
  - vector
  - struct config / result
  - runtime pipeline

> Bài 15 **chưa ép bạn làm detector hay segmentation model**.  
> Ở giai đoạn này, object region vẫn có thể đến từ:
>
> - ROI cấu hình sẵn
> - proposal box từ các bài trước
>
> Mục tiêu chính là:
>
> ```text
> robot-frame points của một ROI
> → ước lượng object 3D center + size
> ```

---

# 2. Mô tả

Ở **Bài 14**, bạn đã có một module có thể:

- đọc stereo pair
- tính disparity map
- tính depth map
- back-project thành point cloud trong camera frame
- transform point cloud sang robot frame
- tính thống kê ROI trong robot frame
- lưu point clouds + report

Bài 15 sẽ **đi tiếp từ robot-frame point cloud sang object localization 3D cơ bản**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc **cặp ảnh stereo**
- tính disparity map
- tính depth map
- back-project thành điểm 3D trong camera frame
- transform sang **robot/base frame**
- gom điểm theo từng **ROI / object region**
- với mỗi object region, ước lượng:
  - **3D center**
  - **min/max X**
  - **min/max Y**
  - **min/max Z**
  - **size_x**
  - **size_y**
  - **size_z**
- lưu:
  - left ROI overlay
  - disparity visualization
  - depth visualization
  - camera point cloud
  - robot point cloud
  - object localization report

Ví dụ robot nhìn một chiếc hộp đặt trên bàn:

- ROI `box_region`
- sau khi transform sang robot frame, robot thu được tập điểm của chiếc hộp
- từ đó robot ước lượng:
  - tâm 3D của hộp
  - chiều rộng xấp xỉ
  - chiều cao xấp xỉ
  - chiều sâu xấp xỉ

Từ đó robot bắt đầu reason kiểu:

```text
vật nằm ở đâu trong không gian robot
tâm vật cách base bao xa
kích thước vật có phù hợp để gắp / nắm hay không
```

---

# 3. Bài 15 nằm ở đâu trong roadmap

## Quy ước hiện tại
- **Đợt 1 = Bài 1 → Bài 5**
- **Đợt 2 = Bài 6 → Bài 10**
- **Đợt 3 = Bài 11 → Bài 15**
- **Đợt 4 = Bài 16 → Bài 20**

Vì vậy:

## **Bài 15 = bài cuối của Đợt 3**
và phải **kết hợp lại kiến thức của Đợt 1 + Đợt 2 + Đợt 3**.

---

# 4. Vì sao Bài 15 là bước tiếp theo hợp lý sau Bài 14

## Bài 13 cho bạn:
- point cloud trong camera frame

## Bài 14 cho bạn:
- point cloud trong robot frame

## Bài 15 nâng thêm một nấc:
- không chỉ có point cloud nữa
- mà **biến point cloud của ROI thành object-level 3D information**

Đây là bước rất hợp lý vì perception của robot thường không dừng ở “đây là đám điểm 3D”, mà sẽ tiến tới:

```text
đây là object
object này có tâm ở đâu
object này rộng / cao / sâu khoảng bao nhiêu
```

Chuỗi suy nghĩ đúng của Đợt 3 đến đây sẽ là:

```text
stereo pair
→ disparity
→ depth
→ camera-frame point cloud
→ robot-frame point cloud
→ 3D object localization
```

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Pair + ROI/Object Config + StereoBM Config + Calibration Config + Intrinsics + Camera-to-Robot Transform
→ Load Left / Right Image Pair
→ Compute Disparity Map
→ Convert Disparity to Depth
→ Back-Project to Camera-Frame Point Cloud
→ Transform to Robot-Frame Point Cloud
→ Group Points by ROI / Object Region
→ Estimate Object 3D Center + Extents
→ Save Overlay + Disparity + Depth + Point Clouds + Object Localization Report
```

Bài này giúp bạn hiểu một module rất quan trọng trong humanoid perception:

> **Robot không chỉ cần point cloud; robot cần thông tin ở mức object như vị trí 3D và kích thước 3D để chuẩn bị cho manipulation / planning.**

---

# 6. Pipeline perception của bài

```text
Stereo Pair Config
→ Read Stereo Pair Records
→ Read ROI / Object Config
→ Read StereoBM Config
→ Read Stereo Calibration Config
→ Read Camera Intrinsics Config
→ Read Point Sampling Config
→ Read Camera-to-Robot Transform Config
→ Create Stereo Camera Sensor Object
→ For Each Stereo Pair:
    → Load Left / Right Image
    → Compute Disparity Map
    → Convert Disparity to Depth Map
    → Back-Project Valid Pixels to Camera-Frame Points
    → Transform Camera Points to Robot-Frame Points
    → Group Robot Points by ROI / Object Region
    → Estimate Object 3D Center / Extents
    → Save Camera / Robot Point Clouds
    → Save Overlay + Disparity + Depth Outputs
→ Write Object Localization Report
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

Ngoài những thứ của Bài 14, Bài 15 bắt đầu chạm rõ hơn vào:

- **object-level reasoning**
- point grouping theo ROI
- center estimation
- extent estimation
- object statistics trong robot frame

---

# 7.4 CV Python

Python không phải runtime localization chính, nhưng có thể dùng để:
- build ROI/object label config
- build transform / intrinsics / calibration / sampling config
- hỗ trợ tổng hợp report

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
- manifest nhiều stereo pairs

## C++
- vector để lưu nhiều scene result
- runtime nhiều ảnh
- config parsing

## CV
- stereo pair handling
- ROI workflow
- proposal / region thinking

---

# 8.3 Phần mới của Đợt 3

## Python
- thêm config cho object labels / object metadata nếu muốn

## C++
- tổ chức object localization runtime bằng OOP

## CV / 3D Vision
- **robot-frame points**
- **3D center estimation**
- **3D size estimation**
- **object-level spatial summary**

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 15, bạn phải nắm được 10 ý rất quan trọng:

## 1. Point cloud là dữ liệu trung gian, object localization mới là bước gần hành động hơn
Robot thường cần biết **object ở đâu**, không chỉ “đây là đám điểm”.

## 2. Một ROI / object region có thể sinh ra một cụm điểm 3D
Từ cụm điểm đó, bạn có thể ước lượng tâm và kích thước xấp xỉ.

## 3. Tâm 3D có thể ước lượng theo nhiều cách
Ở bài starter này, bạn có thể dùng:
- mean của point cloud
- hoặc center từ min/max extents

## 4. Extents là nền cho 3D bounding box sơ cấp
Ví dụ:
- `min_x`, `max_x`
- `min_y`, `max_y`
- `min_z`, `max_z`

## 5. Size 3D được suy ra từ extents
```text
size_x = max_x - min_x
size_y = max_y - min_y
size_z = max_z - min_z
```

## 6. Robot frame quan trọng hơn camera frame khi muốn thao tác
Vì tay robot, base robot, planner… đều thường làm việc trong robot/world frame.

## 7. Localization 3D có thể noisy nếu disparity/depth chưa tốt
Nên chất lượng:
- stereo matching
- calibration
- ROI selection  
sẽ ảnh hưởng trực tiếp đến localization.

## 8. ROI chưa chắc là object sạch hoàn toàn
Nếu ROI chứa cả nền, center 3D có thể bị lệch.  
Đó là lý do sau này bạn sẽ cần segmentation / filtering tốt hơn.

## 9. Đây là nền cho grasp planning / manipulation pre-processing
Ví dụ:
- ước lượng object center để đưa tay tới
- ước lượng chiều cao vật trên bàn

## 10. Đây là điểm chốt rất đẹp cho Đợt 3
Sau Bài 15, bạn đã đi hết chuỗi:

```text
stereo pair
→ disparity
→ depth
→ point cloud
→ camera-to-robot transform
→ object 3D localization
```

---

# 10. Cấu trúc folder

```text
mini_project_15_robot_3d_object_localization_starter/
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
│     ├─ pair_01_depth_visualization.jpg
│     ├─ pair_01_camera_point_cloud.txt
│     ├─ pair_01_robot_point_cloud.txt
│     ├─ pair_02_left_roi_overlay.jpg
│     ├─ pair_02_disparity_visualization.jpg
│     ├─ pair_02_depth_visualization.jpg
│     ├─ pair_02_camera_point_cloud.txt
│     ├─ pair_02_robot_point_cloud.txt
│     └─ object_localization_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ roi_config.txt
│  ├─ stereo_bm_config.txt
│  ├─ stereo_calibration_config.txt
│  ├─ camera_intrinsics_config.txt
│  ├─ point_sampling_config.txt
│  └─ camera_to_robot_transform.txt
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
   │  ├─ StereoCalibrationConfig.hpp
   │  ├─ CameraIntrinsics.hpp
   │  ├─ Transform3D.hpp
   │  ├─ Point3D.hpp
   │  ├─ RobotFramePoint3D.hpp
   │  ├─ LocalizedObject3D.hpp
   │  ├─ ObjectLocalizationSceneResult.hpp
   │  ├─ BaseObjectLocalizer3D.hpp
   │  ├─ StereoObjectLocalizer3D.hpp
   │  └─ ObjectLocalizationReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoObjectLocalizer3D.cpp
      └─ ObjectLocalizationReportWriter.cpp
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
stereo_calibration_config_path
camera_intrinsics_config_path
point_sampling_config_path
camera_to_robot_transform_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in đường dẫn config

---

# 11.2 Python — `ObjectLocalizationConfigBuilder`

**File:**

```text
python/tools/config_builder.py
```

Tạo class con:

```python
class ObjectLocalizationConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính cần có

```python
stereo_pairs
roi_regions
stereo_bm_config
stereo_calibration_config
camera_intrinsics_config
point_sampling_config
camera_to_robot_transform
```

> Nếu muốn, bạn có thể thêm:
```python
object_labels
```
để map ROI name → object label.

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- giống Bài 14

### `add_roi_region(roi_name, x, y, width, height)`
- giống Bài 14

### `set_stereo_bm_config(...)`
- giống Bài 14

### `set_stereo_calibration_config(...)`
- giống Bài 14

### `set_camera_intrinsics_config(fx, fy, cx, cy)`
- giống Bài 14

### `set_point_sampling_config(pixel_step, min_depth_m, max_depth_m)`
- giống Bài 14

### `set_camera_to_robot_transform(
    tx, ty, tz,
    roll_deg, pitch_deg, yaw_deg
)`
- giống Bài 14

### `write_stereo_manifest()`
- giống Bài 14

### `write_roi_config()`
- giống Bài 14

### `write_stereo_bm_config()`
- giống Bài 14

### `write_stereo_calibration_config()`
- giống Bài 14

### `write_camera_intrinsics_config()`
- giống Bài 14

### `write_point_sampling_config()`
- giống Bài 14

### `write_camera_to_robot_transform()`
- giống Bài 14

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 stereo pairs**
- tạo ít nhất **3 ROI / object regions**
- set:
  - stereo BM config
  - stereo calibration config
  - camera intrinsics config
  - point sampling config
  - camera-to-robot transform
- ghi đủ:
  - `config/stereo_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/stereo_bm_config.txt`
  - `config/stereo_calibration_config.txt`
  - `config/camera_intrinsics_config.txt`
  - `config/point_sampling_config.txt`
  - `config/camera_to_robot_transform.txt`

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

- giống Bài 14

---

# 11.6 C++ — `StereoFrameRecord`

**File:**

```text
cpp/include/StereoFrameRecord.hpp
```

- giống Bài 14

---

# 11.7 C++ — `ROIConfig`

**File:**

```text
cpp/include/ROIConfig.hpp
```

- giống Bài 14

---

# 11.8 C++ — `StereoBMConfig`

**File:**

```text
cpp/include/StereoBMConfig.hpp
```

- giống Bài 14

---

# 11.9 C++ — `StereoCalibrationConfig`

**File:**

```text
cpp/include/StereoCalibrationConfig.hpp
```

- giống Bài 14

---

# 11.10 C++ — `CameraIntrinsics`

**File:**

```text
cpp/include/CameraIntrinsics.hpp
```

- giống Bài 14

---

# 11.11 C++ — `Transform3D`

**File:**

```text
cpp/include/Transform3D.hpp
```

- giống Bài 14

---

# 11.12 C++ — `Point3D`

**File:**

```text
cpp/include/Point3D.hpp
```

- giống Bài 14

---

# 11.13 C++ — `RobotFramePoint3D`

**File:**

```text
cpp/include/RobotFramePoint3D.hpp
```

- giống Bài 14

---

# 11.14 C++ — `LocalizedObject3D`

**File:**

```text
cpp/include/LocalizedObject3D.hpp
```

Tạo struct:

```cpp
struct LocalizedObject3D
```

## Thuộc tính cần có

```cpp
std::string pair_name;
std::string roi_name;

int point_count;
bool is_valid;

double center_x;
double center_y;
double center_z;

double min_x;
double max_x;
double min_y;
double max_y;
double min_z;
double max_z;

double size_x;
double size_y;
double size_z;
```

## Ý nghĩa
Struct này biểu diễn **một object 3D đã được localize trong robot frame**.

---

# 11.15 C++ — `ObjectLocalizationSceneResult`

**File:**

```text
cpp/include/ObjectLocalizationSceneResult.hpp
```

Tạo struct:

```cpp
struct ObjectLocalizationSceneResult
```

## Thuộc tính cần có

```cpp
std::string pair_name;

std::string left_image_path;
std::string right_image_path;

std::string left_overlay_output_path;
std::string disparity_output_path;
std::string depth_output_path;
std::string camera_point_cloud_output_path;
std::string robot_point_cloud_output_path;

std::string sensor_name;
int sensor_id;

int image_width;
int image_height;

int object_count;
int total_point_count;
bool is_valid;

std::vector<LocalizedObject3D> localized_objects;
```

---

# 11.16 C++ — `BaseObjectLocalizer3D`

**File:**

```text
cpp/include/BaseObjectLocalizer3D.hpp
```

Tạo class trừu tượng:

```cpp
class BaseObjectLocalizer3D
```

## Hàm cần có

```cpp
virtual void load_stereo_manifest(const std::string& path) = 0;
virtual void load_roi_config(const std::string& path) = 0;
virtual void load_stereo_bm_config(const std::string& path) = 0;
virtual void load_stereo_calibration_config(const std::string& path) = 0;
virtual void load_camera_intrinsics_config(const std::string& path) = 0;
virtual void load_point_sampling_config(const std::string& path) = 0;
virtual void load_camera_to_robot_transform(const std::string& path) = 0;
virtual void run_object_localization() = 0;
virtual ~BaseObjectLocalizer3D() = default;
```

---

# 11.17 C++ — `StereoObjectLocalizer3D`

**File:**

```text
cpp/include/StereoObjectLocalizer3D.hpp
cpp/src/StereoObjectLocalizer3D.cpp
```

Tạo class kế thừa:

```cpp
class StereoObjectLocalizer3D : public BaseObjectLocalizer3D
```

## Thuộc tính cần có

```cpp
private:
    std::vector<StereoFrameRecord> stereo_records;
    std::vector<ROIConfig> roi_configs;
    StereoBMConfig stereo_bm_config;
    StereoCalibrationConfig stereo_calibration_config;
    CameraIntrinsics camera_intrinsics;
    Transform3D camera_to_robot_transform;

    int pixel_step;
    double min_depth_m;
    double max_depth_m;

    std::vector<ObjectLocalizationSceneResult> scene_results;
```

---

## Hàm cần có

### Load / Read config
- `read_stereo_manifest(...)`
- `read_roi_config(...)`
- `read_stereo_bm_config(...)`
- `read_stereo_calibration_config(...)`
- `read_camera_intrinsics_config(...)`
- `read_point_sampling_config(...)`
- `read_camera_to_robot_transform(...)`
- các hàm `load_...(...) override`

---

## Stereo / depth / point cloud part

### `cv::Rect clamp_roi_to_image(const ROIConfig& roi_cfg, const cv::Mat& image) const;`

### `cv::Mat compute_disparity_map(const cv::Mat& left_bgr, const cv::Mat& right_bgr) const;`
- giống Bài 14

### `cv::Mat compute_depth_map(const cv::Mat& disparity_map) const;`
- giống Bài 14

### `cv::Mat build_disparity_visualization(const cv::Mat& disparity_map) const;`
- giống Bài 14

### `cv::Mat build_depth_visualization(const cv::Mat& depth_map) const;`
- giống Bài 14

### `Point3D back_project_pixel_to_point(
    int u,
    int v,
    float depth_value,
    const std::string& roi_name
) const;`
- giống Bài 14

### `std::vector<Point3D> build_roi_camera_points(
    const std::string& roi_name,
    const cv::Rect& roi_rect,
    const cv::Mat& depth_map
) const;`
- giống Bài 14

### `RobotFramePoint3D transform_camera_point_to_robot_point(
    const Point3D& camera_point
) const;`
- giống Bài 14

### `std::vector<RobotFramePoint3D> transform_camera_points_to_robot_points(
    const std::vector<Point3D>& camera_points
) const;`
- giống Bài 14

---

## Object localization part

### `LocalizedObject3D localize_object_from_robot_points(
    const std::string& pair_name,
    const std::string& roi_name,
    const std::vector<RobotFramePoint3D>& robot_points
) const;`

## Hành vi tổng quát
1. nếu không có point hợp lệ → object invalid
2. tính:
   - `min_x`, `max_x`
   - `min_y`, `max_y`
   - `min_z`, `max_z`
3. tính center:
   - cách 1: mean toàn bộ points
   - hoặc cách 2: center từ extents
4. tính size:
   ```text
   size_x = max_x - min_x
   size_y = max_y - min_y
   size_z = max_z - min_z
   ```
5. build `LocalizedObject3D`

---

### `void draw_object_overlay(
    cv::Mat& left_image,
    const std::vector<LocalizedObject3D>& objects
) const;`

## Hành vi
- vẽ ROI lên ảnh trái
- ghi:
  - tên ROI
  - center_z
  - size_x / size_y / size_z hoặc một phần tóm tắt

---

### `void save_camera_points_as_txt(
    const std::string& output_path,
    const std::vector<Point3D>& camera_points
) const;`
- giống Bài 14

### `void save_robot_points_as_txt(
    const std::string& output_path,
    const std::vector<RobotFramePoint3D>& robot_points
) const;`
- giống Bài 14

---

### `ObjectLocalizationSceneResult process_single_stereo_pair(
    const StereoFrameRecord& record
);`

## Hành vi tổng quát
1. đọc ảnh trái / phải
2. compute disparity
3. compute depth
4. loop qua ROI:
   - build camera-frame points
   - transform sang robot-frame points
   - localize object 3D từ robot points
5. gộp điểm camera / robot toàn scene
6. lưu:
   - camera point cloud txt
   - robot point cloud txt
7. build disparity / depth visualization
8. draw object overlay
9. build `ObjectLocalizationSceneResult`

---

### `void run_object_localization() override;`
- loop qua stereo pairs

### Getter

```cpp
const std::vector<ObjectLocalizationSceneResult>& get_scene_results() const;
```

---

# 11.18 C++ — `ObjectLocalizationReportWriter`

**File:**

```text
cpp/include/ObjectLocalizationReportWriter.hpp
cpp/src/ObjectLocalizationReportWriter.cpp
```

Tạo class:

```cpp
class ObjectLocalizationReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<ObjectLocalizationSceneResult>& scene_results
);`

## Format gợi ý

```text
[3D Object Localization Scene]
Pair Name: pair_01
Left Image: assets/stereo_pairs/pair_01_left.jpg
Right Image: assets/stereo_pairs/pair_01_right.jpg
Left Overlay Output: assets/outputs/pair_01_left_roi_overlay.jpg
Disparity Output: assets/outputs/pair_01_disparity_visualization.jpg
Depth Output: assets/outputs/pair_01_depth_visualization.jpg
Camera Point Cloud Output: assets/outputs/pair_01_camera_point_cloud.txt
Robot Point Cloud Output: assets/outputs/pair_01_robot_point_cloud.txt
Sensor: head_stereo_camera
Object Count: 3
Total Point Count: 1840
Valid: true

  [Localized Object]
  ROI Name: box_region
  Point Count: 720
  Center: (0.22, 0.04, 0.92)
  Min X: 0.15
  Max X: 0.29
  Min Y: -0.03
  Max Y: 0.10
  Min Z: 0.84
  Max Z: 1.02
  Size X: 0.14
  Size Y: 0.13
  Size Z: 0.18
  Valid: true

----------------------------------------
```

---

# 11.19 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- in thông tin sensor
- tạo `StereoObjectLocalizer3D`
- load:
  - `config/stereo_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/stereo_bm_config.txt`
  - `config/stereo_calibration_config.txt`
  - `config/camera_intrinsics_config.txt`
  - `config/point_sampling_config.txt`
  - `config/camera_to_robot_transform.txt`
- chạy `run_object_localization()`
- tạo `ObjectLocalizationReportWriter`
- ghi report ra:
  - `assets/outputs/object_localization_report.txt`

## Pipeline `main.cpp`

```text
Create StereoCameraSensor
→ Load Stereo Pair Manifest
→ Load ROI Config
→ Load StereoBM Config
→ Load Stereo Calibration Config
→ Load Camera Intrinsics Config
→ Load Point Sampling Config
→ Load Camera-to-Robot Transform Config
→ Run 3D Object Localization Pipeline
→ Save Overlay + Disparity + Depth + Camera/Robot Point Clouds
→ Write Object Localization Report
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
- depth map computation
- back projection sang 3D
- camera-frame point cloud
- robot-frame point cloud
- rigid transform camera → robot
- localized object 3D result
- 3D center estimation
- 3D size estimation
- report scene-level

---

# 13. Output mong muốn

## File config
```text
config/stereo_pair_manifest.txt
config/roi_config.txt
config/stereo_bm_config.txt
config/stereo_calibration_config.txt
config/camera_intrinsics_config.txt
config/point_sampling_config.txt
config/camera_to_robot_transform.txt
```

## Ảnh / point cloud output
```text
assets/outputs/pair_01_left_roi_overlay.jpg
assets/outputs/pair_01_disparity_visualization.jpg
assets/outputs/pair_01_depth_visualization.jpg
assets/outputs/pair_01_camera_point_cloud.txt
assets/outputs/pair_01_robot_point_cloud.txt
```

## File report
```text
assets/outputs/object_localization_report.txt
```

---

## Ví dụ `object_localization_report.txt`

```text
[3D Object Localization Scene]
Pair Name: pair_01
Left Image: assets/stereo_pairs/pair_01_left.jpg
Right Image: assets/stereo_pairs/pair_01_right.jpg
Left Overlay Output: assets/outputs/pair_01_left_roi_overlay.jpg
Disparity Output: assets/outputs/pair_01_disparity_visualization.jpg
Depth Output: assets/outputs/pair_01_depth_visualization.jpg
Camera Point Cloud Output: assets/outputs/pair_01_camera_point_cloud.txt
Robot Point Cloud Output: assets/outputs/pair_01_robot_point_cloud.txt
Sensor: head_stereo_camera
Object Count: 3
Total Point Count: 1840
Valid: true

  [Localized Object]
  ROI Name: box_region
  Point Count: 720
  Center: (0.22, 0.04, 0.92)
  Min X: 0.15
  Max X: 0.29
  Min Y: -0.03
  Max Y: 0.10
  Min Z: 0.84
  Max Z: 1.02
  Size X: 0.14
  Size Y: 0.13
  Size Z: 0.18
  Valid: true
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo manifest stereo pairs
- tạo ROI / object region config
- tạo stereo BM config
- tạo calibration / intrinsics / sampling / transform config

Tức là Python làm phần:

```text
3D Object Localization Pipeline Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- compute disparity
- compute depth
- back-project thành camera-frame points
- transform sang robot-frame points
- localize object 3D từ robot-frame points
- lưu point clouds + object report

Tức là C++ làm phần:

```text
Stereo 3D Object Localization Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **biến stereo pair thành disparity**
- **biến disparity thành depth**
- **biến depth thành point cloud**
- **đưa point cloud sang robot frame**
- **ước lượng object center / size trong không gian 3D**

Tức là CV làm phần:

```text
Stereo Pair → Disparity → Depth → Point Cloud → Robot Frame → 3D Object Localization
```

---

# 15. Checklist hoàn thành

- [ ] Tạo đúng cấu trúc folder
- [ ] Python tạo được `stereo_pair_manifest.txt`
- [ ] Python tạo được `roi_config.txt`
- [ ] Python tạo được `stereo_bm_config.txt`
- [ ] Python tạo được `stereo_calibration_config.txt`
- [ ] Python tạo được `camera_intrinsics_config.txt`
- [ ] Python tạo được `point_sampling_config.txt`
- [ ] Python tạo được `camera_to_robot_transform.txt`
- [ ] Python có class cha / class con
- [ ] Python có list / dict / string / function / loop / if else
- [ ] C++ có `BaseSensor`
- [ ] C++ có `StereoCameraSensor`
- [ ] C++ có `StereoFrameRecord`
- [ ] C++ có `ROIConfig`
- [ ] C++ có `StereoBMConfig`
- [ ] C++ có `StereoCalibrationConfig`
- [ ] C++ có `CameraIntrinsics`
- [ ] C++ có `Transform3D`
- [ ] C++ có `Point3D`
- [ ] C++ có `RobotFramePoint3D`
- [ ] C++ có `LocalizedObject3D`
- [ ] C++ có `ObjectLocalizationSceneResult`
- [ ] C++ có `BaseObjectLocalizer3D`
- [ ] C++ có `StereoObjectLocalizer3D`
- [ ] C++ load được stereo manifest
- [ ] C++ load được ROI config
- [ ] C++ load được stereo BM config
- [ ] C++ load được stereo calibration config
- [ ] C++ load được camera intrinsics config
- [ ] C++ load được point sampling config
- [ ] C++ load được camera-to-robot transform config
- [ ] C++ compute được disparity map
- [ ] C++ compute được depth map
- [ ] C++ back-project được camera points
- [ ] C++ transform được sang robot points
- [ ] C++ localize được object 3D
- [ ] C++ tính được center 3D
- [ ] C++ tính được size 3D
- [ ] C++ build được report

---

# 16. Gợi ý mở rộng

## 1. Dùng object center để chọn target grasp
Bạn có thể chọn object gần robot nhất hoặc object ở ROI quan trọng nhất.

## 2. Thêm filtering point cloud trước khi localize
Ví dụ:
- bỏ outlier
- chỉ lấy depth trong khoảng gần object

## 3. Dùng proposal box / detector output thật thay cho ROI cố định
Khi đó project sẽ gần perception pipeline thật hơn.

## 4. Chuẩn bị cho Đợt 4
Sau Bài 15, bạn đã hoàn thành Đợt 3 với chuỗi stereo-3D rất đẹp.  
Bước hợp lý cho **Đợt 4** có thể là đi sang:

```text
Tracking / Temporal Perception / Object Association
```

hoặc nếu bám sát humanoid AI perception 3D hơn thì có thể là:

```text
multi-object scene reasoning
pose estimation
tabletop manipulation perception
```

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 15**, bạn sẽ chốt xong **Đợt 3** theo đúng trục stereo perception → 3D reasoning:

- **Bài 11**: disparity map + ROI disparity analysis
- **Bài 12**: depth map + ROI depth analysis
- **Bài 13**: point cloud trong camera frame
- **Bài 14**: camera-frame point cloud → robot-frame point cloud
- **Bài 15**: **robot-frame point cloud → 3D object localization**

Tức là bạn đã đi hết chuỗi:

```text
stereo pair
→ disparity
→ depth
→ point cloud
→ camera-to-robot transform
→ object 3D localization
```

Đây là một điểm dừng rất đẹp trước khi sang **Đợt 4**, vì lúc này bạn không còn chỉ xử lý ảnh nữa mà đã bắt đầu **biến dữ liệu stereo thành thông tin object 3D có thể dùng cho robot**.
