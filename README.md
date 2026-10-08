# Library-Management-System
This is done for you c# subject project.
dotnet run --launch-profile https // to run the app with HTTPS (self-signed dev cert) and hot reload.
# InfoLMS — Library Management System (C# / .NET 8 rewrite)

A full rewrite of the original Java Spring Boot + React LMS into **pure C#**:
**ASP.NET Core Blazor Server** (UI + backend in one project) with **EF Core** on
**Microsoft SQL Server Express**.

## Stack

| Layer      | Technology |
|------------|------------|
| UI         | Blazor Server (.NET 8), custom CSS design system, cursor-glow / hover micro-interactions |
| Backend    | ASP.NET Core 8 (same project — no separate API needed) |
| Auth       | Cookie authentication, role claims (`Admin`, `Librarian`, `Student`), BCrypt password hashing |
| Database   | SQL Server Express, EF Core 8 (code-first) |
| Email      | MailKit (SMTP) — logs to console if no SMTP host is configured |
| Payments   | Mocked payment gateway (`IPaymentService`) — swap in Razorpay/Stripe later |

## What's included

- Registration → librarian/admin approval workflow
- Role-based dashboards: **Student**, **Librarian**, **Admin**
- Book catalog search/browse, borrow requests, approve/reject, issue & return
- Live overdue-fine calculation + mock fine payment
- Membership plans (Basic / Super / Elite) with borrow limits & block/unblock
- "Request a new book" workflow with librarian notify-when-available
- In-app notifications (bell icon + full notification center)
- Admin: user management (approve/reject/block/role), book CRUD, payment ledger

## Scoped down from the original

To keep this rewrite coherent, the multi-step **Academic Info** and **Work
Experience** sub-forms from registration, and the **Razorpay** live gateway
integration, were not fully rebuilt — the entities (`AcademicInfo`,
`WorkExperience` equivalents) aren't included, and payments use a mock
gateway that records a receipt instantly. Both are straightforward
extensions:
- Add entities + a repeating `EditForm` section on `Register.razor`.
- Implement `IPaymentService` against a real provider's .NET SDK; everything
  else (fine clearing, receipts, notifications) already calls through the
  interface, so no other code changes are needed.

## Prerequisites

1. **.NET 8 SDK** — https://dotnet.microsoft.com/download/dotnet/8.0
2. **SQL Server Express** (or any SQL Server instance) —
   https://www.microsoft.com/sql-server/sql-server-downloads
3. EF Core CLI tools (one-time):
   ```bash
   dotnet tool install --global dotnet-ef
   ```

## Setup

1. **Connection string** — edit `LMS.Web/appsettings.json` if your SQL Server
   Express instance name differs from the default:
   ```json
   "DefaultConnection": "Server=.\\SQLEXPRESS;Database=LmsDb;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true"
   ```
   For SQL auth instead of Windows auth, use:
   ```
   Server=.\SQLEXPRESS;Database=LmsDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True
   ```

2. **Restore & create the database migration** (from the repo root):
   ```bash
   cd LMS.Web
   dotnet restore
   dotnet ef migrations add InitialCreate
   ```
   (`Program.cs` calls `db.Database.Migrate()` automatically on startup, so
   you don't need to run `dotnet ef database update` separately — it happens
   the first time you run the app.)

3. **Run**:
   ```bash
   dotnet run
   ```
   Then open the URL shown in the console (e.g. `https://localhost:7080`).

4. **Log in** with a seeded demo account (created automatically on first run):
   - Admin — `admin@lms.local` / `Admin@123`
   - Librarian — `librarian@lms.local` / `Librarian@123`
   - Or register a new **Student** account at `/register` (it will need
     approval from the admin or librarian account before it can log in).

## Optional: real email

Fill in `Smtp` in `appsettings.json` (host/port/from/user/password). Until
then, emails are written to the application log instead of being sent, so
the app runs out-of-the-box without any mail credentials.

## Project layout

```
LMS.Web/
  Data/
    Entities/       — EF Core entity classes (Book, User, IssueBook, ...)
    AppDbContext.cs
    DbSeeder.cs      — demo admin/librarian accounts + sample books
  Services/          — all business logic (1 interface + 1 impl per concern)
  Components/
    Layout/          — MainLayout (sidebar/topbar/cursor-glow), NavMenu
    Pages/
      Login.razor, Register.razor, Home.razor, AccessDenied.razor
      Student/       — dashboard, browse, my-books, membership, requests, profile
      Librarian/     — requests, issued books, book CRUD, memberships, new-book requests
      Admin/         — overview, user management, book CRUD, payments
  wwwroot/
    css/app.css      — full design system (dark theme, gradients, glow, cards)
    js/interop.js    — cursor-follow glow animation
```

## Notes on the UI/UX

- Dark, glassy theme with a gradient-accented sidebar and a soft **cursor-follow
  glow** (`wwwroot/js/interop.js` + `.cursor-glow` in `app.css`) that gives the
  whole app a "living" feel without any heavy JS framework.
- Cards lift and glow on hover, buttons have press/hover motion, tables have
  row-hover highlighting, and status pills are colour-coded (pending/approved/
  overdue/etc.) for fast scanning.
- Everything is hand-rolled CSS (no Bootstrap/Tailwind dependency) so you can
  restyle freely by editing the CSS variables at the top of `app.css`.
