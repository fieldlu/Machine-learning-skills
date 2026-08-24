---
name: ml-experiment-tracking
description: |
  实验可复现纪律手册。当用户开新实验想记全该记的、同一份代码两次结果对不上、复现不了自己
  或别人的数字、纠结随机种子/配置/环境快照怎么管、检查点存什么才能断点续训、或实验做完要
  归档时激活。
  trigger: reproducibility 可复现、random seed 随机种子、配置版本化 Hydra/config、环境快照
  pip freeze、checkpoint 续训、DVC 数据版本、experiment tracking。
  动作: 新实验启动 checklist + 实验结束归档 checklist，让每个数字都能回答"从哪来"。
  不适用于: 结果归因（ml-diagnosis）、比较协议设计（ml-evaluation-design）、泄漏排查
  （ml-leakage-defense）。
source_book: "工程实践共识（业界公开文献与社区共识，非书内内容）"
source_chapter: "ML工程实践共识: 实验可复现纪律（补全西瓜书工程落地盲点）"
tags: [reproducibility, experiment-tracking, mlops, machine-learning]
related_skills:
  - slug: ml-evaluation-design
    relation: composes-with
  - slug: ml-diagnosis
    relation: composes-with
  - slug: ml-leakage-defense
    relation: contrasts-with
---

# 实验可复现纪律 — 每个数字都要能回答"你从哪来"

## R — 原文 (Reading)

> Machine learning is experimental in a way traditional software is not... A large fraction of ML work involves running experiments, and the results of those experiments need to be tracked and compared.（转述）
>
> — 转述自 Sculley 等, "Hidden Technical Debt in Machine Learning Systems", NeurIPS 2015 第 3 节（来源性质：业界公认技术债综述）

> Reproducibility requires recording code, data, configuration, and environment; missing any one of them can make results irreproducible.（转述）
>
> — 转述自 Pineau 等, "Improving Reproducibility in Machine Learning Research" / NeurIPS reproducibility program 的通行要求（来源性质：社区可复现性倡议）

> **来源说明**: 本 skill 属批C"工程实践共识"系列——《机器学习》(西瓜书)不含工程落地内容，
> 其最接近的锚点是第2章对测试集选择与算法随机性的讨论（"性能比较不能直接比大小"的前提之一
> 就是运行可控）。R 段改引业界公开文献并标注来源性质；凡无法保证逐字精确处一律标（转述）。

---

## I — 方法论骨架 (Interpretation)

一次实验 = 代码 × 数据 × 配置 × 环境 × 随机性，五个变量任何一个没钉住，结果就不可复现。纪律的核心不是装工具，而是让**每个产出的数字都能回溯到这五元组**：

1. **随机种子**: Python/NumPy/框架/GPU 全链路设种子并记录；同时承认 GPU 非确定性算子可能残留波动——种子保证的是"可追溯"，不承诺逐位一致。
2. **配置版本化**: 超参与流程参数全部进配置文件（Hydra/YAML），随输出一起落盘；禁止散落在 notebook 单元格里的魔法数字。
3. **环境快照**: `pip freeze` / lockfile 与实验同存；三个月后的库版本可以让同一份代码给出不同数字。
4. **输出目录规范**: 按 `{experiment}_{timestamp}` 命名，配置、日志、指标、模型放同一目录——目录本身就是实验档案。
5. **检查点策略**: best（按验证指标）+ latest（含 optimizer/scheduler 状态以支持 resume）双轨保存。
6. **数据版本**: 记录数据集哈希/版本标签；数据变了而代码没变，是"复现失败"最常见的隐藏原因。

两条元原则：**记录发生在实验开始时而非结束时**（事后回忆的配置必然缺项）；**复现不了的实验等于没做过**（不可信，不可写进论文或汇报）。这条纪律在科研场景是学术诚信问题，不只是工程洁癖。

---

## A1 — 业界公开案例 (Past Application)

> 注: 西瓜书无对应案例，本节用业界公开案例充当类比素材。

### 案例 1: "深度学习是炼金术吗"之争与 Ali Rahimi 的图灵演讲 (2017)
- **问题**: 社区被反复质问：调参靠玄学、结果难以精确复现，这样的研究算科学吗？
- **方法论的使用**: NIPS 2017 时间检验奖演讲把矛头指向"不记录、不复核"的工作方式，主张回归实验科学的记账纪律——控制变量、留档、可重复。
- **结论**: 深度学习要从炼金术升级为化学，第一步就是实验记录的系统化。
- **结果**: 直接推动了此后 NeurIPS 可复现性挑战赛与论文代码/附录强制政策的普及。

### 案例 2: 机器学习复现危机的系统证据 (Pineau 等, 2020s 倡议 + 多项复现研究)
- **问题**: 多项调查显示大量已发表 ML 论文的数字无法由第三方复现。
- **方法论的使用**: 社区开出标准处方——提交时附代码、配置、环境说明、随机种子声明；NeurIPS 把可复现性清单纳入投稿流程。
- **结论**: 复现性不是美德说教而是可执行的提交物清单，五元组（码/数/配/境/种子）缺一不可。
- **结果**: "reproducibility checklist"成为顶会标配，也反向塑造了工业界实验平台（MLflow/W&B 类）的字段设计。

### 案例 3: 书内原则回声——测试集选择与算法随机性 (c2)
- **问题**: 为什么两个模型的测试分数不能直接比大小？
- **方法论的使用**: 原书指出测试集本身的选取与算法的随机性都会引入扰动，因此需要多次重复与统计检验。
- **结论**: 要做统计比较，前提是每次运行的随机性被记录、被控制——实验追踪是该论证的工程前提。
- **结果**: 本 skill 把"记录种子→支持多次重复→支撑假设检验"接进原书的比较检验体系（ml-evaluation-design）。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. 用户准备开始训练/大规模实验，问"要不要设 seed""配置怎么管""输出往哪放"。
2. 用户两次跑同一份代码结果不一样，怀疑代码有 bug 或想找出差异来源。
3. 用户一个月后回来找某个最好结果的超参/数据版本，翻遍聊天记录和 notebook 找不到。
4. 用户训练到一半中断，不知道 checkpoint 里该存什么才能续训（只存了 model_state_dict）。
5. 用户要写论文/汇报，审稿人或导师问"换个种子还成立吗""用的什么环境"，答不上来。

### 语言信号 (用户的话里出现这些就应激活)

- "**复现**不了"/"两次结果**不一样**"/"跑不出论文里的数字"/"reproduce"/"irreproducible"
- "**随机种子**怎么设"/"random seed"/"为什么固定了 seed 还是不一样"
- "**实验记录**/**tracking** 用什么"/"MLflow/W&B 怎么选"/"实验太多管不过来"
- "**checkpoint** 该保存什么"/"断点**续训**"/"resume training"
- "**pip freeze**/**环境**记录"/"配置文件/**Hydra**"/"DVC/**数据版本**"

### 与相邻 skill 的区分

- 与 `ml-evaluation-design` 的区别: 那是设计评估方案（切分/度量/比较协议），本 skill 是给整个实验过程上账本。衔接：其"多次重复+假设检验"的要求依赖本 skill 记录的种子与环境才可执行。
- 与 `ml-diagnosis` 的区别: 那是解释结果为何不好；本 skill 保证结果可信可查。若诊断发现"两次结果不一致"，先经本 skill 排除未控随机性/配置漂移，再进入偏差方差归因。
- 与 `ml-leakage-defense` 的区别: 那是查"分数为什么虚高"（信息穿越）；本 skill 管"分数为什么对不上"（过程失控）。都属"结果不可信"，但病因与处方完全不同。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 按两份 checklist 执行:

### Checklist A: 新实验启动 (开工前逐项打勾)

1. **定身份**: 给实验起名 `{experiment}_{YYYYMMDD_HHMMSS}` 并建输出目录（conf/logs/metrics/ckpt 子目录）。
   - 完成标准: 目录存在且命名合规；本次实验的一切产物都将落在其中。
2. **钉种子**: 统一 set_seed 函数覆盖 random/numpy/torch(+cuda)/PYTHONHASHSEED；种子号写入配置。
   - 完成标准: 配置文件里能查到种子值；代码中无游离的随机调用绕过该函数。
3. **锁配置**: 所有超参、路径开关进 Hydra/YAML 配置；启动时自动 dump 一份到输出目录。
   - 完成标准: 输出目录里存在本次运行的完整 resolved config；代码里无裸写的魔法数字。
4. **存环境**: `pip freeze > requirements.txt`（或 lockfile）落入输出目录；GPU 型号/CUDA 版本一并记入日志。
   - 完成标准: 环境文件时间戳早于训练开始。
5. **验数据**: 记录数据集哈希（SHA256 前 12 位）或版本标签 + 预处理管道版本。
   - 完成标准: 日志中有数据指纹；能回答"这份结果是哪个数据版本跑出来的"。
6. **判停条件**: 用户明确表示只是随手试玩、不要档案 → 只执行步骤 2（种子），其余跳过并提示风险。

### Checklist B: 实验结束归档 (收尾前逐项打勾)

1. **指标落盘**: 最终指标 + 关键中间曲线写入 metrics 文件（JSON/CSV），不留在终端滚动区。
   - 完成标准: 指标文件存在于输出目录且含运行标识。
2. **检查点齐备**: best（按验证指标）与 latest（含 optimizer/scheduler/epoch 状态）都在 ckpt/ 下。
   - 完成标准: 从 latest 能 resume、从 best 能直接推理，二者均验证过加载成功。
3. **配置回填**: 若训练中手动改过参数，把实际生效值同步回 config 存档。
   - 完成标准: 存档 config = 实际运行的 config（对照日志确认）。
4. **一句话摘要**: 在实验索引表（README/表格）追加一行——名字、目的、关键改动 vs 上次、结论数字、指针。
   - 完成标准: 一个新人只读索引行就能决定要不要细看这个实验。
5. **复核声明**: 能否用当前目录独立重跑出主数字？不能 → 补缺口后才算归档完成。
   - 判停条件: 缺口无法补（如原始数据已被覆盖）→ 显式标记该实验"部分可复现"并列出缺失项，不得当作完全可信结论引用。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- 探索性的一次性脚本（用户明说不打算保留）：全套档案会拖垮节奏；按判停条件降级为只钉种子即可。
- 结果好坏的归因（为什么 loss 不降）：那是 ml-diagnosis 的事；本 skill 只负责"这个结果是谁、在哪、用什么跑出来的"。
- 已上线系统的监控告警：那是 MLOps 监控范畴（漂移检测、服务指标），超出单次实验的账本边界。

### 业界警告的失败模式

- **notebook 黑洞**: 结果只在某个再也没打开过的 notebook 输出格里，超参散落各 cell——三个月后自己都无法还原。
- **只存 model_state_dict**: 断点续训时 optimizer/scheduler/epoch 全丢，学习率 schedule 从头开始，"续训"实为静默换了个实验。
- **事后补记录**: 凭记忆回填配置，漏掉中途改过的参数；记录必须发生在运行时。
- **种子迷信**: 固定了种子就以为万无一失——cuDNN 非确定性、数据加载并行度仍会引入波动；种子是可追溯性手段，不是逐位复现保证。
- **数据悄悄变了**: 代码一行未动但上游表结构/清洗逻辑更新，旧结果从此对不上——没有数据指纹就永远查不出原因。

### 作者盲点 / 时代局限 (为何需要本 skill)

- 西瓜书的工程落地环节不在书内（BOOK_OVERVIEW 批判: 不讲特征管道版本化等落地环节）——书中连"调参后全量重训"这类流程纪律都只有一句带过，更无配置/环境/检查点管理；本 skill 即对该盲点的补全。
- 书成于 2016 年前，深度学习实验的 GPU 非确定性、大 checkpoint 管理、分布式训练记录等问题不在其视野内。
- 书中的"多次重复取平均"隐含"每次重复都被正确记录"的前提，但从未交代如何保证——这正是本 skill 接管的环节。

### 容易混淆的邻近方法论

- **实验追踪 ≠ 实验设计**: tracking 是记账（发生了什么），evaluation-design 是立规矩（怎样比才算数）；先有后者定义要记什么，前者才有的放矢。
- **MLOps ≠ 实验追踪**: MLOps 覆盖部署、监控、再训练全生命周期；本 skill 只取其中"实验期"的一段。
- **Git 版本控制 ≠ 实验追踪**: Git 管代码文本，不管"哪次运行用了哪份数据+哪个配置"；两者互补不可互替。

---

## 相关 skills

- composes-with: ml-evaluation-design（其多次重复+检验协议依赖本 skill 的种子/环境记录）, ml-diagnosis（"结果不一致"先排除过程失控再归因）
- contrasts-with: ml-leakage-defense（分数对不上=过程失控 vs 分数虚高=信息穿越）

---

## 审计信息

- **验证通过**: 批C·工程实践共识单元（书内仅 c2 提供原则回声）；V1 ✓ / V2 ✓ / V3 ✓ 待阶段 3 统一复核
- **测试通过率**: 待阶段 3 测试 (详见 test-prompts.json)
- **skill_version**: 0.0.1
- **蒸馏时间**: 2026-08-24
