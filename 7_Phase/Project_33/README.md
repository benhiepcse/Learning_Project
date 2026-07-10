# 🤖 Bài 33: Transformation Graph Path Finder — Tìm đường đi biến đổi giữa các frame cho Humanoid Robot AI Perception

> Mini Project số 33 trong **Đợt 7**  
> **Bài 33 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7**.  
> Từ **Đợt 7 trở đi**, project phải bắt đầu mang dáng dấp của **runtime robot perception thật**:
>
> - có **frame graph**
> - có **path finding**
> - có **queue / stack / unordered_map / vector**
> - có **BFS / parent reconstruction**
> - có **smart pointers / enum class / Advanced C++**
> - có **camera geometry + robot coordinate frames**

---

# 📌 Mục lục

- [1. Bài 33 lấy gì từ Bài 31–32](#1-bài-33-lấy-gì-từ-bài-3132)
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

# 1. Bài 33 lấy gì từ Bài 31–32

## Bài 31
- queue task scheduler
- stack reverse history
- pinhole camera model
- intrinsic matrix

## Bài 32
- extrinsic transform command queue
- source frame → camera frame
- command runtime với smart pointer + STL

## Bài 33
Bài 33 nâng tiếp một bước rất quan trọng trong robot perception:

```text
không còn chỉ có 1 extrinsic transform cố định nữa
mà robot có cả một graph các frame
và phải tìm đường đi từ frame A sang frame B
```

Ví dụ:

```text
world → base → torso → head → left_camera
```

hoặc:

```text
world → base → right_arm → wrist_camera
```

Robot perception thật thường không làm việc với “1 ma trận duy nhất”, mà với **một hệ frame liên kết với nhau**. Vì vậy Bài 33 sẽ bắt đầu đưa bạn vào tư duy:

- **frame graph**
- **transform path**
- **BFS**
- **path reconstruction**
- **compose transform dọc theo path**

---

# 2. Mô tả

Giả sử robot có các frame sau:

- `world`
- `base`
- `torso`
- `head`
- `left_camera`
- `right_camera`
- `left_hand`
- `right_hand`

Giữa các frame này có các cạnh transform, ví dụ:

- `world -> base`
- `base -> torso`
- `torso -> head`
- `head -> left_camera`
- `head -> right_camera`
- `torso -> left_hand`
- `torso -> right_hand`

Nếu perception module muốn biến một điểm từ `world` sang `left_camera`, nó phải:

1. tìm đường đi trong graph:
```text
world → base → torso → head → left_camera
```

2. compose các transform trên đường đi đó

3. transform point sang frame đích

4. nếu frame đích là camera frame thì có thể project ra pixel

**Bài 33** sẽ yêu cầu bạn xây một **Transformation Graph Path Finder** có khả năng:

- load danh sách frame nodes
- load transform edges
- build adjacency graph
- dùng **BFS** để tìm path giữa source frame và target frame
- reconstruct path bằng `parent map`
- compose transform dọc path
- transform point
- ghi report

<p align="center">
  <img src="images/project_33.png" width="800">
</p>

---

# 3. Mục tiêu perception

Sau bài này, bạn phải hiểu pipeline:

```text
Python config builder
→ tạo frame node list
→ tạo transform edge list
→ tạo query transform requests

C++ runtime
→ build frame graph
→ BFS tìm path giữa source frame và target frame
→ reconstruct path
→ compose transform trên path
→ transform 3D point từ source sang target
→ nếu target là camera frame thì có thể project ra pixel
→ ghi report
```

---

# 4. Pipeline

```text
Load Frame Nodes
Load Transform Edges
Load Transform Requests
Load Source Points

Build Graph:
    frame_name -> neighbor frames

For each transform request:
    source_frame, target_frame, point_name

    Step 1:
        BFS to find path from source_frame to target_frame

    Step 2:
        reconstruct path using parent map

    Step 3:
        compose transform chain along path

    Step 4:
        transform point to target frame

    Step 5:
        if target frame is a camera frame:
            optionally project to pixel

Write:
    path report
    transform result report
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
- queue preview
- stack debug preview
- `if __name__ == "__main__"`

## C++
- class / inheritance
- `enum class`
- `std::queue`
- `std::stack`
- `std::vector`
- `std::unordered_map`
- `std::unordered_set`
- `std::unique_ptr`
- `std::shared_ptr`
- `std::optional` *(nếu muốn nâng cao)*
- `const reference`
- `virtual` / `override`
- BFS / parent reconstruction

## Geometry / CV
- extrinsic transform
- transform chain
- frame composition
- camera frame projection *(optional phần mở rộng)*

---

# 6. DSA + Algorithm bắt buộc

## DSA bắt buộc

### 1. `std::unordered_map<std::string, std::vector<std::string>>`
Dùng làm adjacency list:

```cpp
frame_graph["torso"] = {"head", "left_hand", "right_hand"}
```

---

### 2. `std::queue<std::string>`
Dùng cho BFS.

---

### 3. `std::unordered_map<std::string, std::string>`
Dùng làm `parent map` để reconstruct path.

---

### 4. `std::stack<std::string>`
Có thể dùng để đảo ngược path hoặc ghi debug history.

---

### 5. `std::vector<TransformEdge>`
Dùng để lưu toàn bộ transform edges.

---

## Algorithm bắt buộc

## Algorithm 1 — Build frame graph
Từ danh sách edges:

```text
(parent_frame, child_frame, transform)
```

xây adjacency list.

---

## Algorithm 2 — BFS path finding
```text
push source_frame vào queue
mark visited

while queue not empty:
    pop current
    nếu current == target → stop
    duyệt neighbors của current
    nếu neighbor chưa visited:
        mark visited
        parent[neighbor] = current
        push neighbor
```

---

## Algorithm 3 — Path reconstruction
Nếu BFS tìm được target:

```text
path = []
cur = target
while cur != source:
    path.push_back(cur)
    cur = parent[cur]
path.push_back(source)
reverse(path)
```

---

## Algorithm 4 — Compose transform chain
Giả sử path là:

```text
world → base → torso → head → left_camera
```

thì:

```text
T_world_left_camera =
T_head_left_camera
* T_torso_head
* T_base_torso
* T_world_base
```

hoặc theo quy ước bạn chọn, miễn là nhất quán.

---

## Algorithm 5 — Transform point
Sau khi có transform tổng hợp:

```text
P_target = T_source_to_target * P_source
```

---

# 7. Cấu trúc folder

```text
mini_project_33_transformation_graph_path_finder/
│
├─ README.md
│
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ frame_graph_report.txt
│     ├─ transform_path_report.txt
│     ├─ transform_result_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ frame_node_list.txt
│  ├─ transform_edge_list.txt
│  ├─ transform_request_list.txt
│  └─ point3d_list.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     └─ graph_preview.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ Point3D.hpp
   │  ├─ TransformMatrix.hpp
   │  ├─ TransformEdge.hpp
   │  ├─ TransformRequest.hpp
   │  ├─ TransformResult.hpp
   │  ├─ GraphDebugRecord.hpp
   │  ├─ FrameGraph.hpp
   │  ├─ BasePathFinder.hpp
   │  ├─ BFSPathFinder.hpp
   │  ├─ TransformComposer.hpp
   │  ├─ TransformRuntime.hpp
   │  └─ TransformReportWriter.hpp
   │
   └─ src/
      ├─ FrameGraph.cpp
      ├─ BFSPathFinder.cpp
      ├─ TransformComposer.cpp
      ├─ TransformRuntime.cpp
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
frame_node_list_path
transform_edge_list_path
transform_request_list_path
point3d_list_path
```

## Hàm cần có

### `show_project_info()`
- in tên project
- in path config

### `@classmethod create_default_paths(cls, project_root)`
- trả về dict path mặc định

### `@staticmethod validate_frame_name(name)`
- không cho rỗng

---

# 8.2 Python — `TransformationGraphConfigBuilder`

Tạo class con:

```python
class TransformationGraphConfigBuilder(BaseConfigBuilder):
```

## Thuộc tính

```python
frame_nodes
transform_edges
transform_requests
points
```

## Hàm cần có

### `add_frame_node(frame_name)`
Ví dụ:
- `world`
- `base`
- `torso`
- `head`
- `left_camera`

### `add_transform_edge(parent_frame, child_frame, rotation_matrix, translation_vector)`
- `rotation_matrix` là 3x3 list
- `translation_vector` là `[tx, ty, tz]`

### `add_transform_request(request_name, source_frame, target_frame, point_name)`
Ví dụ:
```text
request_01: world -> left_camera for landmark_01
```

### `add_point3d(name, x, y, z)`

### `write_frame_node_list()`
### `write_transform_edge_list()`
### `write_transform_request_list()`
### `write_point3d_list()`

---

# 8.3 Python — `graph_preview.py`

Tạo class:

```python
class PythonFrameGraphPreview:
```

## Bắt buộc dùng
- `dict`
- `list`
- `deque` hoặc list cho BFS preview

## Hàm cần có

### `add_edge(parent, child)`
- build graph preview

### `preview_neighbors(frame_name)`
- trả neighbors

### `bfs_preview(source, target)`
- preview path theo BFS ở mức Python

---

# 8.4 C++ — `Point3D`

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

# 8.5 C++ — `TransformMatrix`

Tạo struct 4x4 hoặc rotation + translation riêng.  
Đề cử đơn giản cho project này:

```cpp
struct TransformMatrix
{
    double r11, r12, r13;
    double r21, r22, r23;
    double r31, r32, r33;
    double tx, ty, tz;
};
```

## Hàm gợi ý
```cpp
static TransformMatrix identity();
Point3D apply(const Point3D& p) const;
TransformMatrix compose(const TransformMatrix& next) const;
```

---

# 8.6 C++ — `TransformEdge`

```cpp
struct TransformEdge
{
    std::string parent_frame;
    std::string child_frame;
    TransformMatrix transform;
};
```

---

# 8.7 C++ — `TransformRequest`

```cpp
struct TransformRequest
{
    std::string request_name;
    std::string source_frame;
    std::string target_frame;
    std::string point_name;
};
```

---

# 8.8 C++ — `TransformResult`

```cpp
struct TransformResult
{
    std::string request_name;
    std::string source_frame;
    std::string target_frame;
    std::vector<std::string> path_frames;
    bool path_found;
    Point3D transformed_point;
};
```

---

# 8.9 C++ — `GraphDebugRecord`

```cpp
struct GraphDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 8.10 C++ — `FrameGraph`

Tạo class:

```cpp
class FrameGraph
```

## Thuộc tính

```cpp
private:
    std::unordered_map<std::string, std::vector<std::string>> adjacency;
    std::vector<TransformEdge> edges;
```

## Hàm cần có

### `void add_edge(const TransformEdge& edge)`
- thêm edge vào graph
- cập nhật adjacency
- **nên** cho graph 2 chiều để BFS tìm đường dễ hơn:
  - `parent -> child`
  - `child -> parent`

### `const std::vector<std::string>& neighbors(const std::string& frame) const`
### `const std::vector<TransformEdge>& get_edges() const`
### `bool has_frame(const std::string& frame) const`

### `TransformMatrix get_transform_between(const std::string& from, const std::string& to) const`
- nếu có edge trực tiếp `from -> to`, trả transform đó
- nếu có edge `to -> from`, bạn có thể:
  - hoặc cài thêm inverse transform
  - hoặc đơn giản hóa bằng cách config sẵn cả 2 chiều
- với bài này, **đề cử**: khi add edge thì thêm luôn cả chiều ngược trong config hoặc runtime để đỡ nặng phần inverse.

---

# 8.11 C++ — `BasePathFinder`

Tạo abstract class:

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

# 8.12 C++ — `BFSPathFinder`

Kế thừa:

```cpp
class BFSPathFinder : public BasePathFinder
```

## Hàm override

```cpp
std::vector<std::string> find_path(
    const FrameGraph& graph,
    const std::string& source,
    const std::string& target
) override;
```

## Yêu cầu
- dùng `std::queue<std::string>`
- dùng `std::unordered_set<std::string>` cho visited
- dùng `std::unordered_map<std::string, std::string>` cho parent
- reconstruct path nếu tìm thấy

---

# 8.13 C++ — `TransformComposer`

Tạo class:

```cpp
class TransformComposer
```

## Hàm cần có

### `TransformMatrix compose_path_transform(
    const FrameGraph& graph,
    const std::vector<std::string>& path
) const;`

**Ý tưởng**
Nếu path là:

```text
world → base → torso → head → left_camera
```

thì duyệt từng cặp:
- `(world, base)`
- `(base, torso)`
- `(torso, head)`
- `(head, left_camera)`

và compose transform dần.

---

# 8.14 C++ — `TransformRuntime`

Tạo class:

```cpp
class TransformRuntime
```

## Thuộc tính

```cpp
private:
    FrameGraph graph;
    std::vector<Point3D> points;
    std::vector<TransformRequest> requests;
    std::vector<TransformResult> results;

    std::shared_ptr<BasePathFinder> path_finder;
    std::stack<GraphDebugRecord> debug_history;
```

## Hàm cần có

### `load_frame_node_list(const std::string& path)`
### `load_transform_edge_list(const std::string& path)`
### `load_transform_request_list(const std::string& path)`
### `load_point3d_list(const std::string& path)`

### `const Point3D* find_point_by_name(const std::string& point_name) const`

### `void run()`
Với mỗi request:
1. tìm point
2. BFS path
3. compose transform
4. transform point
5. lưu `TransformResult`
6. push debug record vào stack

### Getter

```cpp
const std::vector<TransformResult>& get_results() const;
std::vector<GraphDebugRecord> get_debug_history_reverse();
const FrameGraph& get_graph() const;
```

---

# 8.15 C++ — `TransformReportWriter`

Tạo class:

```cpp
class TransformReportWriter
```

## Hàm cần có

### `write_frame_graph_report(...)`

Ví dụ:

```text
[Frame Graph]
world -> base
base -> torso
torso -> head
head -> left_camera
head -> right_camera
```

### `write_transform_path_report(...)`

Ví dụ:

```text
[Transform Path]
Request: request_01
Path Found: true
Path: world -> base -> torso -> head -> left_camera
```

### `write_transform_result_report(...)`

Ví dụ:

```text
[Transform Result]
Request: request_01
Source Frame: world
Target Frame: left_camera
Point: landmark_01
Result: X=0.34, Y=-0.12, Z=1.82
```

### `write_reverse_debug_history(...)`

Ví dụ:

```text
[Reverse Debug History]
request_03: path found
request_02: point missing
request_01: transformed successfully
```

---

# 8.16 C++ — `main.cpp`

## Yêu cầu

```text
Create TransformRuntime
→ Load frame nodes
→ Load transform edges
→ Load transform requests
→ Load point3d list
→ Run runtime
→ Write graph report
→ Write path report
→ Write transform result report
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
- C++ `std::queue`
- C++ `std::stack`
- C++ `std::vector`
- C++ `std::unordered_map`
- C++ `std::unordered_set`
- C++ `std::shared_ptr`
- BFS path finding
- parent map reconstruction
- transform composition
- transform result report

---

# 10. Output mong muốn

## Config

```text
config/frame_node_list.txt
config/transform_edge_list.txt
config/transform_request_list.txt
config/point3d_list.txt
```

## Reports

```text
assets/outputs/frame_graph_report.txt
assets/outputs/transform_path_report.txt
assets/outputs/transform_result_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 11. Vai trò trong Humanoid Robot

## Python

```text
Frame Graph Config Builder + Graph Preview Tool
```

## C++

```text
Transformation Graph Runtime + BFS Path Finder
```

Đây là chỗ C++ bắt đầu giống robot runtime hơn hẳn, vì bạn đang xử lý:

- frame graph
- path finding
- transform composition
- request processing

## Computer Vision / Robot Geometry

```text
source point in frame A
→ find path A → B
→ compose transforms
→ transform point into frame B
```

Đây là tư duy cực quan trọng cho:

- camera frame
- robot base frame
- end-effector frame
- TF tree / TF2 về sau trong ROS2

---

# 12. Checklist hoàn thành

- [ ] Python tạo đủ 4 config
- [ ] Python preview BFS graph
- [ ] C++ có `FrameGraph`
- [ ] C++ có `BasePathFinder`
- [ ] C++ có `BFSPathFinder`
- [ ] C++ có `TransformComposer`
- [ ] C++ có `TransformRuntime`
- [ ] C++ dùng `unordered_map`
- [ ] C++ dùng `unordered_set`
- [ ] C++ dùng `queue` cho BFS
- [ ] C++ dùng `stack` cho debug history
- [ ] C++ reconstruct path đúng
- [ ] C++ compose transform dọc path
- [ ] C++ ghi đủ 4 report

---

# 13. Gợi ý mở rộng

## 1. Hỗ trợ inverse transform thật sự
Thay vì config cả 2 chiều, hãy tự tính inverse của rigid transform.

## 2. Thêm weighted graph
Mỗi edge có cost / confidence.

## 3. Thêm camera projection sau khi tới camera frame
Nếu target là `left_camera`, project luôn sang pixel.

## 4. Chuẩn bị cho Bài 34

Bài 34 nên đi tiếp theo hướng:

```text
Stereo Frame Graph Landmark Projector
```

Tức là sau khi tìm được path tới `left_camera` / `right_camera`, bạn sẽ:
- transform point sang cả 2 camera
- project sang ảnh trái / phải
- tính disparity cơ bản
- bắt đầu nối lại với stereo perception.
