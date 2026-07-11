# 🤖 Bài 50: Robot-Centric Obstacle Distance Reasoning Lab — Bộ suy luận khoảng cách vật cản theo góc nhìn của robot cho Humanoid Robot AI Perception

> Mini Project số 50 trong **Đợt 10**  
> **Bài 50 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9 + 10** và đi tiếp trực tiếp từ **Bài 49**.
>
> Nếu:
>
> - **Bài 41 → 45** đi từ calibration, rectification, correspondence, block matching
> - **Bài 46** chuyển sang stereo calibration analyzer
> - **Bài 47** đánh giá depth map
> - **Bài 48** dựng point cloud trong camera frame
> - **Bài 49** biến đổi point cloud sang robot frame
>
> thì **Bài 50** sẽ là bước perception cực kỳ quan trọng để robot bắt đầu **ra nhận định về vật cản**:
>
> ```text
> robot-frame point cloud
> → chia vùng nguy hiểm / vùng quan sát
> → tính nearest obstacle / average clearance
> → đánh giá mức độ cản trở phía trước
> → sinh obstacle risk report cho robot
> ```
>
> Đây là bước chuyển từ:
>
> - **“robot đã có point cloud trong robot frame”**
>
> sang:
>
> - **“robot hiểu vùng nào đang nguy hiểm, vật cản gần nhất ở đâu, hành lang đi thẳng có bị chặn hay không”**
>
> Và đó chính là loại đầu ra rất sát với các bài toán:
>
> - locomotion safety
> - walking corridor checking
> - local obstacle awareness
> - navigation pre-check
> - reactive perception cho humanoid robot

---

# 📌 Mục lục

- [1. Vì sao Bài 50 xuất hiện sau Bài 49](#1-vì-sao-bài-50-xuất-hiện-sau-bài-49)
- [2. Đợt 10 đang học gì](#2-đợt-10-đang-học-gì)
- [3. Bài 50 nâng từ Bài 49 lên chỗ nào](#3-bài-50-nâng-từ-bài-49-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật obstacle reasoning của project](#11-luật-obstacle-reasoning-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 50 xuất hiện sau Bài 49

## Bài 49 đang làm gì?
Bài 49 đã giúp bạn có:

```text
camera-frame point cloud
→ camera-to-robot transform
→ robot-frame point cloud
→ robot-centric region statistics
```

Tức là bạn đã đưa dữ liệu perception về đúng **hệ tọa độ robot**.

## Nhưng còn thiếu gì?
Robot không chỉ cần “có point cloud trong robot frame”.  
Robot cần một bước suy luận cao hơn:

> Trong point cloud đó, **vật cản gần nhất ở đâu?**  
> **Hành lang phía trước có đang bị chặn không?**  
> **Bên trái hay bên phải nguy hiểm hơn?**  
> **Có đủ khoảng trống để bước tiếp không?**

Đây là lúc perception bắt đầu tạo ra **tín hiệu hữu ích cho navigation / locomotion / safety**.

## Bài 50 lấp đúng chỗ đó
Bài 50 sẽ làm:

```text
robot-frame point cloud
→ chia zone nguy hiểm
→ tính nearest obstacle / clearance
→ chấm mức nguy cơ từng vùng
→ sinh walking-safety / obstacle report
```

Đây là bước chuyển từ **robot-frame geometry** sang **robot-centric obstacle reasoning**.

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

## Vì sao Bài 50 vẫn bám đúng Đợt 10?
Vì Bài 50 vẫn bám hoàn toàn vào chuỗi stereo-depth-point-cloud của Đợt 10:
- disparity / depth / point cloud đến từ các bài 46–49
- robot-centric vector reasoning
- NumPy cho region masks / distance arrays
- C++ OOP cho obstacle analysis runtime

Nó đồng thời là bài “khóa sổ” rất đẹp cho Đợt 10, vì sau bài này bạn đã đi trọn chuỗi:

```text
Stereo → Disparity → Depth → Point Cloud → Robot Frame → Obstacle Reasoning
```

---

# 3. Bài 50 nâng từ Bài 49 lên chỗ nào

## Bài 49
- robot-frame point cloud
- transform và region statistics cơ bản

## Bài 50
- obstacle reasoning thực sự:
  - nearest obstacle
  - average clearance
  - front corridor blockage
  - zone risk scoring

### Nói ngắn gọn:
- **Bài 49** hỏi: “Các điểm 3D nằm ở đâu trong robot frame?”
- **Bài 50** hỏi: “Những điểm đó nói gì về mức an toàn di chuyển của robot?”

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Robot-Centric Obstacle Distance Reasoning Lab**

System này nhận đầu vào là:
- một hoặc nhiều **robot-frame point clouds**
- một bộ **zone rules** để chia vùng trước / trái / phải / trung tâm / gần / xa
- một bộ **safety thresholds** để đánh giá nguy cơ

## Ví dụ robot-frame point cloud
```text
[
  (0.10, 0.00, 0.80),
  (0.25, 0.02, 1.10),
  (-0.18, 0.01, 0.95),
  (0.05, -0.03, 0.55),
  (0.32, 0.00, 2.40)
]
```

> Bạn có thể tự quy ước trục. Ví dụ:
>
> - `+Z`: phía trước robot
> - `+X`: bên phải robot
> - `-X`: bên trái robot
> - `Y`: trục đứng / cao thấp
>
> Chỉ cần **nhất quán** trong toàn project.

## Nhiệm vụ của system
### Với mỗi robot-frame point cloud:
1. chia point cloud thành các vùng:
   - `FRONT`
   - `LEFT`
   - `RIGHT`
   - `CENTER_CORRIDOR`
   - `NEAR_ZONE`
   - `FAR_ZONE`
2. tính:
   - nearest obstacle ở phía trước
   - nearest obstacle trong center corridor
   - average clearance theo từng vùng
   - số điểm nguy hiểm dưới ngưỡng an toàn
3. đánh giá mức nguy cơ:
   - `SAFE`
   - `CAUTION`
   - `DANGER`
4. ghi obstacle reasoning report

<p align="center">
  <img src="../../images/project_50.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build robot-frame point cloud config
→ build zone rule config
→ preview region masks bằng NumPy
→ preview nearest obstacle / clearance
→ preview risk labels

C++
→ load robot-frame point clouds
→ classify obstacle zones
→ compute obstacle distances
→ compute clearance / density / risk
→ export robot-centric obstacle reports
```

Mục tiêu cốt lõi:
- hiểu point cloud chỉ thật sự hữu ích khi được chuyển thành **thông tin an toàn / nguy cơ**
- hiểu robot-centric reasoning là cầu nối giữa perception và action
- hiểu cách chia vùng phía trước / corridor / trái / phải để phục vụ locomotion
- chuẩn bị nền cho:
  - walking safety
  - reactive obstacle avoidance
  - local path feasibility checks
  - footstep / navigation perception

---

# 6. Pipeline tổng thể

```text
Load Robot-Frame Point Cloud Config
Load Zone Rule Config
Load Safety Threshold Config
Load Runtime Config

Create ObstacleZoneClassifier
Create ClearanceComputationEngine
Create RiskScoringEngine
Create RobotObstacleReasoningLab

For each robot-frame point cloud:
    1. classify points into robot-centric zones
    2. compute nearest obstacle distance for each zone
    3. compute average clearance / density / count
    4. assign zone risk levels
    5. assign overall cloud risk summary
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
- boolean masking
- vectorized distance computation
- robotics vector basics

## C++
- class / inheritance
- virtual functions / polymorphism
- vector
- unordered_map
- pointer / reference
- report-oriented runtime design

## Robotics Perception / Geometry
- robot-frame point cloud
- obstacle distance
- clearance
- corridor reasoning
- risk thresholding

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Biểu diễn **obstacle reasoning pipeline**:

```text
RobotFramePointCloud
→ ZoneClassification
→ ObstacleDistance
→ ClearanceAnalysis
→ RiskScoring
→ SafetyReport
```

## 2. Python BST
Lưu `cloud_id`, `zone_rule_id`, `risk_profile_id`.

## 3. `std::vector<PointCloudSample>`
Danh sách robot-frame point clouds.

## 4. `std::vector<ObstacleZoneRule>`
Rule chia vùng obstacle.

## 5. `std::vector<ZoneObstacleStatistic>`
Thống kê theo vùng.

## 6. `std::vector<CloudRiskResult>`
Kết quả risk của từng cloud.

## 7. `std::stack<ObstacleDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Zone classification
Bạn phải chia ít nhất các vùng:

### `FRONT`
điểm ở phía trước robot

### `LEFT`
điểm nằm lệch bên trái

### `RIGHT`
điểm nằm lệch bên phải

### `CENTER_CORRIDOR`
điểm nằm trong hành lang đi thẳng của robot

### `NEAR_ZONE`
điểm có khoảng cách dưới ngưỡng gần

### `FAR_ZONE`
điểm còn lại / xa hơn ngưỡng

---

## Algorithm 2 — Distance computation
Với mỗi điểm, tính ít nhất:
- khoảng cách Euclidean tới robot
- hoặc forward distance theo trục tiến

Từ đó suy ra:
- nearest front obstacle
- nearest center-corridor obstacle
- nearest left / right obstacle

---

## Algorithm 3 — Clearance computation
Cho mỗi vùng, tính:
- `min_distance`
- `max_distance`
- `average_distance`

Bạn có thể định nghĩa **clearance** là:
- khoảng cách nhỏ nhất đến vật cản trong vùng
- hoặc trung bình của top-k điểm gần nhất

Nhưng phải ghi rõ cách định nghĩa.

---

## Algorithm 4 — Risk scoring
Bạn phải có ít nhất 3 mức:

### `SAFE`
khoảng cách còn rộng / ít điểm cản

### `CAUTION`
đã có vật cản tương đối gần

### `DANGER`
vật cản quá gần hoặc mật độ cản cao ở vùng trung tâm

---

## Algorithm 5 — Overall obstacle summary
Sau khi có risk từng vùng, sinh kết luận cho cả cloud:
- cloud an toàn hay không
- vùng nào nguy hiểm nhất
- nearest blocking region là gì

---

# 9. Cấu trúc folder

```text
mini_project_50_robot_centric_obstacle_distance_reasoning_lab/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ zone_obstacle_statistics_report.txt
│     ├─ clearance_report.txt
│     ├─ obstacle_risk_report.txt
│     ├─ walking_safety_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ robot_point_cloud_config.txt
│  ├─ zone_rule_config.txt
│  ├─ safety_threshold_config.txt
│  └─ runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ obstacle_reasoning_graph_preview.py
│     ├─ obstacle_bst.py
│     ├─ numpy_obstacle_reasoning_preview.py
│     └─ synthetic_robot_cloud_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ PointCloudSample.hpp
   │  ├─ ObstacleZoneRule.hpp
   │  ├─ ZoneObstacleStatistic.hpp
   │  ├─ CloudRiskResult.hpp
   │  ├─ ObstacleDebugRecord.hpp
   │  ├─ BaseObstacleZoneClassifier.hpp
   │  ├─ ObstacleZoneClassifier.hpp
   │  ├─ BaseClearanceComputationEngine.hpp
   │  ├─ ClearanceComputationEngine.hpp
   │  ├─ BaseRiskScoringEngine.hpp
   │  ├─ RiskScoringEngine.hpp
   │  ├─ RobotObstacleReasoningLab.hpp
   │  └─ RobotObstacleReportWriter.hpp
   │
   └─ src/
      ├─ ObstacleZoneClassifier.cpp
      ├─ ClearanceComputationEngine.cpp
      ├─ RiskScoringEngine.cpp
      ├─ RobotObstacleReasoningLab.cpp
      └─ RobotObstacleReportWriter.cpp
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
zone_rule_config_path
threshold_config_path
runtime_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `RobotObstacleReasoningConfigBuilder`

Tạo class con:

```python
class RobotObstacleReasoningConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `add_robot_frame_cloud(
    cloud_id,
    points_xyz,
    target_label=None
)`

### `set_zone_rules(
    front_min_z,
    corridor_abs_x_max,
    left_x_max,
    right_x_min,
    near_distance_threshold
)`

### `set_safety_thresholds(
    danger_distance,
    caution_distance,
    max_safe_center_points
)`

### `write_cloud_config()`
### `write_zone_rule_config()`
### `write_threshold_config()`
### `write_runtime_config()`

---

# 10.3 Python — `obstacle_reasoning_graph_preview.py`

Tạo class:

```python
class ObstacleReasoningGraphPreview:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
RobotFramePointCloud
→ ZoneClassification
→ ObstacleDistance
→ ClearanceAnalysis
→ RiskScoring
→ WalkingSafetyReport
```

---

# 10.4 Python — `obstacle_bst.py`

Tạo BST cho `cloud_id` / `rule_id`.

---

# 10.5 Python — `numpy_obstacle_reasoning_preview.py`

Tạo class:

```python
class NumPyObstacleReasoningPreview:
```

## Hàm cần có

### `classify_zones(points_xyz, corridor_abs_x_max, front_min_z, near_distance_threshold)`
### `compute_point_distances(points_xyz)`
### `compute_zone_statistics(points_xyz, zone_labels)`
### `score_risk(zone_statistics, danger_distance, caution_distance)`

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
    std::string target_label;
    std::vector<Point3D> points;
};
```

---

# 10.8 C++ — `ObstacleZoneRule`

```cpp
struct ObstacleZoneRule
{
    double front_min_z;
    double corridor_abs_x_max;
    double left_x_max;
    double right_x_min;
    double near_distance_threshold;
};
```

---

# 10.9 C++ — `ZoneObstacleStatistic`

```cpp
enum class ObstacleZoneType
{
    FRONT,
    LEFT,
    RIGHT,
    CENTER_CORRIDOR,
    NEAR_ZONE,
    FAR_ZONE
};

struct ZoneObstacleStatistic
{
    std::string cloud_id;
    ObstacleZoneType zone_type;

    int point_count;
    double min_distance;
    double max_distance;
    double average_distance;
};
```

---

# 10.10 C++ — `CloudRiskResult`

```cpp
enum class RiskLevel
{
    SAFE,
    CAUTION,
    DANGER
};

struct CloudRiskResult
{
    std::string cloud_id;

    RiskLevel overall_risk;
    std::string most_dangerous_zone;

    double nearest_front_distance;
    double nearest_center_distance;
};
```

---

# 10.11 C++ — `ObstacleDebugRecord`

```cpp
struct ObstacleDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.12 C++ — `BaseObstacleZoneClassifier`

Tạo abstract class:

```cpp
class BaseObstacleZoneClassifier
{
public:
    virtual std::unordered_map<ObstacleZoneType, std::vector<Point3D>> classify(
        const PointCloudSample& cloud,
        const ObstacleZoneRule& rule
    ) const = 0;

    virtual ~BaseObstacleZoneClassifier() = default;
};
```

---

# 10.13 C++ — `ObstacleZoneClassifier`

Kế thừa `BaseObstacleZoneClassifier`.

## Nhiệm vụ
- chia point cloud vào các vùng FRONT / LEFT / RIGHT / CENTER_CORRIDOR / NEAR / FAR

---

# 10.14 C++ — `BaseClearanceComputationEngine`

Tạo abstract class:

```cpp
class BaseClearanceComputationEngine
{
public:
    virtual std::vector<ZoneObstacleStatistic> compute_zone_statistics(
        const std::string& cloud_id,
        const std::unordered_map<ObstacleZoneType, std::vector<Point3D>>& zone_points
    ) const = 0;

    virtual ~BaseClearanceComputationEngine() = default;
};
```

---

# 10.15 C++ — `ClearanceComputationEngine`

Kế thừa `BaseClearanceComputationEngine`.

## Nhiệm vụ
- tính min / max / average distance cho từng vùng
- xác định nearest point của từng vùng

---

# 10.16 C++ — `BaseRiskScoringEngine`

Tạo abstract class:

```cpp
class BaseRiskScoringEngine
{
public:
    virtual CloudRiskResult score(
        const std::string& cloud_id,
        const std::vector<ZoneObstacleStatistic>& zone_stats
    ) const = 0;

    virtual ~BaseRiskScoringEngine() = default;
};
```

---

# 10.17 C++ — `RiskScoringEngine`

Kế thừa `BaseRiskScoringEngine`.

## Nhiệm vụ
- chấm `SAFE / CAUTION / DANGER`
- chọn `most_dangerous_zone`
- ghi nearest front / center distance

---

# 10.18 C++ — `RobotObstacleReasoningLab`

Tạo class trung tâm:

```cpp
class RobotObstacleReasoningLab
```

## Thuộc tính

```cpp
private:
    std::vector<PointCloudSample> clouds;
    ObstacleZoneRule zone_rule;

    std::shared_ptr<BaseObstacleZoneClassifier> classifier;
    std::shared_ptr<BaseClearanceComputationEngine> clearance_engine;
    std::shared_ptr<BaseRiskScoringEngine> risk_engine;

    std::vector<ZoneObstacleStatistic> zone_statistics;
    std::vector<CloudRiskResult> risk_results;
    std::stack<ObstacleDebugRecord> debug_history;
```

## Hàm cần có

### `load_cloud_config(const std::string& path)`
### `load_zone_rule_config(const std::string& path)`
### `load_threshold_config(const std::string& path)`

### `run()`
Pseudo:

```text
for each cloud:
    zone_points = classifier.classify(cloud, zone_rule)
    stats = clearance_engine.compute_zone_statistics(cloud.cloud_id, zone_points)
    risk = risk_engine.score(cloud.cloud_id, stats)

    save stats
    save risk
    push debug history
```

### `const std::vector<ZoneObstacleStatistic>& get_zone_statistics() const`
### `const std::vector<CloudRiskResult>& get_risk_results() const`
### `std::vector<ObstacleDebugRecord> get_debug_history_reverse()`

---

# 10.19 C++ — `RobotObstacleReportWriter`

Tạo class:

```cpp
class RobotObstacleReportWriter
```

## Hàm cần có

### `write_zone_obstacle_statistics_report(...)`
Ví dụ:

```text
[Zone Obstacle Statistics]
Cloud: base_cloud_01
Zone: FRONT
Point Count: 35
Min Distance: 0.62
Average Distance: 1.18
```

### `write_clearance_report(...)`
Ví dụ:

```text
[Clearance Report]
Cloud: base_cloud_01
Center Corridor Clearance: 0.74
Front Clearance: 0.62
```

### `write_obstacle_risk_report(...)`
Ví dụ:

```text
[Obstacle Risk]
Cloud: base_cloud_01
Overall Risk: DANGER
Most Dangerous Zone: CENTER_CORRIDOR
Nearest Front Distance: 0.62
Nearest Center Distance: 0.74
```

### `write_walking_safety_report(...)`
Ví dụ:

```text
[Walking Safety]
Cloud: base_cloud_01
Recommendation: STOP_AND_REPLAN
```

### `write_reverse_debug_history(...)`

---

# 10.20 C++ — `main.cpp`

## Yêu cầu

```text
Load robot-frame point cloud config
Load zone rule config
Load safety threshold config
Load runtime config

Create:
    ObstacleZoneClassifier
    ClearanceComputationEngine
    RiskScoringEngine

Create RobotObstacleReasoningLab
Run
Write reports
```

---

# 11. Luật obstacle reasoning của project

## Luật 1 — Mọi phân tích phải được thực hiện trên robot-frame point cloud
Không được lấy cloud camera-frame để chấm walking safety.

## Luật 2 — Zone rules phải nhất quán với quy ước trục
Ví dụ:
- `+Z` là phía trước
- `+X` là bên phải
- `-X` là bên trái

Nếu bạn dùng quy ước khác, phải ghi rõ.

## Luật 3 — Risk phải phụ thuộc ít nhất vào khoảng cách gần nhất
Không được chấm risk hoàn toàn ngẫu nhiên.

## Luật 4 — Center corridor phải được ưu tiên trong scoring
Vì đây là vùng đi thẳng của robot, thường quan trọng nhất cho locomotion.

## Luật 5 — Báo cáo cuối phải có khuyến nghị
Ít nhất 1 trong các kiểu:
- `SAFE_TO_MOVE`
- `SLOW_DOWN`
- `STOP_AND_REPLAN`

---

# 12. Output mong muốn

## Config
```text
config/robot_point_cloud_config.txt
config/zone_rule_config.txt
config/safety_threshold_config.txt
config/runtime_config.txt
```

## Reports
```text
assets/outputs/zone_obstacle_statistics_report.txt
assets/outputs/clearance_report.txt
assets/outputs/obstacle_risk_report.txt
assets/outputs/walking_safety_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build robot-frame cloud config
- test zone masks / obstacle distances bằng NumPy
- preview risk labels trước khi viết runtime C++

## C++
- chạy obstacle reasoning runtime
- tính khoảng cách vật cản theo vùng
- chấm risk cho cloud
- sinh walking safety report

## Computer Vision / Robot Perception
Đây là project rất quan trọng vì nó là bước perception đầu tiên có **ý nghĩa hành động**:
- robot không chỉ “thấy” point cloud
- robot bắt đầu **đánh giá nguy cơ di chuyển**

Bài 50 là nền rất đẹp cho các hướng tiếp theo như:
- local planner perception support
- footstep safety pre-check
- traversability / free-space reasoning
- dynamic obstacle monitoring

---

# 14. Checklist hoàn thành

- [ ] Python build đủ config cho robot-frame clouds / zone rules / thresholds
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho cloud ids / rule ids
- [ ] Python có NumPy preview cho zone classification / obstacle distance
- [ ] C++ có `ObstacleZoneRule`
- [ ] C++ có `ZoneObstacleStatistic`
- [ ] C++ có `CloudRiskResult`
- [ ] C++ có `ObstacleZoneClassifier`
- [ ] C++ có `ClearanceComputationEngine`
- [ ] C++ có `RiskScoringEngine`
- [ ] C++ có `RobotObstacleReasoningLab`
- [ ] C++ ghi đủ 5 report

---
