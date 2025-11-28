# CWA 天氣預報 API 專案說明

## 專案概述

這是一個使用 Node.js + Express 開發的天氣預報 API 服務，串接中央氣象署（CWA）開放資料平台 API，提供台灣各縣市的 36 小時天氣預報資料。

## 專案架構

### 技術堆疊

- **後端框架**: Express.js 4.18.2
- **HTTP 客戶端**: Axios 1.6.0
- **環境變數管理**: dotenv 16.3.1
- **跨域支援**: CORS 2.8.5
- **開發工具**: nodemon 3.0.1（自動重啟）

### 專案結構

```
CwaWeather-backend/
├── server.js              # Express 伺服器主檔案（路由、控制器、業務邏輯）
├── .env                   # 環境變數配置（包含 CWA API Key）
├── .gitignore            # Git 忽略檔案清單
├── package.json          # 專案配置與依賴套件
├── package-lock.json     # 套件版本鎖定
├── README.md             # 使用說明文件
└── project.md            # 專案技術文件（本文件）
```

### 核心檔案說明

#### server.js

這是專案的核心檔案，包含所有的業務邏輯：

- **第 1-16 行**: 環境設定與中介軟體配置
  - 載入環境變數
  - 設定 Express 中介軟體（CORS、JSON 解析）
  - 配置 CWA API 基礎 URL 和 API Key

- **第 18-138 行**: `getWeather()` 控制器函數
  - 處理天氣資料請求
  - 從路徑參數或查詢參數中取得城市名稱
  - 呼叫 CWA API 取得天氣資料
  - 解析並整理天氣資料
  - 錯誤處理與回應

- **第 140-164 行**: 路由定義
  - 首頁路由（顯示 API 使用說明）
  - 健康檢查路由
  - 天氣查詢路由（支援路徑參數和查詢參數）

- **第 166-174 行**: 錯誤處理與 404 處理

- **第 176-179 行**: 伺服器啟動

## 功能說明

### API 端點

#### 1. 首頁 - API 說明

```
GET /
```

回應範例：
```json
{
  "message": "歡迎使用 CWA 天氣預報 API",
  "endpoints": {
    "weather": "/api/weather/:city (例如: /api/weather/高雄市 或 /api/weather/臺北市)",
    "weatherQuery": "/api/weather?city=城市名稱 (例如: /api/weather?city=高雄市)",
    "health": "/api/health"
  },
  "availableCities": [
    "臺北市", "新北市", "桃園市", "臺中市", "臺南市", "高雄市",
    "基隆市", "新竹市", "新竹縣", "苗栗縣", "彰化縣", "南投縣",
    "雲林縣", "嘉義市", "嘉義縣", "屏東縣", "宜蘭縣", "花蓮縣",
    "臺東縣", "澎湖縣", "金門縣", "連江縣"
  ]
}
```

#### 2. 健康檢查

```
GET /api/health
```

回應範例：
```json
{
  "status": "OK",
  "timestamp": "2025-11-28T12:00:00.000Z"
}
```

#### 3. 取得天氣預報（路徑參數方式）

```
GET /api/weather/:city
```

範例：
- `/api/weather/高雄市`
- `/api/weather/臺北市`
- `/api/weather/新北市`

#### 4. 取得天氣預報（查詢參數方式）

```
GET /api/weather?city=城市名稱
```

範例：
- `/api/weather?city=高雄市`
- `/api/weather?city=臺北市`

### 天氣資料回應格式

```json
{
  "success": true,
  "data": {
    "city": "高雄市",
    "updateTime": "資料更新時間說明",
    "forecasts": [
      {
        "startTime": "2025-11-28 18:00:00",
        "endTime": "2025-11-29 06:00:00",
        "weather": "多雲時晴",
        "rain": "10%",
        "minTemp": "25°C",
        "maxTemp": "32°C",
        "comfort": "悶熱",
        "windSpeed": "偏南風 3-4 級"
      }
    ]
  }
}
```

### 錯誤處理

API 會回傳適當的 HTTP 狀態碼和錯誤訊息：

- `400`: 參數錯誤（未提供城市名稱）
- `404`: 查無資料（城市名稱錯誤）
- `500`: 伺服器錯誤（API Key 未設定或 CWA API 錯誤）

錯誤回應範例：
```json
{
  "error": "參數錯誤",
  "message": "請提供城市名稱參數"
}
```

## 重大修改記錄

### 修改前的實作方式

原本的專案設計是固定取得特定城市的天氣資料：

1. **函數名稱**: `getKaohsiungWeather`（固定取得高雄市天氣）
2. **API 請求**: `locationName` 參數被硬編碼為「新北市」
3. **路由設計**: 固定路由 `/api/weather/kaohsiung`
4. **限制**: 無法查詢其他城市的天氣資料

#### 修改前的關鍵程式碼片段

```javascript
// 舊版函數定義
const getKaohsiungWeather = async (req, res) => {
  // ...
  const response = await axios.get(
    `${CWA_API_BASE_URL}/v1/rest/datastore/F-C0032-001`,
    {
      params: {
        Authorization: CWA_API_KEY,
        locationName: "新北市",  // 固定的城市名稱
      },
    }
  );
  // ...
};

// 舊版路由
app.get("/api/weather/kaohsiung", getKaohsiungWeather);
```

### 修改後的實作方式

新版本支援動態查詢任意城市的天氣資料：

1. **函數名稱**: `getWeather`（通用天氣查詢函數）
2. **參數處理**: 從路徑參數 (`req.params.city`) 或查詢參數 (`req.query.city`) 取得城市名稱
3. **路由設計**:
   - 支援路徑參數：`/api/weather/:city`
   - 支援查詢參數：`/api/weather?city=城市名稱`
4. **彈性**: 可查詢台灣任何縣市的天氣資料

#### 修改後的關鍵程式碼片段

```javascript
// 新版函數定義
const getWeather = async (req, res) => {
  // 從路徑參數或查詢參數中取得城市名稱
  const cityName = req.params.city || req.query.city;

  // 檢查是否提供城市名稱
  if (!cityName) {
    return res.status(400).json({
      error: "參數錯誤",
      message: "請提供城市名稱參數",
    });
  }

  // ...
  const response = await axios.get(
    `${CWA_API_BASE_URL}/v1/rest/datastore/F-C0032-001`,
    {
      params: {
        Authorization: CWA_API_KEY,
        locationName: cityName,  // 動態的城市名稱
      },
    }
  );
  // ...
};

// 新版路由（支援兩種參數傳遞方式）
app.get("/api/weather/:city", getWeather);
app.get("/api/weather", getWeather);
```

### 修改的技術細節

#### 1. 參數驗證機制

新增了參數驗證邏輯，確保使用者必須提供城市名稱：

```javascript
const cityName = req.params.city || req.query.city;

if (!cityName) {
  return res.status(400).json({
    error: "參數錯誤",
    message: "請提供城市名稱參數",
  });
}
```

#### 2. 錯誤訊息改善

錯誤訊息現在會顯示具體的城市名稱：

```javascript
if (!locationData) {
  return res.status(404).json({
    error: "查無資料",
    message: `無法取得 ${cityName} 天氣資料，請確認城市名稱是否正確`,
  });
}
```

#### 3. 路由彈性設計

支援兩種 URL 格式，提供更好的 API 使用體驗：

```javascript
// RESTful 風格的路徑參數
app.get("/api/weather/:city", getWeather);

// 傳統的查詢參數
app.get("/api/weather", getWeather);
```

#### 4. 首頁說明更新

首頁現在會顯示所有可用的城市清單和使用範例：

```javascript
app.get("/", (req, res) => {
  res.json({
    message: "歡迎使用 CWA 天氣預報 API",
    endpoints: {
      weather: "/api/weather/:city (例如: /api/weather/高雄市 或 /api/weather/臺北市)",
      weatherQuery: "/api/weather?city=城市名稱 (例如: /api/weather?city=高雄市)",
      health: "/api/health",
    },
    availableCities: [
      "臺北市", "新北市", "桃園市", "臺中市", "臺南市", "高雄市",
      "基隆市", "新竹市", "新竹縣", "苗栗縣", "彰化縣", "南投縣",
      "雲林縣", "嘉義市", "嘉義縣", "屏東縣", "宜蘭縣", "花蓮縣",
      "臺東縣", "澎湖縣", "金門縣", "連江縣"
    ]
  });
});
```

## 使用範例

### 1. 使用路徑參數查詢

```bash
# 查詢高雄市天氣
curl http://localhost:3000/api/weather/高雄市

# 查詢臺北市天氣
curl http://localhost:3000/api/weather/臺北市

# 查詢新北市天氣
curl http://localhost:3000/api/weather/新北市
```

### 2. 使用查詢參數查詢

```bash
# 查詢高雄市天氣
curl http://localhost:3000/api/weather?city=高雄市

# 查詢臺北市天氣
curl http://localhost:3000/api/weather?city=臺北市
```

### 3. 使用 JavaScript Fetch

```javascript
// 使用路徑參數
fetch('http://localhost:3000/api/weather/高雄市')
  .then(response => response.json())
  .then(data => console.log(data));

// 使用查詢參數
fetch('http://localhost:3000/api/weather?city=臺北市')
  .then(response => response.json())
  .then(data => console.log(data));
```

## CWA API 說明

### API 端點

本專案使用 CWA 的「一般天氣預報-今明 36 小時天氣預報」資料集：

- **API ID**: F-C0032-001
- **完整 URL**: `https://opendata.cwa.gov.tw/api/v1/rest/datastore/F-C0032-001`
- **文件**: https://opendata.cwa.gov.tw/dist/opendata-swagger.html

### 請求參數

- `Authorization`: CWA API Key（必填）
- `locationName`: 縣市名稱（選填，不提供則回傳所有縣市）

### 天氣要素說明

API 回傳的天氣資料包含以下要素：

- **Wx**: 天氣現象（例如：多雲時晴、晴時多雲）
- **PoP**: 降雨機率（單位：%）
- **MinT**: 最低溫度（單位：°C）
- **MaxT**: 最高溫度（單位：°C）
- **CI**: 舒適度（例如：悶熱、舒適）
- **WS**: 風速風向（例如：偏南風 3-4 級）

## 環境設定

### 必要的環境變數

在 `.env` 檔案中設定：

```env
CWA_API_KEY=your_api_key_here
PORT=3000
NODE_ENV=development
```

### 取得 CWA API Key

1. 前往 [氣象資料開放平臺](https://opendata.cwa.gov.tw/)
2. 註冊/登入帳號
3. 前往「會員專區」→「取得授權碼」
4. 複製 API 授權碼
5. 將授權碼填入 `.env` 檔案

## 開發與部署

### 安裝依賴套件

```bash
npm install
```

### 開發模式（自動重啟）

```bash
npm run dev
```

### 正式環境

```bash
npm start
```

## 注意事項

1. **API Key 保護**:
   - `.env` 檔案已加入 `.gitignore`，不會被提交到版本控制
   - 請勿在程式碼中硬編碼 API Key

2. **API 使用限制**:
   - CWA API 有每日呼叫次數限制
   - 建議實作快取機制以減少 API 呼叫次數

3. **城市名稱格式**:
   - 必須使用正確的縣市名稱（包含「市」或「縣」）
   - 例如：「高雄市」而非「高雄」
   - 「臺北市」而非「台北市」（注意使用「臺」而非「台」）

4. **錯誤處理**:
   - 請妥善處理 API 錯誤回應
   - 建議實作重試機制

## 未來改進方向

1. **快取機制**: 實作 Redis 或記憶體快取，減少 API 呼叫
2. **資料驗證**: 加入城市名稱驗證，提前回傳錯誤
3. **API 版本管理**: 實作 `/v1/api/weather` 等版本化路由
4. **日誌系統**: 加入結構化日誌記錄（例如使用 Winston）
5. **測試**: 加入單元測試和整合測試
6. **文檔**: 使用 Swagger/OpenAPI 產生 API 文件
7. **限流**: 實作 rate limiting 避免濫用
8. **監控**: 加入效能監控和錯誤追蹤（例如使用 Sentry）

## 授權

MIT
