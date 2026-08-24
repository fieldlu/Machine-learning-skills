---
name: ml-automl-nas
description: |
  AutoML 与 NAS 适用判断：自动化的是选择环节，数据质量仍是人的责任。用户问"要不要用
  AutoML/NAS 怎么回事"、纠结搜索空间设计、少 epoch 代理评估选出的架构训完不对劲、大模型
  时代还要不要自跑 NAS 时激活。动作: 划算性判定→NAS 三要素→代理偏差校准→留出终审。
  与 HPO 分界: HPO 调参数，NAS 搜架构本身。
  不适用于: 已定架构调参(ml-hpo-strategy)、数据质量排查(ml-leakage-defense)。
  trigger: AutoML, NAS neural architecture search, DARTS, supernet
source_book: 经典文献共识（Elsken NAS 综述 2019; Zoph & Le NAS 2017; Feurer Auto-Sklearn 2015; Cai COTS 2019; Li Hyperband 2017）
source_chapter: "经典文献共识(2015-2019): NAS三要素/代理指标偏差/COTS成本现实/AutoML适用边界"
tags: [automl, nas, hyperparameter-optimization, architecture-search]
related_skills:
  - slug: ml-hpo-strategy
    relation: contrasts-with
  - slug: ml-leakage-defense
    relation: depends-on
  - slug: ml-task-matching
    relation: composes-with
  - slug: ml-methodology-router
    relation: composes-with
---

# AutoML 与 NAS — 自动化的是选择，不是判断

## R — 原文 (Reading)

> **来源说明**: 本 skill 属扩充批D——主题超出西瓜书覆盖范围（西瓜书 f12 仅一句调参折中原则），R 段改引 AutoML/NAS 奠基文献并标注来源性质；凡无法保证逐字精确处一律标（转述）。

> Neural architecture search is defined by three components: a search space (which architectures can be represented), a search strategy (how the space is explored), and a performance estimation scheme (how candidate architectures are measured).（转述）
>
> — 转述自 T. Elsken 等《Neural Architecture Search: A Survey》(JMLR 2019), 第1节（来源性质：综述公认表述）

> Training each candidate to convergence is prohibitively expensive; methods therefore use fewer epochs, smaller proxies, or weight sharing—but these lower-fidelity estimates introduce bias and noise into the search.（转述）
>
> — 转述自 T. Elsken 等 (JMLR 2019) 第4节；另见 B. Zoph & Q. Le《Neural Architecture Search with Reinforcement Learning》(ICLR 2017) 中以并行训练与早停控制成本的实践（来源性质：奠基论文公认表述）

---

## I — 方法论骨架 (Interpretation)

AutoML 的本质是把"人肉试错做选择"换成"机器在定义好的空间里搜"。它自动化的只是**选择环节**，而选择的前提——空间怎么定、数据是否干净、目标是什么——仍然是人的责任。

- **AutoML 解决什么**: 表格任务的 pipeline 组合（特征预处理×模型族×超参的三重选择，如 Auto-sklearn/TPOT 的贝叶斯优化+元学习+集成）；超参搜索自动化（Optuna/Hyperband）；以及最深的一档——架构本身的搜索(NAS)。共同前提是：**候选空间有限且可枚举/可采样，单次评估可自动化，评价目标单一明确**。
- **NAS 三要素**: ①搜索空间——允许哪些结构存在（层数范围/算子集合/连接方式）；空间太窄搜不出超越设计者的东西，太空宽则搜索成本指数爆炸；②搜索策略——在空间里怎么走（强化学习/进化算法/贝叶斯优化/可微松弛 DARTS 类）；③性能估计——怎么给每个候选拍板。三要素互相牵制：空间决定策略可行性，估计精度决定策略能否学到真信号。
- **代理指标陷阱**: 完整训练每个候选不可承受，于是用少 epoch/小数据子集/权重共享(one-shot/supernet)来估性能——但这些低保真信号有系统性偏差：早停分数与最终收敛性能的相关性并不保证，权重共享下各候选互相干扰导致排名失真。**用代理指标选出的 top-k 应作为短名单，而非最终答案**。
- **成本现实（COTS/GTNAS 的教训）**: 早期 NAS 动辄数百 GPU 天，远超它要替代的人工设计成本；后续工作(COTS 用曲线外推预测最终精度、GTNAS 引入迁移学习)把成本压低数个量级——但结论是：NAS 的性价比只在特定窗口成立。
- **什么时候划算**: 特征规整的表格任务上 AutoML 常是高性价比默认（人类调 pipeline 的时间更贵）；小到中型架构设计且有充足算力余量时 NAS 可作探索工具。**大模型时代的天平整体倒向另一边**：预训练模型的架构已被巨额算力搜过一遍，下游正确动作是微调/提示（`ml-pretraining-paradigm`），自己跑 NAS 多半是在重新发明轮子的劣化版。

一句话：**先判划算性（人的时间 vs 机的算力）→ 再定三要素 → 全程校准代理指标偏差 → 数据质量与问题定义永远留在人手里。**

---

## A1 — 文献中的经典应用 (Past Application)

*（本批主题超出西瓜书覆盖范围，A1 改引奠基文献的经典案例，均为学界公认的原始工作叙述）*

### 案例 1: Zoph & Le 的 RL 搜索 CIFAR-10 架构
- **问题**: 新任务该用什么网络结构，靠研究者直觉+消融试错，周期长且依赖经验；作者想验证"架构设计本身可以变成一个可优化的决策过程"。
- **方法论的使用**: 把生成架构描述的过程建模为 RNN 策略的动作序列（每步输出一个层的超参），验证集精度为奖励，策略梯度更新 RNN——即用 RL 在定义好的搜索空间里找高分结构。
- **结论**: 搜索得到的架构在 CIFAR-10 上达到与当时人工设计 SOTA 可比的水平（转述），证明"机器能搜出好架构"这一命题成立。
- **结果**: 该工作确立 NAS 问题框架（空间/策略/估计三件套由此定型），但其代价同样著名：约 800 块 GPU 训练 28 天（转述），直接催生了后续全部降本研究。

### 案例 2: COTS 用曲线外推砍掉大部分训练成本
- **问题**: NAS 的主要开销在每个候选都要训练；多数平庸候选训练到一半就已注定无望，全量训练纯属浪费。
- **方法论的使用**: Klein 等的 COTS 让每个候选只训练少量 epoch，用学习曲线外推(加权概率模型)预测其最终精度，据此提前淘汰无望者、把预算集中给有希望的候选。
- **结论**: 性能估计不必等完整训练——低保真+智能外推可以在保持搜索质量的同时把总成本降低一个数量级以上（转述其报告幅度）。
- **结果**: 曲线外推/早停淘汰成为 NAS 与 HPO 共用的标准省钱手段（Hyperband 同属此思路家族），也是"代理指标偏差必须显式管理"这一纪律的直接来源。

### 案例 3: Auto-sklearn 的表格任务自动化
- **问题**: 表格任务上"选预处理器×选模型×调超参"的组合空间巨大，非专家用户既没知识也没时间逐项试。
- **方法论的使用**: Feurer 等把贝叶斯优化（在组合空间搜配置）、元学习（从历史任务的相似度热启动）、集成构造（把搜索中表现尚可的多个模型加权融合）三者叠加，全自动交付最终 pipeline。
- **结论**: 对特征规整的中小规模表格任务，自动化 pipeline 的开箱成绩稳定优于未经调优的单模型基线（转述其论文对照结论），AutoML 在此场景的价值主张成立。
- **结果**: Auto-sklearn 及同代工具(TPOT/H2O AutoML)成为表格 AutoML 的代表；"表格任务值得先跑一遍 AutoML 当强基线"成为广泛实践。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. 用户拿到一个表格任务想偷懒："有没有自动化的工具帮我选模型和调参""AutoML 靠谱吗"。
2. 用户听说 NAS 想跟风："能不能用 NAS 给我的任务搜个网络""DARTS 怎么用"，需要被评估是否划算。
3. 用户在设计搜索实验时卡在三要素：搜索空间怎么圈、用 RL 还是贝叶斯还是进化、每个候选训多少 epoch 才可信。
4. 用户发现按代理指标（少 epoch/代理数据集）选出的最优架构完整训练后表现平平甚至不如随机候选——代理偏差事故现场。
5. 大模型语境下的反向咨询："我还需要自己做 NAS 吗"——需要有人告诉他不用的理由和例外。

### 语言信号 (用户的话里出现这些就应激活)

- "**AutoML**/**automl 工具**""AutoSklearn/**TPOT**/H2O/**NNI** 选哪个"
- "**NAS**/**neural architecture search**/架构搜索""**DARTS**""one-shot/**supernet** 权重共享"
- "**搜索空间**怎么定/search space design""搜索策略 RL 还是进化/贝叶斯"
- "少 epoch 估计不准/**代理指标**偏差""搜出来的架构训完不行/proxy ranking 失真"
- "要不要自己搜架构""大模型时代还需要 NAS 吗"

### 与相邻 skill 的区分

- 与 `ml-hpo-strategy` 的区别（本批最关键的分界）: **HPO 是给定架构后搜超参配置；NAS 是连架构本身一起搜**。用户问 Optuna/Hyperband 怎么分配调参预算 → hpo-strategy；问搜索空间里放不放"层数/算子/连接"这类结构维度 → 本 skill。实践中 NAS 的性能估计大量借用 HPO 的省钱手段（Hyperband 式早停），两 skill 在工具层衔接但在问题上互斥。
- 与 `ml-leakage-defense` 的区别: AutoML 会放大而不是消除数据质量风险——嵌套的自动化选择更容易把验证信息漏进管道；防漏纪律归 leakage-defense，本 skill 只负责指路并回扣。
- 与 `ml-task-matching` 的区别: 那是人做的第一性选型检查；AutoML 是把这个选择过程交给机器——但机器的选择空间仍是人划定的。决定"要不要外包选型"时两边都要出场。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 应按以下步骤执行:

1. **划算性判定（永远的第一步）**
   - 估算两侧成本: 人工侧（团队对这类任务的经验、手工选型+调参预计工时）；机器侧（可用算力×预期搜索时长）。同时核对时代默认: 若任务是 LLM 相关，先查有无现成预训练模型可直接用。
   - 完成标准: 输出"划算/不划算/有更优路线"三选一判定及两侧成本依据。
   - 判停条件: 任务可用预训练模型解决 → 推荐微调/提示路线（转 `ml-pretraining-paradigm`），明说"自跑 NAS 不划算"的理由后终止；判定为表格任务且人工经验薄弱 → 进步骤 2 走 AutoML 路线；确需自定义架构且算力充裕 → 进步骤 3 走 NAS 路线。

2. **AutoML 工具路线（表格类）**
   - 选型: 通用表格首选成熟框架（Auto-sklearn/TPOT/H2O AutoML 一类）；已有 Optuna 基础设施则在其上扩 pipeline 搜索。
   - 完成标准: 工具选定并跑通一次端到端搜索，产出"自动化 pipeline vs 朴素基线"的对照结果。
   - 判停条件: 对照中朴素 GBDT+手调已接近自动化结果 → 采纳简单方案收尾，不再加码自动化复杂度。

3. **NAS 三要素设计**
   - 圈定搜索空间（层数/算子/连接的范围，写明上限与理由）；选搜索策略（预算小→随机/贝叶斯，预算中→进化或 Hyperband 式资源分配，需可微端到端→DARTS 类并知其稳定性争议）；定性能估计方案（低保真档位+外推/早停规则）。
   - 完成标准: 三要素各有一段书面定义，且互相兼容性检查通过（如空间大小与策略样本预算匹配）。
   - 判停条件: 算力预估超过用户承受线且无法压缩 → 回步骤 1 重议路线，禁止硬着头皮开搜。

4. **代理指标偏差校准（NAS 路线必经）**
   - 抽取若干代理排名靠前与靠后的候选做完整训练对照，计算代理排名与最终性能的秩相关性。
   - 完成标准: 有实测的秩相关系数或至少一组"代理 vs 最终"的矛盾案例记录；相关性差 → 收紧低保真协议（更多 epoch/更大子集）或改用多保真加权。
   - 判停条件: 权重共享(supernet)路线中发现候选间干扰严重（秩相关接近 0） → 放弃 one-shot 结论，退回独立采样评估。

5. **验收与人责交接**
   - 无论哪条路线，最终配置必须在从未参与搜索的留出数据/时间切片上复核一次；同步声明"数据质量、泄漏防御、问题定义不在自动化范围内"。
   - 完成标准: 留出集复核报告 + 人责清单（哪些事已自动化、哪些事仍归人）双份交付。
   - 判停条件: 留出集成绩显著低于搜索期成绩 → 优先怀疑搜索过程对验证集的隐性过拟合（选择偏置），回 `ml-leakage-defense` 排查后重审。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 架构已定、只差调参 → `ml-hpo-strategy`；把 NAS 叙事套在纯超参问题上会引入不必要的复杂度。
- 数据本身可疑（标签噪声/泄漏嫌疑/分布漂移）→ 先走 `ml-leakage-defense`；垃圾进垃圾出，自动化只会更快地产出精致的垃圾。
- 大模型下游应用（微调/提示/RAG）→ `ml-pretraining-paradigm`；这不是 NAS 的战场。

### 文献反复警告的失败模式

- **搜索空间决定天花板**: 搜索策略再聪明也只能找到空间里存在的结构；早期 NAS 的成功常被归功于搜索算法，实则精心设计的（模仿人工经验的）搜索空间功不可没——"自动化超越人类设计"的说法要打折听。
- **代理指标陷阱（本 skill 核心 B 条）**: 少 epoch 分数与最终性能的相关性因任务而异且经常很差；权重共享让候选们互相污染排名。任何"按代理指标选出"的结论都只是短名单假设，必须抽样完整训练验证后才可采信。
- **COTS/GTNAS 揭示的成本真相**: 经典 NAS 结果的算力账单（数百 GPU 天级）远超其替代的人力成本；引用"NAS 达到 SOTA"时必须同时报成本，否则是不完整的结论。
- **在验证集上的隐性过拟合**: 搜索成千上万次评估本身会耗尽验证集的信息量——被反复挑剩下的配置天然过拟合验证分布；嵌套验证/留出终审不可省略（与 HPO 共享此教训）。
- **可解释性与复现性代价**: 自动化产出的 pipeline 常是反直觉组合，且随机种子不同结果迥异；交付时应附搜索日志与多 seed 方差，否则难以维护。

### 作者盲点 / 时代局限（外推声明）

- **必须声明**: 本 skill 主题超出西瓜书覆盖范围——西瓜书(2016)对自动化调参仅有 f12 一句折中原则；本 skill 为外推补全，素材为 Elsken (2019)、Zoph & Le (2017)、Feurer (2015)、Klein COTS (2019) 等文献的共识转述，均标"（转述）"。
- 本 skill 主叙事截止于 2019-2020 年的 NAS 浪潮；此后领域重心已转移——预训练+适配范式使"从头搜架构"的需求大幅萎缩，zero-shot/NAS-Bench 类免训练估计器虽缓解了成本但未改变大局。执行时应意识到"NAS 是否还值得做"本身就是当代默认答案偏向否的问题。
- AutoML 文献的性能对照多以 Kaggle 型表格任务为样本，对其在流式/超大规模/隐私受限场景的表现着墨甚少；外推到这些场景缺乏证据。

### 容易混淆的邻近方法论

- **HPO（`ml-hpo-strategy`）**: 调给定架构的参数 vs 搜架构本身；工具链高度重叠（贝叶斯优化/Hyperband 两边通用），但问题层级不同——先答"搜什么层次"，再用那边的预算分配技术。
- **特征工程（`ml-feature-engineering`）**: AutoML 能自动尝试常见预处理组合，但理解业务的特征构造仍是人的核心价值；自动化覆盖不了特征语义。
- **元学习**: 同样讲"从历史任务学"，但目标是学会新任务的学习方法而非直接给出配置；概念近邻，问题不同。

---

## 相关 skills

- depends-on: ml-leakage-defense（自动化搜索放大验证集泄漏风险，防漏纪律前置）
- contrasts-with: ml-hpo-strategy（搜架构 vs 调参数，本批最关键分界）
- composes-with: ml-task-matching（外包选型前先做第一性判断）, ml-pretraining-paradigm（大模型时代的替代路线）, ml-feature-engineering（自动化管不到的人责区）, ml-methodology-router

---

## 审计信息

- **来源性质**: 扩充批D·核心缺口——主题超出西瓜书(2016)覆盖范围，R 段为 AutoML/NAS 奠基文献的转述表述（均已标注"（转述）"）
- **验证通过**: 待流水线三重验证复核
- **测试通过率**: 待阶段 4 测试 (详见 test-prompts.json)
- **skill_version**: 0.0.1
- **蒸馏时间**: 2026-08-24
