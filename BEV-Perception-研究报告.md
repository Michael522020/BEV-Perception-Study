# Birds-eye-view-Perception 仓库深度研究报告

> **研究对象**：[OpenDriveLab/Birds-eye-view-Perception](https://github.com/OpenDriveLab/Birds-eye-view-Perception)
> **研究日期**：2026-10-30　|　**仓库快照**：`master` @ `dd329bd`（2025-07-21）
> **关联论文**：*Delving into the Devils of Bird's-Eye-View Perception: A Review, Evaluation and Recipe*，IEEE T-PAMI 2023（arXiv:2209.05324，DOI `10.1109/TPAMI.2023.3333838`）
> **作者单位**：上海人工智能实验室（Shanghai AI Lab）× 商汤研究院（SenseTime Research），联合港大、港中文、北航、华为、上交
> **许可证**：Apache-2.0

---

## 目录

1. [一句话结论](#1-一句话结论)
2. [仓库档案：基本盘数据](#2-仓库档案基本盘数据)
3. [它到底解决什么问题：BEV 感知的四个核心痛点](#3-它到底解决什么问题bev-感知的四个核心痛点)
4. [仓库解剖：三块拼图](#4-仓库解剖三块拼图)
5. [模块一 · 文献综述层（docs/）](#5-模块一--文献综述层docs)
6. [模块二 · BEV Toolbox（bev_toolbox/）](#6-模块二--bev-toolboxbev_toolbox)
7. [模块三 · Waymo Playground（waymo_playground/）](#7-模块三--waymo-playgroundwaymo_playground)
8. [模块四 · BEV 算法家族（nuScenes_playground/）](#8-模块四--bev-算法家族nuscenes_playground)
9. [工程化复现路线图](#9-工程化复现路线图)
10. [学习与研究路径建议](#10-学习与研究路径建议)
11. [优点、局限与风险提示](#11-优点局限与风险提示)
12. [附录 A · 关键源码索引](#附录-a--关键源码索引)
13. [附录 B · 完整参考文献与外部链接](#附录-b--完整参考文献与外部链接)

---

## 1. 一句话结论

**这是一个"综述 + 工具箱 + 算法家族"三合一的 BEV（鸟瞰图）感知开源工程**：它把 2022–2023 年自动驾驶 BEV 感知从"论文各自为战"的状态，收敛成一套可运行、可消融、可迁移的工程基线——上层用 117 篇带官方链接的文献 + 6 张工业/学术路线图梳理领域全景，中层用 `bev_toolbox` 抽象出与框架无关的 BEV 数据增强原语，下层用 `waymo_playground`（BEVFormer++ 配方）和 `nuScenes_playground`（VCD / GAPretrain / FocalDistiller）三套可跑的实现把"配方"和"新算法"真正落地。

它的真正价值不在代码量，而在于**它是这个领域少见的、把"怎么做"和"为什么这么做"放在同一个仓库里的项目**。

---

## 2. 仓库档案：基本盘数据

### 2.1 社区指标（截至 2026-10-30 抓取自 GitHub API）

| 指标 | 数值 | 指标 | 数值 |
|---|---|---|---|
| ⭐ Stars | **1,385** | 🍴 Forks | **122** |
| 👀 Watchers | 1,385 | ❓ Open Issues | 4 |
| 创建时间 | 2022-08-20 | 最后 push | 2025-07-21 |
| 默认分支 | `master` | 仓库体积 | ~25 MB |
| 许可证 | Apache-2.0 | Releases | 仅 `v0.1`（2022-10-14） |
| Topics | autonomous-driving, birds-eye-view, camera-detection, lidar-detection, perception-algorithm | | |

### 2.2 代码体量（实测）

```
939 个文件 · 814 个 .py · 47 个 .md · 18 个 .sh · 147,281 行（py+md+sh）
```

| 子项目 | 文件数 | 代码行数 | 定位 |
|---|---:|---:|---|
| `nuScenes_playground/VCD` | 520 | 85,491 | VCD 官方实现（**内含完整 mmdet3d 分叉**） |
| `waymo_playground/projects` | 123 | 27,668 | mmdet3d 插件（模型/数据集/损失/Hook） |
| `nuScenes_playground/FocalDistiller` | 168 | 22,657 | FD3D 官方实现（内含 mmdet3d 插件 + KD 模块） |
| `waymo_playground/tools` | 42 | 10,308 | 训练/评测/数据转换脚本 |
| `bev_toolbox` | 8 | **567** | ★ 工具箱本体——**全部核心逻辑只有 567 行** |
| `docs` | 13 | 273 | 综述清单 + 环境/数据文档 |

> **值得玩味的一点**：整个仓库 14.7 万行代码里，真正属于"这个项目原创"的只有 `bev_toolbox` 的 567 行和 `waymo_playground` 插件的几千行；VCD（8.5 万行）和 FocalDistiller（2.3 万行）本质上是 fork 了 mmdet3d / BEVFormer 的代码树。**这说明该仓库的组织方式是"论文代码库聚合 + 自研工具箱"，而不是"统一框架下的方法库"。** 理解这一点，才能正确评估它的可维护性（见 §11）。

### 2.3 贡献者与活跃度

| 贡献者 | 提交数 | 身份 |
|---|---:|---|
| ChonghaoSima | 52 | 上海 AI Lab，论文共同一作、OpenDriveLab 负责人 |
| cyberknight2077 | 45 | 上海 AI Lab |
| DevLinyan | 30 | 上海 AI Lab，VCD / GAPretrain 第一作者 |
| cyberknight42 | 18 | 上海 AI Lab |
| Zeng-Jia | 14 | 上海 AI Lab，FocalDistiller 第一作者 |
| hli2020 | 7 | Hongyang Li，上海 AI Lab / OpenDriveLab 负责人 |
| faikit / EloiZ / Petros626 / HenryJunW | 9 | 社区贡献（多为 issue 修复、README 校对） |

**时间线**（取自 `git log` 与 README "What's New"）：

```
2022-08-20  仓库创建
2022-10-13  v0.1 发布：数据增强 + Waymo 数据管线 + BEVFormer baseline + 1/5 Waymo mini 数据集
2023-04-06  GAPretrain / FocalDistiller 论文上线
2023-10-26  VCD 论文上线
2023-11-04  ★ 综述被 IEEE T-PAMI 接收
2023-11-27  FocalDistiller 代码发布
2024-01-06  修正 config 路径 + 百度网盘链接
2025-07-02  README 更新
2025-07-21  更新 bev_lidar.md 文献清单（最后一次提交）
```

> ⚠️ **活跃度判断**：核心代码开发集中在 2022-10 至 2023-12，2024 年后基本只有文档维护。**这是一个"已完结的学术项目"，不是一个持续演进的工程。** 依赖锁定在 torch 1.9.1 / mmcv-full 1.4.1 / mmdet3d 0.18.1 这套 2022 年的 OpenMMLab 栈上（见 `docs/installation.md`），今天想在新卡（如 40 系/50 系）上复现会遇到 CUDA 算子编译问题。

---

## 3. 它到底解决什么问题：BEV 感知的四个核心痛点

要读懂这个仓库，得先理解 BEV 感知在 2022 年面临的四道坎。这也是综述论文（arXiv:2209.05324）的立论框架：

### 痛点 ① 2D 到 3D 的信息坍塌

单目/多目图像天然是透视投影，丢掉了深度这一维。要做 3D 检测，必须做 **视角变换（view transformation）**。综述把视角变换归纳为三条技术路线：

| 路线 | 核心机制 | 代表工作 |
|---|---|---|
| **2D → 3D（Lift-Splat-Shoot 类）** | 逐像素预测**深度分布**，把 2D 特征"抬升"到 3D 体素，再洒到 BEV 平面 | LSS (ECCV'20)、BEVDet、BEVDepth、BEVDet4D、BEVStereo |
| **3D → 2D（查询投影类）** | 预定义 3D 参考点/query，用内外参投影回图像平面采样特征 | BEVFormer (ECCV'22)、DETR3D、PETR/PETRv2、PersFormer |
| **纯网络隐式变换** | 用 MLP/Transformer 直接学映射，不显式建模几何 | ImVoxelNet、OFTNet、GitNet 等 |

> 对应插图：`figs/cam_pipeline_v2.jpg`（相机 BEV 管线）、`figs/lidar_pipeline.jpg`（激光雷达 BEV 管线）。

### 痛点 ② BEV 网格上的真值从哪来

BEV 网格化表示（BEV segmentation / occupancy）需要稠密的 3D 真值，而 Waymo / nuScenes 的标注只有稀疏 3D 框。**这是 BEV 分割任务长期落后于检测任务的根本原因**，也是综述中"数据集"章节配了整张 `figs/dataset_table.jpg` 的原因。

### 痛点 ③ 多源特征怎么融合

Camera / LiDAR / Radar 特征在 BEV 空间对齐后如何融合？综述与 `docs/paper_list/bev_fusion.md` 给出了从早期 MVX-Net、PointPainting、CLOCs 到 Transformer 化方案（TransFusion、DeepInteraction、BEVFusion）的完整谱系。

### 痛点 ④ 换个传感器配置还能用吗

6 相机 vs 5 相机 vs 11 相机，间距、FOV 全变。**"参数无关的泛化"** 是综述点名的未来方向。

### 这个仓库的回应方式

```
痛点①  → docs/paper_list/bev_camera.md / bev_lidar.md / bev_fusion.md 三张谱系表（35+21+5 条）
         + waymo_playground 里三种视角变换路线的实现（BEVFormer 的 3D→2D query 路线）
痛点②  → 综述中的 Occupancy 分支 + 数据集对照表（docs/paper_list/dataset.md）
痛点③  → VCD：用「视觉中心多模态专家」蒸馏出纯相机学生，等价于把融合能力"蒸馏"进相机模型
痛点④  → bev_toolbox 把「视角变换无关」的增强原语抽出来（多视图尺度/翻转 + 内外参同步变换）
```

**一句话**：痛点①②③靠"综述"回答（告诉你有哪些路可走），痛点③的工程答案和痛点④的抽象答案靠"工具箱"和"算法家族"回答。

---

## 4. 仓库解剖：三块拼图

```
Birds-eye-view-Perception/
├── README.md                      # 项目门面：综述 + 工具箱 + 算法家族导航
├── CITATION.cff                   # 引用元数据（引用名 BEVPerception-Survey-Recipe）
├── setup.py                       # pip install bev-toolbox（包名 bev-toolbox）
├── LICENSE                        # Apache-2.0
│
├── docs/                          # 【拼图 1】文献综述层
│   ├── installation.md            #   环境：torch1.9.1 / mmcv1.4.1 / mmdet3d0.18.1
│   ├── data_preparation.md        #   数据：Waymo v1.3 → Kitti 格式转换 + mini 数据集
│   ├── waymo_setup.md             #   训练/评测一键脚本
│   └── paper_list/                #   ★ 7 份分类清单 + 3 张分类索引
│       ├── academia.md            #     学术全景（2 张大图）
│       ├── industry.md            #     工业路线（1 张大图）
│       ├── dataset.md             #     数据集对照（1 张大图）
│       ├── bev_camera.md          #     ★ 35 篇：BEV 相机
│       ├── bev_lidar.md           #     ★ 21 篇：BEV 激光雷达
│       ├── bev_fusion.md          #     ★  5 篇：BEV 融合
│       ├── camera_detection.md    #     14 篇：传统单目 3D 检测
│       ├── lidar_detection.md     #      9 篇：传统点云 3D 检测
│       ├── lidar_segmentation.md  #     18 篇：传统点云分割
│       └── sensor_fusion.md       #     15 篇：传统传感器融合
│
├── bev_toolbox/                   # 【拼图 2】框架无关的 BEV 增强工具箱（567 行）
│   ├── init_toolbox.py            #   init_toolbox_mmdet3d() 入口
│   ├── data_aug/
│   │   ├── functional.py          #   ★ 纯函数实现（numpy + cv2，无框架依赖）
│   │   └── transforms.py          #   ★ 可调用类（多视图尺度 / 多视图水平翻转）
│   └── wrapper_mmdet3d/
│       └── data_aug_mmdet3d.py    #   mmdet3d PIPELINES 注册包装
│
├── waymo_playground/              # 【拼图 3a】Waymo 上的 BEVFormer++ 配方
│   ├── projects/
│   │   ├── configs/               #   配置：baseline / ms_flip / 各种 schedule
│   │   └── mmdet3d_plugin/        #   ★ 插件：模型、数据集、损失、Hook、注意力
│   └── tools/                     #   train_video.py / test_video.py / create_data.py
│
├── nuScenes_playground/           # 【拼图 3b】BEV 算法家族
│   ├── GAPretrain.md              #   几何感知预训练（仅文档，代码未释出）
│   ├── VCD/                       #   ★ VCD 官方实现（含 mmdet3d 分叉 + distiller/）
│   └── FocalDistiller/            #   ★ FD3D 官方实现（含 kd/ 蒸馏模块）
│
├── example/                       # 5 路相机样例图 + 坐标系说明
└── figs/                          # 10 张配图（总览/管线/消融/数据集/工业路线）
```

---

## 5. 模块一 · 文献综述层（`docs/`）

### 5.1 设计思路：图 + 清单的双轨制

这个仓库的文献部分做了一件很聪明的事——**把"需要看图才能理解的关系"和"需要逐条检索的文献"分开承载**：

- **图（`figs/*.jpg`，共 10 张）承载非结构化知识**
  - `general_overview.png` — BEV 感知总览：按输入模态三分（Camera / LiDAR / Fusion），并展开下层的检测/分割/占据三大基础任务
  - `cam_pipeline_v2.jpg` / `lidar_pipeline.jpg` — 两条主流管线的数据流
  - `important_methods.jpg` — 近年重要方法脉络
  - `important_methods_leaderboard.jpg` — 重要方法在不同设置/榜单上的性能
  - `dataset_table.jpg` — 数据集对照
  - `industrial_pipeline.jpg` — **工业界路线图**（Tesla / 地平线 / 毫末等量产方案）
  - `cam_ablation.jpg` / `lidar_ablation.jpg` — 相机/雷达消融实验
  - `waymo_coordinate.png` — Waymo 自车坐标系定义

- **清单（`docs/paper_list/*.md`）承载可检索条目**
  每条统一格式：`标题 / [paper] / [project] / [supplemental] / 会议年份 / 别名`，共 **117 条**：

  | 清单 | 条目数 | 说明 |
  |---|---:|---|
  | `bev_camera.md` | 35 | LSS、BEVDet 系列、BEVDepth、DSGN、LIGA-Stereo、BEVFormer、PETR/PETRv2、DETR3D、LaRa、HDMapNet、BEVSegFormer… |
  | `bev_lidar.md` | 21 | VoxelNet、SECOND、CenterPoint、PV-RCNN/++、SA-SSD、Voxel R-CNN、SST、AFDetV2、PointPillars、PIXOR… |
  | `bev_fusion.md` | 5 | UVTR、MVFuseNet、UniFormer、BEVFusion(MIT)、BEVFusion(ADLab) |
  | `lidar_segmentation.md` | 18 | PointNet/++、SpiderCNN、DGCNN、KPConv、RandLA-Net、PolarNet、Cylinder3D、SPVConv、2DPASS… |
  | `sensor_fusion.md` | 15 | MVX-Net、MMF、ContFuse、AutoAlignV2、DeepFusion、TransFusion、DeepInteraction、PointPainting… |
  | `camera_detection.md` | 14 | Mono3D、Deep3DBox、3D-RCNN、CenterNet、Pseudo-LiDAR、MonoPSR、OFTNet、CaDDN、FCOS3D、PGD… |
  | `lidar_detection.md` | 9 | VoteNet、PointRCNN、Part-A²、H3DNet、Pointformer、BRNet、Group-Free、RBGNet、3DSSD… |
  | **合计** | **117** | |

### 5.2 评价

**优点**：分类维度切得干净（模态 × 任务 × 传统/现代），每条都带官方代码链接，是**可以直接当书签栏用**的导航。

**局限**：
1. **停更于 2025-07**，2023 年后的重要工作（如 BEVDepth 之后的 BEVFormer v2、StreamPETR、SparseBEV、FB-BEV、UniAD、BEVWorld、GaussianLSS 等）全部缺席。
2. `academia.md` / `industry.md` / `dataset.md` **只是图片**，没有文字索引或链接表——图片一旦看不全，信息就丢了。
3. `bev_fusion.md` 只有 5 条，相对 camera(35) / lidar(21) 明显偏轻，而这恰恰是当前工业界最热的方向。
4. 部分条目有笔误（如 `lidar_detection.md` 里 Group-Free 的 paper 链接写成相对路径 `_Object_Detection_via_Transformers_ICCV_2021_paper.pdf`，点不开）。
5. **综述与仓库内容重复度低**：综述讲的是 100+ 篇论文，但仓库只实现了 BEVFormer 一条主线的变体，"survey" 和 "recipe" 的耦合度不高。

---

## 6. 模块二 · BEV Toolbox（`bev_toolbox/`）

### 6.1 定位

一个**只依赖 numpy + opencv** 的独立小包（`pip install bev-toolbox`），把 BEV 感知中"与具体框架无关"的数据增强逻辑抽出来，同时提供 mmdet3d 的即插即用包装。全部逻辑 567 行，代码质量相当高——注释完整、类型标注齐全、docstring 规范。

### 6.2 核心 API

```python
from bev_toolbox.data_aug import RandomScaleImageMultiViewImage, RandomHorizontalFlipMultiViewImage
```

#### ① `RandomScaleImageMultiViewImage(scales=[0.5, 1.0, 1.5])`

多视图同步随机缩放。关键在于**缩放后必须同步更新相机内参和 lidar2img 矩阵**：

```python
# bev_toolbox/data_aug/functional.py:36
scale_factor = np.eye(4)
scale_factor[0, 0] *= rand_scale
scale_factor[1, 1] *= rand_scale
imgs_new        = [cv2.resize(img, (x_size[i], y_size[i]), ...) for i, img in enumerate(imgs)]
cam_intrinsics_new = [scale_factor @ K for K in cam_intrinsics]   # K ← S·K
lidar2img_new      = [scale_factor @ l2i for l2i in lidar2img]      # l2i ← S·l2i
```

> **为什么重要**：多相机是 BEV 感知的标配，但**多视图随机缩放在公开实现里几乎没人做**——因为它要求"所有相机用同一个 scale"（否则相机之间的相对几何会失真，BEV 拼接出现错位），且内外参要同步改。这个实现把这件事做对了，而且 5 路相机共用一个随机 scale。

#### ② `RandomHorizontalFlipMultiViewImage(flip_ratio=0.5, dataset='waymo')`

多视图水平翻转，是**这个工具箱最有技术含量的部分**。它要同时处理 4 类数据：

| 对象 | Waymo 坐标系 | nuScenes 坐标系 | 源码 |
|---|---|---|---|
| 图像 | `np.flip(img, axis=1)` | 同左 | `functional.py:77` |
| 3D 框 | `y → -y`，`yaw → -yaw + π` | `x → -x`，`yaw → -yaw` | `functional.py:101` |
| 相机内外参 | 外参左乘 `diag(1,-1,1,1)`；`fx → -fx`，`cx → w - cx` | 外参左乘 `diag(-1,1,1,1)` | `functional.py:118` |
| CAN bus | `canbus[-1] → -canbus[-1]` | 同左 | `functional.py:160` |

> **坐标系差异的细节**（这是最容易踩坑的地方）：
> - **Waymo**：自车坐标系 x 轴朝前、y 轴朝左。绕垂直轴翻转 180° 后，一个中心在 (x, y)、朝向 yaw 的框，变为中心 (x, −y)、朝向 **−yaw + π**（因为翻转把朝向也反了 180°）。
> - **nuScenes**：x 轴朝右、y 轴朝前。翻转轴不同，因此是 `x → −x`，朝向变为 **−yaw**（不补 π，因为翻转轴就是物体的纵轴）。
>
> `docs/paper_list/` 里没有这篇，但 `example/example.md` + `figs/waymo_coordinate.png` 给出了 Waymo 坐标系定义，README 也贴了两个数据集的官方坐标系链接。

### 6.3 mmdet3d 包装层

`bev_toolbox/wrapper_mmdet3d/data_aug_mmdet3d.py` 用 `@PIPELINES.register_module()` 把两个类注册进 mmdet3d 的 pipeline 体系，用法：

```python
# waymo_playground/tools/train_video.py
from bev_toolbox.init_toolbox import init_toolbox_mmdet3d
init_toolbox_mmdet3d()

# 配置文件里直接写
train_pipeline = [
    dict(type='RandomHorizontalFlipMultiViewImage', dataset='waymo'),
    dict(type='RandomScaleImageMultiViewImage', scales=[0.9, 1.0, 1.1]),
]
```

> ⚠️ **实现瑕疵**（源码可见，非笔误）：
> 1. `RandomScaleImageMultiViewImage.__call__` 里 `results['cam_intrinsic']` 的更新被注释掉了（`# results['cam_intrinsic'] = cam_intrinsics_new`），只更新了 `lidar2img`。若下游模块直接读 `cam_intrinsic` 会拿到旧值——实践中因为都从 `lidar2img` 推算所以没暴露。
> 2. `RandomHorizontalFlipMultiViewImage.__call__` 里同样**没有回写 `cam_intrinsic`**，只更新了 `lidar2cam` 和 `lidar2img`。
> 3. `horizontal_flip_cam_params` 对 `cam_intrinsic` 是**原地修改**（`cam_intrinsic[0,0] = -cam_intrinsic[0,0]`），会污染调用方的原始数组。

### 6.4 工具箱没做的事

`bev_toolbox/README.md` 的 "Ongoing Features" 勾选框很坦诚地列出了未完成项：

- [ ] Grid Mask（实际在 `waymo_playground` 的 detector 里实现了，未进 toolbox）
- [ ] Test-time Augmentation
- [ ] Weighted Box Fusion
- [ ] Two-stage Ensemble
- [ ] BEV Encoder / Loss Family 的抽象
- [ ] BEV 可视化
- [ ] 完整的 API 文档
- detectron2 适配（"计划中"，未实现）

---

## 7. 模块三 · Waymo Playground（`waymo_playground/`）

这是整个仓库**算法含量最高、最值得精读**的部分。它是 BEVFormer 在 **Waymo Open Dataset** 上的工程化复现 + 系统性增强，产出了综述论文中的 **BEVFormer++**。

### 7.1 为什么要做 Waymo 而不是 nuScenes？

- **Waymo 是 2022 年 Waymo Open Dataset Challenge 的主战场**，BEVFormer++ 在 **3D Camera-Only Detection 赛道拿到第 1 名**（最终分 56.2 LET-mAPL）。这是整个 toolbox 存在的意义。
- Waymo 的传感器配置（1 前 + 4 侧 + 1 后，共 6 个，实际用 5 路）与 nuScenes 差异大，可以验证"配方"是否可迁移。
- 提供了 **1/5 数据量的 Waymo mini（Kitti 格式）**，让没有完整算力的人也能跑通全流程——这是"playground"一词的由来。

### 7.2 关键设计：自车运动补偿（ego-motion compensation）

`waymo_playground/projects/mmdet3d_plugin/models/modules/bev_transformer.py` 是整个 playground 的算法核心。BEVFormer 的时序融合需要把上一帧 BEV 特征对齐到当前帧，这个仓库实现了三种对齐手段：

**(a) 旋转补偿 `rotate_prev_bev='waymo_rotate'`**（`bev_transformer.py:283`）

```python
prev_bev = prev_bev.reshape(bev_h, bev_w, -1).permute(2, 0, 1)
prev_bev = rotate(prev_bev, rotation_angle, center=[70, 150])   # 绕 BEV 网格中心旋转
prev_bev = prev_bev.permute(1, 2, 0).reshape(bev_h * bev_w, num_prev_bev, -1)
```

注意 `center=[70, 150]` 是**硬编码**的 BEV 网格中心（对应 `bev_h=300, bev_w=220` 和 `point_cloud_range=[-35,-75,-3,75,75,4]` 的配置），换配置必须同步改这个参数——**这是一处明显的技术债**。

**(b) 平移补偿 `use_shift='waymo_shift'`**（`bev_transformer.py:266`）

从 CAN bus 读取自车位移 `(delta_x, delta_y)` 和航向角，转成 BEV 网格单位的位移量：

```python
delta_x = can_bus[0]; delta_y = can_bus[1]
ego_angle = can_bus[-2] / np.pi * 180
translation_angle = np.arctan2(delta_y, delta_x) / np.pi * 180
bev_angle = ego_angle - translation_angle
shift_y = translation_length * np.sin(bev_angle/180*np.pi) / grid_length_y / bev_h
shift_x = translation_length * np.cos(bev_angle/180*np.pi) / grid_length_x / bev_w
```

**(c) CAN bus 条件化 `use_can_bus=True`**（`bev_transformer.py:306`）

把 18 维 CAN bus 向量经 MLP 编码后**加到 BEV embedding 上**，让网络显式感知自车状态（速度、转向角等）：

```python
self.can_bus_mlp = nn.Sequential(
    nn.Linear(18, self.embed_dims // 2), nn.ReLU(inplace=True),
    nn.Linear(self.embed_dims // 2, self.embed_dims), nn.ReLU(inplace=True))
...
can_bus = self.can_bus_mlp(can_bus)
bev_embed = bev_embed + can_bus * self.use_can_bus
```

> **这三点合起来，是 BEVFormer 原始实现没有的**，属于本仓库对时序融合的实质性改进，也解释了为什么在 Waymo 上能拿到第一名。

### 7.3 配方清单与消融结果

`bev_toolbox/README.md` 给出了**在 Waymo mini（1/5 数据）上、用 R101+DCN 骨干、DETR 头**的逐项消融（LET-mAPL / LET-mAPH / Car 的 L1-mAPH）：

| # | Trick | LET-mAPL | LET-mAPH | L1/mAPH (Car) | 相对基线 |
|---|---|---:|---:|---:|---|
| 0 | **Baseline** | 34.9 | 46.3 | 25.5 | — |
| 1 | Multi-scale resize + Flip | 35.6 | 46.9 | **26.8** | +0.7 / +0.6 / **+1.3** |
| 2 | Conv offset in TSA | 35.9 | 48.1 | 25.6 | +1.0 / +1.8 / +0.1 |
| 3 | Deformable view encoder | **36.1** | 48.1 | 25.9 | +1.2 / +1.8 / +0.4 |
| 4 | Corner pooling | 35.6 | 46.9 | 26.0 | +0.7 / +0.6 / +0.5 |
| 5 | 2d auxiliary loss | 35.3 | 47.4 | 24.6 | **−0.6**（分类指标反而掉了） |
| 6 | 2d auxiliary loss + 可学习 loss 权重 | **36.2** | 48.1 | 25.4 | **+1.3** / +1.8 |
| 7 | Smooth L1 loss | — | — | 26.2 | Car 精度 +0.7 |
| 8 | Label smoothing | 36.0 | 46.7 | — | +1.1 / +0.4 |
| 9 | 2× BEV scale | — | — | 25.5 | 持平 |
| 10 | Sync BN | — | — | 25.5 | 持平 |
| 11 | EMA | — | — | 25.6 | +0.1 |

**从这张表能读出的真实结论**（比表本身更有价值）：

1. **数据增强是性价比最高的单项**（+0.7 mAPL 且 Car 精度 +1.3），而它在原始 BEVFormer 里是缺的。
2. **可学习 loss 权重比裸 2D 辅助损失更重要**：单独加 2D 辅助损失会掉 0.6 mAP，加上可学习权重后才变成 +1.3。**这说明"辅助损失"这类技巧必须配权重调度，否则负收益。**
3. **大部分 trick 的增益在 1 个点以内**——单点 trick 收益有限，论文的 56.2 LET-mAPL 靠的是**组合 + 多检测头集成 + LET-IoU NMS**。
4. **2× BEV scale 和 Sync BN 在这份数据上完全没动**（状态是 ☐ TODO，数字留空），说明它们要么没验证过要么收益不可测。

> ⚠️ **数据一致性小问题**：`bev_toolbox/README.md` 的 Baseline 是 34.9 / 46.3，而 `waymo_playground/projects/configs/bevformer/data_aug/README.md` 的 Baseline 是 34.6 / 46.1。同一份表格两个版本，且 data_aug/README 里的 "Multi-scale resize, Flip" 行 mAPH 是空缺。README 自己也注明："*results were run on data with png format. We are revalidating these results on the data with jpg format*"。**这些数字属于未最终校验的中间态，引用需谨慎。**

### 7.4 代码结构：三类"配方"的实现位置

| Trick 类型 | 实现文件 | 说明 |
|---|---|---|
| **数据增强** | `bev_toolbox/data_aug/`（已抽成包） | 翻转、尺度、Grid Mask（在 detector 内） |
| **BEV 编码器** | `models/modules/encoderV2.py`、`models/attns/` | 6 个注意力实现：V2/V4 多尺度可变形注意力、3D 版、BEV 交叉可变形注意力、时序自注意力、空间交叉注意力 |
| **检测头** | `models/dense_heads/` | 5 个头：`bevformer_detr_head`、`bevformer_detr_head2`、`bevformer_centerpoint_head`、`bevformer_anchor_head`（FreeAnchor）、`bevformer_detr_head_corner_pooling` |
| **损失** | `models/loss/` | `cwd.py`（Channel-wise Divergence，用于蒸馏）、`dice_loss.py`、`fl_loss_w_lable_smoothing.py`（Focal Loss + 标签平滑） |
| **训练技巧** | `models/hooks/` | `sync_norm.py`（CustomSyncNormHook）、`hooks.py`（BaseEMAHook + 指数/线性动量变体） |
| **后处理** | `core/bbox/coders/` | `nms_free_coder.py`（DETR 式无 NMS 解码）、`nms_coder.py` |
| **匹配** | `core/bbox/assigners/hungarian_assigner_3d.py`、`core/bbox/match_costs/` | 3D 匈牙利匹配 + `BBox3DL1Cost` / `IoUCost` |
| **优化器** | `models/opt/adamw.py` | 自定义 AdamW |

**Corner Pooling 的实现**（`bevformer_detr_head_corner_pooling.py`）值得单独一提，它把 CornerNet 的四角池化思想搬到了 BEV 特征图上：

```python
class MyCornerPool(BaseModule):
    def __init__(self, in_channels, ...):
        self.top_pool = CornerPool('top'); self.left_pool = CornerPool('left')
        self.bottom_pool = CornerPool('bottom'); self.right_pool = CornerPool('right')
        self.aftpool_conv = ConvModule(in_channels*2, in_channels, 3, padding=1, ...)

    def forward(self, x):
        top_feat, left_feat = self.top_pool(x), self.left_pool(x)
        bottom_feat, right_feat = self.bottom_pool(x), self.right_pool(x)
        return self.aftpool_conv(torch.cat([top_feat + left_feat, bottom_feat + right_feat], dim=1)) + x
```

> 逐层对 BEV 特征图做上下左右四个方向的 Corner Pooling，相邻两方向相加后拼接卷积，残差连接输出。**动机是 BEV 特征图上的目标通常表现为"局部块状极值"，方向性池化能强化这种结构先验。** 这在仓库里是最有"论文味道"的一处独立贡献（消融增益 +0.7 mAPL）。

### 7.5 数据集与评测

- `mmdet3d_plugin/datasets/waymo_dataset.py`、`waymo_datasetV2.py`（V2 为配置默认）、`lidar_waymo.py`、`eval_waymo.py`
- `core/evaluation/kitti2waymo.py` — Kitti 格式 → Waymo 官方评测格式转换
- `core/evaluation/eval_hooks.py`、`lyft_eval.py`、`nuscnes_eval.py`（注意文件名有拼写错误 `nuscnes`）
- 预训练权重：`r101_dcn_fcos3d_pretrain.pth`（来自 BEVFormer 官方 storage release）

---

## 8. 模块四 · BEV 算法家族（`nuScenes_playground/`）

这是仓库的"算法产出"部分，聚焦**知识蒸馏**这条主线——因为 BEV 感知的主流范式是 BEVFormer / BEVDepth 这类**重型 Transformer**，推理成本高，而 LiDAR 模型又贵又需要激光雷达。**"把多模态专家的能力蒸馏进纯相机学生"** 是最划算的落地路径。

| 方法 | 会议 | 教师（Expert） | 学生（Apprentice） | 状态 |
|---|---|---|---|---|
| **VCD** | NeurIPS 2023 | 视觉中心多模态检测器 | 纯相机检测器 | ✅ 代码 + 权重（online） |
| **GAPretrain** | arXiv 2023 | 仅 LiDAR 检测器 | 纯相机检测器 | ⚠️ **仅文档，代码未释出** |
| **FocalDistiller (FD3D)** | CVPR 2023 | 相机检测器 | 相机检测器 | ✅ 代码（部分），无权重 |

### 8.1 VCD — Leveraging Vision-Centric Multi-Modal Expertise（NeurIPS 2023）

**核心思想**：训练一个"视觉中心的多模态专家"（以相机分支为主干、融合 LiDAR 特征），然后把它的知识蒸馏给一个轻量的纯相机学生（BEVDet4D-R50）。

**代码结构**（`nuScenes_playground/VCD/`）：

```
VCD/
├── setup.py                    # 注册 bev_pool_v2_ext CUDA 算子
├── configs/
│   ├── vcd/                    # vcd-convnext-base.py（教师）/ vcd-r50.py（学生）
│   │                           # MotionDistiller_convnext-r50-4motion.py（蒸馏）
│   │                           # Combine_convnext-r50.py
│   └── <25 个模型配置>          # centerpoint / second / pointpillars / 3dssd / pgd …
├── mmdet3d/                    # ★ mmdet3d 的完整分叉（含 CUDA 算子源码）
│   ├── distiller/              # ★ 本仓库新增
│   │   ├── builder.py          #   DISTILLER / DISTILL_LOSSES 注册表
│   │   ├── distillers/
│   │   │   ├── motion_distiller.py    # ★ MotionDistiller（运动感知蒸馏）
│   │   │   └── combine_distiller.py   #   CombineDistiller（组合蒸馏）
│   │   └── losses/
│   │       ├── klloss.py, l1loss.py, l2loss.py, bceloss.py,
│   │       ├── maskedbceloss.py, norml2loss.py, cwd.py
│   │   └── ...
│   ├── models/detectors/vcd.py       # ★ class VCD(CenterPoint)
│   └── ops/bev_pool_v2/              # BEVPoolv2 CUDA 算子
└── tools/                      # dist_train.sh / dist_test.sh / dist_distill.sh / dist_test_stu.sh
```

**`MotionDistiller` 的设计**（`distillers/motion_distiller.py`）值得拆开看：

- **双模型结构**：内部同时持有 `teacher`（冻结，`requires_grad=False`，`eval()` 模式）和 `student`。
- **多位置蒸馏**：可分别对 **PV（图像空间）**、**BEV（鸟瞰空间）**、**Depth（深度分布）** 三处开蒸馏通道，每路都有独立的 loss 集合：

```python
distill_on_PV    = False   # 配置里关掉了
distill_on_BEV   = True    # ★ 主战场
distill_on_depth = False
channel_PV = 64;  channel_BEV = 256;  channel_depth = 118
feats_align_norm = 'bn'    # 用 BN 对齐师生特征分布
```

- **通道数对齐**：师生 BEV 特征通道不同（ConvNeXt-B 教师 vs R50 学生）时，用 `_make_deconv_layers()` 构造反卷积适配层把学生通道映射到教师通道。
- **动态物体处理**：`single_adj_objects(inputs, starts, ends)` + `prepare_inputs(inputs)` 处理"相邻帧之间的运动物体"——这是 VCD 论文的 "vision-centric multi-modal" 关键：教师能"看到"LiDAR 遮挡不住的部分，学生需要从时序里补上。

**配置示例**（`configs/vcd/MotionDistiller_convnext-r50-4motion.py`）：

```python
teacher_cfg = 'configs/vcd/vcd-convnext-base.py'   # ConvNeXt-B 多模态专家
student_cfg = 'configs/vcd/vcd-r50.py'              # 纯相机 R50 学生
distill_on_BEV = True
feats_align_norm = 'bn'
```

**官方结果**（`VCD/README.md`）：

| 方法 | Backbone | 输入尺寸 | 帧数 | mAP | NDS | 备注 |
|---|---|---|---|---:|---:|---|
| Baseline | R50 | 256×704 | 8+1 | 0.389 | 0.493 | 线上复现 |
| **VCD-A** | R50 | 256×704 | 8+1 | **0.410** | **0.518** | +2.1 mAP / +2.5 NDS |
| Baseline\* | R50 | 256×704 | 8+1 | 0.418 | 0.542 | 离线结果，BEV 特征 256×256 |
| **VCD-A\*** | R50 | 256×704 | 8+1 | **0.446** | **0.566** | **+2.8 mAP / +2.4 NDS** |
| VCD-E | ConvNeXt-B | 768×1408 | 8+1 | 0.677 | 0.711 | 离线强教师 |

> **离线 vs 线上**是 BEV 领域的重要区分：线上（online）指推理时用摄像头序列做时序推理；离线（offline）指推理时可用未来的帧。VCD 的离线代码**至今仍是 TODO**（README 里明确标注 `[ ] VCD offline code`）。

**评分的跨实现比较**（论文 v4 口径）：BEVFormer base (R101-DCN, 900×1600) 在 nuScenes test 上 **51.7 NDS / 41.6 mAP**；VCD-E 教师（ConvNeXt-B）离线可达 **67.7 mAP / 71.1 NDS**；VCD 蒸馏出的 R50 学生离线 **44.6 mAP / 56.6 NDS**。

### 8.2 FocalDistiller / FD3D（CVPR 2023）

**核心思想**：在知识蒸馏中，"把全局特征整体对齐"是错的——3D 几何推理天然产生**噪声和混淆区域**，整体蒸馏会把噪声也传给学生。FD3D 的做法是：

1. 用一组 **query 去定位实例级区域**（instance-level areas），生成 **masked feature**，强化这些区域的特征表达；
2. 再用这些 query **搜索细粒度的代表性位置**（representative fine-grained positions）做**精炼蒸馏**。

**代码结构**（`nuScenes_playground/FocalDistiller/`）：

```
FocalDistiller/
├── projects/mmdet3d_plugin/
│   ├── bevformer/
│   │   ├── dense_heads/bevformer_distiller.py    # ★ BEVFormerDistiller(DETRHead)
│   │   └── distill_branch/
│   │       ├── transformerDistiller.py           #   Transformer 层蒸馏
│   │       └── decoderDistiller.py               #   Decoder 层蒸馏
│   └── kd/distillers/heatmap_detection_distiller.py  # ★ HeatmapDetectionDistiller
├── projects/configs/
│   ├── bevformer/bevformer_{base,small}.py
│   └── distiller/base_distill_small_with_pv-cwd_bev-l2-heatmap.py  # ★
└── tools/distillation.py, dist_distill.sh, testStu.py
```

**`HeatmapDetectionDistiller`** 的关键机制是**用 forward hook 捕获教师/学生的中间特征**：

```python
def regitster_hooks(student_module, teacher_module):
    def hook_teacher_forward(module, input, output): ...
    def hook_student_forward(module, input, output): ...
```

这样可以在**不改动主干网络结构**的前提下，指定任意模块对做蒸馏——工程上比侵入式改造优雅得多。

**官方结果**：

| 方法 | Backbone | 输入 | BEV 分辨率 | NDS | mAP | GFLOPS | FPS |
|---|---|---|---|---:|---:|---:|---:|
| BEVFormer-Base (T) | R101-DCN | 900×1600 | 200×200 | 51.76 | 41.66 | 1323.41 | 1.8 |
| BEVFormer-Small (S) | R101-DCN | 450×800 | 100×100 | 46.83 | 35.09 | 416.46 | 5.9 |
| **+ HeatmapDistiller** | R101-DCN | 450×800 | 100×100 | **48.98** | **37.27** | 416.46 | 5.9 |

> **零推理开销**：GFLOPS 与 FPS 完全不变（蒸馏只作用于训练），NDS **+2.15**、mAP **+2.18**。这是纯粹"训练技巧"型收益，对量产部署极其友好。FPS 在 RTX 2080Ti 上测得。

论文摘要中还提到，把 FD3D 应用到 DETR3D 上有 **+3.17 NDS**。

### 8.3 GAPretrain（arXiv 2023）

**核心思想**：现有几何感知预训练（如用深度任务预训练 backbone）忽略了**视角变换**这一环节，导致图像 backbone 学到的空间知识与 view transformation 模块"对不齐"（spatial knowledge misalignment）。GAPretrain 用 **LiDAR 点云作为几何丰富的引导模态**，在预训练阶段通过**统一的 BEV 表示 + 结构提示**来弥合模态鸿沟。

**结果**：在 BEVFormer 上达到 nuScenes val **46.2 mAP / 55.5 NDS**，分别 **+2.7 / +2.1**。

> ⚠️ **代码未释出**（`GAPretrain.md` 明确写 "code coming soon"，至今未更新）。这个模块目前只有论文摘要和 BibTeX，实际价值是"读论文时的索引"。

---

## 9. 工程化复现路线图

### 9.1 环境（`docs/installation.md`）

```
Miniconda py3.8
  ├── torch 1.9.1 + cu111, torchvision 0.10.1, torchaudio 0.9.1
  ├── gcc-5 (mmcv-full 需要) / gcc-6 备选
  ├── mmcv-full 1.4.1  (cu111 / torch1.9.0 轮子)
  ├── mmdet 2.19.0
  ├── mmsegmentation 0.20.0
  ├── mmdetection3d v0.18.1  ← 从 gitee 克隆（规避 GitHub 网络问题）
  ├── timm 0.5.4
  └── waymo-open-dataset-tf-2-6-0 1.4.1 + lingvo 0.10.0
```

> 📌 **注意**：`docs/installation.md` 里的 mmdetection3d 和 bevformer 都指向 **gitee 镜像**，说明作者团队自己也遇到过 GitHub 访问问题。

### 9.2 数据

```shell
# 完整 Waymo v1.3 → Kitti 格式
cd code
python tools/create_data.py waymo --root-path ./data/waymo --out-dir ./data/waymo --extra-tag waymo

# Waymo mini (1/5, Kitti 格式)：Google Drive / 百度网盘
cat training.tar.gz* | tar -xzv
```

目录结构（`docs/data_preparation.md`）：

```
waymo_playground/
├── ckpts/r101_dcn_fcos3d_pretrain.pth
└── data/waymo_mini/
    ├── training/
    ├── waymo_calibs.pkl
    ├── waymo_mini_infos_train.pkl
    ├── waymo_mini_infos_val.pkl
    └── gt.bin / gt.bin.pkl
```

### 9.3 训练与评测

```shell
cd waymo_playground
sh ./tools/dist_train_video.sh projects/configs/bevformer/waymo_mini_r101_baseline.py 8
sh ./tools/dist_test_video.sh  projects/configs/bevformer/waymo_mini_r101_baseline.py \
                                work_dirs/waymo_mini_r101_baseline/latest.pth 8
```

带数据增强的配置：

```shell
sh ./tools/dist_train_video.sh projects/configs/bevformer/data_aug/waymo_mini_r101_ms_flip.py 8
```

### 9.4 工具箱独立使用

```bash
pip install numpy opencv-python
pip install bev-toolbox
```

```python
import cv2, numpy as np
from bev_toolbox.data_aug import RandomScaleImageMultiViewImage

transform = RandomScaleImageMultiViewImage(scales=[0.9, 1.0, 1.1])
imgs       = [cv2.imread(f'example/cam{i}_img.jpg') for i in range(5)]
cam_intr   = [np.load(f'example/cam{i}_intrinsic.npy') for i in range(5)]
cam_extr   = [np.load(f'example/cam{i}_extrinsic.npy') for i in range(5)]
lidar2img  = [np.load(f'example/cam{i}_lidar2img.npy') for i in range(5)]

imgs_new, cam_intr_new, lidar2img_new = transform(imgs, cam_intr, cam_extr, lidar2img)
```

> ⚠️ `example/` 目录里**只有 5 张 jpg 和 example.md，没有 .npy 标定文件**——README 引用它们的链接指向 Google Drive，且 issue #29 就是在催这个缺失。所以上面的示例代码**开箱即用跑不通**，需要自行下载标定。

### 9.5 VCD 复现

```bash
pip install openmim && mim install mmcv-full && mim install mmdet && mim install mmsegmentation
git clone https://github.com/open-mmlab/mmdetection3d.git && cd mmdetection3d && pip install -e .

# 训练教师
bash tools/dist_train.sh configs/vcd/vcd-convnext-base.py 8
# 训练学生
bash tools/dist_train.sh configs/vcd/vcd-r50.py 8
# 蒸馏
bash tools/dist_distill.sh configs/vcd/MotionDistiller_convnext-r50-4motion.py 8
# 评测学生
bash tools/dist_test_stu.sh configs/vcd/MotionDistiller_convnext-r50-4motion.py ckpts/... 8
```

环境建议直接follow **BEVDet2.0** 的指南（README 作者自己的建议），因为 VCD 是 fork 自 BEVDet 系。

### 9.6 FocalDistiller 复现

```bash
# 把 FocalDistiller 目录移到 mmdetection3d 下
mv nuScenes_playground/FocalDistiller <mmdetection3d>/FocalDistiller

# 训练教师/学生
tools/dist_train.sh ./projects/configs/bevformer/bevformer_base.py 8
# 蒸馏
./tools/dist_distill.sh projects/configs/distiller/base_distill_small_with_pv-cwd_bev-l2-heatmap.py 8
# 评测学生
PYTHONPATH=".":$PYTHONPATH python -m torch.distributed.launch --nproc_per_node=1 \
  tools/testStu.py projects/configs/distiller/base_distill_small_with_pv-cwd_bev-l2-heatmap.py \
  ckpts/..._ep24.pth --launcher pytorch --eval bbox
```

---

## 10. 学习与研究路径建议

### 路径 A · 纯入门（1–2 天）

**目标**：搞懂 BEV 是什么、为什么要鸟瞰图。

1. 读 `figs/general_overview.png`（总览图，先建立坐标系认知）
2. 读综述论文的 Introduction + Fig. 2（视角变换三分类）
3. 读 `docs/paper_list/bev_camera.md`，只精读 5 篇：**LSS → BEVDet → BEVDepth → BEVFormer → PETR**
4. 看 `example/example.md` + `figs/waymo_coordinate.png`，理解 Waymo 自车坐标系
5. 跑通 `bev_toolbox` 的 `RandomScaleImageMultiViewImage` 示例（需自备 npy，或改用仓库内 jpg 手动构造 K/lidar2img）

**避坑**：不要一上来就装 mmdet3d 0.18.1 的老环境（CUDA 编译会浪费一整天）。

### 路径 B · 复现 BEVFormer++（1–2 周）

**目标**：跑通 Waymo playground，理解每一项 trick。

1. 环境：按 §9.1 装（**建议用 docker 或 conda 隔离，别污染主环境**）
2. 数据：下 Waymo mini（~50GB 级视版本），跑 `create_data.py`
3. 训练 baseline → 复现 34.9 LET-mAPL
4. 逐个打开配置开关：`data_aug/waymo_mini_r101_ms_flip.py` → +0.7
5. 读 `models/modules/bev_transformer.py` 的自车运动补偿三件套
6. 读 `models/dense_heads/bevformer_detr_head_corner_pooling.py` 理解 Corner Pooling
7. 读 `models/attns/` 6 个注意力实现，对照 BEVFormer 原论文

**建议的实验**：把 `bev_h_/bev_w_` 和 `point_cloud_range` 改一下，观察 `rotate(prev_bev, center=[70,150])` 硬编码中心导致的错位——这本身就是一个很好的复现报告素材。

### 路径 C · 研究者：蒸馏方向（2–4 周）

**目标**：搞懂"重模型蒸馏进轻模型"这条产线路径。

1. **VCD**（推荐先读）：`VCD/README.md` → `distillers/motion_distiller.py` 的 `forward_train` → `distill_BEV` 配置 → `losses/` 里 7 种蒸馏 loss
   - 重点理解 `MotionDistiller` 如何用 `prepare_inputs` / `single_adj_objects` 处理时序运动物体
   - 重点理解 `_make_deconv_layers` 的通道适配
2. **FocalDistiller**：`HeatmapDetectionDistiller` 的 **hook 机制**（不改主干就能蒸馏任意模块对）→ `bevformer_distiller.py` 的 decoder 蒸馏 → `base_distill_small_with_pv-cwd_bev-l2-heatmap.py` 里 PV/BEV/L2-heatmap 三路 loss 的组合
3. **GAPretrain**：只有摘要，理解"几何感知预训练 × 视角变换对齐"的问题设定
4. 自己在 **nuScenes** 上做 teacher→student 蒸馏，**注意先测 baseline 再蒸馏**（很多"提升"其实是 baseline 没调好）

### 路径 D · 工具箱/工程向（1 周）

**目标**：把 `bev_toolbox` 的思路迁移到自己项目。

1. 通读 `bev_toolbox/data_aug/functional.py`（179 行）——重点是 **flip 的坐标系分支**
2. 用 `bev_toolbox/wrapper_mmdet3d/data_aug_mmdet3d.py` 的注册方式接进自己的 mmdet3d 工程
3. **动手补齐 TODO**：Grid Mask 已在 detector 里实现，抽回 toolbox；补 Test-time Augmentation；补 Weighted Box Fusion
4. 补 `example/` 缺失的 .npy 标定文件（顺带解决 issue #29）

### 适合做的研究课题（基于本仓库的空白点）

| 课题 | 依据 | 难度 |
|---|---|---|
| **参数无关的 BEV 泛化**：不重训直接适配新相机数/新间距 | 综述痛点④，仓库完全没碰 | ★★★ |
| **BEV 占据预测（Occupancy）**：综述提到但仓库无实现 | `docs/paper_list` 里 occupancy 只在总览图出现 | ★★★ |
| **时序对齐的鲁棒性**：硬编码旋转中心是明显破绽 | `bev_transformer.py:286` | ★★ |
| **多模态教师的更优蒸馏位点**：VCD 只开 BEV 一路，PV/Depth 是现成空位 | `MotionDistiller_convnext-r50-4motion.py` | ★★ |
| **BEV 网格真值自动化生成**：解决痛点② | 综述 | ★★★★ |
| **边缘部署**：BEV Transformer 的稀疏化/蒸馏 | 与你手上的 RK3588S 课题天然契合 | ★★★ |

---

## 11. 优点、局限与风险提示

### ✅ 优点

1. **稀缺的三合一结构**：综述 + 可运行配方 + 新算法，在 BEV 领域几乎没有第二个这样的仓库。
2. **工具箱抽象质量高**：567 行、纯 numpy+cv2、注释/类型/dict 规范，**它是真的想被复用**，不是自娱自乐的 demo。
3. **坐标系处理严谨**：Waymo / nuScenes 两套坐标系的翻转语义都正确处理了，这是多数 BEV 仓库偷懒的地方。
4. **消融数据齐全**：13 项 trick 逐项指标，虽然部分未完成，但方向性结论可靠。
5. **踩坑记录诚实**：README 的 "Ongoing Features" 勾选框明确列出未完成项，比很多仓库夸大其词要强。
6. **工业视角**：`figs/industrial_pipeline.jpg` 和 `docs/paper_list/industry.md` 是很多纯学术仓库没有的。
7. **论文-代码闭环**：VCD / FocalDistiller 都是自研方法的一手实现，不是复现别人的。

### ⚠️ 局限

1. **文档与代码脱节**：`docs/installation.md` 引用的 `code/` 目录在仓库里**根本不存在**（仓库结构是 `waymo_playground/`）；`data_preparation.md` 的路径也需自行调整。
2. **依赖栈严重过时**：torch 1.9.1 + mmcv-full 1.4.1 + mmdet3d 0.18.1（2022 年）。在 2026 年的新 GPU 上编译 `bev_pool_v2`、`iou3d` 等 CUDA 算子是主要障碍（仓库里带了预编译的 `metrics_ops.so`，但依赖 CUDA 版本）。
3. **数据资产不在仓库里**：Waymo mini、预训练权重、checkpoint 全在 Google Drive / 百度网盘。**国内访问 Google Drive 需梯子**，百度网盘限速——这是最大的复现障碍。
4. **消融数字未最终校验**：README 自述 png→jpg 数据格式切换后需要重新验证。
5. **代码风格不统一**：`nuscnes_eval.py` 拼写错误、单行 `if ...: shift_x = shift_x` 这类无操作语句（`bev_transformer.py:272`）、注释掉的死代码。
6. **硬编码超参**：`rotate(prev_bev, center=[70, 150])` 与配置强耦合；`example/` 的示例缺 .npy。
7. **仅 1 个 Release（v0.1）**，无 tag 化的子项目版本管理，引用不友好。
8. **VCD 离线代码、FocalDistiller 完整实现、GAPretrain 代码全部缺失**——算法家族的"完整度"低于宣传。

### 🔒 风险提示

- **Apache-2.0**：可商用闭源分发，但需保留版权声明与 LICENSE；**商标/专利另算**。
- **论文代码 ≠ 商用代码**：BEVFormer 系代码大量引用 mmdet3d（Apache-2.0）和 BEVFormer（Apache-2.0），但 `bev_pool_v2` 算子、`metrics_ops.so` 等二进制的来源与授权需要单独确认。
- **数据集许可严格**：Waymo Open Dataset 有独立的非商业/商用条款，**代码许可不等于数据许可**。nuScenes 有 CC BY-NC-SA 类限制（商用需申请）。
- **模型权重的许可**：README 里链到的 checkpoint（含 BEVFormer 官方 `r101_dcn_fcos3d_pretrain.pth`）继承各自训练数据的许可，量产前务必单独过法务。
- **本仓库未做安全加固**：训练/推理脚本无输入校验，仅适合作为研究代码使用。

---

## 附录 A · 关键源码索引

| 我想看… | 去这里 |
|---|---|
| 多视图同步缩放 + 内外参同步 | `bev_toolbox/data_aug/functional.py:36` `scale_image_multiple_view` |
| 多视图水平翻转（图像） | `bev_toolbox/data_aug/functional.py:77` `horizontal_flip_image_multiview` |
| **3D 框翻转（两套坐标系）** | `bev_toolbox/data_aug/functional.py:101` `horizontal_flip_bbox` |
| **相机参数翻转（两套坐标系）** | `bev_toolbox/data_aug/functional.py:118` `horizontal_flip_cam_params` |
| CAN bus 翻转 | `bev_toolbox/data_aug/functional.py:160` `horizontal_flip_canbus` |
| 可调用增强类 | `bev_toolbox/data_aug/transforms.py:28,84` |
| mmdet3d 插件注册 | `bev_toolbox/wrapper_mmdet3d/data_aug_mmdet3d.py` |
| **自车运动补偿（旋转）** | `waymo_playground/.../models/modules/bev_transformer.py:283` |
| **自车运动补偿（平移）** | `waymo_playground/.../models/modules/bev_transformer.py:266` |
| **CAN bus 条件化** | `waymo_playground/.../models/modules/bev_transformer.py:306` |
| 3D 参考点生成 | `waymo_playground/.../models/modules/bev_transformer.py:141` `get_reference_points` |
| BEV 编码器 V2 | `waymo_playground/.../models/modules/encoderV2.py:29` |
| **Corner Pooling 头** | `waymo_playground/.../models/dense_heads/bevformer_detr_head_corner_pooling.py` |
| 5 种检测头 | `waymo_playground/.../models/dense_heads/`（detr_head / detr_head2 / centerpoint_head / anchor_head / corner_pooling） |
| 6 种注意力实现 | `waymo_playground/.../models/attns/`（V2/V4/3D/BEV 交叉/TSA/SCA） |
| EMA / SyncBN Hook | `waymo_playground/.../models/hooks/hooks.py:42,155,170` / `sync_norm.py:94` |
| 匈牙利匹配 3D | `waymo_playground/.../core/bbox/assigners/hungarian_assigner_3d.py` |
| 匹配代价 | `waymo_playground/.../core/bbox/match_costs/match_cost.py` |
| 无 NMS 解码 | `waymo_playground/.../core/bbox/coders/nms_free_coder.py` |
| Waymo 评测 | `waymo_playground/.../core/evaluation/eval_hooks.py` + `datasets/eval_waymo.py` |
| **VCD 主蒸馏器** | `nuScenes_playground/VCD/mmdet3d/distiller/distillers/motion_distiller.py:25` |
| VCD 组合蒸馏器 | `nuScenes_playground/VCD/mmdet3d/distiller/distillers/combine_distiller.py:25` |
| VCD 蒸馏损失库 | `nuScenes_playground/VCD/mmdet3d/distiller/losses/`（7 种） |
| VCD 检测器 | `nuScenes_playground/VCD/mmdet3d/models/detectors/vcd.py:51` |
| **FD3D 蒸馏器（hook 机制）** | `nuScenes_playground/FocalDistiller/projects/mmdet3d_plugin/kd/distillers/heatmap_detection_distiller.py:18` |
| FD3D 头蒸馏 | `nuScenes_playground/FocalDistiller/projects/mmdet3d_plugin/bevformer/dense_heads/bevformer_distiller.py:30` |
| FD3D 蒸馏分支 | `nuScenes_playground/FocalDistiller/projects/mmdet3d_plugin/bevformer/distill_branch/` |

---

## 附录 B · 完整参考文献与外部链接

### 核心论文

- **综述**：Li, Hongyang; Sima, Chonghao; Dai, Jifeng; et al. *"Delving into the Devils of Bird's-Eye-View Perception: A Review, Evaluation and Recipe."* IEEE Transactions on Pattern Analysis and Machine Intelligence, 2023. DOI: `10.1109/TPAMI.2023.3333838` · [arXiv:2209.05324](https://arxiv.org/abs/2209.05324) · [IEEE Xplore](https://ieeexplore.ieee.org/document/10321736)
- **VCD**：Huang, Linyan; Li, Zhiqi; Sima, Chonghao; Wang, Jingdong; Qiao, Yu; Li, Hongyang. *"Leveraging Vision-Centric Multi-Modal Expertise for 3D Object Detection."* NeurIPS 2023. · [arXiv:2310.15670](https://arxiv.org/abs/2310.15670)
- **FocalDistiller / FD3D**：Zeng, Jia; Chen, Li; Deng, Hanming; Lu, Lewei; Yan, Junchi; Qiao, Yu; Li, Hongyang. *"Distilling Focal Knowledge from Imperfect Expert for 3D Object Detection."* CVPR 2023, pp. 992–1001. · [CVF 开放获取](https://openaccess.thecvf.com/content/CVPR2023/html/Zeng_Distilling_Focal_Knowledge_From_Imperfect_Expert_for_3D_Object_Detection_CVPR_2023_paper.html)
- **GAPretrain**：Huang, Linyan; Wang, Huijie; Zeng, Jia; Zhang, Shengchuan; Cao, Liujuan; Ji, Rongrong; Yan, Junchi; Li, Hongyang. *"Geometric-aware Pretraining for Vision-centric 3D Object Detection."* arXiv:2304.03105, 2023. · [arXiv:2304.03105](https://arxiv.org/abs/2304.03105)

### 仓库与榜单

- 本仓库：<https://github.com/OpenDriveLab/Birds-eye-view-Perception>
- 工具箱 PyPI：<https://pypi.org/project/bev-toolbox/>
- OpenDriveLab 主页：<https://opendrivelab.com/>
- BEVFormer（基线）：<https://github.com/fundamentalvision/BEVFormer>
- BEVDet 系列：<https://github.com/HuangJunJie2017/BEVDet>
- PETR / PETRv2：<https://github.com/megvii-research/PETR>
- BEVDepth / BEVStereo：<https://github.com/Megvii-BaseDetection/BEVDepth>
- BEVFusion (MIT)：<https://github.com/mit-han-lab/bevfusion>
- Waymo Open Dataset：<https://waymo.com/open/>
- nuScenes：<https://www.nuscenes.org/>

### 本报告的方法说明

- **仓库数据**：通过 `git clone --depth 1`（SSH）拉取 `master` 分支，快照 `dd329bd`；仓库指标通过 GitHub REST API（`gh api`）于 2026-10-30 实时抓取；代码体量通过 `find`/`wc -l` 实测统计。
- **论文信息**：arXiv:2209.05324 与 T-PAMI 版本信息通过 Web 检索交叉验证。
- **所有"作者原话"**：均来自仓库内文件（README / docs / 源码 docstring），已在正文中标注来源行号。
- **未做的事**：未在本机实际跑通训练/评测（需要 CUDA 环境 + Waymo 数据），因此**所有性能数字均转述自仓库 README 与论文，未经本地复现验证**。

---

*报告完成于 2026-10-30 · 配套 HTML 版本见 `BEV-Perception-研究报告.html` · 本研究仅用于技术学习与研究交流*
