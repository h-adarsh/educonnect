# EduConnect — EdTech CRM & Public Inquiry System

## Streamlined Lead Capture and Real-Time Inquiry Management for Educational Institutions

EduConnect is an open-source, full-stack CRM and inquiry management system built specifically for educational institutions, academies, and universities. It bridges the gap between prospective applicants—such as parents, students, and partner schools—and admissions or administrative teams. Prospective leads can seamlessly submit inquiries categorized by topic (admissions, scholarships, campus events, and general queries), while administrative officers gain access to a secure, role-restricted CRM dashboard with custom JWT authentication, multi-criteria filtering, live debounced search, and status lifecycle management (`new`, `in_progress`, `resolved`).

---

### Tech Stack

- **Client** - React 18, Vite, Tailwind CSS, React Router v6, Lucide Icons
- **Server** - Nodejs, Express, PostgreSQL (Supabase), JWT (`jsonwebtoken` + `bcryptjs`), Zod, Helmet, Express Rate Limit

---

## System Architecture & Workflow

The diagram below illustrates how inquiries flow from public consumers into the system and how administrative personnel interact with the CRM:

```mermaid
flowchart TD
    subgraph Public_Users["Public Users (Parents, Students, Schools)"]
        A["Public Inquiry Form (React / Tailwind)"]
    end

    subgraph Backend_Gateway["Express API Server"]
        B["Rate Limiter & Security Headers (Helmet)"]
        C["Zod Validation Middleware"]
        D["Public Inquiries Controller"]
        E["Custom JWT Auth Guard (requireAuth)"]
        F["Admin Inquiries & Auth Controller"]
    end

    subgraph Data_Layer["Database"]
        G[("Supabase PostgreSQL Database")]
    end

    subgraph Admin_Staff["Admissions & Admin Staff"]
        H["Admin CRM Dashboard (React SPA)"]
    end

    A -->|"POST /api/inquiries"| B
    B --> C
    C --> D
    D -->|"Insert Inquiry"| G

    H -->|"POST /api/auth/login"| B
    B --> F
    F -->|"Verify bcrypt Hash"| G
    F -->|"Issue Signed JWT"| H

    H -->|"GET /api/inquiries (Bearer Token)"| E
    H -->|"PATCH /api/inquiries/:id/status"| E
    E --> F
    F -->|"Query / Update"| G
```

---

## Installation and Usage Instructions (For End-Users)

If you are an evaluator, admissions officer, or consumer who wants to run and explore EduConnect on your machine without modifying the source code, follow these quick steps:

### 1. Quick Launch (Run Locally)

Ensure you have **Node.js (v18+)** installed. In your terminal, run:

```bash
# 1. Clone the repository
git clone https://github.com/h-adarsh/edtech-crm-inquiry-system.git
cd edtech-crm-inquiry-system

# 2. Start the Backend API (Terminal 1)
cd backend
npm install
npm start

# 3. Start the Frontend Client (Terminal 2)
cd frontend
npm install
npm run dev
```

Once running, access the interfaces directly in your browser:
- **Public Inquiry Form:** [http://localhost:5173](http://localhost:5173)
- **Admin CRM Login:** [http://localhost:5173/login](http://localhost:5173/login)
- **Backend API Status:** [http://localhost:5050/api/health](http://localhost:5050/api/health)

### 2. Demo Admin Credentials

You can sign in to the administrative portal using the pre-seeded account:
- **Email:** `admin@edtech.com`
- **Password:** `AdminPassword123!`
*(The Admin Login page includes a one-click **"Fill Demo Admin"** button for quick testing).*

### 3. How to Use the System as a Consumer

#### Submitting a Public Inquiry
1. Navigate to the home page (`/`).
2. Select your **User Type** (`Parent`, `Student`, or `School`).
3. Select an **Inquiry Category** (`Scholarship`, `Concert`, `Admission`, or `General Query`).
4. Enter your contact details (Name, Email, Phone number) and your inquiry message.
5. Click **Submit Inquiry**. You will receive an instant confirmation upon submission.

#### Managing Inquiries as an Administrator
1. Navigate to `/login` and sign in with your admin credentials.
2. Once on the **Dashboard** (`/dashboard`):
   - **Filter by Category:** Narrow down leads by Admissions, Scholarships, Concerts, or General Queries.
   - **Filter by User Type:** Isolate inquiries submitted by Parents, Students, or Schools.
   - **Live Search:** Type a candidate's name or phone number into the search bar (results update dynamically).
   - **Update Status:** Change an inquiry's status directly from the dropdown to `new`, `in_progress`, or `resolved`.
   - **Pagination:** Navigate between result pages using the footer pagination controls.

#### Consuming via REST API (Direct Client Usage)
End-users and client integrations can interact with the API endpoints directly:

- **Submit an Inquiry (Public):**
  ```bash
  curl -i -X POST http://localhost:5050/api/inquiries \
    -H "Content-Type: application/json" \
    -d '{
      "user_type": "Parent",
      "category": "Admission",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "phone": "+91 98765 43210",
      "message": "Looking for admission guidelines for Grade 11."
    }'
  ```

- **Authenticate as Admin:**
  ```bash
  curl -i -X POST http://localhost:5050/api/auth/login \
    -H "Content-Type: application/json" \
    -d '{
      "email": "admin@edtech.com",
      "password": "AdminPassword123!"
    }'
  ```

- **Query Inquiries with Bearer Token:**
  ```bash
  curl -i -X GET "http://localhost:5050/api/inquiries?status=new&page=1&limit=10" \
    -H "Authorization: Bearer <YOUR_JWT_TOKEN>"
  ```

---

## Installation and Usage Instructions (For Contributors)

Follow these technical instructions to clone the repository, set up the development environment, execute database migrations, and run the complete stack locally.

### Prerequisites

- **Node.js:** v18.0.0 or higher (v20+ recommended)
- **npm:** v9.0.0 or higher
- **PostgreSQL Database:** A PostgreSQL instance or a free [Supabase](https://supabase.com) project.

---

### Step 1: Clone the Repository

```bash
git clone https://github.com/h-adarsh/edtech-crm-inquiry-system.git
cd edtech-crm-inquiry-system
```

---

### Step 2: Database Provisioning

1. Log into your [Supabase Dashboard](https://supabase.com/dashboard) and create a new project (or use local PostgreSQL).
2. Open the **SQL Editor**.
3. Copy the contents of [`schema.sql`](file:///Users/adarshsingh/Desktop/projects/edtech-crm-inquiry-system/schema.sql) from the root directory, paste into the editor, and click **Run**.
4. Retrieve your **PostgreSQL Connection String / Pooler URI** from **Project Settings** → **Database**.

---

### Step 3: Backend Setup & Seed Scripts

1. Navigate to the backend directory and copy the environment template:
   ```bash
   cd backend
   cp .env.example .env
   ```

2. Configure `backend/.env` with your credentials:
   ```env
   DATABASE_URL=postgresql://postgres.[YOUR_PROJECT_REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres
   JWT_SECRET=super_secret_jwt_key_at_least_32_characters_long
   JWT_EXPIRES_IN=8h
   PORT=5050
   NODE_ENV=development
   FRONTEND_URL=http://localhost:5173
   ```

3. Install dependencies:
   ```bash
   npm install
   ```

4. Seed the default admin user and demo inquiries:
   ```bash
   npm run seed:admin
   npm run seed:demo
   ```

5. Start the backend development server:
   ```bash
   npm run dev
   ```
   The API will be available at `http://localhost:5050`. Verify health at `http://localhost:5050/api/health`.

---

### Step 4: Frontend Setup

1. Open a new terminal tab, navigate to the frontend directory, and set up `.env`:
   ```bash
   cd frontend
   cp .env.example .env
   ```

2. Configure `frontend/.env`:
   ```env
   VITE_API_URL=http://localhost:5050/api
   ```

3. Install dependencies and start the Vite dev server:
   ```bash
   npm install
   npm run dev
   ```
   The client application will run at `http://localhost:5173`.

---

### Step 5: Running Automated Tests

Run the backend test suite (testing custom JWT authentication, Zod validation, parameter binding, rate limiting, and status lifecycle):

```bash
cd backend
npm test
```

---

## Contributor Expectations

We welcome contributions to EduConnect! To maintain code quality, security, and consistent architecture, please adhere to the following guidelines:

### 1. Workflow & Issues First
- Before starting work on major features or architectural refactors, **open an issue** to discuss the proposed change.
- Once agreed upon, assign the issue to yourself or mention you are working on it.

### 2. Branch Naming Conventions
Create a descriptive branch for your work:
- `feature/<short-description>` — For new functionality (e.g., `feature/export-csv`)
- `fix/<short-description>` — For bug fixes (e.g., `fix/inquiry-phone-regex`)
- `docs/<short-description>` — For documentation improvements (e.g., `docs/api-specs`)
- `refactor/<short-description>` — For code cleanup without changing external behavior

### 3. Commit Message Standards
Follow [Conventional Commits](https://www.conventionalcommits.org/):
- `feat: add export inquiries to CSV functionality`
- `fix: prevent duplicate form submissions during network latency`
- `test: add unit tests for query string sanitization`
- `docs: update contributor setup steps for Docker`

### 4. Code Quality & Security Ground Rules
- **No Direct Supabase Auth:** Do not introduce third-party proprietary auth SDKs into the backend; all administrative authentication must continue using custom JWT and `bcryptjs`.
- **Parameterized SQL Only:** Always use parameterized queries (`$1`, `$2`, ...) to protect against SQL injection vulnerabilities.
- **Strict Input Validation:** All newly introduced endpoints must define declarative Zod schemas in `schemas/`.
- **Tests Required:** Any new backend route or significant business logic change must be accompanied by tests in `backend/tests/`.

### 5. Submitting Pull Requests
1. Ensure `npm test` passes in `backend/`.
2. Ensure the frontend builds cleanly without errors (`npm run build`).
3. Push your branch and open a Pull Request targeting the `main` branch.
4. Fill out the PR description with:
   - Linked issue number (`Closes #<issue_number>`)
   - Summary of changes
   - Manual testing steps or screenshots of UI changes
5. Request a review from the maintainers. Keep commits clean and squashed where appropriate before merging.
