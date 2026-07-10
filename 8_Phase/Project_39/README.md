# 🤖 Bài 39: Stereo Multi-Landmark Track Table Analyzer — Bộ phân tích bảng track nhiều landmark cho Humanoid Robot AI Perception

> Mini Project số 39 trong **Đợt 8**  
> **Bài 39 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8** và đi tiếp trực tiếp từ **Bài 38**.
>
> Nếu:
>
> - **Bài 36** đã có **Stereo Disparity Track Manager**
> - **Bài 37** đã có **Stereo Depth Command Graph Runner**
> - **Bài 38** đã có **Stereo Landmark Motion Queue Simulator**
>
> thì **Bài 39** sẽ đẩy tiếp sang bài toán:
>
> ```text
> nhiều landmark chạy đồng thời
> → nhiều track được cập nhật song song
> → gom toàn bộ track vào track table
> → phân tích chất lượng từng track
> → xếp hạng track theo độ ổn định / outlier / depth variance
> ```
>
> Đây là bước rất quan trọng vì từ chỗ chỉ theo dõi **một landmark hoặc xử lý từng observation**, bạn bắt đầu nhìn perception như **một hệ nhiều đối tượng / nhiều track cùng tồn tại**.

---

# 📌 Mục lục

- [1. Vì sao Bài 39 xuất hiện sau Bài 38](#1-vì-sao-bài-39-xuất-hiện-sau-bài-38)
- [2. Đợt 8 đang học gì](#2-đợt-8-đang-học-gì)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Ý tưởng Multi-Landmark Track Table](#5-ý-tưởng-multi-landmark-track-table)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật phân tích track table](#11-luật-phân-tích-track-table)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 39 xuất hiện sau Bài 38

## Bài 38 đang làm gì?
Bài 38 đã có:

```text
moving landmark 3D
→ stereo observation queue
→ command graph
→ disparity / depth / validation
→ temporal track update
```

Nhưng trọng tâm của Bài 38 vẫn là:
- **mô phỏng nguồn observation**
- **chạy pipeline**
- **cập nhật track**

Tức là nó vẫn thiên về **runtime processing**.

## Bài 39 nâng lên ở đâu?
Bài 39 giữ nguyên runtime đó, nhưng thêm một tầng **phân tích toàn cục trên nhiều track**:

```text
track_1: cup_01
track_2: ball_01
track_3: box_01
track_4: bottle_01
...
```

Sau khi toàn bộ track đã được cập nhật, bạn sẽ làm tiếp:

- track nào ổn định nhất?
- track nào có nhiều outlier nhất?
- track nào có depth dao động mạnh nhất?
- track nào đáng tin để dùng cho downstream perception?

Tức là từ “có track” bạn nâng lên thành **đánh giá chất lượng của cả bảng track**.

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

## Vì sao Bài 39 vẫn bám Đợt 8?
Vì bài này sẽ ép bạn dùng mạnh:
- **unordered_map** làm **track table**
- **vector / deque** để lưu history, thống kê, ranking
- **abstract analyzer** + **inheritance**
- **queue** cho observation stream
- **stack** cho reverse debug history
- **projection / disparity / depth** từ các bài trước để tạo dữ liệu phân tích

---

# 3. Mô tả

Bạn sẽ xây một mini runtime tên là:

# **Stereo Multi-Landmark Track Table Analyzer**

Runtime này nhận observation từ nhiều landmark như:

- `cup_01`
- `ball_01`
- `box_01`
- `bottle_01`

Mỗi landmark sẽ sinh ra một **StereoTrack** riêng.  
Sau khi tất cả observation đã được xử lý xong, bạn sẽ có một **track table**:

```text
unordered_map<string, StereoTrack>
```

Sau đó, bạn tạo thêm **bộ phân tích track table** để:

## Với mỗi track:
1. lấy số lượng observation hợp lệ
2. lấy số outlier
3. lấy smoothed disparity hiện tại
4. lấy smoothed depth hiện tại
5. tính độ dao động disparity / depth trong history
6. chấm điểm chất lượng track

## Với toàn bộ bảng track:
1. sắp xếp track theo stability score
2. tìm track nhiều outlier nhất
3. tìm track depth dao động mạnh nhất
4. xuất bảng xếp hạng track

<p align="center">
  <img src="images/project_39.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu được pipeline:

```text
Python
→ tạo motion config cho nhiều landmark
→ tạo tracking config
→ tạo command graph config
→ tạo analyzer config

C++
→ mô phỏng nhiều landmark
→ sinh observation queue
→ chạy command graph
→ cập nhật nhiều StereoTrack
→ build track table
→ phân tích toàn bộ track table
→ xếp hạng / thống kê / ghi report
```

Mục tiêu không chỉ là chạy pipeline, mà còn là:
- hiểu **nhiều track tồn tại đồng thời**
- hiểu **track table analytics**
- hiểu cách chọn track “tốt” cho perception downstream

---

# 5. Ý tưởng Multi-Landmark Track Table

Giả sử sau khi chạy xong pipeline, bạn có:

```text
cup_01:
    valid = 12
    outlier = 1
    smoothed_depth = 1.82
    depth_variance = 0.03

ball_01:
    valid = 10
    outlier = 4
    smoothed_depth = 2.15
    depth_variance = 0.21

box_01:
    valid = 15
    outlier = 0
    smoothed_depth = 1.46
    depth_variance = 0.01
```

Bộ analyzer phải kết luận được kiểu như:

- **box_01** là track ổn định nhất
- **ball_01** là track nhiều outlier nhất
- **ball_01** cũng là track depth biến động mạnh nhất
- **cup_01** là track khá tốt nhưng kém `box_01`

---

# 6. Pipeline tổng thể

```text
Load Stereo Intrinsics
Load Tracking Config
Load Motion Config
Load Command Graph Config
Load Analyzer Config

Create Motion Simulator
Create Command Graph Runner
Create Track Table Analyzer

Simulate landmarks across frames
Build observation queue
Run command graph
Get track table from track manager

For each track in track table:
    compute track statistics
    compute stability score
    classify track quality

Rank tracks
Write reports
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
- ranking preview / statistics preview

## C++
- class / inheritance
- abstract class / pure virtual
- copy constructor
- `std::queue`
- `std::stack`
- `std::deque`
- `std::vector`
- `std::unordered_map`
- `std::sort`
- `std::shared_ptr`
- statistics / ranking analyzer design

## Computer Vision / Geometry
- stereo observation
- disparity / depth history
- temporal smoothing
- quality analysis cho track

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. `std::unordered_map<std::string, StereoTrack>`
Track table trung tâm.

## 2. `std::vector<TrackAnalysisResult>`
Lưu kết quả phân tích từng track.

## 3. `std::deque<StereoObservationRecord>`
History của từng track.

## 4. `std::queue<StereoObservationRecord>`
Observation stream.

## 5. `std::stack<AnalyzerDebugRecord>`
Reverse debug history.

## 6. `std::vector<std::shared_ptr<BaseTrackAnalyzer>>`
Cho phép chạy nhiều analyzer trên cùng track table.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Observation pipeline
Dùng lại Bài 38:
- simulate landmark motion
- build observation queue
- run command graph
- update track table

---

## Algorithm 2 — Track statistics
Với mỗi track, tính tối thiểu:

### `valid_observation_count`
Số observation hợp lệ.

### `outlier_count`
Số observation bị reject.

### `smoothed_disparity`
Giá trị disparity đã smooth.

### `smoothed_depth`
Giá trị depth đã smooth.

### `disparity_variance`
Độ dao động disparity trong history.

### `depth_variance`
Độ dao động depth trong history.

---

## Algorithm 3 — Stability score
Tạo một công thức chấm điểm track.  
Ví dụ:

```text
stability_score
= valid_observation_count
- 2 * outlier_count
- alpha * depth_variance
```

Bạn được quyền thiết kế công thức riêng, miễn là hợp lý và giải thích được.

---

## Algorithm 4 — Track ranking
Sắp xếp `TrackAnalysisResult` theo:
- stability score giảm dần
- nếu bằng nhau thì ít outlier hơn đứng trước

---

## Algorithm 5 — Worst-track finder
Tìm:
- track nhiều outlier nhất
- track depth variance lớn nhất

---

# 9. Cấu trúc folder

```text
mini_project_39_stereo_multi_landmark_track_table_analyzer/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ landmark_motion_report.txt
│     ├─ generated_observation_queue_report.txt
│     ├─ stereo_track_summary_report.txt
│     ├─ track_table_analysis_report.txt
│     ├─ track_ranking_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_intrinsics_config.txt
│  ├─ stereo_tracking_config.txt
│  ├─ stereo_command_graph_config.txt
│  ├─ landmark_motion_config.txt
│  └─ track_analyzer_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ trajectory_preview.py
│     ├─ ranking_preview.py
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
   │  ├─ StereoCommandGraphRunner.hpp
   │  ├─ TrackAnalysisResult.hpp
   │  ├─ AnalyzerDebugRecord.hpp
   │  ├─ BaseTrackAnalyzer.hpp
   │  ├─ StabilityScoreAnalyzer.hpp
   │  ├─ OutlierStatisticsAnalyzer.hpp
   │  ├─ TrackTableAnalyzer.hpp
   │  └─ StereoTrackAnalysisReportWriter.hpp
   │
   └─ src/
      ├─ StereoTrack.cpp
      ├─ StereoTrackManager.cpp
      ├─ ConstantVelocityMotionModel.cpp
      ├─ StereoMotionSimulator.cpp
      ├─ StereoCommandGraphRunner.cpp
      ├─ StabilityScoreAnalyzer.cpp
      ├─ OutlierStatisticsAnalyzer.cpp
      ├─ TrackTableAnalyzer.cpp
      └─ StereoTrackAnalysisReportWriter.cpp
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
tracking_path
graph_path
motion_path
analyzer_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoTrackTableConfigBuilder`

Tạo class con:

```python
class StereoTrackTableConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `set_stereo_intrinsics(...)`
### `set_tracking_config(...)`
### `set_graph_order(command_names)`
### `set_analyzer_weights(alpha_depth_variance, outlier_penalty)`

### `add_moving_landmark(
    landmark_id,
    x, y, z,
    vx, vy, vz
)`

### `write_intrinsics_config()`
### `write_tracking_config()`
### `write_graph_config()`
### `write_motion_config()`
### `write_analyzer_config()`

---

# 10.3 Python — `ranking_preview.py`

Tạo class:

```python
class RankingPreview:
```

## Hàm cần có

### `compute_stability_score(valid_count, outlier_count, depth_variance, alpha, penalty)`
### `sort_tracks(track_dict_list)`
### `show_top_k(track_dict_list, k)`

---

# 10.4 C++ — `TrackAnalysisResult`

```cpp
struct TrackAnalysisResult
{
    std::string landmark_id;

    int valid_observation_count;
    int outlier_count;

    double smoothed_disparity;
    double smoothed_depth;

    double disparity_variance;
    double depth_variance;

    double stability_score;
    std::string quality_label;
};
```

---

# 10.5 C++ — `AnalyzerDebugRecord`

```cpp
struct AnalyzerDebugRecord
{
    std::string analyzer_name;
    std::string message;
};
```

---

# 10.6 C++ — `BaseTrackAnalyzer`

Tạo abstract class:

```cpp
class BaseTrackAnalyzer
{
public:
    virtual std::string name() const = 0;

    virtual void analyze(
        const std::unordered_map<std::string, StereoTrack>& track_table,
        std::vector<TrackAnalysisResult>& output_results
    ) = 0;

    virtual ~BaseTrackAnalyzer() = default;
};
```

---

# 10.7 C++ — `StabilityScoreAnalyzer`

Kế thừa:

```cpp
class StabilityScoreAnalyzer : public BaseTrackAnalyzer
```

## Nhiệm vụ
- duyệt toàn bộ track table
- tính:
  - valid count
  - outlier count
  - smoothed disparity
  - smoothed depth
  - disparity variance
  - depth variance
  - stability score
- gán `quality_label`

## Gợi ý quality
- `EXCELLENT`
- `GOOD`
- `WEAK`
- `UNSTABLE`

---

# 10.8 C++ — `OutlierStatisticsAnalyzer`

Kế thừa:

```cpp
class OutlierStatisticsAnalyzer : public BaseTrackAnalyzer
```

## Nhiệm vụ
- phân tích track nào có:
  - nhiều outlier nhất
  - outlier ratio cao nhất
- có thể bổ sung cờ cảnh báo vào `TrackAnalysisResult`

---

# 10.9 C++ — `TrackTableAnalyzer`

Tạo class trung tâm:

```cpp
class TrackTableAnalyzer
```

## Thuộc tính

```cpp
private:
    std::vector<std::shared_ptr<BaseTrackAnalyzer>> analyzers;
    std::vector<TrackAnalysisResult> results;
    std::stack<AnalyzerDebugRecord> debug_history;
```

## Hàm cần có

### `register_analyzer(std::shared_ptr<BaseTrackAnalyzer> analyzer)`
### `run(const std::unordered_map<std::string, StereoTrack>& track_table)`
- chạy lần lượt từng analyzer

### `std::vector<TrackAnalysisResult> get_results_sorted_by_stability() const`
- sort giảm dần theo stability score

### `TrackAnalysisResult get_worst_outlier_track() const`
### `TrackAnalysisResult get_highest_depth_variance_track() const`
### `std::vector<AnalyzerDebugRecord> get_debug_history_reverse()`

---

# 10.10 C++ — `StereoTrackAnalysisReportWriter`

Tạo class:

```cpp
class StereoTrackAnalysisReportWriter
```

## Hàm cần có

### `write_track_table_analysis_report(...)`
Ví dụ:

```text
[Track Analysis]
Landmark: cup_01
Valid: 12
Outlier: 1
Smoothed Depth: 1.82
Depth Variance: 0.03
Stability Score: 10.4
Quality: GOOD
```

### `write_track_ranking_report(...)`
Ví dụ:

```text
[Track Ranking]
1. box_01   score=14.7
2. cup_01   score=10.4
3. bottle_01 score=8.8
4. ball_01  score=3.2
```

### `write_reverse_debug_history(...)`

---

# 10.11 C++ — `main.cpp`

## Yêu cầu

```text
Load intrinsics + tracking + graph + motion + analyzer config
Create motion simulator
Create command graph runner
Simulate multi-landmark observation queue
Run command graph
Get track table
Create TrackTableAnalyzer
Register analyzers
Run analyzers
Write reports
```

---

# 11. Luật phân tích track table

## Luật 1 — Mỗi `landmark_id` chỉ có 1 StereoTrack trong track table

## Luật 2 — Track analysis chỉ chạy sau khi observation pipeline kết thúc

## Luật 3 — Ranking phải sort giảm dần theo stability score

## Luật 4 — Một track có quá ít valid observation thì không được xếp `EXCELLENT`

## Luật 5 — Nếu outlier ratio quá cao thì track phải bị hạ quality

---

# 12. Output mong muốn

## Config
```text
config/stereo_intrinsics_config.txt
config/stereo_tracking_config.txt
config/stereo_command_graph_config.txt
config/landmark_motion_config.txt
config/track_analyzer_config.txt
```

## Reports
```text
assets/outputs/landmark_motion_report.txt
assets/outputs/generated_observation_queue_report.txt
assets/outputs/stereo_track_summary_report.txt
assets/outputs/track_table_analysis_report.txt
assets/outputs/track_ranking_report.txt
assets/outputs/reverse_debug_history.txt
```

---

# 13. Vai trò trong Humanoid Robot

## Python
- build config cho multi-landmark simulation
- preview ranking logic
- chỉnh trọng số analyzer

## C++
- quản lý observation pipeline nhiều landmark
- duy trì track table
- phân tích chất lượng toàn bộ bảng track
- xuất ranking cho downstream perception

## Computer Vision / Robot Perception
Đây là bước rất quan trọng vì perception thực tế gần như luôn phải xử lý **nhiều object / nhiều landmark cùng lúc**.  
Bài 39 giúp bạn chuyển tư duy từ:

```text
1 observation / 1 track
```

sang:

```text
nhiều landmark
→ nhiều track
→ cần bảng đánh giá chất lượng tổng thể
```

---

# 14. Checklist hoàn thành

- [ ] Python build đủ 5 config
- [ ] Python có ranking preview
- [ ] C++ có `BaseTrackAnalyzer`
- [ ] C++ có `StabilityScoreAnalyzer`
- [ ] C++ có `OutlierStatisticsAnalyzer`
- [ ] C++ có `TrackTableAnalyzer`
- [ ] C++ phân tích được toàn bộ track table
- [ ] C++ sort được ranking
- [ ] C++ tìm được worst outlier track
- [ ] C++ tìm được highest depth variance track
- [ ] C++ ghi đủ 6 report

---

# 15. Gợi ý mở rộng

## 1. Thêm analyzer cho “track lifespan”
Track nào tồn tại ổn định lâu nhất qua nhiều frame.

## 2. Thêm analyzer cho “depth drift”
So depth đầu track và cuối track chênh nhau bao nhiêu.

## 3. Xuất CSV / bảng thống kê
Để sau này dễ nối sang Python visualization.

## 4. Chuẩn bị cho Bài 40
Bài 40 nên đi tiếp theo hướng:

# **Stereo Track Quality Dashboard Generator**

Ý tưởng:
- đọc kết quả track analysis
- sinh bảng dashboard / summary card
- gom các chỉ số:
  - best track
  - worst track
  - outlier leaderboard
  - depth variance leaderboard
- bắt đầu chuyển từ “analyze” sang “reporting / dashboard / evaluation layer”

Tức là Bài 40 sẽ là bước rất đẹp để **kết thúc Đợt 8** bằng một project thiên về **evaluation + visualization + perception reporting**.
