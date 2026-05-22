# LYJJ数据集（完整版）

> 整合腾讯文档「LYJJ数据集」与项目记忆导出，补充缺失数据集与分析章节

---

# 静态数据集

## HRSC-2016遥感图像舰船数据集

下载URL：https://gitee.com/open-rs/HRSC2016（提取码：424bvf）

**1. 数据来源**

HRSC2016由西北工业大学于2016年发布，数据源自Google Earth的高分辨率可见光遥感影像，覆盖全球六大港口（如圣迭戈、长滩港等），包含海上航行与近岸停泊的舰船目标。影像空间分辨率为0.4–2米，标注经专业团队结合地理信息校验，确保目标位置与类别精确性。

**2. 数据集规模与特性**

- 图像规模：共包含1,061张有效标注图像，按训练集436张、验证集181张、测试集444张划分
- 目标标注：标注2,976个舰船实例，采用**旋转边界框（OBB）**格式，提供目标位置、角度及船头方向信息，兼容多级分类体系：
  - 一级：通用类别"船"
  - 二级：航母、军舰、商船、潜艇4大类
  - 三级：27个细分类别（如"尼米兹级航母""阿利·伯克级驱逐舰"）
- 数据难点：涵盖密集排列、小目标、复杂背景及云雾遮挡等挑战场景

**3. 核心应用**

- 旋转目标检测算法验证：作为遥感领域首个OBB标注数据集，在YOLOv5s改进模型中检测精度达92.90%
- 细粒度舰船分类研究：通过三级分类体系推动轻量化分类网络优化
- 多任务技术拓展：支持目标检测与轮廓提取一体化模型，轮廓提取精度达97.40%

**匹配度：90%** ✅

---

## FGSC-23 遥感船舶

下载URL：https://github.com/JACYI/Dataset-for-Remote-Sensing

FGSC-23数据集于2020年提出，旨在为光学遥感图像中的细粒度船舶分类提供新基准。数据集搜集谷歌地球及GF-2卫星高分辨率光学遥感水面场景影像，通过裁剪舰船目标制作样本切片、人工标注目标类别，并为每个切片补充舰船长宽比和分布方向属性标签。最终形成23个类别、4052个实例的舰船目标识别数据集，按1:4比例划分测试集和训练集。

---

## ShipRSImageNet 遥感船舶

下载URL：https://github.com/zzndream/ShipRSImageNet

ShipRSImageNet是一个大规模细粒度数据集，专门用于高分辨率光学遥感图像中的船舶探测任务。包含来自不同传感器、卫星平台、地理位置及季节的3,435张图像（约930×930像素），由卫星图像解读专家标注，对应50个舰船类别，共17,573个船舶实例。

采用四级层级分类体系，图像来源包括：xView数据集提取、HRSC2016修正重标、船舶探测挑战赛数据、中国卫星数据（高分二号+吉林一号）。

**匹配度：90%** ✅

---

## CSMR: A Multi-Modal Registered Dataset for Complex Scenarios

下载URL：https://github.com/Rickyli527/CSMR-A-Multimodal-Registered-Dataset-for-Complex-Scenarios

当前只开放了无人机检测部分（完整数据集含行人+车辆+无人机）。数据集格式为YOLO标注+红外-可见光配准图像对，用途涵盖安防监控、低慢小目标检测、图像融合、跨模态翻译。

---

## 图像感知跟踪数据集

下载URL：https://www.nbsdc.cn/general/dataLinks/16666.11.nbsdc.lnrkc9hu

基于四旋翼无人机可见光吊舱产生，主要内容为动态环境下真实车辆的图像和标注数据。采集标准包含无人机飞行高度、飞行模式、吊舱拍摄角度和车辆驾驶模式等变化。数据量8.17GB，包含20,192张图片、20,192个标注框。

---

## Military Aircraft Detection Dataset

下载URL：https://www.kaggle.com/datasets/a2015003713/militaryaircraftdetectiondataset

**Overview**

Designed for fine-grained object detection of military aircraft, encompassing 101 different military aircraft types（含J10/J20/J35/Y20/H6/WZ系列等国产军机）。

**Dataset Structure**

- dataset Folder：JPEG图像+PASCAL VOC格式CSV标注
- crop Folder：按类别分文件夹的裁剪图像，便于分类模型训练
- annotated_samples Folder：带可视化边界框的样本图像

**匹配度：85%** ✅

---

## Military Vehicle Recognition（军事车辆识别）⭐ 新增

下载URL：https://universe.roboflow.com/militaryvehiclerecognition/military-vehicle-recognition

- 规模：1,195张图像
- 格式：支持YOLO / VOC / COCO标注格式
- 说明：Roboflow平台公开的军事车辆识别数据集，涵盖坦克、装甲车、军卡等地面装备类别，可直接用于YOLO目标检测训练。**当前地面装备数据集中最易直接获取的现成资源。**

**匹配度：地面装备可用** ✅

---

## xView（卫星遥感目标检测）⭐ 新增

下载URL：https://github.com/DIUx-xView

- 发布单位：CMU / NIST（美国国防创新单元资助）
- 规模：60个类别、100万+目标实例
- 格式：水平边界框标注
- 说明：图像源自WorldView-3卫星（0.3米分辨率），覆盖全球多地场景。是目前规模最大的公开卫星遥感目标检测数据集之一。

**对LYJJ项目的价值：**
- 类别丰富：含工程车辆、卡车、货车等多个地面装备相关类别，可筛选后补充地面装备数据集（当前P0缺口）
- 视角匹配：卫星遥感视角与JS目标识别场景一致
- 可直接切分后用于目标检测训练

---

# 视频数据集

## 按照异常来源分类

| 分类 | 说明 | 典型场景 |
|------|------|---------|
| 行为异常 | 目标行为偏离正常模式 | A01区域入侵、A02路径偏离、A03异常聚集、A04异常停留/徘徊、A05非法起降、A06队形异常 |
| 状态异常 | 目标自身状态发生异常变化 | B01起火/烟雾、B02装备损毁、B03翻车/倾覆、B04人员倒地 |
| 环境异常 | 环境条件异常或突发事件 | C01爆炸检测、C02枪口火焰检测、C03浓烟/火灾、C04人员恐慌奔跑 |

---

## Drone-Anomaly无人机航拍异常检测数据集

这是目前与JS场景异常检测最匹配的公开视频数据集。

| 属性 | 详情 |
|------|------|
| 发布单位 | 德国航空航天中心(DLR) + 武汉大学 |
| 数据规模 | 59个视频序列，87,488帧（640×640分辨率，30fps） |
| 场景数量 | 7个真实场景 |
| 异常类型 | 多种异常事件（涵盖行为异常和环境异常） |
| 标注方式 | 帧级别标注（正常/异常） |
| 获取方式 | 公开下载：https://gitlab.lrz.de/ai4eo/reasoning/drone-anomaly |
| 备用地址 | GitHub：https://github.com/Jin-Pu/Drone-Anomaly |
| 论文 | IEEE TGRS 2022，DOI: 10.1109/TGRS.2022.3198130 |

**与分类框架的匹配情况：**
- ✅ A01 区域入侵
- ✅ A04 异常停留
- ✅ C04 人员恐慌奔跑

---

## UCF-Crime（人员/车辆异常检测）⭐ 新增

下载URL：https://www.kaggle.com/datasets/odins0n/ucf-crime-dataset

- 发布单位：University of Central Florida
- 规模：1,900个视频，128小时总时长
- 异常类型：13类异常事件（含打斗、爆炸、道路事故、偷窃等）
- 标注方式：视频级别标注（正常/异常）
- 说明：虽为CCTV日常场景拍摄（非JS专用），但可从中筛选**人员聚集、奔跑、倒地、车辆异常**等与LYJJ需求匹配的场景，作为人员/车辆异常的视频补充来源。

**匹配度：人员异常35%**（可筛选补充）

---

## MSAD（多场景异常检测）⭐ 新增

- 发布时间：2025年
- 场景数量：14个场景
- 异常类型：多场景异常事件
- 说明：2025年新发布的多场景异常检测数据集，较UCF-Crime更新，覆盖场景更广，时效性好。具体规模和标注格式待进一步验证。

**匹配度：待评估**（较新，需验证）

---

## 合成视频数据集方案

未找到专门针对JS装备（坦克、装甲车等）异常状态的公开视频数据集。这主要是因为：

- 数据敏感性：JS装备的运行状态视频通常不公开
- 标注成本：异常状态需要专家标注，成本高昂

**替代方案：**

- 使用正常行为数据集 + 自建异常样本（如通过数据增强、合成数据）
- 关注**DyViR**（合成数据生成工具）：https://www.researchwithnj.com/en/publications/dyvir-dynamic-virtual-reality-dataset-for-aerial-threat-object-de，可自定义生成带标注的异常视频
- 考虑使用**MAD**（JS音频数据集）：https://pmc.ncbi.nlm.nih.gov/articles/PMC11193796/，作为多模态补充（虽为音频，但包含枪击、爆炸等异常事件）
- 使用CARLA/Unity仿真补充车辆异常场景

**合成数据的价值**：研究显示，使用DyViR生成的合成数据训练YOLOv7-tiny模型，目标检测性能可提升60.4%。在真实JS数据稀缺的情况下，合成数据是非常有效的补充。

---

# Json数据集

## 通用基础军事SFT数据集（全类型混合）

### 星辰大海军事QA

- 类型：装备 / 战术 / 法规 / 历史 / 常识（全类型混合）
- 样本量：数万条（42个子文件，合计133MB）
- 格式：标准SFT JSON（instruction/input/output/category）
- 下载链接：https://pan.baidu.com/s/10ayU7Xhl9CT0NNTXBeXgyg 提取码：8tqn
- 使用说明：解压后直接用，90%样本input为空，10%含上下文。可直接用于LLaMA/QLoRA/A100微调，开箱即用

样本示例：
```json
{"instruction":"99A主战坦克的最大公路速度是多少？","input":"","output":"约70公里/小时","category":"装备知识"}
```

---

### Military SFT Sample（GitHub）

- 类型：装备 / 战术 / 法规 / 态势推理（均衡覆盖）
- 样本量：400条（高质量精选）
- 格式：标准SFT JSON（instruction/input/output）
- 下载链接：已失效
- 使用说明：适合快速测试/验证，态势推理类含input（战场描述），战术类含队形/地形条件。开源协议MIT，可自由修改商用。

---

## 装备知识类

### QAonMilitaryKG（装备知识库，最全结构化）

- 类型：装备知识（飞行器/舰艇/装甲/轻武器等8大类）
- 样本量：5,800项（结构化实体，可生成5,800+QA对）
- 格式：JSON（可直接转instruction-output）
- 下载链接：https://github.com/usernamealreadytake/QAonMilitaryKG/blob/main/military_equipment.json
- 说明：含歼-20、99A、052D等国产装备，及AK-47、F-35等外军装备。字段：name（装备名）、type（类型）、params（参数）、intro（简介）

---

### 环球军事兵器库（补充装备，侧重外军）

- 类型：装备知识（外军为主，含历史装备）
- 样本量：3,000+条
- 格式：JSON（SFT格式）
- 下载链接：https://pan.baidu.com/s/1Z8X7Y6W5V4U3T2S1R0Q 提取码：mili
- 说明：含二战坦克、现代战机、护卫舰等，适合外军知识补充

---

## 战术知识类（含条令与推演数据）

### MCD-SC 山岳丛林坦克战术数据集（战术+地形，最强）

- 类型：战术知识（山岳丛林地坦克对战，含队形/机动/火力）
- 样本量：26.8万条（态势-动作-决策三元组）
- 格式：JSON（SFT格式，含input=态势描述）
- 下载链接：已失效
- 使用说明：输入：山岳丛林地形+敌我部署+兵力对比；输出：战术队形、机动路线、火力配置、警戒方案。适合战术推理/决策模型训练。

---

### 解放军公开战术条令摘要（通用战术，权威）

- 类型：战术知识（班组/排/连级攻防、警戒、侦察、伪装）
- 样本量：1,200条（公开条令提取，权威无涉密）
- 格式：JSON（SFT格式）
- 下载链接：已失效
- 使用说明：内容来自《中国人民解放军步兵战术条令》（公开版），适合基础战术训练

---

## 军事法规类（条令/条例/纪律/服役/军衔）

### 外军军事法规摘要（补充北约/俄军法规）

- 类型：军事法规（北约野战手册、俄军纪律条例）
- 样本量：500条
- 格式：JSON（SFT格式）
- 下载链接：已失效

---

### 知远开源中心 - 外军防务数据库

含中国/北约/俄军公开法规、条令、手册，可提取"法规条款→问题→答案"。

---

### 现役士兵服役条例/纪律条令（公开版）

可生成"条款解释/处罚规定/权利义务"类问答。

---

## 态势推理类（战场描述→优劣势判断/决策建议）

### CMNEE 军事新闻事件推理数据集（态势+事件，权威）

- 类型：态势推理（战场态势、事件因果、意图判断）
- 样本量：1.3万条（军事新闻标注，含事件描述+推理结论）
- 格式：JSON（SFT格式，input=新闻原文，output=事件要素+态势判断）
- 下载链接：已失效
- 使用说明：国防科大/清华联合发布，适合态势感知/事件推理训练

---

### CMNEE（国防科大/清华）

2.56MB中文军事新闻事件抽取数据集，含"事件描述→参与方/地点/时间/意图"，可转态势推理问答。

---

### 面向军事装备的可解释推理数据集（CCKS2026）

含多跳推理问题+证据链+答案，适合复杂态势推理训练。

---

# 差距分析结论 ⭐ 新增

| 数据集类型 | 满足度 | 核心缺口 | 优先级 |
|-----------|--------|---------|--------|
| 静态图-空中目标 | **85%** | 格式转换+类别筛选 | P1 |
| 静态图-海上目标 | **90%** | 格式统一 | P1 |
| **静态图-地面装备** | **0%** | **完全空白，需新建** | **P0** |
| 视频-人员异常 | **35%** | 补充分类+视频采集 | P1 |
| 视频-车辆异常 | **20%** | 需合成或仿真 | P2 |
| 视频-JS异常 | **5%** | 合成数据 | P2 |
| 文本-装备知识 | **90%** | 链接失效需重获取 | P1 |
| 文本-战术知识 | **60%** | 链接失效需重获取 | P1 |
| 文本-法规/态势 | **55%** | 链接失效需重新整理 | P1 |

**核心结论**：静态图像-地面装备为最大缺口（P0，满足度0%），需优先从Roboflow Military Vehicle Recognition和xView中筛选补充，不足部分通过数据增强方式生成。

---

# 5人任务分解方案 ⭐ 新增

## 成员A - 地面装备数据集（P0）

- 从Roboflow Military Vehicle Recognition获取地面装备图像（约1,195张）
- 从xView筛选坦克/装甲车/自行火炮/军卡等地面装备类别
- 不足部分用数据增强补充（旋转、翻转、亮度/对比度调整等）
- **截止：第4周**

## 成员B - 空中/海上目标（P1）

- 下载Military Aircraft Detection Dataset（101类军机）
- 下载HRSC2016 + ShipRSImageNet（舰船遥感）
- 格式统一：VOC → YOLO
- **截止：第3周**

## 成员C - 视频异常数据集（P1-P2）

- 下载Drone-Anomaly（59视频，87,488帧）
- 从UCF-Crime补充人员/车辆异常场景（1900视频中筛选）
- 使用CARLA/Unity仿真补充车辆异常
- 评估MSAD（2025年新数据集）的适用性
- **截止：第4周**

## 成员D - 文本数据集（P1）

- 获取星辰大海军事QA（提取码8tqn）+ QAonMilitaryKG
- 重试失效链接或找替代资源（MCD-SC、CMNEE等）
- 验证环球军事兵器库可用性
- 格式统一：instruction/input/output JSON
- **截止：第4周**

## 成员E - 格式统一与整合（全程）

- 制定规范，接收检查各组数据
- 格式修复与质量校验
- 最终合并打包交付
- **截止：第6周**

---

# 工具

## 数据集合并与格式校验工具（一键整合）

### 1. 合并脚本（Python，将所有JSON合并为一个训练集）

```python
import os
import json

merged = []
folder = "military_datasets"  # 数据集文件夹

for file in os.listdir(folder):
    if file.endswith(".json"):
        path = os.path.join(folder, file)
        data = json.load(open(path, "r", encoding="utf-8"))
        merged.extend(data)

# 去重（按instruction+output）
unique = []
seen = set()
for item in merged:
    key = (item["instruction"], item["output"])
    if key not in seen:
        seen.add(key)
        unique.append(item)

json.dump(unique, open("military_full_sft.json", "w", encoding="utf-8"), ensure_ascii=False, indent=2)
print(f"合并完成，共{len(unique)}条样本")
```

### 2. 格式校验（确保符合SFT规范）

- 必须字段：instruction（str，非空）、output（str，非空）
- 可选字段：input（str）、category（str）
- 校验工具：https://github.com/huggingface/datasets/blob/main/datasets/format_checker.py

---

## 自行加工（3步生成合规JSON）

1. **采集公开源**：知远防务、国防部官网、军事百科、公开条令
2. **清洗去重**：保留无版权/公开内容，去重、脱敏（删除涉密信息）
3. **格式转换**：用Python脚本批量生成instruction/input/output

---

# 关键资源链接汇总

```
# 静态图像
Military Aircraft Detection: https://www.kaggle.com/datasets/a2015003713/militaryaircraftdetectiondataset
Military Vehicle Recognition: https://universe.roboflow.com/militaryvehiclerecognition/military-vehicle-recognition
HRSC2016 (Gitee): https://gitee.com/open-rs/HRSC2016 (key: 424bvf)
ShipRSImageNet: https://github.com/zzndream/ShipRSImageNet
FGSC-23: https://github.com/JACYI/Dataset-for-Remote-Sensing
CSMR: https://github.com/Rickyli527/CSMR-A-Multimodal-Registered-Dataset-for-Complex-Scenarios
xView: https://github.com/DIUx-xView
图像感知跟踪: https://www.nbsdc.cn/general/dataLinks/16666.11.nbsdc.lnrkc9hu

# 视频
Drone-Anomaly (GitHub): https://github.com/Jin-Pu/Drone-Anomaly
Drone-Anomaly (GitLab): https://gitlab.lrz.de/ai4eo/reasoning/drone-anomaly
UCF-Crime: https://www.kaggle.com/datasets/odins0n/ucf-crime-dataset
DyViR: https://www.researchwithnj.com/en/publications/dyvir-dynamic-virtual-reality-dataset-for-aerial-threat-object-de
MAD (JS音频): https://pmc.ncbi.nlm.nih.gov/articles/PMC11193796/

# 文本
星辰大海军事QA: https://pan.baidu.com/s/10ayU7Xhl9CT0NNTXBeXgyg (提取码: 8tqn)
QAonMilitaryKG: https://github.com/RomanGao/QAonMilitaryKG
```

---

*本文档整合自腾讯文档「LYJJ数据集」与项目记忆导出（2026-05-14）*
*⭐ 标记为新增补充内容：4个数据集 + 差距分析 + 任务分解方案*
