# 派遣素材与报告红色划线复现

- 2026-10-25，UI-DISPATCH-001；父级DEMO-COM-001/VIS-001/AUTH-001。用户授权六张素材导入填入、还原报告红色划线并查看接近参考的效果。只做隔离素材样例与富文本展示，不扩展DEMO-COM-002/CLUE-001收集/报告结算，不写正式数据资产。
- 使用用户原PNG，分别是蹄印/路牌/告示与伤药/旧地图/麻绳；复制到ReferenceAssets/SourcePNG，纹理保持原尺寸/比例/透明。128×80与256×160非POT图不拉成方图；POT物品图生成mip。
- 报告控件同名替换为ObservationRichTextBlock，保留字号/位置/段首双空格；新<red>关键词</>渲染红字下划线、支持换行和Clone；原<mark>见闻底纹保持。
- Dispatch.PreviewReferenceAssets为明确的素材样例，使用参考图的见闻/三物品/结果标签与两小段示例报告；原Dispatch.PreviewExistingData和其它实际数据入口保持读取已有DA。未把样例伪装成正式数据。
- 参考中的鹿主图/新头像未单独提供，沿用当前原资源；不恢复已删除的证物/收集板块。退出条件为新图真实背景显示、报告红划线跟字换行、无标记原文露出，当前功能回归通过。
- 新阶段：导入脚本仅保存六新纹理与Completed报告控件，原WBP写前备份。构建DispatchReferenceBuild.log。
- 首轮Build成功181.35秒，导入Author退出0/0错误/1SSL警告，六新纹理实际构建尺寸保持原样；Completed备份ReportUnderline_20261025_110606。首轮Verify数据/见闻滚动/翻页3/3成功退出0，报告红字下划线实际可见且跟随换行。
- 首轮实图发现原PNG透明边使物品偏小，报告继承当前字体偏大。参考样例内进行UMG显示UV裁边/保持图案比例，物品框调整为112×96（进行中）/112×78（返回），不改PNG像素或正式WBP格子；报告样例14号、强调更深红，见闻样例淡黄强调增亮。只针对明确Reference样例，真实数据入口与正式布局不改。

- 最终Build2成功10.57秒，Preview2正常关闭；Reference_01_InProgress_Final.png / Reference_02_Completed_Final.png已实际复看：三条图文完整显示、仅两条分隔线、三物品保持原比例；报告红字下划线无字面标签泄露且换行跟随。八件保护资产全部哈希未变，证据Saved/Logs/DispatchReferenceProtectedResult.csv。
- 本步代码/资产与局部自动验证完成，人工审美验收待用户；父级完整派遣/作者流程仍需重验。参考样例中的伤势/精神/物品结果/报告为明确临时展示，不是新增正式数据记录；真实数据入口不变。
