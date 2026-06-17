# Low-Cost Robot Arm Sorting

一个面向真实机械臂落地的具身智能项目仓库。

当前第一版任务聚焦在固定桌面场景下的双色方块分类放置：

```text
红色方块 -> 左盒
蓝色方块 -> 右盒
```

目标不是直接堆最复杂的模型，而是先形成一个稳定、可评估、可扩展的真实闭环：

```text
demonstration collection
-> dataset loader
-> policy training
-> inference and robot control
-> evaluation
```

## Current Scope

当前仓库聚焦以下内容：

```text
1. 第一版真实机械臂项目设计
2. demonstration 数据结构定义
3. 真实项目工程骨架
4. 后续 BC / ACT / Diffusion Policy 接入接口
```

当前还没有正式开始放入真实采集数据和训练脚本，仓库先作为真实项目的主仓骨架。

## Repository Structure

```text
low-cost-robot-arm-sorting/
├─ README.md
├─ docs/
│  └─ real-robot-project-draft.md
├─ configs/
│  ├─ task/
│  ├─ training/
│  └─ robot/
├─ data/
│  ├─ raw/
│  ├─ processed/
│  └─ splits/
├─ scripts/
│  ├─ collect/
│  ├─ train/
│  ├─ infer/
│  └─ eval/
├─ src/
│  ├─ datasets/
│  ├─ policies/
│  ├─ control/
│  ├─ perception/
│  └─ utils/
├─ models/
│  ├─ checkpoints/
│  └─ exported/
├─ logs/
│  ├─ train/
│  └─ eval/
├─ assets/
│  ├─ images/
│  └─ demos/
└─ notes/
```

## Key Document

项目草案位于：

[`docs/real-robot-project-draft.md`](docs/real-robot-project-draft.md)

当前已经整理了：

```text
任务定义
success criteria
minimal step format
minimal episode format
demonstration collection checklist
failure episode policy
demonstration scale suggestion
evaluation protocol
first-version system modules
```

## Recommended First Build Order

建议按这个顺序推进第一版真实项目：

```text
1. Demonstration Collection
2. Dataset Loader
3. Policy Training（先 BC baseline）
4. Inference And Robot Control
5. Evaluation And Logging
6. Perception / Target Selection（按需要增强）
```

## Near-Term Next Step

当前最自然的下一步是：

```text
把 Demonstration Collection 模块细化成真正的采集脚本设计
```

也就是继续明确：

```text
一条 episode 如何开始
每一步数据如何落盘
成功 / 失败 / 超时如何记录
```
