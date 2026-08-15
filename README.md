# FCJ Internship Report Website

An **Internship Report Site** built with [Hugo](https://gohugo.io/) and the [Hugo Learn Theme](https://github.com/matcornic/hugo-theme-learn), designed to document work logs, proposals, technical blogs, workshop projects, and evaluations during the **First Cloud AI Journey (FCJ)** Bootcamp at **Amazon Web Services (AWS) Vietnam**.

---

## Project Overview

This repository serves as a structured online report portfolio for the AWS Vietnam Workforce Bootcamp. It features bilingual support (Vietnamese & English), markdown-driven content organization, and documentation-style navigation via the Hugo Learn theme.

### Student Details

| Field | Value |
|-------|--------|
| **Full Name** | Nguyen Ho Phuong Tay |
| **Phone** | +84 846777901 |
| **Email** | phuongtay52636@gmail.com |
| **University** | Sai Gon University |
| **Major** | Information Technology (Class: DCT1221) |
| **Company** | Amazon Web Services Vietnam Company Limited |
| **Program** | Workforce Bootcamp — First Cloud AI Journey |
| **Duration** | June 22, 2026 – August 15, 2026 |

---

## Features

- **Hugo Static Site Generator** — Fast builds and lightweight static output.
- **Hugo Learn Theme** — Clean documentation-style UI with collapsible sidebar, breadcrumbs, and built-in search.
- **Bilingual Support** — Vietnamese (default) and English via `_index.vi.md` / `_index.en.md`.
- **Custom Theme Overrides** — Logo, menu, and footer customized under `layouts/partials/`.
- **Structured Report Sections:**
  1. **Worklog** — Weekly activity tracking (Weeks 1–8)
  2. **Proposal** — Project proposals and technical plans
  3. **Blogs Posted** — Technical articles and published content
  4. **Events Participated** — AWS community events and workshops attended
  5. **Workshop** — Hands-on lab: secure hybrid access to S3 via VPC endpoints
  6. **Self-Evaluation** — Growth reflection and performance assessment
  7. **Sharing & Feedback** — Program feedback and mentorship notes
- **CI/CD Ready** — GitHub Actions for GitHub Pages + Vercel config with Hugo `0.134.3`

---

## Project Structure

```text
.
├── archetypes/                 # Content templates for new pages
├── content/                    # Site content (Markdown)
│   ├── 1-Worklog/              # Weekly worklogs (Week 1–8)
│   │   ├── 1.1-Week1/
│   │   ├── ...
│   │   └── 1.8-Week8/
│   ├── 2-Proposal/             # Internship project proposal
│   ├── 3-BlogsPosted/          # Published blogs
│   │   ├── 3.1-Blog1/
│   │   ├── 3.2-Blog2/
│   │   └── 3.3-Blog3/
│   ├── 4-EventParticipated/    # Events attended
│   │   ├── 4.1-Event1/
│   │   └── 4.2-Event2/
│   ├── 5-Workshop/             # Workshop lab chapters
│   │   ├── 5.1-Workshop-overview/
│   │   ├── 5.2-Prerequiste/
│   │   ├── 5.3-S3-vpc/
│   │   ├── 5.4-S3-onprem/
│   │   ├── 5.5-Policy/
│   │   └── 5.6-Cleanup/
│   ├── 6-Self-evaluation/
│   ├── 7-Feedback/
│   ├── _index.en.md            # Home page (English)
│   └── _index.vi.md            # Home page (Vietnamese)
├── layouts/                    # Custom HTML layout overrides
│   ├── partials/
│   │   ├── logo.html
│   │   ├── menu.html
│   │   ├── menu-footer.html
│   │   └── custom-footer.html
│   └── shortcodes/
│       ├── tabs.html
│       ├── tab.html
│       └── ghcontributors.html
├── static/                     # Static assets (images, CSS, fonts)
│   ├── css/
│   ├── fonts/
│   ├── images/                 # Avatar and illustration assets
│   └── AWS_Logo.svg
├── themes/
│   └── hugo-theme-learn/       # Theme (Git submodule)
├── .github/workflows/
│   └── hugo.yml                # Deploy to GitHub Pages
├── config.toml                 # Main Hugo configuration
├── vercel.json                 # Vercel build configuration
└── README.md                   # Project documentation
```

### Workshop Chapters (`5-Workshop`)

Hands-on lab: **Secure Hybrid Access to Amazon S3 using VPC Endpoints**.

| Chapter | Topic |
|---------|--------|
| 5.1 | Workshop overview |
| 5.2 | Prerequisites |
| 5.3 | Access S3 from VPC (Gateway Endpoint) |
| 5.4 | Access S3 from on-premises (Interface Endpoint + DNS) |
| 5.5 | VPC Endpoint Policies (optional) |
| 5.6 | Cleanup |

---

## Prerequisites

Before running the project locally, ensure you have:

1. **Git** — [Download Git](https://git-scm.com/)
2. **Hugo Extended** — [Install Hugo](https://gohugo.io/installation/)
   - Recommended version: **`0.134.3`** or higher (matches CI / Vercel)
   - Verify:

     ```bash
     hugo version
     ```

   - macOS (Homebrew):

     ```bash
     brew install hugo
     ```

---

## Getting Started / How to Run

### 1. Clone the Repository

The theme is included as a Git submodule. Clone with `--recursive`:

```bash
git clone --recursive https://github.com/nguyentay52636/fcj-workshop-template.git
cd fcj-workshop-template
```

If you already cloned without `--recursive`:

```bash
git submodule update --init --recursive
```

### 2. Run Locally (Development Server)

```bash
hugo server
```

Or include draft pages:

```bash
hugo server -D
```

- Open: [http://localhost:1313/](http://localhost:1313/)
- Default language: **Vietnamese**
- Switch language via the language selector in the UI
- The site hot-reloads when content or config changes

### 3. Build for Production

```bash
hugo --minify
```

Static files are generated in the `public/` directory.

---

## Content Conventions

| Language | File naming |
|----------|-------------|
| Vietnamese (default) | `_index.vi.md` |
| English | `_index.en.md` |

Minimum front matter example:

```yaml
---
title: "Week 6 Worklog"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---
```

- Place images under `static/images/` and reference them as `/images/your-file.png`
- Keep Vietnamese and English pages in sync when updating content

### Key configuration (`config.toml`)

| Parameter | Value | Description |
|-----------|--------|-------------|
| `defaultContentLanguage` | `vi` | Default site language |
| `theme` | `hugo-theme-learn` | Documentation theme |
| `themeVariant` | `workshop` | Custom color variant |
| `Languages.vi.weight` | `1` | Vietnamese first in switcher |
| `Languages.en.weight` | `2` | English secondary |

### Customizing the UI

| File | Purpose |
|------|---------|
| `layouts/partials/logo.html` | Sidebar logo |
| `layouts/partials/menu.html` | Navigation menu |
| `layouts/partials/menu-footer.html` | Menu footer |
| `layouts/partials/custom-footer.html` | Page footer extras |
| `static/css/` | Additional / override styles |

---

## Deployment

### Deploying to GitHub Pages

Pushing to the `main` branch triggers `.github/workflows/hugo.yml`:

1. Checks out the repo with submodules
2. Sets up Hugo Extended `0.134.3`
3. Builds with `hugo --minify`
4. Publishes the `public/` folder to the `gh-pages` branch

### Deploying to Vercel

1. Push your code to GitHub
2. Import the project into [Vercel](https://vercel.com)
3. Vercel reads `vercel.json` and builds with Hugo `0.134.3`

```json
{
  "build": {
    "env": {
      "HUGO_VERSION": "0.134.3"
    }
  }
}
```

---

## Useful Links

- [AWS Study Group — Blog](https://awsstudygroup.com)
- [AWS Study Group — Facebook Group](https://www.facebook.com/groups/awsstudygroupfcj)
- [Hugo Documentation](https://gohugo.io/documentation/)
- [Hugo Learn Theme](https://github.com/matcornic/hugo-theme-learn)

---

## License & Acknowledgments

- **Theme:** [Hugo Learn Theme](https://github.com/matcornic/hugo-theme-learn) by Mathieu Cornic
- **Organization:** AWS Study Group / First Cloud AI Journey (FCJ)
- **Company:** Amazon Web Services Vietnam
