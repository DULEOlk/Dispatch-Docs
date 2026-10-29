# Dispatch Narrative Runtime Foundation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 建立统一剧情系统的纯运行时基础，并用第一天双分支测试证明它可以表达“玩家行为 → 剧情事实 → 条件判断 → 后续内容”。

**Architecture:** 内容资产只承载内容，`UStoryEpisodeDefinition` 和 `UStoryCampaignDefinition` 承载因果结构，`UNarrativeRuntimeSubsystem` 解释结构并维护统一事实账本。运行时通过异步动作请求与现有信件、对话、线索、委托、撰写系统隔离；这些适配器在下一阶段接入。

**Tech Stack:** Unreal Engine 5.6、C++、`UPrimaryDataAsset`、`UGameInstanceSubsystem`、Gameplay Tags、Unreal Automation Tests。

**Spec:** [统一剧情系统设计](C:/Users/DULEOLK/.codex/visualizations/2026/08/06/019fd933-cca8-7a52-aeaa-ae3639ae03f7/2026-08-23-dispatch-narrative-authoring-system-design.md)

---

## 全局约束

- 跨资产引用使用直接对象引用或软对象引用；`FName` 只用于一个资产内部的节点、入口和子结果 ID。
- 本计划不改变现有信件、对话、线索、委托、撰写、UMG、StateTree 或关卡行为。
- 本计划不删除任何遗留类型、DataTable、蓝图或资产。
- 核心运行时不得依赖 World、Actor、Widget 或编辑器模块，确保自动化测试可独立运行。
- 转移选择必须确定性：优先级从高到低；优先级相同按数组顺序；最多允许一个默认转移。
- 一次性节点和动作结果必须幂等，重复回调不能重复写入剧情事实。
- 新增测试统一放在 `Dispatch.Narrative` 测试分类下。
- 设计与计划文件保留在仓库外的本地目录；仓库中只提交源码和测试。
- 不覆盖用户已有修改，不批量保存或改写 `.uasset`。

## 文件地图

运行时源码：

- `Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h`
- `Source/Dispatch/Public/Story/Narrative/NarrativeFactLedger.h`
- `Source/Dispatch/Private/Story/Narrative/NarrativeFactLedger.cpp`
- `Source/Dispatch/Public/Story/Narrative/NarrativeRuleEvaluator.h`
- `Source/Dispatch/Private/Story/Narrative/NarrativeRuleEvaluator.cpp`
- `Source/Dispatch/Public/Story/Narrative/StoryEpisodeDefinition.h`
- `Source/Dispatch/Private/Story/Narrative/StoryEpisodeDefinition.cpp`
- `Source/Dispatch/Public/Story/Narrative/StoryCampaignDefinition.h`
- `Source/Dispatch/Private/Story/Narrative/StoryCampaignDefinition.cpp`
- `Source/Dispatch/Public/Story/Narrative/NarrativeRuntimeSubsystem.h`
- `Source/Dispatch/Private/Story/Narrative/NarrativeRuntimeSubsystem.cpp`

测试源码：

- `Source/Dispatch/Private/Tests/NarrativeFactLedgerTests.cpp`
- `Source/Dispatch/Private/Tests/NarrativeRuleEvaluatorTests.cpp`
- `Source/Dispatch/Private/Tests/NarrativeDefinitionTests.cpp`
- `Source/Dispatch/Private/Tests/NarrativeRuntimeTests.cpp`
- `Source/Dispatch/Private/Tests/NarrativeDay01BranchTests.cpp`

## Task 1：统一剧情事实与事实账本

**Files:**

- Create: `Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h`
- Create: `Source/Dispatch/Public/Story/Narrative/NarrativeFactLedger.h`
- Create: `Source/Dispatch/Private/Story/Narrative/NarrativeFactLedger.cpp`
- Test: `Source/Dispatch/Private/Tests/NarrativeFactLedgerTests.cpp`

- [ ] **Step 1: 先写失败测试**

覆盖以下行为：

1. 相同事实键第一次写入后可以读取最新值。
2. 同一键写入完全相同的值返回 `false`，历史记录不增加。
3. 同一键写入不同值返回 `true`，最新值更新，历史记录保留前后两个值。
4. `SourceAsset`、`SubjectAsset` 或 `SubID` 不同的事实不能互相覆盖。
5. `Reset()` 清空最新值、历史和序号。

- [ ] **Step 2: 编译测试并确认它因类型不存在而失败**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

预期：编译失败，错误仅指向尚未创建的 Narrative 类型。

- [ ] **Step 3: 实现最小事实类型**

在 `NarrativeTypes.h` 中定义：

```cpp
UENUM(BlueprintType)
enum class EStoryFactType : uint8
{
    LetterState,
    DialogueChoice,
    DialogueResult,
    ClueCollected,
    LeadUnlocked,
    CommissionState,
    CommissionOutcome,
    ItemDelivered,
    NPCState,
    NPCTrust,
    WritingOption,
    WritingCredibility,
    StoryTag,
    StoryValue,
    NodeCompleted
};

UENUM(BlueprintType)
enum class EStoryFactValueType : uint8
{
    None,
    Boolean,
    Name,
    Integer
};
```

实现以下结构：

```cpp
USTRUCT(BlueprintType)
struct DISPATCH_API FStoryFactKey
{
    GENERATED_BODY()

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    EStoryFactType Type = EStoryFactType::StoryTag;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSoftObjectPtr<UObject> SourceAsset;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TSoftObjectPtr<UObject> SubjectAsset;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FName SubID = NAME_None;

    bool IsValid() const;
    bool operator==(const FStoryFactKey& Other) const;
    friend uint32 GetTypeHash(const FStoryFactKey& Key);
};
```

`FStoryFactValue` 包含 `ValueType`、`BoolValue`、`NameValue`、`IntegerValue`，提供 `MakeBool`、`MakeName`、`MakeInteger`、相等比较。`FStoryFactRecord` 包含 `Key`、`Value`、`DayNumber`、`SourceEpisodeID`、`SourceNodeID`、`Sequence`。

- [ ] **Step 4: 实现账本接口**

```cpp
struct DISPATCH_API FNarrativeFactLedger
{
    bool RecordFact(const FStoryFactRecord& Record);
    bool TryGetLatestFact(const FStoryFactKey& Key, FStoryFactRecord& OutRecord) const;
    bool HasFact(const FStoryFactKey& Key) const;
    const TArray<FStoryFactRecord>& GetHistory() const;
    void Reset();
};
```

使用 `TMap<FStoryFactKey, FStoryFactRecord>` 保存最新值，使用 `TArray<FStoryFactRecord>` 保存有变化的历史；账本内部生成单调递增 `Sequence`，不相信调用方传入的序号。

- [ ] **Step 5: 运行针对性测试**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative.Facts;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

预期：`Dispatch.Narrative.Facts` 全部通过。

- [ ] **Step 6: 提交本任务**

```powershell
git add Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h Source/Dispatch/Public/Story/Narrative/NarrativeFactLedger.h Source/Dispatch/Private/Story/Narrative/NarrativeFactLedger.cpp Source/Dispatch/Private/Tests/NarrativeFactLedgerTests.cpp
git commit -m "feat: add narrative fact ledger"
```

## Task 2：条件、效果和确定性转移选择

**Files:**

- Modify: `Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h`
- Create: `Source/Dispatch/Public/Story/Narrative/NarrativeRuleEvaluator.h`
- Create: `Source/Dispatch/Private/Story/Narrative/NarrativeRuleEvaluator.cpp`
- Test: `Source/Dispatch/Private/Tests/NarrativeRuleEvaluatorTests.cpp`

- [ ] **Step 1: 写条件判断失败测试**

测试 `Exists`、`NotExists`、布尔相等、名称相等、整数相等/大于/大于等于/小于/小于等于，以及 `bInvert`。值类型不匹配必须返回不满足，不能隐式转换。

- [ ] **Step 2: 写组合与转移失败测试**

验证：

- 一个 `ConditionSet` 内所有条件必须满足。
- 多个 `ConditionSet` 是“任意一组满足”。
- 优先级高的转移胜出。
- 同优先级按数组顺序选择。
- 没有普通转移满足时选择唯一默认转移。
- 没有匹配项且没有默认转移时返回失败。

- [ ] **Step 3: 定义条件、效果和转移数据**

```cpp
UENUM(BlueprintType)
enum class ENarrativeComparison : uint8
{
    Exists,
    NotExists,
    BoolEquals,
    NameEquals,
    IntegerEquals,
    IntegerGreater,
    IntegerGreaterOrEqual,
    IntegerLess,
    IntegerLessOrEqual
};

USTRUCT(BlueprintType)
struct DISPATCH_API FNarrativeCondition
{
    GENERATED_BODY()
    UPROPERTY(EditAnywhere, BlueprintReadWrite) FStoryFactKey Key;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) ENarrativeComparison Comparison = ENarrativeComparison::Exists;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) FStoryFactValue ExpectedValue;
    UPROPERTY(EditAnywhere, BlueprintReadWrite) bool bInvert = false;
};

USTRUCT(BlueprintType)
struct DISPATCH_API FNarrativeConditionSet
{
    GENERATED_BODY()
    UPROPERTY(EditAnywhere, BlueprintReadWrite) TArray<FNarrativeCondition> AllConditions;
};
```

`ENarrativeEffectType` 只包含 `SetFact` 和 `AddIntegerFact`。`FNarrativeEffect` 包含 `EffectType`、`Key`、`Value`。`FNarrativeTransitionDefinition` 包含 `TransitionID`、`DisplayLabel`、`ConditionAlternatives`、`NextNodeID`、`Priority`、`bDefault`。

- [ ] **Step 4: 实现无状态规则解释器**

```cpp
class DISPATCH_API FNarrativeRuleEvaluator
{
public:
    static bool Matches(const FNarrativeCondition& Condition, const FNarrativeFactLedger& Ledger);
    static bool MatchesAnySet(const TArray<FNarrativeConditionSet>& Alternatives, const FNarrativeFactLedger& Ledger);
    static bool ApplyEffects(const TArray<FNarrativeEffect>& Effects, int32 DayNumber, FName EpisodeID, FName NodeID, FNarrativeFactLedger& Ledger);
    static const FNarrativeTransitionDefinition* SelectTransition(const TArray<FNarrativeTransitionDefinition>& Transitions, const FNarrativeFactLedger& Ledger);
};
```

空的 `ConditionAlternatives` 表示无条件满足；空的单个 `ConditionSet` 也表示该组满足。选择时先扫描满足的非默认转移，再使用唯一默认转移。

- [ ] **Step 5: 运行规则测试并提交**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative.Rules;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
git add Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h Source/Dispatch/Public/Story/Narrative/NarrativeRuleEvaluator.h Source/Dispatch/Private/Story/Narrative/NarrativeRuleEvaluator.cpp Source/Dispatch/Private/Tests/NarrativeRuleEvaluatorTests.cpp
git commit -m "feat: add narrative rule evaluation"
```

## Task 3：Episode 数据资产与静态校验

**Files:**

- Modify: `Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h`
- Create: `Source/Dispatch/Public/Story/Narrative/StoryEpisodeDefinition.h`
- Create: `Source/Dispatch/Private/Story/Narrative/StoryEpisodeDefinition.cpp`
- Test: `Source/Dispatch/Private/Tests/NarrativeDefinitionTests.cpp`

- [ ] **Step 1: 写 Episode 查询和校验失败测试**

创建瞬态 `UStoryEpisodeDefinition`，验证：有效资产通过；重复节点 ID、缺失默认入口、命名入口指向不存在节点、转移目标不存在、多个默认转移、非终点没有出口、完成节点缺少 Episode 出口、出口入口无效、不可达节点以及同步 `None` 节点死循环均返回明确错误。

- [ ] **Step 2: 定义剧情动作和节点**

```cpp
UENUM(BlueprintType)
enum class ENarrativeActionType : uint8
{
    None,
    PresentLetter,
    StartDialogue,
    OpenCommission,
    WaitForClue,
    OpenWriting,
    RequestScene,
    CompleteEpisode
};

USTRUCT(BlueprintType)
struct DISPATCH_API FNarrativeActionDefinition
{
    GENERATED_BODY()
    UPROPERTY(EditAnywhere, BlueprintReadOnly) ENarrativeActionType Type = ENarrativeActionType::None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TSoftObjectPtr<UObject> ContentAsset;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TSoftObjectPtr<UObject> TargetAsset;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FName ContextID = NAME_None;
};
```

`FNarrativeNodeDefinition` 包含：`NodeID`、`DisplayName`、`EntryConditionAlternatives`、`EntryEffects`、`Action`、`CompletionEffects`、`Transitions`、`FailureNodeID`、`bRunOnce`。

在 `StoryEpisodeDefinition.h` 中先前置声明 `class UStoryEpisodeDefinition;`，再定义 Episode 出口，避免把“下一天去哪”隐藏在 StateTree 或自由字符串里：

```cpp
USTRUCT(BlueprintType)
struct DISPATCH_API FNarrativeEpisodeExitDefinition
{
    GENERATED_BODY()
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FName ExitID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TArray<FNarrativeConditionSet> ConditionAlternatives;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TArray<FNarrativeEffect> Effects;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) TObjectPtr<UStoryEpisodeDefinition> NextEpisode = nullptr;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) FName NextEntryID = NAME_None;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) int32 Priority = 0;
    UPROPERTY(EditAnywhere, BlueprintReadOnly) bool bDefault = false;
};
```

`CompleteEpisode` 节点的 `Action.ContextID` 可指定一个 `ExitID`；为 `None` 时由运行时按条件和优先级选择出口。`NextEpisode == nullptr` 表示 Campaign 已结束，结局定义在后续计划接入。

- [ ] **Step 3: 实现 Episode 资产**

```cpp
UCLASS(BlueprintType)
class DISPATCH_API UStoryEpisodeDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") FName EpisodeID;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") int32 DayNumber = 0;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") FName DefaultEntryID;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") TMap<FName, FName> NamedEntries;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") TArray<FNarrativeNodeDefinition> Nodes;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") TArray<FNarrativeEpisodeExitDefinition> Exits;

    const FNarrativeNodeDefinition* FindNode(FName NodeID) const;
    FName ResolveEntryNodeID(FName EntryID) const;
    const FNarrativeEpisodeExitDefinition* SelectExit(FName RequestedExitID, const FNarrativeFactLedger& Ledger) const;
    bool ValidateDefinition(TArray<FText>& OutErrors) const;

#if WITH_EDITOR
    virtual EDataValidationResult IsDataValid(FDataValidationContext& Context) const override;
#endif
};
```

`DefaultEntryID` 直接指向入口节点。`NamedEntries` 用于后续从某个结果进入同一天或下一天的指定入口。`ResolveEntryNodeID(NAME_None)` 返回默认入口；非空值必须先作为命名入口键解析，不允许把任意节点 ID 当外部入口。

- [ ] **Step 4: 实现图级静态分析**

验证器先建立节点索引，再验证引用，最后从所有入口做可达性遍历。同步死循环只分析 `Action.Type == None` 的连接，避免把等待外部输入的合法循环误判为立即死循环。出口还要验证：`ExitID` 唯一、最多一个默认出口、`NextEpisode` 不为空时 `NextEntryID` 能被目标 Episode 解析；所有 `CompleteEpisode` 节点都能指定有效出口或拥有可选择的默认出口。

- [ ] **Step 5: 运行定义测试并提交**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative.Definitions;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
git add Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h Source/Dispatch/Public/Story/Narrative/StoryEpisodeDefinition.h Source/Dispatch/Private/Story/Narrative/StoryEpisodeDefinition.cpp Source/Dispatch/Private/Tests/NarrativeDefinitionTests.cpp
git commit -m "feat: add narrative episode definitions"
```

## Task 4：Campaign 总览资产

**Files:**

- Create: `Source/Dispatch/Public/Story/Narrative/StoryCampaignDefinition.h`
- Create: `Source/Dispatch/Private/Story/Narrative/StoryCampaignDefinition.cpp`
- Modify: `Source/Dispatch/Private/Tests/NarrativeDefinitionTests.cpp`

- [ ] **Step 1: 写 Campaign 校验失败测试**

验证有效 Campaign 能按 Episode ID 和 Day Number 查询。以下情况必须失败：起始 Episode 为空、起始入口不存在、Episode 为空引用、Episode ID 重复、非零 Day Number 重复、子 Episode 自身无效、Episode 出口指向不在本 Campaign 中的目标。

- [ ] **Step 2: 实现 Campaign 资产**

```cpp
UCLASS(BlueprintType)
class DISPATCH_API UStoryCampaignDefinition : public UPrimaryDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") FName CampaignID;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") TObjectPtr<UStoryEpisodeDefinition> StartingEpisode;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") FName StartingEntryID;
    UPROPERTY(EditAnywhere, BlueprintReadOnly, Category="Narrative") TArray<TObjectPtr<UStoryEpisodeDefinition>> Episodes;

    UStoryEpisodeDefinition* FindEpisode(FName EpisodeID) const;
    UStoryEpisodeDefinition* FindEpisodeForDay(int32 DayNumber) const;
    bool ValidateDefinition(TArray<FText>& OutErrors) const;

#if WITH_EDITOR
    virtual EDataValidationResult IsDataValid(FDataValidationContext& Context) const override;
#endif
};
```

Campaign 使用直接引用，因为它是项目的剧情总索引；Episode 内的具体内容资产仍可使用软引用。

- [ ] **Step 3: 运行定义测试并提交**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative.Definitions;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
git add Source/Dispatch/Public/Story/Narrative/StoryCampaignDefinition.h Source/Dispatch/Private/Story/Narrative/StoryCampaignDefinition.cpp Source/Dispatch/Private/Tests/NarrativeDefinitionTests.cpp
git commit -m "feat: add narrative campaign definition"
```

## Task 5：同步剧情运行时

**Files:**

- Create: `Source/Dispatch/Public/Story/Narrative/NarrativeRuntimeSubsystem.h`
- Create: `Source/Dispatch/Private/Story/Narrative/NarrativeRuntimeSubsystem.cpp`
- Create: `Source/Dispatch/Private/Tests/NarrativeRuntimeTests.cpp`

- [ ] **Step 1: 写同步运行失败测试**

覆盖以下路径：

1. `StartCampaign` 从 Campaign 指定入口启动。
2. `None` 节点依次应用进入效果和完成效果，然后按规则进入下一节点。
3. 完成节点写入唯一 `NodeCompleted` 事实。
4. `bRunOnce` 节点重复进入时不重复应用效果。
5. 无匹配转移且无失败节点时进入失败状态并返回明确错误。
6. 同步节点链超过 128 步时中止，防止坏数据卡死游戏线程。

- [ ] **Step 2: 定义运行状态和 Episode 结果**

```cpp
UENUM(BlueprintType)
enum class ENarrativeNodeRunState : uint8
{
    Idle,
    Running,
    WaitingForAction,
    EpisodeCompleted,
    Failed
};

USTRUCT(BlueprintType)
struct DISPATCH_API FNarrativeEpisodeResult
{
    GENERATED_BODY()
    UPROPERTY(BlueprintReadOnly) bool bSucceeded = false;
    UPROPERTY(BlueprintReadOnly) FName CompletedEpisodeID = NAME_None;
    UPROPERTY(BlueprintReadOnly) FName ExitID = NAME_None;
    UPROPERTY(BlueprintReadOnly) TObjectPtr<UStoryEpisodeDefinition> NextEpisode = nullptr;
    UPROPERTY(BlueprintReadOnly) FName NextEntryID = NAME_None;
};
```

- [ ] **Step 3: 实现运行时公开接口**

```cpp
UCLASS()
class DISPATCH_API UNarrativeRuntimeSubsystem : public UGameInstanceSubsystem
{
    GENERATED_BODY()

public:
    bool StartCampaign(UStoryCampaignDefinition* Campaign, FText& OutError);
    bool StartEpisode(UStoryEpisodeDefinition* Episode, FName EntryID, FText& OutError);
    bool RecordExternalFact(const FStoryFactRecord& Fact);

    ENarrativeNodeRunState GetRunState() const;
    UStoryCampaignDefinition* GetActiveCampaign() const;
    UStoryEpisodeDefinition* GetActiveEpisode() const;
    FName GetActiveNodeID() const;
    const FNarrativeFactLedger& GetFactLedger() const;

private:
    bool EnterNode(FName NodeID, FText& OutError);
    bool CompleteSynchronousNode(FText& OutError);
    bool AdvanceFromCurrentNode(FText& OutError);
    void FailRuntime(const FText& Error);
};
```

`StartCampaign` 必须先调用 Campaign 验证；失败时不得部分初始化状态。`StartEpisode` 的 `EntryID` 只能是 `NAME_None` 或 Episode 的命名入口键，并通过 `ResolveEntryNodeID` 解析。`EnterNode` 先检查节点进入条件，应用进入效果，再决定同步完成或等待动作。`None` 与 `CompleteEpisode` 是同步动作；其余动作进入 `WaitingForAction`。

处理 `CompleteEpisode` 时，根据 `Action.ContextID` 选择指定出口，或让 Episode 按条件选择出口；随后应用出口效果并生成 `FNarrativeEpisodeResult`。运行时只报告 `NextEpisode` 和 `NextEntryID`，不自动启动下一天，宏观转场交给 Plan 2 的 StoryDirector / StateTree。

- [ ] **Step 4: 实现 `NodeCompleted` 幂等键**

完成节点时写入：

- `Type = NodeCompleted`
- `SourceAsset = ActiveEpisode`
- `SubID = NodeID`
- `Value = true`

`bRunOnce` 通过该键判断，而不是另建一套不可存档状态。

- [ ] **Step 5: 运行同步运行时测试并提交**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative.Runtime.Synchronous;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
git add Source/Dispatch/Public/Story/Narrative/NarrativeRuntimeSubsystem.h Source/Dispatch/Private/Story/Narrative/NarrativeRuntimeSubsystem.cpp Source/Dispatch/Private/Tests/NarrativeRuntimeTests.cpp
git commit -m "feat: add synchronous narrative runtime"
```

## Task 6：异步动作请求与完成回调

**Files:**

- Modify: `Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h`
- Modify: `Source/Dispatch/Public/Story/Narrative/NarrativeRuntimeSubsystem.h`
- Modify: `Source/Dispatch/Private/Story/Narrative/NarrativeRuntimeSubsystem.cpp`
- Modify: `Source/Dispatch/Private/Tests/NarrativeRuntimeTests.cpp`

- [ ] **Step 1: 写异步动作失败测试**

验证：动作节点只广播一次请求并进入等待；只有完全相同的 `RequestID` 才能完成；过期、重复或错误 ID 回调返回 `false`；成功结果先写 `ProducedFacts` 再走成功转移；失败结果走 `FailureNodeID`，不能广播成功完成；监听器在广播过程中立即回调也能安全完成。

- [ ] **Step 2: 定义稳定的请求和结果结构**

```cpp
USTRUCT(BlueprintType)
struct DISPATCH_API FNarrativeActionRequest
{
    GENERATED_BODY()
    UPROPERTY(BlueprintReadOnly) FGuid RequestID;
    UPROPERTY(BlueprintReadOnly) FName EpisodeID = NAME_None;
    UPROPERTY(BlueprintReadOnly) FName NodeID = NAME_None;
    UPROPERTY(BlueprintReadOnly) FNarrativeActionDefinition Action;
};

USTRUCT(BlueprintType)
struct DISPATCH_API FNarrativeActionResult
{
    GENERATED_BODY()
    UPROPERTY(BlueprintReadWrite) FGuid RequestID;
    UPROPERTY(BlueprintReadWrite) bool bSucceeded = false;
    UPROPERTY(BlueprintReadWrite) TArray<FStoryFactRecord> ProducedFacts;
    UPROPERTY(BlueprintReadWrite) FName ErrorCode = NAME_None;
};
```

- [ ] **Step 3: 增加委托与公开方法**

声明 Blueprint 动态委托和原生多播委托。公开：

```cpp
UPROPERTY(BlueprintAssignable)
FNarrativeActionRequestedDynamic OnNarrativeActionRequested;

FNarrativeActionRequestedNative& OnNarrativeActionRequestedNative();

UFUNCTION(BlueprintCallable, Category="Narrative")
bool CompleteAction(const FNarrativeActionResult& Result, FText& OutError);

UFUNCTION(BlueprintCallable, Category="Narrative")
bool FailAction(FGuid RequestID, FName ErrorCode, FText& OutError);

UFUNCTION(BlueprintPure, Category="Narrative")
bool IsWaitingForAction() const;
```

Plan 2 的 `AStoryDirector` 必须只通过这组接口接入，不允许直接改运行时内部状态。

- [ ] **Step 4: 实现防重入广播**

在广播前完成以下状态更新：生成并保存 `ActiveRequest`、把状态设为 `WaitingForAction`。广播返回后不假设仍处于等待状态，因为监听器可能已经同步调用 `CompleteAction`。

- [ ] **Step 5: 运行异步测试并提交**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative.Runtime.Async;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
git add Source/Dispatch/Public/Story/Narrative/NarrativeTypes.h Source/Dispatch/Public/Story/Narrative/NarrativeRuntimeSubsystem.h Source/Dispatch/Private/Story/Narrative/NarrativeRuntimeSubsystem.cpp Source/Dispatch/Private/Tests/NarrativeRuntimeTests.cpp
git commit -m "feat: add narrative action requests"
```

## Task 7：第一天双分支竖切测试

**Files:**

- Create: `Source/Dispatch/Private/Tests/NarrativeDay01BranchTests.cpp`

- [ ] **Step 1: 用瞬态资产搭建 Day 1 测试剧本**

测试内创建 Campaign、Episode 和占位内容对象，不创建 `.uasset`。节点链为：

```text
Start
  → PresentLetter
  → StartDialogue
  → ChooseResponse
      ├─ 如实说明 → CommissionTruth → OpenWriting → WriteTruth → CompleteDayTruth
      └─ 隐瞒异常 → CommissionOfficial → OpenWriting → WriteOfficial → CompleteDayOfficial
```

每个异步节点由测试监听器捕获请求并返回对应结果，模拟 Plan 2 之后真实系统的行为。

- [ ] **Step 2: 定义“如实记录”路线的事实与断言**

依次产生：

- `DialogueChoice = Truth`
- `CommissionOutcome = PartialSuccess`
- `LeadUnlocked = DocumentTampered`
- `WritingOption = Truthful`
- `StoryValue(Ending.Truthful) = 1`

测试创建一个瞬态 Day 2 Episode，并把如实出口直接引用它的命名入口 `InvestigateRecords`。结算结果必须返回该 Episode 和入口。断言 `DocumentTampered` 存在、`Ending.Truthful == 1`，并且完整历史顺序可追踪到各节点。

- [ ] **Step 3: 定义“迎合官方”路线的事实与断言**

依次产生：

- `DialogueChoice = Conceal`
- `CommissionOutcome = Failure`
- `WritingOption = Official`
- `StoryValue(Ending.Official) = 1`

同一个瞬态 Day 2 Episode 还提供命名入口 `RecordKeeperMissing`。结算结果必须返回该 Episode 和入口。断言此路线不存在 `DocumentTampered`，`Ending.Official == 1`，事实历史与如实路线不同。

- [ ] **Step 4: 验证重复回调不会重复结算**

保存最后一个请求并重复提交。第二次必须返回 `false`，事实历史数量、当前节点和 Episode 结果都保持不变。

- [ ] **Step 5: 运行 Day 1 测试并提交**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative.Day01;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
git add Source/Dispatch/Private/Tests/NarrativeDay01BranchTests.cpp
git commit -m "test: prove day one narrative branches"
```

## Task 8：阶段验收与接口冻结

**Files:**

- Review only: `Source/Dispatch/Public/Story/Narrative/`
- Review only: `Source/Dispatch/Private/Story/Narrative/`
- Review only: `Source/Dispatch/Private/Tests/Narrative*.cpp`

- [ ] **Step 1: 完整编译 Editor 目标**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

预期：`DispatchEditor` 编译成功，无新增警告。

- [ ] **Step 2: 运行全部 Narrative 测试**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch.Narrative;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

预期：事实、规则、定义、同步/异步运行时和 Day 1 分支全部通过。

- [ ] **Step 3: 运行现有 Dispatch 测试集**

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' '-ExecCmds=Automation RunTests Dispatch;Quit' -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache
```

预期：现有系统没有回归。

- [ ] **Step 4: 审计改动范围**

```powershell
git status --short
git diff -- Source/Dispatch/Public/Story/Narrative Source/Dispatch/Private/Story/Narrative Source/Dispatch/Private/Tests
```

确认没有修改：

- `Content/` 下的现有 `.uasset`
- 现有 StateTree、UMG 和地图
- 现有信件、对话、线索、委托、撰写运行时代码
- 仓库内的 `docs/`

- [ ] **Step 5: 冻结 Plan 2 所需接口**

在代码评审中逐项确认：

1. 事实键能唯一表达资产、对象和子结果。
2. 条件/效果足以表达 Day 1 两条路线，不依赖系统特例。
3. Episode / Campaign 验证错误能定位到具体资产、节点和转移。
4. `FNarrativeActionRequest` / `FNarrativeActionResult` 可由 `AStoryDirector` 使用，不暴露内部状态。
5. 运行时可以由事实历史重建，不存在隐藏的不可存档剧情结果。
6. 现有系统尚未接入，当前 Demo 行为没有变化。

- [ ] **Step 6: 如审计产生修正，单独提交修正**

只在确有改动时提交；不要创建空提交。

```powershell
git add Source/Dispatch/Public/Story/Narrative Source/Dispatch/Private/Story/Narrative Source/Dispatch/Private/Tests
git commit -m "fix: harden narrative runtime contracts"
```

## Plan 1 完成定义

本计划只有在以下条件全部满足时才算完成：

- `DispatchEditor` 编译成功。
- `Dispatch.Narrative` 全部通过。
- 现有 `Dispatch` 测试没有回归。
- Day 1 两条路线产生不同事实、不同下一入口和不同结局倾向。
- 重复动作结果不会重复推进或重复记账。
- 没有修改现有内容资产、UI、StateTree 或旧系统行为。
- Plan 2 可以只通过动作请求/结果接口接入现有系统。
