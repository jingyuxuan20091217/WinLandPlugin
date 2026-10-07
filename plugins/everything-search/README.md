# Everything 搜索 — WinIsland 插件

把 [Everything](https://www.voidtools.com/) 装进 WinIsland（WinLand）任务栏岛：点一下弹出搜索卡片，输入即搜，回车直接打开。

## 功能

| 能力 | 说明 |
| --- | --- |
| **岛上胶囊** | 显示当前状态；点一下弹出搜索卡片 |
| **实时搜索** | 输入即搜（70 ms 防抖），结果列表就地刷新 |
| **键盘操作** | `↑` / `↓` 选择，`回车` 打开，`Ctrl+回车` 在资源管理器中定位 |
| **鼠标操作** | 直接点结果打开；`Esc` 或点击卡片外关闭 |
| **焦点移交** | 打开文件后卡片自动关闭，焦点交给新窗口 |
| **最近搜索** | 记住搜索过的关键词，空状态下点一下即可复用 |
| **拖放搜索** | 把文件拖到岛上，用「在 Everything 中查找」直接搜同名文件 |
| **跟随主题** | 配色跟随岛体明暗自动切换 |

## 安装

1. 安装并**运行** [Everything](https://www.voidtools.com/)（本插件通过它检索，自身不建索引）
2. 在 WinIsland「设置 → 插件市场」里安装本插件，或把 `.lwp` 拖进插件目录
3. 重启 WinIsland

插件不需要管理员权限，不联网，只读取 Everything 的检索结果。

## 使用

点岛上胶囊打开卡片 → 输入关键词 → `↑`/`↓` 选择 → `回车` 打开。

| 快捷键 | 作用 |
| --- | --- |
| `回车` | 用默认程序打开选中的文件 |
| `Ctrl+回车` | 在资源管理器中定位 |
| `↑` / `↓` | 在结果间移动 |
| `Esc` / 点击卡片外 | 关闭卡片 |

打开文件后卡片会**自动关闭**，把焦点交给新打开的窗口，所以不需要再点一下。

## 两种连接方式

插件优先使用 Everything 官方 SDK，缺失时自动退回窗口消息，**无需任何配置**：

| 后端 | 条件 | 能力 |
| --- | --- | --- |
| **官方 SDK** | 插件目录下存在 `Everything64.dll` | 可匹配**文件名和路径** |
| **窗口消息** | 默认，只要 Everything 在运行 | 仅匹配**文件名** |

想要路径匹配，就把 Everything 安装目录下的 `Everything64.dll` 复制到插件目录，然后重启 WinIsland。

## 常见问题

**搜索不到结果？**
确认 Everything 正在运行（托盘有图标）。插件的设置页会显示当前连接状态。

**只能按文件名搜，不能按路径搜？**
说明在用窗口消息后端。把 `Everything64.dll` 放进插件目录即可启用路径匹配。

**卡片里打不了字？**
需要 WinIsland 1.1.3 或更高版本 —— 输入依赖宿主的输入会话机制。

## 兼容性

- WinIsland / WinLand **1.1.3+**（宿主 SDK 2.4.0）
- Windows 10 1809+ / Windows 11
- 已在 Windows 11 (26100)、2560×1600 @150% 上测试

## 实现要点

- **输入由宿主承担。** 宿主 1.1.3 起提供输入会话：卡片里的文本控件获得焦点时，宿主会临时解除浮层的 `WS_EX_NOACTIVATE` 并抢到前台，因此 `TextBox` 可以直接打字。插件不自己创建任何窗口。
- **UI 用代码构建而非 XAML。** 插件程序集解析不了自己生成的 XAML（宿主的资源索引里没有 XBF 文件）。
- **结果行模板用 `XamlReader.Load`。** `DataTemplate` 没有工厂构造函数，`LoadContent` 也不是虚方法；模板里只出现框架类型和绑定。
- **不打包宿主程序集。** `WinIsland.Core.dll`、`Microsoft.WinUI.dll`、`WinRT.Runtime.dll`、`Microsoft.Windows.SDK.NET.dll` 都由宿主提供。
- **查询在后台线程执行。** Everything 的检索会阻塞，不能放在 UI 线程上。

## 项目结构

```
src/EverythingSearch/
├── plugin.json                     # 插件清单
├── EverythingSearchPlugin.cs       # 入口：岛上胶囊 + 打开卡片 + 拖放
├── IslandPalette.cs                # 跟随岛体明暗的共享画刷
├── SearchHistory.cs                # 最近搜索的持久化
├── ShellActions.cs                 # 打开、定位
├── Everything/                     # 两个可互换的检索后端
│   ├── IEverythingClient.cs
│   ├── EverythingSdk.cs
│   ├── EverythingSdkClient.cs
│   ├── EverythingIpcClient.cs
│   └── EverythingService.cs
└── Views/
    ├── IslandLauncherView.cs       # 岛上的胶囊
    ├── SearchCardView.cs           # 搜索卡片
    └── EverythingSettingsPage.cs   # 设置页
```

## 许可

MIT
