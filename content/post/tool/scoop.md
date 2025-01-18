+++
date = '2025-01-03T00:28:59+08:00'
draft = true
title = 'Scoop'
+++

# Scoop
## 为什么使用 Scoop
在 windows 下对软件进行统一的管理。
通过命令行一键安装常用开发软件。

## 安装 Scoop
scoop 需要 PowerShell 5.1 或更高版本作为运行环境。
```bash
# 设置 PowerShell 执行策略
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# 下载安装脚本
irm get.scoop.sh -outfile 'install.ps1'
# 执行安装脚本
.\install.ps1 -ScoopDir 'D:\Scoop'
```

## 配置代理
