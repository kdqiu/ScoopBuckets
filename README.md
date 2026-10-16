# ScoopBuckets

[![Tests](https://github.com/kdqiu/ScoopBuckets/actions/workflows/ci.yml/badge.svg)](https://github.com/kdqiu/ScoopBuckets/actions/workflows/ci.yml) [![Excavator](https://github.com/kdqiu/ScoopBuckets/actions/workflows/excavator.yml/badge.svg)](https://github.com/kdqiu/ScoopBuckets/actions/workflows/excavator.yml)

Personal [Scoop](https://scoop.sh) bucket for Windows command-line installer.

## Available manifests

| Manifest | Description | Version |
| --- | --- | --- |
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

## How do I contribute new manifests?

1. Enable GitHub Actions for the repository:
   - `Settings` → `Actions` → `General` → `Actions permissions`
   - Select `Allow all actions and reusable workflows`, then `Save`
2. Copy `bucket/app-name.json.template` to `bucket/<app-name>.json`
   and fill in the fields. See the [App Manifests](https://github.com/ScoopInstaller/Scoop/wiki/App-Manifests)
   wiki page for the full specification.
3. Verify the manifest locally with the helper scripts. Note that they all
   operate on the whole `bucket/` directory:

```pwsh
.\bin\formatjson.ps1
.\bin\checkver.ps1
.\bin\checkurls.ps1
```

4. Commit and push. CI will validate the manifest and open a PR.
5. To keep the bucket indexed on `https://scoop.sh`, add the repository
   topic `scoop-bucket`.

### Updating manifests

The `Excavator` workflow polls each manifest's `checkver` URL every 4 hours
and opens a Pull Request when a new version is detected. To do the same
locally:

```pwsh
.\bin\auto-pr.ps1
```

The upstream target defaults to `kdqiu/ScoopBuckets:master` and can be
overridden: `.\bin\auto-pr.ps1 -upstream <username>/<bucketname>:<branch>`.