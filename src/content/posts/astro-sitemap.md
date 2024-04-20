---
title: Astro Sitemap 設定
published: 2024-04-20
description: 在 Astro 中設定網站地圖 (Sitemap)
image: 'https://media.virtualxnews.com/2024/04/7f0cc24970e45a7142f633d86c12b488.png'
tags: [Astro, Sitemap, 網站地圖]
category: 前端
draft: false
---

# Astro Sitemap 設定

## 安裝

Astro 包含一個 `astro add` 指令，用來自動設定官方整合。也可以選擇[手動安裝](https://docs.astro.build/zh-cn/guides/integrations-guide/sitemap/#%E6%89%8B%E5%8A%A8%E5%AE%89%E8%A3%85)整合。

在新的終端機視窗中執行以下其中一個指令：(這邊使用自動設定)

```
pnpm astro add sitemap
```

## 使用

`@astrojs/sitemap` 需要網站 URL 來生成網站地圖。在 `astro.config.*` 中使用 `site` 屬性加入你的網站 URL。這個 URL 必須以 `http:` 或 `https:` 開頭。

**astro.config.mjs**

```javascript
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  // ...
  site: 'https://your-domain.com',
  integrations: [sitemap()],
});
```

使用 `astro build` 指令來建構正式網站。會在 `dist/` 目錄（或你所設定的自訂建構目錄）中找到兩個檔案：`sitemap-index.xml` 和 `sitemap-0.xml`。

驗證網站地圖是否已生成後，你可以將它們加入網站的 `<head>` 標籤和爬蟲讀取的 `robots.txt` 檔案中。

**src/layouts/Layout.astro**

```html
<head>
  <link rel="sitemap" href="/sitemap-index.xml" />
</head>
```

**public/robots.txt**

```
User-agent: *
Allow: /

Sitemap: https://<YOUR SITE>/sitemap-index.xml
```

若要動態生成 `robots.txt`，請新增一個名為 `robots.txt.ts` 的檔案，並加入以下程式碼：

**src/pages/robots.txt.ts**

```javascript
import type { APIRoute } from 'astro';

const robotsTxt = `
User-agent: *
Allow: /
Sitemap: ${new URL('sitemap-index.xml', import.meta.env.SITE).href}
`.trim();

export const GET: APIRoute = () => {
  return new Response(robotsTxt, {
    headers: {
      'Content-Type': 'text/plain; charset=utf-8',
    },
  });
};
```

## 包含兩個頁面的網站生成檔案範例

**sitemap-index.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <sitemap>
    <loc>https://your-domain.com/sitemap-0.xml</loc>
  </sitemap>
</sitemapindex>
```

**sitemap-0.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:news="http://www.google.com/schemas/sitemap-news/0.9" xmlns:xhtml="http://www.w3.org/1999/xhtml" xmlns:image="http://www.google.com/schemas/sitemap-image/1.1" xmlns:video="http://www.google.com/schemas/sitemap-video/1.1">
  <url>
    <loc>https://your-domain.com/</loc>
  </url>
  <url>
    <loc>https://your-domain.com/second-page/</loc>
  </url>
</urlset>
```

> 參考資料：
> 
> [@astrojs/sitemap](https://docs.astro.build/zh-cn/guides/integrations-guide/sitemap/)
> 
> [CSS-tricks.com](https://css-tricks.com/wp-content/uploads/2021/05/astro-homepage.png)