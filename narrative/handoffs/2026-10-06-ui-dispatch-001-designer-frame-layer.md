# UI-DISPATCH-001 交接：交付框设计器选取层级

- 日期：2026-10-06
- 父级总设计：`DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`
- 子任务：`UI-DISPATCH-001`

## 用户目标

只调整 Widget 层级，使用户在 UMG 设计器中点击交付框即可修改样式；不新增运行时可调参数，不改变交互逻辑。

## 实施

`WBP_PreparationSlot` 的 `Image_Registration` 已设为 `Canvas_SlotLayers` 中 Z=20 的最上视觉层，并保持 `HitTestInvisible`。因此设计器可直接选中印刷框，运行时鼠标命中仍由 `Button_PreparationSlot` 处理。

## 证据

- 备份：`Saved/CodexBackups/PreparationSlotDesignerFrame_20261006_171805/WBP_PreparationSlot.uasset`
- 作者日志：`Saved/Logs/DesignerFrame-Apply.log`，`PreparationSlotDesignerFrame` 返回0
- Development 构建成功

## 未覆盖与下一步

未修改父卷宗布局、正式 DA、派遣事务或视觉参数抽取。用户需在 UE 中刷新/重新打开 `WBP_PreparationSlot`，点击框面确认可选中 `Image_Registration`，再进行样式调整。
