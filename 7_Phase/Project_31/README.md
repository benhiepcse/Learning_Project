# 🤖 Bài 31: Pinhole Camera Task Scheduler with Stack/Queue — Bộ lập lịch tác vụ Camera Geometry cho Humanoid Robot AI Perception

> Mini Project số 31 trong **Đợt 7 — Cấu trúc project chuẩn**  
> **Bài 31 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7**.  
> Từ **Đợt 7 trở đi**, project sẽ đánh mạnh hơn vào:
>
> - **Algorithms**
> - **DSA Structures**
> - **STL Containers**
> - **Advanced C++**
> - tổ chức runtime giống module robot thật hơn

---

# 📌 Mục lục

- [1. Đợt 7 có gì mới](#1-đợt-7-có-gì-mới)
- [2. Mô tả](#2-mô-tả)
- [3. Mục tiêu perception](#3-mục-tiêu-perception)
- [4. Pipeline](#4-pipeline)
- [5. Kiến thức cần](#5-kiến-thức-cần)
- [6. DSA + Algorithm bắt buộc](#6-dsa--algorithm-bắt-buộc)
- [7. Cấu trúc folder](#7-cấu-trúc-folder)
- [8. Yêu cầu mini-project](#8-yêu-cầu-mini-project)
- [9. Điều kiện bắt buộc](#9-điều-kiện-bắt-buộc)
- [10. Output mong muốn](#10-output-mong-muốn)
- [11. Vai trò trong Humanoid Robot](#11-vai-trò-trong-humanoid-robot)
- [12. Checklist hoàn thành](#12-checklist-hoàn-thành)
- [13. Gợi ý mở rộng](#13-gợi-ý-mở-rộng)

---

# 1. Đợt 7 có gì mới

Theo roadmap, **Đợt 7 — Ngày 13–14** có chủ đề:

```text
Cấu trúc project chuẩn
```

## Python

### Phase 7 — Project Structure
- `requirements.txt`

### Phase 8 — Data Structures
- `Stack`
- `Queue`

## C++

### Phase 6 — Object-Oriented Programming
- `new / delete`
- smart pointers:
  - `std::unique_ptr`
  - `std::shared_ptr`
- `enum`

## Computer Vision

### Phase 4 — Camera Geometry
- `Pinhole Camera Model`
- `Intrinsic Matrix`

---

# 2. Mô tả

Bài 31 yêu cầu bạn xây một mini-project cho humanoid robot perception để quản lý các tác vụ camera geometry bằng **Stack / Queue**.

Robot sẽ có nhiều tác vụ nhỏ:

```text
LOAD_IMAGE
VALIDATE_INTRINSICS
PROJECT_3D_TO_PIXEL
CHECK_PIXEL_BOUNDARY
SAVE_REPORT
```

Thay vì chạy code lung tung trong `main.cpp`, bạn sẽ tạo một **task scheduler**:

- `Queue` dùng cho tác vụ perception chạy theo thứ tự FIFO
- `Stack` dùng cho lịch sử thao tác / undo / debug trace
- `enum class` dùng để định nghĩa loại task
- `smart pointer` dùng để quản lý object task an toàn hơn
- `Intrinsic Matrix K` dùng để project điểm 3D sang pixel 2D

<p align="center">
  <img src="../../images/project_31.png" width="800">
</p>

---

# 3. Mục tiêu perception

Sau bài này, bạn phải hiểu pipeline:

```text
Python config builder
→ tạo camera_intrinsics_config.txt
→ tạo camera_task_queue.txt
→ C++ load task queue
→ push task vào std::queue
→ execute task theo FIFO
→ lưu lịch sử vào std::stack
→ project 3D point bằng intrinsic matrix
→ ghi report
```

---

# 4. Pipeline

```text
Config
→ Load Intrinsic Matrix K
→ Load 3D Point List
→ Load Camera Task Queue
→ Build Task Objects
→ Push Tasks into Queue
→ While Queue Not Empty:
    → Pop Front Task
    → Execute Task
    → Push Task Result into History Stack
→ Export Execution History
→ Export Projection Report
```

---

# 5. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- `requirements.txt`
- file handling
- list
- dict
- stack bằng list
- queue bằng `collections.deque`
- `if __name__ == "__main__"`

## C++
- class / inheritance
- constructor / destructor
- `enum class`
- `std::unique_ptr`
- `std::shared_ptr`
- `std::vector`
- `std::queue`
- `std::stack`
- `std::unordered_map`
- pass by reference
- `const reference`
- `virtual` / `override`
- `.hpp / .cpp`

## Computer Vision / Geometry
- pinhole camera model
- intrinsic matrix:
```text
K = [ fx  0  cx
      0  fy  cy
      0   0   1 ]
```

- projection:
```text
u = fx * X / Z + cx
v = fy * Y / Z + cy
```

---

# 6. DSA + Algorithm bắt buộc

Từ Bài 31 trở đi, mỗi project phải có phần thuật toán rõ hơn.

## DSA bắt buộc trong bài này

### 1. Queue
Dùng để quản lý task perception:

```cpp
std::queue<std::unique_ptr<BaseCameraTask>>
```

Ý nghĩa:

```text
task nào vào trước → chạy trước
```

---

### 2. Stack
Dùng để lưu execution history:

```cpp
std::stack<TaskExecutionRecord>
```

Ý nghĩa:

```text
task chạy sau cùng nằm trên cùng
có thể pop ra để debug ngược lại
```

---

### 3. Vector
Dùng để lưu danh sách 3D points:

```cpp
std::vector<Point3D>
```

---

### 4. Unordered Map
Dùng để map task type string sang enum:

```cpp
std::unordered_map<std::string, CameraTaskType>
```

---

## Algorithm bắt buộc

### Algorithm 1 — Parse Task Queue
```text
read task file
→ parse từng dòng
→ convert task name sang enum
→ tạo task object tương ứng
→ push vào queue
```

### Algorithm 2 — Execute FIFO Task Queue
```text
while queue not empty:
    task = front
    pop
    execute
    push execution record vào stack
```

### Algorithm 3 — Project 3D Points
```text
for each point:
    if Z <= 0 → invalid
    else:
        u = fx * X / Z + cx
        v = fy * Y / Z + cy
        check pixel inside image boundary
```

### Algorithm 4 — Reverse Debug History
```text
while history stack not empty:
    top
    pop
    write reverse execution order
```

---

# 7. Cấu trúc folder

```text
mini_project_31_pinhole_camera_task_scheduler/
│
├─ README.md
│
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ camera_projection_report.txt
│     ├─ task_execution_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ camera_intrinsics_config.txt
│  ├─ camera_image_config.txt
│  ├─ point3d_list.txt
│  └─ camera_task_queue.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ task_queue_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ CameraIntrinsics.hpp
   │  ├─ CameraImageConfig.hpp
   │  ├─ Point3D.hpp
   │  ├─ Pixel2D.hpp
   │  ├─ CameraTaskType.hpp
   │  ├─ TaskExecutionRecord.hpp
   │  ├─ BaseCameraTask.hpp
   │  ├─ ValidateIntrinsicsTask.hpp
   │  ├─ ProjectPointsTask.hpp
   │  ├─ CheckBoundaryTask.hpp
   │  ├─ CameraTaskScheduler.hpp
   │  └─ CameraReportWriter.hpp
   │
   └─ src/
      ├─ ValidateIntrinsicsTask.cpp
      ├─ ProjectPointsTask.cpp
      ├─ CheckBoundaryTask.cpp
      ├─ CameraTaskScheduler.cpp
      └─ CameraReportWriter.cpp
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

## Thuộc tính

```python
project_name
intrinsics_config_path
image_config_path
point3d_list_path
task_queue_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in các path config

### `@staticmethod validate_positive_number(value, field_name)`
- kiểm tra số dương

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

---

# 8.2 Python — `CameraTaskConfigBuilder`

Tạo class con:

```python
class CameraTaskConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
intrinsics
image_config
point3d_list
task_queue
```

## Hàm cần có

### `set_intrinsics(fx, fy, cx, cy)`
- kiểm tra `fx > 0`, `fy > 0`

### `set_image_config(width, height)`
- kiểm tra width / height > 0

### `add_point3d(name, x, y, z)`
- thêm điểm 3D
- nếu `z <= 0` vẫn cho ghi, nhưng đánh dấu để C++ xử lý invalid

### `add_task(task_name)`
- dùng Python `Queue` hoặc list để lưu task
- chỉ nhận:
  - `VALIDATE_INTRINSICS`
  - `PROJECT_POINTS`
  - `CHECK_BOUNDARY`
  - `SAVE_REPORT`

### `write_intrinsics_config()`

Format:

```text
fx=600
fy=600
cx=320
cy=240
```

### `write_image_config()`

Format:

```text
width=640
height=480
```

### `write_point3d_list()`

Format:

```text
name=target_01,x=0.2,y=0.1,z=1.0
name=target_02,x=-0.3,y=0.2,z=2.0
```

### `write_task_queue()`

Format:

```text
VALIDATE_INTRINSICS
PROJECT_POINTS
CHECK_BOUNDARY
SAVE_REPORT
```

---

# 8.3 Python — `task_queue_builder.py`

Tạo class:

```python
class PythonTaskQueuePreview:
```

## DSA Python bắt buộc

Dùng:

```python
from collections import deque
```

## Hàm cần có

### `push_task(task_name)`
- push task vào queue

### `pop_task()`
- pop theo FIFO

### `preview_execution_order()`
- trả list task theo thứ tự chạy

### `build_debug_stack()`
- dùng list như stack để lưu task đã chạy thử

---

# 8.4 C++ — `CameraIntrinsics`

**File:**

```text
cpp/include/CameraIntrinsics.hpp
```

```cpp
struct CameraIntrinsics
{
    double fx;
    double fy;
    double cx;
    double cy;

    bool is_valid() const;
};
```

---

# 8.5 C++ — `CameraImageConfig`

```cpp
struct CameraImageConfig
{
    int width;
    int height;

    bool is_inside(int u, int v) const;
};
```

---

# 8.6 C++ — `Point3D`

```cpp
struct Point3D
{
    std::string name;
    double x;
    double y;
    double z;
};
```

---

# 8.7 C++ — `Pixel2D`

```cpp
struct Pixel2D
{
    std::string point_name;
    double u;
    double v;
    bool is_valid;
    bool inside_image;
};
```

---

# 8.8 C++ — `CameraTaskType`

**File:**

```text
cpp/include/CameraTaskType.hpp
```

Tạo enum:

```cpp
enum class CameraTaskType
{
    VALIDATE_INTRINSICS,
    PROJECT_POINTS,
    CHECK_BOUNDARY,
    SAVE_REPORT,
    UNKNOWN
};
```

---

# 8.9 C++ — `TaskExecutionRecord`

```cpp
struct TaskExecutionRecord
{
    CameraTaskType task_type;
    std::string task_name;
    bool success;
    std::string message;
};
```

---

# 8.10 C++ — `BaseCameraTask`

Tạo abstract class:

```cpp
class BaseCameraTask
{
public:
    virtual TaskExecutionRecord execute() = 0;
    virtual CameraTaskType type() const = 0;
    virtual std::string name() const = 0;
    virtual ~BaseCameraTask() = default;
};
```

---

# 8.11 C++ — `ValidateIntrinsicsTask`

Kế thừa:

```cpp
class ValidateIntrinsicsTask : public BaseCameraTask
```

## Constructor

```cpp
ValidateIntrinsicsTask(const CameraIntrinsics& intrinsics);
```

## Hàm override

```cpp
TaskExecutionRecord execute() override;
CameraTaskType type() const override;
std::string name() const override;
```

## Hành vi
- kiểm tra `fx > 0`, `fy > 0`
- nếu invalid → fail

---

# 8.12 C++ — `ProjectPointsTask`

Kế thừa:

```cpp
class ProjectPointsTask : public BaseCameraTask
```

## Constructor

```cpp
ProjectPointsTask(
    const CameraIntrinsics& intrinsics,
    const std::vector<Point3D>& points,
    std::shared_ptr<std::vector<Pixel2D>> projected_pixels
);
```

## Hành vi
- project từng point 3D sang pixel
- nếu `z <= 0` → pixel invalid
- lưu kết quả vào `projected_pixels`

---

# 8.13 C++ — `CheckBoundaryTask`

Kế thừa:

```cpp
class CheckBoundaryTask : public BaseCameraTask
```

## Constructor

```cpp
CheckBoundaryTask(
    const CameraImageConfig& image_config,
    std::shared_ptr<std::vector<Pixel2D>> projected_pixels
);
```

## Hành vi
- duyệt `projected_pixels`
- kiểm tra pixel có nằm trong ảnh không
- cập nhật `inside_image`

---

# 8.14 C++ — `CameraTaskScheduler`

**File:**

```text
cpp/include/CameraTaskScheduler.hpp
cpp/src/CameraTaskScheduler.cpp
```

## Thuộc tính

```cpp
private:
    CameraIntrinsics intrinsics;
    CameraImageConfig image_config;
    std::vector<Point3D> points;

    std::queue<std::unique_ptr<BaseCameraTask>> task_queue;
    std::stack<TaskExecutionRecord> history_stack;

    std::shared_ptr<std::vector<Pixel2D>> projected_pixels;

    std::unordered_map<std::string, CameraTaskType> task_type_map;
```

## Hàm cần có

### `load_intrinsics_config(const std::string& path)`
- load `fx, fy, cx, cy`

### `load_image_config(const std::string& path)`
- load width / height

### `load_point3d_list(const std::string& path)`
- load vector point

### `load_task_queue(const std::string& path)`
- parse task string
- convert sang enum
- tạo task object bằng `std::make_unique`
- push vào queue

### `CameraTaskType parse_task_type(const std::string& task_name) const`
- dùng `unordered_map`

### `void run()`
Algorithm FIFO:

```text
while task_queue not empty:
    task = move(front)
    pop
    result = task->execute()
    history_stack.push(result)
```

### `std::vector<TaskExecutionRecord> get_history_in_reverse_order()`
- pop từ stack ra vector

### Getter

```cpp
const std::vector<Pixel2D>& get_projected_pixels() const;
```

---

# 8.15 C++ — `CameraReportWriter`

Tạo class:

```cpp
class CameraReportWriter
```

## Hàm cần có

### `write_projection_report(...)`

Format:

```text
[Projected Pixel]
Point: target_01
u: 440
v: 300
Valid: true
Inside Image: true
```

### `write_task_execution_report(...)`

Format:

```text
[Task Execution]
Task: VALIDATE_INTRINSICS
Success: true
Message: Intrinsics valid
```

### `write_reverse_debug_history(...)`

Format:

```text
[Reverse Debug History]
SAVE_REPORT
CHECK_BOUNDARY
PROJECT_POINTS
VALIDATE_INTRINSICS
```

---

# 8.16 C++ — `main.cpp`

## Yêu cầu

```text
Create CameraTaskScheduler
→ Load intrinsics config
→ Load image config
→ Load point3d list
→ Load task queue
→ Run scheduler
→ Write projection report
→ Write task execution report
→ Write reverse debug history
```

---

# 9. Điều kiện bắt buộc

Project bắt buộc có:

- OOP Python
- OOP C++
- inheritance Python
- inheritance C++
- Python `requirements.txt`
- Python `deque`
- Python module
- `if __name__ == "__main__"`
- C++ `enum class`
- C++ `std::queue`
- C++ `std::stack`
- C++ `std::vector`
- C++ `std::unordered_map`
- C++ `std::unique_ptr`
- C++ `std::shared_ptr`
- C++ `virtual` / `override`
- C++ pass by reference / const reference
- pinhole projection algorithm
- FIFO task execution algorithm
- reverse stack debug algorithm

---

# 10. Output mong muốn

## Config

```text
config/camera_intrinsics_config.txt
config/camera_image_config.txt
config/point3d_list.txt
config/camera_task_queue.txt
```

## Reports

```text
assets/outputs/camera_projection_report.txt
assets/outputs/task_execution_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 11. Vai trò trong Humanoid Robot

## Python

```text
Project Config Builder + Task Queue Builder
```

Python tạo config và kiểm thử thứ tự task.

## C++

```text
Runtime Camera Geometry Task Scheduler
```

C++ chạy scheduler, project điểm 3D, quản lý task bằng STL containers.

## Computer Vision

```text
Pinhole Camera Model + Intrinsic Matrix + 3D→2D Projection
```

CV geometry giúp robot biết một điểm 3D sẽ xuất hiện ở pixel nào trên ảnh.

---

# 12. Checklist hoàn thành

- [ ] Python tạo `requirements.txt`
- [ ] Python tạo đủ 4 config
- [ ] Python dùng `deque`
- [ ] Python có class cha / class con
- [ ] C++ có `CameraTaskType enum class`
- [ ] C++ có `BaseCameraTask`
- [ ] C++ có ít nhất 3 task kế thừa
- [ ] C++ dùng `std::queue`
- [ ] C++ dùng `std::stack`
- [ ] C++ dùng `std::unordered_map`
- [ ] C++ dùng `std::unique_ptr`
- [ ] C++ dùng `std::shared_ptr`
- [ ] C++ project được 3D point sang pixel
- [ ] C++ ghi đủ 3 report

---

# 13. Gợi ý mở rộng

## 1. Thêm Priority Queue
Sau khi quen queue thường, nâng lên:

```cpp
std::priority_queue
```

để task quan trọng chạy trước.

## 2. Thêm Undo Task
Dùng stack để rollback kết quả projection cuối cùng.

## 3. Thêm Projection Error Analyzer
So sánh pixel projected với pixel ground truth.

## 4. Chuẩn bị cho Bài 32

Bài 32 nên đi tiếp theo hướng:

```text
Extrinsic Transform Command Queue
```

Tức là thêm:
- extrinsic matrix
- transform 3D point từ robot frame sang camera frame
- dùng queue để quản lý transform commands
- dùng STL containers và smart pointer mạnh hơn
