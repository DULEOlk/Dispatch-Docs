# 记录本关联与统一读取交接

- 身份：UI-DISPATCH-001；DEMO-COM-001/AUTH-001/VIS-001；2026-10-25 Asia/Shanghai。
- 用户授权：完成前文讨论的关联记录和统一读取入口，不增加NPC选择操作。实现范围/退出条件见../plans/2026-10-25-notebook-associated-records.md。
- 现有全部源码/UI/美术修改保留；本轮不写DA内容，不改Widget布局，不提交推送。
- 当前：已读相关数据/玩法/技术边界，准备失败测试。记录本/预览数据入口相关历史验证需要重验；父级全链不升级。
- 尚未完成：runtime适配、关联批量入口、预览接线、验证。

## 已实现与当前验证
- 新增NotebookCommissionData.h/cpp：FNotebookCommissionRecord仅保存Commission/Executor引用与预览阶段日期、可选正式InstanceID；ReadRecord每次读DA，ReadInstance验证CommissionID/NPCID并优先实际NPCInfo。详细伤势仍无源显示空；旧DA无写入。
- 新增NotebookCommissionRecords.cpp：BuildAssociatedQuestRecords原子替换、重复ID拒绝；ReadQuestDispatchPresentation标签打开按关联重读；正式模式自动合并CommissionSubsystem实例、订阅状态提交/NPCInfo事件，退出/隔离模式解绑。相同CommissionID列表显示最新StateChangeSerial实例，已取消最新实例不会回退到旧实例。旧BuildQuestList/SetQuestDispatchPresentation保留。
- 页面打开期间阶段/状态变化原位更新；独立OpenDispatchQuestID避免Active→Completed离开当前筛选后停止刷新。保留原筛选/滚动，旧返回和排序回归不变。
- StoryNotebookPreviewData每项新增已关联执行人NPC DA引用和预览当前日；仅作者配置，不增游戏选择UI。BuildAssociatedRecords与正式页面使用同一读取器，协调器禁用runtime源后批量接入，不再只BuildQuestList。
- ReadInstance只有Resolved且bOutcomeEffectsApplied才允许读取ResultSummary；Ready/Reporting不读报告。实际返回日未被现有实例记录，不能把DueDay当实际返回日，因此该字段仍—；预览完成日可独立输入。携带物逐件去向、已知见闻仍沿用既有显示输入，未新增权威来源；不声称这些玩法已接全。
- 红灯NotebookAssociationRedBuild成功19.80s；NotebookAssociationRedTest原Entry因缺批量关联入口/预览Executor属性按预期失败。后续构建修正了TObjectPtr迭代声明和私有筛选测试调用，最终NotebookAssociationBuild成功7.67s。
- NotebookAssociationVerify.log真实RHI14/14成功，进程退出0：Association.Entry/Instance/LiveRefresh/ReadSources/Widget 5项；NotebookQuickPreview.Data/Entry/Lifecycle 3项；RenderedPresentation、AwaitingReport、DispatchEntry、MetaFields、MetaMapping、ReportPriority共6项。LiveRefresh使用隔离真实GI/subsystem+委托快照夹具，验证自动列表/事件/NPC状态/页内阶段/报告边界/解绑，不替代完整派遣事务测试。ReadSources覆盖六委托复用三NPC、重读修改资料、坏引用、缺资料清空。
- 未保存任何Widget、DA或地图，未提交/推送。设计/代码/局部自动验证完成，人工正式六委托三NPC资料尚未填写和验收。当前等待只读代码审查，父级全链仍需重验。

## 最终复核
- 只读审查发现并修复：自动合并原本可能覆盖显式InstanceID及其Executor引用，导致同委托另一派遣人的资料显示在当前页。现显式InstanceID优先、Executor引用保留，页面打开时锁定原关联源；自动列表只在无显式关联时选最新实例。
- LiveRefresh新增同CommissionID两个不同NPC实例、销毁Actor后仍凭关联DA显示、另一实例事件不改变已打开页的验证。审查复核确认原身份问题已修复，无其他具体重要遗留。
- NotebookAssociationFinalBuild成功46.00秒；NotebookAssociationFinalVerify真实RHI14/14成功，TEST COMPLETE退出0、进程退出0。范围与首轮相同，LiveRefresh含上述新回归。渲染回归Association_Regression.png已复看；本轮无Widget树/美术修改。diff --check无内容错误（仅已有CRLF提醒）。
- 全部工具进程结束。设计/代码/局部自动验证完成；人工六委托三NPC正式资料与效果待填写/验收。没有任何DA/UI/地图资产写入；没有提交推送。后续按逐资产/字段/引用清单授权填写内容，不因本步数据入口完成而宣布报告/收集/完整派遣总项完成。
