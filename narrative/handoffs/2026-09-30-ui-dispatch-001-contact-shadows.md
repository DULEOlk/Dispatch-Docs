# UI-DISPATCH-001：卷宗与交付清单接触阴影

## 身份与范围

- 日期/时区：2026-09-30，Asia/Shanghai。
- 用户目标：两块 UI 视觉上放在木桌上，不再悬浮。
- 授权范围：卷宗与清单底板后的阴影及必要作者工具/测试；不改内部 UI、木桌、美术材质、平移、物品数据与派遣逻辑。
- 项目/分支/HEAD：`F:/Epic Games/Unreal Projects/Dispatch`，`main`，开工 `211637638e96271d57f09041172bd56174c16db9`。
- 子任务/父级：`UI-DISPATCH-001` / `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`；不覆盖其他父级玩法与人工验收。
- 开工条件：用户确认“窄实影＋短柔影”、工作区干净、UE 无运行进程，均满足。
- 退出条件：两块底板在固定双视角有贴桌轮廓影、聚焦回归通过、用户可在原生 UE 窗口复看；前两项已完成，用户审美验收待做。

## 工作区

- 开始时 `git status --short --untracked-files=all` 为空；无其他用户脏改。
- 改动：`WBP_CommissionDispatch.uasset` 新增 `Image_DossierShadowContact/Soft`、`Image_FolioShadowContact/Soft`；`CommissionDeskPanCommandlet.cpp` 添加仅定向执行的 `ApplyContactShadows`；`CommissionDeskPanTests.cpp` 添加结构/遮挡/点击穿透断言。
- 复用现有 RingClip 与 FolioHD 透明底图轮廓，不生成新贴图。软影偏移 14×20、alpha .10；接触影偏移 6×9、alpha .27；四层在 `Canvas_DeskContent` 的桌面之上、实物底板之下，随桌面平移。
- 保存前的 Widget 及布局清单备份：`Saved/CodexBackups/CommissionContactShadows_20260930_200750/`。
- 未提交、未推送；Git `docs/` 忽略规则不变。

## 完成与验证

- 设计确认：贴桌的双层深棕轮廓影，不做方框黑晕。
- 代码/资产：已接入四层；仅展示且 `HitTestInvisible`。
- 自动验证：新 `ContactShadows` 在当前无阴影资产上先按预期失败四层缺失（`Saved/Logs/ContactShadows-RED.log`）；DispatchEditor 构建退出 0；作者工具首次因编辑器态内嵌清单实例无 WidgetTree 安全拒写，改从清单蓝图读后 `ContactShadows SAVED` 且退出 0（`ContactShadows-Apply2.log`）；独立重载测试退出 0（`ContactShadows-GREEN.log`）。`Dispatch.Widget.CommissionDispatch` 13/13、进程退出 0（`ContactShadows-Focused.log`）。
- 宽回归：`Dispatch.Widget` 74 项中 71 通过、3 失败、进程退出 1；失败仍为 `CommissionSelection.HoverDimsPeers`、`CommissionSelection.IdleDimOption`、`CommissionUIBase.Behavior`，与上一步同类历史失败一致（`ContactShadows-WidgetRegression.log`）。`git diff --check` 退出 0，只有既有 LF→CRLF 提示。
- 双视角画面：`Saved/Automation/CommissionDeskPan/Dossier_1920x1080.bmp`、`Sheet_1920x1080.bmp` 已目视检查轮廓与遮挡；WidgetRenderer 亮度与用户原生窗口不一致，不能代替人工色调验收。
- 对应矩阵的卷宗聚焦自动验证已重验；用户外观人工状态仍待确认，父级总项不升级。

## 未完成与风险

- 用户尚未在原生 UE 窗口评价阴影力度及接地感；若过暗/过淡仅调整四层透明度与偏移，不重画纸面。
- 宽 Widget 三项既有失败未触碰。
- 无 UE 进程留存。

## 下一步

- 请用户在卷宗和清单两个视角实机查看阴影；根据反馈小幅调整强度，再重跑聚焦与必要视觉检查。
- STATUS、IMPLEMENTATION_MATRIX 已同步；主要子任务 `UI-DISPATCH-001` 继续，父级未完成。
