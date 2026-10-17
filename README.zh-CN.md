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
