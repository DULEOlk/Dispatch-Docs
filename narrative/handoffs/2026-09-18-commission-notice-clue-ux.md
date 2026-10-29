# 委托公告板与线索发现体验记录交接

## 身份与范围

- 日期/时区：2026-09-18 / Asia/Shanghai
- 用户本次目标与最新修正：记录 `WBP_NoticeBoardUI` 接取委托原型；保留线索收集的价值，同时避免线索过度隐蔽或硬限制导致玩家最终无法理解。
- 已授权范围 / 仍待确认范围：已授权本地文档和截图归档；仍待确认公告页是否展示关联线索、线索提示/补救规则、委托人/报酬字段合同及任何 UI/代码/资产实现。
- 项目绝对路径、分支、HEAD：`F:\Epic Games\Unreal Projects\Dispatch`；`main`；`ceae683fc8c65d25608f847a0194dce174a7a4d5`
- 当前阶段及具体小任务：UI 原型与线索体验设计记录
- 子任务 ID：`DOC-UI-005`
- 覆盖的父级 `DEMO-*` 编号：`DEMO-VIS-001`、`DEMO-COM-001`、`DEMO-CLUE-001`、`DEMO-EXP-001`
- 明确不覆盖的相邻总设计项：不实现正式委托/七日剧情、线索可见性合同、推论算法、派遣规则、公告板 Widget 重做或完整人工玩法验收。
- 开工条件及是否满足：用户已提供原型并明确要求记录；已读取创作入口、玩法/总设计、矩阵、状态、规则和相关源码；满足。
- 退出条件和用户可观察成果：截图可在本地文档中查看；新 UI 记录能说明接取页与线索/推论/派遣/报告边界；矩阵、STATUS 和交接互相可追溯；满足文档记录范围。

## 工作区

- 开始时已有的修改/未跟踪文件：开始前 `git status --short` 显示已有 Blueprint、C++、Editor 源码和未跟踪样例/资产修改；全部保留，未作为本任务成果。
- 本轮实际修改的文件和作用：
  - `docs/narrative/ui/COMMISSION_NOTICE_UI.md`：新建公告板与线索发现体验记录。
  - `docs/narrative/ui/images/commission-notice-board-2026-09-18.png`：保存用户提供的原型截图。
  - `docs/narrative/README.md`、`CREATIVE_OVERVIEW.md`：增加自然语言路由和文档导航。
  - `docs/narrative/IMPLEMENTATION_MATRIX.md`：登记 `DOC-UI-005`，补充父级映射和等待状态。
  - `docs/narrative/STATUS.md`：记录当前设计、代码静态映射和未授权范围。
  - 本交接文件。
- 用户资产、其他任务改动及保护说明：`Content/BluePrints/Widget/WBP_NoticeBoardUI.uasset` 等已有修改未由本轮保存或撤销；不对其来源作归因。
- 是否提交/推送，若有列 commit：未提交、未推送。`docs/` 仍按项目规则保持本地忽略。

## 完成与验证

- 已实现行为（与设计目标分开）：无业务实现。本轮完成文档/截图归档；截图源文件和目标文件 SHA256 均为 `82A970507D41B2D243178695717BB4AE0CC58036D96407E053982019D1458CB0`。
- 本轮四类状态变化（设计 / 代码 / 自动验证 / 人工验收）：设计记录完成但交互机制仍待验证；代码未修改；文档链接和截图哈希已核对；UI/线索玩法人工验收未开始。
- 因本轮修改被标记“需要重验”的既有矩阵项：无。本轮未修改业务代码、Blueprint、配置、DataAsset 或 `.uasset`。
- 实际执行命令、构建配置、测试过滤器：使用 `git status --short`、相关源码 `Get-Content`/`rg`、`Get-FileHash`；未构建、未运行 Unreal 自动化或 PIE。
- 通过/失败/未运行数、退出状态：文档/截图归档检查通过；业务测试未运行（不适用于纯文档记录）。
- 日志/证据路径：`docs/narrative/ui/COMMISSION_NOTICE_UI.md`、`docs/narrative/ui/images/commission-notice-board-2026-09-18.png`。
- 人工验证结果或未验证事项：未验证玩家是否会卡住、是否能分辨接取/线索/推论/派遣，也未验证加入提示后是否破坏思考。

## 未完成与风险

- 问题编号、触发条件、源码路径/函数/行号：暂无实现缺陷结论。设计风险是公告页信息过少会让玩家盲搜，信息过多会提前揭底；相关现状见 `UNoticeBoardQuestWidget::DisplayQuestInfo`、`ANoticeBoard::AcceptQuest`、`ANoticeBoard::SetCommissionSlot` 和 `UClueSubsystem`。
- 是实际复现、静态追踪还是待确认推测：字段映射和状态边界为静态源码核对；线索体验问题为用户提出的待验证设计风险，不是已复现 bug。
- 已尝试步骤及结果，避免下次重复：已确认公告页没有独立的线索进度面板；已确认线索记录包含来源/日期/委托上下文；未提前新增“委托人/报酬”字段。
- 当前运行中的工具/进程/代理及日志：全部结束。

## 下一步

- 下一个最小动作及验收方式：先完成一个可玩的 2～3 日切片，再观察公告页、线索收集、笔记组合和派遣准备的实际卡点；用文档中六条观察问题记录证据。
- 依赖条件与是否需要用户输入：需要真实可玩流程和用户确认的公告页/线索交互合同；在此之前不改 UI 或数据结构。
- 需修改的蓝图/资产（尚未执行）：若后续获批，需先列出 `WBP_NoticeBoardUI` 控件、线索笔记入口、字段/引用和不包含范围；当前无修改计划获批。
- 本轮决策及原因，链接 DECISIONS：未形成新的稳定产品决策；暂定 UX 目标保存在 [公告板与线索发现](../ui/COMMISSION_NOTICE_UI.md)，避免把候选手段提前锁死。
- 已同步更新 STATUS：是。
- 已同步更新 IMPLEMENTATION_MATRIX：是；登记 `DOC-UI-005` 并补充 `DEMO-VIS-001`、`DEMO-EXP-001`、`DEMO-CLUE-001`、`DEMO-COM-001` 映射。
- 当前主要进行中子任务 / 等待队列是否已同步：主要实施子任务仍无；`DOC-UI-005` 文档范围关闭，后续实现等待可玩切片和用户授权。

