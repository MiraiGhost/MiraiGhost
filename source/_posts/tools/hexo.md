---
title: 从零开始的Hexo
date: 2023-03-10 15:11:16
tags: Blog,Hexo
---
# 关于本文
本文从零开始教你怎么用 Hexo 生成个人博客，使用 GitHub Action 自动部署，通过 GithubPage访问。

## 环境准备
安装 Nodejs, Git,VSCode。
需要注册 GitHub, Cloudflare 账号

## Hexo 安装
在VSCode里打开你想存放Blog的文件夹。
通过Ctrl+Shift+`快捷键唤出终端。
```bash
npm install -g hexo-cli
```

### Hexo 初始化
确保终端路径是你想要的
运行后将会在你指定的文件夹生成文件
```bash
npm install 
hexo initial
```
这时Hexo已安装完毕。

### Hexo 基本命令
```bash
Hexo new --path A/B "文章名"
```
在路径 "./A/B" 下生成 "文章名.md"文件

```bash
Hexo server
```
通过md文件生成页面，并启动本地服务器。可通过浏览器查看页面。

```bash
Hexo generate
```
生成blog的静态页面

## 上传到 GitHub
在VSCode中登陆你的GitHub账户。
打开源代码管理功能,初始化仓库,设置名称为你的GitHub用户名。
开启同步。
这一步可能会遇到一些问题，但没关系。

### 可能的问题
由于墙的的存在大多数问题都是因为和GitHub服务器的通信问题。
这里可以使用Watt Toolkit的GitHub加速功能解决。

魔法失灵了怎么办
通常是代理设置出现了问题
在您的本地仓库文件夹通常能找到.gitconfig文件
```bash
[user]
	name = 你的名字
	email = 你的邮箱@gmail.com
[gui]
	recentrepo = 你仓库的路径
[http]
	sslVerify = true #开了更安全，不开可能会出点小问题
	proxy = http://127.0.0.1:10802 #根据你的魔法设置而定
[https]
	proxy = socks5://127.0.0.1:10801 #根据你的魔法设置而定

```



## 页面自动生成
Github Action
此时打开Github，你会发现多了一个和你账户名一样的仓库。

在仓库的.github文件夹新建workflows文件夹，
并在此新建一个后缀为 .yml 的文件，名称随意。
例如 blogAction.yml
```bash
name: Pages #名称随意

on:
  push:
    branches:
      - master  # 你放blog的仓库分支，通常是master
jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write  # 不是很懂，应该是检测到对仓库write操作执行
    steps:
      - uses: actions/checkout@v3
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 19.x        # 这里和你本地的Node.js版本一致。
        uses: actions/setup-node@v2   #  V    在终端输入 Node.js --version 即可查看版本
        with:                         #  V   大多数软件查看版本都是 软件名 --version
          node-version: '19'          # 这里也一样
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies                 
        run: npm install                            # 我是不是在哪见过你
    #   - name: unInstall markdown-it               # 如果你的blog安装了额外的插件，也需要在这里添加             
    #     run: npm un hexo-renderer-marked --save                 
      - name: Build
        run: npm run build                          # 这里 Hexo generate也行，可别忘了哦
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3         
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }} # 生成的public文件会被放在分支 gh-pages 里
          publish_dir: ./public
```
github会自动执行.github/workflows/name.yml 文件,并执行GitHubAction操作。
可以在 Action 查看过程和结果。

## 设置 GitHubPage
页面已经在 gh-pages 分支躺好了接下来就是显示你的Blog。
在Setting > Page > Source 选择 Deploy from a branch。
在 Branch 选择存放页面文件的 gh-pages 分支。

## 结束
此时，你的博客已部署完毕。
通过 yourgithubname.github.io 就可以访问。

### 个人页访问链接不好看怎么办
首先你需要一个域名。
然后找到你的个人信息页面 > Pages > Add a domains
填入你的域, 下一步, github会让你进行解析设置。
打开你购买域名的平台打开控制台进入云解析,新建解析选择你的解析类型为 ANAME,在两输入框中依次填入在上一步中github给你的两个值。第一个值别把你的域名填进去了。例 dfasfd21334github.你的域名。
在你仓库的page设置设置自定义域为你的域名。
保存后github会自动为你签发网站证书。

### 我不能通过我的域访问我的博客
那是因为在github设置自定义域名会在你页面文件位置(分支 gh-page)生成CNAME文件，GitHub在进行Action操作会覆盖掉gh-page里的文件。
Hexo构建时会通过source文件夹里的模板生成页面。

在你的Blog/source文件夹下添加一个无后缀名为CNAME的文件。
文件内容为你的域名。

### 不会魔法的人无法访问我的博客怎么办
Cloudflare提供免费的CDN加速。
在控制台添加你的域名,然后添加CNAME域名解析
第一个值: 你的域名前缀，如果你没有使用前缀填入@就行
第二个值: github提供给你的访问链接
保存即可




