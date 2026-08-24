# Contributing to Machine-learning-skills

感谢关注本仓库！这是一套从周志华《机器学习》（西瓜书）蒸馏的 AI-agent 方法论 skill 合集。欢迎以下类型的贡献：

## 贡献方式

### 1. 修正内容错误
skill 内容以原书为唯一知识源。若发现某条引用与原书不符、或对方法论的理解有偏差：
- 提 Issue 时请注明：涉及的 skill 名 + 具体段落 + 原书章节页码
- PR 请附上原书原文佐证

### 2. 补充测试用例
每个 skill 的 `test-prompts.json` 是 darwin-skill 兼容格式。欢迎补充：
- 新的 `should_trigger` 场景（真实使用中遇到的语言信号）
- 新的 `should_not_trigger` 混淆诱饵（尤其是跨 skill 边界场景）
- 提交时请同步更新 `test-results.md`

### 3. 扩展新 skill
新增 skill 必须走与现有 skill 相同的质量流水线：

```
候选单元 → 三重验证(V1跨域/V2预测力/V3独特性) → RIA++六段构造 → test-prompts.json → 盲测≥80%通过
```

- **R** 原文引用 ≤150 字/条并标注章节
- **I** 用自己的话重写方法论骨架
- **A1** 书中作者亲自使用的案例（问题→用法→结论→结果）
- **A2** ≥3 条中英双写触发信号；description ≤300 字
- **E** 每步有完成标准与判停条件
- **B** 反场景 + 书中警告 + 作者盲点
- frontmatter 六字段齐全且通过 YAML 校验

### 4. 适配其他 agent 平台
本合集遵循 Claude Code skill 规范（SKILL.md + frontmatter）。欢迎提交 Cursor / 其他平台的适配层，但请保持 `skills/` 目录为单一事实源。

## 规范

- 文件编码 UTF-8；中文为主，术语保留英文
- 每个 PR 一个主题，不混合无关改动
- 提交信息用 Conventional Commits（`feat:` / `fix:` / `docs:` / `test:`）

## 致谢

所有方法论的知识产权属于周志华教授及清华大学出版社。本仓库仅做面向 AI-agent 的二次组织与工程化封装。
