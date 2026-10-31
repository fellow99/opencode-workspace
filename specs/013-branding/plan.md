# 013-branding 技术方案（As-Built）

> 记录三版品牌化的实际落点、脚本、补丁与缺口。
> 对应规格：[spec.md](./spec.md)
> 最后更新：2026-11-01

## 1. 技术上下文

### 1.1 物料

| 文件 | 尺寸/格式 | 用途 |
|---|---|---|
| `assets/logo.svg` | 矢量 | 字标/网页/派生 |
| `assets/logo.png` | 200×200，含透明 | 通用 logo |
| `assets/logo-1024-with-bg.png` | 1024×1024，不透明 | 应用图标 |
| `assets/logo-200-with-bg.png` | 200×200，不透明 | 小图标 |
| `assets/logo.ico` | ICO | Windows |
| `assets/logo.icns` | ICNS | macOS |
| `assets/screenshots/` | PNG | 产品站点/文档 |

### 1.2 各版落点

| 版别 | 应用名 | UI title | 图标 | 守卫 | 状态 |
|---|---|---|---|---|---|
| hos | `OC Desktop` | 构建期 `branding.mjs` 改归档 title/字标 | `assets/brand/*` + `runtime-overlays` | `scripts/verify-branding.mjs`（37 项） | ✅ |
| android | `OC Desktop` | 补丁 `0010-branding-oc-desktop.patch` 改 `<title>` | 自适应图标 + `scripts/make-brand-materials.mjs` | `scripts/verify-branding.mjs` + `smoke-node24` title | ✅ |
| ark | `OC Desktop Ark`（壳层） | 归档仍为 `OpenCode` | 壳层图标/加载页 | 无 | ❌ 缺口 |

## 2. 宪法合规

| 条款 | 判定 | 说明 |
|---|---|---|
| N2 `013-branding` | ✅ | hos 从 `103-branding` 迁移；父模块新建 |
| FR-BRAND-001 | 🔶 | hos/android ✅；ark 应用名带 Ark 后缀，需确认产品口径 |
| FR-BRAND-002 | ❌ | ark Web UI 未品牌化 |
| FR-BRAND-004 | 🔶 | hos/android 有守卫；ark 无 |
| S5 凭据 | ✅ | 品牌物料不含凭据 |

## 3. 研究结论

1. **hos**：`scripts/lib/branding.mjs` 在 `collect-opencode` 阶段改归档内的 title/字标/PWA；`verify-branding.mjs` 对 `app-archive.bin` 做 30–37 项断言；真机三轮验证。
2. **android**：补丁 0010 改 `index.html` title；`make-brand-materials.mjs` 生成图标；`verify-branding.mjs` 断言 `strings.xml`、图标、manifest；`smoke-node24` 断言 title。真机验证。
3. **ark**：`AppScope`/`entry` 资源显示 `OC Desktop Ark`，加载页有品牌文案；但 `app-archive.bin` 解包实测 `<title>=OpenCode`、`OC Desktop` 0 处，无 0010 等价补丁、无品牌脚本/守卫。
4. **命名口径**：父宪法要求用户可见应用名为 `OC Desktop`；ark 当前 `OC Desktop Ark` 是否保留版别后缀需产品确认 `[NEEDS CLARIFICATION]`。本方案按“应用名统一 OC Desktop，版别后缀仅文档”执行，ark 资源改名列入补齐任务。

## 4. 数据模型

```
BrandAsset {
  source: "assets/logo*.png|svg|ico|icns"
  derived: ["app icon", "pwa icon", "web logo"]
  hash: "sha256"
}
BrandCheck {
  edition: "hos|android|ark"
  checks: ["app_name", "document.title", "pwa_manifest", "icon", "about_page"]
  result: "pass|fail"
}
```

## 5. 接口契约

### 5.1 提供

- 品牌物料与派生规则；
- 各版品牌守卫命令；
- 关于页/非官方声明文案口径。

### 5.2 消费

- `assets/` 物料；
- 上游 UI 归档（`app-archive.bin` / `index.html`）；
- 子工程打包脚本与补丁。

## 6. 实现策略

### 6.1 hos

- 构建期 `scripts/lib/branding.mjs`：
  - 替换 `index.html` 的 `<title>`、PWA manifest、字标；
  - 图标从 `assets/brand/*` 派生；
- `scripts/verify-branding.mjs`：
  - 对归档解包/二进制做 37 项断言；
  - 失败非零退出，阻断打包；
- `runtime-overlays/` 回盖资源，保证 hvigor 不覆盖品牌资源。

### 6.2 android

- 补丁 `patches/v2.2.5/0010-branding-oc-desktop.patch`：
  - `<title>` 改为 `OC Desktop`；
  - 处理 `<title data-sm>` 等变体；
- `scripts/make-brand-materials.mjs`：
  - 从物料生成 mipmap 图标；
- `scripts/verify-branding.mjs`：
  - `strings.xml app_name == OC Desktop`；
  - 无 `OC Desktop Android` 显示名；
  - 图标/关于页/manifest 检查；
- `smoke-node24.mjs` 断言首页 title。

### 6.3 ark 补齐计划（未实现）

1. 新建 `patches/v2.2.5/0010-branding-oc-desktop.patch`（或下一可用编号），改归档 title/manifest/字标；
2. 增加 `scripts/lib/branding.mjs` 或复用 hos 脚本（注意 ark 是 resfile 直读，需改 `app-archive.bin`/`index.html`）；
3. 增加 `scripts/verify-branding.mjs` 与 `smoke-node24` title 断言；
4. 资源显示名统一为 `OC Desktop`（若产品确认去掉 Ark 后缀）；
5. 构建后解包 `app-archive.bin` 验证 `OC Desktop` 出现、`OpenCode` 指称收敛；
6. 真机截图取证；更新 `oc-desktop-ark/specs/013-branding/` 状态。

## 7. 测试与验证

| 版别 | 命令 | 通过标准 |
|---|---|---|
| hos | `node scripts/verify-branding.mjs <archive>` | 37 项断言通过 |
| hos | `node scripts/smoke-node22.mjs` | title 断言通过 |
| android | `node scripts/verify-branding.mjs` | app_name/图标/manifest 通过 |
| android | `node scripts/smoke-node24.mjs` | title 以 OC Desktop 开头 |
| ark | 待补 | 当前无守卫；以归档解包实测为准 |

## 8. 文件清单

| 文件 | 作用 |
|---|---|
| `assets/*` | 品牌物料 |
| `oc-desktop-hos/scripts/lib/branding.mjs` | hos 品牌变换 |
| `oc-desktop-hos/scripts/verify-branding.mjs` | hos 守卫 |
| `oc-desktop-hos/specs/013-branding/` | hos 品牌规格 |
| `oc-desktop-android/patches/v2.2.5/0010-branding-oc-desktop.patch` | android title 补丁 |
| `oc-desktop-android/scripts/make-brand-materials.mjs` | android 图标生成 |
| `oc-desktop-android/scripts/verify-branding.mjs` | android 守卫 |
| `oc-desktop-android/specs/013-branding/` | android 品牌规格 |
| `oc-desktop-ark/specs/013-branding/` | ark 缺口记录与补齐计划 |

## 9. 缺口与风险

| # | 项 | 状态 | 处置 |
|---|---|---|---|
| BR-01 | ark Web UI 未品牌化 | ❌ | 按 §6.3 补齐 |
| BR-02 | ark 应用名带 Ark 后缀 | ⚠️ | 产品确认；默认统一 `OC Desktop` |
| BR-03 | 图标透明度市场接受度 | ⚠️ | 发布前复核商店要求 |
| BR-04 | 上游升级导致 title/manifest 位置变化 | ⚠️ | 每次升级重跑守卫；补丁按 tag 重建 |
