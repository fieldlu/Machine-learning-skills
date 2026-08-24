# 《机器学习》（西瓜书）核心术语表 · GLOSSARY

> **蒸馏自周志华《机器学习》（清华大学出版社，2016，"西瓜书"），定义忠实原书上下文**（多为原文压缩，保留作者的限定语），不是百科式定义。
>
> 本表由阶段 1 的 193 条候选术语裁剪而来，收录约 60 条核心术语。裁剪标准：①跨章复用 ≥3 次（如过拟合贯穿 ch2/3/4/5/6/11），或②为某个子 skill 的关键概念（如"好而不同"之于 ml-ensemble-design）。完整 193 条底稿见 `books/ml-jiqi-xuexi/candidates/glossary.md`。
>
> "关联 skill"列指向仓库 `skills/` 下对应子 skill；路由入口见 `skills/ml-methodology-router`。

---

## 一、基本概念与全局骨架（第1章）

| 术语 | 英文 | 一句话定义（作者用法） | 关联 skill |
|---|---|---|---|
| 泛化 | generalization | 学得的模型适用于**没在训练集中出现的新样本**的能力；全书唯一的最高目标 | 全部（ml-methodology-router 总纲） |
| 归纳偏好 | inductive bias | 学习算法在假设空间中对假设进行选择的启发式或"价值观"（如"更平滑更好"）；无偏好的算法在 NFL 意义下平均等价于随机胡猜 | ml-task-matching |
| 版本空间 | version space | 与训练集一致的假设集合；多个一致假设并存说明有限样本从未唯一确定模型 | ml-task-matching |
| NFL 定理 | No Free Lunch theorem | 所有可能问题等概率出现时，任何两个学习算法的期望性能相同——脱离具体问题谈算法优劣毫无意义 | ml-task-matching |
| 奥卡姆剃刀 | Occam's razor | "若有多个假设与观察一致，选最简单的那个"；常用但并不平凡的归纳偏好原则——"何谓更简单"本身说不清，须靠验证兑现 | ml-task-matching |
| 过拟合 | overfitting | 把训练样本自身的一些特点当作了所有潜在样本的一般性质，导致泛化性能下降；**无法彻底避免只能缓解**（若可避免则 P=NP） | ml-diagnosis, ml-pitfall-audit |
| 欠拟合 | underfitting | 对训练样本的一般性质尚未学好；比过拟合容易克服（增加模型容量、增训练轮数） | ml-diagnosis |
| 假设 | hypothesis | 学得模型对应了关于数据的某种潜在规律，故亦称假设；假设空间即学习算法考虑的所有可能假设 | ml-theory-compass |
| 监督学习/无监督学习 | supervised / unsupervised learning | 训练数据拥有标记信息（分类、回归为代表）vs 不拥有标记信息（聚类为代表）的学习任务；半监督介于两者之间 | ml-task-matching |

## 二、模型评估与比较检验（第2章）

| 术语 | 英文 | 一句话定义（作者用法） | 关联 skill |
|---|---|---|---|
| 训练误差/经验误差 | empirical error / training error | 学习器在训练集上的误差；实际能做的是使它最小化，但它 ≠ 泛化误差 | ml-evaluation-design, ml-diagnosis |
| 泛化误差 | generalization error | 学习器在**新样本**上的误差；不可直接观测，只能用测试误差近似，可分解为偏差²+方差+噪声 | ml-diagnosis, ml-theory-compass |
| 错误率/精度 | error rate / accuracy | 错误率=分类错误样本占比，精度=1−错误率；最基础度量，但**类别不平衡时失效** | ml-evaluation-design, ml-imbalanced-learning |
| 测试集/验证集 | testing set / validation set | 测试集用于估计泛化能力、必须与训练集互斥；验证集是模型选择中用于调参的数据 | ml-evaluation-design |
| 留出法 | hold-out | 直接将数据集划分为互斥的训练集和测试集；单次划分不稳定，需多次随机划分取均值±波动并保持类别比例 | ml-evaluation-design |
| 交叉验证法 | cross validation | 将数据分为 k 个互斥子集，每次用 k−1 个训练、剩 1 个测试，返回 k 次均值（k 常取 10）；折间训练集重叠会破坏错误率独立性 | ml-evaluation-design |
| 自助法/包外估计 | bootstrapping / out-of-bag estimate | 有放回采样 m 次，约 36.8% 样本不出现、可作测试集；改变了初始分布引入估计偏差（大数据集不推荐）；同一采样机制被 Bagging 挪用于制造多样性 | ml-evaluation-design, ml-ensemble-design |
| 调参 | parameter tuning | 对每个参数选定范围和步长、从候选值中选优，本质也是模型选择，是计算开销与估计精度的折中；**选定后须用全量数据重训最终模型** | ml-evaluation-design, ml-diagnosis |
| 性能度量 | performance measure | 衡量模型泛化能力的评价标准，反映任务需求——模型好坏是相对的，换一个度量结论可能反转 | ml-evaluation-design |
| 查准率/查全率 | precision / recall | P=挑出的瓜中好瓜比例；R=所有好瓜中被挑出的比例；一对矛盾的度量，偏重哪个由任务需求决定（Fβ 以 β 显式编码取舍） | ml-evaluation-design, ml-imbalanced-learning |
| ROC 曲线/AUC | ROC curve / AUC | 真正例率对假正例率的曲线及其下面积，衡量**排序质量**（AUC=1−排序损失）；对角线对应随机猜测 | ml-evaluation-design |
| 非均等代价/代价敏感 | unequal cost / cost-sensitive | 不同类型错误后果不同时为错误赋予不等同代价；目标从最小化错误次数变为最小化总体代价，通常只关心**代价比值** | ml-evaluation-design, ml-imbalanced-learning |
| 假设检验 | hypothesis test | 测试错误率只是随机变量的一次实现，对泛化性能的判断须做统计检验；**比较不能直接比大小** | ml-evaluation-design |
| Friedman 检验/Nemenyi 后续检验 | Friedman test / Nemenyi post-hoc test | 多个算法在一组数据集上按序值排序的检验；拒绝"全部相同"后再用 Nemenyi 临界值域 CD 区分配对——检验图上横线段交叠=无显著差别 | ml-evaluation-design |
| 偏差-方差分解 | bias-variance decomposition | 泛化误差 = 偏差² + 方差 + 噪声，分别刻画拟合能力、数据扰动影响、问题固有难度；两者冲突（窘境），训练程度是调节旋钮 | ml-diagnosis, ml-ensemble-design |
| 噪声 | noise | 当前任务上**任何学习算法**所能达到的期望泛化误差的下界——问题固有难度，不是"脏数据"；给一切优化画了天花板 | ml-diagnosis |

## 三、经典模型关键概念（第3–7章）

| 术语 | 英文 | 一句话定义（作者用法） | 关联 skill |
|---|---|---|---|
| 正则化/结构风险 | regularization / structural risk | Ω(f) 描述期望模型具有的性质（如复杂度小），与经验风险折中以削减假设空间；C 是两项汇率，贝叶斯视角下 Ω 即先验 | ml-diagnosis, ml-dimensionality |
| 对数几率回归 | logistic regression | 用线性回归模型的预测结果逼近真实标记的对数几率；名为"回归"，实为分类学习方法 | ml-task-matching |
| 类别不平衡 | class-imbalance | 不同类别的训练样例数目差别很大；此时永远猜多数类也能达到高精度却毫无价值 | ml-imbalanced-learning |
| 再缩放 | rescaling | 令决策基于观测几率 y/(1−y) > m⁻/m⁺ 判正；把比值换成 cost₊/cost₋ 即代价敏感学习；前提"训练集是无偏采样"往往不成立 | ml-imbalanced-learning, ml-pitfall-audit |
| 多分类拆解 | OvO / OvR / MvM | 把多分类任务拆为若干二分类任务求解的三种经典策略；OvO 开销随类数 N 平方增长（万类≈5000 万个分类器，当场判死） | ml-imbalanced-learning |
| 间隔/支持向量 | margin / support vector | 两个异类支持向量到超平面的距离之和；找最大间隔超平面是 SVM 基本型，训练完成后最终模型仅与支持向量有关 | ml-task-matching |
| 软间隔 | soft margin | 允许部分样本不满足约束以换整体鲁棒；"完美线性可分"可能恰是过拟合的假象——训练集满分是红旗不是捷报 | ml-diagnosis, ml-pitfall-audit |
| 核函数/核技巧 | kernel function / kernel trick | k(xᵢ,xⱼ)=⟨φ(xᵢ),φ(xⱼ)⟩：跳过高维甚至无穷维空间中内积的直接计算；**核函数选择是 SVM 的最大变数**——选错核等于映射进错误空间 | ml-dimensionality, ml-theory-compass, ml-task-matching |
| 替代损失 | surrogate loss | 用来替代数学性质差的 0/1 损失的凸连续函数（hinge/指数/对率为代表）；替代品决定产物基因，但代理最优 ≠ 原指标最优 | ml-theory-compass |
| 先验/后验概率 | prior / posterior | 类先验 P(c)=各类样本所占比例；后验 P(c\|x)=获得观测后对该类别的信念；贝叶斯定理联系两者 | ml-bayesian-thinking |
| 判别式/生成式模型 | discriminative / generative models | 直接建模 P(c\|x) 来预测 vs 先对联合概率分布 P(x,c) 建模再获后验；决策树/BP/SVM 属前者，贝叶斯分类器属后者 | ml-bayesian-thinking |
| 极大似然估计 | maximum likelihood estimation (MLE) | 在参数所有取值中找到使数据出现"可能性"最大的值；致命前提是概率分布形式靠猜——**猜错则全盘误导**，且后果被伪装成"参数没调好" | ml-bayesian-thinking, ml-pitfall-audit |
| 朴素贝叶斯分类器 | naive Bayes classifier | 采用属性条件独立性假设的贝叶斯分类器；假设在现实中很难成立却常有好性能 | ml-bayesian-thinking |
| 贝叶斯网 | Bayesian network | 借助有向无环图刻画属性依赖关系、用条件概率表描述联合分布；是独立性假设放松谱系（朴素→半朴素→任意依赖）的"任意依赖"端 | ml-bayesian-thinking |
| 推断/近似推断 | inference / approximate inference | 通过证据变量的观测推测查询变量取值的过程；精确推断 NP 难，降级路径分随机化（MCMC/吉布斯采样）与确定性（变分推断）两条 | ml-bayesian-thinking |
| EM 算法/隐变量 | Expectation-Maximization / latent variable | 存在未观测的隐变量时，交替执行"E 步据当前参数推断隐变量分布、M 步最大化期望似然"直至收敛；只收敛到局部最优，k-means/高斯混合/HMM 共用此骨架 | ml-bayesian-thinking |

## 四、集成学习（第8章）

| 术语 | 英文 | 一句话定义（作者用法） | 关联 skill |
|---|---|---|---|
| 集成学习 | ensemble learning | 通过构建并结合多个学习器来完成学习任务；收益的前提是个体"好而不同"，盲目堆数量只是白付算力 | ml-ensemble-design |
| 好而不同 | accuracy & diversity | 个体学习器要有一定准确性（不能太坏），且彼此有多样性（有差异）；两者本身存在冲突，权衡恰是集成学习的核心问题 | ml-ensemble-design |
| Boosting | Boosting | 串行化集成：根据基学习器表现调整样本分布，让先前做错的样本在后续受到更多关注；可将弱学习器提升为强学习器，主要关注**降低偏差** | ml-ensemble-design, ml-diagnosis |
| Bagging/随机森林 | Bagging / Random Forest | 并行式集成：自助采样出 T 个训练集各训一个基学习器后投票/平均，主要关注**降低方差**，对不稳定学习器（决策树、神经网络）效用明显；随机森林在样本扰动外再加属性扰动 | ml-ensemble-design |
| Stacking | Stacking | 用另一个学习器来结合个体学习器的输出；次级训练集必须经交叉验证产生，否则初级学习器"背答案"的记忆映射会让线下虚高线上崩 | ml-ensemble-design, ml-pitfall-audit |
| 多样性增强 | diversity enhancement | 在学习过程中引入随机性制造差异的四个通道：数据样本扰动、输入属性扰动、输出表示扰动、算法参数扰动；稳定基学习器（线性模型、SVM、朴素贝叶斯、kNN）对样本扰动不敏感，须换通道 | ml-ensemble-design |

## 五、聚类降维：高维、距离与特征选择（第9–11章）

| 术语 | 英文 | 一句话定义（作者用法） | 关联 skill |
|---|---|---|---|
| 维数灾难 | curse of dimensionality | 高维情形下数据样本稀疏、距离计算困难等问题，是一切机器学习方法共同面临的严重障碍；两大主流对策：降维与特征选择 | ml-dimensionality |
| k 近邻 | k-Nearest Neighbor (kNN) | 给定测试样本，找出训练集中与其最近的 k 个邻居投票预测；懒惰学习代表，但其赖以生存的"近邻"在高维失效（最近邻与最远邻距离趋同） | ml-dimensionality |
| 距离度量/非度量距离 | distance measure / non-metric distance | 距离须满足非负性、同一性、对称性、直递性；有序属性用闵可夫斯基距离、无序属性用 VDM；公理可为任务语义让步（"人""马"都似"人马"而彼此不似） | ml-dimensionality |
| 主成分分析 | Principal Component Analysis (PCA) | 从最近重构性（点到超平面距离近）与最大可分性（投影后方差大）两条等价路径导出的线性降维；舍弃最小特征值方向还能顺带去噪 | ml-dimensionality |
| 流形学习 | manifold learning | 借鉴拓扑流形概念的降维：流形局部与欧氏空间同胚，可在局部建立映射再推广全局；本真距离是贴面爬行的测地线距离，警惕近邻图的短路与断路 | ml-dimensionality |
| 度量学习 | metric learning | 基于数据样本直接"学习"出一个合适的距离度量（马氏距离矩阵）；与其间接挑空间不如显式参数化中间设定塞进优化目标 | ml-dimensionality |
| 特征选择 | feature selection | 从给定特征集合中选出相关特征子集的过程（过滤式/包裹式/嵌入式三路线）；注意删"冗余"特征可能反而坏事——冗余特征恰为任务所需的中间概念时有益 | ml-dimensionality |
| 岭回归/LASSO | ridge regression / LASSO | L₂/L₁ 范数正则化的最小二乘；L₁ 菱形等值线的尖角使交点常落在坐标轴上（分量恰为零），更易获得稀疏解——求解过程本身就是特征选择 | ml-dimensionality |

## 六、理论与进阶：保证的前提（第12–13章为主）

| 术语 | 英文 | 一句话定义（作者用法） | 关联 skill |
|---|---|---|---|
| PAC 可学习性 | Probably Approximately Correct learning | 以较大概率（至少 1−δ）学得误差满足预设上限 ε 的模型——把"要准确率达到 99%"这类模糊要求译成"置信度+精度预算"的可判定契约 | ml-theory-compass |
| VC 维 | Vapnik-Chervonenkis dimension | 能被假设空间打散的最大示例集大小；度量无限假设空间的复杂度，**与数据分布无关**——因此普适但松 | ml-theory-compass |
| Rademacher 复杂度 | Rademacher complexity | 假设空间 H 与随机噪声在数据集上的相关性；在一定程度上考虑了数据分布，故比基于 VC 维的界更紧（"量身定制"） | ml-theory-compass |
| 稳定性/经验风险最小化 | stability / Empirical Risk Minimization (ERM) | 算法在输入变化（移除/替换一个样例）时输出是否随之大幅变化；**ERM 且稳定即可学习**——稳定性分析不必考虑假设空间的所有假设 | ml-theory-compass |
| 最小描述长度 | Minimum Description Length (MDL) | 学习即数据压缩：总码长=模型描述字节+数据编码字节；一套公式统一了 MLE/AIC/BIC，把奥卡姆剃刀"何谓简单"变成可量化评分 | ml-theory-compass |
| 未标记样本/半监督学习 | unlabeled sample / semi-supervised learning | 无标记但包含数据分布信息的廉价样本；**模型假设不准时，利用它们反倒会降低泛化性能**——免费午餐会反噬，须保留纯监督基线对照 | ml-pitfall-audit, ml-theory-compass |
| 主动学习/伪标记 | active learning / pseudo-label | 先用已有模型挑最有价值的样本请专家查询（与半监督的区别在于有无外界交互）/ 把学习器对未标记样本的预测当作其"标记"使用——自训练与协同训练的核心机制，漂移场景下会自我强化偏差 | ml-pitfall-audit, ml-theory-compass |

---

## 统计与使用说明

- **收录词条：62 条**（源：candidates/glossary.md 的 193 条；裁掉的主要是各章算法专属细节词，如增益率/SMO/势函数/FOIL 增益/Q-学习等——它们服务于具体算法实现，而非跨场景的方法论判断）
- 按主题分组：基本概念 9 · 模型评估 16 · 经典模型 16 · 集成 6 · 聚类降维 8 · 理论与进阶 7
- 使用方式：任一子 skill 被 ml-methodology-router 派发后，若用户对术语含义有疑问，回查本表而非重新解释；本表定义均为周志华原书语境下的用法，引用到其他语境（尤其深度学习/大模型场景）时注意外推边界。
