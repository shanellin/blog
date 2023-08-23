---
title: "Hugo 自架部落格（四）- SEO"
categories: ["Frontend"]
tags: ["Hugo"]
Keywords: ["seo", "favicon", "favicon not found", "cannot search blog", "robots", "sitemap", "搜尋不到網站"]
images: ["/images/avatars.png"]
date: 2023-08-23T19:32:29+08:00
draft: false
---

## 前言
---

SEO (Search Engine Optimizing)，主要影響網站在搜尋引擎的排序結果，好的內容固然重要，但若是不了解其運作機制，曝光率可能會大打折扣，甚至撰寫不當還會導致排名不進反退。

會想寫這篇是因為部落格剛上線就遇到 `網站沒出現在搜尋結果` 的問題，這無疑是前端仔最無法忍受的，沒出現不就等於白努利？所以筆者抱持著積極樂觀的心態深入研究，進而了解了 `Google 搜尋的運作方式`、`黑白帽 SEO`、`Open Graph`，接下來就說說這幾個面向吧。

## 網站沒出現在搜尋結果
---

上網查了下這可能是因為搜尋引擎尚未對其建立索引導致的，想想也對，每天要上線的網站何其多，總不可能自己的網站就有特權，但與其呆呆等，不如來了解 {{<NewTabLink href="https://developers.google.com/search/docs/fundamentals/how-search-works?visit_id=638282074426450036-488990237&rd=1&hl=zh-tw" title="Google 搜尋的運作方式">}} 沒準能加速進程。

![google-search](/images/seo/google-search.png)

### Google 搜尋三階段：

#### ㄧ、檢索

參考以上流程圖大概可以知道這階段是最耗時的，首先 Google 會不間斷地從網路上尋找新網站，找到後將它加進已知網站清單並持續跟進所有已知網站是否有更新內容，接著進入網站後 Google 會先轉譯網站上的資源，如：Javascript...等以完成最大內容渲染，也就是俗稱的 LCP（largest-contentful-paint），最後透過 GoogleBot（檢索器、爬蟲）擷取網頁內容。

> 注意此階段 GoogleBot 擷取前會先確認該網站 {{<NewTabLink href="https://developers.google.com/search/docs/crawling-indexing/robots/robots_txt?hl=zh-tw#disallow" title="robots.txt">}} 規則是否允許存取網頁。

#### 二、建立索引

此階段會解讀網頁內容，如：title、h1、image、video...等，自動為網站建立可用來供訪客搜尋的關鍵點，如：電商網站的商品圖會顯示在 google 圖片搜尋結果中，並可透過 `前往` 到達目標網站。

![google-image-example](/images/seo/google-image-example.png)

> 注意此階段除了會確認該網站 {{<NewTabLink href="https://developers.google.com/search/docs/crawling-indexing/block-indexing?hl=zh-tw" title="meta header">}} 是否允許建立索引外，還會比對資料庫中其他網站內容，審查網站內容品質。

#### 三、提供搜索結果

此階段為訪客在瀏覽器搜尋關鍵字時透過索引資料庫找出符合的網頁，再傳回與訪客的查詢內容最相關且品質最佳的結果。

### 解決方法：

可以發現除了 `檢索` 階段 Google 需要從網路上大海撈針外，其餘階段均是針對性地對網站進行 SEO 處理。所以我們的重點自然也就放在如何 `提早讓 Google 尋找到我們新架的網站`，好在 Google 對此有提供方案，就是 `主動提交 Sitemap`。

首先登入 {{<NewTabLink href="https://search.google.com/search-console/not-verified?original_url=/search-console/ownership&original_resource_id" title="Google Search Console">}}，點擊左上角 `新增資源` 輸入 {{<NewTabLink href="https://shanelin-blog.com/articles/create-blog-3-publish/" title="Hugo 自架部落格（三）- 搭配 Godaddy 部署到 Cloudflare Pages">}} 購買的網域，並驗證擁有權。

![google-verify-domain-name](/images/seo/google-verify-domain-name.png)

然後在左側菜單選擇 `Sitemap` 新增 Sitemap，一般來說 sitemap 會在 `https://<domain>/sitemap.xml`，點擊提交，之後 Google 會需要大約一天的時間建立索引。

![google-submit-sitemap](/images/seo/google-submit-sitemap.png)

大概一天後就可在瀏覽器用 `<domain>` 或 `site:<domain>` 搜尋到自己的網站了。

![google-search-blog](/images/seo/google-search-blog.png)
![site-search-blog](/images/seo/site-search-blog.png)

> 注意 favicon 應根據 {{<NewTabLink href="https://developers.google.com/search/docs/appearance/favicon-in-search?hl=zh-tw" title="定義網站小圖示">}} 設定成正方形，且邊長為 48 的倍數，如：48 x 48、96 x 96、144 x 144 等等。

## 黑白帽 SEO
---

搜尋引擎會根據網站內容進行排名，有好的 SEO，自然也有不好的，在網上這被稱為 `黑白帽SEO`，簡單來說 `黑帽SEO` 就是班級考試中企圖靠作弊取得靠前排名的學生。

### 以下為黑帽常見行為：

#### 關鍵字

1. 加入毫無相關的關鍵字到 keywords meta，如：在遊戲中加入所有遊戲分類，擾亂搜尋。
2. 在文章各角落不合理地塞滿重複關鍵字，如：title、keywords、description、h1、h2...等。

#### 文章內容

1. 標題、描述與內文無關。
2. 大量複製他人文章。

#### 連結

1. 購買第三方點擊連結服務。
2. 惡意購買或入侵知名網站，添加隱藏連結。
3. 劫持知名網站，重定向到特定網站。
4. 架設網站農場，重定向到特定網站。
5. 引入與內容無關的知名網站連結，想藉此提高權威分數，如：維基百科。

> 除了堅持禁用 `黑帽SEO` 外，根據 {{<NewTabLink href="https://developers.google.com/search/docs/fundamentals/seo-starter-guide?hl=zh-tw" title="搜尋引擎最佳化 (SEO) 入門指南">}} 優化網站內容才是提升排名的最佳途徑。

## Open Graph
---

Open Graph 全名是 Open Graph Protocol，主要用於在社交媒體平台上優化連結的預覽顯示。當你在社交媒體上分享連結，如：Facebook、Twitter、LinkedIn 等平台，這些平台會嘗試獲取連結中的元數據，並顯示一個包含標題、描述、圖像等信息的預覽卡片。

> `og:title` -> 預覽卡片的標題  
> `og:description` -> 預覽卡片的描述  
> `og:image` -> 預覽卡片的圖像  
> `og:url` -> 預覽卡片的網站連結

貼心的是 {{<NewTabLink href="https://gohugo.io/templates/internal/" title="Hugo官方">}} 有提供內置模板 `_internal/opengraph.html`、`_internal/twitter_cards.html`，會自動為我們引入 og 相關 meta 資源，且 {{<NewTabLink href="https://shanelin-blog.com/articles/create-blog-1-prepare/" title="Hugo 自架部落格（一）- 準備">}} 提及的 `m10c` 模板（`./themes/m10c/layouts/_default/baseof.html`）也已包含以下：

```toml
{{ template "_internal/twitter_cards.html" . }}
{{ template "_internal/opengraph.html" . }}
```

我們只需要透過 {{<NewTabLink href="https://www.opengraph.xyz/" title="Preview and Generate Open Graph Meta Tags">}} 輸入網站網址，就可以看到 og 預覽畫面：

![og-preview](/images/seo/og-preview.png)

但很快會發現 `預覽圖是空白的`，這是因為 Hugo 並不會自動擷取文章內任何圖片作為 `og:image`，我們需要手動在 `.md` 最上方 Front Matter 區塊加入：

```toml
images: ["xxx.png"]
```

以此篇文章為例，最後結果為：

```toml
title: "Hugo 自架部落格（四）- SEO"
categories: ["Frontend"]
tags: ["Hugo"]
Keywords: ["seo", "favicon", "favicon not found", "cannot search blog", "robots", "sitemap", "搜尋不到網站"]
images: ["/images/avatars.png"]
date: 2023-08-23T19:32:29+08:00
draft: true
```
