---
name: ml-cnn-vision
description: |
  卷积网络适用判断与设计纪律。用户问"该不该用 CNN/图像怎么设计""卷积核/感受野/池化和
  BatchNorm 放哪"、"ResNet 为何有效"、拿 CNN 做表格序列数据时激活。动作: 验卷积归纳偏好
  匹配(局部性/平移不变)→组件链(小核堆叠/池化/BN)→演进逻辑(VGG→ResNet 残差动机)→何时
  不用 CNN；含 ViT 对先验价值重估。不适用于: 跨族选型(ml-transformer-era)、训练排障
  (ml-deep-training-playbook)。trigger: CNN convolution 卷积, receptive field, residual
source_book: 经典文献共识（LeCun LeNet-5 1998; He ResNet 2016; Goodfellow《Deep Learning》第9章）
source_chapter: "经典文献共识(1998-2016): 卷积归纳偏好/组件设计纪律/架构演进逻辑/ViT 挑战"
tags: [cnn, computer-vision, inductive-bias, resnet, architecture-design]
related_skills:
  - slug: ml-transformer-era
    relation: contrasts-with
  - slug: ml-task-matching
    relation: depends-on
  - slug: ml-deep-training-playbook
    relation: composes-with
  - slug: ml-overfitting-modern
    relation: composes-with
  - slug: ml-methodology-router
    relation: composes-with
---

# 卷积网络的适用判断 — 归纳偏好匹配了才值得用

## R — 原文 (Reading)

> **来源说明**: 本 skill 属扩充批D——主题超出西瓜书覆盖范围（西瓜书第 5 章未涉及卷积网络），R 段改引 CNN 奠基文献并标注来源性质；凡无法保证逐字精确处一律标（转述）。

> Convolutional networks combine three architectural ideas to ensure some degree of shift and distortion invariance: local receptive fields, shared weights, and spatial or temporal sub-sampling.（转述）
>
> — 转述自 Y. LeCun 等《Gradient-Based Learning Applied to Document Recognition》(Proc. IEEE, 1998), 即 LeNet-5 论文（来源性质：奠基论文公认表述）

> Degradation is not caused by overfitting: deeper plain networks have higher *training* error. We hypothesize that it is easier to optimize the residual mapping than the original, unreferenced mapping—letting stacked layers fit F(x) = H(x) − x, so H(x) = F(x) + x with identity shortcuts.（转述）
>
> — 转述自 K. He 等《Deep Residual Learning for Image Recognition》(CVPR 2016)（来源性质：奠基论文公认表述）

---

## I — 方法论骨架 (Interpretation)

用不用 CNN，取决于数据的**信号结构**与卷积的**归纳偏好**是否对得上，而不是"这是不是图像任务"的表面标签：

- **归纳偏好三件套**: 局部性（相邻像素相关性强，特征先在小邻域内提取）、参数共享（同一模式在图中任何位置同样有用——平移不变性）、下采样层次化（浅层边缘纹理→深层部件语义）。信号满足这三条（图像、谱图、某些时频表示）→ 卷积是高性价比先验；不满足 → 先验变成枷锁。
- **组件决策链**: ①感受野靠堆叠——两层 3×3 等效 5×5 视野但参数更少、非线性更多，VGG 式小核堆叠是默认；②池化的职责是把空间分辨率降下来换取平移鲁棒性与计算量控制，现代替代品是步长卷积；③BatchNorm 插在卷积后、激活前，让深层堆叠可训（其工程细节归 `ml-deep-training-playbook`）；④通道数随深度递增、分辨率随深度递减，是"语义变浓、位置变粗"的标准交换。
- **架构演进逻辑**: LeNet 证明手写数字可端到端；AlexNet 靠 GPU+ReLU+Dropout 把同一思想放大到 ImageNet；VGG 用统一的小核堆叠把"深度换表达力"推到 19 层；再加深撞上**退化问题**——更深的普通网络训练误差反而更高（优化困难而非过拟合）。ResNet 的残差连接 F(x)+x 给恒等映射留出默认通路，梯度获得跨层高速通道，上百层从此可训。
- **何时不该用 CNN**: 表格数据没有空间局部结构，卷积先验纯属浪费甚至有害（GBDT 常胜）；序列数据的主干选择另有路线（见 `ml-rnn-sequence`）；小数据上强先验虽好但预训练模型几乎总是更强起点。

一句话：**先验匹配定资格（局部性+平移不变），组件链定施工（小核堆叠/池化/BN 位置），演进史定上限认知（残差解锁深度），数据形态定去留（表格序列另寻主干）。**

---

## A1 — 文献中的经典应用 (Past Application)

*（本批主题超出西瓜书覆盖范围，A1 改引奠基文献的经典案例，均为学界公认的原始实验叙述）*

### 案例 1: ResNet 对"退化问题"的定性（本 skill 核心 A1）
- **问题**: He 等在 CIFAR-10 上发现 56 层普通卷积网的训练误差反而高于 20 层——若按旧直觉应怀疑过拟合，但训练误差也更高，矛盾指向别处。
- **方法论的使用**: 先拆假设：加深本身没错（20>浅层），错在更深的普通结构**优化不动**；再重构问题：与其让堆叠层直接拟合目标映射 H(x)，不如让其拟合残差 F(x)=H(x)−x 并配恒等捷径——若最优解就是恒等映射，把权重推向零比穿过非线性拟合恒等容易得多。
- **结论**: 退化是优化问题而非容量问题；残差重构后 152 层乃至千层网络均可稳定训练。
- **结果**: ResNet 拿下 2015 年 ImageNet/COCO 多项冠军；残差连接成为此后几乎所有深层架构（含 Transformer）的标配组件。

### 案例 2: LeNet-5 的端到端数字识别
- **问题**: 传统 OCR 依赖人工特征提取（边缘检测+手工特征+分类器拼接），每个环节的误差层层放大。
- **方法论的使用**: LeCun 等用局部感受野+权值共享+下采样的卷积结构，把字符图像直接映射到类别，梯度端到端回传；共享权重同时大幅压缩参数量以适应当时算力与小样本。
- **结论**: 特征可以学而不必手造，且卷积先验让学习在有限样本下可行。
- **结果**: LeNet-5 承接了美国大量支票手写数字识别的实际业务；其结构三要素沿用至今。

### 案例 3: VGG 的"小核堆叠"纪律
- **问题**: 2014 年前后各家以五花八门的大核/异形核追求更大感受野与名次，架构缺乏可复制的规律。
- **方法论的使用**: VGG 团队固定设计语言——全部 3×3 小核逐层堆叠、每两次/三次堆叠后接一次 2×2 最大池化、通道数翻倍随分辨率减半；用统一规则隔离出"深度"这一个变量做消融。
- **结论**: 两层 3×3 的组合在参数更少的前提下比单层大核视野相同而非线性更强；深度本身（在残差出现前的极限内）就是主要收益来源。
- **结果**: VGG 的均一化设计成为后续架构的模板式参考，其权重长期作为通用视觉特征被社区复用。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. 用户拿到一个图像/网格状数据任务（分类/检测前置骨干），纠结要不要用 CNN、用什么深度的 CNN。
2. 用户在设计网络时问具体组件："3×3 还是 5×5""池化用 max 还是 avg、要不要保留""BatchNorm 放激活前还是激活后"。
3. 用户读到 ResNet/残差连接，想弄懂"为什么更深反而更差""恒等映射为什么能救训练"。
4. 用户想给表格数据或一维序列"也套个 CNN"，需要被拦下或给出正当理由。
5. 用户好奇 ViT 出现后 CNN 是否过时、归纳偏好在多大数据量下还值多少钱。

### 语言信号 (用户的话里出现这些就应激活)

- "**CNN**/**卷积**适合什么任务""图像分类用什么网络""convolution 该不该用"
- "**卷积核**多大""**感受野**怎么算/receptive field""**池化**/**pooling** 要不要"
- "**BatchNorm** 放哪里""BN 在激活前还是后""channel 数怎么定"
- "**ResNet**/**残差**为什么有效""为什么网络越深越难训/deeper worse""退化问题"
- "LeNet/AlexNet/**VGG** 区别""CNN 过时了吗""**ViT** 还需要 CNN 吗"

### 与相邻 skill 的区分

- 与 `ml-transformer-era` 的区别: 那是跨架构族的横向选型（CNN vs RNN vs Transformer 三方对照、数据规模轴）；本 skill 是选定 CNN 路线后的**纵向设计与理解纪律**。用户还在"选谁" → transformer-era；已确定走卷积路线问怎么设计/为何如此 → 本 skill。ViT 挑战话题两边共管：选型视角归 transformer-era，CNN 先验价值的历史定位归本 skill B 段。
- 与 `ml-deep-training-playbook` 的区别: 那是训练执行层排障（loss 不动/NaN）；本 skill 是设计与原理层。设计定了之后训不动 → playbook。
- 与 `ml-neural-training` 的区别: 那是西瓜书第 5 章 MLP 语境的训练决策（隐层数/早停），不含卷积结构概念；全连接小网络归它，带卷积结构的归本 skill。
- 与 `ml-task-matching` 的区别: 那是算法族级别的第一性检查器；本 skill 是其在视觉形态上的专门展开。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 应按以下步骤执行:

1. **归纳偏好匹配检验（资格判定）**
   - 回答三个问题: 数据是否有局部相关性强的空间/网格结构？模式是否具有平移等变性？是否存在由浅到深的层次化语义？
   - 完成标准: 明确写出"三条满足几条"及证据；图像/谱图/时频图通常三条全中。
   - 判停条件: 表格数据（无空间结构）→ 劝退 CNN，推荐 GBDT/MLP 路线（转 `ml-task-matching` 复核），本链终止；一维序列 → 转 `ml-rnn-sequence` 评估序列主干，本链终止；三条满足 → 进步骤 2。

2. **起点选择：从零还是预训练**
   - 默认先评估预训练 backbone 微调（现代共识起点）；只有数据分布极特殊（如医学影像特殊模态）或教学目的才考虑从零训练。
   - 完成标准: 写明起点选择及理由（数据量×领域距离）。
   - 判停条件: 用户实际要解决的是"自训还是微调" → 转 `ml-pretraining-paradigm` 决策树，本链终止。

3. **组件决策链施工**
   - 按"小核堆叠(3×3)→卷积后激活前插 BN→池化/步长卷积降分辨率→通道随深度翻倍"的默认纪律给出配置建议，并对每个非默认选择说明理由。
   - 完成标准: 输出一份含层数/核尺寸/通道数/归一化位置的架构草案，且每个偏离默认之处附一句依据。
   - 判停条件: 若用户数据量极小(<1k 张)且无预训练可用 → 建议降级为浅层网络+强正则或传统特征基线，暂停深架构施工。
   - 判停条件: 若预期深度超过 ~20 层普通堆叠 → 必须引入残差连接（步骤 4 说明动机），不得交付无残差的超深普通网络。

4. **深度上限与残差动机确认**
   - 向用户交代演进逻辑给出的边界: 无残差的普通网络在某个深度会遇到退化（训练误差反升），残差连接通过恒等通路解除此限制。
   - 完成标准: 结论中明确包含"退化是优化问题而非容量问题"的表述及残差连接有无的决定。
   - 判停条件: 若用户问的是"为什么我加层后反而更差"且已排除 bug → 直接按退化问题作答并给出残差方案，不再展开完整演进史。

5. **验证与迭代交接**
   - 架构草案交训练验证；训练中的故障（loss 不动/发散/NaN）按 playbook 流程排查；出现过拟合按正则工具箱处理。
   - 完成标准: 交接说明写明"设计层已完成、后续归 XX skill 管"的边界。
   - 判停条件: 训练正常但精度不达标 → 设计嫌疑重新上升（容量/感受野不足），回到步骤 3 复审，而非盲目加训练轮数。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 表格/结构化数据硬上 CNN → 卷积先验无处安放，GBDT 几乎总是更好更省；这是 `ml-task-matching` 的地盘。
- 用户还在 CNN/RNN/Transformer 三方摇摆 → `ml-transformer-era` 的横向对照先行，本 skill 不替它回答"选谁"。
- 训练过程故障（loss 不降/NaN/发散）→ `ml-deep-training-playbook`；本 skill 只负责设计与原理层。

### 文献反复警告的失败模式

- **ViT 的挑战——数据量决定先验的价值**: Dosovitskiy 等 (2021) 显示纯 Transformer 在足够大数据（加增强预训练 JFT-300M 量级）上超过最强 CNN，而在 ImageNet-1k 中等数据上弱于 CNN——说明卷积先验是"小数据时代的拐杖、大数据时代的脚镣"；这与"先验必须匹配任务"并不矛盾，而是给"匹配"加了数据量的条件项（回扣 task-matching 的归纳偏好框架）。引用任何"CNN 是视觉标配"的说法都需注明这一时代条件。
- **把残差当万能补丁**: 残差解决的是深层的优化通路问题；网络太浅时加残差收益甚微，且残差不修复错误标注、坏超参与数据管道 bug。
- **池化迷信/全盘废除**: max 池化丢位置信息对分类是特性、对密集预测可能是缺陷；现代检测分割常用步长卷积+空洞卷积替代，但"完全不要下采样"会让计算量失控。
- **BN 位置与批大小的忽视**: BN 依赖批统计，小 batch 下统计失真；推理忘切换滑动平均统计是经典事故（细节归 deep-training-playbook）。
- **照抄经典深度**: ImageNet 上的 152 层配方搬到小数据小图上是灾难；深度应随数据量与任务复杂度伸缩。

### 作者盲点 / 时代局限（外推声明）

- **必须声明**: 本 skill 主题超出西瓜书覆盖范围——西瓜书(2016)第 5 章止于 BP 网络与浅层讨论，卷积网络仅一笔带过；本 skill 为外推补全，素材为 LeCun (1998)、He (2016)、Dosovitskiy (2021) 等奠基文献的公认共识转述，均标"（转述）"。
- 经典 CNN 文献的叙事中心是 ImageNet 式大规模有监督分类；自监督预训练时代（MAE/DINO 等）里"什么结构算好的视觉先验"正在被重写，本 skill 的演进史叙事有其截止时间。
- 移动端/边缘部署的效率维度（剪枝/量化/蒸馏）不在本 skill 覆盖内，只做指路。

### 容易混淆的邻近方法论

- **Transformer 选型（`ml-transformer-era`）**: 一横一纵——那边比较各族架构，这边深入卷积一族的设计纪律；ViT 话题为两 skill 共同边界区，按上文分工处理。
- **训练排障（`ml-deep-training-playbook`）**: 设计层 vs 执行层；"网络设计得对但训不动"是那边的事。
- **传统 CV 流水线（SIFT/HOG+SVM）**: 同样利用局部性与平移不变性，但是手工特征而非学习特征；小样本极端场景仍有余温，勿与现代 CNN 叙事混为一谈。

---

## 相关 skills

- depends-on: ml-task-matching（归纳偏好匹配是其一般框架的特例）
- contrasts-with: ml-transformer-era（纵向设计 vs 横向选型）
- composes-with: ml-deep-training-playbook（设计定型后接力训练排障）, ml-pretraining-paradigm（起点选择）, ml-overfitting-modern（过拟合工具箱）, ml-methodology-router

---

## 审计信息

- **来源性质**: 扩充批D·核心缺口——主题超出西瓜书(2016)覆盖范围，R 段为 CNN 奠基文献的转述表述（均已标注"（转述）"）
- **验证通过**: 待流水线三重验证复核
- **测试通过率**: 待阶段 4 测试 (详见 test-prompts.json)
- **skill_version**: 0.0.1
- **蒸馏时间**: 2026-08-24
