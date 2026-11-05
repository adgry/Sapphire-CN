# Sapphire 上传 GitHub（`adgry/Sapphire-CN`）与软件内自更新改造设计规格

## 1. 目标
1. 将已完成简体中文汉化的 `Sapphire` 项目完整初始化并推送至用户公开 GitHub 仓库 `https://github.com/adgry/Sapphire-CN`。
2. 改造应用内置的软件更新模块（`UpdateChecker` 及相关界面入口），将更新检查源与跳转链接统一指向 `adgry/Sapphire-CN`。
3. 建立“双重发版机制”（GitHub Actions 自动构建发布工作流 `.github/workflows/release.yml` + 本地一键编译发版脚本 `scripts/release.sh`），确保后续在 `adgry/Sapphire-CN` 更新版本后，已安装的 `Sapphire` 客户端能自动收到更新提醒（灵动岛实时活动、系统通知、“设置 → 关于”页面），并支持一键下载、校验、覆盖安装与自动重启。

## 2. 核心设计要点
1. **更新源切换与旧缓存清理（`UpdateChecker.swift`）**
   - 新增静态常量 `UpdateChecker.updateRepository = "adgry/Sapphire-CN"`，统一派生 `releasesAPIURL`、`releasesWebURL` 与 `repositoryWebURL`。
   - 启动初始化时校验 `SapphireReleasesCacheRepository`；若与当前 `updateRepository` 不一致，自动清空旧仓库遗留的 `ETag`、Releases 缓存及待安装更新状态，杜绝串源。
   - 空 Releases 列表兼容：若仓库暂无 Release，返回 `.upToDate` 而非报错。
2. **版本号比较增强（`SapphireVersionOrdering`）**
   - 对主版本号 `>= 3` 的版本采用标准数值语义化版本比较（`3.5 < 3.5.1 < 3.6 < 3.9 < 3.10 < 4.0`），避免旧十进制小数规则导致 `3.10 < 3.5`；对历史 `2.x` 版本保留原有比较逻辑以兼容全部既有单测。
3. **自签名更新包免拦截与权限兜底**
   - 解压更新包后自动移除 `com.apple.quarantine` 扩展属性（`QuarantineAttribute.remove` + `/usr/bin/xattr -cr`），防止未公证的个人 GitHub Release 构建在自动重启时被 macOS Gatekeeper 拦截报错。
   - 当安装目录当前用户不可写且未安装特权辅助工具 `sapphireHelper` 时，标准更新模式自动回退至 `osascript` 管理员授权替换应用包。
4. **双重自动发版流水线**
   - **本地一键发版脚本 `scripts/release.sh`**：一行命令 `./scripts/release.sh <新版本号> [更新说明]` 自动更新 `VERSION` 与 `project.pbxproj` 中的 `MARKETING_VERSION`、编译 `Release` 版 `Sapphire.app`、执行 `codesign --force --deep --sign -` 绑定 `Info.plist`、使用 `COPYFILE_DISABLE=1 zip -qryy` 生成通过 `UpdateChecker.validateArchiveEntries` 安全校验的 `Sapphire-arm64.zip`、提交推送代码并调用 `gh release create` 发布到 `adgry/Sapphire-CN`。
   - **GitHub Actions 工作流 `.github/workflows/release.yml`**：支持推送 `v*` 标签、修改根目录 `VERSION` 文件推送到 `main` 分支、或在网页端手动触发 `workflow_dispatch`，在云端自动完成构建、签名、打包并发布 GitHub Release。
5. **仓库卫生与 `.gitignore` 修正**
   - 从原仓库 `.gitignore` 中移除会误忽略核心源码的 `/Tools` 与 `/Sapphire/Services/Gemini` 规则，补充忽略 `/build/`、`DerivedData/`、`*.profraw`，移除上游专属的 `.github/workflows/cla.yml`。
