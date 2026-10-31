# 012 — 构建工作流（building-workflow）功能规格

> **Module:** 012-building-workflow ｜ **Status:** Implemented（Android）/ Not Implemented（HarmonyOS） ｜ **Last Updated:** 2026-11-01
> **需求名称:** 构建工作流（跨版别公共模块）
> **涉及工程:** 父工程 + `oc-desktop-android`；`oc-desktop-hos` / `oc-desktop-ark` 暂为本地流水线
> **说明:** 本模块是 `0XX` 公共模块；**鸿蒙工程暂无**等价的云端 tag 发布工作流。

## 1. 模块概述

### 1.1 目的

把“从上游源码到可安装产物”的构建过程定义为一条**可复现、可门禁、可发布**的工作流，并明确三版当前的落地差异。

### 1.2 解决的问题

- 构建步骤散落在 README/脚本中，容易漏步、版本漂移；
- 上游补丁、载荷、host 冒烟、签名、发布之间缺少强制门禁；
- 鸿蒙与 Android 的构建形态不同，需要统一验收口径而不是强求同一实现。

### 1.3 范围

**包含**：上游检出与基线守卫、补丁、bundle 构建、载荷收集、插件物化、host 冒烟、平台构建、签名、产物哈希、GitHub Release（Android）。

**不包含**：签名证书办理、应用商店后台提交、真机安装步骤（见各子工程 README/安装文档）。

## 2. 用户故事

- 作为维护者，我能用一条 tag 触发 Android 云端构建并得到可安装 APK + 校验值。
- 作为构建者，我能在本地按同样顺序构建鸿蒙 HAP，并在任一门禁失败时立即停止。
- 作为评审者，我能确认产物来自锁定的上游与工具链，而不是“某台机器上的偶然结果”。
- 作为发布者，我能在没有签名 secrets 时得到 debug 包，并清楚知道它不是正式签名包。

## 3. 功能需求

### 3.1 上游与补丁

- **FR-012-001**：构建 MUST 检出锁定的上游提交；MUST NOT 使用浮动分支。
- **FR-012-002**：构建前 MUST 执行补丁基线守卫与脏树守卫；失败 MUST 停止。
- **FR-012-003**：补丁 MUST 幂等；重复执行不得破坏工作树。

### 3.2 载荷与门禁

- **FR-012-004**：构建 MUST 产出完整载荷：服务端 bundle、UI 归档、必要原生模块、插件。
- **FR-012-005**：载荷 MUST 有 SHA-256 清单或等价校验；失败 MUST 阻断打包。
- **FR-012-006**：host 冒烟 MUST 覆盖 `/api/info`、内嵌 UI、SSE 首事件；品牌化版别 MUST 覆盖 title。
- **FR-012-007**：插件 MUST 在打包前通过语义/围栏/上限测试；失败 MUST 阻断。
- **FR-012-008**：任一红门 MUST NOT 继续后续步骤；MUST NOT 静默降级。

### 3.3 平台构建与签名

- **FR-012-009**：Android MUST 支持 `assembleDebug` / `assembleRelease`；有签名材料时 MUST 产出签名 APK。
- **FR-012-010**：鸿蒙 MUST 支持 `assembleHap`；有签名材料时 MUST 产出签名 HAP，无材料时 MUST 明确 unsigned。
- **FR-012-011**：签名材料 MUST NOT 入库；CI secrets 缺失时 MUST 明确回退 debug 并提示。
- **FR-012-012**：产物 MUST 有稳定命名与 SHA-256 校验值。

### 3.4 发布

- **FR-012-013**：Android MUST 支持 push `v*` tag 触发 GitHub Release；`workflow_dispatch` MUST 只构建不发布。
- **FR-012-014**：Release MUST 附带 APK + `.sha256`；MUST 说明上游基线与引擎版本。
- **FR-012-015**：鸿蒙当前 MUST 以本地构建 + 侧载为准；若新增 CI，MUST 复用本模块门禁并补规格。
- **FR-012-016**：发布说明 MUST NOT 声称未验证的能力已可用；无真机证据 MUST 标 `⬜`。

## 4. 关键实体

| 实体 | 说明 | 关键属性 |
|---|---|---|
| 工作流 | CI 定义 | 触发、runner、步骤、权限 |
| 门禁 | 阻断条件 | 补丁、载荷、冒烟、插件、品牌 |
| 产物 | 可安装包 | 文件名、SHA-256、签名状态 |
| Secrets | 签名材料 | keystore、口令、alias |
| Release | 发布记录 | tag、附件、说明 |

## 5. 验收场景

### 场景 1：Android tag 构建

- Given 仓库配置了可用 secrets（或未配置）
- When push `v2.2.5` tag
- Then CI 完成载荷五步 + APK + Release；附件含 APK 与 `.sha256`

### 场景 2：无签名 secrets

- Given 未配置 `KEYSTORE_BASE64`
- When 触发构建
- Then 产出 debug 签名 APK，并有明确 notice；不得假装是 release 签名

### 场景 3：门禁失败

- Given 上游基线不匹配
- When 执行构建
- Then 补丁守卫失败，后续步骤不执行，CI 红

### 场景 4：鸿蒙本地构建

- Given DevEco/ohpm/hvigor 就绪
- When 按 README 顺序执行
- Then 产出 HAP；无签名材料时为 unsigned，并有说明

### 场景 5：产物可校验

- Given 构建产物
- When 计算 SHA-256
- Then 与发布附件/清单一致

## 6. 非功能需求

- **可复现**：固定上游、bun、JDK、SDK；禁止 floating。
- **可观测**：每步有日志；失败能定位到步骤。
- **安全**：secrets 不落盘到仓库；日志不打印口令。
- **时效**：单次 Android CI SHOULD ≤ 60 min；超时即失败。

## 7. 假设与约束

- Android CI runner 预装 Android SDK 且 license 已接受。
- 鸿蒙构建依赖 DevEco Studio/命令行工具与 Windows PowerShell 7+。
- 共享 submodule 构建需串行；CI 使用独立检出目录。

## 8. 依赖

- `011-version-2.2.5`（基线/版本）；
- 子工程 `102-payload-pipeline` / `112-build-pipeline` / `106-upstream-adaptation`；
- `013-branding`（品牌守卫）；
- Android `.github/workflows/build-release.yml`。
