# 统一 NPC 对话与动态交互 UI 设计

## 目标

建立一条完全由剧情数据和 C++ 驱动的 NPC 对话链路：剧情决定 NPC 何时主动谈话、玩家当前能询问什么以及对话如何分支；NPC 蓝图不再创建 Widget、拼接固定按钮或向委托 UI 手工传值。Widget Blueprint 仅负责布局和美术，不包含业务事件图逻辑。

本设计与轻量 NPC 剧情场景系统衔接：`StateTree / UNPCSceneSubsystem` 负责 NPC 场景和主动对话，玩家按 E 时由统一对话系统读取当前场景的交互集合。

## 已确认的产品规则

- NPC 主动对话是剧情行为，由 StateTree、触发器或 NPC Scene 的 `EntryDialogueTreeID` 启动；NPC 不再通过视野球和自由判断触发剧情谈话。
- NPC 主动剧情对话自然结束后关闭对话 UI，不自动追加“询问 / 委托 / 离开”。
- 玩家主动按 E 时先显示当前剧情阶段配置的动态交互菜单，不先播放固定问候。
- 玩家选择询问项后播放对应对话；询问结束默认返回交互菜单，选择“离开”才关闭整个交互会话。
- “交互菜单选项”和 `DT_NPCDialogue` 中的“剧情分支选项”是两类数据，共用显示组件但不能混为同一概念。
- 选项数量和文字均不固定；系统只显示当前配置且满足条件的选项。
- 委托 UI 尚未定稿。系统保留 `OpenCommission` 动作接口，但没有有效处理器或配置时不显示该入口。
- 所有运行时逻辑使用 C++。允许 `WBP_NPCDialogue` 和选项条目 WBP 存在，但它们只摆放 C++ `BindWidget` 所需控件并设置美术资源。

## 非目标

- 不制作完整对话图可视化编辑器。
- 不制作语音、口型、镜头切换或 Sequencer 对话系统。
- 不在本任务中完成委托 UI 的最终信息结构和派遣流程。
- 不让 NPC 使用 AI 自由生成或自主选择剧情对话。
- 不重做 NPC Scene 的移动、锚点和 Routine 主体逻辑。

## 现状与迁移边界

项目当前有两条并存链路：

1. 新链路 `UDialogueSubsystem + DT_NPCDialogue`，能够读取 `LineNode / ChoiceNode / ClueNode / ExitNode`，但缺少统一 UI 表现层。
2. 旧链路由 `ANPCCharacter` 和 NPC 蓝图调用 `OnNPCSubtitleRequested`、`OnShowManualChoices`、`OnShowStoryOfferChoices`、`OnRequestCommissionUI` 等事件，手工创建 `WBP_NPCSubtitle`、固定三按钮选项和委托 UI。

本任务以新链路为唯一运行入口。旧蓝图事件先标记弃用，在所有现有 NPC 内容迁移并验证后删除。旧 WBP 暂时保留用于视觉对照，不继续扩展其逻辑。

## 总体架构

```text
StateTree / NPC Scene / 玩家按 E
                ↓
       UDialogueSubsystem
      （会话、节点、选项）
                ↓
UDialogueUIControllerSubsystem
 （本地玩家 UI 和输入生命周期）
                ↓
 WBP_NPCDialogue（纯布局）
                ↓
WBP_DialogueOptionEntry（纯布局）
```

数据职责：

```text
DT_NPCScenes       NPC 在哪、做什么、何时主动谈话、当前 InteractionSetID
DT_NPCInteractions 玩家主动交互时有哪些功能/询问入口
DT_NPCDialogue     具体台词、剧情条件、剧情分支和退出节点
```

采用独立 `DT_NPCInteractions`，而不是把菜单数组塞进场景行或伪装成 Dialogue `ChoiceNode`。这样移动/场景、功能菜单和叙事分支保持独立。

## 交互菜单数据

新增 `NPC/NPCInteractionTypes.h`。

### 动作类型

```cpp
UENUM(BlueprintType)
enum class ENPCInteractionActionType : uint8
{
    StartDialogue,
    OpenNPCInfo,
    OpenCommission,
    Exit
};
```

首版不增加通用字符串命令或任意 Blueprint 回调，避免把类型安全的菜单重新变成松散事件系统。

### 单个交互选项

```cpp
USTRUCT(BlueprintType)
struct FNPCInteractionOption
{
    GENERATED_BODY()

    FName OptionID;
    FText OptionText;
    ENPCInteractionActionType ActionType;
    FName DialogueTreeID;
    TArray<FDialogueCondition> Conditions;
    bool bReturnToMenu = true;
};
```

规则：

- `OptionID` 在同一 Interaction Set 内唯一，作为稳定选择键，不使用过滤后的数组下标。
- `OptionText` 是实际显示文字，例如“询问昨晚的袭击”，不是固定类别名。
- `DialogueTreeID` 仅在 `StartDialogue` 时必填。
- `Conditions` 全部满足时才显示该项。
- `bReturnToMenu` 仅影响从该项启动的玩家询问对话；剧情主动对话不读取它。
- `Exit` 始终可以显式配置。若一组有效选项中没有 `Exit`，运行时自动追加本地化的默认“离开”，防止玩家无法关闭会话。

### 交互集合行

```cpp
USTRUCT(BlueprintType)
struct FNPCInteractionSetRow : public FTableRowBase
{
    GENERATED_BODY()

    TArray<FNPCInteractionOption> Options;
};
```

DataTable 行名即 `InteractionSetID`，建议命名：

```text
Interaction.Innkeeper.Day01.AfterQuest
Interaction.Guard.Day01.AtGate
```

## NPC Scene 与 Dialogue Component 连接

`FNPCSceneRow` 新增：

```cpp
FName InteractionSetID;
```

场景示例：

```text
RowName                Scene.Innkeeper.Day01.AfterQuest
EntryDialogueTreeID    Dialogue.Innkeeper.Day01.Report
InteractionSetID       Interaction.Innkeeper.Day01.AfterQuest
```

`UNPCDialogueComponent` 新增：

- `DefaultInteractionSetID`
- 瞬态 `SceneInteractionSetID`
- `GetInteractionSetID()`
- `SetSceneInteractionSetID()`
- `ClearSceneInteractionSetID()`

解析顺序：当前场景覆盖值优先，其次 NPC 默认集合，否则没有玩家交互菜单。

现有 `InteractionDialogueTreeID` 在第一轮迁移中保留并标记弃用。如果 `InteractionSetID` 为空而旧字段有效，运行时生成一个仅含该询问和“离开”的兼容菜单，并输出一次迁移提示日志。全部内容迁移后再单独删除旧字段。`StoryDialogueTreeID` 和 `DailyDialogueTreeID` 继续作为旧内容兼容数据，本任务不直接删除。

`UNPCSceneSubsystem` 在场景到达后设置 `SceneInteractionSetID`，停止或恢复场景时恢复之前的集合值。Entry 对话明确以 Story 会话模式启动。

## 对话会话模型

`UDialogueSubsystem` 是唯一对话状态所有者，新增：

```cpp
enum class EDialogueSessionMode : uint8
{
    Story,
    PlayerInteraction
};
```

核心公开接口：

```cpp
bool StartStoryDialogueForNPC(ANPCCharacter* NPC, FName DialogueTreeID);
bool StartNPCInteraction(ANPCCharacter* NPC);
bool ContinueDialogue();
bool SelectDialogueChoice(FName ChoiceID);
bool SelectInteractionOption(FName OptionID);
void CloseDialogueSession();
```

原有 `StartDialogue` 和 `StartDialogueForNPC` 保留为兼容入口；StateTree/NPC Scene 迁移到明确的 Story 接口，玩家按 E 迁移到 `StartNPCInteraction`。

会话记录至少包含：

- 模式、目标 NPC 和当前根对话 ID；
- 当前节点 ID；
- 当前 Interaction Set ID；
- 当前可见剧情选项与交互选项；
- 玩家询问结束后是否返回菜单；
- NPC 行为/Routine 恢复快照和重入代次。

任意时刻只允许一个本地对话会话。新 Story 会话具有最高优先级，会关闭当前玩家交互菜单或询问对话后接管 UI。无效 ID、NPC 销毁或表格缺失必须安全关闭/回退并记录包含 ID 的日志。

## 节点推进语义

当前启动代码会在根 `LineNode` 有 `NextNodeID` 时立即广播下一节点。新 UI 下必须移除该自动跳转。

- `LineNode`：显示当前台词并等待 `ContinueDialogue()`。
- `ChoiceNode`：隐藏继续提示，显示满足条件的动态剧情选项。
- `ClueNode`：显示台词和线索通知，玩家确认后继续。
- `ExitNode`：发出退出信号并结束当前对话。

从 Story 会话到达退出节点后关闭 UI。从 PlayerInteraction 启动的询问对话到达退出节点后，根据该交互项的 `bReturnToMenu` 返回菜单或关闭会话。

## 剧情 Choice 数据

扩展 `FDialogueChoiceOption`：

```cpp
FName ChoiceID;
FText ChoiceText;
FName NextNodeID;
TArray<FDialogueCondition> Conditions;
```

`ChoiceID` 在当前 `ChoiceNode` 内唯一。`GetAvailableChoices` 只返回满足条件的选项；选择时使用 `ChoiceID` 查回原始选项，避免过滤后数组下标错位。空 ID、重复 ID、缺失 `NextNodeID` 或找不到目标节点均拒绝选择并输出节点/选项 ID。

节点级 `EntryConditions` 继续决定节点是否可进入，Choice 的 `Conditions` 只决定单个选项是否可见。首版不实现自动寻找条件不满足节点的替代分支；数据错误会明确结束/失败，而不是静默跳过到未知节点。

## 动态交互选项执行

`StartNPCInteraction` 按以下顺序执行：

1. 验证 NPC 和 `InteractionSetID`。
2. 查询 `DT_NPCInteractions`。
3. 使用当前 NPC 上下文评估每个选项条件。
4. 校验动作所需数据和处理器。
5. 过滤不可用项，必要时追加 Exit。
6. 进入 `PlayerInteraction` 模式并广播菜单 ViewData。

动作行为：

- `StartDialogue`：启动对应树；结束后按 `bReturnToMenu` 处理。
- `OpenNPCInfo`：向 UI 控制器发出强类型特殊页面请求；关闭页面后返回当前交互菜单。
- `OpenCommission`：只有 NPC 可用且委托页面处理器已注册时才可见；首版不规定其最终页面内容。
- `Exit`：关闭会话并恢复玩家输入、NPC 行为和场景 Routine。

交互项条件复用 `FDialogueCondition`。涉及 `CommissionState`、`TrustLevel` 等条件时，评估对象必须是当前会话的精确 NPC，不能读取任意全局 NPC。

## C++ UI 控制层

新增 `UDialogueUIControllerSubsystem : ULocalPlayerSubsystem`，因为 Widget、输入模式和焦点属于本地玩家而不是全局 GameInstance。

职责：

- 监听 `UDialogueSubsystem` 的强类型状态事件；
- 延迟加载并只创建一个统一对话 Widget；
- 在 Line、DialogueChoices、InteractionMenu 三种表现模式间切换；
- 把选项 ViewModel 提供给 ListView；
- 将继续、选中、返回和关闭操作转发给 DialogueSubsystem；
- 管理鼠标、键盘/手柄焦点及 `APlayerCharacter::Enter/ExitNPCDialogueMode`；
- 会话结束、地图切换、本地玩家销毁时清理 Widget 和绑定。

UI 控制器不包含剧情规则，DialogueSubsystem 不引用具体 WBP。两者只通过 ViewData 和操作接口通信。

## C++ Widget 基类与纯布局 WBP

新增：

- `UDialogueScreenWidget : UUserWidget`
- `UDialogueOptionEntryWidget : UUserWidget, IUserObjectListEntry`
- `UDialogueOptionViewModel : UObject`

`UDialogueScreenWidget` 通过 `BindWidget` 要求：

```text
Text_SpeakerName
Text_Dialogue
List_Options
Panel_Continue
```

它提供 C++ 方法：

```cpp
ShowLine(...);
ShowDialogueChoices(...);
ShowInteractionMenu(...);
HideDialogue();
```

`WBP_NPCDialogue` 继承该类，只摆放命名控件、字体、背景、动画资源和导航样式，事件图必须为空。`WBP_DialogueOptionEntry` 同样只摆放按钮/文本；点击、选择和 ListView 条目刷新由 C++ 基类完成。

旧 `WBP_NPCSubtitle`、`WBP_NPCChoiceOptions` 不再作为运行入口。视觉确认完成前不删除资源，避免用户现有美术布局不可恢复。

## 项目设置

`UDispatchContentSettings` 新增：

```cpp
TSoftObjectPtr<UDataTable> NPCInteractionTable;
TSoftClassPtr<UDialogueScreenWidget> DialogueWidgetClass;
```

编辑器配置：

```text
Project Settings
→ Dispatch Content
→ NPC Interaction Table = DT_NPCInteractions
→ Dialogue Widget Class = WBP_NPCDialogue
```

选项 Entry 类由 `WBP_NPCDialogue` 的 ListView Designer 属性设置为 `WBP_DialogueOptionEntry`；它是布局依赖，不需要 NPC 蓝图传入。

## NPC Character 简化

`ANPCCharacter::InteractWithPlayer` 只调用 `StartNPCInteraction(this)`。NPC 不再创建 Widget 或向 Widget 传值。

完成迁移后删除或弃用以下旧职责：

- `OnOpenDialogue`
- `OnShowManualChoices`
- `OnShowStoryOfferChoices / OnHideStoryOfferChoices`
- `OnNPCSubtitleRequested`
- `OnRequestNPCInfo`
- `OnRequestCommissionUI`
- `CreateNPCDialogueUI` 等 NPC 蓝图事件图实现

NPC 蓝图只保留 Mesh、AnimBP、碰撞/音效和美术参数；Routine 动画响应仍属于表现层，不属于对话 UI 业务。

## 移除旧主动接近对话职责

旧 `VisionSphere / ENPCProximityTalkState / TryNoticePlayer / BeginApproachPlayer / StartProactiveStoryOffer` 不再触发剧情对话。主动剧情统一由 StateTree/NPC Scene 调度。

为降低迁移风险，第一阶段先禁用入口并将测试迁移到 Story 流；确认关卡不再引用后，第二阶段删除状态机字段、函数和蓝图事件。`InteractionBox` 保留用于玩家按 E。

如果未来剧情要求 NPC 走近玩家再说话，应由显式 Story 流完成移动和对话，不重新引入自主视野触发。

## 生命周期与异常处理

- Story 对话开始会抢占玩家交互会话；旧会话先恢复输入/NPC 快照，再建立新会话。
- NPC Scene 替换或停止时只结束该场景实际拥有的 Story 对话，不误关同 NPC 的新会话。
- NPC 被销毁、地图切换、玩家控制器失效或 Widget 类加载失败时关闭会话并恢复输入。
- Dialogue/Interaction DataTable 未配置、行结构错误、ID 缺失或重复时输出明确日志，不显示空菜单。
- 如果所有配置项均被条件过滤，系统仍提供 Exit；如果 Widget 创建失败，直接关闭会话，不能把玩家留在输入锁定状态。
- 委托/人物信息特殊页面失败时返回交互菜单；不存在有效交互上下文时关闭会话。
- 结束事件允许监听者启动下一会话，旧会话清理通过代次令牌保证不会清掉新上下文。

## 自动化测试

至少覆盖：

- `FNPCInteractionSetRow` 和配置加载；
- 无效/重复 Interaction Set、OptionID 和动作参数；
- 不同 NPC Scene 解析不同 InteractionSetID；
- NPC 主动 Story 对话结束后关闭而不显示通用菜单；
- 玩家按 E 直接显示动态交互菜单；
- 0、1、多个选项及自动 Exit；
- 条件不满足的交互项和剧情 Choice 被过滤；
- Choice 过滤后通过稳定 ChoiceID 跳转正确节点；
- LineNode 不再启动后立即跳过，Continue 后才推进；
- 玩家询问结束返回菜单，`bReturnToMenu=false` 时关闭；
- OpenNPCInfo 页面返回菜单；
- 未注册委托处理器时不显示 OpenCommission；
- Exit、NPC 销毁、地图切换和 UI 创建失败均恢复玩家输入/NPC 状态；
- Story 抢占玩家交互以及结束回调重入安全；
- UI 控制器只创建一个 Widget，并正确映射三种表现模式；
- 旧 NPC 蓝图事件不再是新运行链路的必要条件；
- 现有 NPC Scene、Dialogue、Story.Flow 自动化套件无新增回归。

## 实施与内容迁移顺序

1. 增加 Interaction 数据结构、设置入口和失败测试。
2. 重构 DialogueSubsystem 会话模式、稳定 ID 选择和逐行推进。
3. 增加 LocalPlayer UI 控制器、C++ Widget 基类和 ViewModel。
4. 创建两个纯布局 WBP 并配置项目设置。
5. NPC Scene 与 DialogueComponent 接入 `InteractionSetID` 和旧字段兼容。
6. `ANPCCharacter::InteractWithPlayer` 切换到统一入口；禁用旧主动接近对话入口。
7. 创建 `DT_NPCInteractions`，迁移现有场景和 Dialogue Choice 数据。
8. 清空现有 NPC 蓝图中的 UI 业务图，验证所有 NPC 共用统一界面。
9. 完整编译、自动化和关卡手测后，再删除弃用 C++ 事件与旧固定选项 Widget。

## 编辑器手动工作

代码完成后仍需用户在编辑器中：

- 创建 `DT_NPCInteractions` 并配置交互集合；
- 创建 `WBP_NPCDialogue` 和 `WBP_DialogueOptionEntry` 的纯布局；
- 在项目设置中配置 Interaction Table 和 Dialogue Widget Class；
- 为现有 NPC Scene 填写 `InteractionSetID`；
- 为 Dialogue Choice 补充唯一 ChoiceID 和可选条件；
- 迁移并最终清理 NPC 蓝图旧 UI 事件图；
- 根据后续委托 UI 设计补充 OpenCommission 的页面处理器。

## 完成标准

- NPC 主动对话和玩家主动交互在产品行为上明确分离。
- 所有 NPC 共用一个 C++ 管理的对话 UI，不需要 NPC 蓝图业务逻辑。
- 玩家交互菜单及剧情分支选项均支持任意数量和条件过滤。
- Story 对话、询问返回菜单、特殊页面和退出的生命周期可预测且不会锁死输入。
- 现有 NPC Scene 正式对话/交互对话语义完成迁移，无固定三按钮依赖。
- 委托 UI 未定稿不阻塞对话系统，也不会显示无法执行的入口。
