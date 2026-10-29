# Dispatch Story Workspace 与统一内容制作设计

日期：2026-08-26  
状态：待用户最终确认  
位置说明：本设计保存在仓库外的本地目录，不提交到项目 Git。

## 1. 背景

Dispatch 已经拥有信件、对话、线索、委托、NPC、每日撰写、StoryDirector、StateTree，以及第一阶段完成的 Narrative Runtime。当前主要问题不是缺少更多剧情内容，而是：

- 内容分散在多个 Data Asset、DataTable、蓝图和文件夹中。
- 不同功能使用不同 ID、引用与结果表达，难以查找关联。
- 线索等 UI 为了测试而在 Widget 内写死临时数据和独立开关，难以复用。
- 委托派发、委托结算、每日撰写等交互仍未形成完整玩家闭环。
- StateTree、蓝图和内容资产之间的职责不够清晰，Demo 尚未形成完整游戏流程。

## 2. 目标

建立一个名为 **Story Workspace（剧情制作编辑器）** 的统一制作入口，使设计者可以：

- 集中创建、登记、搜索和编辑所有正式内容资产。
- 在同一窗口查看内容库、剧情图、宏观流程、校验与预览。
- 使用真实 Data Asset、真实 Widget 和真实 Subsystem 进行隔离 UI 预览。
- 在内容功能稳定后，用 Campaign/Episode 剧情图组织跨系统因果。
- 让 StateTree 只负责技术流程和宏观生命周期。
- 以后能从一个 Campaign 入口查看从开局到结局的完整路线。

## 3. 非目标

第一批实现不做以下内容：

- 不创作第一日至第七日的具体剧情。
- 不一次完成整个可视化剧情图编辑器。
- 不一次完成所有委托、撰写和 NPC UI。
- 不删除现有 DataTable、蓝图或旧接口。
- 不在正式游戏中加入调试面板。
- 不创建每个 Widget 各自的测试场景资产或测试布尔开关。
- 不让剧情图直接管理镜头、关卡加载、输入模式或 Actor 生成。

## 4. 唯一制作入口

不新增额外的 Workspace Data Asset。`UStoryCampaignDefinition` 继续作为唯一入口；打开例如 `DA_StoryCampaign_Demo` 时，由自定义资产编辑器启动 Story Workspace。

Campaign 增加或保存以下引用：

- `ContentCatalog`：指向 `UStoryContentCatalog`。
- `Episodes`：现有 `UStoryEpisodeDefinition` 列表。
- `RootStateTree`：指向 Demo 的根 StateTree。

Story Workspace 是编辑器窗口名称，不是第四种运行时权威资产。

## 5. 编辑器窗口结构

Story Workspace 使用一个窗口和四个标签页：

### 5.1 内容库

- 左侧：信件、对话、NPC、线索、推论、委托、物品、日期与撰写分类。
- 中间：当前分类的正式资产列表、搜索与状态。
- 右侧：使用 Unreal Details View 直接编辑选中资产。
- 底部：引用关系、校验结果和预览入口。

内容库提供：

- 新建并自动登记资产。
- 从资产选择器添加已有资产。
- 扫描指定内容目录中未登记的匹配资产。
- 确认后批量登记扫描结果。
- 复制资产。
- 从 Catalog 移除引用，但默认不删除 `.uasset`。
- 在 Content Browser 中定位资产。
- 校验选中资产或整个 Catalog。

### 5.2 剧情图

后续阶段提供 Campaign 总览图和每日 Episode 图。节点引用内容库中的正式 DA，连线表达条件、结果和下一节点。

### 5.3 流程 / StateTree

第一版只显示 Campaign 关联的根 StateTree、宏观阶段摘要和“打开 StateTree”操作。后续接入 StoryDirector/Narrative 生命周期桥接。

### 5.4 校验与预览

集中显示跨资产错误、引用消费者、UI 预览配置和未来的路线模拟结果。

## 6. 编辑器模块边界

新增独立的 `DispatchEditor` Editor 模块。

### Dispatch Runtime 模块保存

- 内容 Data Asset 类型。
- `UStoryContentCatalog`。
- 内容统一接口。
- Campaign/Episode/Narrative Runtime。
- 正式游戏 Subsystem 与 Widget 基类。

### DispatchEditor 模块保存

- Story Workspace Toolkit。
- Campaign 自定义资产打开行为。
- 内容库 Slate/Details 界面。
- Asset Registry 扫描与 AssetTools 创建操作。
- Catalog 校验展示。
- 编辑器预览请求与 PIE 启动协调。
- 后续的 GraphEditor、路线模拟和运行追踪。

Shipping 构建不依赖或打包 `DispatchEditor`。

## 7. 正式内容目录

新增 `UStoryContentCatalog : UPrimaryDataAsset`，使用显式、类型安全的软引用数组保存正式内容：

- `Letters`
- `Dialogues`
- `NPCs`
- `Clues`
- `Inferences`
- `Commissions`
- `Items`（预留分类，等正式物品 Definition 建立后启用）
- `Days`

Catalog 的引用是权威列表。运行时不通过目录扫描猜测正式内容。

当前物品仍由 `FDispatchItemInfo` 等旧数据结构表示，并不存在可直接登记的独立物品 DA。第一批不为了填满分类而新造一套物品系统；内容库可以显示该分类为“尚未迁移”，正式物品 Definition 与旧物品数据迁移放到后续独立设计中。

编辑器扫描仅用于发现和登记遗漏资产：扫描结果必须由设计者确认后才能写回 Catalog。剧情图引用了未登记资产时，校验器报告错误并提供“加入目录”操作。

## 8. 统一内容接口

不强制现有 `ULetterDefinition`、`UDialogueDefinition`、`UCommissionDefinition` 立即更换共同父类，避免破坏已有资产。

为正式内容增加统一接口，至少暴露：

- Content ID。
- 显示名称。
- 内容类型。
- 图标或预览资源。
- 数据校验结果。
- 直接引用的其他内容资产。

现有 Definition 逐步实现该接口。编辑器通过接口统一显示资产，但右侧仍编辑各自的专用字段。

## 9. 新内容资产

### 9.1 `UClueDefinition`

表示一条独立原始线索，只描述事实本身：

- `ClueID`
- 标题
- 说明
- 图标/证据图
- Subject/Category
- 可选来源元数据

线索不保存与其他线索的组合关系。

### 9.2 `UInferenceDefinition`

表示由两条或多条线索形成的推论。`RequiredClues` 采用无序、不可重复的正式 `UClueDefinition` 引用集合。

推论保存：

- `InferenceID`
- Required Clues
- 标题、旁注、推论正文、调查方向
- 成功形成后产生的稳定 Narrative Fact

推论不直接打开委托、对话或撰写 UI。剧情图以后根据它产生的事实决定跨系统后续，避免内容资产再次成为隐藏剧情控制器。

现有 `FCommissionLeadDefinition` 暂时保留，通过兼容适配读取；等正式推论资产稳定并完成引用审计后再迁移。

### 9.3 `UNPCDefinition`

表示 NPC 的身份和默认展示数据：

- `NPCID`
- 姓名、简介和肖像
- 初始身体/信任/工作状态
- 内容标签
- 可选角色类或场景表现引用

NPC 的运行时实例状态仍由正式 NPC 系统保存，不写回 Definition。

### 9.4 日期与撰写

`UDayDefinition` 继续保存当前日期展示数据和 `FDailyWritingTemplate`。现阶段不额外拆出 Writing Template DA，因为每个模板天然属于某一天；以后出现跨日复用需求时再拆分。

## 10. 隔离 UI 预览

所有 UI 共用一个开发用预览地图：

`/Game/Developers/StoryPreview/L_StoryUIPreview`

该地图不属于 Campaign、StateTree 或正式关卡流程，也不作为玩家可进入关卡。

预览流程：

1. 设计者在 Story Workspace 选择正式内容资产和预览输入。
2. `DispatchEditor` 创建只存在于编辑器内存中的 Preview Request。
3. 编辑器使用 `L_StoryUIPreview` 启动独立 PIE 会话。
4. Editor Preview Coordinator 在 PIE World 就绪后，使用正式公开 API 初始化该 PIE GameInstance 中的 Subsystem。
5. 打开真实 Widget Class 并使用真实输入路径。
6. 设计者测试选择、取消、确认、组合和提交。
7. 结束 PIE 后，GameInstance、Subsystem 状态和 Preview Request 全部销毁。

Preview Request 不保存成正式 DA，不写存档，不显示游戏内测试面板。

预览示例：

- 线索：选择 Clue A、Clue B、Inference C 和 Day，预览真实 Notebook 组合流程。
- 委托：选择 Commission、NPC 和可用准备物，预览真实派发页面。
- 撰写：选择 Day 和临时已获得事实，预览真实可用选项和提交。
- 对话：选择 Dialogue 和 NPC，预览真实对话 UI。

当前 `bEnableSimpleClueTestOnConstruct` 在新预览通过验证前保留；之后废弃并移除，不复制到其他 Widget。

## 11. Narrative 与 StateTree 的最终分工

### 内容资产

回答“它是什么”：信件正文、线索内容、委托规则、对话内容、撰写选项。

### Narrative Campaign/Episode

回答“为什么发生、之后去哪”：玩家结果、跨系统条件、下一节点、下一日入口和结局。

### StateTree

回答“什么时候、怎样执行”：Boot、Opening、Tutorial、StartDay、RunNarrativeEpisode、DayTransition、Ending、FailureRecovery。

StateTree 不重复保存具体玩家选择的剧情意义。内容资产不直接决定另一个内容系统的正式后续。Narrative Runtime 是跨系统剧情因果的唯一运行时权威。

## 12. 第一批实施范围

第一批只交付能解决当前内容分散和线索测试困难的基础闭环：

1. 新增并注册 `DispatchEditor` 模块。
2. 新增统一内容接口。
3. 新增 `UStoryContentCatalog`。
4. 新增 `UClueDefinition`、`UInferenceDefinition`、`UNPCDefinition`。
5. 为 Campaign 增加 Catalog 和 Root StateTree 引用。
6. 实现打开 Campaign 即进入 Story Workspace。
7. 实现内容库分类、搜索、新建、添加已有、扫描、移出、定位和 Details 编辑。
8. 实现 Catalog 与内容资产校验。
9. 建立 `L_StoryUIPreview` 预览基础设施。
10. 完成第一种真实预览：线索列表、选择、组合和推论。
11. 保持旧 DT_Clues 和现有测试开关兼容，直到新预览验收通过。

第一批不实现完整剧情图、委托/撰写预览、StateTree 桥接、正式物品 DA 迁移或旧资产删除。

## 13. 错误处理与校验

保存或预览前至少检查：

- Content ID 为空或重复。
- Catalog 引用为空、类型错误或资产无法加载。
- 线索/推论 Required Clues 为空、重复或少于两条。
- 推论引用未登记线索。
- Campaign 引用未登记内容。
- Day、Commission、Dialogue 等现有 Definition 自身校验失败。
- Preview 所需 Widget Class 或 Preview Map 缺失。

校验错误包含可点击资产路径，并尽量提供“定位”“加入目录”或“打开修复”操作。严重错误阻止预览；警告不阻止保存。

## 14. 测试策略

- Runtime 自动化测试：Catalog、统一接口、Clue/Inference/NPC Definition 校验和旧数据兼容。
- Editor 自动化测试：资产创建、登记、扫描、重复 ID、移出不删除、Campaign 打开行为。
- Preview 集成测试：Preview Request 只影响预览 PIE GameInstance；停止 PIE 后状态销毁。
- Clue UI 测试：正式 DA 驱动真实 Notebook，A+B 成功、无效组合失败、关闭预览无残留。
- 完整 `DispatchEditor` 编译和现有 `Dispatch` 回归测试。

## 15. 第一批完成标准

- 打开 `DA_StoryCampaign_Demo` 会进入 Story Workspace。
- 内容库可集中创建、添加、搜索和编辑正式内容 DA。
- 新线索和推论不需要修改 Widget C++ 或添加测试开关。
- 可以从内容库选择正式线索和推论，一键启动隔离 Notebook 预览。
- 停止预览后不改变正式存档或 Demo 状态。
- Shipping 不包含编辑器模块或预览入口。
- 旧内容继续工作，没有删除 `.uasset` 或 DataTable。
- 新旧自动化测试和 Editor 编译全部通过。
