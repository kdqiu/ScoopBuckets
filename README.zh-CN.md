<div align="center">

# 个人 Scoop 软件源

简体中文 | [English](README.md)

一个自用的 [Scoop](https://scoop.sh) 软件源，面向 Windows。

</div>

## Scoop 是什么

Scoop 是一个 Windows 命令行软件安装器。你不需要在安装向导里一步步点选，
只要执行一条命令，软件就会被放到用户目录下一个自包含的文件夹中。

```pwsh
scoop install centbrowser
```

每个软件由一个称为 **manifest**（清单）的 JSON 文件描述，它告诉 Scoop 去哪里
下载、以及校验和应该是什么。由于 Scoop 会自行处理依赖并验证哈希，清单通常很
小，多数不到二十行。

清单存放在 **bucket**（软件源）中。`main` 和 `extras` 是官方源，安装 Scoop 时
已自动添加；其余的，包括本仓库，都是你自己添加的源。这样就能把自己的软件源
和公共源并列使用，而不改动官方条目。

## 可用清单

| 清单 | 说明 | 版本 |
| --- | --- | --- |
| [aitiy](bucket/aitiy.json) | 本地优先的桌面效率工具，集成鼠标手势、智能截图、OCR、贴图、选词工具栏与屏幕边缘触发 | 1.0.11 |
| [centbrowser](bucket/centbrowser.json) | Cent Browser — 功能最丰富的 Chromium 内核浏览器 | 5.3.1184.25 |

## 安装

```pwsh
scoop bucket add scoop-kdqiu https://github.com/kdqiu/scoop-kdqiu
scoop install scoop-kdqiu/centbrowser
```

卸载：

```pwsh
scoop uninstall centbrowser
```

## 编写清单

可以复制 [`bucket/centbrowser.json`](bucket/centbrowser.json) 作为起点，也可以
直接新建文件。最简结构是五个字段：

```json
{
    "version": "1.0.0",
    "description": "One line describing the app.",
    "homepage": "https://example.com/",
    "url": "https://example.com/app.zip",
    "hash": "the sha256 of app.zip"
}
```

### 核心字段

| 字段 | 必需 | 作用 |
| --- | --- | --- |
| `version` | 是 | 版本号，`scoop list` 会显示它。 |
| `description` | 是 | 单行描述，显示在搜索结果中。 |
| `homepage` | 是 | 项目主页。 |
| `url` | 是 | 下载地址。应指向具体文件，而不是「最新版本」页面。 |
| `hash` | 是 | 下载文件的 SHA256。不一致时 Scoop 会拒绝安装。 |
| `license` | 否 | 许可证标识或名称，如 `Freeware`。 |
| `notes` | 否 | 安装后显示的补充说明。 |
| `architecture` | 否 | 当不同架构的安装包不同时使用，如 `"64bit": { "url": …, "hash": … }`。 |
| `depends` | 否 | 需要预先安装的其他清单，如 `"depends": "extras/vcredist2022"`。 |
| `suggest` | 否 | 安装后推荐的相关软件，不会实际安装。 |
| `bin` | 否 | 需要加入 PATH 的可执行文件。 |
| `env_add_path` | 否 | 需要前置到 PATH 的目录。 |
| `shortcuts` | 否 | 开始菜单与桌面快捷方式，格式为 `[可执行文件, 名称]`。 |
| `persist` | 否 | 升级时保留的目录，如用户配置。 |
| `pre_install` | 否 | 解压文件前执行的 PowerShell，常用于结束正在运行的进程。 |
| `post_install` | 否 | 解压后执行的 PowerShell，如写入配置文件。 |
| `pre_uninstall` | 否 | 卸载前执行的 PowerShell。 |
| `post_uninstall` | 否 | 卸载后执行的 PowerShell。 |
| `uninstaller` | 否 | `scoop uninstall` 时执行的脚本。 |
| `checkver` | 否 | 在哪里查新版本，附一个提取版本号的 `regex`。 |
| `autoupdate` | 否 | 新版本发布时用于重写 `url` 与 `hash` 的模板。 |

### 更新与卸载

`checkver` 抓取一个 URL 并用正则提取最新版本号，`autoupdate` 再把它代入
`url` 并刷新哈希。请把 `checkver` 指向**以纯文本包含版本号**的页面——如果页面
是用 JavaScript 动态生成的，抓取时看不到版本号，正则很可能匹配到无关的数字。

`license` 优先使用 SPDX 标识符，如 `MIT`、`Apache-2.0`。`Freeware`、`Proprietary`、
`Shareware`、`Unknown` 只在没有合适标准标识符时作为兜底值使用。

## 保持版本稳定

软件装好后往往还会自己更新，或是被 Scoop 顺带升级。三条通道需要分别处理。

### 阻止软件自我更新

应用自带的更新器不受 Scoop 管理，一旦它把文件换了，Scoop 记录的哈希就与磁盘
上的实际内容不一致，下次 `scoop update` 会直接报错。可选手段按侵入性从低到高：

| 手段 | 适用场景 | 代价 |
| --- | --- | --- |
| 写配置项关闭自动检查 | 应用提供设置项 | 需知道具体配置键 |
| 删除独立更新程序 | 包内含 `updater.exe` 之类 | 文件被删后功能可能异常 |
| `pre_install` 拦截 | 应用启动时才检查 | 每次安装都重跑 |
| 阻断网络 | 应用只走 HTTP 检查 | 影响其他功能 |
| `pre_uninstall` 清理 | 自更新已发生 | 事后补救 |

**优先用配置项，其次才动文件。** 删除更新程序会让应用在某些路径下报错，
而配置项是可逆的、不会破坏安装完整性。

针对本仓库的两个清单，实际情况不同：

**Cent Browser** — Chromium 内核，官方 FAQ 明确：每 6 小时检查一次更新，
设置位于「其他设置」→「自动更新 Cent Browser」。它会写注册表
`HKEY_CURRENT_USER\SOFTWARE\CentBrowser`，便携版则另有
`centbrowserupdater.exe` 负责更新。由于 Chromium 系还支持组策略，
用 `post_install` 写入 `Local State` 的 `browser.enabled_labs_experiments`
或注册表策略键更稳妥；若使用安装版，还提供了 `--cb-auto-update` 参数。

**Aitiy** — 单文件应用，安装包内**只有 `Aitiy.exe` 与 `AitiyOcr.dll`，
不含任何独立更新程序**。更新检查由主程序自身发起（请求带
`Aitiy Update Check/1.0` 标识，指向 `aitiy.com`），并有 `UpdateTipProgress`
更新提示界面。因此「删除独立更新程序」这一手段对它不适用。

对 Aitiy 有效的做法是：在 `post_install` 中写入 `.aitiy` 配置目录下的
自动更新开关，使其不再检查；并在 `pre_install` 中结束已有进程，避免
它在安装期间自我替换。若某版本没有对应配置键，退而求其次的做法是
在 `post_install` 里清理更新临时目录并还原被替换的文件。

需要强调的是，这两种做法都依赖软件的内部实现，软件更新后可能失效——
写清单时应当把配置键写在 `post_install` 里，而不是预置在仓库中，
这样每次版本升级都会重新应用。

### 让 Bucket 自动抓取新版

仓库已内置 [Excavator](https://github.com/ScoopInstaller/Excavator) 工作流，
定义在 `.github/workflows/excavator.yml`：每 4 小时运行一次，读取每个清单的
`checkver` 与 `autoupdate`，检测到新版本就自动开Pull Request。

启用前需要在网页端完成两步：

1. **允许所有 Actions**
   `Settings` → `Actions` → `General` → `Actions permissions`
   → 选 `Allow all actions and reusable workflows` → `Save`

2. **授予工作流读写权限**
   `Settings` → `Actions` → `General` → `Workflow permissions`
   → 确保选中了 `Read repository contents and packages permissions` → `Save`

第二项容易被忽略：工作流声明了 `permissions: contents: write`，
若仓库默认设为只读，开 PR 时会因权限不足而失败。

也可以在 Actions 页面手动触发 `Excavator` 工作流验证一次，
这样能尽早发现配置问题，而不必等到定时任务触发。

### 禁止 Scoop 自动升级某个软件

若某个软件必须停留在当前版本（例如上游刚出的新版有bug），
用 `scoop hold`：

```pwsh
scoop hold centbrowser
```

这会在该应用的 `install.json` 中写入 `hold: true`，此后
`scoop update *` 会跳过它。需要恢复时：

```pwsh
scoop unhold centbrowser
```

`hold` 只影响 Scoop 的更新流程，不影响应用自身的更新器——
两者需要分别处理。
