# 第一轮数据规范交付

## 已实现

- Episode 动作验证：信件、对话、委托、等待线索、撰写分别要求对应 DA 类型；空引用、加载失败、错误类型报错。
- 对话/委托的可选 TargetAsset 要求 NPCDefinition。RequestScene 仍沿用 ContextID 场景键，不宣称已迁移场景系统。
- 每个节点的 TransitionID 必填且唯一；默认分支/默认章节出口不允许填写会被忽略的条件。
- Episode 日次不能为负；Campaign 的 Maximum Days 默认 7，校验章节和目录中的日期范围。它目前只是创作校验，不会修改 DayFlowManager 或推进日期。
- 配置 Catalog 的 Campaign 检查动作、条件、效果的资产引用是否登记，并检查主要事实类型的来源 DA 类型。旧的无 Catalog Campaign 暂时保留结构校验兼容性，不应作为新内容的配置方式。
- 缺失值和类型错误的比较即使取反也不满足。判断尚未获得使用 NotExists；不能用“值不等于 X”代替未知状态判断。
- 整数效果加法溢出会拒绝整批效果，保持已有记录不变。
- 编辑字段增加说明；Episode 的 Nodes 按 NodeID 标识数组条目。
- Episode/Campaign 的 Validate Definition 暴露为蓝图可调用函数，返回是否通过与逐条错误。

## 使用

1. 重启编辑器载入新 C++ 反射字段。
2. 打开已有 Episode DA，在 Nodes 设置 Action.Type 与 ContentAsset，在 Transitions 配置条件和目标 NodeID。现有 Narrative 页面仍为概览，本轮没有交付图形连线编辑器。
3. 把涉及的正式 DA 登记到 Campaign.ContentCatalog，再把 Episode 放入 Campaign.Episodes。
4. 对 Episode/Campaign 使用内容浏览器的资产验证；也可以在编辑器工具蓝图调用 Validate Definition 读取 Out Errors。错误包含章节、节点或分支等定位信息。
5. 同优先级仍按原数组顺序选择，数值越大优先级越高；默认分支只在普通分支均不满足时选择。入口条件不满足不等于等待，会走 FailureNodeID 或停止。
6. Run Once 沿用当前 Campaign 会话内成功一次的语义；不是每天一次，StartCampaign 会重置该会话。

无需改现有 UMG 控件名称；本轮没有修改、删除正式 DA/DT、蓝图或地图，也没有提交 Git。

## 明确未完成

这次是数据规范基础增量，不能称为完整第一批联动交付。计划中的触发入口、每天/每实例重复策略、完整事实值合同及可读冲突诊断仍需补齐；返回天数归于派遣生命周期步骤。

Director-Campaign 初始化、真实玩法事实桥接、等待条件自动推进、取消回执、NPC 活动、换日事务、可视化剧情图也不属于本次已实现项。不能据此声称正式第一天已经能运行。

## 验证

新增 Contract.Authoring/Catalog/Values 三项测试先运行失败，确认原实现存在缺口。修改后上一轮 Dispatch.Narrative 共 21 项通过；最终验证结果由本轮回复补充。

只读审查代理上轮因账户用量限制未启动，没有取得独立审查结论。不能将它报告为审查通过。
