# registry-deploy

Cloudflare Pages 部署仓库，托管 `registry.presto.app` 的所有静态文件。

## 结构

```
templates/       ← template-registry CI 推送
plugins/         ← plugin-registry CI 推送（将来）
agent-skills/    ← agent-skill-registry CI 推送（将来）
_headers         ← Cloudflare Pages 自定义 headers
```

## 说明

- 本仓库无需构建步骤，根目录即部署内容
- `templates/`、`plugins/`、`agent-skills/` 由各自 registry 的 CI 自动推送，请勿手动编辑
- `_headers` 是唯一需要手动维护的文件
