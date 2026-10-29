# UI-DISPATCH-001：清单底部双按钮槽预览

## 身份与范围

- 日期/时区：2026-10-01，Asia/Shanghai。
- 用户目标：让当前交付页背景底部两个按钮拥有类似新预览的嵌入式槽位。
- 授权：先生成看效果；没有授权覆盖现行底图或导入 UE。
- 项目/分支/HEAD：`F:/Epic Games/Unreal Projects/Dispatch`，`main`，`211637638e96271d57f09041172bd56174c16db9`。
- 子任务/父级：`UI-DISPATCH-001` / `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`；不覆盖相邻物品数据、派遣/存档总项。
- 开工条件：新预览及现行 FolioHD 源图可读，用户确认先生成，已满足。
- 退出条件：给用户看两个空槽与整张空白底板合成预览；已生成，待用户评价。

## 工作区

- 开始时仅有上一轮阴影相关三个未提交文件：`WBP_CommissionDispatch.uasset`、`CommissionDeskPanCommandlet.cpp`、`CommissionDeskPanTests.cpp`；均未触碰。
- 本轮没有改 Git 跟踪文件、Widget、Texture 或正式 DataAsset。生成图仍在本机 `.codex/generated_images`，仅是预览，不作为项目运行资产。
- 未提交、未推送；现行 FolioHD 未覆盖。

## 完成与验证

- 设计：底部做无字、无图标的旧铜双空槽与中央隔板；交互按钮/文字以后仍由 UMG 单独绘制。
- 预览：透明装饰条 `C:/Users/DULEOLK/.codex/generated_images/01a0cd8e-d09b-7b03-88dc-48fd6804dd4b/exec-659efdb4-6cd4-4992-a1ac-5164a1f99f86.png`；整体合成图 `C:/Users/DULEOLK/.codex/generated_images/01a0cd8e-d09b-7b03-88dc-48fd6804dd4b/exec-e8456d62-c6e9-49d8-96d9-f72015f62532.png`。两图已目视检查，无物品列表/按钮文字，透明外沿存在。
- 四类状态：视觉设计候选待用户确认；代码/UE 资产未实施；自动测试未运行；UE 原生窗口人工验收未做。

## 未完成与风险

- 合成图由生成模型绘制，不能假定其 FolioHD 像素、alpha、按钮坐标和 UE 源图完全一致；实际接入须以现行 680×850 底板及 UMG 按钮位置重新定尺。
- 当前双槽内侧为深色，真实禁用/启用按钮的文字和状态仍需单独设计/验证。
- 无运行中 UE 工具或进程。

## 下一步

- 请用户确认槽位材质和整体比例；若认可，再以版本化素材接入 UE，保留旧 FolioHD 与旧 Widget 备份，并验证按钮点击区域和双视角画面。
- STATUS、IMPLEMENTATION_MATRIX 已同步；主要实施子任务仍为 `UI-DISPATCH-001`，父级不升级。
