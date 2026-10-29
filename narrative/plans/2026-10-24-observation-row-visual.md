# 相关见闻单条视觉控件 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [x]`) syntax for tracking.

**Goal:** 提供可独立编辑的相关见闻行，左插图、右日期来源/正文、淡黄重点背景、底部分隔线。
**Architecture:** 独立 WBP_RelatedObservationRow 与只负责展示的 native base；纯视觉富文本支持 `<mark>重点</>`，按文本正常换行。现有页面不注入正式/示例内容，预览命令仅在临时实例中排列示例。
**Tech Stack:** UE5.6 UMG / Slate / DispatchEditor author commandlet。
**Spec:** 用户2026-10-24确认上一轮列出的四个视觉区域，并明确现在只做视觉、不做收集。

## Global Constraints

- UI-DISPATCH-001，父级DEMO-COM-001 / DEMO-VIS-001 / DEMO-AUTH-001；沿用当前工作区，不还原用户布局。
- 不接收集/报告/标签跳转，不写正式DA，不保存演示剧情，不提交/推送。
- 行高度随内容增长，透明背景；插图按比例显示；重点底色仅绘制，不含按钮/手形/点击。
- 本次是可逆视觉实现，不新增镜像式自动测试；使用真实Slate渲染、现有PageAssets/PageModes回归和保护资产哈希验证。

## Review Focus

- 长中文和跨行mark应正常折行，底色跟随各行。
- 无插图应保留稳定的插图区，无白色默认块。
- 无重点文本应普通显示；标记不显示到正文。
- 原用户页面尺寸、布局、三格和其他素材应保持。
- 样例只在临时实例渲染，不能保存到正式页面或DA。

### Task 1: 独立视觉行与背景预览

**Files:** 新增 Runtime 的 RelatedObservationRowBase.h/cpp、ObservationRichTextBlock.h/cpp；Editor的RelatedObservationRowSetup.cpp；只增加Commandlet flag。
**Interfaces:** 属性SourceLine / BodyText / Illustration；SynchronizeProperties()更新绑定Text_Source / Text_Body / Image_Illustration；不提供游戏业务API。

- [x] 添加纯展示行与mark背景文字绘制，创建WBP_RelatedObservationRow，复用既有字体、墨线。
- [x] Development构建成功后执行AuthorObservationRow；已存在资产时保留用户编辑。
- [x] PreviewObservationRow输出独立行、书页和长文预览；无收集入口，超出原视口使用既有滚动。
- [x] 运行PageAssets+PageModes，核对截图、保护资产哈希及编译/退出结果。
- [x] 独立只读review，记录实际证据并同步CURRENT/矩阵；不提交。
