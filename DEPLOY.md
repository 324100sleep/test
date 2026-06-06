# Deploy Guide - Open Issue 整理助手

## 项目结构

```
index.html              → GitHub Pages 前端（单页应用）
worker/
├── wrangler.toml       → Cloudflare Workers 配置
├── package.json        → 依赖配置
└── src/
    └── index.js        → Worker 核心逻辑
```

## 第一步：部署 Cloudflare Workers（后端）

### 准备工作
1. 注册/登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 获取你的 AI API Key（如 OpenAI / DeepSeek 等）

### 创建 KV Namespace
1. 进入 Cloudflare Dashboard → Workers & Pages → KV
2. 点击 "Create namespace"，命名为 `ISSUE_KV`
3. 复制 Namespace ID

### 配置 wrangler.toml
打开 `worker/wrangler.toml`，将 `YOUR_KV_NAMESPACE_ID` 和 `YOUR_KV_NAMESPACE_PREVIEW_ID` 替换为你的 KV ID。

### 部署 Worker
```bash
cd worker
npm install          # 安装依赖
npx wrangler deploy  # 部署到 Cloudflare
```

部署完成后，你会获得一个 Worker 地址，例如：
`https://issue-refiner.your-name.workers.dev`

### 配置 API Key
在 Cloudflare Dashboard 中：
1. 进入 Workers → issue-refiner → Settings → Variables
2. 添加环境变量 `AI_API_KEY`，值为你的 AI API Key（如 OpenAI sk-...）
3. 保存并重新部署

### （可选）配置 API 地址
默认使用 OpenAI API (`https://api.openai.com/v1`)。如需使用 DeepSeek 等其他服务：
- 在 Worker 环境变量中添加 `AI_API_BASE`，值为对应 API 地址
- 或者在前端设置中手动配置

## 第二步：部署 GitHub Pages（前端）

### 创建 GitHub 仓库
1. 在 GitHub 新建一个仓库（如 `open-issue-manager`）
2. 在本地项目目录执行：
```bash
git init
git add .
git commit -m "init: Open Issue 整理助手"
git remote add origin https://github.com/你的用户名/open-issue-manager.git
git push -u origin main
```

### 启用 GitHub Pages
1. 进入 GitHub 仓库 → Settings → Pages
2. Source 选择 `Deploy from a branch`
3. Branch 选择 `main`，目录选 `/ (root)`
4. 点击 Save
5. 等待几分钟，访问 `https://你的用户名.github.io/open-issue-manager/`

## 第三步：使用前配置

### 配置 Worker URL
1. 打开你的 GitHub Pages 页面
2. 点击右上角 ⚙️ 设置按钮
3. 在 "Worker 服务地址" 输入你部署的 Worker 地址（如 `https://issue-refiner.your-name.workers.dev`）
4. 可选择模型（默认 gpt-4o-mini）和 API 地址
5. 点击保存

### 分享给他人
- 将 GitHub Pages 链接分享给同事朋友
- 他们打开后直接使用，无需任何配置
- 所有 AI 费用由 Cloudflare Worker 中的 API Key 承担
- 每日 Token 上限 100 万，超限后自动停止

## 本地测试

直接用浏览器打开 `index.html` 即可测试界面（但 AI 提炼功能需配置 Worker URL）：

```bash
# 直接双击 index.html 或用 Live Server 打开
```

测试 Worker：
```bash
cd worker
npx wrangler dev
```

## 每日 Token 用量查看

在 Cloudflare Dashboard → Workers → issue-refiner → KV 中查看 `usage:YYYY-MM-DD` 的值。

## 注意事项

- API Key 仅存储在 Cloudflare Worker 环境变量中，前端永远无法读取
- 数据存储在浏览器 localStorage 中，清除浏览器缓存会丢失数据
- 建议定期导出 CSV 备份
- 如需更换 AI 模型，可在设置中修改
