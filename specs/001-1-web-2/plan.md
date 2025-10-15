# Implementation Plan: 美股智慧分析儀表板

**Branch**: `001-1-web-2-us-stock-dashboard` | **Date**: 2025-10-15 | **Spec**: [美股智慧分析儀表板規格書](./spec.md)
**Input**: Feature specification from `/specs/001-1-web-2/spec.md`

**Note**: This template is filled in by the `/speckit.plan` command. See `.specify/templates/commands/plan.md` for the execution workflow.

## Summary

建立一個前後端分離的美股智慧分析儀表板系統，提供即時股票查詢、互動式圖表、技術指標分析及個人化觀察清單功能。前端使用ReactJS構建使用者介面，後端使用FastAPI提供RESTful API，數據源來自Yahoo Finance API，使用PostgreSQL進行數據持久化。

## Technical Context

**Language/Version**: Python 3.9+, JavaScript/ES6+
**Primary Dependencies**: FastAPI, React 18+, yfinance, pandas, SQLAlchemy, Recharts, Supabase
**Storage**: Supabase (基於PostgreSQL，用於用戶數據和觀察清單)，文件緩存（用於API響應緩存）
**Testing**: pytest (後端), Jest/React Testing Library (前端) 或 NEEDS CLARIFICATION
**Target Platform**: Web瀏覽器（Chrome, Firefox, Safari, Edge）
**Project Type**: web (前後端分離架構)
**Performance Goals**: 股票數據查詢<3秒，圖表渲染<2秒，系統可用性>99%，支持500併發用戶
**Constraints**: 市場開盤時間數據每5秒更新，處理Yahoo Finance API限制，響應時間<200ms (p95)
**Scale/Scope**: 支持三大指數（道瓊、納斯達克、S&P 500），主要美股和加密貨幣，目標1000+用戶

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### 程式碼品質與可讀性
- ✅ 所有程式碼必須以清晰度和可維護性為主要考量
- ✅ 使用有意義的變數名稱、清晰的函數結構
- ✅ 金融資料處理邏輯必須特別清晰，確保數據計算的準確性
- ✅ 已規劃程式碼審查流程，確保金融計算的準確性（通過快速開始指南中的代碼審查部分）

### 使用者體驗一致性
- ✅ 所有面向使用者的元件必須在整個應用程式中提供一致的體驗
- ✅ 股價圖表必須提供一致的視覺呈現，包括顏色編碼（上漲為綠色，下跌為紅色）
- ✅ 對於超過2秒的數據載入，必須向使用者提供載入指示器
- ✅ 已規劃實施無障礙標準確保應用程式對所有使用者都可使用（通過UI組件設計指南）

### 效能需求
- ✅ 所有程式碼在發布前必須達到定義的效能基準
- ✅ 金融數據API調用必須優化以在可接受的時間限制內執行
- ✅ 關鍵使用者路徑必須在預期負載條件下進行效能測試
- ✅ 即時股價數據更新必須在1秒內完成
- ✅ 已規劃實施效能監控和回歸測試（通過Docker配置和測試框架）

### 數據準確性與文件
- ✅ 所有金融數據處理必須確保最高準確性
- ✅ 技術指標計算（如移動平均線、RSI）必須透過註解和範例來解釋
- ✅ 文件必須與程式碼變更保持同步
- ✅ 已實施完整的API文件（通過OpenAPI規範）
- ✅ 必須透過程式碼審查、配對程式設計和定期技術討論來鼓勵知識分享
- ✅ 所有數據變更必須有審計軌跡（通過數據模型中的時間戳和用戶追蹤）

### 持續改進
- ✅ 程式碼必須定期重構以提高品質和可維護性
- ✅ 數據獲取效能指標必須持續監控和優化
- ✅ 使用者回饋必須收集並納入未來的改進中
- ✅ 技術債務必須被追蹤並系統性地解決
- ✅ 新技術指標和圖表類型必須在提供明確效益時進行評估和採用
- ✅ 程式碼庫必須在不斷變化的市場需求中演進，同時保持高標準
- ✅ 已規劃實施技術債務追蹤系統（通過項目管理流程）

**最終狀態**: ✅ 完全通過 - 所有憲法要求已在設計階段得到充分考慮和規劃

## Project Structure

### Documentation (this feature)

```
specs/001-1-web-2/
├── plan.md              # This file (/speckit.plan command output)
├── research.md          # Phase 0 output (/speckit.plan command)
├── data-model.md        # Phase 1 output (/speckit.plan command)
├── quickstart.md        # Phase 1 output (/speckit.plan command)
├── contracts/           # Phase 1 output (/speckit.plan command)
└── tasks.md             # Phase 2 output (/speckit.tasks command - NOT created by /speckit.plan)
```

### Source Code (repository root)

```
backend/
├── src/
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── stock.py
│   │   └── watchlist.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── stock_service.py
│   │   ├── auth_service.py
│   │   └── watchlist_service.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── endpoints/
│   │   │   ├── __init__.py
│   │   │   ├── stocks.py
│   │   │   ├── auth.py
│   │   │   └── watchlist.py
│   │   └── dependencies.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   └── security.py
│   ├── database.py
│   └── main.py
├── tests/
│   ├── __init__.py
│   ├── test_stocks.py
│   ├── test_auth.py
│   └── test_watchlist.py
├── requirements.txt
└── Dockerfile

frontend/
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Header.jsx
│   │   │   ├── Footer.jsx
│   │   │   └── Loading.jsx
│   │   ├── stock/
│   │   │   ├── SearchBar.jsx
│   │   │   ├── PriceChart.jsx
│   │   │   ├── StatsCard.jsx
│   │   │   └── TechnicalIndicators.jsx
│   │   ├── watchlist/
│   │   │   ├── WatchlistTable.jsx
│   │   │   └── WatchlistCard.jsx
│   │   └── auth/
│   │       ├── LoginForm.jsx
│   │       └── RegisterForm.jsx
│   ├── pages/
│   │   ├── DashboardPage.jsx     # 儀表板 (顯示指數概覽與觀察清單)
│   │   ├── StockDetailPage.jsx   # 美股/加密貨幣詳情頁
│   │   ├── LoginPage.jsx
│   │   ├── RegisterPage.jsx
│   │   └── WatchlistPage.jsx
│   ├── hooks/
│   │   ├── useAuth.js
│   │   ├── useStockData.js
│   │   └── useWatchlist.js
│   ├── services/
│   │   ├── api.js
│   │   ├── stockService.js
│   │   ├── authService.js
│   │   └── watchlistService.js
│   ├── store/
│   │   ├── index.js
│   │   ├── authSlice.js
│   │   ├── stockSlice.js
│   │   └── watchlistSlice.js
│   ├── utils/
│   │   ├── formatters.js
│   │   └── validators.js
│   ├── styles/
│   │   └── globals.css
│   ├── App.jsx
│   └── main.js
├── public/
│   └── index.html
├── package.json
└── Dockerfile

supabase/
├── migrations/               # 數據庫遷移文件
├── functions/              # Supabase Edge Functions
└── seeders.sql             # 初始數據

docker-compose.yml
README.md
.gitignore
```

**Structure Decision**: 採用前後端分離的Web應用程式架構，backend目錄包含FastAPI應用程式，frontend目錄包含React應用程式。這種結構支持獨立開發和部署前後端，並符合現代Web應用程式的最佳實踐。

## Complexity Tracking

*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |
