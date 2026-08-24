<div align="center">

# Machine-learning-skills

**从周志华《机器学习》（西瓜书）蒸馏的 AI-agent 方法论 Skill 合集**
*A methodology skill collection distilled from Zhou Zhihua's *Machine Learning* ("Watermelon Book") — not summaries, but executable judgment frameworks.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
![Skills](https://img.shields.io/badge/skills-10-2563eb)
![Blind Tests](https://img.shields.io/badge/blind%20tests-90%2F90%20passed-16a34a)
![Verified Units](https://img.shields.io/badge/triple--verified%20units-97-f59e0b)
[![Made with Claude Code](https://img.shields.io/badge/made_with-Claude_Code-8b5cf6)](https://claude.com/claude-code)

</div>

---

## 这是什么 / What is this

这不是又一份机器学习知识点摘要，而是一套 **AI-agent 可直接调用的决策方法论合集**。它把西瓜书里反复出现的判断纪律——"脱离任务谈算法优劣无意义"、"先修尺子再谈诊断"、"训练精度满分是红旗不是捷报"、"理论给出边界而非答案"——蒸馏成 **1 个路由总控 + 9 个原子决策框架**。每个 skill 都带：

> This is **not** a knowledge digest. Each skill is an **executable judgment framework**: anchored quotes from the original book, worked cases, trigger conditions, step-by-step execution, stop criteria, and boundary warnings — producing deliverables (task profiles, diagnosis conclusions, audit reports), not vague advice.

- 📖 **原文锚点**（R）：每条方法论回指原书章节，引用 ≤150 字并标注出处
- 🦴 **方法论骨架**（I）：作者的原话被重写为可操作的判断结构
- 📚 **书中案例**（A1）：作者亲自演算过的案例（问题→用法→结论→结果）
- 🎯 **触发场景**（A2）：中英双写的语言信号，让 agent 知道何时激活、何时不该
- ✅ **执行步骤**（E）：每步有完成标准与判停条件
- ⚠️ **边界警告**（B）：何时失效 + 作者盲点 + 反场景

## ✨ 特性 / Features

- **🔬 97 处三重验证单元** — 所有内容通过 V1 跨域复现 / V2 预测力 / V3 独特性三关验证，淘汰明细公开在 `rejected/` 思路下可追溯
- **📐 RIA++ 六段同构** — 10 个 skill 共享同一结构（Reading / Interpretation / Past Application / Future Trigger / Execution / Boundary），agent 读一个就认识全部
- **🧭 总控路由** — `ml-methodology-router` 是唯一调度入口：先定位你卡在工作流哪一段（选型 / 实验设计 / 结果诊断 / 特殊场景），再派发对应子 skill
- **🧪 90 条盲测全部通过** — 每个 skill 配 `test-prompts.json`（should_trigger + 混淆诱饵 + edge case），由未接触期望答案的独立盲测员判卷，darwin-skill 兼容可直接进化
- **🔗 Zettelkasten 链接网络** — skill 间的关系（depends-on / contrasts-with / composes-with）全部显式声明，见 [INDEX.md](./INDEX.md) 的引用图

## 📦 安装 / Install

**方式一：Claude Code 用户级安装（推荐）**

```bash
git clone https://github.com/fieldlu/Machine-learning-skills.git
cp -r Machine-learning-skills/skills/* ~/.claude/skills/
```

**方式二：直接 clone 后按宿主放置**

```bash
# 项目级（仅当前项目可用）
cp -r skills/<skill-slug> <project>/.claude/skills/    # Claude Code
cp -r skills/<skill-slug> <project>/.cursor/skills/    # Cursor
```

**方式三：手动复制单个 skill**

只需要某个专项（比如只要诊断）？直接把 `skills/ml-diagnosis/` 一个目录拷进宿主 skills 目录即可。每个 skill 目录自包含（SKILL.md + test-prompts.json + test-results.md），不依赖仓库其他文件。唯一建议：如果全装，router 会自动帮你串联其余 9 个。

## 🚀 快速开始 / Quick Start

装好后，像平常一样向 agent 提问即可。三个真实场景：

**① "该用什么算法？"** → router 定位到选型前思考 → 派发 `ml-task-matching`

> **你**：我们要做一个欺诈检测，团队说 XGBoost 是榜单第一，直接上？
>
> **agent**：先放下榜单——NFL 定理证明脱离具体任务谈算法优劣没有意义。先刻画你的任务：样本量多大？正负比例多少？错误代价对称吗？欺诈检测通常类别极不平衡且漏检代价高，这意味着"准确率最高"的算法几乎注定是错的选择。我们先做任务画像，再匹配归纳偏好……

**② "模型过拟合了！"** → router 定位到结果诊断 → 派发 `ml-diagnosis`

> **你**：训练精度 100%，验证集才 82%，是不是再多训几轮就好了？
>
> **agent**：训练集满分在这里是**红旗而不是捷报**。用偏差-方差分解归因：训练/验证间隙大说明方差主导（背题了），再加轮数只会更糟。处方方向是降方差：更多数据、加强正则、早停，或接一个 Bagging 类集成。另注：过拟合无法彻底避免只能缓解（能根除就等于解决了 NP 难问题），目标是管理风险而非消灭它。

**③ "准备上线了"** → router 定位到横向审查 → 派发 `ml-pitfall-audit`

> **你**：离线 AUC 0.99，下周上线。
>
> **agent**：结果好得可疑时，先过六问前提审查再排期：①训练数据是无偏采样吗？②测试错误率是独立采样吗？③分布形式假设对吗？④……⑥这个 0.99 会不会是信息泄露？每问都有可观测的诊断信号和修正动作，三十秒审查换掉的是上线后的翻车归因。

纯代码实现问题（sklearn 语法、报错栈）不会被路由——直接回答，不走方法论流程。

## 🗂️ 10 个 Skill 一览

| Skill | 一句话 | 典型触发语 |
|---|---|---|
| [`ml-methodology-router`](./skills/ml-methodology-router/SKILL.md) | 总入口：定位你卡在工作流哪一段再派发 | 任何 ML 决策类提问的第一站 |
| [`ml-task-matching`](./skills/ml-task-matching/SKILL.md) | 没有最强算法，只有匹配任务的算法（NFL → 刻画任务 → 匹配归纳偏好） | "该用什么模型""榜单第一能搬吗" |
| [`ml-evaluation-design`](./skills/ml-evaluation-design/SKILL.md) | 实验三层设计：切数据 · 选尺子 · 信比较 | "留出还是交叉验证""高 0.5% 显著吗" |
| [`ml-diagnosis`](./skills/ml-diagnosis/SKILL.md) | 先分清病种再开药：偏差/方差/噪声归因 | "过拟合怎么办""训练精度 100%！" |
| [`ml-imbalanced-learning`](./skills/ml-imbalanced-learning/SKILL.md) | 类别不平衡：确认度量失守 → 再缩放三路线选路 | "accuracy 99.9% 但召回为 0""SMOTE 有坑吗" |
| [`ml-ensemble-design`](./skills/ml-ensemble-design/SKILL.md) | 集成设计：好而不同 · Boosting 降偏差 / Bagging 降方差 | "多模型融合会更好吗""stacking 线上线下不一致" |
| [`ml-dimensionality`](./skills/ml-dimensionality/SKILL.md) | 高维困境：症状诊断 → 降维/特征选择/稀疏/度量学习四主线选路 | "p≫n 怎么办""kNN 忽好忽坏" |
| [`ml-bayesian-thinking`](./skills/ml-bayesian-thinking/SKILL.md) | 概率建模的假设纪律：分布形式先于参数、独立性档位、EM 与推断降级 | "朴素贝叶斯假设不成立""EM 不收敛" |
| [`ml-pitfall-audit`](./skills/ml-pitfall-audit/SKILL.md) | 上线前六问前提审查：无偏采样/独立采样/分布形式/误差独立/桥接假设/信息泄露 | "线下好线上崩""结果好得可疑" |
| [`ml-theory-compass`](./skills/ml-theory-compass/SKILL.md) | 把学习理论当提问框架：PAC 三问、容量刻度、MDL、替代损失 | "需要多少数据才够""理论上说该信几分" |

新手建议沿 [INDEX.md](./INDEX.md) 的推荐路径走一遍（router → task-matching → evaluation-design → diagnosis → 专项 → pitfall-audit 收尾）；老手按语言信号直取专项即可。

## 🧠 这套东西怎么造出来的 / How it was built

采用 [cangjie-skill](https://github.com/) 的 **RIA-TV++ 六阶段流水线**，从原书 440 页出发，全程人工把关：

```mermaid
graph LR
    A[原书 440 页<br/>22 docx · 16 章] --> B[阶段1<br/>5 提取器并行]
    B --> C[376 条候选<br/>框架44·原则50·反例51·案例38·术语193]
    C --> D["阶段1.5 三重验证<br/>V1 跨域 · V2 预测力 · V3 独特性"]
    C --> M["合并去重<br/>147 个独立单元"]
    M --> D
    D --> E[92 条通过<br/>55 条淘汰入 rejected/]
    E --> F[阶段2 RIA++ 构造<br/>97 处单元组装]
    F --> G[阶段3 Zettelkasten<br/>10 skills 互链]
    G --> H[阶段4 盲测<br/>90/90 通过]
```

流水线关键纪律：**提取 ≠ 录取**。五路提取器（框架/原则/反例/案例/术语）各自动扫全书产出候选后，独立验证员对每条候选跑三道关卡——能否跨域复现（V1）、能否预测真实场景（V2）、是否只是常识包装（V3）——三条全过才录取，淘汰记录留档可查。

## 📊 验证漏斗 / Verification Funnel

| 阶段 | 数量 | 明细 |
|---|---|---|
| 文本源 | 440 页 · ~367k 字符 | 22 个 docx 分块，16 章 |
| 五提取器候选 | **376 条** | 框架 44 + 原则 50 + 反例 51 + 案例 38 + 术语 193 |
| 合并去重后独立单元 | **147 个** | 框架/原则池 94→58；反例/案例池 89 |
| 三重验证通过 | **92 条**（63%） | 39 框架/原则 + 53 反例/案例；淘汰 55 条留档 |
| 组装进 10 个 skill | **97 处单元引用** | 个别核心单元服务多个 skill（如偏差-方差） |
| 盲测压测 | **90/90 通过** | 每 skill 9 题：4 trigger + 3 诱饵 + 2 边界 |

盲测方法：盲测员只拿到 10 个 skill 的 name + description 目录（不接触期望答案与内部结构），对 90 条测试提示逐一选择应否触发及路由去向。机器判卷 95.6%，人工复核 4 条边界题后确认真实通过率 100%（含 2 条脚本解析误记）。判卷明细见各 skill 的 [`test-results.md`](./skills/ml-diagnosis/test-results.md)。

## ⚠️ 边界声明 / Known Limitations

诚实地说清这套 skill 的适用范围，正如每个 skill 自己的 B 段所要求的那样：

- **时代局限**：原书成书于 2015–2016 年。Transformer、预训练–微调范式、scaling law 完全不在书中；深度学习仅在两处简述，作者甚至转引了"深度学习热潮大过实际贡献"的争议观点。涉及大模型时代的判断请勿直接套用。
- **i.i.d. 假设贯穿全书**：对分布漂移、域适应只有只言片语（原书序言中李未院士已专门指出这一点）。数据分布会漂移的生产环境需自行补课。
- **不覆盖工程落地环节**：全书不讲数据质量管理、特征管道版本化、线上–线下一致性工程、A/B 发布与监控。"评估"止步于离线指标——这正是 `ml-pitfall-audit` 存在的原因，但也仅止于事前审查。
- **教材体例约束**：为教学清晰牺牲了对"多个技巧组合使用时的相互作用"的讨论；真实项目中方法从不孤立出现。

但反方向的辩护也成立：凡进入严肃生产或科研的场景（医疗、工业、科研发现），样本昂贵、错误代价高——这套三层评估体系与偏差-方差思维依然是不可替代的基本功。它教的不是某个算法，而是**对任何学习结果保持怀疑并科学验证的态度**。

## 🙏 致谢 / Acknowledgments

- 本仓库的**唯一知识源**是周志华所著《机器学习》（清华大学出版社，2016，"西瓜书"）。所有原文引用版权归原书作者与出版社所有；本仓库是对其方法论结构的二次组织与工程化封装，用于学习与研究目的。
- 若本仓库内容对你的工作有帮助，请同时引用原书：周志华. 机器学习. 北京：清华大学出版社，2016。
- 本仓库与原作者及出版社无隶属关系；发现引用与原书不符请提 Issue 并附页码。
- 流水线工具：cangjie-skill（拆书蒸馏）；测试兼容：darwin-skill；构建过程由 Claude Code 辅助完成。

## 📄 License

[MIT](./LICENSE) © 2026 fieldlu
