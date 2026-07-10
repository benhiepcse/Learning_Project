# 🤖 Bài 37: Stereo Depth Command Graph Runner — Bộ chạy command graph cho pipeline stereo depth trong Humanoid Robot AI Perception

> Mini Project số 37 trong **Đợt 8**  
> **Bài 37 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8** và đi tiếp trực tiếp từ **Bài 36**.
>
> Nếu:
>
> - **Bài 35** tập trung vào **depth reconstruction + validation**
> - **Bài 36** tập trung vào **temporal disparity/depth tracking**
>
> thì **Bài 37** sẽ đẩy thêm một tầng kiến trúc:
>
> ```text
> stereo observation
> → command graph
> → node-by-node processing
> → disparity
> → depth
> → validation
> → track update
> → report export
> ```
>
> Nghĩa là bạn không chỉ xử lý “có depth hay không”, mà còn học cách **tổ chức perception pipeline thành graph các command / stage** — rất gần với tư duy build runtime module cho robot.

---

# 📌 Mục lục

- [1. Vì sao Bài 37 xuất hiện sau Bài 36](#1-vì-sao-bài-37-xuất-hiện-sau-bài-36)
- [2. Đợt 8 đang học gì](#2-đợt-8-đang-học-gì)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Ý tưởng command graph](#5-ý-tưởng-command-graph)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật chạy graph](#11-luật-chạy-graph)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 37 xuất hiện sau Bài 36

## Bài 36 đang làm gì?
Bài 36 có tư duy:

```text
observation theo thời gian
→ track history
→ outlier rejection
→ moving average smoothing
→ smoothed depth
```

Nó rất tốt ở chỗ bạn đã bắt đầu quản lý **temporal stereo track**.  
Nhưng ở Bài 36, phần runtime vẫn khá “liền một cục”: observation vào → updater xử lý → track cập nhật.

## Bài 37 nâng lên ở đâu?
Bài 37 tách toàn bộ runtime đó thành **các command node độc lập**:

```text
1. Load observation
2. Compute disparity
3. Reconstruct depth
4. Validate observation
5. Update track
6. Export result
```

Tức là thay vì 1 class làm nhiều thứ, bạn sẽ học cách chia perception pipeline thành **graph / command chain**.

Điều này rất hợp với humanoid robot AI perception vì trong thực tế bạn thường có:

```text
sensor input
→ preprocessing
→ feature / geometry / depth
→ filtering / validation
→ tracking / fusion
→ output publishing
```

---

# 2. Đợt 8 đang học gì

Theo roadmap của bạn, **Đợt 8** gồm:

## Python — Phase 8
- `Linked List`
- `Stack`
- `Queue`

## C++ — Phase 6
- `Copy Constructor`
- hoàn thiện `Inheritance`
- `Virtual Functions`
- `abstract class`
- `pure virtual`

## Computer Vision — Phase 4
- `Extrinsic Matrix`
- `Projection 3D → 2D` fileciteturn11file0

## Vì sao Bài 37 vẫn bám đúng Đợt 8?
Vì Bài 37 sẽ ép bạn dùng:

- **Queue** → hàng đợi observation / command ready
- **Stack** → reverse debug / rollback log
- **Linked List** → command execution trace preview phía Python
- **abstract class / pure virtual** → base command node
- **inheritance** → mỗi stage là một command con
- **copy constructor** → copy graph state / snapshot nếu muốn
- **CV geometry** → disparity, depth, extrinsic-aware report

---

# 3. Mô tả

Bạn sẽ xây một mini runtime tên là:

# **Stereo Depth Command Graph Runner**

Runtime này nhận một chuỗi stereo observations như:

```text
landmark_id, frame_index, u_left, u_right
```

và chạy chúng qua một **graph command pipeline**.

Ví dụ một observation của `cup_01` đi qua graph:

```text
ObservationInputCommand
→ DisparityCommand
→ DepthReconstructionCommand
→ ValidationCommand
→ TrackUpdateCommand
→ ReportCommand
```

Mỗi command sẽ chỉ làm **một trách nhiệm nhỏ**.

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu được cách tổ chức perception pipeline như sau:

```text
Python
→ tạo config graph
→ tạo sequence observation
→ mô tả thứ tự command cần chạy
→ preview execution chain

C++
→ build command graph runtime
→ với mỗi observation:
    chạy qua từng command node
    cập nhật observation context
    cập nhật track state nếu hợp lệ
    ghi debug log / execution trace
→ xuất report cuối
```

Mục tiêu không chỉ là tính đúng disparity / depth, mà còn là:

- hiểu **pipeline decomposition**
- hiểu **command pattern**
- hiểu **graph / stage execution**
- hiểu **perception runtime modularity**

---

# 5. Ý tưởng command graph

Thay vì một hàm lớn:

```cpp
process_observation(...)
```

bạn sẽ có nhiều command node.

## Ví dụ graph tối thiểu

```text
[Input Command]
      ↓
[Disparity Command]
      ↓
[Depth Reconstruction Command]
      ↓
[Validation Command]
      ↓
[Track Update Command]
      ↓
[Report Command]
```

Mỗi command sẽ:
- đọc `StereoObservationContext`
- cập nhật dữ liệu vào context
- trả về success / fail
- quyết định observation có được đi tiếp hay không

---

# 6. Pipeline tổng thể

```text
Load Stereo Intrinsics
Load Tracking Config
Load Command Graph Config
Load Observation Sequence

Create Track Manager
Create Command Graph Runner

For each observation:
    Build ObservationContext

    Run command graph:
        InputCommand
        DisparityCommand
        DepthReconstructionCommand
        ValidationCommand
        TrackUpdateCommand
        ReportCommand

Save:
    observation command trace
    final track summary
    reverse debug history
    graph execution report
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- dict / list
- linked list
- stack / queue
- file handling
- config builder
- command graph preview

## C++
- class / inheritance
- abstract class / pure virtual
- virtual function override
- `std::queue`
- `std::stack`
- `std::deque`
- `std::unordered_map`
- `std::vector`
- `std::shared_ptr`
- copy constructor
- command pattern
- graph execution logic

## Computer Vision / Geometry
- disparity
- depth reconstruction
- stereo observation validation
- track update logic
- extrinsic-aware perception thinking

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. `std::queue<StereoObservationRecord>`
Observation stream theo thời gian.

## 2. `std::vector<std::shared_ptr<BaseStereoCommand>>`
Danh sách command của graph theo thứ tự chạy.

## 3. `std::unordered_map<std::string, StereoTrack>`
Bảng track cho nhiều landmark.

## 4. `std::deque<StereoObservationRecord>`
History buffer của từng track.

## 5. `std::stack<GraphDebugRecord>`
Debug history theo chiều ngược.

## 6. Python Linked List
Dùng để preview execution chain hoặc observation history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Disparity
```text
d = u_left - u_right
```

## Algorithm 2 — Depth
```text
depth = fx * baseline / disparity
```

## Algorithm 3 — Validation
Observation invalid nếu:
- disparity <= 0
- depth <= 0
- disparity quá lệch so với history nếu track đã ổn định

## Algorithm 4 — Track update
Nếu observation hợp lệ:
- push vào history
- trim history
- tính lại smoothed disparity
- tính lại smoothed depth

## Algorithm 5 — Command graph execution
Chạy command từ trái sang phải.  
Nếu command fail:
- ghi log
- đánh dấu observation failed
- dừng observation đó hoặc cho skip command tiếp theo tùy thiết kế.

---

# 9. Cấu trúc folder

```text
mini_project_37_stereo_depth_command_graph_runner/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ observation_command_trace.txt
│     ├─ stereo_track_summary_report.txt
│     ├─ reverse_debug_history.txt
│     └─ graph_execution_report.txt
│
├─ config/
│  ├─ stereo_intrinsics_config.txt
│  ├─ stereo_tracking_config.txt
│  ├─ stereo_command_graph_config.txt
│  └─ stereo_observation_sequence.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ command_graph_preview.py
│     ├─ linked_trace_preview.py
│     └─ sequence_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoIntrinsics.hpp
   │  ├─ StereoTrackingConfig.hpp
   │  ├─ StereoObservationRecord.hpp
   │  ├─ StereoObservationContext.hpp
   │  ├─ StereoTrackState.hpp
   │  ├─ GraphDebugRecord.hpp
   │  ├─ StereoTrack.hpp
   │  ├─ StereoTrackManager.hpp
   │  ├─ BaseStereoCommand.hpp
   │  ├─ InputCommand.hpp
   │  ├─ DisparityCommand.hpp
   │  ├─ DepthReconstructionCommand.hpp
   │  ├─ ValidationCommand.hpp
   │  ├─ TrackUpdateCommand.hpp
   │  ├─ ReportCommand.hpp
   │  ├─ StereoCommandGraphRunner.hpp
   │  └─ StereoGraphReportWriter.hpp
   │
   └─ src/
      ├─ StereoTrack.cpp
      ├─ StereoTrackManager.cpp
      ├─ InputCommand.cpp
      ├─ DisparityCommand.cpp
      ├─ DepthReconstructionCommand.cpp
      ├─ ValidationCommand.cpp
      ├─ TrackUpdateCommand.cpp
      ├─ ReportCommand.cpp
      ├─ StereoCommandGraphRunner.cpp
      └─ StereoGraphReportWriter.cpp
```

---

# 10. Yêu cầu mini-project

# 10.1 Python — `BaseConfigBuilder`

**File**
```text
python/tools/config_builder.py
```

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
observation_sequence_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in các config path

### `@classmethod create_default_paths(cls, root_dir)`
- tạo path mặc định

### `@staticmethod validate_positive_number(value, field_name)`
- validate `fx`, `baseline`, threshold

---

# 10.2 Python — `StereoCommandGraphConfigBuilder`

Tạo class con:

```python
class StereoCommandGraphConfigBuilder(BaseConfigBuilder):
```

## Dữ liệu cần build
- stereo intrinsics
- tracking config
- graph command order
- observation sequence

## Hàm cần có

### `set_stereo_intrinsics(left_fx, right_fx, baseline, image_width, image_height)`

### `set_tracking_config(max_history_size, disparity_outlier_threshold, min_valid_history_for_stable_track)`

### `set_graph_order(command_names)`
Ví dụ:
```python
[
    "InputCommand",
    "DisparityCommand",
    "DepthReconstructionCommand",
    "ValidationCommand",
    "TrackUpdateCommand",
    "ReportCommand",
]
```

### `add_observation(landmark_id, frame_index, u_left, u_right)`

### `write_intrinsics_config()`
### `write_tracking_config()`
### `write_graph_config()`
### `write_observation_sequence()`

---

# 10.3 Python — `command_graph_preview.py`

Tạo class:

```python
class CommandNode:
```

và:

```python
class CommandGraphPreview:
```

## Mục tiêu
- mô phỏng graph command bằng linked style hoặc list style
- in ra execution order

## Hàm cần có

### `add_command(command_name)`
### `to_list()`
### `show_graph()`

---

# 10.4 Python — `linked_trace_preview.py`

Tạo class linked list để mô phỏng execution trace của 1 observation.

```python
class TraceNode:
    ...
```

```python
class LinkedExecutionTrace:
```

## Hàm cần có
### `append_step(step_name, status)`
### `to_list()`
### `clear()`

---

# 10.5 Python — `sequence_generator.py`

Tạo class:

```python
class StereoSequenceGenerator:
```

## Hàm cần có

### `generate_track_sequence(...)`
- sinh sequence disparity / pixel ổn định

### `inject_invalid_disparity(...)`
- tạo case `u_left <= u_right`

### `inject_outlier(...)`
- disparity bất thường

---

# 10.6 C++ — `StereoObservationContext`

Tạo struct:

```cpp
struct StereoObservationContext
{
    StereoObservationRecord observation;

    bool command_failed;
    std::string failed_command_name;

    bool disparity_computed;
    bool depth_computed;
    bool validation_passed;
    bool track_updated;
    bool report_written;
};
```

Context này là “gói dữ liệu sống” đi xuyên suốt các command.

---

# 10.7 C++ — `StereoTrackState`

```cpp
struct StereoTrackState
{
    std::string landmark_id;
    int valid_observation_count;
    int outlier_count;
    bool has_stable_estimate;
    double smoothed_disparity;
    double smoothed_depth;
    int latest_frame_index;
};
```

---

# 10.8 C++ — `StereoTrack`

Tạo class:

```cpp
class StereoTrack
```

## Thuộc tính
```cpp
private:
    StereoTrackState state;
    std::deque<StereoObservationRecord> history;
```

## Constructor
```cpp
StereoTrack(const std::string& landmark_id);
StereoTrack(const StereoTrack& other); // copy constructor
```

## Hàm cần có
### `push_history(const StereoObservationRecord& observation)`
### `trim_history(int max_size)`
### `double compute_average_disparity() const`
### `const std::deque<StereoObservationRecord>& get_history() const`
### `StereoTrackState& get_state()`
### `const StereoTrackState& get_state() const`

---

# 10.9 C++ — `StereoTrackManager`

Tạo class quản lý track:

```cpp
class StereoTrackManager
```

## Thuộc tính
```cpp
private:
    StereoIntrinsics intrinsics;
    StereoTrackingConfig config;
    std::unordered_map<std::string, StereoTrack> tracks;
```

## Hàm cần có
### `StereoTrack& get_or_create_track(const std::string& landmark_id)`
### `bool is_outlier(const StereoTrack& track, double disparity) const`
### `void update_track(StereoObservationContext& context)`
### `const std::unordered_map<std::string, StereoTrack>& get_tracks() const`

---

# 10.10 C++ — `BaseStereoCommand`

Tạo abstract class:

```cpp
class BaseStereoCommand
{
public:
    virtual std::string name() const = 0;

    virtual bool execute(
        StereoObservationContext& context,
        StereoTrackManager& track_manager
    ) = 0;

    virtual ~BaseStereoCommand() = default;
};
```

Đây là lõi của Bài 37.

---

# 10.11 C++ — `InputCommand`

```cpp
class InputCommand : public BaseStereoCommand
```

## Nhiệm vụ
- xác nhận observation đầu vào tồn tại
- kiểm tra cơ bản `frame_index >= 0`
- log rằng observation đã vào graph

---

# 10.12 C++ — `DisparityCommand`

```cpp
class DisparityCommand : public BaseStereoCommand
```

## Nhiệm vụ
- tính `disparity_raw = u_left - u_right`
- cập nhật cờ `disparity_computed`
- nếu disparity <= 0 thì fail command

---

# 10.13 C++ — `DepthReconstructionCommand`

```cpp
class DepthReconstructionCommand : public BaseStereoCommand
```

## Nhiệm vụ
- dùng `fx * baseline / disparity`
- cập nhật `depth_raw`
- nếu disparity chưa hợp lệ thì fail

---

# 10.14 C++ — `ValidationCommand`

```cpp
class ValidationCommand : public BaseStereoCommand
```

## Nhiệm vụ
- kiểm tra:
  - disparity > 0
  - depth > 0
  - nếu track đã có stable estimate thì disparity không lệch quá threshold
- set `validation_passed`

---

# 10.15 C++ — `TrackUpdateCommand`

```cpp
class TrackUpdateCommand : public BaseStereoCommand
```

## Nhiệm vụ
- nếu validation pass:
  - cập nhật track manager
  - push history
  - trim history
  - tính smoothed disparity / smoothed depth
  - set `track_updated = true`

---

# 10.16 C++ — `ReportCommand`

```cpp
class ReportCommand : public BaseStereoCommand
```

## Nhiệm vụ
- đánh dấu observation đã được ghi vào trace/report buffer
- set `report_written = true`

---

# 10.17 C++ — `GraphDebugRecord`

```cpp
struct GraphDebugRecord
{
    std::string command_name;
    std::string message;
};
```

---

# 10.18 C++ — `StereoCommandGraphRunner`

Tạo class:

```cpp
class StereoCommandGraphRunner
```

## Thuộc tính

```cpp
private:
    std::queue<StereoObservationRecord> observation_queue;
    std::vector<std::shared_ptr<BaseStereoCommand>> commands;
    StereoTrackManager track_manager;
    std::stack<GraphDebugRecord> debug_history;
    std::vector<StereoObservationContext> finished_contexts;
```

## Hàm cần có

### `load_observation_sequence(const std::string& path)`
- đẩy observation vào queue

### `register_command(std::shared_ptr<BaseStereoCommand> command)`
- thêm command vào graph

### `run()`
Pseudo:

```text
while queue not empty:
    obs = front
    pop

    create context from obs

    for each command in commands:
        success = command->execute(context, track_manager)

        push debug record

        if not success:
            context.command_failed = true
            context.failed_command_name = command->name()
            break

    save finished context
```

### `const StereoTrackManager& get_track_manager() const`
### `std::vector<GraphDebugRecord> get_debug_history_reverse()`
### `const std::vector<StereoObservationContext>& get_finished_contexts() const`

---

# 10.19 C++ — `StereoGraphReportWriter`

Tạo class:

```cpp
class StereoGraphReportWriter
```

## Hàm cần có

### `write_observation_command_trace(...)`
Ví dụ:

```text
[Observation Trace]
Landmark: cup_01
Frame: 12
DisparityComputed: true
DepthComputed: true
ValidationPassed: true
TrackUpdated: true
FailedCommand: none
```

### `write_track_summary_report(...)`

### `write_reverse_debug_history(...)`

### `write_graph_execution_report(...)`
Ví dụ:

```text
[Graph Execution Report]
Observation Count: 20
Success Count: 17
Failed Count: 3
Failure Commands:
- DisparityCommand: 1
- ValidationCommand: 2
```

---

# 10.20 C++ — `main.cpp`

## Yêu cầu

```text
Load intrinsics + tracking config
Build StereoTrackManager
Build StereoCommandGraphRunner

Register commands:
    InputCommand
    DisparityCommand
    DepthReconstructionCommand
    ValidationCommand
    TrackUpdateCommand
    ReportCommand

Load observation sequence
Run graph
Write all reports
```

---

# 11. Luật chạy graph

## Luật 1 — Observation nào cũng phải đi qua InputCommand
Không được bỏ qua node đầu vào.

## Luật 2 — Nếu `DisparityCommand` fail
- không được chạy `DepthReconstructionCommand`
- observation đó dừng luôn

## Luật 3 — Nếu `ValidationCommand` fail
- observation không được update track

## Luật 4 — `TrackUpdateCommand` chỉ chạy khi observation hợp lệ

## Luật 5 — `ReportCommand` chỉ ghi trạng thái cuối của observation

---

# 12. Output mong muốn

## Config
```text
config/stereo_intrinsics_config.txt
config/stereo_tracking_config.txt
config/stereo_command_graph_config.txt
config/stereo_observation_sequence.txt
```

## Reports
```text
assets/outputs/observation_command_trace.txt
assets/outputs/stereo_track_summary_report.txt
assets/outputs/reverse_debug_history.txt
assets/outputs/graph_execution_report.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build command graph config
- build observation sequence
- preview execution order
- preview linked execution trace

## C++
- chạy command graph runtime
- xử lý từng observation
- update track state
- log lỗi theo command
- tạo report cho perception pipeline

## Computer Vision / Robot Perception
Bài này rất sát tư duy perception thật:

```text
stereo sensor data
→ compute disparity
→ reconstruct depth
→ validate
→ track / filter
→ output result
```

Chỉ khác là bạn đang làm ở quy mô mini project, chưa nối ROS2.

---

# 14. Checklist hoàn thành

- [ ] Python build đủ intrinsics / tracking / graph / sequence config
- [ ] Python có command graph preview
- [ ] Python có linked execution trace preview
- [ ] C++ có `BaseStereoCommand`
- [ ] C++ có đủ 6 command con
- [ ] C++ có `StereoObservationContext`
- [ ] C++ có `StereoCommandGraphRunner`
- [ ] C++ dùng `queue` cho observation
- [ ] C++ dùng `vector<shared_ptr<...>>` cho command graph
- [ ] C++ dùng `unordered_map` cho track table
- [ ] C++ dùng `deque` cho history
- [ ] C++ có copy constructor cho `StereoTrack`
- [ ] C++ ghi được observation trace
- [ ] C++ ghi được graph execution report
- [ ] C++ ghi được reverse debug history

---

# 15. Gợi ý mở rộng

## 1. Branching graph
Thay vì graph tuyến tính, cho phép:

```text
DisparityCommand
 ├─ ValidationBranch
 └─ FailureBranch
```

## 2. Command timing
Đo thời gian chạy từng command.

## 3. Extrinsic-aware depth output
Sau khi có depth, chuyển point từ **camera frame** sang **robot frame** bằng extrinsic transform.

## 4. Chuẩn bị cho Bài 38
Bài 38 nên đi tiếp theo hướng:

# **Stereo Landmark Motion Queue Simulator**

Ý tưởng:

```text
landmark moving in 3D
→ project to left/right camera across frames
→ build observation queue
→ run command graph
→ track depth over time
```

Tức là Bài 38 sẽ nối:
- **camera projection**
- **motion simulation**
- **observation generation**
- **command graph runtime**
- **temporal depth tracking**

thành một khối perception simulation đầy đủ hơn.
