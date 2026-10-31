# OC Desktop 整体架构（ARCHITECTURE）

> 描述三版别共享的系统结构、进程模型、启动时序、数据流、插件/补丁机制与构建拓扑。
> 技术版本见 [`TECH.md`](./TECH.md)；需求见 [`overall-spec.md`](./overall-spec.md)；实现约定见 [`overall-plan.md`](./overall-plan.md)。
> 证据：根 `README.md`、`docs/能力对照清单.md`、三个子工程的 `specs/ARCHITECTURE.md` / `docs/工程规划*.md` / 源码。未落地项标 `⬜`。

---

## 1. 系统上下文

```
                         ┌──────────────────────────────────────┐
                         │ 上游 opencode（git submodule, 只读）  │
                         │ v2.2.5 / 9bbb8ce18c / Branch_v2.2.5  │
                         │ 服务端 + Web UI + 插件机制            │
                         └───────────────┬──────────────────────┘
                                         │ 幂等补丁 + bundle 构建
             ┌───────────────────────────┼───────────────────────────┐
             │                           │                           │
   ┌─────────▼─────────┐      ┌──────────▼─────────┐      ┌──────────▼─────────┐
   │ oc-desktop-ark    │      │ oc-desktop-hos     │      │ oc-desktop-android │
   │ 进程内 Node       │      │ Electron 37        │      │ 进程内 Node        │
   │ + ArkWeb          │      │ + Chromium         │      │ + 系统 WebView     │
   │ HAP               │      │ HAP                │      │ APK                │
   └─────────┬─────────┘      └──────────┬─────────┘      └──────────┬─────────┘
             │                           │                           │
             └───────────────────────────┼───────────────────────────┘
                                         │
                              ┌──────────▼──────────┐
                              │ 本机 HTTP + SSE 服务 │
                              │ 127.0.0.1:<随机端口> │
                              │ 密码 + 同源 Web UI   │
                              └─────────────────────┘
```

**共同契约**：同一上游基线、同一 stock CLI bundle、同一 `serve` 语义、同一套 Web UI、同一套补丁/插件治理、同一版本号规则。差异只在“谁提供 Node、谁负责渲染、平台缺口怎么补”。

---

## 2. 共享分层模型

```
┌────────────────────────────────────────────────────────────────────┐
│ L4 渲染层                                                           │
│   ArkWeb / Chromium / 系统 WebView                                  │
│   上游 SolidJS SPA：fetch('/api/*') + SSE('/api/event')              │
└──────────────────────────▲─────────────────────────────────────────┘
                           │ 同源 HTTP（零 CORS / 零业务注入桥）
┌──────────────────────────┴─────────────────────────────────────────┐
│ L3 服务层                                                           │
│   opencode v2.2.5 stock CLI bundle：serve --port 0                  │
│   /（内嵌 UI）· /api/* · /auth/* · /api/event（SSE）                │
│   数据：node:sqlite / better-sqlite3（WAL）                          │
└──────────────────────────▲─────────────────────────────────────────┘
                           │ 同进程 import()
┌──────────────────────────┴─────────────────────────────────────────┐
│ L2 运行时层                                                         │
│   node-bridge：dlopen(libnode.so) + node::Start                     │
│   Electron：主进程 Node 运行时                                      │
│   环境契约 / 日志 / 崩溃记录 / 宿主能力桥                            │
└──────────────────────────▲─────────────────────────────────────────┘
                           │ 宿主调用 / 状态回读
┌──────────────────────────┴─────────────────────────────────────────┐
│ L1 宿主层                                                           │
│   ArkTS / Kotlin / Electron 主进程                                  │
│   启动编排 · 加载页/错误页 · 权限 · 目录选择 · 外链 · 单实例          │
└────────────────────────────────────────────────────────────────────┘
```

**架构原则**：壳薄、契约硬；业务逻辑全部在 Node 侧；宿主只做启动编排、观测、宿主能力、渲染。宿主与 Node 之间不新增业务 IPC/自定义协议，业务通道只有同源 HTTP/SSE。

---

## 3. 三版进程模型

### 3.1 `oc-desktop-ark`

| 实体 | 所在 | 说明 |
|---|---|---|
| UIAbility（EntryAbility） | 应用主进程 / 主线程 | 窗口、ArkWeb、启动状态机 |
| Node 运行时 | 同一进程的 detached 线程 | `node::Start` 每进程一次；崩溃 = 应用崩溃 |
| ArkWeb 渲染 | ArkWeb 内部管理 | 只通过 HTTP 与服务交互 |
| 子进程 | 不创建 Node 子进程 | 应用域子进程 JIT 被 seccomp 拦；系统 `sh` 可 spawn |

- 无 Electron、无 Chromium、无 sidecar。
- 载荷优先从 `resfile/opencode/` 只读直读；回退 `tar.gz` 由 Node 自解。
- 默认 compat（jitless）构建，纯 JS fetch/photon/wasm 垫片。

### 3.2 `oc-desktop-hos`

| 实体 | 所在 | 说明 |
|---|---|---|
| Electron 主进程 | 应用主进程 | 创建窗口、启动 Node、加载 UI |
| Node 运行时 | Electron 主进程内（默认 `inprocess`） | 另一候选 `utility` / `worker` 未定标 |
| Chromium 渲染进程 | Electron 管理 | 当前真机通道 |
| ArkWeb（R2） | 目标通道 | ArkTS 承载未落地 `⬜` |
| 子进程 | 不创建 Node 子进程 | 应用域 fork 的 node JIT 有 SIGSYS 风险 |

- Electron 自带 Node 22 / Chromium 138 / ABI 138。
- 载荷 `opencode-dist.tar.gz` 首启流式解压到 `<userData>/opencode-dist/`。
- 数据库 better-sqlite3（aarch64-ohos 原生模块）。

### 3.3 `oc-desktop-android`

| 实体 | 所在 | 说明 |
|---|---|---|
| MainActivity | 应用主进程 / 主线程 | WebView、加载页、错误页、返回语义 |
| Node 运行时 | 同一进程 | `rt.start` → `bootstrap.js` → `opencode-server.js` |
| 系统 WebView | 应用进程内 | 同源加载本机 UI |
| 子进程 | 不 spawn Node | 应用数据分区 noexec；`/system/bin/sh` 可用 |
| 前台服务 | 可选 | 保活能力默认关闭 |

- 载荷 `assets/opencode-dist.tar.gz` 由 Kotlin 字节拷贝到 `incoming/`，Node 自解包（幂等 SHA marker）。
- 数据库 `node:sqlite`（WAL）。
- 插件：Git / 搜索 / 工作区 roots / pnpm。

---

## 4. 启动时序（共同骨架）

```
应用启动
  → 显示应用内加载页（不是白屏）
  → [1] 环境装配：HOME / XDG_* / TMPDIR / PATH / OPENCODE_*
  → [2] 凭据装配：读取或生成连接密码，写沙箱（0600）
  → [3] 载荷就位：只读直读 / 解压 / 自解包 + 哈希校验
  → [4] 宿主能力注册（先于 Node 启动）
  → [5] 启动 Node 运行时（返回“已投递”）
  → [6] 就绪判据：marker / 日志 / 健康检查（不是“进程起来了”）
  → [7] 加载同源 UI：http://127.0.0.1:<port>/?auth_token=***
  → [8] 首屏成功：清理导航历史，进入可用状态
  ↘ 任一步失败 → 错误页（阶段 + 原因 + 日志尾 + 重启提示）
```

版别差异：

| 阶段 | ark | hos | android |
|---|---|---|---|
| 载荷 | resfile 直读 / tar.gz 回退 | tar.gz 首启解压 | assets tar → Node 自解包 |
| 就绪 | marker + `ARK_SERVER_READY` 日志 | `/api/info` + trace | marker(pid) + `/global/health` |
| 鉴权 | `?auth_token=` | 配对码 → 会话 Cookie；Basic 兜底 | `?auth_token=` |
| 失败页 | ArkTS 错误页 | data: loading 失败页 | 原生加载层 + 错误页 |

---

## 5. 数据流

| 方向 | 载体 | 内容 |
|---|---|---|
| 宿主 → Node（启动） | 进程环境 + argv | `OPENCODE_*` / `XDG_*` / `NO_PROXY` / `--hostname` / `--port 0` |
| Node → 宿主（观测） | 文件 + 日志 + marker | `server.ready.json`、`logs/server.log`、崩溃记录 |
| 宿主 → Node（能力） | `HostBridge` / `rt.host` JSON 调用 | 权限、目录选择、外链、通知 |
| UI ↔ 服务 | HTTP + SSE | 会话、消息、工具、权限、事件流 |
| 宿主 → UI（凭据） | 首屏 URL 查询参数 | `auth_token`（一次性，前端读取后抹除） |
| 构建期 → 设备 | tar.gz / resfile / assets | 上游 bundle、UI 归档、插件、原生模块 |

**数据落点**（典型）：

```
<应用沙箱>/
├── connection.json            # 连接密码（0600）
├── data/opencode/opencode.db  # SQLite（WAL）
├── config/opencode/           # 配置 + 本地插件
├── cache/ · state/ · tmp/     # 缓存/状态/临时
├── workspace/                 # 默认工作区
├── logs/                      # 宿主/服务端日志
└── payload/                   # 载荷与就绪 marker
```

---

## 6. 补丁与插件架构

### 6.1 补丁

- 上游 `opencode/` 只读；每个版别在自己的 `patches/<tag>/` 下维护编号补丁。
- 补丁必须幂等、可反向、基线守卫；换 tag 新建目录。
- 当前 v2.2.5 补丁矩阵：

| 版别 | 补丁 | 作用 |
|---|---|---|
| ark | 0001 / 0003 / 0006 / 0007 / 0008 | node target/ohos 条件、运行时打桩、shell、工作区选择、Git 凭据页 |
| hos | 0001–0008 | 在 ark 基础上增加 sqlite、TUI/zed、string-width 等 Electron/ohos 适配 |
| android | 0001 / 0003 / 0006 / 0007 / 0008 / 0009 / 0010 / 0011 | 平台适配 + 工作区导入 + 品牌化 + 硬件返回 |

### 6.2 插件

- L1 本地插件：构建期物化到配置目录，copy 不用 symlink，digest 校验/篡改自愈。
- 插件通过上游 `context.rpc.register` 暴露能力；UI 通过最小补丁消费。
- 当前插件矩阵：

| 插件 | hos | ark | android |
|---|---|---|---|
| Git（isomorphic-git，21 工具 + VCS 覆盖 + 凭据页） | ✅ 204 | ✅ 204 | ✅ 204 |
| 工作区选择器（roots/browse/mkdir/权限） | ✅ 203 | ✅ 203 | ✅ 203（SAF 导入制） |
| 纯 JS 搜索（grep/glob） | ✅ 206 | ❌ 无替代 | ✅ 206 |
| 纯 JS shell（常驻 shell + 哨兵） | ✅ 207 | ❌ 走上游 bash | ❌ 走上游 bash |
| pnpm（进程内） | ❌ | ❌ | ✅ 205 |
| 插件运行时/技能 | ✅ 201/202 | 上游机制 | 上游机制 |

---

## 7. 构建与发布拓扑

### 7.1 本地构建（三版共同骨架）

```
上游源码（固定 tag/提交）
  → 幂等补丁（基线守卫）
  → bun install + build-node --bundle-only
  → 裁剪（非目标平台 ELF/PTY/FFF）
  → 收集插件 + 品牌化/图标
  → tar.gz / resfile / assets + SHA-256 清单
  → host 冒烟门禁
  → HAP / APK 构建
  → 签名（材料不入库）
```

### 7.2 云端发布

- Android：GitHub Actions，push `v*` tag → 载荷五步 → APK → GitHub Release（APK + SHA-256）。当前工作流仍锁 v2.1.2 `a517687fde`，需随 `011-version-2.2.5` 收敛。
- 鸿蒙：暂无云端 tag 发布工作流，当前为本地构建 + 侧载；后续按 `012-building-workflow` 补齐。

---

## 8. 失败模式与处置

| 失败 | 观测点 | 处置 |
|---|---|---|
| 载荷缺失/损坏 | 加载页阶段文案 + 日志 | 错误页；提示重跑 collect / 重装 |
| Node 启动失败（无 JIT/ACL、SIGTRAP） | 崩溃记录 / 日志 | 错误页“缺少必需权限”；不提供假重试 |
| 服务端 import 失败 | `logs/server.log` 栈 | 错误页 + 栈尾；定位补丁/降级 |
| 端口冲突 | 启动日志 | 设计上使用随机端口；出现即缺陷 |
| Web 组件打不开 loopback | `onErrorReceive` / WebView 错误 | 切换网络路线或错误页 |
| 鉴权失败 | UI 停留登录页 | 诊断显示“凭据不匹配”+ 重置入口 |
| 插件加载失败 | 插件日志 / UI | 显式报错；不静默跳过 |
| 品牌化漂移 | 品牌守卫断言 | 构建失败；修复补丁/脚本 |

---

## 9. 关键架构决策与代价

| # | 决策 | 理由 | 代价 |
|---|---|---|---|
| A1 | 上游 stock bundle，不 fork | 降低维护成本；复用官方 UI/服务 | 只能通过 argv/env/补丁表达配置 |
| A2 | Node 进程内运行 | 应用域子进程 JIT 受限；零 IPC | 崩溃即应用崩溃；`node::Start` 每进程一次 |
| A3 | 同源 HTTP + SSE | 零 CORS、零业务注入桥 | 需要随机端口 + 密码 + 同源加载 |
| A4 | 随机端口 + marker | 避免端口冲突；支持多实例隔离 | 渲染层 origin 随端口变化，localStorage 可能丢失 |
| A5 | 幂等补丁 + 基线守卫 | 可重复构建；不污染上游 | 补丁维护成本；共享 submodule 需串行构建 |
| A6 | 本地插件 L1 | 安全/合规；无在线安装面 | 插件必须构建期物化，扩展性受限 |
| A7 | 进程内 Git / 纯 JS 搜索 | 平台无系统 git/rg | 功能/性能受限，需如实报告上限 |
| A8 | 三版共享编号与规格体系 | 对齐能力、降低沟通成本 | 版别差异必须在模块文档中显式登记 |

---

## 10. 已知差异与风险

| 项 | 状态 |
|---|---|
| hos ArkWeb（R2）通道 | 目标形态；当前 ArkTS 承载未落地，真机走 Chromium `⬜` |
| ark Web UI 品牌化 | 未实现；归档 `<title>` 仍为 `OpenCode`，见 `013-branding` 缺口 |
| ark 导航围栏 | 规格有要求，`Index.ets` 未见拦截回调 `⚠️` |
| hos 随机端口导致渲染层状态丢失 | 已定位，未修 `⚠️` |
| android CI 基线 | 工作流仍锁 v2.1.2；源码/补丁已 v2.2.5 `⚠️` |
| android TECH/规划叙述基线 | 部分仍为 v2.1.2 `⚠️` |
| 多版真机证据 | Android v2.2.5 较完整；ark/hos v2.2.5 仅主机编译，真机待补 `⬜` |
| 端口随机与 localStorage | ark/android 机制相同，跨重启主题/项目列表保持未专项验证 `⬜` |
