# ![Logo Placeholder](./frontend/src/assets/Cognis.jpg ":size=120x120")

# **Cognis**

### *AI-Powered Digital Forensic Intelligence Platform*

---

### 🎥 Project Demo  
[![YouTube Video](https://img.shields.io/badge/Watch%20Demo-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://youtu.be/ooD6o1OPmwo)

## 1. Overview

**Cognis** is a full-stack digital forensic intelligence platform built for **law enforcement**, **cybercrime units**, and **digital forensic investigators**. It automates the ingestion, classification, and analysis of **UFDR (Universal Forensic Data Reports)** from mobile and digital devices.

Cognis provides:

* Secure **case management** with investigator assignments
* **AI-assisted querying** and summarization of forensic evidence
* **PDF report generation** for judicial documentation
* **Role-based dashboards** with global and scoped access
* **Dual storage model** — locally analyzed, remotely preserved via MinIO

---

## 2. Features

### **🔐 Secure Authentication & Role System**

* JWT-based authentication (`/auth/login`)
* Roles: **admin**, **investigator**
* Admins manage users, assign cases, and view all data
* Investigators have scoped access to assigned cases only

### **📁 Centralized Case Management**

* Create and manage cases (`/cases/create`)
* Assign investigators via `CaseAssignment(user_id, case_id)`
* Enforces data access boundaries per investigator

### **📦 Automated UFDR Ingestion with Dual Storage (Local + MinIO)**

* Upload `.zip` UFDR reports via `/ufdr/upload`
* File temporarily stored locally for parsing and analysis
* After extraction and artifact generation, the local copy is deleted
* Finalized copy persists securely in **MinIO object storage** for reference or reanalysis
* Metadata (tool, hash, device type, timestamps) stored in PostgreSQL

### **🧠 AI-Powered Chat Assistant**

* `/chat/ask/{ufdr_id}?q=...` enables natural-language queries
* Integrates **Google Gemini** via FastAPI async wrapper
* Provides structured insights, citations, and summarized outputs

### **📄 Comprehensive Report Generation**

* `/report/{ufdr_id}` → Jinja2 + wkhtmltopdf / WeasyPrint
* Generates branded PDF reports with embedded case metadata

### **📊 Role-Based Dashboards**

* `/dashboard/summary` API with role-specific aggregation
* **Admin:** global analytics | **Investigator:** case-limited view

---

## 3. System Architecture

```text
┌────────────────────────────────────────────────────┐
│                    FRONTEND (React + Vite)         │
│  • Authentication & Role Routing                   │
│  • Case and UFDR Management                        │
│  • AI Chat Interface (Gemini/OpenAI)               │
│  • PDF Report Viewer and Downloader                │
│  • Role-based Dashboards                           │
└────────────────────────────────────────────────────┘
                       │ REST API (JWT)
                       ▼
┌────────────────────────────────────────────────────┐
│                    BACKEND (FastAPI)               │
│  • Routes: auth, cases, ufdr, chat, report, dash   │
│  • ORM: SQLAlchemy (async)                         │
│  • LLM: Gemini / OpenAI Wrapper                    │
│  • Security: JWT, OAuth2PasswordBearer             │
│  • Reports: Jinja2 + wkhtmltopdf / WeasyPrint      │
│  • Cache: Redis (optional)                         │
│  • Storage: MinIO integration for UFDR files       │
└────────────────────────────────────────────────────┘
                        │
                        ▼
┌────────────────────────────────────────────────────┐
│                    DATABASE / STORAGE              │
│  • PostgreSQL + pgvector for embeddings            │
│  • Local FS (temporary UFDR analysis)              │
│  • MinIO (persistent evidence repository)          │
│  • Redis (optional cache)                          │
└────────────────────────────────────────────────────┘
```

---

## 4. Tech Stack

| Layer          | Technologies                                                                |
| -------------- | --------------------------------------------------------------------------- |
| **Frontend**   | React (Vite), TailwindCSS, shadcn/ui, Lucide Icons, ReactMarkdown           |
| **Backend**    | FastAPI (Async), SQLAlchemy ORM, Pydantic, Jinja2, WeasyPrint / wkhtmltopdf |
| **Database**   | PostgreSQL + pgvector                                                       |
| **Storage**    | MinIO (permanent), Local FS (temporary)                                     |
| **AI Layer**   | Google Gemini (FastAPI Wrapper)                                             |
| **Auth**       | JWT (PyJWT), OAuth2PasswordBearer                                           |
| **Cache**      | Redis (optional)                                                            |
| **Deployment** | Gunicorn + Uvicorn Workers (Manual setup, no Docker)                        |

---

## 5. Getting Started

### **Backend Setup (FastAPI)**

1. Navigate to backend:

   ```bash
   cd backend
   ```
2. Create virtual environment:

   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   ```
3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```
4. Create `.env`:

   ```ini
   DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/cognis
   JWT_SECRET=change-this
   JWT_ALGORITHM=HS256
   GEMINI_API_KEY=your-gemini-key
   UPLOAD_DIR=uploads
   MINIO_ENDPOINT=127.0.0.1:9000
   MINIO_ACCESS_KEY=minioadmin
   MINIO_SECRET_KEY=minioadmin
   MINIO_BUCKET=cognis
   ```
5. Run migrations:

   ```bash
   alembic upgrade head
   ```
6. Start server:

   ```bash
   uvicorn app.main:app --reload
   ```

---

### **Frontend Setup (React)**

1. Navigate:

   ```bash
   cd frontend
   ```
2. Install dependencies:

   ```bash
   npm install
   ```
3. Create `.env.local`:

   ```bash
   VITE_API_BASE_URL=http://localhost:8000/api/v1
   ```
4. Run development server:

   ```bash
   npm run dev
   ```

---

## 6. Usage

### Admin Flow

1. Login using default credentials: `admin` / `admin`
2. Create investigator accounts
3. Create cases and assign investigators

### Investigator Flow

1. Login to access assigned cases only
2. Upload UFDR files — automatically stored in both local and MinIO
3. Local file analyzed → extracted → deleted post-analysis
4. Persistent copy remains in MinIO for long-term retrieval
5. Generate and download PDF reports

---

## 7. Database Entities

| Table                | Fields                                 | Purpose                      |
| -------------------- | -------------------------------------- | ---------------------------- |
| **users**            | id, username, role                     | Application users            |
| **cases**            | id, title, created_by                  | Investigation cases          |
| **case_assignments** | case_id, user_id                       | Case–investigator mapping    |
| **ufdr_files**       | id, filename, meta, storage_url        | UFDR uploads and MinIO links |
| **artifacts**        | id, ufdr_file_id, type, extracted_text | Extracted evidence           |
| **chat_sessions**    | id, ufdr_id, history                   | Conversation memory          |

---

## 8. File Layout

```
Cognis/
├── backend/
│   ├── app/
│   │   ├── api/routes/ (auth, cases, ufdr, chat, report, dashboard)
│   │   ├── core/ (security, llm, pdf_renderer, config, storage)
│   │   ├── models/
│   │   ├── utils/
│   │   └── main.py
│   ├── uploads/ (temporary UFDR workspace)
│   └── alembic/
└── frontend/
    ├── src/components/
    ├── src/services/api.js
    └── .env.local
```

---

## 9. Security Model

| Role                 | Permissions                                                                                     |
| -------------------- | ----------------------------------------------------------------------------------------------- |
| **Admin**            | Create users, assign cases, full data access                                                    |
| **Investigator**     | Restricted to assigned cases                                                                    |
| **API**              | JWT-secured routes                                                                              |
| **Data Access**      | Verified via backend joins, not client filters                                                  |
| **Storage Security** | Local temporary files deleted post-analysis; MinIO used for secure, persistent evidence storage |

---

## 10. Testing

Run tests from backend:

```bash
pytest -q
```

---

## 11. .gitignore

```
.env
.vscode/
.idea/
__pycache__/
*.pyc
/uploads/
*.sqlite
frontend/node_modules/
frontend/dist/
```

---

## 12. Roadmap

| Feature           | Description                                     |
| ----------------- | ----------------------------------------------- |
| 🔍 Evidence Graph | Visualize entity relationships                  |
| ☁️ Cloud Storage  | Enhance MinIO integration for multi-region sync |
| 🧾 Audit Logs     | Track investigator queries                      |
| 🧠 Vector Search  | Enable semantic retrieval                       |
| ⚡ Streamed Chat   | Real-time LLM responses                         |

---

## Summary

Cognis is a **production-ready AI forensic platform** combining FastAPI, React, and Gemini with hybrid storage (Local + MinIO) for secure and efficient handling of UFDR-based investigations. The system ensures scalability, reliability, and compliance with forensic evidence retention standards.
