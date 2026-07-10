# 🤖 Bài 49: Camera-to-Robot Point Cloud Transformer — Bộ biến đổi point cloud từ camera frame sang robot frame cho Humanoid Robot AI Perception

> Mini Project số 49 trong **Đợt 10**  
> **Bài 49 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10** và đi tiếp trực tiếp từ **Bài 48**.
>
> Nếu:
>
> - **Bài 41 → 45** đi từ calibration, rectification, correspondence, block matching
> - **Bài 46** chuyển sang stereo calibration analyzer
> - **Bài 47** đánh giá depth map
> - **Bài 48** dựng point cloud trong **camera frame**
>
> thì **Bài 49** sẽ là bước rất quan trọng để đưa perception về đúng ngôn ngữ robot:
>
> ```text
> camera-frame point cloud
> → extrinsic transform
> → robot-frame point cloud
> → chia vùng trước / trái / phải / gần / xa
> → obstacle distance report
> ```
>
> Đây là bước chuyển từ:
>
> - **“camera nhìn thấy gì trong frame của camera?”**
>
> sang:
>
> - **“robot đang thấy gì trong hệ tọa độ của chính robot?”**
>
> Và đó là đúng kiểu bài toán perception mà humanoid robot cần để:
>
> - ước lượng vật cản phía trước
> - biết vật ở bên trái / phải thân robot
> - tính khoảng cách gần nhất trong vùng trung tâm
> - chuẩn bị cho navigation, locomotion, manipulation, obstacle reasoning

---

# 📌 Mục lục

- [1. Vì sao Bài 49 xuất hiện sau Bài 48](#1-vì-sao-bài-49-xuất-hiện-sau-bài-48)
- [2. Đợt 10 đang học gì](#2-đợt-10-đang-học-gì)
- [3. Bài 49 nâng từ Bài 48 lên chỗ nào](#3-bài-49-nâng-từ-bài-48-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật transform & robot-frame analysis của project](#11-luật-transform--robot-frame-analysis-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 49 xuất hiện sau Bài 48

## Bài 48 đang làm gì?
Bài 48 đã giúp bạn có:

```text
depth map
→ back-project
→ point cloud trong camera frame
→ point filtering
→ point statistics
```

Tức là bạn đã có **3D points**, nhưng các điểm này vẫn nằm trong **camera frame**.

## Nhưng còn thiếu gì?
Trong robot thật, perception thường không dừng ở camera frame.  
Robot cần trả lời câu hỏi:

> Điểm này nằm **ở đâu so với thân robot / base frame / chest frame / head frame**?

Nếu không chuyển hệ tọa độ, robot sẽ rất khó dùng point cloud để:
- tránh vật cản trước mặt
- xác định vật nằm bên trái / bên phải robot
- tính khoảng cách từ robot tới mục tiêu
- kết hợp point cloud với planning / control / locomotion

## Bài 49 lấp đúng chỗ đó
Bài 49 sẽ làm:

```text
camera-frame point cloud
→ rigid transform / extrinsic transform
→ robot-frame point cloud
→ robot-centric spatial analysis
→ obstacle report
```

Đây là bước chuyển từ **3D geometry trong camera** sang **3D geometry mà robot có thể trực tiếp sử dụng**.

---

# 2. Đợt 10 đang học gì

Theo roadmap bạn gửi, **Đợt 10** xoay quanh:

## Python
### **Phase 9 — NumPy**
- reshape
- matrix multiplication
- robotics vector basics

## C++
- class
- inheritance
- virtual function
- vector
- pointer / reference

## Computer Vision
### **Phase 5 — Stereo Vision & Depth**
- Stereo Camera
- Disparity

## Vì sao Bài 49 vẫn bám đúng Đợt 10?
Vì Bài 49 tận dụng trực tiếp:
- **matrix multiplication** để làm transform 3D
- **robotics vector basics** cho điểm và pose
- stereo depth / point cloud của Bài 48
- OOP C++ để tổ chức transform pipeline

Nó đồng thời là bước cầu nối rất đẹp sang các đợt sau về robot-centric perception.

---

# 3. Bài 49 nâng từ Bài 48 lên chỗ nào

## Bài 48
- point cloud trong **camera frame**
- thống kê XYZ trong hệ camera

## Bài 49
- point cloud trong **robot frame**
- phân tích theo vùng có nghĩa với robot:
  - phía trước
  - bên trái
  - bên phải
  - gần / xa
  - vùng trung tâm

### Nói ngắn gọn:
- **Bài 48** hỏi: “Camera thấy những điểm 3D nào?”
- **Bài 49** hỏi: “Robot thấy vật cản ở đâu so với chính nó?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Camera-to-Robot Point Cloud Transformer**

System này nhận đầu vào là:
- một hoặc nhiều **camera-frame point clouds**
- một hoặc nhiều **extrinsic transforms**
- một số **robot-centric analysis rules**

## Mỗi input point cloud có thể đến từ:
- Bài 48 xuất ra
- hoặc synthetic point cloud sample bạn tự tạo

## Mỗi transform sẽ mô tả:
- `camera_frame_name`
- `robot_frame_name`
- rotation
- translation

Bạn có thể biểu diễn transform theo 1 trong 2 cách:

## Cách A — rotation matrix + translation
```text
R (3x3)
t (3x1)
```

## Cách B — homogeneous transform
```text
T (4x4)
```

## Nhiệm vụ của system
### Với mỗi point cloud:
1. đọc camera-frame points
2. đọc transform camera → robot
3. biến đổi toàn bộ điểm sang robot frame
4. phân loại điểm theo vùng:
   - front
   - left
   - right
   - near
   - far
   - center corridor
5. tính:
   - khoảng cách gần nhất phía trước
   - số điểm trong vùng trung tâm
   - khoảng Z / X / Y trong robot frame
6. ghi robot-centric report

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build camera-frame point cloud config
→ build extrinsic transform config
→ preview transform bằng NumPy
→ preview robot-frame point cloud
→ preview obstacle region summary

C++
→ load point clouds
→ load camera-to-robot transform
→ transform all points to robot frame
→ compute robot-centric spatial summaries
→ export obstacle / region reports
```

Mục tiêu cốt lõi:
- hiểu point cloud phải được đưa về **robot frame** thì robot mới dùng được
- hiểu rigid transform / homogeneous transform trong perception
- hiểu robot-centric vùng trước / trái / phải / trung tâm là gì
- chuẩn bị nền cho:
  - obstacle detection
  - local mapping
  - walking safety
  - manipulation target localization

---

# 6. Pipeline tổng thể

```text
Load Camera-Frame Point Cloud Config
Load Camera-to-Robot Transform Config
Load Robot Analysis Runtime Config

Create PointCloudTransformEngine
Create RobotRegionAnalysisEngine
Create ObstacleDistanceEngine
Create CameraToRobotPointCloudTransformer

For each point cloud sample:
    1. load camera-frame points
    2. load transform camera -> robot
    3. transform all points to robot frame
    4. analyze robot-centric regions
    5. compute nearest obstacle distances
    6. save result

After all inputs:
    write reports
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- BST / Graph / NumPy từ các đợt trước
- matrix multiplication
- homogeneous coordinates
- robotics vector basics

## C++
- class / inheritance
- virtual functions / polymorphism
- vector
- pointer / reference
- report-oriented runtime design

## Computer Vision / Robotics Geometry
- point cloud
- rigid transformation
- homogeneous transformation
- camera frame
- robot frame
- spatial region reasoning

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **camera-to-robot perception pipeline**:

```text
CameraPointCloud
→ ExtrinsicTransform
→ RobotFramePointCloud
→ RegionClassification
→ ObstacleDistance
→ RobotCentricReport
```

## 2. Python BST
Lưu `cloud_id`, `transform_id`, `robot_frame_id`.

## 3. `std::vector<PointCloudSample>`
Danh sách camera-frame point clouds.

## 4. `std::vector<ExtrinsicTransform>`
Danh sách transform camera → robot.

## 5. `std::vector<Point3D>`
Danh sách điểm 3D.

## 6. `std::vector<RobotFramePointCloudResult>`
Kết quả point cloud trong robot frame.

## 7. `std::vector<RobotRegionStatistic>`
Thống kê vùng robot-centric.

## 8. `std::stack<TransformDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Point transform
Với point camera-frame:

```text
p_cam = [Xc, Yc, Zc]^T
p_robot = R * p_cam + t
```

hoặc dùng homogeneous:

```text
p_robot_h = T_robot_from_camera * p_cam_h
```

---

## Algorithm 2 — Robot-centric region classification
Bạn phải định nghĩa ít nhất các vùng:

### `FRONT`
điểm nằm phía trước robot

### `LEFT`
điểm nằm bên trái robot

### `RIGHT`
điểm nằm bên phải robot

### `CENTER_CORRIDOR`
điểm nằm gần trục đi thẳng của robot

Bạn có thể tự chọn quy ước trục, nhưng phải ghi rõ trong README / report.

---

## Algorithm 3 — Nearest obstacle distance
Tìm ít nhất:
- điểm gần nhất trong vùng **front**
- điểm gần nhất trong **center corridor**

---

## Algorithm 4 — Region statistics
Tính cho từng vùng:
- point count
- average distance
- min distance
- max distance

---

## Algorithm 5 — Optional transform validation
Bạn nên có kiểm tra:
- ma trận rotation hợp lệ
- kích thước transform đúng
- cloud rỗng / không rỗng

---

# 9. Cấu trúc folder

```text
mini_project_49_camera_to_robot_point_cloud_transformer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ robot_frame_point_cloud_report.txt
│     ├─ robot_region_statistics_report.txt
│     ├─ obstacle_distance_report.txt
│     ├─ transform_validation_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ point_cloud_config.txt
│  ├─ extrinsic_transform_config.txt
│  ├─ robot_analysis_runtime_config.txt
│  └─ region_rule_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ robot_transform_graph_preview.py
│     ├─ cloud_bst.py
│     ├─ numpy_robot_transform_preview.py
│     └─ synthetic_cloud_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ PointCloudSample.hpp
   │  ├─ ExtrinsicTransform.hpp
   │  ├─ RobotFramePointCloudResult.hpp
   │  ├─ RobotRegionStatistic.hpp
   │  ├─ TransformDebugRecord.hpp
   │  ├─ BasePointCloudTransformEngine.hpp
   │  ├─ CameraToRobotTransformEngine.hpp
   │  ├─ BaseRobotRegionAnalysisEngine.hpp
   │  ├─ RobotRegionAnalysisEngine.hpp
   │  ├─ BaseObstacleDistanceEngine.hpp
   │  ├─ ObstacleDistanceEngine.hpp
   │  ├─ CameraToRobotPointCloudTransformer.hpp
   │  └─ RobotPointCloudReportWriter.hpp
   │
   └─ src/
      ├─ CameraToRobotTransformEngine.cpp
      ├─ RobotRegionAnalysisEngine.cpp
      ├─ ObstacleDistanceEngine.cpp
      ├─ CameraToRobotPointCloudTransformer.cpp
      └─ RobotPointCloudReportWriter.cpp
```

---

# 10. Yêu cầu mini-project

# 10.1 Python — `BaseConfigBuilder`

Tạo class:

```python
class BaseConfigBuilder:
```

## Thuộc tính
```python
project_name
cloud_config_path
transform_config_path
runtime_config_path
region_rule_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `CameraToRobotTransformerConfigBuilder`

Tạo class con:

```python
class CameraToRobotTransformerConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_point_cloud_sample(
    cloud_id,
    points_xyz,
    camera_frame_name,
    target_label=None
)`

### `add_extrinsic_transform(
    transform_id,
    camera_frame_name,
    robot_frame_name,
    rotation_matrix,
    translation_vector
)`

### `set_region_rules(
    front_min_z,
    left_max_x,
    right_min_x,
    corridor_abs_x_max,
    near_distance_threshold
)`

### `write_cloud_config()`
### `write_transform_config()`
### `write_runtime_config()`
### `write_region_rule_config()`

---

# 10.3 Python — `robot_transform_graph_preview.py`

Tạo class:

```python
class RobotTransformGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
CameraPointCloud
→ CameraToRobotTransform
→ RobotFramePointCloud
→ RegionClassification
→ ObstacleDistance
→ Report
```

---

# 10.4 Python — `cloud_bst.py`

Tạo BST cho `cloud_id` / `transform_id`.

---

# 10.5 Python — `numpy_robot_transform_preview.py`

Tạo class:

```python
class NumPyRobotTransformPreview:
```

## Hàm cần có

### `transform_points(points_xyz, rotation_matrix, translation_vector)`
### `build_homogeneous_transform(rotation_matrix, translation_vector)`
### `classify_regions(points_xyz_robot, corridor_abs_x_max, near_distance_threshold)`
### `compute_nearest_front_distance(points_xyz_robot)`

---

# 10.6 C++ — `Point3D`

```cpp
struct Point3D
{
    double x;
    double y;
    double z;
};
```

---

# 10.7 C++ — `PointCloudSample`

```cpp
struct PointCloudSample
{
    std::string cloud_id;
    std::string camera_frame_name;
    std::string target_label;

    std::vector<Point3D> points;
};
```

---

# 10.8 C++ — `ExtrinsicTransform`

```cpp
struct ExtrinsicTransform
{
    std::string transform_id;
    std::string camera_frame_name;
    std::string robot_frame_name;

    std::vector<std::vector<double>> rotation_matrix;   // 3x3
    std::vector<double> translation_vector;             // 3x1
};
```

---

# 10.9 C++ — `RobotFramePointCloudResult`

```cpp
struct RobotFramePointCloudResult
{
    std::string cloud_id;
    std::string robot_frame_name;

    std::vector<Point3D> robot_points;
};
```

---

# 10.10 C++ — `RobotRegionStatistic`

```cpp
enum class RobotRegionType
{
    FRONT,
    LEFT,
    RIGHT,
    CENTER_CORRIDOR
};

struct RobotRegionStatistic
{
    std::string cloud_id;
    RobotRegionType region_type;

    int point_count;
    double min_distance;
    double max_distance;
    double average_distance;
};
```

---

# 10.11 C++ — `TransformDebugRecord`

```cpp
struct TransformDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.12 C++ — `BasePointCloudTransformEngine`

Tạo abstract class:

```cpp
class BasePointCloudTransformEngine
{
public:
    virtual RobotFramePointCloudResult transform_cloud(
        const PointCloudSample& cloud,
        const ExtrinsicTransform& transform
    ) const = 0;

    virtual ~BasePointCloudTransformEngine() = default;
};
```

---

# 10.13 C++ — `CameraToRobotTransformEngine`

Kế thừa `BasePointCloudTransformEngine`.

## Nhiệm vụ
- áp dụng transform cho toàn bộ điểm camera-frame
- tạo robot-frame point cloud

---

# 10.14 C++ — `BaseRobotRegionAnalysisEngine`

Tạo abstract class:

```cpp
class BaseRobotRegionAnalysisEngine
{
public:
    virtual std::vector<RobotRegionStatistic> analyze_regions(
        const RobotFramePointCloudResult& cloud
    ) const = 0;

    virtual ~BaseRobotRegionAnalysisEngine() = default;
};
```

---

# 10.15 C++ — `RobotRegionAnalysisEngine`

Kế thừa `BaseRobotRegionAnalysisEngine`.

## Nhiệm vụ
- chia vùng FRONT / LEFT / RIGHT / CENTER_CORRIDOR
- tính thống kê từng vùng

---

# 10.16 C++ — `BaseObstacleDistanceEngine`

Tạo abstract class:

```cpp
class BaseObstacleDistanceEngine
{
public:
    virtual std::unordered_map<std::string, double> compute_obstacle_distances(
        const RobotFramePointCloudResult& cloud
    ) const = 0;

    virtual ~BaseObstacleDistanceEngine() = default;
};
```

---

# 10.17 C++ — `ObstacleDistanceEngine`

Kế thừa `BaseObstacleDistanceEngine`.

## Nhiệm vụ
Tính ít nhất:
- nearest front obstacle
- nearest center-corridor obstacle

---

# 10.18 C++ — `CameraToRobotPointCloudTransformer`

Tạo class trung tâm:

```cpp
class CameraToRobotPointCloudTransformer
```

## Thuộc tính

```cpp
private:
    std::vector<PointCloudSample> clouds;
    std::vector<ExtrinsicTransform> transforms;

    std::shared_ptr<BasePointCloudTransformEngine> transform_engine;
    std::shared_ptr<BaseRobotRegionAnalysisEngine> region_engine;
    std::shared_ptr<BaseObstacleDistanceEngine> obstacle_engine;

    std::vector<RobotFramePointCloudResult> results;
    std::vector<RobotRegionStatistic> region_statistics;
    std::stack<TransformDebugRecord> debug_history;
```

## Hàm cần có

### `load_cloud_config(const std::string& path)`
### `load_transform_config(const std::string& path)`
### `load_runtime_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each point cloud:
    find corresponding transform
    robot_cloud = transform_engine.transform_cloud(cloud, transform)

    region_stats = region_engine.analyze_regions(robot_cloud)
    obstacle_info = obstacle_engine.compute_obstacle_distances(robot_cloud)

    save result
    save region_stats
    push debug history
```

### `const std::vector<RobotFramePointCloudResult>& get_results() const`
### `const std::vector<RobotRegionStatistic>& get_region_statistics() const`
### `std::vector<TransformDebugRecord> get_debug_history_reverse()`

---

# 10.19 C++ — `RobotPointCloudReportWriter`

Tạo class:

```cpp
class RobotPointCloudReportWriter
```

## Hàm cần có

### `write_robot_frame_point_cloud_report(...)`
Ví dụ:

```text
[Robot Frame Point Cloud]
Cloud: head_cloud_01
Robot Frame: base_link
Point Count: 120
```

### `write_robot_region_statistics_report(...)`
Ví dụ:

```text
[Robot Region Statistics]
Cloud: head_cloud_01
Region: FRONT
Point Count: 52
Average Distance: 1.48
```

### `write_obstacle_distance_report(...)`
Ví dụ:

```text
[Obstacle Distance]
Cloud: head_cloud_01
Nearest Front Obstacle: 0.92
Nearest Center Corridor Obstacle: 1.10
```

### `write_transform_validation_report(...)`
Ví dụ:

```text
[Transform Validation]
Transform: head_cam_to_base
Rotation matrix size: 3x3
Translation vector size: 3
```

### `write_reverse_debug_history(...)`

---

# 10.20 C++ — `main.cpp`

## Yêu cầu

```text
Load point cloud config
Load transform config
Load runtime config
Load region rule config

Create:
    CameraToRobotTransformEngine
    RobotRegionAnalysisEngine
    ObstacleDistanceEngine

Create CameraToRobotPointCloudTransformer
Run
Write reports
```

---

# 11. Luật transform & robot-frame analysis của project

## Luật 1 — Transform phải đúng camera frame → robot frame
Không được áp nhầm chiều nếu config đang mô tả `robot_from_camera`.

## Luật 2 — Region rules phải được ghi rõ theo quy ước trục
Ví dụ:
- `+Z` là phía trước
- `+X` là bên phải
- `-X` là bên trái

hoặc một quy ước khác, nhưng phải nhất quán.

## Luật 3 — Khoảng cách obstacle phải tính trên robot-frame cloud
Không được lấy cloud camera-frame đi tính obstacle report.

## Luật 4 — Nếu point cloud rỗng sau transform thì report phải ghi rõ
Không được âm thầm bỏ qua.

## Luật 5 — Ít nhất phải có FRONT và CENTER_CORRIDOR obstacle distance
Vì đây là hai vùng rất sát với bài toán robot di chuyển phía trước.

---

# 12. Output mong muốn

## Config
```text
config/point_cloud_config.txt
config/extrinsic_transform_config.txt
config/robot_analysis_runtime_config.txt
config/region_rule_config.txt
```

## Reports
```text
assets/outputs/robot_frame_point_cloud_report.txt
assets/outputs/robot_region_statistics_report.txt
assets/outputs/obstacle_distance_report.txt
assets/outputs/transform_validation_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build cloud + transform config
- preview transform bằng NumPy
- test robot-centric vùng trước / trái / phải

## C++
- transform point cloud sang robot frame
- phân tích vùng có nghĩa với robot
- tính khoảng cách obstacle
- tạo robot-centric perception report

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó biến:
- **camera perception**
thành
- **robot-centric perception**

Đó là một bước bắt buộc nếu sau này bạn muốn làm:
- obstacle avoidance
- navigation
- stepping safety
- manipulation target localization
- local 3D scene understanding cho humanoid

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho cloud / transform / region rules
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho cloud ids / transform ids
- [ ] Python có NumPy preview cho transform point cloud
- [ ] C++ có `PointCloudSample`
- [ ] C++ có `ExtrinsicTransform`
- [ ] C++ có `RobotFramePointCloudResult`
- [ ] C++ có `RobotRegionStatistic`
- [ ] C++ có `CameraToRobotTransformEngine`
- [ ] C++ có `RobotRegionAnalysisEngine`
- [ ] C++ có `ObstacleDistanceEngine`
- [ ] C++ có `CameraToRobotPointCloudTransformer`
- [ ] C++ ghi đủ 5 report

---

# 15. Gợi ý mở rộng

## 1. Thêm nhiều camera cùng lúc
Ví dụ:
- head stereo
- chest depth camera
- wrist camera

rồi đưa tất cả về cùng `base_link`.

## 2. Thêm transform chain
Ví dụ:
```text
camera → head_link → torso_link → base_link
```

thay vì chỉ 1 transform trực tiếp.

## 3. Thêm obstacle zone scoring
Ví dụ chấm điểm mức nguy hiểm của vật cản ở:
- center corridor
- left foot zone
- right foot zone

## 4. Chuẩn bị cho Bài 50
Sau Bài 49, hướng đi rất đẹp là:

# **Bài 50: Robot-Centric Obstacle Distance Reasoning Lab**

Ý tưởng:
```text
robot-frame point cloud
→ chia zone nguy hiểm
→ nearest obstacle / average clearance
→ obstacle risk summary
→ walking safety report
```

Tức là:
- **Bài 48**: camera-frame point cloud
- **Bài 49**: robot-frame point cloud
- **Bài 50**: obstacle reasoning thật sự trong robot frame
