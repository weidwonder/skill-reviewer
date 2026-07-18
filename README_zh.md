# skill-reviewer

*[English](./README.md)*

一个用来审查其他 Agent Skill 质量的技能（Agent Skill）——每个 skill 上线前都该有、却常常没有的那道质量关卡。它完全站在「真正要用这个 skill 干活的 Agent」视角通读被审技能，把"感觉哪里不对"变成一份可执行的 P0/P1/P2 分级报告，把触发失灵、悬空引用、隐式逻辑漏洞这些坑，挡在它们真正搞砸一次生产运行之前。

> 非目标：本技能不负责从零创建或重构 skill（那是 skill-creator / darwin-skill 之类构建技能的职责），只做审查与反馈。

## 特性

- **23 项审查清单**，分三层：
  - **机械层校验（1.1–1.4）**：frontmatter 合规（字段白名单、name 规则）、description 触发质量（含关键词堆砌阈值、排除子句）、Markdown 结构完整性、安全扫描（破坏性命令、硬编码凭证、allowed-tools 过度授权）
  - **通用检查（2.1–2.16）**：工程信息泄露、流程与工具完备性（含决策判据/完成判据与脚本样例实跑比对）、前后文冲突、git 版本管理与资产迁移、冗余与体积阈值、孤立文档与引用深度、违背常识、案例缺失、异常场景说明、正例优先与反例污染、隐式逻辑遗漏、Runtime 中立性、目标/非目标清晰度、reference 读取条件、结构一致性（术语漂移/近似重复/结构断裂）
  - **依赖与安装检查（3.1–3.4）**：独立安装指导、环境隔离、状态/凭证 JSON 分离（凭证必须 gitignore）、失败驱动安装
- **使用者视角**：始终假装第一次拿到被审 skill、要完全依靠它完成任务
- **分级报告**：每条问题落到「位置（文件:行）+ 问题 + 建议」，附通过项、不适用项与可选的 8 维成熟度评分卡
- **用户判决机制**：孤立文档、疑似违背机制、禁区补充、反例收窄等问题先暂存，审查完统一向用户提问，不打断式逐条问
- **大技能拆分审查**：文件数 > 20 时按分支并行派发子代理，全局检查项（跨文件冲突、孤立文档、版本管理、结构一致性）保留在主会话
- **两个可选深度模式**（默认不启用，审查完成后询问）：
  - **增强 Review**：联网对照领域现状，评估过时 / 不一致 / 遗漏 / 过度限制
  - **实测验证**：触发率正负样本抽测、RED/GREEN/REFACTOR 行为压力测试、meta 反馈

## 安装

方式一：软链（推荐，便于跟随仓库更新）

```bash
git clone <本仓库> ~/projects/skills/skill-reviewer
ln -s ~/projects/skills/skill-reviewer/skill ~/.claude/skills/skill-reviewer
```

方式二：使用 `dist/` 下的 `.skill` 打包文件（zip 格式），解压到对应 runtime 的 skills 目录下的 `skill-reviewer/`。

> 上面的 `~/.claude/skills/` 是 Claude Code 的目录约定，仅作示例；其他支持 Agent Skills 的 runtime 请替换为其对应的 skills 目录。

## 使用

对支持 Agent Skills 的 Agent 直接说：

- 「帮我 review 一下 ~/projects/skills/foo」
- 「审查这个 skill 的质量」
- 「对这个 SKILL.md 给点反馈意见」

流程：确认目标 → 盘点文件与引用图 → 按清单审查 → 输出分级报告并集中询问判决项 →（可选）深度模式。只有在用户要求修改时才会动手改被审 skill。

## 目录结构

```
skill-reviewer/
├── skill/                          # runtime 技能包（软链目标）
│   ├── SKILL.md                    # 主工作流
│   └── references/
│       ├── review-checklist.md     # 23 项审查清单
│       ├── report-template.md      # 分级报告模板 + 评分卡
│       ├── enhanced-review.md      # 增强 Review 模式
│       └── behavior-testing.md     # 实测验证模式
├── dist/                           # 打包产物（.skill）
└── README.md
```

## 版本

版本由 git tag 管理，当前 `v0.4.0`。

- `v0.1.0` — 初始版本：三层清单、拆分审查、判决机制、增强 Review
- `v0.2.0` — 用自身对自身做了一轮完整自审（含子代理拆分实战），修复拆分职责对齐等 3 个 P1；新增机械层校验；表述改为工具中立
- `v0.3.0` — 博采众长：吸收 darwin-skill、obra/superpowers、philschmid、skill-validator、verdict/skillscore 的做法，新增实测验证模式、安全扫描、Runtime 中立性/非目标/读取条件检查、评分卡
- `v0.4.0` — 逻辑一致性强化：面向"被多个 LLM/Agent 来回修改"的 skill，新增 2.16 结构一致性（术语漂移/近似重复/结构断裂）；2.2 补决策判据与完成判据检查；2.12 隐式逻辑走查放开为对所有 skill 执行（对话约束沉淀仍限构建对话）；评分卡新增"逻辑一致性"维度（8 维）

## 设计参考

- darwin-skill（本机同门技能）— Review 维度、runtime 卫生、评分卡
- [obra/superpowers](https://github.com/obra/superpowers) — RED/GREEN/REFACTOR 子代理行为测试法
- [Testing Claude Skills](https://www.philschmid.de/testing-skills) — 触发率正负样本评测
- [skill-validator](https://github.com/agent-ecosystem/skill-validator) — 分级 token 阈值、关键词堆砌检测
- [Anthropic Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) — 渐进式披露、time-sensitive 信息规避
