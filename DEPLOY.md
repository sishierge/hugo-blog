# 部署到 GitHub + Cloudflare Pages

本目录 `hugo-blog` 若自带 `.git`，表示它**可以单独作为一个 GitHub 仓库**；若你希望整站放在更大的「博客」父仓库里，请先**删除** `hugo-blog/.git` 再把它当作普通子目录加入父仓库（二选一，勿嵌套）。

下面两种方式任选其一：**推荐 A（Cloudflare 直连 GitHub）**，免维护 GitHub Actions；需要自定义流水线时用 **B**。

---

## A. Cloudflare Pages 连接 GitHub（推荐）

1. **推送代码到 GitHub**  
   - **仅推送本博客**（`hugo-blog` 为仓库根目录时）：
     ```bash
     cd D:/博客/hugo-blog
     git remote add origin https://github.com/<你的用户名>/<仓库名>.git
     git branch -M main
     git push -u origin main
     ```
   - **若使用父目录 `博客` 作为仓库根**（且已去掉 `hugo-blog/.git`）：
     ```bash
     cd D:/博客
     git add hugo-blog .gitignore .github
     git commit -m "Add Hugo blog"
     git push -u origin main
     ```

2. **登录 Cloudflare** → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**。

3. **选择仓库**后配置构建：
   - **Root directory（根目录）**：若仓库根就是 `hugo-blog`，留空；若在父仓库 `博客` 里，填 **`hugo-blog`**
   - **Build command**（须拉取主题子模块；若 Hugo 非 extended 请改用下文 curl 示例）：  
     `git submodule update --init --recursive && hugo version && hugo --gc --minify --baseURL "$CF_PAGES_URL"`
   - **Build output directory**：`public`
   - **Environment variables**：
     - `HUGO_VERSION` = `0.147.0`（与本地 [hugo-bin](https://github.com/gohugoio/hugo/releases) 对齐；需 **Extended** 时选带 extended 的安装方式，或使用下方「自定义命令」）

   若 Cloudflare 默认安装的 Hugo 不含 Extended，可将 **Build command** 改为使用官方安装脚本（示例，版本可按需调整）：
   ```bash
   git submodule update --init --recursive && curl -sL https://github.com/gohugoio/hugo/releases/download/v0.147.0/hugo_extended_0.147.0_linux-amd64.tar.gz | tar -xz -C /tmp && /tmp/hugo --gc --minify --baseURL "$CF_PAGES_URL"
   ```
   （请将 `v0.147.0` 与文件名与 [Release](https://github.com/gohugoio/hugo/releases) 一致。）  
   主题 `themes/reimu` 为 **Git submodule**，构建前必须执行 `git submodule update --init --recursive`（使用已含 extended 的镜像时，也请在 `hugo` 命令前保留该子模块步骤）。

   若 **Root directory** 为 `hugo-blog`，构建命令开头同样要拉子模块，例如：
   ```bash
   git submodule update --init --recursive && hugo --gc --minify --baseURL "$CF_PAGES_URL"
   ```
   （前提是 Cloudflare 提供的 Hugo 为 **extended** 且版本满足主题要求。）

4. **保存并部署**。首次构建完成后会得到 `https://<项目名>.pages.dev`。

5. **自定义域名**（可选）：在该 Pages 项目 → **Custom domains** 里绑定你的域名，并在 DNS 按提示添加记录。

`CF_PAGES_URL` 由 Cloudflare 在构建时注入，用于 `--baseURL`，无需手写站点地址。

---

## B. 使用本仓库的 GitHub Actions 部署到 Cloudflare Pages

适用于希望用 Actions 统一构建、再调用 Cloudflare API 上传 `public/` 的场景。

### 1. 在 Cloudflare 准备

1. 打开 [Cloudflare Dashboard](https://dash.cloudflare.com/) → **Workers & Pages** → **Create** → **Pages**，可先创建一个 **Project name**（例如 `hugo-blog`），与下面变量一致。
2. 获取 **Account ID**：右侧边栏或 **Overview** 页面可见。
3. 创建 **API Token**：**My Profile** → **API Tokens** → **Create Token** → 使用模板 **Edit Cloudflare Workers** 或自定义权限，至少包含 **Account** → **Cloudflare Pages** → **Edit**。保存 Token 字符串（只显示一次）。

### 2. 在 GitHub 仓库配置 Secrets / Variables

**Settings** → **Secrets and variables** → **Actions**：

| Name | 说明 |
|------|------|
| `HUGO_BASE_URL` | 站点根 URL，如 `https://hugo-blog.pages.dev` 或你的自定义域名 `https://blog.example.com`（须与最终访问地址一致，含 `https://`） |
| `CLOUDFLARE_API_TOKEN` | 上一步的 API Token |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare Account ID |

**Variables**（可选）：

| Name | 说明 |
|------|------|
| `CLOUDFLARE_PAGES_PROJECT_NAME` | Pages 项目名称，默认 `hugo-blog` |

### 3. 推送触发

向 `main` 或 `master` 推送且变更在 `hugo-blog/` 或工作流文件时，会自动构建并部署。也可在 **Actions** 里手动 **Run workflow**。

---

## Giscus 主题 CSS（若使用 Reimu 生成的 Giscus 样式）

评论框 iframe 会请求你站点上的 `/css/giscus_reimu_*.css`。若线上发现主题不生效，需为这些静态资源允许 `https://giscus.app` 跨域。本仓库已在 `static/_headers` 中为 `/css/*` 添加示例规则；部署到 Cloudflare Pages 后会随站点生效，可按需收紧路径。

---

## 故障排查

- **构建失败：找不到 hugo extended**：使用 A 方案中的 `curl` 安装 extended 版本，或提高 `HUGO_VERSION` 并确认镜像含 extended。
- **样式/链接错乱**：检查 `--baseURL` 是否与浏览器地址一致（含 `https`、是否带尾部路径）。
- **Actions 报错缺少 `HUGO_BASE_URL`**：在 GitHub Secrets 中配置该项。
