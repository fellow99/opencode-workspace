# 013-branding-r2 — 包名修改 任务清单（父工程）

> **需求编号:** 013-branding-r2 ｜ **需求名称:** 包名修改 ｜ **git 分支:** 不改动
> **Last Updated:** 2026-11-02

## 变更映射

| 工程 | 现包名 | 新包名 |
|---|---|---|
| oc-desktop-android | `org.fellow99.oc.OCDesktopAndroid`（源码包 `org.fellow99.oc.desktop.android`） | `top.fellow99.oc.OCDesktopAndroid` |
| oc-desktop-hos | `org.fellow99.oc.OCDesktop` | `top.fellow99.oc.OCDesktopHos` |
| oc-desktop-ark | `org.fellow99.oc.OCDesktopArk` | `top.fellow99.oc.OCDesktopArk` |

## 任务依赖序

1. 各子工程依次完成代码/配置改动（android → hos → ark）。
2. android 额外迁移源码包目录 `app/src/main/java/org/fellow99/oc/desktop/android/` → `app/src/main/java/top/fellow99/oc/OCDesktopAndroid/`，并同步 8 个 Kotlin 文件的 `package` 声明。
3. 各子工程编译构建，发现问题修复。
4. 更新父工程 + 各子工程 `specs/013-branding/spec.md` 中的包名字段。
5. 请求代码审查（requesting-code-review）→ 接收并修复（receiving-code-review）。
6. 各子工程按 Conventional Commits 提交；父工程更新 submodule 指针与文档。
7. 推送后父工程 / 子工程 `codegraph sync`。

## 验收

- 三包名分别变为 `top.fellow99.oc.OCDesktopAndroid` / `top.fellow99.oc.OCDesktopHos` / `top.fellow99.oc.OCDesktopArk`。
- android 源码目录与 `namespace`/`applicationId`/8 个 Kotlin `package` 一致。
- 三工程均能编译构建通过。
- `specs/013-branding`（父 + 各子）包名字段同步。
