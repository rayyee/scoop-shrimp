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
| `workbuddy-cn` | 腾讯 WorkBuddy 国内版（codebuddy.cn） |

## 自动更新（GitHub Actions）

仓库带两个 workflow，都不需要任何 secret：

| 文件 | 触发 | 做什么 |
|---|---|---|
| `.github/workflows/excavator.yml` | 每 4 小时 + 手动 | 跑 Scoop 的 Excavator（等价于 `auto-pr.ps1 -Dir bucket -Push`）：对 `bucket/` 下**所有**清单执行 `checkver`，有新版就 autoupdate 出新的 `version` / `url` / `hash` 并直接提交到 `main` |
| `.github/workflows/ci.yml` | push / PR | 校验每个清单：JSON 合法性、`version` / `description` / `homepage` / `license` / `checkver` / `autoupdate` 齐全、`hash` 非空 |

要点：

- **新加清单不用改 workflow。** Excavator 扫的是整个 `bucket/` 目录，`checkver` + `autoupdate` 写对了就会被自动接管。
- **Excavator 直接推 `main`**（不建 PR），所以 CI 是唯一的刹车；清单字段缺失或 hash 为空会让 CI 红。
- 手动跑一次：仓库 Actions → **Excavator** → *Run workflow*；本地等价命令是 `scoop checkver -Dir bucket -u`（也可以 `-App <name>` 只跑一个）。
- 本地何时能看到新版：`scoop update` 会先 `git pull` 所有 bucket 再比版本，所以 Excavator 一提交，你下次 `scoop update` / `scoop status` 就能看到（`scoop update <app>` 只看已拉的清单，不会自己去问上游接口）。

## Hermes-CN-Desktop-portable

```powershell
scoop install shrimp/Hermes-CN-Desktop-portable
```

A “Hermes Agent CN Desktop” shortcut will appear in your Start Menu after installation.

Version tracking is driven by the manifest’s `checkver` and `autoupdate` blocks pointing at the upstream GitHub Releases, and bumped automatically by the bucket-wide Excavator workflow (see [自动更新](#自动更新github-actions)).

```powershell
scoop update Hermes-CN-Desktop-portable
scoop uninstall Hermes-CN-Desktop-portable
```

Uninstall removes the application directory, Start Menu shortcut, shim, and the `current` junction. The portable `data/` folder lives inside the version directory, so it is removed together with the app. No extra cleanup step is needed for the current portable packaging.

## workbuddy-cn

腾讯 WorkBuddy **国内版**，来自 `codebuddy.cn`。

> 与官方 `extras` 桶里的 `workbuddy`（`workbuddy.ai` 国际版）是两套独立产品，互不冲突：
>
> | | `workbuddy-cn`（本桶） | `extras/workbuddy` |
> |---|---|---|
> | 下载源 | `download.codebuddy.cn` | `download.codebuddy.ai` |
> | 主程序 | `WorkBuddy.exe` | `WorkBuddyAI.exe` |
> | 数据目录 | `%APPDATA%\WorkBuddy` | `%APPDATA%\WorkBuddy AI` |
> | 更新接口 | `www.codebuddy.cn/v2/update` | `www.workbuddy.ai/v2/update` |
>
> 因此两者可以同时安装，数据目录和快捷方式都不会互相覆盖。

```powershell
scoop install shrimp/workbuddy-cn
```

安装后开始菜单会出现 “WorkBuddy” 快捷方式。

### 安装方式

官方只提供 NSIS 安装包。清单把它当 7z 解（`#/dl.7z`），再由 `pre_install` 从 `$PLUGINSDIR\app-64.7z` 里取出真正的 Electron 应用，所以不需要管理员权限，也不会写入注册表或 Program Files。

### 数据目录

应用会把数据写在 `%APPDATA%\WorkBuddy` 和 `%USERPROFILE%\.workbuddy`。安装时这两个目录会被换成指向 `persist\workbuddy-cn` 的 junction，于是：

- 升级、卸载重装都不会丢登录态和配置；
- 卸载只摘掉 junction，`persist` 里的数据保留，重新安装自动接回；
- 如果这两个目录本来就有数据（例如以前用官方安装包装过），首次安装会把它们搬进 `persist`；若 `persist` 里已经有数据，则旧目录改名为 `*.original.<时间戳>` 备份，绝不覆盖任何一方。

彻底清理残留数据：

```powershell
scoop uninstall workbuddy-cn
Remove-Item "$env:SCOOP\persist\workbuddy-cn" -Recurse -Force
```

### 更新

```powershell
scoop update
scoop update workbuddy-cn
```

`checkver` 直接读官方更新接口 `https://www.codebuddy.cn/v2/update?platform=workbuddy-win32-x64-user`，一次拿到 `productVersion`、`url`、`sha256hash` 三个值，`autoupdate` 用它们拼 URL 并填 hash，不依赖 winget-pkgs 或任何第三方仓库。

清单的 bump 由 [Excavator](#自动更新github-actions) 每 4 小时自动完成，正常情况下你只需要 `scoop update`。两点需要注意：

1. **不会自动降级。** 官方接口在灰度发布或回滚期间可能返回比清单更低的版本（例如接口仍指向 `5.8.3.41334564`，而 CDN 上已有 `5.9.2.42136757`）。`checkver` 里做了版本比较，只有接口版本更高时才提示更新，避免 `scoop checkver -u` 把版本改小。
2. **不要对这份清单用 `scoop checkver -u -ForceUpdate`。** 强制更新时脚本会走“无需更新”分支，URL 为空。
3. hash 走 `autoupdate.hash.jsonpath` 从接口取，**不会下载 439 MiB 的安装包**，所以 CI 里跑一次只要几秒。

### 卸载

```powershell
scoop uninstall workbuddy-cn
```

只删除应用目录、快捷方式和两个数据 junction；`persist` 数据保留（见上）。
