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

## Final Deliverable

项目计划在 2027 年 3 月前完成第一版，最终交付标准不是“机械臂偶然成功一次”，而是一个可复现、可评估的完整闭环：

```text
遥操作采集 demonstration
-> episode 数据保存与检查
-> Dataset Loader
-> BC baseline 训练与 checkpoint 保存
-> 真机加载模型并闭环执行
-> 固定协议评估
-> 量化对比实验与失败分析
```

最低交付内容：

```text
固定机位、固定背景、固定光照
红色方块放左盒，蓝色方块放右盒
40 条成功 episode 跑通第一版，随后扩充至 60 条
BC baseline
红蓝任务各测试 10 次，共 20 轮正式评估
记录分类、抓取、放置和端到端任务成功率
记录平均完成时间、平均步数和失败类型
完成 20 条训练数据与 60 条训练数据的量化对比
README、实验表、失败案例和 2-3 分钟 Demo 视频
```

ACT 是时间允许时的增强实验，Diffusion Policy 不属于第一版的必做范围。

## Current Scope

当前仓库聚焦以下内容：

```text
1. 第一版真实机械臂项目设计
2. demonstration 数据结构定义
3. 真实项目工程骨架
4. 后续 BC / ACT / Diffusion Policy 接入接口
```

当前还没有正式开始放入真实采集数据和训练脚本，仓库先作为真实项目的主仓骨架。

## Current Progress

当前已经完成的是项目目标、数据闭环、最小 step / episode 格式、失败数据处理原则、评估协议和总体目录骨架的规划。

当前尚未完成的是：

```text
真实机械臂接口
相机与遥操作接入
collect_episode.py
真实 episode 数据
Dataset Loader
BC 训练与真实推理
正式评估
```

现阶段首先补齐的不是更多目录或完整代码，而是理解项目如何从一个最小脚本逐步生长。目录只在出现真实职责分离或代码复用需求时启用，不要求一开始填满。

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
先用自然语言描述一次 episode 从开始到结束依次发生什么
```

暂时不考虑函数名、类和完整实现，只回答：

```text
场景如何重置
每一步获得哪些观测
action 从哪里来并由谁执行
step 在什么时候记录
episode 如何结束并保存
```

流程清楚后，先在单个 `collect_episode.py` 中做最小实现；只有某段能力需要被其他脚本复用，或单文件职责已经明显过多时，再将其拆到 `src/control/`、`src/datasets/` 等目录。

## Schedule

考研前，具身智能固定每周 3 小时：周内 3 次各 30 分钟保持连续性，周日 90 分钟完成可运行或可检查的阶段产出。408 仍是主线。

| 时间 | 阶段目标 | 完成标志 |
|---|---|---|
| 2026-09-11 至 09-30 | 恢复项目主线，理解目录和采集流程 | 能独立说明闭环、step/episode/dataset，并写出 episode 自然语言流程 |
| 2026-10 | 完成单文件模拟采集程序 | 不连接硬件也能生成、结束、保存并检查一条模拟 episode |
| 2026-11 | 完成模拟 Dataset Loader 与 BC 闭环 | 能从模拟 episode 读取 observation/action、训练并保存 BC 模型 |
| 2026-12 至初试 | 收束文档和硬件方案，不扩展算法 | 明确机械臂、相机、遥操作和官方接口；保持每周最低接触 |
| 初试后第 1-2 周 | 跑通真实硬件基础链路 | 能读取图像和 robot_state，并通过遥操作安全执行 action |
| 初试后第 3-4 周 | 跑通真实数据与第一版策略 | 保存真实 episode，采集首批 40 条成功数据并完成 BC 真机闭环 |
| 2027-02 上半月 | 扩充数据并完成量化实验 | 数据扩充到 60 条，完成 20 条与 60 条训练数据的对比 |
| 2027-02 下半月 | 正式评估和项目展示 | 完成 20 轮评估、失败分析、README、实验表和 Demo 视频 |

每周只设置一个主要产出。周内短时学习负责理解、设计和小函数，周日负责把本周内容运行或验证完成。
