# Cloudflare Pages 自动部署指南

## 🚀 快速设置（5 分钟）

### 步骤 1：登录 Cloudflare

1. 访问：https://dash.cloudflare.com/
2. 登录您的账号
3. 进入 **Pages** 页面

### 步骤 2：连接 GitHub 仓库

1. 点击 **"创建应用程序"** 或 **"Create application"**
2. 选择 **"Connect to Git"**
3. 点击 **"Connect to GitHub"** 按钮
4. 授权 Cloudflare 访问您的 GitHub 账号

### 步骤 3：选择仓库

1. 在仓库列表中找到：**ransomzhang/daily-news-web**
2. 点击 **"Begin setup"**

### 步骤 4：配置构建设置

在 **Build settings** 中填写：

| 设置项 | 值 |
|--------|-----|
| **框架预设** | None |
| **构建命令** | `python3 build.py` |
| **构建输出目录** | `dist` |
| **根目录** | `/`（留空或 `/`） |

### 步骤 5：环境变量（可选）

如果需要 Python 3，可以添加：

| 变量名 | 值 |
|--------|-----|
| `PYTHON_VERSION` | `3.14` |

### 步骤 6：部署

1. 点击 **"Save and Deploy"**
2. 等待首次部署完成（约 1-2 分钟）
3. 部署成功后，Cloudflare 会提供一个 URL：`https://your-project.pages.dev`

---

## ✅ 自动部署已设置

现在每次您推送代码到 `main` 分支时，Cloudflare 会自动：

1. 检测到新的提交
2. 运行 `python3 build.py`
3. 将 `dist/` 目录部署到 CDN
4. 更新网站内容

---

## 🔄 日常更新流程

以后更新日报时，只需：

```bash
cd /Users/zhangransom/daily-news

# 1. 生成新日报
python3 scripts/generate_report.py \
  --db data/news.db \
  --output output/$(date +%Y-%m-%d).md \
  --date $(date +%Y-%m-%d)

# 2. 构建网站
cd website
python3 build.py

# 3. 推送到 GitHub（自动触发 Cloudflare 部署）
git add -A
git commit -m "Add daily report for $(date +%Y-%m-%d)"
git push origin main
```

推送后 1-2 分钟，Cloudflare 会自动部署新版本！

---

## 🌐 自定义域名（可选）

### 步骤 1：添加域名

1. 在 Cloudflare Pages 项目中
2. 进入 **Custom domains**
3. 点击 **"Set up a custom domain"**
4. 输入您的域名（如 `news.yourdomain.com`）

### 步骤 2：配置 DNS

Cloudflare 会自动添加 DNS 记录，或您可以手动添加：

| 类型 | 名称 | 内容 |
|------|------|------|
| CNAME | news | your-project.pages.dev |

---

## 📊 监控部署

在 Cloudflare Pages 控制台，您可以：

- 查看部署历史
- 查看构建日志
- 回滚到之前的版本
- 预览部署（Pull Requests）

---

## 🐛 故障排查

### 问题 1：构建失败

**可能原因**：Python 未安装或版本不兼容

**解决方案**：
- Cloudflare Pages 默认支持 Python 3
- 如果需要特定版本，在环境变量中设置 `PYTHON_VERSION`

### 问题 2：找不到文件

**可能原因**：构建输出目录配置错误

**解决方案**：
- 确认 **构建输出目录** 为 `dist`（小写）
- 确认 `build.py` 脚本输出到 `dist/` 目录

### 问题 3：链接无法跳转

**可能原因**：使用了相对路径

**解决方案**：
- `build.py` 已经生成正确的绝对路径
- 确认 `dist/` 目录中的 HTML 文件链接格式为 `https://x.com/...`

---

## 📝 部署检查清单

- [ ] GitHub 仓库已创建
- [ ] `build.py` 可正常运行
- [ ] Cloudflare Pages 已连接 GitHub
- [ ] 构建命令：`python3 build.py`
- [ ] 输出目录：`dist`
- [ ] 首次部署成功
- [ ] 自定义域名已配置（可选）

---

## 🎉 完成！

您的日报网站现在会自动部署到：
**https://your-project.pages.dev**

每次推送代码后，网站会在 1-2 分钟内自动更新！
