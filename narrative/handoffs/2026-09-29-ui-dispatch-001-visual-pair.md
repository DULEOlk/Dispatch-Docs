# UI-DISPATCH-001：环形卷宗与高清清单 UE 视觉小步

## 身份与范围

- 日期/时区：2026-09-29 / Asia/Shanghai。
- 用户目标：采用截图中的环形夹子版双页卷宗，并把它和此前的高清竖版交付清单一起放进 UE 观察。选择的是生成版环形卷宗 `exec-2b1736b2-3681-4afe-81fb-5f192a64e204.png`（1558×1010）；先前用户 3116×2018 缺口卷宗未导入。
- 项目/分支/HEAD：`F:\Epic Games\Unreal Projects\Dispatch`；`main`；开工时 `11939c7`。无提交/推送。
- 子任务及父级：既有 `UI-DISPATCH-001`；`DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。
- 不覆盖：委托/NPC 内容、三槽、物品/背包、选择回填、正式派遣、存档、桌面平移逻辑及其他 UI 总设计项。
- 开工条件：两张透明 PNG 可读且 alpha 有效、当前两个 Image/旧画刷可识别、UE 编辑器未运行、备份两个 Widget；已满足。
- 退出条件与可见成果：新 Texture/Widget 从独立 UE 进程重载，聚焦测试通过，初始卷宗和右移清单两张 1920×1080 UE 画面可查看；均满足。用户原生窗口观感另行验收。

## 工作区与实施

- 开始已有大量源码、Blueprint、美术脏改及未跟踪资产（包括 `WBP_CommissionDispatch` 与 `WBP_PreparationSelection`）；均保留，不重置、不删除旧纹理。开工执行 `git status --short` 与相关 C++ diff。目标两个 PNG 文件此前不存在。
- 新增源 PNG 与 UI Texture：`Content/UI/Commission/Background/T_DispatchDossier_RingClip.*`、`Content/UI/Commission/PreparationSelection/T_PreparationSelection_FolioHD.*`。源 PNG 与用户选中/提供文件 SHA256 相等。
- `WBP_CommissionDispatch.Image_BackgroundDossier` 只改为环形卷宗 Texture；`WBP_PreparationSelection.Image_SelectionBoard` 只改为高清清单 Texture。迁移工具增量位于 `Source/DispatchEditor/Private/CommissionDeskPanCommandlet.cpp`，断言/双视角渲染增量位于 `Source/DispatchEditor/Private/Tests/CommissionDeskPanTests.cpp`。其余文字、尺寸、槽位、动画和业务 C++ 未改。
- 两个 Widget 的修改前备份和资源清单：`Saved/CodexBackups/DossierFolioVisualPair_20260929_182019/`。旧 V3 清单及旧卷宗 Texture 保留。`docs/` 是本地 Git 忽略资料，不强制提交。

## 完成与验证

- 设计选择：环形夹子和用户高清竖版清单已明确；并非最终视觉定稿。
- 资产接入：两张新纹理与两个画刷已经保存；UE 独立进程能重载。
- TDD：新素材路径/导入尺寸断言先对旧资产 RED（`Saved/Logs/DossierFolio-RED.log`）；导入后 `SelectionFolioLayout` GREEN。
- 构建：`Build.bat DispatchEditor Win64 Development -Project=... -NoUBA -Log=Saved/Logs/DossierFolio-BuildFinal.log` 退出 0。无 `-NoUBA` 的初次构建遭 `C:\ProgramData\Epic\UnrealBuildAccelerator` 权限错误，已中止；其自生成的 62 MB 失败日志已删除，未删资产。
- 导入：`-run=CommissionDeskPan -ApplyDossierFolioVisualPair -NullRHI -DDC-ForceMemoryCache` 输出 `DossierFolioVisualPair SAVED`，但全局 DDC 无可写节点使该进程退出 1；不能将此调用报告为成功。后续独立图形 UE 进程重载并验证资产。
- 聚焦：`Automation RunTests Dispatch.Widget.CommissionDispatch` 11/11 成功、退出 0。最终日志 `Saved/Logs/DossierFolio-FinalTests.log`；首次重载通过日志 `Saved/Logs/DossierFolio-FocusedGREEN.log`。
- 宽回归：`Automation RunTests Dispatch.Widget` 69 成功、3 失败、退出 1；失败为历史记录中的 `CommissionSelection.HoverDimsPeers`、`CommissionSelection.IdleDimOption`、`CommissionUIBase.Behavior`，见 `Saved/Logs/DossierFolio-WidgetSuite.log`。不称全套通过。
- 实际 UE 渲染且已开发侧目视：`Saved/Automation/CommissionDeskPan/Dossier_1920x1080_RingFolioHD.bmp`（初始视角）、`Sheet_1920x1080_RingFolioHD.bmp`（平移后）；四分辨率清单渲染亦由聚焦测试生成。白色委托图与 NPC 肖像是无数据测试夹具的占位，不是新底图的背景。
- 人工验收：用户尚未在原生窗口确认色调、层次、点击或清晰度。总设计项不标完成。

## 风险与下一步

- 环形卷宗来自图像生成预览，1558×1010，不是用户先前 3116×2018 高清缺口版。当前窗口约 1545×1006 逻辑像素，尺寸接近原生；更高 DPI 的实际锐度仍需玩家窗口目视。
- 清单新纸边/装订形状与 V3 不同，但控件未重排；当前 UE 截图可供用户判断是否需要后续微调。不要从自动化通过推断材质已获审美批准。
- `git diff --check` 在本机 Git 的 `sh.exe` 信号管道权限错误下未取得结果；构建与测试证据独立取得。
- 下一个最小动作：让用户查看两张 UE 预览并在编辑器/PIE 实际看一次初始和右移视角；根据具体截图只修有问题的材质或定位，不顺手扩展业务功能。
- 全部 UE/build/test 进程已结束。STATUS 与 IMPLEMENTATION_MATRIX 已同步；主要子任务仍是 `UI-DISPATCH-001`，其他任务未并行。
