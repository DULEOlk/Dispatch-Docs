# UI-DISPATCH-001：交付页贴桌阴影增强

## 身份与范围

- 日期/时区：2026-09-30，Asia/Shanghai。
- 用户目标与最新修正：卷宗阴影可接受，右侧交付清单仍像悬浮；用户确认仅增强右侧。
- 授权范围：右侧底板背后的阴影及其定向测试/作者工具；左侧卷宗、内部 UI、木桌与业务逻辑不动。
- 项目/分支/HEAD：`F:/Epic Games/Unreal Projects/Dispatch`，`main`，`211637638e96271d57f09041172bd56174c16db9`。
- 当前阶段/子任务：视觉接地感修正，`UI-DISPATCH-001`；父级 `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。
- 不覆盖：委托文本、物品/背包、真实选择/回填、派遣、存档及其他总设计项。
- 开工条件：用户授权右侧定向修改、核对 Git 脏改和 UE 进程，已满足。
- 退出条件：右侧轮廓影可辨、不出现额外暗带，聚焦测试通过；开发侧已达到，原生窗口审美验收待用户复看。

## 工作区

- 开始时已有且本任务延续的修改：`WBP_CommissionDispatch.uasset`、`CommissionDeskPanCommandlet.cpp`、`CommissionDeskPanTests.cpp`，均来自前一步未提交的接触阴影工作；未跟踪文件无关改动未触碰。
- 本轮实际修改：右侧两层阴影的透明度、偏移及轮廓尺寸；在原命令工具中补充右侧定向调参/几何修复入口，调整新建阴影时的尺寸；更新 ContactShadows 测试。左侧阴影参数未改。
- Widget 定向备份：`Saved/CodexBackups/FolioContactShadowTune_20260930_213538/` 与 `Saved/CodexBackups/FolioShadowGeometry_20260930_222655/`。
- 未提交、未推送；用户其他资产及已有本地提交保持。

## 完成与验证

- 设计：右侧实体清单在深木桌上需要比左卷宗略强的近接阴影，不加大范围浮空光晕。
- 实施：接触层偏移 10×14、alpha .48；柔层 24×32、alpha .22。发现旧阴影将 FolioHD 纹理按 840×850 清单 Widget 拉伸，造成右侧宽暗带；已与真实 `Image_SelectionBoard` 680×850 尺寸及位置对齐。
- 自动：强度断言先 RED（`FolioShadowTune-RED.log`）；几何断言先 RED（`FolioShadowGeometry-RED.log`）；调整、重载后 `Dispatch.Widget.CommissionDispatch` 13/13 通过，进程退出 0（`FolioShadowGeometry-Focused.log`）。构建成功。宽 `Dispatch.Widget` 71/74，退出 1，三项失败仍为 `CommissionSelection.HoverDimsPeers`、`IdleDimOption`、`CommissionUIBase.Behavior`（`FolioShadowGeometry-WidgetRegression.log`）。
- 开发侧画面：`Saved/Automation/CommissionDeskPan/Sheet_1920x1080.bmp` 已目视检查，右侧多余宽暗带消失；WidgetRenderer 色调偏亮，不能代替用户原生窗口。
- 四类状态：设计已确认；代码/资产已实现；聚焦自动通过而宽回归非全绿；用户人工外观待复验。
- 本轮变更影响的聚焦项已重验；父级总项和真实派遣/数据验证不升级。

## 未完成与风险

- 实际 PIE/独立窗口的右侧接地感、阴影力度仍需用户评价；不得将开发侧预览认作人工通过。
- 三项宽回归既有失败未在本视觉小步处理。
- 无运行中工具/进程/代理。

## 下一步

- 用户在清单视角看实际画面，确认阴影是否足够贴桌、是否过重；如仍不合适，仅微调右侧两层并重验。
- 依赖用户视觉反馈；尚未授权修改其他蓝图/素材或提交。
- STATUS、IMPLEMENTATION_MATRIX 已同步；主要子任务仍为 `UI-DISPATCH-001`，父级保持未完成。
