# Cloudflare R2 CDN 自动化 - 快速开始

## 🎯 你需要做什么（3 个步骤）

### 1️⃣ Cloudflare 配置（10 分钟）

在 [Cloudflare Dashboard](https://dash.cloudflare.com/) 中：

**创建 R2 存储桶：**
1. 进入 **R2 对象存储**
2. 创建存储桶：名称 `presto-cdn`
3. 记录 **Account ID**（在 R2 页面可以找到）

**创建 API Token：**
1. 点击 **管理 R2 API 令牌** → **创建 API 令牌**
2. 配置：
   - 权限：对象读和写
   - 存储桶：`presto-cdn`
   - TTL：永久
3. **保存**：Access Key ID 和 Secret Access Key

**配置自定义域名：**
1. 进入存储桶 `presto-cdn` → **设置**
2. **自定义域名** → **连接域** → 输入：`cdn.presto.c-1o.top`
3. **公开访问** → **允许访问**

### 2️⃣ GitHub 配置（5 分钟）

在 `Presto-io/registry-deploy` 仓库：

**Settings → Secrets and variables → Actions → New repository secret：**

| Secret 名称 | 值 | 从哪里获取 |
|------------|---|-----------|
| `R2_ACCESS_KEY_ID` | Access Key ID | 第 1 步创建的 API Token |
| `R2_SECRET_ACCESS_KEY` | Secret Access Key | 第 1 步创建的 API Token |
| `R2_ACCOUNT_ID` | Account ID | R2 页面 |

**可选（使用默认值可以跳过）：**

| Secret 名称 | 默认值 |
|------------|-------|
| `R2_BUCKET` | `presto-cdn` |
| `CDN_DOMAIN` | `cdn.presto.c-1o.top` |

### 3️⃣ 首次同步（2 分钟）

1. 进入 `Presto-io/registry-deploy` 仓库
2. **Actions** → **Sync Binaries to R2 CDN**
3. **Run workflow** → 输入版本：`v1.0.0` → **Run workflow**
4. 等待完成（约 3-5 分钟）

## ✅ 验证

```bash
# 测试 CDN URL
curl -I https://cdn.presto.c-1o.top/templates/binaries/v1.0.0/gongwen/darwin-arm64

# 检查 registry.json
curl -s https://presto.c-1o.top/templates/registry.json | jq '.templates[0].platforms["darwin-arm64"]'
```

## 🚀 自动化流程

配置完成后，**无需任何手动操作**：

```
发布 Release (presto-official-templates)
    ↓ 自动触发
下载二进制 → 上传到 R2 → 更新 registry.json → 验证 CDN
    ↓
完成！CDN 自动同步
```

下次在 `presto-official-templates` 发布新版本时，CDN 会自动同步！

## 📊 成本

- **前 10GB 存储**：免费
- **前 100 万次操作**：免费
- **所有出站流量**：**完全免费**（无限）

每个版本约 100MB，可以免费存储 10+ 个版本。

## 📚 详细文档

完整配置指南请查看：[R2-CDN-SETUP.md](./R2-CDN-SETUP.md)

## ❓ 问题排查

**CDN URL 返回 404？**
1. 确认存储桶已开启公共访问
2. 确认自定义域名已配置
3. 检查 DNS：`dig cdn.presto.c-1o.top`

**上传失败？**
1. 检查 API Token 权限
2. 确认 Secrets 配置正确

**registry.json 未更新？**
1. 检查 GitHub Actions 日志
2. 确认 workflow 有写入权限

---

**需要帮助？** 查看 [完整配置指南](./R2-CDN-SETUP.md) 或联系维护者。
