# MDGJX · 秒达工具箱 — 插件官方仓库

> 秒达工具箱（MDGJX）的插件官方仓库：维护官方与社区插件的**元数据**、**构建打包**与**发布上线**，并兼容转换 uTools（`.upx`）插件生态。

## 项目简介

本仓库是秒达工具箱（MDGJX）桌面端插件体系的后端支撑仓库，承担以下职责：

- **插件登记**：以 TypeScript 声明式配置（`fn_miaoda_registerConfig`）注册每个插件的 id、版本、名称、菜单、运行时等信息；
- **uTools 兼容**：将 uTools 插件包（`.upx`）解包并转换为 MDGJX 可运行的格式（`external-config-a` 运行时）；
- **构建打包**：逐个编译插件，产出 `插件名@版本.tar.gz` 及 SHA-256 校验文件；
- **发布部署**：通过 SSH/SFTP 上传至服务器，并同步至腾讯云 COS，支持测试（test）与正式（prod）两套环境。

## 目录结构

| 目录 | 说明 |
| --- | --- |
| [`api`](api/) | 扩展开发 API 类型定义（React + Node.js），适配自 Raycast API |
| [`common`](common/) | 共享类型定义：`MiaodaBasicConfig`、菜单项、运行时配置等 |
| [`utils`](utils/) | 扩展开发工具函数库 |
| [`static`](static/) | 全局静态资源目录（预留） |
| [`extensions`](extensions/) | 各插件源码，如 `it-tools`、`SRK-Toolbox`、`hello-world` 模板等 |
| [`meta`](meta/) | 插件元数据生成器：生成 `miaoda-dist.json` 与汇总文件 `miaoda-dist-all.json`，内含 `.upx` 转换工具 |
| [`scripts`](scripts/) | 构建、打包、发布流水线脚本 |
| [`e2e`](e2e/) | 端到端转换测试：`.upx` 样例插件包 |
| [`third-party`](third-party/) | 待转换的第三方项目源码 |
| `ext-version.json` | 插件整体版本号（构建 / 发布时使用） |

## 技术栈

| 类别 | 技术 |
| --- | --- |
| 语言 | TypeScript、Node.js、React 18 |
| 包管理 | npm（各目录独立包） |
| 插件打包 | `compressing`、`asar`（`.upx` / `.asar` 解包）、`tar` |
| 元数据生成 | TypeScript 编译 + `lodash`，插件名转拼音使用 `tiny-pinyin` |
| 构建 / 发布 | Bash 脚本、`jq`、`shasum`、SSH / SFTP、腾讯云 COS（`coscli`） |

## 核心功能

1. **插件元数据管理** — 通过 `fn_miaoda_registerConfig()` 集中注册插件配置，支持 `disabled` 开关与排序；
2. **uTools 插件兼容转换** — 解析 `.upx` → 解压 `.gz` / `.asar` → 读取 `plugin.json` → 自动生成 `miaoda-dist.json`（`meta/src/convert-upx.ts`）；
3. **插件构建打包** — 执行 `md-prod-setup` / `md-prod-pack`，输出 `插件名@版本.tar.gz` 并附带 SHA-256 校验文件，支持增量跳过已构建版本；
4. **发布部署** — 上传产物与元数据到服务器并同步 COS，服务器端自动解包、写入安装确认标记；
5. **内置插件** — 目前启用的官方插件：
   - `it-tools`：it-tools 中文版（格式转换、加解密、编码解码、文本处理等常用工具）
   - `SRK-Toolbox`：CyberChef 工具集（Base64、URL 编码、HTML 实体等编解码）
   - 另有 `hello-world`（开发模板）、`lodash`（文档）等示例与备选插件

## 插件运行时类型

| 类型 | 说明 |
| --- | --- |
| `web-static-embedded` | 静态资源内嵌于插件包，随包分发 |
| `web-static-standalone` | 静态资源独立部署，插件包内仅记录端口与在线地址 |
| `external-config-a` | 兼容外部插件（如 uTools 转换而来），由 `plugin.json` 描述 |

## 快速开始

环境要求：Node.js ≥ 18、npm、`jq`、bash。

```bash
# 1. 生成插件元数据（miaoda-dist.json / miaoda-dist-all.json）
cd meta
npm install
npm run build

# 2. 转换 e2e/upx 下的 .upx 样例包（可选）
export MDGJX_EXT_ROOT=$PWD   # 仓库根目录
bash scripts/run-convert-upx.sh

# 3. 构建全部启用插件
bash scripts/build-miaoda-ext.sh

# 4. 发布（默认测试环境，正式环境设置 releaseOrTest=prod）
export releaseOrTest=test
bash scripts/upload-miaoda-ext.sh
```

## 开发一个新插件

1. 在 `extensions/` 下新建目录（参考 [`extensions/hello-world`](extensions/hello-world/) 模板）；
2. 在 [`meta/src/ext/`](meta/src/ext/) 中调用 `fn_miaoda_registerConfig()` 注册插件配置，并在 [`meta/src/index.ts`](meta/src/index.ts) 中引入；
3. 在插件目录内配置 `package.json` 的 `md-prod-setup` / `md-prod-pack` 脚本；
4. 运行 `npm run build`（meta 目录）生成元数据，再执行构建与发布脚本。

## 相关脚本一览

| 脚本 | 用途 |
| --- | --- |
| `scripts/main-update-miaoda-config.sh` | 同步共享类型（m-types）并重建元数据 |
| `scripts/run-convert-upx.sh` | 批量转换 `e2e/upx` 中的 `.upx` 插件 |
| `scripts/build-miaoda-ext.sh` | 编译并打包全部启用插件（tar.gz + sha256） |
| `scripts/upload-miaoda-ext.sh` | 上传插件包与元数据至服务器并同步 COS |
| `scripts/upload-miaoda-gstatic.sh` | 上传全局静态资源（预留） |
| `scripts/convert-external-into-miaoda.sh` | 将 `third-party/` 项目复制到 `extensions/` 并做 utools → mdgjx 文本替换 |
| `scripts/get-ext-version.sh` | 读取 `ext-version.json` 中的整体版本号 |
| `scripts/dev-run-miaoda-ext.sh` | 本地开发运行指定插件 |

## 许可证

[AGPL-3.0](LICENSE)
