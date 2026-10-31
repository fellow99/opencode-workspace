# OC Desktop 项目结构（STRUCTURE）

> 生成方式：2026-11-01 扫描父工程与三个子工程目录，排除 `.git/`、构建缓存、`node_modules/`、`oh_modules/`、`logs/`、`build/`。
> 用途：给规格、评审、 onboarding 提供目录与入口索引；不替代各子工程 `STRUCTURE.md`。
> 标注：`[submodule]` 独立仓库｜`[生成]` 构建/运行生成、通常不入库｜`[本地]` 本机材料、gitignore。

---

## 一、父工程根目录

```
D:/opencode-workspace/
├── README.md                     # 英文项目介绍（三版矩阵/能力/布局）
├── README_zh.md                  # 中文项目介绍
├── LICENSE                       # MIT
├── .gitmodules                   # 6 个 submodule 定义
├── .gitignore
├── assets/                       # 共享品牌物料
│   ├── logo.svg / logo.png
│   ├── logo-1024-with-bg.png / logo-200-with-bg.png
│   ├── logo.ico / logo.icns
│   └── screenshots/
├── docs/
│   └── 能力对照清单.md            # 三版能力矩阵 + 源码核实 + 不一致台账
├── specs/                        # ← 本次生成的父工程规范文档
│   ├── README.md
│   ├── STRUCTURE.md
│   ├── TECH.md
│   ├── ARCHITECTURE.md
│   ├── constitution.md
│   ├── overall-spec.md
│   ├── overall-plan.md
│   ├── 011-version-2.2.5/{spec.md,plan.md}
│   ├── 012-building-workflow/{spec.md,plan.md}
│   └── 013-branding/{spec.md,plan.md}
├── opencode/                     [submodule] 上游 opencode（只读）
├── oc-desktop-ark/               [submodule] HarmonyOS Edition
├── oc-desktop-hos/               [submodule] Electron on HarmonyOS Edition
├── oc-desktop-android/           [submodule] Android Edition
├── oc-desktop-website/           [submodule] 产品站点（不在本次范围）
├── harmonypc-electron/           [submodule] Electron-on-HarmonyOS 参考壳
├── harmonypc-electron-versions/  [local] 运行时归档/版本材料（gitignored symlink）
└── logs/                         [本地/生成] 开发/评审/测试证据（gitignored）
```

### 1.1 Submodule 清单（`.gitmodules`）

| 路径 | 仓库 | 角色 |
|---|---|---|
| `opencode/` | `anomalyco/opencode` | 上游，只读，pin v2.2.5 `9bbb8ce18c` |
| `oc-desktop-ark/` | `fellow99/oc-desktop-ark` | 鸿蒙 Ark 版 |
| `oc-desktop-hos/` | `fellow99/oc-desktop-hos` | 鸿蒙 Electron 版 |
| `oc-desktop-android/` | `fellow99/oc-desktop-android` | Android 版 |
| `oc-desktop-website/` | `fellow99/oc-desktop-website` | 产品站点 |
| `harmonypc-electron/` | `fellow99/harmonypc-electron` | 参考工程 |

---

## 二、`oc-desktop-ark/` 结构

```
oc-desktop-ark/
├── README.md
├── AppScope/
│   ├── app.json5                 # bundleName / versionName / versionCode / icon / label
│   └── resources/base/element/string.json
├── entry/
│   ├── src/main/ets/
│   │   ├── app/                  # Connection / OpencodeRuntime / Payload
│   │   ├── common/               # BuildConfig / Constants / Log / NodeMode / Ports
│   │   ├── entryability/         # EntryAbility
│   │   ├── host/                 # HostCapabilities
│   │   └── pages/                # Index.ets（加载页 + ArkWeb）
│   ├── src/main/resources/
│   │   ├── rawfile/boot.html
│   │   ├── resfile/opencode/     # 载荷（生成/投放）
│   │   └── base|en_US|zh_CN/     # 资源
│   └── build-profile.json5
├── payload/
│   ├── opencode-server.js        # Node 入口
│   ├── bridge/                   # bootstrap.js / host.js
│   ├── lib/                      # env.js / plugins.js / ready.js
│   └── jitless/                  # fetch-shim / photon-shim / wasm-patch / undici
├── plugins/
│   ├── oc-plugin-git/            # isomorphic-git（21 工具）
│   └── oc-plugin-workspace-roots/# 三根工作区 RPC
├── patches/
│   ├── v2.0.26/                  # 历史锚点（5 条）
│   ├── v2.1.2/                   # 历史锚点（5 条）
│   └── v2.2.5/                   # 当前（0001/0003/0006/0007/0008）
├── scripts/                      # apply-patches / build-opencode / collect-opencode /
│                                 # collect-plugins / smoke-node24 / verify-runtime /
│                                 # verify-git-plugin / check-acl-surface
├── libs/@fellow99/node-bridge.har
├── runtime.lock.json             # libnode SHA-256 / 体积 / ABI
├── oh-package.json5              # node-bridge 0.1.0 + node-runtime 24.2.0
├── docs/000-工程规划.md           # 唯一权威规划
├── docs/ACL配置.md / ACL-无ACL评估.md
└── specs/                        # 规格体系（见 §2.1）
```

### 2.1 `oc-desktop-ark/specs/` 模块

```
specs/
├── README.md / STRUCTURE.md / TECH.md / ARCHITECTURE.md / constitution.md
├── SPECS_CHECKLIST.md
├── overall-spec.md / overall-plan.md / overall-data-model.md / overall-test-cases.md
├── 011-version-2.1.2/{spec,plan,tasks,test-cases}.md
├── 011-version-2.2.5/{spec,plan,tasks,test-cases}.md
├── 013-branding/{spec,plan}.md                    # 当前 gap record
├── 101-runtime-integration/{spec,plan}.md
├── 102-payload-pipeline/{spec,plan}.md
├── 103-arkweb-shell/{spec,plan}.md
├── 104-host-capabilities/{spec,plan}.md
├── 105-auth/{spec,plan}.md
├── 106-upstream-adaptation/{spec,plan}.md
├── 107-compliance/{spec,plan}.md
├── 109-regression/{spec,plan}.md
├── 110-jitless/{spec,plan,tasks,test-cases}.md
├── 203-plugin-workspace-picker/{spec,plan,tasks,test-cases}.md
└── 204-plugin-git/{spec,plan,tasks,test-cases}.md
```

> `012-building-workflow` 在 ark 暂无（鸿蒙无云端 tag 发布工作流）。

---

## 三、`oc-desktop-hos/` 结构

```
oc-desktop-hos/
├── README.md
├── AppScope/
│   ├── app.json5                 # org.fellow99.oc.OCDesktop / 2.2.5 / 20205
│   └── resources/
├── electron/                     # entry 模块：Electron 运行时 + Ability 装配
│   ├── src/main/ets/
│   │   ├── entryability/         # EntryAbility / BrowserAbility / StatusBar...
│   │   ├── pages/                # Index / WebPage / Login / Window...
│   │   ├── application/          # AbilityStage
│   │   └── process/              # CustomChildProcess
│   └── libs/arm64-v8a/           # Electron SO / Chromium 资源（生成/投放）
├── web_engine/                   # HAR：ArkTS 桥接层 + resfile 产物
│   └── src/main/ets/
│       ├── ability/              # WebAbility / WebBaseAbility ...
│       ├── adapter/              # 60+ Adapter（权限/文件/通知/...）
│       ├── components/           # WebWindow / SubWindow ...
│       └── jsbindings/
├── src-main/                     # Electron 主进程源码
│   ├── main.js                   # 启动、服务端、窗口、渲染通道、鉴权
│   ├── sidecar.js
│   ├── tar-extract.js
│   └── package.json
├── scripts/                      # collect-runtime / build-opencode / collect-app /
│                                 # collect-opencode / smoke-node22 / verify-branding /
│                                 # test-harmony-plugins / verify-git-plugin
├── patches/
│   ├── v2.0.26/                  # 历史（8 条）
│   ├── v2.1.2/                   # 历史（8 条）
│   └── v2.2.5/                   # 当前（0001–0008）
├── plugins/
│   ├── oc-plugin-git/
│   ├── oc-plugin-harmony-search/
│   ├── oc-plugin-harmony-shell/
│   └── oc-plugin-workspace-roots/
├── skills/harmony-runtime-capabilities/SKILL.md
├── runtime-overlays/             # copy 后回盖白名单（模块清单/显示名）
├── docs/                         # 工程规划 V1/V2 + 能力清单 + ACL + toybox + 真机记录
│   ├── 工程规划-V2.md            # 唯一权威规划
│   ├── 鸿蒙环境能力清单*.md
│   ├── ACL申请清单.md
│   ├── toybox命令清单.md
│   ├── 真机验证记录.md
│   ├── 203-插件-选择工作区目录.md
│   └── 204-插件-GIT.md
├── specs/                        # 规格体系（见 §3.1）
├── signing.debug.local.json.sample
├── signing.release.local.json.sample
├── build-profile.json5 / hvigorfile.ts / oh-package.json5
└── assets/brand/                 # 品牌图标母版
```

### 3.1 `oc-desktop-hos/specs/` 模块

```
specs/
├── README.md
├── 011-version-2.0.26/{spec,plan,tasks,test-cases}.md
├── 011-version-2.1.2/{spec,plan,tasks,test-cases}.md
├── 011-version-2.2.5/{spec,plan,tasks,test-cases}.md
├── 013-branding/{spec,plan,tasks,test-cases}.md
├── 101-runtime-host/{spec,plan}.md
├── 102-artifact-bootstrap/{spec,plan}.md
├── 103-render-channel/{spec,plan}.md
├── 105-request-hooks/{spec,plan}.md
├── 108-signing-identity/{spec,plan}.md
├── 111-lifecycle/{spec,plan}.md
├── 112-build-pipeline/{spec,plan}.md
├── 113-server-ui-load/{spec,plan}.md
├── 114-workspace-auth/{spec,plan}.md
├── 115-data-dirs-sqlite/{spec,plan}.md
├── 201-plugin-runtime/{spec,plan}.md
├── 202-skill-runtime-capabilities/{spec,plan}.md
├── 203-plugin-workspace-picker/{spec,plan,tasks,test-cases}.md
├── 204-plugin-git/{spec,plan,tasks,test-cases}.md
├── 206-js-tools/{spec,plan,tasks,test-cases}.md
└── 207-plugin-shell/{spec,plan}.md
```

> `012-building-workflow` 在 hos 暂无；105/108/111 等编号与父宪法对齐表一致。

---

## 四、`oc-desktop-android/` 结构

```
oc-desktop-android/
├── README.md
├── app/
│   ├── build.gradle.kts           # applicationId / versionName 2.2.5 / versionCode 20205
│   ├── src/main/
│   │   ├── AndroidManifest.xml    # 权限/Activity/单实例
│   │   ├── java/org/fellow99/oc/desktop/android/
│   │   │   ├── App.kt
│   │   │   ├── MainActivity.kt    # WebView / 加载页 / 错误页 / 返回
│   │   │   ├── OpencodeRuntime.kt # 六阶段启动编排
│   │   │   ├── ConnectionStore.kt
│   │   │   ├── HostCapabilities.kt
│   │   │   ├── Diagnostics.kt
│   │   │   └── KeepaliveService.kt
│   │   ├── assets/                # opencode-dist.tar.gz（生成） + manifest + notices
│   │   └── res/                   # 图标/字符串/network_security_config
│   └── src/debug/                 # 探针源集（已清理）
├── payload/
│   ├── opencode-server.js         # Node 入口
│   ├── unpack.js                  # tar.gz 自解包
│   └── lib/                       # env.js / plugins.js / ready.js
├── plugins/
│   ├── oc-plugin-git/
│   ├── oc-plugin-pnpm/
│   ├── oc-plugin-search/
│   └── oc-plugin-workspace-roots/
├── patches/
│   ├── v2.1.2/                    # 历史（8 条）
│   └── v2.2.5/                    # 当前（0001/0003/0006/0007/0008/0009/0010/0011）
├── scripts/                       # apply-patches / build-opencode / collect-opencode /
│                                  # collect-plugins / smoke-node24 / verify-runtime /
│                                  # verify-branding / verify-git-plugin / regression /
│                                  # make-brand-materials / lib/png.mjs
├── runtime.lock.json              # node-runtime 24.22.1 锁值
├── gradle/libs.versions.toml      # AGP 8.9.2 / Kotlin 2.1.20 / SDK 36
├── settings.gradle.kts / build.gradle.kts
├── .github/workflows/build-release.yml
├── docs/                          # 工程规划 + 能力清单 + 真机记录 + 安装 + evidence
└── specs/                         # 规格体系（见 §4.1）
```

### 4.1 `oc-desktop-android/specs/` 模块

```
specs/
├── README.md / STRUCTURE.md / TECH.md / ARCHITECTURE.md / constitution.md
├── SPECS_CHECKLIST.md
├── overall-spec.md / overall-plan.md / overall-data-model.md / overall-test-cases.md
├── 011-version-2.1.2/spec.md
├── 011-version-2.2.5/{spec,plan,tasks,test-cases}.md
├── 012-building-workflow/{spec,plan,tasks,test-cases}.md
├── 013-branding/{spec,plan}.md
├── 101-runtime-integration/spec.md
├── 102-payload-pipeline/spec.md
├── 103-webview-shell/spec.md
├── 104-host-capabilities/spec.md
├── 105-auth/spec.md
├── 106-upstream-adaptation/{spec,plan}.md
├── 107-compliance/{spec,plan}.md
├── 109-regression/{spec,plan,as-built}.md
├── 110-jitless/spec.md
├── 203-plugin-workspace-picker/spec.md
├── 204-plugin-git/{spec,plan}.md
├── 205-pnpm-plugin/{spec,plan,as-built}.md
└── 206-js-search/{spec,plan}.md
```

---

## 五、父工程 `specs/` 文件索引

| 文件 | 作用 |
|---|---|
| `README.md` | 父工程规范文档索引、模块清单、导航 |
| `STRUCTURE.md` | 本文件：目录与入口 |
| `TECH.md` | 三版技术选型/版本矩阵/质量门 |
| `ARCHITECTURE.md` | 整体架构、进程模型、启动时序、补丁/插件、构建拓扑 |
| `constitution.md` | 宪法：文档分工、编号规则、证据/安全/提交/质量门 |
| `overall-spec.md` | 技术无关的整体规格与验收总则 |
| `overall-plan.md` | 技术具体的整体方案、横切、测试、部署、风险 |
| `011-version-2.2.5/` | 公共版本模块 |
| `012-building-workflow/` | 公共构建工作流 |
| `013-branding/` | 公共品牌化 |
| `REVIEW_REPORT.md` | 本次规格整理的 code review 记录（如已生成） |

> 按需求约定：父工程**不生成** `API.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`。

---

## 六、关键入口速查

| 目的 | 路径 |
|---|---|
| 三版能力对照 | `docs/能力对照清单.md` |
| 三版 README | `oc-desktop-{ark,hos,android}/README.md` |
| 三版规划 | `oc-desktop-ark/docs/000-工程规划.md`、`oc-desktop-hos/docs/工程规划-V2.md`、`oc-desktop-android/docs/000-工程规划.md` |
| 上游基线 | `opencode/`（submodule） |
| 补丁 | `oc-desktop-*/patches/v2.2.5/` |
| 构建脚本 | `oc-desktop-*/scripts/` |
| Android CI | `oc-desktop-android/.github/workflows/build-release.yml` |
| 品牌物料 | `assets/` |
| 真机证据 | `oc-desktop-{ark,hos,android}/logs/`、`docs/真机验证记录.md`、`docs/evidence/` |
| 父规格 | `specs/` |
| 子规格 | `oc-desktop-{ark,hos,android}/specs/` |
