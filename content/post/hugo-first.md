---
title: "使用 Hugo 在 Github Pages 建立靜態網站"
date: 2021-03-30T21:19:31+08:00
draft: false

categories:
  - hugo
tags:
  - hugo

---

## Hugo 簡介

Hugo 是使用Go語言編寫的靜態網頁生成器，基於Go語言開發，沒有像HTML5錯綜複雜的依賴關係。除此之外，還擁有活躍的社群提供多樣主題與完善的功能給使用者，支援多種類的前端語言，整體架構也靈活易懂，最大優點是編譯速度極快，建立網站時間平均不到1秒種，使用者修改時能即時預覽。

#### Hugo 安裝

* MacOS & Linux

MacOS使用者可使用Homebrew安裝

```shell
brew install hugo
```

* Window

Window使用者須視使用者環境下載適用的版本 https://github.com/gohugoio/hugo/releases
或者透過`chocolatey`安裝
```shell
choco install hugo -confirm
```
安裝好後再去設定環境變數，例如: `C:\hugo`，記得將`hugo.exe`放進此路徑資料夾中

#### Hugo 指令測試
安裝好後透過CMD輸入`hugo version`可查看版本
```shell
hugo v0.81.0-59D15C97 linux/amd64 BuildDate=2021-02-19T17:07:12Z VendorInfo=gohugoio
```

## 建立 Project
透過CMD輸入
```shell
hugo new site mysite
```
則會在此路徑下建立`mysite`的資料夾，`mysite`可換任意名稱
資料夾結構如下:
```shell
mysite/
│
├── archetypes/
│ │
│ └─ default.md  # 預設 markdown
├── content/     # 頁面、文章（markdown）
├── data/        # 資料庫
├── layouts/     # 自定義的樣板
├── static/      # 靜態資源
├── themes/      # 網站的主題
└── config.toml  # 網站的配置
```

#### 加入Hugo主題
Hugo預設是沒有任何主題，因此需到 https://themes.gohugo.io/
下載喜歡的主題，我是使用 [Erblog](https://github.com/ertuil/erblog)

記得需要先用git對初始化
```shell
git init
```

接著將主題下載到`mysite/theme/`資料夾下，最後只需要將`mysite/theme/`中的`config.toml`取代`mysite/`中的就可以了，詳細使用`config.toml`須看者主題下方的說明!

#### 新增頁面
透過以下指令可以建立基本頁面
```shell
hugo new post/hello-hugo.md
```
此指令會在`mysite/content/`中建立`post`資料夾，並建立`hello-hugo.md`，可以透過`Markdown`語法來撰寫頁面

#### 本地端測試
預覽草稿
```shell
hugo server -D
```

預覽無草稿
```shell
hugo server
```

預覽路徑 `http://localhost:1313/`

#### 發佈到Github Pages上
需要有安裝`git`、`Github帳號`

首先建立兩個repository
一個任意名字，另一個為 :`帳號名稱.github.io`
設定`config.toml`中的baseURL為`http://帳號.github.io/`

透過輸入
```shell
hugo
```
建立`public`資料夾，並將`mysite`發佈到任意名字的repository，將`public`資料夾中的發佈到`帳號名稱.github.io`

發布流程
```shell
git init
git add .
git commit -m "mysite"
git push origin master

cd public
git init
git remote add origin https://github.com/帳號/帳號.github.io.git
git add .
git commit -m "mysite"
git push origin master
```

發布完後，在網址列輸入`http://帳號.github.io`即可看到自己架設的靜態網頁