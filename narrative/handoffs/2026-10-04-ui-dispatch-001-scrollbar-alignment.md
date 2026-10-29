# 交付列表滚动条粗细与对齐修正

## 身份与范围

- 2026-10-04 Asia/Shanghai，F:/Epic Games/Unreal Projects/Dispatch，main，HEAD 21163763。
- 子任务 UI-DISPATCH-001，父级 DEMO-COM-001、DEMO-VIS-001、DEMO-AUTH-001；唯一主要实施任务不变。
- 用户目标：使用其四张素材，修正长而粗的滑块及箭头/轨道/滑块错位。开工条件为用户原生截图、现有Widget和源码可核对，已满足。退出条件为显示同轴、短滑块、真实滚动交互可用；自动视觉已满足，人工PIE交互待验收。
- 不覆盖正式背包/线索候选规则、DataAsset、派遣、存档或其他页面。

## 工作区

- 开始检查git status、相关diff和未跟踪资产，全部保留；未提交/推送。
- 修改Source/Dispatch/Public/Widget/PreparationSelectionWidgetBase.h与Private/Widget/PreparationSelectionWidgetBase.cpp：绑定独立轨道/滑块、按ScrollBox偏移同步滑块、鼠标轨道拖动；保留鼠标滚轮及上下按钮逻辑。
- 修改Source/DispatchEditor/Private/CommissionDeskPanCommandlet.cpp的定向作者入口，并保存Content/BluePrints/Widget/Menu/Quest/WBP_PreparationSelection.uasset。第一次布局备份Saved/CodexBackups/PreparationScrollbarGeometry_20261004_225405/；关闭原生条前备份Saved/CodexBackups/PreparationNativeScrollbar_20261004_231316/。四张用户Texture没有重导或修改。

## 完成与验证

- 根因：UE 5.6 UScrollBox::SynchronizeProperties先设置ScrollBarVisibility，随后AlwaysShowScrollbar=true令SScrollBar重新设为Visible；四个列表项几乎填满视窗，原生滑块覆盖用户素材并接近整条轨道长度。第一次几何修改后真实渲染仍复现粗条；关闭AlwaysShowScrollbar并折叠原生条后再次渲染消失。
- 设计与代码：轨道7×306、滑块9×120、两箭头共用x=578.5中心线。轨道UImage负责鼠标输入，拖动映射到ScrollBox最大偏移；NativeTick把滚轮/按钮偏移同步到滑块。静态图始终显示短滑块，内容不足时拖动不会产生位移。
- 自动：Development构建Saved/Logs/UBT-PreparationScrollbarGeometry.log与UBT-PreparationNativeScrollbar.log退出0；DebugGame构建Saved/Logs/UBT-PreparationScrollbarGeometry-DebugGame.log退出0。两个作者日志PreparationScrollbarGeometry-Apply.log与PreparationNativeScrollbar-Apply.log均报告SAVED；本机DDC无可写节点使进程退出1。第一次真实RHI渲染测试PreparationScrollbarGeometry-Render.log通过，但截图仍有叠绘；第二次PreparationNativeScrollbar-Render.log中DeskPanRenderedGeometry成功且命令退出0。最终图Saved/Automation/CommissionDeskPan/Sheet_1920x1080.bmp已目视检查：没有粗长原生滑块，素材同轴。先前nullrhi渲染崩溃由改用真实RHI绕开。
- 人工：用户实际UE窗口及多于四条隔离预览数据的滚轮/拖动/按钮点击尚待验收。相关旧验证仍需要重验，不能把定向视觉测试升级为整体人工通过。

## 未完成与下一步

- 在UE重开交付页查看实际DPI下的笔触宽度和对齐。临时使用6条以上隔离预览数据，检查拖动轨道时列表与滑块同步、鼠标滚轮及上下按钮同步；不要改正式DA。
- 如视觉仍需微调，只调整同轴的7/9像素宽度与素材UV边界，保留用户原PNG。
- 所有工具进程已结束；STATUS和IMPLEMENTATION_MATRIX已同步，总设计项不升级。
