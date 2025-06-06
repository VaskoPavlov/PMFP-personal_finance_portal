## Project Idea: Personal Finance Management Portal

_To showcase my alignment with TBI Bank’s technical requirements—and to make my application stand out—I propose building a “Personal Finance Management Portal.” This prototype demonstrates end-to-end fintech capabilities using JavaScript, .NET, HTML/CSS, jQuery/AJAX, and a relational database, while also highlighting my banking domain knowledge. Below is a structured plan:_

---

### 1. Project Overview & Goals

1. **Account Overview**  
   Authenticated users see all their virtual “bank” accounts (Checking, Savings, Credit).

2. **Transaction History**  
   Users can filter and search transactions by date range, amount, or description.

3. **Internal Transfers**  
   Permits moving funds between user accounts.

4. **Expense Categorization**  
   Allows tagging transactions as “Groceries,” “Utilities,” “Entertainment,” etc., to generate monthly spending reports.

5. **Budget Dashboard**  
   Visual summary of monthly income vs. expenses, using interactive charts.

6. **Secure Authentication & Authorization**  
   Role-based access: **Customer** vs. **Admin**.

---

### 2. Technology Stack

> **See file:** `project_idea.md`

---

### 3. Data Model & Relational Schema

_All tables will be normalized (3NF). Key tables include:_

#### **Users**
- `UserId` (PK, GUID)  
- `Email` (string, unique)  
- `PasswordHash` (string)  
- `FullName` (string)  
- `Role` (enum: Customer, Admin)  
- `CreatedAt` (datetime)

#### **Accounts**
- `AccountId` (PK, GUID)  
- `UserId` (FK → Users.UserId)  
- `AccountType` (enum: Checking, Savings, Credit)  
- `Balance` (decimal)  
- `Currency` (char(3), e.g., “BGN”)  
- `CreatedAt` (datetime)

#### **Categories**
- `CategoryId` (PK, int)  
- `UserId` (FK → Users.UserId)  
- `Name` (string, e.g., “Groceries,” “Utilities”)

#### **Transactions**
- `TransactionId` (PK, GUID)  
- `AccountId` (FK → Accounts.AccountId)  
- `CategoryId` (FK → Categories.CategoryId, nullable)  
- `Amount` (decimal)  
- `TransactionType` (enum: Debit, Credit)  
- `Description` (nvarchar(255))  
- `Timestamp` (datetime)  
- `CreatedAt` (datetime)

#### **AuditLog**
- `LogId` (PK, int, identity)  
- `UserId` (FK → Users.UserId)  
- `Action` (nvarchar(100), e.g., “TransferFunds,” “UpdateProfile”)  
- `Details` (nvarchar(max))  
- `OccurredAt` (datetime)

> **Bonus Tables**

#### **Budgets**
- `BudgetId` (PK, GUID)  
- `UserId` (FK → Users.UserId)  
- `MonthYear` (char(7), e.g., “2025-06”)  
- `TotalBudget` (decimal)

#### **BudgetAllocations**
- `AllocationId` (PK, int, identity)  
- `BudgetId` (FK → Budgets.BudgetId)  
- `CategoryId` (FK → Categories.CategoryId)  
- `AllocatedAmount` (decimal)

---

#### **Stored Procedures & Indexes**

- **`sp_GetAccountSummary(@UserId)`**  
  Returns each account’s current balance.

- **`sp_GetTransactions(@UserId, @FromDate, @ToDate)`**  
  Returns all transactions in date range, ordered by `Timestamp DESC`  
  **Indexed on:** `(AccountId, Timestamp)`

- **`sp_PostTransfer(@FromAccountId, @ToAccountId, @Amount, @UserId)`**  
  1. Check sufficient `Balance` on `FromAccountId`  
  2. Debit `FromAccountId`, Credit `ToAccountId`  
  3. Insert two `Transactions` (one debit, one credit), with same `Timestamp`  
  4. Insert into `AuditLog`  
  5. Wrap in a transaction (`BEGIN TRAN / COMMIT`)

- **`sp_GetMonthlySpending(@UserId, @MonthYear)`** _(Bonus)_  
  Aggregates total debit amounts per category for budget chart.

---

### 4. Back-End Implementation (ASP.NET Core)

#### **Folder Structure**

```plaintext
src/
├── API/
│   ├── Controllers/
│   ├── DTOs/
│   └── Filters/
├── Core/
│   ├── Models/
│   ├── Interfaces/
│   └── Services/
└── Infrastructure/
    ├── Data/
    ├── Repositories/
    └── Migrations/
```

#### **Controllers**
- **AuthController**  
  - `POST /api/auth/register` → Registers new user (optional: activation email)  
  - `POST /api/auth/login` → Issues JWT

- **AccountsController**  
  - `GET /api/accounts/summary` → Calls `sp_GetAccountSummary`  
  - `GET /api/accounts/{id}/transactions` → Calls `sp_GetTransactions`

- **TransactionsController**  
  - `GET /api/transactions?from=YYYY-MM-DD&to=YYYY-MM-DD` → Filtered transactions for all user accounts  
  - `POST /api/transactions/transfer` → Executes `sp_PostTransfer`

- **CategoriesController** _(Bonus)_  
  - `GET /api/categories` → Returns user’s category list  
  - `POST /api/categories` → Adds new category

- **BudgetController** _(Bonus)_  
  - `GET /api/budgets/{monthYear}` → Returns budget allocations and monthly spending (`sp_GetMonthlySpending`)  
  - `POST /api/budgets` → Creates or updates monthly budget

#### **Services & Repositories**
- **IAccountService & AccountService**  
  Wrap calls to stored procedures via EF Core’s `Database.ExecuteSqlRawAsync`.

- **ITransactionService & TransactionService**  
  Business logic for transaction validation, category assignment.

- **IAuthService & AuthService**  
  Handles password hashing (ASP.NET Identity) and JWT issuance.

- **IAuditService & AuditService**  
  Inserts audit entries.

#### **Security**
- Use **ASP.NET Identity** for initial user provisioning.  
- Configure JWT with symmetric key (stored in `appsettings.json`, secured in production).  
- Apply `[Authorize]` on controllers, and `[Authorize(Roles="Admin")]` on admin endpoints.  
- Enforce HTTPS and secure cookie settings.

#### **Error Handling & Logging**
- Add a global exception filter (`ApiExceptionFilter`) to convert exceptions into uniform `ProblemDetails` JSON.  
- Use **Serilog** (or built-in `ILogger`) to write structured logs to Console and a local rolling file.

---

### 5. Front-End Implementation

_While my day-to-day role uses Next.js/React, I will implement a hybrid approach that showcases both modern SPA techniques and classic jQuery/AJAX proficiency—key for TBI Bank’s requirements._

#### 5.1 React + TypeScript (Next.js) Pages

1. **`/pages/index.tsx` (Dashboard)**
   - On component mount (`useEffect`), call `GET /api/accounts/summary` via `fetch` or Axios.  
   - Render account cards (e.g., “1234 ·· ··5678 | BGN 1,250.00”).  
   - Display a mini expense pie chart (using Chart.js) showing percentage spent vs. remaining budget (if budgets are set).

2. **`/pages/transactions.tsx`**
   - Date‐range picker (using a React date‐picker component).  
   - On filter change, call `GET /api/transactions?from=…&to=…`.  
   - Render a table (React component) with columns: Date, Account, Type, Category, Amount, Description.  
   - Allow sorting by clicking column headers (React state).

3. **`/pages/transfer.tsx` (Payments)**
   - Form fields:  
     - **From Account** (dropdown populated via `GET /api/accounts/summary`)  
     - **To Account** (dropdown of the same user’s accounts)  
     - **Amount** (number input → validate `value > 0` and `value <= selectedFromBalance`)  
     - **Category** (optional, dropdown from `GET /api/categories`)  
     - **Description** (text input, max length: 255)  
   - On submit, call `POST /api/transactions/transfer` with JSON body:  
     ```json
     {
       "fromAccountId": "GUID",
       "toAccountId": "GUID",
       "amount": 150.00,
       "categoryId": 3,
       "description": "June rent share"
     }
     ```  
   - Show success or error toast (using a React toast library).

4. **`/pages/profile.tsx`**
   - Fetch current user info from `GET /api/users/me`.  
   - Pre-populate form fields (Name, Email, etc.).  
   - On save, `PUT /api/users/me`, then show a confirmation dialog.

---

#### 5.2 Progressive Enhancement with HTML/jQuery/AJAX

_To demonstrate jQuery/AJAX experience:_

1. **Landing Page (`/public/index.html`)**
   - Pure static HTML/CSS (no React).  
   - Contains a **Sign In** button that triggers a jQuery click handler to show a login modal.  
   - Login modal submits credentials via `$.ajax` to `POST /api/auth/login`, stores JWT in `localStorage`, then redirects to `/dashboard.html`.

2. **Dashboard Page (`/public/dashboard.html`)**
   - Uses jQuery’s `$.ajax` to fetch `/api/accounts/summary` (with Authorization header) and builds account cards by appending HTML fragments.  
   - Has a **View Transactions** link; when clicked, uses AJAX to populate a `<table>` with `sp_GetTransactions` results.  
   - Implements date filters using **jQuery UI DatePicker**; when the user selects new dates, it calls `/api/transactions?from=…&to=…` and redraws the table.  
   - All styling done with **Tailwind CSS** classes embedded in the HTML.

> This dual approach (Next.js/React + jQuery/AJAX) shows mastery of both modern frameworks and legacy patterns, satisfying TBI Bank’s request for HTML, CSS, jQuery, and AJAX.

---

### 6. Relational Database Setup & Tools

#### **Local Environment**
- **MS SQL Server Express (LocalDB)** for development.  
- Use **EF Core Migrations** to create the schema; add stored procedures manually or via EF migration scripts.  
- Seed data: 2 demo users (one “Customer,” one “Admin”), each with 2 accounts and 10–15 sample transactions (spanning last three months).

#### **Indexing Strategy**
- **Transactions**: Nonclustered index on `(AccountId, Timestamp DESC)` to optimize filtered queries.  
- **Accounts**: Index on `UserId`.  
- **Categories**: Index on `UserId`.

---

### 7. Development Workflow & Collaboration Tools

#### **Git & Bitbucket**
- Initialize repository with `main` branch → create `develop` branch → feature branches (`feature/auth`, `feature/accounts-summary`, etc.).  
- For each feature:  
  1. Open a pull request in Bitbucket.  
  2. Assign two reviewers.  
  3. Incorporate feedback → merge into `develop`.

#### **Jira**
- Project “PFM Portal” with Epics:  
  1. User Authentication & Roles  
  2. Account Summary Module  
  3. Transactions Module  
  4. Transfers & Categories  
  5. Budgeting & Reports (Bonus)  
  6. PDF Statement Generation (Bonus)

- Break down each Epic into user stories with acceptance criteria, story points, and sprint assignments.

---

### 8. Bonus Integrations (Fintech Focus)

1. **PDF Statement Generator**  
   - **Endpoint:** `GET /api/transactions/statement?from=YYYY-MM-DD&to=YYYY-MM-DD`  
   - Uses **iTextSharp** (or **PdfSharp**) to generate a PDF containing all transactions in that period, with:
     - Header (user name, account number)  
     - A table of transactions  
     - Final balance summary  
   - Returns a `FileContentResult` (`application/pdf`) so the front end can prompt a download.

2. **Exchange-Rate Widget**  
   - Front end calls a free FX API (e.g., [exchangerate.host](https://exchangerate.host)) to display the current **BGN ↔ EUR** rate.  
   - Show a small Chart.js trendline (last 30 days) next to account balances.

3. **Audit & Compliance Dashboard (Admin)**  
   - Admins can log in and view an **Audit Log** page (`/admin/audit.html`) that uses jQuery/AJAX to call `/api/audit?page=1&pageSize=50`.  
   - The table displays:  
     - `Timestamp`  
     - `User`  
     - `Action`  
     - `Details`  
   - Filters: by date, by Action type.

---

### 9. Milestones & Timeline

| **Week** | **Focus**                                                                                                                                                    |
|:--------:|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **1**    | • Project scaffolding (ASP.NET Core solution + Next.js project)<br>• Database schema design & EF migrations<br>• Configure Identity/JWT, initial seed data<br>• Jira backlog creation, initial sprint planning |
| **2**    | • Implement `AuthController` (register, login)<br>• Implement `AccountsController` & `sp_GetAccountSummary`<br>• Build React Dashboard page to display account cards<br>• Set up Gitflow branches & PR process |
| **3**    | • Implement `TransactionsController` & stored procedure `sp_GetTransactions`<br>• Build React Transactions page with date filters & table<br>• Add jQuery/AJAX dashboard under `/public` for progressive enhancement |
| **4**    | • Create `sp_PostTransfer` procedure; build `POST /api/transactions/transfer`<br>• React Transfer page with validation; jQuery/AJAX form fallback<br>• Implement category management (`CategoriesController`)       |
| **5**    | • Develop PDF statement endpoint (`sp_GetTransactions` → PDF via iTextSharp)<br>• Build front-end download button (React + jQuery)<br>• Integrate Exchange-Rate Widget in both React & jQuery versions               |
| **6**    | • Admin AuditLog APIs & front-end page (React + jQuery)<br>• Add Budget Tables & Reports (Bonus)<br>• Write unit tests (XUnit for .NET, Jest for JS utilities)<br>• Document endpoints with Swagger & create README |
| **7**    | • Finalize styling, cross-browser testing, responsive tweaks<br>• Deploy a demo to Azure (or equivalent)<br>• Record a 5-minute walkthrough video<br>• Prepare a concise slide deck (optional)                     |

---

### 10. How This Project “Pops” My Application

1. **Direct Mapping to TBI Bank’s Stack**  
   - Back end in **ASP.NET Core (.NET 7)**, front end using **JavaScript/jQuery/AJAX** and **React/Next.js** (leveraging existing Next.js/TypeScript skills).  
   - Database work in **MS SQL Server**, including stored procedures and indexing—showing proficiency with relational DBs (portable to PostgreSQL/MySQL/Oracle).  
   - Full use of **Git** (Bitbucket) and **Jira**, mimicking a real fintech team environment.

2. **Highlights Banking Domain Knowledge**  
   - Designed a mock **Core Banking** environment: account ledgers, transaction posting, balance reconciliation, and audit logs—mirroring real-world financial software requirements.  
   - Expense categorization and budgeting modules demonstrate understanding of how banks provide value-added services (reports, statements, budgeting tools) to customers.

3. **Demonstrates Adaptability & Learning Agility**  
   - Although currently specializing in React/Next.js, I will extend that skill set to **ASP.NET Core** and integrate **jQuery/AJAX** modules—showing ability to bridge front-end and back-end seamlessly.  
   - By including both progressive enhancement (jQuery/AJAX) and modern SPA (React) approaches, I prove I can support legacy codebases and build future-proof features.

4. **Tangible Proof of Skill**  
   - Upon completion, I will share a **live demo link** and a **public Bitbucket repository**.  
   - Code will be well-documented (Swagger/OpenAPI, README) so reviewers can inspect service abstractions, stored procedures, UI workflows, and security measures.  
   - A short video walkthrough (hosted on a private YouTube link) will narrate how each requirement was met: user authentication, account summary, AJAX filters, PDF statements, and audit logging.

---

<p align="center">🚀 Ready to build the Personal Finance Management Portal! 🚀</p>
