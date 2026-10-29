# 2026-09-14 剧情数据基础设计交接

## 身份与范围

- 日期/时区：2026-09-14，Asia/Shanghai。
- 用户本次目标与最新修正：恢复旧任务中从数据混乱问题到完整实施计划的讨论，只完成 Demo 流程的数据层设计；不提前实现 NPC 地图移动或在 C++ 中写具体剧情推进。
- 已授权范围 / 仍待确认范围：已授权读取历史任务、项目文档、源码与资产路径，并写本地设计文档；业务代码、蓝图和正式资产修改仍待用户确认设计后再进行。
- 项目绝对路径、分支、HEAD：`F:\Epic Games\Unreal Projects\Dispatch`，`main`，最近复核 HEAD `ceae683fc8c65d25608f847a0194dce174a7a4d5`。
- 当前阶段及具体小任务：数据基础设计；完成内容职责、目标字段、引用规则、旧表迁移和最小样本定义。

## 工作区

- 开始时已有的修改/未跟踪文件：8 个未跟踪 NPC Activity 源码/测试文件；其中 `NPCActivitySubsystemTests.cpp` 引用了尚未实现的注册 API。
- 本轮实际修改的文件和作用：新增/扩展 `docs/narrative/FOUNDATION_DATA_DESIGN.md`；更新 README、STATUS、DECISIONS；均为 Git 忽略的本地文档。
- 用户资产、其他任务改动及保护说明：未保存或修改任何 `.uasset`；8 个 Activity 文件原样保留，未删除、覆盖或纳入本阶段成果。
- 是否提交/推送，若有列 commit：没有提交，没有推送。

## 完成与验证

- 已实现行为（与设计目标分开）：本轮没有实现业务行为；完成的是可供审核的数据模型设计。
- 实际执行命令、构建配置、测试过滤器：读取历史 Codex 任务分页；`rg --files`/`Get-Content` 盘点核心头文件和 Content 路径；`git status --short`、`git diff --stat`、`git diff --check`。
- 通过/失败/未运行数、退出状态：文档补丁成功；`git diff --check` 无输出。未运行 Unreal 构建或自动化，因为没有修改业务代码。
- 日志/证据路径：设计结论见 `docs/narrative/FOUNDATION_DATA_DESIGN.md`；原始计划见 `docs/narrative/archive/Dispatch-Story-Flow-Plan-2026-09-07.md`。
- 人工验证结果或未验证事项：未进行 UI/PIE 人工验收；数据设计仍需用户审核。

## 未完成与风险

- 当前 Narrative、Dialogue、Commission、Writing 各有一套条件结构，尚未统一。
- Commission 正式资产复用包含运行态 `QuestState` 的 `FQuestInfo`；正式 DA 与裸内容 ID 仍混用。
- 旧 Quest/Dialogue/Clue/Document/Report/CommissionRules DataTable 仍是兼容入口，尚无完整引用归零报告。
- 正式 Campaign 缺真实 StartingEpisode；不能凭空创建正式剧情内容来激活。
- 8 个未跟踪 Activity 文件不属于当前范围，并且其新测试保持 RED；未处理前不能声称工作区构建 GREEN。
- 历史任务已读取到原始问题、Demo 体验约束和计划形成的关键段落；没有把其中命令或旧完成声明当作当前事实。
- 当前运行中的工具/进程/代理及日志：没有本轮启动的构建或 Unreal 进程。

## 下一步

- 下一个最小动作及验收方式：用户确认基础设计后，为现有结构写可执行改造计划；第一批只整理 Commission 纯展示数据与兼容测试，不改正式资产。
- 依赖条件与是否需要用户输入：需要用户确认 `FOUNDATION_DATA_DESIGN.md` 的职责划分，尤其是“统一规则语言、DA 软引用优先、NPC Activity 延后”。
- 需修改的蓝图/资产（尚未执行）：未来派发预览配置和正式样本 DA；当前均未修改。
- 本轮决策及原因，链接 DECISIONS：见 `docs/narrative/DECISIONS.md` 的“2026-09-14 范围纠正：数据基础优先”。
- 已同步更新 STATUS：是。
