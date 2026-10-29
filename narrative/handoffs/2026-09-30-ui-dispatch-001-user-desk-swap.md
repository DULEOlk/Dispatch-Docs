# UI-DISPATCH-001 用户桌面图原位替换交接

## 身份与范围

- 日期/时区：2026-09-30，Asia/Shanghai。
- 用户目标：把当前桌面背景换成其上传的 `C:/Users/DULEOLK/AppData/Local/Temp/codex-clipboard-7b0a180f-76a4-4640-b214-59a95d1f7e43.png` 看效果。
- 授权范围：只替换现行桌面 Texture 的源图并重导；不编辑图片像素，不改 Widget、裁剪、850 像素平移、卷宗、清单、三槽或其他数据/业务。
- 项目：`F:/Epic Games/Unreal Projects/Dispatch`，分支 `main`，开工 HEAD `11939c7`。
- 稳定子任务：`UI-DISPATCH-001`；父级 `DEMO-COM-001`、`DEMO-VIS-001`、`DEMO-AUTH-001`。不覆盖真实物品/背包/派遣/存档及总体验验收。
- 开工条件：用户 PNG 是 2172×724 24-bit RGB，和现纹理同尺寸；开工前已核对 Git 脏改、未跟踪目标文件及 UE 进程，满足。
- 用户可观察成果：两个 UE 1920×1080 视角渲染；实际原生 PIE 审美仍待用户确认。

## 工作区与实施

- 大量既有未提交 Widget、源码、纹理及旧资源删除原样保留，没有 reset/clean/提交/推送。
- 替换前 `T_DispatchDesk_DarkDecor.png` 与 `.uasset` 备份到 `Saved/CodexBackups/UserDeskSwap_20260930/`；重导工具另备份旧 UAsset 到 `Saved/CodexBackups/DarkDecorDeskReimport_20260930_130041/`。
- 用户 PNG 逐字节复制到 `Content/UI/Commission/Background/T_DispatchDesk_DarkDecor.png`；源与目标 SHA256 同为 `BF1F0F5A5CDBD1CA8E683D6918315DAA7F79B6C9444500CF762C6AA462BBA23D`。没有使用图像生成或像素编辑。
- 使用已存在的 `CommissionDeskPan -ReimportDarkDecorDesk` 分支原位重导同名 UI Texture；`WBP_CommissionDispatch.Image_PanDesk` 原画刷路径不变。未改源码、Widget、配置或 DA。

## 验证与四类状态

- 设计：用户提供确切图片，选择明确。
- 实施：源图和 UE Texture 已替换。
- 自动：独立 UE 进程 `Automation RunTests Dispatch.Widget.CommissionDispatch` 12/12 成功、退出 0；`Saved/Logs/UserDeskSwap-Focused.log`。包括资源绑定、尺寸、三槽平移/返回和四种分辨率渲染。未重跑宽 `Dispatch.Widget`，其本轮状态为需要重验。
- 人工：未在用户的原生 PIE/Standalone 窗口验收新颜色。
- 重导 `Saved/Logs/UserDeskSwap-Reimport.log` 有 `FactoryCreateFile` 和 `DarkDecorDesk REIMPORTED`，但全局 DDC 无可写节点使命令进程退出 1。独立加载/自动化结果与此分开报告。
- UE 渲染：`Saved/Automation/CommissionDeskPan/Dossier_1920x1080_UserDesk.bmp`、`Sheet_1920x1080_UserDesk.bmp`；平移后图的 SHA 与上版不同，已目视确认木纹/装饰改变。测试夹具中的大白块是未绑定委托/NPC 图片占位。
- 当前清单视角仅取到用户图的中间至右侧木纹，最右边墨水瓶/笔羽被固定 16:9 裁剪排除；没有擅自移动用户原图内容。
- 本轮无 UE/构建/测试进程或子代理在运行。

## 下一步

- 用户在 UE 原生窗口点交付槽往返两页，确认新桌面色调及是否需要让右侧道具入镜。
- 若要调整道具或视角，须另行明确范围；当前授权的“原图试入”不包含它们。
- 已同步更新 `STATUS.md`、`IMPLEMENTATION_MATRIX.md` 及 UI 记录。`UI-DISPATCH-001` 仍为主要进行中任务，父级总项不升级。
