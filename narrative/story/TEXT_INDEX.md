# Dispatch 游戏内文本索引

更新日期：2026-09-16（Asia/Shanghai）。  
性质：连接剧本创作稿、用户确认状态与正式 DataAsset 的唯一文本清单。

本文不替代 [STORY_BIBLE.md](STORY_BIBLE.md) 的世界真相和人物认知，也不替代 DataAsset 中游戏实际采用的运行版本。用户确认文本内容后，仍需单独批准资产修改范围。

> **范围标记（2026-10-28）**：文本索引中的七天、最终收束和六委托批次均按当前设计口径处理。六委托批次是开发预览内容；正式天数和剧情文本待定，可能延长，不能据此推导正式时间线。

## TextID 规则

格式：`TXT-<TYPE>-<STABLE_NAME>`。

| TYPE | 文本类型 | 通常对应内容 |
|---|---|---|
| `DLG` | 对话 | 完整会话、节点与选项 |
| `LTR` | 信件/公告/可读物 | 来信、公告、书页、正式文件 |
| `COM` | 委托 | 委托标题、说明和准备反馈 |
| `RPT` | 返回报告 | NPC 返回后的报告内容 |
| `CLU` | 线索 | 玩家明确获得的信息 |
| `INF` | 推论 | 玩家连接线索后形成的判断方向 |
| `ENV` | 环境文本 | 标牌、物件、现场观察等 |
| `WRT` | 每日撰写 | 撰写段落和选项 |
| `END` | 验收/结局 | 最终收束阶段的验收和结局文本（日期待定） |

`STABLE_NAME` 使用稳定语义名称，不使用日期、数组序号或文件当前行号。修改正文不更换 TextID；文本含义完全改变时创建新 ID，并把旧项标为已替代。

## 文本清单

| TextID | 类型 | 所属人物 / Arc / 日期 | 揭示或误导目的 | 前置知识 | 草稿路径 | 创作确定度 | 制作进度 | 正式 DA 路径 | 最近确认 |
|---|---|---|---|---|---|---|---|---|---|
| `TXT-DLG-FORMAT-EXAMPLE` | DLG | 格式示例 | 展示本表填写方式；不是正式剧情 | 无 | 本表 | 想法 | 草稿 | 无 | 未确认 |

上表第一行只用于格式示例，不属于正式剧情。创建第一项真实文本时可以删除该行，不能把它录入游戏资产。

## 2026-10-25 记录本六委托批次

本批仅接入现有开发预览资产；用户已明确授权写入范围。2026-10-26用户分别确认六主图/六卡图/六右下附图保存，Presentation.DetailImage关联CommissionMain，ListIcon关联CommissionCards，MapImage关联CommissionInsets。旧“NPC继续暂空”已过期：第4/5/6委托已分别关联维拉/塞维林/罗兰，前三条未关联；本轮来源测试重新通过。不自动视为完整多日剧情或奖励结算。

维拉三条见闻仅存在于本机记录本预览配置，用户允许压缩以恢复三条同屏。当前摘要统一见[记录本当前段落](../ui/PLAYER_NOTEBOOK_UI.md#observation-layout-current)，[用户原稿](../evidence/2026-10-28-context-audit/vera-observations-original.md)仅供恢复。以下TextID用于稳定追踪，不代表正式线索或正式DA写入。

| TextID | 来源 | 内容状态 | 实施与验收 |
|---|---|---|---|
| TXT-COM-VERA-OBS-STATION | 第2日 · 边境驿站 | 原稿用户提供；授权摘要，摘要待复看 | 隔离配置已填、三条同屏渲染通过，未写正式DA |
| TXT-COM-VERA-OBS-TRACKS | 第1日 · 岔路口车辙 | 原稿用户提供；授权摘要，摘要待复看 | 同上 |
| TXT-COM-VERA-OBS-RUMORS | 第1日 · 村镇传闻 | 原稿用户提供；授权摘要，摘要待复看 | 同上 |

| TextID | 类型 | 所属人物 / Arc / 日期 | 揭示或误导目的 | 前置知识 | 草稿路径 | 创作确定度 | 制作进度 | 正式 DA 路径 | 最近确认 |
|---|---|---|---|---|---|---|---|---|---|
| TXT-COM-MISSING_COURIER | COM | 记录本六委托批次 | 失踪的信使 | 对应预览阶段 | batches/2026-10-25-notebook-commissions.json | 用户确认 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_01_MissingForester | 2026-10-25 |
| TXT-COM-SILENT_BELL | COM | 记录本六委托批次 | 不能敲响的钟 | 对应预览阶段 | batches/2026-10-25-notebook-commissions.json | 用户确认 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_02_WellBell | 2026-10-25 |
| TXT-COM-HEARTLESS_DEER | COM | 记录本六委托批次 | 无心脏的鹿 | 对应预览阶段 | batches/2026-10-25-notebook-commissions.json | 用户确认 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_03_EmptyCrates | 2026-10-25 |
| TXT-COM-MISSING_CARAVAN | COM | 记录本六委托批次 | 消失的商队 | 对应预览阶段 | batches/2026-10-25-notebook-commissions.json | 用户确认 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_04_MedicineEscort | 2026-10-25 |
| TXT-COM-INK_ERASED_VILLAGE | COM | 记录本六委托批次 | 被墨水涂掉的村庄 | 对应预览阶段 | batches/2026-10-25-notebook-commissions.json | 用户确认 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_05_MissingVillagers | 2026-10-25 |
| TXT-COM-NORTH_ROAD_MARKERS | COM | 记录本六委托批次 | 修复北路路标 | 对应预览阶段 | batches/2026-10-25-notebook-commissions.json | 用户确认 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_06_RoadMarkers | 2026-10-25 |
| TXT-RPT-MISSING_CARAVAN | RPT | 同批次返回报告 | 消失的商队返回信息 | 已听取后回看；待听时隐藏 | batches/2026-10-25-notebook-commissions.json | 用户确认范围内整理 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_04_MedicineEscort / Outcome.ResultSummary | 2026-10-25 |
| TXT-RPT-NORTH_ROAD_MARKERS | RPT | 同批次返回报告 | 修复北路路标返回信息 | 已听取后回看；待听时隐藏 | batches/2026-10-25-notebook-commissions.json | 用户确认范围内整理 | 已录入，待人工验收 | /Game/Developers/NotebookPreview/Commissions/DA_NBP_06_RoadMarkers / Outcome.ResultSummary | 2026-10-25 |

## 创作确定度

- **想法**：只有初步文本方向。
- **试验中**：正在检查人物声音、信息渠道和玩法作用。
- **已确认**：文字和情节作为当前创作基线。
- **已替代**：不再采用，保留原因及替代 TextID。

## 制作进度

```text
草稿 → 待审阅 → 内容已确认 → 已录入 DA
     → 自动验证通过 → 人工验收通过
```

“内容已确认”只批准文字和情节，**不批准资产写入**。进入“已录入 DA”之前必须：

1. 指定明确文本批次；
2. 检查 Git、现有资产与 [数据设计](../FOUNDATION_DATA_DESIGN.md)；
3. 列出拟创建或修改的 DataAsset、字段、引用和不包含范围；
4. 获得用户对该录入批次的明确授权。

## 新文本登记流程

1. 从 Story Bible 确认世界真相、作者/说话者知道什么以及玩家怎样接触文本。
2. 确认文本承担揭示、误导、人物塑造、委托说明或撰写表达中的哪一种作用。
3. 分配稳定 TextID，并在上表登记。
4. 短文本可以直接写在索引的附属小节；长文本按 TextID 存入 [texts/](texts/README.md)。
5. 用户审阅后更新“创作确定度”和“制作进度”。
6. 正式录入、自动验证和人工验收分别更新，不合并状态。

## 变更影响

修改 Story Bible 的世界真相、人物知识、七日位置或委托线时，必须检查相关 TextID，并把不再可靠的条目标记为“待审阅”或“已替代”。修改 DataAsset 但未同步创作稿时，不得自动把文档覆盖为游戏版本；先核对哪一处是用户认可的新内容。

