# Learning & Assessment Portal (LAP)

An enterprise Learning Management System (LMS) that supports course management, adaptive assessments, leaderboards, user enrollment, course reviews, and discussion forums. This monorepo contains both the backend (C#/.NET 8 API) and the frontend (React/TypeScript).

## Repository Structure

```
learning-and-assessment-portal-backend-development/
├── learning-and-assessment-portal-backend-development/   (C#/.NET 8 API)
│   ├── LAP.API/               -- ASP.NET Core Web API (controllers, middleware, auth)
│   ├── LAP.Application/       -- Business logic (MediatR CQRS, FluentValidation, DTOs)
│   ├── LAP.Domain/            -- Domain entities (23 entities)
│   ├── LAP.Infrastructure/    -- EF Core, PostgreSQL, repositories, seed data, logging
│   ├── LAP.Shared/            -- Cross-cutting utilities (BCrypt hashing)
│   ├── LAP.Test/              -- Unit & integration tests (xUnit)
│   │   ├── LAP.UnitTest/
│   │   └── LAP.IntegrationTest/
│   ├── Design/OpenApi/        -- OpenAPI 3.0.3 specification
│   └── LearningPortalAPI.sln
│
└── learning-and-assessment-portal-frontend-development/  (React 19 / TypeScript / Vite)
    ├── src/
    │   ├── core/              -- Providers (Auth, Course, Enrollment, Theme), routing, config
    │   ├── features/          -- Feature modules (admin, auth, home, leaderboard, user)
    │   ├── shared/            -- Reusable UI components, hooks, services, types, constants
    │   └── tests/             -- Jest unit tests
    ├── Design/                -- Frontend design artifacts
    └── package.json
```

## Tech Stack

### Backend

| Concern          | Technology                                                   |
| ---------------- | ------------------------------------------------------------ |
| Language         | C# (.NET 8)                                                  |
| Framework        | ASP.NET Core Web API                                         |
| Database         | PostgreSQL (Npgsql + Entity Framework Core 8)                |
| Architecture     | Clean Architecture (Domain / Application / Infrastructure / API) |
| CQRS             | MediatR (commands & queries per feature)                     |
| Object Mapping   | AutoMapper                                                   |
| Validation       | FluentValidation                                             |
| Auth             | JWT Bearer (access + refresh tokens)                         |
| Authorization    | Feature-based RBAC (Admin, Student) with `[FeatureAuthorize]` |
| Logging          | Serilog (Console + File sinks)                               |
| API Docs         | Swashbuckle/Swagger, API versioning (v1)                     |
| CSV/Excel        | CsvHelper, MiniExcel                                         |
| Password Hashing | BCrypt.Net-Next                                              |
| Testing          | xUnit, Moq, FluentAssertions, InMemory EF Core, WebApplicationFactory |

### Frontend

| Concern               | Technology                                                       |
| --------------------- | ---------------------------------------------------------------- |
| Language              | TypeScript 6.x                                                   |
| Framework             | React 19                                                         |
| Build Tool            | Vite 8                                                           |
| UI Library            | Material UI (MUI) v9 + Emotion                                   |
| Routing               | React Router DOM 7.x                                             |
| Forms                 | React Hook Form                                                  |
| HTTP Client           | Axios                                                            |
| API Client Generation | Orval (typed Axios services from the Swagger spec)               |
| Testing               | Jest 30 + React Testing Library                                  |
| Linting               | ESLint 10 with TypeScript-ESLint, React Hooks, React Refresh     |

## Features

- **Authentication & User Management** -- Registration, login, logout, refresh token, profile view/update, password reset, profile image upload. Roles: **Admin** and **Student**.
- **Course Management (Admin)** -- Create, update, soft-delete courses, manage content items and topic hierarchy, manage enrollments, and view admin dashboard summaries.
- **Course Discovery & Learning (Student)** -- Browse/filter/search courses, course overview, content view, progress tracking, enrollment, and recommendations.
- **Assessments** -- One assessment per course, question bank CRUD, CSV export/import templates, answer submission, results, paginated history, tier-based scoring.
- **Leaderboards** -- Per-course and overall platform leaderboards with podium display and stats.
- **Course Reviews** -- Create, update, delete course reviews/ratings.
- **Discussion Forums** -- Post and view forum messages per course.
- **Reference Data** -- Generic reference term lookups (categories, difficulty levels, content types, genders, designations, etc.) seeded from CSV files.

## Getting Started

### Prerequisites

- .NET 8 SDK
- Node.js (18+)
- PostgreSQL (running)

### Backend

```bash
# 1. Configure database connection in LAP.API/appsettings.json
#    ConnectionStrings:DefaultConnection = "Host=...;Port=5432;Database=...;Username=...;Password=..."

# 2. Set "Seeding": true on first run (seeds reference data + auth data)

# 3. Restore and run
dotnet restore LearningPortalAPI.sln
dotnet run --project LAP.API

# API: http://localhost:5020 (HTTP) or https://localhost:7180 (HTTPS)
# Swagger UI: /swagger in Development mode
```

### Frontend

```bash
# 1. Install dependencies
cd learning-and-assessment-portal-frontend-development
npm install

# 2. Configure .env (default: VITE_API_BASE_URL=http://localhost:5020)

# 3. Run dev server
npm run dev
# Frontend: http://localhost:5173
```

Other scripts:

```bash
npm run build        # Production build (tsc + vite build)
npm run preview      # Preview production build
npm run lint         # ESLint
npm run test         # Jest unit tests
npm run test:watch   # Jest in watch mode
```

### Regenerating the API Client (Orval)

With the backend running at `http://localhost:5020`, run from the frontend directory:

```bash
npx orval
```

This regenerates typed Axios API services and models from the Swagger spec.

## Configuration

### Backend (`appsettings.json`)

| Section                      | Key                       | Default                       | Description                    |
| ---------------------------- | ------------------------- | ----------------------------- | ------------------------------ |
| `ConnectionStrings`          | `DefaultConnection`       | _(set yours)_                 | PostgreSQL connection string   |
| `JwtSettings`                | `SecretKey`               | _(empty — must be set)_       | JWT signing key                |
| `JwtSettings`                | `Issuer`                  | `LearningPortalAPI`           | JWT issuer                     |
| `JwtSettings`                | `Audience`                | `LearningPortalClient`        | JWT audience                   |
| `JwtSettings`                | `ExpiryInMinutes`         | `60`                          | Access token TTL               |
| `JwtSettings`                | `RefreshTokenExpiryInDays`| `7`                           | Refresh token TTL              |
| `Seeding`                    |                           | `false`                       | Enable on first run            |
| `CorsSettings`               | `AllowedOrigins`          | `["http://localhost:5173"]`   | CORS allowed origins           |
| `FileStorageOptions`         | `StorageRoot`             | `C:/LAP`                      | Root for file uploads          |
| `FileStorageOptions`         | `QuestionTemplatePath`    | `C:/Template`                 | Question import/export templates |
| `Serilog:WriteTo:File:Args`  | `path`                    | `C:/LAP/Logs/log-.txt`        | Log file path (daily rolling)  |

### Frontend (`.env`)

| Variable            | Default                  | Description      |
| ------------------- | ------------------------ | ---------------- |
| `VITE_API_BASE_URL` | `http://localhost:5020`  | Backend API base URL |

## Testing

### Backend

- **Unit Tests** (`LAP.Test/LAP.UnitTest/`) -- xUnit with Moq and InMemory EF Core, organized by feature (Auth, Assessment, Course, Enrollment, Forum, Leaderboard, ReferenceData, User, etc.) plus validators and mapping profiles.
- **Integration Tests** (`LAP.Test/LAP.IntegrationTest/`) -- xUnit using `WebApplicationFactory<Program>` targeting actual controller endpoints, with a `TestAuthHandler` for authenticated requests.
- **Coverage** -- Coverlet with Cobertura XML output.

### Frontend

- **Unit Tests** (`src/tests/unit/`) -- Jest with React Testing Library (jsdom), mirroring the feature structure across admin, auth, home, leaderboard, user, and shared modules.