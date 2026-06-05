# 🏗️ System Architecture

## Overview

Smartgentools is a production-grade Facebook automation platform built with modern technologies and following industry best practices.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INTERFACES                           │
├─────────────┬──────────────────┬───────────┬────────────────┤
│ React Web   │ Mobile PWA       │ Chrome    │ Admin Dashboard │
│ Dashboard   │ (iOS/Android)    │ Extension │                │
└─────┬───────┴────────┬─────────┴─────┬─────┴────────┬───────┘
      │                │               ��              │
      └────────────────┼───────────────┴──────────────┘
                       │
           ┌───────────▼──────────────┐
           │   Express.js API Layer   │
           │  (REST + WebSocket)      │
           └────────────┬─────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌────▼─────┐  ┌─────▼──────┐  ┌─────▼─────┐
   │   Bull    │  │  MongoDB   │  │  Auth &   │
   │   Queue   │  │  Database  │  │  Session  │
   │  (Redis)  │  │            │  │  Manager  │
   └────┬──────┘  └────────────┘  └───────────┘
        │
   ┌────▼──────────────────────────────┐
   │  Worker Pool (Node.js)             │
   │                                    │
   │  ├─ Playwright Automation          │
   │  ├─ Task Executor                  │
   │  ├─ Comment Scanner                │
   │  └─ Error Handler & Retry Logic    │
   └────┬───────────────────────────────┘
        │
   ┌────▼─────────────────────────────┐
   │  Playwright Browsers              │
   │  (Headless Chrome Instances)      │
   └────┬─────────────────────────────┘
        │
   ┌────▼──────────────────────────────┐
   │  Facebook.com                      │
   │  (Browser Session - Cookie-based)  │
   └────────────────────────────────────┘
```

## Component Details

### 1. Frontend Layer

#### React Web Dashboard
- **Purpose**: Main control panel for users
- **Technology**: React 18, TypeScript, Vite, Tailwind CSS
- **Features**:
  - Real-time task monitoring
  - Group selection interface
  - Post composer
  - Execution logs viewer
  - User settings

#### Mobile PWA
- **Purpose**: iOS/Android app experience
- **Technology**: React PWA, Service Workers
- **Capabilities**:
  - Offline support
  - Install as app
  - Push notifications
  - Touch-optimized UI

#### Chrome Extension
- **Purpose**: Quick share from browser
- **Technology**: Manifest V3
- **Capabilities**:
  - One-click sharing
  - Current page detection
  - Background sync

### 2. API Layer

#### Express.js Server
- **Endpoints**:
  - `POST /api/tasks` - Create new task
  - `GET /api/tasks/:id` - Get task status
  - `GET /api/groups` - List approved groups
  - `POST /api/auth/login` - User authentication
  - `GET /api/sessions` - Manage Facebook sessions

- **Middleware**:
  - JWT authentication
  - Rate limiting
  - CORS protection
  - Request validation
  - Error handling

- **WebSocket Events**:
  - `task:created` - New task added
  - `task:progress` - Task execution update
  - `task:completed` - Task finished
  - `group:executed` - Group share status

### 3. Data Layer

#### MongoDB Collections

**Tasks Collection**
```javascript
{
  _id: ObjectId,
  userId: String,
  type: Enum["AUTO_SHARE", "COMMENT_REPLY"],
  status: Enum["PENDING", "RUNNING", "SUCCESS", "FAILED"],
  content: {
    text: String,
    mediaUrls: [String],
    link: String
  },
  targetGroups: [String],
  executedGroups: [{
    groupId: String,
    status: String,
    sharedUrl: String,
    timestamp: Date
  }],
  logs: [{ level, message, timestamp }],
  createdAt: Date,
  completedAt: Date
}
```

**Groups Collection**
```javascript
{
  _id: ObjectId,
  groupId: String,
  groupName: String,
  groupUrl: String,
  category: String,
  approved: Boolean,
  createdAt: Date
}
```

**Sessions Collection**
```javascript
{
  _id: ObjectId,
  userId: String,
  sessionId: String,
  cookies: [{ name, value, domain, path, ... }],
  userAgent: String,
  profileId: String,
  createdAt: Date,
  expiresAt: Date,
  isValid: Boolean
}
```

### 4. Queue System

#### Bull Queue (Redis-backed)

```typescript
// Job structure
{
  id: taskId,
  data: Task,
  attempts: 3,
  backoff: {
    type: "exponential",
    delay: 2000
  },
  status: "active" | "completed" | "failed"
}
```

**Queue Processing Flow**:
1. Task added to queue → Status: PENDING
2. Worker picks task → Status: RUNNING
3. Task completes → Status: SUCCESS
4. Task fails → Retry with exponential backoff
5. Max retries exceeded → Status: FAILED

### 5. Worker Pool

#### Playwright Automation Engine

**Browser Lifecycle**:
```
Browser Pool (max 5 browsers)
├── Browser 1 (task-1)
├── Browser 2 (task-2)
├── Browser 3 (idle)
├── Browser 4 (idle)
└── Browser 5 (idle)
```

**Task Execution Flow**:
```
1. Fetch session (cookies)
2. Launch browser context
3. Load cookies into context
4. Navigate to Facebook
5. Execute automation steps
6. Capture results
7. Close browser
8. Update database
9. Send webhook
```

### 6. Security Architecture

#### Authentication Flow

```
User Login
    ↓
POST /api/auth/login (email, password)
    ↓
Validate credentials
    ↓
Generate JWT Token
    ↓
Return { token, refreshToken, user }
    ↓
Store in localStorage (frontend)
    ↓
Include in Authorization header: "Bearer {token}"
    ↓
Validate on each request (middleware)
```

#### Cookie Encryption

```
Facebook Cookies (Plain)
    ↓
Encrypt with AES-256-GCM
    ↓
Store in database
    ↓
(When needed)
    ↓
Decrypt with key
    ↓
Load into Playwright browser context
```

#### Session Management

- JWT access token: 7 days expiry
- Refresh token: 30 days expiry
- Cookie session: 60 days or until invalid
- Automatic session validation on use

## Deployment Architecture

### Local Development
- Docker Compose orchestrates all services
- Hot reload for frontend & backend
- Local MongoDB & Redis

### Production (Railway.app / Docker)

```
┌─────────────────────────────────────────┐
│          Docker Container               │
├─────────────────────────────────────────┤
│                                          │
│  ┌────────────────┐  ┌──────────────┐  │
│  │ Express API    │  │  Worker Pool │  │
│  │ (Port 3000)    │  │  (5 workers) │  │
│  └────────────────┘  └──────────────┘  │
│                                          │
└─────────────────────────────────────────┘
           ↓              ↓
    ┌──────────────┐  ┌─────────────┐
    │  MongoDB     │  │   Redis     │
    │  Atlas       │  │   Upstash   │
    └──────────────┘  └─────────────┘
           ↑
    ┌──────────────┐
    │  Frontend    │
    │  (Vercel)    │
    └──────────────┘
```

## Scalability Considerations

### Horizontal Scaling
- Stateless API servers (multiple instances)
- Shared Redis queue
- Shared MongoDB database
- Load balancer (Railway/Docker Swarm)

### Worker Scaling
- Increase `MAX_CONCURRENT_BROWSERS` per instance
- Deploy additional worker containers
- Auto-scaling based on queue size

### Database Optimization
- Indexing on userId, createdAt, status
- TTL on logs (auto-cleanup after 30 days)
- Connection pooling

### Caching Strategy
- Redis cache for approved groups
- In-memory cache for selectors
- Browser context reuse

## Error Handling & Recovery

### Task Failure Scenarios

1. **Network Error** → Retry with exponential backoff
2. **Facebook Selector Changed** → Log error, notify admin
3. **Session Expired** → Mark session invalid, request re-auth
4. **Browser Crash** → Restart browser, retry task
5. **Rate Limit** → Add to queue with delay
6. **Timeout** → Mark failed after 5 minutes

### Recovery Strategies

```typescript
class RetryEngine {
  async executeWithRetry(fn, maxRetries = 3) {
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fn();
      } catch (error) {
        if (i === maxRetries - 1) throw error;
        const delay = Math.pow(2, i) * 1000;
        await sleep(delay);
      }
    }
  }
}
```

## Monitoring & Logging

### Metrics Collected
- Task success rate
- Average execution time
- Queue depth
- Worker utilization
- Error rates by type

### Logging
- Winston for structured logging
- Separate logs for: API, Worker, Errors
- Log rotation (daily, 30-day retention)
- Sentry for error tracking

### Observability
```
Tasks Created      → API logs + dashboard
Tasks Executed     → Worker logs + database
Errors Occurred    → Sentry alerts + admin email
User Actions       → Analytics service
```

## Data Flow Example

### Auto-Share Task Execution

```
1. USER ACTION (Frontend)
   └─ Select groups, write post, click "Share"

2. API REQUEST
   └─ POST /api/tasks
      └─ Validate input
      └─ Create Task (MongoDB)
      └─ Add to Bull Queue
      └─ Return taskId to frontend

3. REAL-TIME UPDATE
   └─ WebSocket: "task:created"
   └─ Frontend: Show task in list

4. QUEUE PROCESSING
   └─ Worker: Pick task from queue
   └─ Update status: RUNNING
   └─ Fetch session (cookies)
   └─ Launch browser

5. AUTOMATION
   └─ For each group:
      ├─ Navigate to group
      ├─ Click share button
      ├─ Fill content
      ├─ Submit
      ├─ Capture screenshot
      ├─ Update executedGroups[]
      ├─ WebSocket: "task:progress"

6. COMPLETION
   └─ Close browser
   └─ Update Task status: SUCCESS
   └─ Store logs
   └─ WebSocket: "task:completed"
   └─ Send email notification

7. USER SEES
   └─ Task status: SUCCESS
   └─ Execution details with links
   └─ Shared URLs for each group
   └─ Full execution logs
```

## Technology Choices Rationale

| Component | Choice | Reason |
|-----------|--------|--------|
| Frontend | React + TypeScript | Type safety, component reusability, large ecosystem |
| Backend | Node.js + Express | Fast development, handles async well, shared JavaScript |
| Database | MongoDB | Flexible schema, JSON-like documents, free tier available |
| Queue | Bull + Redis | Reliable, distributed, built-in retry logic |
| Automation | Playwright | Modern, supports multiple browsers, robust selectors |
| Deployment | Docker | Consistency, portability, easy scaling |

## Future Enhancements

- [ ] Schedule posts for later times
- [ ] AI-powered comment response suggestions
- [ ] Integration with other social platforms
- [ ] Analytics dashboard with performance metrics
- [ ] Multi-account management
- [ ] Custom workflow builder
- [ ] API for third-party integrations
- [ ] Mobile app (native iOS/Android)
