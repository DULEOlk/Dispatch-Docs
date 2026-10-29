# 可配置派遣生命周期阶段交接

## 身份与范围

- 日期/时区：2026-09-12，Asia/Shanghai。
- 用户本次目标与最新修正：继续完成 `Dispatch-Story-Flow-Plan-2026-09-07.md` 并持续记录；使用当前本地目录，不创建 worktree，不提交或推送，保留所有已有修改。
- 已授权范围 / 仍待确认范围：阶段 3 源码、自动化测试和本地文档已授权；未授权提交、推送、删除/覆盖资产。正式返回报告的 authored NPC 交互/对话内容属于后续阶段 7 联动样例，不在本阶段有限源码范围内。
- 项目绝对路径、分支、HEAD：`F:/Epic Games/Unreal Projects/Dispatch`；`main`；`fd705ffc92997c7c5994b1acf1466b78f3b554f7`。
- 当前阶段及具体小任务：阶段 3“可配置派遣生命周期”完成运行时合同与自动化闭环，下一阶段为阶段 4“日终条件、撰写封存和七日收尾”。

## 工作区

- 开始时已有的修改/未跟踪文件：承接阶段 1/2 的剧情源码、Campaign/Catalog 资产、未跟踪 `StoryActionExecutorPIETests.cpp`；另有 `WBP_PlayerNotebook.uasset` 修改、`001.uasset`/`009.uasset` 删除及四个未跟踪 Notebook Tab/UI 资产。详见 `STATUS.md` 2026-09-11 工作区复核。
- 本轮实际修改的文件和作用：
  - `Source/Dispatch/Public/Commission/CommissionDefinition.h`、`CommissionTypes.h`：新增执行日数、最早派遣日、末日策略、Reporting 状态、可靠出发/结果/物理返程快照字段。
  - `Source/Dispatch/Private/Commission/CommissionDefinition.cpp`、`CommissionRuleEvaluator.cpp`、`CommissionSubsystem.cpp` 及对应公开头：验证并复制生命周期配置；确认派遣原子提交；锁定同一次评估上下文；自动到期；物理返程与报告事务；末日未决查询；FIFO 状态通知。
  - `Source/Dispatch/Private/Story/Flow/DayFlowManager.cpp`、`Source/Dispatch/{Public,Private}/Story/StoryDirector.*`：换日后调用到期处理；向委托提供当前日与 Campaign 最大日数。
  - `Source/Dispatch/Private/Widget/UCommissionUIBase.cpp`、公开头：保留准备物/NPC reaction 展示但不在派遣前暴露 OutcomeID。
  - `Source/Dispatch/Private/Tests/{CommissionDefinitionTests,CommissionSubsystemTests,CommissionUIBaseTests,DailyWritingFlowTests,StoryCommissionBridgeTests}.cpp`：阶段 3 RED/GREEN 与回归覆盖。
  - `Source/DispatchEditor/Private/Tests/StoryActionExecutorPIETests.cpp`：CommissionFlow PIE 夹具补 Day1 日期权威，并精确声明 Preview Map 每次一次的既有 StateTree 启动错误。
- 用户资产、其他任务改动及保护说明：未主动修改或保存 Notebook、Inventory UI、地图、正式 Commission DA 或 Blueprint；所有并发/既有资产变更均保留。Campaign/Catalog 的已知准备接线来自上一阶段，不归本阶段创建。
- 是否提交/推送，若有列 commit：未提交、未推送。

## 完成与验证

- 已实现行为（与设计目标分开）：
  - 正式 Commission DA/旧 RuleSet 均支持 `ExecutionDays`、`EarliestDispatchDay`、`RejectDispatch/KeepUnresolved`，非法日数拒绝。
  - Day1 派出、执行 2 日的实例保存 DueDay=3；DayFlow 在换日事务中单次处理到期，重复检查无重复返回。
  - 出发实例保存 Definition、RuleSetID、NPC、准备物、Dispatch/Due 日、命中分支、完整评估上下文与 Outcome；出发后编辑 DA 不改变既有结算。
  - 到期立即释放精确 NPC 并应用身体/委托状态；线索、物品、命名结果和剧情信号只在报告结算时提交。Begin→Cancel→Begin→Resolve 可用，重复 Resolve 不发第二次奖励。
  - 默认拒绝跨 Campaign 末日的派遣；显式 KeepUnresolved 会保留 Active 未决实例供结局查询且不伪造返程。
  - 统一派发 UI 继续反馈准备槽与作者 PreviewReactionText，但 `CurrentOutcomeID` 始终不提前泄露。
- 实际执行命令、构建配置、测试过滤器：`Build.bat DispatchEditor Win64 Development -Project=... -WaitMutex -FromMsBuild`；`UnrealEditor-Cmd.exe ... -ExecCmds=Automation RunTests <filter>;Quit -unattended -nop4 -nullrhi -nosplash -DDC-ForceMemoryCache`。最终过滤器：`Dispatch.Commission`、`Dispatch.Widget.CommissionUIBase`、`Dispatch.Story.DayFlow`、`Dispatch.Narrative.GameplayBridge.Commission`、`Dispatch.NPC.Components.Behavior`、`Dispatch.Editor.Narrative.ActionExecutor.Commission`。
- 通过/失败/未运行数、退出状态：最终构建退出 0；最终回归分别 16/16、3/3、2/2、1/1、1/1、4/4 通过，目标自动化和宿主均退出 0。TDD 中预期 RED 及两次 PIE 夹具诊断失败均已记录在 STATUS，最终不存在未解决目标测试失败。
- 日志/证据路径：
  - `Saved/Logs/Narrative-Commission-Stage3-Regression.log`
  - `Saved/Logs/Narrative-Commission-UI-Stage3-Regression.log`
  - `Saved/Logs/Narrative-DayFlow-Stage3-Regression.log`
  - `Saved/Logs/Narrative-Commission-Bridge-Stage3-Regression.log`
  - `Saved/Logs/Narrative-NPC-Commission-Stage3-Regression.log`
  - `Saved/Logs/Narrative-ActionExecutor-Commission-Stage3-Green3.log`
  - RED/GREEN 细分日志见 `STATUS.md` 阶段 3 条目。
- 人工验证结果或未验证事项：未进行正式地图手动点击、真实 authored 返回报告对话、UMG 焦点/输入或 NPC 移动验收。完整“返回 NPC→听报告”的内容接线需在阶段 7 用可替换样例验证；阶段 3 只证明 Blueprint 可查询/Begin/Cancel/Resolve 合同。

## 未完成与风险

- 问题编号、触发条件、源码路径/函数/行号：
  - S3-M1：`/Game/Developers/StoryPreview/L_StoryUIPreview` 每次 PIE 启动会输出一次 `UStateTreeComponent::ValidateStateTreeReference: The State Tree asset is not set`；CommissionFlow 测试精确 expected 1 次。根因资产接线尚未在本阶段修改。
  - S3-M2：正式报告入口/对话内容尚无生产调用者；子系统 API 位于 `CommissionSubsystem.h`，Stage 7 需要从 NPC/UI authored 内容调用并验证完成/取消回执。
  - S2-B1（承接）：正式 `DA_StoryCampaign_Demo` 仍缺真实 StartingEpisode，BP Director 未激活该无效 Campaign；详见上一阶段交接。
- 是实际复现、静态追踪还是待确认推测：S3-M1 为 PIE 实际复现；S3-M2 为源码调用链静态追踪并由全局 `rg` 确认；S2-B1 为正式资产 contract 实际读回。
- 已尝试步骤及结果，避免下次重复：普通/延迟生成测试 Director 均不能消除早于 latent command 的 Preview Map StateTree 错误，证明不是测试 Actor；不要再次通过放宽 Commission 日期合同解决。三项 CommissionFlow 现各只接受同一消息恰好 1 次。
- 当前运行中的工具/进程/代理及日志，或明确“全部结束”：全部结束；无子代理；无 UnrealEditor-Cmd 进程。

## 下一步

- 下一个最小动作及验收方式：只读审计阶段 4 的 `DayFlowManager`、`DailyWritingSubsystem`、`WritingDesk`、`ABedInteractable` 与相关 StateTree 节点，建立“显式未满足原因 + 收尾阶段状态机”的首个 RED，不立即改资产。
- 依赖条件与是否需要用户输入：当前无需用户输入；若必须修改正式 Blueprint/地图、删除/恢复 UI 资产或选择真实剧情 StartingEpisode，再请求确认。
- 需修改的蓝图/资产（尚未执行）：阶段 4 首批无；正式返回报告内容和 Campaign StartingEpisode 均保留后续。
- 本轮决策及原因，链接 DECISIONS：遵守 `DECISIONS.md` 的日期权威、物理返回/报告获知分离和“不凭空编造正式内容”；未新建第二套 NPC 交互或日期系统。
- 已同步更新 STATUS：是。
