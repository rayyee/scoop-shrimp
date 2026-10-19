# scoop-shrimp

A tiny Scoop bucket. Short and sharp.

## Add this bucket

```powershell
scoop bucket add shrimp https://github.com/rayyee/scoop-shrimp
```

## Available apps

| App | Description |
|-----|-------------|
| `Hermes-CN-Desktop-portable` | Hermes Agent 中文社区桌面版便携版 |

## Install

```powershell
scoop install shrimp/Hermes-CN-Desktop-portable
```

A “Hermes Agent CN Desktop” shortcut will appear in your Start Menu after installation.

## Update

```powershell
scoop update
scoop update Hermes-CN-Desktop-portable
```

Version tracking is driven by the manifest’s `checkver` and `autoupdate` blocks pointing at the upstream GitHub Releases. No external CI is required.

## Uninstall

```powershell
scoop uninstall Hermes-CN-Desktop-portable
```

Uninstall removes the application directory, Start Menu shortcut, shim, and the `current` junction. The portable `data/` folder lives inside the version directory, so it is removed together with the app. No extra cleanup step is needed for the current portable packaging.
