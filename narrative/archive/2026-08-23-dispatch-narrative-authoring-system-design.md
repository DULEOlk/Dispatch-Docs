# Dispatch 统一剧情制作与因果图设计

日期：2026-08-23

## 1. 背景与问题

Dispatch 已经具备信件、NPC 对话、委托、线索组合、每日撰写、世界状态、StoryDirector 和 StateTree 等运行时模块。这些模块可以互相联动，但内容制作入口仍然分散：设计者需要分别进入多个 Data Asset、DataTable、StateTree 和蓝图设置条件与结果，无法直观看见“玩家行为为什么导致下一段剧情”。

当前缺失的不是另一个总控制器，而是统一的剧情创作与因果表达层。StateTree 目前承担了过多剧情步骤，却更适合控制场景生命周期、教程、转场和阻塞演出；具体剧情意义与玩家选择后果应由专门的剧情图负责。

## 2. 目标

- 提供一个真正像“写剧本”的统一制作入口。
- 在一张按天组织的剧情图中查看玩家行为、条件、结果和后续分支。
- 让信件、对话、线索、委托、NPC、交付物和每日撰写使用同一套跨系统条件与结果。
- 让玩家选择产生真实的后续差异，而不只是改变隐藏数值。
- 保持内容资产模块化，但在剧情编辑器中统一创建、查找和编辑。
- 从任意剧情事实追踪其产生位置和使用位置。
- 支持校验、路线模拟、运行时调试和存档恢复。
- 先完成第一日纵向样板，再扩展七天与结局。

## 3. 非目标

- 不使用剧情图控制所有游戏技术细节。
- 不把教程、输入模式、镜头、Actor 生成和场景加载全部迁入剧情图。
- 不把所有台词、图片和委托数据塞入一个巨型结构体或单一 Data Asset。
- 不在第一阶段一次性删除所有旧接口或旧 DataTable。
- 不允许 StateTree、内容 DA 和剧情图同时成为跨系统剧情因果的权威来源。

## 4. 最终架构

### 4.1 剧情图：决定发生什么

剧情图负责：

- 当前剧情节点。
- 玩家行为产生的剧情事实。
- 分支条件。
- 跨系统结果。
- 下一段剧情。
- 下一日入口。
- 结局选择。

剧情图不直接管理场景加载、镜头、教程和输入状态。

### 4.2 StateTree：执行宏观游戏流程

StateTree 只保留：

- 游戏启动与开场。
- 教程。
- 日期和关卡生命周期。
- Actor 生成与场景准备。
- 镜头、过场和特殊阻塞演出。
- 等待当日剧情完成。
- 夜晚转场。
- 结局演出。
- 技术失败恢复。

根流程收敛为：

```text
Boot
→ Opening
→ Tutorial
→ StartDay
→ RunNarrativeEpisode
→ DayTransition
→ Ending
→ FailureRecovery
```

StateTree 不再保存玩家选择的剧情意义，也不再重复实现对话、委托或撰写分支。

### 4.3 内容资产：提供具体内容

- `ULetterDefinition`：一封完整信件。
- `UDialogueDefinition`：一次完整对话会话及内部台词跳转。
- `UCommissionDefinition`：一个完整委托及内部结果计算。
- `UClueDefinition`：一条完整线索，逐步替代 `DT_Clues`。
- `UDayDefinition`：日期展示、线索页和每日撰写模板。
- NPC 定义资产：角色身份、基础展示和初始状态。

内容资产可以返回稳定结果，但不得自行决定另一个系统的跨内容后续。

### 4.4 剧情状态：记住已经发生的事

`UNarrativeRuntimeSubsystem` 是剧情位置和剧情规则的唯一运行时权威。`UStoryWorldStateSubsystem` 继续保存标签、数值和命名结果，但不自行选择剧情路线。

## 5. 剧情资产组织

### 5.1 Campaign

新增 `UStoryCampaignDefinition`，整个 Demo 只有一个正式 Campaign：

```text
DA_StoryCampaign_Demo
├─ DA_StoryEpisode_Day01
├─ DA_StoryEpisode_Day02
├─ DA_StoryEpisode_Day03
├─ DA_StoryEpisode_Day04
├─ DA_StoryEpisode_Day05
├─ DA_StoryEpisode_Day06
├─ DA_StoryEpisode_Day07
└─ Ending Definitions
```

Campaign 保存：

- Campaign ID。
- 开局 Episode 和 Entry ID。
- 七天 Episode 直接引用。
- 结局入口。
- 全局角色与内容目录。
- 根 StateTree 引用。

### 5.2 Episode

一个 `UStoryEpisodeDefinition` 表示一天或一个独立章节，保存：

- Episode ID。
- Day Number。
- 默认 Entry ID。
- 可选的其他 Entry ID。
- 当日出场角色和可用内容。
- 剧情节点与连线。
- 当日完成出口。
- 下一日入口映射。

每一天一张子剧情图；Campaign 提供七天和结局的总览图，避免单张巨型图。

## 6. 剧情节点模型

第一版支持以下节点：

- Start。
- Dialogue。
- Letter。
- Clue。
- Lead/Inference。
- Commission。
- Item Delivery。
- NPC State Change。
- Writing。
- Scene Request。
- Branch。
- Jump To Episode。
- Ending。
- Failure/Fallback。

每个节点统一拥有：

```text
Node ID
Display Name
Entry Conditions
Action
Completion Result
Transitions
Failure Transition
Run Once
```

内容节点使用直接资产引用。内部稳定标识（例如 `ChoiceID`、`OutcomeID`、`OptionID`）必须由所选资产提供下拉列表，禁止依赖自由手填字符串。

## 7. 统一规则表达

每条跨系统规则统一写成：

```text
WHEN：检查时间或触发事件
IF：条件
THEN：结果
NEXT：下一个剧情节点
```

### 7.1 检查时间

- Immediate。
- OnDialogueFinished。
- OnCommissionResolved。
- OnWritingSubmitted。
- OnDayStarted。
- OnDayEnded。
- OnEpisodeFinished。

### 7.2 条件

- 当前日期。
- 是否拥有线索。
- 是否形成推论。
- 是否阅读信件。
- 对话是否完成及 Choice/Result。
- 委托状态及 Outcome。
- 是否向某 NPC 交付某物。
- NPC 信任、身体和行为状态。
- 撰写 Option、可信度与隐藏立场。
- 剧情标签、命名结果和数值。
- 某节点或 Episode 是否完成。

条件支持 All、Any、Not 和嵌套组合。

### 7.3 结果

- 开始或开放对话。
- 开放委托。
- 投递信件。
- 获得线索或推论。
- 修改 NPC 信任、身体和行为状态。
- 生成或移除 NPC。
- 解锁撰写选项。
- 增加结局倾向。
- 设置剧情事实、标签或数值。
- 请求播放场景。
- 跳转节点、Episode 或下一日入口。
- 选择结局。

纯数据结果由 Narrative Runtime 应用；涉及世界、镜头、UI 或场景的动作交给 StoryDirector/StateTree 执行。

## 8. 标准事件与事实账本

现有系统统一输出标准事件：

- LetterDelivered / LetterOpened / LetterRead。
- DialogueStarted / DialogueChoiceSelected / DialogueCompleted / DialogueFailed。
- ClueCollected / LeadUnlocked。
- CommissionOpened / CommissionDispatched / ItemDelivered / CommissionResolved / CommissionFailed。
- WritingOptionSelected / WritingSubmitted。
- NPCStateChanged。
- DayStarted / DayEnded。

Narrative Runtime 将需要持久化的事件转换为 `FStoryFactRecord`，至少包含：

- Fact Type。
- 内容资产引用。
- 本地结果 ID。
- Day。
- Source Episode/Node。
- 时间或顺序号。
- 数值或名称载荷。

调试器能够显示一条事实由哪个节点产生、被哪些条件使用，以及为何触发当前跳转。

## 9. 运行时流程

`UNarrativeRuntimeSubsystem` 保存：

- 当前 Campaign。
- 当前 Episode 和 Entry。
- 当前节点状态。
- 已完成节点。
- 已执行一次性规则。
- 事实账本。
- 当前等待的 Action Request。
- 下一日入口和结局结果。

异步节点状态为：

```text
Ready → Running → WaitingForResult → Completed / Failed
```

每次动作带唯一 Request ID。内容系统返回结果时必须匹配 Request ID，避免旧回调、重复广播或重入导致错误推进。失败节点必须有显式失败出口或 Episode 默认失败出口。

## 10. StoryDirector 桥接

StoryDirector 负责：

- 启动 Campaign、Episode 和 Entry。
- 在 StateTree 与 Narrative Runtime 之间转发生命周期请求。
- 将剧情动作请求路由到信件、对话、委托、线索、撰写和场景系统。
- 将内容完成结果送回 Narrative Runtime。
- 在 Episode 完成后通知 StateTree 转场。
- 请求指定结局演出。

StoryDirector 不保存跨系统分支规则。

## 11. 统一制作界面

打开 `DA_StoryCampaign_Demo` 后显示：

- 顶部：总览、第一日至第七日、结局标签页。
- 左侧：角色、线索、信件、对话、委托、物品、撰写、场景、结局素材库。
- 中间：剧情节点与分支连线。
- 右侧：选中节点或连线的条件、结果和内容详情。
- 底部：校验、引用、路线模拟和运行时追踪。

编辑器提供内容快捷创建按钮。底层仍创建模块化资产，但设计者不需要离开剧情编辑器寻找文件。

## 12. 校验与模拟

保存或运行前检查：

- 缺失资产引用。
- 重复 Node/Entry ID。
- 无法从开局到达的节点。
- 没有出口或默认出口的判断节点。
- 两个互斥分支可能同时成立。
- 条件引用了没有任何节点会产生的事实。
- 内容资产内部 ID 不存在。
- 跨日引用错误。
- 没有可进入路线的结局。
- 循环没有退出条件。

模拟器允许选择玩家结果，在不启动完整关卡的情况下计算可到达路线。PIE 调试时高亮当前节点，并显示每个出口满足或失败的原因。

## 13. 内容职责边界与迁移

保留内容内部逻辑：

- DialogueDefinition 管理本次会话内部台词与选择跳转。
- CommissionDefinition 管理委托准备和 Outcome 计算。
- ClueSubsystem 管理收集、日期和组合。
- DailyWritingSubsystem 管理可用选项、提交、可信度和隐藏立场。

迁移到剧情图的逻辑：

- Dialogue Effect 直接 OpenCommission、启动后续对话或修改跨系统结果。
- Commission Outcome 直接启动对话、投递信件或决定第二日。
- StateTree 内针对具体玩家选择的剧情判断。
- 蓝图中隐藏的剧情后续调用。

旧接口先标记为 Migration/Deprecated；第一日完全迁移并通过引用审计前不得删除。

## 14. 第一日纵向样板

第一日必须覆盖：

```text
开场
→ 投递并阅读信件
→ NPC 对话选择
→ 获得线索
→ 组合线索形成推论
→ 开放不同委托
→ 派遣 NPC
→ 委托结算和物品交付
→ 解锁不同撰写选项
→ 提交撰写
→ 进入两个真实不同的第二日入口
```

至少实现：

- Truth 路线：获得关键线索、委托部分成功、如实撰写、第二日继续调查。
- Conceal 路线：缺少关键线索、委托失败、迎合官方、第二日记录员失踪。

差异必须体现在可见内容、NPC 状态和后续剧本，不得只修改隐藏数值。

## 15. 存档

剧情存档至少包含：

- Campaign、Episode、Entry 和当前节点。
- 已完成节点和已执行规则。
- 事实账本。
- Pending Request。
- NPC 状态与信任。
- 委托实例和结果。
- 线索和推论。
- 撰写结果。
- 下一日入口。
- 结局倾向。

需要覆盖对话中、委托进行中、撰写前后和跨日后的存读档测试。

## 16. 建设顺序

1. 统一事件、事实、条件、结果和转换类型。
2. Campaign/Episode/Graph 数据资产与纯逻辑运行时。
3. 现有内容系统标准事件适配。
4. StoryDirector 与宏观 StateTree 桥接。
5. 专用剧情图编辑器、校验器和模拟器。
6. 第一日纵向样板。
7. 运行时调试器。
8. 剧情存档。
9. 第二日至第七日和全部结局迁移。
10. 旧跨系统接口引用审计与清理。

每个阶段完成后独立编译和自动化测试，不在一个长任务中无检查地同时迁移七天。

## 17. 完成标准

- 设计者只需打开 Campaign 编辑器即可查找和编辑当日剧情相关内容。
- 可以从开局沿分支查看到任意结局。
- 可以查询线索、Choice、Outcome 或 Writing Option 的产生位置与所有消费者。
- 玩家选择能够进入至少两个真实不同的第二日剧本。
- StateTree 不包含具体玩家选择的跨系统剧情因果。
- 内容 DA 不再隐藏启动其他内容的正式逻辑。
- 第一日至第七日均有默认可完成路线和安全出口。
- 玩家无需测试蓝图手动设置结果即可连续完成七天并进入不同结局。
- 存档能够恢复剧情位置和所有关键事实。
- Campaign 全局校验无错误，完整自动化测试通过。

