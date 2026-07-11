# 🤖 Bài 36: Stereo Disparity Track Manager — Bộ quản lý track disparity theo thời gian cho Humanoid Robot AI Perception

> Mini Project số 36 trong **Đợt 8**  
> **Bài 36 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8**.  
> Từ **Đợt 8** trở đi, project vẫn giữ mạch **robot perception runtime**, nhưng bắt đầu đánh mạnh hơn vào:
>
> - **Algorithms**
> - **DSA Structures**
> - **STL Containers**
> - **Advanced C++**
> - **temporal perception**
> - **camera geometry + stereo depth history**
>
> Nếu **Bài 35** là:
>
> ```text
> left/right pixel
> → disparity
> → depth reconstruction
> → so với ground truth
> → depth validation
> ```
>
> thì **Bài 36** sẽ nâng lên thành:
>
> ```text
> landmark qua nhiều frame
> → disparity history
> → track manager
> → smoothing / moving average
> → outlier rejection
> → depth estimate ổn định theo thời gian
> ```

---

# 📌 Mục lục

- [1. Đợt 8 có gì mới](#1-đợt-8-có-gì-mới)
- [2. Bài 36 lấy gì từ Bài 31–35](#2-bài-36-lấy-gì-từ-bài-3135)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Pipeline](#5-pipeline)
- [6. Kiến thức cần](#6-kiến-thức-cần)
- [7. DSA + Algorithm bắt buộc](#7-dsa--algorithm-bắt-buộc)
- [8. Cấu trúc folder](#8-cấu-trúc-folder)
- [9. Yêu cầu mini-project](#9-yêu-cầu-mini-project)
- [10. Điều kiện bắt buộc](#10-điều-kiện-bắt-buộc)
- [11. Output mong muốn](#11-output-mong-muốn)
- [12. Vai trò trong Humanoid Robot](#12-vai-trò-trong-humanoid-robot)
- [13. Checklist hoàn thành](#13-checklist-hoàn-thành)
- [14. Gợi ý mở rộng](#14-gợi-ý-mở-rộng)

---

# 1. Đợt 8 có gì mới

Theo roadmap bạn vừa gửi, **Đợt 8 (Ngày 15–16)** là:

## Chủ đề
**Camera model + command queue** fileciteturn11file0

## Python — Phase 8: Data Structures
- `Python Linked List`
- `Python Stack`
- `Python Queue`
- hoàn thiện phần còn lại nếu Đợt 7 chưa xong fileciteturn11file0

## C++ — Phase 6: OOP
- `C++ Copy Constructor`
- `C++ Inheritance` (hoàn thiện)
- `C++ Virtual Functions`
  - abstract class
  - pure virtual cơ bản fileciteturn11file0

## Computer Vision — Phase 4: Camera Geometry
- `Extrinsic Matrix`
- `Projection 3D → 2D` fileciteturn11file0

## Vì sao Bài 36 lại đặt như vậy?
Vì sau Đợt 7 bạn đã có:
- stereo projection
- depth reconstruction
- validation cho **một request / một snapshot**

Sang Đợt 8, điểm hợp lý nhất là thêm **temporal data structure + track manager** để biến snapshot stereo thành **chuỗi quan sát theo thời gian**.  
Như vậy Bài 36 vừa:
- dùng **Linked List / Queue / Stack**
- dùng **abstract class / pure virtual**
- vẫn bám **stereo geometry + robot perception**

---

# 2. Bài 36 lấy gì từ Bài 31–35

## Bài 31
- queue task scheduler
- stack reverse history
- pinhole camera model
- intrinsic matrix

## Bài 32
- extrinsic transform command queue
- source frame → camera frame

## Bài 33
- transformation graph
- BFS path finding
- parent reconstruction
- compose transform chain

## Bài 34
- stereo frame graph landmark projector
- left/right projection
- disparity computation

## Bài 35
- stereo depth reconstruction validator
- depth estimate vs ground truth
- quality label

## Bài 36
Bài 36 không còn xem stereo depth như **một ảnh chụp tĩnh** nữa.  
Thay vào đó, nó quản lý **một landmark qua nhiều frame thời gian**:

```text
frame_001: disparity = 26.4
frame_002: disparity = 27.1
frame_003: disparity = 110.0   <-- outlier
frame_004: disparity = 26.8
frame_005: disparity = 26.5
```

Track manager sẽ phải:
- lưu lịch sử disparity / depth của từng landmark
- reject outlier
- tính **smoothed disparity**
- tính **smoothed depth**
- ghi quality theo thời gian

---

# 3. Mô tả

Giả sử robot đang nhìn một landmark `cup_01` hoặc `table_corner_01` trong nhiều frame liên tiếp.  
Mỗi frame sẽ cho bạn một cặp quan sát:

- `u_left`
- `u_right`

Từ đó bạn có:

```text
disparity = u_left - u_right
depth = fx * baseline / disparity
```

Nhưng trong thực tế, disparity theo thời gian có thể bị:
- nhiễu
- outlier
- frame miss
- depth nhảy loạn

Vì vậy Bài 36 sẽ xây một **Stereo Disparity Track Manager** có nhiệm vụ:

## Với mỗi landmark track:
1. nhận observation mới theo frame index
2. tính disparity và depth tức thời
3. đưa observation vào track history
4. loại outlier
5. tính moving average disparity
6. tính smoothed depth
7. xuất trạng thái track hiện tại

<p align="center">
  <img src="../../images/project_36.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này, bạn phải hiểu pipeline:

```text
Python config builder
→ tạo stereo track sequence cho nhiều landmark
→ tạo intrinsics / baseline
→ tạo tracking config

C++ runtime
→ load stereo observations theo frame
→ group observation theo landmark track
→ với mỗi observation:
    compute disparity
    compute depth
    validate observation
    insert vào track history
    reject outlier nếu cần
    update smoothed disparity / smoothed depth
→ xuất track report
```

---

# 5. Pipeline

```text
Load Stereo Intrinsics
Load Tracking Config
Load Stereo Observation Sequence

Build Track Manager

For each observation in time order:
    landmark_id, frame_index, u_left, u_right

    Step 1:
        compute disparity = u_left - u_right

    Step 2:
        if disparity invalid:
            mark observation invalid
        else:
            compute raw depth

    Step 3:
        find / create track for landmark

    Step 4:
        compare with recent history
        reject outlier if deviation too large

    Step 5:
        insert valid observation into track history

    Step 6:
        recompute smoothed disparity
        recompute smoothed depth
        update track state

Write:
    observation report
    track summary report
    reverse debug history
```

---

# 6. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- `requirements.txt`
- file handling
- dict / list
- **linked list tư duy**
- stack / queue
- `if __name__ == "__main__"`

## C++
- class / inheritance
- **copy constructor**
- **abstract class**
- **pure virtual**
- `std::vector`
- `std::deque`
- `std::queue`
- `std::stack`
- `std::unordered_map`
- `std::unique_ptr`
- `std::shared_ptr`
- pass by reference / const reference
- temporal track manager design

## Computer Vision / Geometry
- stereo disparity
- depth reconstruction
- temporal smoothing
- outlier rejection logic
- perception tracking history

---

# 7. DSA + Algorithm bắt buộc

## DSA bắt buộc

### 1. `std::unordered_map<std::string, StereoTrack>`
Map từ `landmark_id` → track object.

---

### 2. `std::deque<StereoObservationRecord>`
Dùng làm **history buffer** cho mỗi track:
- push observation mới vào cuối
- pop đầu nếu vượt quá history length

---

### 3. `std::queue<StereoObservationRecord>`
Dùng để load và xử lý observations theo thời gian FIFO.

---

### 4. `std::stack<TrackDebugRecord>`
Dùng cho reverse debug history.

---

### 5. Python Linked List
Ở phía Python, bạn sẽ làm preview cấu trúc track history bằng linked list đơn giản để bám đúng Đợt 8.

---

## Algorithm bắt buộc

## Algorithm 1 — Disparity computation
```text
d = u_left - u_right
```

### Rule tối thiểu
- nếu `d <= 0` → invalid
- nếu `u_left` hoặc `u_right` thiếu / âm bất thường → invalid

---

## Algorithm 2 — Raw depth reconstruction
Nếu disparity hợp lệ:

```text
depth_raw = fx * baseline / disparity
```

---

## Algorithm 3 — Outlier rejection
Giả sử track đã có `smoothed_disparity` hiện tại.

Observation mới có disparity `d_new`.

### Rule gợi ý
Nếu:

```text
abs(d_new - smoothed_disparity) > disparity_outlier_threshold
```

thì đánh dấu observation là outlier và **không đưa vào smoothing**.

---

## Algorithm 4 — Moving average smoothing
Giả sử history buffer đang chứa các disparity hợp lệ gần nhất:

```text
d1, d2, d3, d4, d5
```

thì:

```text
smoothed_disparity = (d1 + d2 + d3 + d4 + d5) / N
smoothed_depth = fx * baseline / smoothed_disparity
```

---

## Algorithm 5 — History buffer maintenance
Nếu số observation vượt quá `max_history_size`:

```text
push_back(new_obs)
if size > max_history_size:
    pop_front()
```

---

## Algorithm 6 — Reverse debug history
Pop stack để ghi log ngược.

---

# 8. Cấu trúc folder

```text
mini_project_36_stereo_disparity_track_manager/
│
├─ README.md
│
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ stereo_observation_report.txt
│     ├─ stereo_track_summary_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_intrinsics_config.txt
│  ├─ stereo_tracking_config.txt
│  └─ stereo_observation_sequence.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ linked_track_preview.py
│     └─ sequence_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ StereoIntrinsics.hpp
   │  ├─ StereoTrackingConfig.hpp
   │  ├─ StereoObservationRecord.hpp
   │  ├─ StereoTrackState.hpp
   │  ├─ TrackDebugRecord.hpp
   │  ├─ BaseTrackUpdater.hpp
   │  ├─ MovingAverageTrackUpdater.hpp
   │  ├─ StereoTrack.hpp
   │  ├─ StereoDisparityTrackManager.hpp
   │  └─ StereoTrackReportWriter.hpp
   │
   └─ src/
      ├─ MovingAverageTrackUpdater.cpp
      ├─ StereoTrack.cpp
      ├─ StereoDisparityTrackManager.cpp
      └─ StereoTrackReportWriter.cpp
```

---

# 9. Yêu cầu mini-project

# 9.1 Python — `BaseConfigBuilder`

**File**
```text
python/tools/config_builder.py
```

Tạo class cha:

```python
class BaseConfigBuilder:
```

## Thuộc tính
```python
project_name
stereo_intrinsics_config_path
stereo_tracking_config_path
stereo_observation_sequence_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in path config

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

### `@staticmethod validate_positive_number(value, field_name)`
- dùng cho `fx`, `baseline`, threshold

---

# 9.2 Python — `StereoTrackConfigBuilder`

Tạo class con:

```python
class StereoTrackConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
stereo_intrinsics
tracking_config
observation_sequence
```

## Hàm cần có

### `set_stereo_intrinsics(left_fx, right_fx, baseline, image_width, image_height)`
- tối thiểu validate `left_fx > 0`, `right_fx > 0`, `baseline > 0`

### `set_tracking_config(
    max_history_size,
    disparity_outlier_threshold,
    min_valid_history_for_stable_track
)`
- validate các giá trị dương

### `add_observation(
    landmark_id,
    frame_index,
    u_left,
    u_right
)`
Ví dụ:
```text
cup_01, frame 1, u_left=420.0, u_right=394.0
```

### `write_stereo_intrinsics_config()`
### `write_stereo_tracking_config()`
### `write_stereo_observation_sequence()`

---

# 9.3 Python — `linked_track_preview.py`

Tạo class linked list đơn giản:

```python
class ObservationNode:
    ...
```

và:

```python
class LinkedTrackPreview:
```

## Mục tiêu
- mô phỏng history của 1 track bằng linked list
- đúng tinh thần **Python Linked List** của Đợt 8

## Hàm cần có

### `append_observation(frame_index, disparity)`
### `to_list()`
### `trim_to_max_size(max_size)`

---

# 9.4 Python — `sequence_generator.py`

Tạo class:

```python
class StereoSequenceGenerator:
```

## Hàm cần có

### `generate_clean_sequence(...)`
- sinh disparity / pixel sequence mượt

### `inject_outlier(...)`
- chèn một observation nhiễu mạnh

### `generate_depth_track_case(...)`
- tạo một sequence hoàn chỉnh cho 1 landmark

---

# 9.5 C++ — `StereoIntrinsics`

```cpp
struct StereoIntrinsics
{
    double left_fx;
    double right_fx;
    double baseline;
    int image_width;
    int image_height;
};
```

---

# 9.6 C++ — `StereoTrackingConfig`

```cpp
struct StereoTrackingConfig
{
    int max_history_size;
    double disparity_outlier_threshold;
    int min_valid_history_for_stable_track;
};
```

---

# 9.7 C++ — `StereoObservationRecord`

```cpp
struct StereoObservationRecord
{
    std::string landmark_id;
    int frame_index;

    double u_left;
    double u_right;

    bool disparity_valid;
    double disparity_raw;

    bool depth_valid;
    double depth_raw;

    bool is_outlier;
    bool accepted_into_track;
};
```

---

# 9.8 C++ — `StereoTrackState`

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

# 9.9 C++ — `TrackDebugRecord`

```cpp
struct TrackDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 9.10 C++ — `BaseTrackUpdater`

Tạo abstract class:

```cpp
class BaseTrackUpdater
{
public:
    virtual void update_track(
        class StereoTrack& track,
        StereoObservationRecord observation
    ) = 0;

    virtual ~BaseTrackUpdater() = default;
};
```

## Ý nghĩa
Đây là chỗ dùng **abstract class / pure virtual** của Đợt 8.

---

# 9.11 C++ — `MovingAverageTrackUpdater`

Kế thừa:

```cpp
class MovingAverageTrackUpdater : public BaseTrackUpdater
```

## Thuộc tính
```cpp
private:
    StereoIntrinsics intrinsics;
    StereoTrackingConfig config;
```

## Constructor

```cpp
MovingAverageTrackUpdater(
    const StereoIntrinsics& intrinsics,
    const StereoTrackingConfig& config
);
```

## Hàm override

```cpp
void update_track(
    StereoTrack& track,
    StereoObservationRecord observation
) override;
```

## Hành vi
1. kiểm tra disparity hợp lệ
2. nếu track đã có `smoothed_disparity`, kiểm tra outlier
3. nếu outlier → tăng `outlier_count`, không đưa vào history smoothing
4. nếu valid → push vào history buffer
5. trim history nếu vượt quá `max_history_size`
6. tính lại `smoothed_disparity`
7. tính lại `smoothed_depth`
8. cập nhật `has_stable_estimate`

---

# 9.12 C++ — `StereoTrack`

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

### `void push_history(const StereoObservationRecord& observation)`
### `void trim_history(int max_size)`
### `double compute_average_disparity() const`
### `const std::deque<StereoObservationRecord>& get_history() const`
### `StereoTrackState& get_state()`
### `const StereoTrackState& get_state() const`

---

# 9.13 C++ — `StereoDisparityTrackManager`

Tạo class:

```cpp
class StereoDisparityTrackManager
```

## Thuộc tính

```cpp
private:
    StereoIntrinsics intrinsics;
    StereoTrackingConfig config;

    std::queue<StereoObservationRecord> observation_queue;
    std::unordered_map<std::string, StereoTrack> tracks;

    std::shared_ptr<BaseTrackUpdater> updater;
    std::stack<TrackDebugRecord> debug_history;
```

## Hàm cần có

### `load_stereo_intrinsics_config(const std::string& path)`
### `load_stereo_tracking_config(const std::string& path)`
### `load_stereo_observation_sequence(const std::string& path)`

### `double compute_disparity(double u_left, double u_right) const`
- trả `u_left - u_right`

### `double reconstruct_depth(double disparity) const`
- `left_fx * baseline / disparity`

### `StereoObservationRecord build_observation_record(
    const std::string& landmark_id,
    int frame_index,
    double u_left,
    double u_right
) const;`

### `StereoTrack& get_or_create_track(const std::string& landmark_id)`

### `void run()`
Algorithm:

```text
while observation_queue not empty:
    obs = front
    pop

    build raw disparity / raw depth
    track = get_or_create_track(obs.landmark_id)
    updater->update_track(track, obs)

    push debug record
```

### Getter

```cpp
const std::unordered_map<std::string, StereoTrack>& get_tracks() const;
std::vector<TrackDebugRecord> get_debug_history_reverse();
```

---

# 9.14 C++ — `StereoTrackReportWriter`

Tạo class:

```cpp
class StereoTrackReportWriter
```

## Hàm cần có

### `write_stereo_observation_report(...)`

Format gợi ý:

```text
[Observation]
Landmark: cup_01
Frame: 12
u_left: 421.3
u_right: 394.8
Disparity Raw: 26.5
Depth Raw: 1.88
Outlier: false
Accepted: true
```

### `write_track_summary_report(...)`

Format gợi ý:

```text
[Track Summary]
Landmark: cup_01
Valid Observation Count: 8
Outlier Count: 1
Stable: true
Smoothed Disparity: 26.7
Smoothed Depth: 1.86
Latest Frame: 14
```

### `write_reverse_debug_history(...)`

Format gợi ý:

```text
[Reverse Debug History]
cup_02 frame 15: outlier rejected
cup_01 frame 14: smoothed depth updated
cup_01 frame 13: accepted observation
```

---

# 9.15 C++ — `main.cpp`

## Yêu cầu

```text
Create StereoDisparityTrackManager
→ Load stereo intrinsics
→ Load stereo tracking config
→ Load stereo observation sequence
→ Run
→ Write stereo observation report
→ Write track summary report
→ Write reverse debug history
```

---

# 10. Điều kiện bắt buộc

Project bắt buộc phải có:

- OOP Python
- OOP C++
- inheritance Python
- inheritance C++
- Python linked list preview
- Python queue / stack / generator
- C++ copy constructor
- C++ abstract class
- C++ pure virtual
- C++ `std::deque`
- C++ `std::queue`
- C++ `std::stack`
- C++ `std::unordered_map`
- disparity computation
- depth reconstruction
- outlier rejection
- moving average smoothing
- temporal track summary

---

# 11. Output mong muốn

## Config

```text
config/stereo_intrinsics_config.txt
config/stereo_tracking_config.txt
config/stereo_observation_sequence.txt
```

## Reports

```text
assets/outputs/stereo_observation_report.txt
assets/outputs/stereo_track_summary_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 12. Vai trò trong Humanoid Robot

## Python

```text
Track Config Builder + Linked Track Preview + Sequence Generator
```

Python sẽ đóng vai trò:
- tạo sequence quan sát stereo theo thời gian
- tạo config cho tracker
- preview logic linked list / queue

## C++

```text
Stereo Temporal Track Runtime
```

C++ sẽ đóng vai trò:
- nhận stream observation
- update track state
- smoothing disparity / depth
- reject outlier
- duy trì track history

## Computer Vision / Robot Perception

```text
stereo observation theo thời gian
→ disparity history
→ depth history
→ track smoothing
→ robust temporal estimate
```

Đây là bước rất quan trọng vì robot perception không sống ở **1 frame duy nhất**, mà luôn phải xử lý **chuỗi frame liên tiếp**.

---

# 13. Checklist hoàn thành

- [ ] Python tạo đủ 3 config
- [ ] Python có linked list preview
- [ ] Python có sequence generator
- [ ] C++ có `BaseTrackUpdater`
- [ ] C++ có `MovingAverageTrackUpdater`
- [ ] C++ có `StereoTrack`
- [ ] C++ có copy constructor cho `StereoTrack`
- [ ] C++ dùng `deque` làm history buffer
- [ ] C++ dùng `queue` cho observation stream
- [ ] C++ dùng `unordered_map` cho track table
- [ ] C++ tính disparity / depth raw
- [ ] C++ reject outlier
- [ ] C++ cập nhật smoothed disparity / depth
- [ ] C++ ghi đủ 3 report

---

# 14. Gợi ý mở rộng

## 1. Median filter thay cho moving average
Tạo thêm class:
```cpp
MedianTrackUpdater : public BaseTrackUpdater
```

để so sánh với moving average.

## 2. Track state machine
Thêm các trạng thái:
- `NEW`
- `TENTATIVE`
- `STABLE`
- `LOST`

## 3. Multi-landmark stereo stream
Cho nhiều landmark cùng chạy song song để thấy `unordered_map<landmark_id, track>` hoạt động ra sao.

## 4. Chuẩn bị cho Bài 37

Bài 37 nên đi tiếp theo hướng:

```text
Stereo Depth Command Graph Runner
```

Ý tưởng:
- không chỉ có **track manager**
- mà còn có **command graph / task graph** cho từng bước:
  - compute disparity
  - reconstruct depth
  - validate
  - update track
  - export report

Tức là bạn sẽ đẩy từ **track-level runtime** sang **graph-based stereo processing runtime** — rất hợp với hướng bạn muốn tăng dần thuật toán + DSA + Advanced C++.
