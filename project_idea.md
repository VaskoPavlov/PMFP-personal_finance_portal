## Technology Stack
| Layer / Component                | Technology & Tools                       | Purpose & Justification                                                                                                 |
| -------------------------------- | ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Back-end API**                 | ASP.NET Core (.NET 7), C# 8              | Core RESTful endpoints for accounts, transactions, budgets.                                                             |
| **Authentication/Authorization** | ASP.NET Identity with JWT Tokens         | Secure login, role checks, token-based session management.                                                              |
| **Database**                     | MS SQL Server 2022                       | User, Account, Transaction, Category, and Audit Log tables.                                                             |
| **ORM & Data Access**            | Entity Framework Core 7                  | Mapping C# models to MS SQL tables, migrations, and LINQ queries.                                                       |
| **Front-end Framework**          | React 18 (Next.js for SSR) + TypeScript  | Responsive UI, server-side rendering for faster initial load.                                                           |
| **Progressive Enhancement**      | HTML5, CSS3 (Tailwind CSS), jQuery, AJAX | Ensures basic operations (transaction search, filters) still work if JS fails; demonstrates proficiency in jQuery/AJAX. |
| **Version Control**              | Git (Gitflow) → Bitbucket                | Branching strategy (feature → develop → main), pull requests, code reviews.                                             |
| **Issue Tracking & Planning**    | Jira                                     | Backlog grooming, sprint planning, user story tracking.                                                                 |
| **CI/CD Pipeline (Bonus)**       | Bitbucket Pipelines                      | Automatically run build/tests on `develop` branch pushes.                                                               |
| **Charting Library**             | Chart.js (vanilla JavaScript plugin)     | Dynamic expense vs. income graphs; easy jQuery integration.                                                             |
