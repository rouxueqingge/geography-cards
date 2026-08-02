# 地理卡片库 · 部署到 Cloudflare Pages（连 GitHub 自动部署）

目标：把 GitHub 仓库 `rouxueqingge/geography-cards`（Quartz 源码）接到 Cloudflare Pages，
以后 **push 一次就自动构建 + 发布**，不用本地重 build，也不用管服务器。

> 前提：你已注册 Cloudflare 账号（邮箱即可，免费）。
> 这个站是**静态站**，Cloudflare Pages 免费、不限流量、自带全球 CDN + HTTPS。

---

## 一、在 Cloudflare 创建 Pages 项目并连 GitHub

1. 登录 https://dash.cloudflare.com → 左侧 **Workers 和 Pages**（或 Workers & Pages）。
2. 点 **创建**（Create）→ **Pages** → 选 **连接到 Git（Connect to Git）**。
3. 首次会跳到 GitHub 授权页 → 点 **Authorize Cloudflare**（允许读取你的仓库）。
4. 在仓库列表里选 **`rouxueqingge/geography-cards`** → 点 **开始设置（Begin setup）**。

---

## 二、填构建参数（最关键，照抄）

在「构建设置（Build settings）」里填：

| 项目 | 填什么 |
|------|--------|
| **项目名称（Project name）** | `geo-cards`（随意，会生成 `geo-cards.pages.dev`） |
| **生产分支（Production branch）** | `main` |
| **构建命令（Build command）** | `npx quartz build` |
| **构建输出目录（Build output directory）** | `public` |
| **Node.js 版本** | 选 **22**（或 20+；仓库要求 ≥22） |

> ⚠️ Node 版本千万别用默认的 18，否则 Quartz 构建会报错。
> 若面板里没有 Node 版本选项，就在仓库根目录加一个 `.nvmrc` 文件，内容只写一行：`22`
> （加了 `.nvmrc` 后需 `git push` 一次，Cloudflare 会自动读它）。

5. 点 **保存并部署（Save and Deploy）**。
6. 等 1–3 分钟（首次要 `npm install` 装依赖），看到 ✅ **Success** 即上线。
7. 访问 **`https://geo-cards.pages.dev`**（项目名即子域名前缀）。

---

## 三、（可选）绑定自己的域名

Cloudflare Pages 默认给 `*.pages.dev` 域名，国内能访问但偶尔不稳。想更稳可绑自定义域名：

1. Pages 项目里点 **自定义域（Custom domains）** → 输入你的域名（如 `cards.你的域名.com`）。
2. Cloudflare 会生成一条 **CNAME** 记录，去你的域名 DNS 后台添加即可（若域名本身在 Cloudflare，自动搞定）。
3. 证书自动签发，不用管。

> 注意：国内绑定自定义域名**需要域名已备案**（ICP 备案）。未备案域名在国内会被拦截。
> 如果只是自己/学生用，`*.pages.dev` 默认域名够用，无需备案。

---

## 四、以后怎么更新内容

1. 改 `content/` 下卡片 → `git push` 到 GitHub。
2. **两个站同时更新**（互不干扰）：
   - GitHub Actions 自动重建 → 国外站 `rouxueqingge.github.io/geography-cards/`
   - Cloudflare 检测到 push → 自动构建 → `geo-cards.pages.dev`
3. 都不用本地 build（C 盘满、依赖不全也无所谓，云端构建）。

---

## 五、注意事项

- **与 GitHub Pages 共存无冲突**：Cloudflare 用自己的构建流水线，不影响 GitHub Actions。
- **`baseUrl` 问题（可选修正）**：`quartz.config.yaml` 里的 `baseUrl` 现为
  `rouxueqingge.github.io/geography-cards`，只影响 Cloudflare 站点的 `<head>` 元信息
  （og:url / canonical / sitemap 域名）。**不影响正常访问**。若想让 Cloudflare 站的 SEO 元信息准确，
  可后续做成「按环境变量切换 baseUrl」，或接受现状即可。
- **Cloudflare 免费版无中国大陆节点**：国内流量走香港/日本/新加坡节点，速度"还行"但非秒开，
  比 CloudStudio 北京节点慢、比 GitHub Pages 快。它是为**稳定 + 可绑域名 + 全自动**而非"最快"。
- **Gitee 仓库 `geofans/geo-cards`** 仅作代码备份（Gitee Pages 已停服），与此部署无关。

---

## 六、排错

| 现象 | 原因 / 解决 |
|------|------------|
| 构建报 `node version` 相关错 | Node 版本选成 18 了 → 改 22 或加 `.nvmrc` |
| 构建报依赖装不上 | 重试；Cloudflare 偶发网络问题，重跑部署即可 |
| 页面能开但样式乱 | 输出目录填错（必须 `public`）→ 改设置重部署 |
| `*.pages.dev` 国内打不开 | 换 CloudStudio 北京节点，或绑已备案自定义域名 |
