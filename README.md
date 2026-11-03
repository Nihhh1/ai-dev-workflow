# AI 需求开发工作流

一套独立、可搬运的 AI 开发流程 SKILL。把 OpenSpec 的“提案 → 实现 → 归档”拆成可单独替换的环节，去掉只写不读的历史归档，只维护一份描述功能当前状态的模块文档。

思路来源：《拆开 OpenSpec：我理想中的 AI 开发流程》（nihhh1.github.io，2026-11-01）。

## 流程

```
① 需求完整性检查   requirement-check   需求文档（已经过 grill-me）→ 检查清单 → 只追问缺口 → requirement.md
② 任务与测试拆分   task-breakdown      requirement.md → plan.md（任务 + 测试矩阵 + 执行记录）
③ 按文档逐个实现   plan-execute        每完成一个任务在 plan.md 勾选；中断后从第一个未勾选任务继续
④ 生成变动文档     change-report       changes.md：改了什么、实现逻辑、测试结果、风险
⑤ AI 代码审查      ai-code-review-enhanced（可替换）→ review.md；必须修正的问题回到 ③
⑥ 归档清理         模块文档已在 ③ 的最后一个任务更新；经确认后删除工作区
```

总入口是 `dev-workflow`：每次调用先读项目配置和工作区文件，判断进行到哪一步，再交给对应技能。

## 目录

```
ai-dev-workflow/
├── README.md
├── skills/                          # 复制或软链到 AI 工具的 skills 目录
│   ├── dev-workflow/                # 总编排：阶段判断、断点续做、规模分级
│   │   └── templates/dev-workflow.config.md   # 项目配置模板
│   ├── requirement-check/           # 需求完整性检查 + requirement.md
│   ├── task-breakdown/              # 任务与测试拆分（改编自 superpowers writing-plans，MIT）
│   ├── plan-execute/                # 按计划实现（改编自 superpowers executing-plans，MIT）
│   └── change-report/               # 变动文档
├── templates/
│   └── agents-md-snippet.md         # 追加到项目 AGENTS.md 的说明片段
└── examples/
    └── bagpackdefense_backend.dev-workflow.md # 一个 TSRPC 游戏后端项目的配置示例
```

## 依赖的外部技能

| 技能 | 用在 | 说明 |
|------|------|------|
| `grilling` / `grill-me` | ① | 来自 [mattpocock/skills](https://github.com/mattpocock/skills)，按依赖顺序逐轮追问并给推荐答案 |
| `ai-code-review-enhanced` | ⑤ | 规范审查 + 需求审查两个并行子代理；可在项目配置「审查技能」中替换 |
| `markitdown` | ①（可选） | 把 docx/xlsx/pdf 需求文档转成 Markdown |
| 项目自己的模块文档技能 | ③ 最后一个任务 | 例如 `generate-module-desc`；在项目配置「模块文档」中指定 |

## 安装

任选一种。

**用户级（所有项目可用）**——软链，便于统一更新：

```bash
# Claude Code
for s in ~/Documents/ai-dev-workflow/skills/*/; do ln -sfn "$s" ~/.claude/skills/"$(basename "$s")"; done
# Codex 等读取 ~/.agents/skills 的工具
mkdir -p ~/.agents/skills
for s in ~/Documents/ai-dev-workflow/skills/*/; do ln -sfn "$s" ~/.agents/skills/"$(basename "$s")"; done
```

**项目级（跟随仓库分发给团队）**——复制：

```bash
cp -R ~/Documents/ai-dev-workflow/skills/* <repo>/.agents/skills/    # 或 <repo>/.claude/skills/
```

## 接入一个项目

1. 安装技能（见上）。
2. 在仓库根目录创建 `.dev-workflow.md`：复制 `skills/dev-workflow/templates/dev-workflow.config.md` 填写，或参考 `examples/` 中的示例。也可以不建，第一次运行 `dev-workflow` 时它会扫描仓库起草一份，确认后写入。
3. 把 `templates/agents-md-snippet.md` 的内容追加到项目 `AGENTS.md`。
4. 把工作区目录（默认 `docs/dev`）加入 `.gitignore`。

## 使用

- 开始：把需求文档交给 AI，说“按 dev-workflow 开发这个需求”。
- 继续：新窗口里说“继续开发”或“继续 <需求名>”，会从 `plan.md` 第一个未勾选的任务接着做。
- 改需求：说明变化，AI 先改 `requirement.md`（版本 +1），再调整 `plan.md`，最后改代码。
- 小改动：直接说明要改什么，按规模分级跳过工作区。

每个需求的工作区：

```
<工作区目录>/<YYYY-MM-DD>-<feature>/
├── source/            原始需求文档
├── requirement.md     需求（唯一依据）
├── plan.md            任务总览复选框 + 测试矩阵 + 执行记录
├── logs/              测试输出
├── changes.md         变动文档
└── review.md          审查结果与处理结论
```

## 测试矩阵

`task-breakdown` 要求每份计划覆盖六类测试，不适用的必须写明原因：

| 类型 | 内容 |
|------|------|
| 功能 F | 每条验收标准至少一条 |
| 边界与异常 B | 时间边界、上限、配置缺失、非法参数、每个错误码 |
| 并发与幂等 C | 重复请求、并发、抢资源、重试 |
| 性能 P | 调用次数预算（防 N+1）、耗时预算、并发吞吐、基准脚本、线上观察点 |
| 兼容 M | 旧数据、发布期间新旧版本共存、配置热更 |
| 接口联调 I | 可选，需要真实环境 |

## 替换某个环节

各环节之间只通过工作区里的文件交接（`requirement.md` → `plan.md` → `changes.md` → `review.md`）。替换某个技能时，只要新技能读写同样的文件、保持 `plan.md` 中 `## 任务总览` 的 `- [ ] Task N：` 格式，其他环节不用改。

## 与 OpenSpec 的区别

| | OpenSpec | 本流程 |
|---|---|---|
| 需求澄清 | 提案阶段由 AI 生成 | grill-me 追问 + 完整性检查清单 |
| 文档格式 | 固定的提案/设计/规格/任务 | 每个环节的技能自己定义 |
| 小改动 | 也要走完整流程 | 按规模分级跳过环节 |
| 中途改需求 | 需要强制同步变更目录 | 只改本次的 requirement.md 和 plan.md |
| 归档结果 | 主规格 + 不断增长的历史变更 | 只有一份模块文档；工作区删除 |
| 改动历史 | 归档目录 | git 提交和 MR/PR |

## 来源与许可

- `task-breakdown`、`plan-execute` 改编自 [obra/superpowers](https://github.com/obra/superpowers) 的 `writing-plans`、`executing-plans`（MIT，Copyright (c) 2025 Jesse Vincent），许可证见各目录的 `LICENSE.txt`。
- 其他技能为原创。
