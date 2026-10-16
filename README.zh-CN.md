# ScoopBuckets

简体中文 | [English](README.md)

个人使用的 [Scoop](https://scoop.sh) 软件源，面向 Windows 命令行安装器。

## 可用清单

| 清单 | 说明 | 版本 |
| --- | --- | --- |
| [aitiy](bucket/aitiy.json) | Aitiy — 集成鼠标手势、快捷键、截图与OCR 的桌面效率工具 | 1.0.10 |
| [centbrowser](bucket/centbrowser.json) | Cent Browser — 功能最丰富的 Chromium 内核浏览器 | 5.3.1184.25 |

## 如何安装

添加软件源，然后按清单名安装：

```pwsh
scoop bucket add ScoopBuckets https://github.com/kdqiu/ScoopBuckets
scoop install ScoopBuckets/centbrowser
```

卸载：

```pwsh
scoop uninstall centbrowser
```
