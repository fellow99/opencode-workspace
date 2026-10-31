# OC Desktop 父工程宪法（Constitution）

> **地位**：本文件是 OC Desktop 多工程规格体系的**最高约束**。父工程 `specs/**` 与各子工程 `specs/**` 的 `spec.md` / `plan.md`、实现与发布门禁，若与本文冲突，以本文为准。
> **来源**：条款从根 `README.md`、`docs/能力对照清单.md`、三个子工程的 README / 工程规划 / 既有宪法中**抽取**，不是发明。
> **适用范围**：父工程、`oc-desktop-hos`、`oc-desktop-ark`、`oc-desktop-android`。`opencode/` 为只读上游 submodule，`oc-desktop-website/` 与 `harmonypc-electron/` 为产品站点/参考工程，不纳入本宪法强约束。
> **修改**：改本文必须同步父工程 `specs/README.md` 的索引与受影响子工程的规格索引；涉及编号调整时，必须保留旧目录或迁移记录，禁止静默复用编号。

---

## 一、文档分工

### D1 父工程规范文档关注整体规划、设计，以及各子工程共同的需求

父工程 `specs/` 负责：

- 跨版别共同的产品目标、能力边界、验收总则；
- 三版共同的运行时形态、服务端形态、鉴权、渲染、插件、补丁、品牌化、构建发布规则；
- 版别之间的差异矩阵与对齐结论；
- 模块编号规则、文档治理、证据口径、安全边界、质量门；
- 公共模块 `0XX` 的规格（版本更新、构建工作流、品牌化）。

父工程**不**承担：

- 单个子工程的逐文件实现细节；
- 子工程独有的临时探针、真机日志、未定标方案；
- 上游 `opencode/` 的二次文档化（以上游源码与文档为准）。

### D2 各子工程关注本工程具体的技术实现、独立的需求

子工程 `specs/` 负责：

- 本工程语言/框架/构建链的具体落点（文件、类、脚本、补丁 hunk、版本号）；
- 本工程独有的基础能力与插件（例如 hos 的 Electron 宿主、android 的 pnpm、ark 的 jitless）；
- 本工程的启动时序、数据目录、权限声明面、真机证据；
- 对父工程公共模块的**落地实现**与差异说明。

子工程**不**得：

- 另立一套与父工程冲突的编号体系；
- 把本工程临时方案写成跨版别结论；
- 复制父工程已经定义的共同需求原文后自行改语义。

### D3 文档权威顺序

1. 本宪法；
2. 父工程 `specs/overall-spec.md` / `specs/overall-plan.md`；
3. 子工程 `docs/` 中的工程规划（各工程唯一权威规划）；
4. 子工程 `specs/overall-*.md`；
5. 子工程 `specs/<NNN>-*/spec.md` + `plan.md`；
6. 代码注释与 README。

当子工程规划与父工程宪法冲突时，以父工程宪法为准；当子工程规格与其工程规划冲突时，以工程规划为准并登记差异。

---

## 二、模块编号规则

### N1 编号分段

| 段 | 用途 | 归属 | 示例 |
|---|---|---|---|
| `0XX` | 各子工程**公共、共同**的需求模块 | 父工程 `specs/` 定义；各子工程落地 | `011-version-2.2.5`、`012-building-workflow`、`013-branding` |
| `1XX` | 各工程自己的**基础功能模块** | 各子工程 `specs/` | `101-runtime-integration`、`102-payload-pipeline`、`103-*-shell` |
| `2XX` | 各工程的**补丁、插件模块** | 各子工程 `specs/` | `201-plugin-runtime`、`203-plugin-workspace-picker`、`204-plugin-git` |

编号一旦发布**不复用、不回填**；新增模块追加到对应段末尾。目录名统一为 `<三位编号>-<kebab-name>`。

### N2 `0XX` 公共模块

#### `011-version-<VERSION>` —— 版本更新

- 版本更新类模块**统一命名为** `011-version-<VERSION>`。
- 历史名称 `101-version-<VERSION>` 一律改为 `011-version-<VERSION>`；需求原文中出现的“父工程 `specs/101-version-<VERSION>`”按本条规定解释为 `011-version-<VERSION>`。
- 父工程创建 `specs/011-version-<VERSION>/`，描述跨版别共同的上游基线、版本号规则、补丁矩阵、构建与验收要求。
- 各子工程创建 `specs/011-version-<VERSION>/`，描述本工程的落地差异、真机证据与遗留项。
- 当前共同基线：opencode **v2.2.5**（提交 `9bbb8ce18c`，分支 `Branch_v2.2.5`）；应用版本 **2.2.5** / `versionCode 20205`。

#### `012-building-workflow` —— 构建工作流

- 父工程创建 `specs/012-building-workflow/`，描述跨版别可复现构建、门禁、产物与发布流程。
- Android 版已有 GitHub Actions tag 构建发布工作流，模块编号为 `012-building-workflow`。
- **鸿蒙工程暂无**等价的云端 tag 发布工作流；`oc-desktop-hos` / `oc-desktop-ark` 当前以本地四阶段/五阶段构建脚本为主，不在本迭代虚构 CI 模块。后续若增加，按 `012-building-workflow` 补规格。
- 任何版别新增 CI，必须复用本模块的门禁口径（补丁基线、载荷、host 冒烟、产物哈希），不得只做“能编译”。

#### `013-branding` —— 品牌化

- 父工程创建 `specs/013-branding/`，定义统一品牌名 **OC Desktop**、图标物料、UI 标题、非官方声明与验证口径。
- 各子工程必须有 `specs/013-branding/` 模块文档。
- 若某工程尚未实现品牌化，模块文档必须如实记录**当前缺口、源码证据与补齐计划**，不得把未实现写成已完成。
- 当前状态：`oc-desktop-hos` 已实现完整品牌化；`oc-desktop-android` 已实现补丁 + 图标 + 关于页 + 守卫；`oc-desktop-ark` 壳层有 `OC Desktop Ark` 文案，但 Web UI 仍未品牌化（归档 `<title>` 为 `OpenCode`），列为缺口。

### N3 `1XX` 基础功能模块对齐

以 `oc-desktop-android` 为基准，三版尽量使用相同编号表示同一类基础能力。当前建议基线：

| 编号 | 主题 | Android | ark | hos |
|---|---|---|---|---|
| `101` | 运行时接入 / 宿主启动 | `101-runtime-integration` | `101-runtime-integration` | `101-runtime-host` |
| `102` | 载荷流水线 / 产物就位 | `102-payload-pipeline` | `102-payload-pipeline` | `102-artifact-bootstrap` |
| `103` | 渲染壳 | `103-webview-shell` | `103-arkweb-shell` | `103-render-channel` |
| `104` | 宿主能力 | `104-host-capabilities` | `104-host-capabilities` | 由 `114-workspace-auth` 等模块分担 |
| `105` | 鉴权 / 凭据 | `105-auth` | `105-auth` | `105-request-hooks` |
| `106` | 上游适配 / 补丁 | `106-upstream-adaptation` | `106-upstream-adaptation` | 由 `112-build-pipeline` 与 `204-plugin-git` 等承载 |
| `107` | 合规 | `107-compliance` | `107-compliance` | 由工程规划与 ACL 文档承载 |
| `108` | 签名与身份 | — | — | `108-signing-identity` |
| `109` | 回归基线 | `109-regression` | `109-regression` | 由 `011-version-*` 的真机记录承载 |
| `110` | jitless / 诊断形态 | `110-jitless` | `110-jitless` | 不适用（Electron 完整 JIT） |
| `111` | 生命周期编排 | — | — | `111-lifecycle` |
| `112` | 本机构建流水线 | — | — | `112-build-pipeline` |
| `113` | 服务端与 UI 加载 | — | — | `113-server-ui-load` |
| `114` | 工作区授权 | `203-plugin-workspace-picker`（插件层） | `203-plugin-workspace-picker`（插件层） | `114-workspace-auth` |
| `115` | 数据目录与 SQLite | 由 `101` / `102` 承载 | 由 `101` / `102` 承载 | `115-data-dirs-sqlite` |

> 说明：上表是**对齐基线**，不是强制所有工程都拥有全部编号。各工程可缺少某编号；缺少时应在 `specs/README.md` 说明由哪个模块承载，避免读者误以为遗漏。

### N4 `2XX` 补丁 / 插件模块对齐

| 编号 | 模块 | hos | ark | android |
|---|---|---|---|---|
| `201` | 插件运行时与降级档位 | ✅ | 由上游机制承载 | 由上游机制承载 |
| `202` | 壳专用技能与能力清单 | ✅ | — | — |
| `203` | 工作区选择器插件 | ✅ | ✅ | ✅ |
| `204` | Git 插件 | ✅ | ✅ | ✅ |
| `205` | pnpm 进程内插件 | — | — | ✅ |
| `206` | 纯 JS 搜索 / 工具插件 | ✅ | — | ✅ |
| `207` | 纯 JS shell 工具插件 | ✅ | — | — |

- 历史编号 `108-git-plugin` 一律改为 `204-plugin-git`。
- 历史编号 `112-js-search` 一律改为 `206-js-search`。
- 历史编号 `102-harmony-js-tools` 一律改为 `206-js-tools`。
- 历史编号 `010-tool-shell`（hos）一律改为 `207-plugin-shell`。
- 历史目录 `203-workspace-picker` 统一改为 `203-plugin-workspace-picker`，与 hos/ark 对齐。

### N5 历史目录迁移

迁移必须保留可追溯性：

- Git 仓库内使用 `git mv`，保留文件历史；
- 子工程 `specs/README.md` 记录“旧编号 → 新编号”；
- 规格正文中的模块引用、FR 编号前缀同步更新；
- 不在旧目录保留空壳或占位文档。

---

## 三、证据与文档纪律

### E1 源码与实测是事实源

- 每条结论必须能追溯到源码路径、构建产物、真机命令或上游文档。
- 未实测的结论必须标 `⬜ 待验证`，不得写成已完成。
- 推断必须标 `⚠️ 推断` 或 `[NEEDS CLARIFICATION]`。

### E2 证据标记

| 标记 | 含义 |
|---|---|
| ✅ | 源码确认且有实测/构建证据 |
| 🟢 | 源码确认，未见完整实测 |
| 🔶 | 部分可用 / 受限 / 沿用他工程 |
| ⬜ | 未实现或待验证 |
| ❌ | 平台不可得或明确不做 |
| ⚠️ | 冲突、推断或风险 |

### E3 文档结构

- 每个模块目录必须同时有 `spec.md` 与 `plan.md`；只有一半视为未完成。
- `spec.md` 写“做什么 + 怎么算做完”，技术无关，使用 RFC 2119 的 MUST / SHOULD / MAY。
- 需求编号统一为 `FR-<模块编号>-<三位序号>`。历史模块若沿用 `R1..Rn` 局部编号，必须在模块 README/规格头部登记“`Rn` 等价 `FR-<模块编号>-00n`”，新写模块不得继续使用 `Rn`。
- `plan.md` 写“怎么做 + 在哪做 + 怎么验”，必须引用真实文件路径、依赖版本与 `spec.md` 的 FR 编号。
- 可选 `tasks.md` / `test-cases.md` / `as-built.md` 仅在有实际内容时创建，禁止空占位。

### E4 不编造

- 不写未读过的代码行为；
- 不写不存在的接口、脚本、版本；
- 不把上游未来版本能力写成当前能力；
- 不把“计划中”写成“已实现”。

---

## 四、技术与安全边界

### S1 上游只读、补丁幂等

- `opencode/` submodule 只读；所有改动以 `patches/<tag>/` 幂等补丁表达。
- 补丁必须可反向校验；换基线必须新建补丁目录，不就地改历史补丁。
- 同一时刻只允许一个工程对共享 submodule 打补丁；构建脚本必须做基线守卫与脏树守卫。

### S2 本地服务、最小暴露

- 服务只监听本机回环地址作为产品路径；确需 `0.0.0.0` 时必须保留随机端口 + 密码 + UI 提示。
- 密码由宿主生成并持久化在应用沙箱，权限最小化；不得写入日志、文档、截图或提交。
- `auth_token` 只用于首屏导航；日志与诊断必须脱敏为 `?auth_token=***`。

### S3 权限最小化

- 权限声明面 = 使用面；多一项必须写明理由。
- 不申请与产品能力无关的权限；不把 ACL 缺口静默降级为“可用”。
- 文件/目录访问必须经用户授权或 SAF/Picker 导入，不假设用户目录可写。

### S4 不隐藏平台真相

- 平台做不到的能力必须在 UI / 诊断 / 文档中可见。
- 启动失败必须落到可读错误页（阶段 + 原因 + 日志尾 + 重启提示）。
- 不允许白屏、静默重试、假装成功。

### S5 凭据与密钥

- API Key、签名口令、keystore、`auth_token` 实值永不入库。
- 示例统一使用 `<DEEPSEEK_API_KEY>`、`***`、`<TOKEN>` 等占位符。
- 提交前自检 `sk-`、`ghp_`、`AKIA`、`BEGIN ... PRIVATE KEY` 等模式。

---

## 五、提交与评审纪律

### G1 提交信息

使用 Conventional Commits：

```
<type>[optional scope]: <description>
```

`type` 仅取 `feat | fix | docs | style | refactor | perf | test | build | ci | chore | revert`。描述用祈使句、现在时、<72 字符；一个逻辑变更一个提交。禁止使用其他前缀，禁止 AI 署名脚注。

### G2 代码评审

- 完成规格整理、模块重构、发布前，必须按 `requesting-code-review` 模板发起评审：给出 `DESCRIPTION`、`REQUIREMENTS`、`BASE_SHA`、`HEAD_SHA`。
- 收到评审意见后按 `receiving-code-review` 处理：先核实代码/文档事实，再决定修复或技术性反驳；禁止表演式认同。
- Critical 立即修；Important 在继续前修；Minor 可登记后处理。
- 评审报告落 `specs/REVIEW_REPORT.md` 或子工程 `logs/<YYYYMMDD>-N/REVIEW_REPORT.md`，记录范围、发现、处置与结论。

### G3 质量门

| 门 | 判据 |
|---|---|
| 规格门 | 每个模块 `spec.md` + `plan.md` 成对；索引链接可解析；编号符合本宪法 |
| 补丁门 | 基线正确、工作树干净、补丁可反向 |
| 载荷门 | 上游 bundle 构建成功、SHA-256 清单一致、host 冒烟通过 |
| 构建门 | HAP / APK 可构建；有签名材料则签名 |
| 品牌门 | UI 标题、应用名、图标符合 `013-branding` |
| 证据门 | 真机结论有环境、步骤、判据、证据落点；未实测标 `⬜` |
| 评审门 | 无未决 Critical；Important 已修复或书面接受 |

任一红门不得进入发布。

---

## 六、变更控制

| 变更 | 必须同步 |
|---|---|
| 上游 tag 升级 | `specs/011-version-<新版本>/`、各子工程 `patches/<新 tag>/`、`TECH.md`、`overall-plan.md`、`runtime.lock.json` |
| Node 运行时升级 | 子工程 `TECH.md`、`overall-plan.md`、回归记录、锁值校验脚本 |
| 包名 / 显示名变更 | `013-branding`、`AppScope` / `AndroidManifest`、三套 locale 资源、备案材料 |
| 模块编号调整 | 本宪法 §二、父/子 `specs/README.md`、规格正文引用、FR 编号前缀 |
| 权限 / ACL 变更 | 子工程 ACL 文档、`overall-data-model.md`、`107-compliance`、权限守卫脚本 |
| 构建工作流变更 | `012-building-workflow`、子工程 README、工作流 YAML、测试用例 |

---

## 七、非目标

- 不 fork 上游 opencode；不做长期分叉。
- 不内嵌 Electron/Chromium 到 ark/android；不在鸿蒙版规避 ArkWeb 合规要求。
- 不做 PTY、系统 git、`rg`、本地 stdio MCP、LSP 外部进程、在线插件安装。
- 不做手机专属适配（Android 可用但不做专属设计；鸿蒙只做 2in1/tablet）。
- 不做云端账号、跨设备同步、后台常驻/开机自启。
- 不把父工程规格写成子工程实现手册；不把子工程临时方案上升为跨版别结论。
