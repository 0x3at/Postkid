# API Playground - Implementation Tracker
## Development Roadmap & Technical Breakdown

### 📊 Implementation Overview

Each stack follows identical phases but leverages technology-specific patterns and best practices. All implementations share the same API contracts (`docs/API-Reference/`) and conceptual data model (`docs/Architecture/Overall Architecture.md`), targeting a PostgreSQL database, while showcasing different architectural approaches as defined in `docs/Tech-Stacks/Technology Stacks.md` and `docs/Specs/Yellow Paper.md`.

**Order of Implementation (Revised)**: Django → Node.js → Java → C#

---

## 🐍 Stack 1: Python/Django + React

### Technology Ecosystem (Aligns with `docs/Stacks.md`)

#### Backend Stack (Django 5.0)
| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Framework** | Django + Django REST Framework | 5.0 + 3.14 | REST API, ORM, Admin |
| **Database** | PostgreSQL | 15+ | Primary data store |
| **Database Driver** | psycopg3 | Latest | PostgreSQL adapter |
| **Cache** | Redis (Optional) | 7.0+ | Rate limiting (if distributed), caching |
| **Task Processing** | Synchronous / Django Background Tasks | N/A | Initial async request execution |
| **Authentication** | `django-rest-framework-simplejwt` | Latest | JWT token management |
| **Rate Limiting** | `django-ratelimit` | Latest | API abuse prevention |
| **CORS** | `django-cors-headers` | Latest | Cross-origin requests |
| **API Docs** | `drf-spectacular` | Latest | OpenAPI 3.0 generation |
| **Validation** | DRF Serializers / Pydantic (Optional) | 3.14 / Latest | Request/response validation |
| **Testing** | `pytest-django`, `factory-boy`, `pytest-cov` | Latest | Unit and integration tests |
| **Code Quality** | `black`, `flake8`, `isort`, `mypy` | Latest | Formatting and linting |
| **Configuration** | `python-decouple` / Env Vars | Latest | Environment configuration |

#### Frontend Stack (React 18)
| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Framework** | React | 18.x | UI components |
| **Language** | TypeScript | 5.0.x | Type safety |
| **Build Tool** | Vite | 4.0.x | Fast development builds |
| **UI Library** | Tailwind CSS | 3.3.x | Utility-first styling |
| **Components** | Headless UI | Latest | Accessible components |
| **State Management** | Zustand | Latest | Client state |
| **Server State** | TanStack Query (React Query) | Latest | API data management |
| **Routing** | React Router | 6.x | Client-side routing |
| **Forms** | React Hook Form + Zod | Latest | Form handling & validation |
| **HTTP Client** | Axios | Latest | API requests |
| **Testing** | Jest + React Testing Library + MSW | Latest | Component testing |

### Implementation Phases (Simplified Task Processing)

#### Phase 1: Foundation & Authentication (Week 1)
**Epic: User Authentication System**

**Backend Tasks:**
- [ ] **Project Setup**
  - Initialize Django project, configure settings (`python-decouple`).
  - Configure PostgreSQL connection (psycopg3).
  - Set up Redis (optional, for `django-ratelimit` if distributed).
  - Create Docker development environment (`docker-compose.yml`).
  - Configure linters/formatters (`black`, `flake8`, `isort`, `mypy`) and pre-commit hooks.
- [ ] **User Model & Authentication**
  - Extend Django User model (if custom fields needed beyond `first_name`, `last_name`).
  - Configure DRF with `django-rest-framework-simplejwt`.
  - Implement `/auth/register` endpoint with email validation.
  - Implement `/auth/login`, `/auth/logout` endpoints.
  - Implement `/auth/refresh` endpoint.
  - Configure `django-cors-headers`.
- [ ] **Security Implementation**
  - Set up `django-ratelimit` for auth endpoints.
  - Implement password validation rules (Django built-in or custom).
  - Configure JWT settings (lifetimes, rotation if applicable via `simplejwt`).
  - Add basic request logging middleware.

**Frontend Tasks:**
- [ ] **Project Setup**
  - Initialize React project with TypeScript and Vite
  - Configure Tailwind CSS with custom design tokens
  - Set up ESLint, Prettier, and TypeScript strict mode
  - Configure absolute imports and path mapping
  - Set up testing environment (Jest, RTL, MSW)

- [ ] **Authentication UI**
  - Create responsive login form with validation
  - Create registration form with password confirmation
  - Implement JWT token storage and refresh logic
  - Create protected route wrapper component
  - Add loading states and error handling
  - Create user context and authentication hooks

**Acceptance Criteria:**
- [ ] Users can register with email validation and strong password requirements
- [ ] Login returns JWT tokens with proper expiration handling
- [ ] Token refresh works automatically on API calls
- [ ] Rate limiting prevents brute force attacks
- [ ] All forms have proper validation and error states
- [ ] Authentication state persists across browser refreshes

#### Phase 2: Collection Management (Week 1.5)
**Epic: API Collection CRUD**

**Backend Tasks:**
- [ ] **Data Models**
  - Create Collection model with user foreign key
  - Add validation for collection names (unique per user)
  - Implement soft delete functionality
  - Create database migrations
  - Add model signals for audit logging

- [ ] **API Endpoints**
  - Implement CollectionViewSet with proper permissions
  - Add filtering, pagination, and search functionality
  - Create collection sharing endpoints (bonus)
  - Add collection statistics endpoints (endpoint count, last used)
  - Implement collection duplication feature

- [ ] **Serializers & Validation**
  - Create nested serializers for collection with endpoint counts
  - Add custom validation for collection names
  - Implement permission checks for collection access
  - Add data transformation for API responses

**Frontend Tasks:**
- [ ] **Collection Management UI**
  - Create collection list view with search and filters
  - Implement collection creation modal with form validation
  - Add collection editing and deletion functionality
  - Create collection cards with statistics and quick actions
  - Implement drag-and-drop reordering
  - Add bulk operations (delete, export)

- [ ] **State Management**
  - Create collection store with Zustand
  - Implement optimistic updates for CRUD operations
  - Add React Query for server state management
  - Create collection-related custom hooks
  - Add error boundary for collection operations

**Acceptance Criteria:**
- [ ] Users can create collections with unique names
- [ ] Collection list shows statistics (endpoint count, last modified)
- [ ] Search and filtering work correctly
- [ ] CRUD operations have proper error handling
- [ ] UI updates optimistically for better UX
- [ ] Collections are scoped to authenticated users only

#### Phase 3: Endpoint Builder (Week 2)
**Epic: API Endpoint Configuration**

**Backend Tasks:**
- [ ] **Endpoint Model**
  - Create Endpoint model with Collection foreign key
  - Add JSON fields for headers, query params, body
  - Implement HTTP method validation
  - Add URL validation with security checks (no private IPs)
  - Create timeout configuration with limits

- [ ] **API Implementation**
  - Create EndpointViewSet with nested routing under collections
  - Implement variable interpolation system for URLs/headers
  - Add endpoint validation for common security issues
  - Create endpoint duplication and templating features
  - Add endpoint import/export functionality

- [ ] **Security & Validation**
  - Implement SSRF protection (block private networks)
  - Add URL scheme validation (HTTP/HTTPS only)
  - Validate JSON structures for headers and body
  - Add request size limits
  - Implement timeout validation (1-30 seconds)

**Frontend Tasks:**
- [ ] **Endpoint Builder UI**
  - Create tabbed interface for endpoint configuration
  - Implement HTTP method selector with color coding
  - Add URL input with variable highlighting
  - Create headers editor with key-value pairs
  - Implement query parameters builder
  - Add JSON body editor with syntax highlighting and validation

- [ ] **Advanced Features**
  - Create variable management system
  - Add pre-request script editor (Monaco editor)
  - Implement test script editor for response validation
  - Add endpoint documentation section
  - Create endpoint templates and snippets

- [ ] **UX Enhancements**
  - Add real-time validation feedback
  - Implement auto-save functionality
  - Create endpoint preview panel
  - Add copy/paste functionality for endpoints
  - Implement keyboard shortcuts for common actions

**Acceptance Criteria:**
- [ ] All HTTP methods supported with proper validation
- [ ] Variable interpolation works in URLs, headers, and body
- [ ] Security validation prevents SSRF attacks
- [ ] JSON editors provide syntax highlighting and error detection
- [ ] Auto-save prevents data loss
- [ ] Endpoint configuration is intuitive and responsive

#### Phase 4: Request Execution Engine (Week 2.5)
**Epic: HTTP Request Processing**

**Backend Tasks:**
- [ ] **Request Execution Service**
  - Create HTTP client service (e.g., using `requests` library) with configurable timeouts.
  - Implement variable interpolation.
  - Add detailed request/response logging to `RequestLog` model.
  - **If async needed for non-blocking IO during external calls:** Use Django's async views or `django-background-tasks` for simple deferral. *No Celery setup.*
  - Implement response parsing and storage.
- [ ] **Security & Rate Limiting** (as original, ensure `django-ratelimit` covers execution endpoints).
- [ ] **Response Processing** (as original).

**Frontend Tasks:** (Largely as original, focusing on clear status updates)
- [ ] **Request Execution UI**.
- [ ] **Response Features**.
- [ ] **Execution Management** (Cancel may be harder without dedicated task queue, focus on timeout). *Retry mechanism might be client-side or simple backend logic for now.*

**Acceptance Criteria:** (As original, adjusted for simplified async).

#### Phase 5: Request History & Logging (Week 3)
**Epic: Request Tracking and Analytics**
(Tasks remain similar, focused on DRF/Django ORM query optimization and React UI for display).
- [ ] Backend: `RequestLog` model enhancements, Analytics API (DRF), Performance Optimization (DB queries). *Log processing via simple background tasks if needed, not Celery.*

#### Phase 6: Advanced Features & Polish (Week 3.5)
**Epic: Production Ready Features**
(Tasks like WebSocket for real-time updates are good but complex; confirm if in initial scope per YellowPaper. Team features are bonus.)
- [ ] **WebSocket Implementation (If core to MVP)**
  - Set up Django Channels (if chosen for WebSockets).
  - Create consumers for relevant real-time updates.
- [ ] **Production Optimizations** (Error handling, health checks, metrics - standard Django/DRF practices).

---

## 🟨 Stack 2: Node.js/TypeScript (Express.js + Next.js)

### Technology Ecosystem (Aligns with `docs/Stacks.md`)

#### Backend Stack (Express.js)
| Component          | Technology                             | Version         | Purpose                                      |
|--------------------|----------------------------------------|-----------------|----------------------------------------------|
| **Runtime**        | Node.js                                | 20.x            | JavaScript runtime                           |
| **Framework**      | Express.js                             | 4.18.x          | Web framework                                |
| **Language**       | TypeScript                             | 5.x             | Type safety                                  |
| **Build Tool**     | `tsc` or `esbuild`                     | Latest          | TypeScript compilation                       |
| **Database**       | PostgreSQL                             | 15+             | Primary data store                           |
| **ORM**            | Prisma                                 | 5.x             | Type-safe database client                    |
| **Authentication** | Passport.js (`passport-jwt` strategy)  | Latest          | Auth strategies                              |
| **JWT Library**    | `jsonwebtoken`                         | Latest          | Token management                             |
| **Rate Limiting**  | `express-rate-limit`                   | Latest          | API protection (optional Redis store)        |
| **API Docs**       | `swagger-jsdoc` + `swagger-ui-express` | Latest          | OpenAPI 3.0 generation                       |
| **Validation**     | Zod (or similar like Joi)              | Latest          | Schema validation                            |
| **Testing**        | Jest, Supertest, `@testcontainers/postgresql` | Latest    | Testing framework                            |
| **Task Processing**| Simple Async Operations / In-Memory Queue | N/A            | Initial async operations (No Bull/Agenda)    |
| **HTTP Client**    | `axios` or Node.js `fetch`             | Latest          | External API calls                           |
| **Configuration**  | `dotenv` / Env Vars                    | Latest          | Environment configuration                    |
| **Code Quality**   | ESLint, Prettier, TypeScript (strict mode) | Latest      | Formatting and linting                       |

#### Frontend Stack (Next.js 14)
| Component            | Technology                             | Version    | Purpose                                  |
|----------------------|----------------------------------------|------------|------------------------------------------|
| **Framework**        | Next.js (App Router)                   | 14.x       | React framework                          |
| **Language**         | TypeScript                             | 5.x        | Type safety                              |
| **UI Library**       | Tailwind CSS                           | 3.x        | Styling                                  |
| **Components**       | `shadcn/ui` (or Radix UI)              | Latest     | Accessible components                    |
| **State Management** | Zustand                                | Latest     | Client state                             |
| **Server State**     | TanStack Query (React Query)           | Latest     | Data fetching & caching                  |
| **Forms**            | React Hook Form + Zod                  | Latest     | Form handling & validation               |
| **HTTP Client**      | Native `fetch` API (wrapped)           | N/A        | API requests to Express backend          |
| **Testing**          | Jest + React Testing Library + MSW     | Latest     | Component testing                        |

### Implementation Phases (Aligning with Django's detailed phases)

#### Phase 1: Foundation & Authentication (Express Backend, Next.js Client)
**Epic: User Authentication System**

**Backend Tasks (Express.js):**
- [ ] **Project Setup**
  - Initialize Node.js project with TypeScript (`tsc --init` or `esbuild` setup).
  - Set up Express.js server structure.
  - Configure Prisma ORM with PostgreSQL connection.
  - Create Docker development environment (`docker-compose.yml` for Express, DB, Redis if used for rate-limiting).
  - Configure linters/formatters (ESLint, Prettier) and pre-commit hooks.
- [ ] **User Model & Authentication (Prisma + Passport.js)**
  - Define User model in `schema.prisma`. Run migrations.
  - Implement Passport.js with `passport-jwt` strategy for JWT authentication.
  - Create services/controllers for user registration (with email validation, password hashing using `bcrypt`).
  - Implement `/auth/register` endpoint.
  - Implement `/auth/login` endpoint (issue JWTs - access & refresh).
  - Implement `/auth/logout` endpoint (e.g., if using token blocklist with Redis, or client-side only).
  - Implement `/auth/refresh` endpoint.
  - Configure CORS middleware for Next.js frontend origin.
- [ ] **Security Implementation**
  - Set up `express-rate-limit` for auth endpoints.
  - Implement password strength validation (e.g., using `zxcvbn` or custom rules).
  - Configure JWT settings (secret, expiration times).
  - Add basic request logging middleware (e.g., `morgan`).

**Frontend Tasks (Next.js):**
- [ ] **Project Setup**
  - Initialize Next.js 14 project with TypeScript and App Router (`create-next-app`).
  - Configure Tailwind CSS and `shadcn/ui` (or Radix UI).
  - Set up ESLint, Prettier, and TypeScript strict mode.
  - Configure absolute imports (`paths` in `tsconfig.json`).
  - Set up testing environment (Jest, React Testing Library, MSW).
- [ ] **Authentication UI & Logic**
  - Create responsive Login and Registration pages/components using React Hook Form and Zod for validation.
  - Implement client-side JWT storage (e.g., secure cookie, local storage - consider security implications) and refresh logic (calling backend refresh endpoint).
  - Create protected route/layout components or middleware for Next.js App Router.
  - Implement loading states and error handling for auth operations.
  - Create Zustand store and/or React Context for authentication state and user information.
  - Develop custom hooks for accessing auth state and actions.

**Acceptance Criteria:** (Same as Django stack)
- [ ] Users can register with email validation and strong password requirements.
- [ ] Login returns JWT tokens with proper expiration handling.
- [ ] Token refresh works correctly.
- [ ] Rate limiting prevents brute force attacks on auth endpoints.
- [ ] All forms have proper client-side and server-side validation and error states.
- [ ] Authentication state persists across browser refreshes on the client.

#### Phase 2: Collection Management (Express Backend, Next.js Client)
**Epic: API Collection CRUD**

**Backend Tasks (Express.js):**
- [ ] **Data Models (Prisma)**
  - Define `Collection` model in `schema.prisma` (user foreign key, name, description, etc.).
  - Add validation rules (e.g., unique name per user) at service/controller level.
  - Implement soft delete functionality (e.g., `deletedAt` field or status flag).
  - Run Prisma migrations.
  - (Optional) Add audit logging for model changes if not handled by a generic logger.
- [ ] **API Endpoints (Express Routes & Controllers)**
  - Implement Express router for `/collections` (GET list, POST create).
  - Implement Express router for `/collections/:id` (GET one, PUT update, DELETE one).
  - Ensure endpoints are protected by Passport.js JWT authentication.
  - Implement proper authorization (user can only access/modify their own collections).
  - Add filtering (e.g., by name), pagination, and sorting for listing collections.
  - (Bonus) Collection sharing, statistics, duplication features.
- [ ] **Services & Validation (Zod/Custom)**
  - Create services to encapsulate business logic for collections.
  - Use Zod schemas for validating request payloads for create/update operations.
  - Ensure responses are structured according to `docs/API-Reference/`.

**Frontend Tasks (Next.js):**
- [ ] **Collection Management UI**
  - Create Collection list page/component (displaying cards/table).
  - Implement search and filter UI for collections.
  - Create Collection creation/editing modal/page with React Hook Form + Zod.
  - Implement delete confirmation for collections.
- [ ] **State Management (Zustand + TanStack Query)**
  - Use Zustand for any local UI state related to collections.
  - Use TanStack Query for fetching, caching, and mutating collection data from the backend.
  - Implement optimistic updates for CRUD operations for better UX.
  - Create custom hooks for collection data operations.
  - Implement error boundaries or local error handling for collection operations.

**Acceptance Criteria:** (Same as Django stack)

#### Phase 3: Endpoint Builder (Express Backend, Next.js Client)
**Epic: API Endpoint Configuration**

**Backend Tasks (Express.js):**
- [ ] **Data Model (Prisma)**
  - Define `Endpoint` model in `schema.prisma` (collection foreign key, method, URL, headers, query params, body, timeout, etc.).
  - Run Prisma migrations.
- [ ] **API Implementation (Express Routes & Controllers)**
  - Implement nested routes under collections: `/collections/:collectionId/endpoints` (GET list, POST create).
  - Implement routes for specific endpoints: `/collections/:collectionId/endpoints/:endpointId` (GET one, PUT update, DELETE one).
  - Implement variable interpolation system logic (if handled server-side before saving/execution).
  - (Bonus) Endpoint duplication, templating, import/export logic.
- [ ] **Security & Validation (Zod/Custom)**
  - Implement SSRF protection for URL field (block private networks, validate schemes).
  - Validate JSON structures for headers, body.
  - Enforce request size limits (via Express middleware like `body-parser` limits).
  - Validate timeout settings (e.g., 1-30 seconds).

**Frontend Tasks (Next.js):**
- [ ] **Endpoint Builder UI**
  - Create tabbed interface for endpoint configuration (URL, Method, Headers, Query Params, Body, Auth, Pre-request Script, Tests).
  - Implement HTTP method selector.
  - URL input with variable highlighting.
  - Key-value pair editor for headers and query parameters.
  - JSON body editor (e.g., using Monaco Editor or a simpler `textarea` with validation).
- [ ] **Advanced Features UI**
  - Variable management system UI.
  - Pre-request script editor (e.g., Monaco Editor).
  - Test script editor (e.g., Monaco Editor).
- [ ] **UX Enhancements** (Real-time validation, auto-save consideration, preview panel).

**Acceptance Criteria:** (Same as Django stack)

#### Phase 4: Request Execution Engine (Express Backend, Next.js Client)
**Epic: HTTP Request Processing**

**Backend Tasks (Express.js):**
- [ ] **Request Execution Service**
  - Create service to handle outgoing HTTP requests (using `axios` or Node `fetch`).
  - Implement variable interpolation engine if not done at endpoint save time.
  - Implement detailed request/response logging to `RequestLog` model via Prisma.
  - For async execution (if needed beyond simple `async/await` for the external call): Use simple in-memory queue or `setImmediate` for light deferral. *No Bull/Agenda setup.*
  - Implement response parsing (JSON, XML, HTML, text, binary) and storage.
- [ ] **Security & Rate Limiting (`express-rate-limit`)**
  - Apply rate limiting to request execution endpoints.
  - Enforce request size limits for outgoing requests.
  - Implement timeout enforcement for external calls.
- [ ] **Response Processing** (Capture headers, metadata, body, time, size).

**Frontend Tasks (Next.js):**
- [ ] **Request Execution UI** (Panel for variables, real-time status placeholder, response viewer with syntax highlighting, headers/metadata display, time/size indicators).
- [ ] **Response Features** (Format toggle, search, export - some may be simplified initially).
- [ ] **Execution Management** (Request cancellation via `AbortController` if using `fetch`/`axios`. No complex queue display).

**Acceptance Criteria:** (Same as Django stack, adjusted for simplified async).

#### Phase 5: Request History & Logging (Express Backend, Next.js Client)
**Epic: Request Tracking and Analytics**

**Backend Tasks (Express.js):**
- [ ] **RequestLog Model Enhancement (Prisma)**: Optimize queries, consider full-text search if DB supports well with Prisma.
- [ ] **Analytics API (Express Routes & Controllers)**: Endpoints for stats, filtering, aggregation.
- [ ] **Performance Optimization**: Database query optimization (Prisma query tuning), pagination for logs. *Log processing via simple async operations if needed.*

**Frontend Tasks (Next.js):** (Largely as original, using TanStack Query for data fetching).
- [ ] **History Dashboard**.
- [ ] **Log Management**.
- [ ] **Data Visualization**.

**Acceptance Criteria:** (Same as Django stack).

#### Phase 6: Advanced Features & Polish (Express Backend, Next.js Client)
**Epic: Production Ready Features**

**Backend Tasks (Express.js):**
- [ ] **WebSocket Implementation (If core to MVP)**
  - Set up WebSocket library (e.g., `ws` or `socket.io`) with Express.
  - Implement logic for real-time updates (e.g., request execution status).
- [ ] **Production Optimizations**: Comprehensive error handling, health check endpoint (`/health`), metrics collection (e.g., `prom-client`), security headers (e.g., `helmet`).

**Frontend Tasks (Next.js):**
- [ ] **Real-time Features** (WebSocket client-side handling).
- [ ] **UI/UX Polish** (Loading skeletons, error boundaries, accessibility, responsive design, dark mode).
- [ ] **Performance Features** (Virtual scrolling, lazy loading, PWA considerations if applicable).

**Acceptance Criteria:** (Same as Django stack).

---

## ☕ Stack 3: Java/Spring Boot + Angular

### Technology Ecosystem (Aligns with `docs/Stacks.md`)

#### Backend Stack (Spring Boot 3.2)
| Component          | Technology                                   | Version      | Purpose                                      |
|--------------------|----------------------------------------------|--------------|----------------------------------------------|
| **Language**       | Java                                         | 17/21 LTS    | Programming language                         |
| **Framework**      | Spring Boot (inc. Spring Web, Spring Security 6) | 3.2.x      | Application framework                        |
| **Data Access**    | Spring Data JPA (Hibernate)                  | 3.2.x (HB 6.2) | Data access layer                            |
| **Connection Pool**| HikariCP                                     | Bundled      | DB connection pooling                        |
| **Database**       | PostgreSQL                                   | 15+          | Primary database                             |
| **Authentication** | Spring Security (JWT based)                  | 6.x          | Authentication/authorization                 |
| **Rate Limiting**  | Bucket4j                                     | Latest       | API protection (optional Redis store)        |
| **API Docs**       | SpringDoc OpenAPI 3                          | Latest       | API documentation (Swagger UI)               |
| **Task Processing**| Spring `@Async` methods                      | N/A          | Initial async operations (No RabbitMQ)       |
| **Validation**     | Spring Validation (Hibernate Validator)      | Latest       | Request validation                           |
| **Testing**        | JUnit 5, Mockito, Spring Test, Testcontainers | Latest    | Testing framework                            |
| **Build Tool**     | Maven / Gradle                               | 3.9+ / 8.0+  | Build automation                           |
| **Code Quality**   | SpotBugs, Checkstyle, JaCoCo                 | Latest       | Static analysis, coverage                    |
| **Configuration**  | `application.yml` / Env Vars                 | N/A          | Environment configuration                    |

#### Frontend Stack (Angular 17)
| Component            | Technology                             | Version    | Purpose                                      |
|----------------------|----------------------------------------|------------|----------------------------------------------|
| **Framework**        | Angular                                | 17.x       | Frontend framework                           |
| **Language**         | TypeScript                             | 5.0.x      | Type safety                                  |
| **UI Library**       | Angular Material + CDK                   | 17.x       | Component library                            |
| **State Management** | NgRx + RxJS                              | 17.x       | State management, reactivity                 |
| **HTTP Client**      | Angular `HttpClient`                     | 17.x       | HTTP requests                                |
| **Forms**            | Angular Reactive Forms                   | 17.x       | Form handling & validation                   |
| **Testing**          | Jasmine + Karma (Playwright/Cypress for E2E if essential) | Latest | Testing framework                  |
| **Build Tool**       | Angular CLI                            | 17.x       | Build & development tool                     |

### Implementation Phases (Aligning with Django's detailed phases)

#### Phase 1: Foundation & Authentication (Spring Boot Backend, Angular Client)
**Epic: User Authentication System**

**Backend Tasks (Spring Boot):**
- [ ] **Project Setup**
  - Initialize Spring Boot project (Maven/Gradle), Java 17/21.
  - Configure `application.yml` for profiles (dev, prod).
  - Set up PostgreSQL connection (Spring Data JPA, Hibernate, HikariCP).
  - Create Docker development environment (`docker-compose.yml` for Spring Boot, DB, Redis if used for rate-limiting).
  - Configure code quality tools (SpotBugs, Checkstyle, JaCoCo) in build file.
- [ ] **User Model & Authentication (JPA Entities + Spring Security)**
  - Define `UserEntity` with JPA annotations.
  - Implement Spring Security configuration (`SecurityFilterChain`) for JWT authentication.
  - Create `UserDetailsService` implementation.
  - Develop services/controllers for user registration (email validation, password hashing with `PasswordEncoder`).
  - Implement `/auth/register` endpoint.
  - Implement `/auth/login` endpoint (issue JWTs - access & refresh).
  - Implement `/auth/logout` endpoint (if server-side action needed).
  - Implement `/auth/refresh` endpoint.
  - Configure CORS (e.g., via `WebMvcConfigurer` or Spring Security).
- [ ] **Security Implementation**
  - Set up Bucket4j for rate limiting auth endpoints.
  - Implement password strength validation.
  - Configure JWT properties (secret, expiration) via application properties.
  - Add request logging filter/interceptor.

**Frontend Tasks (Angular):**
- [ ] **Project Setup**
  - Initialize Angular 17 project using Angular CLI (with TypeScript 5.0).
  - Set up Angular Material and CDK.
  - Configure ESLint, Prettier, and TypeScript strict mode.
  - Set up path mapping in `tsconfig.json`.
  - Configure testing environment (Jasmine, Karma).
- [ ] **Authentication UI & Logic**
  - Create Login and Registration components with Angular Reactive Forms and Material components.
  - Implement client-side JWT storage (e.g., `localStorage` or a secure cookie strategy if using BFF) and refresh logic.
  - Create route guards (`CanActivate`) for protected routes.
  - Implement loading states and error handling using RxJS patterns.
  - Develop NgRx store (actions, reducers, effects, selectors) for authentication state and user information.
  - Create an `AuthService` and HTTP interceptor for attaching JWTs to outgoing requests and handling 401 errors for token refresh.

**Acceptance Criteria:** (Same as Django stack)

#### Phase 2: Collection Management (Spring Boot Backend, Angular Client)
**Epic: API Collection CRUD**

**Backend Tasks (Spring Boot):**
- [ ] **Data Models (JPA Entities)**
  - Define `CollectionEntity` (user foreign key, name, description, etc.) with JPA annotations.
  - Add validation annotations (`@NotNull`, `@Size`, etc.) on entity fields.
  - Implement soft delete (`@SQLDelete`, `@Where`).
  - Create Flyway/Liquibase migration scripts for schema changes.
  - (Optional) Use `@EntityListeners` for audit logging.
- [ ] **API Endpoints (Spring MVC Controllers)**
  - Implement `@RestController` for `/collections` (GET list, POST create).
  - Implement `@RestController` for `/collections/{id}` (GET one, PUT update, DELETE one).
  - Secure endpoints with Spring Security (`@PreAuthorize` or security matchers).
  - Implement proper authorization (user can only access/modify their own collections).
  - Add filtering, pagination (`Pageable`), and sorting for listing collections.
  - (Bonus) Collection sharing, statistics, duplication features.
- [ ] **Services & DTOs**
  - Create `@Service` classes for collection business logic.
  - Use DTOs for request/response payloads, with validation annotations.
  - Map between Entities and DTOs (manually or with MapStruct).

**Frontend Tasks (Angular):**
- [ ] **Collection Management UI**
  - Create Collection list component (e.g., using Angular Material Table or Cards).
  - Implement search and filter UI.
  - Create Collection creation/editing modal/dialog (Angular Material Dialog) with Reactive Forms.
  - Implement delete confirmation.
- [ ] **State Management (NgRx + RxJS)**
  - Create NgRx store (actions, reducers, effects, selectors) for collections.
  - Use effects for fetching, creating, updating, deleting collections via `HttpClient`.
  - Implement optimistic updates or clear indicators for data changes.
  - Create services and custom pipes/directives as needed.

**Acceptance Criteria:** (Same as Django stack)

#### Phase 3: Endpoint Builder (Spring Boot Backend, Angular Client)
**Epic: API Endpoint Configuration**

**Backend Tasks (Spring Boot):**
- [ ] **Data Model (JPA Entities)**
  - Define `EndpointEntity` (collection foreign key, method, URL, headers, query params, body, timeout, etc.) with JPA annotations.
  - Use appropriate types for JSON-like data (e.g., `String` with custom type converter, or JSONB type if DB/Hibernate supports well).
  - Create Flyway/Liquibase migration scripts.
- [ ] **API Implementation (Spring MVC Controllers)**
  - Implement nested REST controllers: `/collections/{collectionId}/endpoints`.
  - Logic for variable interpolation (if server-side).
- [ ] **Security & Validation**
  - Implement SSRF protection for URL field in service layer.
  - Validate JSON structures, request size (Spring Boot properties), timeout settings.

**Frontend Tasks (Angular):**
- [ ] **Endpoint Builder UI** (Using Angular Material components, Reactive Forms).
  - Tabbed interface for endpoint sections.
  - HTTP method selector.
  - URL input with variable highlighting (custom directive or component).
  - Key-value editor for headers/params.
  - JSON body editor (e.g., ngx-monaco-editor or simpler).
- [ ] **Advanced Features UI** (Variable management, script editors if using Monaco).
- [ ] **UX Enhancements**.

**Acceptance Criteria:** (Same as Django stack)

#### Phase 4: Request Execution Engine (Spring Boot Backend, Angular Client)
**Epic: HTTP Request Processing**

**Backend Tasks (Spring Boot):**
- [ ] **Request Execution Service (`@Service`)**
  - Create service using `WebClient` (preferred for non-blocking) or `RestTemplate` for outgoing HTTP calls.
  - Implement variable interpolation.
  - Detailed request/response logging to `RequestLogEntity` via Spring Data JPA.
  - Use Spring `@Async` for deferring the external call if it's long-running and non-blocking for the user's immediate response is critical.
  - Response parsing and storage.
- [ ] **Security & Rate Limiting (Bucket4j)**
  - Apply rate limiting to request execution endpoints.
  - Enforce request size limits, timeout for external calls.
- [ ] **Response Processing** (Capture headers, metadata, body, time, size).

**Frontend Tasks (Angular):**
- [ ] **Request Execution UI** (Material components, status updates via NgRx/RxJS).
- [ ] **Response Features** (Format toggle, search, export - simplify if needed).
- [ ] **Execution Management** (Request cancellation via `HttpClient` unsubscribe or `AbortController` with `fetch` if `WebClient` is not directly exposed).

**Acceptance Criteria:** (Same as Django stack, adjusted for Spring `@Async`).

#### Phase 5: Request History & Logging (Spring Boot Backend, Angular Client)
**Epic: Request Tracking and Analytics**

**Backend Tasks (Spring Boot):**
- [ ] **RequestLog Model Enhancement (JPA/SQL)**: Optimize queries (`@Query` in repositories), indexing.
- [ ] **Analytics API (Spring MVC Controllers)**: Endpoints for stats, filtering, aggregation using Spring Data Projections or custom queries.
- [ ] **Performance Optimization**: Database query optimization, pagination. *Log processing via `@Async` or `@Scheduled` tasks if needed.*

**Frontend Tasks (Angular):** (Largely as original, using NgRx effects and services for data).
- [ ] **History Dashboard**.
- [ ] **Log Management**.
- [ ] **Data Visualization**.

**Acceptance Criteria:** (Same as Django stack).

#### Phase 6: Advanced Features & Polish (Spring Boot Backend, Angular Client)
**Epic: Production Ready Features**

**Backend Tasks (Spring Boot):**
- [ ] **WebSocket Implementation (If core to MVP)**
  - Set up Spring WebSockets (with STOMP over WebSocket).
  - Implement `@MessageMapping` controllers for real-time updates.
- [ ] **Production Optimizations**: Comprehensive error handling (`@ControllerAdvice`), health check endpoint (Spring Boot Actuator), metrics (Actuator + Micrometer), security headers.

**Frontend Tasks (Angular):**
- [ ] **Real-time Features** (WebSocket client-side handling using libraries like `ngx-stomp` or native WebSockets).
- [ ] **UI/UX Polish** (Loading indicators, Material Design consistency, accessibility, responsive design).
- [ ] **Performance Features** (Lazy loading modules, OnPush change detection, trackBy for lists, PWA considerations).

**Acceptance Criteria:** (Same as Django stack).

---

## 🟣 Stack 4: C#/.NET + Blazor WASM

(Renamed from Stack 5)

### Technology Ecosystem (Aligns with `docs/Stacks.md`)

#### Backend Stack (.NET 8 Web API)
| Component          | Technology                                   | Version    | Purpose                                      |
|--------------------|----------------------------------------------|------------|----------------------------------------------|
| **Framework**      | .NET 8 Web API (ASP.NET Core)                | 8.0.x      | REST API framework                           |
| **Database**       | PostgreSQL                                   | 15+        | Primary database                             |
| **ORM**            | Entity Framework Core                        | 8.0.x      | Object-relational mapping                    |
| **DB Driver**      | Npgsql.EntityFrameworkCore.PostgreSQL        | 8.0.x      | PostgreSQL EF Core provider                  |
| **Authentication** | ASP.NET Core Identity + JWT Bearer           | 8.0.x      | User management, JWT auth                    |
| **Rate Limiting**  | `AspNetCoreRateLimit`                        | Latest     | API protection (optional Redis store)        |
| **API Docs**       | Swashbuckle (OpenAPI/Swagger)                | Latest     | API documentation                            |
| **Task Processing**| ASP.NET Core `BackgroundService`             | N/A        | Initial async operations (No Hangfire)       |
| **Validation**     | Data Annotations / FluentValidation          | N/A        | Model validation                             |
| **Testing**        | xUnit, Moq, FluentAssertions, Testcontainers | Latest     | Testing framework                            |
| **Code Quality**   | Roslyn Analyzers (StyleCop)                  | N/A        | Static analysis                              |
| **Configuration**  | `appsettings.json` / Env Vars                | N/A        | Environment configuration                    |

#### Frontend Stack (Blazor WASM .NET 8)
| Component            | Technology                                   | Version    | Purpose                                      |
|----------------------|----------------------------------------------|------------|----------------------------------------------|
| **Framework**        | Blazor WebAssembly                           | .NET 8     | Component framework                          |
| **UI Library**       | Blazor Bootstrap (or similar)                | Latest     | Component library                            |
| **State Management** | Blazor built-in features / Scoped Services   | N/A        | State management                             |
| **HTTP Client**      | `HttpClient` (Blazor configured)             | N/A        | HTTP requests                                |
| **Forms**            | `EditForm` / Data Annotations                | N/A        | Form handling & validation                   |
| **Testing**          | bUnit                                        | Latest     | Component testing                            |

### Implementation Phases (Aligning with Django's detailed phases)

#### Phase 1: Foundation & Authentication (.NET Backend, Blazor WASM Client)
**Epic: User Authentication System**

**Backend Tasks (.NET Web API):**
- [ ] **Project Setup**
  - Initialize ASP.NET Core Web API project (.NET 8).
  - Configure `appsettings.json` for profiles (Development, Production).
  - Set up PostgreSQL connection with Entity Framework Core (Npgsql provider).
  - Create Docker development environment (`docker-compose.yml` for .NET API, DB, Redis if used for rate-limiting).
  - Configure code quality tools (Roslyn Analyzers, StyleCop).
- [ ] **User Model & Authentication (EF Core + ASP.NET Core Identity)**
  - Define `ApplicationUser` extending `IdentityUser` with custom fields.
  - Configure `DbContext` to use ASP.NET Core Identity with EF Core.
  - Implement JWT authentication (configure JWT Bearer options, token generation service).
  - Create API controllers/endpoints for user registration (email validation, password hashing via Identity).
  - Implement `/auth/register` endpoint.
  - Implement `/auth/login` endpoint (issue JWTs).
  - Implement `/auth/logout` endpoint (if server-side action needed).
  - Implement `/auth/refresh` endpoint.
  - Configure CORS policy for Blazor WASM frontend origin.
- [ ] **Security Implementation**
  - Set up `AspNetCoreRateLimit` for auth endpoints.
  - Implement password policies via ASP.NET Core Identity options.
  - Configure JWT settings (issuer, audience, key, expiration) via `appsettings.json`.
  - Add basic request logging middleware.

**Frontend Tasks (Blazor WASM):**
- [ ] **Project Setup**
  - Initialize Blazor WebAssembly project (.NET 8).
  - Integrate Blazor Bootstrap (or similar UI library).
  - Configure static analysis (if applicable beyond Roslyn Analyzers used by backend).
  - Set up testing environment (bUnit).
- [ ] **Authentication UI & Logic**
  - Create Login and Registration Razor components with `EditForm` and Data Annotations for validation.
  - Implement client-side JWT storage (e.g., `localStorage` or Blazor's Protected Browser Storage) and refresh logic.
  - Create `AuthenticationStateProvider` implementation for Blazor's auth system.
  - Implement loading states and error handling.
  - Develop services for authentication state and user information, possibly using cascaded parameters or simple DI services for state.
  - Configure `HttpClient` (e.g., using `IHttpClientFactory` and a delegating handler) to attach JWTs to outgoing requests and handle 401s.

**Acceptance Criteria:** (Same as Django stack)

#### Phase 2: Collection Management (.NET Backend, Blazor WASM Client)
**Epic: API Collection CRUD**

**Backend Tasks (.NET Web API):**
- [ ] **Data Models (EF Core Entities)**
  - Define `Collection` entity (user foreign key using `ApplicationUser.Id`, name, description, etc.) with EF Core fluent API or data annotations.
  - Add validation attributes.
  - Implement soft delete (e.g., query filters or status flag).
  - Create EF Core migrations.
  - (Optional) Implement audit logging.
- [ ] **API Endpoints (ASP.NET Core Controllers)**
  - Implement API controller for `/collections` (GET list, POST create).
  - Implement API controller for `/collections/{id}` (GET one, PUT update, DELETE one).
  - Secure endpoints using `[Authorize]` attribute and JWT Bearer scheme.
  - Implement proper authorization (user can only access/modify their own collections, e.g., using policies or resource-based auth).
  - Add filtering (e.g., using LINQ), pagination, and sorting for listing collections.
  - (Bonus) Collection sharing, statistics, duplication features.
- [ ] **Services & DTOs**
  - Create service classes to encapsulate business logic for collections.
  - Use DTOs for request/response payloads, with validation attributes.
  - Map between Entities and DTOs (e.g., using AutoMapper or manual mapping).

**Frontend Tasks (Blazor WASM):**
- [ ] **Collection Management UI**
  - Create Collection list component (displaying cards/table using Blazor Bootstrap components).
  - Implement search and filter UI.
  - Create Collection creation/editing modal/page using `EditForm`.
  - Implement delete confirmation.
- [ ] **State Management & Data Fetching**
  - Use injected services (`HttpClient`-based) to fetch and mutate collection data.
  - Manage UI state within components or through simple DI services.
  - Implement optimistic updates or clear indicators for data changes.
  - Implement error handling for API calls.

**Acceptance Criteria:** (Same as Django stack)

#### Phase 3: Endpoint Builder (.NET Backend, Blazor WASM Client)
**Epic: API Endpoint Configuration**

**Backend Tasks (.NET Web API):**
- [ ] **Data Model (EF Core Entities)**
  - Define `Endpoint` entity (collection foreign key, method, URL, headers, query params, body, timeout, etc.).
  - Use appropriate types for JSON-like data (e.g., `string` mapped to `jsonb` in PostgreSQL, or separate related tables).
  - Create EF Core migrations.
- [ ] **API Implementation (ASP.NET Core Controllers)**
  - Implement nested REST controllers: `/collections/{collectionId}/endpoints`.
  - Logic for variable interpolation (if server-side).
- [ ] **Security & Validation**
  - Implement SSRF protection for URL field in service layer.
  - Validate JSON structures, request size (ASP.NET Core request size limits), timeout settings.

**Frontend Tasks (Blazor WASM):**
- [ ] **Endpoint Builder UI** (Using Blazor Bootstrap components, `EditForm`).
  - Tabbed interface.
  - HTTP method selector.
  - URL input with variable highlighting.
  - Key-value editor for headers/params.
  - JSON body editor (custom component or a simple `textarea` with Blazor validation).
- [ ] **Advanced Features UI** (Variable management, script editors - potentially simpler text areas initially).
- [ ] **UX Enhancements**.

**Acceptance Criteria:** (Same as Django stack)

#### Phase 4: Request Execution Engine (.NET Backend, Blazor WASM Client)
**Epic: HTTP Request Processing**

**Backend Tasks (.NET Web API):**
- [ ] **Request Execution Service**
  - Create service using `HttpClient` (from `IHttpClientFactory`) for outgoing HTTP calls.
  - Implement variable interpolation.
  - Detailed request/response logging to `RequestLog` entity via EF Core.
  - Use `BackgroundService` for deferring the external call if it's long-running and non-blocking for the user's immediate response is critical.
  - Response parsing and storage.
- [ ] **Security & Rate Limiting (`AspNetCoreRateLimit`)**
  - Apply rate limiting to request execution endpoints.
  - Enforce request size limits, timeout for external calls.
- [ ] **Response Processing** (Capture headers, metadata, body, time, size).

**Frontend Tasks (Blazor WASM):**
- [ ] **Request Execution UI** (Blazor components, status updates via component state/DI services).
- [ ] **Response Features** (Format toggle, search, export - simplify if needed).
- [ ] **Execution Management** (Request cancellation via `CancellationToken` with `HttpClient`).

**Acceptance Criteria:** (Same as Django stack, adjusted for `BackgroundService`).

#### Phase 5: Request History & Logging (.NET Backend, Blazor WASM Client)
**Epic: Request Tracking and Analytics**

**Backend Tasks (.NET Web API):**
- [ ] **RequestLog Model Enhancement (EF Core/LINQ)**: Optimize queries (LINQ to Entities), indexing.
- [ ] **Analytics API (ASP.NET Core Controllers)**: Endpoints for stats, filtering, aggregation using LINQ.
- [ ] **Performance Optimization**: Database query optimization, pagination. *Log processing via `BackgroundService` if needed for aggregation.*

**Frontend Tasks (Blazor WASM):** (Largely as original, using injected services for data).
- [ ] **History Dashboard**.
- [ ] **Log Management**.
- [ ] **Data Visualization**.

**Acceptance Criteria:** (Same as Django stack).

#### Phase 6: Advanced Features & Polish (.NET Backend, Blazor WASM Client)
**Epic: Production Ready Features**

**Backend Tasks (.NET Web API):**
- [ ] **WebSocket Implementation (If core to MVP)**
  - Set up ASP.NET Core SignalR for real-time updates.
  - Implement SignalR Hubs for relevant real-time communication.
- [ ] **Production Optimizations**: Comprehensive error handling (exception middleware), health check endpoint (ASP.NET Core Health Checks), metrics (e.g., AppMetrics or OpenTelemetry), security headers.

**Frontend Tasks (Blazor WASM):**
- [ ] **Real-time Features** (SignalR client-side handling).
- [ ] **UI/UX Polish** (Loading indicators, Blazor Bootstrap consistency, accessibility, responsive design).
- [ ] **Performance Features** (Virtualization for large lists, lazy loading assemblies if applicable, PWA considerations).

**Acceptance Criteria:** (Same as Django stack).

---

## 📊 Cross-Stack Comparison Matrix (Revised)

| Feature | Django (🐍) | Node.js (🟨) | Java (☕) | C# (🟣) |
|---------|--------|---------|------|----|
| **Task Processing (Initial)** | Django Background Tasks / Sync | Simple Async / In-Memory Queue | Spring `@Async` | ASP.NET `BackgroundService` |
| **Type Safety (Backend)** | ⭐⭐⭐ (mypy opt-in) | ⭐⭐⭐⭐⭐ (TypeScript) | ⭐⭐⭐⭐⭐ (Java) | ⭐⭐⭐⭐⭐ (C#) |
| **Performance (General)** | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Ecosystem Maturity** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Deployment Complexity (Initial)** | ⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| *Other ratings like Real-time, Community, Enterprise Ready can be kept if desired but consider the simplified scope.* |

---

## 🎯 Portfolio Presentation Strategy (Revised)

### Landing Page Structure
```
api-playground.dev/
├── /                     # Main landing with stack comparison
├── /django              # Django implementation demo
├── /nodejs              # Node.js implementation demo  
├── /java                # Java implementation demo
├── /dotnet              # .NET implementation demo
├── /architecture        # Technical architecture docs (links to docs/architecture/readme.md)
└── /source              # Link to GitHub repository
```

### Key Differentiators to Highlight (Focus on Simplified Implementations)
1. **Identical Feature Set**: Same API contracts across all stacks.
2. **Architectural Patterns**: Different approaches to common problems (e.g., request handling, data access, auth) within the simplified scope.
3. **Code Quality & Best Practices**: Adherence to idiomatic patterns for each language/framework.
4. **Developer Experience**: Ease of setup and development for each stack (Docker-based).

### Technical Blog Posts (Revised Focus)
1. "Building the Same API Playground in 4 Stacks: A Comparative Journey (Initial Implementations)"
2. "Simplified Asynchronous Tasks: A Look Across Django, Node.js, Spring Boot, and .NET Core"
3. "Authentication Deep Dive: JWT Strategies in Python, Node.js, Java, and C#"
4. "Full-Stack TypeScript: Node.js/Express Backend with Next.js Frontend"
5. "Choosing Your Player: Django ORM vs Prisma vs Spring Data JPA vs EF Core for PostgreSQL"

This tracker should now better reflect the decisions made in `docs/Stacks.md` and other documentation.
The next step would be to flesh out the detailed tasks for Node.js, Java, and .NET stacks similar to the Django one.
