# 🤖 Bài 18: Robot Pick Candidate Vision Starter — Đánh giá object nào phù hợp để gắp trong robot workspace cho Humanoid Robot AI Perception

> Mini Project số 18 trong **Đợt 4 — Bài 16 → Bài 20**  
> **Bài 18 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.  
> Nếu **Bài 17** đã giúp robot dùng perception để **theo dõi object mục tiêu** và sinh ra follow command, thì **Bài 18** sẽ đi tiếp sang một nhánh rất quan trọng của roadmap Đợt 4:
>
> **Pick and Place Vision** — đánh giá object nào là **pick candidate** cho humanoid robot.

---

# 📌 Mục lục

- [1. Bài 18 lấy gì từ Đợt 4](#1-bài-18-lấy-gì-từ-đợt-4)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 18 nằm ở đâu trong roadmap](#3-bài-18-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 18 là bước tiếp theo hợp lý sau Bài 17](#4-vì-sao-bài-18-là-bước-tiếp-theo-hợp-lý-sau-bài-17)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. Kiến thức Đợt 1 + 2 + 3 + 4 được dùng như thế nào](#8-kiến-thức-đợt-1--2--3--4-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
  - [11.1 Python — BaseConfigBuilder](#111-python--baseconfigbuilder)
  - [11.2 Python — PickCandidateConfigBuilder](#112-python--pickcandidateconfigbuilder)
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
  - [11.15 C++ — PickDecisionConfig](#1115-c--pickdecisionconfig)
  - [11.16 C++ — PickCandidateDecision](#1116-c--pickcandidatedecision)
  - [11.17 C++ — PickCandidateSceneResult](#1117-c--pickcandidatesceneresult)
  - [11.18 C++ — BasePickCandidateEvaluator](#1118-c--basepickcandidateevaluator)
  - [11.19 C++ — StereoPickCandidateEvaluator](#1119-c--stereopickcandidateevaluator)
  - [11.20 C++ — PickCandidateReportWriter](#1120-c--pickcandidatereportwriter)
  - [11.21 C++ — main.cpp](#1121-c--maincpp)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 18 lấy gì từ Đợt 4

Theo roadmap, **Đợt 4** tập trung vào:

- **Robot Perception Pipeline**
- **Object Following**
- **Pick and Place Vision**
- **Table Cleaning Robot**
- **Humanoid Robot Perception**

Nếu:
- **Bài 16** = pipeline supervisor
- **Bài 17** = object following vision

thì **Bài 18** sẽ đi vào nhánh tiếp theo:

## **Pick and Place Vision**

Tức là robot không chỉ biết object ở đâu, mà còn phải đánh giá:

- object này **có nằm trong workspace** không?
- object này **có quá xa / quá gần** không?
- object này **có quá to / quá nhỏ** không?
- object này **có hợp để gắp** không?
- object này **có phải target để pick** không?

## Phần mới của Đợt 4 mà Bài 18 dùng
### Robotics Vision / Manipulation-Oriented Perception
- object localization trong robot frame
- workspace filtering
- size filtering
- distance filtering
- pickability decision

### Python
- config builder cho pick decision
- report helper

### C++
- runtime đánh giá **pick candidate**
- decision logic cho manipulation perception

---

# 2. Mô tả

Ở **Bài 17**, bạn đã có một hệ thống có thể:

- localize object 3D
- chọn target object để follow
- sinh follow command như:
  - `TURN_LEFT`
  - `TURN_RIGHT`
  - `MOVE_FORWARD`
  - `STOP`

Bài 18 sẽ **đổi trọng tâm từ “theo object” sang “đánh giá object để gắp”**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc stereo pair
- đọc ROI / object config
- đọc stereo / depth / transform config
- localize object 3D trong robot frame
- đánh giá từng object theo **pick rules**
- gắn nhãn:
  - `PICK_CANDIDATE`
  - `REJECT_TOO_FAR`
  - `REJECT_OUT_OF_WORKSPACE`
  - `REJECT_TOO_LARGE`
  - `REJECT_INVALID`
- vẽ overlay lên ảnh trái
- ghi report pick-candidate cho từng scene

---

# 3. Bài 18 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**

Vì vậy:

## **Bài 18 = bài thứ ba của Đợt 4**
và phải **kết hợp lại kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.

---

# 4. Vì sao Bài 18 là bước tiếp theo hợp lý sau Bài 17

## Bài 17 cho bạn:
- target object localization
- follow decision logic

## Bài 18 thêm:
- manipulation-oriented perception
- pick candidate evaluation

Tức là chuyển từ:

```text
“robot nên di chuyển thế nào để theo object”
```

sang:

```text
“object nào đủ điều kiện để robot đưa tay tới và gắp”
```

Đây là bước rất hợp lý vì trong humanoid robot, **follow object** và **pick object** là hai nhánh hành vi perception quan trọng nhưng khác nhau.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Pair + ROI/Object Config + StereoBM Config + Calibration Config + Intrinsics + Camera-to-Robot Transform + Pick Decision Config
→ Load Left / Right Image Pair
→ Compute Disparity Map
→ Convert Disparity to Depth
→ Back-Project to Camera-Frame Point Cloud
→ Transform to Robot-Frame Point Cloud
→ Localize Object 3D
→ Evaluate Workspace / Distance / Size Constraints
→ Decide Pick Candidate Label
→ Save Overlay + Disparity + Depth + Pick Report
```

Bài này giúp bạn hiểu một module rất quan trọng trong humanoid manipulation perception:

> **Perception cho thao tác không chỉ là “thấy object”, mà là “đánh giá object có thể thao tác được hay không”.**

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
→ Read Pick Decision Config
→ Create Stereo Camera Sensor Object
→ For Each Stereo Pair:
    → Load Left / Right Image
    → Compute Disparity Map
    → Convert Disparity to Depth Map
    → Back-Project Valid Pixels to Camera-Frame Points
    → Transform Camera Points to Robot-Frame Points
    → Group Points by ROI / Object Region
    → Localize Object 3D
    → Evaluate Pick Candidate Decision
    → Save Overlay + Disparity + Depth + Robot Point Cloud + Pick Report
```

---

# 7. Kiến thức cần

## Python
- class / inheritance
- list / dict
- string
- file handling
- config builder
- report helper

## C++
- class / inheritance
- abstract class
- virtual function
- vector
- enum / decision label
- struct
- function tách nhỏ
- header / source structure

## Computer Vision
- stereo pair
- disparity
- depth
- point cloud
- camera-to-robot transform
- object localization
- pick candidate decision logic

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 được dùng như thế nào

# 8.1 Đợt 1
- Python / C++ OOP cơ bản
- function
- loop / if else
- cấu trúc module

# 8.2 Đợt 2
- ROI / image region thinking
- config parsing
- vector / list / dict

# 8.3 Đợt 3
- disparity
- depth
- point cloud
- camera → robot transform
- object localization 3D

# 8.4 Đợt 4
- perception pipeline
- follow / pick / action-oriented decision
- humanoid robot perception logic

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 18, bạn phải nắm được 10 ý rất quan trọng:

## 1. Pick candidate không chỉ dựa vào “đây là object”
Robot còn cần kiểm tra object có **nằm trong vùng thao tác** và **có kích thước phù hợp** không.

## 2. Một object có thể thấy rõ nhưng vẫn không pick được
Ví dụ:
- quá xa
- quá to
- nằm lệch khỏi workspace
- point cloud không đủ tin cậy

## 3. Workspace filtering là phần rất quan trọng
Ví dụ:
- object phải nằm trong khoảng X/Y/Z nào đó so với robot base

## 4. Kích thước object ảnh hưởng trực tiếp tới khả năng grasp
Một object quá lớn hoặc quá dẹt có thể không phù hợp với grasp rule đang dùng.

## 5. Pick candidate evaluation là cầu nối perception → manipulation planning
Bài này chưa làm grasp planner, nhưng đã tạo **đầu vào sạch hơn** cho planner.

## 6. Không phải ROI nào cũng là object đáng pick
Có ROI chỉ là vùng nền hoặc object phụ.

## 7. Robot frame là nơi đánh giá pickability hợp lý hơn camera pixel
Vì workspace và tầm với của robot được định nghĩa trong hệ tọa độ robot.

## 8. Pick decision config rất quan trọng
Ví dụ:
- min/max reachable distance
- lateral workspace range
- height range
- max object size

## 9. Đây là nền cho pick-and-place vision thực sự
Sau này bạn có thể thêm:
- grasp pose estimation
- tabletop object selection
- hand approach direction

## 10. Đây là bước rất đúng trước khi sang Table Cleaning Robot
Vì table cleaning cũng cần biết:
- vật nào nên nhặt
- vật nào nên đẩy
- vật nào bỏ qua

---

# 10. Cấu trúc folder

```text
mini_project_18_robot_pick_candidate_vision_starter/
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
│     ├─ pair_01_left_pick_overlay.jpg
│     ├─ pair_01_disparity_visualization.jpg
│     ├─ pair_01_depth_visualization.jpg
│     ├─ pair_01_robot_point_cloud.txt
│     ├─ pair_02_left_pick_overlay.jpg
│     ├─ pair_02_disparity_visualization.jpg
│     ├─ pair_02_depth_visualization.jpg
│     ├─ pair_02_robot_point_cloud.txt
│     └─ pick_candidate_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ roi_config.txt
│  ├─ stereo_bm_config.txt
│  ├─ stereo_calibration_config.txt
│  ├─ camera_intrinsics_config.txt
│  ├─ point_sampling_config.txt
│  ├─ camera_to_robot_transform.txt
│  └─ pick_decision_config.txt
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
   │  ├─ PickDecisionConfig.hpp
   │  ├─ PickCandidateDecision.hpp
   │  ├─ PickCandidateSceneResult.hpp
   │  ├─ BasePickCandidateEvaluator.hpp
   │  ├─ StereoPickCandidateEvaluator.hpp
   │  └─ PickCandidateReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoPickCandidateEvaluator.cpp
      └─ PickCandidateReportWriter.cpp
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
pick_decision_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in toàn bộ đường dẫn config

---

# 11.2 Python — `PickCandidateConfigBuilder`

Tạo class con:

```python
class PickCandidateConfigBuilder(BaseConfigBuilder):
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
pick_decision_config
```

## `pick_decision_config`

Ví dụ:

```python
{
    "target_roi_name": "object_region",
    "min_valid_points": 120,

    "min_reach_distance_m": 0.20,
    "max_reach_distance_m": 1.10,

    "min_workspace_y_m": -0.35,
    "max_workspace_y_m": 0.35,

    "min_workspace_z_m": 0.10,
    "max_workspace_z_m": 1.20,

    "max_object_size_x_m": 0.25,
    "max_object_size_y_m": 0.25,
    "max_object_size_z_m": 0.30
}
```

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- thêm stereo pair

### `add_roi_region(roi_name, x, y, width, height)`
- thêm ROI / object region

### `set_stereo_bm_config(...)`
- giống Bài 17

### `set_stereo_calibration_config(...)`
- giống Bài 17

### `set_camera_intrinsics_config(...)`
- giống Bài 17

### `set_point_sampling_config(...)`
- giống Bài 17

### `set_camera_to_robot_transform(...)`
- giống Bài 17

### `set_pick_decision_config(...)`
**Hành vi**
- lưu config đánh giá pick candidate
- kiểm tra:
  - `max_reach_distance_m > min_reach_distance_m`
  - `max_workspace_y_m > min_workspace_y_m`
  - `max_workspace_z_m > min_workspace_z_m`
  - các ngưỡng size > 0

### `write_pick_decision_config()`

Format gợi ý:

```text
target_roi_name=object_region
min_valid_points=120
min_reach_distance_m=0.20
max_reach_distance_m=1.10
min_workspace_y_m=-0.35
max_workspace_y_m=0.35
min_workspace_z_m=0.10
max_workspace_z_m=1.20
max_object_size_x_m=0.25
max_object_size_y_m=0.25
max_object_size_z_m=0.30
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
  - pick decision config
- ghi đủ toàn bộ file config của project

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

- giống các bài trước

---

# 11.6 C++ — `StereoFrameRecord`

**File:**

```text
cpp/include/StereoFrameRecord.hpp
```

- giống các bài trước

---

# 11.7 C++ — `ROIConfig`

**File:**

```text
cpp/include/ROIConfig.hpp
```

- giống các bài trước

---

# 11.8 C++ — `StereoBMConfig`

**File:**

```text
cpp/include/StereoBMConfig.hpp
```

- giống các bài trước

---

# 11.9 C++ — `StereoCalibrationConfig`

**File:**

```text
cpp/include/StereoCalibrationConfig.hpp
```

- giống các bài trước

---

# 11.10 C++ — `CameraIntrinsics`

**File:**

```text
cpp/include/CameraIntrinsics.hpp
```

- giống các bài trước

---

# 11.11 C++ — `Transform3D`

**File:**

```text
cpp/include/Transform3D.hpp
```

- giống các bài trước

---

# 11.12 C++ — `Point3D`

**File:**

```text
cpp/include/Point3D.hpp
```

- giống các bài trước

---

# 11.13 C++ — `RobotFramePoint3D`

**File:**

```text
cpp/include/RobotFramePoint3D.hpp
```

- giống các bài trước

---

# 11.14 C++ — `LocalizedObject3D`

**File:**

```text
cpp/include/LocalizedObject3D.hpp
```

- giống Bài 15 / 16 / 17

---

# 11.15 C++ — `PickDecisionConfig`

**File:**

```text
cpp/include/PickDecisionConfig.hpp
```

Tạo struct:

```cpp
struct PickDecisionConfig
{
    std::string target_roi_name;
    int min_valid_points;

    double min_reach_distance_m;
    double max_reach_distance_m;

    double min_workspace_y_m;
    double max_workspace_y_m;

    double min_workspace_z_m;
    double max_workspace_z_m;

    double max_object_size_x_m;
    double max_object_size_y_m;
    double max_object_size_z_m;
};
```

---

# 11.16 C++ — `PickCandidateDecision`

**File:**

```text
cpp/include/PickCandidateDecision.hpp
```

Tạo struct:

```cpp
struct PickCandidateDecision
{
    std::string pair_name;
    std::string roi_name;

    bool target_found;
    bool target_valid;
    bool in_reach_distance;
    bool in_workspace;
    bool size_valid;

    std::string decision_label;
};
```

## `decision_label` gợi ý
- `"PICK_CANDIDATE"`
- `"REJECT_TOO_FAR"`
- `"REJECT_OUT_OF_WORKSPACE"`
- `"REJECT_TOO_LARGE"`
- `"REJECT_INVALID"`

---

# 11.17 C++ — `PickCandidateSceneResult`

**File:**

```text
cpp/include/PickCandidateSceneResult.hpp
```

Tạo struct:

```cpp
struct PickCandidateSceneResult
{
    std::string pair_name;

    std::string left_image_path;
    std::string right_image_path;

    std::string left_overlay_output_path;
    std::string disparity_output_path;
    std::string depth_output_path;
    std::string robot_point_cloud_output_path;

    bool is_valid;

    std::vector<LocalizedObject3D> localized_objects;
    std::vector<PickCandidateDecision> pick_decisions;
};
```

---

# 11.18 C++ — `BasePickCandidateEvaluator`

**File:**

```text
cpp/include/BasePickCandidateEvaluator.hpp
```

Tạo abstract class:

```cpp
class BasePickCandidateEvaluator
{
public:
    virtual void load_stereo_manifest(const std::string& path) = 0;
    virtual void load_roi_config(const std::string& path) = 0;
    virtual void load_stereo_bm_config(const std::string& path) = 0;
    virtual void load_stereo_calibration_config(const std::string& path) = 0;
    virtual void load_camera_intrinsics_config(const std::string& path) = 0;
    virtual void load_point_sampling_config(const std::string& path) = 0;
    virtual void load_camera_to_robot_transform(const std::string& path) = 0;
    virtual void load_pick_decision_config(const std::string& path) = 0;
    virtual void run_pick_candidate_evaluation() = 0;
    virtual ~BasePickCandidateEvaluator() = default;
};
```

---

# 11.19 C++ — `StereoPickCandidateEvaluator`

**File:**

```text
cpp/include/StereoPickCandidateEvaluator.hpp
cpp/src/StereoPickCandidateEvaluator.cpp
```

Class kế thừa:

```cpp
class StereoPickCandidateEvaluator : public BasePickCandidateEvaluator
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
    PickDecisionConfig pick_config;

    int pixel_step;
    double min_depth_m;
    double max_depth_m;

    std::vector<PickCandidateSceneResult> scene_results;
```

---

## Nhóm hàm load config
- `load_stereo_manifest(...)`
- `load_roi_config(...)`
- `load_stereo_bm_config(...)`
- `load_stereo_calibration_config(...)`
- `load_camera_intrinsics_config(...)`
- `load_point_sampling_config(...)`
- `load_camera_to_robot_transform(...)`
- `load_pick_decision_config(...)`

---

## Nhóm hàm perception core

Tái sử dụng logic từ Bài 15 / 16 / 17:

### `cv::Mat compute_disparity_map(...)`
### `cv::Mat compute_depth_map(...)`
### `Point3D back_project_pixel_to_point(...)`
### `RobotFramePoint3D transform_camera_point_to_robot_point(...)`
### `LocalizedObject3D localize_object_from_robot_points(...)`

---

## Nhóm hàm pick decision

### `bool is_target_object(const LocalizedObject3D& object) const;`
- kiểm tra `roi_name == target_roi_name`

### `bool is_object_valid_for_pick(const LocalizedObject3D& object) const;`
- `object.is_valid`
- `point_count >= min_valid_points`

### `bool is_in_reach_distance(const LocalizedObject3D& object) const;`
- center distance nằm trong `[min_reach_distance_m, max_reach_distance_m]`

### `bool is_in_workspace(const LocalizedObject3D& object) const;`
**Gợi ý rule**
- `center_y` nằm trong `[min_workspace_y_m, max_workspace_y_m]`
- `center_z` nằm trong `[min_workspace_z_m, max_workspace_z_m]`

### `bool is_pick_size_valid(const LocalizedObject3D& object) const;`
**Gợi ý rule**
- `size_x <= max_object_size_x_m`
- `size_y <= max_object_size_y_m`
- `size_z <= max_object_size_z_m`

### `PickCandidateDecision build_pick_decision(
    const std::string& pair_name,
    const LocalizedObject3D& object
) const;`

## Rule gợi ý

```text
nếu invalid → REJECT_INVALID
nếu không đủ reach distance → REJECT_TOO_FAR
nếu ngoài workspace → REJECT_OUT_OF_WORKSPACE
nếu quá lớn → REJECT_TOO_LARGE
ngược lại → PICK_CANDIDATE
```

---

### `void draw_pick_overlay(
    cv::Mat& left_image,
    const std::vector<LocalizedObject3D>& objects,
    const std::vector<PickCandidateDecision>& decisions
) const;`

**Hành vi**
- vẽ ROI
- ghi:
  - roi name
  - center distance
  - pick decision

---

### `PickCandidateSceneResult process_single_stereo_pair(
    const StereoFrameRecord& record
);`

**Hành vi tổng quát**
1. đọc ảnh trái / phải
2. compute disparity
3. compute depth
4. build robot-frame point cloud
5. localize object 3D
6. build pick decision cho từng object
7. draw overlay
8. save outputs
9. build scene result

---

### `void run_pick_candidate_evaluation() override;`
- loop qua stereo pairs

### Getter
```cpp
const std::vector<PickCandidateSceneResult>& get_scene_results() const;
```

---

# 11.20 C++ — `PickCandidateReportWriter`

**File:**

```text
cpp/include/PickCandidateReportWriter.hpp
cpp/src/PickCandidateReportWriter.cpp
```

Tạo class:

```cpp
class PickCandidateReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<PickCandidateSceneResult>& scene_results
);`

## Format gợi ý

```text
[Pick Candidate Scene]
Pair Name: pair_01
Valid: true

  [Localized Object]
  ROI Name: object_region
  Center: (0.22, 0.04, 0.92)
  Size: (0.14, 0.13, 0.18)

  [Pick Decision]
  Target Found: true
  Target Valid: true
  In Reach Distance: true
  In Workspace: true
  Size Valid: true
  Label: PICK_CANDIDATE

----------------------------------------
```

---

# 11.21 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- tạo `StereoPickCandidateEvaluator`
- load toàn bộ config
- chạy `run_pick_candidate_evaluation()`
- tạo `PickCandidateReportWriter`
- ghi report ra:
  - `assets/outputs/pick_candidate_report.txt`

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
→ Load Pick Decision Config
→ Run Pick Candidate Evaluation Pipeline
→ Save Overlay + Disparity + Depth + Robot Point Cloud
→ Write Pick Candidate Report
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
- robot-frame point cloud
- localized object 3D
- pick decision logic
- pick candidate output
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
config/pick_decision_config.txt
```

## Ảnh / point cloud output
```text
assets/outputs/pair_01_left_pick_overlay.jpg
assets/outputs/pair_01_disparity_visualization.jpg
assets/outputs/pair_01_depth_visualization.jpg
assets/outputs/pair_01_robot_point_cloud.txt
```

## File report
```text
assets/outputs/pick_candidate_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo manifest stereo pairs
- tạo ROI config
- tạo stereo / calibration / intrinsics / transform config
- tạo pick decision config
- hỗ trợ report

Tức là Python làm phần:

```text
Pick Candidate Vision Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- compute disparity
- compute depth
- build robot-frame point cloud
- localize object 3D
- đánh giá pick candidate
- ghi report

Tức là C++ làm phần:

```text
Pick Candidate Vision Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **biến stereo pair thành disparity**
- **biến disparity thành depth**
- **biến depth thành point cloud**
- **đưa point cloud sang robot frame**
- **localize object**
- **đánh giá object có phù hợp để pick không**

Tức là CV làm phần:

```text
Stereo Pair → Disparity → Depth → Point Cloud → Robot Frame → Object Localization → Pick Candidate Decision
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ file config
- [ ] Python có class cha / class con
- [ ] C++ có `BasePickCandidateEvaluator`
- [ ] C++ có `StereoPickCandidateEvaluator`
- [ ] C++ load đủ config
- [ ] C++ compute disparity
- [ ] C++ compute depth
- [ ] C++ tạo robot-frame point cloud
- [ ] C++ localize object 3D
- [ ] C++ build pick decision
- [ ] C++ vẽ pick overlay
- [ ] C++ ghi pick report

---

# 16. Gợi ý mở rộng

## 1. Thêm grasp score đơn giản
Ví dụ:
- object càng gần center workspace càng được điểm cao
- object càng nhỏ càng dễ pick hơn

## 2. Thêm tabletop filtering
Chỉ pick object có độ cao nằm trên mặt bàn.

## 3. Thêm preferred object class / ROI priority
Ví dụ:
- ưu tiên `cup_region`
- bỏ qua `background_region`

## 4. Chuẩn bị cho Bài 19
Sau Bài 18, bước hợp lý cho **Bài 19** là:

```text
Robot Table Cleaning Vision Starter
```

Tức là perception không chỉ tìm object để gắp, mà còn bắt đầu phân loại:
- object nào cần nhặt
- object nào cần đẩy
- object nào là vùng mặt bàn
