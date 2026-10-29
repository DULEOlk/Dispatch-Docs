# Notebook bronze and silver tab split

## 身份与范围

- 日期/时区：2026-09-12 / Asia/Shanghai。
- 用户本次目标与最新修正：复刻已成功的金色页签拆层，先拆铜色物品与银色线索图标，点击功能后续再做。
- 已授权范围 / 仍待确认范围：已授权本地确定性逐像素拆分；未授权导入 Content、保存 Widget 资产或修改点击逻辑。
- 项目绝对路径、分支、HEAD：`F:\Epic Games\Unreal Projects\Dispatch`，`main`，`fd705ff`。
- 当前阶段及具体小任务：Notebook 页签素材准备；主体层、固定根部层和左移预览。

## 工作区

- 开始时已有的修改/未跟踪文件：剧情系统、Notebook 源码与 Widget/UI 资产均有既存差异；详见同期 `git status --short` 和 STATUS。
- 本轮实际修改的文件和作用：`Saved/CodexGenerated/NotebookTabs/SplitLayers/` 下新增通用拆分/校验脚本及六张 PNG；`docs/narrative/STATUS.md` 更新过程与结果。
- 用户资产、其他任务改动及保护说明：未覆盖 Downloads 源图；未修改或清理 Content 中既有 `.uasset`。最终核对新增看到 `BP_NPC_Test.uasset` 为 modified，来源不能可靠区分，已保留且不计入本轮成果。
- 是否提交/推送：否。

## 完成与验证

- 已实现行为：按金色页签同一参考切口，将每张源图拆为同画布 `Body`、`FixedRoot`，并生成 Body 左移 4px、Root 固定的 `HoverPreview`。
- 首次生成式编辑失败：输出被重采样并重绘，未复制进项目；随后按用户确认改用现有金色页签的确定性逐像素算法。
- 铜色输出尺寸：1024×580；银色输出尺寸：1666×944，均与各自源文件一致。
- 机器校验：两套 Body/Root 原位 SourceOver 叠放后，premultiplied RGBA 与源图差异像素均为 0、最大通道差均为 0；六张输出均非空。
- 视觉检查：切口沿右侧皮革折叠区，主体/固定根部覆盖正常，左移预览未见透明裂缝。
- 输出目录：`Saved/CodexGenerated/NotebookTabs/SplitLayers/`。
- SHA256：
  - `T_NotebookTab_Clue_Silver_Body.png` `A31DDB9109D050973A135009E6556A561505965C8D5151EA7E5C99A605ACBA55`
  - `T_NotebookTab_Clue_Silver_FixedRoot.png` `AA7AB499F1DACF3A54996F37E87BB1F9EFD5A136B0D6319245092973C7DF3277`
  - `T_NotebookTab_Clue_Silver_HoverPreview.png` `A6858812615D6055DF94D6704902303E9EB36822DD69E31F7601F15AA79E8834`
  - `T_NotebookTab_Inventory_Bronze_Body.png` `57B800F292F66F20D13D96298B341E8256C445E0D1AF201E3996587871F47F1C`
  - `T_NotebookTab_Inventory_Bronze_FixedRoot.png` `1924CAB78469F10F9584AAE2442757C90620981BAB8AE7B2536A22630E119AAA`
  - `T_NotebookTab_Inventory_Bronze_HoverPreview.png` `78C1D1C2C2EEAD7A9CFE24FD104FA9AC4ECB4857B13A49ECDCE980192031B97E`
- 人工验证：仅文件级视觉检查；尚未在 UMG DPI/缩放环境叠放。

## 未完成与风险

- 尚未导入六张 PNG 为 Texture2D，也未设置 UI Texture Group、Alpha/compression、mipmap 或 sRGB 导入参数。
- 尚未为物品与线索按钮创建 `Image_Body` / `Image_FixedRoot` 层和点击/悬浮状态。
- 银色画布比铜色大，UMG 接入时必须通过一致的 SizeBox/Brush 尺寸归一化，不能直接假定像素尺寸相同。
- 当前运行中的工具/进程/代理：全部结束。

## 下一步

- 下一个最小动作：用户确认后，把两套 Body/FixedRoot 导入现有 Tabs Content 目录，设置与金色相同的纹理参数，并在正式 Widget 中只做静态同坐标叠放验收。
- 后续验收：再复刻金色按钮的 Hover/Pressed/Selected 状态与切页行为，并跑 Notebook TabInteraction 回归。
- 需修改的蓝图/资产：预计为六张新 Texture2D 与 `WBP_PlayerNotebook`，本轮尚未执行。
- 已同步更新 STATUS：是。
