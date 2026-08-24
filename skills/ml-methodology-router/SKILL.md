---
name: ml-methodology-router
description: |
  ML 方法论问题总入口（Machine Learning Methodology Router），统辖 27 个子 skill 的决策方法合集。
  用户提出任何涉及机器学习决策/判断的问题时应最先激活本 skill：先判断其处于工作流哪一段
  （选型前 / 实验设计与数据准备 / 模型族内部施工 / 训练与调参 / 结果诊断 / 特殊场景 /
  大模型场景 / 工程交付），再派发到对应子 skill 并说明理由。触发信号："该用什么算法/模型"、
  "X 和 Y 哪个好"、"榜单第一为什么输"、"怎么设计实验/评估"、"过拟合了怎么办"、"效果不好"、
  "accuracy 99% 却没用"、"类别不平衡"、"冲榜/集成/融合模型"、"特征太多/降维"、"聚类分几类"、
  "贝叶斯/概率图/生成模型"、"loss 不降/NaN"、"warmup/Adam 还是 SGD"、"dropout 设多少"、
  "超参怎么搜/Optuna"、"结果复现不了/checkpoint 存什么"、"是不是有泄漏"、"多分类怎么拆"、
  "只有几百条标注"、"强化学习适不适合"、"要可解释的规则"、"Transformer 还是 CNN"、
  "要不要微调/LoRA"、"prompt 时好时坏"、"LLM 榜单可信吗"、"模型要上线"、"结果好得可疑"、
  "理论保证何时能信"；英文: "which model should I use", "how to evaluate", "overfitting",
  "class imbalance", "data leakage", "hyperparameter search", "reproducibility",
  "fine-tune vs prompt", "benchmark contamination", "too good to be true",
  "when can I trust the theory"。不路由：纯代码实现问题（如 sklearn 调参语法、报错栈、
  API 用法）——直接回答，不走本路由。
source_book: 《机器学习》周志华（批A 九个子 skill 原文锚定）；批B/批C 十八个子 skill 为时代局限的外推补全（经典文献共识/工程实践共识）
source_chapter: 全书（路由层，主锚点：第1章绪论、第2章模型评估与选择）
tags: [machine-learning, methodology, routing, meta-skill]
related_skills:
  # —— 选型前 ——
  - slug: ml-task-matching
    relation: composes-with
  - slug: ml-rl-decision-loop
    relation: composes-with
  # —— 实验设计与数据准备 ——
  - slug: ml-evaluation-design
    relation: composes-with
  - slug: ml-experiment-tracking
    relation: composes-with
  - slug: ml-feature-engineering
    relation: composes-with
  - slug: ml-leakage-defense
    relation: composes-with
  - slug: ml-semisupervised
    relation: composes-with
  # —— 模型族内部施工 ——
  - slug: ml-bayesian-thinking
    relation: composes-with
  - slug: ml-graphical-models
    relation: composes-with
  - slug: ml-rule-learning
    relation: composes-with
  - slug: ml-clustering-toolkit
    relation: composes-with
  - slug: ml-svm-playbook
    relation: composes-with
  - slug: ml-neural-training
    relation: composes-with
  - slug: ml-multiclass-strategies
    relation: composes-with
  # —— 训练与调参 ——
  - slug: ml-deep-training-playbook
    relation: composes-with
  - slug: ml-overfitting-modern
    relation: composes-with
  - slug: ml-hpo-strategy
    relation: composes-with
  # —— 结果诊断 ——
  - slug: ml-diagnosis
    relation: composes-with
  # —— 特殊场景 ——
  - slug: ml-imbalanced-learning
    relation: composes-with
  - slug: ml-dimensionality
    relation: composes-with
  - slug: ml-ensemble-design
    relation: composes-with
  # —— 大模型场景 ——
  - slug: ml-transformer-era
    relation: composes-with
  - slug: ml-pretraining-paradigm
    relation: composes-with
  - slug: ml-prompting-methodology
    relation: composes-with
  - slug: ml-llm-evaluation
    relation: composes-with
  # —— 工程交付 ——
  - slug: ml-pitfall-audit
    relation: composes-with
  - slug: ml-theory-compass
    relation: composes-with
---

# ML 方法论总控路由（ML Methodology Router）

## R — 原文 (Reading)

> 机器学习的本质是从有限样本出发在巨大假设空间中做选择以获得泛化能力，因此一切学习系统的设计都围绕"如何评估、如何选择、如何缓解过拟合"这三个问题展开。"泛化"——学得模型适用于没在训练集中出现的新样本的能力，是全书唯一最高目标。
>
> — 周志华《机器学习》第1章绪论主旨（据阶段 0 BOOK_OVERVIEW 整书理解提炼）

---

## I — 方法论骨架 (Interpretation)

本 skill 是二十七个子 skill 的调度器，自己不做任何具体分析。它做的事只有一件：在任何 ML 方法论问题被回答之前，先定位"用户此刻卡在工作流的哪一段"，再把问题交给管那一段的子 skill。

西瓜书的论证顺序就是这张路由图的来源：第1章先立根本目标（泛化）并证明"脱离任务谈算法优劣无意义"（NFL），第2章立刻建立评判武器库（评估方法 × 性能度量 × 比较检验 + 偏差-方差归因），之后各模型族、集成、降维、概率建模才围绕这条评估主线并列展开。也就是说：**先明确任务与目标，再谈怎么评，最后才轮到用什么模型**——用户的提问无论从哪句开始，都可以映射回这条主干上的某个位置。

批A 九个子 skill 原文锚定于西瓜书；批B/批C 十八个新子 skill 是对原书时代局限的外推补全：深度训练栈、HPO、实验可复现、特征工程、泄漏防御出自工程实践共识，Transformer/预训练范式/提示方法/LLM 评估出自经典文献共识——派发到这些 skill 时应保留其自带的时代边界声明。

八段工作流（路由表的分组依据）：
1. **选型前**——问题还没被刻画清楚，或正在争论"谁更强"、该不该进某个范式；
2. **实验设计与数据准备**——要跑实验、定度量、管数据、记过程；
3. **模型族内部施工**——已选定某族模型，需要该族的专属决策树；
4. **训练与调参**——训练动力学故障、正则配置、超参搜索；
5. **结果诊断**——已有结果但不符合预期，需要归因；
6. **特殊场景**——问题带有结构性特殊信号（不平衡/高维/集成）；
7. **大模型场景**——架构选型、范式档位、提示施工、LLM 能力评估；
8. **工程交付**——上线前审计、理论适用性横向审查。

---

## A1 — 书中的应用 (Past Application)

### 案例 1: 全书章节编排本身就是一次路由决策
- **问题**: 为什么一本机器学习教材在讲任何具体模型之前，先用整整一章讲"模型评估与选择"？
- **方法论的使用**: 周志华把第2章（评估三件套 + 偏差-方差分解）置于所有模型章（线性/决策树/神经网络/SVM/贝叶斯，第3–7章）之前——编排上的主张是：没有科学的尺子，就没有资格谈模型好坏。
- **结论**: 第2章成为全书枢纽，此后每章结尾都回扣"泛化性能"这条主线。
- **结果**: BOOK_OVERVIEW 骨架确认"1→2 提出根本困难→建立评判武器库，之后各方法族围绕评估主线并列铺开"。

### 案例 2: Boosting vs Bagging 选型消费的是第2章的语言
- **问题**: 第8章面对"该用串行还是并行集成"的选型问题时，需要一个归因标准。
- **方法论的使用**: 作者没有发明新标准，而是路由回第2章的偏差-方差分解：偏差主导的问题走串行 Boosting 逐轮聚焦错例，方差主导的问题走并行 Bagging/随机森林。
- **结论**: "Boosting 降偏差、Bagging 降方差"之所以成立，因为它消费的正是偏差-方差的诊断语言——两章互锁（verified 单元 f26 的证据链）。
- **结果**: 一个跨章路由把散落的章节连成了决策路径，这正是本 skill 对九个子 skill 做的事。

### 案例 3: 同一数学的两种面目——再缩放与代价敏感同源
- **问题**: 第3章讲类别不平衡的再缩放（按 m⁻/m⁺ 校正判决），第2章讲非均等代价的代价敏感学习，读者要不要学两遍？
- **方法论的使用**: 识别出二者是同一公式：把再缩放比值中的 m⁻/m⁺ 换成 cost₊/cost₋，不平衡处理即刻变成代价敏感学习。
- **结论**: 表面不同的"场景"实为同构问题，应路由到同一个数学内核。
- **结果**: 本路由表中 ml-imbalanced-learning 与 ml-evaluation-design（代价敏感分支）的串联关系即源于此。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. **选型争论**: 团队在吵"微调小模型 vs 大 API"、"XGBoost 还是深度网络"、"某某榜单第一为什么在我们数据上不行"、"要不要用强化学习"。
2. **实验开工前**: 用户要做对比实验、写论文汇报结果、设计 A/B 或离线评估，不知道留出还是交叉验证、用 accuracy 还是 F1、seed 和配置怎么管。
3. **数据准备与施工**: 拿到新表不知道各列怎么处理、只有几百条标注想榨干未标注数据、已选定某族模型需要该族的专属决策树。
4. **训练故障与调参**: "loss 不降/NaN"、纠结初始化归一化 warmup、dropout 设多少、超参搜索怎么选算法、预算只够 N 次 trial。
5. **结果不如意**: "模型效果不好"、"好像过拟合了"、"验证集掉点了"，需要系统归因而不是瞎调。
6. **上线/信任关口**: 新模型准备上线、"结果好得可疑"、线下线上对不上，需要审查隐藏前提或做泄漏排查。
7. **大模型场景**: "Transformer 还是 CNN"、"要不要微调/LoRA"、"prompt 时好时坏"、"LLM 榜单可信吗"。

### 语言信号 (用户的话里出现这些就应激活)

- "该用什么算法/模型"、"哪个模型更好"、"SOTA/榜单"、"RL 能不能做 X"（→ 选型前）
- "怎么设计实验"、"怎么评估"、"数据该怎么划分"、"用什么指标"、"复现不了"、"特征这列怎么处理"（→ 实验设计与数据准备）
- "SVM 参数怎么选"、"贝叶斯网结构"、"HMM 还是 CRF"、"聚类分几类"、"要可解释的规则"（→ 模型族内部施工）
- "loss 不降/NaN"、"warmup 要不要加"、"dropout 放哪层"、"grid 还是 random search"（→ 训练与调参）
- "效果不好"、"过拟合/欠拟合"、"掉点了"、"学习曲线封顶"（→ 结果诊断）
- "99% 精度"、"不平衡"、"冲榜"、"特征太多"（→ 特殊场景）
- "Transformer 还是 CNN"、"LoRA 还是全参"、"prompt 时好时坏"、"benchmark 可信吗"（→ 大模型场景）
- "要上线了"、"结果好得可疑"、"是不是有泄漏"、"理论上说"（→ 工程交付）

### 与相邻 skill 的区分

- 本 skill 与九个子 skill 是**父子关系而非竞争关系**：router 只负责定位与派发，绝不亲自展开分析；一旦定位完成，控制权移交对应子 skill。
- 与 `ml-pitfall-audit` 的区别: pitfall-audit 是可被路由到的终点之一，同时也是唯一会被 router **主动追加**的横向检查（凡"即将上线/结果可疑"信号出现，即使已路由到其他 skill 也要追加提示）。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 应按以下步骤执行:

### 步骤 1: 边界回退检查（先做，最快）

- 动作: 判断问题是否为**纯代码实现/语法问题**——sklearn/PyTorch 等库的参数怎么填、报错栈解读、API 用法、画图代码。
- 完成标准: 二选一——是 → 不走路由，直接回答并结束；否 → 进入步骤 2。
- 注意: "调参语法"不路由，但"该调哪些超参、为什么"属于方法论，要路由。

### 步骤 2: 阶段定位

- 动作: 把用户问题映射到八段工作流之一——选型前 / 实验设计与数据准备 / 模型族内部施工 / 训练与调参 / 结果诊断 / 特殊场景 / 大模型场景 / 工程交付。
- 完成标准: 能用一句话说出"用户处于 X 阶段，卡点是 Y"。说不出卡点就先向用户澄清，禁止盲派。

### 步骤 3: 特殊信号扫描

- 动作: 扫描九类结构性信号——①类别不平衡 ②高维/特征过多 ③集成/冲榜 ④概率建模/生成式 ⑤上线前/结果可疑 ⑥理论保证疑问 ⑦训练动力学故障(loss 不降/NaN) ⑧泄漏嫌疑(线下线上不一致/too good to be true) ⑨大模型范式疑问(fine-tune/prompt/benchmark)。
- 判停条件: 任一信号命中 → 直接跳到步骤 4 派发对应专项 skill（信号优先级高于阶段归属，因为它们有专属处置流程）。

### 步骤 4: 查路由表派发

#### ① 选型前

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| 问"该用什么算法/模型"、争论"X 比 Y 强"、拿榜单说事 | 任务尚未刻画清楚，或结论试图脱离任务谈优劣 | `ml-task-matching` | NFL 定理：脱离具体问题谈算法优劣无意义；必须先匹配任务特性与归纳偏好，再谈算法 |
| 问"强化学习适不适合做 X"、reward 怎么设计、agent 训练不动 | 疑问指向序贯决策范式资格（延迟奖赏+动作影响后续局面缺一不可） | `ml-rl-decision-loop` | RL 与监督学习的分界在资格验证：有标签静态数据走选型，交互轨迹才进 RL；范式内路线（有模型 DP vs 免模型 MC/TD、探索利用折中）自成决策树 |

#### ② 实验设计与数据准备

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| 要跑实验/比较模型/设计评估/选指标/"提升 0.5% 显著吗" | 结论将来自一次尚未设计的测量 | `ml-evaluation-design` | 泛化误差不可直接观测，评估方案 × 性能度量 × 比较检验三层共同决定结论可信度 |
| 复现不了结果、seed/配置/checkpoint 怎么管、实验归档 | 疑问指向过程记账而非结论好坏 | `ml-experiment-tracking` | 每个数字都要能回答"从哪来"：种子、配置版本化、环境快照、续训内容是可信度的地基 |
| 新数据集各列怎么处理、编码/标准化/分箱、滞后特征会不会偷看未来 | 问题发生在"从原始字段造特征池"的加法环节 | `ml-feature-engineering` | 先切分→体检→逐列决策的处理链；与 dimensionality 的"挑选"分界：先构造后挑选 |
| 怀疑线下虚高线上暴跌、变换器该在切分前还是后 fit、时序能否 shuffle | 效果"太好"且不可信，或管道存在信息穿越嫌疑 | `ml-leakage-defense` | 三大泄漏类型定位+检测信号+上线清单的专项审计；与 pitfall-audit 分界：专项深挖 vs 六问总审 |
| 只有几百条标注想用大量未标注数据、pseudo-label 掉点、协同训练前提 | 标注稀缺且未标注同源，疑问指向桥梁假设的安全性 | `ml-semisupervised` | 资格线→假设验证→四路线选型→强制纯监督基线对照；模型假设不准时未标记样本会反噬 |

#### ③ 模型族内部施工

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| 朴素贝叶斯/EM/GMM 的分布与独立性假设、隐变量估计 | 单分类器的概率建模纪律问题 | `ml-bayesian-thinking` | MLE 猜错分布全盘误导；独立性谱系（朴素→半朴素→贝叶斯网）按数据量选档；EM 多初值+推断降级路径成套 |
| 多变量依赖怎么建图、HMM 还是 CRF、变分还是 MCMC、推断跑不动 | 问题指向多变量网络的图结构与推断路线 | `ml-graphical-models` | 有向/无向判关系性质→结构定源→推断决策树（精确可行性看团规模）；与 bayesian-thinking 分界：单模型假设纪律 vs 多变量网络 |
| 要可解释的 if-then 规则、命题够不够、默认规则频繁触发 | 可解释是硬需求且已锁定规则路线 | `ml-rule-learning` | 命题 vs 一阶判定→搜索方向选择→剪枝与后处理防贪心锁死；与 graphical-models 分界：确定性规则 vs 概率软依赖 |
| 聚类分几类、kmeans 还是 dbscan、轮廓系数怎么看、簇当类别用行不行 | 无标记数据的分组与评估全流程 | `ml-clustering-toolkit` | 外部/内部指标→按形状选范式→按属性有序性选距离→k 值扫描+语义复核；警告聚类没有客观标准、簇≠真实类别 |
| SVM 参数怎么选、选哪个核、完美可分沾沾自喜、大数据硬跑非线性核 | 已选定 SVM 后的专属构造性决策树 | `ml-svm-playbook` | 默认软间隔（完美可分可能是过拟合假象）、文本用线性核、核选择是最大变数、非线性核 O(m²) 判死大数据 |
| NN 不收敛/震荡/局部极小/几个隐藏层/早停时机——经典浅层 BP 语境 | 训练动力学故障但属于 Sigmoid+BP trick 体系 | `ml-neural-training` | 可分性定结构下限（XOR 数学边界）→loss 曲线分诊→多组初值/SGD 跳局部极小→早停+正则两手 |
| 二分类器解决 N 类问题、"ovo ovr 区别"、ECOC 码本设计、万类仍套 OvO | 基分类器不支持原生多分类，需外接拆解架构 | `ml-multiclass-strategies` | 开销矩阵选 OvO/OvR/MvM-ECOC；大类数平方开销提前判死；拆解任务的失衡通常相互抵消 |

#### ④ 训练与调参

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| 深度网络 loss 不降/NaN/发散、初始化 Xavier 还是 He、BN 还是 LN、warmup、梯度消失爆炸 | 架构已定后的训练动力学故障，属深度现代训练栈 | `ml-deep-training-playbook` | "数据管道→小样本过拟合→初始化归一化→学习率→优化器调度→梯度监控"固定排查链，先廉价后昂贵每步带完成标准；与 neural-training 分界：经典 BP 小网络归那边 |
| Dropout 放哪层/比例、weight decay 与 L2 区别、正则叠加打架、"训练 loss=0 但泛化好" | train-val gap 已出现后的深度正则工具箱与双下降甄别 | `ml-overfitting-modern` | 插值区间的新形态要求重新标定经典红旗直觉；先单手段消融再组合；与 deep-playbook 分界：优化端 vs 泛化端 |
| grid/random/贝叶斯优化选哪个、搜索空间怎么设、N 次 trial 预算分配、止损 | 归因已完成、确认值得精修后的搜索算法选型 | `ml-hpo-strategy` | 偏差主导时开搜是最高频预算浪费（先 diagnosis）；对数尺度/Hyperband 早停淘汰/调参过拟合验证集的甄别 |

#### ⑤ 结果诊断

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| 说"效果不好/过拟合/欠拟合/掉点"、"该加数据还是加模型"、训练精度 100% 是不是好事 | 已有结果，需要归因而非动作 | `ml-diagnosis` | 偏差-方差分解给出系统归因：先诊断主导项（偏差/方差/噪声）再开药方；"太好了也是病"的红旗换尺 |

#### ⑥ 特殊场景

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| 类别极度不平衡、"accuracy 99%" 却无用、SMOTE/阈值移动 | 少数类占比悬殊使均等代价度量失守 | `ml-imbalanced-learning` | accuracy 在此失效是高频事故；再缩放三路线须按部署约束反推 |
| 高维数据/特征选择/降维/kNN 失效/p≫n/矩阵补全 | 维数远大于有效样本或距离度量可疑 | `ml-dimensionality` | 维数灾难是一切方法的共同障碍，症状→路线（降维/特征选择/度量学习）有成套决策树；与 feature-engineering 分界：挑选 vs 构造 |
| 集成学习、竞赛冲榜、堆模型、stacking 线上线下不一致 | 打算用多个学习器组合 | `ml-ensemble-design` | 收益来自"好而不同"：多样性制造是核心技术活，盲目堆数量白付算力 |

#### ⑦ 大模型场景

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| Transformer 还是 CNN/RNN、QKV 直觉、位置编码有什么用、小数据能上大架构吗 | 自训骨架的架构选型（数据形态×规模×算力三轴） | `ml-transformer-era` | 小表格 GBDT 常胜神经网络、小数据 CNN/RNN 仍是正解、大规模序列才轮到 Transformer；SOTA 搬运前先校验规模前提 |
| 要不要自己训/微调还是 API、LoRA 还是全参、灾难性遗忘、多少数据能微调 | 疑问指向范式档位升降级而非档位内施工 | `ml-pretraining-paradigm` | 从零训练/全参微调/PEFT/纯提示四档选最低够用档；数据<1k 禁止直接微调；微调后必做通用能力回归测试 |
| few-shot 示例怎么选、CoT 该不该加、输出格式不稳、prompt 改来改去时好时坏 | 已确定留在提示档内部的单应用开发态迭代 | `ml-prompting-methodology` | 回归测试集驱动迭代、禁止无评估的玄学改写、CoT 只在多步推理启用、结构化输出靠 schema；与 pretraining-paradigm 分界：档位内 vs 档位间 |
| LLM 榜单排名和实测不一致、测试集是不是被污染、GPT 打分靠谱吗、人工评估协议 | LLM 能力的科学度量（选型态/验收态），通用评估三层假设失效处 | `ml-llm-evaluation` | benchmark 分布鸿沟+NFL 重演、三探针查污染、LLM-as-judge 校正三类偏差、人工评估 rubric 与一致性校验 |

#### ⑧ 工程交付

| 用户情境/信号 | 判断依据 | 派发到 | 为什么派到这里 |
|---|---|---|---|
| 新模型即将上线、结果好得可疑、投稿/交接前 sanity check | 决策即将不可逆，或指标异常漂亮 | `ml-pitfall-audit` | 训练精度满分可能是红旗；隐藏前提（无偏采样/误差独立/分布形式/i.i.d.）被打破时结论反转；六问总审，泄漏专项存疑再转 leakage-defense |
| 想知道理论保证何时适用、"理论上说应该"、"需要多少数据才够" | 疑问指向保证的条件与松紧 | `ml-theory-compass` | PAC/VC/Rademacher/稳定性给出理论承诺的适用条件与刻度，防止误用修辞 |

- 完成标准: 输出包含——派发目标 skill 名 + 一句"为什么派到那里"（引用上表判断依据列）。

### 步骤 5: 串联检查

- 动作: 判断是否需要第二个 skill 接力。书内验证过的常见链路:
  - `ml-diagnosis` 判定方差主导 → 追加 `ml-ensemble-design`（Bagging 降方差）
  - `ml-evaluation-design` 完成、且模型要交付 → 追加 `ml-pitfall-audit`
  - `ml-task-matching` 中用户援引理论保证 → 追加 `ml-theory-compass`
  - `ml-evaluation-design` 发现不平衡 → 移交 `ml-imbalanced-learning`
  - `ml-imbalanced-learning` / `ml-ensemble-design` 的隐藏前提疑问 → 回接 `ml-pitfall-audit`
- 批B/批C 扩展后的高频链路:
  - `ml-diagnosis` 确认值得精修（非偏差主导）→ 移交 `ml-hpo-strategy` 定怎么搜
  - `ml-feature-engineering` 构造完成、交付前 → 追加 `ml-leakage-defense` 审管道
  - `ml-transformer-era` 结论是"不必自训骨架" → 移交 `ml-pretraining-paradigm` 选档位；判停档位 0 → 再移交 `ml-prompting-methodology`
  - `ml-prompting-methodology` 回归测试显示到达提示档天花板 → 回接 `ml-pretraining-paradigm` 升档
  - `ml-pretraining-paradigm` 微调收益验收 → 追加 `ml-llm-evaluation` 的污染检查与回归测试纪律
  - `ml-deep-training-playbook` 训练正常但 gap 出现 → 移交 `ml-overfitting-modern`
  - `ml-neural-training`（经典语境）与现代器件问题并存时 → 按时代分界分流 `ml-deep-training-playbook`
  - `ml-experiment-tracking` 排除过程失控后仍"结果对不上/不好" → 移交 `ml-diagnosis`
  - `ml-pitfall-audit` 六问中泄漏一问存疑 → 深挖转介 `ml-leakage-defense`
- 完成标准: 要么给出至多 2 个 skill 的接力顺序，要么明确声明"单 skill 足够"。
- 判停条件: 单次派发严禁超过 2 个子 skill——超过说明问题该拆分成多次对话。

---

## B — 边界 (Boundary)

### 不要在以下情况使用此 skill

- **纯代码实现问题**（sklearn 怎么调参语法、报错信息、绘图代码）: 这些是检索/调试问题，路由只会拖延回答。直接作答。
- **纯数学推导作业**（"帮我推 SVM 对偶"）: 属于演算而非方法论决策，除非用户追问推导背后的取舍。
- **非 ML 话题**: 本合集不覆盖常规软件开发、数据清洗脚本等。
- **二十七个 skill 都不覆盖的领域**: 分布式训练工程（多卡通信/数据管道死锁）、NAS 架构搜索的完整问题域、安全红队评估（越狱/偏见专项）、因果推断、RAG 系统端到端设计、MLOps 线上监控告警——明确告知超出本合集覆盖范围，按通用工程常识回答并注明"此部分不出自西瓜书/经典文献共识"，禁止假装路由。
- **分布漂移与域适应**: 训练与线上数据分布本身随时间变化不是泄漏也不是诊断问题，处置方向是监控与再训练——本合集各 skill 均基于 i.i.d. 或近 i.i.d. 假设，此类提问只做边界声明不做处方。

### 作者在书中警告的失败模式

- 脱离具体问题空谈算法优劣（NFL 寓意，第1章）——路由本身也不能代替任务刻画。
- 用训练误差代替泛化误差做决策（第2章）——任何"效果好"的断言先问测量方式。
- "彻底消灭过拟合"式的宣称（若可避免则 P=NP）——对任何声称保持怀疑并要求验证机制。

### 作者的盲点 / 时代局限（路由结论需打折的区域）

- 成书于 2015–2016: 深度学习仅简述，Transformer、预训练-微调、scaling law、few-shot 评测完全缺失——批B/批C 十八个子 skill 正是对这一时代局限的外推补全（经典文献共识/工程实践共识），派发到它们时结论需按各自 B 段声明打折。
- i.i.d. 假设贯穿全书: 分布漂移、域适应场景下评估体系的根基动摇（李未院士序言已专门质疑）。
- 工程落地环节薄弱: 数据质量管理、线上-线下一致性、A/B 上线流程不在书中——experiment-tracking/leakage-defense/hpo-strategy 三个子 skill 是工程共识补全，非原书内容。
- 教材体例约束: 书中方法从不讨论"多技巧组合时的相互作用"，而真实项目方法从不孤立出现——这正是本 router 串联步骤存在的理由，但也意味着串联建议是合理外推而非原文。
- 大模型方法论快速演化: pretraining-paradigm/prompting-methodology/llm-evaluation 的具体器件（PEFT 变体、judge 校正手段）仍在快速迭代，执行时以最新实证为准。

### 容易混淆的邻近方法论

- 不要把 router 当成"ML 百科问答入口"——知识查询（"什么是 SVM"）不需要路由，直接答；只有**决策/判断类**问题才路由。
- 不要因为问题里出现"模型"二字就激活——"帮我把这个 ONNX 模型部署到手机"是工程问题。

---

## 相关 skills

- dispatches-to（全部二十七个子 skill，按八段工作流分组，见 frontmatter related_skills）:
  - 选型前: `ml-task-matching`, `ml-rl-decision-loop`
  - 实验设计与数据准备: `ml-evaluation-design`, `ml-experiment-tracking`, `ml-feature-engineering`, `ml-leakage-defense`, `ml-semisupervised`
  - 模型族内部施工: `ml-bayesian-thinking`, `ml-graphical-models`, `ml-rule-learning`, `ml-clustering-toolkit`, `ml-svm-playbook`, `ml-neural-training`, `ml-multiclass-strategies`
  - 训练与调参: `ml-deep-training-playbook`, `ml-overfitting-modern`, `ml-hpo-strategy`
  - 结果诊断: `ml-diagnosis`
  - 特殊场景: `ml-imbalanced-learning`, `ml-dimensionality`, `ml-ensemble-design`
  - 大模型场景: `ml-transformer-era`, `ml-pretraining-paradigm`, `ml-prompting-methodology`, `ml-llm-evaluation`
  - 工程交付: `ml-pitfall-audit`, `ml-theory-compass`
- 组合约定: router 是唯一入口；子 skill 之间不得互相调用，接力一律经由 router 的步骤 5 串联规则。
- 共享术语表: 见仓库根目录 `GLOSSARY.md`（西瓜书 62 条核心术语 + 批B/批C 现代术语增补）。

---

## 审计信息

- **验证通过**: V1 ✓ / V2 ✓ / V3 ✓（router 无独立验证单元；由阶段 1.5 通过的 39 条框架单元 + 53 条反例/案例素材按 skill_groups.json 组装；批B/批C 扩容后路由表 27 行经交叉核对各子 skill frontmatter 与 A2/B 段）
- **测试通过率**: 待阶段 4 压力测试（test-prompts.json 已生成，9 条/文件）
- **skill_version**: 0.0.1
- **蒸馏时间**: 2026-08-24
