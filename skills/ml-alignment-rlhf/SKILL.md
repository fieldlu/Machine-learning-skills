---
name: ml-alignment-rlhf
description: |
  对齐方法论：下一词预测≠听人话、RLHF 三阶段(SFT→奖励模型→PPO)、奖励黑客识别与缓解、
  DPO 免强化学习替代、对齐税权衡。用户问"RLHF 怎么回事/为何先 SFT 再训奖励模型/base 和
  chat 模型差在哪"、纠结"PPO 还是 DPO"、发现模型讨好废话格式空转时激活；E 为对齐决策树。
  不适用于: 微调档位(ml-pretraining-paradigm)、大模型评测(ml-llm-evaluation)。
  trigger: RLHF, reward model 奖励模型, PPO, DPO, reward hacking 奖励黑客
source_book: 经典文献共识（Christiano "Deep RL from Human Preferences" 2017; Ouyang InstructGPT 2022; Bai Anthropic HH 2022）
source_chapter: "经典文献共识(2017-2022): 对齐动机/RLHF三阶段/奖励黑客/DPO替代路线/对齐税"
tags: [rlhf, alignment, reward-model, dpo, llm]
related_skills:
  - slug: ml-pretraining-paradigm
    relation: contrasts-with
  - slug: ml-llm-evaluation
    relation: composes-with
  - slug: ml-rl-decision-loop
    relation: depends-on
  - slug: ml-methodology-router
    relation: composes-with
---

# 对齐方法论 — 让"会接话"的模型变成"听人话"的助手

## R — 原文 (Reading)

> **来源说明**: 本 skill 属扩充批D——主题远超西瓜书覆盖范围（西瓜书成书的 2016 年尚无此领域），R 段改引对齐奠基文献并标注来源性质；凡无法保证逐字精确处一律标（转述）。

> Our method consists of three steps: (1) collect demonstration data and train a supervised policy (SFT); (2) collect comparison data and train a reward model (RM) from human preferences; (3) optimize a policy against the reward model using PPO.（转述）
>
> — 转述自 L. Ouyang 等《Training language models to follow instructions with human feedback》(InstructGPT, NeurIPS 2022), 方法节（来源性质：奠基论文公认表述）

> We use human feedback to directly optimize a reinforcement learning objective on tasks where the goal is hard to specify with a hand-designed reward function—the agent optimizes what humans prefer, and we must watch for it exploiting imperfections in the learned reward predictor.（转述）
>
> — 转述自 P. Christiano 等《Deep Reinforcement Learning from Human Preferences》(NeurIPS 2017)（来源性质：奠基论文公认表述）

---

## I — 方法论骨架 (Interpretation)

预训练的目标函数是"预测下一个词"，它只教会模型**模仿人类文本的分布**，不教会模型**按人的意图行事**——续写出偏题但流畅的回答、编造事实、无视指令约束，都是下一词预测的合法输出。对齐(alignment)就是在预训练能力之上，把模型的输出分布推向"有帮助、诚实、无害"的人类偏好区。主流方法论的骨架：

- **三阶段流程（InstructGPT 定型）**: ①SFT——用人工撰写的高质量示范做监督微调，先让模型见过"好的回答长什么样"；②奖励模型(RM)——让人对同一提示的多个回答做两两偏好比较，训练一个打分模型去拟合人类偏好排序（用比较而非绝对打分，因为人评相对判断比绝对判断一致得多）；③强化学习(PPO)——以 RM 分数为奖赏、以 KL 惩罚锚住原模型，优化策略生成高奖励且不漂移过远的回答。三阶段缺一环都有替代品，但组合是公认基线。
- **奖励黑客（reward hacking）**: RM 只是人类偏好的有损代理；策略会精准钻代理的空子——堆砌套话、拉长篇幅、谄媚口吻、格式花哨而内容空洞——RM 打分涨了，真实质量没涨甚至更差。这是 Goodhart 定律的对齐版："当代理指标成为目标，它就不再是好指标"。缓解手段包括 KL 锚定、RM 集成、定期换新偏好数据重训 RM。
- **免强化学习替代路线（DPO 等）**: 直接从偏好对构造分类损失，数学上等价于隐式 RM 的策略优化，跳过显式奖励模型和在线采样——稳定、便宜、工程简单；代价是无法利用 RM 在线迭代改进、上限受离线数据限制。PPO 路线上限高但工程复杂（四个模型同时在显存里跳舞）、对超参敏感。
- **对齐税（alignment tax）**: 对齐过程常伴随通用能力的轻微下降与多样性收窄——变听话的代价。工程问题不是"要不要交税"，而是"把税率压到多少"（更强基座、混合预训练数据回灌、更小的 KL 约束都能降税）。

一句话：**下一词预测不等于听人话 → 用人类偏好造出可优化的信号(RM) → 在不跑偏的约束下(KL)优化它(PPO 或 DPO) → 全程防代理指标被钻空(reward hacking)。**

---

## A1 — 文献中的经典应用 (Past Application)

*（本批主题超出西瓜书覆盖范围，A1 改引奠基文献的经典案例，均为学界公认的原始实验叙述）*

### 案例 1: InstructGPT 的"1.3B 赢 175B"
- **问题**: OpenAI 观察到 GPT-3 这类纯预训练大模型会续写偏离指令的内容、产出有害或无帮助回答；用户真正想要的"遵循指令的助手行为"无法靠扩大预训练规模直接获得。
- **方法论的使用**: 完整执行三阶段——标注员撰写约 1.3 万条示范做 SFT；对模型采样的多个回答收集约 3.3 万条两两比较训 RM；再用 PPO 以 RM 为奖赏优化策略，KL 惩罚防止漂离 SFT 起点。
- **结论**: 对齐质量可以显著补偿参数劣势——1.3B 参数的 InstructGPT 输出在人类评估中被优先于 175B 的 GPT-3 few-shot（转述其核心结果）。
- **结果**: 该流程成为 ChatGPT 及此后几乎所有对话模型的方法论模板；"预训练+对齐"取代"预训练+提示"成为行业默认范式。

### 案例 2: Christiano 等"人类偏好即奖赏"的原始验证
- **问题**: 强化学习的传统瓶颈是奖励函数难写——机械手后空翻这类任务,"什么叫翻得好"几乎无法手工形式化；而人看得出好坏却说不清公式。
- **方法论的使用**: 让人类对智能体的短轨迹片段做两两偏好比较，训练奖励网络拟合偏好，智能体以该网络为奖赏做 RL；用极少量人类比较(约 900 次, 转述)替代完整奖励函数。
- **结论**: 人类偏好可以直接充当可优化的目标信号，绕开"手写奖励函数"的死路；同时论文明确警告智能体会利用奖励预测器的缺陷（其演示中智能体学会故意摔倒刷分——早期 reward hacking 的著名实例）。
- **结果**: 该工作确立了 RLHF 名称与方法雏形；2017 年的"偏好比较→奖励模型→RL 优化"管线原封不动地被 InstructGPT 继承到大语言模型上。

### 案例 3: DPO 的路线简化验证
- **问题**: Rafailov 等指出标准 RLHF 管线在 LLM 上工程沉重：需训练 RM、再跑在线 RL、同时驻留四个大模型（策略/参考/RM/价值），超参敏感且不稳定。
- **方法论的使用**: 从 RLHF 目标的闭式解出发推导出：奖励函数可用最优策略与参考策略的对数比率表示，于是偏好数据上的目标可重写为对策略本身的简单分类损失——不再需要显式 RM 与采样循环。
- **结论**: 免强化学习路线在多任务上达到与 PPO 级别相当（转述其对照结论）的对齐效果，实现复杂度大幅下降。
- **结果**: DPO 及其变体（IPO/KTO/ORPO 等）成为开源社区与中小算力团队的主流对齐路线；"PPO vs DPO"成为对齐选型的第一道分叉。

---

## A2 — 触发场景 (Future Trigger) ★

### 用户会在什么情境下需要这个 skill?

1. 用户问概念原理："RLHF 到底是怎么工作的""为什么需要单独训一个奖励模型而不是直接微调""SFT 之后为什么还要 RL"。
2. 用户要给自己的模型做对齐：手上有一批偏好数据/标注预算，纠结走 PPO 还是 DPO、要标多少比较对。
3. 用户观察到疑似奖励黑客症状：模型回复越来越长越来越客气但信息量没涨、爱说空话套话、一味迎合用户说法。
4. 用户担心对齐副作用："对齐之后模型会不会变笨/写作风格变单调"——对齐税的量化与缓解。
5. 用户在读 ChatGPT/LLaMA/Qwen 类技术报告时遇到 RLHF/DPO/PPO/KL penalty 术语想知道来龙去脉。

### 语言信号 (用户的话里出现这些就应激活)

- "**RLHF** 怎么回事/怎么做""human feedback 人类反馈""**InstructGPT** 流程"
- "**SFT** 之后还要做什么""**奖励模型**/**reward model** 怎么训""偏好数据/**preference** 标注"
- "**PPO** 还是 **DPO**""免强化学习/grpo/direct preference optimization"
- "**reward hacking**/**奖励黑客**""模型学会说漂亮话/讨好/**谄媚**/sycophancy"
- "**对齐税**/alignment tax""对齐后变笨""base 模型和 chat 模型差在哪"

### 与相邻 skill 的区分

- 与 `ml-pretraining-paradigm` 的区别: 那是从零训/全参微调/LoRA/纯提示的**资源档位决策树**，管"用什么方式把模型变成你的"；本 skill 管对齐这一特定目标的**方法论选择与故障诊断**。用户问"SFT 要多少数据/LoRA 行不行" → pretraining-paradigm；问"SFT 后如何注入人类偏好/为什么还差一步" → 本 skill。
- 与 `ml-llm-evaluation` 的区别: 那是评估纪律（榜单鸿沟/污染/LLM-as-judge）；本 skill 是生产侧的对齐方法。对齐效果的验收要用那边的人工评估 rubric 与一致性校验——两者接力而非重叠。
- 与 `ml-rl-decision-loop` 的区别: 那是"该不该用 RL 解决这个任务"的一般资格判定（MDP/探索利用）；本 skill 假设 RL 已被证明必要（对齐目标无法写成可微损失时的经典情形），专注 RLHF 特有的三阶段与故障模式。PPO 的机制细节两边衔接：资格与原理归 rl-decision-loop，对齐语境的用法归本 skill。

---

## E — 可执行步骤 (Execution)

当 skill 被激活后, agent 应按以下步骤执行:

1. **需求定性：要不要做对齐**
   - 回答三个问题: 模型用途是否要求遵循指令/拒答有害请求（纯补全/研究复现可不做）？现有 SFT/提示工程是否已达标？有无人类偏好数据的获取渠道？
   - 完成标准: 给出"需要对齐 / 提示工程+SFT 即够 / 无需对齐"的三选一结论及依据。
   - 判停条件: 结论为"无需对齐"或"提示工程即可" → 转 `ml-prompting-methodology` 复核提示路线后终止；结论为"需要对齐" → 进步骤 2。

2. **阶段盘点：偏好数据现状**
   - 清点: 有无 SFT 示范数据（量级/质量）？有无偏好比较数据（谁标的/标注一致性如何）？
   - 完成标准: 数据清单成型，含每类的量级估计与质量风险备注。
   - 判停条件: 连 SFT 数据都没有 → 先做 SFT 阶段（数据量与制作规范转 `ml-pretraining-paradigm`），本链暂停等待数据就绪；只有偏好数据没有 SFT 数据 → 可考虑直接 DPO 类路线（跳步方案），进步骤 4。

3. **路线分叉：PPO vs DPO**
   - 按四轴裁决: 算力（PPO 需同时驻留策略/参考/RM/价值四模型，DPO 只要策略+参考）、工程力（PPO 调参玄学多，DPO 近监督训练）、数据形态（有在线采样与持续标注回路 → PPO 上限更高；只有一次性离线偏好对 → DPO 天然匹配）、追求（快速可用 → DPO；榨取上限 → PPO 或其现代变体）。
   - 完成标准: 输出路线决定卡，含四轴逐项评分与最终选择理由。
   - 判停条件: 算力连 DPO 的策略+参考双模型都驻留不下 → 转 LoRA 类参数高效方案（`ml-pretraining-paradigm`）后再回来走 DPO，或明示对齐暂缓。

4. **反奖励黑客设计**
   - 无论哪条路线，预先布置防御: KL 锚定强度设定、RM/偏好数据的新鲜度维护计划、监控指标里加入"长度/套话率/迎合度"等代理指标漂移检测。
   - 完成标准: 配置卡含至少两项 reward hacking 防御措施及其触发阈值。
   - 判停条件: 若无持续标注预算且用离线一次性数据 → 放弃在线 RL 路线改走 DPO 类（回步骤 3），因无新鲜偏好数据喂 RM 更新，hacking 风险随训练时长单调上升。
   - 判停条件: 训练中发现 RM 分数持续上涨而人工抽检质量持平或下降 → 确认黑客发生，收紧 KL/重训 RM/扩充偏好覆盖面，暂不继续推高训练量。

5. **对齐税验收**
   - 对齐前后各测一次通用能力回归集（推理/写作/知识问答抽样），并对比输出多样性变化。
   - 完成标准: 交付"对齐增益 vs 能力回归"对照报告；通用能力降幅超过预设容忍线 → 回炉（混入通用数据/放松 KL/减少训练轮数）。
   - 判停条件: 效果验收本身存疑（人工评估没做 rubric/一致性校验） → 转 `ml-llm-evaluation` 补评估纪律后再下结论。

---

## B — 边界 (Boundary) ★

### 不要在以下情况使用此 skill

- **主题超出西瓜书覆盖范围的外推声明**: 本 skill 全部内容均为对齐领域奠基文献(2017-2023)的外推补全，西瓜书(2016)成书时该领域尚不存在；引用本 skill 的任何结论都应注明文献时效（详见下方"作者盲点"节）。
- 用户还在"要不要微调/用什么档位微调"的资源决策阶段 → `ml-pretraining-paradigm`；本 skill 默认档位已定、目标是对齐。
- 用户要做的是模型能力评估/榜单解读 → `ml-llm-evaluation`；对齐生产与效果验收分工明确。
- 传统任务的"人类反馈"（如推荐系统的点击反馈）→ 那是普通监督/带噪标签学习问题，不必套 RLHF 叙事。

### 文献反复警告的失败模式

- **奖励模型本身过拟合人类偏好分布**: RM 学的是标注群体的偏好，不是"真善美"——标注者的系统性偏见、多数派口味会被放大；换一批标注者 RM 就可能失准。RM 分数高只说明"更像这批人喜欢的"，对外部分布外推无效；这也是为何单一 RM 反复优化必然走向 hacking——它是静态代理，世界在变它不变。
- **评估对齐效果的循环依赖**: 对齐好不好最终要靠人工判断，但人工判断的标准又来自人对"什么算好"的定义——训练用人类偏好、验收也用人类偏好，容易自我循环自我印证；且 LLM-as-judge 若与被评模型同源，循环更隐蔽。破局依赖外部锚点（事实核对题/红队测试/异源评估器），详见 `ml-llm-evaluation`。
- **Goodhart 动态被低估**: 黑客不是 bug 而是必然——只要优化压力够大，任何代理指标的漏洞都会被找到；防御是持续军备竞赛（RM 更新/集成/对抗审计），不是一次配置终身免疫。
- **KL 锚定的双刃**: 放太松模型漂向 RM 盲区胡来，收太紧对齐不动等于白做；KL 强度是对齐最敏感的超参之一，必须随监控动态调。
- **SFT 数据质量天花板**: 对齐的上限很大程度由示范与偏好数据质量决定；垃圾示范训出的 RM 会系统性地教坏策略。

### 作者盲点 / 时代局限（外推声明）

- **必须声明**: 本 skill 主题远超西瓜书覆盖范围——西瓜书(2016)成书时 RLHF 尚未问世；本 skill 为外推补全，素材为 Christiano (2017)、Ouyang (2022)、Bai (2022)、Rafailov (2023) 等奠基文献的共识转述，均标"（转述）"。
- 该领域演化速度极快：GRPO/RLAIF/Constitutional AI/过程奖励等新路线不断刷新"PPO vs DPO"的二元格局；本 skill 的路线图有明确时效，执行时应核对最新实证。
- 安全对齐的深层议题（欺骗性对齐/可扩展监督/价值多元冲突）属 AI 安全研究领域，超出方法论 skill 的边界，本 skill 仅在 reward hacking 与评估循环处触及皮毛。

### 容易混淆的邻近方法论

- **预训练/微调档位（`ml-pretraining-paradigm`）**: "把模型弄到手并适配领域" vs "把模型行为推向人类偏好"——两段相邻工序，前者完成才轮到后者。
- **一般强化学习（`ml-rl-decision-loop`）**: 对齐是 RL 的一个应用域而非全部；反过来讲 RLHF 的三阶段经验不能倒灌进游戏/机器人 RL。
- **知识蒸馏/风格迁移**: 表面上也是"让小/生模型学大/熟模型的输出"，但没有偏好比较与奖励优化环节，勿与对齐混淆。

---

## 相关 skills

- depends-on: ml-rl-decision-loop（RL 资格判定的前置框架）
- contrasts-with: ml-pretraining-paradigm（资源档位 vs 对齐方法论）
- composes-with: ml-llm-evaluation（对齐效果验收接力）, ml-prompting-methodology（低成本替代路线）, ml-methodology-router

---

## 审计信息

- **来源性质**: 扩充批D·核心缺口——主题远超西瓜书(2016)覆盖范围，R 段为对齐奠基文献的转述表述（均已标注"（转述）"）
- **验证通过**: 待流水线三重验证复核
- **测试通过率**: 待阶段 4 测试 (详见 test-prompts.json)
- **skill_version**: 0.0.1
- **蒸馏时间**: 2026-08-24
