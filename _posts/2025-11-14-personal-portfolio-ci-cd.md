---
layout: post # Do not change
category: [devops, git, ci-cd] # Project categories
title: "CI/CD Pipeline for this Personal Portfolio" # Project title
date:   2025-11-14 21:30:00 -0800 # Today's date
author: LinLinHua # Your author nick
prevPart: _posts/2025-11-13-receipt-inbox.md # Links to your previous project
# nextPart: (This is your latest project)
og_description: "Building the automated CI/CD pipeline for this site using rbenv, Jekyll, Bundler, and GitHub Pages."
---

This portfolio website is itself a software project, built not with a drag-and-drop editor, but with a full CI/CD (Continuous Integration / Continuous Deployment) pipeline.

The goal was to create a professional, fast, and secure static site that could be automatically deployed just by pushing a commit to GitHub. This process involved solving several common (and frustrating) development challenges.

### 1. Development Environment Setup

The first challenge was overcoming the limitations of the locked-down, pre-installed system Ruby on macOS (v2.6.10), which caused constant `Gem::FilePermissionError` issues.

**Solution:**
* Installed **`rbenv`** to manage a clean, user-space Ruby environment.
* Installed a modern, stable version of Ruby (`3.2.3`), completely isolating the project from the protected system files.

### 2. Dependency & Version Management

The second challenge was a series of version conflicts between the theme and the new Ruby environment.

**Solution:**
* Used **`Bundler`** and the `Gemfile` to manage all project dependencies.
* **Debugged and resolved** multiple version conflicts by upgrading the project's core `jekyll` gem from `4.2.0` to `4.4.1` and removing conflicting gems (like `minima`).

### 3. Theme Debugging & Overrides

After fixing the environment, the `jekyll-theme-simplex` theme itself crashed the server due to two separate bugs related to the new Ruby version:
1.  An `undefined method \`.tainted?'` error.
2.  A `Could not parse name of post` error from the `post_url` tag.

**Solution:**
* I debugged the theme's source code, identified the broken Liquid tags, and fixed them.
* Using Jekyll's **override system**, I created a local `_layouts/post.html` file in this project. This local file overrides the theme's broken file, patching the bugs without needing to "fork" the entire theme.

### 4. Automated Deployment (CI/CD)

The entire site is hosted for free on **GitHub Pages**.
* A CI/CD workflow is configured to watch the `main` branch.
* Any `git push` to `main` automatically triggers a new Jekyll build on GitHub's servers and deploys the new static site to the public URL.

### Technologies Used
* **Jekyll** (Static Site Generator)
* **rbenv** (Ruby Environment Manager)
* **Bundler** (Dependency Management)
* **Git** (Version Control)
* **GitHub Pages** (CI/CD & Hosting)
* **YAML**, **HTML**, **Liquid**

---

### Project Links

<div class='sx-button'>
  <a href='https://github.com/LinLinHua/LinLinHua.github.io' class='sx-button__content theme' target='_blank' rel='noopener noreferrer'>
    <img src='/assets/img/icons/c.svg'/> View Source for This Website
  </a>
</div>