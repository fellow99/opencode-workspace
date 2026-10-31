# 011-version-2.2.5 技术方案（As-Built）

> 本文是父工程公共版本模块的技术方案，记录 v2.2.5 升级的**实际落地方式**与待收敛项。
> 对应规格：[spec.md](./spec.md)
> 最后更新：2026-11-01

## 1. 技术上下文

### 1.1 运行环境

| 版别 | 运行时 | 渲染 | 构建 |
|---|---|---|---|
| ark | Node v24.2.0（ABI 137） | ArkWeb | hvigor 6.24.2 / HarmonyOS SDK 6.1.1(24) |
| hos | Node v22.17.0（Electron 37 / ABI 138） | Chromium 138 | hvigor / PowerShell |
| android | Node v24.22.1（arm64-v8a） | 系统 WebView | Gradle 8.12 / AGP 8.9.2 / JDK 17 |
| host 脚本 | Node 24 | — | bun 1.4.x（CI 1.4.2） |

### 1.2 依赖

| 依赖 | 版本/来源 | 用途 |
|---|---|---|
| opencode | v2.2.5 / `9bbb8ce18c` | 服务端 + UI |
| bun | `^1.4.2` | 上游 bundle 构建 |
| `@fellow99/node-bridge` / `node-runtime` | 0.1.0 / 24.2.0 | ark 运行时 |
| `io.github.fellow99:node-bridge` / `node-runtime` | 0.2.0 / 24.22.1 | android 运行时 |
| Electron | 37（`v37.2.3-20260825.1`） | hos 运行时 |
| better-sqlite3 | aarch64-ohos / ABI 138 | hos 数据库 |
| `node:sqlite` | Node 内置 | ark/android 数据库 |

## 2. 宪法合规

| 条款 | 判定 | 说明 |
|---|---|---|
| N2 `011-version-<VERSION>` | ✅ | 历史 `101-version-*` 已迁移为 `011-version-*` |
| S1 上游只读 | ✅ | 补丁在 `patches/v2.2.5/`，不提交上游 |
| E1/E2 证据 | ✅ | 有真机/主机证据分档；未实测标 `⬜` |
| G1 提交规范 | ✅ | 版本/规格迁移使用 `docs:` 前缀 |
| G3 质量门 | 🔶 | host 门禁齐全；鸿蒙 v2.2.5 真机待补；android CI 基线待收敛 |

## 3. 研究结论

1. **基线一致**：三版 `patches/v2.2.5/` 均存在；ark 5 条、hos 8 条、android 8 条。补丁编号沿用上游差异，不按连续序号强凑。
2. **版本号一致**：三版源码均为 `2.2.5` / `20205`；根 README、子工程 README 已对齐。
3. **运行时差异**：ark/android 使用 `node:sqlite`，hos 使用 better-sqlite3；这是平台构建差异，不是版本漂移。
4. **真机证据不均**：Android v2.2.5 有较完整真机记录；ark/hos 本轮仅主机 unsigned HAP 编译，缺真机。
5. **CI 漂移**：`oc-desktop-android/.github/workflows/build-release.yml` 仍检出 `a517687fde`（v2.1.2），与源码 v2.2.5 不一致 `⚠️`。
6. **文档漂移**：`oc-desktop-android/specs/TECH.md`、`docs/000-工程规划.md` 部分仍以 v2.1.2 叙述 `⚠️`。

## 4. 数据模型

### 4.1 版本实体

```
VersionRelease {
  versionName: "2.2.5"
  versionCode: 20205
  upstream: { tag: "v2.2.5", commit: "9bbb8ce18c", branch: "Branch_v2.2.5" }
  patches: { ark: 5, hos: 8, android: 8 }
  runtime: { ark: "24.2.0/137", hos: "22.17.0/138", android: "24.22.1/arm64" }
  evidence: { android: "真机", ark: "主机编译", hos: "主机编译" }
}
```

### 4.2 校验规则

- `versionCode == major*10000 + minor*100 + patch`；
- 补丁目录 tag = 上游 tag；
- 运行时锁 SHA-256/体积与 `runtime.lock.json` 一致；
- 品牌 title 以 `OC Desktop` 开头（品牌化版别）。

## 5. 接口契约

### 5.1 提供

- 父工程 `011-version-2.2.5/spec.md`：跨版别版本要求；
- 版本矩阵与补丁矩阵：供 `012-building-workflow`、`013-branding`、各子工程版本规格引用。

### 5.2 消费

- `opencode` submodule 的 tag/commit；
- 各子工程 `patches/v2.2.5/`、`runtime.lock.json`、`AppScope/app.json5` / `app/build.gradle.kts`；
- `docs/能力对照清单.md` 的真机证据结论。

## 6. 实现策略

### 6.1 升级顺序

1. 固定上游：`git -C opencode checkout 9bbb8ce18c`（或 `git submodule update`）。
2. 为三版新建/迁移 `patches/v2.2.5/`；保留旧 tag 目录。
3. 更新应用版本号与 README。
4. 更新运行时锁值（如变化）。
5. 重建载荷：`apply-patches → build-opencode → collect-* → smoke`。
6. 构建 HAP/APK；有设备则真机回归。
7. 更新版本规格、父/子索引与证据。

### 6.2 各版关键命令

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
pwsh -File scripts\build-debug.ps1
```

**android**

```bash
node scripts/apply-patches.mjs
OPENCODE_NODE_TARGET=node24 node scripts/build-opencode.mjs
node scripts/collect-plugins.mjs
node scripts/collect-opencode.mjs
node scripts/smoke-node24.mjs
./gradlew :app:assembleDebug
```

### 6.3 错误处理

- 基线守卫失败：停止构建，不自动 reset；
- 补丁反向校验失败：视为补丁集不一致，重建补丁；
- 载荷哈希失败：拒绝启动，错误页提示重装/重建；
- 品牌断言失败：阻断发布，修复补丁/脚本。

### 6.4 性能

- 记录包体、首启耗时、SSE 首事件；
- 与上一版本对比；显著恶化必须写原因。

## 7. 测试与验证

| 层级 | 命令/手段 | 通过标准 |
|---|---|---|
| 补丁 | `apply-patches --check` | 基线正确、工作树干净、补丁可反向 |
| 运行时 | `verify-runtime.mjs` | SHA/体积/ABI/对齐一致 |
| 载荷 | `build-opencode` + `collect-opencode` | 产物齐全、清单一致 |
| host 冒烟 | `smoke-node22/24` | `/api/info` 200、SPA、SSE 首事件 |
| 品牌 | `verify-branding` + smoke title | 品牌断言通过 |
| 构建 | hvigor / gradle | HAP/APK 产出 |
| 真机 | 安装/启动/会话/流式/工作区/Git | 按版别用例；无设备标 `⬜` |

## 8. 文件清单

| 文件 | 作用 |
|---|---|
| `specs/011-version-2.2.5/spec.md` | 公共版本规格 |
| `specs/011-version-2.2.5/plan.md` | 本文件 |
| `opencode`（submodule） | 上游基线 |
| `oc-desktop-{ark,hos,android}/patches/v2.2.5/` | 各版补丁 |
| `oc-desktop-{ark,hos,android}/runtime.lock.json` | 运行时锁 |
| `oc-desktop-{ark,hos,android}/AppScope/app.json5` 或 `app/build.gradle.kts` | 版本号 |
| `.github/workflows/build-release.yml` | Android CI（待更新基线） |

## 9. 待收敛项

| # | 项 | 状态 | 建议 |
|---|---|---|---|
| V-01 | android CI 仍锁 v2.1.2 | ⚠️ | 更新 `ref` 到 `9bbb8ce18c`，跑 `workflow_dispatch` 彩排 |
| V-02 | android `TECH.md` / 规划仍写 v2.1.2 | ⚠️ | 随本版本规格回写 |
| V-03 | ark/hos v2.2.5 真机 | ⬜ | 有设备时补跑测试用例 |
| V-04 | ark 品牌化缺失 | ❌ | 见 `013-branding` |
| V-05 | 端口随机导致渲染层状态丢失 | ⬜ | ark/android 补跨重启用例 |
