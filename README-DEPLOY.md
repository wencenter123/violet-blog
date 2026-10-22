# 薇尔莉特的手记 · 部署说明

Firefly 6.16.8 / Astro 7.3.2 ｜ 构建产物 `dist/`

---

## 一、改内容

| 想改什么 | 文件 | 搜索关键字 |
|---|---|---|
| 站名、副标题、描述、网址 | `src/config/siteConfig.ts` | `title` / `subtitle` / `site_url` |
| 主题色（当前深蓝紫 265） | `src/config/siteConfig.ts` | `hue` |
| 头像、昵称、签名、社交图标 | `src/config/profileConfig.ts` | `avatar` / `name` / `bio` / `links` |
| 首页大标题、轮播文案、壁纸 | `src/config/backgroundWallpaper.ts` | `homeText` / `src` |
| 侧栏公告 | `src/config/announcementConfig.ts` | `content` |
| 页脚版权 | `src/config/FooterConfig.html` | 直接改 HTML |
| 导航菜单 | `src/config/navBarConfig.ts` | 按顺序 `links.push(...)` |

文章写在 `src/content/posts/`，动态在 `src/content/dynamic/`。

**重点**：`siteConfig.ts` 的 `site_url` 必须等于最终访问网址，当前填的是
`https://violet-blog.pages.dev`。Cloudflare 上的项目名如果不叫 `violet-blog`，
这里要跟着改，否则 RSS、sitemap 和分享卡片里的链接会错。

---

## 二、本地运行

```bash
cd "/f/DSH work/violet-blog"
export PATH="/f/temp/components/nodejs:$PATH"

# 开发（改文件自动刷新）
node node_modules/astro/bin/astro.mjs dev      # http://127.0.0.1:4321

# 生产构建
node node_modules/astro/bin/astro.mjs build    # 输出到 dist/
```

装依赖（只第一次需要）：

```bash
npx -y pnpm@11.22.0 install \
  --registry=https://repo.huaweicloud.com/repository/npm/ \
  --network-concurrency=64
```

> 用华为云镜像，npmjs 直连只有十几 KB/s。`pnpm build` 不接受 `--registry` 参数。

---

## 三、部署到 Cloudflare Pages

### 方案 A：接 GitHub 自动构建（视频里的做法）

1. GitHub 新建**空仓库**（不要勾 README / .gitignore / license）
2. 推送：

```bash
cd "/f/DSH work/violet-blog"
git remote add origin https://github.com/<用户名>/violet-blog.git
git push -u origin main
```

3. Cloudflare 控制台 → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
4. 选仓库，配置：

| 字段 | 值 |
|---|---|
| Project name | `violet-blog` |
| Production branch | `main` |
| Framework preset | `Astro` |
| Build command | `pnpm build` |
| Build output directory | `dist` |
| 环境变量 | `NODE_VERSION` = `22.23.0` |

5. Save and Deploy，首次构建 5–10 分钟

### 方案 B：直接上传 dist（最快）

Cloudflare → Pages → **Upload assets** → 项目名 `violet-blog` →
把 `dist/` **里面的内容**拖进去（不是 dist 这一层）→ Deploy。

---

## 四、访问问题

`*.pages.dev` 和 `*.workers.dev` 在国内多数被 DNS 污染
（实测 `violet.2516734546.workers.dev` 解析到 `108.160.170.45`，curl 超时）。

想稳定访问只能绑**自定义域名**：Pages 项目 → Custom domains → 添加，
域名 DNS 托管到 Cloudflare 即可，境外托管不需要备案。

---

## 五、已知事项

- **图片版权**：站内图片取自 B 站壁纸视频封面，版权归动画制作委员会与上传者。
  页脚与「关于」页已加来源声明，建议仅作个人非商业使用。
- **评论系统**：未配置。需要就改 `commentConfig.ts` 接 Waline / Twikoo。
- **搜索**：构建时 Pagefind 生成本地索引，无需外部服务。
- **友链 / 书签 / 项目页**：仍是主题示例数据，见 `friendsConfig.ts`、`booknavConfig.ts`，
  或在 `siteConfig.ts` 的 `pages` 里关掉。
