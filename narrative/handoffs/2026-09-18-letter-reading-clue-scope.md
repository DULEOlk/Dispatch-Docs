# 信件阅读与可读文本线索范围交接

## 身份与范围

- 日期/时区：2026-09-18 / Asia/Shanghai
- 用户本次目标与最新修正：记录 `WBP_LetterUI` 阅读原型；确认当前功能是阅读，并保留“线索可能来自委托以外的文本”作为待定设计范围。
- 已授权范围 / 仍待确认范围：已授权本地文档和截图归档；仍待确认不同文本来源的线索覆盖率、触发方式、视觉表现、回看与漏读补救。
- 项目绝对路径、分支、HEAD：`F:\Epic Games\Unreal Projects\Dispatch`；`main`；`ceae683fc8c65d25608f847a0194dce174a7a4d5`
- 当前阶段及具体小任务：信件阅读与可读文本线索范围记录
- 子任务 ID：`DOC-UI-007`
- 覆盖的父级 `DEMO-*` 编号：`DEMO-VIS-001`、`DEMO-CLUE-001`、`DEMO-EXP-001`
- 明确不覆盖的相邻总设计项：不实现信件阅读重做、富文本线索、线索触发合同、正式文本、委托/对话/报告内容或七日剧情。
- 开工条件及是否满足：用户提供信件 UI 原型并提出线索范围问题；已核对 Letter/Readable 数据、Widget 和现有富文本路径；满足。
- 退出条件和用户可观察成果：截图可在本地文档中查看；阅读功能和当前代码差距已记录；多来源线索的未决问题可追踪；满足文档记录范围。

## 工作区

- 开始时已有的修改/未跟踪文件：开始前已有 Blueprint、C++、Editor 源码及未跟踪样例/资产修改；全部保留。
- 本轮实际修改的文件和作用：
  - `docs/narrative/ui/LETTER_READING_UI.md`：新建信件阅读与线索范围记录。
  - `docs/narrative/ui/images/letter-reading-early-prototype-2026-09-18.png`：保存用户提供的原型截图。
  - `docs/narrative/README.md`、`CREATIVE_OVERVIEW.md`：增加信件 UI/可读文本线索路由。
  - `docs/narrative/IMPLEMENTATION_MATRIX.md`：登记 `DOC-UI-007`，补充父级映射。
  - `docs/narrative/STATUS.md`：记录当前阅读功能、数据映射和未决范围。
  - 本交接文件。
- 用户资产、其他任务改动及保护说明：`Content/BluePrints/Widget/WBP_NoticeBoardUI.uasset`、`Source/Dispatch/Private/Quest/LetterDefinition.cpp` 等已有修改未由本轮保存或撤销；不对其来源作归因。
- 是否提交/推送，若有列 commit：未提交、未推送。`docs/` 按项目规则保持本地忽略。

## 完成与验证

- 已实现行为（与设计目标分开）：无业务实现。本轮完成文档/截图归档；截图源文件与目标文件 SHA256 均为 `39622412C0BF9BFAE5023C2141C171C2DDF096A950F7E869202DB3B366A109B4`。
- 本轮四类状态变化（设计 / 代码 / 自动验证 / 人工验收）：设计记录完成但线索范围/表现仍待确认；代码未修改；截图哈希、源码映射和文档路由已核对；信件/可读文本人工玩法验收未开始。
- 因本轮修改被标记“需要重验”的既有矩阵项：无。本轮未修改业务代码、Blueprint、配置、DataAsset 或 `.uasset`。
- 实际执行命令、构建配置、测试过滤器：使用 `git status --short`、`rg`、`Get-Content`、`Get-FileHash`；未构建、未运行 Unreal 自动化或 PIE。
- 通过/失败/未运行数、退出状态：文档/截图归档检查通过；业务测试未运行（不适用于纯文档记录）。
- 日志/证据路径：`docs/narrative/ui/LETTER_READING_UI.md`、`docs/narrative/ui/images/letter-reading-early-prototype-2026-09-18.png`。
- 人工验证结果或未验证事项：未验证玩家如何发现信件中的线索、漏读是否可补救、不同文本来源是否需要不同表现。

## 未完成与风险

- 问题编号、触发条件、源码路径/函数/行号：暂无实现缺陷结论。设计风险是把所有可读文本都标成线索会破坏阅读节奏，完全隐藏则可能造成盲搜；现状见 `ULetterWidgetBase::BuildLetterBody`、`FLetterContentSegment`、`UClueRichTextBlock` 和 `UQuestContentSubsystem::ResolveReadableDocument`。
- 是实际复现、静态追踪还是待确认推测：数据字段和 Widget 差距为静态源码核对；线索范围和表现问题为用户提出的待确认设计问题，不是已复现 bug。
- 已尝试步骤及结果，避免下次重复：已确认正式信件有类型化 Clue 引用，但当前阅读 Widget 仅显示普通正文；已确认旧可读文档路径可生成 `<clue>` 富文本标记；未将其视为正式信件解决方案。
- 当前运行中的工具/进程/代理及日志：全部结束。

## 下一步

- 下一个最小动作及验收方式：准备一封信件、一段环境文字、一段对话和一份报告的短文本切片，比较线索的点击、读完确认、选择回应和自然强调方案；记录玩家是否理解来源和后果。
- 依赖条件与是否需要用户输入：需要用户确认文本来源范围和体验目标；在此之前不改 Widget、富文本或数据字段。
- 需修改的蓝图/资产（尚未执行）：若后续获批，先列出 `WBP_LetterUI`、`ULetterWidgetBase`、富文本控件、线索来源字段和不包含范围；当前无修改获批。
- 本轮决策及原因，链接 DECISIONS：未形成新的稳定产品决策；候选表现保存在 [信件阅读与可读文本线索记录](../ui/LETTER_READING_UI.md)，避免过早统一规则。
- 已同步更新 STATUS：是。
- 已同步更新 IMPLEMENTATION_MATRIX：是；登记 `DOC-UI-007` 并映射 `DEMO-VIS-001`、`DEMO-CLUE-001`、`DEMO-EXP-001`。
- 当前主要进行中子任务 / 等待队列是否已同步：主要实施子任务仍无；`DOC-UI-007` 文档范围关闭，后续实现等待可玩文本切片和用户授权。

