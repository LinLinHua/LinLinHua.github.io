# linlinhua.github.io

Personal portfolio of Lin-Lin Hua — GPU Systems Engineer & Researcher.

## Site structure

```
_config.yml             ← Site settings (title, url, plugins)
_layouts/
  default.html          ← Base HTML wrapper (nav, footer, CSS)
  post.html             ← Individual project page template
_includes/
  nav.html              ← Navigation bar
  footer.html           ← Footer
_data/
  author.yml            ← Personal info, experience, skills, education
  certificates.yml      ← Awards and scholarships
_posts/                 ← One .md file per project
  2026-03-03-flashattention.md
  2026-03-02-tensorrt.md
  2026-03-01-cuda-gemm.md
  2025-11-13-receipt-inbox.md
  2025-11-12-intent-classification.md
  2021-09-01-mor-gans.md
  2019-03-01-hfo2.md
  2018-09-01-optics.md
assets/
  css/main.css          ← All styles
  img/posts/            ← Project images and screenshots
  pdf/                  ← Resume, papers, proposals
work/index.html         ← All projects listing page
about/index.html        ← About page (pulls from _data/author.yml)
contact/index.html      ← Contact page
index.html              ← Homepage
```

## Local development

Install Ruby and Jekyll (one-time setup):
```bash
gem install bundler jekyll
bundle install
```

Run locally:
```bash
bundle exec jekyll serve
```
Open http://localhost:4000

## Adding a new project

1. Create a new file in `_posts/` named `YYYY-MM-DD-your-title.md`
2. Copy the front matter from any existing post and update the fields
3. Add images to `assets/img/posts/`
4. Push to `main` — GitHub Pages deploys automatically

## Updating personal info

- Edit `_data/author.yml` to update bio, experience, education, skills
- Edit `_data/certificates.yml` to add new awards or scholarships

## Deployment

Push to `main` branch. GitHub Pages builds automatically (~1 min).
