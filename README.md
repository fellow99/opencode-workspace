# opencode-workspace

> **opencode 的 HarmonyOS 工作区**（submodule 容器）：固定上游基线，聚合 **两个鸿蒙版别**、
> 进程内 Node 运行时库与产品官网。所有子工程共享同一个上游 `opencode`，互不复制源码。

## 一、两个鸿蒙版别（统一口径）

| 版别 | 工程 | 运行时与渲染 | 包名 |
|---|---|---|---|
| **HarmonyOS Edition** | [`oc-desktop-ark`](oc-desktop-ark) | **进程内 Node v24.2.0**（`@fellow99/node-bridge`）+ **ArkWeb**；无 Electron、无 Chromium | `org.fellow99.oc.OCDesktopArk` |
| **Electron on HarmonyOS Edition** | [`oc-desktop-hos`](oc-desktop-hos) | **Electron 37**（Node 22.17.0 / Chromium 138）+ ArkWeb / Chromium 渲染 | `org.fellow99.oc.OCDesktop` |

两者是**同一产品的两个独立版别**，可同机共存：

- 同名上游基线：opencode **v2.0.26**（tag `v2.0.26` / 提交 `9b4ec5714d` / 分支 `Branch_v2.0.26`）；
- 同一份官方 Web UI：都由服务端**同源托管**，零前端分叉；
- 差异只在「**谁提供 Node**、**谁渲染**」：Ark 版用进程内 Node + ArkWeb；Electron 版用 Electron 自带 Node + Electron 窗口。

> 两者均**非 opencode 团队构建**，与上游无隶属关系；opencode 为 MIT 开源项目，本工作区仅做第三方移植。

## 二、子工程

| 工程 | 定位 | 一句话 |
|---|---|---|
| [`opencode`](opencode) | 上游（submodule，**只读基线**） | opencode v2.0.26 源码；两个版别只对它做**幂等补丁**，不建立长期分叉。 |
| [`ohos-node-bridge`](ohos-node-bridge) | **运行时库**（HAR + NAPI） | `@fellow99/node-bridge`：把**真正的 Node.js** 以 `NAPI 引导 + dlopen + node::Start` 嵌进 HarmonyOS 应用**自己的进程**，封装为 ArkTS 组件库；同时提供运行时产物（`runtime/`：`libnode-arm64.so` v24.2.0 / ABI 137）、`provision.mjs` 投放工具与七项 ELF 门禁。**它不是应用**，是 HarmonyOS Edition 的底座。 |
| [`oc-desktop-ark`](oc-desktop-ark) | **HarmonyOS Edition**（应用） | 无 Electron、无 Chromium：`EntryAbility` 把 Node 运行时投进应用进程，Node 侧以 `serve` 语义跑 opencode 服务端（同源托管 `/api/*` 与内嵌 Web UI），ArkWeb 加载 `http://127.0.0.1:<port>/`。含 `specs/`（9 个模块 + 项目级规格）、`payload/`（Node 宿主入口）、`patches/v2.0.26/`（0001/0003/0006）与构建流水线。 |
| [`oc-desktop-hos`](oc-desktop-hos) | **Electron on HarmonyOS Edition**（应用） | 用 `harmonypc-electron` 提供的 Electron-on-HarmonyOS 运行时承载 opencode 服务端；已构建、签名，并在真机完成安装启动与基本流程验证（v2.0.26，证据见 `oc-desktop-hos/docs/鸿蒙环境能力清单-v2.0.26.md` §10 / §11.2）。含 8 条上游补丁（`patches/v2.0.26/0001–0008`）与 ACL / 能力清单 / 真机记录。 |
| [`harmonypc-electron`](harmonypc-electron) | Electron-on-HarmonyOS 运行时（submodule） | Electron 鸿蒙外壳与 `libelectron.so` 适配层；**只有 Electron on HarmonyOS Edition 使用**，HarmonyOS Edition **不依赖**它。 |
| [`oc-desktop-website`](oc-desktop-website) | 产品官网（submodule） | 纯静态落地页（中英双语）：两个鸿蒙版别的定位、能力与仓库入口。 |
| `assets/` | 品牌物料 | Logo、截图等（官网与安装包共用）。 |
| `harmonypc-electron-versions/` | 本地材料（gitignore） | Electron 发行版归档与原生模块编译材料；不入库。 |

## 三、责任分界

```
opencode（上游，v2.0.26）
   │  幂等补丁（各自 patches/，互不共用）
   ├── oc-desktop-ark ── 依赖 ──> ohos-node-bridge（Node v24.2.0，进程内）
   └── oc-desktop-hos ── 依赖 ──> harmonypc-electron（Electron 37）
                                        │
                                   oc-desktop-website（对外统一口径）
```

- **运行时契约**以 `ohos-node-bridge/docs/000-工程规划.md` 为准；
- **两个版别的设计依据**分别是 `oc-desktop-ark/docs/000-工程规划.md` 与 `oc-desktop-hos/docs/工程规划-V2.md`；
- `../opencode` 是**两个版别共用**的 submodule：打补丁前须校验基线 `9b4ec5714d` 且工作树干净（`scripts/apply-patches.mjs` 已内置守卫），**同一时刻只允许一个版别处于「已打补丁」状态**。

## 四、快速开始

```bash
git clone --recurse-submodules https://github.com/fellow99/opencode-workspace.git
cd opencode-workspace

# HarmonyOS Edition（进程内 Node + ArkWeb）
cd oc-desktop-ark
./scripts/build-bridge-har.sh          # 兄弟工程 → libs/@fellow99/node-bridge.har
./scripts/provision-runtime.sh         # ../ohos-node-bridge/runtime → entry/libs/arm64-v8a/
node scripts/apply-patches.mjs         # 0001/0003/0006（会弄脏 ../opencode）
OPENCODE_NODE_TARGET=node24 node scripts/build-opencode.mjs
node scripts/collect-opencode.mjs      # → resfile/opencode（≈62 MB）
node scripts/smoke-node24.mjs --allow-version-drift
```

Electron on HarmonyOS Edition 的构建链见 [`oc-desktop-hos/README.md`](oc-desktop-hos/README.md)。

## 五、许可

MIT（各子工程自带 `LICENSE`）。随包分发 Node.js / Electron 时须附各自的第三方许可清单
（Ark 版见 `oc-desktop-ark/specs/107-compliance`，Electron 版见 `oc-desktop-hos/docs/ACL申请清单.md` 与包内许可文件）。
