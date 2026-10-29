# Dispatch 长文本草稿目录

本目录用于保存不适合直接放进 [TEXT_INDEX.md](../TEXT_INDEX.md) 表格的长对话、信件、委托说明、返回报告、线索、撰写和结局草稿。

## 文件规则

- 文件名使用对应 TextID，例如 `TXT-LTR-ARRIVAL-NOTICE.md`。
- 每个文件开头必须反向链接 [TEXT_INDEX.md](../TEXT_INDEX.md)。
- 必须注明相关 `ArcID`、`RevealID`、人物或日期，并链接 [STORY_BIBLE.md](../STORY_BIBLE.md) 的相关章节。
- 必须分别写创作确定度和制作进度。
- 同一 TextID 的草稿不要复制到多个文件；修订在原文件进行，完全替代时创建新 TextID。
- 最终运行版本以正式 DataAsset 为准；Markdown 保存创作、审阅和迁移依据，不是运行时内容源。

## 推荐文件头

```markdown
# TXT-LTR-ARRIVAL-NOTICE

- 类型：LTR
- 相关 ArcID：无
- 相关 RevealID：无
- 出现日期/渠道：试验中
- 创作确定度：草稿
- 制作进度：草稿
- Text Index：[返回索引](../TEXT_INDEX.md)
- Story Bible：[相关设定](../STORY_BIBLE.md)
```

上例只展示格式，不是正式文本或正式 ID。写作前应先在 Text Index 登记真实条目。
