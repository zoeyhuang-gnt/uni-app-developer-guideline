---

#  uni-app 小程序開發指南

uni-app 是基於 **Vue.js** 的跨平台開發框架，支援「一套代碼，多端發佈」。本指南將深度解析從目錄結構到核心技術的開發規範。

---

##  1. 專案目錄結構深度解析

一個標準化的 uni-app 專案應遵循以下架構，以維持代碼的可維護性：

```text
my-uni-app/
├── src/
│   ├── api/               # 統一管理後端介面請求
│   ├── components/        # 自定義可複用組件
│   ├── pages/             # 頁面檔案（建議按功能模組分資料夾）
│   │   └── index/
│   │       └── index.vue
│   ├── static/            # 靜態資源（不經過編譯，直接複製到發行包）
│   ├── store/             # 狀態管理 (Pinia 或 Vuex)
│   ├── utils/             # 通用工具函數（日期處理、正則驗證等）
│   ├── uni_modules/       # 存放符合 uni-app 插件規範的組件
│   ├── App.vue            # 應用入口：監聽應用生命週期、配置全域樣式
│   ├── main.js            # 入口文件：初始化 Vue 實例、掛載全域屬性
│   ├── pages.json         # 頁面路由與全域外觀配置
│   └── manifest.json      # 應用配置（AppID、權限、多端特有設定）
├── .env.development       # 開發環境變數
├── .env.production        # 生產環境變數
└── vite.config.ts         # Vite 編譯設定
```

---

##  2. 環境配置與編譯設定

### 2.1 模式和環境變量 (Environment Variables)
透過 `.env` 檔案定義不同環境下的 API 地址或其他配置。
*   **代碼中調用**：`import.meta.env.VITE_APP_API_URL`
*   **自定義腳本** (`package.json`)：
    ```json
    "scripts": {
      "dev:test": "uni -m test",  // 使用 .env.test 檔案
      "build:h5": "uni build -p h5"
    }
    ```

### 2.2 條件編譯 (Conditional Compilation) 
這是 uni-app 的靈魂功能，允許在同一套代碼中為不同平台撰寫專屬邏輯。
*   **語法**：`#ifdef PLATFORM` ... `#endif`
*   **範例**：
    ```javascript
    // #ifdef MP-WEIXIN
    console.log("這段代碼只會出現在微信小程序");
    // #endif

    // #ifdef H5
    console.log("這段代碼只會出現在瀏覽器端");
    // #endif
    ```

### 2.3 設計稿及尺寸單位
*   **rpx (Responsive Pixel)**：uni-app 核心單位，將螢幕寬度分為 **750rpx**。
*   **最佳實踐**：設計稿請以 **750px** (iPhone 6 基準) 出圖，開發者直接照圖標註數值寫入 rpx，即可實現全螢幕適配。
*   **注意**：在 `vue` 檔案的 `style` 中，如果需要固定尺寸（如邊框線），請使用 `px`。

---

##  3. 核心設定 (Global & Page Settings)

### 3.1 全域設定 (pages.json -> globalStyle)
定義所有頁面的預設外觀：
```json
"globalStyle": {
  "navigationBarTextStyle": "white",
  "navigationBarTitleText": "公司專案",
  "navigationBarBackgroundColor": "#007AFF",
  "enablePullDownRefresh": false,
  "renderingMode": "seperated" // 效能優化：分離渲染
}
```

### 3.2 頁面設定 (pages.json -> pages)
針對特定頁面覆蓋全域配置，例如設定下圖下拉刷新：
```json
{
  "path": "pages/list/list",
  "style": {
    "navigationBarTitleText": "資料列表",
    "enablePullDownRefresh": true,
    "onReachBottomDistance": 100 // 距離底部 100rpx 時觸發加載更多
  }
}
```

### 3.3 Babel 與 Vite 設定
*   **Babel**：確保舊版手機瀏覽器的兼容性。
*   **Vite 設定**：可配置 `easycom` 模式，讓組件無需 `import` 即可直接在模板中使用，大幅提升開發速度。

---

##  4. 路由與導航功能

### 4.1 路由跳轉 API
| API | 功能描述 | 適用場景 |
| :--- | :--- | :--- |
| `uni.navigateTo` | 保留當前頁，跳轉新頁 | 普通分頁進入詳情頁 |
| `uni.redirectTo` | 關閉當前頁，跳轉新頁 | 登入後進入個人中心 |
| `uni.reLaunch` | 關閉所有頁面，重啟應用 | 切換角色或更換語言時 |
| `uni.switchTab` | 跳轉至 TabBar 頁面 | 回到首頁或購物車 |

### 4.2 路由參數傳遞
```javascript
// A 頁面跳轉
uni.navigateTo({
  url: '/pages/detail/detail?id=123&name=demo'
});

// B 頁面接收 (onLoad 生命週期)
onLoad((options) => {
  console.log(options.id); // 輸出 123
});
```

### 4.3 安全指南：路由攔截器
利用 `uni.addInterceptor` 實現簡單的登入權限校驗：
```javascript
const whiteList = ['/pages/login/login', '/pages/index/index'];

uni.addInterceptor('navigateTo', {
  invoke(e) {
    const token = uni.getStorageSync('token');
    if (!token && !whiteList.includes(e.url)) {
      uni.navigateTo({ url: '/pages/login/login' });
      return false; // 攔截跳轉
    }
    return true;
  }
});
```

---

##  5. 靜態資源引用規範

*   **本地圖片**：放置於 `static` 目錄下。
*   **路徑寫法**：
    *   模板中：`<image src="@/static/logo.png"></image>`
    *   CSS 中：`background-image: url('~@/static/bg.jpg');`
*   **小程序限制**：
    1.  背景圖若大於 **40KB**，小程序端建議使用網路圖片位址或轉為 **Base64**。
    2.  本地字體檔案建議放置於 CDN，避免增加小程序包體積。

---

##  6. 節點獲取與 DOM 操作

在 uni-app 中，由於邏輯層與視圖層分離，無法直接操作 DOM（如 `document.querySelector`）。

### 6.1 SelectorQuery API
用於獲取節點的佈局資訊（寬高、位置）：
```javascript
const query = uni.createSelectorQuery().in(this);
query.select('.my-view').boundingClientRect(data => {
  console.log("節點離頂部距離：" + data.top);
}).exec();
```

### 6.2 IntersectionObserver API
用於監聽節點是否進入可視區域（常用於圖片懶加載或吸頂效果）：
```javascript
const observer = uni.createIntersectionObserver(this);
observer.relativeToViewport().observe('.target-node', (res) => {
  if (res.intersectionRatio > 0) {
    console.log("目標出現在螢幕中了！");
  }
});
```

---

##  7. 開發最佳實務 (Best Practices)

1.  **分包加載 (Subpackaging)**：當專案體積超過 2MB 時，務必在 `pages.json` 中配置 `subPackages`，否則無法發佈小程序。
2.  **避免 `setData` 過於頻繁**：雖然 uni-app 自動處理數據同步，但過多的大對象更新會導致頁面卡頓。
3.  **定義全域變量**：建議使用 `Pinia` 管理用戶資訊，或將常用的 API 工具掛載至 `uni.$api`。
4.  **H5 跨域處理**：在 `vite.config.ts` 中配置 `server.proxy` 解決開發環境的跨域問題。

---
