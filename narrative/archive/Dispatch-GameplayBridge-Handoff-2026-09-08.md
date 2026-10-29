# 正式玩法连接：Campaign 启动与线索事件

本轮是有界增量，不是完整七日剧情系统完成。没有修改正式地图、DA、Widget 资产，没有增加实时钟或自动跨日。

## 本轮已接通

- StoryDirector 新增 Campaign 与 GameplayBridge。配置 Campaign 时使用新剧情入口；留空继续旧 RootStoryTree 入口。
- 新入口当前不会同时启动旧根 StateTree，也不自动匹配旧特殊事件规则。Campaign.RootStateTree 仍未接到这个入口；不要直接替换现有完整关卡入口而期待旧生成、引导、教程任务全部自动运行。
- Campaign 内容目录成为本次会话的正式线索来源；结束时恢复此前目录。
- 实际 ClueSubsystem 收集成功后，下一帧提交 ClueCollected 事件：SourceAsset 是目录里的线索 DA，值为 true，SubID/SubjectAsset 留空，日期取收集记录，未知时用当前 DayFlow/当前 Episode 日号。
- Episode.Triggers 匹配该事件并执行条件/效果，再自动完成匹配 DA 的 WaitForClue 节点。
- 已有线索在入口条件判断之前载入为快照，不重新触发奖励；连续等待期间出现的新收集事件先结算触发规则。
- 提交失败保留事件 ID 和队列，停止自动重试。通过 GameplayBridge.GetLastError 查看原因，修复后可调用 FlushGameplayEvents 重试；StopStoryFlow 清理会话。
- 第二个连接层不能接管已启动的 Campaign。停止不删除玩家已经获得的线索。

## 最小验证配置（正式 DA，不是写死测试内容）

1. Campaign.ContentCatalog 登记待收集的 Clue DA。
2. StartingEpisode 中建立 WaitForClue 节点，ContentAsset 选该 DA；连接后续节点。
3. 在 Episode.Triggers 添加规则：EventKey.Type=ClueCollected，SourceAsset=同一个 DA，Effects 填剧情事实变化。
4. 在专用验证关卡的 StoryDirector 配置 Campaign，调用 StartStoryFlow 或启用自动启动。
5. 通过现有调查/信件/对话入口真正收集该线索；无需手动再发送相同剧情事件。
6. 不要把 Widget 内存演示数据当成正式收集。独立 UI 预览保持原状。

## 尚未接通，不应误认为完成

- 对话选择/结束、委托派出/到期/结果、推论解锁、每日撰写提交等事件适配。
- PresentLetter、StartDialogue 等动作的完整自动 UI 执行适配（本轮自动处理的动作只有 WaitForClue）。
- 根 StateTree 与 Campaign 的完整职责协作、跨 Episode/日期调度、委托按天返回、正式存档恢复。
- 编辑器中真实地图和 UI 的手动游玩验收。

## 验证

- DispatchEditor Development 编译通过。
- Dispatch.Narrative：23 项通过，日志 Saved/Logs/Bridge-Narrative-Final.log。
- Dispatch.Story：41 项通过，日志 Saved/Logs/Bridge-Story-Final.log；两组共 64 项通过。
- 新集成测试使用真实 GameInstance、ClueSubsystem、Runtime、Director；覆盖实际收集、延迟事件、已有线索入口条件、重复收集、重入收集、失败回滚、目录恢复、第二个连接层和停止。
- 核心空实现与已有线索顺序问题均观察到失败后修复再通过；代码审查提出的三项顺序/错误问题已修复复核。
