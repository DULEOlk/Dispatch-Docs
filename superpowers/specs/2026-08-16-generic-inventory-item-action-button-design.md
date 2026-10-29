# 通用背包物品操作按钮设计

## 目标

将背包详情页中仅面向信件命名的 `Button_ReadLetter` 改为通用物品操作入口，使同一个按钮以后可以根据当前物品显示并执行“查看”“装备”或“卸下”。本次只重构按钮入口，并保持已经可用的信件查看功能；装备系统不在本次范围内。

## 方案选择

采用一个动态按钮，而不是三个重叠按钮。动态按钮只有一套布局、点击绑定和可见性状态，不会出现多个按钮同时显示或互相拦截点击的问题。

代码使用当前操作状态区分行为：

- `None`：没有可执行操作，按钮折叠。
- `View`：当前选中物品是信件，按钮显示“查看”，点击后打开信件界面。
- `Equip`：为以后可装备物品预留，本次不产生该状态。
- `Unequip`：为以后已装备物品预留，本次不产生该状态。

## 命名

- UMG 按钮：`Button_ItemAction`
- UMG 按钮文字：`Text_ItemAction`
- 绑定函数：`BindInventoryItemAction`
- 刷新函数：`RefreshInventoryItemAction`
- 点击回调：`HandleItemActionClicked`

信件领域函数 `CanReadSelectedLetter` 和 `OpenSelectedLetter` 保留，因为它们只负责判断和打开信件，不承担通用按钮职责。

旧名称 `Button_ReadLetter`、`Text_ReadLetter` 和匿名名称 `Button_139` 不再作为运行时兼容入口，避免新旧命名继续混用。蓝图控件必须改名为新的正式名称并勾选 `Is Variable`。

## 数据流

1. 玩家选择背包格子。
2. 笔记本保存当前预览的信件定义。
3. 刷新函数根据当前选择解析操作状态。
4. `View` 状态显示按钮并把文字设为“查看”；`None` 状态折叠按钮。
5. 玩家点击按钮时，根据已经解析的操作状态分发到对应领域函数。
6. 若选择已经失效或信件界面无法创建，现有打开函数返回失败，不改变背包状态。

## 蓝图和点击层级

`WBP_PlayerNotebook` 中的按钮需要命名为 `Button_ItemAction`，内部文字需要命名为 `Text_ItemAction`。按钮保持 `Visible`；按钮内部文字、按钮装饰图片和可能覆盖按钮的非交互装饰图设置为 `Hit Test Invisible`，让鼠标事件到达按钮。

代码只负责控件绑定和行为，不修改按钮美术样式。

## 测试

- 自动化测试使用正式名称创建 `Button_ItemAction` 和 `Text_ItemAction`，不再依赖 `Button_139`。
- 普通物品对应 `None`，按钮折叠。
- 信件对应 `View`，按钮显示且文字为“查看”。
- 清除选择后状态回到 `None`，按钮折叠。
- 点击 `View` 状态仍通过 `OpenSelectedLetter` 打开配置的 `ULetterWidgetBase` 子类。
- C++ 编译必须通过，且源代码中不再保留旧按钮和旧回调名称。

## 非目标

- 本次不新增饰品定义字段。
- 本次不实现装备槽、装备状态持久化或装备效果。
- 本次不自动重做按钮美术和布局。
