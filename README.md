<div align="center">

# Personal Scoop Bucket

[简体中文](README.zh-CN.md) | English

A personal [Scoop](https://scoop.sh) bucket for Windows.

</div>

## What is Scoop?

Scoop is a command-line installer for Windows. Instead of walking through a
wizard, you run one command and the software lands in a self-contained
directory under your user profile.

```pwsh
scoop install centbrowser
```

A package is described by a JSON file called a **manifest**, which tells Scoop
where to download the release and what its checksum should be. Because Scoop
resolves dependencies and verifies hashes for you, a manifest is small — most
of them are under twenty lines.

Manifests live in **buckets**. `main` and `extras` are the official ones and
are added automatically; everything else, including this repository, is a
bucket you add yourself. That makes it easy to keep your own software source
next to the shared ones without touching the official entries.

## Available manifests

| Manifest | Description | Version |
| --- | --- | --- |
| [aitiy](bucket/aitiy.json) | Local-first desktop productivity tool with mouse gestures, screenshots, OCR and screen edge triggers | 1.0.11 |
| [centbrowser](bucket/centbrowser.json) | Cent Browser — The Most Feature Rich Chromium | 5.3.1184.25 |

## Install

```pwsh
scoop bucket add scoop-kdqiu https://github.com/kdqiu/scoop-kdqiu
scoop install scoop-kdqiu/centbrowser
```

To uninstall:

```pwsh
scoop uninstall centbrowser
```

## Writing a manifest

Copy [`bucket/centbrowser.json`](bucket/centbrowser.json) as a starting point,
or write the file from scratch. The minimal shape is four fields:

```json
{
    "version": "1.0.0",
    "description": "One line describing the app.",
    "homepage": "https://example.com/",
    "url": "https://example.com/app.zip",
    "hash": "the sha256 of app.zip"
}
```

### Core fields

| Field | Required | Purpose |
| --- | --- | --- |
| `version` | yes | Version string shown by `scoop list`. |
| `description` | yes | One line, shown in search results. |
| `homepage` | yes | Project page. |
| `url` | yes | Download location. Point at a specific file, not a "latest" page. |
| `hash` | yes | SHA256 of the downloaded file. Scoop refuses to install on a mismatch. |
| `license` | no | License identifier or name, e.g. `Freeware`. |
| `notes` | no | Extra lines shown after installation. |
| `architecture` | no | Use when a build differs per architecture, e.g. `"64bit": { "url": …, "hash": … }`. |
| `depends` | no | Other manifests required first, e.g. `"depends": "extras/vcredist2022"`. |
| `suggest` | no | Related apps offered after install. Does not pull anything in. |
| `bin` | no | Executables to expose on the PATH. |
| `env_add_path` | no | Directories to prepend to the PATH. |
| `shortcuts` | no | Start Menu and desktop entries, as `[executable, name]` pairs. |
| `persist` | no | Folders kept across upgrades, e.g. user settings. |
| `pre_install` | no | PowerShell run before files are laid down. Use it to stop a running instance. |
| `post_install` | no | PowerShell run after extraction, e.g. to write config files. |
| `pre_uninstall` | no | PowerShell run before removal. |
| `post_uninstall` | no | PowerShell run after removal. |
| `uninstaller` | no | Script to run on `scoop uninstall`. |
| `checkver` | no | Where to find new versions, plus a `regex` to extract the number. |
| `autoupdate` | no | Template for rewriting `url` and `hash` on a new release. |

### Update and removal

`checkver` fetches a URL and applies a regex to pull out the latest version.
`autoupdate` then substitutes it into `url` and refreshes the hash. Point
`checkver` at a page that contains the version as literal text — if the page
builds it in JavaScript, the fetch will not see it and the regex may match
unrelated digits instead.

For `license`, prefer an SPDX identifier such as `MIT` or `Apache-2.0`.
`Freeware`, `Proprietary`, `Shareware` and `Unknown` are fallbacks for when no
standard identifier applies.
