# Sapphire 与 Vorssaint — 功能差距对照与改进计划

整理于 2026-10-07，基于对两个代码库的只读审阅。Vorssaint（github.com/vorssaint/vorssaint-utils）采用 **GPL-3.0** 许可证：仅作为功能特性与实现思路的参考对照，其任何代码、结构或字符串均不得引入 Sapphire。

状态图例：**✓** 已完成 · **◐** 部分实现 · **✗** 缺失 · **⚠** 已存在但损坏/不安全。
工作量评估：S（≤1 天）· M（数天）· L（一周以上）。

## 第一阶段 — Sapphire 现有已发布功能中的安全性与正确性缺陷

| # | 缺陷描述 | 涉及位置 | 工作量 |
|---|---|---|---|
| 1 | 风扇控制：应用退出/崩溃/XPC 断开时辅助进程（Helper）从未恢复自动模式，无心跳租约机制，系统唤醒时未清空去重缓存 → 风扇可能无限期维持在强制转速状态；界面显示的 RPM 转速数据陈旧过时。 | `com.shariq.sapphireHelper/XPCServer.swift:26-31`, `Helper.swift:216-221,1053-1125`, `Services/Battery/FanManager.swift` | S–M |
| 2 | 保持唤醒：启动 `caffeinate` 子进程时未携带 `-w <pid>` 参数（主进程崩溃后仍残留运行），重启无退避限制，且即使关闭“保持唤醒”功能，开发任务自动唤醒逻辑仍会启动。 | `Services/System/CaffeinateManager.swift:73-157,252,511-525`, `BatteryManager.swift:90` | S |
| 3 | 退出保护：“额外修饰键”模式直接放行了 ⌃⌘Q（锁定屏幕）/ ⇧⌘Q（退出登录）；`extraModifier` 配置项实际从未生效。 | `SystemEnhanceQuitProtection.swift:226-238,318` | S |
| 4 | 鼠标控制：吞掉了鼠标中键点击事件（以及映射为“无”的第 4/5 侧键）。 | `Services/Mouse/MouseControlManager.swift:241-243,282-284` | S |
| 5 | 超级键（Super Key）：使用默认的右 ⌘ 映射为 Hyper 键时容易出现卡键（通用 ⌘ 修饰标志与合成的左 ⌘ 冲突）。 | `Services/Mouse/SuperKeyManager.swift:220-229` | S |
| 6 | 鼠标侧键抬起竞态问题（按下事件异步派发，抬起事件可能先于按下到达）。 | `MouseControlManager.swift:315-321` | S |
| 7 | 自动退出（AutoQuit）：可能误杀窗口位于其他桌面空间（Space）的应用（未授予屏幕录制权限时会被误判为无窗口）。 | `SystemEnhanceAutoQuit.swift:84-136`, `SystemEnhanceWindowRegistry.swift:695` | M |
| 8 | 访达（Finder）⌘X/⌘V：从未真正移动文件；在访达文本输入框中吞掉了 ⌘X；在事件拦截（Event Tap）线程上直接操作剪贴板与 LaunchServices；按 F2 重命名时发送的是 Return 键。 | `Services/Clipboard/FinderCutPasteManager.swift:134,152-203,171-179,260,278` | M |
| 9 | 程序坞点击（Dock Clicks）：吞掉了鼠标按下事件（导致当前活跃图标无法拖拽），程序坞变化后图标缓存过时 → 误操作到错误的应用。 | `SystemEnhanceDockClicks.swift:242`, `SystemEnhanceSupport.swift:441-540` | M |
| 10 | 显示器软件调暗为空操作（no-op）；DDC 硬件调光失败时无软件回退；按键连发时每次都在主线程休眠等待 DDC 写入；屏幕配置变更时引发大量重复重建风暴。 | `System/Display/Model/OtherDisplay.swift:16,77-80,122`, `Support/Arm64DDC.swift:106-118`, `SystemHUD.swift:200-204,579` | M |
| 11 | DMG 安装器：同步等待磁盘卸载阻塞主线程（最长可达 24 秒）；手动文件拷贝将 Framework 符号链接展开为普通文件夹 → 破坏应用签名。 | `Services/DMGInstaller/DMGInstallerManager.swift:226-265,394-425` | S–M |
| 12 | 媒体/文件转换：视频格式转换始终失败（直接构造 `AVFileType(ext)`），文件名在最后一个 `_` 处被截断损坏，静默覆盖已有文件，在主线程处理图像，转换失败后进度条永不结束；媒体优化器丢失 EXIF 方向信息与色彩配置文件；PDF 页面被硬裁剪为 Letter 尺寸；剪贴板自动优化会丢弃剪贴板中的其他数据类型。 | `Services/Dropbox/FileConversionManager.swift:72-203`, `Services/MediaOptimizer/MediaOptimizer.swift:99-182,408`, `MediaOptimizerManager.swift:116-153` | S–M |
| 13 | 系统监控（Stats）：CPU 时钟滴答差值计算在计数器回绕时触发溢出崩溃（系统长时间运行后必现）；`EnergyReader` 每 5 秒解析了错误的 `top` 采样帧；每 3 秒全量读取所有 SMC 键值；存在 IOKit 资源泄漏；内存“已用”数值统计偏高。 | `Services/Stats/Stats.swift:651-686,762,886-943,1042-1114`, `Services/Battery/EnergyReader.swift` | S–M |
| 14 | 混音器（Mixer）：拖动均衡器（EQ）时 `retiredEQSetups` 无限增长；所有 CoreAudio HAL 操作均在主线程执行；销毁阶段可能出现释放后重用（use-after-free）；在信号处理函数中调用了 CoreAudio。 | `Services/Audio/MultiAudioManager.swift:577-871,1018-1183,1580-1613` | M |
| 15 | 剪贴板与文件暂存架：所有剪贴板捕获与转码均在主线程执行，历史记录默认无上限，每次按键均触发无缓存全文检索；“锁屏时自动清空”在真实锁屏时从未触发；文件暂存架泄漏临时文件副本且永不过期清理；便签小组件未自动保存；在主线程扫描“下载”目录。 | `ClipboardManager.swift:421,477-578`, `SettingsModel.swift:1284-1286`, `ClipboardAutoClearManager.swift:68-74`, `FileShelfManager.swift:37,67-110,137-150`, `NotesWidgetView.swift:292-306`, `FileProgressTracker.swift:278-336` | M |
| 16 | 应用更新检测：每次启动都执行全量网络扫描（忽略已保存的 `lastCheckedAt`），每次启动重复播报相同的更新项，运行 Homebrew 时未传入必要环境变量（导致 `brew update` 缓慢，需 sudo 的 cask 更新会挂起长达 30 分钟）。 | `App Settings/InstalledAppUpdates.swift:1093-1097,1438,2120-2161,2678` | S–M |
| 17 | 其他杂项：OCR 截图浮层混用了 `NSScreen.main` 与主显示器边界（在副屏上截取像素错位），且缺少屏幕录制权限预检；随身镜（Mirror）无法取消正在启动中的会话；XDR 支持依赖硬编码机型列表且会重置所有显示器的 Gamma；指针加速开关为空操作且会连带修改触控板；滚动方向反转会误作用于触控板，且会重新派发每一个滚动事件。 | `OCRScreenshotMonitor.swift:148,206,218-237`, `MirrorCameraManager.swift:112-181`, `XDRUtil.swift:55-59`, `XDRGamma.swift:206`, `MouseControlManager.swift:142,172-217,399-418` | M |

## 第一阶段 b — 深度排查发现（2026-10-07，第二轮审阅）

### 基础设施
| # | 问题描述 | 涉及位置 | 工作量 |
|---|---|---|---|
| I1 | `$settings` 订阅回调在新值写入存储之前就已触发（`current` 在 `didSet` 中才更新）→ 导致 NearDrop/启动台（Launchpad）开关需重启应用才生效、禁用启动台后菜单栏图标仍残留、鼠标控制开关状态反向生效、连续互通发送旧版专注模式设置、文件进度/应用活动监控读取到陈旧列表。修复方案：在写入存储后再发布变更；将原始 `sink` 迁移至 `changes(of:)`；补充回归测试。 | `Models/SettingsModel.swift:2606-2616`, `AppDelegate.swift:433-472,874,1386,1418`, `ContinuityFocusModeManager.swift:117-125`, `AppActivityMonitor.swift:80-101` | S–M |
| I2 | 只要授予了辅助功能权限，窗口注册表（Window Registry）就会常驻运行，即使所有依赖功能均已关闭（空闲状态下主要的 CPU 占用来源）。需改为按引用计数按需获取/释放。 | `AppDelegate.swift:755`, `SystemEnhanceWindowRegistry.swift:50-65` | S |
| I3 | 设置数据块达 135 KB / 682 个字段，每次微小变更及每次启动都会全量重新编码；其中约 60 KB 是每个 `CodableColor` 中重复序列化的 ICC 色彩配置数据，程序坞预设内联占用了 42 KB。需改为按颜色分量编码、拆分预设文件、跳过启动时的无意义持久化。 | `ShortcutModels.swift:60-73`, `SettingsModel.swift:2707-2722,2839-2898` | M |
| I4 | 容错解码在降级或切换更新通道时会静默丢弃新版设置字段并直接覆盖保存。需备份原始数据负载，并引入 `schemaVersion` 版本守卫。 | `SettingsModel.swift:2192-2231,2709-2714` | S |
| I5 | 启动链路：Firebase 在主线程同步初始化；在完成新手引导前就提前弹窗请求通知权限、注册开机自启项及特权辅助进程；在主线程查询已配对蓝牙设备；每次启动都调用 `NSUpdateDynamicServices`；在主线程同步读取钥匙串（Keychain）。 | `AppDelegate.swift:346,360,447-455,751,803,886`, `FirebaseBootstrap.swift:23-82`, `HelperManager.swift:394-418`, `APIKeyManager.swift:35-60` | S–M |
| I6 | 崩溃恢复安全性：强制低电量模式、合盖自动静音及非租约风扇控制路径未写入状态日志（Journal）；缺少启动期连续崩溃标记；未实现 `applicationShouldHandleReopen`（隐藏菜单栏图标后无法再次呼出应用设置）。 | `LidAngleAutomationManager.swift:148-196`, `FanManager.swift:640-645`, `AppDelegate` | 每项 S |
| I7 | `GlobalEventTap`：在持有分发锁期间调用 `tapCreate` → 导致注册或重建期间事件跳过处理函数；`.replace` 返回了未持有（unretained）的新事件对象。 | `System/GlobalEventTap.swift:161-262` | S |
| I8 | `URLSession` 泄漏（13 处中有 11 处从未调用 `invalidate`；即圈即搜每次重定向都新建会话与代理）；`LyricsDatabase` 缓存无容量上限。 | `UpdateChecker.swift:550`, `CircleToSearchLookup.swift:238`, `LyricsDatabase.swift:14` | S |
| I9 | 任意应用触发辅助功能广播后，只要读到一次假阴性的 `AXIsProcessTrusted` 就会直接终止 Sapphire；`PermissionsManager` 在每次应用激活时全量重新发布状态；本地网络权限“已授予”状态并非真实检测结果。 | `System/Accessibility.swift:125-172`, `App/PermissionsManager.swift:127-474` | S |
| I10 | 菜单栏：`isAgentHosted` 仅在启动时求值一次（若代理稍后启动，整次会话期间分隔符均无法隐藏任何图标）；每次更新均完整拷贝一份 `Settings`。同时运行两个 Sapphire 实例会相互覆盖设置文件。 | `StatusBarController.swift:399,533`, `SapphireInstanceCoordination` | S |

### 灵动岛 / 实时活动
| # | 问题描述 | 涉及位置 | 工作量 |
|---|---|---|---|
| N1 | 根宿主视图随窗口大小自动缩放 → 每次 `setFrame` 都会触发两棵 SwiftUI 视图树全量重排（展开/收起/悬停时耗时 20–50 ms）。应改为固定尺寸舞台并仅移动原点；普通鼠标悬停时不要切换为全屏大窗口。 | `AppDelegate.swift:1577-1580`, `NotchHostWindowSizer`, `NotchController.swift:349,2661-2672` | S–M |
| N2 | 每次展开灵动岛都会重新挂载小组件平面（Widget Plane）父视图 → 导致 SwiftUI 销毁视图图并触发小组件全量重建。应将平面宿主保留为常驻同级节点，通过图层（Layer）控制显隐，并用显式的 `presented` 环境变量控制定时器启停。 | `NotchWidgetPlane.swift:295-305` | M |
| N3 | `@EnvironmentObject ActiveAppMonitor` 被重新引入 `NotchController`（导致切换应用卡顿修复出现回退）；视图 `body` 直接读取 `activityContent`/`activityHasBottomContent` → 每句歌词滚动都会触发所有显示器上的全量重绘。 | `NotchController.swift:65,298-347,527-542,606` | S |
| N4 | 多显示器问题：`TrackpadGestureHandler` 回调采用后写入覆盖策略（滑动手势会隐藏错误屏幕上的灵动岛）、`MenuBarInteractionManager` 未按单个控制器独立挂起、可聚焦窗口硬编码取 `notchWindows.first`、音乐队列与即圈即搜通知会同时展开所有显示器的灵动岛。 | `NotchController.swift:625-660,667,1035-1236`, `AppDelegate.swift:1643-1670` | S–M |
| N5 | 未加状态守卫的 `asyncAfter` 模糊/缩放重置在状态变更后仍会误触发；窗口尺寸计算器回退到 `NSScreen.main`；`.full` 内容相等性判断忽略了视图本身 → 导致视图更新被丢弃；`showLyricsBinding` 已成死代码；计时器/开发活动数字文本需迁移至 `LayerNumericText`。 | `NotchController.swift:1480-1522`, `NotchHostWindowSizer.swift:63,90`, `AppModels.swift:217`, `LiveActivityComponents.swift:584,1388` | S |
| N6 | 液态玻璃材质：保持单一持久化材质表面，避免每帧调用 `_setPath:`（可在两端做渐隐处理，或采用固定路径 + 遮罩）——视觉效果需经 Shariq 确认；Core Animation 形变（裁剪揭示动画）同样需经视觉确认。 | `NotchSurfaceBackground.swift`, `LiquidGlassView.swift` | M / L |

### 媒体与系统服务
| # | 问题描述 | 涉及位置 | 工作量 |
|---|---|---|---|
| M1 | 正在播放（Now Playing）事件流：每次进程退出都会重复重启两次（自身重连 + `onListenerTerminated`），泄漏未受追踪的 `perl` 子进程；缺少启动令牌与重启配额限制；共享缓冲区存在多进程输出交错污染。每次媒体控制命令都会派生新的 `perl` 进程；系统休眠或锁屏时未暂停监听流。 | `NativeMediaController.swift:206-319`, `MusicManager.swift:1736-1739,2147` | M |
| M2 | 专辑封面在主线程按全尺寸解码；“相同封面”判等逻辑仅比对数据长度 + 3 个采样字节。 | `NativeMediaController.swift:443-472` | S–M |
| M3 | 蓝牙电量读取：共用单一 `continuation` → 第二次刷新会使第一次刷新永久悬空（Continuation 泄漏）；`system_profiler` 解析器始终返回空数组 `[]` 却仍耗时 1–3 秒；每次设备连接都会触发包含 12 秒 GATT 超时的串行探测链；连接蓝牙鼠标时会误发虚假的 AirPods“已连接”提示。 | `BluetoothBatteryReader.swift:38-311`, `BluetoothManager.swift:211-257`, `MagicBattery.swift:258-263` | M |
| M4 | 日历服务：48 小时日程快照在跨天/系统时间调整/时区变更/睡眠唤醒时均不刷新；里程碑提醒仅以 `eventIdentifier` 为键 → 导致重复日程的后续场次永远不再提醒；应用中分散创建了 7 个以上的 `EKEventStore` 实例。 | `CalendarService.swift:148-173`, `LiveActivityManager.swift:1858-2000` | S–M |
| M5 | 歌词服务：LRCLIB 请求未携带歌曲时长参数、请求失败后永久阻断后续重试、歌词翻译结果未缓存、`trimLyricsCache` 缓存淘汰顺序写反。 | `MusicUtilities.swift:385-412`, `MusicManager.swift:2682-2693,2878-2887,3086-3121` | S |
| M6 | 计时器在系统休眠唤醒后延迟响铃（`Timer` 时钟在休眠期间暂停）；常驻 `log stream` 子进程每 1 秒无条件拉起且无孤儿进程保护。 | `TimerManager.swift:134-139,379-674` | S–M |
| M7 | 通知数据库查询、plist 解码及应用图标查找均在主线程执行；同时到达的多条通知会相互覆盖。 | `NotificationManager.swift:237-334` | S |
| M8 | 开发者实时活动：Claude 项目路径转换规则仅映射了 `/ . _`，导致包含空格的路径永远无法匹配。 | `DevAgentSessionActivity.swift:55-62` | S |
| M9 | `ProcessRunner` 在同一个文件描述符上挂载了两个读取器；`SystemLogParser` 丢弃了跨块半行文本与多字节 UTF-8 字符；Spotify 心跳定时器泄漏；`BatteryDataLogger` 日志文件无限膨胀且在主线程解码；天气数据在系统唤醒后不刷新；`LyricsAudioTap` 在音频格式不匹配失败时发生泄漏；在非主线程调用了非线程安全的 `setenv`。 | `ProcessRunner.swift:67-70`, `SystemLogParser.swift:41-48`, `SpotifyNetwork.swift:614-626`, `BatteryLogEntry.swift:162-190`, `WeatherViewModel.swift:192-205`, `LyricsAudioTap.swift:41`, `LogReader.swift:62-69` | 每项 S |

（已分发给第一波负责人的事项：`MouseControlManager` 的 I1 问题，`FileProgressTracker` 的 Chrome 历史记录数据库拷贝 + 事件合并 + I1 问题。）

## 第二阶段 — 缺陷所指向的性能与架构演进

- **功能运行时按需加载：** 目前约 30 个管理器在 `AppDelegate.swift:760-800` 中无条件全部启动；应将其改造为受各自功能开关控制的惰性初始化（为后续“仅启用所需模块”+ 预设方案 + 按功能按需申请权限奠定基础）。
- **应用切换器（App Switcher）：** 迁移至共享的 `GlobalEventTap`，不在事件拦截回调内执行 SwiftUI 布局，支持临时接管系统 ⌘Tab 快捷键、最近使用（MRU）排序、搜索过滤、按桌面空间/显示器筛选以及纯图标模式。
- **统一的 HID 层滚轮变换管线：** 在指针线程上进行原地事件修改（仅限外接鼠标，支持按光标下方的应用排除）→ 在同一条路径中统一实现滚动方向反转、线性滚动、横向滚动修饰键以及平滑滚动；支持按设备独立配置指针加速度并在退出时安全还原。
- **剪贴板串行通道：** 建立单一的后台串行剪贴板访问通道，由剪贴板历史、自动清空、链接净化（Clean URL）、纯文本粘贴及文本片段（Snippets）共同复用。
- **系统监控采样策略：** 仅在存在可见消费视图时运行采样，支持按指标设置不同采样步长，并使用环形缓冲区（Ring Buffer）存储历史记录。
- 在所有辅助功能（AX）窗口尺寸写入操作（窗口磁吸分区、最大化）前后临时挂起 `AXEnhancedUserInterface`。

## 第三阶段 — Sapphire 缺失或较为薄弱的功能领域

| 功能领域 | 缺失（✗）/ 较薄弱（◐）项 | 工作量 |
|---|---|---|
| 截图与录制 | ✗ 统一捕获浮层（支持带放大镜与多屏独立选区的截图/OCR/取色/二维码识别），✗ 截图快速预览/置顶贴图/标注/隐私打码/背景衬底/长截图拼接，✗ 屏幕录制器（基于 `SCStream` + `AVAssetWriter`，独立系统音轨与麦克风音轨、裁剪、导出 GIF），✗ 屏幕取色器，✗ 二维码识别 | 每项 M–L |
| 音频控制 | ◐ 超过 100% 的音量增强（带前瞻限幅器与平滑渐变），✗ 单应用音频输出设备路由，✗ 应用置顶/排序，✗ 输出设备切换快捷键 + 拔出耳机自动降低音量，◐ 首选输入设备锁定 + 麦克风静音快捷键，✗ 阻止系统“音乐”应用自动弹出，◐ 基于播放器音频分流（Tap）的真实实时 EQ 频谱柱 | 每项 S–M |
| 系统监控 | ◐ 带历史趋势图的监控面板、能效核/性能核（E/P 核）区分、基于 `proc_pid_rusage` 的单进程能耗统计，✗ 网络监控模块（64 位计数器、累计流量、内外网 IP、网速测试），◐ 更丰富的菜单栏监控读数，◐ 带系统运行时间门限的高温/低电量告警 | M–L |
| 窗口管理 | ◐ 窗口磁吸分区预设（二分屏/三分屏/四分屏、窗口间距、循环切换），✗ 移至下一台显示器，✗ 还原窗口原尺寸，✗ 屏幕边缘磁吸 + 修饰键拖拽移动/缩放，◐ 程序坞预览窗口移动/磁吸/自动隐藏悬停保持，✗ 固定桌面空间（Space）顺序 | S–L |
| 输入增强 | ✗ 平滑/线性/横向滚动，✗ 触控板三指轻点中键，◐ Caps Lock 映射超级键（崩溃安全重映射），◐ 文本片段库 + 后缀触发匹配，◐ 按功能独立配置应用排除列表，◐ 全局快捷键冲突检测 | S–M |
| 剪贴板与文件 | ◐ 剪贴板置顶 + ⌘1–9 快速粘贴 + 内容预览 + 忽略应用列表，◐ 手动净化链接（Clean URL）+ 扩展跟踪参数规则，◐ 跟随光标的文件暂存架（不使用全局指针监听——改为由拖拽剪贴板变更事件驱动） | M |
| 效率工具 | ✗ 命令面板（聚合检索应用/窗口/文件/剪贴板/文本片段/菜单项/计算器/单位换算/表情符号），✗ 快捷控制面板，✗ 快捷开关组，✗ 轮盘菜单（Radial Menu），◐ 随手记便签（多标签页、Markdown、自动保存），✗ 屏幕键盘清洁模式（带可靠防锁死退出机制），◐ AI 编程助手用量监控（配额上限/重置时间/Token 消耗/任务完成通知，支持 OpenCode/Copilot） | S–L |
| 应用管理 | ◐ 批量更新应用 + 跳过指定版本，✗ Homebrew 可视化管理，◐ 卸载残留/启动代理（Launch Agents）/日志清理 + 定时清理计划，◐ 卸载器提权处理 + 安装收据（Receipts）清理 + Homebrew Cask 联动，✗ 端口占用管理器，✗ 即时通讯软件下载文件定期清理 | S–L |
| 能耗与显示 | ◐ 保持唤醒触发条件（按运行应用/电源状态/外接显示器），◐ 单显示器独立亮度页 + ½ 与 ¼ 微调步进 + 额外软件调暗，✗ 系统休眠期间自动关闭蓝牙 | S–M |
| 平台化能力 | ✗ 模块化功能按需启用 + 场景预设，◐ 基于版本化键白名单的设置导入导出，◐ 按功能引导的权限申请流程，✗ 多语言本地化支持 | M–L |

Sapphire 目前已处于领先地位的优势项（需保持）：全局共享单一 `GlobalEventTap`、单应用独立均衡器（EQ）/8D/环绕音效、更广泛的应用更新来源识别、带 Team ID 校验的签名自动更新、连续互通（Continuity）、灵动岛/实时活动、歌词显示。
