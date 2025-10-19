# Research Findings: 美股智慧分析儀表板

**建立日期**: 2025-10-15  
**目的**: 解決實施計劃中的技術未知數和依賴關係

## 前端技術棧研究

### React狀態管理: Redux Toolkit vs Zustand

**Decision**: 選擇 Redux Toolkit

**Rationale**: 
- Redux Toolkit 提供了更完整的功能，包括時間旅行調試和開發工具
- 對於需要管理複雜狀態的金融應用程式，Redux的生態系統更成熟
- Redux Toolkit簡化了傳統Redux的樣板代碼，提高了開發效率
- 團隊熟悉度較高，有豐富的學習資源

**Alternatives considered**: 
- Zustand: 更輕量級，但對於複雜的金融數據狀態管理可能功能不足

### UI函式庫: Tailwind CSS + Ant Design

**Decision**: 選擇 Tailwind CSS 混合 Ant Design

**Rationale**:
- Tailwind CSS 提供了高度可定制的實用工具類，允許快速構建自定義設計
- Ant Design 提供了預構建的高品質組件，特別是表格、表單和數據展示組件
- 混合使用可以平衡設計靈活性和開發效率
- Ant Design 的組件特別適合數據密集型應用程式

**Alternatives considered**:
- 單獨使用 Material-UI: 功能豐富但定制性較低
- 單獨使用 Tailwind CSS: 需要從頭構建所有組件，開發時間較長

### 圖表函式庫: Recharts

**Decision**: 選擇 Recharts

**Rationale**:
- Recharts 是React原生函式庫，與我們的前端棧完美集成
- 提供了聲明式API，易於理解和維護
- 支持自定義組件，可以輕鬆添加互動功能
- 社區活躍，文檔完整
- 對於金融圖表常見的需求（如縮放、工具提示）有良好支持

**Alternatives considered**:
- Chart.js: 功能強大但與React集成較複雜
- D3.js: 非常靈活但學習曲線陡峭，開發時間較長

## 後端技術棧研究

### FastAPI性能與中間件

**Decision**: 使用 FastAPI + Uvicorn

**Rationale**:
- FastAPI 基於Starlette和Pydantic，提供了高性能的異步框架
- 自動API文檔生成減少了文檔維護工作
- 類型提示支持提高了代碼可讀性和可靠性
- 與Python生態系統中的數據處理函式庫（pandas、numpy）集成良好
- 支持依賴注入，便於測試和模組化

**Performance considerations**:
- 使用異步處理來管理多個並發API請求
- 實施緩存策略以減少對Yahoo Finance API的調用
- 使用連接池優化數據庫連接

### yfinance API限制與緩存策略

**Decision**: 實施多層緩存策略

**Rationale**:
- Yahoo Finance API沒有官方速率限制文檔，但社區報告建議限制請求頻率
- 實施混合緩存策略（PostgreSQL緩存表 + Python內存緩存）來存儲近期的API響應，減少對外部API的直接調用
- 使用數據庫作為長期緩存，存儲歷史數據
- 實施請求隊列和速率限制以避免被API封鎖

**Implementation strategy**:
- 即時數據（當前價格）：緩存1-2秒
- 日內數據（1D、5D）：緩存1分鐘
- 長期歷史數據（1M、3M、6M、1Y、5Y）：緩存1小時
- 實施後台任務定期更新緩存

### 技術指標計算: pandas-ta

**Decision**: 選擇 pandas-ta

**Rationale**:
- pandas-ta 是專為金融技術分析設計的函式庫
- 與pandas DataFrame無縫集成
- 提供了廣泛的技術指標，包括SMA、RSI、MACD等
- 性能優化，適合處理大量數據
- 文檔完整，有豐富的示例

**Alternatives considered**:
- 手動實現：完全控制但開發時間長，容易出錯
- TA-Lib：性能高但安裝複雜，依賴外部C函式庫

## 數據庫設計研究

### PostgreSQL 數據庫解決方案

**Decision**: 選擇直接使用 PostgreSQL

**Rationale**:
- 直接使用 PostgreSQL 提供了完全的數據庫控制權和靈活性
- 強大的JSON支持，適合存儲API響應和複雜數據結構
- 支持時間序列數據的高效查詢，特別適合金融數據分析
- 強一致性保證金融數據的準確性
- 豐富的索引選項，優化查詢性能
- 使用 FastAPI + SQLAlchemy + asyncpg 提供高性能的異步數據庫操作
- 通過自定義 JWT 實現提供安全的身份驗證
- 使用 WebSocket 實現實時數據更新功能
- 開源且龐大的社區支持，穩定可靠
- 避免第三方服務依賴，提高系統自主可控性

**Alternatives considered**:
- Supabase: 雖然方便但增加了不必要的依賴層
- Firebase: NoSQL數據庫，不適合結構化金融數據
- MongoDB: 靈活但對事務支持較弱，不適合金融數據
- AWS Amplify: 功能豐富但學習曲線較陡峭
- TimescaleDB: 專為時間序列設計但生態系統較小

### 數據表結構優化

**Decision**: 使用關係型設計結合JSON欄位

**Rationale**:
- 用戶數據和觀察清單使用傳統關係型設計確保一致性
- 股票元數據使用JSONB類型存儲，以適應不同數據源
- 價格歷史數據使用專用表，優化時間序列查詢
- 實施適當的索引策略平衡查詢性能和存儲空間

## 安全性研究

### JWT實作最佳實踐

**Decision**: 使用JWT with refresh token策略

**Rationale**:
- JWT適合無狀態API認證，支持分布式部署
- Access token有效期短（15-30分鐘）降低安全風險
- Refresh token有效期長（7-30天）存儲在安全HTTP-only cookie中
- 實施token輪換機制提高安全性
- 使用自定義實現而非第三方服務，提高系統自主可控性

**Implementation considerations**:
- 使用強密碼學算法（HS256或RS256）
- 實施適當的錯誤處理，不泄露敏感信息
- 在PostgreSQL中維護token黑名單以支持登出功能
- 使用bcrypt進行密碼哈希處理
- 實施適當的速率限制防止暴力破解

### 數據驗證與淨化

**Decision**: 使用Pydantic進行數據驗證

**Rationale**:
- Pydantic與FastAPI原生集成
- 提供類型驗證和序列化/反序列化
- 自動生成API文檔中的數據模型
- 支持自定義驗證器處理特定金融數據格式

## 部署與監控研究

### 容器化策略

**Decision**: 使用Docker和docker-compose

**Rationale**:
- Docker確保開發和生產環境一致性
- docker-compose簡化本地開發環境設置
- 便於水平擴展和負載均衡
- 支持CI/CD管道

### 監控與日誌

**Decision**: 使用Prometheus + Grafana

**Rationale**:
- Prometheus提供強大的指標收集和查詢功能
- Grafana提供靈活的儀表板和可視化
- 監控API響應時間、錯誤率和資源使用情況
- 設置警報及時響應系統問題

## 總結

基於研究結果，我們確定了以下技術棧：

**前端**:
- React 18 + Redux Toolkit
- Tailwind CSS + Ant Design
- Recharts for 圖表
- React Router for 導航
- Axios for HTTP請求

**後端**:
- Python 3.9+ + FastAPI
- yfinance + pandas + pandas-ta
- SQLAlchemy + asyncpg
- Pydantic for 數據驗證
- JWT with refresh token for 認證

**數據庫與後端服務**:
- 直接 PostgreSQL with 混合關係型和JSON存儲
- 自定義 JWT 實現 for 身份驗證
- WebSocket for 即時數據更新

**部署**:
- Docker + docker-compose
- FastAPI for 服務器端邏輯
- 混合緩存策略（PostgreSQL緩存表 + Python內存緩存）for 緩存
- Prometheus + Grafana for 監控

這個技術棧平衡了性能、開發效率、可維護性和可擴展性，特別適合金融數據應用的特殊需求。直接使用PostgreSQL提供了完全的數據庫控制權，自定義JWT實現提高了系統自主可控性，同時保持了高性能和安全性。