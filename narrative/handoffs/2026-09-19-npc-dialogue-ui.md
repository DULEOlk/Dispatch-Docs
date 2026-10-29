# 阶段交接：NPC 对话与选项 Widget 原型

- 日期：2026-09-19
- 子任务：`DOC-UI-009`
- 父级：`DEMO-VIS-001`、`DEMO-EXP-001`、`DEMO-NPC-001`
- 状态：文档和截图归档完成；没有修改业务资产

## 记录结果

已保存用户提供的 NPC Widget 目录截图：

- `docs/narrative/ui/images/npc-dialogue-widgets-early-prototype-2026-09-18.png`

并新增 [NPC_DIALOGUE_UI.md](../ui/NPC_DIALOGUE_UI.md)。静态核对显示 `Config/DefaultGame.ini` 的 `DialogueWidgetClass` 指向 `WBP_NPCDialogue`；主界面与 `WBP_NPCDialogueOptionEntry` 分别对应对话展示/选项条目。用户说明 `WBP_NPCChoiceOptions` “好像已舍弃不用”，本轮将其记录为疑似弃用，未删除、移动或修改引用。

## 后续

如要处理旧 Widget，先做只读资产引用审计；只有用户确认目标和删除范围后，才能迁移或删除。对话视觉、具体选项信息、关闭/锁定规则及正式文本均未确认。
