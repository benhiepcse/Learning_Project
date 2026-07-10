# 🤖 Bài 20: Humanoid Multi-Task Perception Manager

> Mini Project số 20 trong **Đợt 4**  
> Kết hợp kiến thức của **Đợt 1 + Đợt 2 + Đợt 3 + Đợt 4**.

---

## 📌 Mục tiêu chính

Bài này xây một hệ thống **Humanoid Multi-Task Perception Manager** để robot có thể nhìn một scene và quyết định:

```text
FOLLOW_OBJECT
PICK_OBJECT
PUSH_OBJECT
IGNORE_OBJECT
```

Nếu các bài trước xử lý từng nhánh riêng lẻ:

```text
Bài 16 → Perception Pipeline Supervisor
Bài 17 → Object Following Vision
Bài 18 → Pick Candidate Vision
Bài 19 → Table Cleaning Vision
```

thì **Bài 20** sẽ gom tất cả lại thành một manager tổng hợp:

```text
stereo input
→ object localization
→ follow decision
→ pick decision
→ table cleaning decision
→ multi-task arbitration
→ final scene task summary
```

---

## 🧠 Kiến thức cần

### Python

```text
OOP
Module
File Handling
List / Dict
Config Builder
Graph / BST nếu muốn mở rộng
```

### C++

```text
Class
Inheritance
Virtual Function
Enum Class
Vector
Smart Pointer
Struct
File Parsing
Report Writer
```

### Computer Vision / Robot Perception

```text
Stereo Camera
Depth / Disparity
Object Localization
Camera-to-Robot Transform
Object Following
Pick Candidate
Table Cleaning
Humanoid Perception Pipeline
```

---

## 🧱 Cấu trúc folder

```text
mini_project_20_humanoid_multi_task_perception_manager/
│
├─ README.md
├─ requirements.txt
│
├─ config/
│  ├─ stereo_frame_config.txt
│  ├─ roi_config.txt
│  ├─ stereo_bm_config.txt
│  ├─ calibration_config.txt
│  ├─ intrinsic_config.txt
│  ├─ camera_to_robot_config.txt
│  ├─ follow_decision_config.txt
│  ├─ pick_decision_config.txt
│  ├─ table_cleaning_config.txt
│  └─ multi_task_decision_config.txt
│
├─ assets/
│  ├─ left_images/
│  ├─ right_images/
│  └─ outputs/
│     ├─ object_task_report.txt
│     ├─ scene_task_summary_report.txt
│     ├─ multi_task_runtime_report.txt
│     └─ debug_history.txt
│
├─ python/
│  ├─ main_config_builder.py
│  └─ tools/
│     ├─ config_builder.py
│     ├─ scene_graph_preview.py
│     └─ task_bst.py
│
└─ cpp/
   ├─ main.cpp
   ├─ include/
   │  ├─ BaseSensor.hpp
   │  ├─ StereoCameraSensor.hpp
   │  ├─ StereoFrameRecord.hpp
   │  ├─ ROIConfig.hpp
   │  ├─ StereoBMConfig.hpp
   │  ├─ StereoCalibrationConfig.hpp
   │  ├─ CameraIntrinsics.hpp
   │  ├─ Transform3D.hpp
   │  ├─ Point3D.hpp
   │  ├─ RobotFramePoint3D.hpp
   │  ├─ LocalizedObject3D.hpp
   │  ├─ TaskPriority.hpp
   │  ├─ FollowDecisionConfig.hpp
   │  ├─ PickDecisionConfig.hpp
   │  ├─ TableCleaningDecisionConfig.hpp
   │  ├─ MultiTaskDecisionConfig.hpp
   │  ├─ ObjectTaskDecision.hpp
   │  ├─ SceneTaskSummary.hpp
   │  ├─ BaseMultiTaskPerceptionManager.hpp
   │  ├─ HumanoidMultiTaskPerceptionManager.hpp
   │  └─ MultiTaskPerceptionReportWriter.hpp
   │
   └─ src/
      ├─ StereoCameraSensor.cpp
      ├─ HumanoidMultiTaskPerceptionManager.cpp
      └─ MultiTaskPerceptionReportWriter.cpp
```

---

# 🐍 Python Requirements

## `BaseConfigBuilder`

```python
class BaseConfigBuilder:
    def __init__(self, project_name: str):
        self.project_name = project_name

    def show_project_info(self):
        print(f"Project: {self.project_name}")

    def validate_positive_number(self, value, name):
        if value <= 0:
            raise ValueError(f"{name} must be positive")
```

---

## `MultiTaskPerceptionConfigBuilder`

```python
class MultiTaskPerceptionConfigBuilder(BaseConfigBuilder):
    def __init__(self, project_name: str):
        super().__init__(project_name)
        self.stereo_frames = []
        self.rois = []
        self.configs = {}

    def add_stereo_frame(self, frame_id, left_path, right_path):
        self.stereo_frames.append({
            "frame_id": frame_id,
            "left_path": left_path,
            "right_path": right_path
        })

    def add_roi(self, object_id, x, y, width, height, label):
        self.rois.append({
            "object_id": object_id,
            "x": x,
            "y": y,
            "width": width,
            "height": height,
            "label": label
        })

    def set_follow_decision_config(self, center_tolerance, max_follow_distance):
        self.configs["follow"] = {
            "center_tolerance": center_tolerance,
            "max_follow_distance": max_follow_distance
        }

    def set_pick_decision_config(self, max_pick_distance, max_lateral_offset):
        self.configs["pick"] = {
            "max_pick_distance": max_pick_distance,
            "max_lateral_offset": max_lateral_offset
        }

    def set_table_cleaning_config(self, table_min_z, table_max_z):
        self.configs["table_cleaning"] = {
            "table_min_z": table_min_z,
            "table_max_z": table_max_z
        }

    def set_multi_task_decision_config(self, follow_weight, pick_weight, clean_weight):
        self.configs["multi_task"] = {
            "follow_weight": follow_weight,
            "pick_weight": pick_weight,
            "clean_weight": clean_weight
        }
```

---

# 🧩 C++ Core Structures

## `StereoFrameRecord`

```cpp
struct StereoFrameRecord
{
    std::string frame_id;
    std::string left_image_path;
    std::string right_image_path;
};
```

---

## `ROIConfig`

```cpp
struct ROIConfig
{
    std::string object_id;
    int x;
    int y;
    int width;
    int height;
    std::string label;
};
```

---

## `Point3D`

```cpp
struct Point3D
{
    double x;
    double y;
    double z;
};
```

---

## `RobotFramePoint3D`

```cpp
struct RobotFramePoint3D
{
    double forward;
    double lateral;
    double height;
};
```

---

## `LocalizedObject3D`

```cpp
struct LocalizedObject3D
{
    std::string object_id;
    std::string label;

    Point3D camera_point;
    RobotFramePoint3D robot_point;

    double confidence;
};
```

---

## `TaskPriority`

```cpp
enum class TaskPriority
{
    FOLLOW_OBJECT,
    PICK_OBJECT,
    PUSH_OBJECT,
    IGNORE_OBJECT,
    INVALID_OBJECT
};
```

---

## `FollowDecisionConfig`

```cpp
struct FollowDecisionConfig
{
    double center_tolerance;
    double max_follow_distance;
};
```

---

## `PickDecisionConfig`

```cpp
struct PickDecisionConfig
{
    double max_pick_distance;
    double max_lateral_offset;
};
```

---

## `TableCleaningDecisionConfig`

```cpp
struct TableCleaningDecisionConfig
{
    double table_min_z;
    double table_max_z;
};
```

---

## `MultiTaskDecisionConfig`

```cpp
struct MultiTaskDecisionConfig
{
    double follow_weight;
    double pick_weight;
    double clean_weight;
};
```

---

## `ObjectTaskDecision`

```cpp
struct ObjectTaskDecision
{
    std::string object_id;
    std::string label;

    TaskPriority task_priority;

    double follow_score;
    double pick_score;
    double cleaning_score;
    double final_score;

    std::string reason;
};
```

---

## `SceneTaskSummary`

```cpp
struct SceneTaskSummary
{
    std::string frame_id;

    int follow_count;
    int pick_count;
    int push_count;
    int ignore_count;

    TaskPriority scene_priority;
    std::string summary;
};
```

---

# 🧠 C++ Manager Design

## `BaseMultiTaskPerceptionManager`

```cpp
class BaseMultiTaskPerceptionManager
{
public:
    virtual void load_configs() = 0;
    virtual void run() = 0;

    virtual std::vector<ObjectTaskDecision> get_object_decisions() const = 0;
    virtual SceneTaskSummary get_scene_summary() const = 0;

    virtual ~BaseMultiTaskPerceptionManager() = default;
};
```

---

## `HumanoidMultiTaskPerceptionManager`

```cpp
class HumanoidMultiTaskPerceptionManager : public BaseMultiTaskPerceptionManager
{
private:
    std::vector<StereoFrameRecord> stereo_frames;
    std::vector<ROIConfig> rois;
    std::vector<LocalizedObject3D> localized_objects;
    std::vector<ObjectTaskDecision> object_decisions;

    FollowDecisionConfig follow_config;
    PickDecisionConfig pick_config;
    TableCleaningDecisionConfig cleaning_config;
    MultiTaskDecisionConfig multi_task_config;

    SceneTaskSummary scene_summary;

public:
    void load_configs() override;
    void run() override;

    std::vector<ObjectTaskDecision> get_object_decisions() const override;
    SceneTaskSummary get_scene_summary() const override;

private:
    double compute_follow_score(const LocalizedObject3D& object) const;
    double compute_pick_score(const LocalizedObject3D& object) const;
    double compute_cleaning_score(const LocalizedObject3D& object) const;

    ObjectTaskDecision decide_object_task(const LocalizedObject3D& object) const;
    SceneTaskSummary build_scene_summary() const;
};
```

---

# 🧾 Report Writer

## `MultiTaskPerceptionReportWriter`

```cpp
class MultiTaskPerceptionReportWriter
{
public:
    void write_object_task_report(
        const std::string& path,
        const std::vector<ObjectTaskDecision>& decisions
    ) const;

    void write_scene_task_summary_report(
        const std::string& path,
        const SceneTaskSummary& summary
    ) const;

    void write_runtime_report(
        const std::string& path
    ) const;
};
```

---

# 🚀 `main.cpp`

```cpp
#include <iostream>
#include <memory>

#include "HumanoidMultiTaskPerceptionManager.hpp"
#include "MultiTaskPerceptionReportWriter.hpp"

int main()
{
    auto manager = std::make_shared<HumanoidMultiTaskPerceptionManager>();

    manager->load_configs();
    manager->run();

    MultiTaskPerceptionReportWriter writer;

    writer.write_object_task_report(
        "assets/outputs/object_task_report.txt",
        manager->get_object_decisions()
    );

    writer.write_scene_task_summary_report(
        "assets/outputs/scene_task_summary_report.txt",
        manager->get_scene_summary()
    );

    writer.write_runtime_report(
        "assets/outputs/multi_task_runtime_report.txt"
    );

    std::cout << "Humanoid Multi-Task Perception Manager finished successfully.\n";

    return 0;
}
```

---

# ⚙️ Decision Logic Gợi Ý

## Follow Score

```text
follow_score cao khi:
- object nằm gần center
- object ở khoảng cách vừa phải
- object có confidence cao
```

---

## Pick Score

```text
pick_score cao khi:
- object nằm phía trước robot
- khoảng cách không quá xa
- lateral offset nhỏ
- object nằm trong vùng có thể gắp
```

---

## Cleaning Score

```text
cleaning_score cao khi:
- object nằm trong table zone
- object cần được dọn
- object không phải invalid object
```

---

## Final Score

```text
final_score =
    follow_score * follow_weight
  + pick_score * pick_weight
  + cleaning_score * clean_weight
```

---

# 📤 Output mong muốn

```text
assets/outputs/object_task_report.txt
assets/outputs/scene_task_summary_report.txt
assets/outputs/multi_task_runtime_report.txt
assets/outputs/debug_history.txt
```

Ví dụ:

```text
[Object Task Decision]
Object ID: obj_01
Label: bottle
Follow Score: 0.62
Pick Score: 0.91
Cleaning Score: 0.73
Final Task: PICK_OBJECT
Reason: Object is reachable, centered, and suitable for pick operation.
```

```text
[Scene Task Summary]
Frame ID: frame_001
Follow Objects: 1
Pick Objects: 2
Push Objects: 1
Ignored Objects: 3
Scene Priority: PICK_OBJECT
Summary: The scene contains multiple reachable objects. Pick task should be prioritized.
```

---

# ✅ Checklist hoàn thành

- [ ] Python tạo được toàn bộ config cho stereo / ROI / decision
- [ ] Python có config builder rõ ràng
- [ ] Python có graph preview hoặc BST nếu muốn mở rộng
- [ ] C++ có đủ structs / enums
- [ ] C++ có abstract manager
- [ ] C++ có concrete manager
- [ ] C++ tính được follow score
- [ ] C++ tính được pick score
- [ ] C++ tính được cleaning score
- [ ] C++ chọn được task cuối cho từng object
- [ ] C++ sinh được scene summary
- [ ] C++ ghi đủ report
- [ ] Folder đúng cấu trúc mini-project

---

# 🌟 Vai trò trong Humanoid Robot

Bài này rất quan trọng vì nó mô phỏng một tầng perception gần với robot thật:

```text
Robot nhìn scene
→ hiểu từng object
→ gán task cho từng object
→ chọn priority của scene
→ gửi decision cho behavior / planning layer
```

Đây là bước nối giữa:

```text
Computer Vision
→ Robot Perception
→ Task Selection
→ Humanoid Behavior
```

---

# 🔥 Gợi ý mở rộng

Sau Bài 20, bạn có thể mở rộng sang Đợt 5 bằng:

```text
Bài 21: Depth-Based Object Distance Estimator
Bài 22: Disparity Quality Analyzer
Bài 23: Point Cloud Object Extractor
Bài 24: Robot-Frame Obstacle Zone Classifier
Bài 25: Stereo Vision Perception Debug Dashboard
```
