# 用户复看：见闻与报告修正

2026-10-29，仅本机保存，人工验收待用户复看。

- [塞维林短见闻及底线](LayoutReview_05.png)：两行摘要，单条保留一条装饰底线；与维拉使用同一个行控件。图仍空。
- [罗兰报告两段](LayoutReview_06.png)：保留全文，合成任务结果/调查发现；普通输入换行不新增段落，空行分段。沿用用户保存的报告框位置(32,592)、尺寸(618.4,223.6)，动态字号17；文字高186.3，小于区域223.6。
- [Development构建](NotebookLayoutReviewGreenBuild.log)、[DebugGame构建](NotebookLayoutReviewDebugBuild.log)均成功；[Development运行测试](NotebookLayoutReviewGreenVerify.log)12项全通过、退出0。
- [红测](NotebookLayoutReviewRedVerify.log)复现了单条缺线、见闻较长和五段分句问题，未把旧基线当本轮通过。
- [输入指纹](manifest.json)包含实际源码、配置、文本批次和用户Completed页资产哈希；该资产只读，未被工具保存覆盖。[预览输入摘录](NotebookPreviewInputs.ini)仅用于恢复隔离模拟。

来源链：本机EditorPerProjectUserSettings的NotebookPreviewQuests → Story Workspace Entry → Record → ReadRecord → WBP_RelatedObservationRow。没有经过DataTable，不是两套硬编码页面。
