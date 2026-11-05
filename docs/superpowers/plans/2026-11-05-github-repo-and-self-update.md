# Sapphire 上传 GitHub 与软件内自更新实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 将汉化后的 Sapphire 上传至公开仓库 `adgry/Sapphire-CN`，并将软件内自更新指向该仓库，配套本地一键发版脚本与 GitHub Actions 自动发版工作流。

**Architecture:** 在 `UpdateChecker.swift` 中集中管理 `adgry/Sapphire-CN` 更新源、缓存隔离、SemVer 3.x 版本比较、隔离属性清理与安装提权兜底；通过根目录 `VERSION`、`scripts/release.sh` 与 `.github/workflows/release.yml` 产出符合 `UpdateChecker` 校验规范的 `Sapphire-arm64.zip` GitHub Release 资产。

**Tech Stack:** Swift / AppKit / SwiftUI, Xcode (`xcodebuild`), Git / GitHub CLI (`gh`), GitHub Actions.

---

### Task 1: 改造 `UpdateChecker` 与应用内仓库链接并补充单元测试
- **Files:**
  - Modify: `Sapphire/App Settings/UpdateChecker.swift`
  - Modify: `Sapphire/App/OnboardingView.swift`
  - Modify: `Sapphire/App Settings/SettingsPanes.swift`
  - Modify: `Sapphire/Services/Music/MusicUtilities.swift`
  - Modify: `SapphireTests/AppsSettingsRevampTests.swift`
- **Steps:**
  1. 在 `UpdateChecker` 中定义 `updateRepository = "adgry/Sapphire-CN"` 及对应 API/Web URL，增加切换仓库自动清理缓存逻辑。
  2. 升级 `SapphireVersionOrdering`（`major >= 3` 使用标准数值比较），兼容空 Releases 数组返回 `.upToDate`。
  3. 在 `UpdateChecker.installAndRelaunch` 中加入 `xattr -cr` 清除隔离属性，以及无 `sapphireHelper` 时的管理员授权替换兜底。
  4. 更新 `OnboardingView.swift`、`SettingsPanes.swift`、`MusicUtilities.swift` 中的 GitHub 链接指向 `adgry/Sapphire-CN`。
  5. 在 `AppsSettingsRevampTests.swift` 中新增针对 `UpdateChecker.updateRepository`、`releasesAPIURL` 及 `3.x` 版本排序的单元测试。

### Task 2: 修正 `.gitignore`、创建一键发版脚本与 GitHub Actions 自动发版工作流
- **Files:**
  - Modify: `.gitignore`
  - Modify: `.gitattributes`
  - Delete: `.github/workflows/cla.yml`
  - Create: `VERSION`
  - Create: `scripts/release.sh`
  - Create: `.github/workflows/release.yml`
  - Modify: `README.md`
- **Steps:**
  1. 修正 `.gitignore`（保留 `Sapphire/Services/Gemini` 与 `Tools`，忽略 `/build/`、`DerivedData/`、`*.profraw`），清理根目录临时空文件并移除 `.github/workflows/cla.yml`。
  2. 创建 `VERSION`（初始值 `3.5`）、`scripts/release.sh`（一键涨版本号 + Release 构建 + ad-hoc 签名 + 打包 `Sapphire-arm64.zip` + 推送并创建 GitHub Release）和 `.github/workflows/release.yml`。
  3. 在 `README.md` 中补充仓库地址与后续版本更新发布指南。

### Task 3: 编译测试、创建并推送 `adgry/Sapphire-CN`、发布首个 `v3.5` 基准版本并真机验证
- **Steps:**
  1. 运行 `xcodebuild ... test` 验证编译 0 错误且新增/既有单元测试全部通过。
  2. 编译 `Release` 版 `Sapphire.app`，生成 `Sapphire-arm64.zip`，并验证其通过 `UpdateChecker.validateArchiveEntries` 校验。
  3. 初始化 Git 仓库并推送至新建公开仓库 `https://github.com/adgry/Sapphire-CN`，上传 `v3.5` 基准 Release（含 `Sapphire-arm64.zip`），实测客户端更新检查接口返回最新状态。
