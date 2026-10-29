# 派遣页装饰素材与头像作者布局

## 目标

整理 \`DispatchPages\` 目录中的 8 张派遣页装饰贴图，让 Widget 与素材分开；三个派遣页的 \`Image_NPCPortrait\` 使用用户在设计器中保存的位置和尺寸。

## 范围

- 移动 8 张贴图到 \`Content/UI/Commission/Dispatch/Decorations/\` 的用途子目录，并由 Unreal 更新引用。
- 保留 \`WBP_DispatchPage_Preparation\`、\`WBP_DispatchPage_InProgress\`、\`WBP_DispatchPage_Completed\`、\`WBP_RelatedObservationRow\` 在原目录。
- 三个页面的头像作者布局基线为位置 \`(31.334627, 76.684608)\`、尺寸 \`(215.807251, 248.432007)\`；不再使用运行时额外放大或重排。
- 保留现有图片、文字、页面层级和数据读取；不修改正式委托 DataAsset、派遣结算、存档或剧情内容。

## 验收

- 资产移动后 8 张贴图均可加载，三个页面引用均解析到新路径，原目录只保留 4 个 Widget。
- 三个页面的头像槽位置和尺寸与作者基线一致，运行时只更新 Brush/Visibility。
- 派遣页面资产、头像绑定和记录本关联读取的真实 RHI 测试通过。

