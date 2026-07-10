# 🤖 Bài 35: Stereo Depth Reconstruction Validator — Bộ kiểm chứng tái tạo độ sâu từ stereo cho Humanoid Robot AI Perception

> Mini Project số 35 trong **Đợt 7**  
> **Bài 35 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7**.  
> Nếu **Bài 34** đã giúp bạn:
>
> - đưa landmark 3D qua **frame graph**
> - project sang **left camera** và **right camera**
> - lấy **left pixel / right pixel**
> - tính **disparity**
>
> thì **Bài 35** sẽ đi thêm bước rất quan trọng của stereo perception:
>
> ```text
> left pixel + right pixel
> → disparity
> → reconstruct depth
> → so với depth ground truth từ landmark 3D
> → tính depth error
> ```
>
> Đây là bài bắt đầu đánh mạnh hơn vào **thuật toán stereo depth**, **validation**, **error analysis**, đồng thời vẫn giữ nhịp:
>
> - **Algorithms**
> - **DSA Structures**
> - **STL Containers**
> - **Advanced C++**
> - **robot perception runtime mindset**

---

# 📌 Mục lục

- [1. Bài 35 lấy gì từ Bài 31–34](#1-bài-35-lấy-gì-từ-bài-3134)
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

# 1. Bài 35 lấy gì từ Bài 31–34

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
Bài 35 sẽ dùng kết quả stereo projection để làm một bước quan trọng hơn:

```text
left pixel + right pixel + stereo intrinsics + baseline
→ disparity
→ depth reconstruction
→ so sánh với depth ground truth
→ tính sai số depth
→ phân loại chất lượng stereo estimate
```

Tức là bạn không dừng ở việc “landmark hiện lên ở đâu trên 2 ảnh” nữa, mà bắt đầu trả lời:

- stereo depth **ước lượng được bao nhiêu**
- depth thật của landmark là bao nhiêu
- sai số là bao nhiêu
- landmark nào đang được reconstruct tốt / kém

---

# 2. Mô tả

Giả sử robot có stereo rig:

- `left_camera`
- `right_camera`

và đã biết:
- `fx`
- `baseline`
- left/right intrinsics
- transform graph từ source frame tới 2 camera

Với mỗi landmark 3D, bạn sẽ:

## Bước 1
Dùng frame graph để đưa landmark sang `left_camera` và `right_camera`.

## Bước 2
Project landmark sang:
- `left pixel = (uL, vL)`
- `right pixel = (uR, vR)`

## Bước 3
Tính disparity:

```text
d = uL - uR
```

## Bước 4
Từ disparity, reconstruct depth:

```text
Z_est = fx * baseline / d
```

## Bước 5
Lấy **ground truth depth** từ landmark trong left camera frame:

```text
Z_gt = Z_left_camera
```

## Bước 6
Tính lỗi:

```text
depth_error = |Z_est - Z_gt|
relative_error = depth_error / |Z_gt|
```

## Bước 7
Phân loại kết quả:
- `GOOD`
- `WEAK`
- `INVALID`

<p align="center">
  <img src="images/project_35.png" width="800">
</p>

---

# 3. Mục tiêu perception

Sau bài này, bạn phải hiểu pipeline:

```text
Python config builder
→ build frame graph
→ build stereo intrinsics + baseline
→ build landmark list
→ build stereo depth validation request list

C++ runtime
→ tìm path tới left_camera và right_camera
→ transform landmark sang 2 camera frame
→ project sang 2 ảnh
→ tính disparity
→ reconstruct depth
→ lấy ground-truth depth
→ tính depth error
→ ghi depth validation report
```

---

# 4. Pipeline

```text
Load Frame Nodes
Load Transform Edges
Load Stereo Intrinsics
Load Landmark List
Load Depth Validation Requests

Build Frame Graph

For each request:
    find landmark

    path source → left_camera
    compose transform
    landmark -> left_camera
    project -> left pixel

    path source → right_camera
    compose transform
    landmark -> right_camera
    project -> right pixel

    if left/right valid:
        disparity = u_left - u_right

    if disparity valid:
        depth_est = fx * baseline / disparity
        depth_gt = Z_left_camera
        depth_error = abs(depth_est - depth_gt)
        relative_error = depth_error / abs(depth_gt)

    classify quality
    store result

Write reports
```

---

# 5. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- `requirements.txt`
- file handling
- dict / list
- graph preview
- queue / stack preview
- `if __name__ == "__main__"`

## C++
- class / inheritance
- `enum class`
- `std::queue`
- `std::stack`
- `std::vector`
- `std::unordered_map`
- `std::unordered_set`
- `std::shared_ptr`
- `std::unique_ptr`
- `std::optional` *(nếu muốn nâng cao)*
- `const reference`
- `virtual` / `override`
- BFS path finding
- transform composition
- stereo depth validation runtime

## Computer Vision / Geometry
- pinhole camera model
- stereo projection
- disparity
- stereo depth reconstruction:
```text
Z = fx * baseline / disparity
```
- depth error analysis

---

# 6. DSA + Algorithm bắt buộc

## DSA bắt buộc

### 1. `std::unordered_map<std::string, std::vector<std::string>>`
Làm adjacency list cho frame graph.

### 2. `std::queue<std::string>`
Dùng cho BFS path finding.

### 3. `std::unordered_map<std::string, std::string>`
Dùng cho `parent map`.

### 4. `std::stack<DepthDebugRecord>`
Dùng cho reverse debug history.

### 5. `std::vector<Point3D>`
Lưu landmark list.

### 6. `std::vector<StereoDepthValidationResult>`
Lưu kết quả validation.

---

## Algorithm bắt buộc

## Algorithm 1 — Stereo projection
Giống Bài 34:
- path source → left_camera
- path source → right_camera
- transform sang 2 camera
- project sang 2 ảnh

---

## Algorithm 2 — Disparity
Nếu cả hai pixel hợp lệ:

```text
d = uL - uR
```

### Rule tối thiểu
- nếu `d <= 0` → disparity invalid
- nếu left/right projection invalid → disparity invalid

---

## Algorithm 3 — Depth reconstruction
Nếu disparity hợp lệ:

```text
Z_est = fx * baseline / d
```

---

## Algorithm 4 — Ground-truth depth
Ground-truth depth lấy từ **tọa độ Z của landmark trong left camera frame**:

```text
Z_gt = left_camera_point.z
```

---

## Algorithm 5 — Error analysis
```text
depth_error = abs(Z_est - Z_gt)
relative_error = depth_error / abs(Z_gt)
```

### Gợi ý rule quality
- `GOOD` nếu `relative_error <= 0.10`
- `WEAK` nếu `0.10 < relative_error <= 0.25`
- `INVALID` nếu disparity / projection không hợp lệ hoặc `relative_error > 0.25`

---

## Algorithm 6 — Reverse debug history
Pop stack để ghi log ngược.

---

# 7. Cấu trúc folder

```text
mini_project_35_stereo_depth_reconstruction_validator/
│
├─ README.md
│
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ transform_path_report.txt
│     ├─ stereo_projection_report.txt
│     ├─ stereo_depth_validation_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ frame_node_list.txt
│  ├─ transform_edge_list.txt
│  ├─ stereo_intrinsics_config.txt
│  ├─ landmark_list.txt
│  └─ depth_validation_request_list.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ depth_preview.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ Pixel2D.hpp
   │  ├─ StereoIntrinsics.hpp
   │  ├─ TransformMatrix.hpp
   │  ├─ TransformEdge.hpp
   │  ├─ DepthValidationRequest.hpp
   │  ├─ StereoDepthValidationResult.hpp
   │  ├─ DepthDebugRecord.hpp
   │  ├─ FrameGraph.hpp
   │  ├─ BasePathFinder.hpp
   │  ├─ BFSPathFinder.hpp
   │  ├─ TransformComposer.hpp
   │  ├─ StereoDepthValidator.hpp
   │  └─ DepthReportWriter.hpp
   │
   └─ src/
      ├─ FrameGraph.cpp
      ├─ BFSPathFinder.cpp
      ├─ TransformComposer.cpp
      ├─ StereoDepthValidator.cpp
      └─ DepthReportWriter.cpp
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
frame_node_list_path
transform_edge_list_path
stereo_intrinsics_config_path
landmark_list_path
depth_validation_request_list_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in path config

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

### `@staticmethod validate_frame_name(name)`
- không rỗng

---

# 8.2 Python — `StereoDepthConfigBuilder`

Tạo class con:

```python
class StereoDepthConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
frame_nodes
transform_edges
stereo_intrinsics
landmarks
depth_requests
```

## Hàm cần có

### `add_frame_node(frame_name)`
### `add_transform_edge(parent_frame, child_frame, rotation_matrix, translation_vector)`

### `set_stereo_intrinsics(
    left_fx, left_fy, left_cx, left_cy,
    right_fx, right_fy, right_cx, right_cy,
    image_width, image_height, baseline
)`
- `baseline > 0`

### `add_landmark(name, source_frame, x, y, z)`

### `add_depth_request(request_name, landmark_name)`

### `write_frame_node_list()`
### `write_transform_edge_list()`
### `write_stereo_intrinsics_config()`
### `write_landmark_list()`
### `write_depth_request_list()`

---

# 8.3 Python — `depth_preview.py`

Tạo class:

```python
class PythonStereoDepthPreview:
```

## Hàm cần có

### `compute_disparity(u_left, u_right)`
### `reconstruct_depth(fx, baseline, disparity)`
### `estimate_relative_error(depth_est, depth_gt)`

---

# 8.4 C++ — `Point3D`

```cpp
struct Point3D
{
    std::string name;
    std::string source_frame;
    double x;
    double y;
    double z;
};
```

---

# 8.5 C++ — `Pixel2D`

```cpp
struct Pixel2D
{
    double u;
    double v;
    bool is_valid;
    bool inside_image;
};
```

---

# 8.6 C++ — `StereoIntrinsics`

```cpp
struct StereoIntrinsics
{
    double left_fx, left_fy, left_cx, left_cy;
    double right_fx, right_fy, right_cx, right_cy;
    int image_width;
    int image_height;
    double baseline;
};
```

---

# 8.7 C++ — `TransformMatrix`

```cpp
struct TransformMatrix
{
    double r11, r12, r13;
    double r21, r22, r23;
    double r31, r32, r33;
    double tx, ty, tz;

    static TransformMatrix identity();
    Point3D apply(const Point3D& p) const;
    TransformMatrix compose(const TransformMatrix& next) const;
};
```

---

# 8.8 C++ — `TransformEdge`

```cpp
struct TransformEdge
{
    std::string parent_frame;
    std::string child_frame;
    TransformMatrix transform;
};
```

---

# 8.9 C++ — `DepthValidationRequest`

```cpp
struct DepthValidationRequest
{
    std::string request_name;
    std::string landmark_name;
};
```

---

# 8.10 C++ — `StereoDepthValidationResult`

```cpp
struct StereoDepthValidationResult
{
    std::string request_name;
    std::string landmark_name;
    std::string source_frame;

    std::vector<std::string> path_to_left;
    std::vector<std::string> path_to_right;

    bool left_path_found;
    bool right_path_found;

    Point3D left_camera_point;
    Point3D right_camera_point;

    Pixel2D left_pixel;
    Pixel2D right_pixel;

    bool disparity_valid;
    double disparity;

    bool depth_valid;
    double depth_estimated;
    double depth_ground_truth;
    double depth_error;
    double relative_error;

    std::string quality_label; // GOOD / WEAK / INVALID
};
```

---

# 8.11 C++ — `DepthDebugRecord`

```cpp
struct DepthDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 8.12 C++ — `FrameGraph`

Giống Bài 33–34.

## Thuộc tính
```cpp
std::unordered_map<std::string, std::vector<std::string>> adjacency;
std::vector<TransformEdge> edges;
```

## Hàm cần có
- `add_edge(...)`
- `neighbors(...)`
- `has_frame(...)`
- `get_transform_between(from, to)`

---

# 8.13 C++ — `BasePathFinder`

```cpp
class BasePathFinder
{
public:
    virtual std::vector<std::string> find_path(
        const FrameGraph& graph,
        const std::string& source,
        const std::string& target
    ) = 0;

    virtual ~BasePathFinder() = default;
};
```

---

# 8.14 C++ — `BFSPathFinder`

Dùng:
- `std::queue`
- `std::unordered_set`
- `std::unordered_map`

để tìm path.

---

# 8.15 C++ — `TransformComposer`

## Hàm cần có

### `TransformMatrix compose_path_transform(
    const FrameGraph& graph,
    const std::vector<std::string>& path
) const;`

---

# 8.16 C++ — `StereoDepthValidator`

Tạo class:

```cpp
class StereoDepthValidator
```

## Thuộc tính

```cpp
private:
    FrameGraph graph;
    StereoIntrinsics intrinsics;
    std::vector<Point3D> landmarks;
    std::vector<DepthValidationRequest> requests;
    std::vector<StereoDepthValidationResult> results;

    std::shared_ptr<BasePathFinder> path_finder;
    std::stack<DepthDebugRecord> debug_history;
```

## Hàm cần có

### `load_frame_node_list(...)`
### `load_transform_edge_list(...)`
### `load_stereo_intrinsics_config(...)`
### `load_landmark_list(...)`
### `load_depth_request_list(...)`

### `const Point3D* find_landmark(const std::string& landmark_name) const`

### `Pixel2D project_left(const Point3D& camera_point) const`
### `Pixel2D project_right(const Point3D& camera_point) const`

### `double compute_disparity(const Pixel2D& left_pixel, const Pixel2D& right_pixel) const`
- trả `left.u - right.u`

### `double reconstruct_depth(double disparity) const`
- `left_fx * baseline / disparity`

### `std::string classify_quality(bool depth_valid, double relative_error) const`

### `void run()`
Với mỗi request:

1. tìm landmark
2. BFS path tới left/right camera
3. compose transform trái / phải
4. transform landmark sang left/right camera
5. project left/right pixel
6. compute disparity
7. reconstruct depth
8. lấy ground truth depth từ `left_camera_point.z`
9. tính error
10. classify quality
11. push result + debug record

### Getter

```cpp
const std::vector<StereoDepthValidationResult>& get_results() const;
std::vector<DepthDebugRecord> get_debug_history_reverse();
```

---

# 8.17 C++ — `DepthReportWriter`

Tạo class:

```cpp
class DepthReportWriter
```

## Hàm cần có

### `write_transform_path_report(...)`

Ví dụ:

```text
[Path]
Request: request_01
Path to Left: world -> base -> torso -> head -> left_camera
Path to Right: world -> base -> torso -> head -> right_camera
```

### `write_stereo_projection_report(...)`

Ví dụ:

```text
[Stereo Projection]
Landmark: landmark_01
Left Pixel: (421.2, 201.8)
Right Pixel: (392.5, 201.1)
Disparity: 28.7
```

### `write_depth_validation_report(...)`

Ví dụ:

```text
[Depth Validation]
Landmark: landmark_01
Depth Estimated: 1.83
Depth Ground Truth: 1.79
Depth Error: 0.04
Relative Error: 0.0223
Quality: GOOD
```

### `write_reverse_debug_history(...)`

Ví dụ:

```text
[Reverse Debug History]
request_03: disparity invalid
request_02: weak depth estimate
request_01: good depth reconstruction
```

---

# 8.18 C++ — `main.cpp`

## Yêu cầu

```text
Create StereoDepthValidator
→ Load frame nodes
→ Load transform edges
→ Load stereo intrinsics
→ Load landmark list
→ Load depth request list
→ Run
→ Write transform path report
→ Write stereo projection report
→ Write depth validation report
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
- Python preview utility
- C++ `queue`
- C++ `stack`
- C++ `vector`
- C++ `unordered_map`
- C++ `unordered_set`
- BFS path finding
- transform composition
- disparity computation
- depth reconstruction
- error analysis
- quality classification

---

# 10. Output mong muốn

## Config

```text
config/frame_node_list.txt
config/transform_edge_list.txt
config/stereo_intrinsics_config.txt
config/landmark_list.txt
config/depth_validation_request_list.txt
```

## Reports

```text
assets/outputs/transform_path_report.txt
assets/outputs/stereo_projection_report.txt
assets/outputs/stereo_depth_validation_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 11. Vai trò trong Humanoid Robot

## Python

```text
Stereo Depth Validation Config Builder + Preview Tool
```

## C++

```text
Stereo Depth Validation Runtime
```

Đây là một bước rất quan trọng vì bạn đang bắt đầu làm đúng việc của perception engineer:

- lấy observation stereo
- reconstruct depth
- so với ground truth
- đo chất lượng thuật toán

## Computer Vision / Geometry

```text
landmark in source frame
→ left/right projection
→ disparity
→ depth reconstruction
→ error analysis
```

---

# 12. Checklist hoàn thành

- [ ] Python tạo đủ 5 config
- [ ] Python có depth preview utility
- [ ] C++ có `StereoDepthValidator`
- [ ] C++ tìm được path trái/phải
- [ ] C++ project được left/right pixel
- [ ] C++ tính disparity
- [ ] C++ reconstruct được depth
- [ ] C++ tính depth error
- [ ] C++ classify quality
- [ ] C++ ghi đủ 4 report

---

# 13. Gợi ý mở rộng

## 1. Thêm noise model
Tự thêm nhiễu vào `u_left`, `u_right` để xem depth error tăng ra sao.

## 2. Thêm nhiều camera pair
- `head_left` / `head_right`
- `wrist_left` / `wrist_right`

## 3. Thêm histogram depth error
Sau này có thể dùng Python vẽ histogram.

## 4. Chuẩn bị cho Bài 36

Bài 36 nên đi tiếp theo hướng:

```text
Stereo Disparity Track Manager
```

Tức là thay vì chỉ reconstruct depth cho **một snapshot landmark**, bạn sẽ:
- theo dõi landmark qua nhiều frame
- lưu disparity history
- quản lý track bằng STL containers
- tính moving average / outlier rejection
- bắt đầu đi sang **temporal stereo perception**.
