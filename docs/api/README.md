# API Documentation

## Overview

The HR Agent Platform API is a RESTful service that provides comprehensive endpoints for user management, job operations, CV management, and AI-powered features. The API follows REST principles and returns JSON responses.

## Table of Contents
1. [Authentication](#authentication)
2. [Base URLs](#base-urls)
3. [Common Headers](#common-headers)
4. [Error Handling](#error-handling)
5. [Rate Limiting](#rate-limiting)
6. [API Endpoints](#api-endpoints)
7. [Webhooks](#webhooks)
8. [SDK Examples](#sdk-examples)

## Authentication

### JWT Bearer Token
All authenticated endpoints require a JWT token in the Authorization header:

```http
Authorization: Bearer <your-jwt-token>
```

### Authentication Flow
```javascript
// Login to get token
POST /api/v1/auth/login
{
  "email": "user@example.com",
  "password": "password123"
}

// Response
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "refresh_token_here",
    "user": {
      "id": "user_id",
      "email": "user@example.com",
      "userType": "seeker"
    }
  }
}
```

### Token Refresh
```javascript
POST /api/v1/auth/refresh
{
  "refreshToken": "refresh_token_here"
}
```

## Base URLs

- **Production**: `https://api.hr-agent.com`
- **Staging**: `https://staging-api.hr-agent.com`
- **Development**: `http://localhost:8000`

## Common Headers

```http
Content-Type: application/json
Accept: application/json
Authorization: Bearer <token>
X-API-Version: v1
User-Agent: YourApp/1.0
```

## Error Handling

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": {
      "field": "email",
      "issue": "Invalid email format"
    },
    "timestamp": "2025-08-03T10:00:00Z",
    "requestId": "req_123456789"
  }
}
```

### Error Codes
| Code | Status | Description |
|------|--------|-------------|
| `UNAUTHORIZED` | 401 | Invalid or missing authentication |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `VALIDATION_ERROR` | 400 | Invalid input data |
| `RATE_LIMIT_EXCEEDED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Server error |

## Rate Limiting

API endpoints are rate-limited to ensure fair usage:

- **General endpoints**: 1000 requests/hour per user
- **Auth endpoints**: 10 requests/minute per IP
- **Job search**: 100 requests/hour per user
- **AI endpoints**: 50 requests/hour per user

Rate limit headers:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1625097600
```

## API Endpoints

### Authentication Endpoints

#### POST /api/v1/auth/register
Register a new user account.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123",
  "userType": "seeker",
  "firstName": "John",
  "lastName": "Doe",
  "phone": "+1234567890"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "64f8a1b2c3d4e5f6a7b8c9d0",
      "email": "user@example.com",
      "userType": "seeker",
      "profile": {
        "firstName": "John",
        "lastName": "Doe",
        "phone": "+1234567890"
      },
      "createdAt": "2025-08-03T10:00:00Z",
      "emailVerified": false
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "refresh_token_here"
  }
}
```

#### POST /api/v1/auth/login
Authenticate user and get access token.

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

#### POST /api/v1/auth/logout
Logout user and invalidate token.

**Headers:** `Authorization: Bearer <token>`

#### POST /api/v1/auth/forgot-password
Request password reset email.

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

#### POST /api/v1/auth/reset-password
Reset password with token from email.

**Request Body:**
```json
{
  "token": "reset_token_from_email",
  "newPassword": "newpassword123"
}
```

### User Management Endpoints

#### GET /api/v1/users/profile
Get current user's profile.

**Headers:** `Authorization: Bearer <token>`

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "64f8a1b2c3d4e5f6a7b8c9d0",
    "email": "user@example.com",
    "userType": "seeker",
    "profile": {
      "firstName": "John",
      "lastName": "Doe",
      "phone": "+1234567890",
      "location": {
        "city": "San Francisco",
        "country": "USA"
      },
      "bio": "Software developer with 5 years of experience",
      "avatar": "https://cdn.hr-agent.com/avatars/user123.jpg",
      "linkedinProfile": "https://linkedin.com/in/johndoe"
    },
    "preferences": {
      "jobAlerts": true,
      "autoApply": false,
      "emailNotifications": true,
      "jobTypes": ["full-time", "remote"],
      "salaryRange": {
        "min": 80000,
        "max": 120000,
        "currency": "USD"
      }
    }
  }
}
```

#### PUT /api/v1/users/profile
Update user profile.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "profile": {
    "firstName": "John",
    "lastName": "Smith",
    "phone": "+1234567890",
    "location": {
      "city": "New York",
      "country": "USA"
    },
    "bio": "Senior software developer",
    "linkedinProfile": "https://linkedin.com/in/johnsmith"
  }
}
```

#### GET /api/v1/users/preferences
Get user preferences.

#### PUT /api/v1/users/preferences
Update user preferences.

**Request Body:**
```json
{
  "jobAlerts": true,
  "autoApply": true,
  "emailNotifications": true,
  "jobTypes": ["full-time", "contract"],
  "salaryRange": {
    "min": 90000,
    "max": 150000,
    "currency": "USD"
  },
  "locations": ["San Francisco", "Remote"],
  "remoteWork": true
}
```

#### POST /api/v1/users/upload-avatar
Upload user avatar image.

**Content-Type:** `multipart/form-data`
**Body:** File upload (max 5MB, jpg/png)

### Job Management Endpoints

#### GET /api/v1/jobs
Get job listings with filtering and pagination.

**Query Parameters:**
- `page` (integer): Page number (default: 1)
- `limit` (integer): Items per page (default: 20, max: 100)
- `q` (string): Search query
- `location` (string): Job location
- `jobType` (string): full-time, part-time, contract, internship
- `workType` (string): remote, on-site, hybrid
- `salaryMin` (number): Minimum salary
- `salaryMax` (number): Maximum salary
- `experienceLevel` (string): entry, mid, senior, lead
- `skills` (string[]): Required skills
- `sort` (string): createdAt, salary, relevance (default: createdAt)
- `order` (string): asc, desc (default: desc)

**Example Request:**
```http
GET /api/v1/jobs?q=software+engineer&location=San+Francisco&jobType=full-time&salaryMin=100000&page=1&limit=20
```

**Response:**
```json
{
  "success": true,
  "data": {
    "jobs": [
      {
        "id": "64f8a1b2c3d4e5f6a7b8c9d1",
        "title": "Senior Software Engineer",
        "company": {
          "name": "TechCorp",
          "logo": "https://cdn.hr-agent.com/logos/techcorp.png",
          "website": "https://techcorp.com",
          "size": "large",
          "industry": "Technology"
        },
        "description": "We are looking for a senior software engineer...",
        "requirements": [
          "5+ years of experience",
          "React and Node.js expertise",
          "AWS experience"
        ],
        "responsibilities": [
          "Design and develop scalable applications",
          "Mentor junior developers",
          "Collaborate with product teams"
        ],
        "benefits": [
          "Health insurance",
          "401k matching",
          "Flexible hours"
        ],
        "jobType": "full-time",
        "workType": "hybrid",
        "location": {
          "city": "San Francisco",
          "country": "USA",
          "coordinates": [-122.4194, 37.7749]
        },
        "salary": {
          "min": 120000,
          "max": 160000,
          "currency": "USD",
          "period": "annually"
        },
        "skills": ["JavaScript", "React", "Node.js", "AWS"],
        "experienceLevel": "senior",
        "applicationDeadline": "2025-09-01T23:59:59Z",
        "isActive": true,
        "externalUrl": null,
        "source": "internal",
        "createdAt": "2025-08-03T10:00:00Z",
        "applicationsCount": 25
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "pages": 8,
      "hasNext": true,
      "hasPrev": false
    },
    "filters": {
      "appliedFilters": {
        "q": "software engineer",
        "location": "San Francisco",
        "jobType": "full-time",
        "salaryMin": 100000
      },
      "availableFilters": {
        "jobTypes": ["full-time", "part-time", "contract"],
        "workTypes": ["remote", "on-site", "hybrid"],
        "experienceLevels": ["entry", "mid", "senior", "lead"],
        "topSkills": ["JavaScript", "Python", "React", "AWS"],
        "locations": ["San Francisco", "New York", "Remote"]
      }
    }
  }
}
```

#### GET /api/v1/jobs/:id
Get detailed job information.

**Response:**
```json
{
  "success": true,
  "data": {
    "job": {
      "id": "64f8a1b2c3d4e5f6a7b8c9d1",
      "title": "Senior Software Engineer",
      "company": { /* company details */ },
      "description": "Full job description...",
      "requirements": ["5+ years experience", "..."],
      "responsibilities": ["Design applications", "..."],
      "benefits": ["Health insurance", "..."],
      /* ... other job fields ... */
      "applications": [
        {
          "id": "64f8a1b2c3d4e5f6a7b8c9d2",
          "applicantId": "64f8a1b2c3d4e5f6a7b8c9d0",
          "appliedAt": "2025-08-03T11:00:00Z",
          "status": "pending",
          "autoApplied": false
        }
      ],
      "similarJobs": [
        {
          "id": "64f8a1b2c3d4e5f6a7b8c9d3",
          "title": "Software Engineer",
          "company": "AnotherCorp",
          "matchScore": 0.85
        }
      ]
    },
    "userContext": {
      "hasApplied": false,
      "matchScore": 0.92,
      "matchReasons": [
        "Skills match: JavaScript, React, Node.js",
        "Experience level match",
        "Location preference match"
      ],
      "canAutoApply": true
    }
  }
}
```

#### POST /api/v1/jobs
Create a new job posting (recruiters only).

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "title": "Frontend Developer",
  "company": {
    "name": "StartupCorp",
    "website": "https://startupcorp.com",
    "size": "startup",
    "industry": "Technology"
  },
  "description": "We are seeking a talented frontend developer...",
  "requirements": [
    "3+ years React experience",
    "TypeScript knowledge",
    "CSS expertise"
  ],
  "responsibilities": [
    "Develop user interfaces",
    "Collaborate with designers",
    "Optimize performance"
  ],
  "benefits": [
    "Equity package",
    "Health insurance",
    "Remote work"
  ],
  "jobType": "full-time",
  "workType": "remote",
  "location": {
    "city": "Remote",
    "country": "USA"
  },
  "salary": {
    "min": 80000,
    "max": 110000,
    "currency": "USD",
    "period": "annually"
  },
  "skills": ["React", "TypeScript", "CSS", "JavaScript"],
  "experienceLevel": "mid",
  "applicationDeadline": "2025-09-15T23:59:59Z"
}
```

#### PUT /api/v1/jobs/:id
Update job posting (recruiters only).

#### DELETE /api/v1/jobs/:id
Delete job posting (recruiters only).

#### POST /api/v1/jobs/:id/apply
Apply to a job.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "cvId": "64f8a1b2c3d4e5f6a7b8c9d4",
  "coverLetter": "I am excited to apply for this position because..."
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "application": {
      "id": "64f8a1b2c3d4e5f6a7b8c9d5",
      "jobId": "64f8a1b2c3d4e5f6a7b8c9d1",
      "applicantId": "64f8a1b2c3d4e5f6a7b8c9d0",
      "cvId": "64f8a1b2c3d4e5f6a7b8c9d4",
      "coverLetter": "I am excited to apply...",
      "status": "pending",
      "appliedAt": "2025-08-03T12:00:00Z",
      "autoApplied": false
    }
  }
}
```

#### GET /api/v1/jobs/search
Advanced job search with AI-powered matching.

**Query Parameters:**
- All parameters from GET /api/v1/jobs
- `useAI` (boolean): Enable AI-powered search
- `cvId` (string): CV ID for personalized matching

#### GET /api/v1/jobs/recommendations
Get personalized job recommendations.

**Headers:** `Authorization: Bearer <token>`

**Query Parameters:**
- `cvId` (string): CV ID for recommendations
- `limit` (integer): Number of recommendations (default: 10)

### CV Management Endpoints

#### GET /api/v1/cv
Get user's CV list.

**Headers:** `Authorization: Bearer <token>`

**Response:**
```json
{
  "success": true,
  "data": {
    "cvs": [
      {
        "id": "64f8a1b2c3d4e5f6a7b8c9d4",
        "name": "Software Engineer CV",
        "templateId": "64f8a1b2c3d4e5f6a7b8c9d6",
        "isDefault": true,
        "pdfUrl": "https://cdn.hr-agent.com/cvs/user123_cv1.pdf",
        "version": 1,
        "createdAt": "2025-08-01T10:00:00Z",
        "updatedAt": "2025-08-03T10:00:00Z",
        "preview": {
          "personalInfo": {
            "firstName": "John",
            "lastName": "Doe",
            "email": "john@example.com"
          },
          "skills": ["JavaScript", "React", "Node.js"],
          "experienceYears": 5
        }
      }
    ]
  }
}
```

#### POST /api/v1/cv
Create a new CV.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "name": "Senior Developer CV",
  "templateId": "64f8a1b2c3d4e5f6a7b8c9d6",
  "personalInfo": {
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com",
    "phone": "+1234567890",
    "location": "San Francisco, CA",
    "linkedinProfile": "https://linkedin.com/in/johndoe",
    "websiteUrl": "https://johndoe.dev",
    "summary": "Experienced software engineer with expertise in full-stack development..."
  },
  "experience": [
    {
      "company": "TechCorp",
      "position": "Senior Software Engineer",
      "startDate": "2022-01-01",
      "endDate": null,
      "current": true,
      "description": "Lead development of scalable web applications using React and Node.js",
      "achievements": [
        "Improved application performance by 40%",
        "Led team of 5 developers",
        "Implemented CI/CD pipeline"
      ]
    }
  ],
  "education": [
    {
      "institution": "Stanford University",
      "degree": "Bachelor of Science",
      "field": "Computer Science",
      "startDate": "2015-09-01",
      "endDate": "2019-06-01",
      "gpa": 3.8,
      "description": "Relevant coursework: Data Structures, Algorithms, Software Engineering"
    }
  ],
  "skills": [
    {
      "name": "JavaScript",
      "level": "expert",
      "category": "technical"
    },
    {
      "name": "React",
      "level": "expert",
      "category": "technical"
    },
    {
      "name": "Leadership",
      "level": "advanced",
      "category": "soft"
    }
  ],
  "projects": [
    {
      "name": "E-commerce Platform",
      "description": "Built a full-stack e-commerce platform with React and Node.js",
      "technologies": ["React", "Node.js", "MongoDB", "Stripe"],
      "url": "https://github.com/johndoe/ecommerce",
      "startDate": "2023-01-01",
      "endDate": "2023-06-01"
    }
  ],
  "certifications": [
    {
      "name": "AWS Solutions Architect",
      "issuer": "Amazon Web Services",
      "issueDate": "2023-03-15",
      "expiryDate": "2026-03-15",
      "credentialId": "AWS-SAA-123456",
      "url": "https://aws.amazon.com/verification"
    }
  ],
  "languages": [
    {
      "name": "English",
      "proficiency": "native"
    },
    {
      "name": "Spanish",
      "proficiency": "conversational"
    }
  ]
}
```

#### GET /api/v1/cv/:id
Get CV details.

#### PUT /api/v1/cv/:id
Update CV.

#### DELETE /api/v1/cv/:id
Delete CV.

#### POST /api/v1/cv/:id/export
Export CV to PDF.

**Query Parameters:**
- `format` (string): pdf, docx (default: pdf)
- `template` (string): Template style override

**Response:**
```json
{
  "success": true,
  "data": {
    "downloadUrl": "https://cdn.hr-agent.com/exports/cv_64f8a1b2c3d4e5f6a7b8c9d4.pdf",
    "expiresAt": "2025-08-03T22:00:00Z"
  }
}
```

#### GET /api/v1/cv/templates
Get available CV templates.

**Response:**
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "id": "64f8a1b2c3d4e5f6a7b8c9d6",
        "name": "Modern Professional",
        "category": "professional",
        "preview": "https://cdn.hr-agent.com/templates/modern-professional.png",
        "isPremium": false,
        "description": "Clean and modern design suitable for tech professionals"
      },
      {
        "id": "64f8a1b2c3d4e5f6a7b8c9d7",
        "name": "Creative Designer",
        "category": "creative",
        "preview": "https://cdn.hr-agent.com/templates/creative-designer.png",
        "isPremium": true,
        "description": "Eye-catching design perfect for creative professionals"
      }
    ]
  }
}
```

### AI Agent Endpoints

#### POST /api/v1/agent/chat
Chat with AI agent for job search assistance.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "message": "Find me software engineering jobs in San Francisco",
  "context": {
    "cvId": "64f8a1b2c3d4e5f6a7b8c9d4",
    "conversationId": "64f8a1b2c3d4e5f6a7b8c9d8"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "response": {
      "message": "I found 15 software engineering jobs in San Francisco that match your profile. Here are the top 5 recommendations based on your skills and experience:",
      "type": "job_recommendations",
      "data": {
        "jobs": [
          {
            "id": "64f8a1b2c3d4e5f6a7b8c9d1",
            "title": "Senior Software Engineer",
            "company": "TechCorp",
            "matchScore": 0.95,
            "reason": "Perfect match for your React and Node.js expertise"
          }
        ],
        "totalFound": 15
      }
    },
    "conversationId": "64f8a1b2c3d4e5f6a7b8c9d8",
    "suggestedActions": [
      {
        "type": "apply_job",
        "label": "Apply to TechCorp",
        "jobId": "64f8a1b2c3d4e5f6a7b8c9d1"
      },
      {
        "type": "auto_apply",
        "label": "Auto-apply to similar jobs",
        "criteria": { /* search criteria */ }
      }
    ]
  }
}
```

#### POST /api/v1/agent/analyze-cv
Get AI analysis and suggestions for CV improvement.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "cvId": "64f8a1b2c3d4e5f6a7b8c9d4",
  "targetRole": "Senior Software Engineer",
  "targetIndustry": "Technology"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "analysis": {
      "overallScore": 8.5,
      "strengths": [
        "Strong technical skills alignment",
        "Relevant work experience",
        "Good project portfolio"
      ],
      "improvements": [
        {
          "category": "skills",
          "priority": "high",
          "suggestion": "Add cloud computing skills (AWS, Docker) as they appear in 80% of target roles",
          "impact": "Could increase match rate by 25%"
        },
        {
          "category": "experience",
          "priority": "medium",
          "suggestion": "Quantify achievements with specific metrics",
          "impact": "Makes accomplishments more compelling"
        }
      ],
      "missingSkills": [
        {
          "skill": "Docker",
          "frequency": 0.75,
          "importance": "high"
        },
        {
          "skill": "Kubernetes",
          "frequency": 0.60,
          "importance": "medium"
        }
      ],
      "atsOptimization": {
        "score": 7.8,
        "issues": [
          "Use more industry keywords",
          "Standardize section headers"
        ]
      }
    },
    "recommendations": [
      {
        "type": "skill_course",
        "title": "AWS Certified Solutions Architect",
        "provider": "AWS",
        "url": "https://aws.amazon.com/certification/"
      }
    ]
  }
}
```

#### POST /api/v1/agent/job-match
Get AI-powered job matching for a specific CV.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "cvId": "64f8a1b2c3d4e5f6a7b8c9d4",
  "criteria": {
    "location": "San Francisco",
    "jobType": "full-time",
    "salaryMin": 100000,
    "remoteWork": true
  },
  "limit": 20
}
```

#### POST /api/v1/agent/auto-apply
Automatically apply to matching jobs.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "cvId": "64f8a1b2c3d4e5f6a7b8c9d4",
  "criteria": {
    "location": ["San Francisco", "Remote"],
    "jobType": ["full-time"],
    "salaryMin": 120000,
    "experienceLevel": ["senior"],
    "skills": ["React", "Node.js"]
  },
  "preferences": {
    "maxApplications": 10,
    "minMatchScore": 0.8,
    "generateCoverLetter": true,
    "excludeCompanies": ["Company1", "Company2"]
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "autoApplySession": {
      "id": "64f8a1b2c3d4e5f6a7b8c9d9",
      "status": "processing",
      "criteria": { /* applied criteria */ },
      "progress": {
        "jobsFound": 25,
        "jobsFiltered": 15,
        "jobsApplied": 0,
        "jobsFailed": 0
      },
      "estimatedDuration": "5-10 minutes"
    }
  }
}
```

#### GET /api/v1/agent/auto-apply/:sessionId
Check auto-apply session status.

**Response:**
```json
{
  "success": true,
  "data": {
    "session": {
      "id": "64f8a1b2c3d4e5f6a7b8c9d9",
      "status": "completed",
      "startedAt": "2025-08-03T14:00:00Z",
      "completedAt": "2025-08-03T14:08:30Z",
      "results": {
        "jobsFound": 25,
        "jobsFiltered": 15,
        "jobsApplied": 8,
        "jobsFailed": 2,
        "applicationsCreated": [
          {
            "jobId": "64f8a1b2c3d4e5f6a7b8c9d1",
            "applicationId": "64f8a1b2c3d4e5f6a7b8c9da",
            "status": "success",
            "appliedAt": "2025-08-03T14:02:15Z"
          }
        ],
        "failedApplications": [
          {
            "jobId": "64f8a1b2c3d4e5f6a7b8c9db",
            "reason": "External platform temporarily unavailable",
            "canRetry": true
          }
        ]
      }
    }
  }
}
```

### Application Management Endpoints

#### GET /api/v1/applications
Get user's job applications.

**Headers:** `Authorization: Bearer <token>`

**Query Parameters:**
- `status` (string): pending, reviewed, interviewed, rejected, hired
- `autoApplied` (boolean): Filter by auto-applied status
- `page` (integer): Page number
- `limit` (integer): Items per page
- `sort` (string): appliedAt, status, company
- `order` (string): asc, desc

**Response:**
```json
{
  "success": true,
  "data": {
    "applications": [
      {
        "id": "64f8a1b2c3d4e5f6a7b8c9d5",
        "job": {
          "id": "64f8a1b2c3d4e5f6a7b8c9d1",
          "title": "Senior Software Engineer",
          "company": "TechCorp",
          "location": "San Francisco, CA"
        },
        "cv": {
          "id": "64f8a1b2c3d4e5f6a7b8c9d4",
          "name": "Software Engineer CV"
        },
        "status": "reviewed",
        "appliedAt": "2025-08-03T12:00:00Z",
        "autoApplied": false,
        "coverLetter": "I am excited to apply...",
        "timeline": [
          {
            "status": "pending",
            "timestamp": "2025-08-03T12:00:00Z",
            "note": "Application submitted"
          },
          {
            "status": "reviewed",
            "timestamp": "2025-08-03T15:30:00Z",
            "note": "Application reviewed by HR"
          }
        ]
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45,
      "pages": 3
    },
    "stats": {
      "total": 45,
      "pending": 20,
      "reviewed": 15,
      "interviewed": 5,
      "rejected": 3,
      "hired": 2
    }
  }
}
```

#### GET /api/v1/applications/:id
Get application details.

#### PUT /api/v1/applications/:id/status
Update application status (recruiters only).

**Request Body:**
```json
{
  "status": "interviewed",
  "note": "Scheduled for technical interview on Aug 10"
}
```

#### DELETE /api/v1/applications/:id
Withdraw application.

### Analytics Endpoints

#### GET /api/v1/analytics/dashboard
Get user dashboard analytics.

**Headers:** `Authorization: Bearer <token>`

**Query Parameters:**
- `period` (string): 7d, 30d, 90d, 1y (default: 30d)

**Response:**
```json
{
  "success": true,
  "data": {
    "period": "30d",
    "metrics": {
      "applications": {
        "total": 45,
        "thisMonth": 12,
        "changeFromLastMonth": 20
      },
      "profileViews": {
        "total": 128,
        "thisMonth": 35,
        "changeFromLastMonth": 15
      },
      "jobMatches": {
        "total": 234,
        "thisMonth": 78,
        "averageMatchScore": 0.72
      },
      "interviewsScheduled": {
        "total": 8,
        "thisMonth": 3,
        "conversionRate": 0.18
      }
    },
    "trends": {
      "applicationsByDay": [
        { "date": "2025-07-04", "count": 2 },
        { "date": "2025-07-05", "count": 1 },
        { "date": "2025-07-06", "count": 0 }
      ],
      "topSkillsInDemand": [
        { "skill": "React", "frequency": 0.85 },
        { "skill": "AWS", "frequency": 0.73 },
        { "skill": "Python", "frequency": 0.68 }
      ]
    },
    "recommendations": [
      {
        "type": "skill_improvement",
        "message": "Consider adding Docker skills - it appears in 75% of relevant job postings"
      },
      {
        "type": "application_strategy",
        "message": "Your application success rate is higher for remote positions"
      }
    ]
  }
}
```

## Webhooks

The platform supports webhooks for real-time notifications about important events.

### Webhook Configuration

#### POST /api/v1/webhooks
Register a webhook endpoint.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "url": "https://your-app.com/webhooks/hr-agent",
  "events": [
    "application.status_changed",
    "job.new_match",
    "cv.analysis_completed",
    "auto_apply.completed"
  ],
  "secret": "your_webhook_secret_for_verification"
}
```

### Webhook Events

#### application.status_changed
Triggered when an application status changes.

```json
{
  "event": "application.status_changed",
  "data": {
    "applicationId": "64f8a1b2c3d4e5f6a7b8c9d5",
    "jobId": "64f8a1b2c3d4e5f6a7b8c9d1",
    "userId": "64f8a1b2c3d4e5f6a7b8c9d0",
    "oldStatus": "pending",
    "newStatus": "reviewed",
    "changedAt": "2025-08-03T15:30:00Z",
    "note": "Application reviewed by HR"
  },
  "timestamp": "2025-08-03T15:30:00Z",
  "id": "webhook_64f8a1b2c3d4e5f6a7b8c9dc"
}
```

#### job.new_match
Triggered when a new job matches user criteria.

```json
{
  "event": "job.new_match",
  "data": {
    "jobId": "64f8a1b2c3d4e5f6a7b8c9d1",
    "userId": "64f8a1b2c3d4e5f6a7b8c9d0",
    "matchScore": 0.92,
    "job": {
      "title": "Senior Software Engineer",
      "company": "TechCorp",
      "location": "San Francisco, CA"
    }
  },
  "timestamp": "2025-08-03T16:00:00Z",
  "id": "webhook_64f8a1b2c3d4e5f6a7b8c9dd"
}
```

## SDK Examples

### JavaScript/Node.js SDK

```javascript
// hr-agent-sdk.js
class HRAgentSDK {
  constructor(apiKey, baseUrl = 'https://api.hr-agent.com') {
    this.apiKey = apiKey;
    this.baseUrl = baseUrl;
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseUrl}${endpoint}`;
    const response = await fetch(url, {
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
        ...options.headers
      },
      ...options
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error.message);
    }

    return response.json();
  }

  // Authentication
  async login(email, password) {
    return this.request('/api/v1/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
  }

  // Jobs
  async getJobs(filters = {}) {
    const params = new URLSearchParams(filters);
    return this.request(`/api/v1/jobs?${params}`);
  }

  async getJob(jobId) {
    return this.request(`/api/v1/jobs/${jobId}`);
  }

  async applyToJob(jobId, cvId, coverLetter) {
    return this.request(`/api/v1/jobs/${jobId}/apply`, {
      method: 'POST',
      body: JSON.stringify({ cvId, coverLetter })
    });
  }

  // CV Management
  async getCVs() {
    return this.request('/api/v1/cv');
  }

  async createCV(cvData) {
    return this.request('/api/v1/cv', {
      method: 'POST',
      body: JSON.stringify(cvData)
    });
  }

  async exportCV(cvId, format = 'pdf') {
    return this.request(`/api/v1/cv/${cvId}/export?format=${format}`, {
      method: 'POST'
    });
  }

  // AI Agent
  async chatWithAgent(message, context = {}) {
    return this.request('/api/v1/agent/chat', {
      method: 'POST',
      body: JSON.stringify({ message, context })
    });
  }

  async analyzeCV(cvId, targetRole, targetIndustry) {
    return this.request('/api/v1/agent/analyze-cv', {
      method: 'POST',
      body: JSON.stringify({ cvId, targetRole, targetIndustry })
    });
  }

  async autoApplyJobs(cvId, criteria, preferences = {}) {
    return this.request('/api/v1/agent/auto-apply', {
      method: 'POST',
      body: JSON.stringify({ cvId, criteria, preferences })
    });
  }

  // Applications
  async getApplications(filters = {}) {
    const params = new URLSearchParams(filters);
    return this.request(`/api/v1/applications?${params}`);
  }
}

// Usage example
const sdk = new HRAgentSDK('your-api-key');

// Login and search for jobs
const auth = await sdk.login('user@example.com', 'password');
const jobs = await sdk.getJobs({ 
  q: 'software engineer', 
  location: 'San Francisco',
  jobType: 'full-time'
});

// Create and analyze CV
const cv = await sdk.createCV({
  name: 'My CV',
  personalInfo: { /* ... */ },
  experience: [ /* ... */ ]
});

const analysis = await sdk.analyzeCV(cv.data.id, 'Senior Software Engineer', 'Technology');

// Auto-apply to jobs
const autoApply = await sdk.autoApplyJobs(cv.data.id, {
  location: ['San Francisco', 'Remote'],
  salaryMin: 100000
});
```

### Python SDK

```python
# hr_agent_sdk.py
import requests
from typing import Dict, List, Optional

class HRAgentSDK:
    def __init__(self, api_key: str, base_url: str = "https://api.hr-agent.com"):
        self.api_key = api_key
        self.base_url = base_url
        self.session = requests.Session()
        self.session.headers.update({
            "Authorization": f"Bearer {api_key}",
            "Content-Type": "application/json"
        })

    def _request(self, endpoint: str, method: str = "GET", data: Dict = None) -> Dict:
        url = f"{self.base_url}{endpoint}"
        response = self.session.request(method, url, json=data)
        
        if not response.ok:
            error_data = response.json()
            raise Exception(error_data["error"]["message"])
        
        return response.json()

    # Authentication
    def login(self, email: str, password: str) -> Dict:
        return self._request("/api/v1/auth/login", "POST", {
            "email": email,
            "password": password
        })

    # Jobs
    def get_jobs(self, filters: Dict = None) -> Dict:
        params = "?" + "&".join(f"{k}={v}" for k, v in (filters or {}).items())
        return self._request(f"/api/v1/jobs{params}")

    def get_job(self, job_id: str) -> Dict:
        return self._request(f"/api/v1/jobs/{job_id}")

    def apply_to_job(self, job_id: str, cv_id: str, cover_letter: str) -> Dict:
        return self._request(f"/api/v1/jobs/{job_id}/apply", "POST", {
            "cvId": cv_id,
            "coverLetter": cover_letter
        })

    # CV Management
    def get_cvs(self) -> Dict:
        return self._request("/api/v1/cv")

    def create_cv(self, cv_data: Dict) -> Dict:
        return self._request("/api/v1/cv", "POST", cv_data)

    def export_cv(self, cv_id: str, format: str = "pdf") -> Dict:
        return self._request(f"/api/v1/cv/{cv_id}/export?format={format}", "POST")

    # AI Agent
    def chat_with_agent(self, message: str, context: Dict = None) -> Dict:
        return self._request("/api/v1/agent/chat", "POST", {
            "message": message,
            "context": context or {}
        })

    def analyze_cv(self, cv_id: str, target_role: str, target_industry: str) -> Dict:
        return self._request("/api/v1/agent/analyze-cv", "POST", {
            "cvId": cv_id,
            "targetRole": target_role,
            "targetIndustry": target_industry
        })

    def auto_apply_jobs(self, cv_id: str, criteria: Dict, preferences: Dict = None) -> Dict:
        return self._request("/api/v1/agent/auto-apply", "POST", {
            "cvId": cv_id,
            "criteria": criteria,
            "preferences": preferences or {}
        })

# Usage example
sdk = HRAgentSDK("your-api-key")

# Login and search for jobs
auth = sdk.login("user@example.com", "password")
jobs = sdk.get_jobs({
    "q": "software engineer",
    "location": "San Francisco",
    "jobType": "full-time"
})

# Create and analyze CV
cv = sdk.create_cv({
    "name": "My CV",
    "personalInfo": {"firstName": "John", "lastName": "Doe"},
    "experience": [{"company": "TechCorp", "position": "Developer"}]
})

analysis = sdk.analyze_cv(
    cv["data"]["id"], 
    "Senior Software Engineer", 
    "Technology"
)

# Auto-apply to jobs
auto_apply = sdk.auto_apply_jobs(cv["data"]["id"], {
    "location": ["San Francisco", "Remote"],
    "salaryMin": 100000
})
```

This API documentation provides comprehensive coverage of all endpoints, authentication methods, error handling, and practical examples for integrating with the HR Agent platform.
