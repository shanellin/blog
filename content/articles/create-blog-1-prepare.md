---
title: "Hugo 自架部落格（一）- 準備"
categories: ["Frontend"]
tags: ["Hugo"]
date: 2023-08-16T15:41:08+08:00
draft: false
---

## 前言
---

筆者日常開發的作業系統是 MacOS，因此這一系列都會以 MacOS 為出發點，日後若有機會再慢慢補上 Windows 相關內容。

## 開發環境
---

### 一、Git

尚未安裝 Git 可以到 {{<NewTabLink href="https://w3c.hexschool.com/git/fd6f6be" title="在 Mac 上安裝 Git 流程">}} 依照指引安裝。

到 {{<NewTabLink href="https://github.com/" title="官網">}} 登陸/註冊，並創建一個 repo。

### 二、Homebrew (macOS or Linux)

尚未安裝 Homebrew 可以到 {{<NewTabLink href="https://brew.sh/index_zh-tw" title="官網">}} 依照指引安裝。

### 三、Hugo

接著按照以下指令安裝 Hugo：

```shell
brew install hugo
```

確認 Hugo 可正常運行：

```shell
hugo version

# Output:
# hugo v0.117.0-b2f0696cad918fb61420a6aff173eb36662b406e+extended darwin/amd64 BuildDate=2023-08-07T12:49:48Z VendorInfo=brew
```

### 四、VS Code

尚未安裝 VS Code 可以到 {{<NewTabLink href="https://code.visualstudio.com/" title="官網">}} 依照指引安裝。

左側 Extensions 可下載 VS Code 外掛：

{{<NewTabLink href="https://marketplace.visualstudio.com/items?itemName=shd101wyy.markdown-preview-enhanced" title="Markdown Preview Enhanced">}} - 預覽 Markdown 渲染結果

## 建置 Hugo 專案
---

### 一、開新專案

```shell
hugo new site [project name]

# [project name] 可以替換成你想要的專案名稱
```

注意副檔名 `.md` 是必要的。

### 二、佈景主題

首先到 {{<NewTabLink href="https://themes.gohugo.io/" title="官網">}} 挑選心儀的主題，這邊以 {{<NewTabLink href="https://themes.gohugo.io/themes/hugo-theme-m10c/" title="m10c">}} 為例：

```shell
git init
git clone https://github.com/vaga/hugo-theme-m10c.git themes/m10c
```

然後將以下加到主目錄下的 `hugo.toml` 或 `config.toml`（根據 Hugo 版本有所差異）

```toml
theme = "m10c"
```

接著參照 `./themes/m10c/exampleSite/config.toml` 設置你需要的資訊，如：author、description...等。

### 三、建立文章

```shell
hugo new [setion name]/[article name]

# e.g. hugo new articles/create-blog-1-prepare.md
# 在 content 資料夾下新增 articles 資料夾
# 在 articles 資料夾下新增 create-blog-1-prepare.md
```

### 四、運行專案

```shell
# 訪問所有文章，包括草稿（draft: true）。
hugo server -D

# 訪問所有文章，不包括草稿（draft: true）。
hugo server
```

訪問 {{<NewTabLink href="http://localhost:1313/" title="http://localhost:1313/">}}

{{<NextArticle href="/articles/create-blog-2-structure" article="Hugo 自架部落格（二）- 項目結構">}}