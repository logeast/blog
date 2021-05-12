---
title: "使用 Hugo + GitHub Pages + Github Action 搭建个人博客"
date: 2021-05-12T17:15:14+08:00
draft: false
---

# 安装 Hugo

本文以 macOS 系统为例，安装 Hugo，其他系统可参考 [Hugo 安装指南](https://gohugo.io/getting-started/installing)。

HomeBrew 是 macOS 上的包管理器，我们将用它来安装 Hugo。当然也可以用源码编译的方式安装，这里是[安装指引](https://gohugo.io/getting-started/installing#build-from-source-on-mac)。

```bash
brew install hugo
```

验证你的安装。

```bash
hugo version
```

# 创建站点

```bash
hugo new site blog
```
上面的指令将会创建一个名为 `blog` 的站点。

## 选择主题
访问 [themes.gohugo.io](https://themes.gohugo.io/) 站点，选择喜欢的主题，下面以 [cactus](https://themes.gohugo.io/hugo-theme-cactus/) 主题为例。

```bash
cd blog
git init
git clone https://github.com/monkeyWzr/hugo-theme-cactus.git themes/cactus
```

更新 `./blog/config.toml` 文件，指定主题。

```bash
# config.toml

theme = "cactus"
```

或者直接将 `./blog/themes/cactus/config.toml` 拷贝到 `./blog` 目录下，进行自己的更改。

至此，博客站点搭建成功，运行下面指令测试一下。

```bash
hugo server
```

## 添加文档
可以手动在 `./blog/contnet` 目录下创建内容文件，或者用下面指令创建。

```bash
hugo new posts/my-first-post.md
```
这将会创建 `./blog/contnet/posts/my-first-post.md` 文件，默认处于草稿状态，如果需要发布可以更新为 `draft: false`。

```bash
---
title: "My First Post"
date: 2021-05-11T17:15:14+08:00
draft: true
---
```

# 部署 GitHub Pages
我们将采用 `https://<USERNAME>.github.io/<PROJECT>/` 的方式部署我们的博客站点。
如果希望选择 `https://<USERNAME>.github.io/` 的方式，可以参考 [GitHub Pages 文档](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#user--organization-pages) 。

在 GitHub 新建一个名为 `blog` 的仓库，关联并提交我们 `./blog` 站点既可。
## 添加 GitHub Action 自动部署
在 `./blog` 目录下创建 `.github/workflows/gh-pages.yml` 文件，并写入以下内容，然后 push 所有改动到远程仓库。

```bash
name: github pages

on:
  push:
    branches:
      - main  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: recursive  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          # extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

Action 会在 `main` 分支提交时自动触发部署，并自动创建一个名为 `gh-pages` 的分支，我们的打包后的站点就在该分支上。

![image](https://user-images.githubusercontent.com/26041539/117929672-a8195780-b32f-11eb-900f-e3da6b8119c3.png)

如上图所示，打开 创建的 blog 仓库，进入设置页面并选择 pages 菜单，然后选择 `gh-pages` 分支，目录选择为 root。

至此，网站部署完成，访问 [https://<USERNAME>.github.io/blog/](https://logeast.github.io/blog/) 即可看到部署成功的站点了。

## （可选）绑定域名

也可绑定自己的域名，将经过解析的域名填入域名输入框，确认后会生成 CNAME 文件。有一个问题时候每次部署会清空 gh-pages 下的问题导致绑定失败，一种可行的方案是在 main 分支新增 './public/` 目录，将 CNAME 移动到该目录下，以使得 CNAME 得以保存。

现在，可以通过自己的域名访问了。这里是我的博客 [向东_logeast 的网络日志](https://blog.logeast.cn/)，欢迎访问。
