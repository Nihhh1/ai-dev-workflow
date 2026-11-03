# 开发工作流项目配置（bagpackdefense_backend 示例）

> 接入时复制到仓库根目录并重命名为 `.dev-workflow.md`。
> 以下命令已在 2026-11-02 实测：单文件 mocha 与 tsc 类型检查均能稳定退出，退出码为 0。

## 基本
- 默认分支：develop（只在整个分支都属于本需求时作为审查基准；开发可以在任何分支进行）
- 工作区目录：docs/dev（`docs/` 已在 .gitignore 中，工作区只保存在本地）
- 文档与注释语言：简体中文

## 规范来源
- 编码规范：AGENTS.md（CLAUDE.md、GEMINI.md 是它的软链）。关键约束：新增/修改代码必须写中文注释（字段、方法、复杂逻辑）；禁止新增 `any`；按“代码组织要求”插入代码，不搬动无关代码；游戏配置通过 `TableMgr`，服务器配置通过 `configUtil`。
- 框架与领域规范：`.agents/skills/framework/`（IOC、数据读写、Redis、Redis 锁、Lua 缓存、TableMgr、错误码 i18n、活动框架、排行与排行奖励、GM 命令、日志上报等）、`.agents/skills/tsrpc/`。
- 目录结构：协议 `src/interfaces/protocols/<模块>/`（共用结构放 `msgInterface.ts` 的 `*Def` 命名空间）；API `src/interfaces/api/<模块>/`；业务 `src/service/<模块>/*Service`；实体 `src/model/<模块>/`；仓储 `src/dao/<模块>/`；新模块定义 `src/common/define/<模块>.ts`。
- 横切登记点：`src/infrastructure/ioc/types.ts` 与 `inversify.config.ts`、`src/interfaces/ptlIdDefine.ts`、`src/infrastructure/redis/redisKeys.ts`、`src/service/userClear/userClearDataHandler.ts`、`src/service/redDot/redDotService.ts`、GM 命令 `src/service/gmcmd/`。

## 命令
- 包管理器：Yarn 4.3.1，禁止使用 npm
- 单测命令：`yarn mocha --exit --require ts-node/register <file>`
- 全量测试命令：`yarn test:force-exit`
- 类型检查/构建命令：`yarn tsc --noEmit -p tsconfig.json`（全量约 8 秒）
- Lint 命令：无
- 测试通过标准：遵循 `coding-test-workflow` 技能，命令完成且退出码为 0；只看到 passing 但进程未退出不算通过

## 测试
- 测试目录与命名：`test/<模块>/<名称>.test.ts`，接口相关可放 `test/api/`
- 测试编写要求：新测试文件在 `describe` 中使用 `function () { this.timeout(10000); }`，不要用箭头函数
- 单元测试参考：`test/api/starWishActivity.test.ts`、`test/weightedDraw.test.ts`
- 集成测试参考：`test/anniversaryLotteryActivity/stock.integration.test.ts`（需要真实 Redis/Mongo，含 `Promise.all` 并发与 `perf_hooks` 计时写法）
- 接口联调：`api-test` 技能，通过 `../bpd_tools` 真实调用测试服接口；需要环境与测试账号授权
- 性能工具：`ExecutionTimeUtil`（`framework/execution-time-usage`，内网开启 `debugType.executionTime`）；Prometheus 指标 `bpd_mongo_collection_op_total`（按集合和操作统计 Mongo 调用次数）；基准脚本示例 `tools/benchmark-initial-value-clone.js`（`yarn bench:clone`）

## 需求检查扩展项
- 跨天与重置：每日/每周重置时间点、跨天时在线玩家的处理
- 活动期：活动框架下的开启、结束、领奖期，系列活动的期次标识，活动中途下架
- 资产：道具发放来源 ID（资产日志）、背包满或发放失败时是否走邮件补发
- 服务器维度：单服 / 跨服（crosskid）/ 赛季 / 公会维度；合服与赛季切换后的数据归属
- 红点与推送：哪些状态变化需要刷新红点或主动推送
- GM：测试需要的 GM 命令（改时间、加道具、重置数据）
- 角色清理：`userClear` 时本模块数据是否需要清理

## 模块文档
- 位置：`.agents/skills/module-description/<模块>-description/SKILL.md`
- 更新方式：`generate-module-desc` 技能
- 原则：只描述功能当前的样子；删除失效内容，不写“不再读取某配置”之类的历史说明

## 审查
- 审查技能：`ai-code-review-enhanced`
- 额外审查要求：参考仓库 `.ai-review.yml` 中的合入判定规则（Blocker / Major / Minor / Nit）

## 提交约定
- 规范：`git-commit-convention` 技能
- 授权：未经用户明确授权不执行 `git add` / `git commit`；严禁 `git push`，推送由用户手动完成

## 部署约束
k8s 部署：1 个更新服 Pod + n 个业务服 Pod，不停服滚动更新，客户端只连业务服。新旧 Pod 共存期间：协议只增不删、新字段可选；Redis key 结构和 Mongo 文档新旧版本都能读；定时任务注意只在应执行的 Pod 上运行。

## 规模分级覆盖
沿用默认。新活动、新协议、涉及资产发放或结算的改动一律按“大”处理。
