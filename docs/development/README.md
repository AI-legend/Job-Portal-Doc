# Development Setup Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Project Structure](#project-structure)
3. [Environment Setup](#environment-setup)
4. [Database Setup](#database-setup)
5. [Service Configuration](#service-configuration)
6. [Development Workflow](#development-workflow)
7. [Testing Strategy](#testing-strategy)
8. [Troubleshooting](#troubleshooting)

## Prerequisites

### System Requirements
- **Node.js**: v18.0.0 or higher
- **Python**: v3.11 or higher
- **Docker**: v20.0.0 or higher
- **Docker Compose**: v2.0.0 or higher
- **Git**: v2.30.0 or higher

### Development Tools
- **VS Code** (recommended) with extensions:
  - ESLint
  - Prettier
  - TypeScript and JavaScript Language Features
  - Python Extension Pack
  - Docker Extension
  - REST Client
- **Postman** or **Insomnia** for API testing
- **MongoDB Compass** for database management
- **Redis Desktop Manager** for cache inspection

### Package Managers
```bash
# Install Node.js packages
npm install -g yarn pnpm

# Install Python package managers
pip install pipenv poetry

# Install global development tools
npm install -g @nestjs/cli
npm install -g nodemon
npm install -g typescript
pip install black flake8 pytest
```

## Project Structure

```
hr-agent/
├── .env.example                 # Environment variables template
├── .gitignore                   # Git ignore rules
├── docker-compose.yml           # Development containers
├── docker-compose.prod.yml      # Production containers
├── package.json                 # Root package.json for scripts
├── README.md                    # Project documentation
├── docs/                        # Documentation
│   ├── api/                     # API documentation
│   ├── architecture/            # System architecture
│   ├── deployment/              # Deployment guides
│   └── development/             # Development guides
├── frontend/                    # React/Next.js frontend
│   ├── src/
│   │   ├── components/          # React components
│   │   ├── pages/               # Next.js pages
│   │   ├── hooks/               # Custom hooks
│   │   ├── services/            # API services
│   │   ├── store/               # State management
│   │   ├── utils/               # Utility functions
│   │   └── styles/              # Stylesheets
│   ├── public/                  # Static assets
│   ├── tests/                   # Frontend tests
│   ├── package.json             # Frontend dependencies
│   ├── next.config.js           # Next.js configuration
│   └── tailwind.config.js       # Tailwind CSS config
├── backend/                     # Backend services
│   ├── api-gateway/             # API Gateway service
│   │   ├── src/
│   │   │   ├── routes/          # Express routes
│   │   │   ├── middleware/      # Custom middleware
│   │   │   ├── utils/           # Utility functions
│   │   │   └── app.js           # Main application
│   │   ├── tests/               # Gateway tests
│   │   ├── package.json         # Dependencies
│   │   └── Dockerfile           # Container definition
│   ├── user-service/            # User management service
│   │   ├── src/
│   │   │   ├── controllers/     # Request handlers
│   │   │   ├── models/          # Data models
│   │   │   ├── services/        # Business logic
│   │   │   ├── routes/          # API routes
│   │   │   └── app.js           # Service entry point
│   │   ├── tests/               # Service tests
│   │   ├── package.json         # Dependencies
│   │   └── Dockerfile           # Container definition
│   ├── job-service/             # Job management service
│   ├── cv-service/              # CV management service
│   ├── ai-service/              # AI/ML service (Python)
│   │   ├── src/
│   │   │   ├── api/             # FastAPI routes
│   │   │   ├── services/        # Business logic
│   │   │   ├── models/          # ML models
│   │   │   ├── utils/           # Utilities
│   │   │   └── main.py          # FastAPI app
│   │   ├── tests/               # Python tests
│   │   ├── requirements.txt     # Python dependencies
│   │   └── Dockerfile           # Container definition
│   └── scraper-service/         # Web scraping service
├── workflows/                   # n8n workflow definitions
│   ├── job-scraping.json        # Job scraping workflows
│   ├── notifications.json       # Notification workflows
│   └── auto-apply.json          # Auto-apply workflows
├── database/                    # Database schemas and migrations
│   ├── mongodb/                 # MongoDB schemas
│   ├── postgresql/              # PostgreSQL migrations
│   └── redis/                   # Redis configurations
├── infrastructure/              # Infrastructure as Code
│   ├── docker/                  # Docker configurations
│   ├── kubernetes/              # K8s manifests
│   ├── terraform/               # AWS/GCP infrastructure
│   └── helm/                    # Helm charts
├── scripts/                     # Development scripts
│   ├── setup.sh                 # Initial setup script
│   ├── dev-start.sh             # Start development environment
│   ├── test.sh                  # Run all tests
│   └── deploy.sh                # Deployment script
└── tests/                       # Integration tests
    ├── e2e/                     # End-to-end tests
    ├── integration/             # Service integration tests
    └── performance/             # Performance tests
```

## Environment Setup

### 1. Clone Repository
```bash
git clone https://github.com/your-org/hr-agent.git
cd hr-agent
```

### 2. Initial Setup Script
```bash
# Make setup script executable
chmod +x scripts/setup.sh

# Run setup script
./scripts/setup.sh
```

### 3. Environment Variables
```bash
# Copy environment template
cp .env.example .env

# Edit environment variables
nano .env
```

### Environment Variables Configuration
```bash
# .env
# =============================================================================
# GENERAL CONFIGURATION
# =============================================================================
NODE_ENV=development
LOG_LEVEL=debug
API_VERSION=v1

# =============================================================================
# DATABASE CONFIGURATION
# =============================================================================
# MongoDB
MONGODB_URI=mongodb://localhost:27017/hr_agent_dev
MONGODB_TEST_URI=mongodb://localhost:27017/hr_agent_test

# PostgreSQL (for n8n and analytics)
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=hr_agent_dev
POSTGRES_USER=hr_agent
POSTGRES_PASSWORD=dev_password

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=redis_dev_password
REDIS_DB=0

# =============================================================================
# SERVICE CONFIGURATION
# =============================================================================
# API Gateway
API_GATEWAY_PORT=8000
API_GATEWAY_HOST=localhost

# User Service
USER_SERVICE_PORT=8001
USER_SERVICE_HOST=localhost

# Job Service
JOB_SERVICE_PORT=8002
JOB_SERVICE_HOST=localhost

# CV Service
CV_SERVICE_PORT=8003
CV_SERVICE_HOST=localhost

# AI Service
AI_SERVICE_PORT=8004
AI_SERVICE_HOST=localhost

# Scraper Service
SCRAPER_SERVICE_PORT=8005
SCRAPER_SERVICE_HOST=localhost

# Frontend
FRONTEND_PORT=3000
FRONTEND_HOST=localhost
NEXT_PUBLIC_API_BASE_URL=http://localhost:8000

# n8n Workflow Engine
N8N_PORT=5678
N8N_HOST=localhost
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=dev_password

# =============================================================================
# AUTHENTICATION & SECURITY
# =============================================================================
JWT_SECRET=your_jwt_secret_key_for_development
JWT_EXPIRES_IN=24h
REFRESH_TOKEN_SECRET=your_refresh_token_secret
BCRYPT_ROUNDS=10

# Firebase Configuration
FIREBASE_PROJECT_ID=hr-agent-dev
FIREBASE_CLIENT_EMAIL=firebase-adminsdk@hr-agent-dev.iam.gserviceaccount.com
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
FIREBASE_DATABASE_URL=https://hr-agent-dev.firebaseio.com

# =============================================================================
# EXTERNAL API KEYS
# =============================================================================
# OpenAI
OPENAI_API_KEY=sk-your-openai-api-key
OPENAI_MODEL=gpt-4

# Claude (Anthropic)
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key

# Job Board APIs
LINKEDIN_CLIENT_ID=your_linkedin_client_id
LINKEDIN_CLIENT_SECRET=your_linkedin_client_secret
INDEED_PUBLISHER_ID=your_indeed_publisher_id
GLASSDOOR_PARTNER_ID=your_glassdoor_partner_id
GLASSDOOR_API_KEY=your_glassdoor_api_key

# =============================================================================
# CLOUD STORAGE
# =============================================================================
# AWS S3
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
AWS_REGION=us-west-2
AWS_S3_BUCKET=hr-agent-dev-storage

# Google Cloud Storage
GOOGLE_CLOUD_PROJECT_ID=hr-agent-dev
GOOGLE_CLOUD_KEYFILE=path/to/service-account.json
GOOGLE_CLOUD_STORAGE_BUCKET=hr-agent-dev-gcs

# =============================================================================
# EMAIL & NOTIFICATIONS
# =============================================================================
# SendGrid
SENDGRID_API_KEY=your_sendgrid_api_key
FROM_EMAIL=noreply@hr-agent.com
FROM_NAME=HR Agent Platform

# Twilio (SMS)
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=+1234567890

# Slack (for notifications)
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/your/slack/webhook

# =============================================================================
# MONITORING & ANALYTICS
# =============================================================================
# Sentry (Error tracking)
SENTRY_DSN=https://your-sentry-dsn@sentry.io/project-id

# Google Analytics
GOOGLE_ANALYTICS_ID=GA-XXXXXXXXX

# =============================================================================
# DEVELOPMENT TOOLS
# =============================================================================
# Hot reload and development
CHOKIDAR_USEPOLLING=true
FAST_REFRESH=true

# Testing
TEST_TIMEOUT=30000
JEST_TIMEOUT=30000

# =============================================================================
# FEATURE FLAGS
# =============================================================================
ENABLE_AUTO_APPLY=true
ENABLE_AI_RECOMMENDATIONS=true
ENABLE_WEB_SCRAPING=true
ENABLE_PREMIUM_FEATURES=false
```

## Database Setup

### 1. MongoDB Setup
```bash
# Start MongoDB with Docker
docker run -d \
  --name mongodb-dev \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=admin123 \
  -v mongodb_data:/data/db \
  mongo:6.0

# Create development database and user
mongosh mongodb://admin:admin123@localhost:27017/admin
```

```javascript
// MongoDB setup commands
use hr_agent_dev;

// Create application user
db.createUser({
  user: "hr_agent",
  pwd: "dev_password",
  roles: [
    { role: "readWrite", db: "hr_agent_dev" },
    { role: "readWrite", db: "hr_agent_test" }
  ]
});

// Create collections with indexes
db.users.createIndex({ email: 1 }, { unique: true });
db.users.createIndex({ "profile.location.coordinates": "2dsphere" });

db.jobs.createIndex({ title: "text", description: "text" });
db.jobs.createIndex({ "location.coordinates": "2dsphere" });
db.jobs.createIndex({ skills: 1 });
db.jobs.createIndex({ createdAt: -1 });
db.jobs.createIndex({ isActive: 1, createdAt: -1 });

db.cvs.createIndex({ userId: 1 });
db.cvs.createIndex({ "skills.name": 1 });

db.applications.createIndex({ applicantId: 1, createdAt: -1 });
db.applications.createIndex({ jobId: 1 });
db.applications.createIndex({ status: 1 });
```

### 2. PostgreSQL Setup (for n8n)
```bash
# Start PostgreSQL with Docker
docker run -d \
  --name postgres-dev \
  -p 5432:5432 \
  -e POSTGRES_DB=hr_agent_dev \
  -e POSTGRES_USER=hr_agent \
  -e POSTGRES_PASSWORD=dev_password \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:14

# Connect and create n8n database
psql -h localhost -U hr_agent -d hr_agent_dev
```

```sql
-- Create n8n database
CREATE DATABASE n8n_dev;
GRANT ALL PRIVILEGES ON DATABASE n8n_dev TO hr_agent;

-- Create analytics tables
CREATE SCHEMA analytics;

CREATE TABLE analytics.user_events (
    id SERIAL PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    event_data JSONB,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    session_id VARCHAR(255)
);

CREATE TABLE analytics.job_metrics (
    id SERIAL PRIMARY KEY,
    job_id VARCHAR(255) NOT NULL,
    views INTEGER DEFAULT 0,
    applications INTEGER DEFAULT 0,
    date DATE DEFAULT CURRENT_DATE,
    UNIQUE(job_id, date)
);

-- Create indexes
CREATE INDEX idx_user_events_user_id ON analytics.user_events(user_id);
CREATE INDEX idx_user_events_timestamp ON analytics.user_events(timestamp);
CREATE INDEX idx_job_metrics_job_id ON analytics.job_metrics(job_id);
```

### 3. Redis Setup
```bash
# Start Redis with Docker
docker run -d \
  --name redis-dev \
  -p 6379:6379 \
  redis:7-alpine redis-server --requirepass redis_dev_password

# Test connection
redis-cli -h localhost -p 6379 -a redis_dev_password ping
```

## Service Configuration

### 1. Docker Compose Development Setup
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Databases
  mongodb:
    image: mongo:6.0
    container_name: hr-agent-mongodb-dev
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin123
      MONGO_INITDB_DATABASE: hr_agent_dev
    volumes:
      - mongodb_data:/data/db
      - ./database/mongodb/init.js:/docker-entrypoint-initdb.d/init.js
    networks:
      - hr-agent-network

  postgres:
    image: postgres:14
    container_name: hr-agent-postgres-dev
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: hr_agent_dev
      POSTGRES_USER: hr_agent
      POSTGRES_PASSWORD: dev_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/postgresql/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - hr-agent-network

  redis:
    image: redis:7-alpine
    container_name: hr-agent-redis-dev
    ports:
      - "6379:6379"
    command: redis-server --requirepass redis_dev_password
    volumes:
      - redis_data:/data
    networks:
      - hr-agent-network

  # Core Services
  api-gateway:
    build:
      context: ./backend/api-gateway
      target: development
    container_name: hr-agent-api-gateway-dev
    ports:
      - "8000:8000"
    environment:
      - NODE_ENV=development
      - PORT=8000
    volumes:
      - ./backend/api-gateway:/app
      - /app/node_modules
    depends_on:
      - mongodb
      - redis
    networks:
      - hr-agent-network
    command: npm run dev

  user-service:
    build:
      context: ./backend/user-service
      target: development
    container_name: hr-agent-user-service-dev
    ports:
      - "8001:8001"
    environment:
      - NODE_ENV=development
      - PORT=8001
    volumes:
      - ./backend/user-service:/app
      - /app/node_modules
    depends_on:
      - mongodb
      - redis
    networks:
      - hr-agent-network
    command: npm run dev

  job-service:
    build:
      context: ./backend/job-service
      target: development
    container_name: hr-agent-job-service-dev
    ports:
      - "8002:8002"
    environment:
      - NODE_ENV=development
      - PORT=8002
    volumes:
      - ./backend/job-service:/app
      - /app/node_modules
    depends_on:
      - mongodb
      - redis
    networks:
      - hr-agent-network
    command: npm run dev

  cv-service:
    build:
      context: ./backend/cv-service
      target: development
    container_name: hr-agent-cv-service-dev
    ports:
      - "8003:8003"
    environment:
      - NODE_ENV=development
      - PORT=8003
    volumes:
      - ./backend/cv-service:/app
      - /app/node_modules
    depends_on:
      - mongodb
      - redis
    networks:
      - hr-agent-network
    command: npm run dev

  ai-service:
    build:
      context: ./backend/ai-service
      target: development
    container_name: hr-agent-ai-service-dev
    ports:
      - "8004:8004"
    environment:
      - PYTHONPATH=/app
      - DEBUG=true
    volumes:
      - ./backend/ai-service:/app
    depends_on:
      - mongodb
      - redis
    networks:
      - hr-agent-network
    command: uvicorn main:app --host 0.0.0.0 --port 8004 --reload

  scraper-service:
    build:
      context: ./backend/scraper-service
      target: development
    container_name: hr-agent-scraper-service-dev
    ports:
      - "8005:8005"
    environment:
      - PYTHONPATH=/app
      - DEBUG=true
    volumes:
      - ./backend/scraper-service:/app
    depends_on:
      - mongodb
      - redis
    networks:
      - hr-agent-network
    command: uvicorn main:app --host 0.0.0.0 --port 8005 --reload

  # Frontend
  frontend:
    build:
      context: ./frontend
      target: development
    container_name: hr-agent-frontend-dev
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - NEXT_PUBLIC_API_BASE_URL=http://localhost:8000
    volumes:
      - ./frontend:/app
      - /app/node_modules
      - /app/.next
    depends_on:
      - api-gateway
    networks:
      - hr-agent-network
    command: npm run dev

  # n8n Workflow Engine
  n8n:
    image: n8nio/n8n:latest
    container_name: hr-agent-n8n-dev
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=dev_password
      - N8N_HOST=localhost
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=development
      - WEBHOOK_URL=http://localhost:5678
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n_dev
      - DB_POSTGRESDB_USER=hr_agent
      - DB_POSTGRESDB_PASSWORD=dev_password
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/.n8n/workflows
    depends_on:
      - postgres
    networks:
      - hr-agent-network

volumes:
  mongodb_data:
  postgres_data:
  redis_data:
  n8n_data:

networks:
  hr-agent-network:
    driver: bridge
```

### 2. Development Dockerfiles

#### Frontend Dockerfile (Development)
```dockerfile
# frontend/Dockerfile
FROM node:18-alpine AS base
WORKDIR /app

FROM base AS dependencies
COPY package*.json ./
RUN npm ci

FROM base AS development
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

FROM base AS builder
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM base AS production
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
USER nextjs
EXPOSE 3000
ENV PORT 3000
CMD ["node", "server.js"]
```

#### Backend Service Dockerfile (Development)
```dockerfile
# backend/*/Dockerfile
FROM node:18-alpine AS base
WORKDIR /app

FROM base AS dependencies
COPY package*.json ./
RUN npm ci --only=production

FROM base AS development
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 8000
CMD ["npm", "run", "dev"]

FROM base AS builder
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM base AS production
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
USER nextjs
EXPOSE 8000
CMD ["node", "dist/app.js"]
```

#### Python Service Dockerfile (Development)
```dockerfile
# backend/ai-service/Dockerfile
FROM python:3.11-slim AS base
WORKDIR /app

RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

FROM base AS dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base AS development
COPY requirements.txt requirements-dev.txt ./
RUN pip install --no-cache-dir -r requirements.txt -r requirements-dev.txt
COPY . .
EXPOSE 8004
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8004", "--reload"]

FROM base AS production
COPY --from=dependencies /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=dependencies /usr/local/bin /usr/local/bin
COPY . .
RUN useradd --create-home --shell /bin/bash app
USER app
EXPOSE 8004
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8004"]
```

## Development Workflow

### 1. Starting Development Environment
```bash
# Start all services
./scripts/dev-start.sh

# Or manually with Docker Compose
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f api-gateway
docker-compose logs -f frontend
```

### 2. Development Scripts
```bash
# scripts/dev-start.sh
#!/bin/bash
set -e

echo "🚀 Starting HR Agent Development Environment..."

# Check if Docker is running
if ! docker info > /dev/null 2>&1; then
    echo "❌ Docker is not running. Please start Docker first."
    exit 1
fi

# Create network if it doesn't exist
docker network create hr-agent-network 2>/dev/null || true

# Start databases first
echo "📦 Starting databases..."
docker-compose up -d mongodb postgres redis

# Wait for databases to be ready
echo "⏳ Waiting for databases to be ready..."
sleep 10

# Start services
echo "🔧 Starting backend services..."
docker-compose up -d api-gateway user-service job-service cv-service ai-service scraper-service

# Start frontend
echo "🎨 Starting frontend..."
docker-compose up -d frontend

# Start n8n
echo "🔄 Starting workflow engine..."
docker-compose up -d n8n

echo "✅ Development environment started!"
echo ""
echo "🌐 Services are available at:"
echo "  Frontend:     http://localhost:3000"
echo "  API Gateway:  http://localhost:8000"
echo "  n8n:          http://localhost:5678 (admin/dev_password)"
echo ""
echo "📊 Databases:"
echo "  MongoDB:      mongodb://localhost:27017"
echo "  PostgreSQL:   postgresql://localhost:5432"
echo "  Redis:        redis://localhost:6379"
echo ""
echo "🔍 To view logs: docker-compose logs -f [service-name]"
echo "🛑 To stop: docker-compose down"
```

### 3. Hot Reloading Setup

#### Frontend (Next.js)
```json
// frontend/package.json
{
  "scripts": {
    "dev": "next dev",
    "dev:turbo": "next dev --turbo",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "@types/react": "^18.0.0",
    "typescript": "^4.9.0",
    "eslint": "^8.0.0",
    "eslint-config-next": "^13.0.0"
  }
}
```

#### Backend Services (Node.js)
```json
// backend/*/package.json
{
  "scripts": {
    "dev": "nodemon --exec ts-node src/app.ts",
    "dev:debug": "nodemon --exec 'node --inspect=0.0.0.0:9229 -r ts-node/register' src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write src/**/*.ts"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "@types/express": "^4.17.0",
    "typescript": "^4.9.0",
    "ts-node": "^10.9.0",
    "nodemon": "^2.0.0",
    "jest": "^29.0.0",
    "@types/jest": "^29.0.0"
  }
}
```

#### Python Services
```txt
# backend/ai-service/requirements-dev.txt
uvicorn[standard]==0.20.0
fastapi==0.95.0
pytest==7.2.0
pytest-asyncio==0.21.0
black==23.1.0
flake8==6.0.0
mypy==1.1.0
pre-commit==3.1.0
```

### 4. IDE Configuration

#### VS Code Settings
```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "python.formatting.provider": "black",
  "python.linting.enabled": true,
  "python.linting.flake8Enabled": true,
  "eslint.workingDirectories": [
    "frontend",
    "backend/api-gateway",
    "backend/user-service",
    "backend/job-service",
    "backend/cv-service"
  ],
  "typescript.preferences.importModuleSpecifier": "relative",
  "files.exclude": {
    "**/node_modules": true,
    "**/__pycache__": true,
    "**/dist": true,
    "**/.next": true
  },
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/.next": true
  }
}
```

#### VS Code Extensions
```json
// .vscode/extensions.json
{
  "recommendations": [
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "ms-python.python",
    "ms-python.flake8",
    "ms-python.black-formatter",
    "bradlc.vscode-tailwindcss",
    "ms-vscode.vscode-typescript-next",
    "ms-azuretools.vscode-docker",
    "humao.rest-client",
    "mongodb.mongodb-vscode",
    "ms-vscode.vscode-json"
  ]
}
```

#### VS Code Tasks
```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Start Development Environment",
      "type": "shell",
      "command": "./scripts/dev-start.sh",
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      },
      "problemMatcher": []
    },
    {
      "label": "Stop Development Environment",
      "type": "shell",
      "command": "docker-compose down",
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Run Frontend Tests",
      "type": "shell",
      "command": "npm test",
      "options": {
        "cwd": "${workspaceFolder}/frontend"
      },
      "group": "test",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    },
    {
      "label": "Run Backend Tests",
      "type": "shell",
      "command": "./scripts/test.sh",
      "group": "test",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "focus": false,
        "panel": "shared"
      }
    }
  ]
}
```

## Testing Strategy

### 1. Testing Setup Script
```bash
# scripts/test.sh
#!/bin/bash
set -e

echo "🧪 Running HR Agent Test Suite..."

# Start test databases
echo "🗄️ Starting test databases..."
docker-compose -f docker-compose.test.yml up -d mongodb-test postgres-test redis-test

# Wait for databases
sleep 5

# Run frontend tests
echo "🎨 Running frontend tests..."
cd frontend
npm test -- --coverage --watchAll=false
cd ..

# Run backend service tests
echo "🔧 Running backend service tests..."
services=("api-gateway" "user-service" "job-service" "cv-service")

for service in "${services[@]}"; do
    echo "Testing $service..."
    cd "backend/$service"
    npm test -- --coverage --passWithNoTests
    cd ../..
done

# Run Python service tests
echo "🐍 Running Python service tests..."
python_services=("ai-service" "scraper-service")

for service in "${python_services[@]}"; do
    echo "Testing $service..."
    cd "backend/$service"
    python -m pytest tests/ --cov=src --cov-report=term-missing
    cd ../..
done

# Run integration tests
echo "🔗 Running integration tests..."
cd tests/integration
npm test
cd ../..

# Run E2E tests
echo "🌐 Running E2E tests..."
cd tests/e2e
npm test
cd ../..

# Cleanup test databases
echo "🧹 Cleaning up test environment..."
docker-compose -f docker-compose.test.yml down

echo "✅ All tests completed!"
```

### 2. Test Configuration

#### Jest Configuration (Frontend)
```javascript
// frontend/jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  testPathIgnorePatterns: ['<rootDir>/.next/', '<rootDir>/node_modules/'],
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/pages/_app.tsx',
    '!src/pages/_document.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

#### Jest Configuration (Backend)
```javascript
// backend/*/jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/app.ts',
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
};
```

#### Pytest Configuration (Python)
```ini
# backend/*/pytest.ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    --strict-markers
    --disable-warnings
    --cov=src
    --cov-report=term-missing
    --cov-report=html:htmlcov
    --cov-fail-under=80
markers =
    unit: Unit tests
    integration: Integration tests
    slow: Slow tests
```

### 3. Test Examples

#### Frontend Component Test
```typescript
// frontend/tests/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { Button } from '@/components/common/Button';

describe('Button Component', () => {
  test('renders button with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button')).toHaveTextContent('Click me');
  });

  test('calls onClick handler when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click me</Button>);
    
    fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('applies correct variant styles', () => {
    render(<Button variant="primary">Primary Button</Button>);
    expect(screen.getByRole('button')).toHaveClass('bg-blue-600');
  });
});
```

#### Backend Service Test
```typescript
// backend/user-service/tests/controllers/userController.test.ts
import request from 'supertest';
import { app } from '../../src/app';
import { User } from '../../src/models/User';

describe('User Controller', () => {
  beforeEach(async () => {
    await User.deleteMany({});
  });

  describe('POST /api/v1/users/register', () => {
    test('should create a new user', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'password123',
        userType: 'seeker',
        firstName: 'Test',
        lastName: 'User'
      };

      const response = await request(app)
        .post('/api/v1/users/register')
        .send(userData)
        .expect(201);

      expect(response.body.success).toBe(true);
      expect(response.body.data.user.email).toBe(userData.email);
      expect(response.body.data.token).toBeDefined();
    });

    test('should return 400 for invalid email', async () => {
      const userData = {
        email: 'invalid-email',
        password: 'password123',
        userType: 'seeker'
      };

      const response = await request(app)
        .post('/api/v1/users/register')
        .send(userData)
        .expect(400);

      expect(response.body.success).toBe(false);
      expect(response.body.error.code).toBe('VALIDATION_ERROR');
    });
  });
});
```

#### Python Service Test
```python
# backend/ai-service/tests/test_job_matcher.py
import pytest
from unittest.mock import Mock, patch
from src.services.job_matcher import JobMatcherService

class TestJobMatcherService:
    @pytest.fixture
    def job_matcher(self):
        return JobMatcherService()
    
    @pytest.fixture
    def sample_cv(self):
        return {
            "skills": ["Python", "React", "AWS"],
            "experience": [
                {
                    "company": "TechCorp",
                    "position": "Software Engineer",
                    "duration": 36
                }
            ]
        }
    
    @pytest.fixture
    def sample_job(self):
        return {
            "title": "Senior Software Engineer",
            "skills": ["Python", "React", "Docker"],
            "requirements": ["3+ years experience"],
            "description": "We are looking for a senior developer..."
        }
    
    def test_calculate_skill_match(self, job_matcher, sample_cv, sample_job):
        score = job_matcher.calculate_skill_match(sample_cv, sample_job)
        assert 0.0 <= score <= 1.0
        assert score > 0.5  # Should have reasonable match
    
    def test_match_job_returns_score_and_reasons(self, job_matcher, sample_cv, sample_job):
        result = job_matcher.match_job(sample_cv, sample_job)
        
        assert "score" in result
        assert "reasons" in result
        assert isinstance(result["score"], float)
        assert isinstance(result["reasons"], list)
    
    @patch('src.services.job_matcher.openai_client')
    def test_generate_match_explanation(self, mock_openai, job_matcher):
        mock_openai.completions.create.return_value.choices[0].text = "Good match because..."
        
        explanation = job_matcher.generate_match_explanation(0.85, ["skill match"])
        assert isinstance(explanation, str)
        assert len(explanation) > 0
```

### 4. Integration Tests
```typescript
// tests/integration/job-application-flow.test.ts
import request from 'supertest';
import { MongoMemoryServer } from 'mongodb-memory-server';
import mongoose from 'mongoose';

describe('Job Application Flow Integration', () => {
  let mongoServer: MongoMemoryServer;
  let userToken: string;
  let jobId: string;
  let cvId: string;

  beforeAll(async () => {
    mongoServer = await MongoMemoryServer.create();
    await mongoose.connect(mongoServer.getUri());
  });

  afterAll(async () => {
    await mongoose.connection.dropDatabase();
    await mongoose.connection.close();
    await mongoServer.stop();
  });

  test('complete job application flow', async () => {
    // 1. Register user
    const registerResponse = await request(apiGateway)
      .post('/api/v1/auth/register')
      .send({
        email: 'test@example.com',
        password: 'password123',
        userType: 'seeker',
        firstName: 'Test',
        lastName: 'User'
      });

    userToken = registerResponse.body.data.token;

    // 2. Create CV
    const cvResponse = await request(apiGateway)
      .post('/api/v1/cv')
      .set('Authorization', `Bearer ${userToken}`)
      .send({
        name: 'Test CV',
        personalInfo: { firstName: 'Test', lastName: 'User' }
      });

    cvId = cvResponse.body.data.id;

    // 3. Search for jobs
    const jobsResponse = await request(apiGateway)
      .get('/api/v1/jobs?q=software+engineer')
      .set('Authorization', `Bearer ${userToken}`);

    jobId = jobsResponse.body.data.jobs[0].id;

    // 4. Apply to job
    const applicationResponse = await request(apiGateway)
      .post(`/api/v1/jobs/${jobId}/apply`)
      .set('Authorization', `Bearer ${userToken}`)
      .send({
        cvId,
        coverLetter: 'I am interested in this position...'
      });

    expect(applicationResponse.status).toBe(201);
    expect(applicationResponse.body.data.application).toBeDefined();

    // 5. Check application status
    const applicationsResponse = await request(apiGateway)
      .get('/api/v1/applications')
      .set('Authorization', `Bearer ${userToken}`);

    expect(applicationsResponse.body.data.applications).toHaveLength(1);
    expect(applicationsResponse.body.data.applications[0].status).toBe('pending');
  });
});
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Port Already in Use
```bash
# Find process using port
lsof -i :3000

# Kill process
kill -9 <PID>

# Or change port in .env
FRONTEND_PORT=3001
```

#### 2. Database Connection Issues
```bash
# Check if databases are running
docker ps | grep mongo
docker ps | grep postgres
docker ps | grep redis

# Check database logs
docker logs hr-agent-mongodb-dev
docker logs hr-agent-postgres-dev

# Reset databases
docker-compose down -v
docker-compose up -d mongodb postgres redis
```

#### 3. Node.js Module Issues
```bash
# Clear node_modules and package-lock.json
rm -rf node_modules package-lock.json
npm install

# Or in Docker
docker-compose build --no-cache frontend
docker-compose build --no-cache api-gateway
```

#### 4. Python Dependencies Issues
```bash
# Rebuild Python containers
docker-compose build --no-cache ai-service
docker-compose build --no-cache scraper-service

# Check Python version
docker-compose exec ai-service python --version
```

#### 5. Hot Reload Not Working
```bash
# Check file watchers limit (Linux)
cat /proc/sys/fs/inotify/max_user_watches
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf

# Enable polling for Docker volumes
export CHOKIDAR_USEPOLLING=true
```

#### 6. Environment Variables Not Loading
```bash
# Check if .env file exists
ls -la .env

# Verify environment variables in container
docker-compose exec frontend env | grep NEXT_PUBLIC
docker-compose exec api-gateway env | grep NODE_ENV
```

### Debugging Tools

#### 1. Service Health Checks
```bash
# scripts/health-check.sh
#!/bin/bash

services=("frontend:3000" "api-gateway:8000" "user-service:8001" "job-service:8002")

for service in "${services[@]}"; do
    name=$(echo $service | cut -d: -f1)
    port=$(echo $service | cut -d: -f2)
    
    if curl -f "http://localhost:$port/health" > /dev/null 2>&1; then
        echo "✅ $name is healthy"
    else
        echo "❌ $name is not responding"
    fi
done
```

#### 2. Database Connection Tests
```bash
# Test MongoDB connection
docker-compose exec mongodb mongosh --eval "db.adminCommand('ping')"

# Test PostgreSQL connection
docker-compose exec postgres psql -U hr_agent -d hr_agent_dev -c "SELECT 1;"

# Test Redis connection
docker-compose exec redis redis-cli ping
```

#### 3. Log Aggregation
```bash
# scripts/logs.sh
#!/bin/bash

# Follow logs for all services
docker-compose logs -f

# Follow logs for specific service
docker-compose logs -f $1

# Get logs for all services with timestamps
docker-compose logs -t

# Get recent logs
docker-compose logs --tail=100
```

This development setup provides a comprehensive environment for building, testing, and debugging the HR Agent platform efficiently.
