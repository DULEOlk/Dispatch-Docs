# 物品纸卡替换交接

## 身份与范围

- 日期：2026-10-07 Asia/Shanghai；项目`F:/Epic Games/Unreal Projects/Dispatch`，main / 21163763。
- 子任务UI-DISPATCH-001；父级DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001。
- 用户目标：已填物品卡纸张材质不协调，换成与卷宗页面一致的旧纸卡。
- 范围：新纸卡透明PNG/Texture及子Widget的纸卡、阴影素材和比例。不含空槽、正式DA、事务、其他页面和素材删除。

## 工作区与实现

- 开始已有大量源码、Widget、美术改动及两项删除，均保留；未提交/推送/清理。
- 根因之一：v1源图比例约0.815，而Widget显示宽高约1.13；重破边、花角、卷边被压扁且视觉喧闹。
- 内置图像生成先制作v2简洁透明纸卡，真实RHI对照显示偏亮偏平整；基于v2制作更深、更有纤维和边缘厚度的v3。当前绑定`Content/UI/Commission/PreparationSlot/PrintedCard/T_Slot_Parchment_v3.png/.uasset`；v1/v2完整保留。
- `Image_ItemCard`与`Image_CardShadow`共同改用v3，并置于X.12～.88、Y.11～.99的竖版区域；夹子/图标/文本/按钮不变。作者入口在`PreparationPrintedCardAuthor.cpp`，开关在`CommissionCardEffectsCommandlet.cpp`。
- 最终Widget备份`Saved/CodexBackups/PreparationCard_T_Slot_Parchment_v3_20261007_145444/`。完整最终生成提示保存在PrintedCard/README.md。

## 验证与状态

- v2和v3各自使用`Build.bat DispatchEditor Win64 Development -NoUBA`，均退出0；作者均报SAVED，但整体受本机DDC无可写节点影响退出1。
- v3独立真实RHI：`-ExecCmds="Automation RunTests Dispatch.Widget.PreparationSlot+Dispatch.Widget.CommissionDispatch.DeskPanRenderedGeometry;Quit" -RenderOffscreen`，8/8成功、退出0；日志`Saved/Logs/AgedCard-Verify.log`。
- 实图`Saved/Automation/CommissionDeskPan/FilledSelected_NoCornerBleed.png`，v2对照`Saved/Automation/CommissionDeskPan/PlainCardV2.png`，均已复看。v3 PNG 1122×1402，外部透明Alpha0。
- 四类状态：设计目标已落实为当前候选；代码/资产完成；局部自动GREEN；用户原生窗口人工审美待验收。宽组旧失败未重跑，父级总项不升级。
- 本轮进程全部结束；STATUS、IMPLEMENTATION_MATRIX、UI记录已同步。下步用户在原生窗口比较纸张与页纸的明暗质感。
