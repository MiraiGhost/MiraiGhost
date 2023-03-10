---
title: 从零开始的Hexo
date: 2023-03-10 15:11:16
tags: Blog,Hexo
---
# 关于本文
使用 Hexo 生成博客模板，自动上传 Github 并自动生成页面文件并通过GithubPage访问。

## 准备
本地安装 [Nodejs](https://nodejs.org/), [Git](https://git-scm.com/downloads), [VSCode](https://code.visualstudio.com/)。
平台账号准备 [GitHub](https://github.com/signup?source=login), [Cloudflare](https://dash.cloudflare.com/sign-up) 

## Hexo 安装
打开VSCode
 ++ Ctrl+Shift+` ++ 快捷键唤出终端。
端运行以下命令安装hexo
```bash
npm install -g hexo-cli
```
### 初始化 Hexo
完成hexo安装后,执行以下命令会在指定位置 folder 生成所需文件。
```bash
hexo init <folder>
npm install
```

### Hexo 命令
+++primary init
```bash
Hexo init [folder]
```
新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。
本命令相当于执行了以下几步：
1. Git clone hexo-starter 和 hexo-theme-landscape 主题到当前目录或指定目录。
2. 使用 Yarn 1、pnpm 或 npm 包管理器下载依赖（如有已安装多个，则列在前面的优先）。npm 默认随 Node.js 安装。
+++

+++primary new
```bash
hexo new [layout] <title>
```
新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。
:::default no-icon
 hexo new "post title with whitespace"
:::
|参数|描述|
|------|------|
|-p,--path|自定义新文章路径|
|-r,--replace|如果存在同名文章，将其替换|
|-s,--slug|文章的Slug,作为新文章的文件名和发布后的URL|
默认情况下，Hexo 会使用文章的标题来决定文章文件的路径。对于独立页面来说，Hexo 会创建一个以标题为名字的目录，并在目录中放置一个 index.md 文件。你可以使用 --path 参数来覆盖上述行为、自行决定文件的目录：
:::default no-icon
 hexo new page --path about/me "About me"
:::
以上命令会创建一个 source/about/me.md 文件，同时 Front Matter 中的 title 为 "About me"
注意！title 是必须指定的！如果你这么做并不能达到你的目的：
:::default no-icon
 hexo new page --path about/me
:::
此时 Hexo 会创建 source/_posts/about/me.md，同时 me.md 的 Front Matter 中的 title 为 "page"。这是因为在上述命令中，hexo-cli 将 page 视为指定文章的标题、并采用默认的 layout。
+++

+++primary generate
```bash
hexo generate
# 简写为
hexo g
```
生成静态文件。
|选项|描述|
|------|------|
|-d,--deploy|文件生成后立即部署网站|
|-w,--watch|监视文件变动|
|-b,--bail|生成过程中如果发生任何未处理的异常则抛出异常|
|-f,--force|强制重新生成文件
Hexo 引入了差分机制，如果 public 目录存在，那么 hexo g 只会重新生成改动的文件。
使用该参数的效果接近 hexo clean && hexo generate|
|-c,--concurrency|最大同时生成文件的数量，默认无限制|
+++

+++primary publish
```bash
hexo publish [layout] <filename>
```
启动服务器。默认情况下，访问网址为: http://localhost:4000/。
|选项|描述|
|------|------|
|-p,--port|重设端口|
|-s,--static|只使用静态文件|
|-l,--log|启动日记记录，使用覆盖记录格式|
+++


## 部署到 GitHub
在GitHub新建一个名称和你github用户名一样的仓库。
将生成的文件上传到你新建仓库的master分支。

### GitHubAction
在仓库中新建 /.github/workflow/***.yml 文件,github会自动检索workflow下的脚本，并自动执行操作。
```bash
name: Pages #名称随意

on:
  push:
    branches:
      - master  # 你所上传文件存放的分支
jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write  
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 19.x        # 这里和你本地的Node.js版本一致。
        uses: actions/setup-node@v2   
        with:                         
          node-version: '19'          # 这里和你本地的Node.js版本一致。
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies                 
        run: npm install                           
    #   - name: unInstall markdown-it         # 如果你的hexo安装了额外的插件，需要在这里添加             
    #     run: npm un hexo-renderer-marked --save                 
      - name: Build
        run: npm run build       
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }} 
          publish_dir: ./public
```
### GitHubPage设置
在Setting > Page > Source 选择 Deploy from a branch。
在 Branch 选择存放页面文件的 gh-pages 分支。
接下来可以通过 yourgithubname.github.io 访问你的网站。

## 至此Hexo的Github部署基本完毕
以下是可能会遇到的问题

### GitHub页面打不开
安装[Watt Toolkit](https://steampp.net/)
使用它的GitHub加速

### 我使用魔法但文件上传失败
修改代理设置，在本地仓库根目录找到.gitconfig文件进行设置
```bash
[user]
	name = 你的名字
	email = 你的邮箱@gmail.com
[gui]
	recentrepo = 你本地仓库的绝对路径
[http]
	sslVerify = true
    # 在此添加代理 ，注释则删除代理
	proxy = http://127.0.0.1:1080   
[https]
    # 在下面添加 https 代理，删除则删除代理
	proxy = socks5://127.0.0.1:1080 
# proxy 的值视代理软件而定
```


## 自定义域名
首先你需要一个域名。没有可以去阿里云，腾讯云之类的买，第一年一般都很便宜。
### github添加域
找到你的个人信息页面 > Pages > Add a domains 添加域
在你域名的解析页面设置解析
等待github通过验证
### page设置域
在仓库设置页面自定义域选项填入你的域名
在你的域名解析页面添加类型为CNAME的记录
最后结果看起来会是这样
|主机记录|记录类型|记录值|
|------|------|------|
|@|CNAME|githubname.github.io|
|_github-pages-challenge-githubname|TXT|dd1a23fdfdfa2dda2dfd|
:::default no-icon
记录为@时无需前缀，使用域名访问
:::
#### 域名不能访问了
在仓库的sources文件夹添加一个名为CNAME的文件，内容填你的域名。注意文件名不能有后缀。
github自定义域名的操作会在gh-page生成一个CNAME文件，每次action构建页面会覆盖掉这个文件。

#### 访问域名提示不安全
等待github ssl安全证书签发完毕。

## DNS加速
由于某个大型防火墙存在，github访问需要一些设置才能访问。
为了方便访问，可以使用DNS加速。
Cloudflare提供免费的CDN加速。
添加你的域名,然后添加解析。
|主机记录|记录类型|记录值|
|------|------|------|
|@|CNAME|githubname.github.io|





