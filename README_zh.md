# OC Desktop

> [English](./README.md)

**OC Desktop 是 [opencode](https://opencode.ai) 的桌面端。**

Windows / Linux / macOS 已有上游官方桌面发行版；本工作区聚焦把同一份体验带到移动平台 —— 面向鸿蒙与 Android 的**三个独立第三方版别**，均在应用进程内运行 opencode 服务端、同源加载官方 Web UI：

| 版别 | 仓库 | 运行时 | 渲染内核 |
|---|---|---|---|
| **HarmonyOS Edition** | [`oc-desktop-ark`](./oc-desktop-ark) | 进程内 Node v24.2 | ArkWeb（无 Electron、无 Chromium） |
| **Electron on HarmonyOS Edition** | [`oc-desktop-hos`](./oc-desktop-hos) | Electron 37（Node 22 / Chromium 138） | Chromium |
| **Android Edition** | [`oc-desktop-android`](./oc-desktop-android) | 进程内 Node v24.2 | 系统 WebView |

三版均为第三方移植，与上游 opencode 团队无隶属关系。

- 产品官网：<https://fellow99.github.io/oc-desktop-website/>（[仓库](./oc-desktop-website)）
- 上游：[`opencode`](https://github.com/anomalyco/opencode)（submodule，分支 `v2`）

> 🚧 本仓库是移植与工程化实践，不是产品发布：尚未在应用商店分发，安装方式为侧载（HAP / APK）。

---

## 一、三个版别

### 1.1 HarmonyOS Edition —— `oc-desktop-ark`

**进程内 Node + ArkWeb，不带 Electron、不带 Chromium。**

- Node v24.2 经 [`@fellow99/node-bridge`](https://github.com/fellow99/ohos-node-bridge)（`dlopen` + `node::Start`）运行在鸿蒙应用进程内；ArkWeb 从 `127.0.0.1:<随机端口>` 同源加载官方 Web UI。
- 会话用内建 `node:sqlite`（零补丁）；载荷经 resfile 只读直读。
- 默认 **jitless 兼容形态**（纯 JS fetch 垫片 / photon 垫片 / wasm 降级）。
- 目标设备：2in1 / 平板。

### 1.2 Electron on HarmonyOS Edition —— `oc-desktop-hos`

**Electron 37 跑在鸿蒙上（Node 22 / Chromium 138）。**

- Electron 主进程进程内托管 opencode 服务端；官方 Web UI 前端 100% 复用，并做完整品牌化。
- 持久化走 better-sqlite3（aarch64-ohos 原生模块）；载荷首启从 `tar.gz` 流式解压。
- 纯 JS 工具插件覆盖 `shell` / `grep` / `glob`；另含 Git 插件与四目录工作区选择器。
- 目标设备：2in1 / 平板。

### 1.3 Android Edition —— `oc-desktop-android`

**进程内 Node + 系统 WebView，不自带浏览器内核。**

- Node v24.2 经 `io.github.fellow99:node-bridge` 运行在应用进程内；系统 WebView 从 `127.0.0.1:<随机端口>` 同源加载界面。
- 会话用内建 `node:sqlite`；`assets` tar 首启由 Node 自解包。
- 四个进程内插件：Git、搜索、工作区根（SAF 选树导入 + `setCurrent`）与 **pnpm**。
- 运行时 16KB 页对齐；CI 打 `v*` tag 即构建 debug APK 并发 GitHub Release。
- 目标设备：平板 / 折叠屏 / 桌面形态优先（手机可用；minSdk 24 / targetSdk 36）。

### 1.4 Windows / Linux / macOS

使用**上游 opencode 官方桌面发行版（BETA）**：<https://opencode.ai/zh/download>

源码：<https://github.com/anomalyco/opencode/tree/v2>

---

## 二、统一基线（v2.1.2）

三个移动版别共享同一上游基线与工程契约：

- **上游**：opencode **v2.1.2**（`a517687fde`，分支 `v2`），源码核实。
- **官方 Web UI 100% 复用**：应用内服务端与上游同一份 `opencode serve` bundle、同一份 Web UI。
- **幂等补丁模型**：补丁编号管理（`0001`–`0010`），每条附 README 说明上游为何不能收；构建在干净树上重放，脏树即失败。
  - ark 5 条（`0001/0003/0006/0007/0008`）· hos 8 条（`0001`–`0008`）· android 7 条（`0001/0003/0006/0007/0008/0009/0010`）。
- **仅本地插件（L1）**：不做在线安装；插件构建期物化（copy 禁 symlink，digest 幂等、篡改自愈）。
- **统一门禁**：固定 bun 构建上游 bundle、运行时锁值（SHA / 体积）、host 侧 smoke（node24 / node22）、载荷指纹、品牌化断言。

「本机随机端口 + 每启动一密码 + Web UI 同源加载」的架构与上游官方桌面应用同源；三版差异只在「谁提供 Node、谁渲染、平台缺口怎么补」。

---

## 三、能力对照清单

三个移动版别的简化视图；逐项到源码/测试报告核实的完整版见 [`docs/能力对照清单.md`](./docs/能力对照清单.md)。

**图例**：✅ 已实现并实测 · 🔶 部分 / 受限 / 待验 · ⬜ 未投放 · ❌ 平台不可得

| 能力项 | Android | 鸿蒙 Ark | 鸿蒙 Electron |
|---|---|---|---|
| 流式对话 / SSE | ✅ | ✅ | ✅ |
| 图片附件 / 看图分析 | ✅ | ✅ | ⬜ |
| 文件读写编辑 | ✅ | ✅ | ✅ |
| Shell 命令 | ✅ | ✅ | ✅ |
| grep / glob 搜索 | ✅ | 🔶 | ✅ |
| Git（isomorphic-git 插件） | ✅ | ✅ | ✅ |
| 工作区选择器 | ✅ | ✅ | ✅ |
| pnpm 进程内 | ✅ | ❌ | ❌ |
| SQLite 会话 | ✅ | ✅ | ✅ |
| 重启后会话持久 | ✅ | ✅ | 🔶 |
| MCP remote（HTTP/SSE） | 🔶 | 🔶 | 🔶 |
| MCP local（stdio） | ❌ | ❌ | ❌ |
| 终端 PTY | ❌ | ❌ | ❌ |
| Web UI 品牌化 | ✅ | ❌ | ✅ |
| 导航围栏 / 外链分流 | ✅ | ⬜ | ✅ |
| 宿主通知 | ✅ | ⬜ | ⬜ |
| 后台保活 | 🔶 | ❌ | 🔶 |
| jitless 兼容形态 | ⬜ | ✅ | ⬜ |
| 插件在线安装 | ❌ | ❌ | ❌ |
| 渲染内核 | 系统 WebView | ArkWeb | Chromium |
| 真机证据充分度 | 🔶 | ✅ | 🔶 |

平台共同边界：无终端 PTY、不能 spawn 自带 node ELF / 本地 stdio MCP / LSP、无插件在线安装、无开机自启；系统 `sh`/toybox 命令与进程内 Git 可用。

---

## 四、仓库结构

| 路径 | 职责 |
|---|---|
| [`opencode/`](./opencode) | 上游 submodule（分支 `v2`，只读；钉 v2.1.2） |
| [`oc-desktop-ark/`](./oc-desktop-ark) | HarmonyOS Edition（进程内 Node + ArkWeb） |
| [`oc-desktop-hos/`](./oc-desktop-hos) | Electron on HarmonyOS Edition |
| [`oc-desktop-android/`](./oc-desktop-android) | Android Edition（进程内 Node + 系统 WebView） |
| [`oc-desktop-website/`](./oc-desktop-website) | 产品官网（静态落地页，中英双语） |
| [`harmonypc-electron/`](./harmonypc-electron) | 参考：Electron-on-HarmonyOS 外壳 |
| [`assets/`](./assets) | 共享品牌素材（logo、截图） |
| [`docs/`](./docs) | 跨版别文档；见 [`docs/能力对照清单.md`](./docs/能力对照清单.md) |

每个版别是独立仓库、独立生命周期；本工作区以 submodule 方式钉住各自版本。

---

## 五、开发

环境：Windows 主机、Node 24、bun 1.4.x；鸿蒙两版另需 DevEco Studio + HarmonyOS SDK，Android 版需 Android SDK（JDK 17）。

```bash
# 连同 submodule 一起克隆
git clone --recursive https://github.com/fellow99/opencode-workspace.git

# 各版在各自仓库内构建，见各自 README：
#   oc-desktop-ark/README.md
#   oc-desktop-hos/README.md
#   oc-desktop-android/README.md
```

上游 `opencode/` 工作树**只读**：所有改动都是各版 `patches/` 目录里的幂等补丁，不提交进 submodule。

---

## 六、许可证

MIT —— 本工作区与三个版别仓库。随包上游 opencode 遵循其自身许可证；鸿蒙两版另内嵌第三方组件（Electron / Chromium / Node.js），许可声明随包分发。
