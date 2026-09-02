---
title: 使用 GitHub Pages 与国内平台部署静态网站
date: 2026-07-24 15:30:00
categories:
  - 教程
tags:
  - Hexo
  - GitHub Pages
  - Cloudflare Pages
  - 部署
---

静态博客（如 Hexo、Hugo、VuePress 等）生成的是纯 HTML/CSS/JS 文件，非常适合托管在免费的静态页面服务上。国内开发者过去常把 **Gitee Pages** 作为首选，但**该服务已于 2022 年起逐步下线，目前已不可用**。本文以 Hexo 为例，介绍仍可用的 GitHub Pages，以及适合国内访问的替代部署方案。

## 一、可用的静态托管平台对比

| 平台 | 国内访问 | 自动构建 | 自定义域名 / HTTPS | 费用 | 备注 |
| --- | --- | --- | --- | --- | --- |
| GitHub Pages | 较慢，可能被墙 | 支持（Actions） | 支持 | 免费 | 海外友好，最省心 |
| Cloudflare Pages | 较快（有国内边缘节点） | 支持（连接 Git） | 支持 | 免费 | 国内访问相对好，推荐 |
| Vercel | 一般 | 支持 | 支持 | 免费额度 | 需绑定域名走加速 |
| Netlify | 一般 | 支持 | 支持 | 免费额度 | 与 Vercel 类似 |
| 腾讯云静态网站托管 / 阿里云 OSS | 快 | 需配合 CI | 支持 | 付费 | 可加 CDN，企业级稳定 |

> 结论：读者在海外或想零运维，选 **GitHub Pages**；读者主要在国内，选 **Cloudflare Pages** 或云厂商对象存储 + CDN；追求双保险可 GitHub 存源码、CI 同时部署到多个平台。

## 二、方式一：部署到 GitHub Pages

### 1. 创建仓库

- 用户页：`你的用户名.github.io`（仓库名必须严格一致）
- 项目页：任意仓库名，访问地址为 `用户名.github.io/仓库名`

### 2. 安装部署插件

```bash
npm install hexo-deployer-git --save
```

### 3. 配置 `_config.yml`

```yaml
deploy:
  type: git
  repo: https://github.com/用户名/用户名.github.io.git
  branch: main        # 用户页用 main；项目页按实际情况
  # message: 自动提交信息（可选）
```

### 4. 部署

```bash
hexo clean && hexo deploy
```

首次推送需已配置 GitHub 的 SSH key 或账号凭据。部署完成后等待 1~2 分钟，访问 `https://用户名.github.io` 即可。

### 5. （推荐）用 GitHub Actions 自动部署

将源码推送到 GitHub，由 Actions 自动构建发布，无需本地 `hexo deploy`：

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npx hexo clean && npx hexo generate
      - uses: actions/upload-pages-artifact@v3
        with:
          path: public
      - uses: actions/deploy-pages@v4
```

并在仓库 **Settings → Pages** 中选择 `GitHub Actions` 作为构建源。

## 三、方式二：国内友好的替代平台

Gitee Pages 已下线，若你需要**国内访问顺畅且免费**，推荐 **Cloudflare Pages**；若已是云厂商用户，可直接用对象存储 + CDN。

### 方案 A：Cloudflare Pages（推荐）

1. 注册 Cloudflare 账号，进入 **Workers & Pages → Create → Pages → 连接 Git 仓库**。
2. 构建配置：
   - 构建命令：`npx hexo clean && npx hexo generate`
   - 输出目录：`public`
3. 保存后每次 push 自动构建，分配 `*.pages.dev` 域名；可在 **Custom domains** 绑定自己的域名并自动签发 HTTPS。

> Cloudflare 在国内有边缘节点，访问速度通常优于 GitHub Pages。

### 方案 B：云厂商对象存储 + CDN（企业/高稳定需求）

以腾讯云或阿里云为例：

1. 创建对象存储桶，开启「静态网站托管」。
2. 通过 CI（如 GitHub Actions）把 `public/` 同步到存储桶：
   - 腾讯云：`coscli` / `coscmd` 上传
   - 阿里云：`ossutil` 上传
3. 在存储桶前挂载 CDN 加速域名并配置 HTTPS 证书。

```yaml
# GitHub Actions 片段（以腾讯云 COS 为例，需配置 SecretId/SecretKey）
- name: 上传到 COS
  run: |
    npx hexo clean && npx hexo generate
    coscli sync ./public/ cos://你的桶名/ -e ap-guangzhou.myqcloud.com
```

## 四、双线 / 多平台部署建议

若希望海外走 GitHub Pages、国内走 Cloudflare Pages 或 CDN：

1. 源码统一托管在 GitHub；
2. 通过一份 Actions 工作流，把构建产物分别发布到 GitHub Pages 与 Cloudflare Pages（或同步到云存储）；
3. 利用 DNS 解析（如 `CNAME` + 国内/海外分流，或 Cloudflare 的地理路由）将不同访客导向对应站点。

这样既能享受 GitHub 的自动构建，又能保证国内访问速度。

## 五、常见问题

- **GitHub Pages 404**：确认仓库名、分支与部署源设置一致；首次启用需等待几分钟。
- **Gitee Pages 无法访问 / 提示不存在**：Gitee Pages 服务已下线，请改用本文「方式二」中的替代平台。
- **Cloudflare Pages 构建失败**：检查构建命令与输出目录是否填对（Hexo 输出为 `public`），以及 `package.json` 中 `npm ci` 能正常安装依赖。
- **推送被拒绝**：确认部署分支与各平台要求一致（GitHub Pages 用 `main`，云存储无需分支概念）。
- **私人令牌泄露风险**：务必使用 CI Secrets 注入，不要写进仓库代码或配置文件中。

---

无论选择哪种方式，核心流程都是：**写文章 → 生成静态文件 → 推送到页面服务的部署分支 / 存储桶**。理解这一点后，切换平台只是改一下目标地址而已。
