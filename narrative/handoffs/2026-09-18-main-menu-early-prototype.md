# 主菜单与对应地图早期原型交接

## 身份与范围

- 日期/时区：2026-09-18 / Asia/Shanghai
- 用户本次目标与最新修正：记录主菜单早期 UI 设计及对应地图；用户明确它与当前玩法关系不大，后续较晚再修改。
- 已授权范围 / 仍待确认范围：已授权本地文档和截图归档；仍待确认正式项目名称、菜单流程、设置/存档范围和菜单地图的最终用途。
- 项目绝对路径、分支、HEAD：`F:\Epic Games\Unreal Projects\Dispatch`；`main`；`ceae683fc8c65d25608f847a0194dce174a7a4d5`
- 当前阶段及具体小任务：主菜单/入口 UI 历史原型记录
- 子任务 ID：`DOC-UI-006`
- 覆盖的父级 `DEMO-*` 编号：`DEMO-VIS-001`（仅视觉入口记录，不扩展核心玩法范围）
- 明确不覆盖的相邻总设计项：不覆盖委托、线索、派遣、撰写、七日循环、正式地图玩法或菜单实现。
- 开工条件及是否满足：用户提供主菜单截图并明确后期处理；已核对入口配置和相关资产路径；满足。
- 退出条件和用户可观察成果：截图可在本地文档中查看；`WBP_MainMenu`/`L_MainMenu` 资产位置可追溯；记录明确历史原型与核心玩法边界；满足文档记录范围。

## 工作区

- 开始时已有的修改/未跟踪文件：开始前已有 Blueprint、C++、Editor 源码及未跟踪样例/资产修改；全部保留。
- 本轮实际修改的文件和作用：
  - `docs/narrative/ui/MAIN_MENU_UI.md`：新建主菜单与地图早期原型记录。
  - `docs/narrative/ui/images/main-menu-early-prototype-2026-09-18.png`：保存用户提供的原型截图。
  - `docs/narrative/README.md`、`CREATIVE_OVERVIEW.md`：增加主菜单 UI 文档路由。
  - `docs/narrative/IMPLEMENTATION_MATRIX.md`：登记 `DOC-UI-006` 并补充 `DEMO-VIS-001` 映射。
  - `docs/narrative/STATUS.md`：记录当前资产关系和未授权范围。
  - 本交接文件。
- 用户资产、其他任务改动及保护说明：`Content/BluePrints/Widget/WBP_NoticeBoardUI.uasset`、`Content/Maps/L_MainMenu.umap` 等资产未由本轮保存或撤销；不对既有修改来源作归因。
- 是否提交/推送，若有列 commit：未提交、未推送。`docs/` 按项目规则保持本地忽略。

## 完成与验证

- 已实现行为（与设计目标分开）：无业务实现。本轮完成文档/截图归档；截图源文件与目标文件 SHA256 均为 `EC52BC71E601BBBB264C4B4799A9688CD328C380D637A19DF5A07F00406A85A7`。
- 本轮四类状态变化（设计 / 代码 / 自动验证 / 人工验收）：设计记录完成但最终菜单方案未确认；代码未修改；入口资产路径和截图哈希已核对；菜单/地图人工验收未开始。
- 因本轮修改被标记“需要重验”的既有矩阵项：无。本轮未修改业务代码、Blueprint、配置、地图或 `.uasset`。
- 实际执行命令、构建配置、测试过滤器：使用 `git status --short`、`rg`、`Get-Content`、`Get-FileHash`；未构建、未运行 Unreal 自动化或 PIE。
- 通过/失败/未运行数、退出状态：文档/截图归档检查通过；业务测试未运行（不适用于纯文档记录）。
- 日志/证据路径：`docs/narrative/ui/MAIN_MENU_UI.md`、`docs/narrative/ui/images/main-menu-early-prototype-2026-09-18.png`。
- 人工验证结果或未验证事项：未验证菜单按钮行为、设置页面、退出确认、存档入口、地图背景和不同分辨率布局。

## 未完成与风险

- 问题编号、触发条件、源码路径/函数/行号：暂无实现缺陷结论。风险是早期菜单视觉或旧标题被误当作当前需求；相关资产见 `WBP_MainMenu.uasset`、`L_MainMenu.umap` 和 `Config/DefaultEngine.ini`。
- 是实际复现、静态追踪还是待确认推测：资产路径和默认入口为静态核对；最终菜单流程、地图用途和标题均待确认。
- 已尝试步骤及结果，避免下次重复：已确认工程存在主菜单 Widget、菜单地图、GameMode/PlayerController，并确认默认地图配置；没有对地图内容作视觉推断。
- 当前运行中的工具/进程/代理及日志：全部结束。

## 下一步

- 下一个最小动作及验收方式：暂不实施；等核心玩法和整体视觉方向稳定后，再集中确定菜单流程和地图用途，并由用户确认是否开始重做。
- 依赖条件与是否需要用户输入：需要正式项目名称、菜单流程（开始/继续/设置/退出等）和菜单地图用途；当前不阻塞核心玩法工作。
- 需修改的蓝图/资产（尚未执行）：未来若获批，先列出 `WBP_MainMenu`、`L_MainMenu`、GameMode、PlayerController、配置和素材范围；当前无修改获批。
- 本轮决策及原因，链接 DECISIONS：未形成新的稳定产品决策；历史原型和边界保存在 [主菜单与地图早期原型](../ui/MAIN_MENU_UI.md)。
- 已同步更新 STATUS：是。
- 已同步更新 IMPLEMENTATION_MATRIX：是；登记 `DOC-UI-006` 并映射 `DEMO-VIS-001`。
- 当前主要进行中子任务 / 等待队列是否已同步：主要实施子任务仍无；`DOC-UI-006` 文档范围关闭，后续实现等待玩法稳定和用户授权。

