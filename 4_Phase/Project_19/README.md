# 🤖 Bài 19: Robot Table Cleaning Vision Starter — Phân loại object để nhặt / đẩy / bỏ qua trong scene dọn bàn cho Humanoid Robot AI Perception

> Mini Project số 19 trong **Đợt 4 — Bài 16 → Bài 20**  
> **Bài 19 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.  
> Nếu **Bài 18** đã giúp robot đánh giá object nào là **pick candidate** cho thao tác gắp, thì **Bài 19** sẽ đi tiếp sang một bài rất đúng với **Robotics Vision / Humanoid Perception**:
>
> **Table Cleaning Vision** — perception không chỉ đánh giá một object riêng lẻ, mà bắt đầu **đọc toàn scene trên mặt bàn** để quyết định:
>
> - object nào nên **pick**
> - object nào nên **push / sweep**
> - object nào nên **ignore**
> - vùng nào là **tabletop / working area**

---

# 📌 Mục lục

- [1. Bài 19 lấy gì từ Đợt 4](#1-bài-19-lấy-gì-từ-đợt-4)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 19 nằm ở đâu trong roadmap](#3-bài-19-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 19 là bước tiếp theo hợp lý sau Bài 18](#4-vì-sao-bài-19-là-bước-tiếp-theo-hợp-lý-sau-bài-18)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. Kiến thức Đợt 1 + 2 + 3 + 4 được dùng như thế nào](#8-kiến-thức-đợt-1--2--3--4-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
  - [11.1 Python — BaseConfigBuilder](#111-python--baseconfigbuilder)
  - [11.2 Python — TableCleaningConfigBuilder](#112-python--tablecleaningconfigbuilder)
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
  - [11.15 C++ — TableCleaningDecisionConfig](#1115-c--tablecleaningdecisionconfig)
  - [11.16 C++ — TableCleaningDecision](#1116-c--tablecleaningdecision)
  - [11.17 C++ — TableCleaningSceneResult](#1117-c--tablecleaningsceneresult)
  - [11.18 C++ — BaseTableCleaningEvaluator](#1118-c--basetablecleaningevaluator)
  - [11.19 C++ — StereoTableCleaningEvaluator](#1119-c--stereotablecleaningevaluator)
  - [11.20 C++ — TableCleaningReportWriter](#1120-c--tablecleaningreportwriter)
  - [11.21 C++ — main.cpp](#1121-c--maincpp)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 19 lấy gì từ Đợt 4

Theo roadmap, **Đợt 4** tập trung vào:

- **Robot Perception Pipeline**
- **Object Following**
- **Pick and Place Vision**
- **Table Cleaning Robot**
- **Humanoid Robot Perception**

Nếu:
- **Bài 16** = pipeline supervisor
- **Bài 17** = object following
- **Bài 18** = pick candidate evaluation

thì **Bài 19** sẽ đi vào nhánh tiếp theo:

## **Table Cleaning Vision**

Tức là robot không chỉ đánh giá một object đơn lẻ, mà phải nhìn **toàn scene trên bàn** để quyết định:

- vật nào **nhặt lên** được
- vật nào nên **đẩy / sweep**
- vật nào nên **bỏ qua**
- vật nào không hợp lệ / ngoài vùng làm việc

## Phần mới của Đợt 4 mà Bài 19 dùng
### Scene-Level Robotics Vision
- tabletop workspace thinking
- object action labeling
- multi-object scene decision
- clean-up oriented perception

### Python
- config builder cho table cleaning decision
- scene action config
- report helper

### C++
- runtime đánh giá **cleaning action** cho từng object
- tổng hợp scene-level result

---

# 2. Mô tả

Ở **Bài 18**, bạn đã có hệ thống đánh giá:

- object nào là `PICK_CANDIDATE`
- object nào bị reject vì:
  - quá xa
  - ngoài workspace
  - quá lớn
  - invalid

Bài 19 sẽ **đi thêm một bước gần với task robot dọn bàn**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc stereo pair
- đọc ROI / object config
- đọc stereo / depth / transform config
- localize object 3D trong robot frame
- đánh giá từng object theo **table cleaning rules**
- gắn nhãn action cho từng object:
  - `PICK_OBJECT`
  - `PUSH_OBJECT`
  - `IGNORE_OBJECT`
  - `INVALID_OBJECT`
- đếm số object trong scene theo từng loại action
- vẽ overlay lên ảnh trái
- ghi report table-cleaning cho từng scene

<p align="center">
  <img src="images/project_19.png" width="800">
</p>

---

# 3. Bài 19 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**

Vì vậy:

## **Bài 19 = bài thứ tư của Đợt 4**
và phải **kết hợp lại kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.

---

# 4. Vì sao Bài 19 là bước tiếp theo hợp lý sau Bài 18

## Bài 18 cho bạn:
- object localization
- pick candidate evaluation

## Bài 19 thêm:
- multi-object scene action decision
- tabletop clean-up logic

Tức là chuyển từ:

```text
“object này có pick được không?”
```

sang:

```text
“trong cả scene dọn bàn này, object nào nên pick, object nào nên push, object nào nên ignore?”
```

Đây là bước rất đúng để perception bắt đầu phục vụ **household humanoid robot behavior**.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Pair + ROI/Object Config + StereoBM Config + Calibration Config + Intrinsics + Camera-to-Robot Transform + Table Cleaning Decision Config
→ Load Left / Right Image Pair
→ Compute Disparity Map
→ Convert Disparity to Depth
→ Back-Project to Camera-Frame Point Cloud
→ Transform to Robot-Frame Point Cloud
→ Localize Objects 3D
→ Evaluate Cleaning Action for Each Object
→ Aggregate Scene Cleaning Summary
→ Save Overlay + Disparity + Depth + Table Cleaning Report
```

Bài này giúp bạn hiểu một module rất quan trọng trong humanoid household robotics:

> **Perception phải biết biến một scene thành danh sách action hợp lý cho robot.**

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
→ Read Table Cleaning Decision Config
→ Create Stereo Camera Sensor Object
→ For Each Stereo Pair:
    → Load Left / Right Image
    → Compute Disparity Map
    → Convert Disparity to Depth Map
    → Back-Project Valid Pixels to Camera-Frame Points
    → Transform Camera Points to Robot-Frame Points
    → Group Points by ROI / Object Region
    → Localize Objects 3D
    → Evaluate Cleaning Action for Each Object
    → Aggregate Scene Summary
    → Save Overlay + Disparity + Depth + Robot Point Cloud + Cleaning Report
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
- multi-object localization
- table cleaning decision logic

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 được dùng như thế nào

# 8.1 Đợt 1
- Python / C++ OOP cơ bản
- function
- loop / if else
- module

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
- follow / pick / scene-action decision
- humanoid robot perception logic

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 19, bạn phải nắm được 10 ý rất quan trọng:

## 1. Household perception thường là scene-level, không chỉ object-level
Robot dọn bàn phải nhìn cả scene chứ không chỉ 1 object.

## 2. Không phải object nào cũng nên pick
Có object có thể phù hợp hơn với hành động push / sweep.

## 3. Cần có action labeling cho từng object
Ví dụ:
- cốc nhỏ trong workspace → `PICK_OBJECT`
- khăn giấy mỏng ở rìa → `PUSH_OBJECT`
- vật quá xa → `IGNORE_OBJECT`

## 4. Multi-object reasoning quan trọng hơn single-object reasoning
Robot cần biết object nào ưu tiên trước.

## 5. Scene summary rất hữu ích
Ví dụ:
- số object cần pick
- số object cần push
- số object bị ignore

## 6. Tabletop workspace là khái niệm quan trọng
Không phải object nào nhìn thấy cũng nằm trên mặt bàn hoặc trong vùng robot thao tác.

## 7. Một perception pipeline tốt cần phân tách:
- geometry
- localization
- object decision
- scene aggregation

## 8. Đây là bước đệm rất tốt cho task household robot thật
Ví dụ:
- dọn bàn
- gom vật
- phân loại vật cần nhặt

## 9. Đây là cầu nối perception → task planning
Planner có thể lấy scene report để chọn action sequence.

## 10. Đây là bước rất hợp lý trước Bài 20
Vì Bài 20 có thể gom toàn bộ Đợt 4 thành **Humanoid Multi-Task Perception Manager**.

---

# 10. Cấu trúc folder

```text
mini_project_19_robot_table_cleaning_vision_starter/
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
│     ├─ pair_01_left_table_cleaning_overlay.jpg
│     ├─ pair_01_disparity_visualization.jpg
│     ├─ pair_01_depth_visualization.jpg
│     ├─ pair_01_robot_point_cloud.txt
│     ├─ pair_02_left_table_cleaning_overlay.jpg
│     ├─ pair_02_disparity_visualization.jpg
│     ├─ pair_02_depth_visualization.jpg
│     ├─ pair_02_robot_point_cloud.txt
│     └─ table_cleaning_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ roi_config.txt
│  ├─ stereo_bm_config.txt
│  ├─ stereo_calibration_config.txt
│  ├─ camera_intrinsics_config.txt
│  ├─ point_sampling_config.txt
│  ├─ camera_to_robot_transform.txt
│  └─ table_cleaning_decision_config.txt
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
   │  ├─ TableCleaningDecisionConfig.hpp
   │  ├─ TableCleaningDecision.hpp
   │  ├─ TableCleaningSceneResult.hpp
   │  ├─ BaseTableCleaningEvaluator.hpp
   │  ├─ StereoTableCleaningEvaluator.hpp
   │  └─ TableCleaningReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoTableCleaningEvaluator.cpp
      └─ TableCleaningReportWriter.cpp
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
table_cleaning_decision_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in toàn bộ đường dẫn config

---

# 11.2 Python — `TableCleaningConfigBuilder`

Tạo class con:

```python
class TableCleaningConfigBuilder(BaseConfigBuilder):
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
table_cleaning_decision_config
```

## `table_cleaning_decision_config`

Ví dụ:

```python
{
    "target_table_roi_name": "table_region",
    "min_valid_points": 100,

    "pick_max_size_m": 0.25,
    "push_max_size_m": 0.45,

    "pick_min_distance_m": 0.20,
    "pick_max_distance_m": 1.00,

    "push_min_distance_m": 0.15,
    "push_max_distance_m": 1.20,

    "workspace_min_y_m": -0.40,
    "workspace_max_y_m": 0.40,
    "workspace_min_z_m": 0.05,
    "workspace_max_z_m": 1.20
}
```

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- thêm stereo pair

### `add_roi_region(roi_name, x, y, width, height)`
- thêm ROI / object region

### `set_stereo_bm_config(...)`
- giống Bài 18

### `set_stereo_calibration_config(...)`
- giống Bài 18

### `set_camera_intrinsics_config(...)`
- giống Bài 18

### `set_point_sampling_config(...)`
- giống Bài 18

### `set_camera_to_robot_transform(...)`
- giống Bài 18

### `set_table_cleaning_decision_config(...)`
**Hành vi**
- lưu config action cho table cleaning
- kiểm tra:
  - `pick_max_distance_m > pick_min_distance_m`
  - `push_max_distance_m > push_min_distance_m`
  - `push_max_size_m >= pick_max_size_m`
  - workspace min/max hợp lệ

### `write_table_cleaning_decision_config()`

Format gợi ý:

```text
target_table_roi_name=table_region
min_valid_points=100
pick_max_size_m=0.25
push_max_size_m=0.45
pick_min_distance_m=0.20
pick_max_distance_m=1.00
push_min_distance_m=0.15
push_max_distance_m=1.20
workspace_min_y_m=-0.40
workspace_max_y_m=0.40
workspace_min_z_m=0.05
workspace_max_z_m=1.20
```

---

# 11.3 Python — `main_config_builder.py`

## Yêu cầu
- tạo ít nhất **3 stereo pairs**
- tạo ít nhất **4 ROI**
- set:
  - stereo BM config
  - stereo calibration config
  - camera intrinsics config
  - point sampling config
  - camera-to-robot transform
  - table cleaning decision config
- ghi đủ toàn bộ file config của project

---

# 11.4 C++ — `BaseSensor`
- giống các bài trước

# 11.5 C++ — `StereoCameraSensor`
- giống các bài trước

# 11.6 C++ — `StereoFrameRecord`
- giống các bài trước

# 11.7 C++ — `ROIConfig`
- giống các bài trước

# 11.8 C++ — `StereoBMConfig`
- giống các bài trước

# 11.9 C++ — `StereoCalibrationConfig`
- giống các bài trước

# 11.10 C++ — `CameraIntrinsics`
- giống các bài trước

# 11.11 C++ — `Transform3D`
- giống các bài trước

# 11.12 C++ — `Point3D`
- giống các bài trước

# 11.13 C++ — `RobotFramePoint3D`
- giống các bài trước

# 11.14 C++ — `LocalizedObject3D`
- giống Bài 15 / 16 / 17 / 18

---

# 11.15 C++ — `TableCleaningDecisionConfig`

**File:**

```text
cpp/include/TableCleaningDecisionConfig.hpp
```

Tạo struct:

```cpp
struct TableCleaningDecisionConfig
{
    std::string target_table_roi_name;
    int min_valid_points;

    double pick_max_size_m;
    double push_max_size_m;

    double pick_min_distance_m;
    double pick_max_distance_m;

    double push_min_distance_m;
    double push_max_distance_m;

    double workspace_min_y_m;
    double workspace_max_y_m;
    double workspace_min_z_m;
    double workspace_max_z_m;
};
```

---

# 11.16 C++ — `TableCleaningDecision`

**File:**

```text
cpp/include/TableCleaningDecision.hpp
```

Tạo struct:

```cpp
struct TableCleaningDecision
{
    std::string pair_name;
    std::string roi_name;

    bool target_found;
    bool target_valid;
    bool in_workspace;
    bool pick_distance_valid;
    bool push_distance_valid;
    bool pick_size_valid;
    bool push_size_valid;

    std::string action_label;
};
```

## `action_label` gợi ý
- `"PICK_OBJECT"`
- `"PUSH_OBJECT"`
- `"IGNORE_OBJECT"`
- `"INVALID_OBJECT"`

---

# 11.17 C++ — `TableCleaningSceneResult`

**File:**

```text
cpp/include/TableCleaningSceneResult.hpp
```

Tạo struct:

```cpp
struct TableCleaningSceneResult
{
    std::string pair_name;

    std::string left_image_path;
    std::string right_image_path;

    std::string left_overlay_output_path;
    std::string disparity_output_path;
    std::string depth_output_path;
    std::string robot_point_cloud_output_path;

    int pick_count;
    int push_count;
    int ignore_count;
    int invalid_count;

    bool is_valid;

    std::vector<LocalizedObject3D> localized_objects;
    std::vector<TableCleaningDecision> cleaning_decisions;
};
```

---

# 11.18 C++ — `BaseTableCleaningEvaluator`

**File:**

```text
cpp/include/BaseTableCleaningEvaluator.hpp
```

Tạo abstract class:

```cpp
class BaseTableCleaningEvaluator
{
public:
    virtual void load_stereo_manifest(const std::string& path) = 0;
    virtual void load_roi_config(const std::string& path) = 0;
    virtual void load_stereo_bm_config(const std::string& path) = 0;
    virtual void load_stereo_calibration_config(const std::string& path) = 0;
    virtual void load_camera_intrinsics_config(const std::string& path) = 0;
    virtual void load_point_sampling_config(const std::string& path) = 0;
    virtual void load_camera_to_robot_transform(const std::string& path) = 0;
    virtual void load_table_cleaning_decision_config(const std::string& path) = 0;
    virtual void run_table_cleaning_evaluation() = 0;
    virtual ~BaseTableCleaningEvaluator() = default;
};
```

---

# 11.19 C++ — `StereoTableCleaningEvaluator`

**File:**

```text
cpp/include/StereoTableCleaningEvaluator.hpp
cpp/src/StereoTableCleaningEvaluator.cpp
```

Class kế thừa:

```cpp
class StereoTableCleaningEvaluator : public BaseTableCleaningEvaluator
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
    TableCleaningDecisionConfig cleaning_config;

    int pixel_step;
    double min_depth_m;
    double max_depth_m;

    std::vector<TableCleaningSceneResult> scene_results;
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
- `load_table_cleaning_decision_config(...)`

---

## Nhóm hàm perception core

Tái sử dụng logic từ Bài 15 / 16 / 17 / 18:

### `cv::Mat compute_disparity_map(...)`
### `cv::Mat compute_depth_map(...)`
### `Point3D back_project_pixel_to_point(...)`
### `RobotFramePoint3D transform_camera_point_to_robot_point(...)`
### `LocalizedObject3D localize_object_from_robot_points(...)`

---

## Nhóm hàm cleaning decision

### `bool is_object_valid(const LocalizedObject3D& object) const;`
- `object.is_valid`
- `point_count >= min_valid_points`

### `bool is_in_workspace(const LocalizedObject3D& object) const;`
- `center_y` trong `[workspace_min_y_m, workspace_max_y_m]`
- `center_z` trong `[workspace_min_z_m, workspace_max_z_m]`

### `bool is_pick_distance_valid(const LocalizedObject3D& object) const;`
- distance trong `[pick_min_distance_m, pick_max_distance_m]`

### `bool is_push_distance_valid(const LocalizedObject3D& object) const;`
- distance trong `[push_min_distance_m, push_max_distance_m]`

### `bool is_pick_size_valid(const LocalizedObject3D& object) const;`
- size_x / size_y / size_z đủ nhỏ cho pick

### `bool is_push_size_valid(const LocalizedObject3D& object) const;`
- size phù hợp cho push nhưng có thể lớn hơn pick

### `TableCleaningDecision build_cleaning_decision(
    const std::string& pair_name,
    const LocalizedObject3D& object
) const;`

## Rule gợi ý

```text
nếu invalid → INVALID_OBJECT
nếu ngoài workspace → IGNORE_OBJECT
nếu đủ pick distance + pick size → PICK_OBJECT
nếu không pick được nhưng đủ push distance + push size → PUSH_OBJECT
ngược lại → IGNORE_OBJECT
```

---

### `void update_scene_action_counters(
    TableCleaningSceneResult& scene_result,
    const TableCleaningDecision& decision
) const;`

**Hành vi**
- tăng `pick_count / push_count / ignore_count / invalid_count`

---

### `void draw_table_cleaning_overlay(
    cv::Mat& left_image,
    const std::vector<LocalizedObject3D>& objects,
    const std::vector<TableCleaningDecision>& decisions
) const;`

**Hành vi**
- vẽ ROI
- ghi:
  - roi name
  - center distance
  - action label

---

### `TableCleaningSceneResult process_single_stereo_pair(
    const StereoFrameRecord& record
);`

**Hành vi tổng quát**
1. đọc ảnh trái / phải
2. compute disparity
3. compute depth
4. build robot-frame point cloud
5. localize objects 3D
6. build cleaning decision cho từng object
7. update scene counters
8. draw overlay
9. save outputs
10. build scene result

---

### `void run_table_cleaning_evaluation() override;`
- loop qua stereo pairs

### Getter
```cpp
const std::vector<TableCleaningSceneResult>& get_scene_results() const;
```

---

# 11.20 C++ — `TableCleaningReportWriter`

**File:**

```text
cpp/include/TableCleaningReportWriter.hpp
cpp/src/TableCleaningReportWriter.cpp
```

Tạo class:

```cpp
class TableCleaningReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<TableCleaningSceneResult>& scene_results
);`

## Format gợi ý

```text
[Table Cleaning Scene]
Pair Name: pair_01
Valid: true
Pick Count: 2
Push Count: 1
Ignore Count: 1
Invalid Count: 0

  [Localized Object]
  ROI Name: object_region
  Center: (0.22, 0.04, 0.92)
  Size: (0.14, 0.13, 0.18)

  [Cleaning Decision]
  In Workspace: true
  Pick Distance Valid: true
  Push Distance Valid: true
  Pick Size Valid: true
  Push Size Valid: true
  Action: PICK_OBJECT

----------------------------------------
```

---

# 11.21 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- tạo `StereoTableCleaningEvaluator`
- load toàn bộ config
- chạy `run_table_cleaning_evaluation()`
- tạo `TableCleaningReportWriter`
- ghi report ra:
  - `assets/outputs/table_cleaning_report.txt`

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
→ Load Table Cleaning Decision Config
→ Run Table Cleaning Evaluation Pipeline
→ Save Overlay + Disparity + Depth + Robot Point Cloud
→ Write Table Cleaning Report
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
- multi-object localization
- scene-level action decision
- scene action counters
- table cleaning report

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
config/table_cleaning_decision_config.txt
```

## Ảnh / point cloud output
```text
assets/outputs/pair_01_left_table_cleaning_overlay.jpg
assets/outputs/pair_01_disparity_visualization.jpg
assets/outputs/pair_01_depth_visualization.jpg
assets/outputs/pair_01_robot_point_cloud.txt
```

## File report
```text
assets/outputs/table_cleaning_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo manifest stereo pairs
- tạo ROI config
- tạo stereo / calibration / intrinsics / transform config
- tạo table cleaning decision config
- hỗ trợ report

Tức là Python làm phần:

```text
Table Cleaning Vision Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- compute disparity
- compute depth
- build robot-frame point cloud
- localize nhiều object
- gắn action label cho từng object
- tổng hợp scene report

Tức là C++ làm phần:

```text
Table Cleaning Vision Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **biến stereo pair thành disparity**
- **biến disparity thành depth**
- **biến depth thành point cloud**
- **đưa point cloud sang robot frame**
- **localize nhiều object**
- **đánh giá scene để robot dọn bàn**

Tức là CV làm phần:

```text
Stereo Pair → Disparity → Depth → Point Cloud → Robot Frame → Multi-Object Localization → Cleaning Action Decision
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ file config
- [ ] Python có class cha / class con
- [ ] C++ có `BaseTableCleaningEvaluator`
- [ ] C++ có `StereoTableCleaningEvaluator`
- [ ] C++ load đủ config
- [ ] C++ compute disparity
- [ ] C++ compute depth
- [ ] C++ tạo robot-frame point cloud
- [ ] C++ localize nhiều object 3D
- [ ] C++ build cleaning decision
- [ ] C++ cập nhật scene counters
- [ ] C++ vẽ table cleaning overlay
- [ ] C++ ghi table cleaning report

---

# 16. Gợi ý mở rộng

## 1. Thêm object priority score
Ví dụ:
- object gần robot hơn → ưu tiên cao hơn
- object ở trung tâm workspace → ưu tiên cao hơn

## 2. Thêm tabletop clutter level
Ví dụ:
- scene gọn
- scene trung bình
- scene lộn xộn

## 3. Thêm cleaning plan suggestion
Ví dụ:
```text
1. PICK cup_region
2. PUSH tissue_region
3. IGNORE far_box_region
```

## 4. Chuẩn bị cho Bài 20
Sau Bài 19, bước hợp lý cho **Bài 20** là:

```text
Humanoid Multi-Task Perception Manager
```

Tức là gom toàn bộ Đợt 4 thành một project tổng hợp:
- object following
- pick candidate evaluation
- table cleaning decision
- scene report tổng hợp
