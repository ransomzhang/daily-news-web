# Cloudflare Pages 部署修复指南

## 问题诊断

**错误信息：** `✘ [ERROR] Must specify a project name.`

**根本原因：**
Cloudflare Pages 的 Deploy command 配置了手动 wrangler 命令，但缺少项目名称参数。实际上，Cloudflare Pages 的 GitHub 集成会自动部署，不需要手动命令。

---

## 修复步骤

### 正确的 Deploy Command 配置 ✅

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **Pages** → **daily-news-web** 项目
3. 点击 **Settings** → **Build & deployments**
4. 找到 **Build configurations** 部分
5. 修改 Deploy command 为：

```bash
npx wrangler pages deploy dist --project-name=daily-news-web
```

| 设置 | 当前值 | 正确值 |
|------|--------|--------|
| **Build command** | `python3 build.py` | `python3 build.py` ✅ |
| **Deploy command** | `npx wrangler pages deploy dist` | `npx wrangler pages deploy dist --project-name=daily-news-web` ⚠️ |
| **Root directory** | `/` | `/` ✅ |

6. 点击 **Save** 保存更改

**注意：** 项目已包含 `wrangler.toml` 配置文件，wrangler 会自动读取项目名称。

### 验证修复

推送新 commit 或在 Cloudflare Dashboard 点击 **Retry deployment**，应该会成功。

---

### 方案 2：只使用 GitHub Actions（备选）

如果你更喜欢 GitHub Actions 控制部署：

1. 在 Cloudflare Pages Dashboard 中**断开 GitHub 仓库连接**
2. GitHub Actions workflow 已正确配置：
   ```yaml
   projectName: daily-news-web
   directory: dist
   ```

**推荐使用方案 1**，因为：
- ✅ 更简单，Cloudflare 自动处理
- ✅ 无需管理 API token
- ✅ 部署速度更快

---

## 工作原理

### Cloudflare Pages 部署流程

```
GitHub Push → Cloudflare 检测变化 → 运行 Build command → 运行 Deploy command → 部署完成
```

**步骤说明：**
1. **Build command:** `python3 build.py` - 构建网站，输出到 `dist/` 目录
2. **Deploy command:** `npx wrangler pages deploy dist --project-name=daily-news-web` - 部署到 Cloudflare Pages

**wrangler.toml 的作用：**
- 提供项目配置（名称、输出目录等）
- 作为 Deploy command 的备用配置来源
- 确保命令行参数和配置文件一致

---

## 检查清单

修复完成后，验证以下内容：

- [x] `wrangler.toml` 配置文件已创建（包含项目名称）
- [ ] Cloudflare Pages Deploy command 更新为：`npx wrangler pages deploy dist --project-name=daily-news-web`
- [ ] Build command 为 `python3 build.py`
- [ ] Root directory 为 `/`
- [ ] 推送新 commit 或重试部署
- [ ] 部署成功，网站可访问

---

## 常见问题

### Q: 为什么要添加 `--project-name` 参数？

A: Cloudflare Pages 构建环境中，wrangler 无法自动推断项目名称，必须显式指定。

### Q: wrangler.toml 是必需的吗？

A: 不是，但强烈推荐。它提供了明确的配置，避免命令行参数遗漏。

### Q: 可以同时使用 Cloudflare Pages 集成和 GitHub Actions 吗？

A: 可以，但会造成重复部署。建议：
- **推荐：** 只用 Cloudflare Pages 集成（本方案）
- **备选：** 禁用 Cloudflare Pages 集成，只用 GitHub Actions

### Q: 如何确认部署成功？

A: 在 Cloudflare Pages 项目页面查看 **Deployments** 标签，应该看到绿色的 "Success" 状态。

### Q: Deploy command 是必填项吗？

A: 是的，Cloudflare Pages 要求填写 Deploy command。正确的格式是：
```bash
npx wrangler pages deploy dist --project-name=daily-news-web
```
