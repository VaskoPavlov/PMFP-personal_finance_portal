## Project Idea: “Personal Finance Management Portal”

To showcase my alignment with TBI Bank’s technical requirements—and to make my application stand out—I propose building a “Personal Finance Management Portal”. This prototype demonstrates end-to-end fintech capabilities using JavaScript, .NET, HTML/CSS, jQuery/AJAX, and a relational database, while also highlighting my banking domain knowledge. Below is a structured plan:
1. Project Overview & Goals

    * Account Overview: Authenticated users see all their virtual “bank” accounts (Checking, Savings, Credit).

    * Transaction History: Users can filter and search transactions by date range, amount, or description.

    * Internal Transfers: Permits moving funds between user accounts.

    * Expense Categorization: Allows tagging transactions as “Groceries,” “Utilities,” “Entertainment,” etc., to generate monthly spending reports.

    * Budget Dashboard: Visual summary of monthly income vs. expenses, using interactive charts.

    * Secure Authentication & Authorization: Role-based access: “Customer” vs. “Admin.”

2. Technology stack

** SEE file: project_idea.md

3. Data Model & Relational Schema

All tables will be normalized (3NF). Key tables:

    Users

        UserId (PK, GUID)

        Email (string, unique)

        PasswordHash (string)

        FullName (string)

        Role (enum: Customer, Admin)

        CreatedAt (datetime)

    Accounts

        AccountId (PK, GUID)

        UserId (FK → Users.UserId)

        AccountType (enum: Checking, Savings, Credit)

        Balance (decimal)

        Currency (char(3), e.g., “BGN”)

        CreatedAt (datetime)

    Categories

        CategoryId (PK, int)

        UserId (FK → Users.UserId)

        Name (string, e.g., “Groceries,” “Utilities”)

    Transactions

        TransactionId (PK, GUID)

        AccountId (FK → Accounts.AccountId)

        CategoryId (FK → Categories.CategoryId, nullable)

        Amount (decimal)

        TransactionType (enum: Debit, Credit)

        Description (nvarchar(255))

        Timestamp (datetime)

        CreatedAt (datetime)

    AuditLog

        LogId (PK, int, identity)

        UserId (FK → Users.UserId)

        Action (nvarchar(100), e.g., “TransferFunds,” “UpdateProfile”)

        Details (nvarchar(max))

        OccurredAt (datetime)

    Budgets (Bonus)

        BudgetId (PK, GUID)

        UserId (FK → Users.UserId)

        MonthYear (char(7), e.g., “2025-06”)

        TotalBudget (decimal)

    BudgetAllocations (Bonus)

        AllocationId (PK, int, identity)

        BudgetId (FK → Budgets.BudgetId)

        CategoryId (FK → Categories.CategoryId)

        AllocatedAmount (decimal)

Stored Procedures & Indexes

    sp_GetAccountSummary(@UserId): Returns each account’s current balance.

    sp_GetTransactions(@UserId, @FromDate, @ToDate): Returns all transactions in date range, ordered by Timestamp DESC—indexed on (AccountId, Timestamp).

    sp_PostTransfer(@FromAccountId, @ToAccountId, @Amount, @UserId):

        Check sufficient Balance on FromAccountId.

        Debit FromAccountId, Credit ToAccountId.

        Insert two Transactions (one debit, one credit), with same Timestamp.

        Insert into AuditLog.

        Wrap in a transaction (BEGIN TRAN / COMMIT).

    sp_GetMonthlySpending(@UserId, @MonthYear) (Bonus): Aggregates total debit amounts per category for budget chart.

4. Back-end Implementation (ASP.NET Core)

    Folder Structure

    /src
      /API
        Controllers
        DTOs
        Filters
      /Core
        Models
        Interfaces
        Services
      /Infrastructure
        Data
        Repositories
        Migrations

    Controllers

        AuthController:

            POST /api/auth/register → registers new user, sends activation email (optional).

            POST /api/auth/login → issues JWT.

        AccountsController:

            GET /api/accounts/summary → calls sp_GetAccountSummary.

            GET /api/accounts/{id}/transactions → calls sp_GetTransactions.

        TransactionsController:

            GET /api/transactions?from=YYYY-MM-DD&to=YYYY-MM-DD → filtered transactions for all user accounts.

            POST /api/transactions/transfer → executes sp_PostTransfer.

        CategoriesController: (Bonus)

            GET /api/categories → returns user’s category list.

            POST /api/categories → adds new category.

        BudgetController: (Bonus)

            GET /api/budgets/{monthYear} → returns budget allocations and monthly spending (sp_GetMonthlySpending).

            POST /api/budgets → creates or updates monthly budget.

    Services & Repositories

        IAccountService & AccountService: Wrap calls to stored procedures via EF Core’s Database.ExecuteSqlRawAsync.

        ITransactionService & TransactionService: Business logic for transaction validation, category assignment.

        IAuthService & AuthService: Handles password hashing (ASP.NET Identity) and JWT issuance.

        IAuditService & AuditService: Inserts audit entries.

    Security

        Use ASP.NET Identity for initial user provisioning.

        Configure JWT with symmetric key (stored in appsettings.json behind a secret vault in production).

        Apply [Authorize] on controllers, and [Authorize(Roles="Admin")] on any administration endpoints.

        Enforce HTTPS and secure cookie settings.

    Error Handling & Logging

        Add a global exception filter (ApiExceptionFilter) to convert exceptions into uniform ProblemDetails JSON.

        Use Serilog (or built-in ILogger) to write structured logs to Console and a local rolling file.

5. Front-end Implementation

While my day-to-day role uses Next.js/React, I will implement a hybrid approach that showcases both modern SPA techniques and classic jQuery/AJAX proficiency—key for TBI Bank’s requirements.
5.1 React + TypeScript (Next.js) Pages

    /pages/index.tsx (Dashboard)

        On component mount (useEffect), call GET /api/accounts/summary via fetch or Axios (depending on preference).

        Render account cards: “1234 ·· ··5678 | BGN 1,250.00”.

        Display a mini expense pie chart (via Chart.js) showing percentage spent vs. remaining budget (if budgets are set).

    /pages/transactions.tsx

        Date‐range picker (leveraging a React date‐picker component).

        On filter change, call GET /api/transactions?from=…&to=….

        Render a table (React component) with columns: Date, Account, Type, Category, Amount, Description.

        Allow sorting by clicking column headers (React state).

    /pages/transfer.tsx (Payments)

        Form with:

            From Account (dropdown populated via GET /api/accounts/summary).

            To Account (dropdown of same user’s accounts).

            Amount (number input, with client-side validation: value > 0 and value <= selectedFromBalance).

            Category (optional, dropdown from GET /api/categories).

            Description (text input, max length: 255).

        On submit, call POST /api/transactions/transfer with JSON body:

        {
          "fromAccountId": "GUID",
          "toAccountId": "GUID",
          "amount": 150.00,
          "categoryId": 3,
          "description": "June rent share"
        }

        Show success or error toast (using a React toast library).

    /pages/profile.tsx

        Fetch current user info from GET /api/users/me.

        Pre-populate form fields (Name, Email, etc.).

        On save, PUT /api/users/me, then show a confirmation dialog.

5.2 Progressive Enhancement with HTML/jQuery/AJAX

To demonstrate jQuery/AJAX experience:

    Landing Page (/public/index.html)

        A pure static HTML/CSS page (no React).

        Contains a “Sign In” button that triggers a jQuery click handler to show a login modal.

        Login modal submits credentials via $.ajax to POST /api/auth/login, stores JWT in localStorage, then redirects to /dashboard.html.

    Dashboard Page (/public/dashboard.html)

        Uses jQuery’s $.ajax to fetch /api/accounts/summary (with Authorization header) and builds account cards by appending HTML fragments.

        Has a “View Transactions” link that, when clicked, uses AJAX to populate a <table> with sp_GetTransactions results.

        Implements date filters using jQuery UI DatePicker; when user selects new dates, it calls /api/transactions?from=…&to=… and redraws the table.

        All styling done with Tailwind CSS classes embedded in the HTML.

This dual-approach (Next.js/React + jQuery/AJAX) shows mastery of both modern frameworks and legacy patterns, satisfying TBI Bank’s request for HTML, CSS, jQuery, and AJAX.
6. Relational Database Setup & Tools

    Local Environment

        MS SQL Server Express (localdb) for development.

        Use EF Core Migrations to create the schema, then add stored procedures manually or via EF migration scripts.

        Seed data: 2 demo users (one “Customer,” one “Admin”), each with 2 accounts and 10–15 sample transactions (spanning last three months).

    Indexing Strategy

        On Transactions: create a nonclustered index on (AccountId, Timestamp DESC) to optimize filtered queries.

        On Accounts: index on UserId.

        On Categories: index on UserId.

7. Development Workflow & Collaboration Tools

    Git & Bitbucket

        Initialize repository with main branch → create develop branch → feature branches (feature/auth, feature/accounts-summary, etc.).

        For each feature, open a pull request in Bitbucket → assign two reviewers → incorporate feedback → merge into develop.

    Jira

        Create a project “PFM Portal” with Epics:

            User Authentication & Roles

            Account Summary Module

            Transactions Module

            Transfers & Categories

            Budgeting & Reports (Bonus)

            PDF Statement Generation (Bonus)

        Each epic broken into user stories with acceptance criteria, story points, and sprint assignments.

8. Bonus Integrations (Fintech Focus)

    PDF Statement Generator

        Backend: Endpoint GET /api/transactions/statement?from=YYYY-MM-DD&to=YYYY-MM-DD.

        Uses iTextSharp (or PdfSharp) to generate a PDF containing all transactions in that period, with header (user name, account number), a table of transactions, and a final balance summary.

        Returns a FileContentResult (application/pdf) so the front end can prompt a download.

    Exchange-Rate Widget

        Front end calls a free FX API (e.g., exchangerate.host) to display the current BGN ↔ EUR rate.

        Show a small Chart.js trendline (last 30 days) next to account balances.

    Audit & Compliance Dashboard (Admin)

        Admins can log in and view an “Audit Log” page (/admin/audit.html) that uses jQuery/AJAX to call /api/audit?page=1&pageSize=50.

        The table shows: Timestamp, User, Action, Details.

        Filters: by date, by Action type.

9. Milestones & Timeline
Week	Focus
| 1 |	• Project scaffolding (.NET solution + Next.js project)
      • Database schema design & EF migrations  
      • Configure Identity/JWT, initial seed data  
      • Jira backlog creation, initial sprint planning                                             |
| 2 | • Implement AuthController (register, login)
      • Implement AccountsController and sp_GetAccountSummary
      • Build React Dashboard page to display account cards
      • Set up Gitflow branches & PR process |
| 3 | • Implement TransactionsController and stored procedure sp_GetTransactions
      • Build React Transactions page with date filters & table
      • Add jQuery/AJAX dashboard under /public for progressive enhancement |
| 4 | • Create sp_PostTransfer procedure; build POST /api/transactions/transfer
      • React Transfer page with validation; jQuery/AJAX form fallback
      • Implement category management (CategoriesController) |
| 5 | • Develop PDF statement endpoint (sp_GetTransactions → PDF via iTextSharp)
      • Build front-end download button (React + jQuery).
      • Integrate Exchange-Rate Widget in both React & jQuery versions. |
| 6 | • Admin AuditLog APIs & front-end page (React + jQuery).
      • Add Budget Tables & Reports (Bonus).
      • Write unit tests (XUnit for .NET, Jest for any pure JS utilities).
      • Document endpoints with Swagger & create README with setup instructions. |
| 7 | • Finalize styling, cross-browser testing, responsive tweaks.
      • Deploy a demo to Azure (or equivalent) if feasible.
      • Record a 5-minute walkthrough video demonstrating key features.
      • Prepare a concise slide deck (optional) for TBI Bank screening. |
10. How This Project “Pops” My Application
    Directly Maps to TBI Bank’s Stack

        Back end in ASP.NET Core (.NET 7), front end using JavaScript/jQuery/AJAX and React (leveraging my existing Next.js/TypeScript skills).

        Database work entirely in MS SQL Server, including stored procedures and indexing—showing proficiency with relational DBs (and easily portable to PostgreSQL/MySQL/Oracle).

        Full use of Git (Bitbucket) and Jira mimics a real fintech team environment.

    Highlights Banking Domain Knowledge

        I designed a mock “Core Banking” environment: account ledgers, transaction posting, balance reconciliation, and audit logs—mirroring real-world financial-software requirements.

        The expense-categorization and budgeting modules demonstrate an understanding of how banks provide value-added services (reports, statements, budgeting tools) to customers.

    Demonstrates Adaptability & Learning Agility

        Although I currently specialize in React/Next.js, I will extend that skill set to ASP.NET Core and integrate jQuery/AJAX modules—showing I can bridge front-end and back-end seamlessly.

        By including both progressive enhancement (jQuery/AJAX) and modern SPA (React) approaches, I prove I can support legacy codebases and build new, future-proof features.

    Tangible Proof of Skill

        Upon completion, I will share a live demo link and a public Bitbucket repository.

        Code will be well-documented (Swagger/OpenAPI, README) so you can review my implementation details: service abstractions, stored procedures, UI workflows, and security measures.

        A short video walkthrough (hosted on a private YouTube link) will narrate how each requirement was met: user authentication, account summary, AJAX filters, PDF statements, and audit logging.
