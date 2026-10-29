# UI-DISPATCH-001：独立交付列表行尺寸修正

## 身份与范围

- 日期/时区：2026-10-04，Asia/Shanghai。
- 用户本次目标与最新修正：用户在 UE Designer 截图指出新列表行尺寸异常；修正单行编辑预览大小。
- 已授权范围 / 仍待确认范围：修复已报告的新行控件问题；UE 原生重开和审美确认尚待用户。
- 项目绝对路径、分支、HEAD：`F:\Epic Games\Unreal Projects\Dispatch`，`main`，`21163763`。
- 当前阶段及具体小任务：`UI-DISPATCH-001` 的行 Widget 尺寸修复。
- 子任务 ID：`UI-DISPATCH-001`。
- 覆盖的父级 `DEMO-*` 编号：`DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。
- 明确不覆盖的相邻总设计项：正式物品/线索候选、背包、派遣、存档。
- 开工条件及是否满足：已有用户截图、作者代码、UE 5.6 设计器源码；满足。
- 退出条件和用户可观察成果：`WBP_PreparationEntryRow` 打开后只显示一条约 664×72 的行；其完整设计器和 PIE 表现待用户复看。

## 工作区

- 开始时已有的修改/未跟踪文件：沿用既有脏工作区，含用户父 Widget 和 UI 素材改动。新行资产在截图时有未保存星号；随后 UE 日志确认编辑器已关闭，磁盘资产时间仍为 12:36:25，因此修改的是磁盘旧行资产。
- 本轮实际修改的文件和作用：`Source/DispatchEditor/Private/CommissionDeskPanCommandlet.cpp` 增加宽度/Designer 模式并提供定向修复分支；`Content/BluePrints/Widget/Menu/Quest/WBP_PreparationEntryRow.uasset` 保存尺寸属性。
- 用户资产、其他任务改动及保护说明：父 `WBP_PreparationSelection.uasset` 与其它用户改动未改；新行资产保存前备份 `Saved/CodexBackups/PreparationEntryRowSize_20261004_131629/WBP_PreparationEntryRow.uasset`。
- 是否提交/推送，若有列 commit：否。

## 完成与验证

- 已实现行为（与设计目标分开）：行根 `Size_Row` 宽 664、高 72；生成类默认对象的 `DesignSizeMode` 为 `Desired`。新资产作者代码也给未来生成的行设置相同属性。
- 本轮四类状态变化（设计 / 代码 / 自动验证 / 人工验收）：尺寸目标确定；源码与资产写入；构建 0；未新增/运行测试；用户重开后的 Designer/PIE 尚未验收。
- 因本轮修改被标记“需要重验”的既有矩阵项：交付页列表视觉和运行时布局仍需人工复看；父类重构后的旧功能 GREEN 继续需要重验。
- 实际执行命令、构建配置、测试过滤器：`DispatchEditor Win64 Development -NoUBA -MaxParallelActions=2`；`UnrealEditor-Cmd -run=CommissionDeskPan -FixPreparationEntryRowSize -DDC-ForceMemoryCache`。测试过滤器：未运行。
- 通过/失败/未运行数、退出状态：C++ 构建退出 0；资产作者命令输出 SAVED，但因 DDC 环境错误退出 1；测试未运行。
- 日志/证据路径：`Saved/Logs/UBT-PreparationEntryRow-Size.log`、`Saved/Logs/PreparationEntryRow-Size-Author.log`（约 1067 行为 SAVED）。
- 人工验证结果或未验证事项：新设置尚未在原生 UE Designer 目视确认，PIE 多行与点击未查。

## 未完成与风险

- 问题编号、触发条件、源码路径/函数/行号：原 `CreatePreparationEntryRow` 分支只设根高，UUserWidget 默认 `FillScreen`；`CommissionDeskPanCommandlet.cpp`。UE Designer 使用生成类默认对象 `DesignSizeMode` 的证据见引擎 `SDesignerView.cpp` 与 `WidgetBlueprintEditorUtils.cpp`。
- 是实际复现、静态追踪还是待确认推测：过大预览有用户原生截图；根因通过作者代码和 UE 5.6 源码静态追踪；修后原生显示仍待确认。
- 已尝试步骤及结果，避免下次重复：定向作者命令在备份后保存资产并写出预期尺寸/模式；命令进程独立 DDC 报错不代表资产未保存。
- 当前运行中的工具/进程/代理及日志，或明确“全部结束”：构建与作者命令均已结束；UE 编辑器未运行。

## 下一步

- 下一个最小动作及验收方式：用户重新打开 `WBP_PreparationEntryRow`，检查工具栏预览模式显示“期望”、画布只包一行；再在 PIE 交付页检查条目宽度、选中与点击。若不符，按最新截图针对资产布局继续定位。
- 依赖条件与是否需要用户输入：需要 UE 原生复看结果；本轮不需要新的设计决定。
- 需修改的蓝图/资产（尚未执行）：无。
- 本轮决策及原因，链接 DECISIONS：沿用已有独立行 Widget 方案，预览模式设 Desired 以跟随行的期望尺寸。
- 已同步更新 STATUS：是。
- 已同步更新 IMPLEMENTATION_MATRIX：是；`DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001` 均不标完成。
- 当前主要进行中子任务 / 等待队列是否已同步：仍仅 `UI-DISPATCH-001` 进行中。
