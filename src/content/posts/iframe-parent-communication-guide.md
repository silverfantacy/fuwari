---
title: iframe 與父頁跨域訊息傳遞：postMessage API 深度應用
description: 深入探討 iframe 與父層頁面之間的雙向溝通機制，包含 postMessage API 使用方法、安全性考量、實際應用案例和最佳實踐建議。
published: 2025-07-09
image: ''
tags: [JavaScript, iframe, postMessage, 跨域通信, 前端開發, Web API]
category: 前端
draft: false
---

## 前言

在現代 Web 開發中，iframe 仍然是一個重要的技術元件，特別是在嵌入第三方服務、建立安全的支付頁面、或是整合跨域功能時。然而，由於同源政策的限制，iframe 與父層頁面之間的溝通一直是開發者面臨的挑戰。

本文將深入探討如何使用 **postMessage API** 實現安全、可靠的 iframe 與父層溝通，涵蓋從基礎概念到進階應用的完整使用技巧。

## 什麼是 postMessage API？

`postMessage` 是 HTML5 提供的一個 API，專門用於解決跨域窗口之間的安全通信問題。它允許不同來源的窗口（包括 iframe）之間傳遞消息，同時保持適當的安全性。

### API 基本語法

```javascript
window.postMessage(message, targetOrigin, [transfer]);
```

- **message**: 要傳遞的數據（任何可序列化的對象）
- **targetOrigin**: 目標窗口的來源（建議明確指定，避免使用 `*`）
- **transfer**: 可選的 Transferable 對象數組

## 基礎溝通實現

### 1. iframe 向父層發送消息

iframe 子頁面可以使用 `parent.postMessage()` 向父層發送消息：

```javascript
// 在 iframe 子頁面中
function sendMessageToParent(data) {
    // 建議明確指定父層來源，而非使用 '*'
    window.parent.postMessage({
        type: 'childMessage',
        data: data,
        timestamp: Date.now()
    }, 'https://parent-domain.com');
}

// 使用範例
sendMessageToParent({ 
    action: 'resize',
    height: document.body.scrollHeight 
});
```

### 2. 父層監聽 iframe 消息

父層頁面需要設置事件監聽器來接收 iframe 的消息：

```javascript
// 在父層頁面中
window.addEventListener('message', function(event) {
    // 安全性檢查：驗證來源
    if (event.origin !== 'https://trusted-iframe-domain.com') {
        console.warn('收到來自不受信任來源的消息:', event.origin);
        return;
    }
    
    // 處理不同類型的消息
    switch (event.data.type) {
        case 'childMessage':
            handleChildMessage(event.data);
            break;
        case 'resize':
            handleIframeResize(event.data);
            break;
        default:
            console.log('未知的消息類型:', event.data.type);
    }
}, false);

function handleChildMessage(data) {
    console.log('收到子頁面消息:', data);
    // 在這裡處理具體的業務邏輯
}

function handleIframeResize(data) {
    const iframe = document.getElementById('myIframe');
    if (iframe && data.height) {
        iframe.style.height = data.height + 'px';
    }
}
```

## 雙向溝通實現

### 1. 父層向 iframe 發送消息

```javascript
// 父層頁面向 iframe 發送消息
function sendMessageToIframe(iframeId, message) {
    const iframe = document.getElementById(iframeId);
    if (iframe && iframe.contentWindow) {
        iframe.contentWindow.postMessage({
            type: 'parentMessage',
            data: message,
            timestamp: Date.now()
        }, 'https://iframe-domain.com');
    }
}

// 使用範例
sendMessageToIframe('myIframe', {
    action: 'updateContent',
    content: '新的內容數據'
});
```

### 2. iframe 監聽父層消息

```javascript
// 在 iframe 子頁面中
window.addEventListener('message', function(event) {
    // 驗證來源
    if (event.origin !== 'https://parent-domain.com') {
        return;
    }
    
    // 處理父層消息
    if (event.data.type === 'parentMessage') {
        handleParentMessage(event.data);
    }
}, false);

function handleParentMessage(data) {
    console.log('收到父層消息:', data);
    
    switch (data.action) {
        case 'updateContent':
            updatePageContent(data.content);
            break;
        case 'changeTheme':
            changeTheme(data.theme);
            break;
        default:
            console.log('未知的操作:', data.action);
    }
}
```

## 實際應用案例

### 1. iframe 高度自適應

這是最常見的應用場景之一：

```javascript
// iframe 子頁面
function sendHeightToParent() {
    const height = Math.max(
        document.body.scrollHeight,
        document.body.offsetHeight,
        document.documentElement.clientHeight,
        document.documentElement.scrollHeight,
        document.documentElement.offsetHeight
    );
    
    window.parent.postMessage({
        type: 'resize',
        height: height,
        iframeId: 'myIframe' // 支援多個 iframe
    }, 'https://parent-domain.com');
}

// 監聽內容變化
window.addEventListener('load', sendHeightToParent);
window.addEventListener('resize', debounce(sendHeightToParent, 300));

// 如果有動態內容，也要調用
// 例如：Ajax 請求完成後、DOM 更新後等
```

```javascript
// 父層頁面
window.addEventListener('message', function(event) {
    if (event.origin !== 'https://trusted-iframe-domain.com') return;
    
    if (event.data.type === 'resize') {
        const iframe = document.getElementById(event.data.iframeId);
        if (iframe) {
            iframe.style.height = event.data.height + 'px';
        }
    }
});
```

### 2. 跨域表單提交

```javascript
// iframe 中的表單提交
function submitForm(formData) {
    // 模擬表單提交
    fetch('/api/submit', {
        method: 'POST',
        body: JSON.stringify(formData),
        headers: {
            'Content-Type': 'application/json'
        }
    })
    .then(response => response.json())
    .then(result => {
        // 將結果傳遞給父層
        window.parent.postMessage({
            type: 'formSubmission',
            success: result.success,
            message: result.message,
            data: result.data
        }, 'https://parent-domain.com');
    })
    .catch(error => {
        window.parent.postMessage({
            type: 'formSubmission',
            success: false,
            message: error.message
        }, 'https://parent-domain.com');
    });
}
```

### 3. 使用者認證狀態同步

```javascript
// iframe 中的認證狀態變化
function notifyAuthStatusChange(isAuthenticated, userInfo) {
    window.parent.postMessage({
        type: 'authStatusChange',
        isAuthenticated: isAuthenticated,
        userInfo: userInfo
    }, 'https://parent-domain.com');
}

// 父層處理認證狀態
window.addEventListener('message', function(event) {
    if (event.origin !== 'https://auth-domain.com') return;
    
    if (event.data.type === 'authStatusChange') {
        if (event.data.isAuthenticated) {
            // 更新 UI 顯示已登入狀態
            updateUIForLoggedInUser(event.data.userInfo);
        } else {
            // 更新 UI 顯示未登入狀態
            updateUIForGuestUser();
        }
    }
});
```

## 進階技巧與最佳實踐

### 1. 檢測 iframe 是否完全載入

```javascript
// 方法一：使用 onload 事件
const iframe = document.getElementById('myIframe');
iframe.onload = function() {
    console.log('iframe 載入完成');
    // 可以開始發送消息
    sendMessageToIframe('myIframe', { type: 'init' });
};

// 方法二：兼容性更好的寫法
function setupIframeLoadHandler(iframe) {
    if (iframe.attachEvent) {
        // IE 瀏覽器
        iframe.attachEvent('onload', handleIframeLoad);
    } else {
        // 現代瀏覽器
        iframe.onload = handleIframeLoad;
    }
}

function handleIframeLoad() {
    console.log('iframe 載入完成');
    // 初始化邏輯
}

// 方法三：使用 Promise 包裝
function waitForIframeLoad(iframe) {
    return new Promise(resolve => {
        const loadHandler = () => {
            iframe.onload = null; // 清理 handler 避免重複觸發或內存洩漏
            resolve();
        };
        // 先指派 handler，再檢查狀態，以避免 race condition
        iframe.onload = loadHandler;
        // 如果 iframe 已載入，則 onload 事件可能不會再觸發，手動調用 handler
        if (iframe.contentDocument && iframe.contentDocument.readyState === 'complete') {
            loadHandler();
        }
    });
}

// 使用範例
waitForIframeLoad(document.getElementById('myIframe'))
    .then(() => {
        console.log('iframe 已準備就緒');
        // 開始通信
    });
```

### 2. 處理多個 iframe

```javascript
// 管理多個 iframe 的通信
class IframeManager {
    constructor() {
        this.iframes = new Map();
        this.setupMessageListener();
    }
    
    addIframe(id, origin, element) {
        this.iframes.set(id, {
            origin: origin,
            element: element,
            isReady: false
        });
        
        // 監聽 iframe 載入
        element.onload = () => {
            this.iframes.get(id).isReady = true;
            this.sendMessage(id, { type: 'init' });
        };
    }
    
    sendMessage(iframeId, message) {
        const iframe = this.iframes.get(iframeId);
        if (iframe && iframe.isReady) {
            iframe.element.contentWindow.postMessage({
                ...message,
                iframeId: iframeId
            }, iframe.origin);
        }
    }
    
    setupMessageListener() {
        window.addEventListener('message', (event) => {
            // 查找對應的 iframe
            const iframe = [...this.iframes.values()]
                .find(iframe => iframe.origin === event.origin);
            
            if (iframe) {
                this.handleMessage(event.data);
            }
        });
    }
    
    handleMessage(data) {
        console.log('收到消息:', data);
        // 根據不同的 iframe 或消息類型處理
    }
}

// 使用範例
const manager = new IframeManager();
manager.addIframe('widget1', 'https://widget1.com', document.getElementById('widget1'));
manager.addIframe('widget2', 'https://widget2.com', document.getElementById('widget2'));
```

### 3. 錯誤處理和重試機制

```javascript
// 帶有重試機制的消息發送
function sendMessageWithRetry(target, message, targetOrigin, maxRetries = 3) {
    let retryCount = 0;
    
    function attemptSend() {
        try {
            target.postMessage(message, targetOrigin);
            return true;
        } catch (error) {
            console.error('發送消息失敗:', error);
            retryCount++;
            
            if (retryCount < maxRetries) {
                setTimeout(attemptSend, 1000 * retryCount); // 遞增延遲
            }
            return false;
        }
    }
    
    return attemptSend();
}

// 消息確認機制
class MessageConfirmation {
    constructor() {
        this.pendingMessages = new Map();
        this.messageId = 0;
    }
    
    sendMessage(target, message, targetOrigin, timeout = 5000) {
        const id = ++this.messageId;
        
        return new Promise((resolve, reject) => {
            const timeoutId = setTimeout(() => {
                this.pendingMessages.delete(id);
                reject(new Error('消息發送超時'));
            }, timeout);
            
            this.pendingMessages.set(id, {
                resolve,
                reject,
                timeoutId
            });
            
            target.postMessage({
                ...message,
                messageId: id,
                requiresConfirmation: true
            }, targetOrigin);
        });
    }
    
    handleConfirmation(messageId) {
        const pending = this.pendingMessages.get(messageId);
        if (pending) {
            clearTimeout(pending.timeoutId);
            this.pendingMessages.delete(messageId);
            pending.resolve();
        }
    }
}
```

### 4. 性能優化

```javascript
// 使用防抖減少頻繁的消息發送
function debounce(func, wait) {
    let timeout;
    return function executedFunction(...args) {
        const later = () => {
            clearTimeout(timeout);
            func(...args);
        };
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
    };
}

// 使用節流控制消息頻率
function throttle(func, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}

// 應用範例
const debouncedHeightUpdate = debounce(sendHeightToParent, 300);
const throttledScrollUpdate = throttle(sendScrollPosition, 100);

// 監聽事件
window.addEventListener('resize', debouncedHeightUpdate);
window.addEventListener('scroll', throttledScrollUpdate);
```

## 安全性考量

### 1. 來源驗證

```javascript
// 嚴格的來源驗證
function isValidOrigin(origin) {
    const allowedOrigins = [
        'https://example.com',
        'https://subdomain.example.com',
        'https://another-trusted-domain.com'
    ];
    
    return allowedOrigins.includes(origin);
}

window.addEventListener('message', function(event) {
    if (!isValidOrigin(event.origin)) {
        console.warn('拒絕來自不受信任來源的消息:', event.origin);
        return;
    }
    
    // 處理消息
});
```

### 2. 消息內容驗證

```javascript
// 消息格式驗證
function validateMessage(data) {
    // 檢查必要字段
    if (!data.type || typeof data.type !== 'string') {
        return false;
    }
    
    // 檢查消息類型是否在允許列表中
    const allowedTypes = ['resize', 'formSubmission', 'authStatusChange'];
    if (!allowedTypes.includes(data.type)) {
        return false;
    }
    
    // 根據類型進行具體驗證
    switch (data.type) {
        case 'resize':
            return typeof data.height === 'number' && data.height > 0;
        case 'formSubmission':
            return typeof data.success === 'boolean';
        default:
            return true;
    }
}

window.addEventListener('message', function(event) {
    if (!isValidOrigin(event.origin)) return;
    
    if (!validateMessage(event.data)) {
        console.warn('無效的消息格式:', event.data);
        return;
    }
    
    // 處理有效消息
});
```

### 3. 內容安全政策 (CSP)

```html
<!-- 在 HTML 頭部設置 CSP -->
<meta http-equiv="Content-Security-Policy" content="
    default-src 'self';
    frame-src https://trusted-iframe-domain.com;
    frame-ancestors 'self' https://trusted-parent-domain.com;
">
```

## 瀏覽器兼容性

### 1. 基本兼容性

- **Internet Explorer**: 8+ (部分功能需要 polyfill)
- **Chrome**: 所有版本
- **Firefox**: 所有版本
- **Safari**: 所有版本
- **Edge**: 所有版本

### 2. 兼容性處理

```javascript
// 檢測 postMessage 支援
if (typeof window.postMessage !== 'function') {
    console.error('當前瀏覽器不支援 postMessage');
    // 提供替代方案或提示用戶升級瀏覽器
}

// IE 8 兼容性處理
function addEventListenerCompat(element, event, handler) {
    if (element.addEventListener) {
        element.addEventListener(event, handler, false);
    } else if (element.attachEvent) {
        element.attachEvent('on' + event, handler);
    }
}

// 使用兼容性函數
addEventListenerCompat(window, 'message', handleMessage);
```

## 常見問題排查

### 1. 消息未收到

```javascript
// 調試用的消息監聽器
window.addEventListener('message', function(event) {
    console.log('收到消息:', {
        origin: event.origin,
        data: event.data,
        source: event.source
    });
}, false);

// 檢查 iframe 是否已載入
function checkIframeReady(iframeId) {
    const iframe = document.getElementById(iframeId);
    return iframe && iframe.contentWindow && iframe.contentDocument;
}
```

### 2. 跨域問題

```javascript
// 檢查是否為跨域
function isCrossDomain(url) {
    const link = document.createElement('a');
    link.href = url;
    return link.hostname !== window.location.hostname;
}

// 設置 iframe 的 src 時檢查
const iframeUrl = 'https://example.com/widget';
if (isCrossDomain(iframeUrl)) {
    console.log('這是跨域 iframe，需要使用 postMessage');
}
```

### 3. 消息序列化問題

```javascript
// 安全的消息序列化
function safeStringify(obj) {
    try {
        return JSON.stringify(obj);
    } catch (error) {
        console.error('序列化失敗:', error);
        return null;
    }
}

// 安全的消息解析
function safeParse(str) {
    try {
        return JSON.parse(str);
    } catch (error) {
        console.error('解析失敗:', error);
        return null;
    }
}
```

## 實際部署建議

### 1. 開發環境設置

```javascript
// 開發環境的寬鬆設置
const isDevelopment = process.env.NODE_ENV === 'development';

window.addEventListener('message', function(event) {
    if (isDevelopment) {
        // 開發環境允許 localhost
        if (event.origin.includes('localhost') || event.origin.includes('127.0.0.1')) {
            handleMessage(event.data);
            return;
        }
    }
    
    // 生產環境的嚴格驗證
    if (!isValidOrigin(event.origin)) {
        return;
    }
    
    handleMessage(event.data);
});
```

### 2. 生產環境優化

```javascript
// 生產環境的錯誤監控
window.addEventListener('error', function(event) {
    if (event.target && event.target.tagName === 'IFRAME') {
        console.error('iframe 載入錯誤:', event.target.src);
        // 發送錯誤報告到監控系統
    }
});

// 性能監控
const performanceObserver = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        if (entry.name.includes('iframe')) {
            console.log('iframe 性能數據:', entry);
        }
    }
});
performanceObserver.observe({ entryTypes: ['navigation', 'resource'] });
```

## 結論

iframe 與父層的溝通是現代 Web 開發中的重要技術，正確使用 postMessage API 可以實現安全、高效的跨域通信。在實際應用中，需要特別注意：

### 關鍵要點

1. **安全性優先**: 始終驗證消息來源和內容
2. **明確指定來源**: 避免使用 `*` 作為 targetOrigin
3. **錯誤處理**: 實現完整的錯誤處理和重試機制
4. **性能考量**: 使用防抖和節流優化消息頻率
5. **兼容性**: 考慮不同瀏覽器的兼容性問題

### 最佳實踐

- 建立統一的消息格式規範
- 實現消息確認機制
- 提供詳細的錯誤日誌
- 定期檢查和更新安全策略
- 進行充分的跨瀏覽器測試

通過遵循這些原則和實踐，您可以建立穩定、安全的 iframe 通信系統，為用戶提供良好的使用體驗。

---

> **參考資料：**
> - [MDN Web Docs - postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)
> - [HTML5 Specification - Cross-document messaging](https://html.spec.whatwg.org/multipage/web-messaging.html)
> - [postMessage：主頁、iframe 頁可互相傳值](https://www.letswrite.tw/postmessage/)