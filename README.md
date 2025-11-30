[![Node.js Version](https://img.shields.io/badge/node-%3E%3D18-brightgreen)](https://nodejs.org/)
[![Hexo Version](https://img.shields.io/badge/hexo-3.8.0-blue)](https://hexo.io/)

English | [简体中文](./README.zh-CN.md)

---

# hexo-icarus-blog

A fast, elegant & powerful static blog framework powered by [Hexo](https://hexo.io/).

![Icarus Theme Preview](./source/icarus_preview.png)

## Deploy

Generated output dir: `.deploy_git` (safe to exclude when moving)

Deploy to GitHub Pages or Gitee Pages:

```bash
hexo clean && hexo g && gulp && hexo d
```

| Command | Description |
|---------|-------------|
| `hexo clean` | Clean cache files |
| `hexo g` | Generate static files |
| `hexo s` | Start local server (preview) |
| `gulp` | Minify HTML/CSS/JS/images |
| `hexo d` | Deploy to remote |

## Configuration

Edit `_config.yml`:

```yml
deploy:
  type: git
  repository: git@github.com:username/username.github.io.git
  branch: master
```

## Features

- **Icarus Theme** — Clean, responsive design, customized
- **Live2D Widget** — Interactive mascot (hijiki)
- **Gulp Pipeline** — Automated asset minification
- **Markdown** — Write posts in Markdown

## Customization

Based on [hexo-theme-icarus](https://github.com/ppoffice/hexo-theme-icarus) with the following modifications:

- **Navbar** — Custom icon + text logo; embedded search input box
- **Profile Widget** — Added WeChat, Gitee, Weibo links
- **Links Widget** — Icon prefix on link titles
- **Article Time** — Relative timestamp on list view, absolute on post view
- **Thumbnail** — Hidden on post page to reduce visual clutter
- **Summary** — Stripped HTML tags from post excerpts (cleaner layout)
- **Post Layout** — Two-column layout (wider content area vs. default three-column)
- **TOC** — Enabled by default; sticky scrolling for long articles
- **Copyright** — Article footer copyright notice
- **Footer** — Custom site info
- **Live2D** — Interactive hijiki mascot on all pages

## License

MIT
