# Deployment Strategy

## Overview

This document outlines the comprehensive deployment strategy for the AI-Powered Job Seeker Platform, covering containerization, orchestration, CI/CD pipelines, monitoring, and scaling strategies.

## Table of Contents
1. [Infrastructure Architecture](#infrastructure-architecture)
2. [Containerization Strategy](#containerization-strategy)
3. [Kubernetes Deployment](#kubernetes-deployment)
4. [CI/CD Pipeline](#cicd-pipeline)
5. [Environment Management](#environment-management)
6. [Monitoring & Observability](#monitoring--observability)
7. [Security Implementation](#security-implementation)
8. [Scaling Strategy](#scaling-strategy)
9. [Disaster Recovery](#disaster-recovery)

## Infrastructure Architecture

### Cloud Provider Strategy
```
Multi-Cloud Architecture
├── Primary: AWS
│   ├── EKS (Kubernetes)
│   ├── RDS (PostgreSQL)
│   ├── ElastiCache (Redis)
│   ├── S3 (File Storage)
│   ├── CloudFront (CDN)
│   └── Route 53 (DNS)
├── Secondary: Google Cloud
│   ├── GKE (Backup Cluster)
│   ├── Cloud SQL
│   ├── Cloud Storage
│   └── Firebase Services
└── Edge: Cloudflare
    ├── DDoS Protection
    ├── WAF
    ├── Edge Caching
    └── Load Balancing
```

### Network Architecture
```
Production Network
├── VPC (10.0.0.0/16)
│   ├── Public Subnets
│   │   ├── NAT Gateways
│   │   ├── Load Balancers
│   │   └── Bastion Hosts
│   ├── Private Subnets
│   │   ├── Application Servers
│   │   ├── Kubernetes Nodes
│   │   └── Cache Layer
│   └── Database Subnets
│       ├── RDS Instances
│       ├── MongoDB Cluster
│       └── Analytics DB
├── Security Groups
│   ├── Web Tier (80, 443)
│   ├── App Tier (8000-8010)
│   ├── Cache Tier (6379)
│   └── DB Tier (5432, 27017)
└── Monitoring
    ├── VPC Flow Logs
    ├── CloudTrail
    └── Config Rules
```

## Containerization Strategy

### Docker Architecture
```
Container Strategy
├── Base Images
│   ├── node:18-alpine (Frontend/API)
│   ├── python:3.11-slim (AI Services)
│   ├── nginx:alpine (Reverse Proxy)
│   └── redis:7-alpine (Cache)
├── Multi-stage Builds
│   ├── Build Stage
│   ├── Test Stage
│   ├── Security Scan
│   └── Runtime Stage
├── Image Optimization
│   ├── Layer Caching
│   ├── Minimal Base Images
│   ├── Security Hardening
│   └── Size Optimization
└── Registry Strategy
    ├── ECR (Primary)
    ├── Harbor (Self-hosted)
    └── Docker Hub (Public)
```

### Dockerfile Examples

#### Frontend Dockerfile
```dockerfile
# Multi-stage build for React/Next.js frontend
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy built application
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

# Set ownership
RUN chown -R nextjs:nodejs /app
USER nextjs

EXPOSE 3000
ENV PORT 3000

CMD ["node", "server.js"]
```

#### Backend API Dockerfile
```dockerfile
# Multi-stage build for Node.js API
FROM node:18-alpine AS base
WORKDIR /app
RUN apk add --no-cache dumb-init

FROM base AS dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

FROM base AS build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM base AS runtime
ENV NODE_ENV=production
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package*.json ./

# Create non-root user
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001
RUN chown -R nextjs:nodejs /app
USER nextjs

EXPOSE 8000
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/app.js"]
```

#### AI Service Dockerfile
```dockerfile
# Multi-stage build for Python AI service
FROM python:3.11-slim AS base
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

FROM base AS dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM base AS runtime
COPY --from=dependencies /usr/local/lib/python3.11/site-packages /usr/local/lib/python3.11/site-packages
COPY --from=dependencies /usr/local/bin /usr/local/bin

# Copy application code
COPY . .

# Create non-root user
RUN useradd --create-home --shell /bin/bash app
RUN chown -R app:app /app
USER app

EXPOSE 8001
CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8001"]
```

## Kubernetes Deployment

### Cluster Architecture
```yaml
# cluster-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-config
data:
  cluster-name: "hr-agent-prod"
  region: "us-west-2"
  node-groups: |
    - name: general
      instance-types: ["t3.medium", "t3.large"]
      scaling:
        min: 2
        max: 10
        desired: 4
    - name: compute-intensive
      instance-types: ["c5.large", "c5.xlarge"]
      scaling:
        min: 1
        max: 5
        desired: 2
    - name: memory-intensive
      instance-types: ["r5.large", "r5.xlarge"]
      scaling:
        min: 1
        max: 3
        desired: 1
```

### Namespace Strategy
```yaml
# namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    environment: production
---
apiVersion: v1
kind: Namespace
metadata:
  name: staging
  labels:
    environment: staging
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    environment: development
---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
  labels:
    purpose: monitoring
---
apiVersion: v1
kind: Namespace
metadata:
  name: ingress
  labels:
    purpose: networking
```

### Application Deployments

#### Frontend Deployment
```yaml
# frontend-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: production
  labels:
    app: frontend
    tier: presentation
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: presentation
    spec:
      containers:
      - name: frontend
        image: your-registry/hr-agent-frontend:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: API_BASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: api-base-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
      imagePullSecrets:
      - name: registry-secret
---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: production
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
```

#### API Gateway Deployment
```yaml
# api-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: production
  labels:
    app: api-gateway
    tier: application
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
        tier: application
    spec:
      containers:
      - name: api-gateway
        image: your-registry/hr-agent-api-gateway:latest
        ports:
        - containerPort: 8000
        env:
        - name: NODE_ENV
          value: "production"
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-secret
              key: url
        - name: DB_CONNECTION_STRING
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: connection-string
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: production
spec:
  selector:
    app: api-gateway
  ports:
  - port: 80
    targetPort: 8000
  type: ClusterIP
```

#### Microservices Deployment
```yaml
# user-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: production
  labels:
    app: user-service
    tier: application
spec:
  replicas: 2
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
        tier: application
    spec:
      containers:
      - name: user-service
        image: your-registry/hr-agent-user-service:latest
        ports:
        - containerPort: 8001
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_CONNECTION_STRING
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: connection-string
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: jwt-secret
              key: secret
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8001
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8001
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: production
spec:
  selector:
    app: user-service
  ports:
  - port: 80
    targetPort: 8001
  type: ClusterIP
```

#### AI Service Deployment
```yaml
# ai-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-service
  namespace: production
  labels:
    app: ai-service
    tier: application
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ai-service
  template:
    metadata:
      labels:
        app: ai-service
        tier: application
    spec:
      containers:
      - name: ai-service
        image: your-registry/hr-agent-ai-service:latest
        ports:
        - containerPort: 8002
        env:
        - name: OPENAI_API_KEY
          valueFrom:
            secretKeyRef:
              name: ai-secrets
              key: openai-api-key
        - name: MODEL_PATH
          value: "/app/models"
        resources:
          requests:
            memory: "1Gi"
            cpu: "1000m"
            nvidia.com/gpu: 0
          limits:
            memory: "2Gi"
            cpu: "2000m"
            nvidia.com/gpu: 1
        volumeMounts:
        - name: model-storage
          mountPath: /app/models
        livenessProbe:
          httpGet:
            path: /health
            port: 8002
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8002
          initialDelaySeconds: 30
          periodSeconds: 10
      volumes:
      - name: model-storage
        persistentVolumeClaim:
          claimName: ai-models-pvc
      nodeSelector:
        node-type: gpu-enabled
---
apiVersion: v1
kind: Service
metadata:
  name: ai-service
  namespace: production
spec:
  selector:
    app: ai-service
  ports:
  - port: 80
    targetPort: 8002
  type: ClusterIP
```

### Ingress Configuration
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hr-agent-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  tls:
  - hosts:
    - api.hr-agent.com
    - app.hr-agent.com
    secretName: hr-agent-tls
  rules:
  - host: app.hr-agent.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.hr-agent.com
    http:
      paths:
      - path: /api/v1/auth
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /api/v1/users
        pathType: Prefix
        backend:
          service:
            name: user-service
            port:
              number: 80
      - path: /api/v1/jobs
        pathType: Prefix
        backend:
          service:
            name: job-service
            port:
              number: 80
      - path: /api/v1/cv
        pathType: Prefix
        backend:
          service:
            name: cv-service
            port:
              number: 80
      - path: /api/v1/agent
        pathType: Prefix
        backend:
          service:
            name: ai-service
            port:
              number: 80
      - path: /api/v1/.*
        pathType: Prefix
        backend:
          service:
            name: api-gateway-service
            port:
              number: 80
```

### Storage Configuration
```yaml
# storage.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ai-models-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp3
  resources:
    requests:
      storage: 50Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: file-storage-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs
  resources:
    requests:
      storage: 100Gi
```

## CI/CD Pipeline

### GitHub Actions Workflow
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x]
        service: [frontend, api-gateway, user-service, job-service, cv-service]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
        cache-dependency-path: ${{ matrix.service }}/package-lock.json
    
    - name: Install dependencies
      working-directory: ${{ matrix.service }}
      run: npm ci
    
    - name: Run tests
      working-directory: ${{ matrix.service }}
      run: npm test
    
    - name: Run security audit
      working-directory: ${{ matrix.service }}
      run: npm audit --audit-level moderate

  test-python:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.11]
        service: [ai-service, scraper-service]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v4
      with:
        python-version: ${{ matrix.python-version }}
    
    - name: Install dependencies
      working-directory: ${{ matrix.service }}
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install -r requirements-dev.txt
    
    - name: Run tests
      working-directory: ${{ matrix.service }}
      run: |
        pytest tests/ --cov=src --cov-report=xml
    
    - name: Security scan
      working-directory: ${{ matrix.service }}
      run: |
        bandit -r src/
        safety check

  build-and-push:
    needs: [test, test-python]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    strategy:
      matrix:
        service: [frontend, api-gateway, user-service, job-service, cv-service, ai-service, scraper-service]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-${{ matrix.service }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          type=raw,value=latest,enable={{is_default_branch}}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: ${{ matrix.service }}
        file: ${{ matrix.service }}/Dockerfile
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  security-scan:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    strategy:
      matrix:
        service: [frontend, api-gateway, user-service, job-service, cv-service, ai-service, scraper-service]
    
    steps:
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-${{ matrix.service }}:latest
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: Upload Trivy scan results to GitHub Security tab
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'

  deploy-staging:
    needs: [build-and-push, security-scan]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: staging
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2
    
    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region us-west-2 --name hr-agent-staging
    
    - name: Deploy to staging
      run: |
        # Update image tags in deployment files
        for service in frontend api-gateway user-service job-service cv-service ai-service scraper-service; do
          sed -i "s|your-registry/hr-agent-${service}:latest|${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-${service}:${{ github.sha }}|g" k8s/staging/${service}-deployment.yaml
        done
        
        # Apply configurations
        kubectl apply -f k8s/staging/ -n staging
        
        # Wait for rollout
        kubectl rollout status deployment --all -n staging --timeout=600s

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-west-2
    
    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region us-west-2 --name hr-agent-production
    
    - name: Deploy to production
      run: |
        # Update image tags in deployment files
        for service in frontend api-gateway user-service job-service cv-service ai-service scraper-service; do
          sed -i "s|your-registry/hr-agent-${service}:latest|${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-${service}:${{ github.sha }}|g" k8s/production/${service}-deployment.yaml
        done
        
        # Apply configurations
        kubectl apply -f k8s/production/ -n production
        
        # Wait for rollout
        kubectl rollout status deployment --all -n production --timeout=600s
    
    - name: Run smoke tests
      run: |
        # Wait for services to be ready
        sleep 30
        
        # Run basic health checks
        curl -f https://api.hr-agent.com/health || exit 1
        curl -f https://app.hr-agent.com/health || exit 1
    
    - name: Notify deployment
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        channel: '#deployments'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        fields: repo,message,commit,author,action,eventName,ref,workflow
      if: always()
```

### Helm Chart Deployment
```yaml
# helm/hr-agent/Chart.yaml
apiVersion: v2
name: hr-agent
description: AI-Powered Job Seeker Platform
version: 1.0.0
appVersion: "1.0.0"

dependencies:
- name: postgresql
  version: "12.x.x"
  repository: "https://charts.bitnami.com/bitnami"
  condition: postgresql.enabled
- name: redis
  version: "17.x.x"
  repository: "https://charts.bitnami.com/bitnami"
  condition: redis.enabled
- name: nginx-ingress
  version: "4.x.x"
  repository: "https://kubernetes.github.io/ingress-nginx"
  condition: ingress.enabled
```

```yaml
# helm/hr-agent/values.yaml
global:
  imageRegistry: "ghcr.io"
  imagePullSecrets:
    - name: registry-secret

replicaCount:
  frontend: 3
  apiGateway: 2
  userService: 2
  jobService: 2
  cvService: 2
  aiService: 2

image:
  repository: hr-agent
  tag: "latest"
  pullPolicy: Always

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
  hosts:
    - host: app.hr-agent.com
      paths:
        - path: /
          pathType: Prefix
          service: frontend
    - host: api.hr-agent.com
      paths:
        - path: /api/v1
          pathType: Prefix
          service: api-gateway
  tls:
    - secretName: hr-agent-tls
      hosts:
        - app.hr-agent.com
        - api.hr-agent.com

resources:
  frontend:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  apiGateway:
    requests:
      memory: "512Mi"
      cpu: "500m"
    limits:
      memory: "1Gi"
      cpu: "1000m"
  userService:
    requests:
      memory: "256Mi"
      cpu: "250m"
    limits:
      memory: "512Mi"
      cpu: "500m"
  aiService:
    requests:
      memory: "1Gi"
      cpu: "1000m"
    limits:
      memory: "2Gi"
      cpu: "2000m"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

postgresql:
  enabled: true
  auth:
    postgresPassword: "secure-password"
    database: "hr_agent"
  primary:
    persistence:
      enabled: true
      size: 20Gi

redis:
  enabled: true
  auth:
    enabled: true
    password: "redis-password"
  master:
    persistence:
      enabled: true
      size: 8Gi

monitoring:
  enabled: true
  prometheus:
    enabled: true
  grafana:
    enabled: true

secrets:
  jwt:
    secret: "your-jwt-secret"
  database:
    connectionString: "postgresql://user:pass@host:port/db"
  openai:
    apiKey: "your-openai-key"
```

## Environment Management

### Environment Configuration
```yaml
# environments/development.yaml
environment: development
namespace: development

replicas:
  frontend: 1
  backend: 1
  ai-service: 1

resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "200m"

database:
  type: "sqlite"
  host: "localhost"

external-services:
  enabled: false

---
# environments/staging.yaml
environment: staging
namespace: staging

replicas:
  frontend: 2
  backend: 2
  ai-service: 1

resources:
  requests:
    memory: "256Mi"
    cpu: "200m"
  limits:
    memory: "512Mi"
    cpu: "400m"

database:
  type: "postgresql"
  host: "staging-db.cluster.local"

external-services:
  enabled: true
  rate-limits:
    linkedin: 100/hour
    indeed: 200/hour

---
# environments/production.yaml
environment: production
namespace: production

replicas:
  frontend: 3
  backend: 3
  ai-service: 2

resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"

database:
  type: "postgresql"
  host: "prod-db.cluster.local"
  replicas: 3

external-services:
  enabled: true
  rate-limits:
    linkedin: 1000/hour
    indeed: 2000/hour
```

### Secret Management
```yaml
# secret-management.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
  namespace: production
spec:
  refreshInterval: 1m
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: app-secrets
    creationPolicy: Owner
  data:
  - secretKey: jwt-secret
    remoteRef:
      key: hr-agent/production
      property: jwt_secret
  - secretKey: database-url
    remoteRef:
      key: hr-agent/production
      property: database_url
  - secretKey: openai-api-key
    remoteRef:
      key: hr-agent/production
      property: openai_api_key

---
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-west-2
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
```

This deployment strategy provides a comprehensive, scalable, and secure foundation for the HR Agent platform with automated deployments, monitoring, and disaster recovery capabilities.
