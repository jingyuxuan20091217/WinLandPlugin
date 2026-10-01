# WinLandPlugin · WinIsland 社区插件市场

WinIsland（Windows 11 灵动岛）的社区插件仓库。

WinIsland 的「设置 → 插件市场」读取本仓库根目录的 **[index.json](./index.json)** 获取插件列表、版本和图标路径。
Logo 作为独立 PNG 文件存储，索引不内嵌图片数据；只有点「安装」时才下载对应的 `.lwp` 并校验 SHA-256。

清单由 GitHub Action 自动生成，**请勿手改 `index.json` 和下面的表格**。

## 插件清单

<!-- PLUGINS:START -->
| 图标 | 插件（Id） | 版本 | 作者 | 说明 |
| :---: | :--- | :---: | :--- | :--- |
| ![](plugins/clash-verge-island/logo.png) | **Clash 小岛**<br/>`clash-verge-island` | 1.0.1 | hit | 小岛显示 Clash Verge 当前节点和延迟，展开看模式、实时网速和四个网站的延迟，点一下弹出大卡片：切节点、切模式、测延迟。自动读取 Clash Verge 的连接参数，只需在它里面打开「外部控制」。 |
| ![](plugins/clipboard-island/logo.png) | **剪贴板岛**<br/>`clipboard-island` | 1.2.1 | luolangaga | 复制文字或图片时岛上弹提示；展开就是最近几条历史，点任意一条直接粘进你正在输入的窗口 |
| ![](plugins/weather-island/logo.png) | **天气小岛**<br/>`weather-island` | 1.2.0 | luolan | 小岛显示当前温度，展开看三天预报，点一下弹出大卡片：实况细节、24 小时逐时、7 天预报、日出日落与空气质量。配色与天气图标跟随岛体明暗，数据来自 Open-Meteo，不用注册也不用填密钥。 |
| ![](plugins/hw-monitor/logo.png) | **硬件监控**<br/>`hw-monitor` | 1.0.2 | WinIsland | 常驻硬件信息：小岛显示 CPU/GPU/网络/帧率，大岛显示前台窗口 FPS、CPU、GPU 与上下传速度 |
| ![](plugins/device-island/logo.png) | **设备插拔岛**<br/>`device-island` | 1.0.0 | luolangaga | 插 U 盘 / 耳机 / 手柄时岛上自动冒出；展开逐台列出当前设备，带「打开」「安全弹出」 |
<!-- PLUGINS:END -->

## 目录结构

```
WinLandPlugin/
├── index.json                  ← 自动生成：客户端数据源（含 sha256 / 体积 / logo 图片路径）
├── README.md                   ← 本文件（插件表格也由 Action 更新）
├── CONTRIBUTING.md             ← 投稿规范
├── tools/
│   ├── build-index.ps1         ← 扫描 plugins/ 生成 index.json + README 表格
│   └── submit-plugin.ps1       ← 作者本地打包 + 校验（一条命令产出待提交目录）
└── plugins/
    └── hw-monitor/             ← 一个插件一个目录，目录名 = 插件 id
        ├── plugin.json         ← 清单（元数据来源，必须与包内那份一致）
        ├── hw-monitor.lwp      ← 插件包（zip 格式，扩展名 .lwp）
        ├── logo.png            ← 可选，建议 240×240 透明 PNG
        └── README.md           ← 可选，详情页展示的说明
```

## 装插件

1. 打开 WinIsland → 设置 → **插件市场**，列表来自本仓库的 `index.json`；
2. 点「安装」→ 客户端下载 `.lwp`、校验 SHA-256、安装并启用；
3. 「详情」里有完整说明、许可证、主页与包哈希；启用/禁用/删除在「插件管理」。

## 市场源与镜像

**GitHub 是唯一源头**（投稿、Issue、Action 都在这边），下面几个地址只是分发用的镜像。
客户端默认按顺序尝试、命中即停，并记住上次成功的源；也可以在「插件市场 → 市场源」里手动指定：

| 源 | 地址 | 说明 |
| :--- | :--- | :--- |
| GitHub 官方 | `https://raw.githubusercontent.com/luolangaga/WinLandPlugin/main` | 内容最新；部分国内网络解析不了该域名 |
| **GitCode 国内镜像** | `https://api.gitcode.com/api/v5/repos/luolangaga/WinLandPlugin/raw` | <https://gitcode.com/luolangaga/WinLandPlugin> 自动同步本仓库，国内访问快 |
| jsDelivr CDN | `https://cdn.jsdelivr.net/gh/luolangaga/WinLandPlugin@main` | 海外 CDN，分支内容有缓存（更新会延迟） |
| jsDelivr 备用域 | `https://gcore.jsdelivr.net/gh/luolangaga/WinLandPlugin@main` | 同上，`cdn` 域不通时使用 |

路径拼接规则一致：`<源>/index.json`、`<源>/plugins/<id>/<id>.lwp`。

## 投稿插件

见 **[CONTRIBUTING.md](./CONTRIBUTING.md)**。最简流程：

1. 按 WinIsland 插件开发指南（`PLUGIN.md`，SDK 2.0）写好插件并打包成 `.lwp`；
2. `pwsh tools/submit-plugin.ps1 -SourceDir <构建输出目录> -Logo logo.png` 生成 `plugins/<id>/`；
3. Fork 本仓库 → 分支 `add-plugin-<id>` → 提交该目录 → 开 PR。

PR 合并后 Action 会自动重建 `index.json` 与本文件的插件表格，客户端刷新即可看到。

## index.json 格式（schema 1）

| 字段 | 说明 |
|------|------|
| `schema` | 清单格式版本，当前为 `1` |
| `repo` / `updated` / `count` | 仓库名 / 生成时间 / 插件数量 |
| `plugins[]` | 插件条目数组，见下表 |

单条插件：

| 字段 | 说明 |
|------|------|
| `id` | 插件 Id，与目录名、`plugin.json` 一致 |
| `name` / `version` / `description` / `author` | 展示信息 |
| `icon_glyph` | 没有 logo 时的 Segoe 图标字形 |
| `homepage` / `license` / `tags` | 主页 / 许可证 / 标签 |
| `api_version` / `min_host_version` | 兼容性要求，客户端据此禁用不兼容的插件 |
| `package` | 包在仓库内的相对路径 |
| `package_size` / `sha256` | 包大小与哈希，下载后强制校验 |
| `readme` | 说明文件相对路径（详情页按需拉取，列表阶段不请求） |
| `logo` | 独立 logo 图片的相对路径；没有图片时为 `null` |
| `updated_at` | 该插件目录最后一次提交时间 |

## 许可

本仓库自身的脚本与文档为 MIT；每个插件的许可见其 `plugin.json` 的 `license` 字段。
