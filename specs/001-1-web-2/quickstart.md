# 快速開始指南: 美股智慧分析儀表板

**建立日期**: 2025-10-15  
**目的**: 提供開發者快速設置和運行美股智慧分析儀表板系統的指南

## 系統需求

### 基本要求
- Node.js 18+ 
- Python 3.9+
- PostgreSQL 13+
- Git
- Docker (可選，用於容器化部署)

### 開發工具推薦
- VS Code 或其他現代代碼編輯器
- Postman 或 Insomnia (用於API測試)
- pgAdmin 或 DBeaver (用於PostgreSQL管理)

## 項目結構

```
buy-what/
├── backend/                    # FastAPI 後端應用
│   ├── src/
│   │   ├── models/            # 數據模型
│   │   ├── services/          # 業務邏輯
│   │   ├── api/               # API端點
│   │   └── core/              # 核心配置
│   ├── tests/                 # 測試文件
│   ├── requirements.txt       # Python依賴
│   └── Dockerfile            # Docker配置
├── frontend/                   # React 前端應用
│   ├── src/
│   │   ├── components/        # React組件
│   │   ├── pages/             # 頁面組件
│   │   ├── hooks/             # 自定義Hooks
│   │   ├── services/          # API服務
│   │   └── store/             # 狀態管理
│   ├── public/                # 靜態資源
│   ├── package.json           # Node.js依賴
│   └── Dockerfile            # Docker配置
├── specs/001-1-web-2/         # 規格和設計文檔
│   ├── spec.md                # 功能規格
│   ├── plan.md                # 實施計劃
│   ├── research.md            # 研究結果
│   ├── data-model.md          # 數據模型
│   ├── quickstart.md          # 本文件
│   └── contracts/            # API契約
├── docker-compose.yml          # Docker Compose配置
└── README.md                  # 項目說明
```

## 設置步驟

### 1. 克隆項目

```bash
git clone <repository-url>
cd buy-what
```

### 2. 設置PostgreSQL數據庫（通過Supabase）

#### 使用Supabase設置PostgreSQL
1. 訪問 [Supabase](https://supabase.com) 並創建新帳戶
2. 創建新項目，命名為 `stock-dashboard`
3. 記錄項目URL和連接字符串（在項目設置 > Database中找到）

#### 後端數據庫連接設置
```bash
cd backend

# 設置環境變數
cp .env.example .env
# 編輯 .env 文件，添加Supabase連接字符串
DATABASE_URL=postgresql://postgres:password@db.your-project-ref.supabase.co:5432/postgres
```

### 3. 設置後端

```bash
cd backend

# 創建虛擬環境
python -m venv venv

# 激活虛擬環境
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

# 安裝依賴
pip install -r requirements.txt

# 設置環境變數
cp .env.example .env
# 編輯 .env 文件，配置數據庫連接和其他設置

# 運行數據庫遷移
alembic upgrade head

# 啟動開發服務器
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

後端API文檔將在 http://localhost:8000/docs 可用

### 4. 設置前端

```bash
cd frontend

# 安裝依賴
npm install

# 設置環境變數
cp .env.example .env.local
# 編輯 .env.local 文件，配置API端點

# 啟動開發服務器
npm run dev
```

前端應用將在 http://localhost:5173 可用

### 5. 使用Docker Compose（可選）

```bash
# 啟動所有服務
docker-compose up -d

# 查看日誌
docker-compose logs -f

# 停止服務
docker-compose down
```

## 開發工作流程

### 1. 後端開發

#### 添加新的API端點
1. 在 `backend/src/api/endpoints/` 中創建新的端點文件
2. 在 `backend/src/services/` 中實現業務邏輯
3. 在 `backend/src/models/` 中定義數據模型
4. 添加相應的測試到 `backend/tests/`
5. 更新API文檔（FastAPI自動生成）

#### 數據庫遷移
```bash
# 創建新的遷移
alembic revision --autogenerate -m "描述變更"

# 應用遷移
alembic upgrade head
```

#### 運行測試
```bash
# 運行所有測試
pytest

# 運行特定測試文件
pytest tests/test_stocks.py

# 生成覆蓋率報告
pytest --cov=src tests/
```

### 2. 前端開發

#### 添加新組件
1. 在 `frontend/src/components/` 中創建新組件
2. 在 `frontend/src/pages/` 中創建頁面組件
3. 更新路由配置（如果需要）
4. 添加相應的測試
5. 更新狀態管理（如果需要）

#### API集成
```javascript
// frontend/src/services/api.js
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.REACT_APP_API_URL || 'http://localhost:8000/api/v1',
  headers: {
    'Content-Type': 'application/json',
  },
});

// 添加請求攔截器處理認證
api.interceptors.request.use((config) => {
  const token = localStorage.getItem('access_token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
```

#### 運行測試
```bash
# 運行所有測試
npm test

# 運行測試並生成覆蓋率報告
npm run test:coverage
```

## 常見開發任務

### 1. 添加新的股票指標

#### 後端
1. 在 `backend/src/services/stock_service.py` 中添加指標計算邏輯
2. 更新 `backend/src/models/stock.py` 中的相應模型
3. 在API端點中暴露新指標

#### 前端
1. 在 `frontend/src/components/stock/` 中創建顯示組件
2. 更新 `frontend/src/services/stockService.js` 以獲取新數據
3. 在相應頁面中集成新組件

### 2. 添加新的技術指標

#### 後端
1. 在 `backend/src/services/technical_analysis.py` 中實現指標計算
2. 使用 `pandas-ta` 函式庫進行計算
3. 更新API響應模型

#### 前端
1. 在 `frontend/src/components/stock/TechnicalIndicators.jsx` 中添加新指標
2. 更新圖表配置以顯示新指標
3. 添加用戶界面控制項以切換指標顯示

### 3. 實現用戶認證

#### 後端
1. 實現JWT令牌生成和驗證
2. 添加密碼哈希和驗證
3. 創建認證中間件
4. 實現刷新令牌機制

#### 前端
1. 創建登入和註冊表單
2. 實現認證狀態管理
3. 添加路由保護
4. 實現自動令牌刷新

## 部署

### 1. 生產環境設置

#### 後端部署
```bash
# 構建Docker鏡像
docker build -t stock-dashboard-backend ./backend

# 運行容器
docker run -d \
  --name stock-backend \
  -e DATABASE_URL=postgresql://user:password@host:port/dbname \
  -e SECRET_KEY=your-secret-key \
  -p 8000:8000 \
  stock-dashboard-backend
```

#### 前端部署
```bash
# 構建生產版本
cd frontend
npm run build

# 使用Nginx或其他Web服務器提供靜態文件
docker build -t stock-dashboard-frontend ./frontend
docker run -d --name stock-frontend -p 80:80 stock-dashboard-frontend
```

### 2. 環境變量配置

#### 後端 (.env)
```
DATABASE_URL=postgresql://user:password@localhost:5432/stock_dashboard
SECRET_KEY=your-super-secret-key
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7
YFINANCE_CACHE_TTL=300
REDIS_URL=redis://localhost:6379

# Supabase連接（用於開發環境）
DATABASE_URL=postgresql://postgres:password@db.your-project-ref.supabase.co:5432/postgres
```

#### 前端 (.env.local)
```
VITE_API_URL=http://localhost:8000/api/v1
VITE_APP_VERSION=1.0.0
```

#### 未來遷移到AWS RDS
計劃未來從Supabase PostgreSQL遷移到AWS RDS，保持相同的表結構和數據

## 故障排除

### 常見問題

#### 1. 後端啟動失敗
- 檢查Python版本是否為3.9+
- 確認所有依賴已正確安裝
- 檢查數據庫連接配置
- 查看終端中的錯誤消息

#### 2. 前端無法連接後端
- 確認後端服務正在運行
- 檢查CORS配置
- 驗證API端點URL
- 查看瀏覽器控制台錯誤

#### 3. 數據庫連接問題
- 確認PostgreSQL服務正在運行
- 檢查數據庫連接字符串
- 驗證數據庫用戶權限
- 確認數據庫已創建

#### 4. 股票數據獲取失敗
- 檢查yfinance函式庫版本
- 確認網絡連接
- 驗證股票代碼格式
- 查看API速率限制

### 調試技巧

#### 後端調試
```python
# 在代碼中添加日誌
import logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

logger.info("這是一條調試信息")
```

#### 前端調試
```javascript
// 使用console.log進行調試
console.log("調試信息:", data);

// 使用React DevTools
// 安裝瀏覽器擴展程序進行React組件調試
```

## 貢獻指南

1. Fork項目
2. 創建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 開啟Pull Request

### 代碼風格

#### Python (後端)
- 遵循PEP 8風格指南
- 使用Black進行代碼格式化
- 使用flake8進行代碼檢查

#### JavaScript (前端)
- 使用ESLint和Prettier
- 遵循Airbnb JavaScript風格指南
- 使用組件和函數式編程模式

## 許可證

本項目採用MIT許可證 - 詳見 [LICENSE](LICENSE) 文件

## 聯繫方式

- 項目維護者: [維護者姓名]
- 電子郵件: support@stockdashboard.com
- 項目主頁: https://github.com/username/stock-dashboard

## 更多資源

- [FastAPI文檔](https://fastapi.tiangolo.com/)
- [React文檔](https://reactjs.org/)
- [PostgreSQL文檔](https://www.postgresql.org/docs/)
- [yfinance文檔](https://pypi.org/project/yfinance/)
- [Recharts文檔](https://recharts.org/)