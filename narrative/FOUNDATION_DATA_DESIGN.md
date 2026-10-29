# Dispatch 剧情数据基础设计基线

版本：1.0  
更新日期：2026-09-14（Asia/Shanghai）  
性质：支撑 `DEMO_DESIGN.md` 的数据资产、字段、引用、校验与迁移权威模板。

> **时间范围标记（2026-10-28）**：本文提到的七天/七日均为早期流程示例，正式天数待定，可能因剧情规模延长。数据层不应把天数写死为七。

子任务 ID：`DATA-FOUNDATION-001`。  
父级总设计编号：见 [IMPLEMENTATION_MATRIX.md](IMPLEMENTATION_MATRIX.md) 的子任务登记。本文件只定义数据层，不自行宣称父级玩法已经完成。

## 1. 本稿解决什么

本稿只解决 Demo 流程的**内容数据如何拆分、引用、校验和预览**。目标是让设计者能顺畅填写委托、人物、对话、线索与每日内容，不再依赖散落的 DataTable 行、C++ 固定样例或把人物动作与台词塞进人物表。Demo 的体验目标、七日循环和系统职责以 `DEMO_DESIGN.md` 为准。

本阶段明确不做：NPC 地图移动、锚点、日程选择、动作 StateTree、正式多日剧情填充，以及把具体剧情顺序写死在 C++。

历史讨论确认的原始问题：

- 派发预览曾使用 C++ 中创建的临时正文，图片、文字、NPC 和交付物不能由设计者直接修改。
- 委托数据偏展示，缺少“谁执行、带什么、为什么得到何种结果、何时返回”的结构。
- NPC 资料、对话和动作职责混杂，内容增长后难以维护。
- 测试应该跳过接取等前置步骤，但仍读取正式可编辑内容；临时的只能是本次测试的运行状态。

## 2. 总体分层

```text
Campaign（项目/篇章总入口）
  ├─ Content Catalog（所有正式内容的分类目录）
  └─ Episode（剧情因果、入口、条件、动作与出口）

Content Catalog
  ├─ NPC Definition
  ├─ Dialogue Definition
  ├─ Commission Definition
  ├─ Letter Definition
  ├─ Clue Definition
  ├─ Inference Definition
  └─ Day Definition

运行时记录（不写回上述资产）
  ├─ NPC State
  ├─ Commission / Dispatch Instance
  ├─ Dialogue Session
  ├─ Known Clues / Inferences
  └─ Day / Writing State
```

三类数据必须分开：

1. **内容定义**：作者写的正文、图片、规则和引用，保存在正式资产中。
2. **运行状态**：本次游玩中已接取、已读、已派遣、已获知等状态，保存在运行实例/存档中。
3. **剧情因果**：何时检查什么条件、触发什么动作、进入哪个后续，保存在 Episode；不复制正文。

## 3. 正式内容资产

### 3.1 Campaign

保存：Campaign ID、标题、最大日数、Content Catalog、Starting Episode、可选的根协调 StateTree。

职责：提供单一制作入口和内容范围。它不复制任何委托、台词或人物正文，也不直接保存当前日。

### 3.2 Episode

保存：Episode ID、入口、剧情节点、事件触发规则、条件、类型化动作、分支、出口和下一 Episode 引用。

职责：表达剧情因果。例如“听取某报告后开放一段对话”，而不是保存报告全文。自由探索内容可以由事件触发进入，不必全部串成一条强制直线。

### 3.3 Content Catalog

按类型登记正式资产，并提供稳定 ID 到资产引用的唯一解析：NPC、Dialogue、Commission、Letter、Clue、Inference、Day。

校验至少覆盖：重复 ID、空引用、类型错误、引用未登记、同一 ID 指向多个资产。Catalog 只是目录，不成为第二份正文数据库。

### 3.4 NPC Definition

只保存稳定人物资料：NPC ID、姓名、头像、职业/身份、背景简介、用于规则判断的基础特征、初始信任/身体状态。

不得保存：

- 对话正文或每句台词；
- 走路、坐下、播放动画等动作；
- 当前伤势、当前位置、正在执行的委托；
- 针对某份委托的反应文案。

针对委托的反应属于委托规则或关联对话；当前状态属于 NPC 运行状态；地图活动留到后续独立系统。

### 3.5 Dialogue Definition

一段完整会话一个资产，而不是一句话一个资产。保存会话 ID、参与人物、节点、说话者、正文、选项、显示条件、选择效果和退出点。

对话可以引用 Commission/Clue/Letter 等正式资产；NPC Definition 只反向被引用，不内嵌所有对话。

### 3.6 Commission Definition

2026-10-25 UI-DISPATCH-001：记录本详情采用三个独立展示字段：`Presentation.Requester`（委托人）、已有 `Location`（地点）、`Reward`（承诺报酬文本）。通过 FQuestInfo 的 QuestRequester/QuestLocation/QuestReward 传入独立控件。缺值显示“—”；旧 Meta 仅兼容旧接口，不解析任意备注，不从执行NPC或隐藏结果推断。Reward仅用于展示，不决定结算。此步新增可选字段，未改写任何现有DA值；内容填写仍需逐资产授权。

一份委托资产按编辑顺序分为五组：

1. **展示**：Commission ID、标题、图片、委托人、描述、目标、委托人要求。
2. **开放与接取**：出现/可接/过期条件；这些是规则，不从描述文字推断。
3. **派遣配置**：执行日数、最大准备槽、允许的准备类别、真正禁止派遣的条件。
4. **结果规则**：优先级、条件、出发前 NPC 反应、命中的结果引用/结构。
5. **结果内容**：调查进展、NPC 后果、物品去向、返回证据/线索、报告对话、剧情事实或信号。

每份委托必须有无条件默认结果。准备不足通常仍允许派遣并进入默认结果；只有 NPC 已外出、库存不足、委托关闭等真实约束才阻止派遣。UI 不显示隐藏 Outcome ID 或把低收益选择提前标成错误。

“规则”可以直接内嵌在所属委托中；只有确实跨多份委托复用时才拆共享定义，避免一条规则一个资产。

### 3.7 Letter Definition

一封信一个资产，保存信件 ID、标题、发件人、正文、图片/版式和阅读后可产生的内容引用或事件。不把信件正文复制进委托。

### 3.8 Clue Definition 与 Inference Definition

Clue 保存单条可获知信息的 ID、标题、正文、图片、来源表现和分类。Inference 保存所需线索引用、形成后的推论内容，以及可开放的对话/派遣信息或剧情事实。

“资产存在”和“玩家已获得”分开；推论表示玩家发现了联系，不必自动声明为客观真相。

### 3.9 Day Definition

保存日号、标题、当天可展示的内容引用、允许收尾的必要条件，以及固定三个撰写部分及其选项。

不保存小时/分钟计时器。Demo 日期由玩家主动收尾、提交撰写并睡觉推进；自由阅读和探索不消耗日次。

## 4. 共享规则语言

所有 Episode、Dialogue、Commission 和 Day 使用同一套类型化条件/效果语义，避免各系统用字符串重新解释同一个结果。

条件至少能表达：当前日/阶段、是否读信、对话选项、是否拥有线索/推论、委托状态、派遣结果、NPC 当前状态、命名事实或标签。

效果至少能表达：发布/接取委托、开始对话、授予线索、形成推论、写入事实、开放内容、请求进入撰写或章节出口。

每条规则必须能回答：何时触发、作用范围、检查哪些条件、执行哪些效果、重复策略、冲突优先级、失败/取消后怎么办。

## 5. 运行实例不属于内容表

确认派遣时创建 Dispatch Instance，至少快照：实例 ID、委托引用、NPC 引用、出发日、预计返回日、实际交付物及数量、告知的情报、出发时 NPC 状态、命中规则和锁定结果。

之后编辑 Commission DA 不改变已出发实例。NPC 实际返回与玩家听取报告是两个状态；物理后果可先发生，报告中的信息只有在听取后才成为玩家已知内容。重复打开报告不能重复发奖励。

这部分是数据合同，不要求本阶段继续扩展 NPC 场景表现。

## 6. DataAsset 与 DataTable 的使用边界

正式剧情主内容优先使用 DataAsset，因为它们具有跨资产引用、嵌套规则、图片/对话/结果关联和独立校验需求。

DataTable 只适合大量、同构、无复杂引用的叶子数据，例如简单物品参数或将来明确需要批量导入的文本索引。旧 `DT_NPCDialogue`、混合 NPC 表和旧线索表作为迁移来源/兼容层保留，在引用审计与迁移验收前不删除；新内容不再要求设计者同时维护 DA 与旧行名。

## 7. 设计者实际编辑路径

```text
从 Campaign 打开 Content Catalog
  → 创建/选择人物资料
  → 创建完整对话、委托、线索、推论、信件和 Day
  → 在 Commission 中配置派遣条件与结果引用
  → 在 Episode 中只编排触发、条件、动作和后续
  → 运行跨资产校验
  → 选择正式 DA 建立隔离预览
```

打开委托时，编辑器应能查看并跳转到被引用的 NPC、准备内容、结果报告、线索和后续剧情节点；不要求在多个表中复制同一段文字。

## 8. 预览数据规则

派发页面独立预览应引用设计者选择的正式 Commission、NPC、物品、Clue/Evidence 等资产，并创建临时 Commission/NPC/库存/剧情运行状态。关闭预览后清理运行状态，不写存档、不回写 DA。

C++ 固定样例只允许用于自动化夹具或“没有任何配置时”的显式开发占位，不作为设计者日常编辑来源。修改正式 DA 的图片、文字或规则后，再次预览必须直接反映修改。

## 9. 本基础阶段的完成标准

- 每类内容的职责和引用方向确定，NPC 不再混入对话与场景动作。
- 委托能完整表达展示、开放、派遣准备、条件结果和报告引用。
- 统一条件/效果可被 Episode、Dialogue、Commission、Day 共用。
- Catalog 能发现重复、空引用、类型错误和未登记引用。
- 明确旧表到新资产的字段迁移表，但不在未审计前删除旧资产。
- 派发预览读取正式可编辑 DA，只隔离运行状态。
- 用一份委托、两名 NPC、两条线索和一条推论验证数据关系；不要求先实现 NPC 地图活动。

本设计作为后续字段盘点、改造计划、迁移和样例验收的基础。业务代码、正式资产和旧表仍须按阶段计划修改与验证；本文完成不代表这些改造已经实现。

## 10. 当前项目类型盘点与结论

2026-09-14 已按源码完成只读盘点。目标模型的大部分骨架已经存在，不应重写：

| 当前类型 | 结论 | 主要问题 |
|---|---|---|
| `UStoryCampaignDefinition` | 保留 | 已有 Catalog、Episode、MaxDays 和 RootStateTree；正式资产仍缺真实 StartingEpisode，属于内容接线问题 |
| `UStoryContentCatalog` | 保留并作为唯一内容目录 | 已按七种内容分类；需要让新增正式内容只走这里，旧表不再成为并行权威 |
| `UStoryEpisodeDefinition` | 保留 | 已有节点、触发、条件、效果、出口；编辑体验和可读诊断仍需后续完善 |
| `UNPCDefinition` | 保留并收紧字段语义 | 基本已分离；`DispatchStatusText` 是静态展示补丁，容易与真实运行状态冲突；`InitialCurrentState` 只能表示初始值，不能演变成活动配置 |
| `UDialogueDefinition` | 保留 | 已经是一段会话一个 DA；但仍使用独立的 `FDialogueCondition/FDialogueEffect` 和大量 ID，尚未与 Narrative 共享规则语言 |
| `UCommissionDefinition` | 保留并作为委托唯一正文 | 已包含展示、日数、准备、规则与结果；仍存在旧 `FQuestInfo` 的运行状态字段、旧 Lead、旧 ID 和 DataTable RuleSet 兼容层 |
| `UClueDefinition` | 保留 | 已是正式 DA；旧 `DT_Clues` 仍作为兼容来源 |
| `UInferenceDefinition` | 保留 | 已引用 Clue DA；当前运行配对仍主要针对两条线索，设计上不要误报为任意数量已完成 |
| `ULetterDefinition` | 保留 | 正文分段仍用 ClueID；后续应改为可校验的 Clue DA 引用或统一事实引用 |
| `UDayDefinition` | 保留 | 已包含三段撰写；`RequiredWorkCount` 与旧 AvailableEvents 是早期入口，应逐项判断是否迁到统一条件/Episode |
| `FCommissionInstance` 等运行结构 | 保留为运行时 | 不属于设计者正文资产，不应搬进 DataTable 或 DA |

当前数据层最主要的结构性缺口有四项：

1. **同一条件有多套表达**：Narrative、Dialogue、Commission、DailyWriting 分别拥有条件结构，作者要学习并维护多种规则语言。
2. **正式资产引用与裸 ID 混用**：Catalog 已存在，但 Dialogue、Commission、Letter、Writing 中仍有不少 `FName` 内容 ID，跨资产校验和重命名体验不一致。
3. **委托展示仍复用旧 Quest 结构**：`FQuestInfo` 内含 `QuestState`，把作者正文与运行状态再次混到同一结构；正式 Commission DA 不应保存运行态。
4. **旧 DataTable 与正式 DA 双轨**：配置仍可加载 Quest、Dialogue、Clue、CommissionRules、Report 等旧表，设计者无法直观看出哪个才是权威来源。

## 11. 目标字段模型

以下是数据层的目标形态；名称可在实施时按 UE 兼容性调整，但职责不再改变。

### 11.1 通用内容头

所有正式内容实现同一接口并提供：

| 字段 | 用途 |
|---|---|
| `ContentID` | 稳定身份；在一个 Campaign Catalog 内唯一 |
| `DisplayName` | 编辑器列表和诊断使用，不代替正文标题 |
| `Icon` | 可选的目录缩略图 |
| `AuthoringNotes` | 仅制作备注，不进入运行事实或玩家 UI |

不强制立刻新增共同父类并迁移全部序列化字段；现有各自的 LetterID、DialogueID 等可以由接口映射为 ContentID，避免不必要的资产破坏。

### 11.2 NPC 数据

<a id="npc-four-fields-current"></a>

#### NPC 四行信息当前确认（2026-10-28恢复核对）

| 显示字段 | 用户确认的含义与选项 | 当前实现差距 |
|---|---|---|
| 健康 | 总体身体风险：健康、疲劳、受伤、重伤 | 已有 ENPCBodyState；该类型名称不代表UI“身体”评估 |
| 身体 | 正式综合适配规则尚未设计；记录本模拟先展示足以胜任、恐怕吃力、难以胜任 | 已移除仅健康映射；每条隔离预览可启用bPreviewAssessmentEnabled并选择PreviewAssessment，正式ReadInstance不读取样例值；本轮隔离/读取/渲染相关11项通过，人工待复看，不能报正式综合评估完成 |
| 状态 | 心情：平静、警惕、振奋、不安、低落、失控 | 已有ENPCMoodState及初始/运行时映射 |
| 信任 | 合作程度：陌生、普通、友好、信任、亲密 | 已有ENPCTrustLevel |

- 界面名称继续叫“身体”；健康但不适合某委托的NPC也可能“恐怕吃力”。评估与具体NPC/委托相关，不能简单从健康一对一推导，也不能用“正常”或旧伤自由文本代替三项评估。
- ENPCCurrentState是独立内部活动状态：空闲、交谈中、工作中、执行委托中、休息中、不可派遣，不显示在这四行里。
- 2026-10-28用户最新澄清：综合评估的决定条件尚未想好；当前只要求记录本模拟展示三个已定选项的效果。因此本轮目标是每条隔离预览可选择三项之一，不计算适配，不从健康推导，不写NPC正式资产。正式权重与阈值继续待设计；旧助手百分比不是批准规则。预览字段实施/验证进度仍见矩阵，不能仅凭此目标声称已完成。
- 确认来源：原对话“设计委托记录本左页” `01a133fa-a73e-7ac0-a629-6e36076fa45d`；用户三项及健康不等于适配的确认在回合 `01a17d42-1174-7181-9fb2-9af225f5649c`，保留“身体”名称在 `01a17d61-6e3f-71a1-a382-f0176a6c2f72`。详见[原对话核对](handoffs/2026-10-28-original-thread-context-audit.md)。此为设计确认，不能据此推定代码/正式资产/人工验收均完成。

目标 `NPC Definition` 分组：

| 分组 | 字段 |
|---|---|
| Identity | NPCID、DisplayName、Profession |
| Presentation | Portrait、Description |
| Authored Traits | ContentTags/规则特征 |
| Initial State | InitialBodyState、InitialTrustLevel、初始可用状态 |
| Technical Binding | CharacterClass（可选，仅绑定表现类） |

`DispatchStatusText` 不作为长期权威状态。派发 UI 应把稳定人物简介和当前运行状态分别展示；否则 DA 写“健康”而运行时已经受伤时会冲突。

### 11.3 Dialogue 数据

目标 `Dialogue Definition`：

| 层级 | 字段 |
|---|---|
| Session | DialogueID、DisplayName、Participants、StartNodeID |
| Node | NodeID、NodeType、Speaker NPC 引用、LineText、EntryConditions、Effects、Next/Choices |
| Choice | ChoiceID、ChoiceText、Conditions、Effects、NextNodeID |
| Exit | ExitID/ResultID、完成效果和后续事件 |

当前效果只挂在 Node，Choice 自身没有独立 Effects；目标模型需要让每个选择直接拥有自己的效果，避免靠跳到一个隐藏节点才能表达“选择后授予线索/开放委托”。Speaker 使用 NPC 资产引用，`SpeakerName` 只允许作为旁白或兼容字段。

Dialogue 的条件/效果在作者界面中使用统一 Narrative 规则。运行时可以先通过适配器转换，避免一次重写现有 DialogueSubsystem。

### 11.4 Commission 数据

`UCommissionDefinition` 目标字段拆分：

```text
Identity
  CommissionID

Presentation
  Title / Summary / Description / Meta / Location
  ListIcon / DetailImage / MapImage / SpecialRequirements

Availability
  OfferConditions
  AcceptConditions
  ExpirationPolicy

DispatchSetup
  EarliestDispatchDay / ExecutionDays / FinalDayPolicy
  MaxPreparationSlots / AllowedPreparationTypes
  HardDispatchConstraints

OutcomeRules[]
  RuleID / Priority / Conditions / PreviewReactionText / Outcome

DefaultOutcome
  无条件兜底
```

关键整理：

- 将 `FQuestInfo` 中的展示字段迁入纯展示结构，删除/弃用其中的 `QuestState` 写入路径。
- `DefaultOutcomeID` 不只保存 ID；目标是显式默认 Outcome，或保证它能唯一解析到一个无条件规则。
- `FCommissionLeadDefinition` 不再作为新内容的推论权威；改为引用 `UInferenceDefinition`。旧 Lead 仅保留迁移兼容。
- RequiredClueIDs/LeadIDs、GrantedClueIDs 等逐步换成 Soft DA 引用；实例快照仍可同时保存稳定 ID。
- 每条结果规则有稳定 `RuleID`，实例保存 RuleID 而不是只保存数组下标；数组重排后审计仍可读。

`FCommissionOutcome` 继续按以下维度组合，而不是为每种交付组合复制完整结果：

| 维度 | 内容 |
|---|---|
| Investigation | ResultGrade、标题、摘要、调查事实 |
| NPC Consequence | 身体/可用状态、信任或其他明确变化 |
| Inventory | 消耗、归还、获得、遗失 |
| Knowledge | 玩家听报告后获得的 Clue/Inference |
| Presentation | ReturnDialogue、ResultLetter |
| Narrative | 统一事实/效果；旧 NamedResults 和 StoryTags 作为兼容映射 |

### 11.5 Clue、Inference 与 Letter

- Clue 的 `SubjectID/CategoryID/DefaultSourceID` 是分类或来源标签，不用于代替正式内容引用。
- Inference 直接引用所需 Clue DA，并明确 `RequiredClues` 数量规则；Demo 当前先固定两条，编辑器必须把限制写清楚。
- Letter 的线索段从裸 `ClueID` 迁向 Clue DA 引用；段落只控制高亮和阅读交互，不复制 Clue 正文。
- 获得日期、来源实例、是否已读都属于运行记录，不加入 Definition。

### 11.6 Day 与每日撰写

目标 `Day Definition`：

| 分组 | 字段 |
|---|---|
| Identity | DayNumber、DayTitle |
| Presentation | NotebookLocation、NotebookImage、NotebookMarginNote |
| Available Content | 当日可展示 Commission 等正式内容引用 |
| End Requirements | 统一 Narrative Conditions，返回具体未满足原因 |
| Daily Writing | 恰好三个 Section，每段恰好一个无条件默认选项，其余为条件选项 |

`FDailyWritingCondition` 最终也应由统一 Narrative Conditions 表达。Credibility/HiddenStance/EndingTags 是写作结果，不应反向成为选项显示条件。

## 12. 引用规范

在作者资产中优先保存 `TSoftObjectPtr<具体 Definition>`，运行时事实和存档保存资产身份与稳定 SubID：

```text
作者编辑：Commission → ReturnDialogue DA
运行事实：SourceAsset=Commission DA, SubjectAsset=Dialogue DA, SubID=OutcomeID
存档快照：资产路径/PrimaryAssetId + 稳定 ContentID + 实例 ID
```

规则：

- 内容之间不只靠裸 `FName` 猜类型。
- 子元素（NodeID、ChoiceID、OutcomeID、OptionID）可继续使用 FName，但必须在所属资产内唯一。
- EventID、InstanceID、RequestID 继续分别表示事件、真实运行实例和一次动作请求，不能替代内容 ID。
- 软引用用于内容关系；运行中的 Actor/Widget 指针永不写入正式内容资产。

## 13. 旧数据迁移矩阵

| 旧来源 | 新权威位置 | 处理方式 |
|---|---|---|
| `DT_Quests` / `FQuestContentRow` | `UCommissionDefinition.Presentation` | 复制展示字段；DocumentID 改为 Letter/Document 资产引用；不迁入运行状态 |
| `FQuestInfo.QuestState` | Commission/Quest 运行实例 | 从作者 Definition 中移除或弃用 |
| `DT_NPCDialogue` | 每段会话一个 `UDialogueDefinition` | 按会话合并节点；Speaker 映射 NPC DA；条件/效果转统一规则 |
| `DT_Clues` | `UClueDefinition` | 一行生成/对应一个 Clue DA；旧表保持只读兼容直到引用归零 |
| `DT_Documents` | `ULetterDefinition` 或以后独立 Readable Definition | 按文档语义迁移，信件不再仅靠 DocumentID 查找 |
| `CommissionRulesTable` | `UCommissionDefinition.OutcomeRules` | 合并展示与执行规则到同一委托资产；保留一次性迁移读取 |
| ReportTemplate/Section/Option/WritingDefinition 表 | `UDayDefinition.DailyWriting` | 按日聚合为恰好三个 Section；运行提交记录保持独立 |
| `FCommissionLeadDefinition` | `UInferenceDefinition` | 能映射的迁为 Inference 引用；旧字段标记迁移兼容 |
| `NPCSceneTable` | 后续独立场景/活动数据 | 当前阶段不迁、不删除，不放回 NPC Definition |

2026-09-14 序列化兼容调整：现有 UObject 硬引用或 `FName` 属性不原地改型，避免同名属性类型变化使旧 `.uasset` 丢值。正式入口采用新增属性（Letter `Clue`、Outcome `GrantedClues`/`ReturnDialogueReference`/`ResultLetterReference`、Day `NotebookCommissionReferences`、Writing `RequiredClues`）；旧 `ClueID`、`GrantedClueIDs`、`ReturnDialogue`、`ResultLetter`、`NotebookCommissions`、`RequiredClueIDs` 标记弃用并仅在正式字段为空时回退。迁移报告归零前不删除旧属性。

迁移顺序必须是：清点资产与引用 → 生成映射报告 → 创建/补齐新 DA → 双读验证 → 正式路径切到 Catalog → 引用归零后再讨论删除旧表。当前没有删除授权。

## 14. 最小数据样本

数据模型验收不需要先实现完整第一天或 NPC 移动。使用独立测试目录建立：

- 1 个 Campaign、1 个 Episode、1 个 Day；
- 2 个 NPC Definition；
- 1 个 Commission Definition；
- 2 个 Clue、1 个由两条线索形成的 Inference；
- 2 段 Dialogue：接取/准备交流与两个结果报告；
- 1 封 Letter；
- 3 个撰写 Section，每段至少一个默认选项和一个条件选项；
- 2 条委托结果规则加 1 个默认结果。

只验证数据关系：未调查也能派遣并命中默认结果；获得推论后选择不同 NPC/准备能命中另一结果；返回报告引用对应对话并开放撰写选项；修改 DA 图片或文字后预览立即变化。地图移动和正式多日内容均不属于该样本。

## 15. 数据层实施顺序

1. 先写字段迁移与序列化兼容测试，不改正式资产。
2. 抽出纯展示 Commission 数据，阻止 Definition 携带运行状态。
3. 为 Dialogue Choice 补直接 Effects，并建立统一条件/效果适配层。
4. 把 Commission/Letter/Writing 中可明确迁移的裸内容 ID 改为正式 DA 软引用，同时保留兼容读取。
5. 增强 Catalog 跨资产校验与可读错误路径。
6. 提供旧表只读扫描和迁移报告；用户确认后才创建/修改正式 DA。
7. 让派发隔离预览选择正式 DA，不再创建 C++ 正文样例。
8. 用第 14 节样本做自动化与人工数据编辑验收。

这一顺序完成后，设计者的主要工作是创建 DA、填写内容、选择引用和配置规则；新增一份普通委托不需要新增专用 C++ 分支。

本文是当前剧情数据结构的权威模板。历史 DataTable 结构和归档计划仅作为迁移来源；如果实现过程中发现序列化或编辑器限制，需要先记录到 `DECISIONS.md`，再同步修订本文，不能另建第三套并行数据规范。

2026-10-22 记录本UI日期补充：FQuestInfo新增DispatchDay/CompletedDay，0代表未记录，供纸卡印章显示派遣/完成日。它们是调用方提供的展示数据，不持久化真实委托、不把DueDay视为CompletedDay；FStoryNotebookQuestPreviewEntry有同名字段仅供快速预览。不修改正式CommissionDefinition/Campaign。

### 2026-10-25 记录本读取适配补充（UI-DISPATCH-001）

FNotebookCommissionRecord仅为只读关联/预览输入：Commission、Executor引用，可选真实InstanceID，以及隔离预览阶段日期。正式执行关系仍以CommissionInstance.NPCID为权威，不写回Commission/NPC DA。NotebookCommissionData统一映射列表与派遣页；显式实例优先且开页固定身份，实际NPCInfo优先初始状态。Story Workspace预览每条可配置已关联执行人，不增加玩家选择操作。此步不填正式内容，不增加独立剧情账本；报告只在已应用效果的Resolved实例可回看，未知物品去向/见闻/历史返回日不能推断。
