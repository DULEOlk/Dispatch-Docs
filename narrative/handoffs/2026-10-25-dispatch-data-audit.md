# 派遣页数据核对与控件修正

- 2026-10-25 Asia/Shanghai，UI-DISPATCH-001，父级DEMO-COM-001/DEMO-VIS-001/DEMO-AUTH-001；保留用户布局/美术。明确不包含DEMO-COM-002/DEMO-CLUE-001正式报告事务、物品去向规则/奖励/收集。
- 用户六项要求：去重复已返回，正文统一准备页黑色，NPC四行独立控件，每格三结果可切换，报告段首空两字并少些演示文字，核对现有数据并讨论缺口。后续最新指示撤回补新人物字段的必要性：优先使用现有DA/表，不补造伤势和精神状态。
- 本轮仅展示结构/API及WBP，四个NPC值为展示输入；不添加健康/伤势/精神玩法模型或写正式DA。保留旧NPCStatus字符串适配，无法映射的值显示破折号。结果数组为逐槽展示枚举，不从ConsumedItems猜“使用/遗失”。
- 计划：五件UI资产逐件备份，删除重复印章文字、拆字段/色彩、每格三态Switcher；报告展示格式化段首全角两空格，不删正式内容；预览改为读取已有TestCommission/TestNPC实际字段，审计实际Summary/报告引用；测试9种槽切换与清空/四字段刷新，完整背景复看。
- 当前构建DispatchDataCleanupBuild.log，正式DA只读。父级总设计验证继续需要重验。前轮C++硬编码样例仅为视觉夹具，本轮停止将其当接近实际的预览来源。
- 首轮Build失败108.89秒：控制台注册类型应为FAutoConsoleCommand（传带参数delegate），没有FAutoConsoleCommandWithArgs类；Runtime与其他编辑器模块已编译，未执行资产保存，修正入口类型后最小重建。旧StoryUIPreviewCoordinatorTests仍按拆页前从父树查NPCStatus等，是已有过时夹具，本次局部测试不等于其PIE链已通过，父级维持需要重验。

## 实际资产核对与展示优先级

- AuditExistingDispatchData读取真实TestNPC_01：DisplayName=测试执行人、Profession=测试调查员、Portrait空，InitialBody=Healthy/Current=Idle/Trust=Stranger；DispatchStatusText却手写健康良好/身体正常/状态待命/信任一般，与实际信任枚举冲突。
- TestCommission_01“林地里的脚印”，_02“迟到的货车”，_03“旧塔的灯光”，均ExecutionDays1；Default结果摘要仅“测试委托已完成。”，无返回对话引用/消耗/奖励。_01有插图，另外两件DetailImage空。
- 因用户要求优先真实数据，InitializeFromDefinitions现在明确用已有枚举供应Health/活动State/Trust，旧多行DispatchStatusText保留序列化兼容但不覆盖规则状态。具体伤势不造，显示—；不新增NPC数据字段/资产写入。新增测试“手写一般不能盖过实际陌生”。
- 前一轮真实DA预览与3/3局部验证已通过；此映射调整后最小重建/复验。UI资产已保存，新重复执行清理脚本不必要，避免重建用户树。

## 已完成与最终证据

- UI资产修改：父页/准备/执行中/返回/见闻行五件，备份DispatchDataCleanup_20261025_094821。删除Text_ReturnedStamp重复烘焙文字；三页健康/身体/状态/信任拆成各自Label/Value控件，旧整块控件移除；正文/标题/值使用准备页黑色，印章日期与贴图保留红绿。
- 每个返回物品格新增独立Switcher_CarriedItemResult_i，内部Used/Lost/Returned三个贴图，CarriedItemResults按槽位独立选择；None/空物品折叠，不猜真实去向。原Border/重复Label移除，不保留每格固定一种的旧结构。
- ReportText是显示输入，清空会清旧报告；每段加两个全角空格、段间空行，行高1.35。原数据内容不裁剪，已有Summary很短；真实长报告应使用现有返回对话来源，不在UI改写剧情。
- NPC结构化显示优先真实已有初始Body/Current/Trust枚举，分别供应健康/活动状态/信任，具体伤势无源显示—；没有扩展UNPCDefinition/FNPCInfo的新伤势/精神字段。旧NPCStatus字符串保留兼容外部展示输入。
- 新Dispatch.PreviewExistingData可接受委托/NPC资产路径；原PreviewDecoratedPages/PreviewCompletedLayout入口已转向真实数据预览，不再运行旧硬编码鹿/伤势/长报告样例。只读加载定义、模拟页面模式，不创建实际派遣或改变库存/日历。预览未造日期、未知物品去向、未获得见闻。
- Build4 Runtime映射重建成功24.84秒；Build5最终编辑器入口成功9.97秒。Author退出0、0错误/3警告：既有SSL+见闻行文件锁保存重试后恢复，最终保存成功。
- DispatchDataCleanupFinalVerify.log实际InProgressPresentation/ObservationScrollbarInteraction/PageModes 3/3成功，Controller错误0、TEST COMPLETE退出0、进程退出0。扩展用例覆盖独立健康/身体、9种逐槽结果轮换、空结果折叠、无重复已返回控件、报告双空格/清空、状态清空与重建，以及手写一般不能盖过实际陌生。
- 真实DA三页最终图：Saved/Automation/DecoratedDispatch/Existing_01_Preparation_Actual.png、Existing_02_InProgress_Actual.png、Existing_03_Completed_Actual.png，已复看。没有头像如实空白、实际健康/活动/信任枚举显示，身体详细—，报告“测试委托已完成。”已缩进。空物品无结果印章，不冒充已使用/归还。
- 四件被读取的TestNPC/TestCommission数据资产哈希未变，DispatchActualDataProtectedResult.csv四项True；未写正式DA、不提交/推送、不删除美术。所有本轮命令已结束。

## 数据讨论结论与后续

| 页面内容 | 已有数据 | 当前缺口/建议 |
|---|---|---|
| 姓名/职业/简介/头像 | UNPCDefinition的DisplayName/Profession/Description/Portrait；定义读取已接 | 测试头像为空、身份是测试文案，优先填已有字段；本轮不写DA |
| 整体健康/活动/信任 | FNPCInfo与Definition初始BodyState/CurrentState/TrustLevel、组件OnNPCInfoChanged | 当前定义预览读初始值，未自动绑定选中NPC实时组件；建议接现有状态，不造另一张状态表 |
| 具体伤势/精神状态 | 没有独立字段；活动枚举不是警惕/不安 | 用户最新同意先不补，缺值—；不得把手写展示块当规则状态 |
| 出发日/预计/返回 | Instance.DispatchDay/DueDay/ExecutionDays/State，Definition.ExecutionDays | 需要实例→显示适配；现有预计是整数1，不是原示例2–3范围；没有实例不造日期 |
| 携带物与每件去向 | 现有库存/准备列表/Instance.Preparations；Outcome.ConsumedItems/GrantedItems | 缺按物品/数量明确Used/Lost/Returned的结果记录；消耗不能区分使用和遗失，奖励不能直接当归还。需要确认规则后扩现有Outcome/实例快照，不新建重复物品表 |
| 报告 | Outcome.ResultSummary、ReturnDialogueReference、ResultLetterReference | 已有字段应补正式内容并经实例/已知报告适配，不必新建报告表；测试Summary只有一句，返回对话引用为空 |
| 见闻 | 显示数组接口、既有线索/来源系统 | 尚未把玩家已知来源/已听报告映射到本页，不能提前读隐藏LockedOutcome |

- 建议下一阶段先复用现有NPCInfo与CommissionInstance做读取/刷新适配，并补已有资产的头像/文案；物品逐件结果单独确认后再补。正式DA写入仍须列资产/字段/引用再获授权，本次只有讨论。
- 四类状态：上述UI修正/只读DA预览实现并局部验证；人工复看与数据路线讨论待用户。父级DEMO-COM-001/VIS/AUTH仍需全链重验；旧协调器PIE夹具未迁移/未复验，不报整链通过。

## 健康/身体颜色与第几日

- 用户追加：正常健康/身体用绿色，轻伤/伤势用红色；进行中与返回章都要显示第几日。只改展示，不改身体规则/日期系统或正式DA。
- 颜色跟随独立值刷新：健康/正常/良好等已知正常词绿；伤/裂/疲劳/疾病等异常词红；未知/—保持黑。字段标签、活动/信任仍黑。
- 显示输入增加CurrentDay/ReturnedDay/ExpectedDurationDays（纯ViewData）；两个章按输入显示第N日，缺日期显示第—日而非空白/编造真实日期。预计时长读已有ExecutionDays。
- 只读DA预览使用隔离日历：派发第1日、返回日1+现有ExecutionDays，公式与CommissionSubsystem::ConfirmDraft一致。该测试委托执行1日，预览展示进行中第1日/已返回第2日；不推进真实DayFlow或写DA。正式调用仍须传当前/实例返回日期，尚不宣称全实例接线完成。
- 扩展既有用例验证正常绿→异常红→清空、进行中第3日/返回第5日独立输入。当前构建DispatchConditionDateBuild.log。
- DispatchConditionDateBuild成功149.87秒；Verify实际InProgressPresentation/PageModes 2/2成功、Controller错误0、TEST COMPLETE退出0、进程退出0。用例包含正常健康/身体绿、重伤/左臂擦裂红、传入第3日/返回第5日，已有结果切换和清空验证保持通过。
- 两张1920实际背景已复看：Existing_02_InProgress_ColorDate.png、Existing_03_Completed_ColorDate.png；当前DA健康值绿色、身体详细无数据黑色—，印章第1日/第2日完整可见，预计1日读取现有字段。图片位于Saved/Automation/DecoratedDispatch。
- 本次仅源码/显示接口/预览会话修改，没有保存WBP或数据资产；四件DA哈希继续不变（DispatchConditionDateDataProtected.csv）。没有提交或推送。设计/实现/局部验证完成，人工复看待用户，父级全链仍需重验；真实返回日仍由后续实例适配传入，未把当前日误作历史返回日。
