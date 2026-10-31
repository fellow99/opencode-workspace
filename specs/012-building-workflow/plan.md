# 012-building-workflow 技术方案（As-Built）

> 记录三版当前实际构建工作流、Android CI 细节、门禁与已知漂移。
> 对应规格：[spec.md](./spec.md)
> 最后更新：2026-11-01

## 1. 技术上下文

### 1.1 环境

| 版别 | 构建机 | 工具链 | 产物 |
|---|---|---|---|
| ark | Windows + Git-Bash + DevEco/CLT | Node 24、ohpm、hvigor 6.24.2、HarmonyOS SDK 6.1.1(24) | HAP |
| hos | Windows + PowerShell 7+ | Node 22/24、bun、DevEco、hvigor、pwsh | HAP |
| android | Windows/Linux + JDK 17 | Node 24、bun 1.4.2、Gradle 8.12、AGP 8.9.2、SDK 36 | APK |
| Android CI | GitHub Actions `ubuntu-latest` | Node 24、bun 1.4.2、JDK 17、Android SDK | APK + Release |

### 1.2 依赖

| 依赖 | 版本 | 用途 |
|---|---|---|
| opencode | v2.2.5 / `9bbb8ce18c` | 上游 |
| bun | 1.4.2（CI）/ 1.4.x（本地） | bundle 构建 |
| Node（host） | 24 | 构建脚本/冒烟 |
| Node（hos 冒烟） | 22.17.0 | 与 Electron 37 同代 |
| JDK | 17 | Android Gradle |
| Gradle/AGP | 8.12 / 8.9.2 | Android 构建 |
| HarmonyOS SDK/hvigor | 6.1.1(24) / 6.24.2 | 鸿蒙构建 |

## 2. 宪法合规

| 条款 | 判定 | 说明 |
|---|---|---|
| N2 `012-building-workflow` | ✅ | Android 模块已从 `001-building-workflow` 迁移 |
| FR-BUILD-001/002 | ✅ | 固定上游、门禁、哈希 |
| S1 上游只读 | ✅ | CI 独立检出上游；本地脚本守卫 |
| S5 凭据 | ✅ | 签名 secrets 可选；keystore 不入库 |
| G3 质量门 | 🔶 | Android CI 仍锁 v2.1.2，待更新 `⚠️` |

## 3. 研究结论

1. **Android CI 已落地**：tag 触发 → 上游检出 → 载荷五步 → APK → Release；`workflow_dispatch` 只构建不发布。
2. **CI 基线漂移**：工作流 current `ref: a517687fde`（v2.1.2），而源码/补丁为 v2.2.5 `9bbb8ce18c`。这是发布前必须修的高优先级项。
3. **鸿蒙无 CI**：ark/hos 当前为本地脚本；本模块只定义共同门禁，不虚构云端工作流。
4. **签名回退明确**：无 secrets 时 debug 签名；有 secrets 时 release，且缺项会显式报错。

## 4. 数据模型

### 4.1 工作流实体

```
BuildWorkflow {
  trigger: ["push v* tag", "workflow_dispatch"]
  runner: "ubuntu-latest"
  timeout: 60m
  steps: [checkout, upstream, node, bun, jdk, engine-prefetch, payload-1..5, sign, build, collect, upload, release]
  permissions: { contents: write }
}
```

### 4.2 产物

```
dist/
├── oc-desktop-android-<tag>.apk
└── oc-desktop-android-<tag>.apk.sha256
```

### 4.3 签名 secrets

| Secret | 必填条件 | 说明 |
|---|---|---|
| `KEYSTORE_BASE64` | 可选 | keystore base64；存在则走 release |
| `KEYSTORE_PASSWORD` | 有 keystore 时必填 | storePassword |
| `KEY_ALIAS` | 有 keystore 时必填 | keyAlias |
| `KEY_PASSWORD` | 有 keystore 时必填 | keyPassword |

## 5. 接口契约

### 5.1 提供

- 统一构建门禁顺序；
- Android CI 触发/产物/secrets 契约；
- 鸿蒙本地构建命令顺序。

### 5.2 消费

- `011-version-2.2.5` 的基线/版本；
- 子工程脚本：`apply-patches.mjs`、`build-opencode.mjs`、`collect-plugins.mjs`、`collect-opencode.mjs`、`smoke-node*.mjs`；
- `013-branding` 的品牌守卫。

## 6. 实现策略

### 6.1 Android CI（`.github/workflows/build-release.yml`）

实际步骤：

1. `actions/checkout@v4` 检出本仓库；
2. `actions/checkout@v4` 检出上游 `anomalyco/opencode`，`ref: a517687fde`（**待改 `9bbb8ce18c`**），`fetch-depth: 0`；
3. `actions/setup-node@v4`（24）；
4. `oven-sh/setup-bun@v2`（1.4.2）；
5. `actions/setup-java@v4`（temurin 17，Gradle cache）；
6. 预取 `node-runtime-24.22.1.aar` 到 `OC_NODE_RUNTIME_AAR`；
7. 载荷① `apply-patches --repo "$UPSTREAM"`；
8. 载荷② `build-opencode`（`OPENCODE_NODE_TARGET=node24`）；
9. 载荷③ `collect-plugins`；
10. 载荷④ `collect-opencode`；
11. 载荷⑤ `smoke-node24`；
12. 签名注秘（可选）→ `VARIANT=release|debug`；
13. `./gradlew :app:assemble${VARIANT}`；
14. 收集 `app/build/outputs/apk/${VARIANT,,}/app-${VARIANT,,}.apk` → `dist/` + sha256；
15. `upload-artifact`；
16. 仅 tag 触发 `softprops/action-gh-release@v2`。

**待修**：

- 上游 `ref` 改为 `9bbb8ce18c`（v2.2.5）；同步 Release body 的上游/引擎描述；
- 若 `node-runtime` 版本变化，更新预取 URL 与 `OC_NODE_RUNTIME_AAR`；
- 工作流注释中的 `001-building-workflow` 已迁移为 `012-building-workflow`，需复核。

### 6.2 鸿蒙本地构建

**ark**

```bash
ohpm install
node scripts/verify-runtime.mjs
node scripts/apply-patches.mjs
node scripts/build-opencode.mjs
node scripts/collect-opencode.mjs
node scripts/smoke-node24.mjs
hvigorw assembleHap --mode module -p product=default -p buildMode=debug --no-daemon
```

**hos**

```powershell
node scripts\collect-runtime.mjs
node scripts\build-opencode.mjs
node scripts\collect-app.mjs
node scripts\smoke-node22.mjs
pwsh -NoProfile -ExecutionPolicy Bypass -File scripts\build-debug.ps1
```

### 6.3 错误处理

- 任一步非零退出即失败；不捕获后继续。
- 签名 secrets 部分配置时报错并列出缺失项。
- 产物文件不存在直接失败，不做“跳过发布”。
- 失败日志保留在 CI artifact / 本地 stderr。

### 6.4 性能

- Android CI 超时 60 min；Gradle 缓存；引擎 AAR 预取避免冷缓存失败。
- 鸿蒙本地构建耗时主要在上游 bundle 与 hvigor；不做静默缓存跳过。

## 7. 测试与验证

| 项 | 方法 | 通过标准 |
|---|---|---|
| 工作流结构 | 静态断言/YAML 解析 | 触发、步骤、secrets、产物齐全 |
| 冷缓存 | 手动 `workflow_dispatch` | 载荷五步 + APK 全绿 |
| 产物 | `test -f` + sha256 | 文件名/大小写/哈希正确 |
| 签名回退 | 无 secrets 跑一次 | debug APK，有 notice |
| 鸿蒙本地 | 按 README 跑 | HAP 产出；unsigned 有说明 |

## 8. 文件清单

| 文件 | 作用 |
|---|---|
| `oc-desktop-android/.github/workflows/build-release.yml` | Android CI |
| `oc-desktop-android/specs/012-building-workflow/` | Android 模块规格四件套 |
| `oc-desktop-ark/scripts/*.mjs` | ark 构建/门禁 |
| `oc-desktop-hos/scripts/*.mjs` / `*.ps1` | hos 构建/门禁 |
| `oc-desktop-{ark,hos,android}/patches/v2.2.5/` | 补丁 |
| `specs/012-building-workflow/spec.md` | 本模块公共规格 |

## 9. 已知漂移与后续

| # | 项 | 状态 | 处置 |
|---|---|---|---|
| B-01 | Android CI 上游 ref 仍 v2.1.2 | ⚠️ | 更新到 `9bbb8ce18c` 并彩排 |
| B-02 | 鸿蒙无云端 CI | ⬜ | 保持本地构建；需要时补 `012` 子规格 |
| B-03 | Release 说明中的版本/引擎 | ⚠️ | 随 CI 更新同步 |
| B-04 | 真机证据 | 🔶 | Android 较完整；ark/hos 待补 |
