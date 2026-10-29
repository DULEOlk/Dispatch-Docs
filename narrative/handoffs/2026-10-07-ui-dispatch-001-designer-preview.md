# UI-DISPATCH-001 子控件设计器真实比例交接

## 身份与范围

- 日期/时区：2026-10-07，Asia/Shanghai。
- 用户目标：把 `WBP_PreparationSlot` 独立设计器中被拉宽的画布改为实际单格比例，方便手工修改纹饰。
- 授权范围：设计器预览尺寸。父级 `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`；子任务 `UI-DISPATCH-001`，不覆盖正式 DA、派遣事务、父级三格布局、运行时按钮和其他 Widget。
- 项目：`F:\Epic Games\Unreal Projects\Dispatch`；分支 `main`，开工 HEAD `21163763`。
- 开工条件：用户确认只改设计器比例；已满足。退出条件：可恢复备份、独立新进程读回预览尺寸、槽位交互不退化、用户原生设计器复看。前三项已取得；人工复看未完成。
- 用户可观察成果：重开 `WBP_PreparationSlot` 后画布应显示约 `224×217`，而非 `2714×800`。

## 工作区

- 开工时已有大量已修改/未跟踪源码、Widget、纹理；均保留，未清理、未回退，也未提交或推送。
- 本轮改动：`WBP_PreparationSlot.uasset` 的编辑器专用 `DesignSizeMode=Custom` 与 `DesignTimeSize=(224,217)`；`CommissionCardEffectsCommandlet.cpp` 增加作者入口；`PreparationPrintedCardAuthor.cpp` 增加备份写入函数；`PreparationSlotWidgetTests.cpp` 增加回归测试。
- 保存前备份：`Saved/CodexBackups/PreparationSlotDesignerPreview_20261007_105739/WBP_PreparationSlot.uasset`。未修改 `WBP_CommissionDispatch.uasset`。

## 完成与验证

- 设计已确认；代码/蓝图属性已写入；自动定向已验证；人工原生画面待验收。此次修改不提升任一 DEMO 父项的整体验收状态。
- `DispatchEditor Win64 Development` 构建退出0。新测试在旧资产上 RED（尺寸模式和尺寸两项断言失败），保存后独立新进程 `Dispatch.Widget.PreparationSlot` 7/7 成功、退出0，日志 `Saved/Logs/PreparationSlotDesignerPreview-RED.log` 与 `PreparationSlotDesignerPreview-GREEN.log`。
- 作者日志 `Saved/Logs/PreparationSlotDesignerPreview-Author.log` 报 SAVED、命令本体 result 0；进程因本机已有 DDC 无可写节点退出1，不把它当完整成功。
- 真实 RHI 宽回归 `Saved/Logs/PreparationSlotDesignerPreview-WidgetSuite.log` 已完成32项，27成功、5失败（`CommissionDispatch.PreparationCommitFlow`、`PreparationContentHierarchy`、`PreparationListPresentation` 与 `CommissionSelection.HoverDimsPeers`、`IdleDimOption`），均属于本次前已知失败；随后在 `CommissionUIBase.Behavior` 崩溃，等待进程已中断，余项未完成。
- 已完成的父级 `CommissionDispatch` 布局、命中、平移与渲染几何测试通过；未获得新的全套 GREEN。用户尚未在原生设计器打开预览确认外观。

## 未完成与下一步

- 用户重开独立 `WBP_PreparationSlot`，检查设计器左上角尺寸与圆弧比例，并在其层级树中点选纹饰。此变更不会让父级 `WBP_CommissionDispatch` 直接选中嵌套子 Widget 内的图片。
- 全部本轮工具进程已结束。STATUS 和 IMPLEMENTATION_MATRIX 已同步；`UI-DISPATCH-001` 仍是唯一主要进行中子任务。
