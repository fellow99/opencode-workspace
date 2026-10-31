# OC Desktop 整体技术方案（overall-plan）

> `overall-spec.md` 的技术对应物，回答“怎么做、在哪做、怎么验”。
> 版本与技术栈见 [`TECH.md`](./TECH.md)；结构与数据流见 [`ARCHITECTURE.md`](./ARCHITECTURE.md)；约束见 [`constitution.md`](./constitution.md)。
> 事实源：根 `README.md`、`docs/能力对照清单.md`、三个子工程的 README / 工程规划 / 规格 / 源码。未落地项标 `⬜`。

---

## 1. 技术上下文

| 项 | 值 | 来源 |
|---|---|---|
| 上游 | opencode v2.2.5（`9bbb8ce18c` / `Branch_v2.2.5`） | `opencode` submodule、各 `patches/v2.2.5/README.md` |
| 应用版本 | 2.2.5 / `versionCode 20205` | `AppScope/app.json5`、`app/build.gradle.kts` |
| 三版运行时 | Node 24.2.0（ark）/ Node 22.17.0（hos）/ Node 24.22.1（android） | `runtime.lock.json`、Electron 版本、`gradle/libs.versions.toml` |
| 三版渲染 | ArkWeb / Chromium / 系统 WebView | 各工程源码 |
| 三版数据 | `node:sqlite` / `better-sqlite3` / `node:sqlite` | 各工程源码与规格 |
| 构建 | hvigor（ark/hos）、Gradle（android）、Node ESM 脚本 | 各工程脚本 |
| CI | GitHub Actions（android）；鸿蒙暂无 | `.github/workflows/build-release.yml` |
| 证据 | `docs/能力对照清单.md` + 各工程真机记录 | 父/子文档 |

**关键前提**：

- `node::Start` 每进程一次；崩溃是终态；
- 应用域子进程执行 Node/JIT 受限；系统 `sh` 可用；
- 共享 `opencode/` submodule，补丁构建必须串行；
- 服务端端口随机，必须用 marker/健康检查判就绪；
- 同源是零 CORS/零业务注入桥的前提。

---

## 2. 宪法合规

| 原则 | 判定 | 说明 |
|---|---|---|
| D1/D2 文档分工 | ✅ | 父工程写共同需求；子工程写实现与独立需求 |
| N1–N5 编号 | ✅ | `0XX` 公共、`1XX` 基础、`2XX` 插件；已迁移版本/品牌/构建/插件编号 |
| E1–E4 证据 | ✅ | 规格引用源码/实测；未实测标 `⬜` |
| S1 上游只读 | ✅ | 只用幂等补丁；基线守卫 |
| S2 本地服务 | ✅ | loopback + 随机端口 + 密码；`0.0.0.0` 仅 hos 现状 |
| S3 权限最小化 | 🔶 | ark/hos ACL 与 android 权限面已在子工程登记；仍有声明面/使用面差异需复核 |
| S4 不隐藏真相 | ✅ | 三版都有阶段错误页；部分能力未实现但如实返回 not-implemented |
| S5 凭据 | ✅ | 凭据沙箱化；文档/日志脱敏规则已写入子工程宪法 |

---

## 3. 实施策略总览

### 3.1 共享基线

1. **上游固定**：`opencode` submodule 固定到 v2.2.5 `9bbb8ce18c`；每个子工程 `patches/v2.2.5/` 只包含本工程必需补丁。
2. **统一 bundle 语义**：三版都构建 stock CLI bundle，使用 `serve --hostname <host> --port 0`，不新增服务端入口。
3. **统一环境契约**：`HOME` / `XDG_*` / `TMPDIR` / `OPENCODE_*` / `NO_PROXY` 由宿主或 Node 启动脚本显式设置；环境变量名单单一来源。
4. **统一鉴权**：宿主生成高熵密码，沙箱持久化；首屏 `?auth_token=`（hos 用配对 Cookie）；日志脱敏。
5. **统一载荷治理**：上游产物 → 裁剪 → 插件物化 → tar.gz/resfile/assets → SHA-256 清单 → host 冒烟。
6. **统一插件治理**：L1 本地插件；构建期 copy 物化；digest 校验；不实现在线安装。
7. **统一编号/文档**：按父宪法 `0XX/1XX/2XX`，子工程索引与 FR 前缀同步。

### 3.2 版别实现

| 版别 | 运行时接线 | 渲染 | 数据 | 关键差异 |
|---|---|---|---|---|
| ark | `@fellow99/node-bridge` + `@fellow99/node-runtime`，`NodeRuntime.start` | ArkWeb `Web` | `node:sqlite` | 无 Electron/Chromium；resfile 直读；jitless compat |
| hos | Electron 主进程内 `import` bundle | Chromium（ArkWeb R2 未落地） | better-sqlite3 | Electron 37；tar.gz 首启解压；四目录选择器；品牌化完整 |
| android | `io.github.fellow99:node-bridge` + `node-runtime`，`rt.start` | 系统 WebView | `node:sqlite` | assets tar 自解包；SAF 导入；pnpm；GitHub Actions |

---

## 4. 公共模块落地

### 4.1 `011-version-<VERSION>` 版本更新

**目标**：三版同步升级上游基线、应用版本、补丁矩阵、运行时锁值与文档。

**步骤**：

1. 更新 `opencode` submodule 到目标 tag/提交；校验 `git describe` 与基线守卫常量。
2. 新建 `patches/<新 tag>/`，逐条迁移/重生成补丁；旧目录保留为历史锚点。
3. 更新 `AppScope/app.json5` 或 `app/build.gradle.kts` 的 `versionName` / `versionCode`。
4. 重建载荷；更新 `runtime.lock.json`（如运行时变化）。
5. 跑 host 门禁（`smoke-node22` / `smoke-node24`）、品牌守卫、插件测试。
6. 构建 HAP/APK；有设备则做真机回归；无设备则如实记录 `⬜`。
7. 更新 `011-version-<VERSION>/` 的 spec/plan/tasks/test-cases 与父工程索引。

**当前 v2.2.5 事实**：

- 上游 `9bbb8ce18c`；应用 2.2.5/20205；
- 补丁：ark 5、hos 8、android 8；
- Android 真机证据较完整；ark/hos 本轮仅主机编译，真机待补 `⬜`；
- android CI 工作流仍锁 v2.1.2（待收敛 `⚠️`）。

### 4.2 `012-building-workflow` 构建工作流

**共同门禁**：

```
apply-patches（基线/脏树/幂等）
  → build-opencode（bun + bundle-only）
  → collect-plugins（物化/断言）
  → collect-opencode（裁剪/tar.gz/清单）
  → smoke（/api/info + SPA + SSE）
  → 平台构建（HAP/APK）
  → 签名（可选）
  → 产物哈希 + 发布
```

**Android CI（已存在）**：

- 触发：push `v*` tag；`workflow_dispatch` 仅构建验证。
- runner：`ubuntu-latest`；Node 24；bun 1.4.2；JDK 17；Android SDK。
- 上游检出：`anomalyco/opencode`，当前工作流写死 `a517687fde`（v2.1.2，需更新到 `9bbb8ce18c`）。
- 签名 secrets 可选：`KEYSTORE_BASE64` / `KEYSTORE_PASSWORD` / `KEY_ALIAS` / `KEY_PASSWORD`；缺省 debug 签名。
- 产物：`oc-desktop-android-<tag>.apk` + `.sha256`；GitHub Release。

**鸿蒙**：

- 暂无云端 tag 发布工作流；本地流水线为：
  - ark：`ohpm install` → `verify-runtime` → `apply-patches` → `build-opencode` → `collect-opencode` → `smoke-node24` → `hvigorw assembleHap`。
  - hos：`collect-runtime` → `build-opencode` → `collect-app` → `smoke-node22` → `build-debug.ps1`。
- 后续若加 CI，必须复用同一门禁与产物哈希口径。

### 4.3 `013-branding` 品牌化

**统一品牌**：显示名 `OC Desktop`；版别后缀只用于仓库/文档。

**落点**：

| 版别 | 应用名 | UI 标题 | 图标 | 守卫 | 状态 |
|---|---|---|---|---|---|
| hos | OC Desktop | 补丁/脚本改 `OC Desktop` | `assets/brand/*` + 36/37 项断言 | `verify-branding.mjs` | ✅ |
| android | OC Desktop | 0010 补丁改 `<title>` | 自适应图标 + 品牌物料 | `verify-branding.mjs` + smoke title | ✅ |
| ark | OC Desktop Ark | 仍为 `OpenCode`（归档） | 壳层图标/加载页 | 无 | ❌ 缺口 |

**ark 补齐建议**（写入 `oc-desktop-ark/specs/013-branding/`）：

1. 新增 v2.2.5 品牌化补丁（改 `index.html` title、PWA manifest、可能的字标）；
2. 增加 `scripts/lib/branding.mjs` 或等价物料脚本 + `verify-branding.mjs`；
3. 在 `smoke-node24.mjs` 增加 title 断言；
4. 构建后对 `app-archive.bin` 解包实测，确认 `OC Desktop` 出现、`OpenCode` 指称收敛；
5. 真机截图取证。

---

## 5. 横切关注点

### 5.1 日志与观测

- 宿主日志、服务端日志、桥日志分文件；统一前缀（`[ark]` / `[hos]` / `[android]`）。
- 就绪 marker、健康检查、崩溃记录是可证伪判据。
- 凭据、`auth_token`、API Key 永不入日志。

### 5.2 错误处理

- 错误对象至少包含 `{stage, message, hint, detail}`。
- 错误页展示阶段、原因、日志尾、重启提示。
- 平台不可得能力返回结构化 not-implemented，不抛未捕获异常。

### 5.3 安全

- 见父宪法 §四；子工程宪法补充 ACL/权限细节。
- 服务端口随机；密码沙箱 0600；首屏 token 一次性。
- 插件/补丁不得扩大权限声明面。

### 5.4 数据与状态

- 会话数据在 SQLite（WAL）；重启后必须可读。
- 渲染层 localStorage 受随机端口 origin 影响；hos 已确认状态丢失 `⚠️`，ark/android 未专项验证 `⬜`。
- 工作区切换要同步服务端 cwd 与工具围栏。

### 5.5 国际化与文案

- 上游 UI 自带多语言；壳层至少中文。
- 错误页/加载页文案中文；版别名称在关于页/非官方声明中说明。

---

## 6. 测试策略

| 层级 | 手段 | 触发 |
|---|---|---|
| 静态 | `node --check`、JSON 解析、权限声明面比对、链接检查 | 每次提交 |
| 补丁 | `apply-patches --check`、反向校验 | 每次构建前 |
| 运行时锁 | `verify-runtime.mjs`（SHA/体积/ABI/16 KB） | 运行时变更 |
| 载荷 | `build-opencode` + `collect-opencode` + `smoke` | 载荷变更 |
| 插件 | `verify-git-plugin` / `test-harmony-plugins` / pnpm 夹具 | 插件变更 |
| 品牌 | `verify-branding` + smoke title | 品牌/上游升级 |
| 构建 | HAP/APK 构建 | 每阶段 |
| 真机 | 安装 → 启动 → 健康 → 首屏 → SSE → 会话持久化 → 工作区/插件 | 有设备时 |
| 回归 | 包哈希、首启耗时、包体、接口/首屏字段 | 运行时/上游变更 |

**证据要求**：每条真机结论记录环境（设备/系统/版本）、步骤、命令、期望、实际、证据文件；未实测标 `⬜`。

---

## 7. 部署与发布

### 7.1 ark

```bash
ohpm install
node scripts/verify-runtime.mjs
node scripts/apply-patches.mjs
node scripts/build-opencode.mjs
node scripts/collect-opencode.mjs
node scripts/smoke-node24.mjs
hvigorw assembleHap --mode module -p product=default -p buildMode=debug --no-daemon
hdc install -r entry/build/default/outputs/default/entry-default-signed.hap
```

### 7.2 hos

```powershell
node scripts\collect-runtime.mjs
node scripts\build-opencode.mjs
node scripts\collect-app.mjs
pwsh -File scripts\build-debug.ps1
hdc app install -r electron\build\default\outputs\default\electron-default-signed.hap
```

### 7.3 android

```bash
node scripts/apply-patches.mjs
OPENCODE_NODE_TARGET=node24 node scripts/build-opencode.mjs
node scripts/collect-plugins.mjs
node scripts/collect-opencode.mjs
node scripts/smoke-node24.mjs
./gradlew :app:assembleDebug
adb install -r app/build/outputs/apk/debug/app-debug.apk
```

### 7.4 发布

- Android：push `v*` tag → CI → GitHub Release（APK + SHA-256）。
- ark/hos：当前本地构建 + 侧载；上架材料按子工程合规清单准备。
- 签名材料永不入库；无签名材料时明确产出 debug/unsigned 包。

---

## 8. 风险与未决项

| # | 项 | 状态 | 处置 |
|---|---|---|---|
| R1 | hos ArkWeb（R2）通道未落地 | `⬜` | 先以 Chromium 为真机通道；补齐 ArkTS 承载后再验收 R2 |
| R2 | ark Web UI 品牌化缺失 | `❌` | 按 `013-branding` 补丁 + 守卫 + 真机证据 |
| R3 | ark 导航围栏代码缺失 | `⚠️` | 补 `onLoadIntercept`/`onWindowNew` 或调整规格并说明风险 |
| R4 | android CI 仍锁 v2.1.2 | `⚠️` | 更新工作流上游 ref 到 `9bbb8ce18c`，并回归 |
| R5 | android TECH/规划仍写 v2.1.2 | `⚠️` | 随 `011-version-2.2.5` 收敛文档 |
| R6 | hos 随机端口导致渲染层状态丢失 | `⚠️` | 评估固定端口/持久化 origin 方案；先登记 |
| R7 | ark/hos v2.2.5 真机证据缺位 | `⬜` | 有设备时补跑 `011-version-2.2.5` 测试用例 |
| R8 | 端口随机与 localStorage 跨重启 | `⬜` | ark/android 补主题/项目列表保持用例 |
| R9 | 插件在线安装/L2 | `⬜` | v1 不做；如需评估需安全评审 |
| R10 | 多版本文档漂移 | `⚠️` | 以源码/补丁为准；规格整理时登记并逐步回写 |
