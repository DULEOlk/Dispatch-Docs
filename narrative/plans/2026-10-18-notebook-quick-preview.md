# J记录本快速预览 Implementation Plan

> 使用 superpowers:executing-plans 顺序执行；用户已授权快速预览，不实施游戏流程预设。保留脏工作区，不提交/推送。

**Goal:** 在Story Workspace启动完整J记录本的可交互隔离预览。
**Architecture:** 扩展已有PreviewRequest/Coordinator和ValidationPreview。委托引用DA并映射FQuestInfo，物品引用现有AItem蓝图CDO并复制ItemInfo，Day/线索复用Catalog校验和PIE子系统；不写CDO/DA/存档。
**Tech Stack:** UE5.6 C++/Slate/UMG，Development DispatchEditor。
**Spec:** ../handoffs/2026-10-18-ui-notebook-quick-preview.md。

## Constraints / Review Focus
- 空配置可以启动；一条线索也可预览；选了线索必须有有效Catalog Day。
- 缺失/重复委托ID、错误物品类/数量、加载失败给出可定位提示；读源内容，不自动注入固定正文。
- 与原线索/派发预览共用会话归属，不中断无关PIE，结束清理临时状态。
- 保留页签、筛选、详情、信件等现有UI行为；不把UI预览报为真实业务流程已联通。
- 默认资产/用户关卡/正式存档不修改；旧预览回归与真实PIE交互验证。

## Tasks
- [x] 1. 入口回归RED：新增NotebookQuickPreview测试，现有面板拒绝新类型应失败。
- [x] 2. 新增StoryNotebookPreviewData.h与Request：编辑器配置项、DA/CDO映射、输入校验；请求边界测试。
- [x] 3. Coordinator扩展Notebook请求，处理启动后重验/隔离Day与Clue/生成完整Notebook/停止清理；面板新增类型与配置详情接线。
- [x] 4. Development构建、请求/面板/真实PIE生命周期测试与原预览相关回归；复看真实Widget图，更新CURRENT/矩阵/交接，提供启动路径。

## Ledger
- Ruling: 不建第二套运行时或默认测试正文；配置选现有内容，空配置即空态。用户未批准正式DA写入。
- Ruling: 在当前工作区顺序实施，避免把既有未提交Widget美术丢在新worktree；不执行计划模板的自动提交步骤。


- 完成：最终定向7/7，真实PIE三个页面完整截图已复看；用户原生操作体验待验。相关较宽20/21，旧派遣键盘焦点失败不扩项；宽Bootstrap旧空数据断言保留，不改用户内容迎合测试。
- Review ruling: UE EndPIE会保存窗口历史，新预览在ShutdownPIE恢复仅自己启动会话前的窗口偏好；回调不持有已销毁协调器。停止后恢复有真实PIE断言。
- Review ruling: 原生空控件Widget预览会触发旧页签样式空指针，补可选绑定守卫；完整蓝图外观和此前列表尺寸/动画不变，记录本6项相关回归通过。
