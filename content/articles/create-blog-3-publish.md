---
title: "Hugo 自架部落格（三）- 搭配 Godaddy 部署到 Cloudflare Pages"
categories: ["Frontend"]
tags: ["Hugo"]
date: 2023-08-20T10:48:41+08:00
draft: true
---

## 前言
---

雖然有想過 Github、Cloudflare 都有提供自訂義域名的功能，是不是省事點直接撿現成的就好。但轉念一想都自架部落格了，買一個屬於自己的域名，當作給自己認真向上的藉口吧！

事前準備時偶然發現 Cloudflare 居然有提供靜態網頁託管服務 {{<NewTabLink href="https://pages.cloudflare.com/" title="Cloudflare Pages">}}，高速、安全、HTTPS 加密、性能指標與分析、提前預覽，重要的是以上優點免費方案通通都有，甭廢話，就你了。

```shell
雖然有額度上限，但對於部落格而言，綽綽有餘。
```

因此最終決定以 Hugo + Godaddy + CloudFlare pages 作為出發的裝備。

## Cloudflare Pages
---

首先創建一個新的 {{<NewTabLink href="https://dash.cloudflare.com/sign-up/workers-and-pages" title="Cloudflare帳戶">}} 或登錄到您現有的帳戶。

### 一、Wrangler

到項目底下執行：
```shell
yarn add wrangler
```

確保 Wrangler 已有正確安裝：
```shell
npx wrangler --version
```

使用以下命令登錄 Wrangler：
```shell
npx wrangler login
```

### 二、創建＆部署

使用 Hugo 生成生產用的靜態生成生產用的靜態網頁資源：
```shell
hugo

# 僅會包含 draft: false 的文章
```

創建 cloudflare pages 遠端項目：
```shell
npx wrangler pages project create

# Enter the name of your new project: [輸入你要的 cloudflare pages 項目名稱，會影響 domain]
# Enter the production branch name: [production]
```

將生產用的靜態網頁資源推到 cloudflare pages 遠端項目：
```shell
npx wrangler pages deploy ./public

# wrangler pages deploy <OUTPUT_DIRECTORY> --branch=<BRANCH_NAME>
# <BRANCH_NAME> 可用來決定是 preview 還是 production。
```

### 三、查看結果

進到 {{<NewTabLink href="https://dash.cloudflare.com/" title="Dashboard">}} -> `Workers & Pages` -> `Overview`，找到剛剛創建的項目，點擊紅框的 `Visit site`。

![cloudflare-overview](/images/cloudflare/cloudflare-overview.png)

## Godaddy

### ㄧ、購買域名

到 {{<NewTabLink href="https://tw.godaddy.com/" title="官網">}} 挑一個域名，可以參考 {{<NewTabLink href="https://www.wfublog.com/2014/04/how-to-choose-a-domain-name-sop.html?m=1" title="網址/網域 命名的要點 + 流程SOP(筆記)">}} 構思一下自己的專屬域名。（買完才想到要注意這個...

### 二、設定 DNS 託管

進入剛創建的 cloudflare 項目，點擊紅框的 `Custom domains`。

![cloudflare-custom-domain](/images/cloudflare/cloudflare-custom-domain.png)

點擊 `Set up a custom doamin` 並輸入你在 Godaddy 購買的域名，點擊 `Continue`，接著 `Activate domain`。

![cloudflare-custom-1](/images/cloudflare/cloudflare-custom-1.png)

自動跳轉到 `nameserver setup` 頁面，這邊要注意若尚未在 cloudflare 建立域名方案，這頁會自動跳轉到選擇方案頁面，直接選 `Free` 即可。

![cloudflare-free](/images/cloudflare/cloudflare-free.png)

方案建立後會進到 `Review your DNS records` 直接點擊 `Continue`，回到 `nameserver setup` 頁面，下方會指示將 cloudflare 的 nameserver 替換掉 Godaddy 的 nameserver。

暫時不要點擊 `Done, check nameservers` 等變更完 Godaddy 網域設定後才點。

![cloudflare-nameserver](/images/cloudflare/cloudflare-nameserver.png)

### 三、更改 Godaddy 網域設定

登入 {{<NewTabLink href="https://dashboard.godaddy.com/venture/domain" title="Godaddy Dashboard">}}，點選左側 `網域`，找到剛購買的網域，點選 `網域設定`。

![godaddy-dns](/images/godaddy/godaddy-dns.png)

點擊 `DNS` 標籤 -> `名稱伺服器` -> `變更名稱伺服器`。

![godaddy-change](/images/godaddy/godaddy-change.png)

選擇 `我要用自己的名稱伺服器`，填入 cloudflare 的 nameserver，儲存。

![godaddy-my](/images/godaddy/godaddy-my.png)

### 四、查看結果

回到 `nameserver setup` 頁面，點擊 `Done, check nameservers`，接著會進到 `domain settings` 的 `Quick Start Guide`，將下方選項均開啟。

 - [x] Automatic HTTPS REWRITES ON and click on Save.
 - [x] Always use HTTPS ON and click on Save.
 - [x] Brotli ON and click on Save.

等待一段時間，nameservers 檢查成功後，cloudflare 會寄 email 通知，並可到 {{<NewTabLink href="https://dash.cloudflare.com/" title="Dashboard">}} -> `Websites` 確認域名已生效。

![cloudfront-website](/images/cloudflare/cloudfront-website.png)

最後回到 {{<NewTabLink href="https://dash.cloudflare.com/" title="Dashboard">}} -> `Workers & Pages` -> `Overview` -> `Custom domains`，把生效的域名再次填入，認證成功後即可用 Godaddy 的域名在分頁上看到自己的部落格了。





