# 编辑器手动工作清单

代码完成后，需要在编辑器中完成以下操作：

## 1. 创建 DT_NPCInteractions

在 Content/BluePrints/DataTable/NPC/ 下创建 DataTable：
- Row Structure: `FNPCInteractionSetRow` (from NPCInteractionTypes.h)
- 命名: `DT_NPCInteractions`

推荐行名和内容示例：
- `Interaction.Guard.Day01` → Options 中添加一个 StartDialogue 选项 + 一个 Exit 选项

## 2. 创建 WBP_NPCDialogue（纯布局，无事件图逻辑）

继承自 `UDialogueScreenWidget`，在 Content/BluePrints/Widget/NPC/ 下创建：

必须包含以下 BindWidget 控件：
- `Text_SpeakerName` (TextBlock) — 说话者名称
- `Text_Dialogue` (TextBlock) — 对话内容文本
- `List_Options` (ListView) — 选项列表，Entry Widget Class 设为 `WBP_DialogueOptionEntry`
- `Panel_Continue` (任意 Panel, 如 CanvasPanel / HorizontalBox) — 点击继续区域

事件图必须为空。
设置字体、背景、动画和导航样式。

## 3. 创建 WBP_DialogueOptionEntry（纯布局，无事件图逻辑）

继承自 `UDialogueOptionEntryWidget`，在 Content/BluePrints/Widget/NPC/ 下创建：

必须包含以下 BindWidget 控件：
- `Button_Option` (Button) — 选项按钮
- `Text_Option` (TextBlock) — 选项文本

事件图必须为空。

## 4. 配置项目设置

Project Settings → Dispatch Content：
- NPC Interaction Table = DT_NPCInteractions
- Dialogue Widget Class = WBP_NPCDialogue

## 5. 更新 NPC Scene DataTable

为现有 NPC Scene 行填写 `InteractionSetID` 字段。

## 6. 更新 Dialogue DataTable

为现有 ChoiceNode 的 Choices 补充唯一 `ChoiceID`（如 `Choice.Stay`、`Choice.Leave`）和可选的条件。

## 7. 清理 NPC 蓝图旧 UI 事件图

移除以下 BlueprintImplementableEvent 的事件图实现：
- OnOpenDialogue
- OnShowStoryOfferChoices / OnHideStoryOfferChoices
- OnShowManualChoices
- OnNPCSubtitleRequested
- OnRequestNPCInfo
- OnRequestCommissionUI
- OnEnterConversation / OnExitConversation

旧的 WBP_NPCSubtitle、WBP_NPCChoiceOptions、WBP_NPCStoryOfferOptions 等 Widget 可以保留作为视觉参考。

## 8. 委托 UI 处理器

当委托 UI 设计定稿后，注册 OpenCommission 的页面处理器。
