# 投稿指南

感谢投稿！本仓库是 WinIsland 的插件市场，客户端「设置 → 插件市场」直接读取本仓库的 `index.json`。
你只需要提交一个 `plugins/<你的插件 id>/` 目录，清单由 Action 自动生成。

## 一、目录规范

```text
plugins/<id>/
  plugin.json      必填，和 .lwp 包内那份完全一致
  <id>.lwp         必填，插件包（zip 格式，扩展名 .lwp），一个目录只允许一个
  logo.png         建议提供，240×240 透明 PNG，≤ 1 MB（Action 在 index.json 中记录图片路径）
  README.md        可选，详情页展示的说明
```

### README.md 支持的 Markdown 语法

客户端自带一个轻量渲染器（不引入第三方控件），详情页支持：

`#`~`######` 标题、段落、**加粗**、*斜体*、`行内代码`、~~删除线~~、[链接](https://example.com)
（相对链接会解析到本仓库 `main` 分支上的对应文件）、无序/有序列表（含嵌套）、`>` 引用、围栏代码块、表格、`---` 分隔线。

**图片不渲染**：相对路径的图片在客户端无从解析，会退化成 `[图片: alt]` 文本。
需要展示截图请放到 `plugin.json` 的 `homepage` 指向的页面，或在 README 里用绝对链接指向图片所在网页。

## 二、plugin.json

与 WinIsland 的插件清单格式完全一致，额外字段会被忽略：

```json
{
  "id": "hw-monitor",
  "name": "硬件监控",
  "version": "1.0.0",
  "entry_dll": "HardwareMonitor.dll",
  "api_version": 2,
  "min_host_version": "2.0.0",
  "description": "一句话说明（列表里最多显示两行）",
  "author": "你的名字",
  "icon_glyph": "\uE950",
  "homepage": "https://github.com/you/hw-monitor",
  "license": "MIT",
  "tags": ["hardware", "monitor"]
}
```

校验规则（`tools/build-index.ps1` 会在 CI 里执行，任一不通过 PR 就不能合并）：

| 字段 | 规则 |
|------|------|
| `id` | `^[a-z0-9][a-z0-9-]{1,63}$`，且必须等于目录名 |
| `name` | 非空；建议 ≤ 12 个字 |
| `version` | 数字点分版本号（`1.2.3`），且与包内 `plugin.json` **完全相同** |
| `entry_dll` | 包内 `.dll` 文件名，不能含路径，不能是 `WinIsland.Core.dll` |
| `api_version` | 当前为 `2` |
| `min_host_version` | 可选，低于该宿主版本客户端会禁用安装 |
| `description` / `author` / `license` / `homepage` / `tags` | 可选（tags 建议 2-4 个） |

## 三、打包

用 WinIsland 仓库的打包脚本：

```powershell
pwsh tools/pack-plugin.ps1 -ProjectDir samples\HardwareMonitor -Configuration Release
# → samples/dist/hw-monitor.lwp
```

或直接用本仓库的提交助手（会用构建输出目录打 `.lwp`、过滤宿主自带的程序集、校验清单）：

```powershell
pwsh tools/submit-plugin.ps1 -SourceDir .\bin\Release\net10.0-windows10.0.26100.0\win-x64 `
                             -Logo .\logo.png -Readme .\README.md
```

包内**不要**包含：`WinIsland.Core.dll`、`Microsoft.WinUI.dll`、`Microsoft.WindowsAppRuntime*.dll`、
`Microsoft.Windows.SDK.NET.dll`、`WinRT.Runtime.dll`（宿主自带，带了会被警告，也可能被拒）。
自己的第三方依赖必须复制进包（工程里打开 `CopyLocalLockFileAssemblies`）。

## 四、提交 PR

```powershell
gh repo fork luolangaga/WinLandPlugin --clone
cd WinLandPlugin
git checkout -b add-plugin-<id>
# 放入 plugins/<id>/ 目录
git add plugins/<id>
git commit -m "feat: add <id> plugin"
git push origin add-plugin-<id>
gh pr create --title "Add plugin <id>" --body "插件说明 / 截图 / 测试过的宿主版本"
```

PR 里请顺带说明：

- 插件干什么、怎么用；
- 测试过的 WinIsland 版本与 Windows 版本；
- 是否需要管理员权限（例如读 ETW 的帧率）；
- 是否访问网络、读写哪些文件（数据来源要透明）。

## 五、审核标准

- 能正常安装、启用、禁用、卸载，不残留后台进程/文件（WinIsland 的 `plugin.<id>.disabled` 机制要尊重）；
- 不把宿主自带的运行时打进包；
- 不越权：只用自己的设置键（SDK 会自动加 `<id>.` 前缀）、不修改其他插件的数据；
- 说明与行为一致，许可证明确；
- 版本号递增：更新已有插件时同时改 `plugin.json` 的 `version` 并重新打包。

## 六、更新已有插件

同名目录覆盖即可，但 `version` 必须提高（客户端按版本判断「可更新」）。
Action 会重新计算 sha256，用户端刷新市场就能看到更新。
