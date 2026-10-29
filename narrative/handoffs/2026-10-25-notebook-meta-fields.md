# 记录本详情三项资料独立显示

- UI-DISPATCH-001；父级DEMO-COM-001/VIS-001/AUTH-001；2026-10-25。用户要求委托人、地点、报酬三个数据分开，避免整段Meta换行错位。
- 范围：三个独立值控件/标签，沿用分隔线；FQuestInfo独立Requester/Reward展示输入与已有Location；Commission Presentation新增可选Requester/Reward文本并接现有预览/公告适配。不改写任何现有DA值，不把执行NPC当委托人、不从隐藏Outcome推断报酬。不覆盖COM-002/CLUE-001结算/收集。
- 当前工作区既有源/资产改动保留，不提交推送。写UI前备份；旧Meta接口仅兼容旧调用，不再在此页挤成一块，也不猜拆无标签的备注。
- 开工：截图显示预览备注按一整块Meta挤在三标签旁；现有Presentation只有Location独立。退出：三值各自更新/清空/长文本不串行，模板三独立控件可编辑，读取已有Location，未填值—；实图与局部数据/入口回归通过，人工验收分列。相关记录本数据/布局需重验，父级全链仍需重验。
- 当前开始源布局审计与红灯测试。

## 当前结果与证据
- 旧Text_Meta值框仅94.776宽，TextBlock_73合并三行标签；现替换为六个独立TextBlock，保留原字体/墨色/装饰线。标签x64、数据x132宽350，y676/700/724，各高24；关闭自动换行，溢出省略，悬浮查看完整原值。
- FQuestInfo新增QuestRequester/QuestReward；Commission Presentation新增Requester/Reward；Location沿用已有字段。公告和Story Workspace预览适配均传入结构化字段；旧Display迁移解析保持独立值。未解析Meta备注、未猜测委托人/报酬、未写DA。
- ClearQuestSelection清空三值和tooltip，Populate逐项更新，缺值—；旧Text_Meta仅保留原生兼容绑定，本蓝图已移除。无报告/结算/收集事务变化。
- NotebookMetaRedTest.log原始测试因缺独立Requester属性按预期失败。最终NotebookMetaBuild.log构建Succeeded，7.14s。NotebookMetaAuthor.log退出0、0错误，写前备份Saved/CodexBackups/NotebookMeta_20261025_184801/WBP_PlayerNotebook.uasset。
- NotebookMetaVerify.log真实RHI5/5成功，退出0：MetaFields（独立值/缺值/清空/布局）、MetaMapping（公告与legacy）、CommissionList.RenderedPresentation（实图）、DispatchEntry、ReportPriority。实图Saved/Automation/NotebookList/MetaFields_Independent.png已复看，隔离测试数据，不写正式内容。
- 局部实现/自动验证完成；人工审美待用户验收。DEMO-COM-001/VIS-001/AUTH-001的完整流程仍需重验，本步不宣布总项完成。

## 六委托/三NPC统一录入只读核对（2026-10-25）
- 用户认可三项拆分，转问六个委托图片/文字与三个NPC是否有统一填写/接入设计；本轮仅核对，无源码、UI、DA写入，无新增测试运行。
- 已有Story Workspace/SStoryContentLibrary分类创建/选中详情编辑与Catalog统一引用；CommissionDefinition.Presentation保存图片/正文/委托人/地点/报酬，UNPCDefinition保存姓名/职业/头像/描述/初始状态。六委托可引用复用三份NPC资料，执行关联属于实例NPCID，不应把执行NPC固定写进每份委托定义。
- 已有CommissionInstance保存Definition/NPCID/Preparations/派遣日/到期日/状态与事件；NPCInfoComponent保存实际状态。InitializeFromDefinitions仅初始资料展示入口，不代表记录本已有实例订阅。
- 缺口：Notebook预览每项只有Commission/Stage/待听/日期，没有逐项NPC引用；BuildContents仅生成FQuestInfo。记录本SetQuestDispatchPresentation全源码只有测试调用，尚无正式调用方将实例/NPC/已知见闻/已听报告自动转换为显示数据。
- 后续建议复用现有DA/Catalog/实例，补统一记录本适配器和隔离预览实例配置；预览选引用与状态，不重复抄姓名/头像/正文。携带物逐件Used/Lost/Returned、已知见闻来源和已听报告来源仍需接正式记录；隐藏LockedOutcome不可当玩家已知内容读取。不新建平行数据表，不写正式内容，本轮未升级实现状态。
