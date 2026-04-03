# Calendar Life To-Do API 🟩

A robust, enterprise-grade ASP.NET Core (.NET 10) Web API designed to serve as the backend for a "Life Grid" To-Do application. This project visualizes a user's life in days/weeks represented as a grid, tracking daily habits, journaling, and calculating lifetime statistics.

This backend is structured using **Clean Architecture** to ensure maintainability, scalability, and testability, reflecting high standards suitable for top-tier European tech markets.

---

## 🏛️ What is Clean Architecture?

Clean Architecture (popularized by Robert C. Martin) separates software into layers with a strict dependency rule: **Dependencies must point inward, toward the core entities.**

1.  **Domain Layer (Core):** Contains enterprise logic and entities (e.g., `User`, `DailyRecord`). It has *zero* external dependencies.
2.  **Application Layer:** Contains business rules and use cases (e.g., `CompleteTaskCommand`, `GetLifeGridQuery`). It depends only on the Domain.
3.  **Infrastructure Layer:** Implements external concerns (e.g., PostgreSQL database access via Entity Framework Core, external APIs). It depends on the Application layer to implement its interfaces.
4.  **Presentation / API Layer:** The entry point (Controllers/Endpoints). It receives HTTP requests, routes them to the Application layer, and returns responses.

**Why use it?** If tomorrow you want to change the database from PostgreSQL to SQL Server, or the UI from a Web App to a Mobile App, your core business logic (Domain and Application) remains untouched.

---

## 🗄️ Database Schema (PostgreSQL)

Based on the frontend state structure, the relational database is modeled as follows:

### 1. `Users`
| Column      | Type      | Description                          |
| :---------- | :-------- | :----------------------------------- |
| `Id`        | UUID (PK) | Unique identifier                    |
| `Name`      | VARCHAR   | User's display name ("Rocha")        |
| `BirthDate` | DATE      | Used to calculate the grid indexes   |

### 2. `UserSettings`
| Column           | Type         | Description                       |
| :--------------- | :----------- | :-------------------------------- |
| `UserId`         | UUID (PK/FK) | Links to User                     |
| `SquareSize`     | INT          | UI setting (e.g., 20)             |
| `ShowHelp`       | BOOLEAN      | UI setting                        |
| `SetupCompleted` | BOOLEAN      | Indicates if onboarding is done   |

### 3. `DefaultTasks`
| Column     | Type      | Description          |
| :--------- | :-------- | :------------------- |
| `Id`       | UUID (PK) | Unique identifier    |
| `UserId`   | UUID (FK) | Links to User        |
| `Text`     | VARCHAR   | Task description     |
| `Duration` | INT       | Duration in minutes  |

### 4. `DailyRecords`
| Column     | Type      | Description                             |
| :--------- | :-------- | :-------------------------------------- |
| `Id`       | UUID (PK) | Unique identifier                       |
| `UserId`   | UUID (FK) | Links to User                           |
| `DayIndex` | INT       | Days since birth (e.g., 9082)           |
| `Status`   | VARCHAR   | "completed", "failed", or "pending"     |
| `Journal`  | TEXT      | Daily journal entry                     |

### 5. `DailyTasks`
| Column          | Type      | Description          |
| :-------------- | :-------- | :------------------- |
| `Id`            | UUID (PK) | Unique identifier    |
| `DailyRecordId` | UUID (FK) | Links to DailyRecord |
| `Text`          | VARCHAR   | Task description     |
| `IsCompleted`   | BOOLEAN   | Checkbox state       |
| `Duration`      | INT       | Duration in minutes  |

---

## 🚀 Development Checklist & CLI Commands

Follow these steps in order. Check them off `[x]` as you complete them.

### Phase 1: Solution & Architecture Setup

- [ ] **Create the Solution File**
  `dotnet new sln -n LifeGrid`

- [ ] **Create the Layers (Projects)**
  `dotnet new classlib -n LifeGrid.Domain`
  `dotnet new classlib -n LifeGrid.Application`
  `dotnet new classlib -n LifeGrid.Infrastructure`
  `dotnet new webapi -n LifeGrid.Api`

- [ ] **Add Projects to Solution**
  `dotnet sln add LifeGrid.Domain/LifeGrid.Domain.csproj`
  `dotnet sln add LifeGrid.Application/LifeGrid.Application.csproj`
  `dotnet sln add LifeGrid.Infrastructure/LifeGrid.Infrastructure.csproj`
  `dotnet sln add LifeGrid.Api/LifeGrid.Api.csproj`

- [ ] **Set up Dependencies (Clean Architecture Rules)**
  `dotnet add LifeGrid.Application/LifeGrid.Application.csproj reference LifeGrid.Domain/LifeGrid.Domain.csproj`
  `dotnet add LifeGrid.Infrastructure/LifeGrid.Infrastructure.csproj reference LifeGrid.Application/LifeGrid.Application.csproj`
  `dotnet add LifeGrid.Api/LifeGrid.Api.csproj reference LifeGrid.Application/LifeGrid.Application.csproj`
  `dotnet add LifeGrid.Api/LifeGrid.Api.csproj reference LifeGrid.Infrastructure/LifeGrid.Infrastructure.csproj`

### Phase 2: Domain & Application (Core Logic)

- [ ] **Create Entities:** In `LifeGrid.Domain`, create folders for Entities (`User`, `DefaultTask`, `DailyRecord`, `DailyTask`, `UserSetting`).
- [ ] **Define Enums:** Create a `DayStatus` enum (Completed, Failed, Pending).
- [ ] **Create Interfaces:** In `LifeGrid.Application`, create an `Interfaces` folder. Add `ILifeGridDbContext` and repository interfaces.
- [ ] **Install MediatR (CQRS):**
  `dotnet add LifeGrid.Application package MediatR`
- [ ] **Create Use Cases:** Add `Commands` and `Queries` folders in Application (e.g., `UpdateDailyRecordCommand`, `GetUserGridQuery`).

### Phase 3: Infrastructure (Database & EF Core)

- [ ] **Install Entity Framework Core & PostgreSQL packages:**
  `dotnet add LifeGrid.Infrastructure package Microsoft.EntityFrameworkCore.Design`
  `dotnet add LifeGrid.Infrastructure package Npgsql.EntityFrameworkCore.PostgreSQL`
- [ ] **Setup DbContext:** Create `LifeGridDbContext` implementing `ILifeGridDbContext`. Configure your `DbSet` properties.
- [ ] **Fluent API Configuration:** Override `OnModelCreating` to configure foreign keys and constraints (e.g., mapping the `DayIndex`).
- [ ] **Connection String:** Add the PostgreSQL connection string to `appsettings.json` in the API project.

### Phase 4: API Presentation & Migrations

- [ ] **Install EF Core Design in API (needed for migrations):**
  `dotnet add LifeGrid.Api package Microsoft.EntityFrameworkCore.Design`
- [ ] **Register Services:** In `Program.cs` (API), configure MediatR, DbContext (using `AddNpgsql`), and inject your repositories.
- [ ] **Create the Initial Migration:**
  `dotnet ef migrations add InitialCreate --project LifeGrid.Infrastructure --startup-project LifeGrid.Api`
- [ ] **Update the Database:**
  `dotnet ef database update --project LifeGrid.Infrastructure --startup-project LifeGrid.Api`
- [ ] **Create Controllers:**
  - `UsersController` (Settings, Profile)
  - `GridController` (Fetching the grid state, day indexes)
  - `DailyRecordsController` (Updating tasks, submitting journals, setting day to failed/completed)
- [ ] **Swagger/OpenAPI:** Ensure Swagger is enabled in `Program.cs` for easy frontend testing.

### Phase 5: Polish & Run

- [ ] **Run the Project:**
  `dotnet run --project LifeGrid.Api`
- [ ] **Test Endpoints:** Open `https://localhost:<port>/swagger` and test posting a payload that matches your frontend JSON.
- [ ] **Implement CORS:** Ensure `Program.cs` has CORS configured so your frontend can communicate with the backend without browser errors.

---
*Built with code, caffeine, and Italian dreams.* 🇮🇹☕
