# OC Desktop 技术选型（TECH）

> 记录三版**实际采用/明确排除**的技术与版本。版本号以仓库内可验证事实为准：`AppScope/app.json5`、`app/build.gradle.kts`、`oh-package.json5`、`gradle/libs.versions.toml`、`runtime.lock.json`、`patches/<tag>/`、工作流 YAML。
> 证据口径：✅ 源码/配置已核实｜🔶 沿用子工程结论｜⬜ 待验证。生成日期：2026-11-01。

---

## 一、版别与运行时矩阵

| 维度 | `oc-desktop-ark` | `oc-desktop-hos` | `oc-desktop-android` |
|---|---|---|---|
| 应用形态 | HarmonyOS Stage 模型 / ArkUI | HarmonyOS Stage 模型 + Electron 外壳 | Android Activity + 系统 WebView |
| 宿主语言 | ArkTS | ArkTS + JavaScript（Electron 主进程） | Kotlin |
| Node 来源 | `@fellow99/node-bridge` + `@fellow99/node-runtime`（ohpm 双包） | Electron 37 内置 Node | `io.github.fellow99:node-bridge` + `node-runtime`（Maven 双包） |
| Node 版本 | **v24.2.0**（ABI 137） | **v22.17.0**（Electron 37 / Chromium 138 / ABI 138） | **v24.22.1**（arm64-v8a，16 KB 页对齐） |
| 渲染内核 | **ArkWeb** | **Chromium**（Electron；ArkWeb 目标通道未落地） | **系统 WebView** |
| 数据库 | `node:sqlite`（WAL） | `better-sqlite3`（aarch64-ohos / ABI 138） | `node:sqlite`（WAL） |
| 服务端 | opencode v2.2.5 stock CLI bundle | opencode v2.2.5 stock CLI bundle | opencode v2.2.5 stock CLI bundle |
| 补丁数（v2.2.5） | 5（0001/0003/0006/0007/0008） | 8（0001–0008） | 8（0001/0003/0006/0007/0008/0009/0010/0011） |
| 应用版本 | 2.2.5 / 20205 | 2.2.5 / 20205 | 2.2.5 / 20205 |
| 包名 | `org.fellow99.oc.OCDesktopArk` | `org.fellow99.oc.OCDesktop` | `org.fellow99.oc.OCDesktopAndroid` |
| 目标设备 | 2in1 / tablet | 2in1 / tablet | 平板/折叠屏/桌面优先；minSdk 24 / targetSdk 36 |
| 构建产物 | HAP（debug/unsigned/signed） | HAP（debug/unsigned/signed） | APK（debug/release） |

---

## 二、共享上游与基线

| 类别 | 技术 | 版本 | 用途 | 证据 |
|---|---|---|---|---|
| 上游仓库 | opencode | tag `v2.2.5` / 提交 `9bbb8ce18c` / 分支 `Branch_v2.2.5` | 服务端 + Web UI 单一来源 | `opencode` submodule；各工程 `patches/v2.2.5/README.md` |
| 上游 UI | SolidJS SPA | 随上游 | 由服务端同源托管的内嵌 UI | `packages/app`、`app-archive.bin` |
| HTTP 服务 | Effect HTTP | 随上游 | `/api/*`、`/auth/*`、SSE `/api/event` | 上游源码 |
| 数据库 schema | opencode 内置 | 随上游 | 会话/消息/凭据持久化 | 上游 `packages/core` |
| 构建工具 | bun | 1.4.x（CI 钉 1.4.2；上游守卫要求 `^1.4.2`） | `build-node.ts --bundle-only` | 构建脚本 / CI |
| 打包器 | Vite（SSR 单文件） | 随上游 | 产出 `dist-node/opencode.mjs` | 上游配置 |
| 补丁模型 | 幂等 patch + 基线守卫 | 每版别一组 | 平台差异最小化 | `patches/<tag>/`、`apply-patches.mjs` |

---

## 三、`oc-desktop-ark` 技术栈

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 平台 SDK | HarmonyOS SDK | 6.1.1.125 / API 24 | 编译/目标 |
| 构建 | hvigorw | 6.24.2 | `assembleHap` |
| 包管理 | ohpm | 随 DevEco/CLT | HAR/ohpm 依赖 |
| 运行时 | `@fellow99/node-bridge` | 0.1.0 | NAPI 引导 + `dlopen(libnode.so)` + `node::Start` |
| 引擎 | `@fellow99/node-runtime` | 24.2.0 | `libnode.so`（ABI 137） |
| 数据库 | `node:sqlite` | Node 内置 | 会话持久化（WAL） |
| 渲染 | ArkWeb `Web` 组件 | SDK 内置 | 唯一渲染通道；无 Chromium |
| 网络 | 同源 HTTP + SSE | — | 零 CORS、零 preload 桥 |
| 构建脚本 | Node.js ESM 脚本 | Node 24 host | `apply-patches` / `build-opencode` / `collect-opencode` / `smoke-node24` / `verify-runtime` |
| 插件 | `isomorphic-git`、工作区 roots | 随包 | 进程内 Git、工作区选择 |
| 兼容形态 | jitless + pure-JS fetch/photon shim | 随包 | 无 JIT 环境兼容；默认 compat 构建 |

---

## 四、`oc-desktop-hos` 技术栈

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 平台 SDK | HarmonyOS SDK | compatible 5.0.5(17) / target 6.1.1(24) | 编译/目标 |
| 构建 | hvigorw | 6.24.2 级 | `assembleHap` |
| 运行时 | Electron | 37（`v37.2.3-20260825.1`） | 主进程 Node + Chromium |
| Node | Electron 内置 | 22.17.0 | 服务端运行时 |
| 渲染 | Chromium（Electron BrowserWindow） | 138 | 当前真机通道 |
| 目标渲染 | ArkWeb（R2） | — | 目标通道；当前仅主进程分支，ArkTS 承载未落地 `⬜` |
| 数据库 | better-sqlite3 | aarch64-ohos / ABI 138 | 会话持久化 |
| 构建脚本 | Node.js ESM + PowerShell | Node 22/24 host | `collect-runtime` / `build-opencode` / `collect-app` / `smoke-node22` |
| 签名 | hvigorfile 内存注入 | debug/release sample | 签名材料不入库 |
| 插件 | Git、搜索、shell、工作区 roots | 随包 | L1 本地插件 |
| 品牌化 | `scripts/lib/branding.mjs` + `verify-branding.mjs` | 随包 | UI 标题/图标/字标 + 37 项断言 |

---

## 五、`oc-desktop-android` 技术栈

| 类别 | 技术 | 版本 | 用途 |
|---|---|---|---|
| 构建 | Gradle | 8.12 wrapper | Android 构建 |
| AGP | Android Gradle Plugin | 8.9.2 | 应用插件 |
| 语言 | Kotlin | 2.1.20（jvmTarget 17） | 宿主层 |
| SDK | compileSdk / targetSdk | 36 | 平台目标 |
| minSdk | — | 24 | 引擎硬要求 |
| ABI | — | arm64-v8a | 单一 ABI |
| 运行时 | `io.github.fellow99:node-bridge` | 0.2.0 | JNI 引导 + 宿主能力桥 |
| 引擎 | `io.github.fellow99:node-runtime` | 24.22.1 | `libnode.so`（16 KB 对齐） |
| 数据库 | `node:sqlite` | Node 内置 | 会话持久化（WAL） |
| 渲染 | 系统 WebView | 平台内置 | 唯一渲染通道 |
| 网络 | 同源 HTTP + SSE；`networkSecurityConfig` 仅放行 loopback 明文 | — | 零 CORS |
| 并发 | kotlinx-coroutines-android | 1.9.0 | 主线程调度 |
| 构建脚本 | Node.js ESM | Node 24 host | 补丁/构建/收集/冒烟/回归/品牌守卫 |
| 插件 | Git、搜索、工作区 roots、pnpm | 随包 | L1 本地插件 |
| CI | GitHub Actions | `ubuntu-latest` | tag 触发构建 + GitHub Release |

---

## 六、工具链与质量门

| 类别 | 工具 | 用途 |
|---|---|---|
| 补丁 | `scripts/apply-patches.mjs` | 基线守卫、幂等应用、反向校验 |
| 运行时锁 | `scripts/verify-runtime.mjs` + `runtime.lock.json` | `libnode.so` SHA-256/体积/ABI/16 KB 对齐 |
| 载荷 | `build-opencode.mjs` + `collect-opencode.mjs` | bundle 构建、裁剪、tar.gz、SHA-256 清单 |
| host 冒烟 | `smoke-node22.mjs`（hos）/ `smoke-node24.mjs`（ark/android） | `/api/info`、SPA、SSE、品牌 title |
| 插件 | `verify-git-plugin.mjs` / `test-harmony-plugins.mjs` | 插件语义、围栏、上限、凭据 |
| 品牌 | `verify-branding.mjs` / `branding.mjs` | 标题、图标、字标、漂移守卫 |
| 合规 | `check-acl-surface.mjs`（ark）/ ACL 文档 + 守卫 | 声明面 = 使用面 |
| 构建 | `hvigorw assembleHap` / `gradlew assembleDebug|Release` | HAP/APK |
| 真机 | `hdc` / `adb` + 截图 + UI 树 + 日志 | 端到端证据 |
| 回归 | `regression.mjs`（android）/ 回归四件套 | 包哈希、首启、包体、接口与首屏 |
| 提交 | Conventional Commits | 统一提交前缀 |

---

## 七、明确不引入 / 不可得的技术

| 技术 | 原因 |
|---|---|
| Electron / Chromium 于 ark/android | 合规与体积；ark/android 使用 ArkWeb / 系统 WebView |
| PTY / 持久终端 | 平台级不可得；补丁打桩 |
| 系统 git / `rg` / 外部 ELF 子进程 | 应用域不可执行自有 ELF；用进程内 Git / 纯 JS 搜索替代 |
| 本地 stdio MCP / LSP / formatter 外部进程 | 需要 spawn 外部进程；平台限制 |
| 在线插件安装（L2） | 安全与合规；v1 只允许构建期本地插件（L1） |
| `@parcel/watcher` | 无目标平台原生构件；禁用 filewatcher |
| `@ff-labs/fff-node` | 平台/版本不可得；禁用 FFF |
| 后台常驻 / 开机自启 / 托盘 | v1 非目标；部分平台声明但未使用，需如实标注 |
| 云账号 / 跨设备同步 | 非目标；数据留本机 |

---

## 八、版本矩阵与同步落点

| 版本 | 值 | 落点 |
|---|---|---|
| 应用版本 | `2.2.5` / `20205` | `AppScope/app.json5`（hos/ark）、`app/build.gradle.kts`（android） |
| 上游基线 | `v2.2.5` / `9bbb8ce18c` | `opencode` submodule、`patches/v2.2.5/`、`apply-patches.mjs` 的基线守卫 |
| Node（ark） | `v24.2.0` / ABI 137 | `runtime.lock.json`、`oh-package.json5` |
| Node（hos） | `v22.17.0` / ABI 138 | Electron 版本、README、规格 |
| Node（android） | `v24.22.1` / arm64-v8a | `runtime.lock.json`、`gradle/libs.versions.toml` |
| bun | `1.4.x`（CI 1.4.2） | 构建脚本、CI YAML |
| Android SDK | compile/target 36 / min 24 | `gradle/libs.versions.toml` |
| HarmonyOS SDK | 6.1.1(24) | `build-profile.json5` |

> ⚠️ 已知文档漂移：`oc-desktop-android/specs/TECH.md` 与 `docs/000-工程规划.md` 仍以 v2.1.2 为叙述基线，而源码/README/补丁已是 v2.2.5；`oc-desktop-android/.github/workflows/build-release.yml` 仍检出 `a517687fde`（v2.1.2）。以源码与 `patches/v2.2.5/` 为准，规格整理时在 `011-version-2.2.5` 登记为待收敛项。
