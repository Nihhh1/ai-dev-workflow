<!-- 把下面这一节追加到项目 AGENTS.md（或 CLAUDE.md）末尾 -->

## 需求开发工作流

- 新需求或功能迭代按 `dev-workflow` 技能执行：需求完整性检查（`requirement-check`，缺口用 `grilling` 追问）→ 任务与测试拆分（`task-breakdown`）→ 逐个任务实现（`plan-execute`）→ 变动文档（`change-report`）→ 代码审查（`ai-code-review-enhanced`）→ 更新模块文档并清理工作区。
- 项目相关的命令、目录、规范、模块文档位置和提交约定写在仓库根目录 `.dev-workflow.md`。
- 进度以工作区中 `plan.md` 的勾选状态为准；中断或换窗口后说“继续开发”，从第一个未勾选任务接着做。
- 开发中途需求变化：先改 `requirement.md`，再调整 `plan.md`，最后改代码。不允许只改代码不改文档。
- 小改动（单点 bug、文案、单个配置读取）可以跳过工作区，直接修改并测试；模块规则有变化时仍需更新模块文档。
