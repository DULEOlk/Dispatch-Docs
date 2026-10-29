# 交付列表滚动条试换

## 身份与范围

2026-10-04 Asia/Shanghai；项目 F:/Epic Games/Unreal Projects/Dispatch，main；UI-DISPATCH-001，父级 DEMO-COM-001、DEMO-VIS-001、DEMO-AUTH-001。用户提供窄版参考截图，要求制作列表右侧滚动条。沿用当前列表尺寸和数据，只改列表滚动条样式及上下箭头。

## 工作区与实现

保留既有未提交改动；新增 T_PreparationScrollbarReference.png/uasset，修改 WBP_PreparationSelection.uasset、PreparationSelectionWidgetBase.h/.cpp、CommissionDeskPanCommandlet.cpp。未提交/推送。修改前父 Widget 备份 Saved/CodexBackups/PreparationScrollbar_20261004_193214/。

作者入口 StylePreparationScrollbar 使用用户84×813参考图的局部UV作为轨道、滑块、上下箭头画刷；原生 ScrollBox 宽度17、上下各28间距、持续显示滚动条。箭头为 Button_PreparationScrollUp/Down，位置566,219和566,556，宽25高27；运行时点击使 ScrollBox offset增减91并向上夹到0。鼠标滚轮与拖动继续由原生 ScrollBox 负责。

## 验证与未完成

Development与DebugGame构建退出0，日志 Saved/Logs/UBT-PreparationScrollbar.log、UBT-PreparationScrollbar-DebugGame.log。作者命令行编译保存并报告SAVED，日志 Saved/Logs/PreparationScrollbar-Apply.log；但本机DDC不可写使进程退出1，不能记为全流程成功。未运行UI自动测试、未做UE原生拖动或点击验收。四个预览物品近乎填满视口，需临时增加第五/第六个预览项才能直观看到位移；不要将预览项当正式物品。

旧相关视觉与交互证据需重验。STATUS和MATRIX同步；唯一主任务仍UI-DISPATCH-001。下步原生预览确认参考材质、箭头点击、拖动及鼠标滚轮。
