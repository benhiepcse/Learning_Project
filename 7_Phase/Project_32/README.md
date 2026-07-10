# 🤖 Bài 32: Extrinsic Transform Command Queue — Hàng đợi lệnh biến đổi ngoại tại cho Humanoid Robot AI Perception

> Mini Project số 32 trong **Đợt 7**  
> **Bài 32 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7**.  
> Từ **Đợt 7 trở đi**, project không chỉ dừng ở “viết class cho đúng OOP” nữa, mà phải bắt đầu có cảm giác của một **runtime perception thật**:
>
> - có **task / command queue**
> - có **data flow**
> - có **STL containers**
> - có **smart pointers**
> - có **thuật toán xử lý frame / point / transform**
> - có **camera geometry + robot geometry**

---

# 📌 Mục lục

- [1. Bài 32 lấy gì từ Đợt 7](#1-bài-32-lấy-gì-từ-đợt-7)
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

# 1. Bài 32 lấy gì từ Đợt 7

Bài 31 đã dùng:

- **Queue** để quản lý camera tasks
- **Stack** để lưu lịch sử chạy task
- **Pinhole Camera Model**
- **Intrinsic Matrix**
- **smart pointer + enum class + STL containers**

Bài 32 nâng tiếp lên một bước rất quan trọng trong robot perception:

```text
3D point trong robot/world frame
→ transform sang camera frame bằng extrinsic transform
→ project sang ảnh bằng intrinsic matrix
```

Tức là từ **“camera nội tại”** ở Bài 31, bạn chuyển sang **“camera + robot frame relationship”**.

## Trọng tâm của Bài 32
- **Extrinsic Transform**
- **Command Queue**
- **Matrix-like rigid transform logic**
- **queue + stack + vector + unordered_map**
- **smart pointer**
- **Advanced C++ style runtime**

---

# 2. Mô tả

Trong robot thật, một điểm 3D thường **không nằm sẵn trong camera frame**.  
Nó có thể đang ở:

- `world frame`
- `robot base frame`
- `arm frame`
- `head frame`

Muốn biết nó xuất hiện ở pixel nào, robot phải đi qua 2 bước:

## Bước 1 — Extrinsic transform
Biến điểm từ frame nguồn sang **camera frame**

```text
P_camera = R * P_source + t
```

## Bước 2 — Intrinsic projection
Chiếu từ camera frame lên ảnh

```text
u = fx * Xc / Zc + cx
v = fy * Yc / Zc + cy
```

**Bài 32** sẽ bắt bạn xây một **Extrinsic Transform Command Queue**:

- đọc danh sách lệnh transform / projection
- đưa các lệnh vào queue
- thực thi theo thứ tự
- lưu execution history bằng stack
- quản lý point cloud nhỏ bằng vector
- ghi report đầy đủ

---

# 3. Mục tiêu perception

Sau bài này, bạn phải hiểu pipeline:

```text
Python config builder
→ tạo camera intrinsics
→ tạo camera extrinsics
→ tạo source 3D point list
→ tạo command queue

C++ runtime
→ load intrinsics / extrinsics / points / command queue
→ push command objects vào std::queue
→ execute transform + projection pipeline
→ lưu transformed camera points
→ lưu projected pixels
→ push history vào stack
→ xuất report
```

---

# 4. Pipeline

```text
Load Intrinsics K
Load Extrinsic Transform (R, t)
Load Source 3D Points
Load Command Queue

Build shared data buffers:
    source_points
    transformed_camera_points
    projected_pixels

Push commands into queue

while queue not empty:
    pop front command
    execute command
    push execution record into stack

Write:
    transform report
    projection report
    reverse debug history
```

---

# 5. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- `requirements.txt`
- file handling
- list / dict
- stack / queue preview
- `if __name__ == "__main__"`

## C++
- class / inheritance
- `enum class`
- `std::queue`
- `std::stack`
- `std::vector`
- `std::unordered_map`
- `std::unique_ptr`
- `std::shared_ptr`
- `const reference`
- `virtual` / `override`
- constructor / destructor
- command pattern cơ bản

## Geometry / CV
- intrinsic matrix
- extrinsic transform
- rigid transform:
```text
P_camera = R * P_source + t
```

- pinhole projection:
```text
u = fx * Xc / Zc + cx
v = fy * Yc / Zc + cy
```

---

# 6. DSA + Algorithm bắt buộc

## DSA bắt buộc

### 1. `std::queue<std::unique_ptr<BaseTransformCommand>>`
Dùng cho command execution theo FIFO.

### 2. `std::stack<CommandExecutionRecord>`
Dùng cho reverse debug history.

### 3. `std::vector<Point3D>`
Dùng cho source points và transformed points.

### 4. `std::unordered_map<std::string, TransformCommandType>`
Map từ string command sang enum.

### 5. `std::shared_ptr<std::vector<...>>`
Chia sẻ buffer giữa nhiều command.

---

## Algorithm bắt buộc

### Algorithm 1 — Parse command queue
```text
read file
→ parse command name
→ map sang enum
→ tạo command object
→ push queue
```

### Algorithm 2 — Extrinsic transform
Với mỗi source point `(x, y, z)`:

```text
Xc = r11*x + r12*y + r13*z + tx
Yc = r21*x + r22*y + r23*z + ty
Zc = r31*x + r32*y + r33*z + tz
```

### Algorithm 3 — Projection
Nếu `Zc <= 0` thì invalid.  
Ngược lại:

```text
u = fx * Xc / Zc + cx
v = fy * Yc / Zc + cy
```

### Algorithm 4 — Reverse history
Pop stack để lấy thứ tự command đã chạy theo chiều ngược.

---

# 7. Cấu trúc folder

```text
mini_project_32_extrinsic_transform_command_queue/
│
├─ README.md
│
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ transformed_camera_points_report.txt
│     ├─ projected_pixels_report.txt
│     ├─ command_execution_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ camera_intrinsics_config.txt
│  ├─ camera_extrinsics_config.txt
│  ├─ source_point3d_list.txt
│  └─ transform_command_queue.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ command_queue_builder.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ CameraIntrinsics.hpp
   │  ├─ CameraExtrinsics.hpp
   │  ├─ Point3D.hpp
   │  ├─ Pixel2D.hpp
   │  ├─ TransformCommandType.hpp
   │  ├─ CommandExecutionRecord.hpp
   │  ├─ BaseTransformCommand.hpp
   │  ├─ ValidateExtrinsicsCommand.hpp
   │  ├─ TransformPointsCommand.hpp
   │  ├─ ProjectCameraPointsCommand.hpp
   │  ├─ CheckProjectionBoundaryCommand.hpp
   │  ├─ TransformCommandScheduler.hpp
   │  └─ TransformReportWriter.hpp
   │
   └─ src/
      ├─ ValidateExtrinsicsCommand.cpp
      ├─ TransformPointsCommand.cpp
      ├─ ProjectCameraPointsCommand.cpp
      ├─ CheckProjectionBoundaryCommand.cpp
      ├─ TransformCommandScheduler.cpp
      └─ TransformReportWriter.cpp
```

---

# 8. Yêu cầu mini-project

# 8.1 Python — `BaseConfigBuilder`

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
intrinsics_config_path
extrinsics_config_path
source_points_path
command_queue_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in path config

### `@staticmethod validate_positive_number(value, field_name)`
- dùng cho `fx`, `fy`

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

---

# 8.2 Python — `ExtrinsicTransformConfigBuilder`

Tạo class con:

```python
class ExtrinsicTransformConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
intrinsics
extrinsics
source_points
command_queue
```

## Hàm cần có

### `set_intrinsics(fx, fy, cx, cy)`
- validate `fx > 0`, `fy > 0`

### `set_extrinsics(rotation_matrix, translation_vector)`
- `rotation_matrix` là 3x3 list
- `translation_vector` là `[tx, ty, tz]`

### `add_source_point(name, x, y, z)`
- thêm point

### `add_command(command_name)`
Chỉ nhận:
- `VALIDATE_EXTRINSICS`
- `TRANSFORM_POINTS`
- `PROJECT_CAMERA_POINTS`
- `CHECK_PROJECTION_BOUNDARY`
- `SAVE_REPORT`

### `write_intrinsics_config()`
### `write_extrinsics_config()`
### `write_source_points()`
### `write_command_queue()`

---

# 8.3 Python — `command_queue_builder.py`

Tạo class:

```python
class PythonTransformCommandQueuePreview:
```

## Bắt buộc dùng
```python
from collections import deque
```

## Hàm cần có

### `push_command(command_name)`
### `pop_command()`
### `preview_execution_order()`
### `build_debug_stack()`

---

# 8.4 C++ — `CameraIntrinsics`

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

# 8.5 C++ — `CameraExtrinsics`

```cpp
struct CameraExtrinsics
{
    double r11, r12, r13;
    double r21, r22, r23;
    double r31, r32, r33;

    double tx, ty, tz;

    bool is_valid() const;
};
```

## Rule tối thiểu cho `is_valid()`
- không để toàn bộ ma trận và tịnh tiến bằng 0
- bạn có thể kiểm tra gần đúng `det(R) != 0` nếu muốn nâng cao

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

# 8.8 C++ — `TransformCommandType`

```cpp
enum class TransformCommandType
{
    VALIDATE_EXTRINSICS,
    TRANSFORM_POINTS,
    PROJECT_CAMERA_POINTS,
    CHECK_PROJECTION_BOUNDARY,
    SAVE_REPORT,
    UNKNOWN
};
```

---

# 8.9 C++ — `CommandExecutionRecord`

```cpp
struct CommandExecutionRecord
{
    TransformCommandType command_type;
    std::string command_name;
    bool success;
    std::string message;
};
```

---

# 8.10 C++ — `BaseTransformCommand`

```cpp
class BaseTransformCommand
{
public:
    virtual CommandExecutionRecord execute() = 0;
    virtual TransformCommandType type() const = 0;
    virtual std::string name() const = 0;
    virtual ~BaseTransformCommand() = default;
};
```

---

# 8.11 C++ — `ValidateExtrinsicsCommand`

Kế thừa:

```cpp
class ValidateExtrinsicsCommand : public BaseTransformCommand
```

## Constructor

```cpp
ValidateExtrinsicsCommand(const CameraExtrinsics& extrinsics);
```

## Hành vi
- kiểm tra extrinsics hợp lệ
- nếu invalid → fail

---

# 8.12 C++ — `TransformPointsCommand`

Kế thừa:

```cpp
class TransformPointsCommand : public BaseTransformCommand
```

## Constructor

```cpp
TransformPointsCommand(
    const CameraExtrinsics& extrinsics,
    const std::vector<Point3D>& source_points,
    std::shared_ptr<std::vector<Point3D>> transformed_camera_points
);
```

## Hành vi
- duyệt từng source point
- transform sang camera frame:
```text
Xc = r11*x + r12*y + r13*z + tx
Yc = r21*x + r22*y + r23*z + ty
Zc = r31*x + r32*y + r33*z + tz
```
- lưu vào `transformed_camera_points`

---

# 8.13 C++ — `ProjectCameraPointsCommand`

Kế thừa:

```cpp
class ProjectCameraPointsCommand : public BaseTransformCommand
```

## Constructor

```cpp
ProjectCameraPointsCommand(
    const CameraIntrinsics& intrinsics,
    std::shared_ptr<std::vector<Point3D>> transformed_camera_points,
    std::shared_ptr<std::vector<Pixel2D>> projected_pixels
);
```

## Hành vi
- duyệt camera points
- nếu `z <= 0` → invalid
- ngược lại project sang pixel

---

# 8.14 C++ — `CheckProjectionBoundaryCommand`

Kế thừa:

```cpp
class CheckProjectionBoundaryCommand : public BaseTransformCommand
```

## Constructor

```cpp
CheckProjectionBoundaryCommand(
    int image_width,
    int image_height,
    std::shared_ptr<std::vector<Pixel2D>> projected_pixels
);
```

## Hành vi
- kiểm tra pixel có nằm trong ảnh không

---

# 8.15 C++ — `TransformCommandScheduler`

**File**
```text
cpp/include/TransformCommandScheduler.hpp
cpp/src/TransformCommandScheduler.cpp
```

## Thuộc tính

```cpp
private:
    CameraIntrinsics intrinsics;
    CameraExtrinsics extrinsics;
    std::vector<Point3D> source_points;

    std::queue<std::unique_ptr<BaseTransformCommand>> command_queue;
    std::stack<CommandExecutionRecord> history_stack;

    std::shared_ptr<std::vector<Point3D>> transformed_camera_points;
    std::shared_ptr<std::vector<Pixel2D>> projected_pixels;

    std::unordered_map<std::string, TransformCommandType> command_type_map;
```

## Hàm cần có

### `load_intrinsics_config(const std::string& path)`
### `load_extrinsics_config(const std::string& path)`
### `load_source_points(const std::string& path)`
### `load_command_queue(const std::string& path)`

### `TransformCommandType parse_command_type(const std::string& command_name) const`
- dùng `unordered_map`

### `void run()`
```text
while command_queue not empty:
    cmd = move(front)
    pop
    result = cmd->execute()
    history_stack.push(result)
```

### `std::vector<CommandExecutionRecord> get_history_in_reverse_order()`
- pop stack ra vector

### Getter

```cpp
const std::vector<Point3D>& get_transformed_camera_points() const;
const std::vector<Pixel2D>& get_projected_pixels() const;
```

---

# 8.16 C++ — `TransformReportWriter`

Tạo class:

```cpp
class TransformReportWriter
```

## Hàm cần có

### `write_transformed_points_report(...)`

Format gợi ý:

```text
[Camera Frame Point]
Point: landmark_01
Xc: 0.32
Yc: -0.15
Zc: 1.84
```

### `write_projected_pixels_report(...)`

Format gợi ý:

```text
[Projected Pixel]
Point: landmark_01
u: 421.3
v: 198.7
Valid: true
Inside Image: true
```

### `write_command_execution_report(...)`

Format gợi ý:

```text
[Command Execution]
Command: TRANSFORM_POINTS
Success: true
Message: 6 points transformed to camera frame
```

### `write_reverse_debug_history(...)`

Format gợi ý:

```text
[Reverse Debug History]
SAVE_REPORT
CHECK_PROJECTION_BOUNDARY
PROJECT_CAMERA_POINTS
TRANSFORM_POINTS
VALIDATE_EXTRINSICS
```

---

# 8.17 C++ — `main.cpp`

## Yêu cầu

```text
Create TransformCommandScheduler
→ Load intrinsics
→ Load extrinsics
→ Load source points
→ Load command queue
→ Run scheduler
→ Write transformed points report
→ Write projected pixels report
→ Write command execution report
→ Write reverse debug history
```

---

# 9. Điều kiện bắt buộc

Project bắt buộc phải có:

- OOP Python
- OOP C++
- inheritance Python
- inheritance C++
- Python `requirements.txt`
- Python `deque`
- C++ `enum class`
- C++ `std::queue`
- C++ `std::stack`
- C++ `std::vector`
- C++ `std::unordered_map`
- C++ `std::unique_ptr`
- C++ `std::shared_ptr`
- `virtual` / `override`
- extrinsic transform algorithm
- pinhole projection algorithm
- FIFO command execution
- reverse debug stack

---

# 10. Output mong muốn

## Config

```text
config/camera_intrinsics_config.txt
config/camera_extrinsics_config.txt
config/source_point3d_list.txt
config/transform_command_queue.txt
```

## Reports

```text
assets/outputs/transformed_camera_points_report.txt
assets/outputs/projected_pixels_report.txt
assets/outputs/command_execution_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 11. Vai trò trong Humanoid Robot

## Python

```text
Transform Config Builder + Command Queue Builder
```

## C++

```text
Extrinsic Transform Runtime Scheduler
```

C++ sẽ đóng vai trò giống một perception runtime nhỏ:
- load geometry config
- transform point
- project point
- chạy command queue
- ghi history

## Computer Vision / Geometry

```text
source point in robot/world frame
→ extrinsic transform
→ camera frame
→ intrinsic projection
→ pixel
```

Đây là một mảnh rất quan trọng trong perception, vì robot không thể chỉ học intrinsic mà bỏ qua **quan hệ giữa frame robot và frame camera**.

---

# 12. Checklist hoàn thành

- [ ] Python tạo đủ config
- [ ] Python có queue preview bằng `deque`
- [ ] C++ có `TransformCommandType`
- [ ] C++ có `BaseTransformCommand`
- [ ] C++ có ít nhất 4 command kế thừa
- [ ] C++ dùng `std::queue`
- [ ] C++ dùng `std::stack`
- [ ] C++ dùng `std::unordered_map`
- [ ] C++ dùng `std::unique_ptr`
- [ ] C++ dùng `std::shared_ptr`
- [ ] C++ transform source points sang camera frame
- [ ] C++ project camera points sang pixel
- [ ] C++ ghi đủ 4 report

---

# 13. Gợi ý mở rộng

## 1. Thêm `std::priority_queue`
Cho command quan trọng chạy trước.

## 2. Thêm chain transform
Ví dụ:
```text
world → base → head → camera
```

## 3. Thêm batch nhiều camera
- left camera
- right camera

## 4. Chuẩn bị cho Bài 33

Bài 33 nên đi tiếp theo hướng:

```text
Transformation Graph Path Finder
```

Tức là thay vì chỉ có **1 extrinsic transform**, robot sẽ có nhiều frame:

- `world`
- `base`
- `torso`
- `head`
- `left_camera`
- `right_camera`

và phải dùng **graph + BFS / path reconstruction** để tìm chuỗi transform giữa 2 frame.
