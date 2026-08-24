---
name: ml-hpo-strategy
description: |
  超参搜索 (HPO) 算法选型手册。当用户要调超参、纠结网格/随机/贝叶斯优化(TPE)/Hyperband
  选哪种、问搜索空间怎么设计（对数尺度？哪些参数值得搜）、预算只够 N 次 trial 怎么分配、
  或搜索跑一半没起色想止损时激活。
  trigger: hyperparameter search/tuning 超参搜索、grid search 网格搜索、random search
  随机搜索、Bayesian optimization 贝叶斯优化、Optuna、Hyperband、调参预算止损。
  不适用于: 未做归因就开搜（先 ml-diagnosis，偏差主导时白搜）、实验记账
  （ml-experiment-tracking）、六问总审（ml-pitfall-audit）。
source_book: "工程实践共识（业界公开文献与社区共识，非书内内容）"
source_chapter: "ML工程实践共识: HPO策略（补全西瓜书工程落地盲点；书内f12仅给出折中原则）"
tags: [hyperparameter-optimization, bayesian-optimization, hpo, machine-learning]
related_skills:
  - slug: ml-diagnosis
    relation: depends-on
  - slug: ml-evaluation-design
    relation: depends-on
  - slug: ml-experiment-tracking
    relation: composes-with
---

# 超参搜索策略 — 把调参预算花在会还债的地方

## R — 原文 (Reading)

> **来源说明**: 本 skill 属批C"工程实践共识"系列——《机器学习》(西瓜书)对调参仅有 f12 一句
> 折中原则（见下），不含搜索算法选型；R 段改引业界公开文献并标注来源性质；凡无法保证逐字
> 精确处一律标（转述）。

> Random search works better than grid search when only a few dimensions influence performance, because it samples each dimension more finely for the same budget.（转述）
>
> — 转述自 James Bergstra & Yoshua Bengio, "Random Search for Hyper-Parameter Optimization", JMLR 2012（其思想可溯至 Lieberman 1958 对随机搜索优势的早期观察；来源性质：业界公认论文）

> Hyperband allocates budget by aggressively stopping unpromising configurations early and giving survivors more resources.（转述）
>
> — 转述自 Li 等, "Hyperband: A Novel Bandit-Based Approach to Hyperparameter Optimization", JMLR 2017（来源性质：业界公认论文）

### 书内锚点 (唯一相关原文)

> 参数候选值较少时可用密间距小步逼近……训练数据量大时可选取少量候选值……本质上是一种折中：计算开销与性能估计之间的折中。
>
> — 转述自周志华《机器学习》第2章 2.3节关于调参的讨论（工程界编号 f12）

---

## I — 方法论骨架 (Interpretation)

调参 = 在有限 trial 预算下做最优分配。四种主流策略各答一个不同的问题：

1. **网格搜索**: "每种组合都要吗？"——枚举全部笛卡尔积。只在维度 ≤3 且每维候选少时合理；高维下预算被均匀稀释在无关维度上。
2. **随机搜索**: 同样预算撒随机点。关键洞察是**重要维度通常很少**——随机采样让重要维度获得远多于网格的有效分辨率（网格把每个维度锁死在少数几个取值的乘积里）。零基础设施成本，永远是合格基线。
3. **贝叶斯优化 (TPE/SMAC/GP)**: 用历史 trial 结果建代理模型，下一trial采在"可能好"与"不确定"的平衡点。样本贵（每次训练几小时）时省 trial 数；但自身有开销、并行扩展差、且假设相邻配置表现平滑。
4. **Successive Halving / Hyperband**: 不学代理模型，改做**预算分配**——大量配置先给小额资源跑，砍掉一半差的，幸存者加倍资源，多轮淘汰。适合"早停信号可靠"的场景（epoch 数、subset 训练）。

配套纪律三条：**空间按参数性质设计**（学习率/正则强度跨数量级→对数尺度采样；离散结构类如树深→整数格点；且先问该参数是否真值得搜）；**验证集是唯一裁判**（对着测试集调参=测试集泄漏进训练过程）；**设止损线**（连续 N 个 trial 无提升即停，或预算烧完即报——不许"再试一次"无限续杯）。最终选定后须在全量数据上重训。

---

## A1 — 业界公开案例 (Past Application)

> 注: 西瓜书无对应案例（书中只有 f12 折中原则一句），本节用业界公开案例充当类比素材。

### 案例 1: Bergstra & Bengio 的对照实验 (JMLR 2012)
- **问题**: 神经网络超参众多，网格搜索在每个新维度上代价翻倍，实践中怎么搜才划算？
- **方法论的使用**: 在多个基准上以相同 trial 预算对比网格与随机搜索，并画出"分数 vs 各维度取值"的联合可视化。
- **结论**: 关键维度通常只有两三个；随机搜索在这些维度上的有效分辨率远高于等预算网格，整体显著占优。
- **结果**: 随机搜索成为 HPO 的默认基线，"网格 vs 随机"从此有了定量答案。

### 案例 2: Hyperband 与学习率的早期淘汰 (Li et al., JMLR 2017)
- **问题**: 深度模型每个 trial 都要训到底才知道好坏，预算根本不够撒网。
- **方法论的使用**: 利用"训练早期曲线就能区分好坏"这一规律做 successive halving：N 个配置各训少量 epoch → 砍半 → 幸存者加倍 epoch，循环。
- **结论**: 当存在可靠的早期性能代理时，预算分配视角比建模代理模型更便宜也更可并行。
- **结果**: ASHA/Hyperband 进入 Optuna、Ray Tune 等主流框架的默认选项，大规模调参的事实标准之一。

### 案例 3: 书内锚点的落地 (f12)
- **问题**: 西瓜书如何处理调参问题？
- **方法论的使用**: 书中指出调参本质是模型选择，需在"计算开销"与"性能估计质量"间折中——候选少而密则估得准但贵，候选少而疏则省算力但可能错过；选定后必须用全量数据重训。
- **结论**: 这条折中原则正是本 skill 全部算法选型的价值坐标：随机/贝叶斯/Hyperband 都是在不同预算约束下实现同一折中的具体方案。
- **结果**: 本 skill 把一句话原则扩展为可选型的工具箱，同时保留其"全量重训"交付纪律。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. 用户准备调参，问"GridSearchCV / Optuna / random search 该用哪个""贝叶斯优化是不是一定更好"。
2. 用户定义了搜索空间但拿不准尺度：学习率从 1e-5 到 1 该线性扫还是对数扫、树深要不要搜。
3. 用户预算有限（GPU 卡/时间固定），问"N 次 trial 怎么花最值""能不能提前砍掉没希望的配置"。
4. 用户的搜索已经跑了几十上百个 trial 还没起色，不知该继续烧还是收手换方向。
5. 用户发现"最优超参"每次搜索都不一样，怀疑调参本身过拟合了验证集。

### 语言信号 (用户的话里出现这些就应激活)

- "**调参**/**超参搜索**用什么方法"/"grid 还是 random"/"hyperparameter tuning strategy"
- "**贝叶斯优化**值得吗"/"Optuna/TPE/SMAC"/"Bayesian optimization"
- "**搜索空间**怎么设"/"学习率**对数**尺度"/"log scale sampling"/"search space design"
- "**Hyperband**/**successive halving**/早停淘汰"/"预算不够怎么分配"
- "调了这么久**还没提升**要不要继续"/"best config 每次都**不一样**"

### 与相邻 skill 的区分

- 与 `ml-diagnosis` 的区别: 那是先诊断主导项——若偏差主导（模型学不会），换搜索算法毫无意义，应先加特征/换模型族。本 skill 只在"归因完成、确认值得精修"之后接管。顺序铁律：先 diagnosis 开药方，再本 skill 定怎么搜。
- 与 `ml-evaluation-design` 的区别: 那是设计"谁来当裁判"（切分/度量/比较协议）；本 skill 是"怎么向裁判提交选手"。其嵌套交叉验证要求直接约束本 skill 的评估方式。
- 与 `ml-experiment-tracking` 的区别: 那是记账纪律（每次 trial 存了什么）；本 skill 是搜索决策。几十个 trial 的产出正需要 tracking 才能复盘。
- 书内 f12（ml-diagnosis 已收录为调参纪律）讲的是"折中+全量重训"的原则层；本 skill 是算法层的选型展开，二者不重叠。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 按**搜索选型决策流程**执行:

1. **前置闸门: 归因先行**
   - 向用户确认已做过欠拟合/过拟合归因（或代为快速判定）：当前瓶颈是容量不足（偏差）、不稳定（方差）还是纯精修空间？
   - 完成标准: 一句话写明"本次调参针对的主导项/目标指标"。
   - 判停条件: 若属偏差主导（训练误差也高）→ 停，移交 ml-diagnosis 处置（加特征/换模型族），搜索预算冻结。

2. **锁定裁判**
   - 确认验证方案（单一 holdout / K 折 / 嵌套 CV）与主指标已定，测试集封存未动过。
   - 完成标准: 写出"用 X 数据 + Y 指标作为唯一判据"；测试集已被反复使用 → 先走 ml-leakage-defense 清理。

3. **按预算和场景选算法**
   - 决策序: 维度 ≤3 且候选少 → 网格；默认起点 → 随机搜索（宽范围粗扫）；单 trial 昂贵且 trial 总数受限（≲50）→ 贝叶斯优化(TPE)；单 trial 可中途评估且有可靠早期信号 → Successive Halving/Hyperband。
   - 完成标准: 选型附一句理由（预算数 × 早停可行性 × 并行需求）。

4. **设计搜索空间**
   - 逐参数三问: 值得搜吗（对目标的预期影响）？什么尺度（跨数量级→对数；计数→整数格点；比例→[0,1]）？范围多大（先宽后窄两段式）？剔除伪参数与冗余参数。
   - 完成标准: 输出一张参数表（名称/类型/尺度/范围），总组合量与预算匹配。

5. **执行与止损监控**
   - 跑搜索，实时记录每 trial 的配置+分数+耗时（接 ml-experiment-tracking）；预设止损线: 如连续 10 trial 无超越、或预算烧掉 80% 时无改善趋势即停。
   - 完成标准: 止损条件在启动前已写入配置；触发时明确报告"停在第几个 trial、最好成绩是多少"。
   - 判停条件: 中途发现头部 trial 分数挤在一起且接近上限 → 可能撞噪声下界，停并回 ml-diagnosis 复核。

6. **收敛与交付**
   - 对 Top 区域细化（缩小范围二次搜索）→ 锁定 best config → **全量数据重训最终模型** → 封存的测试集只启用一次做终评。
   - 完成标准: 交付物含 best config、全量重训声明、一次性测试集终评数字；并提示"best config 有多稳"（邻域扰动/换种子的敏感性抽查）。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 归因未做就开搜：偏差主导的问题里，任何搜索算法都在弱模型族内空转；这是最高频的预算浪费。
- 目标是神经网络架构本身的设计（NAS 层面）：架构搜索是另一个量级的问题域，本 skill 只覆盖训练超参与少量结构超参（树深/层数）。
- 用户只是想要"某个具体参数的推荐默认值"：查框架文档默认值即可，不值得启动搜索流程。

### 业界警告的失败模式

- **HPO 过拟合验证集**: 几百个 trial 里挑出的 best config 部分是在拟合验证集的噪声——分数虚高，换个划分就缩水；重灾区是 trial 数大 + 验证集小的组合。
- **线性尺度搜学习率**: 学习率/正则强度跨数量级却用 linspace，预算全烧在 0.4~0.6 这种无人区，永远碰不到 1e-3 量级的甜点。
- **贝叶斯优化的隐性成本**: trial 又快又多（秒级）时，代理模型的拟合开销反超收益——此时随机搜索更划算。
- **对着测试集"看一眼"**: 每次 peek 都是把测试信息泄进选择过程；终评必须一次性。
- **忽略种子方差**: 单次 trial 定胜负会把运气当实力；重要结论前先抽几个种子确认不是彩票。

### 作者盲点 / 时代局限 (为何需要本 skill)

- 西瓜书的工程落地环节不在书内（BOOK_OVERVIEW 批判），对调参仅 f12 一句折中原则，无任何算法选型内容；本 skill 即对该盲点的补全。
- 成书于 2016 年前，贝叶斯优化成熟库（Optuna 2019、BoTorch）与 Hyperband(2017)/ASHA 生态均在其视野外；书中"精心挑选候选值"的手工世界观对应的是小规模经典模型时代。
- 书中未讨论"多次调参会过拟合验证集"这一现代共识问题，嵌套交叉验证需求需由本 skill 从业界文献补充。

### 容易混淆的邻近方法论

- **调参 ≠ 调优模型结构**: 学习率/正则是训练旋钮；层数/宽度等结构旋钮影响容量，改动前应回到 ml-diagnosis 归因。
- **AutoML ≠ 无脑黑盒**: AutoML 是本 skill 决策树的自动化封装，不理解选型逻辑就无法设定它的预算与空间上限。
- **早停 (early stopping) ≠ Hyperband 淘汰**: 前者是单个训练内的正则手段（防过拟合），后者是多配置间的预算分配策略；名字相近机制完全不同。

---

## 相关 skills

- depends-on: ml-diagnosis（归因闸门：偏差主导不开搜）, ml-evaluation-design（裁判方案与嵌套 CV 要求）
- contrasts-with: ml-diagnosis（搜索算法选型 vs 调参纪律原则/f12）
- composes-with: ml-experiment-tracking（trial 记账与止损复盘）, ml-leakage-defense（测试集封存与验证集过拟合核查）

---

## 审计信息

- **验证通过**: 批C·工程实践共识单元（书内仅 f12 提供原则锚点）；V1 ✓ / V2 ✓ / V3 ✓ 待阶段 3 统一复核
- **测试通过率**: 待阶段 3 测试 (详见 test-prompts.json)
- **skill_version**: 0.0.1
- **蒸馏时间**: 2026-08-24
