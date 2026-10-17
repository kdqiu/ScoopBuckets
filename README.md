# ScoopBuckets

[简体中文](README.zh-CN.md) | English

Personal [Scoop](https://scoop.sh) bucket for Windows command-line installer.

## Available manifests

| Manifest | Description | Version |
| --- | --- | --- |
| [aitiy](bucket/aitiy.json) | Local-first desktop productivity tool with mouse gestures, screenshots, OCR and screen edge triggers | 1.0.11 |
| [centbrowser](bucket/centbrowser.json) | Cent Browser — The Most Feature Rich Chromium | 5.3.1184.25 |

## How do I install these manifests?

Add the bucket, then install by manifest name:

```pwsh
scoop bucket add ScoopBuckets https://github.com/kdqiu/ScoopBuckets
scoop install ScoopBuckets/centbrowser
```

To uninstall:

```pwsh
scoop uninstall centbrowser
```
