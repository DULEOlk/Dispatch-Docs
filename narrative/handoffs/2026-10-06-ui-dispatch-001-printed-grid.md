# 三格圆弧与竖分隔

## 身份与范围

- 2026-10-06 Asia/Shanghai；`F:/Epic Games/Unreal Projects/Dispatch`，`main`，开工 HEAD `21163763`。
- 子任务 `UI-DISPATCH-001`；父级 `DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001`。
- 用户指出原版纹饰整图铺入后圆像椭圆，参考预览三格有竖线、排列紧凑。用户确认分层矫正和淡墨竖分隔方案。
- 仅准备槽印刷布局；不含原 PNG 重绘、物品纸卡、按钮命中区域、正式 DA、库存/派遣事务、其他页及父级 Demo 完成。
- 开工条件：用户已确认方案；退出条件：可回退 Widget、真实三格截图、圆弧/横纹/竖线/命中定向证据、用户原生审美。前三项已完成，人工待验收。

## 工作区

- 原工作区大量脏源码、蓝图、美术和未跟踪文件均保留；未重置、提交、推送或删除用户资产。
- `Source/DispatchEditor/Private/PreparationPrintedCardAuthor.cpp` 新增受状态守卫的 `ArrangeOriginalPreparationGrid`、`TuneOriginalPreparationGrid`，仅显式命令写入。入口在 `CommissionCardEffectsCommandlet.cpp`。两次保存前分别备份 `Saved/CodexBackups/PreparationPrintedGrid_20261006_232311/` 和 `PreparationPrintedGridCircle_20261006_233603/`。
- `WBP_PreparationSlot.uasset`：原 `Image_Registration` 改为原图圆弧 UV 裁取；新 `Image_RegistrationTopRule` / `BottomRule` 从同一原图裁取，上下横纹跨格；`Border_ColumnDivider` 在第1、2格右缘可见，第3格隐藏。四角原图和物品纸卡保持原样。
- `PreparationSlotWidgetBase.h/.cpp` 让三段纹饰共享既有悬浮/按下明暗，按槽号控制竖线；`PreparationSlotWidgetTests.cpp` 新增 PrintedGrid，并更新原过期的“整图横向铺满”断言为上横纹跨格断言。

## 完成与验证

- 设计：方案已获确认；最终审美未验收。代码/资产：已写入，原 PNG 未编辑。
- TDD：`PreparationPrintedGrid-RED.log` 在旧资产上因缺上下横纹按预期 RED；首轮真实 RHI 8项中7项通过，旧 CornerFeedback 过期断言失败。视觉发现圆略竖长、分隔线偏淡；`PreparationPrintedGrid-TuneRED.log` 针对二次比例/强度按预期 RED。二次微调后最终 Development 构建退出0。
- 两次作者日志 `PreparationPrintedGrid-Apply.log`、`PreparationPrintedGrid-TuneApply.log` 均报 SAVED/命令本体 result 0；整体进程受既有 DDC 无可写节点影响退出1，不能称作者进程成功退出。
- 独立真实 RHI 定向过滤器 `Dispatch.Widget.PreparationSlot+Dispatch.Widget.CommissionDispatch.DeskPanRenderedGeometry+Dispatch.Widget.CommissionDispatch.PreparationSlotRow`：`Saved/Logs/PreparationPrintedGrid-FinalVerify.log` 8/8通过、退出0。空态/物品+悬浮书页 `Saved/Automation/CommissionDeskPan/PrintedCard_Empty.png`、`PrintedCard_FilledHoverEmpty.png` 已目视检查；孤立控件多态在 `Saved/Automation/PreparationSlotVisual/PrintedGridFinal/`。
- 宽 `Dispatch.Widget`：NullRHI 首次在 `CommissionDeskPanTests.cpp:604` 的实际渲染调用访问冲突，未形成有效宽结果。真实 RHI 重跑见 `Saved/Logs/PreparationPrintedGrid-WideRHI.log`，73/79通过、退出1；6项失败为 `PreparationCommitFlow`、`PreparationContentHierarchy`、`PreparationListPresentation`、`HoverDimsPeers`、`IdleDimOption`、`CommissionUIBase.Behavior`，均不在本轮印刷图层范围。MovieScene handled ensure 亦在宽日志中，不能称全绿。
- 人工：原生窗口大小下圆弧、竖线深浅及整组秩序待用户确认。

## 未完成与下一步

- 本轮工具进程已结束。下一步请用户对照真实书页图在原生窗口复看；如需再调，仅微调圆弧/横纹/分隔线的几何和透明度，不重绘原 PNG 或改交互。
- `STATUS.md`、`IMPLEMENTATION_MATRIX.md` 已同步；唯一主要实施子任务仍 `UI-DISPATCH-001`，父级整体状态不升级。
