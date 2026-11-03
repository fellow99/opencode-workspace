# AGENTS.md — OC Desktop 父工程 AI Agent 工作规范

> 本文件定义了本工作区（父工程）中 AI Agent 必须遵守的规范与工作流程。
> 父工程是**多仓库聚合工作区**：各子工程均为独立 git submodule，有自己的生命周期与 AGENTS.md。

---

## 一、工程定位

**OC Desktop 是 [opencode](https://opencode.ai) 的桌面端。** Windows / Linux / macOS 已有上游官方桌面发行版；
本工作区聚焦把同一份体验带到移动平台 —— 面向鸿蒙与 Android 的**三个独立第三方版别**，均在应用进程内运行 opencode 服务端、同源加载官方 Web UI。

| 版别 | 仓库 | 运行时 | 渲染内核 |
|---|---|---|---|
| HarmonyOS Edition | `oc-desktop-ark` | 进程内 Node v24.2 | ArkWeb（无 Electron、无 Chromium） |
| Electron on HarmonyOS Edition | `oc-desktop-hos` | Electron 37（Node 22 / Chromium 138） | Chromium |
| Android Edition | `oc-desktop-android` | 进程内 Node v24.2 | 系统 WebView |
| 产品官网 | `oc-desktop-website` | 纯静态 HTML/CSS/JS | — |
| Electron 参考工程 | `harmonypc-electron` | Electron-on-HarmonyOS 壳 | — |
| 上游 | `opencode`（submodule，只读） | TypeScript 单体仓库 | — |

- 三版共同上游基线：**opencode v2.2.5**（提交 `9bbb8ce18c` / 分支 `Branch_v2.2.5`）。
- `opencode/` 为**只读上游 submodule**：所有改动以幂等补丁维护在各版别 `patches/` 目录，**严禁直接提交到子模块内**。
- 本工作区是移植/工程化实践，非产品发布；安装方式为侧载（HAP / APK）。

---

## 二、仓库结构与职责分工

| 路径 | 职责 | AGENTS.md |
|---|---|---|
| `oc-desktop-ark/` | HarmonyOS Edition（含自身 specs 体系） | 见该目录 |
| `oc-desktop-hos/` | Electron on HarmonyOS Edition | 见该目录 |
| `oc-desktop-android/` | Android Edition | 见该目录 |
| `oc-desktop-website/` | 产品官网 | 见该目录 |
| `harmonypc-electron/` | Electron 参考壳 | 见该目录 |
| `opencode/` | 上游（只读） | 见该目录（上游自带） |
| `specs/` | 父工程共同规格（constitution / overall-* / 公共 `0XX` 模块） | 本文件 §四 |
| `docs/` | 跨版别文档（`能力对照清单.md` 等） | 本文件 §五 |
| `assets/` | 共享品牌资源（logo、截图） | — |

**D1/D2 分工**：
- **父工程 `specs/`** 关注整体规划、设计、三版共同需求、公共 `0XX` 模块（版本、构建、品牌化）。
- **各子工程 `specs/`** 关注本工程具体技术实现、独有能力与插件、启动时序、真机证据。
- **文档权威顺序**（宪法 D3）：父宪法 → 父 `overall-*.md` → 子工程工程规划 → 子工程 `overall-*.md` → 子工程 `specs/<NNN>-*/` → 代码注释与 README。

---

## 三、CodeGraph 代码地图（规范）

### 3.1 索引部署
- **每个子工程在自身根目录**初始化独立 `.codegraph/` 索引（已全部完成）。
- **父工程根目录**另有聚合索引（递归纳入所有子工程源码），用于跨子工程全局检索。
- 验证索引：`codegraph status`（无索引时 `codegraph init -i`）。

### 3.2 查询规范
- 项目内已安装 **`codegraph-cli`** 技能；凡涉及代码库探索，**优先使用 codegraph**，grep/Read 仅用于确认 codegraph 未覆盖的细节。
- **查询位置**：
  - 只在本子工程内检索 → 到该子工程目录执行 `codegraph query/context/callers` 等；
  - 跨子工程 / 全局检索（例：同名 `EntryAbility` 出现在 ark/hos）→ 到**父工程目录**执行。
- 常用命令：
  - 探索功能区：`codegraph context "<任务描述>"`
  - 查符号：`codegraph query <符号>` · 谁调用：`codegraph callers <符号>` · 调用了谁：`codegraph callees <符号>`
  - 变更影响：`codegraph impact <符号>` · 受影响测试：`codegraph affected <文件...>`
  - 状态检查：`codegraph status`

### 3.3 更新规范（强约束）
> **代码更新后必须保持索引同步，索引是代码的真值镜像。**

- **自动**：每个子工程与父工程均已配置 `post-commit` 钩子，提交后自动执行 `codegraph sync`。
- **手动触发**：
  ```bash
  # 子工程内改完代码后，先到该子工程目录
  cd <子工程目录> && codegraph sync
  # 再到父工程目录，让聚合索引同步（子模块指针变化时）
  cd <父工程> && codegraph sync
  ```
- **增量 vs 全量**：日常用 `codegraph sync`（快）；切换分支 / 大幅重构 / 换依赖用 `codegraph index`（全量重建）。
- **故障**：索引锁残留 `codegraph unlock`；缺索引 `codegraph init -i`。
- `post-commit` 钩子源码在 `scripts/git-hooks/post-commit`；若重装/迁移请重新分发到各 `.git/`（parent 用 `.git/hooks`，submodule 用 `.git/modules/<name>/hooks`，opencode 因 husky 用 `.husky/post-commit`）。

---

## 四、父工程规格体系（specs）

父工程 `specs/` 采用 **speckit / specs-as-built** 约定，结构：

```
specs/
├── README.md              # 规格索引
├── constitution.md        # 最高约束（父宪法）
├── overall-spec.md        # 跨版别共同规格（做什么 + 验收）
├── overall-plan.md        # 跨版别共同方案（怎么做）
├── 011-version-2.2.5/     # 公共：版本更新
├── 012-building-workflow/ # 公共：构建工作流
└── 013-branding/          # 公共：品牌化
```

- 编写/维护规格文档**必须使用 `specs-as-built`** 技能；开发任务用 `specs-based-devflow`；规格说明用 `speckit-*` 技能链。
- 改宪法必须同步父工程 `specs/README.md` 索引与受影响子工程规格索引。

---

## 五、文档与证据

- 跨版别权威对照：`docs/能力对照清单.md`（能力矩阵逐条源码可追溯）。
- **源码即真理**：文档声明须可追溯到实际源码；模块行为不明标注 `[NEEDS CLARIFICATION]`，不得臆测。

---

## 六、通用规范

### 6.1 语言与安全
- 与用户交互、文档、提交注释用**中文**；代码/变量/技术术语用**英文**。
- 严禁提交密钥、Token、密码、私钥等敏感信息。
- 严禁使用 `as any`、`@ts-ignore`、`@ts-expect-error` 压制类型错误；严禁空 `catch(e){}`。

### 6.2 Git 提交（Conventional Commits）
- 提交**必须**加载 `git-commit` 技能并使用 Conventional Commits：
  ```
  <type>[optional scope]: <description>
  ```
- 仅限 11 种 type：`feat` `fix` `docs` `style` `refactor` `perf` `test` `build` `ci` `chore` `revert`。
- description 现在时、祈使句、≤72 字符；严禁 `--no-verify`（除非用户明确要求）。
- **子工程提交在各自子工程目录内执行**；父工程提交在父工程根目录（用于更新 submodule 指针、父级文档等）。
- **严禁把 `opencode/` 的改动直接提交进 submodule**（只读）；用各版别 `patches/` 维护。

### 6.3 工作流规范
- 多步骤任务（2 步+）先建 Todo 列表，同一时间仅一个 `in_progress`。
- 修复 Bug 时**只做最小改动**，严禁顺手重构；禁止臆测未读取代码行为。
- 需求规划用 `brainstorming`（设计批准前不得进入实现）。
- 代码审查用 `requesting-code-review` / `receiving-code-review`；调试用 `systematic-debugging`。

---

> **最后更新**：2026-11-02 · **维护者**：fellow99
