# 第一组委托预览物品交接

## 身份与范围

- 2026-10-02，Asia/Shanghai；项目 `F:/Epic Games/Unreal Projects/Dispatch`，`main`，HEAD `21163763`。
- 子任务 `UI-DISPATCH-001`，父级 `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。
- 用户授权：仅给第一组委托预览添加干粮、绷带、油灯、旧地图碎片用于交付 UI 点击；其余委托与正式内容不动。
- 退出条件：本机配置可由 UE 解析四项；用户可在原生窗口看到列表并测试点击。前者有证据，后者待用户验收。

## 工作区与操作

- 开工检查 Git 既有源码/Widget/素材脏改并保留；UE 编辑器当时未运行。
- 配置文件：`Saved/Config/WindowsEditor/EditorPerProjectUserSettings.ini` 第 1 组 `CommissionPreviewAssets`。四项 `Type=Item`、唯一 `Preview_*` ID、中文名称/说明、`AvailableCount=1`、`Icon=None`。第 2/3 组保持空。
- 修改前备份：`Saved/CodexBackups/PreviewPreparations_20261002/EditorPerProjectUserSettings.ini`。无正式 DA、源码、Widget、背包、线索、存档更改；无提交/推送。

## 验证与未完成

- UE 读回命令：`GetAll StoryWorkspaceEditorSettings CommissionPreviewAssets SHOWDEFAULTS`，日志 `Saved/Logs/PreparationPreviewSample-GetAll.log`。引擎导出第 1 组四项 ID、名称、数量 1，并导出第 2/3 组为空。
- 一次 Python 只读诊断因私有 Editor Settings 属性未导出到 Python 而未能读取；未改变项目配置。改用 UE 自带 GetAll 成功。GetAll 把同一 `-ExecCmds` 的 `;Quit` 吞入参数，执行者在获得输出后 Ctrl-C 结束自启 UE 进程，当前无该进程残留。
- 本小步仅为数据配置，未重新构建或运行 UI 自动测试；上一小步聚焦测试不能代替用户原生鼠标点击。人工验收仍待：启动 Story Workspace 委托预览，选择第 1 组委托，点一个准备格，选物品看详情，点“放入当前槽位”，重复填其余格，并试改选/跨格占用。
- `STATUS` 与 `IMPLEMENTATION_MATRIX` 已同步。父级整体不升级；本地预览配置无正式内容意义。
