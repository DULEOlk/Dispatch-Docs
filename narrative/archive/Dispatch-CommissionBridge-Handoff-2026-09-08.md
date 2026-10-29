# 委托操作接入剧情事件

本轮接通现有委托系统的实际操作，属于实施计划步骤 2 的增量。

## 制作配置

1. 在 Campaign.ContentCatalog.Commissions 登记正式委托 DA。
2. 在 Episode.Triggers 新增规则，EventKey.SourceAsset 选择该委托 DA，SubjectAsset 和 SubID 留空。
3. 监听状态变化选 CommissionState；条件使用同一 Key、NameEquals，ExpectedValue 的 Name 填 Active（实际派出）、ReadyToResolve（可结算）或 Resolved（效果已应用）。
4. 监听具体结果选 CommissionOutcome；NameEquals 填委托中配置的 OutcomeID。结果事件在 Resolved 状态事件之后提交，需要同时判断状态和结果时监听 CommissionOutcome。
5. 结果规则可选 OncePerInstance。连接层自动传递真实派遣实例 GUID，不需要蓝图手填。

沿用 StoryDirector 的 Campaign 入口。UI 调用现有 ConfirmDispatch、MarkCommissionReady、ResolveCommission 即可；不要额外从蓝图重复发送相同事件。

## 事件含义

- 只有操作成功后才发布通知；取消草稿、失败操作、重复结算不产生新结果事件。
- 出发时锁定的隐藏结果不会提前提交到剧情事实中。
- 连接层在下一帧处理通知，仍使用已有剧情规则和事件去重。
- 重新启动 Campaign 时，从真实实例载入当前状态快照，入口条件可以查询，不重放触发奖励。
- 同一委托有多个实例时，CommissionState 表示最近一次状态变化；CommissionOutcome 表示最近一次已应用的结果。StateChangeSerial 保证恢复顺序，不表示游玩时间。
- 当前事件作用域是委托 DA；按特定 NPC/实例查询历史仍需要后续数据合同扩展。不能把委托级最新状态理解成每个 NPC 都处于该状态。
- 快照的日期是当前观察日；本轮没有添加历史出发日、实际返回日或存档恢复协议。

## 本轮之外

自动按天返回、实际返回与听取报告分离、NPC 专属剧情条件、对话选择和撰写事件尚未在本轮完成。Resolved 表示现有结算函数成功应用结果，不代表玩家听完报告。

地图、Widget、正式 DA 未由本轮修改。原有独立 UI 预览仍通过原有入口使用。

## 验证结果

DispatchEditor Development 编译通过。Dispatch.Narrative 与 Dispatch.Commission 共 36 项测试通过，日志为 Saved/Logs/CommissionBridge-Verified.log。集成测试覆盖实际派出、可结算、效果应用、结果隐藏、重复结算、实例身份、多实例快照恢复与奖励不重放。审查发现的重复快照启动失败已修复并复核。
