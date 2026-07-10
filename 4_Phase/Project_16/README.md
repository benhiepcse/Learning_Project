# 🤖 Bài 16: Robot Humanoid Perception Mini Pipeline Supervisor — Tổng hợp pipeline perception cho Humanoid Robot

> Mini Project số 16 trong **Đợt 4 — Bài 16 → Bài 20**  
> **Bài 16 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.  
> Nếu **Bài 15** đã giúp robot localize object 3D trong robot frame, thì **Bài 16** bắt đầu chuyển sang tư duy **Humanoid Robot Perception Pipeline**:
>
> ```text
> image / stereo input
> → object region
> → disparity / depth / point cloud
> → robot-frame object localization
> → action intent / perception decision
> ```

---

# 📌 Mục lục

- [1. Đợt 4 có gì mới](#1-đợt-4-có-gì-mới)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 16 nằm ở đâu trong roadmap](#3-bài-16-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 16 là bước tiếp theo sau Bài 15](#4-vì-sao-bài-16-là-bước-tiếp-theo-sau-bài-15)
- [5. Pipeline perception của bài](#5-pipeline-perception-của-bài)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. Cấu trúc folder](#7-cấu-trúc-folder)
- [8. Yêu cầu mini-project](#8-yêu-cầu-mini-project)
- [9. Điều kiện bắt buộc](#9-điều-kiện-bắt-buộc)
- [10. Output mong muốn](#10-output-mong-muốn)
- [11. Vai trò trong Humanoid Robot](#11-vai-trò-trong-humanoid-robot)
- [12. Checklist hoàn thành](#12-checklist-hoàn-thành)
- [13. Gợi ý mở rộng](#13-gợi-ý-mở-rộng)

---

# 1. Đợt 4 có gì mới

Theo roadmap, phần cuối chuyển sang **Humanoid mini pipeline**, tức là không chỉ xử lý từng module riêng nữa, mà bắt đầu ghép thành pipeline phục vụ robot:

## Python
- ôn và áp dụng toàn bộ:
  - OOP
  - dict / list
  - module
  - file handling
  - NumPy / Matplotlib nếu cần
  - config builder / report helper

## C++
- ôn và áp dụng toàn bộ:
  - class
  - inheritance
  - virtual class
  - vector
  - enum
  - smart pointer nếu muốn
  - module runtime rõ ràng

## Computer Vision
- **Phase 8 — Robotics Vision**
  - Robot Perception Pipeline
  - Object Following
  - Pick and Place Vision
  - Table Cleaning Robot
  - Humanoid Robot Perception

---

# 2. Mô tả

Ở **Bài 15**, bạn đã có object 3D localization:

```text
stereo pair
→ disparity
→ depth
→ point cloud
→ robot frame
→ 3D object center / size
```

Bài 16 sẽ không chỉ dừng ở việc “object ở đâu”, mà bắt đầu tạo một **Perception Pipeline Supervisor**.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc stereo pair hoặc scene input
- đọc ROI / object config
- chạy pipeline perception:
  - object region
  - disparity
  - depth
  - point cloud
  - robot-frame transform
  - 3D localization
- đánh giá object nào là **target**
- tạo quyết định perception đơn giản:
  - object có hợp lệ không?
  - object gần hay xa?
  - object có nằm trong vùng theo dõi không?
  - object có phù hợp để pick không?
- ghi report dạng **pipeline-level**

---

# 3. Bài 16 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**

Vì vậy:

## **Bài 16 = bài mở đầu của Đợt 4**

Bài này phải tổng hợp lại toàn bộ:
- nền Python / C++ / CV của Đợt 1
- dataset / ROI / proposal của Đợt 2
- stereo / depth / point cloud / robot frame của Đợt 3
- robotics vision pipeline của Đợt 4

---

# 4. Vì sao Bài 16 là bước tiếp theo sau Bài 15

## Bài 15 cho bạn:
- object 3D center
- object 3D size
- robot-frame object information

## Bài 16 thêm:
- pipeline supervisor
- target selection
- perception decision
- action intent level

Tức là chuyển từ:

```text
Object Localization
```

sang:

```text
Humanoid Perception Pipeline Decision
```

Robot không chỉ cần biết:
```text
object ở đâu
```

mà còn cần biết:
```text
object này có nên follow không?
object này có thể pick không?
object này có nằm trong vùng làm việc không?
```

---

# 5. Pipeline perception của bài

```text
Robot Perception Config
→ Load Stereo Pair / Scene Input
→ Load ROI / Object Region Config
→ Load Stereo / Depth / Transform Config
→ Run 3D Object Localization
→ Evaluate Each Localized Object
→ Select Target Object
→ Generate Perception Decision
→ Save Annotated Outputs
→ Write Pipeline Report
```

---

# 6. Kiến thức cần

## Python
- class / inheritance
- list / dict
- string parsing
- file handling
- config builder
- report template
- optional: Matplotlib summary plot

## C++
- class / inheritance
- abstract class
- virtual function
- vector
- enum
- struct
- function tách nhỏ
- header / source structure

## Computer Vision
- stereo pair
- disparity
- depth
- point cloud
- camera-to-robot transform
- 3D object localization
- robotics vision decision logic

---

# 7. Cấu trúc folder

```text
mini_project_16_robot_humanoid_perception_pipeline_supervisor/
│
├─ README.md
│
├─ assets/
│  ├─ stereo_pairs/
│  │  ├─ pair_01_left.jpg
│  │  ├─ pair_01_right.jpg
│  │  ├─ pair_02_left.jpg
│  │  └─ pair_02_right.jpg
│  │
│  └─ outputs/
│     ├─ pair_01_left_pipeline_overlay.jpg
│     ├─ pair_01_disparity_visualization.jpg
│     ├─ pair_01_depth_visualization.jpg
│     ├─ pair_01_robot_point_cloud.txt
│     ├─ pair_02_left_pipeline_overlay.jpg
│     ├─ pair_02_disparity_visualization.jpg
│     ├─ pair_02_depth_visualization.jpg
│     ├─ pair_02_robot_point_cloud.txt
│     └─ perception_pipeline_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ roi_config.txt
│  ├─ stereo_bm_config.txt
│  ├─ stereo_calibration_config.txt
│  ├─ camera_intrinsics_config.txt
│  ├─ point_sampling_config.txt
│  ├─ camera_to_robot_transform.txt
│  └─ perception_decision_config.txt
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
   │  ├─ PerceptionDecisionConfig.hpp
   │  ├─ ObjectPerceptionDecision.hpp
   │  ├─ PerceptionPipelineSceneResult.hpp
   │  ├─ BasePerceptionPipeline.hpp
   │  ├─ HumanoidPerceptionPipelineSupervisor.hpp
   │  └─ PerceptionPipelineReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ HumanoidPerceptionPipelineSupervisor.cpp
      └─ PerceptionPipelineReportWriter.cpp
```

---

# 8. Yêu cầu mini-project

# 8.1 Python — `BaseConfigBuilder`

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
perception_decision_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in toàn bộ đường dẫn config

---

# 8.2 Python — `HumanoidPipelineConfigBuilder`

Tạo class con:

```python
class HumanoidPipelineConfigBuilder(BaseConfigBuilder):
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
perception_decision_config
```

---

## `perception_decision_config`

Ví dụ:

```python
{
    "target_roi_name": "object_region",
    "min_valid_points": 100,
    "max_target_distance_m": 1.5,
    "min_target_distance_m": 0.2,
    "max_lateral_offset_m": 0.5,
    "max_object_size_m": 0.6,
    "enable_pick_candidate_check": True,
    "enable_follow_candidate_check": True
}
```

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- thêm stereo pair
- kiểm tra chuỗi không rỗng
- `sensor_id >= 0`

### `add_roi_region(roi_name, x, y, width, height)`
- thêm ROI / object region
- kiểm tra ROI hợp lệ

### `set_stereo_bm_config(...)`
- giống Bài 15

### `set_stereo_calibration_config(...)`
- giống Bài 15

### `set_camera_intrinsics_config(...)`
- giống Bài 15

### `set_point_sampling_config(...)`
- giống Bài 15

### `set_camera_to_robot_transform(...)`
- giống Bài 15

### `set_perception_decision_config(...)`
**Hành vi**
- lưu config quyết định perception
- kiểm tra:
  - `min_valid_points > 0`
  - `max_target_distance_m > min_target_distance_m`
  - `max_lateral_offset_m > 0`
  - `max_object_size_m > 0`

### `write_perception_decision_config()`

Format gợi ý:

```text
target_roi_name=object_region
min_valid_points=100
min_target_distance_m=0.2
max_target_distance_m=1.5
max_lateral_offset_m=0.5
max_object_size_m=0.6
enable_pick_candidate_check=1
enable_follow_candidate_check=1
```

---

# 8.3 C++ — Core Structs

## `PerceptionDecisionConfig`

**File:**

```text
cpp/include/PerceptionDecisionConfig.hpp
```

```cpp
struct PerceptionDecisionConfig
{
    std::string target_roi_name;
    int min_valid_points;

    double min_target_distance_m;
    double max_target_distance_m;
    double max_lateral_offset_m;
    double max_object_size_m;

    bool enable_pick_candidate_check;
    bool enable_follow_candidate_check;
};
```

---

## `ObjectPerceptionDecision`

**File:**

```text
cpp/include/ObjectPerceptionDecision.hpp
```

```cpp
struct ObjectPerceptionDecision
{
    std::string pair_name;
    std::string roi_name;

    bool has_valid_3d_object;
    bool is_target_roi;
    bool is_in_distance_range;
    bool is_centered_enough;
    bool is_pick_candidate;
    bool is_follow_candidate;

    std::string decision_label;
};
```

Gợi ý `decision_label`:
- `"INVALID_OBJECT"`
- `"IGNORE"`
- `"FOLLOW_TARGET"`
- `"PICK_CANDIDATE"`
- `"OBSERVE_ONLY"`

---

## `PerceptionPipelineSceneResult`

**File:**

```text
cpp/include/PerceptionPipelineSceneResult.hpp
```

```cpp
struct PerceptionPipelineSceneResult
{
    std::string pair_name;

    std::string left_image_path;
    std::string right_image_path;

    std::string left_overlay_output_path;
    std::string disparity_output_path;
    std::string depth_output_path;
    std::string robot_point_cloud_output_path;

    int object_count;
    int decision_count;

    bool is_valid;

    std::vector<LocalizedObject3D> localized_objects;
    std::vector<ObjectPerceptionDecision> decisions;
};
```

---

# 8.4 C++ — `BasePerceptionPipeline`

**File:**

```text
cpp/include/BasePerceptionPipeline.hpp
```

Tạo abstract class:

```cpp
class BasePerceptionPipeline
{
public:
    virtual void load_stereo_manifest(const std::string& path) = 0;
    virtual void load_roi_config(const std::string& path) = 0;
    virtual void load_stereo_bm_config(const std::string& path) = 0;
    virtual void load_stereo_calibration_config(const std::string& path) = 0;
    virtual void load_camera_intrinsics_config(const std::string& path) = 0;
    virtual void load_point_sampling_config(const std::string& path) = 0;
    virtual void load_camera_to_robot_transform(const std::string& path) = 0;
    virtual void load_perception_decision_config(const std::string& path) = 0;
    virtual void run_pipeline() = 0;
    virtual ~BasePerceptionPipeline() = default;
};
```

---

# 8.5 C++ — `HumanoidPerceptionPipelineSupervisor`

**File:**

```text
cpp/include/HumanoidPerceptionPipelineSupervisor.hpp
cpp/src/HumanoidPerceptionPipelineSupervisor.cpp
```

Class kế thừa:

```cpp
class HumanoidPerceptionPipelineSupervisor : public BasePerceptionPipeline
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
    PerceptionDecisionConfig decision_config;

    int pixel_step;
    double min_depth_m;
    double max_depth_m;

    std::vector<PerceptionPipelineSceneResult> scene_results;
```

---

## Nhóm hàm load config

Cần có:
- `load_stereo_manifest(...)`
- `load_roi_config(...)`
- `load_stereo_bm_config(...)`
- `load_stereo_calibration_config(...)`
- `load_camera_intrinsics_config(...)`
- `load_point_sampling_config(...)`
- `load_camera_to_robot_transform(...)`
- `load_perception_decision_config(...)`

---

## Nhóm hàm perception core

Tái sử dụng logic từ Bài 15:

### `cv::Mat compute_disparity_map(...)`
- tính disparity từ stereo pair

### `cv::Mat compute_depth_map(...)`
- disparity → depth

### `Point3D back_project_pixel_to_point(...)`
- pixel + depth → camera point

### `RobotFramePoint3D transform_camera_point_to_robot_point(...)`
- camera point → robot point

### `LocalizedObject3D localize_object_from_robot_points(...)`
- robot points → object center / size

---

## Nhóm hàm decision logic

### `bool is_object_in_distance_range(const LocalizedObject3D& object) const;`
**Hành vi**
- kiểm tra:
```text
min_target_distance_m <= center_z <= max_target_distance_m
```

### `bool is_object_centered_enough(const LocalizedObject3D& object) const;`
**Hành vi**
- kiểm tra:
```text
abs(center_y) <= max_lateral_offset_m
```

> Bạn có thể quy ước:
> - `center_x` = hướng trước/sau robot
> - `center_y` = lệch trái/phải
> - `center_z` = cao/thấp  
>
> Hoặc nếu project trước đang dùng `Z` làm khoảng cách trước camera, bạn có thể giữ `center_z` là distance để đơn giản.

### `bool is_object_size_pickable(const LocalizedObject3D& object) const;`
**Hành vi**
- kiểm tra:
```text
size_x <= max_object_size_m
size_y <= max_object_size_m
size_z <= max_object_size_m
```

### `ObjectPerceptionDecision build_decision_for_object(
    const LocalizedObject3D& object
) const;`

## Rule gợi ý

```text
nếu object invalid → INVALID_OBJECT
nếu không phải target ROI → IGNORE
nếu đủ gần + centered + size phù hợp → PICK_CANDIDATE
nếu đủ gần + centered nhưng chưa pickable → FOLLOW_TARGET
ngược lại → OBSERVE_ONLY
```

---

## Nhóm hàm output

### `void draw_pipeline_overlay(
    cv::Mat& left_image,
    const std::vector<LocalizedObject3D>& objects,
    const std::vector<ObjectPerceptionDecision>& decisions
) const;`

**Hành vi**
- vẽ ROI / object box
- ghi:
  - roi name
  - center distance
  - decision label

### `PerceptionPipelineSceneResult process_single_stereo_pair(
    const StereoFrameRecord& record
);`

**Hành vi tổng quát**
1. đọc ảnh trái / phải
2. compute disparity
3. compute depth
4. build robot-frame point cloud
5. localize object 3D theo ROI
6. build decision cho từng object
7. draw overlay
8. save output
9. build scene result

### `void run_pipeline() override;`
- loop qua stereo pairs
- gọi `process_single_stereo_pair(...)`

---

# 8.6 C++ — `PerceptionPipelineReportWriter`

**File:**

```text
cpp/include/PerceptionPipelineReportWriter.hpp
cpp/src/PerceptionPipelineReportWriter.cpp
```

Tạo class:

```cpp
class PerceptionPipelineReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<PerceptionPipelineSceneResult>& scene_results
);`

Format gợi ý:

```text
[Humanoid Perception Pipeline]
Pair Name: pair_01
Object Count: 3
Decision Count: 3
Valid: true

  [Object]
  ROI Name: object_region
  Center: (0.22, 0.04, 0.92)
  Size: (0.14, 0.13, 0.18)

  [Decision]
  Target ROI: true
  Distance Range: true
  Centered Enough: true
  Pick Candidate: true
  Follow Candidate: true
  Label: PICK_CANDIDATE

----------------------------------------
```

---

# 8.7 C++ — `main.cpp`

## Yêu cầu

```text
Create StereoCameraSensor
→ Load all configs
→ Run HumanoidPerceptionPipelineSupervisor
→ Save outputs
→ Write perception_pipeline_report.txt
```

---

# 9. Điều kiện bắt buộc

Project bắt buộc có:

- OOP Python
- OOP C++
- Inheritance Python
- Inheritance C++
- Function tách rõ
- Module Python
- Header / Source C++ tách file
- loop
- if / else
- list / dict
- std::vector
- stereo pair input
- disparity
- depth
- robot-frame point cloud
- 3D object localization
- perception decision logic
- pipeline-level report

---

# 10. Output mong muốn

## Config

```text
config/stereo_pair_manifest.txt
config/roi_config.txt
config/stereo_bm_config.txt
config/stereo_calibration_config.txt
config/camera_intrinsics_config.txt
config/point_sampling_config.txt
config/camera_to_robot_transform.txt
config/perception_decision_config.txt
```

## Output

```text
assets/outputs/pair_01_left_pipeline_overlay.jpg
assets/outputs/pair_01_disparity_visualization.jpg
assets/outputs/pair_01_depth_visualization.jpg
assets/outputs/pair_01_robot_point_cloud.txt
assets/outputs/perception_pipeline_report.txt
```

---

# 11. Vai trò trong Humanoid Robot

## Python
```text
Config Builder + Pipeline Setup Tool
```

## C++
```text
Runtime Humanoid Perception Pipeline
```

## Computer Vision
```text
Stereo + Depth + 3D Localization + Robot-Centric Decision
```

---

# 12. Checklist hoàn thành

- [ ] Python tạo đủ 8 file config
- [ ] Python có class cha / class con
- [ ] C++ có abstract pipeline class
- [ ] C++ có pipeline supervisor
- [ ] C++ load đủ config
- [ ] C++ compute disparity
- [ ] C++ compute depth
- [ ] C++ tạo robot-frame points
- [ ] C++ localize object 3D
- [ ] C++ build decision cho object
- [ ] C++ vẽ pipeline overlay
- [ ] C++ ghi pipeline report

---

# 13. Gợi ý mở rộng

## 1. Thêm object following command
Ví dụ:
```text
FOLLOW_LEFT
FOLLOW_RIGHT
MOVE_FORWARD
STOP
```

## 2. Thêm pick/place readiness
Kiểm tra:
- object đủ gần
- object không quá lớn
- object nằm trong vùng thao tác

## 3. Thêm table cleaning mode
Gắn nhãn:
- object cần dọn
- object bỏ qua
- vùng bàn trống

## 4. Chuẩn bị cho Bài 17
Bài 17 nên đi tiếp theo hướng:

```text
Robot Object Following Vision Starter
```

Tức là dùng output của pipeline để sinh quyết định follow object:
- object lệch trái → turn left
- object lệch phải → turn right
- object xa → move forward
- object đủ gần → stop
