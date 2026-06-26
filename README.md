# 팝업 재고 로케이션 매칭 웹사이트

팝업 스토어 운영을 위한 재고 위치 관리 시스템입니다.  
Node.js (Express) + SQLite (better-sqlite3) + Vanilla JS + Tailwind CDN 기반의 단일 페이지 앱입니다.

## 📋 주요 기능

- 📦 **상품등록**: 바코드 + 로케이션 + 수량 등록 (창고/매장 등)
- 🔍 **재고조회**: 바코드 / 상품명 / 로케이션 부분검색, 위치별 보관 수량 표시
- 🔄 **상품이동(창고-매장)**: 위치 간 재고 이동 + 이력 보존
- ➕ **재고추가**: 보충 입고
- 📊 **현황**: 전체 통계, 거래 내역, CSV 다운로드

## 🛠 기술 스택

| 영역 | 기술 |
|---|---|
| 백엔드 | Node.js 18+, Express 4 |
| DB | SQLite (better-sqlite3, WAL 모드) |
| 프론트엔드 | 단일 HTML + Vanilla JS + Tailwind CDN |
| 바코드 스캔 | html5-qrcode (카메라 권한 필요) |

## 🚀 로컬 실행

```bash
# 1) 의존성 설치 (better-sqlite3는 네이티브 빌드)
npm install

# 2) 서버 실행 (PORT 환경변수 가능, 기본 3000)
npm start

# 3) 브라우저에서 접속
# http://localhost:3000
```

첫 실행 시 `popup_location.xlsx`(576 SKU)를 읽어 `popup.db`에 자동 적재합니다.  
`popup.db`가 이미 비어있지 않으면 자동 스킵됩니다.

## ☁️ Render 배포

이 저장소는 Render의 **Web Service**로 그대로 배포할 수 있습니다.

### 1) GitHub에 푸시

```bash
git init
git add .
git commit -m "초기 커밋: 팝업 재고 로케이션 매칭 웹사이트"
git branch -M main
git remote add origin https://github.com/tnals23000-ops/popup-inventory.git
git push -u origin main
```

### 2) Render Web Service 설정

| 항목 | 값 |
|---|---|
| **Environment** | Node |
| **Build Command** | `npm install` |
| **Start Command** | `npm start` |
| **Instance Type** | Free 가능 |
| **Region** | Singapore 권장 |
| **Health Check Path** | `/health` |

### 3) 환경변수

- `PORT` — Render가 자동 주입 (별도 설정 불필요)
- 추가 환경변수는 필요 없습니다.

### 4) 영속 디스크 (선택, 권장)

Render Free 플랜은 디스크가 임시(ephemeral)라서 재시작 시 `popup.db`가 초기 커밋 상태로 돌아갑니다.  
**데이터 영속이 필요하면 Starter 플랜 + Persistent Disk**를 다음과 같이 설정하세요.

| 항목 | 값 |
|---|---|
| Mount Path | `/var/data` |
| Size | 1 GB |

그리고 `server.js`의 `XLSX_PATH` 위쪽에 다음과 같이 DB 경로를 디스크로 옮기는 한 줄을 추가합니다.

```js
const DB_PATH = process.env.DB_PATH || path.join(__dirname, 'popup.db');
// 그리고 new Database('popup.db') → new Database(DB_PATH) 로 변경
```

환경변수 `DB_PATH=/var/data/popup.db` 추가.

## 📂 디렉터리 구조

```
.
├── server.js              # Express 서버 + 모든 API (~35KB)
├── package.json           # Node 의존성 및 start 스크립트
├── popup.db               # SQLite DB (587 SKU, 924개 위치 적재 + 미지정 619)
├── popup_location.xlsx    # 기초재고 마스터 (576 SKU, 1,529개)
├── public/
│   └── index.html         # 단일 페이지 앱 UI (~70KB)
├── .gitignore
└── README.md
```

## 🔑 핵심 API

| Method | Path | 설명 |
|---|---|---|
| GET | `/health` | 헬스체크 |
| GET | `/api/stats` | 전체 통계 (totalProducts, totalCurrentStock, mismatchCount 등) |
| GET | `/api/product/:barcode` | 단일 상품 조회 (stockByLocation 포함) |
| GET | `/api/search/product?q=...` | 상품 부분검색 (stockByLocation 포함) |
| GET | `/api/locations` | 전체 로케이션 목록 |
| POST | `/api/assign` | 상품등록 (바코드 + 위치 + 수량) |
| POST | `/api/move` | 위치 간 이동 |
| POST | `/api/restock` | 재고 추가 |
| POST | `/api/sale` | 출고 (-) |
| POST | `/api/cancel/:transactionId` | 거래 취소 |
| GET | `/api/transactions` | 거래 이력 |
| GET | `/api/export/csv` | 전체 CSV 다운로드 |

CORS는 모든 도메인 허용입니다.

## 🗃 DB 스키마

- `products`: 마스터(바코드, 스타일, 컬러, 사이즈, 초도재고, 현재재고)
- `stock_by_location`: 바코드별/위치별 보관 수량 (핵심 테이블)
- `transactions`: 모든 거래 이력(ASSIGN/MOVE/RESTOCK/SALE/CANCEL)
- `cancelled_transactions`: 취소된 거래 추적

## 📊 현재 데이터 상태

- 마스터 SKU: 587종 (초도 576 + 추가 11)
- 총 재고: 1,543개
- 위치 분포: 924개 (39개 위치) + 미지정 619개
- 위치명 규칙: `창고-XX-스타일-사람-구역` 또는 `매장-...`

## 📝 라이선스

내부 운영용 (비상업).
