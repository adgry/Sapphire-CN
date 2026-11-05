# Sapphire 全项目纯中文汉化设计规格说明书

- **日期**：2026-11-05
- **范围**：全项目用户可见界面、系统权限/扩展配置（`Info.plist` / `Main.storyboard`）、设置搜索引擎与助手、项目文档（Markdown）

## 1. 目标与成功标准

将 `Sapphire`（macOS 灵动岛全能工具）直接替换为纯简体中文版本，不引入多语言资源包（`.xcstrings`）或语言切换开关。

**成功标准：**
1. 主应用 `Sapphire` 及子模块（`Nearby`、`Volume`、`SapphireAndroidWidgets`、`PublicExtensionStubs`、`CaptchaSolver`、`com.shariq.sapphireHelper`）中所有用户可见界面文案（设置面板、侧边栏、灵动岛/实时活动、桌面与灵动岛小组件、引导页、锁屏、菜单栏、HUD、通知、弹窗、无障碍标签、错误提示）全部汉化为自然地道的简体中文。
2. 所有 `Info.plist` 中的权限用途说明（`NS*UsageDescription`）、系统服务菜单（`NSServices`）、文档类型描述（`CFBundleTypeName` / `UTTypeDescription`）与扩展显示名称汉化，且 Xcode 项目默认开发区域设为 `zh-Hans` 以确保 macOS 标准菜单自动显示简体中文。
3. 设置搜索 `SettingsIndex` 与设置助手 `SettingsAssistant` 完整支持中文关键词检索（单字/词组/子串匹配）及英文缩写检索（如 `CPU`、`HUD`、`AirDrop`）。
4. 项目根目录及子目录中的说明文档（`README.md`、`CONTRIBUTING.md`、`CODE_OF_CONDUCT.md`、`SECURITY.md`、`CLA.md`、`CREDITS.md`、`.github/ISSUE_TEMPLATE/*.md`、`Tools/ModelConversion.md`、`CaptchaSolver*/*.docc/*.md`）全部汉化。
5. **零配置回归与零注释改动**：不修改任何 `Codable` / `UserDefaults` 存储键名、持久化枚举 `rawValue`、XPC/IPC 标识、API 参数、AppleScript/Shell 命令、代码注释与内部调试日志（`Logger` / `os_log` / `NSLog` / `print`）。
6. `xcodebuild` 编译 `Sapphire` 及运行 `SapphireTests` 核心单元测试全部通过。

## 2. 统一术语规范

| 英文原文 | 统一简体中文译法 | 备注 |
|---|---|---|
| `Sapphire` | `Sapphire` | 保留应用品牌原名 |
| `Notch` | `灵动岛` | 如 `Notch Bar` → `灵动岛栏`，`Notch Appearance` → `灵动岛外观` |
| `Floating Island` | `悬浮灵动岛` | 无刘海显示器上的灵动岛形态 |
| `Live Activity` / `Live Activities` | `实时活动` | 对齐 Apple 官方译法 |
| `App Activities` | `应用活动` | 会议、通话、录屏等活动状态 |
| `Widget` / `Widgets` | `小组件` | 对齐 macOS 官方译法 |
| `Satellite` / `Satellites` | `伴随挂件` / `两侧挂件` | 灵动岛两侧独立小挂件 |
| `File Shelf` | `文件暂存架` | 拖拽文件暂存区 |
| `Snap Zones` | `窗口磁吸分区` | 窗口贴靠/分屏布局功能 |
| `Dock Layouts` | `程序坞布局` | `Dock` 统一译为 `程序坞` |
| `Menu Bar` | `菜单栏` | 对齐 macOS 官方译法 |
| `Lock Screen` | `锁定屏幕` / `锁屏` | 对齐 macOS 官方译法 |
| `Proximity Unlock` / `Bluetooth Unlock` | `蓝牙靠近解锁` | 蓝牙设备距离自动锁屏/解锁 |
| `Caffeinate` / `Keep Awake` | `保持唤醒` | 防休眠功能 |
| `Eye Break` | `护眼休息` | 20-20-20 护眼提醒 |
| `Focus Session` | `专注时段` | 番茄钟/专注模式 |
| `App Lock` | `应用锁` | 生物识别应用锁定 |
| `Nearby Share` | `附近分享` | Android 跨端文件传输 |
| `Continuity` | `跨端互通` | 手机与 Mac 协同功能（注意：波形样式枚举持久化 `rawValue = "Continuity"` 内部值保持不变，仅 UI 显示译为中文） |
| `Circle to Search` | `画圈搜屏` | 屏幕框选识别搜索 |
| `HUD` | `HUD` / `HUD 浮层` | 音量、亮度、键盘背光指示浮层 |
| `Finder` | `访达` | 对齐 macOS 官方译法 |
| `Trash` | `废纸篓` | 对齐 macOS 官方译法 |
| `Launch at Login` | `登录时启动` / `开机自启` | 对齐 macOS 官方译法 |
| `AirDrop` / `Touch ID` / `Face ID` / `Apple Music` / `Spotify` / `TIDAL` / `Gemini` / `iMessage` | 保留原名 | 平台与第三方服务专有名词 |

## 3. 核心架构设计

### 3.1 显示文案与持久化数据严格解耦
- **禁止新增 `.swift` 文件到 `Sapphire/` 目录**（因为 `Sapphire` 与 `abt` 共享 `PBXFileSystemSynchronizedRootGroup` 例外列表，新增文件会影响 `abt` 编译；所有改动均在原文件内完成）。
- 对于符合 `Codable` 或写入 `UserDefaults` 的 `enum`（如 `SettingsModel.swift` 中的配置枚举）：
  - **绝不修改 `case` 名与英文 `rawValue`**。
  - 统一通过 `displayName`、`label`、`title`、`shortDescription`、`detail` 等计算属性提供简体中文。
  - 若视图中存在直接渲染 `Text(item.rawValue)` 或 `Label(item.rawValue, ...)` 的持久化枚举，改为渲染其 `.displayName`（仅局部私有纯 UI Tab 枚举且不参与持久化的除外，但同样推荐通过计算属性或中文显示）。
- 对于原本调用 `NSLocalizedString("Key", value: "English", comment: "")` 的代码（如 `Nearby/ShareViewController.swift`），直接替换为简体中文默认文案（如 `"已拒绝"`、`"正在发送…"`、`"PIN 码：%@"`），避免缺少 `.strings` 文件时回退显示英文键名。

### 3.2 设置搜索引擎（`SettingsIndex`）与设置助手（`SettingsAssistant`）升级
- **`SettingsIndex.swift` 双模检索**：
  1. 保留对拉丁字母/数字词元的词首 + 词干匹配，且保留 `entry.key`（驼峰拆词）与英文技术词索引，使得搜 `CPU`、`RAM`、`HUD`、`AirDrop`、`SSH` 等英文词依旧精准命中。
  2. 新增 CJK（中日韩统一表意文字）检索支持：当查询词或索引文本包含中文字符时，保留长度 `>= 1` 的中文查询词，支持对 `title`、`detail`、`pane` 进行直接子串包含匹配及双字（bigram）/单字切分加权计分（标题命中权重 4，分类面板命中权重 1.5，描述命中权重 1）。
  3. 扩充中文同义词表（如 `"刘海": ["灵动岛"]`、`"防休眠": ["保持唤醒", "caffeinate"]`、`"开机": ["登录", "启动"]`、`"解压": ["归档", "压缩包"]`、`"分屏": ["磁吸", "窗口"]` 等）与中文停用词（如 `"怎么"`、`"如何"`、`"可以"`、`"设置"`、`"的"`、`"吗"`）。
  4. `parseBool` 同时支持英文（`on/off/true/false/enable/disable`）与中文（`开/开启/打开/启用/是/显示`、`关/关闭/禁用/否/隐藏`），保持 `Value.description` 的 `"on"`/`"off"` 往返序列化兼容。
- **`SettingsIndex+Generated.swift`**：将全部设置项的 `title` 与 `detail` 翻译为与设置面板一致的简体中文，保留 `key`、`section`、`range`、`options` 不变。
- **`SettingsAssistant.swift`**：将界面状态提示、错误文案翻译为中文；在 `isQuestion` 与 `impliedToggle` 中补充中文疑问特征词与开关意图词，保持原有英文规则并存。

### 3.3 构建配置与系统级汉化
- **`Sapphire.xcodeproj/project.pbxproj`**：
  - 将 `developmentRegion` 更新为 `"zh-Hans"`，`knownRegions` 更新为 `("zh-Hans", en, Base)`。
  - 修复公开仓库快照中 `SapphireFileProvider` 目标遗漏指向 `PublicExtensionStubs/FileProvider` 占位源码与 `Info.plist` 的路径问题（对齐 `SapphireFinderSync` 的公开占位配置），使 `Sapphire` 主工程与 `SapphireTests` 可直接完整编译与测试。
- **`Info.plist` 与 `Main.storyboard`**：
  - 将主应用及扩展 `Info.plist` 中的权限描述、文件类型描述、系统服务名称、`CFBundleDevelopmentRegion` 更新为简体中文/`zh-Hans`。
  - 将 `Main.storyboard` 中的默认窗口与主菜单标题汉化。

## 4. 模块划分与并行防冲突边界

为确保零文件写入冲突，按目录所有权严格切分为独立工作包：
1. **基础设施与搜索模型层**：`project.pbxproj`、全部 `Info.plist`、`Main.storyboard`、`Sapphire/Models/*`、`SettingsIndex.swift`、`SettingsIndex+Generated.swift`、`SettingsAssistant.swift`、`SettingsIndexTests.swift`。
2. **设置中心通用视图与子面板（除 `SettingsPanes.swift` 外）**：`Sapphire/App Settings/` 下除 `SettingsIndex*`、`SettingsAssistant.swift`、`SettingsPanes.swift` 外的全部 22 个视图/工具文件。
3. **设置主面板（`SettingsPanes.swift`）**：单独完整负责 13,587 行的 `SettingsPanes.swift`，按行号区间逐段精细汉化。
4. **应用入口、灵动岛核心、实时活动、系统 HUD 与通用工具**：`Sapphire/App/*`、`Sapphire/Notch/*`、`Sapphire/LiveActivities/*`、`Sapphire/OpenBubbles/*`、`Sapphire/Stubs/*`、`Sapphire/System/*`、`Sapphire/Utilities/*`。
5. **小组件层（`Sapphire/Widgets/*` + `SapphireAndroidWidgets/*`）**：49 个小组件文件与 Android 小组件扩展。
6. **后台服务与子模块层（`Sapphire/Services/*` + `Nearby/*` + `Volume/*` + `CaptchaSolver/*` + `Util/*` + `com.shariq.sapphireHelper/*`）**：所有包含用户可见通知、弹窗、菜单栏面板、锁屏视图、音频设置页、认证提示与错误描述的服务文件。
7. **项目文档层（Markdown）**：根目录下全部 `.md`、`.github/ISSUE_TEMPLATE/*.md`、`Tools/*.md`、`CaptchaSolver*/*.docc/*.md`。

## 5. 验证与兜底机制

1. **残留英文自动扫描**：运行自定义 Python 审计脚本，扫描所有 Swift 视图控件参数（`Text`、`Button`、`SettingsRowLabel`、`InfoContainer`、`NSMenuItem`、`NSAlert`、`help`、`placeholder` 等）中是否残留未翻译的英文句子，逐一清零（专有名词白名单除外）。
2. **协议与持久化安全检查**：脚本校验 `SettingsModel.swift` 及各模块中 `Codable` 枚举的 `case` 定义与存储键未被误改。
3. **编译与自动化测试**：执行 `xcodebuild` 编译 `Sapphire` 工程并运行 `SapphireTests`（重点验证 `SettingsIndexTests`、`SettingsStorageTests` 等测试套件 100% 通过）。
