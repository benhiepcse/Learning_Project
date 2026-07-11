# 🤖 Bài 43: Stereo Rectification & Epipolar Geometry Playground — Bộ mô phỏng chỉnh hàng stereo và hình học epipolar cho Humanoid Robot AI Perception

> Mini Project số 43 trong **Đợt 9**  
> **Bài 43 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9** và đi tiếp trực tiếp từ **Bài 42**.
>
> Nếu:
>
> - **Bài 41** tập trung vào **single-camera calibration + back-projection**
> - **Bài 42** mở rộng sang **multi-camera calibration graph**
>
> thì **Bài 43** sẽ đi vào phần lõi nhất của stereo geometry:
>
> ```text
> left camera + right camera
> → calibration
> → undistortion
> → stereo rectification
> → epipolar lines
> → correspondence-ready geometry
> ```
>
> Đây là bước cực kỳ quan trọng cho định hướng **Robot Perception Engineer** vì nó chạm trực tiếp vào:
> - stereo rig
> - epipolar constraint
> - rectification
> - disparity-ready image geometry
>
> và nó cũng là nền tốt cho:
> - depth estimation
> - stereo matching
> - 3D reconstruction
> - visual perception trong robot / embodied RL

---

# 📌 Mục lục

- [1. Vì sao Bài 43 xuất hiện sau Bài 42](#1-vì-sao-bài-43-xuất-hiện-sau-bài-42)
- [2. Đợt 9 đang học gì](#2-đợt-9-đang-học-gì)
- [3. Bài 43 nâng từ Bài 42 lên chỗ nào](#3-bài-43-nâng-từ-bài-42-lên-chỗ-nào)
- [4. Mô tả](#4-mô-tả)
- [5. Mục tiêu perception](#5-mục-tiêu-perception)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật stereo geometry của project](#11-luật-stereo-geometry-của-project)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 43 xuất hiện sau Bài 42

## Bài 42 đang làm gì?
Bài 42 đã có:
- nhiều camera
- calibration graph
- observation theo `camera_id`
- undistortion / back-projection / world transform

Nhưng Bài 42 vẫn là **multi-camera geometry tổng quát**.

## Bài 43 nâng lên ở đâu?
Bài 43 thu hẹp lại đúng bài toán stereo rig:

```text
left_camera
right_camera
```

và tập trung sâu vào 4 ý cực quan trọng:

## 1. Stereo calibration pair
Hai camera trái/phải có:
- intrinsics riêng
- distortion riêng
- extrinsic tương đối giữa 2 camera

## 2. Rectification
Tìm phép biến đổi để hai ảnh được “chỉnh hàng” sao cho:
- epipolar lines nằm ngang
- tìm correspondence dễ hơn

## 3. Epipolar geometry
Từ điểm ở ảnh trái, xác định epipolar line ở ảnh phải.

## 4. Correspondence-ready geometry
Sau rectification, matching sẽ trở thành:
- tìm điểm tương ứng trên **cùng một hàng ảnh** hoặc gần như vậy

Tức là Bài 43 là bước chuyển từ:

```text
multi-camera calibration
```

sang:

```text
stereo-specific geometry
```

---

# 2. Đợt 9 đang học gì

Theo roadmap của bạn, **Đợt 9** gồm:

## Python
### Phase 8 — Data Structures
- `Binary Search Tree`
- `Graph`
  - graph representation
  - BFS / DFS

### Phase 9 — Python Libraries
- `NumPy`
  - array
  - shape
  - indexing
  - math operations

## C++
### Phase 6 — OOP
- `Virtual Functions`
  - polymorphism
- `Enums`

## Computer Vision
### Phase 4 — Camera Geometry
- `Back Projection 2D → 3D`
- `Camera Calibration`
- `Undistortion`

## Vì sao Bài 43 vẫn bám Đợt 9?
Vì nó dùng trực tiếp:
- calibration
- undistortion
- projection geometry
- NumPy để dựng matrix / kiểm tra epipolar relation
- Graph / BST để tổ chức stereo sample / processing pipeline
- C++ polymorphism cho rectification / epipolar / solver runtime

---

# 3. Bài 43 nâng từ Bài 42 lên chỗ nào

## Bài 42
- nhiều camera tổng quát
- observation đa camera
- back-projection / plane / depth

## Bài 43
- thu về đúng **stereo pair**
- thêm:
  - `Fundamental / Essential thinking`
  - `rectification transform`
  - `epipolar line computation`
  - `left-right point consistency preview`

Tức là Bài 43 không còn chỉ là “mỗi camera tự reconstruct point”, mà bắt đầu quan tâm tới **mối quan hệ hình học giữa 2 camera cùng nhìn một scene**.

---

# 4. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Rectification & Epipolar Geometry Playground**

System này sẽ có:

## 1. Stereo pair
- `left_camera`
- `right_camera`

Mỗi camera có:
- intrinsic
- distortion

Ngoài ra có:
- baseline / relative rotation giữa hai camera
- optional stereo extrinsic matrix

## 2. Stereo point observations
Mỗi sample có thể gồm:
- `left pixel`
- `right pixel`
- hoặc chỉ `left pixel + expected epipolar relation`
- optional ground-truth disparity / depth

## 3. Các tác vụ chính
### a. Undistort left/right points
### b. Rectify left/right points
### c. Tính epipolar line
### d. Kiểm tra left-right row consistency sau rectification
### e. Tính disparity sơ bộ nếu có đủ cặp điểm
### f. Sinh report về stereo geometry quality

<p align="center">
  <img src="../../images/project_43.png" width="800">
</p>

---

# 5. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline:

```text
Python
→ build stereo calibration config
→ build stereo observation set
→ preview stereo graph / sample ordering
→ preview NumPy essential / fundamental style matrices

C++
→ load left/right camera model
→ undistort stereo observations
→ rectify stereo observations
→ compute epipolar line / row alignment
→ evaluate disparity-ready geometry
→ export stereo geometry reports
```

Mục tiêu cốt lõi:
- hiểu **vì sao rectification quan trọng**
- hiểu **epipolar line là gì**
- hiểu **stereo matching sẽ dễ hơn sau rectification ra sao**
- chuẩn bị nền cho:
  - block matching
  - SGM
  - disparity / depth estimation

---

# 6. Pipeline tổng thể

```text
Load Left Camera Intrinsics
Load Right Camera Intrinsics
Load Left/Right Distortion
Load Stereo Extrinsic Config
Load Stereo Observation Config

Create StereoCalibrationModel
Create StereoUndistortionEngine
Create StereoRectificationEngine
Create EpipolarGeometryEngine
Create StereoGeometryEvaluationRunner

For each stereo sample:
    1. undistort left point
    2. undistort right point

    3. rectify left point
    4. rectify right point

    5. compute epipolar relation / row difference

    6. if both left/right points exist:
           compute disparity = u_left_rect - u_right_rect

    7. evaluate stereo sample quality

Write:
    undistortion report
    rectification report
    epipolar report
    disparity-ready geometry report
    reverse debug history
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- file handling
- BST
- Graph + BFS / DFS
- NumPy
- config builder

## C++
- class / inheritance
- virtual functions / polymorphism
- enum class
- `std::vector`
- `std::unordered_map`
- `std::stack`
- stereo geometry runtime design

## Computer Vision / Geometry
- stereo calibration
- undistortion
- rectification
- epipolar geometry
- disparity intuition
- left-right consistency

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. Python Graph
Dùng để biểu diễn **stereo processing graph**.

Ví dụ:
```text
LeftRawPoint  -> LeftUndistort  -> LeftRectify
RightRawPoint -> RightUndistort -> RightRectify
LeftRectify + RightRectify -> EpipolarCheck -> DisparityPreview
```

## 2. Python BST
Dùng để lưu `sample_id` hoặc `frame_index` của stereo sample.

## 3. `std::vector<StereoPointObservation>`
Danh sách stereo samples.

## 4. `std::vector<StereoGeometryResult>`
Danh sách kết quả rectification / epipolar / disparity preview.

## 5. `std::stack<StereoGeometryDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Undistortion per camera
Điểm trái dùng distortion của left camera.  
Điểm phải dùng distortion của right camera.

---

## Algorithm 2 — Rectification transform
Bạn có thể làm theo **2 mức độ**:

### Mức 1 — học đúng tư duy, implementation đơn giản
- giả lập rectification bằng cách áp một transform 2D / homography đơn giản
- mục tiêu là hiểu pipeline

### Mức 2 — nếu muốn mạnh hơn
- dùng OpenCV:
  - `stereoRectify`
  - `undistortPoints`
  - hoặc `computeCorrespondEpilines`

Ở Bài 43, mình ưu tiên **pipeline + kiến trúc code + hiểu geometry**, nên bạn được phép đơn giản hóa rectification engine miễn logic đúng.

---

## Algorithm 3 — Epipolar line / row consistency
Sau rectification, kiểm tra:
- `|v_left_rect - v_right_rect|`

Nếu giá trị nhỏ → sample phù hợp với stereo geometry đã rectify.

---

## Algorithm 4 — Disparity preview
Nếu có đủ cặp điểm trái/phải:

```text
disparity = u_left_rect - u_right_rect
```

Không cần làm full stereo matching ở bài này; chỉ cần **preview disparity từ matched pair**.

---

## Algorithm 5 — Stereo sample quality evaluation
Với mỗi sample, tính ít nhất:
- `row_alignment_error = |vL_rect - vR_rect|`
- `disparity`
- `is_valid_rectified_pair`

---

# 9. Cấu trúc folder

```text
mini_project_43_stereo_rectification_epipolar_geometry_playground/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ stereo_undistortion_report.txt
│     ├─ stereo_rectification_report.txt
│     ├─ epipolar_geometry_report.txt
│     ├─ disparity_ready_geometry_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ left_camera_intrinsics.txt
│  ├─ right_camera_intrinsics.txt
│  ├─ left_camera_distortion.txt
│  ├─ right_camera_distortion.txt
│  ├─ stereo_extrinsic_config.txt
│  ├─ stereo_observation_config.txt
│  └─ stereo_runtime_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ stereo_graph_preview.py
│     ├─ stereo_sample_bst.py
│     ├─ numpy_stereo_preview.py
│     └─ sample_generator.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ CameraIntrinsics.hpp
   │  ├─ DistortionCoefficients.hpp
   │  ├─ StereoExtrinsic.hpp
   │  ├─ StereoPointObservation.hpp
   │  ├─ StereoGeometryResult.hpp
   │  ├─ StereoGeometryDebugRecord.hpp
   │  ├─ StereoCalibrationModel.hpp
   │  ├─ BaseStereoUndistortionEngine.hpp
   │  ├─ StereoUndistortionEngine.hpp
   │  ├─ BaseStereoRectificationEngine.hpp
   │  ├─ StereoRectificationEngine.hpp
   │  ├─ BaseEpipolarGeometryEngine.hpp
   │  ├─ EpipolarGeometryEngine.hpp
   │  ├─ StereoGeometryEvaluationRunner.hpp
   │  └─ StereoGeometryReportWriter.hpp
   │
   └─ src/
      ├─ StereoUndistortionEngine.cpp
      ├─ StereoRectificationEngine.cpp
      ├─ EpipolarGeometryEngine.cpp
      ├─ StereoGeometryEvaluationRunner.cpp
      └─ StereoGeometryReportWriter.cpp
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
left_intrinsics_path
right_intrinsics_path
left_distortion_path
right_distortion_path
stereo_extrinsic_path
observation_path
runtime_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoRectificationConfigBuilder`

Tạo class con:

```python
class StereoRectificationConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `set_left_intrinsics(fx, fy, cx, cy, image_width, image_height)`
### `set_right_intrinsics(fx, fy, cx, cy, image_width, image_height)`

### `set_left_distortion(k1, k2, p1, p2, k3=0.0)`
### `set_right_distortion(k1, k2, p1, p2, k3=0.0)`

### `set_stereo_extrinsic(
    baseline,
    roll_deg,
    pitch_deg,
    yaw_deg
)`

### `add_stereo_sample(
    sample_id,
    u_left_raw,
    v_left_raw,
    u_right_raw,
    v_right_raw,
    gt_disparity=None,
    gt_depth=None
)`

### `set_runtime_options(
    enable_epipolar_check,
    row_error_threshold,
    enable_disparity_preview
)`

### `write_left_intrinsics()`
### `write_right_intrinsics()`
### `write_left_distortion()`
### `write_right_distortion()`
### `write_stereo_extrinsic()`
### `write_observation_config()`
### `write_runtime_config()`

---

# 10.3 Python — `stereo_graph_preview.py`

Tạo class:

```python
class StereoProcessingGraph:
```

## Hàm cần có
### `add_edge(src, dst)`
### `bfs(start_node)`
### `dfs(start_node)`
### `show_graph()`

Graph ví dụ:

```text
LeftRaw  -> LeftUndistort  -> LeftRectify
RightRaw -> RightUndistort -> RightRectify
LeftRectify + RightRectify -> EpipolarCheck -> DisparityPreview
```

---

# 10.4 Python — `stereo_sample_bst.py`

Tạo BST cho `sample_id` hoặc `frame_id` của stereo sample.

## Hàm cần có
### `insert(key, payload)`
### `search(key)`
### `inorder()`
### `preorder()`

---

# 10.5 Python — `numpy_stereo_preview.py`

Tạo class:

```python
class NumPyStereoPreview:
```

## Hàm cần có

### `build_intrinsic_matrix(fx, fy, cx, cy)`
### `normalize_pixel(u, v, fx, fy, cx, cy)`
### `compute_disparity(u_left, u_right)`
### `estimate_depth_from_disparity(fx, baseline, disparity)`
### `show_row_alignment(v_left_rect, v_right_rect)`

---

# 10.6 C++ — `StereoExtrinsic`

```cpp
struct StereoExtrinsic
{
    double baseline;
    double roll_deg;
    double pitch_deg;
    double yaw_deg;
};
```

---

# 10.7 C++ — `StereoPointObservation`

```cpp
struct StereoPointObservation
{
    std::string sample_id;

    double u_left_raw;
    double v_left_raw;

    double u_right_raw;
    double v_right_raw;

    bool has_gt_disparity;
    double gt_disparity;

    bool has_gt_depth;
    double gt_depth;
};
```

---

# 10.8 C++ — `StereoGeometryResult`

```cpp
struct StereoGeometryResult
{
    std::string sample_id;

    double u_left_undist;
    double v_left_undist;
    double u_right_undist;
    double v_right_undist;

    double u_left_rect;
    double v_left_rect;
    double u_right_rect;
    double v_right_rect;

    double row_alignment_error;

    bool has_disparity;
    double disparity;

    bool valid_rectified_pair;
};
```

---

# 10.9 C++ — `StereoGeometryDebugRecord`

```cpp
struct StereoGeometryDebugRecord
{
    std::string event_name;
    std::string message;
};
```

---

# 10.10 C++ — `StereoCalibrationModel`

Tạo class:

```cpp
class StereoCalibrationModel
{
private:
    CameraIntrinsics left_intrinsics;
    CameraIntrinsics right_intrinsics;

    DistortionCoefficients left_distortion;
    DistortionCoefficients right_distortion;

    StereoExtrinsic stereo_extrinsic;

public:
    StereoCalibrationModel(
        const CameraIntrinsics& left_intrinsics,
        const CameraIntrinsics& right_intrinsics,
        const DistortionCoefficients& left_distortion,
        const DistortionCoefficients& right_distortion,
        const StereoExtrinsic& stereo_extrinsic
    );

    const CameraIntrinsics& get_left_intrinsics() const;
    const CameraIntrinsics& get_right_intrinsics() const;

    const DistortionCoefficients& get_left_distortion() const;
    const DistortionCoefficients& get_right_distortion() const;

    const StereoExtrinsic& get_stereo_extrinsic() const;
};
```

---

# 10.11 C++ — `BaseStereoUndistortionEngine`

Tạo abstract class:

```cpp
class BaseStereoUndistortionEngine
{
public:
    virtual std::pair<double, double> undistort_left(
        double u_raw,
        double v_raw,
        const StereoCalibrationModel& model
    ) const = 0;

    virtual std::pair<double, double> undistort_right(
        double u_raw,
        double v_raw,
        const StereoCalibrationModel& model
    ) const = 0;

    virtual ~BaseStereoUndistortionEngine() = default;
};
```

---

# 10.12 C++ — `StereoUndistortionEngine`

Kế thừa `BaseStereoUndistortionEngine`.

## Nhiệm vụ
- dùng left distortion cho điểm trái
- dùng right distortion cho điểm phải

---

# 10.13 C++ — `BaseStereoRectificationEngine`

Tạo abstract class:

```cpp
class BaseStereoRectificationEngine
{
public:
    virtual void rectify(
        const StereoPointObservation& sample,
        const StereoCalibrationModel& model,
        StereoGeometryResult& result
    ) const = 0;

    virtual ~BaseStereoRectificationEngine() = default;
};
```

---

# 10.14 C++ — `StereoRectificationEngine`

Kế thừa `BaseStereoRectificationEngine`.

## Nhiệm vụ
- nhận điểm trái/phải đã undistort
- tạo điểm trái/phải đã rectify

### Gợi ý triển khai
Ở mức đơn giản, bạn có thể:
- áp một transform để ép `v_left_rect` và `v_right_rect` gần nhau hơn
- hoặc dùng logic rectification gần đúng

Nếu muốn mạnh hơn, có thể dùng OpenCV stereo rectification API.

---

# 10.15 C++ — `BaseEpipolarGeometryEngine`

Tạo abstract class:

```cpp
class BaseEpipolarGeometryEngine
{
public:
    virtual void evaluate_epipolar(
        const StereoPointObservation& sample,
        const StereoCalibrationModel& model,
        StereoGeometryResult& result
    ) const = 0;

    virtual ~BaseEpipolarGeometryEngine() = default;
};
```

---

# 10.16 C++ — `EpipolarGeometryEngine`

Kế thừa `BaseEpipolarGeometryEngine`.

## Nhiệm vụ
- tính `row_alignment_error`
- nếu hợp lệ thì tính disparity
- set `valid_rectified_pair`

---

# 10.17 C++ — `StereoGeometryEvaluationRunner`

Tạo class trung tâm:

```cpp
class StereoGeometryEvaluationRunner
```

## Thuộc tính

```cpp
private:
    StereoCalibrationModel stereo_model;

    std::vector<StereoPointObservation> samples;

    std::shared_ptr<BaseStereoUndistortionEngine> undistortion_engine;
    std::shared_ptr<BaseStereoRectificationEngine> rectification_engine;
    std::shared_ptr<BaseEpipolarGeometryEngine> epipolar_engine;

    std::vector<StereoGeometryResult> results;
    std::stack<StereoGeometryDebugRecord> debug_history;
```

## Hàm cần có

### `load_stereo_samples(const std::string& path)`

### `run()`
Pseudo:

```text
for each sample:
    1. undistort left/right
    2. rectify left/right
    3. evaluate row alignment
    4. if valid -> compute disparity
    5. save result
    6. push debug history
```

### `const std::vector<StereoGeometryResult>& get_results() const`
### `std::vector<StereoGeometryDebugRecord> get_debug_history_reverse()`

---

# 10.18 C++ — `StereoGeometryReportWriter`

Tạo class:

```cpp
class StereoGeometryReportWriter
```

## Hàm cần có

### `write_stereo_undistortion_report(...)`
Ví dụ:

```text
[Undistortion]
Sample: pair_01
Left Raw  -> Left Undist = (u, v)
Right Raw -> Right Undist = (u, v)
```

### `write_stereo_rectification_report(...)`
Ví dụ:

```text
[Rectification]
Sample: pair_01
Left Rect : (uL, vL)
Right Rect: (uR, vR)
Row Error : 0.42
```

### `write_epipolar_geometry_report(...)`
Ví dụ:

```text
[Epipolar Geometry]
Sample: pair_01
Valid Rectified Pair: true
Row Alignment Error: 0.42
```

### `write_disparity_ready_geometry_report(...)`
Ví dụ:

```text
[Disparity Preview]
Sample: pair_01
Disparity: 18.4
```

### `write_reverse_debug_history(...)`

---

# 10.19 C++ — `main.cpp`

## Yêu cầu

```text
Load left/right intrinsics
Load left/right distortion
Load stereo extrinsic
Load stereo samples

Create StereoCalibrationModel

Create:
    StereoUndistortionEngine
    StereoRectificationEngine
    EpipolarGeometryEngine

Create StereoGeometryEvaluationRunner
Run
Write reports
```

---

# 11. Luật stereo geometry của project

## Luật 1 — Điểm trái và điểm phải phải được undistort theo đúng camera tương ứng
Không được dùng nhầm distortion.

## Luật 2 — Rectification phải chạy sau undistortion
Không rectify trực tiếp từ pixel méo nếu pipeline của bạn là calibration-aware.

## Luật 3 — `row_alignment_error` là chỉ số bắt buộc
Vì đây là chỉ số trực tiếp phản ánh rectification có đang làm cho hai điểm nằm cùng hàng ảnh hay không.

## Luật 4 — Chỉ tính disparity khi pair được coi là hợp lệ
Ví dụ:
```text
row_alignment_error <= threshold
```

## Luật 5 — Nếu có ground-truth disparity
Có thể tính thêm disparity error, nhưng đây là phần mở rộng chứ không bắt buộc ở Bài 43.

---

# 12. Output mong muốn

## Config
```text
config/left_camera_intrinsics.txt
config/right_camera_intrinsics.txt
config/left_camera_distortion.txt
config/right_camera_distortion.txt
config/stereo_extrinsic_config.txt
config/stereo_observation_config.txt
config/stereo_runtime_config.txt
```

## Reports
```text
assets/outputs/stereo_undistortion_report.txt
assets/outputs/stereo_rectification_report.txt
assets/outputs/epipolar_geometry_report.txt
assets/outputs/disparity_ready_geometry_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build stereo configs
- preview stereo pipeline graph
- dùng NumPy để kiểm tra disparity / depth / alignment logic

## C++
- chạy stereo geometry runtime
- undistort / rectify / evaluate stereo pair
- tạo nền cho stereo matching / depth estimation

## Computer Vision / Robot Perception
Đây là một project rất lõi cho **Robot Perception Engineer** vì stereo perception gần như luôn đi qua các khái niệm:
- calibration
- rectification
- epipolar constraint
- disparity

Nếu không nắm Bài 43, rất khó học chắc:
- block matching
- SGM
- stereo depth reconstruction
- depth error analysis

---

# 14. Checklist hoàn thành

- [ ] Python build đủ stereo config
- [ ] Python có graph preview bằng BFS / DFS
- [ ] Python có BST cho stereo sample ids
- [ ] Python có NumPy preview cho disparity / depth / row alignment
- [ ] C++ có `StereoCalibrationModel`
- [ ] C++ có `StereoUndistortionEngine`
- [ ] C++ có `StereoRectificationEngine`
- [ ] C++ có `EpipolarGeometryEngine`
- [ ] C++ xử lý được toàn bộ stereo samples
- [ ] C++ tính được row alignment error
- [ ] C++ preview được disparity
- [ ] C++ ghi đủ 5 report

---

# 15. Gợi ý mở rộng

## 1. Thêm essential / fundamental matrix thật
Cho phép kiểm tra `x_r^T F x_l = 0` ở mức cơ bản.

## 2. Thêm reprojection consistency
Từ disparity / depth, dựng lại 3D rồi project lại sang hai ảnh.

## 3. Thêm visualization ảnh
Vẽ epipolar lines hoặc vẽ row alignment trên ảnh giả lập.

## 4. Chuẩn bị cho Bài 44
Sau Bài 43, hướng đi đẹp nhất cho Đợt 9 là:

# **Bài 44: Stereo Correspondence Cost Explorer**

Ý tưởng:
```text
rectified stereo pair
→ candidate disparity search
→ SAD / SSD / NCC style cost
→ cost table
→ disparity selection preview
```

Lúc đó chuỗi Đợt 9 sẽ rất liền mạch:
- **Bài 41**: calibration + back-projection
- **Bài 42**: multi-camera calibration graph
- **Bài 43**: stereo rectification + epipolar geometry
- **Bài 44**: correspondence cost / disparity search
