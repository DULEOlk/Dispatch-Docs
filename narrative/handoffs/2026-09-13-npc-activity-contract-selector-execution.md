# NPC Activity 合同、选择与显式执行交接

## 身份与范围

- 日期/时区：2026-09-13 / Asia/Shanghai。
- 用户本次目标与最新修正：按既有阶段路线继续阶段 5，不转回笔记本图标工作。
- 已授权范围 / 仍待确认范围：已授权继续此前记录的 NPC Activity 设计；本批实现独立 Activity Definition、纯选择器及到既有 SceneSubsystem 的显式执行。未保存正式 DA、蓝图或地图，未提交或推送。
- 项目绝对路径、分支、HEAD：`F:/Epic Games/Unreal Projects/Dispatch`，`main`，开始与结束 HEAD 均为 `ceae683fc8c65d25608f847a0194dce174a7a4d5`（`实现委托页面切换功能`）。
- 当前阶段及具体小任务：阶段 5；Activity 数据合同、确定性优先级、执行所有权与移动失败诊断。

## 工作区

- 开始时已有的修改/未跟踪文件：`git status --short` 为空；此前剧情、NPC、Notebook 和 UI 差异已进入用户现有 HEAD。
- 本轮实际修改的文件和作用：新增 `NPCActivityDefinition.*`（独立可编辑 DA 合同与校验）、`NPCActivitySelector.*`（纯选择逻辑）、`NPCActivitySubsystem.*`（显式 Scene 执行、运行状态、诊断与所有权清理）、`NPCActivitySelectorTests.cpp` 与 `NPCActivitySubsystemTests.cpp`（五项行为测试）。
- 用户资产、其他任务改动及保护说明：未修改或保存任何 `Content/`、地图、Widget、Campaign 或 NPC Blueprint；没有撤销、覆盖或清理用户资产。
- 是否提交/推送，若有列 commit：否；没有提交或推送。

## 完成与验证

- 已实现行为（与设计目标分开）：Activity 可限制 NPC、日次、DayFlow 阶段和剧情条件；运行状态先固定选择“剧情/委托占用 > 待报告 > 伤势休息 > 普通活动”，再以档内数字优先级选择，同分保留声明顺序。显式刷新会通过既有 SceneSubsystem 启动或替换活动，只清理自身仍拥有的 NPC/Scene；外部剧情 Scene 不被覆盖。无候选进入稳定等待；配置、同步启动和异步移动失败均保留诊断并清除自身失败 Scene 记录。
- 实际执行命令、构建配置、测试过滤器：`Build.bat DispatchEditor Win64 Development -Project=... -WaitMutex -FromMsBuild`；`Automation RunTests Dispatch.NPC.Activity`；最终 `Automation RunTests Dispatch.NPC`。
- 通过/失败/未运行数、退出状态：两个预期 RED 分别因缺 Definition 和 Subsystem 头文件失败；异步移动行为 RED 1/1 按预期失败。最终 DispatchEditor 构建退出 0；`Dispatch.NPC.Activity` 4/4 中间 GREEN；异步移动 1/1 GREEN；最终 `Dispatch.NPC` 22/22 通过，宿主退出 0。
- 日志/证据路径：`Saved/Logs/Narrative-NPC-Activity-Selector-Green.log`、`Narrative-NPC-Activity-Subsystem-Green.log`、`Narrative-NPC-Activity-AsyncMove-Red.log`、`Narrative-NPC-Activity-AsyncMove-Green.log`、`Narrative-NPC-Stage5-Activity-Regression.log`。
- 人工验证结果或未验证事项：未在正式 PIE 地图创建 Activity DA 或观察 NPC 移动；自动化不替代地图人工验收。

## 未完成与风险

- 问题编号、触发条件、源码路径/函数/行号：A1——当前 Activity 刷新是显式 API，尚未从日开始、NPC 状态/委托状态、剧情 Scene 结束自动重选；A2——独立 Activity DA 尚未加入 NPC/Catalog 制作入口或正式资产；A3——尚未在正式地图验证锚点、动画、互动打断和返回位置。
- 是实际复现、静态追踪还是待确认推测：A1/A2 为源码静态事实；A3 为未执行人工验收，不是已复现缺陷。
- 已尝试步骤及结果，避免下次重复：不要在 ActivitySubsystem 里复制 Scene 移动；当前已复用 `PlayNPCScene`。不要仅检查 SceneID 认为移动仍成功；快速路径必须读取 `ENPCSceneStatus::Failed`，已有 RED/GREEN 覆盖。
- 当前运行中的工具/进程/代理及日志，或明确“全部结束”：全部结束。

## 下一步

- 下一个最小动作及验收方式：先为“注册 NPC 后，NPCInfo 状态变化与 Commission Ready 通知会自动重选；外部剧情 Scene 结束后按最新状态恢复”写 RED，再增加事件订阅与解绑，不先制作正式资产。
- 依赖条件与是否需要用户输入：源码批次无需新输入；正式 Activity DA 的 NPC、Scene/Anchor 内容需要后续结合实际地图确认。
- 需修改的蓝图/资产（尚未执行）：后续预计为每个正式 NPC 创建 Activity DA、设置 Scene 行和 Anchor；本批未执行。
- 本轮决策及原因，链接 DECISIONS：沿用 `SYSTEM_DESIGN.md` 与 STATUS 的既定职责，Activity 负责选择，SceneSubsystem 负责表现，Commission/DayFlow 仍各自拥有真实状态与日期。
- 已同步更新 STATUS：是。
