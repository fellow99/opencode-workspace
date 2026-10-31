# 规格整理 Code Review 报告

> 评审方法：按 `requesting-code-review` 模板定义范围与验收要求；因当前环境无子代理派发工具，采用**独立只读自审 + 静态检查**（链接解析、编号扫描、目录清单、FR 前缀、源码抽样）执行；处置按 `receiving-code-review` 的“先核实、再修复/反驳”纪律。
> 评审范围：父工程 `specs/` 全量新增；`oc-desktop-hos`、`oc-desktop-ark`、`oc-desktop-android` 的 `specs/` 编号迁移、公共模块补充与索引更新。
> Base：父 `dff70f4` / hos `93f5497` / ark `cfdd41c` / android `57c553f`；Head：本次工作树（未提交）。
> 日期：2026-11-01

---

## 一、评审要求（REQUIREMENTS）

1. 确认并遵循 `git-commit` 的 Conventional Commits 前缀，不使用其它前缀。
2. 使用 `requesting-code-review` / `receiving-code-review` 完成评审与处置。
3. 使用 `specs-as-built` 方法编写规范文档，源码/构建/真机证据为事实源。
4. 父工程输出：`README.md`、`STRUCTURE.md`、`TECH.md`、`ARCHITECTURE.md`、`constitution.md`、`overall-spec.md`、`overall-plan.md`；不写 `API.md`、`overall-api.md`、`overall-data-model.md`、`overall-test-cases.md`。
5. 编号规则写入父宪法：`0XX` 公共、`1XX` 基础、`2XX` 插件；`101-version-<VERSION>` → `011-version-<VERSION>`；`012-building-workflow`（鸿蒙暂无）；`013-branding`（缺失工程补文档）。
6. 整理父工程与三个子工程的 `specs/`，并以 `oc-desktop-android` 为基准尽量对齐 `1XX`，插件尽量对齐 `2XX`。

---

## 二、Strengths（通过项）

- **父工程文档齐全**：宪法、整体规格/方案、技术选型、架构、结构、索引均已生成；未生成被明确排除的 API/数据模型/测试索引。
- **编号规则落地**：父宪法明确 `0XX/1XX/2XX`、`011-version-*`、`012-building-workflow`、`013-branding`、历史迁移表与证据/安全/提交/质量门。
- **子工程迁移完整**：hos 的 12 个 `0XX` 基础模块全部迁入 `1XX/2XX`；ark/android 的版本、品牌、Git、搜索、构建模块已按公共/插件段重排；所有模块目录均具备 `spec.md` + `plan.md`。
- **品牌化缺口如实记录**：ark 未实现 Web UI 品牌化，`013-branding` 以 gap record 形式记录源码证据与补齐计划，没有伪造成已完成。
- **链接可解析**：对父/子 specs 的 Markdown 相对链接做解析，0 个断链。
- **旧路径清理**：旧模块 slug 仅保留在迁移表/历史说明中；规格正文与索引已指向新编号。
- **构建漂移已登记**：Android CI 仍锁 v2.1.2、ark/hos 真机证据缺位、hos 随机端口状态丢失等，均在父/子规格中标记 `⚠️`/`⬜`。

---

## 三、Issues（发现与处置）

### Critical

无。未发现会阻断文档提交或导致规格体系自相矛盾的 Critical 项。

### Important

| # | 发现 | 核实 | 处置 |
|---|---|---|---|
| I-1 | Android CI 工作流仍检出 `a517687fde`（v2.1.2），与源码/补丁 v2.2.5 `9bbb8ce18c` 不一致 | ✅ 实锤：`.github/workflows/build-release.yml` 的 `ref` 与 Release body | **不在本次规格提交内修**（涉及发布链变更）；已在父 `011-version-2.2.5`、`012-building-workflow`、`TECH.md` 登记 `⚠️`，作为发布前阻塞项 |
| I-2 | ark Web UI 未品牌化，归档 `<title>` 仍为 `OpenCode` | ✅ 实锤：`app-archive.bin` 解包/源码无品牌补丁/脚本 | 新增 `oc-desktop-ark/specs/013-branding/` gap record，记录缺口与补齐方案；父 `013-branding` 标 `❌` |
| I-3 | ark 应用显示名为 `OC Desktop Ark`，与父宪法“显示名统一 OC Desktop”存在产品口径冲突 | ✅ 源码资源确认；产品意图未定 | 在 `013-branding` 标 `[NEEDS CLARIFICATION]`，默认按统一 `OC Desktop` 记录，不擅自改源码 |
| I-4 | Android 既有模块使用 `R1..Rn` 局部需求编号，与父宪法 `FR-<模块>-NNN` 不一致 | ✅ 抽查 15 个 spec | 保留历史编号以不破坏已有引用；在 Android README 与每个 spec 头部登记 `Rn` ↔ `FR-<模块>-00n` 映射；新模块统一 FR |

### Minor

| # | 发现 | 处置 |
|---|---|---|
| M-1 | hos `013-branding` 正文仍含 v2.0.26 历史版本号与“4 条补丁”表述 | 在 spec 头部增加“基线说明”：本文记录首次落地轮，当前基线 v2.2.5，守卫仍适用；不改写历史事实 |
| M-2 | ark 顶层 `TECH/overall-plan/ARCHITECTURE/constitution` 原以 v2.0.26 叙述 | 已更新当前基线、补丁数（5 条）、版本号到 v2.2.5/20205；模块内历史证据保留 |
| M-3 | Android `TECH/STRUCTURE/overall-data-model/constitution` 原以 v2.1.2 叙述 | 已更新上游基线、版本号、补丁数（8 条）与当前产物描述 |
| M-4 | 迁移表保留旧 slug，静态扫描会命中 | 属有意保留的可追溯记录，不删除；扫描器仅对正文/索引报错 |
| M-5 | 父工程未生成 `SPECS_CHECKLIST.md` | 用户输出清单未要求；以 `README.md` 文档总览 + 本评审报告承担完成度追踪 |
| M-6 | 鸿蒙工程无 `012-building-workflow` 模块 | 父宪法与父 `012` 明确“鸿蒙暂无”；hos/ark README 说明由本地构建流水线承载，不虚构 CI |

---

## 四、Verification（复核结果）

| 检查 | 方法 | 结果 |
|---|---|---|
| 相对链接 | Python 解析父/子 `specs/**/*.md` 的 Markdown 链接并检查目标存在 | ✅ 0 broken |
| 模块成对 | 遍历所有编号目录，检查 `spec.md` + `plan.md` | ✅ 全部成对（本次补齐 Android 8 个 plan） |
| 编号重复 | 按三位编号分组检查 | ✅ 仅 `011-version-*` 多版本目录为预期重复 |
| 旧路径 | 扫描旧 slug | ✅ 仅迁移表/历史说明命中 |
| FR 前缀 | 检查每模块至少含本模块 FR 或历史映射说明 | ✅ 通过（Android 历史 R 编号已登记映射） |
| 文档计数 | 父/子索引与实际目录比对 | ✅ 已修正 Android 17、ark 14 模块计数 |
| 敏感信息 | 抽查新增/修改文档 | ✅ 无真实 Key/口令/token；示例为占位符 |
| 源码事实 | 版本号/补丁数/CI 基线/品牌缺口抽样核对 | ✅ 与 `AppScope`/`build.gradle.kts`/`patches`/workflow 一致 |

---

## 五、Assessment

**Ready for documentation commit**（文档整理范围）。

- Critical：0
- Important：4 项均已记录/降级为发布前待办，其中 I-1（CI 基线）与 I-2（ark 品牌化）是**发布门禁**，不是规格文档缺陷。
- Minor：已记录，未阻塞。

**发布前仍需关闭**：

1. 更新 Android CI 上游 `ref` 到 `9bbb8ce18c` 并彩排；
2. ark 品牌化补齐（补丁/脚本/守卫/真机）；
3. ark/hos v2.2.5 真机证据；
4. ark 应用名后缀的产品口径确认。
