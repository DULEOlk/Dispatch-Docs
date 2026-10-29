# Inventory_UI 未引用素材清理

## 身份与范围

- 2026-10-23，Asia/Shanghai。UI-NOTEBOOK-LIST-001；父级DEMO-COM-001 / DEMO-COM-002 / DEMO-VIS-001。用户明确授权清理Inventory_UI并删除无引用素材。
- 仅Content/UI/Inventory_UI递归纹理。不开新玩法子任务，不改正式DA、Widget布局、派遣/报告事务、其他素材库。不提交或推送。
- 开工时已有4个QuestPage纹理删除、4个其他Widget修改和PNG素材库大批删除，均保持。不将这些已有变更报告为本轮删除。

## 设计与实现

- UE完整AssetRegistry同步扫描，检查硬、软、管理、Searchable引用；91纹理中30无资产引用。
- 另外核对Source/Config，保留Tabs/T_NotebookTab_Inventory_Bronze_FixedRoot_UI256与T_NotebookTab_Clue_Silver_FixedRoot_UI256（运行时LoadObject），以及Quest/委托UI_去黑边透明版_v5_补全右上角（编辑作者工具依赖）。纯Bronze旧版与新路径前缀相同，但无精确路径引用，按旧版删除。
- 27纹理通过UE EditorAssetLibrary逐个删除，无强删有引用资产。删除前保存原路径与SHA256并复制至Saved/CodexBackups/InventoryUnused_20261023/，manifest.json记录恢复路径；所有备份逐文件校验一致。8张当前源PNG保留。
- InventoryArtAudit20261023.json为删除前全量引用清单；InventoryArtClean20261023.log记录逐文件删除及INVENTORY_CLEAN_COMPLETE 27，退出0。

## 验证和退出

- 设计：用户已授权。实现：27件删除完成。自动：InventoryArtCleanVerify20261023.log，Dispatch.Widget.Notebook+Dispatch.Widget.QuestNotebook真实RHI执行中。人工：清理后原生目录与界面待用户复看。
- 成果：目录少27件未使用素材，现行纹理/代码动态素材保留；备份可恢复。
- 父级整体与历史准备清单失败未覆盖；其他已有改动不撤销。CURRENT/矩阵同步本步。下一步读取测试结果并记录。
最终退出：InventoryArtCleanVerify20261023.log 12/12成功，Controller错误0，进程退出0；27份备份哈希一致且原位置不存在。所有本轮进程结束；仅本轮27纹理删除，未提交/推送，未改动已有其他UI工作区变化。
用户追加保留0023/0024/0025/0026/0031/0032：从本轮备份恢复被删的0023/0025/0026，SHA256一致；0024/0031/0032本来就有引用未删除。六件均已在原路径确认存在，后续清理无引用也保留。本轮净删除24件；仅恢复原文件，不改实现、不新增测试、未提交推送。
