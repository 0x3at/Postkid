# API Playground: Multi‑Stack Architecture & Development Plan

This document consolidates system design, architecture, and development planning for each of the five technology stacks described in `tracker.md`, `Stacks.md`, `YellowPaper.md`, and `Models.md`. Timelines follow a one‑stack‑per‑week schedule. Reference `tracker.md` for dependency breakdowns. This guide is exhaustive for implementation, architecture, design, deployment, and maintenance.

---

## Table of Contents

1.  **Common Foundations**
    1.1 Shared API Specification
    1.2 Data Model Reference
    1.3 High‑Level Architecture
    1.4 Visualizations & Sequence Diagrams
2.  **Stack‑Specific Designs**
    2.1 🐍 Stack 1: Python / Django + React
    2.2 🟨 Stack 2: Node.js / Express + Next.js
    2.3 ☕ Stack 3: Java / Spring Boot + Angular
    2.4 🟢 Stack 4: Go / Fiber + SvelteKit
    2.5 🟣 Stack 5: C# / ASP.NET Core + Blazor/React
3.  **Cross‑Stack Comparison**


---

## 1. Common Foundations

### 1.1 Shared API Specification

All five stacks implement identical, versioned REST endpoints under `/api/v1/*`:

| Method | Route | Description |
| --- | --- | --- |
| POST | `/api/v1/auth/register` | User registration |
| POST | `/api/v1/auth/login` | User authentication |
| GET | `/api/v1/collections/` | List all collections |
| POST | `/api/v1/collections/{collection_id}/endpoints/` | Create an endpoint within a collection |
| POST | `/api/v1/test/{endpoint_id}/` | Execute a test call |
| GET | `/api/v1/logs/` | Retrieve execution logs |

**Justification:** Uniform interface simplifies SDKs and CLI tooling, enables safe version upgrades, and guarantees identical behavior across stacks.

---

### 1.2 Data Model Reference

Refer to `Models.md` for full schema definitions. Core entities and relations:

| Entity | Table | Key Fields / Relations |
| --- | --- | --- |
| Users | `users` | `id` (UUID PK), `email`, `password_hash`, timestamps |
| Collections | `collections` | `id` (UUID PK), `owner_id` FK → `users.id`, `name`, `created_at` |
| Endpoints | `endpoints` | `id` (UUID PK), `collection_id` FK → `collections.id`, `schema` (JSONB) |
| Request Logs | `request_logs` | `id` (UUID PK), `endpoint_id` FK → `endpoints.id`, `status`, `timestamp` |
| Teams & Shares | `teams`,`team_members`,`collection_shares` | Collaboration tables linking users and collections |

#### Data Model Class Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

classDiagram
    class User {
      +UUID id
      +String email
      +String password_hash
      +DateTime created_at
    }
    class Collection {
      +UUID id
      +UUID owner_id
      +String name
      +DateTime created_at
    }
    class Endpoint {
      +UUID id
      +UUID collection_id
      +Object schema
    }
    class RequestLog {
      +UUID id
      +UUID endpoint_id
      +String status
      +DateTime timestamp
    }
    User "1" -- "*" Collection : owns
    Collection "1" -- "*" Endpoint : contains
    Endpoint "1" -- "*" RequestLog : logs
````

-----

### 1.3 High‑Level Architecture

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

flowchart TB
    subgraph High_Level_Architecture["<span style='font-size:28px;font-weight:900'>🏗️ HIGH LEVEL ARCHITECTURE</span>"]
        
        subgraph Client_Layer["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💻 Client Layer</span>"]
            direction LR
            WebApp["🌐 Web App"] ~~~ MobileApp["📱 Mobile App"]
        end
        
        subgraph API_Gateway["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🚪 API Gateway</span>"]
            direction LR
            LB["⚖️ Nginx / Load Balancer"] ~~~ SSL["🔒 SSL Termination"]
        end
        
        subgraph Application_Layer["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>⚙️ Application Layer</span>"]
            direction LR
            API["🔌 REST API"] ~~~ WS["🔄 WebSocket Server"] ~~~ MQ["📤 Background Workers"]
        end
        
        subgraph Data_Layer["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💾 Data Layer</span>"]
            direction LR
            DB["🗄️ PostgreSQL / SQL Server"] ~~~ Cache["⚡ Redis"] ~~~ Storage["☁️ S3 / MinIO"]
        end
    end

    %% Connections
    WebApp --> LB
    MobileApp --> LB
    LB --> SSL
    SSL --> API
    API --> DB
    API --> Cache
    API --> MQ
    API --> Storage
    MQ --> Cache
    WS --> API
    WS --> WebApp
    WS --> MobileApp

    %% Node styling
    classDef clientNode fill:#ff9e64,stroke:#ff7043,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef gatewayNode fill:#73daca,stroke:#41a6b5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef appNode fill:#bb9af7,stroke:#9d7cd8,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef dataNode fill:#9ece6a,stroke:#73b25a,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px

    %% Apply styles
    class WebApp,MobileApp clientNode
    class LB,SSL gatewayNode
    class API,WS,MQ appNode
    class DB,Cache,Storage dataNode

    %% Subgraph styling
    style High_Level_Architecture fill:#1a1b26,stroke:#3b4261,stroke-width:3px,color:#c0caf5,font-weight:900
    style Client_Layer fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style API_Gateway fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Application_Layer fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Data_Layer fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
```

-----

### 1.4 Visualizations & Sequence Diagrams

#### 1.4.1 Authentication Handshake

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5'
  }
}}%%

sequenceDiagram
  participant U as User
  participant FE as Frontend
  participant API
  participant AS as AuthService
  participant DB as UserDB

  U->>FE: Submit credentials
  FE->>API: POST /api/v1/auth/login
  API->>AS: Validate credentials
  AS->>DB: Lookup user
  DB-->>AS: User record
  AS-->>API: JWT access & refresh tokens
  API-->>FE: Set tokens in HttpOnly cookie
  FE-->>U: Redirect to dashboard
    
```

#### 1.4.2 Request Execution Flow

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5'
  }
}}%%

sequenceDiagram
  participant UI
  participant API
  participant Q as Queue
  participant W as Worker
  participant DB
  participant External

  UI->>API: POST /api/v1/test/{id}
  API->>Q: enqueue({endpoint_id, payload})
  Q->>W: process job
  W->>External: perform HTTP call
  External-->>W: return response
  W->>DB: write request_log
  W-->>UI: push status via WS
```

#### 1.4.3 Background Task Retry Flow

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

flowchart TB
    subgraph Background_Task_Retry_Flow["<span style='font-size:28px;font-weight:900'>🔄 BACKGROUND TASK RETRY FLOW</span>"]
        A["📨 API enqueues task"]
        B["🤖 Worker picks job"]
        C["📡 Execute HTTP call"]
        D["💾 Store log"]
        E["🔁 Retry handler"]
        F["💀 DeadLetter / Alert"]
    end

    A --> B
    B --> C
    C -->|Success| D
    C -->|Failure| E
    E -->|Retry| B
    E -->|Max retries reached| F

    %% Node styling
    classDef primaryNode fill:#7aa2f7,stroke:#5a7ec5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    class A,B,C,D,E,F primaryNode

    %% Subgraph styling
    style Background_Task_Retry_Flow fill:#1a1b26,stroke:#3b4261,stroke-width:3px,color:#c0caf5,font-weight:900
```

#### 1.4.4 Job Lifecycle State Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5'
  }
}}%%

stateDiagram-v2
  [*] --> Enqueued
  Enqueued --> Processing
  Processing --> Completed
  Processing --> Failed
  Failed --> Retrying
  Retrying --> Enqueued
  Failed --> DeadLetter
  Completed --> [*]
  DeadLetter --> [*]
```

-----

## 2\. Stack‑Specific Designs

### 2.1 🐍 Stack 1: Python / Django + React

#### 2.1.1 System Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

flowchart TB
    subgraph Stack_1_System_Diagram["<span style='font-size:28px;font-weight:900'>🐍 STACK 1 - DJANGO SYSTEM</span>"]
        
        subgraph Client["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💻 Client Layer</span>"]
            direction LR
            React["⚛️ React + Vite"] ~~~ RN["📱 React Native"]
        end
        
        subgraph Gateway["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🚪 Gateway Layer</span>"]
            direction LR
            Nginx["⚖️ Nginx LB"] ~~~ SSL1["🔒 SSL Termination"]
        end
        
        subgraph Backend["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🖥️ Backend Layer</span>"]
            direction LR
            Dj["🐍 Django"] ~~~ DRF["🔌 DRF"] ~~~ Auth["🔑 SimpleJWT"] ~~~ Mw["🛡️ Custom Middleware"]
        end
        
        subgraph Services["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🛠️ Services Layer</span>"]
            direction LR
            ReqSvc["📡 Request Execution"] ~~~ ColSvc["📁 Collection Mgmt"] ~~~ UserSvc["👤 User Mgmt"] ~~~ EnvSvc["🌍 Environment Mgmt"]
        end
        
        subgraph Data["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💾 Data Layer</span>"]
            direction LR
            PG["🐘 PostgreSQL"] ~~~ RD["⚡ Redis Cache"] ~~~ FS["☁️ S3 / Local FS"]
        end
        
        subgraph Workers["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>⚙️ Workers Layer</span>"]
            direction LR
            Celery["🌿 Celery Workers"] ~~~ Beat["🥁 Celery Beat"] ~~~ Queue["📋 Redis Queue"]
        end
        
        subgraph External["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🌐 External Services</span>"]
            direction LR
            SMTP["📧 SMTP"] ~~~ Sentry["🐛 Sentry"] ~~~ APIs["🔌 Third‑party APIs"]
        end
    end

    %% Connections
    React --> Nginx
    RN --> Nginx
    Nginx --> SSL1
    SSL1 --> Dj
    Dj --> DRF
    DRF --> Auth
    DRF --> Mw
    Dj --> ReqSvc & ColSvc & UserSvc & EnvSvc
    ReqSvc --> Celery
    Celery --> Queue
    Celery --> APIs
    Queue --> Beat
    Dj --> PG & RD & FS
    Dj --> SMTP & Sentry

    %% Node styling with thicker fonts
    classDef clientNode fill:#ff9e64,stroke:#ff7043,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef gatewayNode fill:#73daca,stroke:#41a6b5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef backendNode fill:#e0af68,stroke:#d19a66,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef serviceNode fill:#bb9af7,stroke:#9d7cd8,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef databaseNode fill:#9ece6a,stroke:#73b25a,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef workerNode fill:#7aa2f7,stroke:#5a7ec5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef externalNode fill:#f7768e,stroke:#db5a6b,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px

    %% Apply styles
    class React,RN clientNode
    class Nginx,SSL1 gatewayNode
    class Dj,DRF,Auth,Mw backendNode
    class ReqSvc,ColSvc,UserSvc,EnvSvc serviceNode
    class PG,RD,FS databaseNode
    class Celery,Beat,Queue workerNode
    class SMTP,Sentry,APIs externalNode

    %% Subgraph styling with enhanced sections
    style Stack_1_System_Diagram fill:#1a1b26,stroke:#3b4261,stroke-width:3px,color:#c0caf5

```

#### 2.1.2 Component Responsibilities

  * **React/Vite & React Native**: SPA & mobile UI, Zustand, React Query, react-hook-form, zod.
  * **Django & DRF**: REST endpoints, serializers, viewsets, custom middleware (logging, rate-limiting).
  * **SimpleJWT**: Access (15 min) & rotating refresh tokens.
  * **Celery & Redis**: Task queue, exponential retry, scheduled cleanup.
  * **PostgreSQL**: JSONB schemas, relational data.
  * **S3 / MinIO**: File storage.

#### 2.1.3 Dependencies

| Package | Version | Purpose | Justification |
| --- | --- | --- | --- |
| django | 4.x | Web framework | Batteries-included, mature community |
| djangorestframework | 3.x | REST toolkit | Serializers, ViewSets |
| djangorestframework-simplejwt | latest | JWT auth | Blacklist, rotation |
| celery | 5.x | Async task queue | Scheduling, retries |
| redis | 4.x | Broker & cache | In-memory store |
| psycopg2-binary | latest | PostgreSQL adapter | Stable, performant |
| boto3 | latest | AWS S3 SDK | Official SDK, MinIO-compatible |
| react | 18.x | Frontend UI | Hooks, concurrency |
| vite | latest | Dev server & bundler | Fast HMR |
| zustand | latest | State management | Minimal API |
| react-query | latest | Data fetching | Caching, invalidation |
| react-router-dom | latest | Routing | Standard SPA nav |
| react-hook-form | latest | Form handling | Performance |
| zod | latest | Schema validation | TS-first |
| axios | latest | HTTP client | Interceptors |
| eslint | latest | Linting | Code quality |
| prettier | latest | Formatting | Consistency |
| jest | latest | Unit testing | React ecosystem |
| @testing-library/react | latest | Component testing | Accessibility focus |

#### 2.1.4 DDD Folder Structure

```text
api_playground/
├── apps/
│   ├── authentication/
│   ├── collections/
│   ├── environments/
│   ├── history/
│   ├── proxy/
│   └── codegen/
├── core/
│   ├── models/
│   ├── serializers/
│   ├── permissions/
│   ├── middleware/
│   └── utils/
├── config/
│   ├── settings/
│   ├── urls.py
│   └── celery.py
└── static/
```

#### 2.1.5 ER Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5'
  }
}}%%

erDiagram
    users ||--o{ collections : owns
    collections ||--o{ endpoints : contains
    endpoints ||--o{ request_logs : logs
```

#### 2.1.6 Service & Background Task Flows

  * **execute\_api\_request**: Celery queues HTTP call; on failure, retry with backoff.
  * **cleanup\_tasks**: Beat schedules retention purge.
  * **notifications**: SMTP email on share events.

#### 2.1.7 Auth & Security Patterns

  * **JWT**: 15 min access, 7 day refresh.
  * **Rate Limiting**: Redis-based per-IP/user.
  * **CSRF**: Exempt API, enforced on forms.
  * **SSRF Prevention**: Block private net ranges.
  * **Audit Logging**: All auth and exec events.

#### 2.1.8 API Design Conventions

  * Envelope: `{ status, data, message, timestamp }`.
  * Versioned `/api/v1/`.
  * Pagination: PageNumberPagination (20).

#### 2.1.9 Frontend Architecture

  * `components/`, `hooks/`, `services/`, `stores/` (Zustand), `pages/` (Vite).

#### 2.1.10 Deployment Topology

  * **Dev:** Docker Compose (Django, React, Redis, Postgres)
  * **Prod:** AWS EKS/ECS, ALB → Gunicorn, React CDN; RDS, ElastiCache, S3.

#### 2.1.11 Performance & Caching

  * Redis/LRU, DB indices.
  * Celery concurrency tuning.

#### 2.1.12 Testing Strategy

  * **Unit:** pytest‑django (80% coverage)
  * **Integration:** DRF `APITestCase`
  * **E2E:** Playwright
  * **Load:** Locust (1k users)

-----

### 2.2 🟨 Stack 2: Node.js / Express + Next.js

#### 2.2.1 System Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

flowchart TB
    subgraph Stack_2_System_Diagram["<span style='font-size:28px;font-weight:900'>🟢 STACK 2 - NODE.JS SYSTEM</span>"]
        
        subgraph Client_Layer2["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💻 Client Layer</span>"]
            NextJS["⚡ Next.js (SSR/CSR)"]
        end
        
        subgraph Gateway_Layer2["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🚪 Gateway Layer</span>"]
            Vercel["▲ Vercel Edge / Nginx"]
        end
        
        subgraph Backend_Layer2["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🖥️ Backend Layer</span>"]
            direction LR
            Express["🚀 Express.js API"] ~~~ NextAuth["🔐 NextAuth.js / Passport"] ~~~ Prisma["💎 Prisma ORM"]
        end
        
        subgraph Worker_Layer2["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>⚙️ Worker Layer</span>"]
            direction LR
            Bull["🐂 Bull Workers"] ~~~ RetryFlow2["🔁 Retry on Error"]
        end
        
        subgraph Data_Layer2["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💾 Data Layer</span>"]
            direction LR
            Postgres["🐘 PostgreSQL"] ~~~ Redis2["⚡ Redis Queue & Cache"]
        end
        
        subgraph External_APIs2["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🌐 External APIs</span>"]
            APIs2["🔌 Third‑party APIs"]
        end
    end

    %% Connections
    NextJS --> Vercel
    Vercel --> Express
    Express --> NextAuth
    Express --> Prisma
    Express --> Bull
    Bull --> Redis2
    Bull --> RetryFlow2
    RetryFlow2 --> Bull
    Prisma --> Postgres
    Express --> Redis2
    Bull --> APIs2

    %% Node styling
    classDef clientNode fill:#ff9e64,stroke:#ff7043,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef gatewayNode fill:#73daca,stroke:#41a6b5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef backendNode fill:#e0af68,stroke:#d19a66,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef workerNode fill:#7aa2f7,stroke:#5a7ec5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef databaseNode fill:#9ece6a,stroke:#73b25a,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef externalNode fill:#f7768e,stroke:#db5a6b,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px

    %% Apply styles
    class NextJS clientNode
    class Vercel gatewayNode
    class Express,NextAuth,Prisma backendNode
    class Bull,RetryFlow2 workerNode
    class Postgres,Redis2 databaseNode
    class APIs2 externalNode

    %% Subgraph styling
    style Stack_2_System_Diagram fill:#1a1b26,stroke:#3b4261,stroke-width:3px,color:#c0caf5,font-weight:900
    style Client_Layer2 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Gateway_Layer2 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Backend_Layer2 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Worker_Layer2 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Data_Layer2 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style External_APIs2 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
```

#### 2.2.2 Component Responsibilities

  * **Next.js 14+**: SSR/CSR, API routes (`/api/*`), React Query/SWR, Tailwind.
  * **Express.js**: Request exec microservice, logging, proxy.
  * **NextAuth.js + Passport + JWT**: Auth strategies (OAuth, JWT).
  * **Prisma ORM**: Type-safe DB client, migrations.
  * **Bull**: Redis-backed queue, repeatable jobs, backoff retry.
  * **PostgreSQL**: Relational store.
  * **Redis**: Cache, rate limiter, queue broker.

#### 2.2.3 Dependencies

| Package | Version | Purpose | Justification |
| --- | --- | --- | --- |
| next | 14.x | React framework | Hybrid SSR/CSR |
| react | 18.x | UI library | Concurrent features |
| next-auth | latest | Auth in Next.js | Simplifies session, OAuth |
| express | 4.18.x | Backend API | Middleware ecosystem |
| passport | 0.6.x | Auth middleware | Strategy plug-ins |
| jsonwebtoken | latest | JWT handling | Lightweight |
| prisma | 5.x | ORM & migrations | TS-first, schema-driven |
| @prisma/client | 5.x | Runtime DB client | Generated |
| bull | 4.x | Job queue | Retry, concurrency |
| pg | latest | Postgres driver | Standard adapter |
| redis | latest | Redis client | High performance |
| express-rate-limit | latest | Rate limiting | Throttling |
| zod | latest | Input validation | TS-first |
| react-hook-form | latest | Form handling | Performance |
| axios | latest | HTTP client | Interceptors |
| eslint | latest | Linting | Code quality |
| prettier | latest | Formatting | Consistency |
| jest | latest | Unit testing | Ecosystem |
| supertest | latest | HTTP integration tests | Express-friendly |
| @testing-library/react | latest | Component testing | Accessibility focus |

#### 2.2.4 DDD Folder Structure

```text
node-playground/
├── src/
│   ├── pages/           # Next.js pages & API routes
│   ├── components/
│   ├── lib/             # Prisma, auth utils
│   ├── services/        # Business logic
│   └── jobs/            # Bull processors
├── prisma/
│   └── schema.prisma    # DB models & migrations
└── public/              # Static assets
```

#### 2.2.5 ER Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5'
  }
}}%%

erDiagram
    users ||--o{ collections : owns
    collections ||--o{ endpoints : contains
    endpoints ||--o{ request_logs : logs
```

#### 2.2.6 Service & Background Task Flows

  * **executeJob**: API → Bull enqueue → Worker HTTP exec → DB log → WS notify.
  * **batchCleanup**: Bull repeatable job cleans logs & temp data.

#### 2.2.7 Auth & Security Patterns

  * **JWT & OAuth**: NextAuth for frontend sessions, JWT bearer for API.
  * **CSRF**: NextAuth double-submit; API stateless.
  * **Rate Limiting**: express‐rate‐limit + Redis store.
  * **SSRF Prevention**: URL whitelist / host validation.
  * **Audit Logging**: Log login, endpoint exec, share events.

#### 2.2.8 API Design Conventions

  * Envelope: `{ status, data, error, timestamp }`.
  * Versioned `/api/v1/`.
  * Swagger via JSDoc.

#### 2.2.9 Frontend Architecture

  * **App Router**: Modular pages & server components.
  * **React Query / SWR**: Data layer.
  * **Zustand**: Client state.
  * **Tailwind CSS**.

#### 2.2.10 Deployment Topology

  * **Frontend:** Vercel.
  * **Backend:** AWS ECS Fargate + ALB.
  * **DB:** RDS Postgres.
  * **Cache/Queue:** ElastiCache Redis.

#### 2.2.11 Performance & Caching

  * ISR, SSG for pages.
  * Redis cache for frequent queries.
  * PG pooling.

#### 2.2.12 Testing Strategy

  * **Unit:** Jest.
  * **Integration:** Supertest.
  * **E2E:** Playwright.

-----

### 2.3 ☕ Stack 3: Java / Spring Boot + Angular

#### 2.3.1 System Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

flowchart TB
    subgraph Stack_3_System_Diagram["<span style='font-size:28px;font-weight:900'>☕ STACK 3 - SPRING BOOT SYSTEM</span>"]
        
        subgraph Client_Layer3["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💻 Client Layer</span>"]
            Angular["🅰️ Angular 17"]
        end
        
        subgraph Gateway_Layer3["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🚪 Gateway Layer</span>"]
            direction LR
            Nginx3["⚖️ Nginx / Ingress"] ~~~ SSL3["🔒 SSL Termination"]
        end
        
        subgraph Backend_Layer3["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🖥️ Backend Layer</span>"]
            direction LR
            SpringBoot["🍃 Spring Boot App"] ~~~ Security["🛡️ Spring Security / JWT"]
            MVC["🏗️ Spring MVC"] ~~~ JPA["💾 Spring Data JPA"]
        end
        
        subgraph Worker_Layer3["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>⚙️ Worker Layer</span>"]
            direction LR
            AMQP["📨 AMQP Listener"] ~~~ RetryFlow3["🔁 @Retryable"]
        end
        
        subgraph Data_Layer3["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💾 Data Layer</span>"]
            direction LR
            Postgres3["🐘 PostgreSQL"] ~~~ RabbitMQ["🐰 RabbitMQ"] ~~~ Redis3["⚡ Redis Cache"]
        end
        
        subgraph External_APIs3["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🌐 External APIs</span>"]
            APIs3["🔌 Third‑party APIs"]
        end
    end

    %% Connections
    Angular --> Nginx3
    Nginx3 --> SSL3
    SSL3 --> SpringBoot
    SpringBoot --> Security & MVC & JPA
    SpringBoot -->|enqueue job| RabbitMQ
    RabbitMQ -->|message| AMQP
    AMQP --> APIs3
    AMQP --> RetryFlow3
    RetryFlow3 --> AMQP
    SpringBoot --> Postgres3
    SpringBoot --> Redis3

    %% Node styling
    classDef clientNode fill:#ff9e64,stroke:#ff7043,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef gatewayNode fill:#73daca,stroke:#41a6b5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef backendNode fill:#e0af68,stroke:#d19a66,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef workerNode fill:#7aa2f7,stroke:#5a7ec5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef databaseNode fill:#9ece6a,stroke:#73b25a,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef externalNode fill:#f7768e,stroke:#db5a6b,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px

    %% Apply styles
    class Angular clientNode
    class Nginx3,SSL3 gatewayNode
    class SpringBoot,Security,MVC,JPA backendNode
class AMQP,RetryFlow3 workerNode
class Postgres3,RabbitMQ,Redis3 databaseNode
class APIs3 externalNode

%% Subgraph styling
style Stack_3_System_Diagram fill:#1a1b26,stroke:#3b4261,stroke-width:3px,color:#c0caf5,font-weight:900
style Client_Layer3 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
style Gateway_Layer3 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
style Backend_Layer3 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
style Worker_Layer3 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
style Data_Layer3 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
style External_APIs3 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
```

#### 2.3.2 Component Responsibilities

  * **Angular 17 + Material**: SPA UI, NgRx for state, Angular Router, Reactive Forms.
  * **Spring Boot 3.2**: Embedded Tomcat, layered architecture (Controller → Service → Repository).
  * **Spring Security + JWT**: Authentication & authorization filters, CSRF settings.
  * **Spring MVC & Data JPA**: REST controllers, Hibernate ORM, Flyway migrations.
  * **RabbitMQ & Asynchronous Workers**: Message broker for request execution and retry.
  * **Redis**: Cache responses, support rate limiting.

#### 2.3.3 Dependencies

| Artifact | Version | Purpose | Justification |
| --- | --- | --- | --- |
| spring-boot-starter-web | 3.2.x | REST API framework | Convention-over-configuration |
| spring-boot-starter-security | 3.2.x | Security module | Comprehensive security filters |
| spring-boot-starter-data-jpa | 3.2.x | JPA & Hibernate support | Repository pattern |
| google-auth-library-oauth2-http | latest | OAuth support | Google strategy for third-party login |
| flyway-core | latest | Database migrations | Versioned, repeatable migrations |
| spring-boot-starter-amqp | latest | RabbitMQ integration | Auto-configured listener containers |
| postgresql | latest | JDBC driver | Official adapter |
| spring-boot-starter-test | latest | Testing (JUnit, MockMvc) | Spring TestContext, Mockito |
| angular/core | 17.x | Frontend framework | CLI, AoT compilation |
| @ngrx/store | latest | State management | Redux-like container |
| @angular/material | latest | UI component library | Material Design compliance |
| @angular/router | 17.x | Client routing | Standard Angular navigation |
| @angular/forms | 17.x | Reactive & template forms | Robust form handling |

#### 2.3.4 DDD Folder Structure

```text
spring-playground/
├── src/
│   ├── main/
│   │   ├── java/com/example/
│   │   │   ├── config/        # Security, DB, MQ configs
│   │   │   ├── controller/    # REST controllers
│   │   │   ├── domain/        # Entities
│   │   │   ├── repository/    # JPA repositories
│   │   │   ├── service/       # Business logic
│   │   │   └── websocket/     # STOMP handlers
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/  # Flyway scripts
│   └── test/                  # Unit & integration tests
```

#### 2.3.5 ER Diagram

```mermaid
erDiagram
    users ||--o{ collections : owns
    collections ||--o{ endpoints : contains
    endpoints ||--o{ request_logs : logs
```

#### 2.3.6 Service & Background Task Flows

  * **RequestExecutionService**: Uses `WebClient`; decorated with `@Retryable` for transient errors.
  * **ScheduledTasks**: `@Scheduled` methods for cleanup and audit notifications.
  * **STOMP WebSocket**: Real-time updates to Angular clients.

#### 2.3.7 Auth & Security Patterns

  * **JWT Tokens**: Custom filter and provider (`JwtAuthenticationFilter`).
  * **CSRF**: Disabled for API; Angular double-submit-cookie pattern.
  * **SSRF Prevention**: Host whitelist in `WebClient` configuration
  * **Audit Logging**: AOP-based logging of user actions (timestamp, user, IP)
  * **Rate Limiting**: Bucket4j per-user/IP.
  * **CORS**: Configured via `application.yml` whitelist.

#### 2.3.8 API Design Conventions

  * **DTOs**: MapStruct for mapping.
  * **Error Handling**: `@ControllerAdvice` with `@ExceptionHandler`.
  * **OpenAPI**: SpringDoc auto-generated docs.

#### 2.3.9 Frontend Architecture

  * **Modules**: Auth, Collections, Endpoints, History.
  * **Shared**: Components, interceptors.
  * **Core**: Services, global state.

#### 2.3.10 Deployment Topology

  * **Docker** multi-stage for API & Angular.
  * **Kubernetes**: Ingress, ConfigMaps, Secrets.
  * **Monitoring**: Prometheus, Grafana.

#### 2.3.11 Performance & Caching

  * **Hibernate 2nd-level** cache via Redis.
  * **Read replicas** for heavy read.
  * **RabbitMQ** consumer concurrency tuning.

#### 2.3.12 Testing Strategy

  * **Unit**: JUnit 5, Mockito.
  * **Integration**: TestContainers (Postgres, RabbitMQ).
  * **E2E**: Cypress.

-----

### 2.4 🟢 Stack 4: Go / Fiber + SvelteKit

#### 2.4.1 System Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

flowchart TB
    subgraph Stack_4_System_Diagram["<span style='font-size:28px;font-weight:900'>🐹 STACK 4 - GO SYSTEM</span>"]
        
        subgraph Client_Layer4["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💻 Client Layer</span>"]
            SvelteKit["🧡 SvelteKit"]
        end
        
        subgraph Gateway_Layer4["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🚪 Gateway Layer</span>"]
            direction LR
            Nginx4["⚖️ Nginx / Ingress"] ~~~ SSL4["🔒 SSL Termination"]
        end
        
        subgraph Backend_Layer4["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🖥️ Backend Layer</span>"]
            direction LR
            Fiber["⚡ Fiber API"] ~~~ JWT4["🔑 golang-jwt"]
            Validator["✅ validator"] ~~~ PGX["🐘 pgx driver"]
        end
        
        subgraph Worker_Layer4["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>⚙️ Worker Layer</span>"]
            direction LR
            Asynq["🔄 Asynq Workers"] ~~~ RetryFlow4["🔁 Configured Retry"]
        end
        
        subgraph Data_Layer4["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💾 Data Layer</span>"]
            direction LR
            Postgres4["🐘 PostgreSQL"] ~~~ Redis4["⚡ Redis (Asynq + Cache)"]
        end
        
        subgraph External_APIs4["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🌐 External APIs</span>"]
            APIs4["🔌 Third‑party APIs"]
        end
    end

    %% Connections
    SvelteKit --> Nginx4
    Nginx4 --> SSL4
    SSL4 --> Fiber
    Fiber --> JWT4 & Validator & PGX
    Fiber -->|enqueue| Asynq
    Asynq --> Redis4
    Asynq --> RetryFlow4
    RetryFlow4 --> Asynq
    Asynq --> APIs4
    PGX --> Postgres4
    Fiber --> Redis4

    %% Node styling
    classDef clientNode fill:#ff9e64,stroke:#ff7043,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef gatewayNode fill:#73daca,stroke:#41a6b5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef backendNode fill:#e0af68,stroke:#d19a66,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef workerNode fill:#7aa2f7,stroke:#5a7ec5,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef databaseNode fill:#9ece6a,stroke:#73b25a,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef externalNode fill:#f7768e,stroke:#db5a6b,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px

    %% Apply styles
    class SvelteKit clientNode
    class Nginx4,SSL4 gatewayNode
    class Fiber,JWT4,Validator,PGX backendNode
    class Asynq,RetryFlow4 workerNode
    class Postgres4,Redis4 databaseNode
    class APIs4 externalNode

    %% Subgraph styling
    style Stack_4_System_Diagram fill:#1a1b26,stroke:#3b4261,stroke-width:3px,color:#c0caf5,font-weight:900
    style Client_Layer4 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Gateway_Layer4 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Backend_Layer4 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Worker_Layer4 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style Data_Layer4 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
    style External_APIs4 fill:#24283b,stroke:#3b4261,stroke-width:2px,color:#c0caf5,font-weight:900
```

#### 2.4.2 Component Responsibilities

  * **SvelteKit**: SSR & client routing, Skeleton UI, Tailwind.
  * **Fiber v2.x**: Fast HTTP server, middleware for auth, logging, rate limit.
  * **golang-jwt**: JWT creation & validation.
  * **validator**: Struct-based request validation.
  * **Asynq**: Redis-backed task queue with retry and scheduling.

#### 2.4.3 Dependencies

| Module | Version | Purpose | Justification |
| --- | --- | --- | --- |
| github.com/gofiber/fiber/v2 | v2.x | HTTP framework | Express-like API, performance |
| github.com/golang-jwt/jwt/v4 | v4.x | JWT handling | RFC-compliant |
| github.com/go-playground/validator | latest | Request validation | Tag-based |
| github.com/hibiken/asynq | latest | Task queue | Reliable retries |
| github.com/jackc/pgx/v4 | v4.x | Postgres driver | Native Go, performant |
| github.com/gofiber/template | latest | HTML templating (optional) | Server-side render |
| github.com/sveltejs/kit | latest | Frontend framework | Hybrid SSR/CSR |
| tailwindcss | latest | Utility-first CSS | Rapid UI |

#### 2.4.4 DDD Folder Structure

```text
go-playground/
├── cmd/           # entry points
├── internal/
│   ├── api/       # Fiber handlers
│   ├── service/   # Business logic
│   ├── model/     # DB models & migrations
│   ├── worker/    # Asynq jobs
│   └── config/    # Config loader
└── web/           # SvelteKit app
```

#### 2.4.5 ER Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5'
  }
}}%%

erDiagram
    users ||--o{ collections : owns
    collections ||--o{ endpoints : contains
    endpoints ||--o{ request_logs : logs
```

#### 2.4.6 Service & Background Task Flows

  * **ExecuteTask**: Asynq worker calls external API, logs to DB; retries per policy.
  * **CleanupTask**: Scheduled retention enforcement.

#### 2.4.7 Auth & Security Patterns

  * **JWT**: Fiber JWT middleware.
  * **Rate Limiting**: Fiber limiter + Redis.
  * **CORS**: Configured via middleware.
  * **Audit Logging**: Middleware logs critical events.

#### 2.4.8 API Design Conventions

  * Envelope: `{ status, data, error }`.
  * Versioned `/api/v1/`.
  * Input binding to structs.

#### 2.4.9 Frontend Architecture

  * **Routes**: File-based.
  * **Stores**: Svelte writable stores.
  * **UI**: Skeleton & Tailwind.

#### 2.4.10 Deployment Topology

  * **Docker** scratch images.
  * **Kubernetes** with HPA & metrics.
  * **Ingress** for TLS.

#### 2.4.11 Performance & Caching

  * **pgx** pooling.
  * **LRU** in-memory + Redis.
  * **Goroutine** pool management.

#### 2.4.12 Testing Strategy

  * **Unit**: Testify, GoMock.
  * **Integration**: Dockerized Postgres CI.
  * **E2E**: Playwright.

-----

### 2.5 🟣 Stack 5: C\# / ASP.NET Core + Blazor/React

#### 2.5.1 System Diagram

```mermaid
%%{init: {
  'theme': 'base',
  'themeVariables': {
    'primaryColor': '#7aa2f7',
    'primaryTextColor': '#1a1b26',
    'primaryBorderColor': '#3b4261',
    'lineColor': '#bb9af7',
    'background': '#1a1b26',
    'mainBkg': '#1a1b26',
    'secondBkg': '#24283b',
    'tertiaryBkg': '#292e42',
    'fontFamily': '"Segoe UI Black", "Arial Black", sans-serif',
    'fontSize': '13px',
    'fontWeight': '900',
    'labelBackground': '#24283b',
    'edgeLabelBackground': '#24283b',
    'clusterBkg': '#24283b',
    'clusterBorder': '#3b4261',
    'defaultLinkColor': '#bb9af7',
    'titleColor': '#c0caf5',
    'sectionFontSize': '24px',
    'sectionFontWeight': '900'
  },
  'elk': {
    'mergeEdges': true,
    'nodePlacementStrategy': 'LINEAR_SEGMENTS',
    'edgeRouting': 'ORTHOGONAL',
    'direction': 'DOWN',
    'spacingFactor': 0.4,
    'nodeSpacing': 25,
    'rankSpacing': 50,
    'compactMode': true,
    'diagramPadding': 4,
    'padding': 8,
    'useMaxWidth': false,
    'wrappingWidth': 75,
    'layeringStrategy': 'LONGEST_PATH',
    'curve': 'cardinal',
    'rankDir': 'TB'
  }
}}%%
%%{init: {"layout":"elk"}}%%

flowchart TB
    subgraph Stack_5_System_Diagram["<span style='font-size:28px;font-weight:900'>💙 STACK 5 - .NET SYSTEM</span>"]
        
        subgraph Client_Layer5["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💻 Client Layer</span>"]
            direction LR
            Blazor["🔥 Blazor WASM"] ~~~ React5["⚛️ React"]
        end
        
        subgraph Gateway_Layer5["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🚪 Gateway Layer</span>"]
            direction LR
            IIS["🏢 IIS / Azure Gateway"] ~~~ SSL5["🔒 SSL Termination"]
        end
        
        subgraph Backend_Layer5["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🖥️ Backend Layer</span>"]
            direction LR
            ASPNET["🌐 ASP.NET Core API"] ~~~ Identity["👤 ASP.NET Identity"]
            EFCore["💎 Entity Framework Core"] ~~~ MediatR["📡 MediatR"]
        end
        
        subgraph Worker_Layer5["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>⚙️ Worker Layer</span>"]
            direction LR
            Hangfire["🔥 Hangfire Jobs"] ~~~ RetryFlow5["🔁 Automatic Retry"]
        end
        
        subgraph Data_Layer5["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>💾 Data Layer</span>"]
            direction LR
            SQLServer["🗄️ SQL Server / PostgreSQL"] ~~~ Redis5["⚡ Redis Cache"]
        end
        
        subgraph External_APIs5["<span style='font-size:22px;font-weight:900;text-transform:uppercase'>🌐 External APIs</span>"]
            APIs5["🔌 Third‑party APIs"]
        end
    end

    %% Connections
    Blazor --> IIS
    React5 --> IIS
    IIS --> SSL5
    SSL5 --> ASPNET
    ASPNET --> Identity & EFCore & MediatR
    ASPNET -->|Enqueue| Hangfire
    Hangfire --> SQLServer
    Hangfire --> RetryFlow5
    RetryFlow5 --> Hangfire
    Hangfire --> APIs5
    EFCore --> SQLServer
    ASPNET --> Redis5

    %% Node styling
    classDef clientNode fill:#ff9e64,stroke:#ff7043,stroke-width:3px,color:#1a1b26,font-weight:800,font-size:13px
    classDef gatewayNode fill:#73
```

#### 2.5.2 Component Responsibilities

  * **Blazor WASM & React + Material-UI**: Dual SPA frontends.
  * **ASP.NET Core 8**: Middleware pipeline, controllers, DI.
  * **Entity Framework Core**: DbContext, migrations.
  * **ASP.NET Identity & JWT**: Identity users, JWT bearer.
  * **Hangfire**: Background jobs, retries, dashboard.
  * **SQL Server / PostgreSQL & Redis**: Data store + cache.

#### 2.5.3 Dependencies

| Package | Version | Purpose | Justification |
| --- | --- | --- | --- |
| Microsoft.AspNetCore.App | 8.x | Core ASP.NET libraries | Bundled |
| Microsoft.EntityFrameworkCore | 8.x | ORM framework | LINQ-based, tooling |
| AspNetCore.Identity.EntityFramework | latest | Identity store | Secure defaults |
| Hangfire.AspNetCore | latest | Background job processing | Dashboard, retry |
| Swashbuckle.AspNetCore | latest | OpenAPI/Swagger | Automatic docs |
| Microsoft.Data.SqlClient | latest | SQL Server driver | Official Microsoft |
| Npgsql.EntityFrameworkCore | latest | PostgreSQL provider | Alternative DB |
| StackExchange.Redis | latest | Redis client | Performance |
| Blazor.WebAssembly | latest | Blazor client framework | WebAssembly |
| React | 18.x | Alternative SPA | Material-UI |
| MediatR | latest | In-process messaging | CQRS patterns |

#### 2.5.4 DDD Folder Structure

```text
dotnet-playground/
├── src/
│   ├── Api/            # Controllers, endpoints
│   ├── Application/    # MediatR handlers
│   ├── Domain/         # Entities, value objects
│   ├── Infrastructure/ # EF, Identity, Hangfire
│   └── WebUI/          # Blazor & React projects
└── tests/              # Unit & integration tests
```

#### 2.5.5 ER Diagram

```mermaid
erDiagram
    users ||--o{ collections : owns
    collections ||--o{ endpoints : contains
    endpoints ||--o{ request_logs : logs
```

#### 2.5.6 Service & Background Task Flows

  * **RequestExecutor**: `HttpClientFactory` calls; Hangfire retries exceptions.
  * **RecurringJobs**: Cleanup & notifications via Hangfire scheduler.

#### 2.5.7 Auth & Security Patterns

  * **JWT Bearer**: Middleware.
  * **Role Policies**: `[Authorize]` attributes.
  * **Rate Limiting**: AspNetCoreRateLimit.
  * **Audit Logging**: Middleware logs key actions.

#### 2.5.8 API Design Conventions

  * RFC 7807 Problem Details.
  * URL & header versioning.
  * Data Annotations & FluentValidation.

#### 2.5.9 Frontend Architecture

  * Blazor components with DI.
  * React with Redux Toolkit & RTK Query.

#### 2.5.10 Deployment Topology

  * **Azure App Service**: WebUI
  * **AKS**: API & workers
  * **Azure SQL / AWS RDS**
  * **Azure Cache for Redis**

#### 2.5.11 Performance & Caching

  * In-memory + Redis cache
  * EF compiled queries
  * Health checks & metrics

#### 2.5.12 Testing Strategy

  * **Unit**: xUnit & Moq
  * **Integration**: TestHost & EF In-Memory
  * **E2E**: Playwright

-----

## 3\. Cross‑Stack Comparison

| Aspect | Django (🐍) | Node.js (🟨) | Java (☕) | Go (🟢) | .NET (🟣) |
| --- | --- | --- | --- | --- | --- |
| Retry Mechanism | Celery | Bull | Spring Retry | Asynq | Hangfire |
| Auth Flow | Sequence 1.4.1 applies across | | | | |
| Background Flow | Flowchart 1.4.3 applies generically | | | | |
| Type Safety | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Performance | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Real‑time Support | Channels | WebSockets | STOMP | WebSockets | SignalR |
| Jobs Broker | Celery | Bull | RabbitMQ | Asynq | Hangfire |
| Deployment Complexity | ⭐⭐ | ⭐⭐⭐ | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
