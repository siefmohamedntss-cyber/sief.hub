# DataPilot AI — AI Data Cleaning & Analysis Dashboard

A local-first MVP for uploading CSV/Excel files, profiling data quality, reviewing cleaning operations, exploring charts, exporting cleaned data, and generating a rule-based analytical report. An external LLM is intentionally optional.

## Features
- CSV and Excel (`.xlsx`, `.xls`) upload with size/row/column limits
- Data preview, column types, missing-value and duplicate profiling
- Reviewable cleaning operations: trim text, normalize text case, drop exact duplicates, fill missing numeric values with the median, and flag outliers
- Versioned in-memory dataset state for a running server process; original data is preserved
- Metrics, chart-ready summaries, and a transparent data-quality score
- CSV and Excel export
- AI-style report generated from computed statistics (no paid API required)
- FastAPI backend, React + TypeScript + Vite frontend, tests

## Requirements
- Python 3.11+
- Node.js 20+
- Windows, macOS, or Linux

## Run locally

### 1. Backend
Windows PowerShell:
```powershell
cd backend
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
uvicorn app.main:app --reload
```

macOS/Linux:
```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
uvicorn app.main:app --reload
```
API docs: http://127.0.0.1:8000/docs

### 2. Frontend (in a second terminal)
```bash
cd frontend
npm install
npm run dev
```
Open the URL Vite prints, usually http://localhost:5173.

## First workflow
1. Upload `sample_data/messy_sales.csv`.
2. Review profile metrics and issues.
3. Select a cleaning operation and click **Apply selected cleaning**.
4. Inspect the updated profile and charts.
5. Download CSV/Excel or generate the analysis report.

## Important MVP limitations
- Dataset state and versions are held in server memory. Restarting the backend clears uploaded data. Do not use this prototype for sensitive or production data.
- Authentication, persistent database storage, background jobs, and production object storage are not included yet.
- The report generator is deterministic and local; it is not an external generative AI model. To add a real LLM, implement a server-side provider in `backend/app/services/reporting.py`, validate its output, and keep API keys out of frontend code.
- Upload limits are configured in `backend/app/config.py`. CSV and Excel files are untrusted input; only supported formats are parsed, and formulas/macros are not executed by this app.
- This is an educational prototype, not a certified data governance product.

## Project layout
```text
datapilot-ai/
├── backend/
│   ├── app/
│   │   ├── api/routes.py
│   │   ├── config.py
│   │   ├── main.py
│   │   ├── schemas.py
│   │   └── services/
│   │       ├── analytics.py
│   │       ├── cleaning.py
│   │       ├── datasets.py
│   │       └── reporting.py
│   ├── tests/test_api.py
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── api.ts
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── styles.css
│   └── package.json
├── sample_data/messy_sales.csv
└── docs/ROADMAP.md
```

## API overview
- `GET /api/health`
- `POST /api/datasets/upload`
- `GET /api/datasets`
- `GET /api/datasets/{id}/preview?limit=50&offset=0`
- `GET /api/datasets/{id}/profile`
- `POST /api/datasets/{id}/clean`
- `GET /api/datasets/{id}/history`
- `GET /api/datasets/{id}/charts`
- `POST /api/datasets/{id}/report`
- `GET /api/datasets/{id}/export?format=csv|xlsx`

## Cleaning safety
Operations are allow-listed. The backend never executes user-supplied Python or expressions. Every applied operation creates a new version from the current version and retains the original DataFrame in memory. Outliers are flagged rather than automatically removed. Missing values are not automatically filled without the user selecting that operation.

## Development
```bash
cd backend
pytest
```
