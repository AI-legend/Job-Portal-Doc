# Technology Stack & Technical Specifications

## Overview

This document provides a comprehensive breakdown of the technology stack, technical specifications, and architectural decisions for the AI-Powered Job Seeker Platform. Each technology choice is justified with reasoning and alternatives considered.

## Table of Contents
1. [Frontend Technology Stack](#frontend-technology-stack)
2. [Backend Technology Stack](#backend-technology-stack)
3. [Database & Storage](#database--storage)
4. [AI & Machine Learning](#ai--machine-learning)
5. [Workflow Automation](#workflow-automation)
6. [Cloud Infrastructure](#cloud-infrastructure)
7. [Development & DevOps](#development--devops)
8. [Security & Compliance](#security--compliance)
9. [Monitoring & Analytics](#monitoring--analytics)
10. [Third-Party Integrations](#third-party-integrations)

## Frontend Technology Stack

### Core Framework: React with Next.js 14

#### React 18.2+
**Choice Reasoning:**
- **Component-Based Architecture**: Reusable UI components for consistency
- **Virtual DOM**: Optimal performance for complex interfaces
- **Rich Ecosystem**: Extensive library ecosystem and community support
- **Developer Experience**: Excellent debugging tools and development workflow

**Configuration:**
```javascript
// React Features Used
- React Hooks for state management
- Context API for global state
- Suspense for lazy loading
- Error Boundaries for error handling
- React.memo for performance optimization
```

#### Next.js 14 (App Router)
**Choice Reasoning:**
- **Full-Stack Framework**: API routes and serverless functions
- **Performance**: Automatic code splitting and optimization
- **SEO**: Server-side rendering and static generation
- **Developer Experience**: Hot reloading and TypeScript support

**Key Features:**
```javascript
// Next.js Configuration
- App Router for modern routing
- Server Components for performance
- API Routes for backend logic
- Image Optimization
- Automatic bundle optimization
- Built-in CSS support
```

### State Management: Redux Toolkit + Zustand

#### Redux Toolkit
**Use Cases:**
- Global application state
- Complex state interactions
- Time-travel debugging
- Middleware for async operations

```javascript
// Redux Store Structure
store/
├── slices/
│   ├── authSlice.js
│   ├── userSlice.js
│   ├── jobsSlice.js
│   ├── cvSlice.js
│   └── aiSlice.js
├── middleware/
│   ├── authMiddleware.js
│   └── loggingMiddleware.js
└── store.js
```

#### Zustand
**Use Cases:**
- Component-level state
- UI state management
- Simple state without complexity
- Performance-critical components

### Styling & UI: Tailwind CSS + Headless UI

#### Tailwind CSS 3.4+
**Choice Reasoning:**
- **Utility-First**: Rapid UI development
- **Consistency**: Design system enforcement
- **Performance**: Purging unused CSS
- **Customization**: Easy theme customization

**Configuration:**
```javascript
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

#### Headless UI
**Features:**
- Accessible components out of the box
- Unstyled components for customization
- Keyboard navigation support
- Screen reader compatibility

### Additional Frontend Libraries

#### Form Management: React Hook Form + Zod
```javascript
// Form validation with Zod
const userSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2),
});

// React Hook Form integration
const { register, handleSubmit, formState: { errors } } = useForm({
  resolver: zodResolver(userSchema),
});
```

#### Data Fetching: TanStack Query (React Query)
```javascript
// API state management
const { data, isLoading, error } = useQuery({
  queryKey: ['jobs', filters],
  queryFn: () => fetchJobs(filters),
  staleTime: 5 * 60 * 1000, // 5 minutes
});
```

#### Animation: Framer Motion
```javascript
// Smooth animations and transitions
const variants = {
  hidden: { opacity: 0, y: 20 },
  visible: { opacity: 1, y: 0 },
};
```

#### Charts & Visualization: Recharts + D3.js
- **Recharts**: Simple charts and graphs
- **D3.js**: Complex data visualizations
- **React Spring**: Advanced animations

## Backend Technology Stack

### Primary Backend: Node.js with Express.js

#### Node.js 20 LTS
**Choice Reasoning:**
- **JavaScript Ecosystem**: Shared language with frontend
- **Performance**: V8 engine optimization
- **Scalability**: Event-driven, non-blocking I/O
- **Package Ecosystem**: Extensive NPM ecosystem

#### Express.js 4.18+
**Features:**
- **Middleware Architecture**: Flexible request processing
- **Routing**: Clean API route organization
- **Performance**: Fast and minimalist framework
- **Community**: Large ecosystem and community support

```javascript
// Express.js API Structure
api/
├── routes/
│   ├── auth.js
│   ├── users.js
│   ├── jobs.js
│   ├── cv.js
│   └── applications.js
├── middleware/
│   ├── auth.js
│   ├── validation.js
│   ├── rateLimit.js
│   └── errorHandler.js
├── controllers/
│   ├── authController.js
│   ├── userController.js
│   └── jobController.js
└── services/
    ├── authService.js
    ├── emailService.js
    └── aiService.js
```

### AI/ML Backend: Python with FastAPI

#### Python 3.11+
**Choice Reasoning:**
- **AI/ML Libraries**: TensorFlow, PyTorch, scikit-learn
- **Natural Language Processing**: NLTK, spaCy, transformers
- **Data Processing**: Pandas, NumPy for data manipulation
- **Integration**: Easy integration with AI APIs

#### FastAPI 0.104+
**Features:**
- **Performance**: High performance, on par with Node.js
- **Type Hints**: Automatic API documentation
- **Async Support**: Native async/await support
- **Validation**: Automatic request/response validation

```python
# FastAPI AI Service Structure
ai_service/
├── routers/
│   ├── cv_analysis.py
│   ├── job_matching.py
│   ├── chat.py
│   └── recommendations.py
├── models/
│   ├── cv_model.py
│   ├── job_model.py
│   └── chat_model.py
├── services/
│   ├── openai_service.py
│   ├── claude_service.py
│   └── nlp_service.py
└── utils/
    ├── text_processing.py
    ├── embeddings.py
    └── similarity.py
```

### API Gateway & Microservices

#### API Gateway (Express.js)
```javascript
// API Gateway Configuration
const gateway = {
  routes: {
    '/api/auth/*': 'http://auth-service:3001',
    '/api/users/*': 'http://user-service:3002',
    '/api/jobs/*': 'http://job-service:3003',
    '/api/ai/*': 'http://ai-service:8000',
    '/api/workflows/*': 'http://n8n-service:5678',
  },
  middleware: [
    rateLimit,
    cors,
    authentication,
    logging,
  ],
};
```

#### Microservices Architecture
```
Microservices
├── api-gateway (Node.js/Express)
├── auth-service (Node.js/Express)
├── user-service (Node.js/Express)
├── job-service (Node.js/Express)
├── cv-service (Node.js/Express)
├── ai-service (Python/FastAPI)
├── notification-service (Node.js/Express)
└── analytics-service (Python/FastAPI)
```

## Database & Storage

### Primary Database: MongoDB 7.0

#### MongoDB Atlas
**Choice Reasoning:**
- **Flexible Schema**: Easy to evolve data models
- **JSON Documents**: Natural fit for JavaScript/Node.js
- **Scalability**: Horizontal scaling with sharding
- **Performance**: Optimized for read-heavy workloads

**Collections Structure:**
```javascript
// MongoDB Collections
{
  users: {
    _id: ObjectId,
    email: String,
    profile: {
      name: String,
      role: "seeker" | "recruiter",
      preferences: Object,
    },
    created_at: Date,
    updated_at: Date,
  },
  jobs: {
    _id: ObjectId,
    title: String,
    company: String,
    description: String,
    requirements: [String],
    location: {
      type: "Point",
      coordinates: [Number, Number],
    },
    posted_by: ObjectId,
    posted_at: Date,
  },
  cvs: {
    _id: ObjectId,
    user_id: ObjectId,
    title: String,
    sections: {
      personal: Object,
      experience: [Object],
      education: [Object],
      skills: [String],
    },
    template_id: String,
    version: Number,
    created_at: Date,
  },
}
```

### Analytics Database: PostgreSQL 15

#### PostgreSQL for Analytics
**Choice Reasoning:**
- **ACID Compliance**: Data integrity for analytics
- **Complex Queries**: Advanced SQL for reporting
- **JSON Support**: Hybrid document-relational capabilities
- **Time-Series**: Efficient for time-based analytics

**Schema Structure:**
```sql
-- Analytics Tables
CREATE TABLE user_events (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    event_type VARCHAR(50) NOT NULL,
    event_data JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE job_applications (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    job_id UUID NOT NULL,
    status VARCHAR(20) NOT NULL,
    applied_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE ai_interactions (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL,
    query TEXT NOT NULL,
    response TEXT NOT NULL,
    tokens_used INTEGER,
    response_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT NOW()
);
```

### Caching: Redis 7.2

#### Redis Configuration
**Use Cases:**
- **Session Storage**: User sessions and JWT tokens
- **API Caching**: Frequently accessed data
- **Job Queue**: Background task processing
- **Real-time Data**: Chat messages and notifications

```javascript
// Redis Usage Patterns
const redisConfig = {
  sessions: {
    keyPattern: 'session:${userId}',
    ttl: 24 * 60 * 60, // 24 hours
  },
  apiCache: {
    keyPattern: 'api:${endpoint}:${params}',
    ttl: 5 * 60, // 5 minutes
  },
  jobQueue: {
    keyPattern: 'job:${type}:${id}',
    ttl: null, // persistent
  },
};
```

### File Storage: AWS S3 + CloudFront

#### AWS S3 Buckets
```javascript
// S3 Bucket Structure
buckets: {
  'hr-platform-cvs': {
    purpose: 'CV documents and PDFs',
    versioning: true,
    encryption: 'AES256',
  },
  'hr-platform-assets': {
    purpose: 'Profile pictures and company logos',
    cdn: 'CloudFront',
    optimization: 'automatic',
  },
  'hr-platform-backups': {
    purpose: 'Database backups',
    retention: '90 days',
    storage_class: 'GLACIER',
  },
}
```

## AI & Machine Learning

### Large Language Models

#### OpenAI GPT-4 Turbo
**Use Cases:**
- **Conversational AI**: Chat interface and user assistance
- **CV Optimization**: Content improvement and suggestions
- **Job Description Analysis**: Requirement extraction and parsing
- **Interview Preparation**: Question generation and feedback

```python
# OpenAI Integration
import openai

class OpenAIService:
    def __init__(self):
        self.client = openai.OpenAI(api_key=os.getenv('OPENAI_API_KEY'))
    
    async def analyze_cv(self, cv_text: str) -> dict:
        response = await self.client.chat.completions.create(
            model="gpt-4-turbo-preview",
            messages=[
                {"role": "system", "content": "You are a CV analysis expert..."},
                {"role": "user", "content": cv_text}
            ],
            temperature=0.3,
            max_tokens=1500
        )
        return self.parse_cv_analysis(response.choices[0].message.content)
```

#### Anthropic Claude 3
**Use Cases:**
- **Document Analysis**: Long document processing and analysis
- **Code Review**: Technical skill assessment and code analysis
- **Complex Reasoning**: Multi-step problem solving and recommendations
- **Safety-Critical Tasks**: Content moderation and sensitive data handling

```python
# Claude Integration
import anthropic

class ClaudeService:
    def __init__(self):
        self.client = anthropic.Anthropic(api_key=os.getenv('ANTHROPIC_API_KEY'))
    
    async def match_jobs(self, cv_data: dict, job_data: dict) -> dict:
        prompt = f"Analyze job match between CV and job posting..."
        response = await self.client.messages.create(
            model="claude-3-opus-20240229",
            max_tokens=2000,
            messages=[{"role": "user", "content": prompt}]
        )
        return self.parse_match_analysis(response.content)
```

### Machine Learning Stack

#### Embeddings & Vector Search
```python
# Vector Database for Semantic Search
from sentence_transformers import SentenceTransformer
import chromadb

class EmbeddingService:
    def __init__(self):
        self.model = SentenceTransformer('all-MiniLM-L6-v2')
        self.client = chromadb.Client()
        self.collection = self.client.create_collection("job_embeddings")
    
    def create_job_embedding(self, job_description: str) -> list:
        return self.model.encode(job_description).tolist()
    
    def find_similar_jobs(self, cv_embedding: list, top_k: int = 10) -> list:
        results = self.collection.query(
            query_embeddings=[cv_embedding],
            n_results=top_k
        )
        return results
```

#### Skill Extraction & NLP
```python
# Natural Language Processing Pipeline
import spacy
from transformers import pipeline

class NLPService:
    def __init__(self):
        self.nlp = spacy.load("en_core_web_sm")
        self.skill_extractor = pipeline(
            "ner",
            model="dbmdz/bert-large-cased-finetuned-conll03-english"
        )
    
    def extract_skills(self, text: str) -> list:
        doc = self.nlp(text)
        entities = self.skill_extractor(text)
        
        skills = []
        for ent in entities:
            if ent['entity'].startswith('B-') or ent['entity'].startswith('I-'):
                skills.append(ent['word'])
        
        return list(set(skills))
```

### AI Model Management

#### Model Versioning & Deployment
```python
# MLOps Pipeline
from mlflow import log_model, register_model
import joblib

class ModelManager:
    def __init__(self):
        self.models = {}
        self.model_versions = {}
    
    def deploy_model(self, model_name: str, model_path: str):
        model = joblib.load(model_path)
        self.models[model_name] = model
        
        # Log to MLflow
        log_model(model, model_name)
        register_model(f"runs:/{run_id}/{model_name}", model_name)
    
    def predict(self, model_name: str, features: dict) -> dict:
        model = self.models.get(model_name)
        if not model:
            raise ValueError(f"Model {model_name} not found")
        
        prediction = model.predict([features])
        confidence = model.predict_proba([features]).max()
        
        return {
            'prediction': prediction[0],
            'confidence': confidence,
            'model_version': self.model_versions[model_name]
        }
```

## Workflow Automation

### n8n Workflow Engine

#### n8n Setup & Configuration
```javascript
// n8n Docker Configuration
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
      - WEBHOOK_URL=https://hooks.yourdomain.com/
      - GENERIC_TIMEZONE=UTC
    volumes:
      - n8n_data:/home/node/.n8n
      - ./n8n/workflows:/home/node/.n8n/workflows
    ports:
      - "5678:5678"
```

#### Custom n8n Nodes
```javascript
// Custom Job Scraping Node
class JobScrapingNode implements INodeType {
    description: INodeTypeDescription = {
        displayName: 'Job Scraper',
        name: 'jobScraper',
        group: ['transform'],
        version: 1,
        description: 'Scrape job listings from various job boards',
        defaults: {
            name: 'Job Scraper',
        },
        inputs: ['main'],
        outputs: ['main'],
        properties: [
            {
                displayName: 'Job Board',
                name: 'jobBoard',
                type: 'options',
                options: [
                    { name: 'LinkedIn', value: 'linkedin' },
                    { name: 'Indeed', value: 'indeed' },
                    { name: 'Glassdoor', value: 'glassdoor' },
                ],
                default: 'linkedin',
            },
        ],
    };

    async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
        const items = this.getInputData();
        const jobBoard = this.getNodeParameter('jobBoard', 0) as string;
        
        const results = await this.scrapeJobs(jobBoard, items[0].json);
        
        return [this.helpers.returnJsonArray(results)];
    }
}
```

#### Workflow Templates
```yaml
# Job Scraping Workflow
workflows:
  job_scraping:
    trigger: schedule(0 */6 * * *) # Every 6 hours
    steps:
      - name: linkedin_scraper
        node: jobScraper
        config:
          jobBoard: linkedin
          searchTerms: ${env.SEARCH_TERMS}
      
      - name: process_jobs
        node: dataProcessor
        config:
          deduplicate: true
          normalize: true
      
      - name: save_to_database
        node: mongodb
        config:
          operation: insert
          collection: jobs

  auto_apply:
    trigger: webhook
    steps:
      - name: validate_application
        node: validator
        config:
          rules: ${env.APPLICATION_RULES}
      
      - name: submit_application
        node: jobBoardAPI
        config:
          platform: ${input.platform}
          credentials: ${env.CREDENTIALS}
      
      - name: track_application
        node: mongodb
        config:
          operation: update
          collection: applications
```

## Cloud Infrastructure

### Multi-Cloud Strategy

#### AWS Primary Cloud
```yaml
# AWS Infrastructure
Region: us-east-1 (primary), us-west-2 (secondary)

Compute:
  EKS Cluster:
    - Node Groups: t3.medium (3-10 nodes)
    - Auto Scaling: CPU/Memory based
    - Network: VPC with private/public subnets
  
  Lambda Functions:
    - Image processing
    - Webhook handlers
    - Scheduled tasks

Storage:
  S3 Buckets:
    - Application assets
    - CV documents
    - Database backups
  
  EFS:
    - Shared storage for containers
    - Workflow templates

Database:
  RDS PostgreSQL:
    - Multi-AZ deployment
    - Read replicas for analytics
    - Automated backups
  
  ElastiCache Redis:
    - Cluster mode for scaling
    - Automatic failover

Networking:
  CloudFront CDN:
    - Global content delivery
    - API caching
  
  Application Load Balancer:
    - SSL termination
    - Health checks
    - Traffic distribution
```

#### GCP Secondary Cloud
```yaml
# GCP Infrastructure (Disaster Recovery)
Region: us-central1

Compute:
  GKE Cluster:
    - Node Pools: e2-standard-2
    - Workload Identity enabled
  
  Cloud Functions:
    - Event-driven processing
    - Backup operations

Storage:
  Cloud Storage:
    - Multi-regional buckets
    - Lifecycle policies
  
  Cloud SQL:
    - PostgreSQL with HA
    - Automated backups

AI/ML:
  Vertex AI:
    - Model deployment
    - Batch predictions
  
  Cloud Natural Language:
    - Text analysis
    - Entity extraction
```

### Kubernetes Configuration

#### Kubernetes Manifests
```yaml
# API Gateway Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: hr-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: hr-platform/api-gateway:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: MONGODB_URI
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: mongodb-uri
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10

---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: hr-platform
spec:
  selector:
    app: api-gateway
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
```

#### Helm Charts
```yaml
# values.yaml for Helm deployment
global:
  imageRegistry: your-registry.com
  imagePullSecrets:
    - hr-platform-registry

services:
  apiGateway:
    enabled: true
    replicas: 3
    image:
      repository: hr-platform/api-gateway
      tag: "latest"
    resources:
      requests:
        memory: 256Mi
        cpu: 250m
      limits:
        memory: 512Mi
        cpu: 500m
  
  userService:
    enabled: true
    replicas: 2
    image:
      repository: hr-platform/user-service
      tag: "latest"
  
  aiService:
    enabled: true
    replicas: 2
    image:
      repository: hr-platform/ai-service
      tag: "latest"
    resources:
      requests:
        memory: 1Gi
        cpu: 500m
      limits:
        memory: 2Gi
        cpu: 1000m

databases:
  mongodb:
    enabled: true
    auth:
      enabled: true
      existingSecret: mongodb-auth
    persistence:
      enabled: true
      size: 100Gi
  
  postgresql:
    enabled: true
    auth:
      existingSecret: postgresql-auth
    primary:
      persistence:
        enabled: true
        size: 50Gi
  
  redis:
    enabled: true
    auth:
      enabled: true
      existingSecret: redis-auth
    master:
      persistence:
        enabled: true
        size: 10Gi
```

## Development & DevOps

### Development Environment

#### Docker Development Setup
```dockerfile
# Frontend Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/next.config.js ./
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]
```

```dockerfile
# Backend API Dockerfile
FROM node:20-alpine
WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

USER node
EXPOSE 3000

CMD ["npm", "start"]
```

```dockerfile
# AI Service Dockerfile
FROM python:3.11-slim
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

#### Docker Compose for Development
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NEXT_PUBLIC_API_URL=http://api-gateway:3001
    depends_on:
      - api-gateway

  api-gateway:
    build:
      context: ./backend/api-gateway
      dockerfile: Dockerfile.dev
    ports:
      - "3001:3001"
    volumes:
      - ./backend/api-gateway:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://mongodb:27017/hr_platform
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis

  user-service:
    build:
      context: ./backend/user-service
      dockerfile: Dockerfile.dev
    ports:
      - "3002:3002"
    volumes:
      - ./backend/user-service:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - MONGODB_URI=mongodb://mongodb:27017/hr_platform

  ai-service:
    build:
      context: ./backend/ai-service
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      - ./backend/ai-service:/app
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}

  mongodb:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password123

  postgresql:
    image: postgres:15
    ports:
      - "5432:5432"
    volumes:
      - postgresql_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=hr_analytics
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password123

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  n8n:
    image: n8nio/n8n:latest
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/.n8n/workflows
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=password123
      - WEBHOOK_URL=http://localhost:5678/

volumes:
  mongodb_data:
  postgresql_data:
  redis_data:
  n8n_data:
```

### CI/CD Pipeline

#### GitHub Actions Workflow
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
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
        service: [frontend, api-gateway, user-service, ai-service]
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      if: matrix.service != 'ai-service'
      uses: actions/setup-node@v4
      with:
        node-version: '20'
        cache: 'npm'
        cache-dependency-path: ${{ matrix.service }}/package-lock.json
    
    - name: Setup Python
      if: matrix.service == 'ai-service'
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
        cache: 'pip'
    
    - name: Install dependencies (Node.js)
      if: matrix.service != 'ai-service'
      run: |
        cd ${{ matrix.service }}
        npm ci
    
    - name: Install dependencies (Python)
      if: matrix.service == 'ai-service'
      run: |
        cd ${{ matrix.service }}
        pip install -r requirements.txt
    
    - name: Run tests (Node.js)
      if: matrix.service != 'ai-service'
      run: |
        cd ${{ matrix.service }}
        npm test
    
    - name: Run tests (Python)
      if: matrix.service == 'ai-service'
      run: |
        cd ${{ matrix.service }}
        pytest
    
    - name: Run linting
      run: |
        cd ${{ matrix.service }}
        if [ "${{ matrix.service }}" = "ai-service" ]; then
          black --check .
          flake8 .
        else
          npm run lint
        fi
    
    - name: Security scan
      uses: securecodewarrior/github-action-add-sarif@v1
      with:
        sarif-file: security-scan.sarif

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Build and push Docker images
      run: |
        services=("frontend" "api-gateway" "user-service" "ai-service")
        for service in "${services[@]}"; do
          docker build -t ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${service}:${{ github.sha }} ./${service}
          docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${service}:${{ github.sha }}
          docker tag ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${service}:${{ github.sha }} ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${service}:latest
          docker push ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${service}:latest
        done

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: staging
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBE_CONFIG }}
    
    - name: Deploy to staging
      run: |
        helm upgrade --install hr-platform-staging ./helm/hr-platform \
          --namespace staging \
          --create-namespace \
          --set global.imageTag=${{ github.sha }} \
          --set global.environment=staging \
          --values ./helm/hr-platform/values-staging.yaml

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
    - uses: actions/checkout@v4
    
    - name: Configure kubectl
      uses: azure/k8s-set-context@v3
      with:
        method: kubeconfig
        kubeconfig: ${{ secrets.KUBE_CONFIG_PROD }}
    
    - name: Deploy to production
      run: |
        helm upgrade --install hr-platform ./helm/hr-platform \
          --namespace production \
          --create-namespace \
          --set global.imageTag=${{ github.sha }} \
          --set global.environment=production \
          --values ./helm/hr-platform/values-production.yaml \
          --wait \
          --timeout=10m
```

### Code Quality & Testing

#### ESLint Configuration
```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'next/core-web-vitals',
    '@typescript-eslint/recommended',
    'prettier',
  ],
  plugins: ['@typescript-eslint'],
  rules: {
    '@typescript-eslint/no-unused-vars': 'error',
    '@typescript-eslint/explicit-function-return-type': 'warn',
    'no-console': 'warn',
    'prefer-const': 'error',
  },
  overrides: [
    {
      files: ['**/*.test.ts', '**/*.test.tsx'],
      env: {
        jest: true,
      },
    },
  ],
};
```

#### Jest Testing Configuration
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapping: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/pages/_app.tsx',
    '!src/pages/_document.tsx',
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

## Security & Compliance

### Authentication & Authorization

#### JWT Token Management
```javascript
// JWT Service
import jwt from 'jsonwebtoken';
import { promisify } from 'util';

class JWTService {
  constructor() {
    this.secret = process.env.JWT_SECRET;
    this.refreshSecret = process.env.JWT_REFRESH_SECRET;
    this.accessTokenExpiry = '15m';
    this.refreshTokenExpiry = '7d';
  }

  generateTokens(payload) {
    const accessToken = jwt.sign(payload, this.secret, {
      expiresIn: this.accessTokenExpiry,
    });
    
    const refreshToken = jwt.sign(payload, this.refreshSecret, {
      expiresIn: this.refreshTokenExpiry,
    });

    return { accessToken, refreshToken };
  }

  async verifyAccessToken(token) {
    try {
      const verify = promisify(jwt.verify);
      return await verify(token, this.secret);
    } catch (error) {
      throw new Error('Invalid access token');
    }
  }

  async verifyRefreshToken(token) {
    try {
      const verify = promisify(jwt.verify);
      return await verify(token, this.refreshSecret);
    } catch (error) {
      throw new Error('Invalid refresh token');
    }
  }
}
```

#### Role-Based Access Control (RBAC)
```javascript
// RBAC Middleware
const permissions = {
  seeker: [
    'profile:read',
    'profile:write',
    'cv:read',
    'cv:write',
    'jobs:read',
    'applications:read',
    'applications:write',
  ],
  recruiter: [
    'profile:read',
    'profile:write',
    'jobs:read',
    'jobs:write',
    'applications:read',
    'candidates:read',
  ],
  admin: [
    '*', // All permissions
  ],
};

function hasPermission(userRole, requiredPermission) {
  const userPermissions = permissions[userRole] || [];
  return userPermissions.includes('*') || userPermissions.includes(requiredPermission);
}

function requirePermission(permission) {
  return (req, res, next) => {
    const { role } = req.user;
    
    if (!hasPermission(role, permission)) {
      return res.status(403).json({
        error: 'Insufficient permissions',
        required: permission,
      });
    }
    
    next();
  };
}
```

### Data Protection & Privacy

#### GDPR Compliance
```javascript
// Data Protection Service
class DataProtectionService {
  async anonymizeUser(userId) {
    // Anonymize personal data while keeping analytics
    const anonymizedData = {
      email: `anonymized_${userId}@example.com`,
      name: 'Anonymized User',
      phone: null,
      address: null,
    };
    
    await User.findByIdAndUpdate(userId, anonymizedData);
    await this.updateRelatedData(userId, anonymizedData);
  }

  async deleteUserData(userId) {
    // Complete data deletion
    await Promise.all([
      User.findByIdAndDelete(userId),
      CV.deleteMany({ userId }),
      Application.deleteMany({ userId }),
      UserEvent.deleteMany({ userId }),
      this.deleteStoredFiles(userId),
    ]);
  }

  async exportUserData(userId) {
    // Export all user data for GDPR compliance
    const userData = await User.findById(userId);
    const cvs = await CV.find({ userId });
    const applications = await Application.find({ userId });
    const events = await UserEvent.find({ userId });

    return {
      profile: userData,
      cvs,
      applications,
      events,
      exportDate: new Date().toISOString(),
    };
  }
}
```

#### Encryption & Data Security
```javascript
// Encryption Service
import crypto from 'crypto';

class EncryptionService {
  constructor() {
    this.algorithm = 'aes-256-gcm';
    this.secretKey = process.env.ENCRYPTION_KEY;
  }

  encrypt(text) {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher(this.algorithm, this.secretKey);
    cipher.setAAD(Buffer.from('hr-platform', 'utf8'));
    
    let encrypted = cipher.update(text, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      encrypted,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex'),
    };
  }

  decrypt(encryptedData) {
    const { encrypted, iv, authTag } = encryptedData;
    
    const decipher = crypto.createDecipher(this.algorithm, this.secretKey);
    decipher.setAAD(Buffer.from('hr-platform', 'utf8'));
    decipher.setAuthTag(Buffer.from(authTag, 'hex'));
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }
}
```

## Monitoring & Analytics

### Application Monitoring

#### Prometheus & Grafana Setup
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api-gateway'
    static_configs:
      - targets: ['api-gateway:3000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'user-service'
    static_configs:
      - targets: ['user-service:3002']

  - job_name: 'ai-service'
    static_configs:
      - targets: ['ai-service:8000']

  - job_name: 'mongodb-exporter'
    static_configs:
      - targets: ['mongodb-exporter:9216']

  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

#### Custom Metrics
```javascript
// Metrics Service
import client from 'prom-client';

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5],
});

const activeUsers = new client.Gauge({
  name: 'active_users_total',
  help: 'Number of active users',
  labelNames: ['user_type'],
});

const jobApplications = new client.Counter({
  name: 'job_applications_total',
  help: 'Total number of job applications',
  labelNames: ['application_type', 'status'],
});

const aiRequestsTotal = new client.Counter({
  name: 'ai_requests_total',
  help: 'Total number of AI service requests',
  labelNames: ['model', 'operation'],
});

const aiRequestDuration = new client.Histogram({
  name: 'ai_request_duration_seconds',
  help: 'Duration of AI requests in seconds',
  labelNames: ['model', 'operation'],
  buckets: [0.5, 1, 2, 5, 10, 30],
});

class MetricsService {
  static recordHttpRequest(method, route, statusCode, duration) {
    httpRequestDuration
      .labels(method, route, statusCode)
      .observe(duration);
  }

  static updateActiveUsers(userType, count) {
    activeUsers.labels(userType).set(count);
  }

  static incrementJobApplications(type, status) {
    jobApplications.labels(type, status).inc();
  }

  static recordAIRequest(model, operation, duration) {
    aiRequestsTotal.labels(model, operation).inc();
    aiRequestDuration.labels(model, operation).observe(duration);
  }
}
```

#### Logging Strategy
```javascript
// Winston Logger Configuration
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: process.env.SERVICE_NAME || 'hr-platform',
    environment: process.env.NODE_ENV || 'development',
  },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' }),
  ],
});

// Console transport for development
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// ELK Stack transport for production
if (process.env.NODE_ENV === 'production') {
  logger.add(new winston.transports.Http({
    host: process.env.LOGSTASH_HOST,
    port: process.env.LOGSTASH_PORT,
  }));
}

export default logger;
```

### Error Tracking & Alerting

#### Sentry Integration
```javascript
// Sentry Configuration
import * as Sentry from '@sentry/node';
import * as Tracing from '@sentry/tracing';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Tracing.Integrations.Express({ app }),
    new Tracing.Integrations.Mongo(),
  ],
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  beforeSend(event) {
    // Filter out sensitive data
    if (event.request) {
      delete event.request.headers?.authorization;
      delete event.request.headers?.cookie;
    }
    return event;
  },
});

// Error handling middleware
function errorHandler(err, req, res, next) {
  Sentry.captureException(err);
  
  logger.error('Unhandled error', {
    error: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    userId: req.user?.id,
  });

  res.status(500).json({
    error: 'Internal server error',
    requestId: req.id,
  });
}
```

## Third-Party Integrations

### Job Board APIs

#### LinkedIn Jobs API
```javascript
// LinkedIn API Service
class LinkedInService {
  constructor() {
    this.clientId = process.env.LINKEDIN_CLIENT_ID;
    this.clientSecret = process.env.LINKEDIN_CLIENT_SECRET;
    this.baseURL = 'https://api.linkedin.com/v2';
  }

  async searchJobs(criteria) {
    const params = new URLSearchParams({
      keywords: criteria.keywords,
      locationId: criteria.locationId,
      f_TPR: criteria.timePosted || 'r86400', // Last 24 hours
      f_JT: criteria.jobType || 'F', // Full-time
      start: criteria.start || 0,
      count: criteria.count || 25,
    });

    const response = await fetch(
      `${this.baseURL}/jobSearch?${params}`,
      {
        headers: {
          'Authorization': `Bearer ${await this.getAccessToken()}`,
          'X-Restli-Protocol-Version': '2.0.0',
        },
      }
    );

    return this.normalizeJobData(await response.json());
  }

  async getJobDetails(jobId) {
    const response = await fetch(
      `${this.baseURL}/jobs/${jobId}`,
      {
        headers: {
          'Authorization': `Bearer ${await this.getAccessToken()}`,
          'X-Restli-Protocol-Version': '2.0.0',
        },
      }
    );

    return this.normalizeJobData(await response.json());
  }

  normalizeJobData(rawData) {
    return {
      id: rawData.id,
      title: rawData.title,
      company: rawData.companyDetails?.name,
      location: rawData.formattedLocation,
      description: rawData.description?.text,
      postedAt: new Date(rawData.listedAt),
      url: rawData.applyUrl,
      requirements: this.extractRequirements(rawData.description?.text),
      salaryRange: rawData.salaryInsights,
      source: 'linkedin',
    };
  }
}
```

#### Indeed Web Scraping
```python
# Indeed Scraper
import asyncio
import aiohttp
from bs4 import BeautifulSoup
from urllib.parse import urlencode

class IndeedScraper:
    def __init__(self):
        self.base_url = "https://www.indeed.com/jobs"
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        }
        
    async def search_jobs(self, criteria):
        params = {
            'q': criteria.get('keywords', ''),
            'l': criteria.get('location', ''),
            'fromage': criteria.get('days_ago', 1),
            'start': criteria.get('start', 0),
            'limit': criteria.get('limit', 50)
        }
        
        url = f"{self.base_url}?{urlencode(params)}"
        
        async with aiohttp.ClientSession() as session:
            async with session.get(url, headers=self.headers) as response:
                html = await response.text()
                return self.parse_job_listings(html)
    
    def parse_job_listings(self, html):
        soup = BeautifulSoup(html, 'html.parser')
        job_cards = soup.find_all('div', {'data-jk': True})
        
        jobs = []
        for card in job_cards:
            job = self.extract_job_data(card)
            if job:
                jobs.append(job)
        
        return jobs
    
    def extract_job_data(self, card):
        try:
            title_elem = card.find('h2', class_='jobTitle')
            title = title_elem.find('a').get_text(strip=True) if title_elem else None
            
            company_elem = card.find('span', class_='companyName')
            company = company_elem.get_text(strip=True) if company_elem else None
            
            location_elem = card.find('div', class_='companyLocation')
            location = location_elem.get_text(strip=True) if location_elem else None
            
            summary_elem = card.find('div', class_='job-snippet')
            summary = summary_elem.get_text(strip=True) if summary_elem else None
            
            return {
                'id': card.get('data-jk'),
                'title': title,
                'company': company,
                'location': location,
                'summary': summary,
                'source': 'indeed',
                'scraped_at': datetime.utcnow().isoformat()
            }
        except Exception as e:
            logger.error(f"Error extracting job data: {e}")
            return None
```

### Communication Services

#### Email Service (SendGrid)
```javascript
// Email Service
import sgMail from '@sendgrid/mail';

class EmailService {
  constructor() {
    sgMail.setApiKey(process.env.SENDGRID_API_KEY);
    this.fromEmail = process.env.FROM_EMAIL;
  }

  async sendWelcomeEmail(user) {
    const msg = {
      to: user.email,
      from: this.fromEmail,
      templateId: 'd-welcome-template-id',
      dynamicTemplateData: {
        name: user.name,
        loginUrl: `${process.env.FRONTEND_URL}/login`,
      },
    };

    try {
      await sgMail.send(msg);
      logger.info('Welcome email sent', { userId: user.id, email: user.email });
    } catch (error) {
      logger.error('Failed to send welcome email', { error, userId: user.id });
      throw error;
    }
  }

  async sendJobAlert(user, jobs) {
    const msg = {
      to: user.email,
      from: this.fromEmail,
      templateId: 'd-job-alert-template-id',
      dynamicTemplateData: {
        name: user.name,
        jobs: jobs.slice(0, 5), // Top 5 jobs
        totalJobs: jobs.length,
        unsubscribeUrl: `${process.env.FRONTEND_URL}/unsubscribe?token=${user.unsubscribeToken}`,
      },
    };

    await sgMail.send(msg);
  }

  async sendApplicationUpdate(user, application) {
    const msg = {
      to: user.email,
      from: this.fromEmail,
      templateId: 'd-application-update-template-id',
      dynamicTemplateData: {
        name: user.name,
        jobTitle: application.jobTitle,
        company: application.company,
        status: application.status,
        dashboardUrl: `${process.env.FRONTEND_URL}/dashboard`,
      },
    };

    await sgMail.send(msg);
  }
}
```

#### SMS Service (Twilio)
```javascript
// SMS Service
import twilio from 'twilio';

class SMSService {
  constructor() {
    this.client = twilio(
      process.env.TWILIO_ACCOUNT_SID,
      process.env.TWILIO_AUTH_TOKEN
    );
    this.fromNumber = process.env.TWILIO_PHONE_NUMBER;
  }

  async sendJobAlert(phoneNumber, jobCount) {
    const message = `🎯 ${jobCount} new job matches found! Check your HR Platform dashboard to view and apply.`;
    
    try {
      const result = await this.client.messages.create({
        body: message,
        from: this.fromNumber,
        to: phoneNumber,
      });
      
      logger.info('SMS job alert sent', { 
        to: phoneNumber, 
        messageSid: result.sid 
      });
      
      return result;
    } catch (error) {
      logger.error('Failed to send SMS', { error, to: phoneNumber });
      throw error;
    }
  }

  async sendApplicationUpdate(phoneNumber, jobTitle, status) {
    const message = `📋 Application update: Your application for "${jobTitle}" is now ${status}.`;
    
    const result = await this.client.messages.create({
      body: message,
      from: this.fromNumber,
      to: phoneNumber,
    });

    return result;
  }
}
```

This comprehensive technology stack documentation provides detailed specifications for all components of the AI-Powered Job Seeker Platform, including code examples, configurations, and integration patterns that enable the full functionality described in the project requirements.
