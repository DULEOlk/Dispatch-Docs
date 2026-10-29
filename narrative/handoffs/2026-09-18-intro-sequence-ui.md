# 阶段交接：开场 CG、字幕与 Intro Sequence 设计记录

- 日期：2026-09-18
- 子任务：`DOC-UI-008`
- 父级：`DEMO-VIS-001`、`DEMO-EXP-001`
- 状态：文档与截图归档完成；代码、蓝图、声音、镜头和正式文本未修改

## 本轮成果

已保存并登记三张用户截图：

- `docs/narrative/ui/images/intro-slideshow-early-prototype-2026-09-18.png`
- `docs/narrative/ui/images/prison-arrival-subtitle-early-prototype-2026-09-18.png`
- `docs/narrative/ui/images/intro-sequence-config-2026-09-18.png`

已新增 [INTRO_SEQUENCE_UI.md](../ui/INTRO_SEQUENCE_UI.md)，记录：

- `WBP_IntroSlideshow` 的开场画面/Caption 容器；
- `WBP_PrisonArrivalSubtitle` 的独立抵达字幕职责；
- `BP_IntroSequence` 当前 9 张幻灯片、`Mus_cg`、淡入/淡出 1 秒和 `Auto Start` 未勾选；
- `FIntroSlideData` / `FIntroCameraBeat` 对插画、停留、声音、偏移/缩放镜头节拍、字幕打字机的现有能力；
- 跳过、暂停、输入锁定、字幕关系、混音和真实游戏窗口演出验收等未决问题。

## 不包含

本轮不确认正式开场剧本、不确认 9 张图是最终数量、不确认是否自动播放或可跳过，也没有修改任何 `.uasset`、C++、Widget、声音、镜头、输入规则或正式文本。

## 验证与下一步

- 图片归档：源文件与项目副本需以 SHA256 相等作为保存证据。
- 自动化测试：未运行；本轮无业务代码变更。
- 人工验收：未进行。
- 最小下一步：在获得实现授权后，做 2～3 张图的演出切片，包含一段偏移/缩放、caption 打字机、音乐、节拍音效和抵达字幕，再观察可读性、节奏与序列结束后的输入/过渡。
