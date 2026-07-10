# 🤖 Bài 38: Stereo Landmark Motion Queue Simulator — Bộ mô phỏng landmark chuyển động và hàng đợi quan sát stereo cho Humanoid Robot AI Perception

> Mini Project số 38 trong **Đợt 8**  
> **Bài 38 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8** và đi tiếp trực tiếp từ **Bài 37**.
>
> Nếu:
>
> - **Bài 36** tập trung vào **temporal disparity/depth track**
> - **Bài 37** tập trung vào **command graph runner cho stereo depth pipeline**
>
> thì **Bài 38** sẽ thêm một tầng cực quan trọng cho perception:
>
> ```text
> landmark chuyển động trong 3D
> → project sang left/right camera theo từng frame
> → sinh observation queue
> → chạy command graph
> → track disparity / depth theo thời gian
> ```
>
> Tức là thay vì chỉ “đọc observation có sẵn từ file”, bạn bắt đầu **tự mô phỏng nguồn dữ liệu perception**.

---

# 📌 Mục lục

- [1. Vì sao Bài 38 xuất hiện sau Bài 37](#1-vì-sao-bài-38-xuất-hiện-sau-bài-37)
- [2. Đợt 8 đang học gì](#2-đợt-8-đang-học-gì)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Ý tưởng mô phỏng landmark chuyển động](#5-ý-tưởng-mô-phỏng-landmark-chuyển-động)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật mô phỏng + chạy queue](#11-luật-mô-phỏng--chạy-queue)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 38 xuất hiện sau Bài 37

## Bài 37 đang làm gì?
Bài 37 đã có:

```text
stereo observation
→ command graph
→ disparity
→ depth
→ validation
→ track update
→ report
```

Nhưng observation vẫn là **đầu vào có sẵn**.

## Bài 38 nâng lên ở đâu?
Bài 38 sẽ cho bạn **tự tạo ra observation** từ một cảnh chuyển động đơn giản.

Ví dụ:

- một `cup_01` di chuyển nhẹ theo trục X
- một `ball_01` tiến gần camera
- một `box_01` đi chéo trong không gian

Mỗi frame bạn sẽ:
1. cập nhật vị trí 3D của landmark
2. project sang ảnh trái/phải
3. sinh observation `(u_left, u_right)`
4. đưa observation vào queue
5. chạy command graph từ Bài 37

Tức là Bài 38 nối thêm **motion simulation → stereo observation generation → depth runtime**.

---

# 2. Đợt 8 đang học gì

Theo roadmap bạn gửi, **Đợt 8** gồm:

## Python — Phase 8
- `Linked List`
- `Stack`
- `Queue`

## C++ — Phase 6
- `Copy Constructor`
- `Inheritance`
- `Virtual Functions`
- `abstract class`
- `pure virtual`

## Computer Vision — Phase 4
- `Extrinsic Matrix`
- `Projection 3D → 2D` fileciteturn11file0

## Vì sao Bài 38 vẫn đúng tinh thần Đợt 8?
Vì bài này bắt bạn dùng đồng thời:
- **Queue** → observation queue theo frame
- **Linked List / history** → track / trace preview
- **abstract class** → motion model / command base
- **projection 3D → 2D** → tự sinh stereo observation từ landmark 3D
- **extrinsic-aware thinking** → left/right camera rig

---

# 3. Mô tả

Bạn sẽ xây một mini runtime tên là:

# **Stereo Landmark Motion Queue Simulator**

Runtime này sẽ mô phỏng một hoặc nhiều landmark 3D di chuyển theo thời gian, ví dụ:

- `cup_01`
- `ball_01`
- `box_01`

Mỗi landmark có:
- vị trí ban đầu `(x, y, z)`
- vận tốc đơn giản `(vx, vy, vz)`

Với mỗi frame:
1. cập nhật vị trí landmark
2. biến landmark sang **left camera** và **right camera**
3. project sang ảnh trái/phải
4. sinh stereo observation
5. push observation vào queue
6. chạy command graph để:
   - compute disparity
   - reconstruct depth
   - validate
   - update track
   - ghi report

<p align="center">
  <img src="images/project_38.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu được pipeline hoàn chỉnh hơn:

```text
Python
→ tạo motion config
→ tạo stereo rig config
→ tạo command graph config
→ preview landmark trajectory

C++
→ mô phỏng landmark motion theo frame
→ project landmark sang left/right image
→ build stereo observation queue
→ chạy command graph cho từng observation
→ update temporal track
→ xuất report
```

Mục tiêu là nối **motion → observation → perception pipeline** thành một khối.

---

# 5. Ý tưởng mô phỏng landmark chuyển động

Giả sử landmark `cup_01` có trạng thái ban đầu:

```text
x = 0.20
y = 0.05
z = 2.00
vx = 0.01
vy = 0.00
vz = -0.02
```

Mỗi frame:

```text
x_new = x_old + vx
y_new = y_old + vy
z_new = z_old + vz
```

Từ vị trí mới, bạn project sang stereo rig:

## Left camera
```text
u_left = fx * X_left / Z_left + cx
```

## Right camera
Nếu right camera lệch baseline theo trục X:

```text
X_right = X_left - baseline
u_right = fx * X_right / Z_right + cx
```

Sau đó tạo observation:

```text
landmark_id = cup_01
frame_index = 7
u_left = ...
u_right = ...
```

---

# 6. Pipeline tổng thể

```text
Load Stereo Intrinsics
Load Tracking Config
Load Motion Config
Load Command Graph Config

Create Motion Simulator
Create Observation Queue Builder
Create Command Graph Runner

For frame = 0 -> N-1:
    For each landmark:
        update 3D position using motion model
        project to left image
        project to right image
        build observation
        push into observation queue

After queue built:
    run command graph for each observation
    update tracks
    save reports
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- linked list
- stack / queue
- file handling
- config builder
- trajectory preview

## C++
- class / inheritance
- abstract class / pure virtual
- virtual override
- copy constructor
- `std::queue`
- `std::deque`
- `std::stack`
- `std::vector`
- `std::unordered_map`
- `std::shared_ptr`
- command graph + simulator design

## Computer Vision / Geometry
- 3D point motion
- pinhole projection
- stereo geometry
- disparity / depth
- temporal track update

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. `std::queue<StereoObservationRecord>`
Hàng đợi observation theo frame.

## 2. `std::vector<MovingLandmark>`
Danh sách landmark đang được mô phỏng.

## 3. `std::unordered_map<std::string, StereoTrack>`
Track table cho từng landmark.

## 4. `std::deque<StereoObservationRecord>`
History của mỗi track.

## 5. `std::vector<std::shared_ptr<BaseStereoCommand>>`
Command graph.

## 6. `std::stack<SimulationDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Landmark motion update
Mỗi frame:

```text
x = x + vx
y = y + vy
z = z + vz
```

---

## Algorithm 2 — Stereo projection
Nếu landmark trong left camera frame là `(X, Y, Z)`:

```text
u_left  = fx * X / Z + cx
u_right = fx * (X - baseline) / Z + cx
```

Bạn có thể đơn giản hóa `v_left = v_right = fy * Y / Z + cy`.

---

## Algorithm 3 — Observation generation
Từ projection:
- landmark_id
- frame_index
- `u_left`
- `u_right`

tạo `StereoObservationRecord`.

---

## Algorithm 4 — Command graph execution
Dùng lại pipeline của Bài 37:
- disparity
- depth
- validation
- track update
- report

---

## Algorithm 5 — Track smoothing
Dùng moving average như Bài 36–37.

---

# 9. Cấu trúc folder

```text
mini_project_38_stereo_landmark_motion_queue_simulator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ landmark_motion_report.txt
│     ├─ generated_observation_queue_report.txt
│     ├─ observation_command_trace.txt
│     ├─ stereo_track_summary_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_intrinsics_config.txt
│  ├─ stereo_tracking_config.txt
│  ├─ stereo_command_graph_config.txt
│  └─ landmark_motion_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ trajectory_preview.py
│     ├─ linked_trace_preview.py
│     └─ command_graph_preview.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoIntrinsics.hpp
   │  ├─ StereoTrackingConfig.hpp
   │  ├─ StereoObservationRecord.hpp
   │  ├─ StereoObservationContext.hpp
   │  ├─ StereoTrackState.hpp
   │  ├─ StereoTrack.hpp
   │  ├─ StereoTrackManager.hpp
   │  ├─ MovingLandmark.hpp
   │  ├─ BaseMotionModel.hpp
   │  ├─ ConstantVelocityMotionModel.hpp
   │  ├─ StereoMotionSimulator.hpp
   │  ├─ BaseStereoCommand.hpp
   │  ├─ InputCommand.hpp
   │  ├─ DisparityCommand.hpp
   │  ├─ DepthReconstructionCommand.hpp
   │  ├─ ValidationCommand.hpp
   │  ├─ TrackUpdateCommand.hpp
   │  ├─ ReportCommand.hpp
   │  ├─ StereoCommandGraphRunner.hpp
   │  └─ StereoSimulationReportWriter.hpp
   │
   └─ src/
      ├─ StereoTrack.cpp
      ├─ StereoTrackManager.cpp
      ├─ ConstantVelocityMotionModel.cpp
      ├─ StereoMotionSimulator.cpp
      ├─ InputCommand.cpp
      ├─ DisparityCommand.cpp
      ├─ DepthReconstructionCommand.cpp
      ├─ ValidationCommand.cpp
      ├─ TrackUpdateCommand.cpp
      ├─ ReportCommand.cpp
      ├─ StereoCommandGraphRunner.cpp
      └─ StereoSimulationReportWriter.cpp
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
intrinsics_path
tracking_config_path
graph_config_path
motion_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoMotionConfigBuilder`

Tạo class con:

```python
class StereoMotionConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `set_stereo_intrinsics(left_fx, right_fx, baseline, image_width, image_height)`

### `set_tracking_config(max_history_size, disparity_outlier_threshold, min_valid_history_for_stable_track)`

### `set_graph_order(command_names)`

### `add_moving_landmark(
    landmark_id,
    x, y, z,
    vx, vy, vz
)`

### `write_intrinsics_config()`
### `write_tracking_config()`
### `write_graph_config()`
### `write_motion_config()`

---

# 10.3 Python — `trajectory_preview.py`

Tạo class:

```python
class LandmarkTrajectoryPreview:
```

## Hàm cần có

### `simulate_constant_velocity(x, y, z, vx, vy, vz, num_frames)`
- trả list các vị trí 3D theo frame

### `preview_landmark(landmark_id, ...)`
- in trajectory

---

# 10.4 C++ — `MovingLandmark`

```cpp
struct MovingLandmark
{
    std::string landmark_id;

    double x, y, z;
    double vx, vy, vz;
};
```

---

# 10.5 C++ — `BaseMotionModel`

Tạo abstract class:

```cpp
class BaseMotionModel
{
public:
    virtual void update(MovingLandmark& landmark) = 0;
    virtual ~BaseMotionModel() = default;
};
```

---

# 10.6 C++ — `ConstantVelocityMotionModel`

Kế thừa:

```cpp
class ConstantVelocityMotionModel : public BaseMotionModel
```

## Hàm override

```cpp
void update(MovingLandmark& landmark) override;
```

## Hành vi
```text
x += vx
y += vy
z += vz
```

---

# 10.7 C++ — `StereoMotionSimulator`

Tạo class:

```cpp
class StereoMotionSimulator
```

## Thuộc tính
```cpp
private:
    StereoIntrinsics intrinsics;
    std::vector<MovingLandmark> landmarks;
    std::shared_ptr<BaseMotionModel> motion_model;
```

## Hàm cần có

### `load_landmark_motion_config(const std::string& path)`
### `void simulate_frames(int num_frames, std::queue<StereoObservationRecord>& output_queue)`
- với mỗi frame:
  - update landmark
  - project sang stereo
  - push observation

### `StereoObservationRecord build_observation(
    const MovingLandmark& landmark,
    int frame_index
) const;`

### `double project_left_u(const MovingLandmark& landmark) const`
### `double project_right_u(const MovingLandmark& landmark) const`

---

# 10.8 C++ — `StereoObservationContext`
Giống Bài 37.

---

# 10.9 C++ — `StereoTrack`, `StereoTrackManager`
Dùng lại tư tưởng của Bài 36–37:
- history
- smoothing
- outlier rejection
- stable track

---

# 10.10 C++ — `BaseStereoCommand`
Giống Bài 37.

---

# 10.11 C++ — 6 command con
Giống Bài 37:
- `InputCommand`
- `DisparityCommand`
- `DepthReconstructionCommand`
- `ValidationCommand`
- `TrackUpdateCommand`
- `ReportCommand`

---

# 10.12 C++ — `StereoCommandGraphRunner`
Dùng lại kiến trúc của Bài 37 nhưng đầu vào queue sẽ đến từ **simulator**, không phải file observation tĩnh.

## Hàm cần có thêm
### `set_observation_queue(std::queue<StereoObservationRecord> queue)`
- nhận queue từ simulator

---

# 10.13 C++ — `StereoSimulationReportWriter`

Tạo class:

```cpp
class StereoSimulationReportWriter
```

## Hàm cần có

### `write_landmark_motion_report(...)`
Ví dụ:

```text
[Landmark Motion]
Landmark: cup_01
Frame 0: x=0.20, y=0.05, z=2.00
Frame 1: x=0.21, y=0.05, z=1.98
Frame 2: x=0.22, y=0.05, z=1.96
```

### `write_generated_observation_queue_report(...)`

### `write_observation_command_trace(...)`

### `write_track_summary_report(...)`

### `write_reverse_debug_history(...)`

---

# 10.14 C++ — `main.cpp`

## Yêu cầu

```text
Load intrinsics + tracking config + graph config + motion config
Create simulator
Create command graph runner
Simulate N frames → build observation queue
Inject queue into graph runner
Run graph
Write all reports
```

---

# 11. Luật mô phỏng + chạy queue

## Luật 1 — Landmark phải có `z > 0`
Nếu landmark đi ra sau camera (`z <= 0`) thì observation phải invalid.

## Luật 2 — Observation được push theo thứ tự frame
FIFO theo thời gian.

## Luật 3 — Graph command chỉ xử lý 1 observation tại 1 thời điểm
Không xử lý song song ở bài này.

## Luật 4 — Track giữ history riêng theo `landmark_id`

## Luật 5 — Có thể cho nhiều landmark chạy đồng thời
Ví dụ:
- `cup_01`
- `ball_01`
- `box_01`

---

# 12. Output mong muốn

## Config
```text
config/stereo_intrinsics_config.txt
config/stereo_tracking_config.txt
config/stereo_command_graph_config.txt
config/landmark_motion_config.txt
```

## Reports
```text
assets/outputs/landmark_motion_report.txt
assets/outputs/generated_observation_queue_report.txt
assets/outputs/observation_command_trace.txt
assets/outputs/stereo_track_summary_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build motion config
- preview trajectory
- build command graph config

## C++
- simulate moving landmark
- generate stereo observations
- run command graph
- update temporal depth track

## Computer Vision / Robot Perception
Bài này bắt đầu rất giống một **perception simulator mini**:

```text
3D landmark motion
→ camera observation generation
→ stereo depth pipeline
→ temporal tracking
```

Đây là bước rất có giá trị vì bạn không còn phụ thuộc hoàn toàn vào data “đã có sẵn”, mà bắt đầu hiểu **sensor observation được sinh ra như thế nào từ thế giới 3D**.

---

# 14. Checklist hoàn thành

- [ ] Python build đủ intrinsics / tracking / graph / motion config
- [ ] Python preview trajectory
- [ ] C++ có `BaseMotionModel`
- [ ] C++ có `ConstantVelocityMotionModel`
- [ ] C++ có `StereoMotionSimulator`
- [ ] C++ sinh được observation queue từ landmark motion
- [ ] C++ dùng lại command graph của Bài 37
- [ ] C++ update track cho nhiều frame
- [ ] C++ ghi đủ 5 report

---

# 15. Gợi ý mở rộng

## 1. Thêm noise vào projection
Thêm nhiễu nhỏ vào `u_left`, `u_right`.

## 2. Thêm nhiều motion model
Ví dụ:
- zig-zag
- circular
- accelerate motion

## 3. Thêm extrinsic transform thật
Cho landmark nằm ở **robot base frame**, rồi transform sang stereo camera frame trước khi project.

## 4. Chuẩn bị cho Bài 39
Bài 39 nên đi tiếp theo hướng:

# **Stereo Multi-Landmark Track Table Analyzer**

Ý tưởng:
- nhiều landmark chạy đồng thời
- nhiều track cùng cập nhật
- thống kê:
  - track nào ổn định nhất
  - track nào nhiều outlier nhất
  - track nào depth biến động mạnh nhất

Tức là Bài 39 sẽ bắt đầu đẩy mạnh **multi-track analysis + STL containers + report analysis**.
