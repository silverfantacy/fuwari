---
title: 告別 Vuex，擁抱 Pinia：Vue 狀態管理無痛轉型攻略
published: 2024-08-28
description: '還在用 Vuex 管理狀態嗎？是時候轉換到更現代、更簡潔的 Pinia 了！Vuex 無痛轉移到 Pinia，比較兩者的差異、提供詳細的遷移步驟與注意事項，讓你享受 Pinia 帶來的開發體驗提升。'
image: ''
tags: [Pinia, Vuex, Vue 2, Vue 3, 狀態管理, 前端開發]
category: '前端'
draft: false 
---

最近在升級專案從 Vue 2 到 Vue 3 的過程中，也將狀態管理從 Vuex 轉換到 Pinia。Pinia 是專為 Vue 3 設計的狀態管理庫，比 Vuex 更簡潔、更現代，能讓程式碼更易於維護。本文將比較 Vuex 和 Pinia 的差異，並提供一些遷移步驟與注意事項，幫助順利完成轉換，避免常見的問題。

## Vuex vs. Pinia

轉型前，先來看看 Pinia 到底贏在哪裡：

| 特性        | Vuex                                  | Pinia                                |
| ----------- | -------------------------------------- | -------------------------------------- |
| API 簡潔度  | 樣板程式碼多到爆，新手看了想哭      | 直覺好懂，程式碼少一半，開發效率 UP！    |
| Vue 3 整合 | Vue 2 時代產物，Vue 3 支援有點卡     | 原生支援 Vue 3 Composition API，靈活度爆表 |
| 模塊化     | modules 雖然好用，但結構有點死板      | 多個獨立 stores，結構自由，想怎麼拆就怎麼拆 |
| TypeScript  | TypeScript 支援... 嗯，有進步空間     | 天生神力加持 TypeScript，型別安全有保障 |
| 開發體驗    | 熱更新 (HMR)？好像有點問題...       | HMR 沒煩惱，開發體驗更上一層樓         |

## 1. 安裝 Pinia：新手村的第一步

廢話不多說，先裝起來再說：

```bash
npm install pinia
```

或

```bash
yarn add pinia
```

## 2. 核心概念：創建 Pinia Stores

Pinia 的 `store` 就像 Vuex 的 `module`，把 Vuex 的 `modules` 轉成 Pinia 的 `stores` 就對了。每個 `store` 裡面有 `state`、`getters` 和 `actions`，概念很像，但寫法更簡潔。

```javascript
// src/stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }), // 狀態
  getters: {
    doubleCount: (state) => state.count * 2, // 計算屬性
  },
  actions: {
    increment() {
      this.count++ // 修改狀態
    },
  },
})
```

## 3. 在組件中使用 Pinia：狀態輕鬆 Get

在組件裡，用 `useStore` 函數把 store 抓進來，然後就可以直接用啦！

```vue
<script setup>
import { useCounterStore } from '@/stores/counter'

const counterStore = useCounterStore()
</script>

<template>
  <div>
    <p>Count: {{ counterStore.count }}</p>
    <p>Double Count: {{ counterStore.doubleCount }}</p>
    <button @click="counterStore.increment">Increment</button>
  </div>
</template>
```

## 4. 不用 `<script setup>` 也能玩 Pinia

如果專案還在用舊版的 Options API，別擔心！Pinia 還是能陪你玩。用 `mapState()` 把 store 裡的東西映射到計算屬性，一樣可以在模板裡直接用。

```vue
<script>
import { mapState } from 'pinia'
import { useCounterStore } from '@/stores/counter'

export default {
  computed: {
    // 把 counterStore.count 映射成本地的 count
    ...mapState(useCounterStore, ['count']),

    // 把 counterStore.doubleCount 映射成本地的 doubled
    ...mapState(useCounterStore, {
      doubled: 'doubleCount',
    }),
  },
}
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Doubled Count: {{ doubled }}</p>
  </div>
</template>
```

## 5. 組件外也要用 Store？沒問題！

Pinia 的 store 实例是共享的，但在組件外使用時，例如：vue-router，記得**先安裝 Pinia** (`app.use(pinia)`)，才能正確拿到 store。

```javascript
import { createRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'

const router = createRouter({ ... })

router.beforeEach((to) => {  
  const userStore = useUserStore() 
  if (to.meta.requiresAuth && !userStore.isLoggedIn) return '/login'
})
```

### 6. 結構大改造：Vuex 變 Pinia

Vuex 愛用一個 store + 多個 modules，Pinia 則是鼓勵你用多個獨立的 stores，程式碼更乾淨俐落。

```
# Vuex 結構
src
└── store
    ├── index.js 
    └── modules
        ├── user.js
        └── cart.js

# Pinia 結構
src
└── stores
    ├── user.js
    └── cart.js
```

### 7. 轉型注意事項：避開地雷區

* **逐步替換**：別一次轉換太多，慢慢來，比較快！
* **命名空間**：Pinia 的 store 預設就有命名空間，不用額外設定。
* **Composition API**：Pinia 和 Composition API 是天生一對，好好利用更靈活！

如有內容任何問題，歡迎聯絡，會盡快回覆！

> 參考資料：
>
> [Pinia 官方文檔](https://pinia.vuejs.org/)
>
> [Pinia - Vuex 的後繼者](https://johnnywang1994.github.io/book/articles/js/pinia-intro.html)