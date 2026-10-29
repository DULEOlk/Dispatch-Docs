# 剧情独立触发规则：本轮交付

本轮承接第一步遗留项，新增的是事件规则执行基础，不是已经接通所有玩法的完整第二步。

## 在哪里填写

打开 StoryEpisodeDefinition DA，在 Narrative / Triggers 添加规则，不必为每条规则另建 DA。

- RuleID：章节内唯一且非空。重复记录按章节 DA + RuleID 标识，发布后不要随意改名。
- EventKey：精确填写事实类型、来源 DA、关联 DA、SubID。不是模糊 ID 匹配。
- DayNumber：0 不限制；其他数字限制事件发生日。不会启动计时器。
- ConditionAlternatives：任一条件组满足，组内全部满足。判断时已经包含本次事件事实。
- Effects：使用现有 SetFact / AddIntegerFact 改变叙事账本。它们不会自动发放背包物品、打开 UI 或改变 NPC；这些正式玩法动作还需连接层。
- Repeat：EveryEvent 每个不同事件；OncePerDay 每日一次；OncePerCampaign 当前流程内每条规则一次；OncePerInstance 每个派遣等实例一次。
- ExclusiveGroup：留空表示独立执行；相同组只执行最高优先级的合格规则。
- Priority：越大越优先；同组最高优先级并列时拒绝整个事件，不随机选、不偷偷按数组顺序选。

同一次提交的所有条件读取同一份“收到事件后、应用规则效果前”的快照，前一条规则不能在同一轮里连锁触发后一条。规则处理不接管当前主线节点，也不自动跳转 NamedEntry。

## 运行入口

从 GameInstance 获取 NarrativeRuntimeSubsystem。

1. Start Campaign：传 Campaign DA，检查返回布尔及 Out Error。这个入口会重置新的流程会话，不要每次打开 UI 都调用。
2. Submit Story Event：提交 FNarrativeTriggerEvent，包含 EventID、InstanceID、Fact。
3. Get Last Trigger Trace：查看上次提交的命中、跳过、冲突或错误原因。

EventID 是一次实际发生事项的稳定 GUID。同一事项重试或重复通知必须使用同一个 EventID；不要在每次 UI 重开时生成新 GUID。不同事项才生成新 ID。OncePerInstance 还必须提供对应实例的稳定 InstanceID。日期必须大于零；有 Campaign 时不得超过 Maximum Days。

成功返回也可能是没有匹配规则、受重复策略跳过或已处理事件的无操作，请查看 Trace。失败时整个事件、效果、执行回执均不写入账本，可以修复配置后用同一 EventID 重试。

RecordExternalFact 仍是只同步状态的 C++ 接口，不会触发规则；本轮没有偷偷改变旧调用的语义。

触发和已处理事件的回执存入现有 NarrativeFactLedger 历史，沿用历史序列化与恢复。不是新增完整游戏存档系统；新 StartCampaign 会清空会话历史。规则成功一次的含义与“每天一次”分开。

## 尚未自动接通

真实收集线索、对话选项、派遣确认/返回、撰写提交目前还没有统一自动调用这个入口。Director 选择 Campaign、正式事件桥接、实际动作适配、等待自动推进与取消回执仍是接下来第二步的工作。此时只在 Episode 填规则，不会自动监听整个地图。

本轮未修改正式 DA、蓝图、地图或 UMG 控件名称；无需替换现有按钮。文档只保存在仓库外。
