# UI-DISPATCH-001 深色装饰桌面阶段交接

## 身份与范围

- 日期/时区：2026-09-29～30，Asia/Shanghai。
- 用户目标与修正：将偏亮的当前纯木桌换成旧参考的深暖棕色，并恢复边缘纸堆、皮书、绳索、墨水瓶、笔羽装饰；不是只压暗色值。
- 已授权范围：保持 3:1 连续桌面、16:9 裁剪和平移，替换桌面图与 UMG 画刷。用户原生窗口视觉验收仍待确认。
- 项目：`F:/Epic Games/Unreal Projects/Dispatch`，分支 `main`，HEAD `11939c7`。
- 子任务：既有 `UI-DISPATCH-001` 的视觉小步；父级 `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。
- 不覆盖：NPC/委托正式文字、卷宗和清单内容、三槽、物品/背包、选择回填、真实派遣、存档、宽屏机制或其他 `DEMO-*` 总项。
- 开工条件：用户明确确认深色加装饰方案，满足。退出成果：新桌面在 UE 两个视角实际渲染、聚焦自动化通过；用户色调及装饰遮挡的人工作品验收尚未满足。

## 工作区

- 开始时已有大量未提交蓝图、C++、纹理和刻意未处理的旧资源删除，均保留。修改前执行 `git status --short`、`git diff --name-status` 并核对未跟踪目标路径。
- 新 `Content/UI/Commission/Background/T_DispatchDesk_DarkDecor.png` 和同名 `.uasset`；`WBP_CommissionDispatch.Image_PanDesk` 画刷从 `T_DispatchDesk_Pan` 改指新资源。原木桌贴图及卷宗/清单画刷保留。
- `Source/DispatchEditor/Private/CommissionDeskPanCommandlet.cpp` 增加受旧画刷路径约束的导入分支和受现有绑定约束的重导分支；`Source/DispatchEditor/Private/Tests/CommissionDeskPanTests.cpp` 增加资源绑定与 2172×724 尺寸回归。这两个源码文件在本轮开始前已经未跟踪，未删除或重建其既有内容。
- 原 Widget 备份 `Saved/CodexBackups/DarkDecorDesk_20260929_203552/`；首次生成源图与中间版源图保留在 `Saved/CodexBackups/DarkDecorDeskArtReview_20260929/`；两轮纹理 UAsset 重导前备份在 `DarkDecorDeskReimport_*`。
- 最终 PNG 为 built-in image_gen 对当前长桌面和用户旧桌图进行风格合成，再按 UE 裁剪反馈定向调整右侧道具位置；最终源 SHA256 `57FB3E7E4C313E9714B822C32ABC7F9EE5E708918C20B5BFB5BE34CFE4433A6D`。采用原用户图的深暖棕/边缘道具，无文本/界面；最终图 2172×724、RGB、无 Alpha。项目内 PNG 是正式源，不能只依赖生成图默认目录。
- 未提交、未推送；未清理任何其他用户资产。

## 完成与验证

- 四类状态：设计确认；本桌面资源及画刷已实施；聚焦自动化通过、宽回归留 3 既有失败；用户原生窗口色调与道具遮挡审美未验收。
- 测试先行：`DarkDecorDesk` 在旧画刷上失败，报实际仍引用 `/Game/UI/Commission/Background/T_DispatchDesk_Pan`，日志 `Saved/Logs/DarkDecorDesk-RED.log`。
- 构建：`Build.bat DispatchEditor Win64 Development -Project=... -NoUBA -Log=...`，`Saved/Logs/DarkDecorDesk-Build.log`、`DarkDecorDesk-ReimportBuild.log`，均退出 0。
- UE 导入/重导：`-run=CommissionDeskPan -ApplyDarkDecorDesk` 与 `-ReimportDarkDecorDesk` 均记录 `SAVED`/`REIMPORTED`，但全局 DDC 无可写节点使进程退出 1；不能把迁移进程本身报成全绿。独立进程重载新资产后通过回归。
- 最终聚焦：`Automation RunTests Dispatch.Widget.CommissionDispatch`，12/12 成功，进程退出 0；`Saved/Logs/DarkDecorDesk-FinalTests.log`。
- 宽回归：`Automation RunTests Dispatch.Widget`，73 项中 70 成功、3 失败，进程退出 1；既有 `CommissionSelection.HoverDimsPeers`、`CommissionSelection.IdleDimOption`、`CommissionUIBase.Behavior`；`Saved/Logs/DarkDecorDesk-WidgetSuite.log`。
- 用户可见证据：`Saved/Automation/CommissionDeskPan/Dossier_1920x1080_DarkDecor.bmp` 与 `Sheet_1920x1080_DarkDecor.bmp`。两者为 UE WidgetRenderer 渲染，非概念合成。后者仅露出墨水瓶上缘与笔羽末端，其余藏于清单后；请用户判断层次是否喜欢。渲染整体比源贴图偏亮，不能替代 PIE 色调检查。
- 源图中心采样 `(1200,270)`：旧桌面 RGB `92,56,32`，最终新桌面 `61,34,16`，证明贴图本身确实压暗。自动夹具白色大块来自没有绑定的委托/NPC 图片占位，不是桌面资源。
- `git diff --check` 因 Git `sh.exe` 无法创建 signal pipe（Win32 error 5）不可用；不是代码差异通过的证据。

## 未完成与风险

- 用户尚未在原生 PIE/Standalone 窗口判断最终深色是否与卷宗、清单协调，也未确认右侧道具藏在纸后的效果。
- 实际清单右侧可见木桌很窄；将整组道具再向右会被固定裁剪挡住，向左会被清单遮住。若用户不喜欢当前“夹在清单后”的层次，优先讨论缩小/分层道具，而不是修改平移距离或清单布局。
- 全局 DDC 路径不可写持续造成命令退出 1；独立重载/自动化成功与此分别报告，不通过改全局环境或引擎脚本绕锁。
- 当前无 UE/构建/测试进程仍在运行；未启动子代理。

## 下一步

- 请用户在原生窗口打开卷宗并点任一交付槽，检查两个视角的实际亮度、装饰是否干扰阅读及转场中是否自然；自动图可先审美比较。
- 若满意，记录人工视觉验收并继续原 `UI-DISPATCH-001` 待办；若不满意，只在用户指出的色调或道具层级上迭代，不自动扩大到数据/交互。
- `STATUS.md`、`IMPLEMENTATION_MATRIX.md` 已同步；当前主要子任务仍 `UI-DISPATCH-001`，父级 `DEMO-COM-001`/`VIS-001`/`AUTH-001` 均未宣称整体完成。
