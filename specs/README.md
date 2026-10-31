# OC Desktop 规格文档索引

**项目名称：** OC Desktop（父工程 `opencode-workspace`）
**版本：** 2.2.5 / `versionCode 20205`（三版一致）
**上游基线：** opencode **v2.2.5**（`9bbb8ce18c` / `Branch_v2.2.5`）
**技术栈：** ArkTS + ArkWeb / Electron 37 + Chromium / Kotlin + 系统 WebView；进程内 Node；`node:sqlite` / better-sqlite3
**文档生成方式：** `specs-as-built`（源码、构建脚本、补丁、真机证据为事实源）
**最后更新：** 2026-11-01

---

## 一、项目介绍

OC Desktop 把开源 AI 编码代理 **opencode** 的桌面体验带到 HarmonyOS 与 Android 设备上，作为**本机应用**运行：应用内自带服务端与 Web UI，同源加载上游 opencode v2 Web UI，数据默认留在本机，只有模型 API 出网。

| 版别 | 工程 | 运行时 | 渲染 | 产物 |
|---|---|---|---|---|
| HarmonyOS Edition | `oc-desktop-ark` | 进程内 Node 24.2（无 Electron） | ArkWeb | HAP |
| Electron on HarmonyOS Edition | `oc-desktop-hos` | Electron 37 / Node 22.17 | Chromium（ArkWeb R2 未落地） | HAP |
| Android Edition | `oc-desktop-android` | 进程内 Node 24.22 | 系统 WebView | APK |

三版共享同一上游基线、同一 stock CLI bundle、同一补丁/插件治理与编号体系；差异只在“谁提供 Node、谁负责渲染、平台缺口怎么补”。详见根 [`README.md`](../README.md) 与 [`docs/能力对照清单.md`](../docs/能力对照清单.md)。

---

## 二、文档总览

| 层级 | 分类 | 文档数量 | 说明 |
|---|---|---|---|
| 父工程 | 项目级顶层文档 | 8 | README / STRUCTURE / TECH / ARCHITECTURE / constitution / overall-spec / overall-plan / REVIEW_REPORT |
| 父工程 | 公共模块 `0XX` | 3 目录 / 6 文件 | 011 版本、012 构建工作流、013 品牌化 |
| 子工程 | `oc-desktop-hos/specs` | 20 模块目录 | 公共 + 基础 + 插件 |
| 子工程 | `oc-desktop-ark/specs` | 14 模块目录 | 公共 + 基础 + 插件 |
| 子工程 | `oc-desktop-android/specs` | 17 模块目录 | 公共 + 基础 + 插件 |
| **合计** | **父 14 文件 + 子 51 模块目录** | | 以实际目录为准 |

> 按需求约定：父工程**不生成** `API.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`。API 契约以上游 `/api/*`（OpenAPI `/doc`）为准。

---

## 三、父工程项目级文档

| 文档 | 路径 | 说明 |
|---|---|---|
| **项目结构** | [STRUCTURE.md](./STRUCTURE.md) | 父/子工程目录树、入口、specs 模块清单、关键路径 |
| **技术选型** | [TECH.md](./TECH.md) | 三版运行时/渲染/数据库/构建/版本矩阵、明确排除项、质量门 |
| **整体架构** | [ARCHITECTURE.md](./ARCHITECTURE.md) | 系统上下文、分层、进程模型、启动时序、数据流、补丁/插件、构建拓扑、风险 |
| **宪法原则** | [constitution.md](./constitution.md) | 文档分工、`0XX/1XX/2XX` 编号规则、证据/安全/提交/评审/质量门 |
| **整体规格** | [overall-spec.md](./overall-spec.md) | 技术无关的系统目的、用户故事、FR（RUNTIME/SERVICE/UI/AUTH/HOST/WORKSPACE/VCS/PLUGIN/VERSION/BRAND/BUILD/COMPLIANCE） |
| **整体方案** | [overall-plan.md](./overall-plan.md) | 技术上下文、宪法合规、三版实现策略、公共模块落地、横切、测试、部署、风险 |
| **评审报告** | [REVIEW_REPORT.md](./REVIEW_REPORT.md) | 本次规格整理的 code review 范围、发现、处置与结论 |
| **索引** | README.md | 本文件 |

---

## 四、父工程公共模块（`0XX`）

### 011 — 版本更新（Version）

> 跨版别同步上游基线、应用版本、补丁矩阵、载荷门禁与证据。历史 `101-version-*` 已统一为 `011-version-*`。

| 文档 | 路径 | 说明 |
|---|---|---|
| 功能规格 | [011-version-2.2.5/spec.md](./011-version-2.2.5/spec.md) | 基线/版本号/补丁/载荷/证据要求（FR-011-*） |
| 技术方案 | [011-version-2.2.5/plan.md](./011-version-2.2.5/plan.md) | 三版升级步骤、命令、待收敛项 |

### 012 — 构建工作流（Building Workflow）

> Android 已有 GitHub Actions tag 构建 + Release；**鸿蒙工程暂无**等价云端工作流，当前为本地构建。

| 文档 | 路径 | 说明 |
|---|---|---|
| 功能规格 | [012-building-workflow/spec.md](./012-building-workflow/spec.md) | 上游/补丁/载荷/门禁/签名/发布要求 |
| 技术方案 | [012-building-workflow/plan.md](./012-building-workflow/plan.md) | Android CI 实际步骤、鸿蒙本地流水线、已知漂移 |

### 013 — 品牌化（Branding）

> 统一显示名 **OC Desktop**、UI 标题、图标、非官方声明与守卫；hos/android 已实现，ark 为缺口。

| 文档 | 路径 | 说明 |
|---|---|---|
| 功能规格 | [013-branding/spec.md](./013-branding/spec.md) | 应用身份/UI/图标/守卫/缺口（FR-013-*） |
| 技术方案 | [013-branding/plan.md](./013-branding/plan.md) | 三版落点、脚本/补丁、ark 补齐方案、风险 |

---

## 五、子工程规格索引

### 5.1 `oc-desktop-hos`（Electron on HarmonyOS）

> 20 个模块目录：`011` 版本（3 个版本）、`013` 品牌、`101–115` 基础、`201–207` 插件。

**索引：** [`oc-desktop-hos/specs/README.md`](../oc-desktop-hos/specs/README.md)

| 分类 | 模块 |
|---|---|
| 公共 | `011-version-2.0.26`、`011-version-2.1.2`、`011-version-2.2.5`、`013-branding` |
| 基础 | `101-runtime-host`、`102-artifact-bootstrap`、`103-render-channel`、`105-request-hooks`、`108-signing-identity`、`111-lifecycle`、`112-build-pipeline`、`113-server-ui-load`、`114-workspace-auth`、`115-data-dirs-sqlite` |
| 插件 | `201-plugin-runtime`、`202-skill-runtime-capabilities`、`203-plugin-workspace-picker`、`204-plugin-git`、`206-js-tools`、`207-plugin-shell` |

### 5.2 `oc-desktop-ark`（HarmonyOS Ark）

> 14 个模块目录：`011` 版本（2 个版本）、`013` 品牌（gap）、`101–110` 基础、`203–204` 插件。

**索引：** [`oc-desktop-ark/specs/README.md`](../oc-desktop-ark/specs/README.md)

| 分类 | 模块 |
|---|---|
| 公共 | `011-version-2.1.2`、`011-version-2.2.5`、`013-branding` |
| 基础 | `101-runtime-integration`、`102-payload-pipeline`、`103-arkweb-shell`、`104-host-capabilities`、`105-auth`、`106-upstream-adaptation`、`107-compliance`、`109-regression`、`110-jitless` |
| 插件 | `203-plugin-workspace-picker`、`204-plugin-git` |

### 5.3 `oc-desktop-android`（Android）

> 17 个模块目录：`011` 版本（2 个版本）、`012` 构建、`013` 品牌、`101–110` 基础、`203–206` 插件。

**索引：** [`oc-desktop-android/specs/README.md`](../oc-desktop-android/specs/README.md)

| 分类 | 模块 |
|---|---|
| 公共 | `011-version-2.1.2`、`011-version-2.2.5`、`012-building-workflow`、`013-branding` |
| 基础 | `101-runtime-integration`、`102-payload-pipeline`、`103-webview-shell`、`104-host-capabilities`、`105-auth`、`106-upstream-adaptation`、`107-compliance`、`109-regression`、`110-jitless` |
| 插件 | `203-plugin-workspace-picker`、`204-plugin-git`、`205-pnpm-plugin`、`206-js-search` |

---

## 六、模块编号一览

| 编号 | 语义 | 父工程 | hos | ark | android |
|---|---|---|---|---|---|
| `011` | 版本更新 | `011-version-2.2.5` | ✅ 3 版本 | ✅ 2 版本 | ✅ 2 版本 |
| `012` | 构建工作流 | `012-building-workflow` | —（本地构建） | —（本地构建） | ✅ `012-building-workflow` |
| `013` | 品牌化 | `013-branding` | ✅ | ❌ gap | ✅ |
| `101` | 运行时接入/宿主 | — | ✅ | ✅ | ✅ |
| `102` | 载荷流水线/产物就位 | — | ✅ | ✅ | ✅ |
| `103` | 渲染壳 | — | ✅ | ✅ | ✅ |
| `104` | 宿主能力 | — | 由 `114` 等分担 | ✅ | ✅ |
| `105` | 鉴权/凭据 | — | ✅ | ✅ | ✅ |
| `106` | 上游适配 | — | 由 `112`/`204` 分担 | ✅ | ✅ |
| `107` | 合规 | — | 规划/ACL 文档 | ✅ | ✅ |
| `108` | 签名与身份 | — | ✅ | — | — |
| `109` | 回归基线 | — | `011` 真机记录 | ✅ | ✅ |
| `110` | jitless/诊断 | — | 不适用 | ✅ | ✅ |
| `111–115` | 生命周期/构建/UI 加载/工作区/数据 | — | ✅ | — | — |
| `201–207` | 插件/补丁 | — | 201–207 | 203–204 | 203–206 |

> 编号规则、迁移表与“不回填/不复用”约束见 [constitution.md](./constitution.md) §二。旧编号仅保留在迁移表/历史说明中。

---

## 七、模块文档结构规范

每个模块目录 `NNN-name/` 至少包含：

| 文件 | 说明 |
|---|---|
| `spec.md` | 做什么 + 验收判据；技术无关；MUST/SHOULD/MAY；FR 编号 |
| `plan.md` | 怎么做 + 落点 + 验证；技术具体；回引 spec FR |
| `tasks.md` | 可选：任务拆解 |
| `test-cases.md` | 可选：测试用例 |
| `as-built.md` | 可选：as-built 证据/修正 |

硬要求：

- `spec.md` 与 `plan.md` **必须成对**；只有一半视为未完成。
- 目录名 = `<三位编号>-<kebab-name>`；新增模块追加到对应段，不回填。
- 需求编号统一 `FR-<模块编号>-<三位序号>`；历史 `R1..Rn` 例外须在模块头部登记 `Rn ↔ FR-<模块>-00n`。
- 事实性断言必须带证据标记：`✅` / `🔶` / `⬜` / `⚠️` / `❌`。

---

## 八、快速导航

| 目标读者 | 推荐阅读顺序 |
|---|---|
| **新加入开发者** | [constitution.md](./constitution.md) → [STRUCTURE.md](./STRUCTURE.md) → [overall-spec.md](./overall-spec.md) → 对应子工程 `README.md` → `101/103` spec |
| **架构师 / Tech Lead** | [ARCHITECTURE.md](./ARCHITECTURE.md) → [TECH.md](./TECH.md) → [overall-plan.md](./overall-plan.md) → 子工程 `ARCHITECTURE.md` |
| **构建 / 发布** | [012-building-workflow/](./012-building-workflow/) → [011-version-2.2.5/](./011-version-2.2.5/) → 子工程构建脚本 |
| **鸿蒙开发** | [oc-desktop-hos/specs/README.md](../oc-desktop-hos/specs/README.md) / [oc-desktop-ark/specs/README.md](../oc-desktop-ark/specs/README.md) |
| **Android 开发** | [oc-desktop-android/specs/README.md](../oc-desktop-android/specs/README.md) |
| **测试 / QA** | 子工程 `overall-test-cases.md` / `109-regression` / `011-version-*` 真机记录 → [REVIEW_REPORT.md](./REVIEW_REPORT.md) |
| **产品 / 合规** | [overall-spec.md](./overall-spec.md) → [013-branding/](./013-branding/) → 子工程 `107-compliance` |

---

## 九、当前已知风险（详见 REVIEW_REPORT）

| # | 项 | 状态 |
|---|---|---|
| 1 | Android CI 仍锁 v2.1.2 `a517687fde`，源码/补丁已 v2.2.5 | ⚠️ 发布前必须更新并彩排 |
| 2 | ark Web UI 未品牌化（`<title>` 仍 `OpenCode`） | ❌ 见 `013-branding` 缺口 |
| 3 | ark/hos v2.2.5 仅主机编译，缺真机证据 | ⬜ 有设备时补跑 |
| 4 | ark 应用名 `OC Desktop Ark` 是否保留后缀 | `[NEEDS CLARIFICATION]` |
| 5 | hos 随机端口导致渲染层状态跨重启丢失 | ⚠️ 已登记，未修 |
| 6 | 端口随机与 localStorage 在 ark/android 未专项验证 | ⬜ 待补用例 |

---

**文档维护者：** OC Desktop 开发团队
