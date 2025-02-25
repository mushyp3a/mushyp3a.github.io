+++
title = "How to Publish Hugo Website from Emacs"
author = ["mushyp3a"]
date = 2025-02-25
tags = ["emacs", "webdev"]
draft = false
+++

## Introduction {#introduction}

This is a short guide on how to setup a Hugo website where pages / posts can easily be captured and published from within Emacs. I will assume you have an Emacs distribution and github account setup.

What I Use (at the time of writing):

-   Doom Emacs (installation method may vary for different distributions)
-   Website hosted with Github Pages
-   [Hugo](https://gohugo.io/)
-   [ox-hugo](https://ox-hugo.scripter.co/) (Emacs package)
-   [Hextra Hugo Theme](https://themes.gohugo.io/themes/hextra/)


## Installation {#installation}


### Hugo {#hugo}

Install using preferred method, I used the following commands:

```nil
sudo apt update
sudo apt install hugo
```

Check if Hugo is correctly installed by using:

```nil
hugo version
```


### ox-hugo {#ox-hugo}

Install the ox-hugo Emacs package - I installed it using [Melpa](https://melpa.org/#/getting-started) but feel free to use your preferred installation method.


## Setup {#setup}


### Hugo Website {#hugo-website}

First create a basic hugo site:

```nil
hugo new site site-name
```


#### Github Pages {#github-pages}

cd into the newly created folder and init your git repository:

```nil
cd site-name
git init
```

Now create a repository on github named "username.github.io" e.g. in my case it is "mushyp3a.github.io". Add that remote repo as the origin for your newly created local one:

```nil
git remote add origin <remote-repo-URL>
```

Now upload all the newly created hugo site files to your remote repository.


#### Hugo Themes {#hugo-themes}

I won't cover the setup of hugo themes as each have different configuration options. However, you can install them by adding a git submodule:

```nil
git submodule add https://link-to-theme.git themes/theme-name
```

And then enable it in **hugo.toml**:

```bash
echo 'theme = "theme-name"' >> config.toml
```


### ox-hugo Configuration {#ox-hugo-configuration}

The basic variables needed to start publishing org notes to your hugo website are **org-hugo-section** and **org-hugo-base-dir**

```elisp
(setq org-hugo-section "website-section")
;;e.g. where you want your pages/posts to appear - "blog" for website.github.io/blog
(setq org-hugo-base-dir "~/path-to-your-hugo-website-folder")
```


## Usage {#usage}

In any **.org** file you can run the command:

```nil
org-hugo-export-wim-md
```

It will use the default values for **org-hugo-section** and **org-hugo-base-dir** set above - e.g. if using "blog" as section a new post will appear (depending on your theme).


### Customise Export Settings Per File {#customise-export-settings-per-file}

**org-hugo-section** and **org-hugo-base-dir** can be set on a per file basis by including metadata at the top of the org mode file in this format:

```nil
#+HUGO_BASE_DIR: /path/to/your/hugo/site
#+HUGO_SECTION: section-name
```


## Publishing {#publishing}

If you also use github pages to host your site then you can manually commit your changes and push your changes to your remote repo but you still need to create a github action to publish the site.

First create a new file ".github/workflows/deploy.yaml" in your hugo site directory. Then, paste in my github action below which automatically publishes on any new push or create your own:

```yaml
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.137.1
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
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
        uses: actions/deploy-pages@v4
```
