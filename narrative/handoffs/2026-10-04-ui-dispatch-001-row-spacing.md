# 列表间距与背景修正

## 身份与范围

- 日期2026-10-04，Asia/Shanghai；项目 F:/Epic Games/Unreal Projects/Dispatch；main工作区。
- 子任务UI-DISPATCH-001；父级DEMO-COM-001、DEMO-VIS-001、DEMO-AUTH-001。
- 用户授权移除整个灰色列表区域并按参考放大列表、补间距。输入与作者入口齐备；退出成果为可原生预览的透明背景、四个较大行及留白。
- 不包含正式DA、背包线索筛选、派遣或存档业务。

## 工作区与实现

- 保留开工时已有大量修改和未跟踪素材；本轮修改作者commandlet、WBP_PreparationSelection和WBP_PreparationEntryRow。未提交/推送。
- 两Widget修改前备份 Saved/CodexBackups/PreparationRowSpacing_20261004_170518/。
- 只读探针确认父页列表472×292、Border_List alpha .13，而行根宽664。列表改坐标90,222，500×364；行484×91，内容83，行尾空8；图标84×76，标记列38，文字列358，名称摘要间距7。
- Border_List透明，按钮四态默认画刷NoDraw，保留自定义选中底纹与图标底纸。铅笔线画刷UV取y=9/24至15/24，高3，无需改用户PNG。
- 详情标题/装饰下移48，图框下移38，类型签下移32，描述下移36并缩高44；消息移至732并高14字体10。底部按钮746位置不动。

## 验证与状态

- 设计已确认；代码和资产已保存；DispatchEditor Win64 Development -NoUBA -MaxParallelActions=2构建退出0，日志Saved/Logs/UBT-RowSpacing.log。
- SpacePreparationRows为只读探针；加-Apply执行定向写入。RowSpacing-Probe.log记录修改前，RowSpacing-Apply.log报告SAVED并已成功编译两Widget；因环境DDC无可写节点进程退出1，不能记全通过。
- 独立重载只读核对日志Saved/Logs/RowSpacing-Readback.log。不等于原生视觉验收，未新增/运行自动UI测试。
- 原生窗口四行可读性、详情长文本是否够用及点击仍待验收；受影响历史证据维持需要重验。

## 下一步

- UE原生预览核对灰底已消失、四行间距、铅笔线、详情与按钮无重叠。较长详情以后可能需要自动换行/滚动，当前未扩展业务。
- 请勿重复执行-Apply：该入口的详情位移是增量。只读不带-Apply可安全重复。
- STATUS/MATRIX已同步，唯一进行中子任务不变。
