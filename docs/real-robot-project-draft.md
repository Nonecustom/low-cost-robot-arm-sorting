# Real Robot Project Draft

## Goal

把前面阶段学到的内容压到一个可执行的真实机械臂第一版项目定义上。

当前目标不是一次把项目做满，而是先明确：

```text
我要做什么任务
我要采什么数据
模型吃什么
模型输出什么
机械臂怎么执行
```

## 1. 第一版真实机械臂任务

第一版任务建议定义为：

```text
固定桌面场景下，
机械臂根据视觉输入识别方块类别，
抓取目标方块，
并放到对应区域。
```

例如：

```text
红色方块 -> 放到左边盒子
蓝色方块 -> 放到右边盒子
```

这个任务的优点是：

```text
目标清晰
评价标准明确
容易录制 demonstration
容易观察成功和失败
同时包含“看懂抓哪个”和“执行抓取放置”两个环节
```

## 2. 我要采集什么样的 demonstration？

第一版 demonstration 可以先理解为：

```text
人通过遥操作 / 主臂控制 follower arm
完成一次完整分类抓取放置过程
```

每一条 demonstration 就是一条 episode。

第一版建议先采集：

```text
固定机位
固定桌面背景
固定目标物体类别（红 / 蓝方块）
固定放置区域
```

这样可以先降低问题难度。

## 3. 每一步 observation 至少包含什么？

第一版 observation 最少包含：

```text
image
robot_state
```

其中：

```text
image:
桌面相机图像

robot_state:
当前关节状态
当前夹爪状态
```

可以先不要求：

```text
多相机
深度图
语言指令
力传感器
```

## 4. 每一步 action 暂时定义成什么？

第一版 action 建议先定义成：

```text
关节空间增量动作
```

也就是：

```text
每个关节下一步调多少
夹爪下一步开合多少
```

这样定义的原因是：

```text
更贴近低成本机械臂的直接控制接口
更符合“根据当前状态微调一点”的控制直觉
```

## 5. 一条 episode 怎么结束？

第一版 episode 结束条件可以定义为：

```text
1. 任务成功
2. 任务失败
3. 达到最大步数 / 最大时间
```

例如：

```text
成功：
目标方块被抓起并放到正确区域

失败：
方块掉落
夹取失败
放错区域
机械臂碰撞或跑偏

超时：
执行太久仍未完成
```

## 6. 训练时模型输入和输出各是什么？

如果用 Behavior Cloning：

```text
输入：
observation

输出：
当前 action
```

如果用 ACT：

```text
输入：
observation

输出：
未来一段 action chunk
```

如果用 Diffusion Policy：

```text
输入：
observation + noisy action sequence + timestep

输出：
predicted noise
```

推理后最终仍然要落到：

```text
当前时刻要执行的 action
```

## 7. 当前版本的项目闭环

第一版真实项目可以先理解成：

```text
camera image + robot state
-> policy
-> action
-> robot execute
-> next observation
```

如果换成 ACT：

```text
camera image + robot state
-> ACT policy
-> action chunk
-> 取当前该执行的一步
-> robot execute
```

如果换成 Diffusion Policy：

```text
camera image + robot state
-> diffusion policy
-> sampled action sequence
-> 取当前该执行的一步
-> robot execute
```

## 8. 当前最合理的第一版路线

不建议一开始就用最复杂的方法。

建议顺序：

```text
1. 先采 demonstration
2. 先跑通 BC baseline
3. 再尝试 ACT
4. 最后再看是否值得接 Diffusion Policy
```

原因是：

```text
BC 最容易先形成真实闭环
ACT 和 Diffusion Policy 适合在 baseline 跑通后再比较
```

## 9. 当前不做什么

第一版真实项目先不做：

```text
多任务
杂乱多物体混杂场景
语言指令控制
深度相机融合
复杂 reward 设计
```

## 10. Success Criteria

第一版项目的成功标准建议定义为：

```text
输入桌面图像后，
系统能够区分红色方块和蓝色方块，
抓起目标方块，
并放到对应区域。
```

最小成功标准可以拆成：

```text
1. 颜色识别正确
2. 抓取成功
3. 放置到正确区域
4. 在固定机位和固定光照下可重复完成
```

建议记录的评估指标：

```text
分类正确率
抓取成功率
放置成功率
端到端任务成功率
平均完成时间
失败类型统计
```

## 11. Minimal Step Format

对于当前双色方块分类放置任务，每一步 step 可以先按最小版本理解为：

```text
step
- image
- robot_state
- action
- step_id
- done / success（可选）
```

其中：

```text
image：
当前相机图像
```

```text
robot_state：
当前各关节角度
当前夹爪状态
```

```text
action：
下一步各关节的偏移量
下一步夹爪的开合偏移量
```

可选增加：

```text
target_info：
目标类别（红 / 蓝）
目标方块在图像中的位置
```

这里要特别区分：

```text
image 是原始观测
target_info 是基于 image 得到的额外信息
```

因此当前最稳的第一版数据流可以记成：

```text
image + robot_state
-> policy
-> action
```

## 12. Minimal Episode Format

一条完整 episode 可以先按最小版本理解为：

```text
episode
- episode_id
- task_name
- steps
```

其中：

```text
episode_id：
这一条示范的唯一编号

task_name：
当前任务名称
例如：
red_block_to_left_box
blue_block_to_right_box

steps：
按时间顺序排列的一组 step
```

也就是说，一条完整示范可以先理解成：

```text
episode
- episode_id
- task_name
- steps
  - step_1
    - image
    - robot_state
    - action
    - step_id
  - step_2
    - image
    - robot_state
    - action
    - step_id
  - ...
```

如果想再多保存一点整体结果，可以增加：

```text
success
total_steps
end_reason
```

其中：

```text
success：
这一条 episode 最终是否成功

total_steps：
这一条 episode 一共有多少步

end_reason：
success / failure / timeout
```

所以当前最小的 episode 结构可以概括成：

```text
一条 episode = 一次完整任务 + 按顺序排列的很多 step
```

## 13. Demonstration Collection Checklist

在开始真实机械臂 demonstration 采集前，至少要确认下面这些点：

### 任务定义

```text
[ ] 当前任务是否已经明确
[ ] 当前任务的成功标准是否已经明确
[ ] 当前 episode 的结束条件是否已经明确
```

### 场景设置

```text
[ ] 相机位置是否固定
[ ] 桌面背景是否固定
[ ] 光照条件是否基本稳定
[ ] 目标物体类别是否固定
[ ] 放置区域是否固定
```

### 数据结构

```text
[ ] 每一步 step 的最小字段是否已经定义
[ ] 一条 episode 的最小字段是否已经定义
[ ] image / robot_state / action 的保存格式是否明确
[ ] action 当前是否定义为关节空间增量动作
```

### 采集流程

```text
[ ] 一条 episode 从哪里开始是否明确
[ ] 一条 episode 在什么情况下结束是否明确
[ ] 遥操作 / 主臂控制流程是否跑通
[ ] 是否知道如何区分成功、失败、超时
```

### 训练前准备

```text
[ ] demonstration 是否按 episode 组织
[ ] 是否能从 episode 中读取 step 序列
[ ] 是否能把 observation 和 action 对齐
[ ] 是否能统计 total_steps 和 success
```

### 第一版采集建议

```text
先采固定机位
先采固定背景
先采固定光照
先采红 / 蓝双色方块
先采固定放置区域
先不做语言指令
```

## 14. Episode Start And Reset Conditions

第一版 demonstration 采集时，可以先固定一个机械臂初始状态，作为每一轮 episode 的统一起点。

开始条件可以定义为：

```text
机械臂回到固定初始状态
桌面上的方块位置恢复到预设位置
目标盒子位置保持固定
当前场景满足本轮任务要求
```

也就是说，一轮 episode 可以开始的标准是：

```text
机械臂、方块、盒子都回到预先定义好的起始配置
```

重置条件可以定义为：

```text
上一轮 episode 结束后，
需要把机械臂恢复到固定初始状态，
把方块放回指定位置，
确认盒子位置没有变化，
然后再开始下一轮。
```

这样做的好处是：

```text
减少采集时的随机干扰
保证 demonstration 起始条件一致
让后续训练和评估更容易比较
```

## 15. 下一步最小任务

下一步不是直接训练真实机械臂模型，而是先继续明确：

```text
第一版 demonstration 采集时，哪些失败 episode 保留，哪些失败 episode 丢弃
```

## 16. Failure Episode Policy

第一版 demonstration 采集时，不建议简单地“失败就全删”，而是分成三类处理：

### 全部保留

```text
成功 episode
```

原因是：

```text
它们是第一版 BC / ACT / Diffusion Policy 训练的主数据
```

### 单独保留

```text
高质量失败 episode
```

这里的“高质量失败”指的是：

```text
开始条件正确
数据记录完整
遥操作轨迹基本自然
任务理解正确
但执行结果失败
```

例如：

```text
已经对准目标但夹取滑落
已经抓起方块但中途掉落
目标类别判断正确但放错区域
```

这些失败 episode 的价值在于：

```text
可以帮助后续分析失败类型
可以支持后续“失败恢复”系统设计
可以用于区分感知问题、控制问题和执行问题
```

建议做法是：

```text
不要先混入第一版主训练集
而是单独保存成 failure_subset
```

### 直接丢弃

```text
脏失败 episode
```

这里的“脏失败”指的是：

```text
开始条件不满足
相机画面异常
桌面场景被破坏
遥操作明显失误且轨迹不自然
image / robot_state / action 记录缺失
```

例如：

```text
机械臂没有回到初始位就开始录制
方块位置摆错
相机遮挡严重或画面模糊
记录过程中文件丢失或 step 不完整
```

这些数据不建议保留，因为：

```text
它们更像环境异常或记录异常
而不是对策略真正有帮助的失败样本
```

### 第一版建议策略

当前最稳的第一版策略可以定成：

```text
成功 episode：全部保留
高质量失败 episode：单独保存
脏失败 episode：直接丢弃
```

这样做的好处是：

```text
先保证第一版训练数据干净
避免失败数据一开始就干扰策略学习
同时保留后续做失败恢复和系统分析的空间
```

## 17. Demonstration Scale Suggestion

第一版真实机械臂 demonstration 不建议一开始就追求很大的数据量，而是先用一个能跑通闭环的规模启动。

建议按两个阶段来做：

### Phase 1：最小可执行规模

```text
红色方块 -> 左盒：20 条成功 episode
蓝色方块 -> 右盒：20 条成功 episode
总计：40 条成功 episode
```

这一阶段的目标不是追求最高成功率，而是先验证：

```text
demonstration 能采
episode 能保存
数据格式能读取
BC baseline 能训练
真实机械臂闭环能跑起来
```

### Phase 2：更稳的第一版规模

如果 Phase 1 已经跑通，再补充到：

```text
红色方块 -> 左盒：30 条成功 episode
蓝色方块 -> 右盒：30 条成功 episode
总计：60 条成功 episode
```

这一阶段的目标是：

```text
让第一版模型更稳定
让评估结果更有参考性
为后续 ACT / Diffusion Policy 提供更稳的数据基础
```

### Failure Subset：单独保留的失败数据

除了成功 episode 之外，建议每类任务额外保留一小部分高质量失败 episode：

```text
每类任务：5 条左右高质量失败 episode
总计：10 条左右高质量失败 episode
```

这些失败数据的处理方式是：

```text
单独保存
先不混入主训练集
用于后续失败分析和失败恢复设计
```

### 当前建议策略

所以当前最稳的第一版 demonstration 规模可以记成：

```text
最小起步量：40 条成功 episode
更稳的第一版量：60 条成功 episode
外加：10 条左右高质量失败 episode（单独保存）
```

这样做的好处是：

```text
不会在采集阶段投入过大
能更快看到第一版真实结果
方便后续按阶段扩充数据规模
```

## 18. Evaluation Protocol

第一版真实机械臂项目不建议只靠“跑成功一次”来判断效果，而应该先固定一个最小评估流程。

### 测试场景设置

第一版建议在相对稳定的场景下做测试：

```text
固定机位
固定光照
固定桌面背景
固定盒子位置
方块摆放在预设范围内
```

这样做的原因是：

```text
先让第一版评估结果可重复
减少环境变化带来的额外干扰
方便分析问题到底出在感知、控制还是执行
```

### 测试轮数建议

第一版可以先按最小可执行规模测试：

```text
红色方块任务：10 轮 episode
蓝色方块任务：10 轮 episode
总计：20 轮测试
```

如果后续系统更稳定，可以再扩展到：

```text
每类任务 20 轮
总计 40 轮测试
```

### 建议统计的指标

第一版至少统计下面这些指标：

```text
分类正确率
抓取成功率
放置成功率
端到端任务成功率
平均完成时间
平均完成步数
失败类型统计
```

这些指标分别回答的是：

```text
分类正确率：
系统有没有先看懂目标类别

抓取成功率：
机械臂有没有把物体稳定抓起

放置成功率：
机械臂有没有把物体放到正确区域

端到端任务成功率：
从识别到抓取再到放置，整条链路是否成功

平均完成时间 / 步数：
系统效率如何，动作是否拖沓

失败类型统计：
失败主要来自识别错误、抓取失败还是放置失败
```

### 第一版评估目标

第一版评估的重点不是追求极高分数，而是回答：

```text
这个系统是否已经形成稳定闭环
失败主要集中在哪个环节
后续应该先优化感知、控制还是执行
```

### 当前最小评估协议

所以当前最小可执行的第一版评估协议可以记成：

```text
固定机位
固定光照
固定背景
红 / 蓝任务各测试 10 次
记录分类、抓取、放置、端到端成功率
同时记录平均完成时间和失败类型
```

## 19. First-Version System Modules

为了避免第一版真实机械臂项目过于笼统，可以先把整个系统拆成 6 个模块。

### 1. Demonstration Collection

作用：

```text
采集真实机械臂 demonstration
把每一轮 episode 按统一格式保存下来
```

这一模块主要负责：

```text
机械臂回到初始状态
开始一轮 episode
按 step 记录 image / robot_state / action
保存 episode_id、task_name、success、end_reason
```

### 2. Dataset Loader

作用：

```text
把保存好的 episode 数据读取出来
转换成训练时可直接使用的数据形式
```

这一模块主要负责：

```text
读取 step 序列
对齐 observation 和 action
区分训练集 / 测试集
统计 total_steps、success、task_name
```

如果后面接 ACT 或 Diffusion Policy，还会进一步负责：

```text
构造 action chunk
构造 action sequence
准备 noisy action sequence 等训练输入
```

### 3. Perception / Target Selection

作用：

```text
根据当前图像判断目标类别
必要时定位目标方块
```

第一版可以先做得简单一些：

```text
只区分红色方块和蓝色方块
必要时输出目标类别或目标方块位置
```

这一模块的输出可以是：

```text
target_class
target_info（可选）
```

### 4. Policy Training

作用：

```text
训练控制策略
```

第一版建议先做：

```text
BC baseline
```

后续再扩展：

```text
ACT
Diffusion Policy
```

这一模块主要负责：

```text
定义模型输入输出
训练模型
保存模型参数
记录训练 loss 和实验结果
```

### 5. Inference And Robot Control

作用：

```text
把当前 observation 输入模型
得到当前要执行的 action
发送给机械臂控制器
```

这一模块主要负责：

```text
读取当前图像和 robot_state
调用训练好的 policy
得到当前 step 的控制动作
把动作发送给真实机械臂
循环执行直到任务结束
```

如果后面使用 ACT 或 Diffusion Policy，这一模块还要处理：

```text
从 action chunk 中取当前一步
从 sampled action sequence 中取当前一步
```

### 6. Evaluation And Logging

作用：

```text
按固定协议测试系统效果
记录成功率、失败类型和时间开销
```

这一模块主要负责：

```text
运行固定轮数测试
统计分类正确率
统计抓取成功率
统计放置成功率
统计端到端成功率
统计平均完成时间和失败类型
```

### 当前最小开发顺序

第一版不建议所有模块一起上，而是按下面顺序推进：

```text
1. Demonstration Collection
2. Dataset Loader
3. Policy Training（先 BC）
4. Inference And Robot Control
5. Evaluation And Logging
6. Perception / Target Selection（按需要逐步增强）
```

### 这一拆分的意义

这样拆分之后，整个项目可以先记成：

```text
采数据
-> 读数据
-> 训策略
-> 控机械臂
-> 做评估
```

而不是一开始就把它看成一个模糊的大系统。
