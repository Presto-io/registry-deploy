# registry-deploy — AI 开发提示词

> 本文件是 `Presto-io/registry-deploy` 仓库的 CLAUDE.md。
> 这是一个待创建的新仓库，作为 Cloudflare Pages 的部署目标。

---

## 仓库职责

作为所有 registry 静态文件的统一部署仓库。各 registry 的 CI 将生成的文件推送到本仓库的对应子目录，Cloudflare Pages 自动部署到 `registry.presto.app`。

本仓库**不需要构建步骤**——纯静态文件托管。

---

## 仓库结构

```
registry-deploy/
  templates/                         ← template-registry CI 推送
    registry.json
    gongwen/
      manifest.json
      README.md
      example.md
      preview-1.svg
      preview-2.svg
      hero-frame-0.svg               ← Hero 动画帧（仅官方模板）
      hero-frame-1.svg
      hero-frame-2.svg
      hero-frame-3.svg
    jiaoan-shicao/
      manifest.json
      README.md
      example.md
      preview-1.svg
  plugins/                           ← plugin-registry CI 推送（将来）
    registry.json
    ...
  agent-skills/                      ← agent-skill-registry CI 推送（将来）
    registry.json
    ...
  _headers                           ← Cloudflare Pages 自定义 headers
  _redirects                         ← Cloudflare Pages 重定向规则（如需要）
  README.md
  CLAUDE.md                          ← 本文件
```

---

## Cloudflare Pages 配置

### 连接方式

在 Cloudflare Dashboard 中：
1. Pages → Create a project → Connect to Git
2. 选择 `Presto-io/registry-deploy` 仓库
3. Build settings:
   - Framework preset: None
   - Build command: （留空，无需构建）
   - Build output directory: `/`（根目录即部署内容）
4. Custom domain: `registry.presto.app`

### 自定义 Headers

**文件：** `_headers`

```
/*
  Access-Control-Allow-Origin: *
  Cache-Control: public, max-age=300, s-maxage=3600
  X-Content-Type-Options: nosniff

/*.json
  Content-Type: application/json; charset=utf-8
  Cache-Control: public, max-age=60, s-maxage=300

/*.svg
  Content-Type: image/svg+xml
  Cache-Control: public, max-age=3600, s-maxage=86400

/*.md
  Content-Type: text/markdown; charset=utf-8
  Cache-Control: public, max-age=300, s-maxage=3600
```

说明：
- CORS 全开（`*`）：registry 数据是公开的，任何来源都可以 fetch
- JSON 缓存短（60s/5min）：索引更新后需要较快生效
- SVG 缓存长（1h/24h）：预览图变化频率低
- `s-maxage` 是 CDN 缓存时间，`max-age` 是浏览器缓存时间

---

## URL 映射

部署后的 URL 结构：

```
https://registry.presto.app/templates/registry.json
https://registry.presto.app/templates/gongwen/manifest.json
https://registry.presto.app/templates/gongwen/example.md
https://registry.presto.app/templates/gongwen/preview-1.svg
https://registry.presto.app/templates/gongwen/README.md
https://registry.presto.app/plugins/registry.json              ← 将来
https://registry.presto.app/agent-skills/registry.json          ← 将来
```

---

## 各 registry 的推送约定

- `template-registry` CI 只写 `templates/` 子目录
- `plugin-registry` CI 只写 `plugins/` 子目录
- `agent-skill-registry` CI 只写 `agent-skills/` 子目录
- 各自不干扰，通过 `target-directory` 参数隔离
- 推送 commit message 格式：`chore: update {type} registry`

---

## 初始化步骤

1. 创建 GitHub 仓库 `Presto-io/registry-deploy`
2. 添加 `_headers` 文件
3. 添加 `README.md`
4. 在 Cloudflare Dashboard 连接仓库
5. 配置自定义域名 `registry.presto.app`
6. 在 DNS 中添加 CNAME 记录指向 Cloudflare Pages
7. 生成 SSH deploy key，配置到 `template-registry` 仓库的 secrets

---

## 注意事项

- 本仓库几乎不需要手动操作，所有内容由 CI 自动推送
- 不要手动编辑 `templates/` 等目录下的文件（会被下次 CI 覆盖）
- `_headers` 和 `_redirects` 是唯一需要手动维护的文件
- 如果需要强制刷新 CDN 缓存，可以在 Cloudflare Dashboard 中 purge cache
