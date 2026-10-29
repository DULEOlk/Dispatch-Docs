# NPC 视野触发主动对话与脚底锚点设计规格

日期：2026-08-03

## 目标

修正 NPC 场景系统当前两个运行时问题：

- `EntryDialogueTreeID` 不再默认在 NPC 到达后无条件播放，避免玩家远离 NPC 或隔墙时出现正式剧情对话。
- `ANPCSceneAnchor` 的世界位置明确表示 NPC 脚底接触地面的点，避免 Character Actor 原点直接对齐地面导致角色陷入地下。

本次只调整 NPC 场景的正式入场对话触发和锚点位置语义，不改变玩家按键交互、动态交互菜单、对话 UI 布局或完整 NPC 日程系统。

## 已选择方案

正式入场对话由场景系统管理，并支持三种明确的触发策略：

```cpp
enum class ENPCSceneEntryDialoguePolicy : uint8
{
	Immediate,
	OnSeePlayer,
	ManualInteraction
};
```

- `Immediate`：NPC 到达后立即播放，用于明确的强制过场。
- `OnSeePlayer`：默认推荐。NPC 到达后等待；玩家进入有效感知范围、进入 NPC 视野并且无遮挡时，NPC 转向玩家并播放正式对话。
- `ManualInteraction`：场景不自动播放 `EntryDialogueTreeID`，由玩家主动交互触发。该策略不替代 `InteractionDialogueTreeID`；正式对话是否消费或重复由现有剧情状态管理。

普通 NPC 剧情场景默认使用 `OnSeePlayer`。需要绝对立即发生的过场必须显式配置 `Immediate`。

## 数据结构

`FNPCSceneRow` 增加：

```cpp
ENPCSceneEntryDialoguePolicy EntryDialoguePolicy = ENPCSceneEntryDialoguePolicy::OnSeePlayer;
float EntryDialogueRadius = 500.0f;
float EntryDialogueHalfAngle = 60.0f;
bool bRequireLineOfSight = true;
```

约束：

- `EntryDialogueRadius` 必须大于零。
- `EntryDialogueHalfAngle` 限制在 `0 < Angle <= 180`。
- 没有配置 `EntryDialogueTreeID` 时，策略字段不产生运行时行为。
- `bWaitForEntryDialogue` 保留原语义：决定 StateTree Task 是否等待本次正式对话结束。它不决定对话何时启动。

## OnSeePlayer 判定

场景完成 Move、Teleport 或 KeepPosition 后，先应用目标 Routine 和交互对话，再进入 `WaitingForPlayer` 状态。每次更新按以下顺序检查：

1. 场景、NPC 和 Entry 对话仍然有效。
2. 当前没有另一个全局对话正在播放。
3. 找到本地玩家 Pawn。
4. NPC 与玩家之间的三维距离不超过 `EntryDialogueRadius`。
5. 从 NPC 朝向到玩家方向的水平夹角不超过 `EntryDialogueHalfAngle`。
6. 当 `bRequireLineOfSight=true` 时，从 NPC 眼睛位置向玩家视点或胶囊体中心进行可见性射线检测；墙体等阻挡命中会拒绝触发。

判定成功后：

1. 暂停 NPC 的 Wandering 等 Routine 行为。
2. 仅绕世界 Z 轴转向玩家，保持角色直立。
3. 调用现有 `StartStoryDialogueForNPC` 启动 `EntryDialogueTreeID`。
4. 启动成功后记录本次 Dialogue ID，避免同一场景重复触发。
5. 启动失败则场景进入 Failed，并按既有回滚路径恢复 NPC。

距离检测不能代替视野角，视野角也不能代替遮挡检测；三者共同防止远距离、背后和隔墙触发。

## 状态与 StateTree

`ENPCSceneStatus` 增加 `WaitingForPlayer`：

```text
Moving
  -> WaitingForPlayer
  -> WaitingForEntryDialogue
  -> Active
```

- `Immediate` 直接从到达进入启动对话流程。
- `OnSeePlayer` 在条件满足前保持 `WaitingForPlayer`。
- `ManualInteraction` 到达后进入 Active，不自动启动正式对话。
- `Story: Play NPC Scene` 遇到 `Moving`、`WaitingForPlayer` 或 `WaitingForEntryDialogue` 时返回 Running。
- 若 `bWaitForEntryDialogue=false`，对话成功启动后 Task 可以返回 Succeeded，但场景仍负责在对话结束后恢复 Routine。
- StateTree 中止、同一 NPC 被新场景接管、NPC/锚点销毁时必须解除等待，不允许旧场景稍后意外启动对话。

## 与玩家主动交互的边界

- `EntryDialogueTreeID` 是剧情正式对话，由上述策略控制。
- `InteractionDialogueTreeID` 是正式对话后的阶段短句，由 NPC 原有近距离交互范围和玩家输入触发。
- 动态交互菜单仍由现有统一 C++ 对话 UI 生成。
- 本次不把 NPC 视野检测用于普通按键交互；普通交互继续要求玩家进入 NPC 的 InteractionBox，避免远距离按键交互。

## 锚点脚底语义

`ANPCSceneAnchor::GetActorLocation()` 表示 NPC 脚底应该接触地面的世界位置。锚点 Actor 的 Yaw 表示 NPC 到达后的基础朝向；Pitch 和 Roll 不应用到 Character。

Teleport 的 Character Actor 目标中心为：

```cpp
AnchorLocation + FVector::UpVector * Capsule->GetScaledCapsuleHalfHeight()
```

规则：

- X/Y 与锚点一致。
- NPC 脚底 Z 与锚点 Z 一致。
- 旋转只复制锚点 Yaw。
- Move 模式将锚点脚底位置作为导航目标；到达后设置锚点 Yaw。
- KeepPosition 不改变 NPC 位置，但若场景配置了有效锚点，仍可使用锚点定义 Routine 中心和目标朝向。
- Wandering 的区域中心以锚点平面位置为准，随机点仍投射到 NavMesh。
- `RestorePrevious` 继续恢复缓存的原始 Actor Transform，不套用脚底偏移。

策划在地图里只需把 `BP_NPCSceneAnchor` 根节点放在地板表面，并通过 Actor 箭头/红色 X 轴判断初始朝向。

## 性能与更新方式

`WaitingForPlayer` 仅存在于已激活且尚未开始 Entry 对话的少量剧情 NPC。检测由 `UNPCSceneSubsystem` 统一更新，不给所有 NPC 添加永久 Tick。

建议使用低频更新（约 0.1 秒一次）完成距离、视野和射线检测。先进行廉价的距离与夹角判断，最后才进行射线检测，避免无意义的物理查询。

## 日志与失败处理

以下情况输出包含 SceneID、NPCID 和 DialogueID 的清晰日志：

- 找不到本地玩家时保持等待，不立即判定场景失败。
- NPC 或锚点在等待期间被销毁时场景失败并清理。
- Entry 对话 ID 无效或启动失败时回滚场景。
- 被其他对话占用时保持等待；其他对话结束后重新检测。
- 射线检测被阻挡属于正常等待条件，不刷 Warning 日志。

## 自动化测试

至少覆盖：

- 新字段默认值和 DataTable 行读取正确。
- `Immediate` 到达后立即启动对话。
- `OnSeePlayer` 在玩家过远时不启动。
- 玩家距离满足但在 NPC 背后时不启动。
- 玩家在视野内但隔墙时不启动。
- 玩家进入距离、视野且无遮挡后只启动一次。
- NPC 在触发前正确转向玩家且不产生 Pitch/Roll。
- 其他对话占用时等待，释放后正常触发。
- 对话启动失败恢复 NPC 原状态。
- 新场景接管或 Task 中止后旧等待不会触发。
- StateTree 对 WaitingForPlayer 返回 Running，并覆盖成功和失败路径。
- Teleport 后 NPC 脚底位置与锚点位置一致。
- 不同胶囊半高的 NPC 均不会陷入地下。
- Move 到达后脚底高度和 Yaw 正确。
- RestorePrevious 恢复原 Actor Transform。

## 编辑器配置示例

```text
SceneID                    Scene.Test.Innkeeper
NPCID                      NPC.Innkeeper
AnchorID                   Anchor.Inn.Counter
ArrivalMode                Teleport
RoutineState               Sitting
EntryDialogueTreeID        Dialogue.Test.Entry
EntryDialoguePolicy        OnSeePlayer
EntryDialogueRadius        500
EntryDialogueHalfAngle     60
bRequireLineOfSight        true
bWaitForEntryDialogue      true
InteractionDialogueTreeID  Dialogue.Test.Inquiry
```

玩家进入旅店并被 NPC 正面看见后才播放 `Dialogue.Test.Entry`；站在旅店外、NPC 背后或隔墙位置均不会播放。
