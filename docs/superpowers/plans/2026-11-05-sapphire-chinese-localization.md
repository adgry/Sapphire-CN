# Sapphire 全项目纯中文汉化实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:dispatching-parallel-agents / superpowers:subagent-driven-development to implement independent domain tasks concurrently after Task 1 & Task 2 establish the build & model foundation. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 Sapphire 全项目的全部用户可见界面、系统权限/扩展配置、设置搜索引擎与项目 Markdown 文档直接汉化为纯简体中文，同时保持配置存储兼容性与自动化测试全部通过。

**Architecture:** 先修复公开构建占位配置并将工程默认开发区域切换为 `zh-Hans`，完成数据模型显示属性解耦与 `SettingsIndex` 中英双模搜索引擎升级；随后按严格不重叠的目录所有权并行汉化设置中心、灵动岛/实时活动、小组件、后台服务/子模块及项目文档；最后执行残留英文扫描、全量编译与 `SapphireTests` 测试验证。

**Tech Stack:** Swift 5 / SwiftUI / AppKit, Xcode (`xcodebuild` / `XCTest`), Property Lists (`Info.plist`), Storyboard, Markdown.

**Spec:** `docs/superpowers/specs/2026-11-05-sapphire-chinese-localization-design.md`

## Global Constraints

- **禁止新增或重命名 `Sapphire/` 下的 `.swift` 文件**：`Sapphire` 与 `abt` 共享 `PBXFileSystemSynchronizedRootGroup` 例外列表，所有修改必须在原文件就地完成。
- **保留源码注释与内部调试日志不动**：不翻译 `//`、`/* */`、`///` 注释，不修改 `Logger`、`os_log`、`print`、`NSLog` 中的内部日志文本。
- **严禁破坏持久化与底层协议**：不修改 `UserDefaults` / JSON / Plist 键名、`Codable` 枚举的 `case` 名与 `rawValue`、URL Scheme、XPC 服务名、AppleScript 脚本代码、Shell 命令参数、HTTP API 路径/JSON 字段、硬件/传感器标识符。
- **统一核心术语**：保留 `Sapphire`、`AirDrop`、`Touch ID`、`Face ID`、`Apple Music`、`Spotify`、`TIDAL`、`Gemini`、`iMessage`；`Notch` → `灵动岛`、`Floating Island` → `悬浮灵动岛`、`Live Activity` → `实时活动`、`Widget` → `小组件`、`File Shelf` → `文件暂存架`、`Snap Zones` → `窗口磁吸分区`、`Dock` → `程序坞`、`Finder` → `访达`、`Trash` → `废纸篓`、`Caffeinate / Keep Awake` → `保持唤醒`、`Eye Break` → `护眼休息`、`Focus Session` → `专注时段`、`App Lock` → `应用锁`、`Nearby Share` → `附近分享`。

## Review Focus

1. **中文单字与中间词搜索失效**：用户输入 `"电池"`、`"音量"`、`"磁吸"`、`"锁"` 等无空格中文词搜索设置时，若仅按空白切词+词首匹配会返回空结果——必须在 `SettingsIndexTests` 中固定中文单字、双字词及自然问句检索断言。
2. **持久化枚举反序列化损坏**：若误改 `SettingsModel.swift` 中 `Codable` 枚举的 `rawValue`（如 `"Continuity"`），会导致已保存配置解码失败且 `SettingsIndex.applying` 抛错——必须保留 `rawValue` 不变并仅汉化 `displayName`/`label`，通过 `SettingsStorageTests` 与 `SettingsIndexTests` 验证。
3. **字符串插值语序生硬或拼接断裂**：如 `"\(count) items"`、`"Every \(mins) min"`、`"Requires Sapphire \(tier)"` 需改为符合中文语序的 `"共 \(count) 项"`、`"每 \(mins) 分钟"`、`"需要 Sapphire \(tier)"`，避免中英夹杂。
4. **`NSLocalizedString` 无 `.strings` 表导致回退英文 Key**：如 `Nearby/ShareViewController.swift` 与 `NearbyTransferLiveActivityView.swift` 中的 `"NFiles"`，必须直接改为中文格式化字符串（如 `"\(count) 个文件"`），不能留裸 `"NFiles"` 键名。
5. **测试套件硬编码英文标题断言失败**：`SapphireTests` 中若有断言旧英文标题或助手句式的测试，需同步验证并更新为中文预期（同时保留对英文底层 `key` / `rawValue` 的断言）。

---

### Task 1: 工程区域配置、公开扩展占位修复与 `Info.plist` / `Main.storyboard` 汉化

**Files:**
- Modify: `Sapphire.xcodeproj/project.pbxproj`
- Modify: `Sapphire/App/Info.plist`
- Modify: `Sapphire/App/Base.lproj/Main.storyboard`
- Modify: `PublicExtensionStubs/Camera/Info.plist`
- Modify: `PublicExtensionStubs/FinderSync/Info.plist`
- Modify: `PublicExtensionStubs/FileProvider/Info.plist`
- Modify: `PublicExtensionStubs/AudioDriver/Info.plist`
- Modify: `SapphireAndroidWidgets/Info.plist`
- Modify: `SapphireHelper/Info.plist`
- Modify: `com.shariq.sapphireHelper/Info.plist`

**Interfaces:**
- Consumes: 无
- Produces: 可直接通过 `xcodebuild -project Sapphire.xcodeproj -scheme Sapphire` 编译并运行 `SapphireTests` 的工程配置；默认开发区域为 `zh-Hans`。

- [ ] **Step 1: 更新 `Sapphire.xcodeproj/project.pbxproj` 区域与 `SapphireFileProvider` 公开占位配置**
  - 将 `developmentRegion = en;` 改为 `developmentRegion = "zh-Hans";`，并在 `knownRegions` 中加入 `"zh-Hans"`。
  - 将 `SapphireFileProvider` 目标（`F2DE0000000000000000*`）对齐 `SapphireFinderSync` 的公开占位方式：`F2DE000000000000000002BB` 指向 `PublicExtensionStubs/FileProvider/FileProviderExtension.swift`（`sourceTree = SOURCE_ROOT`），`Sources` 构建阶段仅保留 `F2DE000000000000000013BB`，Debug/Release 配置中的 `INFOPLIST_FILE` 设为 `PublicExtensionStubs/FileProvider/Info.plist`，`CODE_SIGN_ENTITLEMENTS = ""`。

- [ ] **Step 2: 汉化全部 `Info.plist` 与 `Main.storyboard` 用户可见描述**
  - `Sapphire/App/Info.plist`：汉化 `CFBundleTypeName`（`磁盘映像`、`归档文件`、`Sapphire 程序坞布局`）、`NS*UsageDescription` 全部权限说明、`UTTypeDescription`（`Sapphire 设置备份`、`Sapphire 程序坞布局`、`Sapphire 启动台项目`、`RAR 归档文件`）、`NSServices` 菜单项（`使用 Sapphire 拍照`、`使用 Sapphire 扫描文稿`），并设置 `CFBundleDevelopmentRegion` 为 `zh-Hans`。
  - 其余扩展 `Info.plist`：更新 `CFBundleDevelopmentRegion` 为 `zh-Hans`，并将 `CFBundleDisplayName` / `NSHumanReadableCopyright` / 扩展显示名（如 `Phone` → `手机`、`Sapphire Android Widgets` → `Sapphire Android 小组件`）汉化。
  - `Sapphire/App/Base.lproj/Main.storyboard`：`title="Main Menu"` → `title="主菜单"`，`title="Window"` → `title="窗口"`。

- [ ] **Step 3: 验证主工程与测试目标编译通过**
  - Run: `xcodebuild -project Sapphire.xcodeproj -scheme Sapphire -configuration Debug -destination "platform=macOS" ENABLE_DEBUG_DYLIB=NO CODE_SIGNING_ALLOWED=NO CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO build-for-testing`
  - Expected: `** TEST BUILD SUCCEEDED **`

---

### Task 2: 核心模型显示层汉化与 `SettingsIndex` / `SettingsAssistant` 中文检索升级

**Files:**
- Modify: `Sapphire/Models/SettingsModel.swift` 及 `Sapphire/Models/` 下其余含 UI 文案的模型文件（不改 `SettingsFields+Generated.swift`）
- Modify: `Sapphire/App Settings/SettingsIndex.swift`
- Modify: `Sapphire/App Settings/SettingsIndex+Generated.swift`
- Modify: `Sapphire/App Settings/SettingsAssistant.swift`
- Test: `SapphireTests/SettingsIndexTests.swift`

**Interfaces:**
- Consumes: Task 1 可编译测试工程
- Produces: 所有 `SettingsSection` 与模型枚举的中文 `label` / `displayName` / `shortDescription` / `detail`；支持中文单字/词组/自然问句及英文键名搜索的 `SettingsIndex.search(_:limit:)`。

- [ ] **Step 1: 在 `SapphireTests/SettingsIndexTests.swift` 中编写中文搜索与助手意图失败测试**
  - 新增测试验证中文标题搜索（如 `"全屏活动可见性"` → `"fullScreenActivityVisibility"`、`"低电量"` → `"lowBatteryNotificationPercentage"`、`"波形"` → `"waveformBarCount"`）、中文日常问句搜索（如 `"全屏时怎么隐藏灵动岛？"`、`"开机自动启动"`、`"菜单栏显示 CPU"`、`"自动解压压缩包"`）以及 `SettingsAssistant.isQuestion("怎么开启歌词？") == true`、`SettingsAssistant.impliedToggle(in: "关闭额外调暗", title: "额外调暗") == false`。

- [ ] **Step 2: 运行 `SettingsIndexTests` 确认新中文测试按预期失败**
  - Run: `xcodebuild -project Sapphire.xcodeproj -scheme Sapphire -destination "platform=macOS" ENABLE_DEBUG_DYLIB=NO CODE_SIGNING_ALLOWED=NO CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO -only-testing:SapphireTests/SettingsIndexTests test`
  - Expected: 新增中文断言 FAIL。

- [ ] **Step 3: 升级 `SettingsIndex.swift`、`SettingsIndex+Generated.swift`、`SettingsAssistant.swift` 并汉化 `Sapphire/Models/*` 显示文案**
  - 在 `Sapphire/Models/*` 中汉化全部 `displayName`、`label`、`shortDescription`、`detail`、默认预设名称等用户可见文本（严禁改动 `Codable` `rawValue` 与持久化字段名；`SettingsSection.searchTokens` 保留英文词并追加中文近义词）。
  - 在 `SettingsIndex.swift` 中实现拉丁词干 + CJK 子串/双字词混合检索，补充中文同义词、停用词与 `parseBool` 中文布尔词，汉化 `ChangeError.description`。
  - 在 `SettingsIndex+Generated.swift` 中将全部 `title` 与 `detail` 翻译为简体中文。
  - 在 `SettingsAssistant.swift` 中汉化状态与错误提示，并在 `isQuestion` / `impliedToggle` 中加入中文规则。

- [ ] **Step 4: 重新运行 `SettingsIndexTests` 与 `SettingsStorageTests` 验证全部通过**
  - Run: `xcodebuild -project Sapphire.xcodeproj -scheme Sapphire -destination "platform=macOS" ENABLE_DEBUG_DYLIB=NO CODE_SIGNING_ALLOWED=NO CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO -only-testing:SapphireTests/SettingsIndexTests -only-testing:SapphireTests/SettingsStorageTests test`
  - Expected: `** TEST SUCCEEDED **`

---

### Task 3: 设置中心通用组件与独立子面板汉化（不含 `SettingsPanes.swift`）

**Files:**
- Modify: `Sapphire/App Settings/` 下除 `SettingsIndex.swift`、`SettingsIndex+Generated.swift`、`SettingsAssistant.swift`、`SettingsPanes.swift` 外的全部 23 个 `.swift` 文件（含 `SettingsSidebar.swift`、`SettingsView.swift`、`SettingsComponents.swift`、`OverviewSettingsView.swift`、`DisplaySettingsView.swift`、`AppStorageModels.swift`、`InstalledAppUpdates.swift`、`UpdateChecker.swift`、`LayoutEditorView.swift`、`MenuBarLayoutEditorView.swift`、`MusicSettingsTabs.swift`、`DevActivitySettingsView.swift`、`DataViewerView.swift`、`BatteryDebugMenu.swift` 等）。

**Interfaces:**
- Consumes: Task 2 中 `SettingsSection` 与模型枚举的中文显示属性
- Produces: 完整汉化的设置侧边栏、概览页、显示器设置、应用卸载与更新管理、布局编辑器、软件更新检查器及所有共享设置行组件。

- [ ] **Step 1: 逐文件汉化上述 23 个设置组件与子面板中的所有 UI 字符串、占位符、弹窗、工具提示与时间/大小格式化文案**
- [ ] **Step 2: 检查是否有枚举直接显示 `.rawValue`，若有则改为中文映射或 `.displayName`**

---

### Task 4: 设置中心主面板 `SettingsPanes.swift` 全量精细汉化

**Files:**
- Modify: `Sapphire/App Settings/SettingsPanes.swift`（共 13,587 行）

**Interfaces:**
- Consumes: Task 2 模型显示属性、Task 3 共享设置组件签名（签名保持不变）
- Produces: 100% 汉化的 `SettingsPanes.swift` 全部 30+ 个设置详情面板。

- [ ] **Step 1: 分段完整阅读并汉化 `SettingsPanes.swift` 第 1–4500 行**（权限引导、灵动岛外观、系统增强、应用管理、存储分析、通用设置、文件暂存架、便签、剪贴板、随身镜、保持唤醒、小组件、实时活动、快捷指令、锁屏）
- [ ] **Step 2: 分段完整阅读并汉化 `SettingsPanes.swift` 第 4501–9500 行**（窗口磁吸分区、通知、蓝牙靠近解锁、风扇控制、电池校准与电池统计图表、HUD 浮层、音乐、天气、日历）
- [ ] **Step 3: 分段完整阅读并汉化 `SettingsPanes.swift` 第 9501–13587 行**（护眼休息、专注时段、应用锁、智能助手、体育、财经、程序坞布局、媒体优化、鼠键、监控、归档与 DMG 安装、表情符号、附近分享、跨端互通、关于与账户页）

---

### Task 5: 应用主入口、新手引导、灵动岛核心、实时活动、系统 HUD 与通用工具汉化

**Files:**
- Modify: `Sapphire/App/*.swift`（含 `AppDelegate.swift`、`OnboardingView.swift`、`PermissionsManager.swift`、`HelperManager.swift`、`SapphireInstanceCoordination.swift` 等）
- Modify: `Sapphire/Notch/*.swift`（含 `NotchExpandedChrome.swift`、`NotchIslandBar.swift`、`NotchController.swift`、`NotchConfiguration.swift` 等）
- Modify: `Sapphire/LiveActivities/**/*.swift`（含 `LiveActivityComponents.swift`、`LiveActivityManager.swift`、`DevActivityLiveActivityView.swift`、`NearbyTransferLiveActivityView.swift`、`FileShelfActivityView.swift` 等）
- Modify: `Sapphire/OpenBubbles/*.swift`、`Sapphire/Stubs/*.swift`、`Sapphire/System/**/*.swift`（含 `SystemHUD.swift`、`PillHUDController.swift` 等）、`Sapphire/Utilities/*.swift`（含 `TimeFormatting.swift`、`UserWindowSupport.swift`、`NotchSearchField.swift` 等）

**Interfaces:**
- Consumes: Task 2 模型显示属性
- Produces: 汉化的菜单栏菜单、首次启动引导流、权限提示弹窗、灵动岛顶部工具栏、各类实时活动卡片、系统音量/亮度/背光 HUD 及时间格式化输出。

- [ ] **Step 1: 汉化 `Sapphire/App/*` 与 `Sapphire/OpenBubbles/*`、`Sapphire/Stubs/*`**
- [ ] **Step 2: 汉化 `Sapphire/Notch/*` 与 `Sapphire/LiveActivities/**/*`（修复 `NearbyTransferLiveActivityView` 中的 `"NFiles"` 本地化字符串为直接中文格式）**
- [ ] **Step 3: 汉化 `Sapphire/System/**/*` 与 `Sapphire/Utilities/*`**

---

### Task 6: 全部小组件（`Sapphire/Widgets` 与 `SapphireAndroidWidgets`）汉化

**Files:**
- Modify: `Sapphire/Widgets/**/*.swift`（全部 49 个文件，含 `MusicPlayer/*`、`Audio/*`、`Focus/*`、`Dropbox/*`、`Dev/*`、`Battery/*`、`Calendar/*`、`Weather/*`、`Timer/*`、`Mirror/*`、`Clipboard/*`、`Notes/*`、`Shopify/*`、`Shortcuts/*`、`Zones/*`、`FileOperations/*`、`Shared/*`、`NotchWidgetView.swift`）
- Modify: `SapphireAndroidWidgets/SapphireAndroidWidgets.swift`

**Interfaces:**
- Consumes: Task 2 模型显示属性
- Produces: 全部灵动岛展开小组件与 macOS 桌面 Android 互通小组件中文界面。

- [ ] **Step 1: 汉化 `MusicPlayer/*`（尤其是 6,474 行的 `MusicHubViews.swift`、`MusicPlayerViews.swift`、`LyricsViews.swift`）与 `Audio/*`（均衡器、多设备音频、耳机控制）**
- [ ] **Step 2: 汉化其余全部子目录小组件（`Focus`、`Dropbox`、`Dev`、`Battery`、`Calendar`、`Weather`、`Timer`、`Mirror`、`Clipboard`、`Notes`、`Shopify`、`Shortcuts`、`Zones`、`FileOperations`、`Shared`）与 `SapphireAndroidWidgets.swift`**

---

### Task 7: 后台服务层（`Sapphire/Services`）与外部子模块（`Nearby` / `Volume` / `CaptchaSolver` / `Helper`）汉化

**Files:**
- Modify: `Sapphire/Services/**/*.swift`（所有含 UI 视图、通知、弹窗、菜单栏搜索面板、锁屏小组件、音频设置视图 `AudioSettingsView.swift`、Face ID 提示、归档解压提示、错误 `errorDescription` 的文件）
- Modify: `Nearby/*.swift`（含 `ShareViewController.swift`，将 `NSLocalizedString` 调用改为简体中文默认文案）
- Modify: `Volume/**/*.swift`（含 `AudioEngine.swift`、`DeviceVolumeMonitor.swift`、`DDCController.swift`、`DisplayableApp.swift` 等中的用户可见设备/错误名称，严禁改动 CoreAudio 底层硬件属性匹配字符串）
- Modify: `CaptchaSolver/CaptchaProvider.swift`、`Util/PropertyListModifier.swift`、`com.shariq.sapphireHelper/*.swift`（仅用户可见错误信息，保留 XPC/SMC 键名不动）

**Interfaces:**
- Consumes: Task 2 模型显示属性
- Produces: 汉化的锁屏界面、菜单栏管理面板、音频设置页、系统通知、附近分享弹窗及各服务用户提示。

- [ ] **Step 1: 汉化 `Sapphire/Services/LockScreen/*`、`Menubar/*`、`Audio/AudioSettingsView.swift`、`AppActivities/*`、`FileProgress/*`、`Clipboard/*`、`Dropbox/*`**
- [ ] **Step 2: 汉化 `Sapphire/Services/` 其余子目录（`Auth`、`Archive`、`Battery`、`Bluetooth`、`Browser`、`Calendar`、`DevActivity`、`FaceID`、`Focus`、`Gemini`、`Launchpad`、`Mirror`、`Miscellaneous`、`Music`、`Notes`、`Shortcuts`、`Stats`、`System`、`Timer`、`Wallpaper`、`Weather`、`Zones`、`Virtualization`）中的用户可见文案**
- [ ] **Step 3: 汉化 `Nearby/*`、`Volume/**/*`、`CaptchaSolver/*`、`com.shariq.sapphireHelper/*` 中的用户可见文案**

---

### Task 8: 项目 Markdown 文档全量汉化

**Files:**
- Modify: `README.md`
- Modify: `CONTRIBUTING.md`
- Modify: `CODE_OF_CONDUCT.md`
- Modify: `SECURITY.md`
- Modify: `CLA.md`
- Modify: `CREDITS.md`
- Modify: `.github/ISSUE_TEMPLATE/bug_report.md`
- Modify: `.github/ISSUE_TEMPLATE/feature_request.md`
- Modify: `Tools/ModelConversion.md`
- Modify: `CaptchaSolver/CaptchaSolver.docc/CaptchaSolver.md`
- Modify: `CaptchaSolverInterface/CaptchaSolverInterface.docc/CaptchaSolverInterface.md`
- Modify: `docs/parity/vorssaint-gap-map.md`

**Interfaces:**
- Consumes: 规范术语表
- Produces: 完整流畅的简体中文项目说明与开发文档（保留外部 URL、命令与代码块原样）。

- [ ] **Step 1: 汉化根目录文档（`README.md`、`CONTRIBUTING.md`、`CODE_OF_CONDUCT.md`、`SECURITY.md`、`CLA.md`、`CREDITS.md`）**
- [ ] **Step 2: 汉化 `.github/ISSUE_TEMPLATE/*.md`、`Tools/ModelConversion.md`、`CaptchaSolver*/*.docc/*.md`、`docs/parity/vorssaint-gap-map.md`**

---

### Task 9: 全量残留英文扫描、测试修复与最终编译验收

**Files:**
- Modify: 扫描发现的任何遗漏 UI 文件及因界面文案变更需要同步更新断言的 `SapphireTests/*.swift`

**Interfaces:**
- Consumes: Task 1–8 全部成果
- Produces: 零 UI 英文遗漏、零编译错误、`SapphireTests` 全部通过的最终交付物。

- [ ] **Step 1: 运行 Python UI 字符串审计脚本，排查并清零任何遗漏的英文 UI 字符串**
- [ ] **Step 2: 编译全部目标并运行 `SapphireTests` 单元测试套件，修复因界面文案汉化引起的测试断言差异**
  - Run: `xcodebuild -project Sapphire.xcodeproj -scheme Sapphire -destination "platform=macOS" ENABLE_DEBUG_DYLIB=NO CODE_SIGNING_ALLOWED=NO CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO test`
  - Expected: `** TEST SUCCEEDED **`
