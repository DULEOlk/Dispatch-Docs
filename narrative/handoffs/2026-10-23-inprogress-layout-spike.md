# 执行中页实际背景粗排

## 身份与范围
2026-10-23，F:/Epic Games/Unreal Projects/Dispatch，main基线1437c4b2。
子任务UI-DISPATCH-001，父级DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001；仅执行中页可逆粗排。
用户要求先大概搭配、后续实际测试，并确认最新布局已保存。
方案见plans/2026-10-23-inprogress-layout-spike.md。新中间条目/滚轮样式/业务数据均后续，不接报告/标签/正式DA/事务，不提交/推送。

## 工作区保护
原准备槽与四竖线、新三页、其他UI素材删除及用户布局修改保留。只保存执行中页，一次备份后作者命令；用已有父壳取景，不改变默认Preparation模式。

## 设计/代码/验证/人工
- 设计：沿用户已有素材粗排，方案中的尺寸为本轮建议值可继续调整。
- 工具：新增DispatchInProgressLayoutSetup.cpp，只编辑器显式作者入口；没有新增运行时数据/采集逻辑。
- 验证：修改前DispatchInProgressLayoutInspectVerify PageModes1/1退出0，完整背景截图已复看，白字在纸面偏淡、底部仅一格。
- 当前开始InProgressLayoutBuild，日志Saved/Logs/InProgressLayoutBuild.log。作者/新进程渲染待运行。
- 人工：最终原生窗口比例与滚动条视觉未验收。

## 停止点与下一步
作者命令审计控件/资源/坐标；唯一底部纸框和两标题须匹配才保存。再渲染完整背景，提供现有父页Class Defaults→Commission|Page→Page Mode切页入口。
首轮Author安全拒绝，无资产保存：实际纸图是(-4,616)、209×247，存在大块透明边，不能按截图可见位置猜Canvas矩形。审计已确认唯一Image_7；改为读取现有纹理alpha边界，仅用Brush UV裁去透明边，保持可见左上位置后排三格。标题实际Y348/603.871，中间容器高度随标题计算。
InProgressLayout3Build成功，Author2退出0、无Blueprint错误/ensure，只保存执行中页。纸面可见左上实际为(24.724,673.811)，只裁透明留白；三格208×170/间隔18，中间空Scroll660×176.9。原文件备份Saved/CodexBackups/InProgressLayout_20261023_211716。FinalVerify正在渲染PageModes+PageAssets。
FirstFinalVerify PageAssets+PageModes 2/2通过、Controller错误0、退出0；三纸格实际背景截图已复看，无越界。纸面字体第一版仍偏浅，本轮仅将工具刚设的旧墨色(.12,.07,.035)进一步压深(.025,.012,.005)，既有用户自定义颜色保持；InkBuild成功，InkAuthor运行中。
背景查看更正：已核对UE5.6 UWidgetSwitcher::OnDescendantSelectedByDesigner，会直接临时设置Slate活动子页，OnDeselected恢复。可在父WBP层级选Page_InProgress看全背景，不必改Class Defaults Page Mode。原建议Class Defaults仅作API说明，不推荐作为反复设计操作。

## 最终交付
- 构建：InProgressLayout3Build、InProgressInkBuild Development成功。
- 作者：InProgressLayoutAuthor2 / InProgressInkAuthor均退出0，无Blueprint错误/ensure；仅WBP_DispatchPage_InProgress保存。原始完整备份InProgressLayout_20261023_211716，第一版粗排备份InProgressInk_20261023_212302。
- 最终独立新进程InProgressLayoutDeliveryVerify：PageAssets+PageModes **2/2成功，Controller错误0，退出0**，实际背景1920×1080图已复看。
- 预览图Saved/Automation/DispatchPages/InProgress_Layout_Preview.png（原BMP仅转PNG编码）；纸框可见208×170，间隔18；Grid总678含外部9pxPadding，实际可见跨度660。
- 中间Scroll_RelatedObservations尺寸660×176.9，Box_RelatedObservationEntries为空。动态生成条目、数据和滚轮视觉后续设计，未烘入示例信息。
- 原字体中默认纯白色改为深棕，其余用户颜色保留。NPC、现有标题/装饰/画像位置保留。
- InProgressLayoutProtected.csv四件（父壳、准备槽、准备页、已完成页）哈希无变化；不改用户其他布局/美术/素材删除/正式DA，不提交/推送。
- 全部工具进程结束。粗排已实现，原生设计器与交互体验未冒充人工通过。下步用户按实际背景调比例，再设计见闻条目与滚轮。

## 在Designer看完整背景
打开WBP_CommissionDispatch，层级展开Switcher_DispatchPages，选Page_InProgress。UE原生OnDescendantSelectedByDesigner只临时改变Slate显示，取消选择恢复；无需改Class Defaults PageMode。子页保存/编译后回父页重新选中该页查看。该入口从本地UE5.6源码核对，未通过原生UI工具操纵用户窗口。

## 同阶段微调：底部方格缩小
用户已自行修改并保存布局（截图全部已保存），要求物品栏方一点小一点。只改Grid_CarriedItemSlots尺寸与Padding：160×160方格、间隔32；保持当前整排中心及顶部位置，其他用户修改保留。UI-DISPATCH-001父级不变，局部视觉待重验；不写新业务/测试，不提交/推送。当前构建CarriedSquaresBuild，随后作者备份/保存单个执行中页并用已有PageModes渲染复看。
CarriedSquaresBuild Development成功；CarriedSquaresAuthor退出0、无Blueprint错误/ensure，备份Saved/CodexBackups/CarriedSquares_20261023_213639。仅改当前执行中页Grid宽高678×170→576×160、水平Padding9→16、按当前中心重算X；三格实际160×160/间隔32，当前Y保持。原生用户最新NPC/标题/滚动区/素材其他修改保留。现跑CarriedSquaresVerify PageModes渲染，局部微调视觉待复看。
CarriedSquaresVerify：已有PageModes 1/1通过、Controller错误0、退出0；当前用户保存布局完整背景已复看，三格160正方形居中、间隔32，无越界。预览Saved/Automation/DispatchPages/InProgress_CarriedSquares_160.png。其他四件保护资产哈希变化0，全部本轮进程结束。原生尺寸审美待用户复看；无提交/推送。
