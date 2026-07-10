# 🤖 Bài 34: Stereo Frame Graph Landmark Projector — Chiếu landmark qua frame graph sang camera trái/phải cho Humanoid Robot AI Perception

> Mini Project số 34 trong **Đợt 7**  
> **Bài 34 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7**.  
> Đây là bài nối rất mạnh giữa **robot frame graph** và **stereo perception**.  
> Nếu **Bài 33** đã giúp bạn:
>
> - build **transformation graph**
> - dùng **BFS** tìm path giữa các frame
> - compose transform dọc theo path
>
> thì **Bài 34** sẽ dùng chính graph đó để đưa **một landmark 3D** từ `world/base frame` sang:
>
> - `left_camera`
> - `right_camera`
>
> rồi project sang **ảnh trái / ảnh phải**, từ đó tính:
>
> - **left pixel**
> - **right pixel**
> - **disparity cơ bản**
> - **stereo observation report**

---

# 📌 Mục lục

- [1. Bài 34 lấy gì từ Bài 31–33](#1-bài-34-lấy-gì-từ-bài-3133)
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

# 1. Bài 34 lấy gì từ Bài 31–33

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
- compose transform chain giữa nhiều frame

## Bài 34
Bài 34 sẽ nối **graph transform** với **stereo camera geometry**:

```text
landmark trong world/base frame
→ path tới left_camera
→ path tới right_camera
→ transform sang 2 camera frame
→ project sang ảnh trái / phải
→ tính disparity
```

Đây là bước rất đúng chất **Humanoid Robot AI Perception**, vì robot không chỉ cần biết “điểm ở frame nào”, mà còn cần biết:

- nó xuất hiện ở **camera trái** ở pixel nào
- ở **camera phải** ở pixel nào
- chênh lệch disparity là bao nhiêu
- stereo observation có hợp lệ không

---

# 2. Mô tả

Giả sử robot có frame graph:

```text
world
 └─ base
     └─ torso
         └─ head
             ├─ left_camera
             └─ right_camera
```

Robot cũng có một số landmark 3D trong `world` hoặc `base` frame:

- `landmark_01`
- `landmark_02`
- `landmark_03`

Nhiệm vụ của bài:

## Với mỗi landmark:
1. tìm path từ source frame → `left_camera`
2. compose transform và đưa landmark sang **left camera frame**
3. project sang **left image**
4. tìm path từ source frame → `right_camera`
5. compose transform và đưa landmark sang **right camera frame**
6. project sang **right image**
7. nếu cả hai đều hợp lệ:
   - tính `disparity = u_left - u_right`
8. ghi stereo report

---

# 3. Mục tiêu perception

Sau bài này, bạn phải hiểu pipeline:

```text
Python config builder
→ tạo frame graph
→ tạo stereo camera intrinsics
→ tạo landmark list
→ tạo stereo observation request list

C++ runtime
→ build frame graph
→ tìm path source → left_camera
→ tìm path source → right_camera
→ compose transform chain
→ transform landmark sang 2 camera frames
→ project sang 2 ảnh
→ tính disparity
→ ghi stereo observation report
```

---

# 4. Pipeline

```text
Load Frame Nodes
Load Transform Edges
Load Stereo Intrinsics
Load Landmark List
Load Stereo Observation Requests

Build Frame Graph

For each request:
    source_frame, landmark_name

    Step 1:
        BFS path source_frame → left_camera
        compose transform
        landmark -> left camera frame
        project to left image

    Step 2:
        BFS path source_frame → right_camera
        compose transform
        landmark -> right camera frame
        project to right image

    Step 3:
        if left and right valid:
            disparity = u_left - u_right

    Step 4:
        store stereo observation result

Write:
    path report
    left/right projection report
    stereo disparity report
    reverse debug history
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
- `const reference`
- `virtual` / `override`
- BFS path finding
- transform composition
- stereo observation runtime

## Computer Vision / Geometry
- pinhole camera model
- intrinsic matrix
- stereo camera concept
- left/right projection
- disparity:
```text
d = u_left - u_right
```

---

# 6. DSA + Algorithm bắt buộc

## DSA bắt buộc

### 1. `std::unordered_map<std::string, std::vector<std::string>>`
Làm adjacency list cho frame graph.

### 2. `std::queue<std::string>`
Dùng cho BFS path finding.

### 3. `std::unordered_map<std::string, std::string>`
Dùng cho `parent map`.

### 4. `std::stack<DebugRecord>`
Dùng cho reverse debug history.

### 5. `std::vector<LandmarkRecord>`
Dùng lưu landmarks.

### 6. `std::vector<StereoObservationResult>`
Dùng lưu kết quả stereo.

---

## Algorithm bắt buộc

## Algorithm 1 — BFS path source → left_camera / right_camera
Giống Bài 33, nhưng phải chạy **2 lần** cho mỗi request:
- path tới `left_camera`
- path tới `right_camera`

---

## Algorithm 2 — Compose transform chain
Từ path đã tìm được, compose transform để lấy:

```text
T_source_left_camera
T_source_right_camera
```

---

## Algorithm 3 — Project landmark sang ảnh trái/phải
Nếu điểm camera frame là `(Xc, Yc, Zc)`:

```text
u = fx * Xc / Zc + cx
v = fy * Yc / Zc + cy
```

Nếu `Zc <= 0` → invalid.

---

## Algorithm 4 — Stereo disparity
Nếu left/right projection đều hợp lệ:

```text
disparity = u_left - u_right
```

---

## Algorithm 5 — Reverse debug history
Pop stack để ghi log ngược.

---

# 7. Cấu trúc folder

```text
mini_project_34_stereo_frame_graph_landmark_projector/
│
├─ README.md
│
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ transform_path_report.txt
│     ├─ left_right_projection_report.txt
│     ├─ stereo_disparity_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ frame_node_list.txt
│  ├─ transform_edge_list.txt
│  ├─ stereo_intrinsics_config.txt
│  ├─ landmark_list.txt
│  └─ stereo_observation_request_list.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ stereo_graph_preview.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ Pixel2D.hpp
   │  ├─ StereoIntrinsics.hpp
   │  ├─ TransformMatrix.hpp
   │  ├─ TransformEdge.hpp
   │  ├─ StereoObservationRequest.hpp
   │  ├─ StereoObservationResult.hpp
   │  ├─ StereoDebugRecord.hpp
   │  ├─ FrameGraph.hpp
   │  ├─ BasePathFinder.hpp
   │  ├─ BFSPathFinder.hpp
   │  ├─ TransformComposer.hpp
   │  ├─ StereoLandmarkProjector.hpp
   │  └─ StereoReportWriter.hpp
   │
   └─ src/
      ├─ FrameGraph.cpp
      ├─ BFSPathFinder.cpp
      ├─ TransformComposer.cpp
      ├─ StereoLandmarkProjector.cpp
      └─ StereoReportWriter.cpp
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
stereo_observation_request_list_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in các path config

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

### `@staticmethod validate_frame_name(name)`
- không rỗng

---

# 8.2 Python — `StereoFrameGraphConfigBuilder`

Tạo class con:

```python
class StereoFrameGraphConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
frame_nodes
transform_edges
stereo_intrinsics
landmarks
stereo_requests
```

## Hàm cần có

### `add_frame_node(frame_name)`
Ví dụ:
- `world`
- `base`
- `torso`
- `head`
- `left_camera`
- `right_camera`

### `add_transform_edge(parent_frame, child_frame, rotation_matrix, translation_vector)`

### `set_stereo_intrinsics(left_fx, left_fy, left_cx, left_cy, right_fx, right_fy, right_cx, right_cy, image_width, image_height)`

### `add_landmark(name, source_frame, x, y, z)`
Ví dụ:
```text
landmark_01 nằm trong world frame
```

### `add_stereo_request(request_name, landmark_name)`
Ví dụ:
```text
request_01 -> landmark_01
```

### `write_frame_node_list()`
### `write_transform_edge_list()`
### `write_stereo_intrinsics_config()`
### `write_landmark_list()`
### `write_stereo_request_list()`

---

# 8.3 Python — `stereo_graph_preview.py`

Tạo class:

```python
class PythonStereoGraphPreview:
```

## Hàm cần có

### `add_edge(parent, child)`
### `bfs_preview(source, target)`
### `preview_left_right_paths(source_frame)`
- trả:
  - path tới `left_camera`
  - path tới `right_camera`

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

# 8.9 C++ — `StereoObservationRequest`

```cpp
struct StereoObservationRequest
{
    std::string request_name;
    std::string landmark_name;
};
```

---

# 8.10 C++ — `StereoObservationResult`

```cpp
struct StereoObservationResult
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
};
```

---

# 8.11 C++ — `StereoDebugRecord`

```cpp
struct StereoDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 8.12 C++ — `FrameGraph`

Giống Bài 33 nhưng dùng lại cho stereo runtime.

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

# 8.16 C++ — `StereoLandmarkProjector`

Tạo class:

```cpp
class StereoLandmarkProjector
```

## Thuộc tính

```cpp
private:
    FrameGraph graph;
    StereoIntrinsics intrinsics;
    std::vector<Point3D> landmarks;
    std::vector<StereoObservationRequest> requests;
    std::vector<StereoObservationResult> results;

    std::shared_ptr<BasePathFinder> path_finder;
    std::stack<StereoDebugRecord> debug_history;
```

## Hàm cần có

### `load_frame_node_list(...)`
### `load_transform_edge_list(...)`
### `load_stereo_intrinsics_config(...)`
### `load_landmark_list(...)`
### `load_stereo_request_list(...)`

### `const Point3D* find_landmark(const std::string& landmark_name) const`

### `Pixel2D project_left(const Point3D& camera_point) const`
### `Pixel2D project_right(const Point3D& camera_point) const`

### `void run()`
Với mỗi request:

1. tìm landmark
2. BFS từ `landmark.source_frame` → `left_camera`
3. BFS từ `landmark.source_frame` → `right_camera`
4. compose transform path trái
5. compose transform path phải
6. transform landmark sang left/right camera frame
7. project ra left/right pixel
8. tính disparity nếu hợp lệ
9. lưu `StereoObservationResult`
10. push debug record

### Getter

```cpp
const std::vector<StereoObservationResult>& get_results() const;
std::vector<StereoDebugRecord> get_debug_history_reverse();
```

---

# 8.17 C++ — `StereoReportWriter`

Tạo class:

```cpp
class StereoReportWriter
```

## Hàm cần có

### `write_transform_path_report(...)`

Ví dụ:

```text
[Path]
Request: request_01
Landmark: landmark_01
Path to Left: world -> base -> torso -> head -> left_camera
Path to Right: world -> base -> torso -> head -> right_camera
```

### `write_left_right_projection_report(...)`

Ví dụ:

```text
[Stereo Projection]
Landmark: landmark_01
Left Pixel: (421.2, 201.8)
Right Pixel: (392.5, 201.1)
Left Valid: true
Right Valid: true
```

### `write_stereo_disparity_report(...)`

Ví dụ:

```text
[Disparity]
Landmark: landmark_01
Disparity Valid: true
Disparity: 28.7
```

### `write_reverse_debug_history(...)`

Ví dụ:

```text
[Reverse Debug History]
request_03: right path missing
request_02: projected successfully
request_01: disparity computed
```

---

# 8.18 C++ — `main.cpp`

## Yêu cầu

```text
Create StereoLandmarkProjector
→ Load frame nodes
→ Load transform edges
→ Load stereo intrinsics
→ Load landmark list
→ Load stereo request list
→ Run
→ Write transform path report
→ Write left/right projection report
→ Write disparity report
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
- Python graph preview
- C++ `queue`
- C++ `stack`
- C++ `vector`
- C++ `unordered_map`
- C++ `unordered_set`
- BFS path finding
- transform composition
- stereo left/right projection
- disparity computation
- stereo report

---

# 10. Output mong muốn

## Config

```text
config/frame_node_list.txt
config/transform_edge_list.txt
config/stereo_intrinsics_config.txt
config/landmark_list.txt
config/stereo_observation_request_list.txt
```

## Reports

```text
assets/outputs/transform_path_report.txt
assets/outputs/left_right_projection_report.txt
assets/outputs/stereo_disparity_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 11. Vai trò trong Humanoid Robot

## Python

```text
Stereo Frame Graph Config Builder + Preview Tool
```

## C++

```text
Stereo Landmark Projection Runtime
```

Đây là một bước rất gần robot perception thật, vì bạn đang nối 3 thứ lại với nhau:

```text
frame graph
+ camera geometry
+ stereo observation
```

## Computer Vision / Robot Geometry

```text
landmark in source frame
→ path to left/right camera
→ transform to both camera frames
→ project to left/right image
→ disparity
```

---

# 12. Checklist hoàn thành

- [ ] Python tạo đủ 5 config
- [ ] Python preview left/right path
- [ ] C++ có `StereoLandmarkProjector`
- [ ] C++ có `FrameGraph`
- [ ] C++ có `BFSPathFinder`
- [ ] C++ có `TransformComposer`
- [ ] C++ tìm được path tới left/right camera
- [ ] C++ project được left/right pixel
- [ ] C++ tính disparity
- [ ] C++ ghi đủ 4 report

---

# 13. Gợi ý mở rộng

## 1. Thêm baseline check
Nếu disparity <= 0 thì đánh dấu bất thường.

## 2. Thêm depth reconstruction
Từ disparity tính depth sơ bộ:

```text
Z = fx * baseline / disparity
```

## 3. Thêm nhiều landmark cùng frame khác nhau
Ví dụ:
- landmark trong `world`
- landmark trong `base`
- landmark trong `torso`

## 4. Chuẩn bị cho Bài 35

Bài 35 nên đi tiếp theo hướng:

```text
Stereo Depth Reconstruction Validator
```

Tức là từ:
- left pixel
- right pixel
- disparity

bạn sẽ:
- reconstruct depth
- so với depth ground truth từ landmark 3D
- tính depth error
- bắt đầu đánh mạnh hơn vào **thuật toán stereo depth**.
