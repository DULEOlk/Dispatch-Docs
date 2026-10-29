# UI-DISPATCH-001：交付清单左侧阴影

## 身份与范围

- 日期/时区：2026-09-30，Asia/Shanghai。
- 用户目标：右下阴影已可接受，左边仍悬浮；补左侧窄暗边，上边轻微带到，快速完成。
- 授权范围：现有清单底板后的轮廓阴影；不改卷宗、纸张、内部 UI、桌面、交互或业务数据。
- 项目/分支/HEAD：`F:/Epic Games/Unreal Projects/Dispatch`，`main`，`211637638e96271d57f09041172bd56174c16db9`。
- 子任务/父级：`UI-DISPATCH-001` / `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。不覆盖其他剧情/玩法总项。
- 开工条件：用户明确确认左侧补影；Git 仅有上一轮 3 项未提交阴影文件，UE 编辑器未运行，已满足。
- 退出条件：左边出现贴桌窄暗边、不产生大方框，聚焦验证通过；用户实际窗口审美仍待复看。

## 工作区

- 开始时已修改 `WBP_CommissionDispatch.uasset`、`CommissionDeskPanCommandlet.cpp`、`CommissionDeskPanTests.cpp`；未撤销或覆盖。
- 本轮在同三文件增加清单左侧轮廓阴影、定向备份作者工具与自动断言。
- 资产修改前备份 `Saved/CodexBackups/FolioLeftContact_20260930_234319/`；其他资产保持。
- 未提交、未推送；`docs/` 保持本地忽略。

## 完成与验证

- 设计：左边窄实影，顶部轻微，仍保留右下原接触/短柔影。
- 实施：`Image_FolioShadowLeftContact` 复用 FolioHD alpha 轮廓，680×850，偏移 -12×-4、alpha .28，位于移动桌面且底板之下，`HitTestInvisible`。
- 自动：新增真实 Widget 断言在旧资产上因缺层 RED（`FolioLeftShadow-RED.log`）；DispatchEditor Development 构建退出 0；定向命令 `-AddFolioLeftContactShadow` 保存退出 0（`FolioLeftShadow-Apply.log`）；独立重载 `Dispatch.Widget.CommissionDispatch` 13/13 通过、退出 0（`FolioLeftShadow-Focused.log`）。
- 人工：开发侧已看 `Saved/Automation/CommissionDeskPan/Sheet_1920x1080.bmp`，未以偏亮 WidgetRenderer 画面替代用户原生窗口验收。
- 宽回归：`Dispatch.Widget` 71/74，进程退出 1；失败仍为 `CommissionSelection.HoverDimsPeers`、`IdleDimOption`、`CommissionUIBase.Behavior`，与前版相同（`FolioLeftShadow-WidgetRegression.log`）。`git diff --check` 退出 0，仅有 LF→CRLF 提示。
- 四类状态：设计确认；资产/代码已实施；聚焦自动通过、宽回归非全绿；用户人工外观待验收。父级总体状态不升级。

## 未完成与风险

- 原生窗口暗调下左边强度及接地感待用户复看。
- 宽 `Dispatch.Widget` 的三项历史失败本轮仍在；不在本视觉小步修复。
- 无其他资产删除、提交或推送。

## 下一步

- 请用户在清单视角实机复看；仅按其反馈继续微调左侧。
- STATUS 与 IMPLEMENTATION_MATRIX 已同步；唯一主要实施子任务仍为 `UI-DISPATCH-001`，父级未完成。
