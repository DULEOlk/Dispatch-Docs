# UI-DISPATCH-001：清单左侧阴影收近加深

## 身份与范围

- 日期/时区：2026-10-01，Asia/Shanghai。
- 用户目标：左侧页签/底板边缘仍显悬浮，希望阴影更近、更黑。
- 授权：仅现有 `Image_FolioShadowLeftContact` 强度和位置；不改卷宗、右下阴影、纸张/桌面、内部 UI 或派遣数据。
- 项目/分支/HEAD：`F:/Epic Games/Unreal Projects/Dispatch`，`main`，`211637638e96271d57f09041172bd56174c16db9`。
- 子任务 `UI-DISPATCH-001`；父级 `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。不覆盖其他剧情/玩法总项。
- 开工条件：用户明确给出局部外观反馈；仅前三项未提交阴影文件在工作区，UE 进程未运行，已满足。
- 退出条件：该层更贴近左外沿且更深，聚焦验证通过，用户可在原生窗口复看；审美验收尚待用户。

## 工作区

- 开始时三项已有修改：`WBP_CommissionDispatch.uasset`、`CommissionDeskPanCommandlet.cpp`、`CommissionDeskPanTests.cpp`；未撤销或触及其他文件。
- 本轮仍只改这三项：左侧层从 -12×-4 调到 -6×-2；RGBA 由 (.03,.016,.008,.28) 调为 (.012,.006,.003,.42)；定向作者工具与断言同步。
- 备份 `Saved/CodexBackups/FolioLeftContactTune_20261001_003824/`；未提交、未推送，`docs/` 保持本地忽略。

## 完成与验证

- 设计：更短、更深的左侧接触影，不增加大范围灰晕。
- 实施：仅上述阴影参数；右下、卷宗、材质和交互未改。
- 自动：新断言在旧资产上两项预期 RED（`Saved/Logs/FolioLeftShadowCloser-RED.log`）；DispatchEditor Development 构建退出 0；`-TuneFolioLeftContactShadow` 保存退出 0（`FolioLeftShadowCloser-Apply.log`）；独立重载 `Dispatch.Widget.CommissionDispatch` 13/13、退出 0（`FolioLeftShadowCloser-Focused.log`）。
- 画面：开发侧检查 `Saved/Automation/CommissionDeskPan/Sheet_1920x1080.bmp`；WidgetRenderer 偏亮，不能代替原生窗口。
- 宽回归：`Dispatch.Widget` 71/74，进程退出 1；失败仍为 `CommissionSelection.HoverDimsPeers`、`IdleDimOption`、`CommissionUIBase.Behavior`，无新增失败（`FolioLeftShadowCloser-WidgetRegression.log`）。`git diff --check` 退出 0，仅 LF→CRLF 提示。
- 四类状态：设计已确认；资产/代码已实施；聚焦自动通过、宽回归非全绿；用户人工外观待验收。当前受影响聚焦项已重验，父级总体不升级。

## 未完成与风险

- 用户仍需在真实窗口确认左侧接地感和颜色是否过重。
- `Dispatch.Widget` 旧三项失败本轮仍在，不在本视觉小步修复。
- 当前无其他资产删除、提交或推送；执行进程结束后核对。

## 下一步

- 请用户在原生窗口看效果；仅按反馈继续微调。
- STATUS 与 IMPLEMENTATION_MATRIX 已同步；主要子任务仍为 `UI-DISPATCH-001`，父级未完成。
