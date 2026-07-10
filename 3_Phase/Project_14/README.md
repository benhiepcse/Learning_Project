# 🤖 Bài 14: Robot Camera-to-Robot 3D Transform Starter — Chuyển Point Cloud từ Camera Frame sang Robot/Base Frame cho Humanoid Robot AI Perception

> Mini Project số 14 trong **Đợt 3 — Bài 11 → Bài 15**  
> **Bài 14 tiếp tục kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3** theo đúng rule bạn đã chốt.  
> Nếu **Bài 13** đã giúp robot đi từ **depth map → back-project → point cloud trong camera frame**, thì **Bài 14** sẽ đẩy tiếp sang bước cực kỳ quan trọng trong perception của humanoid robot:
>
> **chuyển các điểm 3D từ camera frame sang robot/base frame để robot có thể reason trong hệ tọa độ của chính nó.**

---

# 📌 Mục lục

- [1. Bài 14 lấy gì từ Đợt 3](#1-bài-14-lấy-gì-từ-đợt-3)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 14 nằm ở đâu trong roadmap](#3-bài-14-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 14 là bước tiếp theo hợp lý sau Bài 13](#4-vì-sao-bài-14-là-bước-tiếp-theo-hợp-lý-sau-bài-13)
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
  - [11.2 Python — CameraToRobotConfigBuilder](#112-python--cameratorobotconfigbuilder)
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
  - [11.14 C++ — ROITransformStats](#1114-c--roitransformstats)
  - [11.15 C++ — CameraToRobotSceneResult](#1115-c--cameratorobotsceneresult)
  - [11.16 C++ — BaseCameraToRobotTransformer](#1116-c--basecameratorobottransformer)
  - [11.17 C++ — StereoCameraToRobotTransformer](#1117-c--stereocameratorobottransformer)
  - [11.18 C++ — CameraToRobotReportWriter](#1118-c--cameratorobotreportwriter)
  - [11.19 C++ — main.cpp](#1119-c--maincpp)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 14 lấy gì từ Đợt 3

Sau Bài 13, trục Đợt 3 của bạn đang là:

- **Stereo Camera**
- **Disparity**
- **Depth**
- **Point Cloud**
- và tiếp theo rất tự nhiên là:
  - **Camera Coordinate → Robot Coordinate**
  - **3D Coordinate Recovery**
  - **robot-centric spatial reasoning**

Vì vậy **Bài 14** lấy đúng phần tiếp theo:

## Phần mới của Đợt 3 mà Bài 14 dùng
### Computer Vision / 3D Geometry
- **camera frame**
- **robot/base frame**
- **rigid transform 3D**
- dùng:
  - rotation
  - translation
- chuyển:
  ```text
  point_camera → point_robot
  ```

### Python
- tiếp tục làm:
  - config builder
  - transform config
  - report helper

### C++
- vẫn dùng:
  - OOP
  - vector
  - struct config / result
  - runtime pipeline

> Bài 14 **chưa bắt buộc dùng Eigen / ROS TF2**.  
> Mục tiêu là hiểu thật chắc tư duy:
>
> ```text
> point 3D trong camera frame
> → áp transform camera-to-robot
> → point 3D trong robot frame
> ```

---

# 2. Mô tả

Ở **Bài 13**, bạn đã có một module có thể:

- đọc stereo pair
- tính disparity map
- tính depth map
- back-project thành point cloud trong **camera frame**
- tính thống kê 3D theo ROI
- lưu point cloud + report

Bài 14 sẽ **đi tiếp từ point cloud camera frame sang robot/base frame**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc **cặp ảnh stereo**
- tính disparity map
- tính depth map
- back-project thành điểm 3D trong **camera frame**
- đọc **camera-to-robot transform config**
- biến từng điểm 3D sang **robot/base frame**
- tính thống kê 3D theo ROI nhưng **trong robot frame**
- lưu:
  - left ROI overlay
  - disparity visualization
  - depth visualization
  - point cloud camera frame
  - point cloud robot frame
  - report

Ví dụ robot nhìn một chiếc hộp đặt trước mặt:

- trong camera frame, hộp có thể ở `(Xc, Yc, Zc)`
- sau transform camera → robot, hộp sẽ có tọa độ `(Xr, Yr, Zr)` trong base frame

Lúc này robot mới bắt đầu reason kiểu:

```text
vật ở phía trước base bao nhiêu mét
vật lệch trái / phải bao nhiêu
vật có nằm trong vùng với tới của tay robot hay không
```

<p align="center">
  <img src="images/project_14.png" width="800">
</p>

---

# 3. Bài 14 nằm ở đâu trong roadmap

## Quy ước hiện tại
- **Đợt 1 = Bài 1 → Bài 5**
- **Đợt 2 = Bài 6 → Bài 10**
- **Đợt 3 = Bài 11 → Bài 15**
- **Đợt 4 = Bài 16 → Bài 20**

Vì vậy:

## **Bài 14 = bài thứ tư của Đợt 3**
và phải **kết hợp lại kiến thức của Đợt 1 + Đợt 2 + Đợt 3**.

---

# 4. Vì sao Bài 14 là bước tiếp theo hợp lý sau Bài 13

## Bài 12 cho bạn:
- depth map

## Bài 13 cho bạn:
- point cloud trong **camera frame**

## Bài 14 nâng thêm một nấc:
- point cloud không còn chỉ ở camera frame nữa
- mà được đưa sang **robot/base frame**

Đây là bước cực kỳ quan trọng vì robot thường không hành động trong camera frame, mà trong:
- **base frame**
- **torso frame**
- **world frame**

Nên chuỗi suy nghĩ đúng sẽ là:

```text
stereo pair
→ disparity
→ depth
→ point cloud in camera frame
→ transform to robot frame
→ robot-centric reasoning
```

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Pair + ROI Config + StereoBM Config + Calibration Config + Intrinsics + Camera-to-Robot Transform
→ Load Left / Right Image Pair
→ Compute Disparity Map
→ Convert Disparity to Depth
→ Back-Project to Camera-Frame Point Cloud
→ Transform Camera Points to Robot/Base Frame
→ Build ROI-Level Robot-Frame Point Sets
→ Compute ROI 3D Statistics in Robot Frame
→ Save Overlay + Disparity + Depth + Camera Point Cloud + Robot Point Cloud + Report
```

Bài này giúp bạn hiểu một module cực quan trọng trong humanoid perception:

> **Perception chỉ thật sự hữu ích cho control/planning khi dữ liệu 3D được đưa về hệ tọa độ robot hoặc world.**

---

# 6. Pipeline perception của bài

```text
Stereo Pair Config
→ Read Stereo Pair Records
→ Read ROI Config
→ Read StereoBM Config
→ Read Stereo Calibration Config
→ Read Camera Intrinsics Config
→ Read Camera-to-Robot Transform Config
→ Create Stereo Camera Sensor Object
→ For Each Stereo Pair:
    → Load Left / Right Image
    → Compute Disparity Map
    → Convert Disparity to Depth Map
    → Back-Project Valid Pixels to Camera-Frame 3D Points
    → Transform Camera-Frame Points to Robot-Frame Points
    → Group Points by ROI
    → Compute ROI Robot-Frame Statistics
    → Save Camera / Robot Point Cloud Files
    → Save Overlay + Disparity + Depth Outputs
→ Write Camera-to-Robot Report
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

Ngoài những thứ của Bài 13, Bài 14 bắt đầu chạm rõ hơn vào:

- **camera frame vs robot frame**
- rigid transform 3D
- transform point cloud
- reasoning theo tọa độ robot

---

# 7.4 CV Python

Python không phải runtime transform chính, nhưng có thể dùng để:
- build transform config
- build intrinsics / calibration / sampling config
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
- thêm config cho camera-to-robot transform

## C++
- tổ chức point transform runtime bằng OOP

## CV / 3D Vision
- **point 3D camera frame**
- **rigid transform**
- **point 3D robot frame**

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 14, bạn phải nắm được 10 ý rất quan trọng:

## 1. Point cloud trong camera frame chưa đủ cho robot hành động
Robot thường cần biết vật nằm ở đâu **so với robot**, không chỉ so với camera.

## 2. Camera-to-robot transform là bước nối perception với planning/control
Nếu robot muốn với tay, tránh vật cản, bước đi…, nó cần tọa độ trong frame phù hợp.

## 3. Rigid transform 3D thường có dạng:
```text
p_robot = R * p_camera + t
```

## 4. `R` là rotation, `t` là translation
- `R` đổi hướng trục
- `t` dời gốc tọa độ

## 5. Một điểm `(Xc, Yc, Zc)` trong camera frame có thể trở thành `(Xr, Yr, Zr)` rất khác trong robot frame
Đó là chuyện bình thường vì gốc và hướng trục khác nhau.

## 6. ROI 3D statistics trong robot frame hữu ích hơn cho thao tác robot
Ví dụ:
- khoảng cách vật tới base
- độ lệch trái/phải so với robot
- độ cao vật so với base

## 7. Camera có thể đặt trên đầu robot, ngực robot, tay robot
Mỗi vị trí sẽ có transform camera → base khác nhau.

## 8. Camera intrinsics và camera extrinsics là hai thứ khác nhau
- **intrinsics**: pixel ↔ camera ray / geometry nội tại camera
- **extrinsics / transform**: camera frame ↔ robot/world frame

## 9. Đây là nền cho TF2 / ROS transform sau này
Bài này làm tay bằng C++ thường trước, để sau sang ROS2 TF2 bạn hiểu bản chất.

## 10. Đây là cầu nối trực tiếp sang object 3D localization / robot interaction
Sau Bài 14, bạn sẽ thuận lợi để sang:
- object 3D center estimation
- tabletop scene understanding
- reachability / manipulation pre-processing

---

# 10. Cấu trúc folder

```text
mini_project_14_robot_camera_to_robot_3d_transform_starter/
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
│     └─ camera_to_robot_report.txt
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
   │  ├─ ROITransformStats.hpp
   │  ├─ CameraToRobotSceneResult.hpp
   │  ├─ BaseCameraToRobotTransformer.hpp
   │  ├─ StereoCameraToRobotTransformer.hpp
   │  └─ CameraToRobotReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoCameraToRobotTransformer.cpp
      └─ CameraToRobotReportWriter.cpp
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

# 11.2 Python — `CameraToRobotConfigBuilder`

**File:**

```text
python/tools/config_builder.py
```

Tạo class con:

```python
class CameraToRobotConfigBuilder(BaseConfigBuilder):
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

---

## `camera_to_robot_transform`
Là dict, ví dụ:

```python
{
    "tx": 0.05,
    "ty": 0.00,
    "tz": 0.20,
    "roll_deg": 0.0,
    "pitch_deg": 10.0,
    "yaw_deg": 0.0
}
```

> Bạn có thể lưu transform dưới dạng **translation + Euler angles** để project dễ hiểu ở giai đoạn này.

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- giống Bài 13

### `add_roi_region(roi_name, x, y, width, height)`
- giống Bài 13

### `set_stereo_bm_config(...)`
- giống Bài 13

### `set_stereo_calibration_config(...)`
- giống Bài 13

### `set_camera_intrinsics_config(fx, fy, cx, cy)`
- giống Bài 13

### `set_point_sampling_config(pixel_step, min_depth_m, max_depth_m)`
- giống Bài 13

### `set_camera_to_robot_transform(
    tx, ty, tz,
    roll_deg, pitch_deg, yaw_deg
)`
**Hành vi**
- lưu transform camera → robot
- kiểm tra:
  - các giá trị là số hợp lệ
  - `pixel_step`, depth range đã được set từ trước nếu bạn muốn enforce pipeline

### `write_stereo_manifest()`
- giống Bài 13

### `write_roi_config()`
- giống Bài 13

### `write_stereo_bm_config()`
- giống Bài 13

### `write_stereo_calibration_config()`
- giống Bài 13

### `write_camera_intrinsics_config()`
- giống Bài 13

### `write_point_sampling_config()`
- giống Bài 13

### `write_camera_to_robot_transform()`
**Format gợi ý**
```text
tx=0.05
ty=0.00
tz=0.20
roll_deg=0.0
pitch_deg=10.0
yaw_deg=0.0
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 stereo pairs**
- tạo ít nhất **3 ROI**
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

- giống Bài 13

---

# 11.6 C++ — `StereoFrameRecord`

**File:**

```text
cpp/include/StereoFrameRecord.hpp
```

- giống Bài 13

---

# 11.7 C++ — `ROIConfig`

**File:**

```text
cpp/include/ROIConfig.hpp
```

- giống Bài 13

---

# 11.8 C++ — `StereoBMConfig`

**File:**

```text
cpp/include/StereoBMConfig.hpp
```

- giống Bài 13

---

# 11.9 C++ — `StereoCalibrationConfig`

**File:**

```text
cpp/include/StereoCalibrationConfig.hpp
```

- giống Bài 13

---

# 11.10 C++ — `CameraIntrinsics`

**File:**

```text
cpp/include/CameraIntrinsics.hpp
```

- giống Bài 13

---

# 11.11 C++ — `Transform3D`

**File:**

```text
cpp/include/Transform3D.hpp
```

Tạo struct hoặc class:

```cpp
struct Transform3D
```

## Thuộc tính cần có

```cpp
double tx;
double ty;
double tz;

double roll_deg;
double pitch_deg;
double yaw_deg;
```

## Hàm gợi ý nên có
### `cv::Matx33f build_rotation_matrix() const;`
- từ roll / pitch / yaw tạo rotation matrix 3x3

> Nếu bạn chưa muốn dùng `cv::Matx33f`, có thể tự tạo struct ma trận 3x3 đơn giản.  
> Nhưng dùng `cv::Matx33f` sẽ gọn hơn.

---

# 11.12 C++ — `Point3D`

**File:**

```text
cpp/include/Point3D.hpp
```

- giống Bài 13  
- biểu diễn **điểm trong camera frame**

---

# 11.13 C++ — `RobotFramePoint3D`

**File:**

```text
cpp/include/RobotFramePoint3D.hpp
```

Tạo struct:

```cpp
struct RobotFramePoint3D
```

## Thuộc tính cần có

```cpp
float x;
float y;
float z;

int u;
int v;

bool is_valid;
std::string roi_name;
```

### Giải thích
- `(x, y, z)` là tọa độ **trong robot/base frame**
- `(u, v)` là pixel gốc trên ảnh trái
- `roi_name` để biết điểm thuộc ROI nào

---

# 11.14 C++ — `ROITransformStats`

**File:**

```text
cpp/include/ROITransformStats.hpp
```

Tạo struct:

```cpp
struct ROITransformStats
```

## Thuộc tính cần có

```cpp
std::string pair_name;
std::string roi_name;

int point_count;

double min_x;
double max_x;
double min_y;
double max_y;
double min_z;
double max_z;
double mean_z;

bool is_valid;
```

> Các thống kê này là **trong robot frame**.

---

# 11.15 C++ — `CameraToRobotSceneResult`

**File:**

```text
cpp/include/CameraToRobotSceneResult.hpp
```

Tạo struct:

```cpp
struct CameraToRobotSceneResult
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

int roi_count;
int total_point_count;
bool is_valid;

std::vector<ROITransformStats> roi_stats;
```

---

# 11.16 C++ — `BaseCameraToRobotTransformer`

**File:**

```text
cpp/include/BaseCameraToRobotTransformer.hpp
```

Tạo class trừu tượng:

```cpp
class BaseCameraToRobotTransformer
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
virtual void run_camera_to_robot_transform() = 0;
virtual ~BaseCameraToRobotTransformer() = default;
```

---

# 11.17 C++ — `StereoCameraToRobotTransformer`

**File:**

```text
cpp/include/StereoCameraToRobotTransformer.hpp
cpp/src/StereoCameraToRobotTransformer.cpp
```

Tạo class kế thừa:

```cpp
class StereoCameraToRobotTransformer : public BaseCameraToRobotTransformer
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

    std::vector<CameraToRobotSceneResult> scene_results;
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
- giống Bài 13

### `cv::Mat compute_depth_map(const cv::Mat& disparity_map) const;`
- giống Bài 13

### `cv::Mat build_disparity_visualization(const cv::Mat& disparity_map) const;`
- giống Bài 13

### `cv::Mat build_depth_visualization(const cv::Mat& depth_map) const;`
- giống Bài 13

### `Point3D back_project_pixel_to_point(
    int u,
    int v,
    float depth_value,
    const std::string& roi_name
) const;`
- giống Bài 13

### `std::vector<Point3D> build_roi_camera_points(
    const std::string& roi_name,
    const cv::Rect& roi_rect,
    const cv::Mat& depth_map
) const;`
- giống Bài 13 nhưng trả ra điểm trong **camera frame**

---

## Transform part

### `RobotFramePoint3D transform_camera_point_to_robot_point(
    const Point3D& camera_point
) const;`

## Hành vi gợi ý
1. build rotation matrix `R` từ roll/pitch/yaw
2. lấy:
   ```text
   p_robot = R * p_camera + t
   ```
3. trả ra `RobotFramePoint3D`

---

### `std::vector<RobotFramePoint3D> transform_camera_points_to_robot_points(
    const std::vector<Point3D>& camera_points
) const;`

- loop qua toàn bộ camera points
- transform từng điểm

---

### `ROITransformStats analyze_robot_frame_points(
    const std::string& pair_name,
    const std::string& roi_name,
    const std::vector<RobotFramePoint3D>& robot_points
) const;`

## Hành vi tổng quát
Tính:
- `point_count`
- `min_x`, `max_x`
- `min_y`, `max_y`
- `min_z`, `max_z`
- `mean_z`

---

### `void draw_roi_overlay(
    cv::Mat& left_image,
    const std::vector<ROITransformStats>& roi_stats
) const;`

## Hành vi
- vẽ ROI lên ảnh trái
- ghi:
  - tên ROI
  - số lượng điểm
  - mean Z trong robot frame

---

### `void save_camera_points_as_txt(
    const std::string& output_path,
    const std::vector<Point3D>& camera_points
) const;`

**Format gợi ý**
```text
x y z u v roi_name
0.12 -0.03 0.85 210 160 object_region
...
```

### `void save_robot_points_as_txt(
    const std::string& output_path,
    const std::vector<RobotFramePoint3D>& robot_points
) const;`

**Format gợi ý**
```text
x y z u v roi_name
0.20 0.05 0.90 210 160 object_region
...
```

---

### `CameraToRobotSceneResult process_single_stereo_pair(
    const StereoFrameRecord& record
);`

## Hành vi tổng quát
1. đọc ảnh trái / phải
2. compute disparity
3. compute depth
4. loop qua ROI:
   - build camera-frame points
   - transform sang robot-frame points
   - analyze robot-frame stats
5. gộp điểm camera / robot toàn scene
6. lưu:
   - camera point cloud txt
   - robot point cloud txt
7. build disparity / depth visualization
8. vẽ overlay
9. build `CameraToRobotSceneResult`

---

### `void run_camera_to_robot_transform() override;`
- loop qua stereo pairs

### Getter

```cpp
const std::vector<CameraToRobotSceneResult>& get_scene_results() const;
```

---

# 11.18 C++ — `CameraToRobotReportWriter`

**File:**

```text
cpp/include/CameraToRobotReportWriter.hpp
cpp/src/CameraToRobotReportWriter.cpp
```

Tạo class:

```cpp
class CameraToRobotReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<CameraToRobotSceneResult>& scene_results
);`

## Format gợi ý

```text
[Camera to Robot Scene]
Pair Name: pair_01
Left Image: assets/stereo_pairs/pair_01_left.jpg
Right Image: assets/stereo_pairs/pair_01_right.jpg
Left Overlay Output: assets/outputs/pair_01_left_roi_overlay.jpg
Disparity Output: assets/outputs/pair_01_disparity_visualization.jpg
Depth Output: assets/outputs/pair_01_depth_visualization.jpg
Camera Point Cloud Output: assets/outputs/pair_01_camera_point_cloud.txt
Robot Point Cloud Output: assets/outputs/pair_01_robot_point_cloud.txt
Sensor: head_stereo_camera
ROI Count: 3
Total Point Count: 1840
Valid: true

  [ROI Robot Stats]
  ROI Name: object_region
  Point Count: 720
  Min X: 0.15
  Max X: 0.34
  Min Y: -0.10
  Max Y: 0.08
  Min Z: 0.80
  Max Z: 1.20
  Mean Z: 0.94
  Valid: true

----------------------------------------
```

---

# 11.19 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- in thông tin sensor
- tạo `StereoCameraToRobotTransformer`
- load:
  - `config/stereo_pair_manifest.txt`
  - `config/roi_config.txt`
  - `config/stereo_bm_config.txt`
  - `config/stereo_calibration_config.txt`
  - `config/camera_intrinsics_config.txt`
  - `config/point_sampling_config.txt`
  - `config/camera_to_robot_transform.txt`
- chạy `run_camera_to_robot_transform()`
- tạo `CameraToRobotReportWriter`
- ghi report ra:
  - `assets/outputs/camera_to_robot_report.txt`

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
→ Run Camera-to-Robot Transform Pipeline
→ Save Overlay + Disparity + Depth + Camera/Robot Point Clouds
→ Write Camera-to-Robot Report
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
- ROI robot-frame statistics
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
assets/outputs/camera_to_robot_report.txt
```

---

## Ví dụ `camera_to_robot_transform.txt`

```text
tx=0.05
ty=0.00
tz=0.20
roll_deg=0.0
pitch_deg=10.0
yaw_deg=0.0
```

## Ví dụ `pair_01_robot_point_cloud.txt`

```text
x y z u v roi_name
0.20 0.05 0.90 210 160 object_region
0.21 0.05 0.91 212 160 object_region
0.22 0.06 0.92 214 160 object_region
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo manifest stereo pairs
- tạo ROI config
- tạo stereo BM config
- tạo calibration / intrinsics / sampling config
- tạo camera-to-robot transform config

Tức là Python làm phần:

```text
Camera-to-Robot Transform Pipeline Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- compute disparity
- compute depth
- back-project thành camera-frame points
- transform sang robot-frame points
- phân tích ROI trong robot frame
- lưu point clouds + report

Tức là C++ làm phần:

```text
Stereo Camera-to-Robot 3D Transform Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **biến stereo pair thành disparity**
- **biến disparity thành depth**
- **biến depth thành camera-frame point cloud**
- **biến camera-frame point cloud thành robot-frame point cloud**
- **đưa perception về hệ tọa độ robot**

Tức là CV làm phần:

```text
Stereo Pair → Disparity → Depth → Camera Point Cloud → Robot Point Cloud
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
- [ ] C++ có `ROITransformStats`
- [ ] C++ có `CameraToRobotSceneResult`
- [ ] C++ có `BaseCameraToRobotTransformer`
- [ ] C++ có `StereoCameraToRobotTransformer`
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
- [ ] C++ build được ROI robot-frame statistics
- [ ] C++ lưu được camera / robot point cloud txt
- [ ] C++ build được report

---

# 16. Gợi ý mở rộng

## 1. Dùng homogeneous transform 4x4
Thay vì `R * p + t`, bạn có thể đóng gói thành ma trận 4x4 để gần với robotics hơn.

## 2. Thêm world frame
Sau robot/base frame, bạn có thể thêm:
```text
camera → robot → world
```

## 3. Dùng proposal box từ Bài 10 thay cho ROI cố định
Tức là transform point cloud theo từng object proposal.

## 4. Chuẩn bị cho Bài 15
Sau Bài 14, bước hợp lý nhất cho **Bài 15** là:

```text
Robot 3D Object Localization Starter
```

tức là:
- đã có robot-frame point cloud
- giờ gom điểm theo ROI / object
- ước lượng **3D center / 3D bounding stats** cho object

---

# 🚀 Sau bài này bạn sẽ có gì?

Sau khi hoàn thành **Bài 14**, bạn sẽ tiếp tục Đợt 3 theo đúng trục stereo-3D-robot frame:

- **Bài 11**: disparity map + ROI disparity analysis
- **Bài 12**: depth map + ROI depth analysis
- **Bài 13**: point cloud trong camera frame
- **Bài 14**: **camera-frame point cloud → robot-frame point cloud**

Tức là bạn đã đi từ:

```text
stereo pair → disparity → depth → point cloud
```

sang

```text
stereo pair → disparity → depth → camera-frame 3D → robot-frame 3D
```

Đây là nền rất đẹp để sang **Bài 15** làm **Robot 3D Object Localization Starter** — nơi bạn bắt đầu định vị object trong **robot frame** thật sự.
