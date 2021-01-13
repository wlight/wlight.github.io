# Hugo博客配置教程


### 创建博客所需 repo

假设你在 GitHub 上的用户名是 `abcd`，那么在 GitHub 上创建下面两个 repo：

- 一个叫 `blog`，用于存放 Hugo 博客的原始文件，包括博客配置，文章的 md 文件等。
- 另一个叫 `abcd.github.io`，用于存放 Hugo 编译生成的网页文件，访问博客时看到的就是网页文件。

记得在创建第二个 repo 的时候，至少选上 README、.gitignore、LICENSE 三个文件中的一个，这样创建出来的 repo 就不是空的。

如果不这样做，将第二个 repo 添加为第一个 repo 的子模块的操作就会失败，处理起来会很麻烦

## 配置源文件 repo

依次执行以下命令：

```bash
*# 将 repo clone 至本机 blog 文件夹中*
$ git clone <https://github.com/abcd/blog.git>
*# 用 blog 文件夹生成 Hugo 博客的初始内容*
$ hugo new site blog --force
*# 将博客主题添加为 blog 这个 repo 的子模块# 这样两者互不影响*
$ cd blog
$ git submodule add <https://github.com/varkai/hugo-theme-zozo> themes/zozo
echo 'theme = "zozo"' >> config.toml
```

## 本地测试博客效果

在 `content\\posts` 目录下新建几个后缀为 `.md` 的 Markdown 文件，随便写上一些内容。然后在命令行执行 `hugo server`，在浏览器中访问 `http://localhost:1313`，就能够看到博客效果了。

## 配置最终网页 repo

```bash
*# 将 public 文件夹与 repo abcd.github.io 相关联*
$ git submodule add -b main <https://github.com/abcd/abcd.github.io.git> public
```

编辑 Hugo 的配置文件 `config.toml`，将 `baseUrl` 字段的值设置为 `baseUrl = "<https://abcd.github.io/>"`

同时在主项目根目录的 `static` 文件夹中新建文件 `CNAME`，文件内容为 `abcd.github.io`。

这样一来，执行 `hugo -D` 所编译生成的最终网页 repo 中的所有页面及代码，相关的根链接就都被设置为 `https://abcd.github.io/`，这样可以保证 GitHub Pages 的功能会正常启用。

除此之外，还需要查看该 repo 的设置界面中，`GitHub Pages` 这一栏的 `Source`，所选的分支是不是 `main`，如果是 `Master`，还需要切换为 `main` 才行。

## 使用自动发布博客的脚本

```bash
***#!/bin/sh**
# If a command fails then the deploy stops*set -e

printf "\\033[0;32mDeploying updates to GitHub...\\033[0m\\n"

*# Build the project.# 使用指定的主题编译博客*
hugo -t zozo *# if using a theme, replace with `hugo -t <YOURTHEME>`# Go To Public folder*cd public

*# Add changes to git.*
git add .

*# Commit changes.*msg**=**"rebuilding site **$(**date**)**"
**if** **[** -n "$*" **]**; **then**msg**=**"$*"
**fi**
git commit -m "$msg"

*# Push source and build repos.# 2020-10-17：注意：现在 GitHub 的主分支已经改名为 main*
git push origin main
```

然后再执行下面的命令，来测试脚本是否可用

```bash
*# 首次执行，需为脚本开启对应权限*
$ chmod +x deploy.sh
*# 调用脚本，发布博客*
$ ./deploy.sh "首次用脚本自动发布博客"
```

## 使用自定义域名

如果不想用 GitHub 的二级域名，而是想用自己的域名，那就需要额外再做一些配置。

假设你购买了域名 `blog.com`，那么就要在你的域名提供商那里，给该域名添加一条 `CNAME` 记录，将二级域名 `www` 的记录值，设置为前面给第二个仓库设置的名称：`abcd.github.io`。

然后修改 Hugo 配置文件 `config.toml` 中 `baseUrl` 字段的值为 `www.blog.com`，同时修改 `static/CNAME` 文件的内容为 `www.blog.com`。

这样一来，当用户访问 `www.blog.com` 的时候，其实显示的就是 `abcd.github.io` 中的内容。这样不需要额外找服务器来存放博客文件，省事多了。

对了，GitHub 默认会为 GitHub Pages 启用 HTTPS，你可能还需要为你的域名开启 HTTPS。

### Git 子模块

### **1.添加子模块**

```bash
> git submodule add <url> <path>
```

- `url` 为子模块的路径(本文为博客`git`地址)， `path` 为子模块存储的目录路径，执行完需重新提交项目

### **2.克隆壳工程**

- 重新克隆完壳工程后，子模块并不会一同出现，需要进行初始化

  ```bash
  >git submodule init
  >git submodule update
  或者：
  >git submodule update --init --recursive
  ```

- 子模块更新，当子模块有了新的提交，如何更新本地项目呢？

  - 首先进入子模块,拉取更新

    ```bash
    >git pull
    ```

  - 返回项目根目录,重新提交项目

### **3.删除子模块**

处于项目根目录，全部执行完记得重新提交项目

- 将子模块从版本控制中移除

  ```bash
  >git rm --cached 子模块名称
  ```

- 删除子模块目录

  ```bash
  >rm -rf 子模块目录
  ```

- 删除.gitmodules需要删除子模块信息

  ```bash
  >vim .gitmodules
  ```

- 删除文件中的子模块信息

  ```bash
  >vim .git/config
  ```

- 删除模块下的子模块目录

  ```bash
  >rm -rf .git/module/子模块名称
  ```

  ### 引用链接


  [基于hugo快速搭建个人博客](https://hts0000.github.io/2020/02/基于hugo快速搭建个人博客/)

