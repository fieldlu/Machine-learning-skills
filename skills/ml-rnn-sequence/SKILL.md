---
name: ml-rnn-sequence
description: |
  序列建模路线决策：RNN/GRU/LSTM/seq2seq+注意力/Transformer 的选择。用户问"序列用 RNN
  还是 LSTM/门控干嘛的"、"翻译、时序预测怎么建模"时激活。
  动作: 任务定形→判时序梯度消失是否需门控→依赖跨度与并行性定 RNN 系还是 Transformer→
  BPTT 截断与 teacher forcing。不适用于: 跨族选型(ml-transformer-era)、视觉
  (ml-cnn-vision)、序贯RL(ml-rl-decision-loop)。trigger: LSTM GRU 门控, seq2seq,
  attention, 时序
source_book: 经典文献共识（Hochreiter & Schmidhuber LSTM 1997; Cho GRU/seq2seq 2014; Sutskever seq2seq 2014; Bahdanau 注意力 2015）
source_chapter: "经典文献共识(1997-2015): 门控机制/梯度消失时序版/seq2seq与注意力动机/路线边界"
tags: [rnn, lstm, gru, seq2seq, attention, sequence-modeling]
related_skills:
  - slug: ml-transformer-era
    relation: contrasts-with
  - slug: ml-deep-training-playbook
    relation: composes-with
  - slug: ml-task-matching
    relation: depends-on
  - slug: ml-rl-decision-loop
    relation: composes-with
  - slug: ml-methodology-router
    relation: composes-with
---

# 序列建模路线决策 — 任务定形，再选主干

## R — 原文 (Reading)

> **来源说明**: 本 skill 属扩充批D——主题超出西瓜书覆盖范围（西瓜书第 5、10 章未系统涉及循环网络），R 段改引序列建模奠基文献并标注来源性质；凡无法保证逐字精确处一律标（转述）。

> Long Short-Term Memory is designed to overcome the error back-flow problems of conventional RNNs: constant error carousels (memory cell with input and output gates) allow errors to flow back through time without vanishing or blowing up.（转述）
>
> — 转述自 S. Hochreiter & J. Schmidhuber, "Long Short-Term Memory", Neural Computation (1997)（来源性质：奠基论文公认表述）

> Encoder-decoder architectures compress a variable-length source sequence into a fixed-length vector, which becomes an information bottleneck for long sentences; Bahdanau et al. let the decoder softly attend to all encoder states weighted by relevance instead.（转述）
>
> — 转述自 K. Cho 等 "Learning Phrase Representations using RNN Encoder-Decoder"(EMNLP 2014)、I. Sutskever 等 "Sequence to Sequence Learning"(NeurIPS 2014)、D. Bahdanau 等 "Neural Machine Translation by Jointly Learning to Align and Translate"(ICLR 2015)（来源性质：奠基论文公认表述）

---

## I — 方法论骨架 (Interpretation)

序列建模的路线选择分三步走：**先给任务定形，再诊断时序训练的病，最后按数据规模和工程约束选主干。**

- **第一步·任务定形**（决定结构出口）: 多对一（整段序列→一个输出，如情感分类）；一对一（逐帧预测，如词性标注）；多对多同步（等长映射）；多对多异步（编码器读入→解码器生成，长度不等，如翻译）；以及流式（边收边出，如在线识别）。出口形态定了，才谈得上选哪族单元。
- **第二步·病因认识**: 普通 RNN 把历史压进一个不断被覆写的隐状态，BPTT 沿时间反传的连乘让远距离误差指数消失或爆炸——这是 BP 梯度消失的**时序版**。门控机制是对症药：LSTM 用细胞状态+输入/输出/遗忘三门，让信息沿"传送带"近乎无衰减地穿越时间步，由门决定何时写入/读出/清空；GRU 是其简化变体（更新门+重置门），参数更少、效果多数场合相当。
- **第三步·路线边界**: seq2seq 把变长输入压成固定向量造成信息瓶颈——长句翻译质量骤降；注意力的动机正是让解码器每一步"回头看"全部编码状态并加权取用。注意力后来独立壮大为 Transformer 的核心，接管了超长依赖与大规模可并行训练的场景。但 RNN 系并未消亡：小数据上先验更省、推理内存恒定 O(1) 与流式天然契合、边缘设备友好——这三类场景它仍常是正解。

一句话：**定形（几对几、要不要流式）→ 认病（时序梯度消失靠门控治）→ 定线（短中小数据/流式走 RNN 系；长依赖/大数据/要并行走 Transformer）。**

---

## A1 — 文献中的经典应用 (Past Application)

*（本批主题超出西瓜书覆盖范围，A1 改引奠基文献的经典案例，均为学界公认的原始工作叙述）*

### 案例 1: LSTM 攻克合成长期依赖任务
- **问题**: Hochreiter & Schmidhuber 面对的合成的"纯延迟 XOR"类任务要求网络在数百个时间步后仍能使用早期输入的信息；普通 RNN 与当时其他递归方法经长时间训练也无法学会——时序梯度消失使远端信号传不回。
- **方法论的使用**: 设计带常数误差传送带的记忆细胞：细胞状态沿时间线性流动、只受门控的乘性修改；输入门/输出门控制写入与读出，使无关扰动进不来、存储的错误信号能原路返回。
- **结论**: 时序梯度消失可以被架构设计而非调参治愈；"长期记忆"与"短期计算"在结构上分离（故称 Long Short-Term Memory）。
- **结果**: LSTM 在数百步延迟的任务上成功学习；此后十余年成为语音识别、机器翻译、语言模型的默认序列组件。

### 案例 2: seq2seq + 注意力拯救长句翻译
- **问题**: Sutskever 等的 encoder-decoder 把整个源句压进一个固定向量，BLEU 分数随句长增长明显崩塌（转述其论文观察）；Cho 等同样报告长句性能退化。
- **方法论的使用**: Bahdanau 等把瓶颈拆掉——解码器每个生成步对全部编码器隐状态算相关性权重（软对齐），按权重加权和得到动态上下文向量；对齐矩阵顺带成为可解释的诊断工具。
- **结论**: 固定向量瓶颈是长句退化的根因；"每步按需回看"优于"一次压缩到底"，且注意力权重提供了翻译对齐的可视化证据。
- **结果**: 注意力机制迅速成为神经机器翻译标配，并为后来的 Transformer（完全抛弃循环结构的 self-attention）铺平道路。

### 案例 3: GRU 的简化验证
- **问题**: LSTM 三门结构参数多、实现繁；门是否都必要、能否砍到两个而不掉点，缺少系统对照。
- **方法论的使用**: Cho 等提出 GRU——更新门合并输入门与遗忘门的功能（新信息写入多少=旧信息遗忘多少），重置门控制读取历史；在多项任务上与 LSTM 同台对照。
- **结论**: 门控的核心收益来自"可控读写"这一最小机制；GRU 以约四分之一的门控复杂度取得与 LSTM 相当的表现（转述其对照结论）。
- **结果**: GRU 成为资源受限场景与快速实验中的常用替代；"先试 GRU 再上 LSTM"成为常见实践路径。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. 用户拿到文本/音频/传感器/金融时序等序列数据，不知道该用什么结构建模，问"RNN/LSTM 还能用吗"。
2. 用户在读代码或论文时遇到 LSTM 门控、seq2seq、teacher forcing、注意力权重等概念想搞懂动机。
3. 用户做情感分析/命名实体识别/机器翻译/时间序列预测，想知道各自该套哪种输入输出形态。
4. 用户要在边缘设备/嵌入式环境做实时流式识别，关心推理内存与延迟，听说 Transformer 又大又不适合流式。
5. 用户的时序预测任务只有几百条样本，担心 Transformer 这种数据吞天兽喂不饱。

### 语言信号 (用户的话里出现这些就应激活)

- "**RNN**/**LSTM**/**GRU** 怎么选""循环神经网络还过时吗""vanishing gradient **时序**"
- "**门控**/**gating** 是干嘛的""forget gate 遗忘门作用""cell state 细胞状态"
- "**seq2seq**""encoder decoder 编码器解码器""**注意力**为什么需要/attention bottleneck"
- "情感分类用什么模型""机器翻译怎么建""**时间序列预测**用什么网络"
- "**流式**/streaming 实时识别""边缘设备部署序列模型""时序任务小数据"

### 与相邻 skill 的区分

- 与 `ml-transformer-era` 的区别: 那是含数据规模×算力的跨族架构总选型，回答"CNN/RNN/Transformer 选谁"；本 skill 是序列一族内部的**纵向路线细化**（RNN→门控→seq2seq→注意力的演进逻辑与各自适用面）。用户问"序列任务该不该上 Transformer" → 两边共管，transformer-era 给规模判断、本 skill 给 RNN 系的存留理由；用户明确在 RNN 系内部选型或理解门控/注意力动机 → 本 skill。
- 与 `ml-cnn-vision` 的区别: 那边管空间局部性的视觉信号；本 skill 管时间/顺序依赖的序列信号。1D 卷积用于序列属于两 skill 边界区：作为特征提取技巧归 cnn-vision 思路，作为序列主干对比归本 skill。
- 与 `ml-neural-training` 的区别: 那是 MLP/BP 层面的通用训练纪律；序列特有的 BPTT、teacher forcing、梯度截断归本 skill 讨论。
- 与 `ml-rl-decision-loop` 的区别: RL 也处理序贯决策，但那是"智能体通过交互学策略"；本 skill 是"监督学习处理观测序列"。有环境交互与奖赏信号 → rl-decision-loop；有序列标注/生成目标 → 本 skill。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 应按以下步骤执行:

1. **任务定形**
   - 把需求写成"输入长度 × 输出长度 × 是否流式": 多对一 / 一对一 / 多对多同步 / 多对多异步(编码-解码) / 流式增量。
   - 完成标准: 用户确认或从需求描述中推断出的定形结论被显式写出（例:"评论→好评/差评 = 多对一非流式"）。
   - 判停条件: 若发现其实没有真正的顺序依赖（打乱元素顺序任务不变，如词袋够用的短文本分类） → 建议先试无序模型作基线，序列结构未必必要；确有顺序依赖 → 进步骤 2。

2. **依赖跨度与小数据评估**
   - 估计有效依赖跨度（几个 token/几个采样点之外的历史还有影响？）、盘点数据量与领域内有无预训练模型可用。
   - 完成标准: 写出"依赖跨度：短/中/长；数据量：少/中/大；预训练可用：是/否"的三元判定。
   - 判停条件: 长依赖+大数据+GPU 充裕 → 直接推荐 Transformer 路线（细节转 `ml-transformer-era`），本 skill 收尾说明 RNN 系落选原因后终止；短中小数据/需流式/资源受限 → 继续 RNN 系路线，进步骤 3。

3. **单元选型：普通 RNN / GRU / LSTM**
   - 默认排除普通 RNN（除非依赖极短且教学演示）；在 GRU 与 LSTM 之间按"先简后繁"原则建议：先 GRU 起步，效果不足再换 LSTM 对照。
   - 完成标准: 给出单元选择及理由（参数预算/依赖跨度/惯例）。

4. **结构出口与训练配置定型**
   - 按步骤 1 的定形接对应出口（多对一直接取末态或池化；异步任务配 seq2seq，若源序列较长必须加注意力）；训练侧注明 BPTT 截断长度、teacher forcing 及其暴露偏差风险、梯度裁剪必开（爆炸在时序方向更常见）。
   - 完成标准: 输出完整结构卡（单元/层数/双向与否/出口形态/截断长度/裁剪阈值）。
   - 判停条件: 若任务实为流式在线场景 → 双向结构禁用，改单向后行并重出结构卡。
   - 判停条件: 异步生成任务若用户后续要做大模型级生成 → 提示现代做法已由预训练 Transformer 接管（`ml-pretraining-paradigm`），避免重复造轮子。

5. **交接验证**
   - 训练故障（loss 不降/NaN）交 `ml-deep-training-playbook`；泛化不佳交 `ml-diagnosis`；效果达标则记录结构决策依据供复现。
   - 完成标准: 交付说明含"哪些问题归哪个下游 skill"的边界声明。
   - 判停条件: 若训练曲线健康但长序列尾部预测明显变差 → 门控有效跨度不足，回到步骤 2 重估依赖跨度并考虑加注意力/层级结构，而非盲目加单元数。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 用户在做跨族架构总选型（表格 vs 图像 vs 文本的宏观比较、scaling law 视角）→ `ml-transformer-era`；本 skill 只管序列一族的纵深。
- 无顺序依赖的数据硬套序列模型 → 步骤 1 已拦截；强行用 RNN 只是给模型增加无谓难度。
- 强化学习的序贯决策（游戏/机器人控制/推荐中的 agent 问题）→ `ml-rl-decision-loop`；"序列"二字重叠但问题本质不同。

### 文献反复警告的失败模式

- **训练不可并行的根本限制**: RNN 第 t 步依赖第 t−1 步的隐状态，时间维天然串行——训练无法像 Transformer 那样对整段并行计算，语料一大训练时长差距悬殊。这不是工程优化能根治的，而是结构固有属性；也是 Transformer 取代 RNN 系成为大规模预训练主干的根本原因之一。
- **长程依赖仍然有限**: 门控缓解了梯度消失，但没有消除——LSTM 的有效记忆跨度实测通常在数十至数百步量级（转述学界共识），"LSTM 能记住任意长的历史"是误读；真正超长依赖仍需注意力/层级结构辅助。
- **teacher forcing 的暴露偏差**: 训练时总以上一步真实值为条件、推理时却只能用自己生成的上一步，误差会滚雪球；scheduled sampling 等缓解手段各有代价，须在评估时用真实推理模式检验。
- **双向结构 ≠ 万能**: BiLSTM 看得到未来，离线任务好用；流式/在线场景不存在未来帧，双向直接不可用。
- **梯度爆炸被忽视**: 门控治的是消失；爆炸要靠梯度裁剪与合理初始化兜底，忘裁剪的时序训练动辄 loss 尖刺。

### 作者盲点 / 时代局限（外推声明）

- **必须声明**: 本 skill 主题超出西瓜书覆盖范围——西瓜书(2016)未把循环网络纳入正文体系；本 skill 为外推补全，素材为 Hochreiter & Schmidhuber (1997)、Cho (2014)、Sutskever (2014)、Bahdanau (2015) 及后续综述共识的转述，均标"（转述）"。
- 本 skill 的 RNN 系叙事截止于注意力崛起前夜；2017 年后 NLP 主干全面转向 Transformer，RNN 系的现代地位集中在小数据、流式、边缘与特定状态跟踪任务（近年 state-space 模型又在重写这条边界），执行时应查最新实证而非默认"RNN 已死"或"LSTM 仍是主流"任一极端。
- 时间序列预测领域另有大量统计方法（ARIMA/状态空间/梯度提升树）在与深度方法同台竞技，深度并非默认最优——选型前应先跑传统基线。

### 容易混淆的邻近方法论

- **Transformer 选型（`ml-transformer-era`）**: 总选型 vs 序列族纵深；两边在"序列任务要不要上 Transformer"处共管，分工见 A2。
- **强化学习（`ml-rl-decision-loop`）**: 序贯决策 vs 序列监督；奖赏信号的有无是分水岭。
- **1D 卷积做序列特征提取**: 属 CNN 归纳偏好在序列上的借用（局部性成立、长程依赖弱），可作为轻量基线，但不改变本 skill 对主干选择的裁决逻辑。

---

## 相关 skills

- depends-on: ml-task-matching（序列形态匹配的一般框架）
- contrasts-with: ml-transformer-era（族内纵深 vs 跨族总选型）, ml-rl-decision-loop（序列监督 vs 序贯决策）
- composes-with: ml-deep-training-playbook（训练故障接力）, ml-pretraining-paradigm（大生成任务的预训练路线）, ml-methodology-router

---

## 审计信息

- **来源性质**: 扩充批D·核心缺口——主题超出西瓜书(2016)覆盖范围，R 段为序列建模奠基文献的转述表述（均已标注"（转述）"）
- **验证通过**: 待流水线三重验证复核
- **测试通过率**: 待阶段 4 测试 (详见 test-prompts.json)
- **skill_version**: 0.0.1
- **蒸馏时间**: 2026-08-24
