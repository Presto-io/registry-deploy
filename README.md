# registry-deploy — AI 开发指南

> 组织级规则见 `../Presto-homepage/docs/ai-guide.md`

所有 registry 静态文件的统一部署仓库。各 registry 的 CI 将生成的文件推送到本仓库的对应子目录，Cloudflare Pages 自动部署到 `presto.c-1o.top`。

本仓库**不需要构建步骤**——纯静态文件托管。

## 仓库结构

```
templates/              ← template-registry CI 推送
  registry.json
  gongwen/
    manifest.json, README.md, example.md, preview-*.svg, hero-frame-*.svg
  jiaoan-shicao/
    manifest.json, README.md, example.md, preview-*.svg
plugins/                ← plugin-registry CI 推送（将来）
agent-skills/           ← agent-skill-registry CI 推送（将来）
.github/
  workflows/
    sync-to-r2.yml      ← 自动同步二进制到 R2 CDN
docs/
  R2-CDN-QUICKSTART.md  ← CDN 配置快速指南
  R2-CDN-SETUP.md       ← CDN 详细配置文档
_headers                ← Cloudflare Pages 自定义 headers（CORS、缓存策略）
_redirects              ← Cloudflare Pages 重定向规则
```

## URL 映射

```
https://presto.c-1o.top/templates/registry.json
https://presto.c-1o.top/templates/gongwen/manifest.json
https://presto.c-1o.top/templates/gongwen/preview-1.svg
https://cdn.presto.c-1o.top/templates/binaries/v1.0.0/gongwen/darwin-arm64  ← CDN 二进制
https://presto.c-1o.top/plugins/registry.json              ← 将来
https://presto.c-1o.top/agent-skills/registry.json          ← 将来
```

## CDN 自动化

### 🎯 快速配置

首次配置请查看：**[CDN 快速开始指南](docs/R2-CDN-QUICKSTART.md)**

只需 3 步：
1. 创建 Cloudflare R2 存储桶和 API Token
2. 配置 GitHub Secrets（3 个必需）
3. 手动触发首次同步

### 🚀 自动化流程

配置完成后，发布新版本时 CDN 会自动同步：

```
presto-official-templates (发布 Release)
    ↓ 触发
registry-deploy (GitHub Actions)
    ↓ 下载
GitHub Releases
    ↓ 上传
Cloudflare R2
    ↓ 通过
cdn.presto.c-1o.top
    ↓ 更新
registry.json (cdn_url 字段)
```

### 📦 R2 存储结构

```
presto-cdn/
  templates/
    binaries/
      v1.0.0/
        gongwen/
          darwin-arm64
          darwin-amd64
          ...
        jiaoan-shicao/
          darwin-arm64
          ...
```

### 📋 需要的 GitHub Secrets

| Secret 名称 | 必需 | 说明 |
|------------|-----|------|
| `R2_ACCESS_KEY_ID` | ✅ | Cloudflare R2 Access Key ID |
| `R2_SECRET_ACCESS_KEY` | ✅ | Cloudflare R2 Secret Access Key |
| `R2_ACCOUNT_ID` | ✅ | Cloudflare Account ID |
| `R2_BUCKET` | ❌ | 存储桶名称（默认：`presto-cdn`） |
| `CDN_DOMAIN` | ❌ | CDN 域名（默认：`cdn.presto.c-1o.top`） |

### 🔧 手动触发同步

如果需要手动触发同步：

1. 进入 **Actions** → **Sync Binaries to R2 CDN**
2. 点击 **Run workflow**
3. 输入版本号（如：`v1.0.0`）
4. 点击 **Run workflow**

## 各 registry 的推送约定

- `template-registry` CI 只写 `templates/` 子目录
- `plugin-registry` CI 只写 `plugins/` 子目录
- `agent-skill-registry` CI 只写 `agent-skills/` 子目录
- 各自不干扰，通过 `target-directory` 参数隔离
- 推送 commit message 格式：`chore: update {type} registry`

## 跨仓库推送方式

各 registry 仓库的 CI 通过 `PRESTO_PAT`（Personal Access Token）推送到本仓库，与 `Presto-Homepage` 的 CI 使用同一模式。不使用 SSH Deploy Key。

## 注意事项

- 本仓库几乎不需要手动操作，所有内容由 CI 自动推送
- 不要手动编辑 `templates/` 等目录下的文件（会被下次 CI 覆盖）
- `_headers` 和 `_redirects` 是唯一需要手动维护的文件
- 如果需要强制刷新 CDN 缓存，可以在 Cloudflare Dashboard 中 purge cache

## 成本说明

### Cloudflare Pages（静态文件）
- **存储**：免费（1GB）
- **带宽**：免费（无限）
- **请求**：免费（10万次/月）

### Cloudflare R2（二进制 CDN）
- **存储**：前 10GB 免费，之后 $0.015/GB/月
- **操作**：前 100 万次免费
- **出站流量**：**完全免费**（无限）✨

## 相关文档

- [CDN 快速开始](docs/R2-CDN-QUICKSTART.md) - 3 步配置指南
- [CDN 详细配置](docs/R2-CDN-SETUP.md) - 完整配置和故障排查
