---
title: Pixel 使用 GTM 追蹤電子商務網站使用者行為
published: 2024-04-12
description: 如何在 Google Tag Manager 中埋設追蹤碼 Pixel，以追蹤使用者的行為，並提供相應的程式碼範例和步驟。通過這些追蹤和分析，電子商務業務可以更好地了解使用者需求和行為，提供更好的購物體驗，並增加銷售機會。
image: 'https://media.virtualxnews.com/2024/04/fd8f06fa24dbcac3151d12b15eb25414.jpg'
tags: [電子商務, Pixel, Google Tag Manager, Meta]
category: 網站追蹤
draft: false 
---


# Pixel 在電子商務網站中追蹤和分析使用者行為

在電子商務網站中，了解使用者在購買過程中的行為是非常重要的。為了追蹤和分析這些行為，我們可以使用追蹤碼 Pixel 在網站中埋設不同的事件。本指南將介紹如何在 Google Tag Manager (GTM) 中埋設這些事件的相關程式碼，以便追蹤和分析使用者行為。

以下是一些常見的電子商務網站按照銷售漏斗所需的事件，以及如何在 GTM 中埋設它們：

## 1. ViewContent (瀏覽內容)

當使用者瀏覽某個商品頁面時，我們可以觸發 ViewContent 事件。這表示使用者正在查看特定商品的詳細資訊。

在 GTM 中，我們可以使用以下程式碼來埋設 ViewContent 事件：

```html
<script>
  fbq('track', 'ViewContent', {
    content_ids: ['商品編號'],
    content_name: '商品名稱',
    content_category: '商品類別',
    // 其他自訂參數
  });
</script>
```

## 2. Search (搜尋)

當使用者在網站上進行搜尋時，我們可以觸發 Search 事件。這表示使用者正在尋找特定的商品或關鍵字。

在 GTM 中，我們可以使用以下程式碼來埋設 Search 事件：

```html
<script>
  fbq('track', 'Search', {
    search_string: '搜尋關鍵字',
    // 其他自訂參數
  });
</script>
```

## 3. Lead (潛在顧客)

當使用者填寫表單或提供聯絡資訊時，我們可以觸發 Lead 事件。這表示使用者對我們的產品或服務表達了興趣。

在 GTM 中，我們可以使用以下程式碼來埋設 Lead 事件：

```html
<script>
  fbq('track', 'Lead', {
    lead_name: '潛在顧客姓名',
    lead_email: '潛在顧客電子郵件',
    // 其他自訂參數
  });
</script>
```

## 4. AddToWishlist (加到願望清單)

當使用者將商品加入願望清單時，我們可以觸發 AddToWishlist 事件。這表示使用者對該商品有興趣，但尚未準備購買。

在 GTM 中，我們可以使用以下程式碼來埋設 AddToWishlist 事件：

```html
<script>
  fbq('track', 'AddToWishlist', {
    content_ids: ['商品編號'],
    content_name: '商品名稱',
    content_category: '商品類別',
    // 其他自訂參數
  });
</script>
```

## 5. AddToCart (加到購物車)

當使用者將商品加入購物車時，我們可以觸發 AddToCart 事件。這表示使用者已經準備購買該商品。

在 GTM 中，我們可以使用以下程式碼來埋設 AddToCart 事件：

```html
<script>
  fbq('track', 'AddToCart', {
    content_ids: ['商品編號'],
    content_name: '商品名稱',
    content_category: '商品類別',
    currency: '貨幣代碼',
    value: '商品價值',
    // 其他自訂參數
  });
</script>
```

## 6. InitiateCheckout (開始結帳)

當使用者開始結帳流程時，我們可以觸發 InitiateCheckout 事件。這表示使用者已經選擇了要購買的商品並開始進行結帳。

在 GTM 中，我們可以使用以下程式碼來埋設 InitiateCheckout 事件：

```html
<script>
  fbq('track', 'InitiateCheckout', {
    content_ids: ['商品編號1', '商品編號2', ...],
    content_name: ['商品名稱1', '商品名稱2', ...],
    content_category: ['商品類別1', '商品類別2', ...],
    currency: '貨幣代碼',
    value: '總價值',
    // 其他自訂參數
  });
</script>
```

## 7. AddPaymentInfo (新增付款資料)

當使用者在結帳過程中輸入付款資料時，我們可以觸發 AddPaymentInfo 事件。這表示使用者已經提供了付款資訊。

在 GTM 中，我們可以使用以下程式碼來埋設 AddPaymentInfo 事件：

```html
<script>
  fbq('track', 'AddPaymentInfo', {
    currency: '貨幣代碼',
    value: '總價值',
    // 其他自訂參數
  });
</script>
```

## 8. CompleteRegistration (完成註冊)

當使用者完成註冊流程時，我們可以觸發 CompleteRegistration 事件。這表示使用者已經成功註冊並成為我們的會員。

在 GTM 中，我們可以使用以下程式碼來埋設 CompleteRegistration 事件：

```html
<script>
  fbq('track', 'CompleteRegistration', {
    user_id: '使用者ID',
    // 其他自訂參數
  });
</script>
```

## 9. Purchase (購買)

當使用者成功完成購買時，我們可以觸發 Purchase 事件。這表示使用者已經購買了商品。

在 GTM 中，我們可以使用以下程式碼來埋設 Purchase 事件：

```html
<script>
  fbq('track', 'Purchase', {
    content_ids: ['商品編號1', '商品編號2', ...],
    content_name: ['商品名稱1', '商品名稱2', ...],
    content_category: ['商品類別1', '商品類別2', ...],
    currency: '貨幣代碼',
    value: '總價值',
    // 其他自訂參數
  });
</script>
```

藉由埋設這些事件，我們可以追蹤使用者在電子商務網站上的行為，並進一步分析和優化銷售策略。請根據你的需求和網站設計，選擇適合的事件並在 GTM 中埋設它們。

使用 Google Tag Manager 可以方便地管理和埋設追蹤碼 Pixel，並且不需要直接修改網站的原始碼。透過建立觸發器、標籤和關聯它們，可以有效地追蹤使用者在網站上的行為。藉由這個範例，可以在 GTM 中埋設追蹤碼 Pixel，並開始了解使用者的行為，優化網站並提升銷售機會。
