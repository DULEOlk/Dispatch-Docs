# Dispatch 剧情系统：本地设计与交接入口

整理日期：2026-09-18（Asia/Shanghai）。项目：Unreal Engine 5.6 / Dispatch。

本目录用于在新对话中恢复工作，不依赖旧对话仍能打开。用户已于 2026-09-09 明确要求将早期设计、后续交接和独立规则写入项目本地 docs，覆盖旧文档“只能保存在仓库外”的存放限制。`docs/` 仍被 Git 忽略，未授权强制提交。

## 从哪里开始

新对话先读 [CURRENT.md](CURRENT.md) 恢复当前停止点，再用 [CREATIVE_OVERVIEW.md](CREATIVE_OVERVIEW.md) 按用户自然语言请求定位玩法、剧本、文本、数据或 UI 资料。用户不需要主动记住或说出任何文件名。过去的路线保留供追溯，不是每轮默认阅读内容。

| 文件 | 用途 |
|---|---|
| [CREATIVE_OVERVIEW.md](CREATIVE_OVERVIEW.md) | 日常创作总入口：自然语言路由、文档职责、剧本/玩法/文本/录入/进度流程 |
| [GAMEPLAY_FLOW.md](GAMEPLAY_FLOW.md) | 玩法全貌：玩家从抵达到最终收束怎样游玩；正式天数待定，包含试验方向 |
| [story/STORY_BIBLE.md](story/STORY_BIBLE.md) | 剧本创作入口：世界真相、总纲、人物与委托线；七日和五条线属旧草案，不预设正式数量 |
| [story/TEXT_INDEX.md](story/TEXT_INDEX.md) | 游戏内文本清单：草稿、确认状态和正式 DataAsset 映射 |
| [story/IDEA_INBOX.md](story/IDEA_INBOX.md) | 尚未评估的新想法；记录不等于采用或实施 |
| [ui/PLAYER_HUD.md](ui/PLAYER_HUD.md) | 玩家主 HUD 的视觉原型、组件意图、未决设计与后续验收入口 |
| [ui/COMMISSION_NOTICE_UI.md](ui/COMMISSION_NOTICE_UI.md) | 委托公告板/派遣选择与卷宗原型、线索发现体验边界与后续验证入口 |
| [ui/MAIN_MENU_UI.md](ui/MAIN_MENU_UI.md) | 主菜单与对应地图的早期原型证据、资产位置和后续重做边界 |
| [ui/LETTER_READING_UI.md](ui/LETTER_READING_UI.md) | 信件阅读原型、当前阅读功能与可读文本线索范围/表现的待确认项 |
| [ui/INTRO_SEQUENCE_UI.md](ui/INTRO_SEQUENCE_UI.md) | 开场 CG、抵达字幕与 Intro Sequence 的画面、声音、镜头节拍和演出待确认项 |
| [ui/NPC_DIALOGUE_UI.md](ui/NPC_DIALOGUE_UI.md) | NPC 对话主界面、选项条目与疑似弃用 ChoiceOptions 原型的职责、映射和待确认项 |
| [ui/PLAYER_NOTEBOOK_UI.md](ui/PLAYER_NOTEBOOK_UI.md) | 记录本委托、物品、线索三页的早期原型、职责分界和待解决体验问题 |
| [ui/BOARD_INSPECTION_UI.md](ui/BOARD_INSPECTION_UI.md) | 告示牌检查时的退出文字提示、当前用途与非功能边界 |
| [ui/DAILY_WRITING_UI.md](ui/DAILY_WRITING_UI.md) | 每日撰写早期概念图、现行三段合同与尚无正式 UI 的状态 |
| [ui/COMMISSION_REPORT_UI.md](ui/COMMISSION_REPORT_UI.md) | 返回 NPC 报告与记录本查看双入口、报告事务和概念表现边界 |
| [ui/CLUE_INFERENCE_UI.md](ui/CLUE_INFERENCE_UI.md) | 双线索组合/推论的已实现基础、概念图与未验收体验 |
| [CURRENT.md](CURRENT.md) | 默认先读：当前停止点、四类状态、下一步和精确证据入口 |
| [STATUS.md](STATUS.md) | 历史状态时间线；仅追溯旧过程或冲突时按需读取 |
| [RULES.md](RULES.md) | 开工检查、实现与验证要求、持续交接规则 |
| [DEMO_DESIGN.md](DEMO_DESIGN.md) | 总体权威模板：Demo 体验、七日循环、线索、派遣、报告、撰写及系统职责 |
| [IMPLEMENTATION_MATRIX.md](IMPLEMENTATION_MATRIX.md) | 唯一总进度表：永久总设计编号、子任务登记、代码/测试/人工验收与交接映射 |
| [FOUNDATION_DATA_DESIGN.md](FOUNDATION_DATA_DESIGN.md) | 数据权威模板：内容拆分、目标字段、引用、校验、旧表迁移与正式 DA 预览 |
| [SYSTEM_DESIGN.md](SYSTEM_DESIGN.md) | 当前实现边界与技术摘要；用于连接设计与现有代码，不与上述两份模板竞争 |
| [DECISIONS.md](DECISIONS.md) | 早期方案如何演变，哪些旧要求已替代，哪些仍需确认 |
| [VALIDATION.md](VALIDATION.md) | 最近一次实际验证与可复用命令；不是实时通过保证 |
| [SOURCES.md](SOURCES.md) | 14 份原始文档的完整本地副本、来源路径和 SHA256 |
| [HANDOFF_TEMPLATE.md](HANDOFF_TEMPLATE.md) | 每次阶段交接的模板 |
| [handoffs/2026-09-09-recovery.md](handoffs/2026-09-09-recovery.md) | 本次从中断任务恢复的记录 |

`archive/` 中保存原文，不修改历史内容。历史计划里的“未勾选”、示例代码、预期通过和 Git 命令都不是当前状态；阅读时必须结合 CURRENT、矩阵相关行和最新源码。`evidence/` 保存测试结果摘要与静态审查记录。

日常创作由 `CREATIVE_OVERVIEW.md` 路由：玩法进入 `GAMEPLAY_FLOW.md`，剧本进入 Story Bible，具体游戏文本进入 Text Index；已有视觉/概念证据按上表进入对应 UI 记录。告示牌检查、每日撰写、委托报告和线索组合分别有独立入口，不能把概念图误当成现有 Widget。成为稳定制作要求后，再用 `DEMO_DESIGN.md` 判断它服务哪个 Demo 目标，用 `FOUNDATION_DATA_DESIGN.md` 判断内容保存在哪里，最后用 `SYSTEM_DESIGN.md`、CURRENT 和当前代码核对实现。旧 `Dispatch-Story-Flow-Plan-2026-09-07.md` 只作为历史来源与阶段背景。

所有新对话和子任务都必须通过 `IMPLEMENTATION_MATRIX.md` 连接总设计：开工登记父级 `DEMO-*` 编号，完成可验证步骤后核对矩阵相关行与 CURRENT，有状态变化就同步；阶段结果写 handoff。总设计不记录易过期百分比，子任务也不能只在对话中宣布完成。

## 新对话可直接使用

> 请先读取项目根目录 AGENTS.md、docs/narrative/CURRENT.md 和 RULES.md，查 CREATIVE_OVERVIEW.md 中本次请求对应的路由/流程；按当前子任务 ID 读取 IMPLEMENTATION_MATRIX.md 的相关行，再定位任务需要的设计资料。核对 git status、相关 git diff 和未跟踪文件，不覆盖已有修改。说明当前进度、验证和下一步，按我本次授权继续。历史 STATUS、handoffs 与 archive 仅按需追溯，同一对话已读且未变的规则不逐轮重读。

## 更新方式

日常只把有变化的停止点与下一步写进 CURRENT，并同步矩阵相关当前字段；采用/替代设计路线记入 DECISIONS。同阶段详细证据更新现有交接，阶段切换或重要设计转向才新建；不要把相同过程复制到多处。旧 STATUS 作为历史时间线保留，不再逐轮重复。把实际命令、日志位置、失败原因和未完成项写清楚，避免下一次重复调查或误报完成。

这套记录减少对话中断造成的信息丢失，但需要持续维护，也不能替代本地备份；Git 忽略的 docs 不会随源码推送到其他电脑。

2026-09-09 本地保留设置：根 `AGENTS.md` 已通过 `.git/info/exclude` 中的 `/AGENTS.md` 排除，`docs/` 沿用 `.gitignore`。无需提交这些文件也能继续本机协作；本地排除配置不会随 Git 上传。直接压缩整个项目文件夹时，Git 忽略规则不会替你移除这些文件。
