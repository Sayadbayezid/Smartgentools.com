# 🚀 Setup & Deployment Guide

## Prerequisites

- **Node.js**: 16.x or higher
- **npm**: 8.x or higher
- **Docker & Docker Compose**: Latest version
- **Git**: Latest version

## Local Development Setup

### 1. Clone Repository

```bash
git clone https://github.com/Sayadbayezid/Smartgentools.com.git
cd Smartgentools.com
```

### 2. Install Dependencies

```bash
# Install all workspace dependencies
npm install

# Or if using pnpm (faster)
pnpm install
```

### 3. Create Environment File

```bash
# Copy the example environment file
cp backend/.env.example backend/.env

# Edit with your values
nano backend/.env
```

### 4. Start Services

```bash
# Start Redis, MongoDB, and Mongo Express
docker-compose up -d

# Verify services are running
docker-compose ps
```

**Services Running**:
- **MongoDB**: `localhost:27017`
- **Redis**: `localhost:6379`
- **Mongo Express**: `http://localhost:8081` (Admin: admin/pass)

### 5. Start Development Servers

**Terminal 1 - Frontend**:
```bash
npm run dev:frontend
# Runs on http://localhost:5173
```

**Terminal 2 - Backend API**:
```bash
npm run dev:backend
# Runs on http://localhost:3000
```

**Terminal 3 - Worker Pool**:
```bash
npm run dev:worker
# Processes tasks from queue
```

**Or run all together**:
```bash
npm run dev
# All three servers in background (requires npm-concurrently)
```

### 6. Access Dashboard

Open browser and navigate to: `http://localhost:5173`

### 7. Test API

```bash
# Get health check
curl http://localhost:3000/api/health

# Create a test task
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "type": "AUTO_SHARE",
    "content": {"text": "Test post"},
    "targetGroups": ["group-1"]
  }'
```

## MongoDB Setup

### Option 1: Local MongoDB (Easiest for Development)

```bash
# Already included in docker-compose
docker-compose up -d mongodb

# Connect with Mongo Express
# Visit http://localhost:8081
# Username: admin
# Password: password
```

### Option 2: MongoDB Atlas (Recommended for Production)

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Create free cluster
3. Create database user
4. Whitelist your IP
5. Copy connection string
6. Update `MONGO_URI` in `.env`:

```env
MONGO_URI=mongodb+srv://username:password@cluster.mongodb.net/facebook-automation
```

## Redis Setup

### Option 1: Local Redis (Development)

```bash
docker-compose up -d redis

# Verify connection
redis-cli ping
# Response: PONG
```

### Option 2: Upstash (Recommended for Production)

1. Go to [Upstash](https://upstash.com/)
2. Create Redis database (free tier)
3. Copy connection string
4. Update `REDIS_URL` in `.env`:

```env
REDIS_URL=redis://default:password@endpoint:port
```

## Facebook Session Setup

### 1. Manual Cookie Capture

```bash
# Use your browser's DevTools to export cookies:
# 1. Open facebook.com
# 2. DevTools → Application → Cookies → facebook.com
# 3. Export cookies (manually or with extension)
# 4. Save as JSON
```

### 2. Via Browser Extension

1. Build extension: `npm run build:extension`
2. Load in Chrome: `chrome://extensions/`
3. Click "Load unpacked" → select `extension/dist/`
4. Login to Facebook through extension
5. Click "Save Session" button
6. Session stored securely in database

## Environment Configuration

### Required Variables

```env
# Core
NODE_ENV=development
PORT=3000

# Database
MONGO_URI=mongodb://localhost:27017/facebook-automation
REDIS_URL=redis://localhost:6379

# Authentication
JWT_SECRET=your-secret-key-minimum-32-characters
JWT_EXPIRY=7d

# Facebook
FACEBOOK_COOKIES_ENCRYPTION_KEY=your-key-32-chars

# Worker
HEADLESS=true
MAX_CONCURRENT_BROWSERS=5
```

### Optional Variables

```env
# Email notifications
SMTP_HOST=smtp.gmail.com
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# Monitoring
SENTRY_DSN=https://...

# Rate limiting
RATE_LIMIT_MAX_REQUESTS=100
RATE_LIMIT_WINDOW_MS=900000
```

## Database Seeding

### Add Approved Groups

```bash
# Create seed data
cat > backend/seeds/groups.json << EOF
[
  {
    "groupId": "facebook-group-id-1",
    "groupName": "Example Group 1",
    "groupUrl": "https://facebook.com/groups/...",
    "category": "tech",
    "approved": true
  },
  {
    "groupId": "facebook-group-id-2",
    "groupName": "Example Group 2",
    "groupUrl": "https://facebook.com/groups/...",
    "category": "business",
    "approved": true
  }
]
EOF

# Run seed
npm run seed:groups
```

## Building for Production

### 1. Build All Packages

```bash
npm run build:all

# Outputs:
# - frontend/dist/        → React SPA
# - backend/dist/         → Compiled Node.js
# - extension/dist/       → Chrome extension
```

### 2. Build Docker Image

```bash
npm run docker:build

# Tagged as: smartgentools:latest
```

### 3. Test Docker Image Locally

```bash
npm run docker:up
# docker-compose up -d

# Verify services
curl http://localhost:3000/api/health

# View logs
npm run docker:logs

# Stop services
npm run docker:down
# docker-compose down
```

## Deployment Options

### Option 1: Railway.app (Recommended - Easiest)

1. **Connect Repository**:
   - Go to [Railway.app](https://railway.app/)
   - Click "Create Project"
   - Connect GitHub
   - Select `Smartgentools.com` repo

2. **Add Services**:
   - Railway auto-detects Dockerfile
   - Add MongoDB Atlas service
   - Add Upstash Redis service

3. **Configure Environment**:
   - Set all variables in Railway dashboard
   - Railway handles secrets securely

4. **Deploy**:
   ```bash
   # Push to GitHub
   git push origin develop
   # Railway auto-deploys
   ```

5. **Monitor**:
   - View logs: Railway dashboard
   - Metrics: CPU, Memory, Network

### Option 2: Docker Hub + Railway

```bash
# Build and push image
docker build -t your-username/smartgentools:latest .
docker push your-username/smartgentools:latest

# In Railway dashboard:
# Select "Docker Image" → your-username/smartgentools:latest
```

### Option 3: Manual VPS (DigitalOcean/Linode)

```bash
# SSH into VPS
ssh root@your-vps-ip

# Install Docker
curl -fsSL https://get.docker.com | sh

# Clone repository
git clone https://github.com/Sayadbayezid/Smartgentools.com.git
cd Smartgentools.com

# Setup environment
cp backend/.env.example backend/.env
nano backend/.env

# Start with docker-compose
docker-compose -f docker-compose.prod.yml up -d

# Setup Nginx reverse proxy
# (See nginx.conf example below)
```

### Option 4: GitHub Actions + Cloud Run

```yaml
# .github/workflows/deploy.yml
name: Deploy to Cloud Run
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: google-github-actions/auth@v1
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
      - name: Build and push
        run: gcloud builds submit --tag gcr.io/PROJECT_ID/smartgentools
      - name: Deploy
        run: gcloud run deploy smartgentools --image gcr.io/PROJECT_ID/smartgentools
```

## Nginx Configuration (for VPS)

```nginx
# /etc/nginx/sites-available/smartgentools
upstream api {
  server localhost:3000;
}

server {
  listen 80;
  server_name api.smartgentools.com;

  location / {
    proxy_pass http://api;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  # SSL with Let's Encrypt
  listen 443 ssl;
  ssl_certificate /etc/letsencrypt/live/api.smartgentools.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/api.smartgentools.com/privkey.pem;
}

# Frontend on Vercel
server {
  listen 80;
  server_name smartgentools.com www.smartgentools.com;
  return 301 https://smartgentools.vercel.app$request_uri;
}
```

## SSL Certificate Setup

### Using Let's Encrypt (Free)

```bash
# Install Certbot
apt-get install certbot python3-certbot-nginx

# Generate certificate
certbot certonly --standalone -d api.smartgentools.com

# Auto-renew
certbot renew --dry-run
```

## Monitoring & Logs

### View Live Logs

```bash
# Backend API logs
docker-compose logs -f backend

# Worker logs
docker-compose logs -f worker

# Database logs
docker-compose logs -f mongodb

# All logs
docker-compose logs -f
```

### Setup Error Tracking (Sentry)

1. Create free account at [Sentry.io](https://sentry.io/)
2. Create project (Node.js)
3. Copy DSN
4. Add to `.env`:
   ```env
   SENTRY_DSN=https://key@sentry.io/project-id
   ```

### Monitor Queue & Tasks

```bash
# Connect to Redis CLI
redis-cli

# View queue stats
LLEN facebook-automation
LRANGE facebook-automation 0 -1

# View queue metrics
INFO stats
```

## Testing

### Unit Tests

```bash
npm run test:backend
npm run test:frontend
```

### E2E Tests

```bash
npm run test:e2e
```

### Load Testing

```bash
# Using artillery
npm install -g artillery

# Create test file
cat > load-test.yml << EOF
config:
  target: "http://localhost:3000"
  phases:
    - duration: 60
      arrivalRate: 10
scenarios:
  - name: "Create task"
    flow:
      - post:
          url: "/api/tasks"
          json:
            type: "AUTO_SHARE"
            targetGroups: ["group-1"]
EOF

# Run load test
artillery run load-test.yml
```

## Troubleshooting

### MongoDB Connection Failed

```bash
# Check MongoDB is running
docker ps | grep mongo

# Verify connection
mongosh "mongodb://localhost:27017"

# Check logs
docker logs smartgentools-mongodb
```

### Redis Connection Failed

```bash
# Check Redis is running
docker ps | grep redis

# Test connection
redis-cli ping

# Check logs
docker logs smartgentools-redis
```

### Worker Not Processing Tasks

```bash
# Check worker is running
docker ps | grep worker

# Check logs
npm run logs:worker

# Verify Redis queue
redis-cli LLEN facebook-automation

# Restart worker
docker-compose restart worker
```

### High Memory Usage

```bash
# Check memory
docker stats

# Reduce browser pool size
# Edit .env: MAX_CONCURRENT_BROWSERS=2

# Restart services
docker-compose restart
```

## Performance Optimization

### Backend

```env
# Increase max connections
NODE_ENV=production

# Database connection pooling
MONGO_MAX_POOL_SIZE=50
REDIS_MAX_RETRIES=3

# Disable debug logging
LOG_LEVEL=warn
```

### Frontend

```bash
# Enable production mode
NODE_ENV=production npm run build

# Analyze bundle
npm run build:analyze
```

### Database

```javascript
// Create indexes
db.tasks.createIndex({ userId: 1, createdAt: -1 });
db.groups.createIndex({ approved: 1 });
db.sessions.createIndex({ userId: 1 });
```

## Backup & Recovery

### MongoDB Backup

```bash
# Backup all data
mongodump --uri="mongodb://admin:password@localhost:27017" \
  --out ./backup

# Restore
mongorestore ./backup
```

### Export Data

```bash
# Export tasks to CSV
mongoexport --collection tasks \
  --uri="mongodb://localhost:27017/facebook-automation" \
  --out tasks.csv --csv
```

## Security Checklist

- [ ] Change all default passwords
- [ ] Enable HTTPS/SSL
- [ ] Setup firewall rules
- [ ] Enable rate limiting
- [ ] Setup CORS properly
- [ ] Rotate JWT secrets regularly
- [ ] Enable database backups
- [ ] Setup monitoring/alerts
- [ ] Use environment variables for secrets
- [ ] Keep dependencies updated

---

**Need Help?**
- Check logs: `npm run docker:logs`
- Read docs: See `/docs` folder
- Open issue: GitHub Issues
