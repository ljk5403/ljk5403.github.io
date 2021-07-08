---
title: 基于Github的博客和笔记系统
date: 2021-07-07 20:21:11
categories: Programming
tags:
---

 <!-- more -->

# 基于Github的博客和笔记系统

打算把正儿八经的笔记集体迁移到文件系统中，结合GitHub备份和在线浏览。同时也方便把好的笔记直接转换为博客。

为此，建立了一个私有仓库Notes，博客依旧放在GitHub Pages。

## 总体思路

使用 GitHub Workflow 自动从Notes构建Blog：

1. 监听资源仓库的commit，推送到blog构建区域（blog仓库的分支：blog_factory）
2. 构建blog，推送到GitHub Pages（即log仓库的分支：master）

## 实现

### 本地整理 Blog

#### blog存放的原则：

由笔记直接整理出的，应当软链接回笔记。变动较大的，也应该在笔记版本上注明存在blog版本。

#### 添加blog头：

使用 `./header_blog.sh $filename` 来快捷添加blog头，并启动vim来编辑。

```sh
header_blog.sh
filename="$*"
if [ ! -n "$filename" ]
then
  echo "File not found!"
  exit 1
fi
echo "---
title: ${filename%.*}
date: $(date '+%Y-%m-%d %H:%M:%S')
categories:
tags:
---

 <!-- more -->

$(cat "$filename")
" > "$filename"
vim "$filename"
```

#### 移动文件到相应文件夹

将md文件移动到`_posts`文件夹中，图片等资源文件移动到`image`文件夹中，并更新md文件中的图片等链接。

```sh
# mover.sh
file="$*"
if [ -f "$file" ] && [ -n "$file" ]
then
  if [ -d "${file%.*}.src" ]
  then
    sed -e "s/\!\[\](/\!\[\](\/image\//g" "$file" >"$file.temp"
    mv "$file" "${file%.*}.source.md"
    mv "$file.temp" "$file"
    mv "${file%.*}.src" image/
  fi
  mv "$file" _posts/
else
  echo "File not found!"
fi
```

关于图片，参考了<https://yanyinhong.github.io/2017/05/02/How-to-insert-image-in-hexo-post/>

### Blog 仓库推送到 blog_factory

```yml
# Notes/.github/workflows/sendBlog.yml
name: Send Blog
on:
  push:
    paths:
      - BlogSource/**

jobs:
  push-blog:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Notes repo
        uses: actions/checkout@v2

      - name: Update Blog Factory
        env:
          UPDATE_SECRET: ${{ secrets.BLOG_UPDATE_KEY }}
          GIT_USER: #你的GitHub用户名
          GIT_EMAIL: #你的GitHub邮箱
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          mkdir -p ~/.ssh
          echo "$UPDATE_SECRET" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git clone "GitHub Pages 仓库地址" --single-branch --branch blog_factory blog_factory --depth=1
          rsync -av --delete BlogSource/_posts blog_factory/source
          rsync -av --delete BlogSource/image blog_factory/source
          cd blog_factory
          git add --all
          if [ -n "`git status -s`" ]
          then
            git status
            git config --global user.name $GIT_USER
            git config --global user.email $GIT_EMAIL
            git commit -m "Automatically pushed from Notes: $(date '+%Y-%m-%d %H:%M:%S')"
            git push
          else
            echo "Nothing to Commit"
          fi
```

### blog_factory 自动构建并推送到GitHub Pages

```yml
# blog_local/.github/workflows/blogBuilder.yml
name: Hexo Deploy
on:
  push:
    branches:
      - blog_factory
    paths:
      - source/**

env:
  GIT_USER: #你的GitHub用户名
  GIT_EMAIL: #你的GitHub邮箱

jobs:
  build-blog:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source repo
        uses: actions/checkout@v2
        with:
          ref: blog_factory

      - name: Set up Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '14'

            # 安装依赖
      - name: Install Dependencies
        run: |
          npm install

          # 从之前设置的 secret 获取部署私钥
      - name: Setup SSH Keys and known_hosts
        env:
          DEPLOY_SECRET: ${{ secrets.BLOG_DEPLOY_SECRET }}
        run: |
          sudo timedatectl set-timezone "Asia/Shanghai"
          mkdir -p ~/.ssh
          echo "$DEPLOY_SECRET" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan github.com >> ~/.ssh/known_hosts
          git config --global user.name $GIT_USER
          git config --global user.email $GIT_EMAIL

          # 生成并部署 `npx hexo clean && npx hexo g -d` or `npm run deploy`
      - name: Deploy
        run: |
          npx hexo clean && npx hexo g -d
```

Tips:

1. 不要在`source/_posts`中放其他类型的文件，可能导致无法`hexo g`。（笔者经历：两个sh文件导致无法读取整个文件夹）

参考：

1. <https://note.junyangz.com/2019/GitHub-Actions-depoly-Hexo/>
2. <https://umm.js.org/p/3d7401da/>

