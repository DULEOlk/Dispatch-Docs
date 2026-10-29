# 统一委托定义与结果系统设计规格

**日期：** 2026-08-04
**状态：** 已完成设计讨论，等待书面规格复核
**项目：** Dispatch 轻量 NPC 委托派遣系统

## 1. 目标

建立一条由 `CommissionID` 驱动的统一委托链路，使一个委托的展示内容、派遣规则、玩家准备、NPC 反应、最终结果和剧情影响都能从同一个入口解析和验证。

系统必须支持：

- 玩家先选定一个待派发委托，再与当前 NPC 交互并打开委托界面；
- 当前 NPC 和最多三个混合准备内容共同决定结果；
- 准备内容可为物品、线索或证据，允许留空；
- UI 随准备内容变化显示当前 NPC 的预判反应；
- 同一委托因 NPC 和准备组合不同产生明确不同的结果；
- 结果在派遣确认时确定并锁定，在委托完成时只结算一次；
- Widget Blueprint 只负责布局和美术，业务逻辑由 C++ 负责；
- 委托规则不散落在 Widget、NPC Blueprint、StateTree 或关卡脚本中。

## 2. 非目标

首版不实现：

- 显式或隐藏的成功率、风险百分比和随机判定；
- NPC 数值属性、复杂能力值或属性成长；
- 多 NPC 同时执行一项委托；
- 可变 NPC 槽位或复杂编队；
- 物品重量和复杂负重；
- 复杂指令、路线规划或完整日程系统；
- 委托运行期间重新计算或改变已经锁定的结果；
- 在 StateTree 中逐项配置委托结果条件；
- 在本功能内重新制作报告撰写系统。

结果信件、返回对话和后续场景可以通过 ID 接入现有系统，但对应内容允许在首个增量中暂时为空。

## 3. 当前项目问题

当前委托相关信息分散在多个结构中：

- `FQuestContentRow` 保存任务标题、摘要、地点、图片和文档引用；
- `FQuestContentRow.DispatchRuleID` 已存在但尚未连接派遣规则；
- `FQuestDefinition` 实际用于报告撰写，不是完整委托定义；
- `FNoticeQuest` 重复保存标题、描述、地点和图片；
- `FQuestInfo` 又保存一份运行时展示信息；
- `UNPCCommissionComponent` 只保存 `FQuestInfo` 和状态，不保存准备内容与结果；
- `UCommissionUIBase` 仍采用旧的“委托列表 + 选择索引”模式；
- 当前没有统一的结果规则、结果评估器和幂等结算流程。

继续在现有结构上直接添加 UI 逻辑，会让条件和结果进一步散落在蓝图、NPC 组件、StateTree 和任务表中。

## 4. 核心设计原则

### 4.1 一个逻辑入口，不制造一张巨型表

统一指所有委托数据都能从一个 `CommissionID` 解析，而不是把对话正文、物品定义、线索正文和报告内容复制进同一行。

每类内容只保留一份权威来源：

- `DT_QuestContent`：标题、摘要、地点、图片和描述文档引用；
- `DT_CommissionRules`：委托派遣配置、条件分支、NPC 预判反应和全部结果效果；
- `DT_NPCDialogue`：正式对话内容；
- 现有物品数据：物品名称、描述、图标和数量；
- `DT_Clues`：线索定义；
- 现有报告表：报告撰写结构。

委托规则通过 ID 引用这些权威内容，不复制它们。

### 4.2 `CommissionID` 与 `QuestID`

首版将可派遣委托的 `CommissionID` 与现有 `QuestID` 视为同一个稳定标识。

命名示例：

```text
Quest.HeartlessDeer
Quest.MerchantMissing
```

`DT_QuestContent` 行名直接作为 `QuestID`。其 `DispatchRuleID` 指向 `DT_CommissionRules` 的行名；正常情况下二者使用相同值。

- `DispatchRuleID=None`：该任务不可派遣 NPC；
- `DispatchRuleID` 有值：必须能解析到对应规则行；
- 规则行内部 `QuestID` 必须与来源任务一致，否则验证失败。

保留 `DispatchRuleID` 允许未来多个任务复用同一套规则，但 Demo 默认使用一对一同名配置。

### 4.3 确定性结果

结果完全由派遣确认时的以下快照决定：

- `CommissionID`；
- 当前 NPC 的 `NPCID`、委托标签、身体状态和信任等级；
- 三个准备槽中的内容；
- 规则明确声明需要读取的线索或世界状态。

系统不使用随机数。结果在确认派遣时计算并保存，但在委托返回之前不向玩家公开。

## 5. 数据模型

### 5.1 准备内容

```cpp
UENUM(BlueprintType)
enum class ECommissionPreparationType : uint8
{
    Item,
    Clue,
    Evidence
};

USTRUCT(BlueprintType)
struct DISPATCH_API FCommissionPreparationRef
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    ECommissionPreparationType Type = ECommissionPreparationType::Item;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FName ContentID = NAME_None;
};
```

`Type + ContentID` 共同构成唯一引用，避免物品、线索和证据使用相同 `FName` 时发生冲突。

三个准备槽是混合槽位：

- 允许不同类型任意组合；
- 允许留空；
- 同一内容默认不能重复占用多个槽；
- 槽位数量首版固定为三，数据仍保留 `MaxPreparationSlots=3` 以便验证；
- 预览和取消界面不能消耗玩家内容；
- 只有确认派遣成功后才处理需要消耗的内容。

### 5.2 NPC 委托标签

在 `UNPCCommissionComponent` 增加 `FGameplayTagContainer CommissionTags`，用于描述与派遣有关的分类能力，而不引入数值属性系统。

示例：

```text
NPC.Skill.Hunting
NPC.Skill.Tracking
NPC.Skill.Medicine
NPC.Trait.Brave
NPC.Trait.Cautious
```

规则同时允许使用精确 `NPCID` 和标签：

- 精确 NPC 条件用于专属剧情；
- 标签条件用于多个 NPC 共用同一结果分支。

### 5.3 分支条件

```cpp
USTRUCT(BlueprintType)
struct DISPATCH_API FCommissionBranchConditions
{
    GENERATED_BODY()

    TArray<FName> AllowedNPCIDs;
    TArray<FName> ForbiddenNPCIDs;
    FGameplayTagContainer RequiredNPCTags;
    FGameplayTagContainer ForbiddenNPCTags;

    TArray<FCommissionPreparationRef> RequiredPreparations;
    TArray<FCommissionPreparationRef> AnyPreparations;
    TArray<FCommissionPreparationRef> ForbiddenPreparations;

    TArray<FName> RequiredClueIDs;
    FGameplayTagContainer RequiredWorldTags;
    FGameplayTagContainer ForbiddenWorldTags;

    bool bUseMinimumTrustLevel = false;
    ENPCTrustLevel MinimumTrustLevel = ENPCTrustLevel::Stranger;

    bool bUseRequiredBodyState = false;
    ENPCBodyState RequiredBodyState = ENPCBodyState::Healthy;
};
```

条件语义固定为：

- `AllowedNPCIDs`：为空表示不限制；非空时当前 NPC 必须在其中；
- `ForbiddenNPCIDs`：命中任意一个即失败；
- `RequiredNPCTags`：必须全部具备；
- `ForbiddenNPCTags`：命中任意一个即失败；
- `RequiredPreparations`：必须全部选择；
- `AnyPreparations`：为空表示不限制；非空时至少选择其中一个；
- `ForbiddenPreparations`：命中任意一个即失败；
- Required 类世界和线索条件必须全部满足；
- Forbidden 类世界条件命中任意一个即失败。

### 5.4 结果效果

```cpp
UENUM(BlueprintType)
enum class ECommissionResultGrade : uint8
{
    Excellent,
    Success,
    Partial,
    Failure
};
```

每个结果至少包含：

```text
OutcomeID
ResultGrade
ResultTitle
ResultSummary
ReturnDialogueTreeID
ResultLetterID
FollowUpSceneID
GrantedItems
ConsumedItems
GrantedClueIDs
NamedResults
StorySignalTags
ResultNPCBodyState
ResultNPCCommissionState
```

其中：

- `OutcomeID` 是剧情和存档使用的稳定结果标识；
- `ResultGrade` 只用于 UI 分类和视觉表现，不代替 `OutcomeID`；
- `ReturnDialogueTreeID` 交给现有 `UDialogueSubsystem`；
- `FollowUpSceneID` 交给现有 `UNPCSceneSubsystem`；
- `NamedResults` 写入现有剧情状态，供 StateTree 条件读取；
- `StorySignalTags` 发送给现有剧情导演；
- 未配置的效果保持为空，不因空引用导致结算失败；
- 已配置但无法解析的必需引用视为数据错误。

物品数量变化使用独立的小结构 `FCommissionItemAmount`，包含 `ItemID` 和正整数 `Count`。命名结果使用 `FCommissionNamedResult`，包含 `Key` 和 `Value`。

### 5.5 结果分支

```cpp
USTRUCT(BlueprintType)
struct DISPATCH_API FCommissionBranchRule
{
    GENERATED_BODY()

    FName OutcomeID = NAME_None;
    int32 Priority = 0;
    FCommissionBranchConditions Conditions;
    FText PreviewReactionText;
    FCommissionOutcome Outcome;
};
```

`PreviewReactionText` 和最终结果放在同一条分支中，确保准备界面显示的 NPC 反应与确认后锁定的结果属于同一规则。

### 5.6 委托规则表

```cpp
USTRUCT(BlueprintType)
struct DISPATCH_API FCommissionRuleSetRow : public FTableRowBase
{
    GENERATED_BODY()

    FName QuestID = NAME_None;
    int32 MaxPreparationSlots = 3;
    TArray<ECommissionPreparationType> AllowedPreparationTypes;
    FName DefaultOutcomeID = NAME_None;
    TArray<FCommissionBranchRule> Branches;
};
```

新增 `DT_CommissionRules`。每行保存一个委托的全部派遣分支，避免再拆出独立的反应表和结果表。

示例：

```text
RowName             Quest.HeartlessDeer
QuestID             Quest.HeartlessDeer
MaxPreparationSlots 3
DefaultOutcomeID    Outcome.Deer.DefaultFailure

Branch 1
  OutcomeID         Outcome.Deer.Excellent
  Priority          300
  RequiredNPCTags   NPC.Skill.Hunting
  RequiredItems     Item.SilverArrow
  RequiredClues     Clue.Deer.FearsFire

Branch 2
  OutcomeID         Outcome.Deer.SuccessInjured
  Priority          200
  RequiredNPCTags   NPC.Skill.Hunting
  AnyPreparations   Item.BasicBow, Item.SilverArrow

Branch 3
  OutcomeID         Outcome.Deer.DefaultFailure
  Priority          0
  Conditions        Empty
```

## 6. 规则匹配与预览

### 6.1 匹配算法

1. 构建不可变评估上下文；
2. 对每条分支执行全部条件判断；
3. 收集满足条件的分支；
4. 选择 `Priority` 最高的分支；
5. 如果最高优先级有多条分支，评估失败并记录清晰错误；
6. 如果没有分支匹配，查找 `DefaultOutcomeID` 指向的无条件分支；
7. 默认分支不存在或带有条件时，整项委托数据无效。

不依赖数组顺序决定结果。

### 6.2 UI 预判反应

每次增删准备内容后，委托 UI 使用当前草稿调用同一个评估器：

- 评估成功：显示匹配分支的 `PreviewReactionText`；
- 反应文本为空：显示规则集的通用默认文本或隐藏反应区域；
- 数据错误：禁用派遣按钮并显示安全的错误提示，同时输出详细日志；
- 玩家取消界面：丢弃草稿，不保存结果、不消耗内容、不改变 NPC。

### 6.3 派遣确认

确认派遣时重新执行一次完整验证，以防 UI 打开期间 NPC 状态或玩家持有内容已变化。

只有在以下操作全部成功时才提交：

1. 委托和规则仍然有效；
2. NPC 仍可接受委托；
3. 准备内容仍然可用；
4. 分支唯一匹配；
5. 需要消耗的物品能够原子扣除；
6. 委托实例成功写入；
7. NPC 成功进入 `OnCommission`。

任一步失败都必须回滚本次提交，不留下部分扣除或错误 NPC 状态。

## 7. 运行时模型

### 7.1 草稿

```cpp
FCommissionDraft
{
    FName CommissionID;
    FName NPCID;
    TWeakObjectPtr<ANPCCharacter> TargetNPC;
    TArray<FCommissionPreparationRef> Preparations;
};
```

草稿只存在于当前 `UCommissionSubsystem` 会话中，不直接写入 NPC 组件。

### 7.2 活动实例

```cpp
UENUM(BlueprintType)
enum class ECommissionInstanceState : uint8
{
    Draft,
    Active,
    ReadyToResolve,
    Resolved,
    Cancelled
};

USTRUCT(BlueprintType)
struct DISPATCH_API FCommissionInstance
{
    GENERATED_BODY()

    FGuid InstanceID;
    FName CommissionID;
    FName NPCID;
    TArray<FCommissionPreparationRef> Preparations;
    FName ResolvedOutcomeID;
    ECommissionInstanceState State = ECommissionInstanceState::Draft;
    bool bOutcomeEffectsApplied = false;
};
```

确认派遣后立即保存 `ResolvedOutcomeID`。委托返回时读取已保存的结果，不重新评估。

### 7.3 `UCommissionSubsystem`

新增 `UCommissionSubsystem : UGameInstanceSubsystem`，统一负责：

- 从 `CommissionID` 解析展示数据和派遣规则；
- 保存玩家当前待派发的 `PendingDispatchCommissionID`；
- 创建、更新、验证和取消草稿；
- 聚合可交付物品、线索和证据；
- 计算预判反应；
- 确认派遣并创建活动实例；
- 标记委托可完成；
- 结算并应用结果；
- 查询活动和已完成实例。

建议公开的核心接口：

```cpp
bool SetPendingDispatchCommission(FName CommissionID);
bool ResolveCommissionDefinition(FName CommissionID, FResolvedCommissionDefinition& OutDefinition) const;
bool BeginDraft(FName CommissionID, ANPCCharacter* TargetNPC);
bool SetPreparationSlot(int32 SlotIndex, const FCommissionPreparationRef& Preparation);
bool ClearPreparationSlot(int32 SlotIndex);
bool EvaluateCurrentDraft(FCommissionEvaluation& OutEvaluation) const;
bool ConfirmDispatch(FCommissionInstance& OutInstance);
void CancelDraft();
bool MarkCommissionReady(FGuid InstanceID);
bool ResolveCommission(FGuid InstanceID, FCommissionOutcome& OutOutcome);
```

`UCommissionSubsystem` 使用 `UGameInstanceSubsystem`，因为待派发选择、活动实例和结果需要跨地图存在。

### 7.4 `UNPCCommissionComponent`

组件职责缩小为当前 NPC 的派遣能力和占用状态：

- 保存 `CommissionTags`；
- 保存当前活动实例 ID 和必要实例快照；
- 判断 NPC 是否可派遣；
- 进入和退出 `Available / OnCommission / Resting / Injured`；
- 不负责读取 DataTable；
- 不负责选择结果；
- 不直接驱动 UI。

现有 `CurrentCommissionID`、`ActiveCommission` 和旧 `AssignCommission` 重载先保留兼容，但新链路以 `FCommissionInstance` 为准。迁移完成后再决定是否弃用旧字段，首个实现增量不删除它们。

## 8. UI 与 NPC 交互流程

### 8.1 当前委托来源

玩家在告示牌或任务页选中一个待派发委托，`UCommissionSubsystem` 保存 `PendingDispatchCommissionID`。

NPC 动态交互菜单中的“派发委托”只有在以下条件全部成立时才显示：

- 存在有效的 `PendingDispatchCommissionID`；
- 当前任务具有有效 `DispatchRuleID`；
- 当前 NPC 可接受委托；
- 委托 UI 处理器已经注册。

### 8.2 打开界面

```text
玩家与 NPC 交互
  → 动态菜单选择“派发委托”
  → UDialogueSubsystem 发出 OpenCommission
  → UDialogueUIControllerSubsystem 暂时隐藏对话 UI
  → UCommissionSubsystem.BeginDraft
  → 创建并初始化 WBP_CommissionUI
```

### 8.3 界面内容

界面固定包含：

1. 当前委托；
2. 当前 NPC；
3. 三个混合准备槽；
4. NPC 预判反应；
5. 派遣和返回。

不包含 NPC 选择列表、委托列表、成功率、属性数值或固定类型槽位。

点击空槽或已有槽时打开统一“可交付内容”列表。列表由 C++ 聚合并生成 ViewModel，Widget Blueprint 不查询背包、线索或证据系统。

### 8.4 关闭行为

- 点击返回：取消草稿并恢复 NPC 动态交互菜单；
- 派遣成功：关闭委托界面并结束本次 NPC 交互；
- 派遣失败：界面保持打开，显示可恢复错误；
- NPC 或世界上下文失效：安全关闭委托界面并清理草稿；
- 剧情强制接管：取消未提交草稿，不影响已经活动的委托实例。

### 8.5 Blueprint 边界

`WBP_CommissionUI` 和内容条目 Blueprint 只负责：

- 控件层级；
- 锚点、尺寸和响应式布局；
- 字体、图片和动画；
- 通过 `BindWidget` 提供约定名称的控件。

Blueprint Event Graph 不负责：

- 创建或拼装委托数据；
- 查询 NPC、任务、背包或线索；
- 匹配结果规则；
- 扣除内容；
- 改变 NPC 状态；
- 打开后续对话和场景。

## 9. 结果结算

委托完成由外部时间系统、剧情信号、调试命令或后续系统调用 `MarkCommissionReady`。本系统不实现完整日程。

`ResolveCommission` 必须：

1. 查找活动实例；
2. 验证状态为 `ReadyToResolve`；
3. 读取实例锁定的 `ResolvedOutcomeID`；
4. 从同一规则集解析结果效果；
5. 原子应用物品、线索、剧情结果、信号和 NPC 状态；
6. 将 `bOutcomeEffectsApplied` 设为 true；
7. 将状态改为 `Resolved`；
8. 通知 UI、对话或剧情系统显示结果。

重复调用同一实例的结算接口不得重复发奖、重复扣物品或重复发送剧情信号。

返回对话和后续场景属于结算后的表现：

- 有 `ReturnDialogueTreeID` 时，通过带精确 NPC 上下文的对话接口播放；
- 有 `FollowUpSceneID` 时，通过 `UNPCSceneSubsystem` 启动；
- 两者同时存在时，默认先播放返回对话，结束后再启动后续场景；
- 表现启动失败必须记录日志，但已经成功应用的核心结果不能重复应用。

## 10. StateTree 边界

StateTree 不保存委托规则。后续只需要轻量节点：

```text
Story: Mark Commission Ready
Story: Resolve Commission
Story: Wait For Commission Result
```

节点只填写 `CommissionID` 或从上下文读取活动实例，结果分支通过 `OutcomeID`、Named Result 或 Story Signal 进入后续剧情。

这保证策划不会在 StateTree 中重复填写 NPC、准备条件和结果效果。

## 11. 数据验证和日志

加载或首次使用规则时验证：

- `QuestID`、`DispatchRuleID` 和规则行存在；
- `MaxPreparationSlots` 首版必须为 3；
- `DefaultOutcomeID` 非空且存在；
- 默认分支没有条件；
- 同一规则集中 `OutcomeID` 唯一；
- 所有准备引用合法；
- 配置的 NPC、对话、场景、线索和物品引用可解析；
- 必须效果不存在非法数量；
- 同优先级规则不会在测试样例中产生歧义。

运行时错误日志至少包含：

```text
CommissionID
RuleSetID
NPCID
InstanceID
OutcomeID
失败阶段
具体无效引用或冲突规则
```

无效数据不能静默选择数组第一项，也不能继续派遣或结算。

## 12. 兼容与迁移

### 12.1 保留现有系统

- 保留 `FQuestContentRow` 和现有展示数据；
- 激活已有 `DispatchRuleID`，不创建第二个重复连接字段；
- 保留现有报告撰写表和 `FQuestDefinition`；
- 保留 `UDialogueSubsystem`、`UNPCSceneSubsystem` 和 StateTree 旧节点；
- 保留 `UNPCCommissionComponent` 旧接口供现有 Blueprint 和测试过渡；
- 不删除现有 `WBP_CommissionUI` 资产，而是修改其 C++ 父类职责；
- 旧 `WBP_CommissionQuestListItem` 暂时保留但新界面不使用。

### 12.2 减少重复信息

`FNoticeQuest` 和 `FQuestInfo` 的重复展示字段不在第一增量中立即删除。新接口应优先传递 ID，并通过 `UQuestContentSubsystem` 生成解析结果。

后续迁移完成后：

- Notice Board 槽位仅保存 `QuestID + RuntimeState`；
- NPC 活动委托保存 `FCommissionInstance`；
- UI 使用 `FResolvedCommissionDefinition`；
- 旧快照结构只作为兼容视图，不再成为权威数据源。

## 13. 自动化测试要求

### 数据和解析

- 正确读取 `FCommissionRuleSetRow`；
- 无效 `CommissionID` 失败；
- 缺少或无效 `DispatchRuleID` 失败；
- 规则行 `QuestID` 不匹配失败；
- 缺少默认结果失败；
- 有条件的默认结果失败；
- 重复 `OutcomeID` 失败；
- 无效准备、NPC、对话、场景、物品或线索引用输出清晰错误。

### 规则评估

- 无准备内容匹配默认结果；
- Required / Any / Forbidden 准备语义正确；
- 精确 NPC 条件正确；
- NPC 标签条件正确；
- 信任和身体状态条件正确；
- 世界标签和线索条件正确；
- 高优先级分支覆盖低优先级分支；
- 同一最高优先级多分支时安全失败；
- 规则结果与数组顺序无关；
- 预判反应来自最终锁定的同一分支。

### 草稿与派遣

- 三种内容可以混合放入三个槽；
- 空槽允许派遣；
- 重复内容被拒绝；
- 非法类型被拒绝；
- 取消草稿不消耗内容、不改变 NPC；
- 确认前内容变化触发重新验证；
- 扣除失败时完整回滚；
- NPC 不可用时确认失败；
- 同一 NPC 不能同时接受第二项委托；
- 派遣成功后保存准备快照和 `ResolvedOutcomeID`。

### 结果与结算

- 派遣后改变世界状态不会改变已锁定结果；
- 非 Ready 状态不能结算；
- 正确应用物品、线索、Named Result、Story Signal 和 NPC 状态；
- 同一实例重复结算不会重复应用效果；
- 返回对话使用正确 NPC 上下文；
- 后续 NPC 场景使用正确 `SceneID`；
- 表现启动失败不会导致核心结果重复结算。

### UI 和交互

- 没有待派发委托时不显示 `OpenCommission`；
- NPC 不可用时不显示 `OpenCommission`；
- 打开委托页隐藏对话页；
- 返回后恢复当前 NPC 的动态交互菜单；
- 三个准备槽变化刷新反应文本；
- 数据错误禁用派遣；
- 派遣成功结束当前交互；
- Blueprint 不需要业务事件图即可完成完整流程。

## 14. 分阶段交付

### 增量一：数据基础与评估器

- 新增委托数据类型；
- 新增 `DT_CommissionRules` 设置入口；
- 激活 `DispatchRuleID`；
- 实现解析、验证和纯规则评估；
- 使用自动化测试覆盖确定性分支。

该增量不修改委托 UI，独立可测试。

### 增量二：运行时草稿与活动实例

- 新增 `UCommissionSubsystem`；
- 新增草稿和实例；
- 扩展 `UNPCCommissionComponent`；
- 实现原子确认、回滚和结果锁定；
- 保留旧接口兼容。

### 增量三：统一委托 UI

- 重构 `UCommissionUIBase`；
- 新增准备槽、统一内容条目和 C++ ViewModel；
- 接入动态 `OpenCommission`；
- 完成返回、失败和派遣成功流程；
- Blueprint 仅保留布局。

### 增量四：结果结算与剧情连接

- 应用结果效果；
- 接入 Named Result、Story Signal、返回对话和 NPC 场景；
- 保证幂等结算；
- 增加轻量 StateTree 节点或接口。

### 增量五：Demo 委托数据

以一个委托完成端到端验证：

- 至少三个结果；
- 至少两个不同 NPC 条件；
- 至少三种准备组合；
- NPC 预判反应能提示准备差异；
- 返回结果对 NPC 状态和剧情产生可观察影响。

该委托跑通后再批量制作其余数据。

## 15. 编辑器手动工作

代码完成后需手动：

- 创建 `DT_CommissionRules`；
- 在 Project Settings → Dispatch Content 配置 Commission Rules Table；
- 给可派遣任务填写 `DispatchRuleID`；
- 给 NPC 配置 `CommissionTags`；
- 配置委托分支、预判反应和结果引用；
- 调整 `WBP_CommissionUI` 布局并绑定约定控件；
- 创建统一准备内容条目 Widget Blueprint；
- 为 Demo 委托配置至少三个结果；
- 为返回结果准备相应对话或 NPC 场景。

## 16. 验收标准

系统完成时必须满足：

1. 策划从一个 `QuestID` 能找到该委托全部派遣规则和结果；
2. 委托 UI 不保存或决定业务规则；
3. NPC Blueprint 不创建委托 UI、不匹配结果、不手工传递结果数据；
4. 同一委托选择不同 NPC 或准备组合会得到确定且不同的结果；
5. UI 显示的 NPC 预判反应与确认后锁定分支一致；
6. StateTree 不重复配置委托条件和效果；
7. 无效数据安全失败并输出可定位日志；
8. 结算只能应用一次；
9. 旧 NPC、对话、任务和报告链路在迁移期间保持兼容；
10. DispatchEditor 编译通过，Commission、NPC、Dialogue、Quest 和 StoryFlow 相关自动化测试通过。
