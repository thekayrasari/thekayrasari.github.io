# thekayrasari.github.io

Personal website built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme, deployed to GitHub Pages via GitHub Actions.

🔗 **Live site:** [thekayrasari.github.io](https://thekayrasari.github.io)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Static site generator | [Hugo](https://gohugo.io/) |
| Theme | [PaperMod](https://github.com/adityatelange/hugo-PaperMod) |
| Styling | CSS (custom overrides in `assets/css/`) |
| Hosting | GitHub Pages |
| CI/CD | GitHub Actions (`.github/workflows/`) |

---

## Project Structure

```
thekayrasari.github.io/
├── .github/
│   └── workflows/          # GitHub Actions deployment pipeline
├── assets/
│   └── css/                # Custom CSS overrides on top of PaperMod
├── content/                # Markdown content (pages, posts, projects)
├── layouts/                # Custom Hugo layout overrides
├── static/                 # Static files served as-is (images, favicon, etc.)
├── themes/
│   └── PaperMod/           # Hugo PaperMod theme (git submodule)
├── config.yml              # Hugo site configuration
└── .gitignore
```

---

## Prerequisites

- [Hugo](https://gohugo.io/installation/) — extended version recommended
- [Git](https://git-scm.com/)

---

## Local Development

### 1. Clone the repository

```bash
git clone --recurse-submodules https://github.com/thekayrasari/thekayrasari.github.io.git
cd thekayrasari.github.io
```

> The `--recurse-submodules` flag is required to pull in the PaperMod theme.

If you already cloned without it, initialize the submodule manually:

```bash
git submodule update --init --recursive
```

### 2. Start the development server

```bash
hugo server -D
```

The site will be available at `http://localhost:1313`. Hugo watches for file changes and hot-reloads automatically. The `-D` flag includes draft content.

### 3. Build for production

```bash
hugo --minify
```

Output is generated in the `public/` directory.

---

## Adding Content

New pages and posts live in the `content/` directory as Markdown files with YAML front matter. Example:

```markdown
---
title: "My New Post"
date: 2026-03-06
draft: false
---

Content goes here.
```

---

## Deployment

Deployment is automated via GitHub Actions. Any push to the `main` branch triggers the workflow in `.github/workflows/`, which builds the Hugo site and pushes the output to GitHub Pages.

No manual deployment step is required.

---

## Customization

- **Site config** — edit `config.yml` to update metadata, navigation, social links, and PaperMod theme options.
- **Custom styles** — add or edit CSS files in `assets/css/`. Hugo bundles these on top of the theme's default styles.
- **Layout overrides** — place custom HTML templates in `layouts/` to override any PaperMod template without modifying the theme submodule directly.

---

## License

MIT — see [LICENSE.md](LICENSE.md).