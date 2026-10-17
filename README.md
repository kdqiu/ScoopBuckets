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

## Keeping versions stable

Once installed, software often keeps updating itself, or gets upgraded by
Scoop as a side effect. These are three separate channels and each needs its
own treatment.

### Stopping an app from updating itself

An app's own updater runs outside Scoop's control. Once it swaps files, the
hash Scoop recorded no longer matches what is on disk, and the next
`scoop update` fails outright. Available approaches, from least to most
invasive:

| Approach | When it applies | Cost |
| --- | --- | --- |
| Write the config key that disables update checks | The app exposes a setting | You need to know the key |
| Delete the standalone updater | The package ships an `updater.exe` | Removing it can break the app |
| Intercept from `pre_install` | The app only checks at startup | Runs on every install |
| Block the network | The app only checks over HTTP | Breaks other features |
| Clean up from `pre_uninstall` | The update already happened | After the fact |

**Prefer the config key, and only touch files as a last resort.** Deleting an
updater makes the app error on some code paths, while a config change is
reversible and leaves the installation intact.

The two manifests here differ:

**Cent Browser** — Chromium based. Its FAQ states that it checks for updates
every 6 hours, with the switch under "Other settings" → "Enable auto update of
Cent Browser". It keeps state in `HKEY_CURRENT_USER\SOFTWARE\CentBrowser`, and
the portable build relies on a separate `centbrowserupdater.exe`. Because the
Chromium family also honours group policy, `post_install` can write the
relevant policy key; the installer additionally accepts a
`--cb-auto-update` flag.

**Aitiy** — a single file app. The archive contains **only `Aitiy.exe` and
`AitiyOcr.dll`, with no standalone updater at all**. The update check is made by
the main binary itself, carrying an `Aitiy Update Check/1.0` user agent against
`aitiy.com`, and results in an `UpdateTipProgress` prompt. Deleting a
standalone updater therefore does not apply here.

For Aitiy, write the auto-update switch into the settings file under the
`.aitiy` directory from `post_install`, and stop any running instance from
`pre_install` so it cannot replace itself mid-install. If a given release ships
no such key, the fallback is to clear the update staging directory from
`post_install` and restore whatever was replaced.

Both approaches depend on internal implementation details and can break when
the app changes. Write them into `post_install` rather than storing the result
in this repository, so they are re-applied on every version bump.

### Letting the bucket catch up automatically

The repository already ships an
[Excavator](https://github.com/ScoopInstaller/Excavator) workflow in
`.github/workflows/excavator.yml`. Every four hours it reads each manifest's
`checkver` and `autoupdate`, then opens a Pull Request when it finds a newer
release.

Two settings must be changed on the web before it can run:

1. **Allow all actions**
   `Settings` → `Actions` → `General` → `Actions permissions`
   → select `Allow all actions and reusable workflows` → `Save`

2. **Grant the workflow write access**
   `Settings` → `Actions` → `General` → `Workflow permissions`
   → make sure `Read repository contents and packages permissions` is
   selected → `Save`

The second one is easy to miss. The workflow declares
`permissions: contents: write`, so if the repository defaults to read only, the
pull request step fails for lack of permission.

You can also trigger `Excavator` manually from the Actions page. That surfaces
configuration problems right away instead of waiting for the schedule.

### Stopping Scoop from upgrading an app

To pin an app to its current version, for example because a fresh upstream
release is broken:

```pwsh
scoop hold centbrowser
```

This writes `hold: true` into that app's `install.json`, after which
`scoop update *` skips it. To undo:

```pwsh
scoop unhold centbrowser
```

`hold` only affects Scoop's own update path. It has no effect on an app's own
updater, which is why the two need separate treatment.
