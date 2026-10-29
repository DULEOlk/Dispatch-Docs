# Dispatch Creative Documentation System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立一个不要求用户记住文件名的创作资料入口，使新任务能根据“修改剧本、增加玩法、编写/录入文本、查询进度”等自然语言请求自动找到权威资料和正确流程。

**Architecture:** 使用 `CREATIVE_OVERVIEW.md` 作为唯一日常入口和请求路由；玩法全貌进入 `GAMEPLAY_FLOW.md`；故事真相、总纲、七日、人物、委托线和揭示关系先集中在 `story/STORY_BIBLE.md`；文本和灵感分别由 `TEXT_INDEX.md` 与 `IDEA_INBOX.md` 管理。现有 Demo、数据、技术和进度文档继续保持各自权威，入口只摘要并链接，不复制它们的详细定义。

**Tech Stack:** 本地 Markdown、PowerShell 只读一致性检查、现有 `AGENTS.md`/`docs/narrative` 文档体系。

**Spec:** `docs/narrative/plans/2026-09-16-creative-documentation-system-design.md`

## Global Constraints

- 使用当前本地目录 `F:\Epic Games\Unreal Projects\Dispatch`，不创建 worktree。
- 只修改根 `AGENTS.md` 和 `docs/narrative/`；不修改 C++、Blueprint、配置或 `.uasset`。
- 保留当前所有已跟踪修改和未跟踪文件，不撤销、不覆盖、不移动。
- 五份奇怪委托只描述为五条核心剧情支柱；允许存在前置、调查、人物和后果类关联内容，不增加固定任务总数合同。
- 当前故事前提标记为“试验中”，不得把验收者、五条委托正文、完整七日剧本或结局写成已确认事实。
- 创作确定度和制作进度必须分列；“内容已确认”不得等同“允许修改 DataAsset”。
- 用户不需要记住文件名；路由义务写入根 `AGENTS.md`、`README.md` 和 `CREATIVE_OVERVIEW.md`。
- 不提交、不推送；`docs/` 继续保持本地忽略状态。

---

### Task 1: 建立创作总入口与根路由

**Files:**
- Create: `docs/narrative/CREATIVE_OVERVIEW.md`
- Modify: `AGENTS.md`
- Modify: `docs/narrative/README.md`

**Interfaces:**
- Consumes: 设计稿中的自然语言请求路由、两套状态与新任务恢复顺序。
- Produces: 后续所有创作任务的唯一日常入口；根入口能够把用户请求路由到玩法、故事、文本、数据或进度资料。

- [x] **Step 1: 创建 `CREATIVE_OVERVIEW.md` 的固定结构**

  写入以下实际章节：一句话概念、当前创作状态、用户只需怎样表达需求、文档导航、创作确定度、制作进度、五类标准工作流、当前未决问题。开头明确写出：“用户不需要记住下列文件名；当前任务负责根据请求自动读取和更新正确资料。”

- [x] **Step 2: 写入自然语言请求路由表**

  路由表逐项覆盖：记录想法、修改剧本/人物/背景、增加或修改玩法、编写对话/信件/报告、确认内容、正式录入、查询进度。每行必须包含“读取资料、默认动作、需要再次授权的行为”。

- [x] **Step 3: 更新根 `AGENTS.md` 的强制读取规则**

  在现有剧情文档读取说明中加入 `CREATIVE_OVERVIEW.md`，并写明：用户未说文件名时也必须自行路由；修改剧本读 Story Bible，修改玩法读 Gameplay Flow，处理游戏文本读 Text Index；实施和进度仍读 Demo Design、Matrix、Status 与 handoff。

- [x] **Step 4: 更新 `README.md` 的入口表**

  将 `CREATIVE_OVERVIEW.md` 放在日常创作入口位置；说明用户只需从它开始，其余文件由任务按路由读取。保留现有技术恢复顺序，不删除 Demo Design、Foundation Data Design、System Design、Matrix、Status、Rules 或归档说明。

- [x] **Step 5: 验证三个入口的一致性**

  Run:

  ```powershell
  Select-String -Path 'AGENTS.md','docs\narrative\README.md','docs\narrative\CREATIVE_OVERVIEW.md' -Pattern 'CREATIVE_OVERVIEW.md|用户不需要记住|自然语言'
  ```

  Expected: 三个文件都明确指向创作总入口；根入口包含无需用户提供文件名的路由义务。

### Task 2: 编写完整玩法全貌

**Files:**
- Create: `docs/narrative/GAMEPLAY_FLOW.md`
- Modify: `docs/narrative/CREATIVE_OVERVIEW.md`

**Interfaces:**
- Consumes: `DEMO_DESIGN.md` 的现有核心体验与七日循环，以及设计稿中“五条核心委托线 + 关联内容”的解释。
- Produces: 面向设计者的完整可玩流程；入口可以从“增加玩法”跳转到该文档。

- [x] **Step 1: 写入玩法文档状态口径**

  文件开头声明它负责“游戏怎样玩”，不负责代码完成度；每个尚未定案的部分使用“试验中”或“待决定”，稳定要求链接到 `DEMO_DESIGN.md` 的 `DEMO-*` 编号。

- [x] **Step 2: 写入从抵达到第七天的完整流程**

  章节必须覆盖：玩家囚犯身份和七天压力、第一天抵达、白天自由活动、结束白天、夜间三段撰写、睡觉换日、第七天记录本验收。验收者身份和具体评价方式标为待决定。

- [x] **Step 3: 写入核心系统反馈循环**

  明确描述：探索/阅读/对话获得信息 → 线索与推论改变判断 → 选择 NPC/物品/告知信息进行派遣 → 返回报告与世界后果 → 每日撰写表达判断 → 第七天收束。保留世界真相、NPC 认知和玩家认知的分离。

- [x] **Step 4: 写入内容分类和非理想路径**

  定义核心委托线、关联委托、人物内容、环境调查四类设计角色；说明不调查、判断错误、准备不足、委托来不及返回仍可推进并形成后果，不把它们全部变成按钮禁用。

- [x] **Step 5: 写入玩法未决问题**

  列出但不替用户决定：五条委托关系、关联委托容量、三个 NPC 的最终角色、验收依据、中间五天内容预算、最短通关路径与可选调查路径。

- [x] **Step 6: 验证没有把试验内容写成已确认规则**

  Run:

  ```powershell
  Select-String -Path 'docs\narrative\GAMEPLAY_FLOW.md' -Pattern '试验中|待决定|五条核心委托|关联委托|第七天'
  ```

  Expected: 文件同时包含当前方向、弹性分类和未决状态；没有“整个 Demo 恰好只有五个任务”的陈述。

### Task 3: 建立集中式 Story Bible

**Files:**
- Create: `docs/narrative/story/STORY_BIBLE.md`
- Modify: `docs/narrative/CREATIVE_OVERVIEW.md`

**Interfaces:**
- Consumes: 当前对话确认的早期故事概念和设计稿规定的九个 Story Bible 章节。
- Produces: 剧本与背景设定的唯一创作权威；后续人物、七日和委托线内容在真正过大前不拆文件。

- [x] **Step 1: 创建九个固定章节**

  依次创建：故事概念与主题、世界真相与背景历史、Demo 故事总纲、七日剧情轮廓、人物/动机/关系/秘密/知识范围、五条核心委托线及关联内容、揭示矩阵、结局与第七天验收方向、未决问题。

- [x] **Step 2: 填入当前唯一可确认的故事概念**

  写入玩家是囚犯、封闭小型区域、七天、诡异事件、五份奇怪核心委托、每日记录和第七天验收，并统一标为“试验中”。不要编造区域名称、NPC 姓名、委托正文、验收者身份或异常真相。

- [x] **Step 3: 为世界真相和人物知识建立模板**

  世界真相条目字段固定为：`StoryFactID`、确定度、事实内容、发生时间、涉及人物、是否必须向玩家揭示。人物知识条目字段固定为：人物、已知内容、误解内容、隐瞒内容、信息来源、可能变化。

- [x] **Step 4: 为委托线和揭示矩阵建立模板**

  核心委托线字段固定为：`ArcID`、表面委托、隐藏问题、关联内容、涉及人物、玩家可能获得的信息、跨日后果、当前状态。揭示矩阵字段固定为：`RevealID`、世界真相、知情者、玩家渠道、最早可见日、相关文本、遗漏后果、状态。

- [x] **Step 5: 记录集中后拆分规则**

  文件开头声明：当前所有故事章节集中于本文件；只有查找已经明显困难时才拆分，并且本文件始终保留索引和权威关系说明。

- [x] **Step 6: 验证九个章节和模板字段**

  Run:

  ```powershell
  Select-String -Path 'docs\narrative\story\STORY_BIBLE.md' -Pattern '^## ','StoryFactID','ArcID','RevealID','试验中'
  ```

  Expected: 九个内容章节全部存在，三个稳定 ID 模板存在，当前概念明确标为试验中。

### Task 4: 建立游戏文本索引与想法收集箱

**Files:**
- Create: `docs/narrative/story/TEXT_INDEX.md`
- Create: `docs/narrative/story/IDEA_INBOX.md`
- Create: `docs/narrative/story/texts/README.md`
- Modify: `docs/narrative/CREATIVE_OVERVIEW.md`

**Interfaces:**
- Consumes: Story Bible 的人物知识、委托线和揭示矩阵，以及数据基础中的正式 DataAsset 类型。
- Produces: 文本从草稿到 DA 验收的索引；灵感不会直接污染正式设计。

- [x] **Step 1: 创建 TextID 命名与文本类型规则**

  使用 `TXT-<TYPE>-<STABLE_NAME>`，类型限定为 `DLG`、`LTR`、`COM`、`RPT`、`CLU`、`INF`、`ENV`、`WRT`、`END`。说明稳定名不使用日期或数组序号，正文修改不更换 TextID。

- [x] **Step 2: 创建文本索引表**

  表列固定为：TextID、类型、所属人物/Arc/日期、揭示或误导目的、前置知识、草稿路径、创作确定度、制作进度、正式 DA 路径、最近确认。加入一行明确标记为“格式示例、不是正式剧情”的示例，并说明创建真实条目时删除示例标志。

- [x] **Step 3: 写入文本制作状态转换和授权边界**

  明确 `草稿 → 待审阅 → 内容已确认 → 已录入 DA → 自动验证通过 → 人工验收通过`；写明“内容已确认”只批准文字和情节，不批准资产写入。

- [x] **Step 4: 创建 Idea Inbox 模板**

  表列固定为：日期、原始想法、涉及玩法/剧本/文本、可能影响、待回答问题、处理状态、去向。处理状态限定为新想法、评估中、已采纳、已搁置、已替代；已采纳项必须链接到新的权威位置。

- [x] **Step 5: 创建长文本目录说明**

  `story/texts/README.md` 说明短文本可留在 Text Index，长文本按 TextID 建 Markdown 文件；文件必须反向链接 Text Index 和 Story Bible 相关 Reveal/Arc；最终运行版本仍以正式 DataAsset 为准。

- [x] **Step 6: 验证状态和授权措辞**

  Run:

  ```powershell
  Select-String -Path 'docs\narrative\story\TEXT_INDEX.md','docs\narrative\story\IDEA_INBOX.md','docs\narrative\story\texts\README.md' -Pattern 'TXT-<TYPE>-<STABLE_NAME>|内容已确认|不批准资产写入|已采纳|DataAsset'
  ```

  Expected: TextID、两套状态、灵感去向和正式资产边界都有明确命中。

### Task 5: 更新设计决定、进度与交接

**Files:**
- Modify: `docs/narrative/DECISIONS.md`
- Modify: `docs/narrative/IMPLEMENTATION_MATRIX.md`
- Modify: `docs/narrative/STATUS.md`
- Modify: `docs/narrative/handoffs/2026-09-16-creative-documentation-design.md`

**Interfaces:**
- Consumes: Tasks 1–4 建立的实际入口和当前 Git 状态。
- Produces: 设计变化的历史原因、可恢复的子任务状态和最终文档交接。

- [x] **Step 1: 在 Decisions 记录内容结构变化**

  新增 2026-09-16 决策，说明旧的“固定五个委托”表述改为“五条核心剧情支柱，可有相关委托和非委托调查内容”；原因是三个 NPC、六个实际活动日和持续扩展的调查玩法不能由五个一次性任务承载。注明这不立即改变数据合同或正式资产。

- [x] **Step 2: 在 Decisions 记录文档权威关系**

  说明 Creative Overview 是路由入口、Gameplay Flow 是可读玩法全貌、Demo Design 是稳定编号权威、Story Bible 是故事创作权威、Text Index 连接剧本文本与 DataAsset、Matrix 是唯一实施进度。

- [x] **Step 3: 更新 Matrix 子任务状态**

  将 `DOC-CREATIVE-001` 标为文档实施完成；退出证据是新任务恢复合同和四种自然语言路由演练通过，不要求用户人工阅读全部入口。代码与自动化列不改变相邻 Demo 项。最新交接继续指向 2026-09-16 handoff。

- [x] **Step 4: 更新 Status 和 handoff**

  记录创建文件、路由规则、状态口径、验证命令、未触碰业务文件和下一步。保留 `DATA-FOUNDATION-001` 等待人工编辑/预览验收的独立状态，不把本任务完成当成数据基础验收。

- [x] **Step 5: 核对 Git 工作区保护**

  Run:

  ```powershell
  git status --short --untracked-files=all
  ```

  Expected: 原有源码、UI 资产、13 个样例资产和新增数据基础源码仍在；没有因本任务删除、恢复或覆盖业务文件。

### Task 6: 执行新任务恢复与链接验收

**Files:**
- Verify: `AGENTS.md`
- Verify: `docs/narrative/README.md`
- Verify: `docs/narrative/CREATIVE_OVERVIEW.md`
- Verify: `docs/narrative/GAMEPLAY_FLOW.md`
- Verify: `docs/narrative/story/STORY_BIBLE.md`
- Verify: `docs/narrative/story/TEXT_INDEX.md`
- Verify: `docs/narrative/story/IDEA_INBOX.md`
- Verify: `docs/narrative/story/texts/README.md`

**Interfaces:**
- Consumes: Tasks 1–5 的全部文档成果。
- Produces: 能证明新任务无需用户提供文件名即可恢复的最终验收记录。

- [x] **Step 1: 检查所有固定文件存在**

  Run:

  ```powershell
  $files = @('AGENTS.md','docs\narrative\README.md','docs\narrative\CREATIVE_OVERVIEW.md','docs\narrative\GAMEPLAY_FLOW.md','docs\narrative\story\STORY_BIBLE.md','docs\narrative\story\TEXT_INDEX.md','docs\narrative\story\IDEA_INBOX.md','docs\narrative\story\texts\README.md'); $missing = $files | Where-Object { -not (Test-Path -LiteralPath $_) }; if ($missing) { throw "Missing: $($missing -join ', ')" }; "FILES=$($files.Count)/$($files.Count)"
  ```

  Expected: `FILES=8/8`.

- [x] **Step 2: 检查路由覆盖所有常见请求**

  Run:

  ```powershell
  $entry = Get-Content -Raw 'docs\narrative\CREATIVE_OVERVIEW.md'; @('修改剧本','增加玩法','编写游戏文本','正式录入','查询进度') | ForEach-Object { if (-not $entry.Contains($_)) { throw "Missing route: $_" } }; 'ROUTES=5/5'
  ```

  Expected: `ROUTES=5/5`.

- [x] **Step 3: 检查权威关系与状态边界**

  Run:

  ```powershell
  rg -n "唯一|权威|试验中|内容已确认|不.*资产|用户不需要记住" AGENTS.md docs/narrative/CREATIVE_OVERVIEW.md docs/narrative/GAMEPLAY_FLOW.md docs/narrative/story
  ```

  Expected: 根入口和创作入口均承担路由；玩法和故事试验内容没有被标成已实现；内容确认与资产许可明确分开。

- [x] **Step 4: 检查设计/实施计划没有残留占位符**

  Run:

  ```powershell
  rg -n "TBD|TODO|PLACEHOLDER|待补" docs/narrative/CREATIVE_OVERVIEW.md docs/narrative/GAMEPLAY_FLOW.md docs/narrative/story
  ```

  Expected: 无匹配。合法的“待决定”是明确状态，不属于占位符。

- [x] **Step 5: 模拟四种新任务恢复**

  按根入口分别追踪“我要修改剧本”“我想增加一个玩法”“这批文本可以录入”“现在完成多少”四句话，逐项在 handoff 记录会读取哪些文件、是否需要再次授权以及最终更新位置。四条都不得要求用户先提供文件名。

- [x] **Step 6: 以代理恢复合同完成验收**

  验证根 `AGENTS.md` 能从常见自然语言请求进入 Creative Overview，并由路由表确定读取资料、默认动作、再次授权和更新位置。用户可以直接表达需求，不承担记忆文件名、阅读全部入口或复述工作流的验收责任。
