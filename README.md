# ColorOS 时钟显秒（ColorOS Clock Seconds）

适用于 **OPPO / 一加 / 真我 ColorOS 15 及以上**的 LSPosed 模块，包名为 `io.github.kelebao.colorosclockseconds`。

让状态栏（左上角）时钟显示**秒**，并支持**单击时钟切换显秒开关**。

> ✅ 已在 **ColorOS 15 真机实测可用**（状态栏显秒、单击切换、下拉通知栏均正常，且启用后不会触发 LSPosed 安全模式）。
>
> 源码仓库：<https://github.com/kelebao/ColorOS15ClockSeconds>
> 问题反馈：<https://github.com/kelebao/ColorOS15ClockSeconds/issues>

---

## 下载安装

从本仓库的 [**Releases**](../../releases/latest) 页面下载 `io.github.kelebao.colorosclockseconds_1.0.3.apk`，然后：

1. 安装 APK（需允许"安装未知来源应用"）
2. 在 **LSPosed** 中启用本模块
3. 作用域勾选：**系统界面（`com.android.systemui`）**
4. 重启 SystemUI（或重启手机）

## 功能

| 功能 | 说明 |
|------|------|
| 状态栏显秒 | 状态栏时钟显示到秒（`HH:mm:ss` / `h:mm:ss a`，跟随系统 12/24 小时制） |
| 点击切换 | **单击**状态栏时钟即切换显秒开/关；也可在设置页改为**长按** |
| 持久化 | 状态存于 `Settings.Secure`，跨进程共享，重启 SystemUI 后保持 |
| 省电 | 关闭显秒后模块立即停止每秒刷新定时器，不再有每秒唤醒 |

## 工作原理

模块基于 **legacy Xposed API**（`de.robv.android.xposed:api:82`），**只 hook 系统界面进程（`com.android.systemui`）**：

- hook 状态栏时钟 `TextView` 的 `setText`（之后）与 `onDraw`（之前），在系统设完文本后**覆写**为带秒的时间格式，**不修改系统任何内部状态**，因此不会破坏 SystemUI、也不会触发 LSPosed 安全模式。
- 用 1 秒级定时器（对齐秒边界）驱动刷新；定时器持有视图的弱引用，视图销毁后自动退出。
- 点击 / 长按时钟翻转 `Settings.Secure` 中的开关（该动作运行在 SystemUI 进程内，天然具备 `WRITE_SECURE_SETTINGS`，无需 adb 授权）。

> 兼容性做法参考开源项目 [`io.github.betterclock`](https://github.com/Xposed-Modules-Repo/io.github.betterclock)（ColorOS 15 真机验证可用）。

## 使用

- **单击状态栏时钟**：切换显秒开/关（默认手势）。
- 若单击与"下拉通知栏"冲突，进入配置页把切换手势改为**长按**。
- 配置页从 **LSPosed 管理器**打开（桌面不生成图标），可查看当前状态、切换手势、刷新状态。

## 省电说明

- **开启显秒时**：模块需要 1 秒级定时器驱动刷新（这是"显示秒"的固有开销，约 60 次/分钟重绘时钟）。定时器使用 uptime 时钟、不持有 wakelock，灭屏时主线程休眠不会触发，不唤醒设备。
- **关闭显秒时**：模块会**立即停止该定时器**，回到系统原本的"每分钟刷新一次"节奏，因此关闭显秒确实更省电。
- 状态采用**进程内缓存**，定时器不会每秒去读 `Settings.Secure`（避免每秒一次 IPC）。

## 兼容性

| 系统版本 | 状态 |
|----------|------|
| ColorOS 15 | ✅ 真机实测可用 |
| ColorOS 16 / 17+ | 同款 hook 逻辑，理论上可用（统一覆写时钟文本，不依赖原生能力） |

需要 **Root + LSPosed**。`minSdk 31` / `targetSdk 35`。

## 构建

需要 **JDK 17+**（推荐 21）与 **Android SDK Platform 35**。

```bash
./gradlew assembleRelease
```

> 国内网络若无法访问 Google Maven，可在 `~/.gradle/init.gradle` 配置阿里云镜像。
> 详见源码仓库 README。

## 排查

查看 LSPosed 日志，过滤关键字：

- `ColorOSClockSeconds: hook installed` —— hook 已注入
- `ColorOSClockSeconds: toggle listener attached` —— 已命中状态栏时钟并挂上点击监听
- `ColorOSClockSeconds: candidate:` —— 列出 SystemUI 内时钟候选控件的资源名（若秒数不显示，把这几行发开发者即可补识别规则）

## 致谢

本模块参考开源项目 [`io.github.betterclock`](https://github.com/Xposed-Modules-Repo/io.github.betterclock) 的稳妥 hook 范式实现，避免 hook 私有类导致 SystemUI 崩溃。
