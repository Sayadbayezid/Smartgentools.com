md
# 🏗️ System Architecture

## Overview

Smartgentools is a production-grade Facebook automation platform built with modern technologies and following industry best practices.

## High-Level Architecture
┌─────────────────────────────────────────────────────────────┐ │ USER INTERFACES │ ├─────────────┬──────────────────┬───────────┬────────────────┤ │ React Web │ Mobile PWA │ Chrome │ Admin Dashboard │ │ Dashboard │ (iOS/Android) │ Extension │ │ └─────┬───────┴────────┬─────────┴─────┬─────┴────────┬───────┘ │ │ │ │ └────────────────┼───────────────┴──────────────┘ │ ┌───────────▼──────────────┐ │ Express.js API Layer │ │ (REST + WebSocket) │ └────────────┬─────────────┘ │ ┌───────────────┼───────────────┐ │ │ │ ┌────▼─────┐ ┌─────▼──────┐ ┌─────▼─────┐ │ Bull │ │ MongoDB │ │ Auth & │ │ Queue │ │ Database │ │ Session │ │ (Redis) │ │ │ │ Manager │ └────┬─────┘ └────────────┘ └───────────┘ │ ┌────▼──────────────────────────────┐ │ Worker Pool (Node.js) │ │ │ │ ├─ Playwright Automation │ │ ├─ Task Executor │ │ ├─ Comment Scanner │ │ └─ Error Handler & Retry Logic │ └────┬───────────────────────────────┘ │ ┌────▼─────────────────────────────┐ │ Playwright Browsers │ │ (Headless Chrome Instances) │ └────┬─────────────────────────────┘ │ ┌────▼──────────────────────────────┐ │ Facebook.com │ │ (Browser Session - Cookie-based) │ └────────────────────────────────────┘
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