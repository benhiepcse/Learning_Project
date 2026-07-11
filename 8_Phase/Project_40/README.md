# 🤖 Bài 40: Stereo Track Quality Dashboard Generator — Bộ tạo dashboard chất lượng track cho Humanoid Robot AI Perception

> Mini Project số 40 trong **Đợt 8**  
> **Bài 40 kết hợp kiến thức của Đợt 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8** và là bài **khóa Đợt 8** sau chuỗi:
>
> - **Bài 36**: Stereo Disparity Track Manager  
> - **Bài 37**: Stereo Depth Command Graph Runner  
> - **Bài 38**: Stereo Landmark Motion Queue Simulator  
> - **Bài 39**: Stereo Multi-Landmark Track Table Analyzer
>
> Nếu **Bài 39** đã giúp bạn:
>
> ```text
> nhiều landmark
> → nhiều track
> → track table
> → stability / outlier / variance analysis
> ```
>
> thì **Bài 40** sẽ đi tiếp thành:
>
> ```text
> track analysis results
> → dashboard generator
> → best / worst track cards
> → leaderboard
> → quality distribution
> → perception evaluation summary
> ```
>
> Tức là bạn chuyển từ **phân tích track** sang **tầng reporting / evaluation / dashboard** — đúng kiểu bước cuối của một perception mini-pipeline.

---

# 📌 Mục lục

- [1. Vì sao Bài 40 xuất hiện sau Bài 39](#1-vì-sao-bài-40-xuất-hiện-sau-bài-39)
- [2. Đợt 8 đang học gì](#2-đợt-8-đang-học-gì)
- [3. Mô tả](#3-mô-tả)
- [4. Mục tiêu perception](#4-mục-tiêu-perception)
- [5. Ý tưởng Dashboard Generator](#5-ý-tưởng-dashboard-generator)
- [6. Pipeline tổng thể](#6-pipeline-tổng-thể)
- [7. Kiến thức cần](#7-kiến-thức-cần)
- [8. DSA + Algorithm bắt buộc](#8-dsa--algorithm-bắt-buộc)
- [9. Cấu trúc folder](#9-cấu-trúc-folder)
- [10. Yêu cầu mini-project](#10-yêu-cầu-mini-project)
- [11. Luật sinh dashboard](#11-luật-sinh-dashboard)
- [12. Output mong muốn](#12-output-mong-muốn)
- [13. Vai trò trong Humanoid Robot](#13-vai-trò-trong-humanoid-robot)
- [14. Checklist hoàn thành](#14-checklist-hoàn-thành)
- [15. Gợi ý mở rộng](#15-gợi-ý-mở-rộng)

---

# 1. Vì sao Bài 40 xuất hiện sau Bài 39

## Bài 39 đang làm gì?
Bài 39 đã có:

```text
multi-landmark motion
→ stereo observation queue
→ command graph
→ track table
→ track analysis
→ ranking
```

Nó đã rất mạnh ở tầng **analysis**:
- tính stability score
- tính outlier statistics
- tính depth variance
- xếp hạng track

## Bài 40 nâng lên ở đâu?
Bài 40 sẽ không dừng ở việc “có kết quả phân tích”, mà đi tiếp đến câu hỏi:

> Nếu tôi là perception engineer hoặc người review pipeline, **tôi muốn nhìn kết quả đó như thế nào để ra quyết định nhanh?**

Vì vậy Bài 40 sẽ gom kết quả từ Bài 39 thành một **dashboard báo cáo** gồm:

- **best track card**
- **worst track card**
- **outlier leaderboard**
- **depth variance leaderboard**
- **quality distribution**
- **overall perception summary**

Tức là bạn chuyển từ **raw analysis** sang **decision-friendly reporting**.

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

## Vì sao Bài 40 vẫn bám Đợt 8?
Vì Bài 40 vẫn dùng trọn bộ:
- **Queue** → observation stream từ pipeline cũ
- **unordered_map / vector / deque** → track table + ranking + dashboard sections
- **abstract class** → dashboard section builder / summary analyzer
- **inheritance** → từng loại dashboard section
- **CV / stereo** → dashboard được dựng từ disparity / depth / stability / outlier metrics

Nói ngắn gọn: Bài 40 là **tầng báo cáo** xây trên đúng dữ liệu stereo perception mà bạn đã tạo ở các bài trước.

---

# 3. Mô tả

Bạn sẽ xây một mini system tên là:

# **Stereo Track Quality Dashboard Generator**

System này lấy đầu vào từ **kết quả track analysis** của Bài 39, ví dụ:

```text
cup_01:
    stability_score = 11.2
    outlier_count = 1
    depth_variance = 0.03
    quality = GOOD

ball_01:
    stability_score = 4.1
    outlier_count = 5
    depth_variance = 0.21
    quality = UNSTABLE

box_01:
    stability_score = 14.7
    outlier_count = 0
    depth_variance = 0.01
    quality = EXCELLENT
```

Sau đó generator sẽ tạo ra **dashboard summary** như:

## Dashboard Cards
- **Best Track**: `box_01`
- **Worst Track**: `ball_01`
- **Most Outlier Track**: `ball_01`
- **Most Stable Track**: `box_01`

## Dashboard Tables / Leaderboards
- Top 5 track theo stability
- Top 5 track nhiều outlier
- Top 5 track depth variance cao

## Dashboard Summary
- số track `EXCELLENT`
- số track `GOOD`
- số track `WEAK`
- số track `UNSTABLE`
- overall pipeline quality label

<p align="center">
  <img src="../../images/project_40.png" width="800">
</p>

---

# 4. Mục tiêu perception

Sau bài này bạn phải hiểu pipeline hoàn chỉnh của Đợt 8:

```text
Python
→ tạo multi-landmark motion config
→ tạo command graph config
→ tạo analyzer config
→ tạo dashboard config

C++
→ simulate landmarks
→ build stereo observations
→ run command graph
→ update track table
→ analyze track table
→ build dashboard summary
→ export perception evaluation reports
```

Mục tiêu của bài không chỉ là “có metric”, mà là:
- biết **tổ chức metric thành dashboard**
- biết **tóm tắt chất lượng pipeline**
- biết **trình bày output perception cho người đọc / reviewer**

---

# 5. Ý tưởng Dashboard Generator

Bạn có thể hình dung dashboard như một bản tóm tắt cuối cùng của pipeline.

## Ví dụ dashboard text

```text
==================================================
Stereo Track Quality Dashboard
==================================================

[Overview]
Total Tracks: 6
Excellent: 2
Good: 2
Weak: 1
Unstable: 1
Overall Pipeline Quality: GOOD

[Best Track]
Landmark: box_01
Score: 14.7
Depth Variance: 0.01
Outlier Count: 0

[Worst Track]
Landmark: ball_01
Score: 4.1
Depth Variance: 0.21
Outlier Count: 5

[Top Stability Ranking]
1. box_01
2. cup_01
3. bottle_01
...

[Outlier Leaderboard]
1. ball_01
2. bottle_01
...

[Depth Variance Leaderboard]
1. ball_01
2. cup_02
...
```

---

# 6. Pipeline tổng thể

```text
Load Stereo Intrinsics
Load Tracking Config
Load Motion Config
Load Command Graph Config
Load Analyzer Config
Load Dashboard Config

Run full stereo pipeline:
    simulate landmarks
    build observation queue
    run command graph
    build track table
    analyze track table

Take TrackAnalysisResult list
Generate dashboard sections:
    overview
    best/worst cards
    ranking leaderboard
    quality distribution
    final summary

Write dashboard reports
```

---

# 7. Kiến thức cần

## Python
- class / inheritance / `super()`
- module
- queue / stack / linked list
- file handling
- config builder
- summary / ranking preview

## C++
- class / inheritance
- abstract class / pure virtual
- copy constructor
- `std::vector`
- `std::unordered_map`
- `std::deque`
- `std::queue`
- `std::stack`
- `std::sort`
- `std::shared_ptr`
- dashboard / reporting design

## Computer Vision / Perception
- disparity / depth metrics
- temporal track quality
- perception evaluation summary
- stability / outlier / variance interpretation

---

# 8. DSA + Algorithm bắt buộc

# 8.1 DSA bắt buộc

## 1. `std::vector<TrackAnalysisResult>`
Danh sách kết quả phân tích track.

## 2. `std::unordered_map<std::string, int>`
Dùng cho quality distribution:
- EXCELLENT
- GOOD
- WEAK
- UNSTABLE

## 3. `std::vector<DashboardCard>`
Lưu các card như best / worst / most outlier.

## 4. `std::vector<std::shared_ptr<BaseDashboardSectionBuilder>>`
Danh sách các builder tạo section dashboard.

## 5. `std::stack<DashboardDebugRecord>`
Reverse debug history.

---

# 8.2 Algorithm bắt buộc

## Algorithm 1 — Best / Worst track selection
Từ `TrackAnalysisResult`:
- best track = stability score lớn nhất
- worst track = stability score nhỏ nhất

---

## Algorithm 2 — Quality distribution
Đếm số track thuộc từng nhãn:
- EXCELLENT
- GOOD
- WEAK
- UNSTABLE

---

## Algorithm 3 — Leaderboard generation
Sort theo:
- stability score giảm dần
- outlier count giảm dần
- depth variance giảm dần

---

## Algorithm 4 — Overall pipeline quality
Bạn phải thiết kế rule để đánh giá toàn pipeline.

### Ví dụ rule
- nếu đa số track là `EXCELLENT/GOOD` và số `UNSTABLE` ít → `GOOD`
- nếu có quá nhiều `UNSTABLE` → `WEAK`
- nếu top track rất mạnh và median track cũng ổn → `VERY GOOD`

Bạn có thể tự đặt nhãn như:
- `EXCELLENT`
- `GOOD`
- `WEAK`
- `UNSTABLE`

---

## Algorithm 5 — Dashboard section building
Mỗi section builder nhận `TrackAnalysisResult list` và sinh ra một phần dashboard riêng.

---

# 9. Cấu trúc folder

```text
mini_project_40_stereo_track_quality_dashboard_generator/
│
├─ README.md
├─ requirements.txt
│
├─ assets/
│  └─ outputs/
│     ├─ stereo_track_summary_report.txt
│     ├─ track_table_analysis_report.txt
│     ├─ track_ranking_report.txt
│     ├─ dashboard_overview_report.txt
│     ├─ dashboard_leaderboard_report.txt
│     ├─ dashboard_summary_report.txt
│     └─ reverse_debug_history.txt
│
├─ config/
│  ├─ stereo_intrinsics_config.txt
│  ├─ stereo_tracking_config.txt
│  ├─ stereo_command_graph_config.txt
│  ├─ landmark_motion_config.txt
│  ├─ track_analyzer_config.txt
│  └─ dashboard_config.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ ranking_preview.py
│     ├─ dashboard_preview.py
│     └─ trajectory_preview.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ TrackAnalysisResult.hpp
   │  ├─ DashboardCard.hpp
   │  ├─ DashboardSummary.hpp
   │  ├─ DashboardDebugRecord.hpp
   │  ├─ BaseDashboardSectionBuilder.hpp
   │  ├─ OverviewSectionBuilder.hpp
   │  ├─ BestWorstSectionBuilder.hpp
   │  ├─ LeaderboardSectionBuilder.hpp
   │  ├─ QualityDistributionSectionBuilder.hpp
   │  ├─ StereoTrackDashboardGenerator.hpp
   │  └─ StereoDashboardReportWriter.hpp
   │
   └─ src/
      ├─ OverviewSectionBuilder.cpp
      ├─ BestWorstSectionBuilder.cpp
      ├─ LeaderboardSectionBuilder.cpp
      ├─ QualityDistributionSectionBuilder.cpp
      ├─ StereoTrackDashboardGenerator.cpp
      └─ StereoDashboardReportWriter.cpp
```

> Lưu ý: Bài 40 **dùng lại pipeline của Bài 39** để tạo `TrackAnalysisResult`.  
> Phần trọng tâm mới ở đây là **dashboard layer**, nên bạn không cần viết lại toàn bộ logic từ Bài 36–39 trong README này; bạn chỉ cần nói rõ là Bài 40 **tích hợp đầu ra của các bài trước**.

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
dashboard_config_path
```

## Hàm cần có
### `show_project_info()`
### `create_default_paths(...)`
### `validate_positive_number(...)`

---

# 10.2 Python — `StereoDashboardConfigBuilder`

Tạo class con:

```python
class StereoDashboardConfigBuilder(BaseConfigBuilder):
```

## Hàm cần có

### `set_dashboard_options(
    top_k_stability,
    top_k_outlier,
    top_k_depth_variance,
    enable_best_worst_section,
    enable_quality_distribution_section
)`

### `write_dashboard_config()`

---

# 10.3 Python — `dashboard_preview.py`

Tạo class:

```python
class DashboardPreview:
```

## Hàm cần có

### `show_overview(total_tracks, quality_counts)`
### `show_best_worst(best_track, worst_track)`
### `show_leaderboard(title, rows)`

---

# 10.4 C++ — `DashboardCard`

```cpp
struct DashboardCard
{
    std::string title;
    std::string landmark_id;
    double score;
    std::string note;
};
```

Ví dụ:
- `"Best Track"`
- `"Worst Track"`
- `"Most Outlier Track"`

---

# 10.5 C++ — `DashboardSummary`

```cpp
struct DashboardSummary
{
    int total_tracks;

    int excellent_count;
    int good_count;
    int weak_count;
    int unstable_count;

    std::string overall_pipeline_quality;
};
```

---

# 10.6 C++ — `DashboardDebugRecord`

```cpp
struct DashboardDebugRecord
{
    std::string section_name;
    std::string message;
};
```

---

# 10.7 C++ — `BaseDashboardSectionBuilder`

Tạo abstract class:

```cpp
class BaseDashboardSectionBuilder
{
public:
    virtual std::string name() const = 0;

    virtual void build(
        const std::vector<TrackAnalysisResult>& analysis_results,
        class StereoTrackDashboardGenerator& dashboard
    ) = 0;

    virtual ~BaseDashboardSectionBuilder() = default;
};
```

---

# 10.8 C++ — `OverviewSectionBuilder`

Kế thừa:

```cpp
class OverviewSectionBuilder : public BaseDashboardSectionBuilder
```

## Nhiệm vụ
- đếm tổng số track
- đếm quality distribution
- quyết định `overall_pipeline_quality`

---

# 10.9 C++ — `BestWorstSectionBuilder`

Kế thừa:

```cpp
class BestWorstSectionBuilder : public BaseDashboardSectionBuilder
```

## Nhiệm vụ
- chọn:
  - best track
  - worst track
  - most outlier track
  - highest depth variance track
- sinh `DashboardCard`

---

# 10.10 C++ — `LeaderboardSectionBuilder`

Kế thừa:

```cpp
class LeaderboardSectionBuilder : public BaseDashboardSectionBuilder
```

## Nhiệm vụ
- tạo leaderboard:
  - stability
  - outlier
  - depth variance

---

# 10.11 C++ — `QualityDistributionSectionBuilder`

Kế thừa:

```cpp
class QualityDistributionSectionBuilder : public BaseDashboardSectionBuilder
```

## Nhiệm vụ
- build section mô tả số track thuộc từng nhãn quality
- có thể kèm tỉ lệ phần trăm

---

# 10.12 C++ — `StereoTrackDashboardGenerator`

Tạo class trung tâm:

```cpp
class StereoTrackDashboardGenerator
```

## Thuộc tính

```cpp
private:
    std::vector<TrackAnalysisResult> analysis_results;

    DashboardSummary summary;
    std::vector<DashboardCard> cards;

    std::vector<TrackAnalysisResult> stability_ranking;
    std::vector<TrackAnalysisResult> outlier_ranking;
    std::vector<TrackAnalysisResult> depth_variance_ranking;

    std::vector<std::shared_ptr<BaseDashboardSectionBuilder>> builders;
    std::stack<DashboardDebugRecord> debug_history;
```

## Hàm cần có

### `set_analysis_results(const std::vector<TrackAnalysisResult>& results)`

### `register_builder(std::shared_ptr<BaseDashboardSectionBuilder> builder)`

### `run()`
- chạy lần lượt từng builder

### `DashboardSummary& get_summary()`
### `const std::vector<DashboardCard>& get_cards() const`
### `const std::vector<TrackAnalysisResult>& get_stability_ranking() const`
### `const std::vector<TrackAnalysisResult>& get_outlier_ranking() const`
### `const std::vector<TrackAnalysisResult>& get_depth_variance_ranking() const`
### `std::vector<DashboardDebugRecord> get_debug_history_reverse()`

---

# 10.13 C++ — `StereoDashboardReportWriter`

Tạo class:

```cpp
class StereoDashboardReportWriter
```

## Hàm cần có

### `write_dashboard_overview_report(...)`
Ví dụ:

```text
[Overview]
Total Tracks: 6
Excellent: 2
Good: 2
Weak: 1
Unstable: 1
Overall Pipeline Quality: GOOD
```

### `write_dashboard_leaderboard_report(...)`
Ví dụ:

```text
[Top Stability]
1. box_01 score=14.7
2. cup_01 score=11.2
3. bottle_01 score=9.4

[Top Outlier]
1. ball_01 outlier=5
2. bottle_01 outlier=3
```

### `write_dashboard_summary_report(...)`
Ví dụ:

```text
[Dashboard Cards]
Best Track: box_01
Worst Track: ball_01
Most Outlier Track: ball_01
Highest Depth Variance Track: ball_01
```

### `write_reverse_debug_history(...)`

---

# 10.14 C++ — `main.cpp`

## Yêu cầu

```text
Run / reuse full pipeline from Bài 39
Get TrackAnalysisResult list
Create StereoTrackDashboardGenerator
Register:
    OverviewSectionBuilder
    BestWorstSectionBuilder
    LeaderboardSectionBuilder
    QualityDistributionSectionBuilder
Run dashboard generator
Write dashboard reports
```

---

# 11. Luật sinh dashboard

## Luật 1 — Dashboard chỉ chạy sau khi đã có `TrackAnalysisResult`
Không được build dashboard từ track raw chưa phân tích.

## Luật 2 — Best track và worst track phải dựa trên cùng một hệ score
Ví dụ đều dựa trên `stability_score`.

## Luật 3 — Nếu số track ít hơn `top_k`
Leaderboard chỉ in số track thực có.

## Luật 4 — Một track không được vừa là “best” vừa bị gắn note “unstable”
Logic quality phải nhất quán.

## Luật 5 — `overall_pipeline_quality` phải dựa trên toàn bộ distribution, không chỉ nhìn top 1 track

---

# 12. Output mong muốn

## Config
```text
config/dashboard_config.txt
```

## Reports
```text
assets/outputs/dashboard_overview_report.txt
assets/outputs/dashboard_leaderboard_report.txt
assets/outputs/dashboard_summary_report.txt
assets/outputs/reverse_debug_history.txt
```

> Nếu bạn muốn, có thể vẫn xuất thêm:
- `stereo_track_summary_report.txt`
- `track_table_analysis_report.txt`
- `track_ranking_report.txt`

để dashboard layer có đủ tài liệu nền.

---

# 13. Vai trò trong Humanoid Robot

## Python
- build dashboard config
- preview summary / leaderboard

## C++
- lấy output từ track analyzer
- gom thành dashboard dễ đọc
- tạo evaluation summary cho perception pipeline

## Computer Vision / Robot Perception
Đây là bước rất giống phần **evaluation/reporting** trong hệ perception thật:

- perception team chạy pipeline
- sinh metric
- sinh ranking
- cuối cùng phải có **dashboard / summary** để đọc nhanh tình trạng hệ thống

Tức là Bài 40 không chỉ là “làm đẹp output”, mà là học cách **đóng gói thông tin kỹ thuật thành báo cáo có thể dùng để ra quyết định**.

---

# 14. Checklist hoàn thành

- [ ] Python build được dashboard config
- [ ] Python có dashboard preview
- [ ] C++ có `BaseDashboardSectionBuilder`
- [ ] C++ có `OverviewSectionBuilder`
- [ ] C++ có `BestWorstSectionBuilder`
- [ ] C++ có `LeaderboardSectionBuilder`
- [ ] C++ có `QualityDistributionSectionBuilder`
- [ ] C++ có `StereoTrackDashboardGenerator`
- [ ] C++ sinh được overview report
- [ ] C++ sinh được leaderboard report
- [ ] C++ sinh được dashboard summary report
- [ ] C++ sinh được reverse debug history

---

# 15. Gợi ý mở rộng

## 1. Xuất dashboard dạng CSV / JSON
Để sau này nối sang web UI hoặc notebook.

## 2. Thêm “trend section”
Nếu chạy nhiều lần pipeline, so sánh dashboard giữa các lần.

## 3. Thêm “track alert section”
Ví dụ:
- track nào unstable quá lâu
- track nào outlier ratio > 40%

## 4. Chuẩn bị cho Đợt 9
Nếu sang **Đợt 9** và bắt đầu mạnh hơn vào **CV / stereo / geometry / estimation**, thì hướng tiếp theo rất đẹp là:

# **Stereo Epipolar Constraint Playground**
hoặc
# **Depth Error Benchmark Suite**

Tức là từ Đợt 8 thiên về **runtime / DSA / tracking / dashboard**, sang Đợt 9 bạn có thể quay lại đánh mạnh hơn vào **core vision geometry + estimation + evaluation**.
