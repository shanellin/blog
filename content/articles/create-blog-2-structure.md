---
title: "Hugo 自架部落格（二）- 項目結構"
categories: ["Frontend"]
tags: ["Hugo"]
date: 2023-08-19T14:25:48+08:00
draft: false
---

## 前言

---

若對 MarkDown 語法還不熟悉，可以參考 HackMD 提供的 {{<NewTabLink href="https://hackmd.io/@eMP9zQQ0Qt6I8Uqp2Vqy6w/SyiOheL5N/%2FBVqowKshRH246Q7UDyodFA?type=book" title="MarkDown語法大全">}}。

## 結構介紹

---

使用 `hugo new site [project name]` 新增專案後，會在專案內部生成以下項目結構：

```shell
.
├── archetypes/
│   └── default.md
├── assets
├── content
├── data
├── hugo.toml
├── layouts
├── static
└── themes
```

### ㄧ、archetypes

存放文章模板，預設有 `archetypes/default.md` 作為初始模板，新增文章時會自動套用。

```toml
# default.md
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
---
```

但若有對應文章資料夾名稱的模板時會優先套用該模板。

### 二、assets

若有使用 {{<NewTabLink href="https://gohugo.io/hugo-pipes/introduction/" title="Hugo Pipes">}}，需要將 Pipes 會用到的靜態資源都放到 assets 裏。

以 `m10c` 主題為例，若要覆蓋元件樣式，需要在 `assets` 新增 `css/_exrea.scss`。

{{<NewTabLink href="https://discourse.gohugo.io/t/difference-between-asset-and-static-folder/41203" title="Difference between asset and static folder?">}}

### 三、content

存放 `hugo new` 新增的文章，可搭配 `archetypes` 讓新文章自動套用各文章分類的模板。如以下：

首先加入 `archetypes/articles.md`，再新增對應模板分類名稱的文章

```shell
hugo new articles/create-blog-1-prepare.md
```

會生成：

```shell
.
├── content/
│   └── articles/
│       └── create-blog-1-prepare.md
```

因 `articles` 對應到 `articles.md` 模板名稱， 所以 `create-blog-1-prepare.md` 會自動套用 `articles.md` 模板。

### 四、data

根據 {{<NewTabLink href="https://www.youtube.com/watch?v=FyPgSuwIMWQ&t=69s" title="官方介紹">}} data 資料夾可以存放靜態資料，如：JSON、YAML、TOML。

可在模板直接讀取做輸出，進一步確保資料夾職能單一，不會有太多污染。

### 五、hugo.toml

```shell
舊版 Hugo 會生成 `config.toml`。
```

環境參數配置檔，用於設置網站建置參數、模板參數。

以此部落格為範例：

```toml
baseURL = '/'       # 網站網址，預設為絕對路徑，若 relativeURLs = true 則為相對路徑。
title = 'Shane Lin' # 網站預設標題。
theme = 'm10c'      # 指定樣式。
paginate = 10       # 每頁文章數，預設為 10。
relativeURLs = true # 將網站路徑改為相對路徑。

[taxonomies]        # 要啟用哪些文章分類。
  category = "categories"
  tag = "tags"

[params]            # 設置模板參數，非固定，需依照各模板需求做配置。
  author = "Shane Lin"
  ...

[menu]              # 設置模板 menu 參數。
  [[menu.main]]
    identifier = "home"
    name = "Home"
    url = "/"
    weight = 1
  ...
```

更多配置可參考 {{<NewTabLink href="https://gohugo.io/getting-started/configuration/#all-configuration-settings" title="Configure Hugo">}}

### 六、layouts

存放客製化佈景元件，在 `.md` 中無法直接使用 `HTML`，但可以根據 {{<NewTabLink href="https://gohugo.io/templates/shortcode-templates/" title="Create your own shortcodes">}} 在 `./layouts/shortcodes` 中加入屬於自己的元件。

在 Hugo 中，預設 MarkDown 連結語法會導航本站到目標網站：

```md
[連結名稱](https://google.com "游標顯示")
```

這時就可以透過 shortcodes 做出開新分頁的連結元件（HyperLink.html）：

```shell
.
├── layouts
│   └── shortcodes # shortcodes 為約定資料夾名稱，不可異動。
│       └── HyperLink.html

```

```md
// HyperLink.html
<a href="{{ .Get "href" }}" rel="noopener" target="\_blank">{{ .Get "title" }}</a>
```

在 `.md` 中使用：

```md
// 要去掉 {{ 和 }} 內的頭尾空白。
{{ <HyperLink href="https://google.com" title="Google"> }}
```

### 七、static

存放靜態資源檔案，如：圖片、js、css 等。

### 八、themes

存放各種佈景主題，可透過 `hugo.toml` 或 `config.toml` 的 `theme` 切換主題，以 {{<NewTabLink href="https://shanelin-blog.com/articles/create-blog-1-prepare/" title="Hugo 自架部落格（一）- 準備">}} 為例，將會看到：

```shell
.
├── themes
│   └── m10c
```