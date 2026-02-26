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
_headers                ← Cloudflare Pages 自定义 headers（CORS、缓存策略）
_redirects              ← Cloudflare Pages 重定向规则
```

## URL 映射

```
https://presto.c-1o.top/templates/registry.json
https://presto.c-1o.top/templates/gongwen/manifest.json
https://presto.c-1o.top/templates/gongwen/preview-1.svg
https://presto.c-1o.top/plugins/registry.json              ← 将来
https://presto.c-1o.top/agent-skills/registry.json          ← 将来
```

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
