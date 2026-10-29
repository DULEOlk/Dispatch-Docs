# 派遣预览复用三委托与九物品

## 当前停止点：三份模拟出发反应已填并验证，用户已确认局部效果

用户商队原句及随后村庄/路标截图原句均逐字填入；外层引号由UI装饰提供。唯一文本在批次preview_reactions。只改三条CommissionPreviewAssets的PreviewReactionText，非空覆盖编辑器复制后的瞬态委托；不写正式DA、记录本、存档。反应原框动态字号保留全文，装饰位置不变。

Development的DispatchReactionGreenBuild、DebugGame的DispatchReactionDebugBuild成功；最终三句输入DispatchThreeReactionsVerify 5/5成功、退出0（含源DA不写回、真实三卡九物品/反应适配、相邻UI、记录本渲染）。DebugGame只构建未运行同组。商队单句阶段5项通过单独保留，不混写成最终三句证据。红测复现原Entry缺反应字段后实现。所有自启进程均结束。

更替检查：UI当前确认与CURRENT卷宗主入口已指向[三句最新实图](../evidence/2026-10-29-dispatch-reaction/README.md)；三卡/九物品沿用前一目录，旧空白反应卷宗为历史。矩阵快照/登记、文本批次、TEXT_INDEX、当前停止点同步。三句实际局部均已打开复看，用户本次明确“可以了”并要求本地保存，局部人工验收已确认，当前截图入口已同步。记录本六条输入与开工备份逐条相同，无提交/推送。

本次本地保存标题：完善记录本标签、委托卷宗与派遣预览展示。只作本地保存，不推送远端。

## 以下为本轮实施过程（非当前运行状态）
子任务 UI-DISPATCH-DATA-002；父级 DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001。
用户授权：派遣预览使用记录本后三个委托，全部用罗兰NPC；九物品名称/图/已有简介填入交付物列表。两个预览按钮独立，只共享引用/测试素材。正式DA/Widget/存档/派遣事务/记录本数据不改。
开工：已有委托4/5/6、罗兰和九图，用户明确要求。退出：三卡正确、罗兰信息及头像显示、九物品列表可选/详情显示、独立草稿，不影响记录本；构建/读取/实图与人工分列。
当前设计已确认待实现；现有派遣入口不支持身体模拟，添加独立可选三项评估字段，默认关闭、不计算健康权重。红测CommissionReuse.IndependentAssessment先确认缺字段；日志DispatchReuseRedBuild/Verify。
当前主要任务是本项，UI-DISPATCH-001上一轮黄底/红线局部已收尾。

红测复现缺模拟字段（退出1）；Entry/Snapshot补独立开关与三选项，仅编辑器快照应用，默认关闭，正式映射不继承。三条CommissionPreviewAssets改引用4/5/6与罗兰，每条9物品AvailableCount=1仅样例。简介6条来自本轮截图、后三条沿用前文已提供内容；未编写新剧情。NotebookPreviewQuests与备份逐条一致。首次绿构建因测试缺HittestGrid头文件失败；补齐后DispatchReuseGreen2Build成功。Development运行CommissionReuse及两项相邻UI过滤器中，日志DispatchReuseGreenVerify.log。

第一轮运行4项中3通过（IndependentAssessment、AuthoredPreparationClick、InProgressPresentation），SavedDataRender失败。定位：测试在创建地图前创建Session，CreateNewMap垃圾回收了无强引用会话，卡片/草稿因此无效。并非源DA数据丢失；测试改TStrongObjectPtr保活，生产逻辑未改。DispatchReuseGreen3Build构建中，之后复跑相同4项。

第二轮4/4通过、退出0，实图暴露长卡名换行挤装饰、罗兰简介截断；当前在既有标题宽度单行动态缩字号、NPC简介原框动态缩字号，缓存作者字号以短文本恢复，不动装饰/位置、不删正文。新增实际几何断言，受影响UI与记录本需重验。源码改动扩大至Card/Page正文适配；正式资产仍不写。DispatchReuseFitBuild构建后跑原4项+Notebook.RenderSavedContent。

字号适配首次构建因局部Slot与UWidget成员同名（C4458）失败，改名TitleSlot后DispatchReuseFit2Build成功；当前DispatchReuseFitVerify运行原4项加记录本RenderSavedContent。卡片标题原宽单行缩字号，NPC简介原框缩字号；不缩短名称或删NPC正文。

## 2026-10-29 用户补充出发反应（当前迭代）
只给消失的商队派遣模拟填用户原句，其他两条不变；不写正式DA/记录本。当前入口缺独立反应字段，拟加PreviewReactionText，仅复制到编辑器瞬态委托；正式源字段不改。长句在原反应框动态缩字号，不动布局。当前已确认待实现，相关5项需要重验，先跑独立字段红测。

用户后续两张截图补齐村庄/路标反应，逐字录入独立PreviewReactionText；无新增源码。商队首轮5项已通过、实际反应图已复看；DebugGame构建会话72038尚待返回。三句新输入将重跑相同5项，日志DispatchThreeReactionsVerify.log。
