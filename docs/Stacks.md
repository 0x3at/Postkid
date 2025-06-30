## 🛠️ Simplified Technology Stack Implementations

This document outlines the refined and simplified technology stacks for each implementation of the API Playground. The goal is to focus on core functionalities, industry-relevant technologies, and a rapid development cycle (aiming for one week per stack). Shared infrastructure components like PostgreSQL and Redis (for caching/rate-limiting) are assumed where specified.

### 1. Python/Django Stack (Reference Implementation)

**Backend Technologies:**
- **Framework**: Django 5.0 + Django REST Framework 3.14
- **Database**: PostgreSQL 15 (shared) with psycopg3
- **ORM**: Django ORM with migrations
- **Authentication**: django-rest-framework-simplejwt + django-cors-headers
- **Rate Limiting**: django-ratelimit (can use Redis if available for distributed limiting)
- **API Documentation**: drf-spectacular (OpenAPI 3.0)
- **Testing**: pytest-django + factory-boy + pytest-cov
- **Linting**: black + flake8 + isort + mypy
- **Task Processing**: *Initial: Synchronous or simple Django Background Tasks. Defer Celery unless Redis is actively used for other core features.*

**Frontend Technologies (React):**
- **Framework**: React 18 + TypeScript 5.0
- **Build Tool**: Vite 4.0
- **UI Library**: Tailwind CSS 3.3 + Headless UI
- **State Management**: Zustand + React Query (TanStack Query)
- **Routing**: React Router 6
- **Forms**: React Hook Form + Zod validation
- **HTTP Client**: Axios with interceptors
- **Testing**: Jest + React Testing Library + MSW

**DevOps & Infrastructure:**
- **Containerization**: Docker + Docker Compose
- **Web Server**: Nginx (as gateway) + Gunicorn (app server)
- **CI/CD**: GitHub Actions
- **Monitoring**: Django Debug Toolbar + django-extensions (for development), standard logging
- **Configuration**: python-decouple or environment variables

### 2. Java/Spring Boot Stack

**Backend Technologies:**
- **Framework**: Spring Boot 3.2 + Spring Web + Spring Security 6
- **Database**: PostgreSQL (shared) with HikariCP
- **ORM**: Spring Data JPA + Hibernate 6.2
- **Authentication**: Spring Security + JWT
- **Rate Limiting**: Bucket4j (can use Redis if available for distributed limiting)
- **API Documentation**: SpringDoc OpenAPI 3 (Swagger)
- **Testing**: JUnit 5 + Mockito + TestContainers
- **Build Tool**: Maven 3.9 or Gradle 8.0
- **Code Quality**: SpotBugs + Checkstyle + JaCoCo
- **Task Processing**: *Initial: Spring `@Async` methods. Defer RabbitMQ/external message brokers.*

**Frontend Technologies (Angular):**
- **Framework**: Angular 17 + TypeScript 5.0
- **UI Library**: Angular Material + CDK
- **State Management**: NgRx + RxJS
- **HTTP Client**: Angular HttpClient with interceptors
- **Forms**: Angular Reactive Forms
- **Testing**: Jasmine + Karma (unit/component), Playwright/Cypress (for E2E if essential)
- **Build Tool**: Angular CLI

**DevOps & Infrastructure:**
- **Containerization**: Docker (multi-stage builds)
- **Application Server**: Embedded Tomcat
- **CI/CD**: GitHub Actions
- **Monitoring**: Micrometer + Spring Boot Actuator, standard logging
- **Configuration**: Standard Spring Boot `application.yml` or `application.properties`

### 3. C#/.NET Stack

**Backend Technologies:**
- **Framework**: .NET 8 Web API + ASP.NET Core
- **Database**: PostgreSQL (shared)
- **ORM**: Entity Framework Core 8.0
- **Authentication**: ASP.NET Core Identity + JWT Bearer
- **Rate Limiting**: AspNetCoreRateLimit (can use Redis if available)
- **API Documentation**: Swashbuckle (OpenAPI/Swagger)
- **Testing**: xUnit + Moq + FluentAssertions + TestContainers
- **Code Quality**: Roslyn Analyzers (built-in or StyleCop)
- **Task Processing**: *Initial: ASP.NET Core `BackgroundService`. Defer Hangfire.*

**Frontend Technologies (Blazor WASM):**
- **Framework**: Blazor WebAssembly
- **UI Library**: Blazor Bootstrap or a similar Blazor component library
- **State Management**: Blazor built-in state management features
- **HTTP Client**: HttpClient (configured for Blazor)
- **Testing**: bUnit

**DevOps & Infrastructure:**
- **Containerization**: Docker (Linux containers)
- **Web Server**: Kestrel
- **CI/CD**: GitHub Actions
- **Monitoring**: Standard ASP.NET Core logging (e.g., Serilog to console)
- **Configuration**: `appsettings.json` and environment variables

### 4. Go/Fiber Stack

**Backend Technologies:**
- **Framework**: Go 1.21 + Fiber v2.50
- **Database**: PostgreSQL (shared) with pgx driver
- **Data Access**: sqlc for type-safe queries (preferred for simplicity over GORM)
- **Authentication**: golang-jwt + custom Fiber middleware
- **Rate Limiting**: `golang.org/x/time/rate` (in-memory) or Redis-backed if available
- **API Documentation**: go-swagger or swag (OpenAPI)
- **Testing**: Testify + GoMock + httptest
- **Build Tool**: Go modules + Makefile
- **Code Quality**: golangci-lint + gofmt + go vet
- **Task Processing**: *Initial: Go routines and channels. Defer Asynq unless Redis is actively used.*

**Frontend Technologies (SvelteKit):**
- **Framework**: SvelteKit + TypeScript
- **UI Library**: Tailwind CSS + Skeleton UI (or similar Svelte UI lib)
- **State Management**: Svelte stores
- **HTTP Client**: Native `fetch` API (wrapped in services)
- **Forms**: Svelte forms (e.g., svelte-forms-lib or custom)
- **Testing**: Vitest + Playwright
- **Build Tool**: Vite + SvelteKit adapter

**DevOps & Infrastructure:**
- **Containerization**: Docker (Alpine or scratch base)
- **Web Server**: Fiber (embedded), Nginx (as gateway)
- **CI/CD**: GitHub Actions
- **Monitoring**: Standard Go logging, `net/http/pprof` for profiling
- **Configuration**: Viper or standard environment variables

### 5. Node.js/TypeScript Stack

**Backend Technologies (Express.js):**
- **Framework**: Node.js 20 + Express 4.18
- **Database**: PostgreSQL (shared) with node-postgres (pg)
- **ORM**: Prisma 5.0 + Prisma Client
- **Authentication**: Passport.js + JWT strategy (passport-jwt)
- **Rate Limiting**: `express-rate-limit` (can use Redis store if available)
- **API Documentation**: Swagger JSDoc + swagger-ui-express
- **Testing**: Jest + Supertest + @testcontainers/postgresql
- **Build Tool**: TypeScript + esbuild (or tsc)
- **Code Quality**: ESLint + Prettier + TypeScript (strict mode)
- **Task Processing**: *Initial: Simple async operations or basic in-memory queue pattern. Defer Bull/Agenda unless Redis is actively used.*

**Frontend Technologies (Next.js):**
- **Framework**: Next.js 14 + TypeScript (App Router)
- **UI Library**: Tailwind CSS + shadcn/ui (or Radix UI)
- **State Management**: Zustand + React Query (TanStack Query)
- **HTTP Client**: Native `fetch` API (wrapped in custom hooks/services)
- **Forms**: React Hook Form + Zod
- **Testing**: Jest + React Testing Library
- **Build Tool**: Next.js built-in (Turbopack/Webpack)

**DevOps & Infrastructure:**
- **Containerization**: Docker (Node Alpine base)
- **Runtime**: `node` (PM2 optional for non-orchestrated environments)
- **CI/CD**: GitHub Actions
- **Monitoring**: Winston (for structured logging to console)
- **Configuration**: `dotenv` for local development, environment variables for production
---
