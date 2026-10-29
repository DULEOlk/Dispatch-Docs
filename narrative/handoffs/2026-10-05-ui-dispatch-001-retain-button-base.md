# UI-DISPATCH-001 保留底座的独立按钮

## 身份与范围

- 2026-10-05，Asia/Shanghai；项目 `F:\Epic Games\Unreal Projects\Dispatch`，main，HEAD 21163763。
- 用户纠正：拆开按钮不等于移除金属承托底座，必须恢复截图里的完整底座。
- 子任务 UI-DISPATCH-001；父级 DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001。
- 授权现有底部 UI 修正；不覆盖正式 DA、派遣事务、列表测试债务或其他页面。
- 开工条件已满足：用户明确纠正、原双槽素材仍在、未运行编辑器。
- 退出条件：底座完整、按钮嵌入槽内、保留独立悬停/按下反馈；自动图可查看，用户交互审美验收仍待进行。

## 工作区

- 已检查 git status、相关源码 diff 和未跟踪项；既有源码、蓝图、素材改动保留，无删除、提交或推送。
- 修改 `CommissionDeskPanCommandlet.cpp` 新增受前置状态保护的 RestorePreparationButtonBase；只保存 WBP_PreparationSelection 与 WBP_CommissionDispatch。
- 恢复 FolioButtonSlots 和父页面三层同轮廓阴影。按钮正常/悬停/按下/禁用均采同一底图槽内 UV，外侧铆钉/连接件固定；悬停亮度 1.65/1.48/1.16，按下 .48/.43/.36，禁用 .55/.53/.49，按下内容 Padding.Top=2。
- 返回槽 99,758 / 214×51；确认槽 362,758 / 235×51。对应同一 680×850 板面，因此正常状态图片精确配准。
- 更新 SelectionFolioLayout 的底座资源、源尺寸 1050×1498、两槽不同 UV 检查；保留四态断言。
- 资产备份 `Saved/CodexBackups/PreparationButtonBase_20261005_143524/`。

## 完成与验证

- 设计：用户明确要求保留底座，无槽版人工验收未通过，决定已记录到 DECISIONS。
- 实现：两张 Widget 已保存，原按钮回调保留；无需生成新素材。
- Development 和 DebugGame 编译成功；最终 Development 只修正测试图片尺寸后再次成功。日志 `UBT-PreparationButtonBase.log`、`UBT-PreparationButtonBase-DebugGame.log`、`UBT-PreparationButtonBase-Final.log` 位于 Saved/Logs。
- 初次 UBA 被沙箱缓存目录阻止，已停止本轮进程并改用 -NoUBA 编译；首次 UV 比较类型歧义已加显式 FBox2f 修正。
- 作者日志 `PreparationButtonBase-Apply.log` 报 SAVED；DDC 无可写节点导致作者退出 1，后续独立加载和渲染确认资产写入。
- 真实 RHI 定向 3 项：ContactShadows、DeskPanRenderedGeometry 成功，SelectionFolioLayout 因旧图片尺寸断言失败；仅修正尺寸断言后 NullRHI 定向复验 1/1 成功、进程退出 0，见 `PreparationButtonBase-LayoutFinal.log`。本轮工具进程全部结束。
- 渲染 `Saved/Automation/CommissionDeskPan/Sheet_ButtonBase_20261005.bmp` 已目视查看，完整底座恢复，文字在槽内，无浮空彩色按钮。
- 未重跑整组；之前三项旧列表结构失败仍未解决，不宣称全组通过。用户原生悬停/按下/禁用观感尚未验收。

## 下一步与风险

- 原生 PIE 检查按钮面提亮/加深、文字下移及点击。自动静态截图不是人工交互验收。
- 只更新 UI-DISPATCH-001 及关联父项证据，不提升父级总设计完成状态。
- STATUS、IMPLEMENTATION_MATRIX 已同步；唯一主要 UI 实施子任务不变。
