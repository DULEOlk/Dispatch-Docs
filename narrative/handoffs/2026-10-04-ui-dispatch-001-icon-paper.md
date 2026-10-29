# 列表图标底纸试换

## 身份与范围

2026-10-04，Asia/Shanghai；F:/Epic Games/Unreal Projects/Dispatch，main工作区。UI-DISPATCH-001，父级DEMO-COM-001、DEMO-VIS-001、DEMO-AUTH-001。用户提供旧纸素材并指出未选中菱形偏灰；授权原位替换图标底纸。输入齐备；退出成果为可预览的旧纸图标底与深棕未选中标记。不覆盖父页布局、正式数据和业务。

## 工作区

保留已有未提交修改；本轮只修改CommissionDeskPanCommandlet、WBP_PreparationEntryRow，新增ListStyle/T_ListIcon_WornPaper.png及uasset。原PNG复制自用户Downloads，未重绘或修改像素。行蓝图保存前备份于Saved/CodexBackups/PreparationIconPaper_时间戳目录，确切位置见IconPaper-Apply.log。未提交/推送。

## 完成与验证

Border_IconPaper使用用户Texture、白色乘色、padding7；Border_IconFrame透明、padding0；Text_MarkerIdle改线性深棕(.18,.09,.035,1)，选中态仍红色图片。物品图、行尺寸、父页未改。

Development构建退出0：Saved/Logs/UBT-IconPaper.log。作者命令-run=CommissionDeskPan -ApplyPreparationIconPaper -DDC-ForceMemoryCache编译保存成功：Saved/Logs/IconPaper-Apply.log；进程因环境DDC无可写节点退出1，不记工具全通过。git diff --check通过。未新增/运行UI测试，未原生视觉验收，相关旧证据需要重验。

## 未完成与下一步

用户素材1254×1254，中心抽样Alpha56/255、角落1/255，原样保留；在浅纸背景上会呈半透明纹理，不能凭黑底附件预期最终深浅。下一步UE原生预览复看。全部本轮构建/作者进程已结束，STATUS/MATRIX已同步，唯一主任务仍为UI-DISPATCH-001。
