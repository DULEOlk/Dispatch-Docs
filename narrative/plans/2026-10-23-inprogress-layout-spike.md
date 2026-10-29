# 执行中页粗排预览

2026-10-23 Asia/Shanghai；UI-DISPATCH-001，父级DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001。
用户已保存素材，要求先大概搭配并在实际背景测试；本轮按可逆资产粗排spike处理，不作最终布局/功能验收。

- 使用现有WBP_CommissionDispatch共用书页；在Designer层级选中Switcher_DispatchPages下Page_InProgress，UE会临时切换Slate显示，不改运行默认值。无需新增背景/运行时窗口。
- 执行中页保留用户NPC、图片、标题与分隔素材位置。默认白字改深棕以适应纸背景。
- 当前底部一张纸框复用为三格：208×170，间隔18，原左上位置保持；Grid_CarriedItemSlots统一排列，可由用户继续编辑。
- 中间在两节标题之间新增Scroll_RelatedObservations与空Box_RelatedObservationEntries，宽660，高度取现有标题间距，预留未来生成条目与滚轮样式。
- 不添加正式见闻、物品、日期；动态条目/滚轮视觉/数据接线后续。另两页及外壳不保存。
- 退出：当前保存布局在实际书页下可见；三格等宽不越界，命名滚动容器存在；独立新进程PageModes渲染复看。Designer/原生手感待用户。

