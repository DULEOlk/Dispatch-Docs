# 记录本关联记录与统一读取

UI-DISPATCH-001，父级DEMO-COM-001/DEMO-AUTH-001/DEMO-VIS-001；用户已明确授权实现前述关联及统一读取，无NPC选择界面，无DA内容写入。

设计：新增仅用于展示的FNotebookCommissionRecord，引用委托DA、已关联NPC DA及可选正式InstanceID。一个统一读取函数按调用时的资产/运行状态生成列表与派遣页ViewData；正式实例覆盖阶段/日期/执行人身份，预览仅使用隔离配置。报告只在Resolved且已应用效果后读取，未听报告不读取隐藏结果。缺资料空态，错NPCID拒绝，不猜人物。保持旧BuildQuestList/SetQuestDispatchPresentation兼容，不改变派遣事务/布局。

实施计划（使用executing-plans逐步内联执行）：
1. 添加关联/统一读取失败用例（六委托三NPC复用、内容修改后重读、ID错配、缺值、Ready报告不泄露）。
2. 实现runtime读取器及记录本批量关联入口；自动读取CommissionSubsystem实例并订阅状态提交，析构解绑。保留已有非实例列表条目。标签开页重读当前资料；预览禁用正式实例来源。
3. 预览配置每条增加“已关联执行人”资产引用，无游戏选择控件；协调器走相同批量入口，预览状态不写实例/DA。
4. Development构建及相关测试（隔离映射、真实实例阶段、预览配置、旧标签返回与排序），实际Widget渲染复看；只读核对无UI/DA保存。

退出：六条可通过关联引用读取三NPC，无重复填写文案；自动正式实例列表/状态读取可验证，开页资料刷新；旧入口回归通过。尚无逐物品Used/Lost/Returned权威结果记录、未接见闻业务来源，不扩展这些玩法，不从LockedOutcome猜测。首次报告仍在NPC流程。
