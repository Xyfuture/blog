---
title: "使用Github Actions部署Hugo Blog"
date: 2022-07-15
draft: false
---

# 前言

这是第三次搭博客了，第一次用hexo构建的，要求机子上必须有nodejs环境，跨设备能力较差，放弃了，第二次使用hugo构建的，hugo本身只有可执行文件，跨设备能力很好，但是其构建时也要维护一个静态资源的环境，也是一堆文件，于是又放弃了。当然前面这两点都是为我不想写博客找借口🤣。

这次构建使用的还是hugo，但是为了拜托环境的依赖，我把构建工作放到Github actions上进行了，现在本地理论上只需要保存原始的markdown文件即可，push到远程仓库之后，由GitHub Action自动完成构建并发布到Github Page上，个人仅需要完成写作工作，构建发布的事情完全自动化了，这下“理论上”也没有拖着不写的理由了。

# Hugo配置

Hugo本身是用go编写的，最后构建出来是一个binary文件，相较于脚本语言，一个binary的便携性大大增强，不需要像nodejs一样先配置node环境。

Hugo的工作比较像编译器，把markdown源文件转换成带样式的html网页。具体的样式也就是主题是可以配置的，有大量的第三方主题可供我们使用，这里使用的主题是LoveIt，比较简洁，支持公式，配置文档写的也还不错。Hugo使用前需要使用`hugo init`创建一个工作目录，在里面我们会放置各类静态文件和主题文件，同时把markdown文件放置到content文件夹下，随后调用`hugo`,Hugo就会根据目录下的各类文件构建出html文件，并把生成的结果放置到public文件夹下。我们把public文件夹下的内容push到Github Page仓库中，访问Github Page就能看到自己的博客了。本地想要直接预览的话可以使用`hugo server`，他会检测文件变化的情况，实时进行构建，比较方便测试时使用。

Hugo在构建网页时会根据工作目录下的config.toml进行定制化配置，比如指定主题，网页中作者的信息等等，所以我们在使用构建网页前要先配置一下config.toml文件，详细的配置信息可以去你使用的第三方主题文档中找，大部分主题会给一个模板，照着改改就可以了。

# 使用Github Actions自动部署

Github Actions是Github推出的一个免费的持续集成服务，可以理解为Github免费给我们一台小服务器，供我们执行一些简单的任务。至于Github Actions的具体内容，这里就不详细讲了，详见[Actions](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

我想要的就是本地只有原始的markdown文件，Hugo以及其相应的工作目录都不应该存储在本地上，当我把本地的markdown文件push到Github上时，Github Actions会自动使用Hugo进行构建，并发布到Github Page上。

我们需要三个仓库实现整件事情，分别是blog仓库用于存放原始的markdown文件，hugo_blog_env仓库用于存放hugo的工作目录和第三方主题，最后一个是存放Github Page的仓库。blog仓库仅存储原始的markdown文件，其作为一个submodule链接到hugo_blog_env仓库的content文件夹下。首先在blog仓库中开一个action，其要在发生push之后，向hugo_blog_env仓库发送repository dispatch信号，告诉hugo_blog_env仓库自己被更新了，hugo_blog_env仓库在收到repository dispatch之后开始构建工作，首先把最新的blog仓库最新的内容拉下来到content文件夹下，然后拉去hugo的binary，开始构建，并将构建的结果部署到Github Page的仓库中，最后把最新的结果提交到hugo_blog_env中。这里面很多脚本都是使用其他人写好的，所以实现起来比较简单。这里把脚本记录一下

blog -> hugo_blog_env

```yaml
name: DeployGitHubPages

on:
  push:
    branches:
      - main  # Set a branch to deploy

jobs:
  update:
    runs-on: ubuntu-18.04
    steps:
      - name: Repository Dispatch
        uses: peter-evans/repository-dispatch@v2
        with:
          token: ${{ secrets.REPO_ACCESS_TOKEN }}
          repository: xyfuture/hugo_blog_env
          event-type: blog_update
```

hugo_blog_env -> Github Page

```yaml
name: DeployGitHubPages

on:
  repository_dispatch:
    types: [blog_update]
#  push:
#    branches:
#      - main  # Set a branch to deploy

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: recursive  # Fetch Hugo themes (true OR recursive)
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod
      
      - name: Pull Content
        run: |
          cd ./content
          git pull origin main
      
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          # hugo-version: '${{ steps.hugo-version.outputs.HUGO_VERSION }}'
          hugo-version: '0.101.0'
          extended: true
          
      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          external_repository: xyfuture/xyfuture.github.io
          publish_branch: main
          publish_dir: ./public

      - uses: stefanzweifel/git-auto-commit-action@v4
```

这里面还有几个需要配置的信息，首先发送Repository Dispatch请求需要申请一个Github PAT(Personal Access Token),然后从一个仓库提交到另一个仓库需要一个Deploy Key(RSA key)。

最后所有这些都完成后，测试一下看看能不能通，如果可以的话构建部署就完全自动了，本地仅同步blog仓库，写了新的markdown就push上去，然后在hugo_blog_env中自动构建出新的网页并发布到Github Page上，速度的话感觉也还行，整个一套跑下来2min之内基本上也就完成了。

# 后记

经过这一番折腾，大致了解了Github Actions的运作流程，也实现了博客的自动部署，以后不更博客的理由可能真的只有懒了，虽然之前也是hhhh。