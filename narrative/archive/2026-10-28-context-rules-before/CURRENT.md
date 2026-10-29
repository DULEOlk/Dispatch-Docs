# 历史入口快照：包含已纠正结论，不作为当前恢复依据

# 当前交接入口

更新：2026-10-28（Asia/Shanghai）。本页是恢复入口；总进度仍以 IMPLEMENTATION_MATRIX.md 为准。

## 2026-10-28 DispatchPages装饰素材与头像布局
- 用户确认整理 `DispatchPages` 装饰素材，并让预览头像采用截图中的作者槽位。
- 8张直属Texture已归入 `/Game/UI/Commission/Dispatch/Decorations/` 的 `Paper/Dividers/Ornaments/Ink`；4个 `WBP_DispatchPage_*`/观察行Widget仍留在原目录。旧目录无残留Texture；本轮不改正式DA、文案、玩法或结算。
- `WBP_DispatchPage_Preparation`、`InProgress`、`Completed` 的 `Image_NPCPortrait` 均保存为位置 `(31.334627, 76.684608)`、尺寸 `(215.807251, 248.432007)`。备份见 `Saved/CodexBackups/DispatchDecorationsPortraits_20261028_191519`。
- 独立真实RHI派遣页/记录本7项成功，`DispatchDecorationsPortraitVerify.log` 报告 `TEST COMPLETE. EXIT CODE: 0`；人工原生预览仍待用户复看。父级整体不升级。

## 2026-10-28 设计口径纠正
- 用户确认：文档中的“五条核心委托线”、七天剧情和过去讨论的委托关系均是旧剧情草案；当前六个委托只是预览/玩法验证，与正式剧情无固定关系。委托数量、关联、顺序、状态、结果、条件和地图剧情后续重新设计。
- `STORY_BIBLE.md`、`CREATIVE_OVERVIEW.md`、`GAMEPLAY_FLOW.md` 已保留原文并加状态标记，避免把旧草案当作正式需求。当前不修改正式委托资产或运行时逻辑。
- 2026-10-28玩法概念已补充到`GAMEPLAY_FLOW.md`和记录本说明：场景互动、告示牌入口、NPC离开/返回、自由睡觉、每日变化、普通委托作用和非理想结果均标为“待Demo验证”；记录本职责限定为信息/委托状态/NPC报告/线索整理与组合，结局影响仍归撰写系统。未修改代码或资产。
- 2026-10-28文档维护：七天/第七天已在总设计、玩法、数据和剧情入口标为“待定，可能延长”；派遣报告文档已换成当前三页预览图，记录本文档已换成当前列表/详情/卷宗预览图。未找到新版本的线索、信件、公告板和早期独立控件图继续作为历史概念图保留。

## 当前焦点
- 2026-10-28三名NPC头像已替换为“委托插图风格_透明底”256头像，仍按NPC_001/002/003映射；心情、活动状态、健康、信任、简介、关联和布局保持。备份NotebookNPCMoodPortraits_20261028_085938；派遣页/记录本真实RHI7项成功、TEST COMPLETE/退出0，人工原生观感待复看。

- 2026-10-27 NPC状态字段已拆分：界面保留“健康、身体、状态、信任”，其中“状态”改读独立心情枚举（平静/警惕/振奋/不安/低落/失控）；旧空闲/执行中/休息中枚举保留为内部活动状态，不再显示。三个记录本NPC已写入初始心情与256头像：NPC_001警惕、NPC_002平静、NPC_003警惕。NPC/记录本/派遣页回归7项成功、真实RHI7项成功，TEST COMPLETE/退出0。备份NotebookNPCMoodPortraits_20261027_212313；旧活动状态、关联、文案、布局和委托图片保留。人工原生预览待复看。

- 2026-10-27派遣页纵向污渍层级已修复：Image_DecorVerticalInk从前景装饰层移入背景装饰层，保留原位置/尺寸，文字区域未改；维护命令保存前自动备份。真实RHI派遣页与记录本渲染回归成功，TEST COMPLETE/退出0。人工原生预览待用户复看。


- 2026-10-27六委托新文案已保存：六个DA及源JSON的地点、卡片摘要、详情、特殊要求按用户原文更新；保留ID、请求人/报酬/天数、状态、NPC关联、报告和18图。第6特殊要求为“暂无特殊要求。”。摘要变长后新增卡片动态字号适配。备份NotebookCommissionText_20261027_130603；文本/图片等保护验证6/6，真实RHI记录本相关3/3、TEST COMPLETE/退出0。人工复看待用户，父级整体不升级。

- 2026-10-27记录本模拟NPC已接入：新建3个NPC资料资产并写入姓名/职业/简介/初始健康状态信任，头像留空；第4→NPC_001维拉、第5→NPC_002塞维林、第6→NPC_003罗兰，第1～3未关联。旧测试已改为当前关联规则；独立真实RHI关联与六委托读取6/6成功、TEST COMPLETE/退出0。派遣模拟、反应文字、头像和正式运行时状态未改。人工查看记录本待用户。

- 2026-10-27 VS首次构建误报已修：CommissionDispatchPageBase.cpp对应头文件移至第一条include；DebugGame Editor真实重编译/链接成功6.80s、退出0，FirstIncludeDebugGameUBT无相关错误。仅include顺序，无资产或运行时逻辑变化；用户VS F5复看待确认。证据FirstIncludeDebugGameBuild.log，沿用六委托handoff。

- 2026-10-27 Menu/Quest素材整理完成：19贴图归入UI/Commission/Decorations的Annotations(5)/CardStains(6)/InkStains(6)/TitleOrnaments(2)，名称/图像/布局保留；原目录仅6直属Widget和DispatchPages。UE移动并更新4Widget引用，23原件备份QuestDecorations_20261027_101607。Move/独立引用Verify退出0，QuestDecorUIVerify真实RHI8/8、TEST COMPLETE/进程退出0；六新DA及Notebook素材SHA变化0。人工待用户，父级整体不升级。交接handoffs/2026-10-08-ui-dispatch-001-asset-organization.md。

- 当前UI-DISPATCH-001增量：用户要求整理BluePrints/Widget/Menu/Quest素材，审计19 Texture、仅Widget引用，无DA/DT。计划保留名称，归入UI/Commission/Decorations的InkStains/Annotations/TitleOrnaments/CardStains，蓝图留原位，保留未使用素材；UE rename并定点保存引用、清理零引用redirector，不改布局/像素/文案。QuestDecorAudit退出0，清单Saved/Codex/quest_decorations_manifest.json。将先备份再移动，移动后局部UI需重验。

- 2026-10-27旧Quest清理已完成：获专项确认后换9处引用/保存6资产，21旧资产经零外部引用及备份SHA核对后已删除；父目录保留。Migrate/Delete均退出0，LegacyQuestArtVerify真实RHI7/7成功、TEST COMPLETE/进程退出0；六新委托及Notebook素材保护SHA变化0。备份LegacyQuestArt_20261027_082500。局部自动通过，人工待用户；父级整体不升级。

- 旧Quest专项范围已由用户确认“确认，按此替换并删除”；正在执行LegacyQuestArtMigrate保存六引用资产，随后新进程零外部引用及备份SHA检查再删21旧资产。授权不扩展至六新DA、NPC、文案或布局。验证待执行，局部仍需重验。

- 旧Quest清理准备完成（沿用UI-DISPATCH-001）：用户新指示“全删、有用换新的”，已备份21旧资产/6引用资产/3测试源码至Saved/CodexBackups/LegacyQuestArt_20261027_082500。新增显式维护命令审计/替换默认Brush及旧DA/DT图片字段，测试改新素材路径；LegacyQuestArtBuild成功34.04s、退出0，FieldAudit成功退出0/saved=0，精确9引用已核对。尚未保存/删除Content；已列DA_TestCommission_01和DT_Quests精确字段范围，等待项目规则要求的专项确认。确认后执行替换→新进程零引用检查并删除→真实RHI局部验证，具体命令/脚本在六委托handoff。局部引用替换验证需重验，父级整体不升级。

- 2026-10-27旧插图清理检查：用户要求确认无影响再删除`/Game/UI/Quest_Notice/Quest`。完整AssetRegistry同步扫描21资产，其中5有文件夹外引用，涉及WBP_NoticeBoardUI、WBP_QuestListItem、WBP_CommissionCard、WBP_PlayerNotebook、DA_TestCommission_01及DT_Quests；另有3测试源码依赖。未满足整删条件，未删除/改写任何Content资产。审计Saved/Codex/old_quest_art_audit.json、Saved/Logs/OldQuestArtAudit.log（退出0）；后续需先明确替换默认图/旧数据引用的范围，正式DA迁移另列字段授权。沿用UI-DISPATCH-001，不扩大为自动迁移。

- 用户修正附图大小已落实：取消运行时128额外上限，按QuestMapImage作者矩形等比显示。截图位置179.927/215.928、尺寸215.436x224.264已在Widget中保存，本轮未改Widget/图片/DA。NotebookInsetDesignerSizeBuild成功21.22s；Verify独立真实RHI6/6、退出0，实际方图215.436x215.436，DesignerSize_Inset03.png已复看，人工待用户。

- 六张右下附图已获批保存MapImage：红蜡封/残烛/白百合/马蹄铁/旧地图/石拱桥，使用CommissionInsets透明纹理。保留作者槽，视觉等比限128，不挡底部资料；其余主图/卡图等字段保持，原DA备份NotebookInsetImages_20261026_233502。NotebookInsetImagesSavedBuild成功28.25s；SavedVerify独立真实RHI6/6、进程退出0，InsetsSaved实图已复看，人工待用户。

- 左页六张卡片小图已获批并保存ListIcon，03为鹿角替换版，六详情主图与作者布局保留。修复首次异步加载占位尺寸，等比例适配读取稳定原图尺寸；NotebookCardImagesFirstLoadBuild成功95.40s，NotebookCardImagesFinalVerify独立真实RHI6/6、进程退出0，CardsSaved三分类实图已复看。六DA原件备份NotebookCardImages_20261026_224143。

- 最新素材已替换：Downloads固定长方形/信使原色版六张透明1024x431已原位重导入六Texture。NotebookRectangleImagesVerify独立真实RHI6/6、进程退出0，Rectangle实图已复看；DA/Widget/预览配置SHA256均保持，旧纹理/源PNG备份NotebookRectangularImages_20261026_203355。

- 唯一主实施子任务 UI-DISPATCH-001，父级 DEMO-COM-001 / DEMO-AUTH-001 / DEMO-VIS-001。
- 六个现有 NotebookPreview 委托已保存用户文案、委托人、展示报酬和执行天数，文件名/CommissionID保留。4商队待听、5村庄进行中、6路标完成并可读报告；未知日期仍0。
- 本轮用户明确确认“确认，保存六张详情主图”，六个 Presentation.DetailImage 已逐一写入透明1024纹理。记录本详情与卷宗左页共用；运行时在作者矩形内等比例适配，未保存或重排Widget。
- 现行文字：每段首行两格、单换行、正文自动字号，有空间轻下移；详情标题侧饰固定、标题动态字号；卡片长标题和地名字号适配；非准备模式隐藏交付清单三层阴影。保留用户最新UMG编辑。

## 四类状态

| 维度 | 当前结论 |
|---|---|
| 设计 | 六主图、六卡图、六右下附图的映射/保存范围已分别由用户确认；NPC暂空，报酬仍只展示。 |
| 代码/资产 | CommissionMain/CommissionCards/CommissionInsets共18图已导入，六DA已有DetailImage/ListIcon/MapImage引用。原比例适配与首次加载正确；附图按作者尺寸，不再限128。未写Widget、Campaign、Catalog、存档或奖励。 |
| 自动验证 | 当前NotebookInsetDesignerSizeVerify独立真实RHI6/6、TEST COMPLETE/进程退出0，覆盖18图映射、原比例及遵从作者大小、正文、入口、优先级和执行页。旧128视觉上限方案已被用户明确替代。 |
| 人工验收 | 隔离试版后用户已确认保存；最终保存数据的记录本/卷宗实图已复看，用户原生预览的观感待确认。 |

## 下一步和边界

重新打开编辑器/Story Workspace记录本预览查看18图。NPC待后续提供；不自动填入、不扩展见闻/物品去向/历史返回日/银币结算。父级完整Demo流程仍需重验，不能因本轮局部通过升级整体完成。

## 证据入口

- 当前交接：handoffs/2026-10-25-notebook-six-commissions.md。
- 文案源：story/batches/2026-10-25-notebook-commissions.json；主图保存清单：Saved/Codex/notebook_main_images_save_scope.md；精确映射及源图哈希：Saved/Codex/notebook_main_images_manifest.json。
- 原六DA备份：Saved/CodexBackups/NotebookMainImages_20261026_192555/。
- 当前18图实图：Saved/Automation/NotebookSixContent/InsetsSaved_Detail_Dev.NotebookPreview.01.png至06.png；三分类卡图可查CardsSaved_List_0.png至2.png。Rectangle仅反映此前主图版、无附图；候选目录仅作保存前证据。
- 六右下附图映射/范围：Saved/Codex/notebook_inset_images_manifest.json与notebook_inset_images_save_scope.md；本轮原DA备份Saved/CodexBackups/NotebookInsetImages_20261026_233502/。
- 恢复规则：RULES.md、CREATIVE_OVERVIEW.md；总映射：IMPLEMENTATION_MATRIX.md；关联读取历史：handoffs/2026-10-25-notebook-associated-records.md；记录本设计：ui/PLAYER_NOTEBOOK_UI.md。


## 2026-10-28 派遣页/NPC姓名显示修正
- 用户要求姓名后不再显示职业。派遣页和记录本的展示数据现在都只使用 NPC `DisplayName`，`Profession` 仍保留在 NPC 资料中。
- 同步更新 `StoryCommissionAssetPreviewTests` 的姓名断言。构建暂未取得：本机 UnrealBuildTool 启动时因 `C:\ProgramData\Epic\UnrealBuildAccelerator\sessions` 无写权限持续报错，另一次启动因 UBT 日志备份权限失败退出6；需恢复 UBA/UBT写权限后重编译并重跑相关派遣/记录本测试。

## 2026-10-28 四项NPC状态补齐
- 用户反馈预览中“身体”显示为 `—`，并指出四项状态此前已确认。原因已定位：派遣页传入健康/状态/信任结构化值后，旧逻辑跳过了 `NPCStatus` 四行文本解析，身体值因此为空。
- `CommissionDispatchPageBase::ApplyNPCPresentation` 已改为结构化字段优先，缺失字段从 `健康/身体/状态/信任` 文本行补齐；截图中的身体值会恢复为已确认内容，其他三项保持当前枚举值。
- `git diff --check`通过。尚未重编译/重跑RHI：本机UBT/UBA仍被 `C:\ProgramData\Epic\UnrealBuildAccelerator\sessions` 写权限阻塞，需恢复构建权限后验证。

## 2026-10-28 评估枚举补齐
- 用户确认“身体”对应三项评估：足以胜任、恐怕吃力、难以胜任。已新增 `ENPCCommissionAssessment`，与 `ENPCBodyState` 健康枚举、`ENPCMoodState` 状态枚举、活动状态分离。
- 派遣页和记录本按当前身体状态提供默认评估：健康→足以胜任，疲劳→恐怕吃力，受伤/重伤→难以胜任；四行状态文本也包含身体一行。`git diff --check`通过，待恢复UBT/UBA权限后重编译和RHI验证。

## 2026-10-28 维拉·凯恩见闻录入
- 用户确认只填记录本三条关联委托中的“消失的商队”，本轮只写维拉·凯恩的三条相关见闻，图片暂不使用。
- 已为记录本预览记录补充 `RelatedObservations` 字段，并在 `Saved/Config/WindowsEditor/EditorPerProjectUserSettings.ini` 的 `DA_NBP_04_MedicineEscort` 条目写入：第2日·边境驿站、第1日·岔路口车辙、第1日·村镇传闻，按用户给出的顺序排列。其余委托、执行人、携带物品和报告未改。
- `git diff --check`通过；需要重新编译/重启编辑器后做预览读取验证，本机UBT/UBA写权限问题仍待解决。

## 2026-10-28 纯文字见闻排版修复（UI-DISPATCH-001）
- 父级DEMO-COM-001/DEMO-VIS-001/DEMO-AUTH-001。用户指出已确认见闻排版出现左侧大空白。
- 根因：RelatedObservationRowBase只隐藏Image_Illustration，外层Size_Illustration仍保留148宽和图片间距，使长正文被挤窄。
- 修复：无插图时折叠Size_Illustration，插图恢复时使用原作者布局；不改字号、页面矩形、正文、数据资产或图片。新增ObservationTextOnlyLayout回归覆盖无图/有图/再次移图。
- DebugGame构建成功；首次宿主仅发现旧的RenderSavedContent（1/1），未发现新测试，故不作为新修复证据。正在补Development构建，日志ObservationTextLayoutDevBuild.log；随后运行实际文字排版与记录本渲染。

2026-10-28用户纠正：已确认样式以用户再次提供的三条图文行截图为准。折叠图片框改通栏是错误理解，已撤回RelatedObservationRowBase中的该改动与新增TextOnlyLayout测试；三条见闻文本与预览字段保留。此前通栏2/2通过只证明错误方案，不能作为用户样式验收。恢复源码后尚未重新编译；需对照原图核对长正文排版，不能声称已恢复最终效果。

2026-10-28 已直接分页读取原对话“设计委托记录本左页”（01a133fa-a73e-7ac0-a629-6e36076fa45d）的四字段及见闻样式相关历史，并复看Reference_01_InProgress_Final.png。恢复结论见 handoffs/2026-10-28-original-thread-context-audit.md。身体是当前NPC/委托综合评估，不能仅按健康映射；见闻以原左图右文、两条边缘对齐分隔线、垂直居中和溢出滚动为基线。本次只读核对及文档记录，未进一步改代码或资产。
