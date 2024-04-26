---
title: 在 Astro 中添加程式碼區塊複製按鈕
published: 2024-04-27
description: 如何在 Astro 中添加程式碼區塊複製按鈕，以方便讀者複製程式碼。
image: 'https://media.virtualxnews.com/2024/04/56bbe738334cf25a8db0b969632242a6.png'
tags: [Astro, 程式碼複製按鈕, JavaScript ]
category: 個人專案
draft: false
---

在 Astro 中添加程式碼區塊複製按鈕，可以讓讀者更方便地複製程式碼。以下是在 Astro 中添加程式碼區塊複製按鈕的環境與方法：

## 環境
- [Fuwari 主題](https://github.com/saicaca/fuwari)
- Tailwind CSS

## 方法

以 Fuwari 主題為例，在 `/src/components/misc/Markdown.astro` 中添加以下完成的程式碼：

```astro
<script>
  const observer = new MutationObserver(addPreCopyButton);
  observer.observe(document.body, { childList: true, subtree: true });

  document.addEventListener("DOMContentLoaded", addPreCopyButton);

  function addPreCopyButton() {
    observer.disconnect();

    let codeBlocks = Array.from(document.querySelectorAll("pre"));

    for (let codeBlock of codeBlocks) {
      if (codeBlock.parentElement?.nodeName === "DIV" && codeBlock.parentElement?.classList.contains("code-block")) continue;

      let wrapper = document.createElement("div");
      wrapper.className = "relative code-block";

      let copyButton = document.createElement("button");
      copyButton.className = "copy-code btn-regular absolute top-0 right-0 text-sm px-3 py-2 rounded-bl-lg rounded-tr-lg opacity-75 hover:opacity-100 transition-all duration-200 ease-in-out";
      copyButton.textContent = 'Copy';

      codeBlock.setAttribute("tabindex", "0");
      if (codeBlock.parentNode) {
        codeBlock.parentNode.insertBefore(wrapper, codeBlock);
      }

      wrapper.appendChild(codeBlock);
      wrapper.appendChild(copyButton);

      copyButton.addEventListener("click", async () => {
        let text = codeBlock?.querySelector("code")?.innerText;

        await navigator.clipboard.writeText(text);

        copyButton.textContent = 'Code Copied';
        copyButton.classList.toggle("opacity-100");

        setTimeout(() => {
          copyButton.textContent = 'Copy';
          copyButton.classList.toggle("opacity-100");
        }, 700);
      });
    }

    observer.observe(document.body, { childList: true, subtree: true });
  }
</script>
```

這邊以 Tailwind CSS 的寫法做範例，你可以根據需求調整 CSS 樣式與寫法。

## 遇到的狀況
在 Astro 的 JavaScript 中，只有在頁面載入時才會載入一次。這導致了一個問題：當從一個頁面切換到另一個頁面時，JavaScript 代碼不會重新載入，因此無法單獨使用 `window.addEventListener("load", (event) => {});` 事件來執行 `addPreCopyButton`。

最初的解決方案是在網址切換時重新執行 `addPreCopyButton`。然而，這種方法存在一些問題。當網址切換時，無法確定畫面是否已經完成渲染，可能導致無法正確添加複製按鈕。即使使用 setTimeout 延遲執行，仍無法確定畫面是否已經渲染完成。

以下是原始解決方案所使用的程式碼：

```javascript
const originalPushState = history.pushState;
const originalReplaceState = history.replaceState;

history.pushState = function() {
    originalPushState.apply(this, arguments);
    checkURL();
};

history.replaceState = function() {
    originalReplaceState.apply(this, arguments);
    checkURL();
};

window.onpopstate = function() {
    checkURL();
};

function checkURL() {
    if (window.location.pathname.startsWith('/posts/')) {
        console.log("URL changed");
    }
}
```

根據我的記憶，可以使用 MutationObserver 來監聽畫面變化。因此，我們可以改用 MutationObserver 來監聽畫面變化，並在變化發生時執行 addPreCopyButton。

以下是使用 MutationObserver 的方法：

```javascript
const observer = new MutationObserver(function (mutations) {
  console.log(mutations);
});

const div = document.querySelector("div");
observer.observe(div, {
  childList: true,
  attributes: true,
  characterData: true,
});
```

在實際執行 addPreCopyButton 時，我們首先斷開 observer 的監聽，以免在添加複製按鈕時再次觸發 observer，造成無限迴圈。待添加複製按鈕完成後，再重新啟動 observer 的監聽。

如果你有更好的解決方案，歡迎聯絡我～

> 參考資料：
> 
> [Adding a Copy Code Button in Astro Markdown Code Blocks](https://timneubauer.dev/blog/copy-code-button-in-astro/#preparing-astro-layout)
>
> [那些被忽略但很好用的 Web API / MutationObserver](https://ithelp.ithome.com.tw/articles/10277536)