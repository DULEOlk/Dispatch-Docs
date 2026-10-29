# 2026-10-29 派遣预览三委托与九交付物

子任务UI-DISPATCH-DATA-002，父级DEMO-COM-001/VIS-001/AUTH-001。

用户目标：记录本后三委托用于独立派遣预览，全部使用罗兰；九物品复用原图、名称和用户已有简介。两入口只共享测试引用，不共享运行状态。

- [三委托选择页](Selection.png)
- [消失的商队卷宗](Dossier_1.png)、[被墨水涂掉的村庄](Dossier_2.png)、[修复北路路标](Dossier_3.png)
- [交付物清单首部/旧路线详情](Delivery_First.png)、[尾部/方向草记详情](Delivery_Last.png)
- [Development最终构建](DispatchReuseFit2Build.log)、[DebugGame构建](DispatchReuseDebugBuild.log)均成功。
- [Development最终5项运行](DispatchReuseFitVerify.log)全部成功、退出0；含独立身体模拟启闭、真实三卡/罗兰/九行及九项详情、相邻交付按钮/进行中展示、记录本真实渲染。DebugGame只构建，未运行同组。
- [当前三条派遣输入](DispatchPreviewInputs.ini)，[唯一文本与图映射](../../story/batches/2026-10-29-dispatch-preview-reuse.json)。六条NotebookPreviewQuests与开工备份逐条一致。

正文与布局：长卡名在原框单行缩字号，罗兰完整简介在原框缩字号；纸张/头像/控件位置和资产不动。九项清单摘要可能按原单行宽度截断，选中后详情显示全文；没有更改交付表布局。委托资产未提供出发反应，本轮不编造NPC台词，引用框保持空。

已打开三卡、卷宗和清单首尾实图复看；新数据/字号外观等待用户验收。旧Before_*仅布局参考，旧三委托/测试执行人/四物品输入已被替代；BeforeFit_*是修复前长标题换行/简介截断证据，不是当前版本。记录本已验收基线保持。

实现仅编辑器快照增加独立身体模拟开关/三选项，默认关闭、不按健康推导；生产文字适配保留作者字号上限用于恢复。正式DA/Widget/存档/库存/结算不改；9物品数量1仅显示样例。未提交/推送，全部本轮进程已结束。

失败记录：红测缺身体模拟字段；首次渲染测试会话被地图GC回收，改强引用保活；构建补HittestGrid头与Slot局部重名后通过。详见本轮交接，不将失败日志隐藏为全程通过。
