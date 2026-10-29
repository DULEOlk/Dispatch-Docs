# DATA-FOUNDATION-001 实施交接

## 身份与范围

- 日期/时区：2026-09-14 / Asia/Shanghai。
- 子任务：`DATA-FOUNDATION-001`；父级编号见 IMPLEMENTATION_MATRIX。
- 当前批次：Commission 纯展示数据与旧资产兼容边界。
- 不包含：正式资产保存、旧表删除、NPC Activity、移动/锚点/动作 StateTree、正式七天正文。
- 工作目录/分支/HEAD：`F:\Epic Games\Unreal Projects\Dispatch`，`main`，开始 HEAD `ceae683fc8c65d25608f847a0194dce174a7a4d5`。

## 工作区保护

- 开始时 8 个未跟踪 NPC Activity 文件含未完成注册 API 测试。按用户授权完整暂存到 `Saved/CodexPaused/NPCActivity-20260914/`，未删除。
- 暂存后 Git 工作区为空；本批只修改 Commission/Widget/Story Workspace C++ 与测试，以及 Git 忽略的本地文档。
- 未修改或保存 `.uasset`，未提交或推送。

## 第一批结果

- 新增 `FCommissionPresentation`：标题、摘要、描述、制作信息、地点、列表图、详情图和地图图；不含 `QuestState`。
- `UCommissionDefinition::Presentation` 成为正式作者字段；旧 `Display` 保留为已弃用序列化回退。
- `GetResolvedPresentation` 在新结构有内容时整体采用新结构，否则映射旧 `Display`。
- CommissionSubsystem、NoticeBoard、Commission Card/Dispatch Widget、Story Workspace Preview Session 均改用统一入口。
- C++ fallback 预览种子暂改写新结构，但仍将在正式 DA-only 预览批次删除生产使用。

## TDD 与验证

- 初始 RED：DispatchEditor 构建退出 1，首错为缺 `FCommissionPresentation`，其余为缺新字段/API 的级联错误。
- 核心构建：UHT、31 项编译/链接成功，退出 0；后续增量构建均退出 0。
- `Narrative-DataFoundation-CommissionPresentation-Green.log`：DataAsset 3/3。
- `Narrative-DataFoundation-CommissionPresentation-Regression.log`：Commission 17/17。
- `Narrative-DataFoundation-NoticeBoard-Regression.log`：NoticeBoard 3/3。
- `Narrative-DataFoundation-CommissionDispatch-Regression.log`：CommissionDispatch 2/2。
- `Narrative-DataFoundation-CommissionPreview-Regression2.log`：Story Workspace Commission 4/4。
- 正式消费链 mutation：`Narrative-DataFoundation-FormalPreviewConsumer-Red.log` 1/1 预期失败；恢复正式读取后 `Narrative-DataFoundation-FormalPreviewConsumer-Green.log` 1/1 通过。

## 已知非本批失败

- 宽过滤器 `Dispatch.Widget.Commission` 发现 15 项，12 通过、3 失败。
- `HoverDimsPeers`、`IdleDimOption` 由正式 Selection 蓝图实例的 idle dim 默认与测试期望冲突触发；精确复现仍失败。
- `CommissionUIBase.Behavior` 在旧动画夹具销毁时触发 MovieScene SharedPlaybackState ensure。
- 三者断言/调用栈不经过本批 Presentation 数据，保持记录，不在数据基础批次中修改交互或动画。

## 下一步

- 审计 Dialogue Choice 当前提交事务、条件/效果结构和 Narrative 适配点。
- 第二批 RED 测试已加入：Choice 直接效果、目标节点效果、重复提交不重放、Choice 效果校验及重复 ChoiceID；生产字段尚未增加。
- 第二批编译 RED 有效：构建退出 1，唯一合同类错误为 `FDialogueChoiceOption` 缺少 `Effects`（两个测试文件共三处）；尚未计为 GREEN。
- 最小实现已构建；Dialogue Effects 3/3 GREEN。Definition 首跑因新增测试把数组自身元素直接 Add 回原容器而触发 UE 保护断言，已改为局部副本，需重建复跑；不是生产代码失败。
- 第二批最终：构建成功；Effects 3/3、Definition 1/1、GameplayBridge 6/6、Narrative Contract 3/3、ActionExecutor Dialogue PIE 3/3。Choice 与 Node 共用效果入口，Choice 事实仍走既有共享桥接，OpenCommission 直接引用可被扫描。
- 完整 Dialogue 13 项中 12 通过；旧 `Commission.OpenOption` 隔离仍失败，因为夹具无 StoryDirector 导致当前日 0 小于最早第 1 日。未改该无关夹具，不把完整过滤器报告为 GREEN。
- 第三批从 Letter→Clue 开始：类型化引用优先、旧 ID 回退和 Catalog 直接引用测试已加入，生产字段尚未增加，等待编译 RED。
- Letter→Clue 编译 RED 有效：唯一直接错误为 `FLetterContentSegment` 缺少 `Clue`；现已加入软引用与旧 ID 解析边界，待构建/GREEN。
- Letter→Clue 1/1 GREEN。Commission Outcome→Granted Clue 的类型化优先/旧 ID 回退/引用枚举/空引用拒绝测试已加入，等待 RED。
- Commission Outcome→Granted Clue 编译 RED 有效（缺少新字段/解析器）；最小兼容实现已加入，待构建和回归。
- Commission Outcome→Granted Clue DataAsset 4/4 GREEN。Outcome Dialogue/Letter 正式软引用优先、旧硬引用回退测试已加入，等待 RED。
- 第三批最终 GREEN：Commission 18/18、Clue 4/4、Writing 4/4、Catalog 1/1、LetterDefinition 2/2、Notebook FormalAssets 1/1、Narrative Contract 3/3；四类正式软引用均保留旧字段序列化回退。
- 第四批字段定位 GREEN：全部有直接内容引用的 Definition 提供具体字段路径；清除补丁重试造成的逐字重复后构建 51 项成功，Catalog 1/1。任务整体下一步是旧表只读迁移报告。
- 第五批迁移报告 GREEN：Migration 5/5、ModuleLoads 1/1；真实 Catalog 只读扫描为 10 行全部 Missing，reference-zero=false，报告在 `Saved/StoryMigration/LegacyContentMigration.tsv`。没有删除旧表或保存正式资产；下一步为正式 DA-only Commission 预览。
- 第六批正式 DA-only 预览 GREEN：生产路径不再使用 C++ authored fallback，必须选择至少一组正式 Commission/NPC DA；缺失配置会给出可操作错误。核心 4/4、编辑器入口 1/1、Coordinator PIE 2/2 通过，并证明 transient snapshot 初始化、修改与 Reset 不污染源属性或 package dirty state。没有保存或修改正式资产；下一步是受保护样例生成，先 transient dry-run/碰撞测试再允许显式 apply。
- 第七批先取得构建 RED：新增 FoundationSample dry-run 图形状、全路径碰撞拒绝和默认/推论准备分支测试；构建唯一首错是预期的新 builder header 缺失。尚未创建 package 或保存 `.uasset`。
- 第七批 transient GREEN：builder 仅在 `/Game/Developers/` 下规划路径并先全量拒绝碰撞；3/3 dry-run/碰撞/分支测试通过，真实 Commission evaluator 证明默认和推论准备结果。尚未创建或保存样例资产，下一门槛是默认目标目录的真实只读 dry-run。
- 最终目标 `/Game/Developers/StoryDataFoundationSample` 只读 dry-run 1/1 GREEN，目录仍不存在且 13 个路径无碰撞。已解锁 Apply 测试先行；生产 Apply 尚未实现。
- 第七批完成：FoundationSample 7/7、ModuleLoads 1/1；`StoryWorkspace.ApplyFoundationSample` 在 collision-free 目标一次创建 13 个 development-only DA。新进程 ReloadedAssets 1/1，重载 Catalog/Campaign 校验和正式 Commission/Scout 隔离预览均通过，源包不变脏。资产统一位于 `/Game/Developers/StoryDataFoundationSample`，正文均明确标记可替换；人工编辑预览仍待用户验收。
- Task 8 自动门槛：fresh build 通过；Commission 18、Dialogue 4、Content 4、Clue 4、Migration 5、Writing 4、Narrative Contract 3、Preview 4、Preview PIE 2、FoundationSample 8、ModuleLoads 1 均 GREEN。Source/Config diff-check 无错误；全仓检查被本机 Git LFS signal-pipe error 5 阻断。中断期间新增的 `WBP_CommissionSelection` 修改和 `Quest_Gold_Body_Extended_Preview` 未跟踪资产按外部/用户修改保留，未纳入样例。子任务等待用户编辑样例正文/图片/引用并确认预览刷新后再关闭；完整 Demo 与正式七天内容仍未完成。
- 先写“Choice 自身效果只提交一次”的行为 RED，再实现最小兼容字段/适配；不创建第二套剧情账本。
