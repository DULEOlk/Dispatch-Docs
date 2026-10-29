# 正式剧情对话不可中途关闭设计规格

日期：2026-08-03

## 问题

统一对话 UI 当前让 Story 与 PlayerInteraction 两种会话共用关闭按钮和 Esc。`CloseDialogueSession()` 又把中途关闭与正常到达 `ExitNode` 广播成相同的结束事件，导致 NPC Scene 把取消误判为完成，StateTree 提前进入后续状态。

## 规则

- `EDialogueSessionMode::Story` 不允许玩家中途关闭。
- Story 模式隐藏 `Button_Close`，Esc 和手柄菜单键不会关闭会话。
- Story 对话只有通过继续或选项正常到达 `ExitNode`，或者当前节点没有下一节点时，才调用正常结束流程。
- `EDialogueSessionMode::PlayerInteraction` 保持现状：关闭按钮和 Esc 可关闭交互菜单/会话。
- 系统级强制结束（场景中止、关卡清理、新场景安全接管）仍可直接调用 `EndDialogue()`，不受玩家关闭限制。
- NPC Scene 与 StateTree 无需增加取消重试状态；它们只会收到 Story 对话的正常完成事件或系统强制清理事件。

## 接口与 UI

`UDialogueSubsystem` 增加只读判定：

```cpp
bool CanPlayerCloseCurrentSession() const;
```

返回规则：仅当会话有效且模式为 `PlayerInteraction` 时返回 true。

`UDialogueScreenWidget`：

- 每次同步会话和模式变化时刷新 `Button_Close` 可见性。
- `OnCloseClicked()` 和 Escape 输入都再次调用该判定，避免仅靠隐藏控件造成绕过。

## 测试

- Story 会话的玩家关闭请求被拒绝，Dialogue ID 和 NPC 上下文保持有效。
- Story 对话正常到达 Exit 后仍广播结束，NPC Scene 进入 Active，StateTree 可成功继续。
- PlayerInteraction 会话仍允许关闭。
- UI 的关闭能力与会话模式一致。
- 场景中止仍能系统强制结束 Story 对话并恢复 NPC。
