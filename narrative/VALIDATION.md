# 验证记录与运行方式

此处保存 **2026-09-09 代码检查时实际运行的结果**。本次仅文档整理，未重跑游戏构建；未来代码变化后不得直接引用这些数字作为新代码通过证明。

| 验证 | 结果 | 原始日志（项目相对路径） |
|---|---|---|
| DispatchEditor Win64 Development | Succeeded / Target is up to date，退出 0 | UBT 当时输出；滚动日志在用户 AppData/Local/UnrealBuildTool/Log.txt，可能被后续构建覆盖 |
| Narrative + Story.Sources | 30 项：29 成功，1 失败；测试退出标记 -1，进程退出 1 | `Saved/Logs/Narrative-Resume-Baseline.log` |
| 相关玩法回归 | 56 项全部成功；退出 0 | `Saved/Logs/Narrative-Resume-Regression.log` |
| Director 与 Scene 回归 | 24 项全部成功；退出 0 | `Saved/Logs/Narrative-Resume-DirectorScene.log` |
| git diff --check -- Source | 退出 0 | 当次命令输出；有换行提示但无 whitespace error |

合计 110 项，109 成功、1 失败。失败：`Dispatch.Narrative.ActionExecutor.Binding`，`StoryActionExecutorTests.cpp:17` 构造 `UNarrativeRuntimeSubsystem` 未指定所需 GameInstance Outer，触发 `UObjectGlobals.cpp` ensure。不是已修复项。

每个 Test Completed 的结果与原日志 SHA256 另存于 [evidence/validation-2026-09-09.json](evidence/validation-2026-09-09.json)，即使滚动日志丢失也保留测试列表。静态审查见 [动作审查](evidence/review-action-events.md)、[运行时审查](evidence/review-runtime-bridge.md)；静态问题尚未运行专用复现测试。

## 构建命令

在项目根目录 PowerShell 运行：

```powershell
& 'F:\Epic Games\games\UE_5.6\Engine\Build\BatchFiles\Build.bat' DispatchEditor Win64 Development '-Project=F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' -WaitMutex -FromMsBuild
```

## 测试命令模板

选择下表过滤器；新运行使用新的日志名，保留旧失败记录。只有需要验证代码时才运行，不因读交接文档而自动启动。

```powershell
$narrativeTestFilter = 'Dispatch.Narrative+Dispatch.Story.Sources'
$narrativeLogFile = 'F:\Epic Games\Unreal Projects\Dispatch\Saved\Logs\Narrative-NextVerification.log'
& 'F:\Epic Games\games\UE_5.6\Engine\Binaries\Win64\UnrealEditor-Cmd.exe' 'F:\Epic Games\Unreal Projects\Dispatch\Dispatch.uproject' "-ExecCmds=Automation RunTests $narrativeTestFilter;Quit" -unattended -nop4 -nullrhi -nosplash -log -DDC-ForceMemoryCache "-abslog=$narrativeLogFile"
```

实际三组过滤器：

```text
Dispatch.Narrative+Dispatch.Story.Sources
Dispatch.Commission+Dispatch.Dialogue+Dispatch.Writing+Dispatch.Clue+Dispatch.Quest.Clue+Dispatch.UI.LetterWidget+Dispatch.Story.Flow+Dispatch.Story.DeliverLetter+Dispatch.Widget.PlayerNotebook.CluePage
Dispatch.Story.Director+Dispatch.Story.Debug+Dispatch.Story.Objective+Dispatch.Story.HUD+Dispatch.NPC.Scene
```

检查日志的 Test Completed、失败的 BeginEvents/EndEvents 和最终 TEST COMPLETE，而不只读末尾退出行。`-nullrhi` 自动化不证明鼠标焦点、视觉表现、天气与 NPC 地图行为。

当时沙箱内构建锁/Zen 路径初始化失败，后按正常权限流程在沙箱外完成验证。没有修改引擎脚本、绕过锁或终止其他任务进程。后续遇到同类问题保留错误，并使用正常权限流程。
