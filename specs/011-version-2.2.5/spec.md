# 011 — 版本更新：2.2.5 功能规格

> **Module:** 011-version-2.2.5 ｜ **Status:** Implemented ｜ **Last Updated:** 2026-11-01
> **需求名称:** 版本更新：2.2.5（跨版别公共模块）
> **涉及工程:** 父工程 + `oc-desktop-hos` + `oc-desktop-ark` + `oc-desktop-android`
> **上游基线:** opencode **v2.2.5**（tag `v2.2.5` / 提交 `9bbb8ce18c` / 分支 `Branch_v2.2.5`）
> **应用版本:** **2.2.5** / `versionCode 20205`
> **历史编号:** 原 `101-version-<VERSION>` 统一改为 `011-version-<VERSION>`；本模块为父工程公共版本规格。

## 1. 模块概述

### 1.1 目的

定义三个版别**同步升级上游 opencode 基线**时必须满足的共同要求：版本号一致、补丁矩阵可追溯、载荷可重建、门禁可复跑、证据可对比。

### 1.2 解决的问题

- 三版别各自维护补丁与构建链，容易出现版本号、基线、补丁数、运行时锁值不一致；
- 上游升级后，UI 契约、serve 语义、鉴权方式可能漂移；
- 没有统一验收口径时，“升级完成”容易变成“本机能编译”。

### 1.3 范围

**包含**：上游基线锁定、应用版本号、补丁目录、载荷重建、host 门禁、品牌守卫、真机回归要求、文档同步。

**不包含**：单个补丁的 hunk 级实现（见各子工程 `106-upstream-adaptation` / `204-plugin-git` / `011-version-*` 的 plan）；具体设备刷机/签名办理。

## 2. 用户故事

- 作为维护者，我能从版本规格一眼看到三版共同的基线、版本号与补丁矩阵。
- 作为构建者，我能按同一套门禁重建三版载荷并判断是否可发布。
- 作为评审者，我能对比升级前后的产物哈希、首启、接口与首屏字段。
- 作为用户，我在三版看到一致的版本号与品牌，不因版别不同而困惑。

## 3. 功能需求

### 3.1 基线一致性

- **FR-011-001**：三个版别 MUST 使用同一上游提交 `9bbb8ce18c`（tag `v2.2.5` / 分支 `Branch_v2.2.5`）。
- **FR-011-002**：每个版别 MUST 在 `patches/v2.2.5/` 下维护本版补丁，MUST NOT 修改历史 `patches/v2.0.26/`、`patches/v2.1.2/`。
- **FR-011-003**：补丁 MUST 幂等、可反向校验，并在应用前做基线/脏树守卫。
- **FR-011-004**：补丁集 MUST 只包含本版必需偏离项；每条补丁 MUST 有 README 说明“为什么上游不能直接接受”。

### 3.2 版本号

- **FR-011-005**：`versionName` MUST 为 `2.2.5`，`versionCode` MUST 为 `20205`（规则 `major*10000 + minor*100 + patch`）。
- **FR-011-006**：版本号 MUST 在单点定义（`AppScope/app.json5` 或 `app/build.gradle.kts`），其它位置引用或校验，不得出现第二份可写真值。
- **FR-011-007**：三版 README、规格索引、发布说明 MUST 与源码版本号一致；不一致 MUST 在版本规格中登记。

### 3.3 载荷与运行时

- **FR-011-008**：升级 MUST 重建上游 bundle 与载荷；MUST NOT 复用旧版本产物冒充新版本。
- **FR-011-009**：载荷 MUST 有 SHA-256 清单或等价完整性校验；校验失败 MUST 拒绝启动。
- **FR-011-010**：运行时锁值（Node 版本、ABI、`libnode.so` SHA-256/体积、16 KB 对齐）变更时 MUST 更新 `runtime.lock.json` 并通过 `verify-runtime`。
- **FR-011-011**：host 冒烟 MUST 覆盖 `/api/info`、内嵌 UI、SSE 首事件；品牌化版别 MUST 覆盖 title 断言。

### 3.4 证据与文档

- **FR-011-012**：升级完成 MUST 有可对比证据：产物哈希、包体、首启耗时、接口/首屏字段、真机轮次（无设备标 `⬜`）。
- **FR-011-013**：有设备时 MUST 验证：启动、会话、流式、模型凭据、工作区、Git；Android 还应验证 SAF/返回键/多模态等本版能力。
- **FR-011-014**：升级 MUST 更新 `TECH.md`、`overall-plan.md`、子工程 `011-version-2.2.5/` 与父工程索引中的版本引用。
- **FR-011-015**：未实测项 MUST 标 `⬜ 待验证`，不得计入“已完成”。

## 4. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 上游基线 | opencode 提交 | tag、commit、branch |
| 应用版本 | 用户可见版本 | versionName、versionCode |
| 补丁集 | 平台偏离项 | 目录、编号、作用、反向校验 |
| 载荷 | 设备侧运行的 bundle/UI | 文件、SHA-256、体积、顶层目录 |
| 运行时锁 | Node 引擎与 ABI | Node 版本、ABI、SHA-256、对齐 |
| 证据轮次 | 真机构建/验证记录 | 日期、设备、结果、日志/截图 |

## 5. 验收场景

### 场景 1：三版基线一致

- Given 三个子工程已 checkout 到 v2.2.5
- When 执行基线守卫
- Then 三版 `apply-patches --check` 均通过，HEAD = `9bbb8ce18c`，工作树干净

### 场景 2：版本号一致

- Given 三版源码
- When 读取 `AppScope/app.json5` / `app/build.gradle.kts`
- Then `versionName=2.2.5`、`versionCode=20205`，README/规格无相反表述

### 场景 3：载荷可重建

- Given 干净上游与固定 bun
- When 执行 build-opencode + collect-opencode
- Then 产出 `opencode.mjs`、UI 归档、SHA-256 清单；smoke 全绿

### 场景 4：品牌不漂移

- Given 品牌化版别（hos/android）
- When 执行 verify-branding / smoke
- Then UI title 为 `OC Desktop`，图标/字标断言通过

### 场景 5：真机回归（有设备）

- Given 签名/可安装产物
- When 安装并启动
- Then 启动阶段可见、首屏可用、会话/流式/凭据/工作区/Git 按版别通过；证据落盘

### 场景 6：无设备

- Given 本轮无鸿蒙设备
- When 完成主机编译
- Then 文档明确写“仅主机编译，真机 `⬜`”，不得写“已发布可用”

## 6. 非功能需求

- **可复现**：同一提交、同一工具链、同一 bun 版本产出可对比。
- **可追溯**：每个补丁/构建步骤能回引 README 与规格。
- **安全**：签名材料、API Key、口令不入库。
- **性能**：升级不得显著恶化首启/包体；变化必须记录。

## 7. 假设与约束

- 上游 v2.2.5 的 `serve`、`/api/*`、Web UI 归档契约与 v2.1.2 基本一致；漂移项由各子工程 `106` 处理。
- 共享 `opencode/` submodule 同一时刻只能被一个工程打补丁；构建必须串行。
- HarmonyOS 设备本轮可能缺位；无真机时只能给主机证据。

## 8. 依赖

- 上游 `opencode` submodule；
- 各子工程 `011-version-2.2.5/`、`106-upstream-adaptation`、`102-payload-pipeline`、`109-regression`；
- 父工程 `TECH.md`、`overall-plan.md`、`012-building-workflow`、`013-branding`。
