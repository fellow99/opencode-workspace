# 013 — 品牌化（branding）功能规格

> **Module:** 013-branding ｜ **Status:** Implemented（hos/android）/ Gap（ark） ｜ **Last Updated:** 2026-11-01
> **需求名称:** 品牌化：OpenCode → OC Desktop / Logo / 图标 / 非官方声明（跨版别公共模块）
> **涉及工程:** 父工程 + `oc-desktop-hos` + `oc-desktop-ark` + `oc-desktop-android`
> **物料来源:** 父工程 `assets/`（logo、SVG、ICO、ICNS、截图）

## 1. 模块概述

### 1.1 目的

把上游 opencode 的品牌收敛为 **OC Desktop**，并保证三版在应用名、Web UI 标题、图标、加载页、关于页、非官方声明与第三方许可上的一致性与可验证性。

### 1.2 解决的问题

- 上游 Web UI 默认标题/字标是 `OpenCode`，直接分发会让用户误以为是官方应用；
- 三版各自改品牌，容易出现标题、图标、包名、文案不一致；
- 上游升级可能把品牌改动覆盖，需要守卫防止漂移；
- 非官方移植身份与许可声明必须随包可见。

### 1.3 范围

**包含**：应用显示名、Web UI title/字标/PWA manifest、应用图标、加载页/关于页、非官方声明、AI 标识、第三方许可、品牌守卫。

**不包含**：上游服务名/法律声明（`OpenCode Console`、`OpenCode Go` 等）的改写；商标注册与法务审查。

## 2. 用户故事

- 作为用户，我在桌面/应用列表看到 **OC Desktop**，在 Web UI 标题也看到 OC Desktop。
- 作为用户，我能从关于页知道这是第三方移植，不是 opencode 官方应用。
- 作为合规评审者，我能用断言脚本确认品牌已生效、未回退。
- 作为维护者，我升级上游后能立刻发现品牌漂移并修复。

## 3. 功能需求

### 3.1 应用身份

- **FR-013-001**：应用显示名 MUST 为 **OC Desktop**；版别后缀（Ark/Android）只用于仓库/文档，不进入用户可见应用名。
- **FR-013-002**：包名 MUST 保持各版独立（`org.fellow99.oc.OCDesktopArk` / `...OCDesktop` / `...OCDesktopAndroid`），不得复用上游包名。
- **FR-013-003**：关于页/设置页 MUST 显示版本号、上游基线、非官方声明与许可入口。

### 3.2 Web UI

- **FR-013-004**：Web UI `document.title` MUST 以 `OC Desktop` 开头。
- **FR-013-005**：PWA manifest / 图标 / 字标 MUST 使用 OC Desktop 物料；MUST NOT 残留上游品牌作为主标识。
- **FR-013-006**：品牌化 MUST 通过幂等补丁或构建期变换实现；MUST NOT 长期 fork 上游 UI 源码。
- **FR-013-007**：上游服务名与法律声明 MUST 保留，不得误改。

### 3.3 图标与物料

- **FR-013-008**：应用图标 MUST 来自父工程 `assets/` 或由其派生的确定性物料。
- **FR-013-009**：发布图标 MUST 不透明、尺寸合规；透明图标仅用于特定场景并需验证市场接受度。
- **FR-013-010**：图标生成 MUST 幂等；重复执行不得产生不同产物。

### 3.4 验证与守卫

- **FR-013-011**：品牌化版别 MUST 有可执行守卫（title 断言、图标断言、字标断言或等价检查）。
- **FR-013-012**：守卫 MUST 在构建/冒烟阶段运行；失败 MUST 阻断发布。
- **FR-013-013**：上游升级后 MUST 重新跑品牌守卫；漂移 MUST 记录并修复。
- **FR-013-014**：未实现品牌化的版别 MUST 在 `specs/013-branding/` 如实记录缺口、证据与补齐计划，MUST NOT 写成已完成。

## 4. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 品牌物料 | logo/图标 | 路径、尺寸、透明度、哈希 |
| 应用身份 | 显示名/包名 | label、bundleName/applicationId |
| UI 品牌 | title/manifest/字标 | 文本、断言 |
| 关于页 | 非官方声明/许可 | 文案、链接 |
| 品牌守卫 | 断言脚本 | 检查项、退出码 |

## 5. 验收场景

### 场景 1：应用名

- Given 安装包
- When 查看桌面/应用列表
- Then 显示名为 `OC Desktop`

### 场景 2：Web UI 标题

- Given 应用已启动
- When 读取 `document.title` 或归档 `index.html`
- Then 以 `OC Desktop` 开头

### 场景 3：图标

- Given 构建产物
- When 检查图标资源
- Then 尺寸/透明度/内容符合物料规范

### 场景 4：非官方声明

- Given 关于页
- When 打开
- Then 可见“非 opencode 官方”“MIT/第三方许可”说明

### 场景 5：品牌漂移

- Given 上游升级后
- When 跑品牌守卫
- Then 若 title/字标回退，守卫失败并阻断发布

### 场景 6：未实现版别

- Given ark 当前归档
- When 检查 `<title>`
- Then 仍为 `OpenCode`；`013-branding` 文档标 `❌` 缺口，不得标 ✅

## 6. 非功能需求

- **一致性**：三版显示名、图标、关于页文案一致。
- **可追溯**：物料来源、生成脚本、断言结果可查。
- **安全**：品牌脚本不得引入网络下载或未校验二进制。
- **性能**：品牌化不显著增加包体；图标多尺寸按需生成。

## 7. 假设与约束

- 父工程 `assets/` 是品牌物料的单一来源。
- 上游 UI 的 title/manifest 位置随版本可能变化；补丁需按 tag 重建。
- 应用商店对图标透明度/尺寸的要求可能变化；发布前复核。

## 8. 依赖

- 父工程 `assets/`；
- 各子工程 `013-branding/`；
- `012-building-workflow` 的品牌守卫步骤；
- `011-version-2.2.5` 的上游基线。
