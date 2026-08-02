# L77Doncic.github.io — Tao Xu 个人主页项目

GitHub Pages 部署的个人学术主页（https://l77doncic.github.io/）。**仓库根目录是构建产物**（GitHub Pages 直接部署它），真正的源码在 `prism/` 子目录（Next.js 15 + Tailwind 4 + TypeScript，来自模板 [xyjoey/PRISM](https://github.com/xyjoey/PRISM)）。

## 项目结构

```
├── index.html, _next/, publications/, awards/, cv/, services/, 404/  ← 构建产物（勿手改！）
├── .nojekyll            ← 必须保留，防止 GitHub Pages 跑 Jekyll 破坏 _next 路径
├── prism/               ← Next.js 源码（本仓库唯一需要编辑的地方）
│   ├── content/         ← 英文内容（TOML / Markdown / BibTeX）
│   ├── content_zh/      ← 中文内容
│   ├── public/          ← 图片（bio.jpg、favicon、papers/ 论文预览图）
│   └── src/             ← 组件与逻辑（一般不用改）
└── CLAUDE.md
```

页面与内容文件的对应关系（`src/app/[slug]/page.tsx` 按 slug 读取 `content/<slug>.toml`）：

| 页面 | 英文内容 | 中文内容 |
|---|---|---|
| 首页 `/` | `content/about.toml` + `bio.md` + `news.toml` + `publications.bib`(selected) | `content_zh/` 对应文件 |
| `/publications` | `content/publications.toml` + `publications.bib` | 同上（bib 中英共用） |
| `/awards` | `content/awards.toml` | `content_zh/awards.toml` |
| `/services` | `content/services.toml` | `content_zh/services.toml` |
| `/cv` | `content/cv.toml` + `cv.md` | `content_zh/cv.toml` + `cv.md` |

## ⚠️ 铁律（血泪教训）

1. **绝对不要手改仓库根的构建产物**（`index.html`、`_next/`、各子页 HTML）。
   2026-08-02 曾因手改 `index.html` 里的 Next.js RSC payload（`self.__next_f.push` 数据）导致 flight 数据损坏，页面完全空白（hydration 失败后 React 用空树替换了 SSR 内容）。页面内容被包在 `<div style="visibility:hidden">` 里，只有 hydration 完成后才显示；失败即空白。
2. **改内容必须重新构建**：源码改了但 `git push` 不带新构建产物 = 线上无变化。构建产物只从 `prism/out/` 同步，不手工编辑。
3. **`.nojekyll` 不能删**。
4. **`prism/node_modules/`、`prism/out/`、`prism/.next/` 已被 .gitignore 忽略**，不要提交。

## 完整工作流（改内容 → 部署）

```bash
cd prism                      # 1. 修改内容（见下方"怎么改"）
npm run build                 # 2. 构建：静态导出到 prism/out/（要求 Node ≥ 22，见 .nvmrc）
rsync -a out/ ..              # 3. 同步构建产物到仓库根（保留 .nojekyll / LICENSE / README* / prism/）
cd ..
git add -A && git commit -m "..." && git push origin main   # 4. 推送即自动部署（约 1-2 分钟）
```

部署完成验证：`curl -sI https://l77doncic.github.io/ | grep last-modified`（时间戳更新即生效）。
本地预览验证：`python3 -m http.server 8899 --directory prism/out`，或跑 `npm run dev`。

## 怎么改内容（最常见任务）

### 1. 加/改论文 → `prism/content/publications.bib`

BibTeX 条目，中英文共用。字段说明：

```
@inproceedings{xu2026robust,          ← 引用键 = 页面上的 id
  selected = {true},                  ← true 才出现在首页 "Selected Publications"
  title = {...},
  author = {Tao Xu* and Ruiya Qi ...},← * = 通讯作者；# = 共同一作；
                                       ← 站点作者（config.toml [author] name）自动高亮
  booktitle = {...},                  ← 会议名（卡片上原样显示）
  year = {2026}, month = jul,
  pages = {73--82}, volume = {16637}, series = {...}, publisher = {...},
  doi = {...}, urldate = {...},
  abstract = {...},                   ← 摘要（Publications 页 Abstract 按钮）
  description = {...},                ← 卡片副标题
  keywords = {...},                   ← 逗号分隔，变成筛选标签
  preview = {KSEM2026.png}            ← 封面图，放 prism/public/papers/
}
```

- 条目顺序 = 页面显示顺序（当前最新在前）。
- `publications.toml` 的 `description` 是页面顶部说明文字。

### 2. 改个人信息/社交链接 → `content/config.toml` 和 `content_zh/config.toml`

- `[site]`：站点标题、meta 描述、favicon、last_updated（页脚日期）
- `[author]`：姓名、头衔、机构（中英各自独立）
- `[social]`：邮箱、位置、Google Scholar、ORCID、GitHub
- `[features]`：`enable_likes`、`enable_one_page_mode`
- `[i18n]`：语言配置（en 默认，zh 可选）
- `[[navigation]]`：导航菜单（要加/减页面就改这里，中英两份都要改）

### 3. 改首页 About 段落 → `content/bio.md`（和 `content_zh/bio.md`）

Markdown 格式；`about.toml` 里 `research_interests` 是右侧兴趣列表，`sections` 控制首页展示哪些区块。

### 4. 改简历 → `content/cv.md`（和 `content_zh/cv.md`）

Markdown 格式，章节标题用 `##`。注意中英文两份要同步改。

### 5. 改奖项/服务 → `content/awards.toml` / `content/services.toml`（及 content_zh）

```toml
type = "card"
title = "Awards"
description = "Awards I have received."

[[items]]
title = "省级二等奖"
subtitle = "第十九届中国大学生计算机设计大赛"
date = "2026年5月"
content = ""
```

### 6. 改新闻 → `content/news.toml`（及 content_zh）

```toml
[[news]]
date = "2026-5"
content = "We won National 3rd Prize in the Huawei ICT Competition 🎉"
```

### 7. 换图片

- 头像：`prism/public/bio.jpg`（config 里 `avatar = "/bio.jpg"`）
- 论文封面：`prism/public/papers/<文件名>`（bib 里 `preview` 对应）
- favicon：`prism/public/favicon.png`（config 里 `favicon`）

## 加新页面（如 Teaching）

1. 建 `content/<slug>.toml`（type 可以是 `card`/`text`/`publication`/`about`，text 配 `<slug>.md`）和 `content_zh/<slug>.toml`
2. `config.toml` + `content_zh/config.toml` 的 `[[navigation]]` 加一项
3. 重新构建部署

## 技术机制（排查问题时参考）

- **hydration 隐藏机制**：SSR 内容包在 `<div style="visibility:hidden">` 中（`ThemeProvider` 的 mounted 状态包装），hydration 完成后切换为可见；`src/app/layout.tsx` 里有 3 秒兜底脚本（慢网/JS 失败时显示 SSR 内容）。改 layout 时不要删它。
- **i18n**：默认 locale 为 en；zh 内容在 `content_zh/`，`[i18n]` 配置在 en 的 config.toml。
- **构建**：`next.config.ts` 为 `output: 'export'` + `trailingSlash: true`；bib 通过 webpack `asset/source` 内联读取。
- **React 418 hydration 警告**：干净源码构建不存在；如果部署产物出现它，说明产物被手改过。
- 线上首页 KSEM 论文曾经靠 post-hydration DOM 注入脚本显示；源码回填后已改为原生渲染（`publications.bib`），**不要再加注入脚本**。

## 验证清单（改动后）

- [ ] `npm run build` 成功（8 个静态页）
- [ ] 本地浏览器检查 `prism/out/` 页面正常（`main` 存在、内容可见、无控制台错误）
- [ ] `rsync` 后 `.nojekyll` 仍在
- [ ] push 后 `curl -sI` 的 last-modified 更新
