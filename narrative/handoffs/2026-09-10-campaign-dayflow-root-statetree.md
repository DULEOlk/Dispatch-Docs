# Campaign、DayFlow 与 RootStateTree 阶段交接

## 身份与范围

- 日期/时区：2026-09-10，Asia/Shanghai。
- 用户本次目标与最新修正：继续完成归档七阶段剧情设计计划并持续记录；本小阶段处理 Campaign/Catalog/日期单一来源与 Campaign RootStateTree 接线。
- 已授权范围 / 仍待确认范围：已授权继续现有计划和修改业务代码；未授权提交、推送、删除或覆盖资产。正式关卡/蓝图人工验收仍待后续执行。
- 项目绝对路径、分支、HEAD：`F:\Epic Games\Unreal Projects\Dispatch`，`main`，`fd705ff`。
- 当前阶段及具体小任务：阶段 2；统一 Campaign→DayFlow 配置来源，并让 StoryDirector 启动 Campaign 的可选 RootStateTree。

## 工作区

- 开始时已有的修改/未跟踪文件：承接上一交接的 12 个已跟踪源码修改和未跟踪 `Source/DispatchEditor/Private/Tests/StoryActionExecutorPIETests.cpp`；当时 `Content/Config` 差异为空。
- 本轮实际修改的文件和作用：
  - `Source/Dispatch/Public/Story/Flow/DayFlowManager.h`、`Private/Story/Flow/DayFlowManager.cpp`：增加 Campaign 配置/安全回滚边界及有效 MaxDays 查询；Campaign Catalog Days/MaxDays 成为实时权威，旧字段保留为无 Campaign fallback。
  - `Source/Dispatch/Private/Story/StoryDirector.cpp`：Campaign 启动前配置已有 DayFlow；Campaign 会话成功后加载/启动可选 RootStateTree，非法运行状态诚实回滚。
  - `Source/Dispatch/Private/Tests/DailyWritingFlowTests.cpp`、`StoryGameplayBridgeTests.cpp`：Campaign DayFlow 单元与 Director 集成覆盖。
  - `Source/DispatchEditor/Private/Tests/StoryActionExecutorPIETests.cpp`：追加正式 BP Director + `ST_DemoStory` 的真实 PIE 根树覆盖。
  - `Source/DispatchEditor/DispatchEditor.Build.cs`：测试模块直接链接 `GameplayStateTreeModule`。
  - `Source/DispatchEditor/Private/Tests/StoryWorkspaceEditorTests.cpp`：正式 Campaign 资产安全迁移 contract。
  - `DA_StoryCampaign_Demo.uasset`、`DA_StoryContentCatalog_Demo.uasset`：准备接线 RootStateTree 与 7 个正式 Day DA；未激活无效 Campaign。
- 用户资产、其他任务改动及保护说明：本轮最终资产审计时，`BP_StoryDirector.uasset` 与写入前备份 SHA256 一致但仍相对 Git 基线修改；`WBP_PlayerNotebook.uasset` 在编辑器运行期间多次出现新写入时间，最终为 17:32:12；未跟踪 `Content/UI/Inventory_UI/Codex_图像_2026年9月10日_17_06_01.uasset` 实际创建于 17:35:38。它们可能来自加载期重编译/自动导入或用户并发编辑，无法可靠区分，全部保留不撤销。此前所有源码修改原样保留。
- 是否提交/推送，若有列 commit：未提交、未推送。

## 完成与验证

- 已实现行为（与设计目标分开）：Campaign 会话可把同一 Campaign 对象作为 DayFlow 实时配置来源；DayFlow 开始后拒绝换源且启动失败会安全回滚；Director 不再忽略 `Campaign.RootStateTree`；空根树仍兼容；根树不直接获得或修改 DayFlow 日期字段，只能继续使用既有 DayFlow API/StateTree tasks。正式 Campaign 已准备 Root 与 Days，但因缺 StartingEpisode 未切换 BP 默认入口。
- 实际执行命令、构建配置、测试过滤器：多次 `Build.bat DispatchEditor Win64 Development`；最终相关构建成功。最终运行 `Dispatch.Narrative.GameplayBridge`、`Dispatch.Story.DayFlow`、`Dispatch.Editor.Story.Flow.CampaignRootStateTree`、`Dispatch.Editor.StoryWorkspace.Editor.FormalCampaignAssetContract`。
- 通过/失败/未运行数、退出状态：最终回归分别 6/6、1/1、1/1、1/1 通过，自动化与进程退出均为 0。此前保留了预期 RED、缺 include、缺模块链接、Standalone/原生 Director 上下文不匹配、资产 contract RED 等失败证据；均已定位，正式内容缺口保持为显式安全状态。
- 日志/证据路径：`Saved/Logs/Narrative-DayFlow-CampaignConfiguration.log`、`Narrative-Director-DayFlow-Red.log`、`Narrative-Director-DayFlow-Green.log`、`Narrative-Campaign-RootStateTree-Red.log`、`Narrative-Campaign-RootStateTree-Green.log`、`Narrative-Campaign-RootStateTree-Green2.log`、`Narrative-Campaign-RootStateTree-Green3.log`、`Narrative-Campaign-RootStateTree-PIE.log`、`Narrative-Campaign-RootStateTree-PIE2.log`、`Narrative-Campaign-DayFlow-Regression.log`、`Narrative-Campaign-GameplayBridge-Regression.log`、`Narrative-Campaign-DayFlow-Rollback-Red.log`、`Narrative-Campaign-DayFlow-Rollback-Green.log`。
- 人工验证结果或未验证事项：尚未在正式 Demo 地图人工走完 Opening→Day 1、日期结束和第二日加载。正式 Campaign Root 与 Catalog Days 已确认，但 StartingEpisode 为空，因此 BP Director 有意不激活它。

## 未完成与风险

- 问题编号、触发条件、源码路径/函数/行号：正式 `DA_StoryCampaign_Demo` 的 `StartingEpisode` 为空；Notebook/BP Director/未跟踪图像资产差异来源待确认；正式地图人工验收未做。
- 是实际复现、静态追踪还是待确认推测：StartingEpisode 缺失由独立新进程 contract 实际读回；资产差异是 Git 实际状态但来源待确认；正式关卡表现待人工确认。DayFlow 启动失败配置残留已通过 RED/GREEN 修复。
- 已尝试步骤及结果，避免下次重复：不要再用原生 `AStoryDirector` 的 Standalone World 验证 `ST_DemoStory`，其 Schema 绑定正式 `BP_StoryDirector_C` 且会得到 Unset；正确 PIE 覆盖已通过。不要把 Unset 当作同步成功。
- 当前运行中的工具/进程/代理及日志，或明确“全部结束”：全部结束，无运行中的 UnrealEditor-Cmd 或构建进程。

## 下一步

- 下一个最小动作及验收方式：由真实剧情内容建立 `UStoryEpisodeDefinition` StartingEpisode 并注册必要引用；Campaign validation 通过后，contract 会要求 BP Director 默认激活 Campaign，再执行正式地图 Opening→Day 1 冒烟。
- 依赖条件与是否需要用户输入：需要用户提供/确认正式 StartingEpisode 内容，或明确允许创建仅用于迁移的占位章节；不能擅自编造正式剧情节点。
- 需修改的蓝图/资产（尚未执行）：新的正式 Episode DA、`DA_StoryCampaign_Demo` Episodes/StartingEpisode，验证通过后才修改 `BP_StoryDirector` 默认 Campaign。`BP_DayFlowManager` 无需清空 legacy fallback。
- 本轮决策及原因，链接 DECISIONS：沿用 `../DECISIONS.md` 的 Campaign 单一入口、DayFlow 独占日期事务、StateTree 只做协调决策；未新增矛盾决策。
- 已同步更新 STATUS：是。
