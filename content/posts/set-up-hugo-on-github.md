---
author: Stu
title: "Setting up your Hugo site on GitHub"
summary: "Automatically building your Hugo site using GitHub Actions and hosting it with GitHub Pages."
date: 2022-12-06T19:40:36Z
draft: true
tags: [blogging, hugo, github]
---

This blog is about taking your Hugo blog and getting it automatically built and hosted on GitHub.

I'm assuming you have a Hugo site ready to go so I won't be covering how to build your Hugo site from scratch. If you're just starting out there are plenty of blogs about how to set up Hugo and the [docs](https://gohugo.io/documentation/) are really good.

# Building your site with GitHub actions.

Your first port of call should be reading the docs from Hugo on how to [host on GitHub](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

Below we'll step through getting your site on to GitHub.

#### Create a GitHub repo

Login to GitHub and create repository named "blog" or similar.

#### Push your Hugo site to GitHub

If you're new to Git it can be quite a learning curve, I highly recommend the [GitHub learning resources](https://docs.github.com/en/get-started/quickstart/git-and-github-learning-resources) and the [Pro Git book](https://git-scm.com/book/en/v2) which goes over getting started and more advanced topics.

Assuming you've got basic knowledge of Git the commands below should be familiar.

Get the .gitignore file from GitHub so we only use git for the required files.
```bash
$ curl -o .gitignore https://raw.githubusercontent.com/github/gitignore/main/community/Golang/Hugo.gitignore
```

Create a Git repo in the root directory of your Hugo site directory, add the files to Git and commit.
```bash
$ git init -b main
$ git add *
$ git commit -m "Initial commit"
```

Add your GitHub repo as a remote to you local Git repo, then push to GitHub.
```bash
$ git remote add origin https://github.com/<your username>/<your repo name>.git
$ git remote push origin main
```

Hopefully the above went smoothly and you can now see your code in your GitHub repo.

#### Add the GitHub Actions workflow

To keep things simple you can use my GitHub Actions workflow by downloading it from GitHub repo.

From the root directory of your Git repo, run:
```bash
curl -o .github/workflows/hugo.yaml https://raw.githubusercontent.com/pragmatarget/blog/main/.github/workflows/hugo.yaml
```

This will download this rather long workflow yaml file:
```yaml
# Workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.111.3
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb          
      - name: Install Dart Sass Embedded
        run: sudo snap install dart-sass-embedded
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v3
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1

```



