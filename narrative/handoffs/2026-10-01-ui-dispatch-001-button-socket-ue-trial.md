# UI-DISPATCH-001：双按钮槽底图 UE 试装

- 父级：`DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。
- 范围：仅交付清单底图及其三层阴影的透明轮廓；不改内部列表、详情、按钮位置/文案/逻辑、物品数据、卷宗、派遣或存档。
- 开工条件：用户认可视觉预览并授权先替换试效；已满足。
- 退出条件：UE 接入、自动验证、呈现截图给用户；自动部分完成，用户原生窗口审美仍待验收。
- 资产：新增 `Content/UI/Commission/PreparationSelection/T_PreparationSelection_FolioButtonSlots.png/.uasset`（1050×1498），原 `T_PreparationSelection_FolioHD` 完整保留。两个 Widget 仅重绑背景及其三层阴影同一新纹理。备份 `Saved/CodexBackups/ButtonSocketFolioTrial_20261001_144557/`。
- 验证：`Saved/Logs/ButtonSocketFolio-Red.log` 中旧资产按新纹理路径/尺寸断言失败；DispatchEditor 构建通过；`Saved/Logs/ButtonSocketFolio-Test.log` 聚焦 13/13 成功；自动图 `Saved/Automation/CommissionDeskPan/Sheet_1920x1080.bmp` 已目视检查双槽、按钮及边缘阴影。原生窗口视觉和真实点击没有替用户确认。
- 工作区：保留前轮阴影相关既有未提交修改；本轮没有提交/推送，也没有清理旧素材。
- 下一步：用户看原生 UE 画面，反馈底槽与按钮贴合程度及阴影接地感；确认后再决定是否收窄材质/阴影调整。
