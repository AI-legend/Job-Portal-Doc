# Backend Architecture

## Overview

The backend follows a microservices architecture pattern with Node.js for API services and Python for AI/ML services. Each service is containerized and can be independently deployed and scaled.

## Architecture Pattern

### Microservices Design
```
Backend Services
├── API Gateway (Express.js)
│   ├── Authentication Middleware
│   ├── Rate Limiting
│   ├── Request Routing
│   └── Response Formatting
├── Core Services
│   ├── User Service (Node.js)
│   ├── Job Service (Node.js)
│   ├── CV Service (Node.js)
│   ├── Notification Service (Node.js)
│   └── Analytics Service (Node.js)
├── AI/ML Services
│   ├── AI Agent Service (Python)
│   ├── Scraper Service (Python)
│   ├── NLP Service (Python)
│   └── Recommendation Engine (Python)
└── External Integrations
    ├── Firebase Auth
    ├── MongoDB Atlas
    ├── Redis Cache
    └── Cloud Storage
```

## Service Architecture

### 1. API Gateway
```
backend/api-gateway/
├── src/
│   ├── middleware/         # Custom middleware
│   │   ├── auth.js        # Authentication middleware
│   │   ├── rateLimit.js   # Rate limiting
│   │   ├── validation.js  # Request validation
│   │   └── cors.js        # CORS configuration
│   ├── routes/            # Route definitions
│   │   ├── auth.js        # Auth routes
│   │   ├── users.js       # User routes
│   │   ├── jobs.js        # Job routes
│   │   ├── cv.js          # CV routes
│   │   └── agent.js       # AI agent routes
│   ├── utils/             # Utility functions
│   │   ├── logger.js      # Logging utility
│   │   ├── response.js    # Response formatter
│   │   └── errors.js      # Error handling
│   ├── config/            # Configuration
│   │   ├── database.js    # DB configuration
│   │   ├── redis.js       # Redis configuration
│   │   └── services.js    # Service endpoints
│   └── app.js             # Main application
├── Dockerfile
├── package.json
└── README.md
```

### 2. User Service
```
backend/user-service/
├── src/
│   ├── controllers/       # Request handlers
│   │   ├── authController.js
│   │   ├── profileController.js
│   │   └── settingsController.js
│   ├── models/            # Data models
│   │   ├── User.js
│   │   ├── Profile.js
│   │   └── Settings.js
│   ├── services/          # Business logic
│   │   ├── authService.js
│   │   ├── profileService.js
│   │   └── emailService.js
│   ├── middleware/        # Service middleware
│   │   ├── validation.js
│   │   └── authorization.js
│   ├── routes/            # Route definitions
│   │   ├── auth.js
│   │   ├── profile.js
│   │   └── settings.js
│   ├── utils/             # Utilities
│   │   ├── hash.js
│   │   ├── token.js
│   │   └── validator.js
│   └── app.js
├── tests/                 # Unit tests
├── Dockerfile
└── package.json
```

### 3. Job Service
```
backend/job-service/
├── src/
│   ├── controllers/
│   │   ├── jobController.js
│   │   ├── applicationController.js
│   │   └── searchController.js
│   ├── models/
│   │   ├── Job.js
│   │   ├── Application.js
│   │   └── Company.js
│   ├── services/
│   │   ├── jobService.js
│   │   ├── searchService.js
│   │   ├── applicationService.js
│   │   └── matchingService.js
│   ├── routes/
│   │   ├── jobs.js
│   │   ├── applications.js
│   │   └── search.js
│   └── app.js
├── tests/
├── Dockerfile
└── package.json
```

### 4. CV Service
```
backend/cv-service/
├── src/
│   ├── controllers/
│   │   ├── cvController.js
│   │   ├── templateController.js
│   │   └── exportController.js
│   ├── models/
│   │   ├── CV.js
│   │   ├── Template.js
│   │   └── Section.js
│   ├── services/
│   │   ├── cvService.js
│   │   ├── pdfService.js
│   │   ├── templateService.js
│   │   └── skillsService.js
│   ├── routes/
│   │   ├── cv.js
│   │   ├── templates.js
│   │   └── export.js
│   └── app.js
├── templates/             # CV templates
├── tests/
├── Dockerfile
└── package.json
```

### 5. AI Agent Service (Python)
```
backend/ai-service/
├── src/
│   ├── api/               # FastAPI application
│   │   ├── main.py
│   │   ├── routes/
│   │   │   ├── agent.py
│   │   │   ├── matching.py
│   │   │   └── recommendations.py
│   │   └── middleware/
│   │       ├── auth.py
│   │       └── logging.py
│   ├── services/          # Business logic
│   │   ├── agent_service.py
│   │   ├── nlp_service.py
│   │   ├── matching_service.py
│   │   └── recommendation_service.py
│   ├── models/            # ML models
│   │   ├── job_matcher.py
│   │   ├── cv_analyzer.py
│   │   └── skill_extractor.py
│   ├── utils/             # Utilities
│   │   ├── text_processing.py
│   │   ├── feature_extraction.py
│   │   └── similarity.py
│   └── config/            # Configuration
│       ├── settings.py
│       └── database.py
├── requirements.txt
├── Dockerfile
└── README.md
```

## Database Design

### MongoDB Schema Design

#### User Collection
```javascript
{
  _id: ObjectId,
  email: String,
  password: String, // hashed
  userType: String, // 'seeker' | 'recruiter'
  profile: {
    firstName: String,
    lastName: String,
    phone: String,
    location: {
      city: String,
      country: String,
      coordinates: [Number] // [longitude, latitude]
    },
    avatar: String, // URL
    bio: String,
    linkedinProfile: String,
    websiteUrl: String
  },
  preferences: {
    jobAlerts: Boolean,
    autoApply: Boolean,
    emailNotifications: Boolean,
    jobTypes: [String], // ['full-time', 'part-time', 'contract']
    salaryRange: {
      min: Number,
      max: Number,
      currency: String
    },
    locations: [String],
    remoteWork: Boolean
  },
  createdAt: Date,
  updatedAt: Date,
  lastLoginAt: Date,
  isActive: Boolean,
  emailVerified: Boolean
}
```

#### Job Collection
```javascript
{
  _id: ObjectId,
  title: String,
  company: {
    name: String,
    logo: String,
    website: String,
    size: String, // 'startup', 'small', 'medium', 'large'
    industry: String
  },
  description: String,
  requirements: [String],
  responsibilities: [String],
  benefits: [String],
  jobType: String, // 'full-time', 'part-time', 'contract', 'internship'
  workType: String, // 'remote', 'on-site', 'hybrid'
  location: {
    city: String,
    country: String,
    coordinates: [Number]
  },
  salary: {
    min: Number,
    max: Number,
    currency: String,
    period: String // 'annually', 'monthly', 'hourly'
  },
  skills: [String],
  experienceLevel: String, // 'entry', 'mid', 'senior', 'lead'
  postedBy: ObjectId, // User ID (recruiter)
  applicationDeadline: Date,
  isActive: Boolean,
  externalUrl: String, // for scraped jobs
  source: String, // 'internal', 'linkedin', 'indeed', 'glassdoor'
  applications: [{
    applicantId: ObjectId,
    appliedAt: Date,
    status: String, // 'pending', 'reviewed', 'interviewed', 'rejected', 'hired'
    cvId: ObjectId,
    coverLetter: String,
    autoApplied: Boolean
  }],
  createdAt: Date,
  updatedAt: Date
}
```

#### CV Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId,
  name: String,
  templateId: ObjectId,
  personalInfo: {
    firstName: String,
    lastName: String,
    email: String,
    phone: String,
    location: String,
    linkedinProfile: String,
    websiteUrl: String,
    summary: String
  },
  experience: [{
    company: String,
    position: String,
    startDate: Date,
    endDate: Date,
    current: Boolean,
    description: String,
    achievements: [String]
  }],
  education: [{
    institution: String,
    degree: String,
    field: String,
    startDate: Date,
    endDate: Date,
    gpa: Number,
    description: String
  }],
  skills: [{
    name: String,
    level: String, // 'beginner', 'intermediate', 'advanced', 'expert'
    category: String // 'technical', 'soft', 'language'
  }],
  projects: [{
    name: String,
    description: String,
    technologies: [String],
    url: String,
    startDate: Date,
    endDate: Date
  }],
  certifications: [{
    name: String,
    issuer: String,
    issueDate: Date,
    expiryDate: Date,
    credentialId: String,
    url: String
  }],
  languages: [{
    name: String,
    proficiency: String // 'basic', 'conversational', 'fluent', 'native'
  }],
  isDefault: Boolean,
  pdfUrl: String, // Generated PDF URL
  version: Number,
  createdAt: Date,
  updatedAt: Date
}
```

### Firebase Integration

#### Authentication Setup
```javascript
// config/firebase.js
import { initializeApp } from 'firebase/app'
import { getAuth } from 'firebase/auth'
import { getFirestore } from 'firebase/firestore'

const firebaseConfig = {
  apiKey: process.env.FIREBASE_API_KEY,
  authDomain: process.env.FIREBASE_AUTH_DOMAIN,
  projectId: process.env.FIREBASE_PROJECT_ID,
  storageBucket: process.env.FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.FIREBASE_MESSAGING_SENDER_ID,
  appId: process.env.FIREBASE_APP_ID
}

const app = initializeApp(firebaseConfig)
export const auth = getAuth(app)
export const db = getFirestore(app)
```

#### Firebase Admin Setup
```javascript
// config/firebaseAdmin.js
import admin from 'firebase-admin'

const serviceAccount = JSON.parse(process.env.FIREBASE_SERVICE_ACCOUNT)

if (!admin.apps.length) {
  admin.initializeApp({
    credential: admin.credential.cert(serviceAccount),
    databaseURL: process.env.FIREBASE_DATABASE_URL
  })
}

export const adminAuth = admin.auth()
export const adminDb = admin.firestore()
```

## API Design

### RESTful API Structure
```
/api/v1/
├── /auth
│   ├── POST   /register
│   ├── POST   /login
│   ├── POST   /logout
│   ├── POST   /refresh
│   ├── POST   /forgot-password
│   └── POST   /reset-password
├── /users
│   ├── GET    /profile
│   ├── PUT    /profile
│   ├── DELETE /account
│   ├── GET    /preferences
│   ├── PUT    /preferences
│   └── POST   /upload-avatar
├── /jobs
│   ├── GET    /              # List jobs with filters
│   ├── POST   /              # Create job (recruiter)
│   ├── GET    /:id           # Get job details
│   ├── PUT    /:id           # Update job (recruiter)
│   ├── DELETE /:id           # Delete job (recruiter)
│   ├── POST   /:id/apply     # Apply to job
│   ├── GET    /search        # Search jobs
│   └── GET    /recommendations # Get recommended jobs
├── /cv
│   ├── GET    /              # List user's CVs
│   ├── POST   /              # Create new CV
│   ├── GET    /:id           # Get CV details
│   ├── PUT    /:id           # Update CV
│   ├── DELETE /:id           # Delete CV
│   ├── POST   /:id/export    # Export CV to PDF
│   └── GET    /templates     # Get CV templates
├── /agent
│   ├── POST   /chat          # Chat with AI agent
│   ├── POST   /analyze-cv    # Analyze CV
│   ├── POST   /job-match     # Match jobs to CV
│   ├── POST   /auto-apply    # Auto-apply to jobs
│   └── GET    /applications  # Get auto-applied jobs
└── /applications
    ├── GET    /              # List user's applications
    ├── GET    /:id           # Get application details
    ├── PUT    /:id/status    # Update application status
    └── DELETE /:id           # Withdraw application
```

### GraphQL Schema (Optional)
```graphql
type User {
  id: ID!
  email: String!
  userType: UserType!
  profile: Profile!
  preferences: Preferences!
  cvs: [CV!]!
  applications: [Application!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Job {
  id: ID!
  title: String!
  company: Company!
  description: String!
  requirements: [String!]!
  jobType: JobType!
  workType: WorkType!
  location: Location!
  salary: Salary
  skills: [String!]!
  experienceLevel: ExperienceLevel!
  applications: [Application!]!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type CV {
  id: ID!
  name: String!
  personalInfo: PersonalInfo!
  experience: [Experience!]!
  education: [Education!]!
  skills: [Skill!]!
  projects: [Project!]!
  pdfUrl: String
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Query {
  me: User
  jobs(filters: JobFilters): [Job!]!
  job(id: ID!): Job
  cvs: [CV!]!
  cv(id: ID!): CV
  recommendations: [Job!]!
}

type Mutation {
  register(input: RegisterInput!): AuthPayload!
  login(input: LoginInput!): AuthPayload!
  updateProfile(input: ProfileInput!): User!
  createJob(input: JobInput!): Job!
  createCV(input: CVInput!): CV!
  applyToJob(jobId: ID!, cvId: ID!): Application!
  autoApplyJobs(cvId: ID!, criteria: JobFilters!): [Application!]!
}
```

## Service Communication

### Event-Driven Architecture
```javascript
// Event Bus Implementation
class EventBus {
  constructor() {
    this.events = {}
  }

  emit(eventName, data) {
    if (this.events[eventName]) {
      this.events[eventName].forEach(callback => callback(data))
    }
  }

  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = []
    }
    this.events[eventName].push(callback)
  }

  off(eventName, callback) {
    if (this.events[eventName]) {
      this.events[eventName] = this.events[eventName].filter(cb => cb !== callback)
    }
  }
}

// Usage in services
const eventBus = new EventBus()

// User service emits events
eventBus.emit('user.registered', { userId, email, userType })
eventBus.emit('user.profileUpdated', { userId, changes })

// Other services listen to events
eventBus.on('user.registered', (data) => {
  // Send welcome email
  emailService.sendWelcomeEmail(data.email)
})

eventBus.on('job.applied', (data) => {
  // Update analytics
  analyticsService.trackApplication(data)
})
```

### Inter-Service Communication
```javascript
// Service Registry
class ServiceRegistry {
  constructor() {
    this.services = new Map()
  }

  register(serviceName, serviceUrl) {
    this.services.set(serviceName, serviceUrl)
  }

  getService(serviceName) {
    return this.services.get(serviceName)
  }

  async healthCheck() {
    const healthStatus = {}
    for (const [name, url] of this.services) {
      try {
        const response = await fetch(`${url}/health`)
        healthStatus[name] = response.ok ? 'healthy' : 'unhealthy'
      } catch (error) {
        healthStatus[name] = 'unreachable'
      }
    }
    return healthStatus
  }
}

// API Client for inter-service communication
class ApiClient {
  constructor(baseUrl, options = {}) {
    this.baseUrl = baseUrl
    this.options = options
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseUrl}${endpoint}`
    const config = {
      headers: {
        'Content-Type': 'application/json',
        ...this.options.headers,
        ...options.headers
      },
      ...this.options,
      ...options
    }

    const response = await fetch(url, config)
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    return response.json()
  }

  get(endpoint, options = {}) {
    return this.request(endpoint, { method: 'GET', ...options })
  }

  post(endpoint, data, options = {}) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
      ...options
    })
  }
}
```

## Caching Strategy

### Redis Implementation
```javascript
// Cache Service
class CacheService {
  constructor(redisClient) {
    this.redis = redisClient
  }

  async get(key) {
    try {
      const value = await this.redis.get(key)
      return value ? JSON.parse(value) : null
    } catch (error) {
      console.error('Cache get error:', error)
      return null
    }
  }

  async set(key, value, ttl = 3600) {
    try {
      await this.redis.setex(key, ttl, JSON.stringify(value))
    } catch (error) {
      console.error('Cache set error:', error)
    }
  }

  async del(key) {
    try {
      await this.redis.del(key)
    } catch (error) {
      console.error('Cache delete error:', error)
    }
  }

  async invalidatePattern(pattern) {
    try {
      const keys = await this.redis.keys(pattern)
      if (keys.length > 0) {
        await this.redis.del(...keys)
      }
    } catch (error) {
      console.error('Cache invalidation error:', error)
    }
  }
}

// Cache middleware
const cacheMiddleware = (ttl = 3600) => {
  return async (req, res, next) => {
    const key = `cache:${req.method}:${req.originalUrl}`
    
    try {
      const cachedResponse = await cacheService.get(key)
      if (cachedResponse) {
        return res.json(cachedResponse)
      }

      // Store original res.json
      const originalJson = res.json
      res.json = function(data) {
        // Cache the response
        cacheService.set(key, data, ttl)
        return originalJson.call(this, data)
      }

      next()
    } catch (error) {
      next()
    }
  }
}
```

## Error Handling

### Global Error Handler
```javascript
// Error types
class AppError extends Error {
  constructor(message, statusCode, errorCode = null) {
    super(message)
    this.statusCode = statusCode
    this.errorCode = errorCode
    this.isOperational = true

    Error.captureStackTrace(this, this.constructor)
  }
}

class ValidationError extends AppError {
  constructor(message, errors = []) {
    super(message, 400, 'VALIDATION_ERROR')
    this.errors = errors
  }
}

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, 'NOT_FOUND')
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED')
  }
}

// Global error handler middleware
const errorHandler = (err, req, res, next) => {
  let error = { ...err }
  error.message = err.message

  // Log error
  console.error(err)

  // Mongoose bad ObjectId
  if (err.name === 'CastError') {
    const message = 'Resource not found'
    error = new NotFoundError(message)
  }

  // Mongoose duplicate key
  if (err.code === 11000) {
    const message = 'Duplicate field value entered'
    error = new ValidationError(message)
  }

  // Mongoose validation error
  if (err.name === 'ValidationError') {
    const message = Object.values(err.errors).map(val => val.message)
    error = new ValidationError('Validation Error', message)
  }

  res.status(error.statusCode || 500).json({
    success: false,
    error: {
      message: error.message || 'Server Error',
      code: error.errorCode,
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  })
}

module.exports = {
  AppError,
  ValidationError,
  NotFoundError,
  UnauthorizedError,
  errorHandler
}
```

## Security Implementation

### Authentication Middleware
```javascript
// JWT middleware
const jwt = require('jsonwebtoken')
const { UnauthorizedError } = require('../utils/errors')

const authenticate = async (req, res, next) => {
  try {
    const token = req.header('Authorization')?.replace('Bearer ', '')
    
    if (!token) {
      throw new UnauthorizedError('No token provided')
    }

    const decoded = jwt.verify(token, process.env.JWT_SECRET)
    req.user = decoded
    
    // Verify token with Firebase if needed
    if (process.env.USE_FIREBASE_AUTH) {
      const decodedToken = await admin.auth().verifyIdToken(token)
      req.user = decodedToken
    }

    next()
  } catch (error) {
    throw new UnauthorizedError('Invalid token')
  }
}

// Role-based authorization
const authorize = (...roles) => {
  return (req, res, next) => {
    if (!req.user) {
      throw new UnauthorizedError('Authentication required')
    }

    if (!roles.includes(req.user.userType)) {
      throw new UnauthorizedError('Insufficient permissions')
    }

    next()
  }
}

module.exports = { authenticate, authorize }
```

### Input Validation
```javascript
// Validation schemas using Joi
const Joi = require('joi')

const schemas = {
  register: Joi.object({
    email: Joi.string().email().required(),
    password: Joi.string().min(8).required(),
    userType: Joi.string().valid('seeker', 'recruiter').required(),
    firstName: Joi.string().min(2).max(50).required(),
    lastName: Joi.string().min(2).max(50).required()
  }),

  createJob: Joi.object({
    title: Joi.string().min(5).max(100).required(),
    description: Joi.string().min(50).required(),
    company: Joi.object({
      name: Joi.string().required(),
      website: Joi.string().uri().optional()
    }).required(),
    jobType: Joi.string().valid('full-time', 'part-time', 'contract', 'internship').required(),
    location: Joi.object({
      city: Joi.string().required(),
      country: Joi.string().required()
    }).required(),
    salary: Joi.object({
      min: Joi.number().positive().required(),
      max: Joi.number().positive().required(),
      currency: Joi.string().length(3).required()
    }).optional()
  })
}

// Validation middleware
const validate = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body)
    if (error) {
      const errors = error.details.map(detail => detail.message)
      throw new ValidationError('Validation failed', errors)
    }
    next()
  }
}

module.exports = { schemas, validate }
```

## Monitoring & Logging

### Logging Setup
```javascript
// Logger configuration
const winston = require('winston')

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: { service: process.env.SERVICE_NAME },
  transports: [
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
})

if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }))
}

// Request logging middleware
const requestLogger = (req, res, next) => {
  const start = Date.now()
  
  res.on('finish', () => {
    const duration = Date.now() - start
    logger.info({
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      userAgent: req.get('User-Agent'),
      ip: req.ip
    })
  })

  next()
}

module.exports = { logger, requestLogger }
```
