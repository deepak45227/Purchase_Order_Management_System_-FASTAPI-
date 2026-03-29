# Purchase Order Management System (FastAPI)

A full-stack Purchase Order Management System with:
- FastAPI backend
- PostgreSQL database
- Vanilla JavaScript frontend (served by FastAPI)

This project is designed to manage vendors, products, and purchase orders with role-based access, stock control, and approval workflow.

## Project Features

### 1. Authentication and Roles
- JWT login (`/api/auth/login`)
- Current user endpoint (`/api/auth/me`)
- Role-based permissions:
- `admin`, `manager`, `viewer`
- `manager/admin` required for create/update/delete actions

### 2. Vendor Management
- Create, read, update vendors
- Soft delete (deactivate vendor instead of hard delete)
- Search and pagination support
- `active_only=true` filter by default in list API

### 3. Product Management
- Create, read, update products
- Soft delete (deactivate product)
- Search by name/SKU and filter by category
- `active_only=true` filter by default in list API

### 4. Purchase Order Engine
- Create PO with multiple line items
- Auto-generate reference number (`PO-YYYYMMDD-XXXX`)
- Snapshot item prices at PO creation
- Auto-calculate:
- subtotal
- fixed 5% tax
- grand total

### 5. Approval Workflow and Business Rules
- PO status lifecycle:
- `DRAFT -> PENDING -> APPROVED -> RECEIVED`
- `PENDING -> REJECTED`
- cancellation rules included
- Stock deduction on `APPROVED`
- Stock restore when cancelling an approved PO
- Prevent delete unless PO is `DRAFT`

### 6. Dashboard Metrics
- Total PO count
- Status-wise summary
- Total PO value
- Recent purchase orders

### 7. Optional AI Description
- Endpoint to generate product description (`/api/ai/generate-description`)
- Uses Anthropic API key if configured
- Safe fallback template if key is missing/fails

## Tech Stack
- Backend: FastAPI, SQLAlchemy, Pydantic, Uvicorn
- Database: PostgreSQL
- Auth: python-jose (JWT), passlib bcrypt
- Frontend: HTML, CSS, JavaScript

## API Modules
- `/api/auth`
- `/api/vendors`
- `/api/products`
- `/api/purchase-orders`
- `/api/ai`

Interactive API docs are available at `/docs`.

## Project Structure

```text
git-ready-copy/
|-- backend/
|   |-- core/
|   |-- db/
|   |-- models/
|   |-- routers/
|   |-- schemas/
|   |-- main.py
|   |-- requirements.txt
|-- frontend/
|   |-- templates/index.html
|   |-- static/css/main.css
|   |-- static/js/
|-- schema_and_seed.sql
|-- docker-compose.yml
|-- .env.example
|-- README.md
```

## Local Setup (Windows)

### 1. Create database
Create PostgreSQL database:

```sql
CREATE DATABASE po_management;
```

### 2. Import schema and seed data

```powershell
psql "postgresql://postgres:root@localhost:5432/po_management" -f schema_and_seed.sql
```

### 3. Install backend dependencies

```powershell
cd backend
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

### 4. Configure environment
Create project-root `.env` (same level as `backend/` and `frontend/`):

```env
DATABASE_URL=postgresql://postgres:root@localhost:5432/po_management
JWT_SECRET_KEY=change-this-in-production
ANTHROPIC_API_KEY=
APP_ENV=development
APP_PORT=8000
```

### 5. Run the application

```powershell
cd backend
venv\Scripts\activate
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

Open:
- App: `http://localhost:8000`
- Swagger: `http://localhost:8000/docs`
- Health: `http://localhost:8000/health`

## Demo Accounts
- `admin / admin123`
- `manager / manager123`
- `viewer / viewer123`

## SQL and Data Notes
- `schema_and_seed.sql` is idempotent for repeated runs.
- It enforces `is_active` defaults and repairs old `NULL` active flags.
- Vendor/Product list endpoints show active records by default.

If data exists in DB but does not show in UI, check `is_active` values:

```sql
SELECT id, name, is_active FROM vendors;
SELECT id, name, is_active FROM products;
```

## Git Push Workflow

### Initial push (already done for this repo)
- Remote: `origin`
- Branch: `main`

### Push new updates

```powershell
cd git-ready-copy
git add .
git commit -m "Describe your change"
git push origin main
```

## Deploy on Render (Make It Live)

This project works well as one Render Web Service + one Render PostgreSQL database.

### Step 1. Push latest code to GitHub

```powershell
cd git-ready-copy
git add .
git commit -m "prepare for render deploy"
git push origin main
```

### Step 2. Create PostgreSQL on Render
1. Open Render Dashboard.
2. Create `New -> PostgreSQL`.
3. Choose name, region, and plan.
4. After creation, copy the **Internal Database URL**.

### Step 3. Create Web Service on Render
1. Create `New -> Web Service`.
2. Connect your GitHub repo.
3. Use these settings:
- Runtime: `Python 3`
- Root Directory: leave empty (repo root)
- Build Command: `pip install -r backend/requirements.txt`
- Start Command: `cd backend && uvicorn main:app --host 0.0.0.0 --port $PORT`
- Health Check Path: `/health`

### Step 4. Add environment variables
In Render Web Service -> Environment, set:

```env
DATABASE_URL=<Render Internal Database URL>
JWT_SECRET_KEY=<long-random-secret>
ANTHROPIC_API_KEY=<optional>
APP_ENV=production
PYTHON_VERSION=3.11.11
```

### Step 5. Initialize database schema on Render
Run once from your local machine using Render's **External Database URL**:

```powershell
psql "<render-external-database-url>" -f schema_and_seed.sql
```

### Step 6. Verify deployment
After deploy:
- Open `https://<your-service>.onrender.com`
- Open `https://<your-service>.onrender.com/docs`
- Test login with demo credentials

## Add Screenshots and Videos
- Put screenshots in `docs/images/`
- Put GIF/video files in `docs/videos/`

Examples:

```md
![Dashboard](docs/images/dashboard.png)
![PO Flow](docs/videos/po-flow.gif)
[Watch full demo](docs/videos/demo.mp4)
```

## Useful Render References
- Deploy FastAPI quickstart: https://render.com/docs/deploy-fastapi
- Web services and port behavior: https://render.com/docs/web-services
- Deploy steps (build/pre-deploy/start): https://render.com/docs/deploys
- Python version setting: https://render.com/docs/python-version

