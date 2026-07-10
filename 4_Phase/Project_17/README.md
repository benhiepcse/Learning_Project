# 🤖 Bài 17: Robot Object Following Vision Starter — Theo dõi object mục tiêu bằng robot-centric perception cho Humanoid Robot AI Perception

> Mini Project số 17 trong **Đợt 4 — Bài 16 → Bài 20**  
> **Bài 17 kết hợp kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.  
> Nếu **Bài 16** đã tạo ra một **Humanoid Perception Pipeline Supervisor** có thể localize object 3D và đưa ra quyết định perception mức cao, thì **Bài 17** sẽ đi tiếp theo hướng rất đúng với roadmap Đợt 4:
>
> **Object Following Vision** — dùng perception để quyết định robot nên **đi thẳng, rẽ trái, rẽ phải hay dừng** khi theo dõi một object mục tiêu.

---

# 📌 Mục lục

- [1. Bài 17 lấy gì từ Đợt 4](#1-bài-17-lấy-gì-từ-đợt-4)
- [2. Mô tả](#2-mô-tả)
- [3. Bài 17 nằm ở đâu trong roadmap](#3-bài-17-nằm-ở-đâu-trong-roadmap)
- [4. Vì sao Bài 17 là bước tiếp theo hợp lý sau Bài 16](#4-vì-sao-bài-17-là-bước-tiếp-theo-hợp-lý-sau-bài-16)
- [5. Mục tiêu perception của bài](#5-mục-tiêu-perception-của-bài)
- [6. Pipeline perception của bài](#6-pipeline-perception-của-bài)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. Kiến thức Đợt 1 + 2 + 3 + 4 được dùng như thế nào](#8-kiến-thức-đợt-1--2--3--4-được-dùng-như-thế-nào)
- [9. Sau bài này bạn sẽ hiểu gì trong AI Perception](#9-sau-bài-này-bạn-sẽ-hiểu-gì-trong-ai-perception)
- [10. Cấu trúc folder](#10-cấu-trúc-folder)
- [11. Yêu cầu mini-project](#11-yêu-cầu-mini-project)
  - [11.1 Python — BaseConfigBuilder](#111-python--baseconfigbuilder)
  - [11.2 Python — ObjectFollowingConfigBuilder](#112-python--objectfollowingconfigbuilder)
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
  - [11.15 C++ — FollowDecisionConfig](#1115-c--followdecisionconfig)
  - [11.16 C++ — FollowCommand](#1116-c--followcommand)
  - [11.17 C++ — ObjectFollowingSceneResult](#1117-c--objectfollowingsceneresult)
  - [11.18 C++ — BaseObjectFollower](#1118-c--baseobjectfollower)
  - [11.19 C++ — StereoObjectFollower](#1119-c--stereoobjectfollower)
  - [11.20 C++ — ObjectFollowingReportWriter](#1120-c--objectfollowingreportwriter)
  - [11.21 C++ — main.cpp](#1121-c--maincpp)
- [12. Điều kiện bắt buộc](#12-điều-kiện-bắt-buộc)
- [13. Output mong muốn](#13-output-mong-muốn)
- [14. Vai trò của bài này trong Humanoid Robot](#14-vai-trò-của-bài-này-trong-humanoid-robot)
- [15. Checklist hoàn thành](#15-checklist-hoàn-thành)
- [16. Gợi ý mở rộng](#16-gợi-ý-mở-rộng)

---

# 1. Bài 17 lấy gì từ Đợt 4

Theo roadmap, **Đợt 4** tập trung vào:

- **Robot Perception Pipeline**
- **Object Following**
- **Pick and Place Vision**
- **Table Cleaning Robot**
- **Humanoid Robot Perception**

Nếu **Bài 16** là bài mở đầu của Đợt 4 với vai trò **pipeline supervisor**, thì **Bài 17** sẽ đi thẳng vào mảng **Object Following Vision**.

## Phần mới của Đợt 4 mà Bài 17 dùng
### Robotics Vision / Decision Layer
- chọn **target object**
- tính object đang lệch trái / phải / gần / xa
- quyết định lệnh theo dõi:
  - `TURN_LEFT`
  - `TURN_RIGHT`
  - `MOVE_FORWARD`
  - `STOP`
  - `SEARCH_TARGET`

### Python
- tiếp tục làm:
  - config builder
  - follow decision config
  - report helper

### C++
- tổ chức runtime perception + decision theo OOP
- tạo command logic dựa trên object localization trong robot frame

---

# 2. Mô tả

Ở **Bài 16**, bạn đã có một perception pipeline có thể:

- đọc stereo pair
- tính disparity map
- tính depth map
- back-project thành point cloud
- transform sang robot frame
- localize object 3D
- đưa ra decision label như:
  - `PICK_CANDIDATE`
  - `FOLLOW_TARGET`
  - `OBSERVE_ONLY`

Bài 17 sẽ **đi tiếp một bước rõ ràng hơn theo hướng robot action**:

> Nếu object là target cần theo dõi, robot sẽ sinh **follow command** dựa trên vị trí của object trong robot frame.

Mini-project này yêu cầu bạn xây một hệ thống nhỏ để robot:

- đọc stereo pair
- đọc ROI / object region config
- đọc stereo / depth / transform config
- localize object 3D trong robot frame
- xác định object mục tiêu để follow
- tính:
  - object có đủ gần hay chưa
  - object lệch trái / phải bao nhiêu
  - object có bị mất hay không
- sinh lệnh theo dõi:
  - `MOVE_FORWARD`
  - `TURN_LEFT`
  - `TURN_RIGHT`
  - `STOP`
  - `SEARCH_TARGET`
- vẽ overlay lên ảnh trái
- ghi report cho từng scene

---

# 3. Bài 17 nằm ở đâu trong roadmap

## Quy ước mini-project
- **Đợt 1 = Bài 1 → 5**
- **Đợt 2 = Bài 6 → 10**
- **Đợt 3 = Bài 11 → 15**
- **Đợt 4 = Bài 16 → 20**

Vì vậy:

## **Bài 17 = bài thứ hai của Đợt 4**
và phải **kết hợp lại kiến thức của Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.

---

# 4. Vì sao Bài 17 là bước tiếp theo hợp lý sau Bài 16

## Bài 16 cho bạn:
- pipeline perception supervisor
- object 3D localization
- target decision ở mức perception

## Bài 17 thêm:
- follow command generation
- action intent gần với robot control hơn

Tức là chuyển từ:

```text
“object này là target”
```

sang:

```text
“robot nên quay trái / quay phải / đi thẳng / dừng để theo object này”
```

Đây là đúng tinh thần **Object Following Vision** trong humanoid robot AI perception.

---

# 5. Mục tiêu perception của bài

Sau khi làm xong bài này, bạn phải hiểu được luồng:

```text
Stereo Pair + ROI/Object Config + StereoBM Config + Calibration Config + Intrinsics + Camera-to-Robot Transform + Follow Decision Config
→ Load Left / Right Image Pair
→ Compute Disparity Map
→ Convert Disparity to Depth
→ Back-Project to Camera-Frame Point Cloud
→ Transform to Robot-Frame Point Cloud
→ Localize Target Object 3D
→ Evaluate Follow Conditions
→ Generate Follow Command
→ Save Overlay + Disparity + Depth + Follow Report
```

Bài này giúp bạn hiểu một module rất quan trọng trong humanoid perception:

> **Perception không chỉ dừng ở “nhìn thấy object”, mà còn phải biến perception thành command logic phục vụ robot behavior.**

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
→ Read Follow Decision Config
→ Create Stereo Camera Sensor Object
→ For Each Stereo Pair:
    → Load Left / Right Image
    → Compute Disparity Map
    → Convert Disparity to Depth Map
    → Back-Project Valid Pixels to Camera-Frame Points
    → Transform Camera Points to Robot-Frame Points
    → Group Points by ROI / Object Region
    → Localize Target Object 3D
    → Evaluate Follow Command
    → Save Overlay + Disparity + Depth + Robot Point Cloud + Follow Report
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
- optional: Matplotlib summary

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
- object localization
- follow decision logic

---

# 8. Kiến thức Đợt 1 + 2 + 3 + 4 được dùng như thế nào

# 8.1 Đợt 1
- Python / C++ OOP cơ bản
- function
- loop / if else
- đọc ảnh cơ bản

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
- robotics vision pipeline
- target decision
- follow command logic

---

# 9. Sau bài này bạn sẽ hiểu gì trong AI Perception

Sau Bài 17, bạn phải nắm được 10 ý rất quan trọng:

## 1. Object following không cần phải bắt đầu bằng tracker phức tạp
Ngay cả với ROI / localized object 3D, bạn đã có thể tạo một **starter follow policy**.

## 2. Follow command có thể sinh từ vị trí object trong robot frame
Ví dụ:
- object lệch trái → `TURN_LEFT`
- object lệch phải → `TURN_RIGHT`
- object ở giữa nhưng còn xa → `MOVE_FORWARD`
- object đủ gần → `STOP`

## 3. Không thấy object hợp lệ thì robot cần `SEARCH_TARGET`
Đây là behavior cơ bản nhưng rất quan trọng.

## 4. Theo dõi object trong robot frame tốt hơn camera pixel thuần túy
Vì robot quan tâm khoảng cách và lệch tương đối trong không gian.

## 5. Object center / size giúp lọc target tốt hơn
Ví dụ:
- object quá nhỏ / quá xa → chưa follow
- object quá lớn / quá gần → stop

## 6. Follow logic là cầu nối perception → navigation / control
Bài này chưa làm control thật, nhưng đã chạm vào phần **command generation**.

## 7. Decision config rất quan trọng
Ví dụ:
- khoảng cách nào được xem là “đủ gần”
- lệch trái/phải bao nhiêu thì cần rẽ
- object mất bao lâu thì search

## 8. Một pipeline perception tốt phải tách phần:
- sensing
- geometry
- localization
- decision

## 9. Đây là bước đệm đẹp sang pick-and-place vision
Vì follow target cũng là một dạng **target selection + action decision**.

## 10. Đây là đúng tinh thần “Humanoid mini pipeline”
Robot không chỉ nhìn object, mà bắt đầu dùng perception để sinh hành vi.

---

# 10. Cấu trúc folder

```text
mini_project_17_robot_object_following_vision_starter/
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
│     ├─ pair_01_left_follow_overlay.jpg
│     ├─ pair_01_disparity_visualization.jpg
│     ├─ pair_01_depth_visualization.jpg
│     ├─ pair_01_robot_point_cloud.txt
│     ├─ pair_02_left_follow_overlay.jpg
│     ├─ pair_02_disparity_visualization.jpg
│     ├─ pair_02_depth_visualization.jpg
│     ├─ pair_02_robot_point_cloud.txt
│     └─ object_following_report.txt
│
├─ config/
│  ├─ stereo_pair_manifest.txt
│  ├─ roi_config.txt
│  ├─ stereo_bm_config.txt
│  ├─ stereo_calibration_config.txt
│  ├─ camera_intrinsics_config.txt
│  ├─ point_sampling_config.txt
│  ├─ camera_to_robot_transform.txt
│  └─ follow_decision_config.txt
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
   │  ├─ FollowDecisionConfig.hpp
   │  ├─ FollowCommand.hpp
   │  ├─ ObjectFollowingSceneResult.hpp
   │  ├─ BaseObjectFollower.hpp
   │  ├─ StereoObjectFollower.hpp
   │  └─ ObjectFollowingReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ StereoObjectFollower.cpp
      └─ ObjectFollowingReportWriter.cpp
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
follow_decision_config_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in toàn bộ đường dẫn config

---

# 11.2 Python — `ObjectFollowingConfigBuilder`

Tạo class con:

```python
class ObjectFollowingConfigBuilder(BaseConfigBuilder):
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
follow_decision_config
```

## `follow_decision_config`

Ví dụ:

```python
{
    "target_roi_name": "object_region",
    "min_valid_points": 100,
    "stop_distance_m": 0.45,
    "follow_distance_m": 1.2,
    "left_turn_threshold_m": -0.10,
    "right_turn_threshold_m": 0.10,
    "max_target_size_m": 0.60,
    "enable_search_if_missing": True
}
```

---

## Hàm cần có

### `add_stereo_pair(pair_name, left_image_path, right_image_path, sensor_name, sensor_id)`
- thêm stereo pair

### `add_roi_region(roi_name, x, y, width, height)`
- thêm ROI / object region

### `set_stereo_bm_config(...)`
- giống Bài 16

### `set_stereo_calibration_config(...)`
- giống Bài 16

### `set_camera_intrinsics_config(...)`
- giống Bài 16

### `set_point_sampling_config(...)`
- giống Bài 16

### `set_camera_to_robot_transform(...)`
- giống Bài 16

### `set_follow_decision_config(...)`
**Hành vi**
- lưu config follow
- kiểm tra:
  - `stop_distance_m > 0`
  - `follow_distance_m > stop_distance_m`
  - `left_turn_threshold_m < right_turn_threshold_m`

### `write_follow_decision_config()`

Format gợi ý:

```text
target_roi_name=object_region
min_valid_points=100
stop_distance_m=0.45
follow_distance_m=1.20
left_turn_threshold_m=-0.10
right_turn_threshold_m=0.10
max_target_size_m=0.60
enable_search_if_missing=1
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
  - follow decision config
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

- giống Bài 15 / 16

---

# 11.15 C++ — `FollowDecisionConfig`

**File:**

```text
cpp/include/FollowDecisionConfig.hpp
```

Tạo struct:

```cpp
struct FollowDecisionConfig
{
    std::string target_roi_name;
    int min_valid_points;

    double stop_distance_m;
    double follow_distance_m;

    double left_turn_threshold_m;
    double right_turn_threshold_m;

    double max_target_size_m;
    bool enable_search_if_missing;
};
```

---

# 11.16 C++ — `FollowCommand`

**File:**

```text
cpp/include/FollowCommand.hpp
```

Tạo struct:

```cpp
struct FollowCommand
{
    std::string pair_name;
    std::string roi_name;

    bool target_found;
    bool target_valid;

    double center_x;
    double center_y;
    double center_z;

    std::string command_label;
};
```

## `command_label` gợi ý
- `"SEARCH_TARGET"`
- `"TURN_LEFT"`
- `"TURN_RIGHT"`
- `"MOVE_FORWARD"`
- `"STOP"`

---

# 11.17 C++ — `ObjectFollowingSceneResult`

**File:**

```text
cpp/include/ObjectFollowingSceneResult.hpp
```

Tạo struct:

```cpp
struct ObjectFollowingSceneResult
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
    std::vector<FollowCommand> follow_commands;
};
```

---

# 11.18 C++ — `BaseObjectFollower`

**File:**

```text
cpp/include/BaseObjectFollower.hpp
```

Tạo abstract class:

```cpp
class BaseObjectFollower
{
public:
    virtual void load_stereo_manifest(const std::string& path) = 0;
    virtual void load_roi_config(const std::string& path) = 0;
    virtual void load_stereo_bm_config(const std::string& path) = 0;
    virtual void load_stereo_calibration_config(const std::string& path) = 0;
    virtual void load_camera_intrinsics_config(const std::string& path) = 0;
    virtual void load_point_sampling_config(const std::string& path) = 0;
    virtual void load_camera_to_robot_transform(const std::string& path) = 0;
    virtual void load_follow_decision_config(const std::string& path) = 0;
    virtual void run_object_following() = 0;
    virtual ~BaseObjectFollower() = default;
};
```

---

# 11.19 C++ — `StereoObjectFollower`

**File:**

```text
cpp/include/StereoObjectFollower.hpp
cpp/src/StereoObjectFollower.cpp
```

Class kế thừa:

```cpp
class StereoObjectFollower : public BaseObjectFollower
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
    FollowDecisionConfig follow_config;

    int pixel_step;
    double min_depth_m;
    double max_depth_m;

    std::vector<ObjectFollowingSceneResult> scene_results;
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
- `load_follow_decision_config(...)`

---

## Nhóm hàm perception core

Tái sử dụng logic từ Bài 15 / 16:

### `cv::Mat compute_disparity_map(...)`
### `cv::Mat compute_depth_map(...)`
### `Point3D back_project_pixel_to_point(...)`
### `RobotFramePoint3D transform_camera_point_to_robot_point(...)`
### `LocalizedObject3D localize_object_from_robot_points(...)`

---

## Nhóm hàm follow decision

### `bool is_target_object(const LocalizedObject3D& object) const;`
- kiểm tra `roi_name == target_roi_name`

### `bool is_object_valid_for_follow(const LocalizedObject3D& object) const;`
- đủ point
- valid
- size không quá lớn

### `std::string decide_follow_command(const LocalizedObject3D& object) const;`

## Rule gợi ý

### Nếu object invalid hoặc không thấy:
```text
SEARCH_TARGET
```

### Nếu object ở rất gần:
```text
STOP
```

### Nếu object lệch trái:
```text
TURN_LEFT
```

### Nếu object lệch phải:
```text
TURN_RIGHT
```

### Nếu object ở giữa và còn xa:
```text
MOVE_FORWARD
```

---

### `FollowCommand build_follow_command(
    const std::string& pair_name,
    const LocalizedObject3D& object
) const;`

### `void draw_follow_overlay(
    cv::Mat& left_image,
    const std::vector<LocalizedObject3D>& objects,
    const std::vector<FollowCommand>& commands
) const;`

**Hành vi**
- vẽ ROI
- ghi:
  - roi name
  - center distance
  - follow command

---

### `ObjectFollowingSceneResult process_single_stereo_pair(
    const StereoFrameRecord& record
);`

**Hành vi tổng quát**
1. đọc ảnh trái / phải
2. compute disparity
3. compute depth
4. build robot-frame point cloud
5. localize object 3D
6. chọn target object
7. build follow command
8. draw overlay
9. save outputs
10. build scene result

---

### `void run_object_following() override;`
- loop qua stereo pairs

### Getter
```cpp
const std::vector<ObjectFollowingSceneResult>& get_scene_results() const;
```

---

# 11.20 C++ — `ObjectFollowingReportWriter`

**File:**

```text
cpp/include/ObjectFollowingReportWriter.hpp
cpp/src/ObjectFollowingReportWriter.cpp
```

Tạo class:

```cpp
class ObjectFollowingReportWriter
```

## Hàm cần có

### `void write_report(
    const std::string& report_path,
    const std::vector<ObjectFollowingSceneResult>& scene_results
);`

## Format gợi ý

```text
[Object Following Scene]
Pair Name: pair_01
Valid: true

  [Localized Object]
  ROI Name: object_region
  Center: (0.22, 0.04, 0.92)
  Size: (0.14, 0.13, 0.18)

  [Follow Command]
  Target Found: true
  Target Valid: true
  Command: MOVE_FORWARD

----------------------------------------
```

---

# 11.21 C++ — `main.cpp`

## Yêu cầu
- tạo ít nhất **1 StereoCameraSensor**
- tạo `StereoObjectFollower`
- load toàn bộ config
- chạy `run_object_following()`
- tạo `ObjectFollowingReportWriter`
- ghi report ra:
  - `assets/outputs/object_following_report.txt`

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
→ Load Follow Decision Config
→ Run Object Following Pipeline
→ Save Overlay + Disparity + Depth + Robot Point Cloud
→ Write Object Following Report
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
- follow decision logic
- follow command output
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
config/follow_decision_config.txt
```

## Ảnh / point cloud output
```text
assets/outputs/pair_01_left_follow_overlay.jpg
assets/outputs/pair_01_disparity_visualization.jpg
assets/outputs/pair_01_depth_visualization.jpg
assets/outputs/pair_01_robot_point_cloud.txt
```

## File report
```text
assets/outputs/object_following_report.txt
```

---

# 14. Vai trò của bài này trong Humanoid Robot

## Python đóng vai trò gì?
Python ở đây đóng vai trò:

- tạo manifest stereo pairs
- tạo ROI config
- tạo stereo / calibration / intrinsics / transform config
- tạo follow decision config
- hỗ trợ report

Tức là Python làm phần:

```text
Object Following Pipeline Config Builder
```

---

## C++ đóng vai trò gì?
C++ là runtime chính của bài này:

- compute disparity
- compute depth
- build robot-frame point cloud
- localize target object 3D
- sinh follow command
- ghi report

Tức là C++ làm phần:

```text
Object Following Vision Runtime
```

---

## Computer Vision đóng vai trò gì?
CV ở đây đóng vai trò:

- **biến stereo pair thành disparity**
- **biến disparity thành depth**
- **biến depth thành point cloud**
- **đưa point cloud sang robot frame**
- **localize target object**
- **đưa perception thành follow command**

Tức là CV làm phần:

```text
Stereo Pair → Disparity → Depth → Point Cloud → Robot Frame → Target Localization → Follow Command
```

---

# 15. Checklist hoàn thành

- [ ] Python tạo đủ file config
- [ ] Python có class cha / class con
- [ ] C++ có `BaseObjectFollower`
- [ ] C++ có `StereoObjectFollower`
- [ ] C++ load đủ config
- [ ] C++ compute disparity
- [ ] C++ compute depth
- [ ] C++ tạo robot-frame point cloud
- [ ] C++ localize object 3D
- [ ] C++ chọn target object
- [ ] C++ sinh follow command
- [ ] C++ vẽ follow overlay
- [ ] C++ ghi follow report

---

# 16. Gợi ý mở rộng

## 1. Thêm velocity output
Ví dụ:
```text
linear_velocity
angular_velocity
```

## 2. Thêm hysteresis để tránh command đổi liên tục
Ví dụ object dao động nhẹ quanh ngưỡng trái/phải.

## 3. Thêm target loss counter
Nếu mất target liên tục nhiều frame thì chuyển hẳn sang `SEARCH_TARGET`.

## 4. Chuẩn bị cho Bài 18
Sau Bài 17, bước hợp lý cho **Bài 18** là:

```text
Robot Pick Candidate Vision Starter
```

Tức là từ object localization + follow logic, bạn bắt đầu đánh giá:
- object nào phù hợp để gắp
- object nào nằm trong workspace
- object nào có grasp-friendly size
