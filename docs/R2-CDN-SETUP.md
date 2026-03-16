# Cloudflare R2 CDN 自动化配置指南

本指南帮助你配置完全自动化的 Cloudflare R2 CDN 系统。

## 架构概览

```
presto-official-templates (发布 Release)
    ↓ 触发器
registry-deploy (GitHub Actions)
    ↓ 下载二进制
GitHub Releases
    ↓ 上传到 R2
Cloudflare R2
    ↓ 自定义域名
cdn.presto.c-1o.top (CDN)
    ↓ 自动更新
registry.json (cdn_url 字段)
```

## 第一步：创建 Cloudflare R2 Bucket

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **R2 对象存储**
3. 点击 **创建存储桶**
4. 存储桶名称：`presto-cdn`（或你喜欢的名称）
5. 选择位置：自动（或选择离用户最近的区域）

## 第二步：创建 R2 API Token

1. 在 R2 页面，点击右上角 **管理 R2 API 令牌**
2. 点击 **创建 API 令牌**
3. 配置：
   - **令牌名称**：`presto-cdn-sync`
   - **权限**：对象读和写
   - **指定存储桶**：应用到特定存储桶 → 选择 `presto-cdn`
   - **TTL**：永久
4. 点击 **创建 API 令牌**
5. **重要**：复制并保存以下信息：
   - Access Key ID
   - Secret Access Key
   - Account ID（在 R2 页面可以找到）

## 第三步：配置自定义域名

### 3.1 添加域名到 Cloudflare

1. 如果 `c-1o.top` 还没有添加到 Cloudflare，请先添加
2. 确保域名使用 Cloudflare 的 nameservers

### 3.2 配置 R2 自定义域名

1. 进入 R2 存储桶 `presto-cdn`
2. 点击 **设置** 标签
3. 滚动到 **自定义域名** 部分
4. 点击 **连接域**
5. 输入：`cdn.presto.c-1o.top`
6. Cloudflare 会自动配置 DNS 和 SSL

### 3.3 配置公共访问

1. 在存储桶设置中，找到 **公开访问** 部分
2. 点击 **允许访问**
3. 确认操作

## 第四步：配置 GitHub Secrets

在 `registry-deploy` 仓库中添加以下 Secrets：

1. 进入 `Presto-io/registry-deploy` 仓库
2. 点击 **Settings** → **Secrets and variables** → **Actions**
3. 点击 **New repository secret**，添加以下 secrets：

### 必需的 Secrets

| Secret 名称 | 值 | 说明 |
|------------|---|------|
| `R2_ACCESS_KEY_ID` | 你的 Access Key ID | 第二步创建的 |
| `R2_SECRET_ACCESS_KEY` | 你的 Secret Access Key | 第二步创建的 |
| `R2_ACCOUNT_ID` | 你的 Cloudflare Account ID | 第二步记录的 |

### 可选的 Secrets（有默认值）

| Secret 名称 | 默认值 | 说明 |
|------------|-------|------|
| `R2_BUCKET` | `presto-cdn` | R2 存储桶名称 |
| `CDN_DOMAIN` | `cdn.presto.c-1o.top` | CDN 自定义域名 |
| `SOURCE_REPO` | `Presto-io/presto-official-templates` | 源仓库 |

## 第五步：手动触发首次同步

1. 进入 `Presto-io/registry-deploy` 仓库
2. 点击 **Actions** 标签
3. 选择 **Sync Binaries to R2 CDN** workflow
4. 点击 **Run workflow**
5. 输入版本号：`v1.0.0`
6. 点击 **Run workflow**

## 第六步：验证部署

等待 workflow 完成后，验证：

### 6.1 检查二进制文件

```bash
# 测试 CDN URL
curl -I https://cdn.presto.c-1o.top/templates/binaries/v1.0.0/gongwen/darwin-arm64

# 应该返回 HTTP 200
```

### 6.2 检查 registry.json

```bash
# 查看 registry.json 中的 cdn_url 字段
curl -s https://presto.c-1o.top/templates/registry.json | jq '.templates[0].platforms["darwin-arm64"]'

# 应该包含 cdn_url 字段
```

## 自动化流程说明

配置完成后，整个流程完全自动化：

1. **发布新版本**：在 `presto-official-templates` 仓库发布 Release
2. **自动触发**：`trigger-cdn-sync.yml` 自动触发 `registry-deploy` 的 workflow
3. **下载二进制**：从 GitHub Releases 下载所有平台的二进制文件
4. **上传到 R2**：使用 rclone 上传到 Cloudflare R2
5. **更新 registry**：自动更新 `registry.json` 中的 `cdn_url` 字段
6. **验证 CDN**：自动验证所有 CDN URL 可访问
7. **提交变更**：自动提交到 `registry-deploy` 仓库

## 文件路径说明

R2 中的文件路径结构：

```
presto-cdn/
  templates/
    binaries/
      v1.0.0/
        gongwen/
          darwin-arm64
          darwin-amd64
          linux-arm64
          linux-amd64
          windows-arm64
          windows-amd64
        jiaoan-shicao/
          darwin-arm64
          ...
```

对应的 CDN URL：

```
https://cdn.presto.c-1o.top/templates/binaries/v1.0.0/gongwen/darwin-arm64
https://cdn.presto.c-1o.top/templates/binaries/v1.0.0/gongwen/darwin-amd64
...
```

## 缓存配置

Cloudflare R2 自带 CDN 缓存。如需自定义缓存策略：

1. 进入 Cloudflare Dashboard
2. 选择域名 `c-1o.top`
3. 进入 **规则** → **页面规则**
4. 创建规则：
   - URL 模式：`cdn.presto.c-1o.top/templates/binaries/*`
   - 缓存级别：缓存所有内容
   - 边缘缓存 TTL：1 个月
   - 浏览器缓存 TTL：4 小时

## 成本估算

Cloudflare R2 定价（2024）：
- **存储**：$0.015/GB/月（前 10GB 免费）
- **Class A 操作**：$4.50/百万次（前 100 万次免费）
- **Class B 操作**：$0.36/百万次（前 1000 万次免费）
- **出站流量**：**免费**（这是最大的优势！）

预估成本（每个版本约 100MB）：
- 存储：前 10 个版本免费
- 操作：几乎免费
- 流量：完全免费（无限）

相比 AWS S3 + CloudFront，Cloudflare R2 能节省 90% 以上的流量成本。

## 故障排查

### 问题 1：上传失败

**错误**：`Access Denied`

**解决**：
1. 检查 R2 API Token 权限
2. 确认 Access Key ID 和 Secret Access Key 正确
3. 确认存储桶名称正确

### 问题 2：CDN URL 无法访问

**错误**：404 Not Found

**解决**：
1. 确认存储桶已开启公共访问
2. 确认自定义域名已正确配置
3. 检查 DNS 解析：`dig cdn.presto.c-1o.top`

### 问题 3：registry.json 未更新

**错误**：cdn_url 字段缺失

**解决**：
1. 检查 GitHub Actions 日志
2. 确认 workflow 有写入权限（Settings → Actions → General → Workflow permissions）
3. 手动重新运行 workflow

## 进阶配置

### 多环境支持

可以创建不同的存储桶用于不同环境：

- `presto-cdn-dev`：开发环境
- `presto-cdn-staging`：测试环境
- `presto-cdn`：生产环境

### 监控和告警

1. 在 Cloudflare Dashboard 中启用 **R2 事件通知**
2. 配置 webhook 发送到你的监控系统
3. 监控指标：
   - 存储空间使用
   - 请求次数
   - 错误率

### 版本回滚

如果新版本有问题：

1. 进入 `registry-deploy` 仓库
2. 找到上一个版本的 commit
3. Revert 该 commit
4. 或手动修改 `registry.json` 指向旧版本路径

## 下一步

1. 完成 Cloudflare R2 配置
2. 配置 GitHub Secrets
3. 手动触发首次同步
4. 验证 CDN URL 可访问
5. 发布新版本测试自动化流程

如有问题，请查看 GitHub Actions 日志或联系维护者。
