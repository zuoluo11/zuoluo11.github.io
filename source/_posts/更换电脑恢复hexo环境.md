---
title: 更换电脑恢复hexo环境
date: 2020-03-28 10:34:38
tags:
categories:
- 生活
---




换了电脑环境，重新将blog从git配置到本地

### 安装node和git

```
Node.js安装包及源码下载地址：https://nodejs.org/en/download/
```

```
下载Git  官方地址为：https://git-scm.com/download/win
```

### 配置git关联Github

```
git config --global user.name "用户名" 
git config --global user.email "邮箱地址"
```

生成ssh秘钥

```
ssh-keygen -t rsa -C "邮箱地址"
```

将ssh秘钥添加到Github上

登录github，打开setting->SSH keys，点击右上角 New SSH key，把生成好的公钥`id_rsa.pub`放进 key输入框中，再为当前的key起一个title来区分每个key。

### 检出hexo

```
git clone 远程仓库地址
```

### 使用

进入文件夹，然后使用hexo的命令使用发布文章

**注意：本地的markdown编辑器使用图片时，设置typora图片保存路径为图片名文件夹下，建议语法如下**

```
{% asset_img image-20200328104323471.png 图片名称%}
```

