# OC Desktop

> [简体中文](./README_zh.md)

**OC Desktop is the desktop form of [opencode](https://opencode.ai).**

opencode already ships an official desktop app for Windows / Linux / macOS. This workspace focuses on bringing the same experience to mobile platforms — **three independent third-party editions** for HarmonyOS and Android, all running the in-process opencode server and loading the official Web UI same-origin:

| Edition | Repository | Runtime | Renderer |
|---|---|---|---|
| **HarmonyOS Edition** | [`oc-desktop-ark`](./oc-desktop-ark) | in-process Node v24.2 | ArkWeb (no Electron, no Chromium) |
| **Electron on HarmonyOS Edition** | [`oc-desktop-hos`](./oc-desktop-hos) | Electron 37 (Node 22 / Chromium 138) | Chromium |
| **Android Edition** | [`oc-desktop-android`](./oc-desktop-android) | in-process Node v24.2 | system WebView |

All three are third-party ports and are not affiliated with the upstream opencode team.

- Product website: <https://fellow99.github.io/oc-desktop-website/> ([repo](./oc-desktop-website))
- Upstream: [`opencode`](https://github.com/anomalyco/opencode) (submodule, branch `v2`)

> 🚧 This is a porting / engineering effort, not a product release. No app-store distribution yet; packages are installed by side-loading (HAP / APK).

---

## 1. Editions

### 1.1 HarmonyOS Edition — `oc-desktop-ark`

**In-process Node + ArkWeb, no Electron and no Chromium.**

- Node v24.2 runs inside the HarmonyOS app process via [`@fellow99/node-bridge`](https://github.com/fellow99/ohos-node-bridge) (`dlopen` + `node::Start`); ArkWeb loads the official Web UI same-origin from `127.0.0.1:<random port>`.
- Sessions use built-in `node:sqlite` (no patches); the payload is served read-only from resfile.
- Default **jitless compatibility build** (pure-JS fetch shim / photon shim / wasm fallback).
- Targets 2in1 / tablet devices.

### 1.2 Electron on HarmonyOS Edition — `oc-desktop-hos`

**Electron 37 on HarmonyOS (Node 22 / Chromium 138).**

- The Electron main process hosts the opencode server in-process; the official Web UI is reused 100% and fully rebranded.
- Persistence via better-sqlite3 (aarch64-ohos native module); payload streamed from `tar.gz` on first launch.
- Plain-JS tool plugins cover `shell` / `grep` / `glob`; a Git plugin and a four-directory workspace selector are included.
- Targets 2in1 / tablet devices.

### 1.3 Android Edition — `oc-desktop-android`

**In-process Node + system WebView, no bundled browser engine.**

- Node v24.2 runs inside the app via `io.github.fellow99:node-bridge`; the system WebView loads the UI same-origin from `127.0.0.1:<random port>`.
- Sessions use built-in `node:sqlite`; the `assets` tar is self-extracted by Node on first launch.
- Four in-process plugins: Git, search, workspace roots (SAF tree import + `setCurrent`), and **pnpm**.
- 16 KB-page-aligned runtime; CI builds a debug APK and publishes a GitHub Release on `v*` tags.
- Targets tablets / foldables / desktop form factors (phones work too; minSdk 24 / targetSdk 36).

### 1.4 Windows / Linux / macOS

Use the **official upstream opencode desktop release (BETA)**: <https://opencode.ai/zh/download>

Source: <https://github.com/anomalyco/opencode/tree/v2>

---

## 2. Shared baseline (v2.1.2)

All three mobile editions share the same upstream baseline and engineering contract:

- **Upstream**: opencode **v2.1.2** (`a517687fde`, branch `v2`), source-verified.
- **Official Web UI 100% reused**: the in-app server serves the same `opencode serve` bundle and the same Web UI as upstream.
- **Idempotent patch model**: patches are numbered (`0001`–`0010`), each with a README documenting why upstream cannot accept it; builds re-apply them on a clean checkout and fail on a dirty tree.
  - ark: 5 patches (`0001/0003/0006/0007/0008`) · hos: 8 patches (`0001`–`0008`) · android: 7 patches (`0001/0003/0006/0007/0008/0009/0010`).
- **Local plugins only (L1)**: no online plugin installation; plugins are materialized at build time (copy, never symlink; digest-checked, tamper-self-healing).
- **Common gates**: upstream bundle built with pinned bun, runtime checksum/size locks, host-side smoke (node24 / node22), payload fingerprint, branding assertions.

The common architecture (random loopback port + per-launch password + same-origin Web UI) is also carried by the official upstream desktop app; the three editions differ only in "who supplies Node, who renders, and how platform gaps are filled".

---

## 3. Capability matrix

Simplified view of the three mobile editions; the fully source-traced matrix lives in [`docs/能力对照清单.md`](./docs/能力对照清单.md).

**Legend**: ✅ available & verified · 🔶 partial / limited / pending verification · ⬜ not shipped · ❌ unavailable on the platform

| Capability | Android | HarmonyOS (ark) | Electron on HOS (hos) |
|---|---|---|---|
| Streaming chat / SSE | ✅ | ✅ | ✅ |
| Image attachment / vision | ✅ | ✅ | ⬜ |
| File read / write / edit | ✅ | ✅ | ✅ |
| Shell commands | ✅ | ✅ | ✅ |
| grep / glob search | ✅ | 🔶 | ✅ |
| Git (isomorphic-git plugin) | ✅ | ✅ | ✅ |
| Workspace picker | ✅ | ✅ | ✅ |
| pnpm in-process | ✅ | ❌ | ❌ |
| SQLite sessions | ✅ | ✅ | ✅ |
| Session persistence across restart | ✅ | ✅ | 🔶 |
| MCP remote (HTTP/SSE) | 🔶 | 🔶 | 🔶 |
| MCP local (stdio) | ❌ | ❌ | ❌ |
| Terminal PTY | ❌ | ❌ | ❌ |
| Web UI branding | ✅ | ❌ | ✅ |
| Navigation fencing / external links | ✅ | ⬜ | ✅ |
| Host notifications | ✅ | ⬜ | ⬜ |
| Background keep-alive | 🔶 | ❌ | 🔶 |
| JIT-less compatibility form | ⬜ | ✅ | ⬜ |
| Online plugin installation | ❌ | ❌ | ❌ |
| Renderer | system WebView | ArkWeb | Chromium |
| Real-device evidence | 🔶 | ✅ | 🔶 |

Shared platform limits: no terminal PTY, no spawning external node ELF / local stdio MCP / LSP, no online plugin installation, no auto-start at boot. System `sh`/toybox commands and in-process Git are available.

---

## 4. Repository layout

| Path | Role |
|---|---|
| [`opencode/`](./opencode) | Upstream submodule (branch `v2`, read-only; pin v2.1.2) |
| [`oc-desktop-ark/`](./oc-desktop-ark) | HarmonyOS Edition (in-process Node + ArkWeb) |
| [`oc-desktop-hos/`](./oc-desktop-hos) | Electron on HarmonyOS Edition |
| [`oc-desktop-android/`](./oc-desktop-android) | Android Edition (in-process Node + system WebView) |
| [`oc-desktop-website/`](./oc-desktop-website) | Product website (static landing page, EN/中文) |
| [`harmonypc-electron/`](./harmonypc-electron) | Reference: Electron-on-HarmonyOS shell |
| [`assets/`](./assets) | Shared brand assets (logo, screenshots) |
| [`docs/`](./docs) | Cross-edition docs; see [`docs/能力对照清单.md`](./docs/能力对照清单.md) |

Each edition is an independent repository with its own lifecycle; this workspace pins them as git submodules.

---

## 5. Development

Prerequisites: Windows host, Node 24, bun 1.4.x; HarmonyOS editions additionally need DevEco Studio + HarmonyOS SDK, and the Android edition needs the Android SDK (JDK 17).

```bash
# clone with submodules
git clone --recursive https://github.com/fellow99/opencode-workspace.git

# each edition is built in its own repo; see its README:
#   oc-desktop-ark/README.md
#   oc-desktop-hos/README.md
#   oc-desktop-android/README.md
```

The upstream `opencode/` tree is **read-only**: all changes are idempotent patches maintained inside each edition's `patches/` directory, never committed into the submodule.

---

## 6. License

MIT — this workspace and the three edition repos. The bundled upstream opencode follows its own license; see each project for details. The HarmonyOS editions additionally embed third-party components (Electron / Chromium / Node.js), whose notices are shipped inside each package.
