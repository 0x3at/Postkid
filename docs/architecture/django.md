# Stack 1: Python/Django + React - Complete Architecture & System Design

## 🏗️ System Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                FRONTEND LAYER                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│  React App (TypeScript + Vite)                                                │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐          │
│  │   Auth UI    │ │ Request Builder│ │ Collections  │ │ Environments │          │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐          │
│  │ Response View│ │ Code Generator │ │   History    │ │ Settings     │          │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘          │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
                                   HTTPS/WSS
                                       │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              LOAD BALANCER                                      │
│                               (Nginx/HAProxy)                                   │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              BACKEND LAYER                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│  Django Application (DRF + Gunicorn)                                           │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐          │
│  │   Auth API   │ │ Collections  │ │ Request Proxy│ │ Code Gen API │          │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘          │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐          │
│  │ Environment  │ │   History    │ │ WebSocket    │ │ Admin Panel  │          │
│  │   Manager    │ │   Tracker    │ │   Handler    │ │              │          │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘          │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            ASYNC PROCESSING                                     │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                            │
│  │    Celery    │ │    Redis     │ │  Beat        │                            │
│  │   Workers    │ │   Message    │ │  Scheduler   │                            │
│  │              │ │   Broker     │ │              │                            │
│  └──────────────┘ └──────────────┘ └──────────────┘                            │
└─────────────────────────────────────────────────────────────────────────────────┘
                                       │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              DATA LAYER                                         │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐                            │
│  │ PostgreSQL   │ │    Redis     │ │  File        │                            │
│  │   Primary    │ │    Cache     │ │  Storage     │                            │
│  │   Database   │ │   & Sessions │ │  (Local/S3)  │                            │
│  └──────────────┘ └──────────────┘ └──────────────┘                            │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## 🔧 Backend Architecture (Django + DRF)

### Application Structure
```
api_playground/
├── config/                 # Django settings
│   ├── settings/
│   │   ├── base.py
│   │   ├── development.py
│   │   ├── production.py
│   │   └── testing.py
│   ├── urls.py
│   └── wsgi.py
├── apps/
│   ├── authentication/     # User management
│   ├── workspaces/        # Workspace management
│   ├── collections/       # Request collections
│   ├── requests/          # API request handling
│   ├── environments/      # Environment variables
│   ├── history/           # Request history
│   ├── proxy/             # External API proxy
│   └── codegen/           # Code generation
├── core/                  # Shared utilities
│   ├── permissions.py
│   ├── middleware.py
│   ├── serializers.py
│   └── utils.py
├── tasks/                 # Celery tasks
└── tests/
```

### Django Apps Architecture

#### 1. Authentication App
**Models:**
- `User` (extends AbstractUser)
- `UserProfile`
- `APIKey`
- `RefreshToken`

**Key Features:**
- JWT authentication with refresh tokens
- API key management
- User profile management
- Social authentication ready

#### 2. Workspaces App
**Models:**
- `Workspace`
- `WorkspaceMember`
- `WorkspaceInvitation`

**Relationships:**
- User → Workspace (many-to-many through WorkspaceMember)
- Workspace → Collections (one-to-many)

#### 3. Collections App
**Models:**
- `Collection`
- `Request`
- `RequestSnapshot`

**Hierarchy:**
```
Workspace
├── Collection (root)
│   ├── Collection (nested)
│   └── Request
└── Collection
    └── Request
```

#### 4. Proxy App
**Components:**
- Request executor service
- Response processor
- Rate limiting middleware
- Security filters

### Database Design

#### Core Models Structure
```python
# Key model relationships
class Workspace(models.Model):
    name = models.CharField(max_length=255)
    owner = models.ForeignKey(User, on_delete=models.CASCADE)
    members = models.ManyToManyField(User, through='WorkspaceMember')
    
class Collection(models.Model):
    workspace = models.ForeignKey(Workspace, on_delete=models.CASCADE)
    parent = models.ForeignKey('self', null=True, on_delete=models.CASCADE)
    name = models.CharField(max_length=255)
    
class Request(models.Model):
    collection = models.ForeignKey(Collection, on_delete=models.CASCADE)
    method = models.CharField(max_length=10)
    url = models.URLField()
    headers = models.JSONField(default=dict)
    body = models.TextField(blank=True)
    auth_config = models.JSONField(default=dict)
```

### API Design Patterns

#### 1. ViewSet Structure
```python
class RequestViewSet(viewsets.ModelViewSet):
    serializer_class = RequestSerializer
    permission_classes = [IsAuthenticated, WorkspacePermission]
    
    @action(detail=True, methods=['post'])
    def execute(self, request, pk=None):
        # Async execution via Celery
        pass
    
    @action(detail=True, methods=['post'])
    def duplicate(self, request, pk=None):
        # Request duplication logic
        pass
```

#### 2. Serializer Patterns
```python
class RequestSerializer(serializers.ModelSerializer):
    class Meta:
        model = Request
        fields = '__all__'
    
    def validate_url(self, value):
        # URL validation logic
        return value
    
    def validate_headers(self, value):
        # Header validation logic
        return value
```

### Async Processing Architecture

#### Celery Task Structure
```python
# Request execution task
@shared_task(bind=True)
def execute_request_task(self, request_data, user_id):
    try:
        # Execute HTTP request
        # Store response in database
        # Send WebSocket notification
        pass
    except Exception as exc:
        # Error handling and retry logic
        raise self.retry(exc=exc, countdown=60, max_retries=3)
```

#### Task Categories
1. **Request Execution** - HTTP request processing
2. **Collection Export** - Large collection exports
3. **Code Generation** - Multi-language code generation
4. **Data Cleanup** - History cleanup, cache invalidation
5. **Notifications** - Email/WebSocket notifications

## ⚛️ Frontend Architecture (React + TypeScript)

### Application Structure
```
src/
├── components/           # Reusable UI components
│   ├── ui/              # Base UI components
│   ├── forms/           # Form components
│   └── layout/          # Layout components
├── pages/               # Route components
│   ├── auth/
│   ├── workspace/
│   ├── collections/
│   └── requests/
├── hooks/               # Custom React hooks
├── services/            # API services
├── stores/              # Zustand stores
├── utils/               # Utility functions
├── types/               # TypeScript definitions
└── constants/           # Application constants
```

### State Management Architecture

#### Zustand Store Structure
```typescript
interface AppStore {
  // Auth state
  user: User | null
  token: string | null
  
  // Workspace state
  currentWorkspace: Workspace | null
  workspaces: Workspace[]
  
  // Request builder state
  currentRequest: RequestBuilder
  
  // UI state
  sidebarOpen: boolean
  theme: 'light' | 'dark'
}
```

#### Store Slices
1. **Auth Slice** - Authentication state
2. **Workspace Slice** - Workspace management
3. **Request Slice** - Request builder state
4. **UI Slice** - UI preferences and state

### Component Architecture Patterns

#### 1. Compound Components
```typescript
// Request builder compound component structure
<RequestBuilder>
  <RequestBuilder.Header />
  <RequestBuilder.Tabs>
    <RequestBuilder.Tab name="params">
      <RequestBuilder.ParamsEditor />
    </RequestBuilder.Tab>
    <RequestBuilder.Tab name="headers">
      <RequestBuilder.HeadersEditor />
    </RequestBuilder.Tab>
    <RequestBuilder.Tab name="body">
      <RequestBuilder.BodyEditor />
    </RequestBuilder.Tab>
  </RequestBuilder.Tabs>
  <RequestBuilder.Actions />
</RequestBuilder>
```

#### 2. Custom Hooks Pattern
```typescript
// Request execution hook
function useRequestExecution() {
  const [isExecuting, setIsExecuting] = useState(false)
  const [response, setResponse] = useState(null)
  
  const executeRequest = useCallback(async (request) => {
    // Execution logic with error handling
  }, [])
  
  return { executeRequest, isExecuting, response }
}
```

### API Integration Layer

#### Service Architecture
```typescript
class ApiService {
  private client: AxiosInstance
  
  constructor() {
    this.client = axios.create({
      baseURL: '/api',
      timeout: 30000,
    })
    
    this.setupInterceptors()
  }
  
  // Request/response interceptors for auth and error handling
  private setupInterceptors() {
    // Token injection, refresh logic, error handling
  }
}
```

## 🔐 Security Architecture

### Authentication Flow
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   Django    │    │  Database   │
│             │    │   Backend   │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
        │                   │                   │
        │  POST /login      │                   │
        ├──────────────────►│                   │
        │                   │  Validate User    │
        │                   ├──────────────────►│
        │                   │                   │
        │  JWT + Refresh    │                   │
        │◄──────────────────┤                   │
        │                   │                   │
        │  API Requests     │                   │
        │  (Bearer Token)   │                   │
        ├──────────────────►│                   │
        │                   │                   │
        │  Token Refresh    │                   │
        ├──────────────────►│                   │
        │  New JWT          │                   │
        │◄──────────────────┤                   │
```

### Security Measures
1. **JWT with Refresh Tokens** - Short-lived access tokens
2. **CORS Configuration** - Controlled cross-origin access
3. **Rate Limiting** - Per-user and per-IP limits
4. **Input Validation** - DRF serializers and Zod schemas
5. **SQL Injection Prevention** - Django ORM usage
6. **XSS Protection** - Content Security Policy
7. **HTTPS Enforcement** - SSL/TLS in production

## 📊 Data Flow Architecture

### Request Execution Flow
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   React     │    │   Django    │    │   Celery    │    │ External    │
│     UI      │    │    API      │    │   Worker    │    │     API     │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
        │                   │                   │                   │
        │  Execute Request  │                   │                   │
        ├──────────────────►│                   │                   │
        │                   │  Queue Task       │                   │
        │                   ├──────────────────►│                   │
        │  Task ID          │                   │                   │
        │◄──────────────────┤                   │                   │
        │                   │                   │  HTTP Request     │
        │                   │                   ├──────────────────►│
        │                   │                   │                   │
        │                   │                   │  HTTP Response    │
        │                   │                   │◄──────────────────┤
        │                   │  Store Result     │                   │
        │                   │◄──────────────────┤                   │
        │  WebSocket Update │                   │                   │
        │◄──────────────────┤                   │                   │
```

## 🚀 Deployment Architecture

### Container Strategy
```dockerfile
# Backend Dockerfile structure
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application
COPY . .

# Run migrations and start server
CMD ["gunicorn", "config.wsgi:application"]
```

### Docker Compose Configuration
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: api_playground
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    
  backend:
    build: ./backend
    depends_on:
      - db
      - redis
    environment:
      DATABASE_URL: postgresql://postgres:password@db:5432/api_playground
      REDIS_URL: redis://redis:6379
      
  celery:
    build: ./backend
    command: celery -A config worker -l info
    depends_on:
      - db
      - redis
      
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
```

### Production Deployment
1. **Load Balancer** - Nginx for SSL termination and load balancing
2. **Application Servers** - Multiple Django/Gunicorn instances
3. **Database** - PostgreSQL with read replicas
4. **Cache** - Redis cluster for caching and sessions
5. **Workers** - Celery workers for async processing
6. **Monitoring** - Prometheus, Grafana, and Sentry integration

## 🧪 Testing Strategy

### Backend Testing
```python
# Integration test example
class RequestExecutionTestCase(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(username='test')
        self.workspace = Workspace.objects.create(name='Test', owner=self.user)
        
    def test_request_execution(self):
        # Test request execution flow
        pass
```

### Frontend Testing
```typescript
// Component test example
describe('RequestBuilder', () => {
  it('should build valid HTTP request', () => {
    render(<RequestBuilder />)
    // Test user interactions and state changes
  })
})
```

### Testing Levels
1. **Unit Tests** - Models, serializers, components
2. **Integration Tests** - API endpoints, user flows
3. **E2E Tests** - Complete user workflows
4. **Performance Tests** - Load testing with locust

## 📈 Performance Optimization

### Backend Optimizations
1. **Database Indexing** - Optimized queries for collections and requests
2. **Query Optimization** - select_related and prefetch_related usage
3. **Caching Strategy** - Redis for frequently accessed data
4. **Connection Pooling** - PostgreSQL connection management
5. **Async Processing** - Celery for heavy operations

### Frontend Optimizations
1. **Code Splitting** - Route-based and component-based splitting
2. **Virtual Scrolling** - For large collections and history
3. **Memoization** - React.memo and useMemo for expensive operations
4. **Bundle Analysis** - Webpack bundle analyzer integration
5. **CDN Integration** - Static asset delivery optimization

## 🔧 Development Workflow

### Git Strategy
- **Main Branch** - Production-ready code
- **Develop Branch** - Integration branch
- **Feature Branches** - Individual features
- **Release Branches** - Release preparation

### CI/CD Pipeline
1. **Code Quality** - Linting, formatting, type checking
2. **Testing** - Unit, integration, and E2E tests
3. **Security Scanning** - Dependency and code security scans
4. **Build** - Docker image creation
5. **Deploy** - Automated deployment to staging/production

This architecture provides a robust foundation for the Django + React implementation of the API Playground, with clear separation of concerns, scalable design patterns, and comprehensive development practices.
