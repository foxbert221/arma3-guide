# Arma 3 无人机侦查仿真场景 — 全流程手册
> 适用：军事AI多模态数据集构建 · 无人机侦查算法验证 · 可见光+热红外双模态采集

---

## 📌 示例场景设定

**场景名称**：Operation SILENT WATCH — 装甲集结区热成像侦查

**场景概述**：模拟一架中低空侦察无人机（高度 80–200m）对敌方装甲集结区的持续监视任务，任务中注入多类视觉异常，用于训练"异常目标检测"算法。

**场景参数：**

| 参数 | 值 |
|------|----|
| 地图 | Altis（默认地图，含沙丘 + 硬化道面 + 机库区域） |
| 无人机飞行高度 | 80 ~ 200m（可调） |
| 侦查角度 | 下视（Near-Nadir）+ 斜视 35°（Oblique） |
| 任务时长 | 8 分钟 / 场景（分 4 个子场景） |
| 输出格式 | 可见光 MP4 + 热成像 MP4 |
| 帧率 | 30fps |
| 分辨率 | 1920×1080 |

---

## 一、环境搭建

### 1.1 必需 Mod 清单

| Mod 名称 | Steam Workshop ID | 用途 |
|---------|-----------------|------|
| **RHS: Armed Forces of the Russian Federation** | 843425103 | 俄系载具（T-72B3/BMP-2/TOS-1A等）及热特征 |
| **RHS: United States Armed Forces** | 843577117 | 美系载具（M1A2/Bradley/M270等）及热特征 |
| **ACE3** | 463939057 | 高质量热成像 + 武器热瞄具 |
| **CBA_A3**（ACE3依赖） | 450814997 | ACE3 前置框架 |
| **CUP Terrains - Core** | 583496184 | 扩展地形支持 |
| **Task Force Arrowhead Radio（TFAR）** | 可选 | 通信模拟（可选） |

安装方法：
```
Steam → 库 → Arma 3 → 属性 → DLC & 模组 → 管理创意工坊内容
```
或直接订阅上述 Workshop ID。

### 1.2 OBS 录制配置

```
OBS Studio 配置：
  - 编码器：NVIDIA NVENC H.264（推荐）/ x264
  - 分辨率：1920×1080
  - 帧率：30fps（与游戏帧率锁定一致）
  - 码率：20 Mbps（保证画质）
  - 输出格式：MP4
  - 音频：关闭（仅录视频）
```

---

## 二、任务编辑器配置

### 2.1 打开任务编辑器

```
主菜单 → 编辑器 → 地图选择：Altis → 进入
```

### 2.2 场景布置要素

**步骤 1：放置玩家控制的无人机**

```
放置单位 → 空 → 载具 → 搜索"AR-2 Darter"或 RHS 中"MQ-9 Reaper"
位置：地图右上角机场跑道附近
侧面：BLUFOR（蓝方）
操控：玩家
```

**步骤 2：放置侦查目标**

```
目标 1：T-72B3（RHS）× 3 辆 — 集结于地图 056070 坐标区
目标 2：BMP-2（RHS）× 2 辆 — 与坦克相距 30m
目标 3：TOS-1A（火箭炮）× 1 辆 — 紧贴 BMP 南侧
目标 4：发电机模型（"Land_PowerGenerator_F" 或 "Land_GenPowerSmall_F"）× 2 — 帐篷旁
目标 5：帐篷（"Land_TentA_F"）× 3 组 — 集结区西侧
目标 6：掩体沙袋（"Land_BagBunker_Small_F"）× 若干 — 目标周围
```

**步骤 3：放置触发器（控制异常注入）**

在编辑器中添加 **Triggers（触发器）**：

```
触发器1：任务开始后 90 秒启动异常序列
  条件：false（由脚本控制激活）
  激活体：黑方  
  效果：执行脚本 anomaly_sequence.sqf
```

**步骤 4：放置初始化脚本（init.sqf）**

```
在编辑器菜单 → 任务 → 属性 → 激活时，填入：
execVM "init.sqf";
```

### 2.3 地形辅助贴图（场景二替代方案）

在目标周围手动放置以下模型，模拟痕迹：
```
"Land_DamageSystem_Track_F"    ← 履带痕（碎石纹路）
"Land_VehicleTrack_01_F"       ← 车辙模型
静止的 "Land_Decal_*" 类贴图面片  ← 弹坑/污迹
```

---

## 三、核心 SQF 脚本

### 3.1 `init.sqf` — 初始化脚本

```sqf
// =============================================
//  init.sqf — 场景初始化
//  Operation SILENT WATCH
// =============================================

// 1. 获取目标引用（与编辑器中单位名称对应）
_targets = [Tank_1, Tank_2, Tank_3, APC_1, APC_2, Rocket_1];
_staticObjs = [Generator_1, Generator_2, Tent_1, Tent_2, Tent_3];

// 2. 所有目标默认显示（可见）
{_x hideObjectGlobal false;} forEach (_targets + _staticObjs);

// 3. 初始尺寸归一
{_x setObjectScale 1;} forEach _targets;

// 4. 启动摄像机系统
execVM "camera_rig.sqf";

// 5. 延迟 90 秒后启动异常序列
[90] spawn {
  params ["_delay"];
  sleep _delay;
  execVM "anomaly_sequence.sqf";
};

hint "Scene Initialized. Anomalies begin in 90s.";
```

---

### 3.2 `camera_rig.sqf` — 双通道摄像机控制

```sqf
// =============================================
//  camera_rig.sqf — 无人机双摄像机绑定
//  支持：下视（Nadir）/ 斜视（Oblique 35°）
// =============================================

// 等待无人机就位
waitUntil {!isNull UAV_drone};

// ---- 可见光摄像机 ----
_camVisible = "camera" camCreate (getPosASL UAV_drone);
_camVisible attachTo [UAV_drone, [0, 0, -0.2]];  // 挂载在无人机机腹
_camVisible camSetDir [0, 0, -1];                  // 默认朝下（Nadir）
_camVisible cameraEffect ["internal", "BACK"];
_camVisible camCommit 0;

// ---- 热成像摄像机（切换TI模式）----
// 切换主摄像机为 TI 模式时：
// 可见光模式：TI 关闭
// 热成像模式：setCamUseTi 1（White-Hot）

// ---- 镜头控制函数 ----
// 调用示例：
//   [_camVisible, "nadir"]   call fnc_setCamAngle;  // 下视
//   [_camVisible, "oblique"] call fnc_setCamAngle;  // 斜视 35°
fnc_setCamAngle = {
  params ["_cam", "_mode"];
  switch (_mode) do {
    case "nadir": {
      _cam camSetDir [0, 0, -1];  // 垂直下视
      _cam camSetFov 0.5;         // 视场角（等效约 50mm 焦距）
    };
    case "oblique": {
      // 斜视（35° 前斜视）
      _cam camSetDir [0.6, 0, -0.8];
      _cam camSetFov 0.4;
    };
  };
  _cam camCommit 0.5;
};

// 默认：下视模式
[_camVisible, "nadir"] call fnc_setCamAngle;

// ---- TI 模式切换 ----
// 在脚本或触发器中调用：
//   _camVisible setCamUseTi true;   // 开启 White-Hot 热成像
//   _camVisible setCamUseTi false;  // 关闭，切回可见光
//   _camVisible camCommit 0;

// ---- 自动循环录制（每 60s 自动切换 TI 状态）----
[] spawn {
  private _ti_state = false;
  while {true} do {
    sleep 60;
    _ti_state = !_ti_state;
    _camVisible setCamUseTi _ti_state;
    _camVisible camCommit 0;
    if (_ti_state) then {
      hint "-- 切换至热成像模式 --";
    } else {
      hint "-- 切换至可见光模式 --";
    };
  };
};
```

---

### 3.3 `anomaly_sequence.sqf` — 异常注入序列

```sqf
// =============================================
//  anomaly_sequence.sqf — 异常序列调度器
//  按时间表依次触发 6 类异常
// =============================================

hint "Anomaly Sequence Started";

// === 异常 1：Tank 凭空出现（0s 后）===
[] spawn {
  private _ghost = Tank_3;
  _ghost hideObjectGlobal true;   // 先隐藏
  hint "[ANOMALY 1] Target disappeared.";
  sleep 8;
  _ghost hideObjectGlobal false;  // 瞬间出现
  hint "[ANOMALY 1] Target appeared from nowhere!";
};

// === 异常 2：热源瞬移（20s 后）===
[20] spawn {
  params ["_delay"];
  sleep _delay;
  private _mover = APC_1;
  private _pos1 = getPosASL _mover;
  private _pos2 = _pos1 vectorAdd [50, 0, 0];  // 东移 50m
  hint "[ANOMALY 2] Heat source teleporting...";
  _mover setPosASL _pos2;
  _mover setVelocity [0, 0, 0];
  sleep 3;
  _mover setPosASL _pos1;  // 瞬回原位
  _mover setVelocity [0, 0, 0];
  hint "[ANOMALY 2] Target snapped back.";
};

// === 异常 3：人员瞬移（40s 后）===
[40] spawn {
  params ["_delay"];
  sleep _delay;
  private _units = units group Soldier_1;
  {
    private _curPos = getPosASL _x;
    private _newPos = _curPos vectorAdd [0, -30, 0];  // 南移 30m
    _x setPosASL _newPos;
    _x setVelocity [0, 0, 0];
  } forEach _units;
  hint "[ANOMALY 3] Personnel teleported 30m south.";
};

// === 异常 4：实体尺寸突然变大（60s 后）===
[60] spawn {
  params ["_delay"];
  sleep _delay;
  private _bigObj = Tank_1;
  hint "[ANOMALY 4] Object scaling anomaly!";
  // 瞬间放大到 2.5 倍
  _bigObj setObjectScale 2.5;
  sleep 5;
  // 瞬间缩回
  _bigObj setObjectScale 1.0;
  hint "[ANOMALY 4] Scale normalized.";
};

// === 异常 5：目标凭空消失（80s 后）===
[80] spawn {
  params ["_delay"];
  sleep _delay;
  hint "[ANOMALY 5] Target vanished!";
  Tank_2 hideObjectGlobal true;
  APC_2 hideObjectGlobal true;
  sleep 10;
  Tank_2 hideObjectGlobal false;
  APC_2 hideObjectGlobal false;
  hint "[ANOMALY 5] Targets reappeared.";
};

// === 异常 6：热源瞬间移动（发动机/发电机）（100s 后）===
[100] spawn {
  params ["_delay"];
  sleep _delay;
  private _gen = Generator_1;
  private _origPos = getPosASL _gen;
  hint "[ANOMALY 6] Heat source displacement detected!";
  // 快速位移序列（模拟热源跳动）
  for "_i" from 1 to 5 do {
    private _offset = [random 20 - 10, random 20 - 10, 0];
    _gen setPosASL (_origPos vectorAdd _offset);
    sleep 0.3;
  };
  _gen setPosASL _origPos;  // 回到原始位置
  hint "[ANOMALY 6] Heat source stabilized.";
};
```

---

### 3.4 `auto_record.sqf` — 自动录制调度

```sqf
// =============================================
//  auto_record.sqf — 自动场景切换与录制调度
//  配合 OBS 录制键位（Start/Stop Hotkey）
// =============================================

// OBS 热键配置：
//   开始录制 = Alt+F9
//   停止录制 = Alt+F10
//   截图 = F12

// 场景切换序列（每个子场景 8 分钟）
_scenes = [
  ["下视_可见光",    "nadir",    false],  // 下视，可见光
  ["下视_热成像",    "nadir",    true],   // 下视，热成像
  ["斜视_可见光",    "oblique",  false],  // 斜视，可见光
  ["斜视_热成像",    "oblique",  true]    // 斜视，热成像
];

{
  private _sceneData = _x;
  private _name = _sceneData select 0;
  private _angle = _sceneData select 1;
  private _ti = _sceneData select 2;

  hint format ["Starting Scene: %1", _name];
  [_camVisible, _angle] call fnc_setCamAngle;
  _camVisible setCamUseTi _ti;
  _camVisible camCommit 0;

  // 提示 OBS 开始录制（操作员手动按 Alt+F9）
  hint format ["[REC START] Scene: %1\n请按 Alt+F9 开始录制", _name];
  sleep 3;

  // 场景持续录制 480s（8分钟）
  sleep 480;

  // 提示停止录制
  hint format ["[REC STOP] Scene: %1\n请按 Alt+F10 停止录制", _name];
  sleep 5;

} forEach _scenes;

hint "All scenes recorded. Mission complete.";
```

---

## 四、录制执行流程

### 4.1 准备阶段（任务启动前）

```
1. 启动 Arma 3，加载 MODs（RHS + ACE3 + CUP Terrains）
2. 启动 OBS Studio，配置好输出路径和参数
3. 将 Arma 3 分辨率设为 1920×1080，帧率上限 30fps
4. 进入任务编辑器，加载场景文件
5. 预览一遍，确认所有单位名称与脚本中的变量名匹配
```

### 4.2 正式录制阶段

```
Step 1：进入任务，等待 init.sqf 执行完成（约 5s）
Step 2：等待 camera_rig.sqf 就位（提示"Scene Initialized"出现）
Step 3：按照 auto_record.sqf 的提示，依次开始/停止录制
Step 4：异常序列（anomaly_sequence.sqf）会在 90s 时自动开始
Step 5：每个子场景录制 8 分钟，共 4 个子场景，总录制时长约 35 分钟
Step 6：录制完成后，退出任务，检查 OBS 输出文件
```

### 4.3 输出文件命名规范

```
output/
├── raw/
│   ├── scene_01_nadir_visible.mp4      ← 下视 可见光
│   ├── scene_02_nadir_thermal.mp4      ← 下视 热成像
│   ├── scene_03_oblique_visible.mp4    ← 斜视 可见光
│   └── scene_04_oblique_thermal.mp4   ← 斜视 热成像
├── frames/                             ← 帧提取后
├── annotations/                        ← 标注文件
└── sft_data/                           ← instruction-tuning 训练数据
```

---

## 五、数据采集与后处理

### 5.1 帧提取（FFmpeg）

```bash
# 安装 FFmpeg
# Windows：https://ffmpeg.org/download.html

# 每秒提取 5 帧（5fps 采样）
ffmpeg -i scene_01_nadir_visible.mp4 \
  -vf fps=5 \
  frames/visible_nadir_%04d.png

# 热成像视频同步提取
ffmpeg -i scene_02_nadir_thermal.mp4 \
  -vf fps=5 \
  frames/thermal_nadir_%04d.png

# 提取特定时间段（如异常发生前后 ±5s）
ffmpeg -i scene_02_nadir_thermal.mp4 \
  -ss 00:01:30 -to 00:01:50 \   # 异常 1：t=90s 时，截取 85-95s 片段
  -vf fps=10 \                   # 这段用 10fps（精细采样）
  frames/anomaly_01_%04d.png
```

### 5.2 异常时间戳记录表

| 异常编号 | 类型 | 录制时间戳 | 子场景 | 关键帧范围 |
|---------|------|-----------|--------|-----------|
| A1 | 目标凭空出现 | T+90s | 场景2/4 | frame_450 ~ 490 |
| A2 | 热源瞬移 | T+110s | 场景2/4 | frame_550 ~ 565 |
| A3 | 人员瞬移 | T+130s | 场景1/3 | frame_650 ~ 660 |
| A4 | 尺寸突然变大 | T+150s | 场景1/3 | frame_750 ~ 775 |
| A5 | 目标消失 | T+170s | 场景2/4 | frame_850 ~ 900 |
| A6 | 热源跳动 | T+190s | 场景2/4 | frame_950 ~ 965 |

### 5.3 图像标注（LabelImg / Roboflow）

```
工具：LabelImg（开源）
标注格式：YOLO format（.txt）/ COCO JSON

标注类别：
  0: tank
  1: armored_vehicle
  2: rocket_artillery
  3: generator
  4: tent
  5: bunker
  6: person

异常帧标注额外字段（JSON）：
  "anomaly_type": "teleport" / "scale" / "disappear" / "appear" / "heat_jump"
  "anomaly_severity": "high" / "medium" / "low"
  "confidence_expected": 0.0 ~ 1.0  （期望模型置信度，用于评估指标）
```

### 5.4 生成 Instruction-Tuning 训练数据

```python
# generate_sft.py
import json
import os
from pathlib import Path

# 模板：目标识别
def make_detection_sample(frame_path, targets, is_thermal=False):
    mode = "热红外" if is_thermal else "可见光"
    return {
        "instruction": f"请分析这张{mode}无人机航拍图像，识别图中的军事目标，列出所有可见目标类型及大致位置。",
        "input": f"图像路径：{frame_path}",
        "output": f"图中可识别军事目标：" + "；".join(
            [f"{t['type']}（位于图像{t['region']}区域）" for t in targets]
        )
    }

# 模板：异常检测
def make_anomaly_sample(frame_path, anomaly):
    return {
        "instruction": "请仔细观察这张无人机热红外图像，判断图中是否存在视觉异常。如有异常，请描述异常类型、位置及可能的原因。",
        "input": f"图像路径：{frame_path}",
        "output": f"检测到视觉异常：{anomaly['desc']}。异常类型为{anomaly['type']}，"
                  f"位于图像{anomaly['region']}，该现象在真实场景中不符合物理规律，"
                  f"判定为{'数据伪造信号' if anomaly['severity']=='high' else '传感器故障或图像处理误差'}。"
    }

# 模板：双模态对比
def make_multimodal_sample(vis_path, tir_path, analysis):
    return {
        "instruction": "对比提供的可见光图像与热红外图像（同一场景同一时刻），分析热源目标分布与可见光目标的对应关系，并指出热红外中独有的热源目标。",
        "input": f"可见光图像：{vis_path}\n热红外图像：{tir_path}",
        "output": analysis
    }

# 示例输出
samples = []
samples.append(make_detection_sample(
    "frames/thermal_nadir_0450.png",
    [
        {"type": "坦克", "region": "左上"},
        {"type": "装甲车", "region": "中央"},
        {"type": "发电机（热源）", "region": "右下"}
    ],
    is_thermal=True
))

samples.append(make_anomaly_sample(
    "frames/thermal_nadir_0460.png",
    {
        "desc": "画面左上角坦克目标瞬间消失后重新出现",
        "type": "目标突发出现（Sudden Appearance）",
        "region": "左上",
        "severity": "high"
    }
))

with open("sft_data/arma3_thermal_sft.json", "w", encoding="utf-8") as f:
    json.dump(samples, f, ensure_ascii=False, indent=2)

print(f"Generated {len(samples)} SFT samples.")
```

### 5.5 数据集最终目录结构

```
arma3_dataset/
├── README.md
├── metadata.json                    ← 场景、时间戳、异常类型总表
├── raw_video/
│   ├── scene_01_nadir_visible.mp4
│   ├── scene_02_nadir_thermal.mp4
│   ├── scene_03_oblique_visible.mp4
│   └── scene_04_oblique_thermal.mp4
├── frames/
│   ├── visible_nadir/               ← 约 1,440 帧（5fps × 480s）
│   ├── thermal_nadir/
│   ├── visible_oblique/
│   └── thermal_oblique/
├── annotations/
│   ├── detection_labels/            ← YOLO 格式标注
│   └── anomaly_labels/              ← JSON 格式异常标注
└── sft_data/
    ├── arma3_detection.json         ← 目标识别 SFT 数据
    ├── arma3_anomaly.json           ← 异常检测 SFT 数据
    └── arma3_multimodal.json        ← 双模态对比 SFT 数据
```

---

## 六、任务验证检查清单

**录制前检查：**
- [ ] 所有 Mod 正确加载（RHS + ACE3 + CUP）
- [ ] 编辑器中单位命名与脚本变量一致
- [ ] OBS 已启动，输出路径已设置
- [ ] Arma 3 帧率锁定 30fps，分辨率 1920×1080
- [ ] camera_rig.sqf 中 UAV_drone 名称已对应编辑器单位名

**录制中检查：**
- [ ] 每个子场景开始/结束时手动按 OBS 录制热键
- [ ] 异常时间戳手动记录到表格（备用）
- [ ] 每段录制完成后在 OBS 文件列表确认文件大小 > 0

**录制后检查：**
- [ ] 4 段视频文件均存在且可播放
- [ ] FFmpeg 帧提取正常（无全黑帧或损坏帧）
- [ ] 异常帧目视可见（打开具体帧确认异常存在）

---

## 七、扩展方向

| 扩展项 | 说明 |
|--------|------|
| 增加夜间场景 | Arma 3 日夜循环：`setDate [2025, 6, 1, 22, 30]` |
| 增加天气效果 | `0 setRain 0.5; 0 setFog 0.2;` |
| 增加无人机 6DoF 运动 | 在 camera_rig 中加入随机抖动（模拟气流） |
| 增加多视角同步 | 同时运行多个 camCreate 实例 |
| 增加场景二（地面痕迹） | 使用预制静态贴图面片组合，手工布置在机库/沙地 |
| 接入 Zeus（实时编排） | 使用 Zeus 实时调度目标行为，增加数据多样性 |
