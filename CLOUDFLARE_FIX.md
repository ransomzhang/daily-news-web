# Cloudflare Pages 部署修复指南

## 问题诊断

**错误信息 1：** `✘ [ERROR] Must specify a project name.`

**错误信息 2：** `✘ [ERROR] Authentication error [code: 10000]`

**根本原因：**
Cloudflare Pages 的 GitHub 集成会**自动部署**构建输出，不需要（也不应该）手动运行 `wrangler pages deploy` 命令。配置 Deploy command 为 wrangler 命令会造成：
1. **重复部署** - Cloudflare 已经自动部署，无需手动命令
2. **认证问题** - 构建环境的 API token 可能权限不足

**关键理解：**
- Cloudflare Pages GitHub 集成 = Build command + **自动部署**
- Deploy command 是为**自定义部署逻辑**设计的（如部署到其他平台）

---

## 修复步骤

### 方案 A：使用 no-op Deploy Command（推荐）✅

**适用场景：** 保留 Cloudflare Pages GitHub 集成的所有功能（自动预览、回滚等）

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **Pages** → **daily-news-web** 项目
3. 点击 **Settings** → **Build & deployments**
4. 找到 **Build configurations** 部分
5. 修改 Deploy command 为：

```bash
echo "Build output ready for Cloudflare Pages auto-deployment"
```

| 设置 | 当前值 | 正确值 |
|------|--------|--------|
| **Build command** | `python3 build.py` | `python3 build.py` ✅ |
| **Deploy command** | `npx wrangler pages deploy dist --project-name=daily-news-web` | `echo "Build output ready..."` ⚠️ |
| **Root directory** | `/` | `/` ✅ |

6. 点击 **Save** 保存更改

**为什么这样做？**
- ✅ 满足 Deploy command 必填要求
- ✅ 不干扰 Cloudflare Pages 的自动部署机制
- ✅ 避免认证问题和重复部署
- ✅ 保留所有 Cloudflare Pages 功能

---

### 方案 B：完全使用 GitHub Actions（备选）

**适用场景：** 想要完全控制部署流程，不需要 Cloudflare 自动集成

1. **禁用 Cloudflare Pages GitHub 集成：**
   - 在 Cloudflare Dashboard 进入 **Pages** → **daily-news-web**
   - 点击 **Settings** → **Build & deployments**
   - 找到 **Source** 部分
   - 点击 **Disconnect** 断开 GitHub 连接

2. **配置 GitHub Secrets：**
   - 进入 GitHub 仓库 **Settings** → **Secrets and variables** → **Actions**
   - 添加以下 secrets：
     - `CLOUDFLARE_API_TOKEN`: 你的 Cloudflare API Token
     - `CLOUDFLARE_ACCOUNT_ID`: 你的账户 ID (`c3fe4a9370cd97253c1195509cf42144`)

3. **确保 GitHub Actions workflow 已启用：**
   - `.github/workflows/deploy.yml` 已配置（✅ 已存在）

**推荐使用方案 A**，因为：
- ✅ 更简单，利用 Cloudflare Pages 原生集成
- ✅ 自动预览部署（pull requests）
- ✅ 更快的部署速度
- ✅ 无需管理 API token

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

### Cloudflare Pages GitHub 集成（方案 A）

```
GitHub Push → Cloudflare Webhook → 运行 Build command → 自动部署 dist → 完成
```

**工作流程：**
1. **Build command:** `python3 build.py` - 构建网站，输出到 `dist/` 目录
2. **自动部署:** Cloudflare 检测到 `dist/` 目录，**自动**部署到 Pages
3. **Deploy command:** 运行 no-op 命令（echo），不影响自动部署

**关键点：**
- Cloudflare Pages GitHub 集成**包含自动部署功能**
- Deploy command 只是额外步骤，不是必需的
- 使用 wrangler 命令会导致重复部署和认证问题

### GitHub Actions 部署（方案 B）

```
GitHub Push → GitHub Actions → Build → cloudflare/pages-deploy action → 部署完成
```

**工作流程：**
1. GitHub Actions workflow 触发
2. 运行 Build command
3. 使用 `cloudflare/pages-deploy` action 部署
4. 需要有效的 `CLOUDFLARE_API_TOKEN`

---

## 检查清单

### 方案 A（no-op 命令）检查清单：

- [ ] Cloudflare Pages Deploy command 更新为：`echo "Build output ready..."`
- [ ] Build command 保持为：`python3 build.py`
- [ ] Root directory 为：`/`
- [ ] 保存设置后，重试部署
- [ ] 部署成功，网站可访问

### 方案 B（GitHub Actions）检查清单：

- [ ] 在 Cloudflare Dashboard 断开 GitHub 连接
- [ ] 在 GitHub Repository Settings 添加 `CLOUDFLARE_API_TOKEN` secret
- [ ] 在 GitHub Repository Settings 添加 `CLOUDFLARE_ACCOUNT_ID` secret
- [ ] 推送新 commit 触发 GitHub Actions
- [ ] 在 Actions 标签页确认 workflow 成功
- [ ] 网站可访问

---

## 常见问题

### Q: 为什么不能使用 `wrangler pages deploy` 命令？

A: Cloudflare Pages GitHub 集成**已经包含自动部署**功能。使用 wrangler 命令会：
- 造成**重复部署**（Cloudflare 自动 + wrangler 手动）
- 引入**认证问题**（构建环境的 token 权限可能不足）
- 失去 Cloudflare Pages 的优势（预览部署、自动回滚等）

### Q: Deploy command 是必填项，该怎么办？

A: 使用 **no-op 命令**（方案 A）：
```bash
echo "Build output ready for Cloudflare Pages auto-deployment"
```
这个命令不会影响 Cloudflare 的自动部署。

### Q: 什么时候应该使用 GitHub Actions 部署（方案 B）？

A: 当你需要：
- 完全控制部署流程
- 在部署前执行额外的自定义步骤
- 不需要 Cloudflare Pages 的自动集成功能

### Q: 两种方案可以同时使用吗？

A: **不建议！** 会造成重复部署和冲突。选择一种方案即可。

### Q: 如何确认 API token 权限？

A: 访问 https://dash.cloudflare.com/profile/api-tokens 查看权限。
如果使用方案 A（no-op），不需要额外的 API token。
如果使用方案 B（GitHub Actions），token 需要包含 "Cloudflare Pages" 编辑权限。

### Q: wrangler.toml 文件还需要吗？

A: 使用方案 A（no-op）时**不需要**，可以删除。
使用方案 B（GitHub Actions）时也不需要，因为 workflow 已明确指定项目名称。
