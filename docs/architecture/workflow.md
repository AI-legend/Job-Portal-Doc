# Workflow Architecture with n8n

## Overview

The workflow automation system using n8n handles complex business processes, data synchronization, and automated tasks across the platform. This includes job scraping, application processing, notifications, and analytics.

## n8n Architecture

### Workflow Engine Structure
```
Workflow Engine (n8n)
├── Core Workflows
│   ├── Job Scraping Workflow
│   ├── Application Processing
│   ├── Notification System
│   ├── Data Synchronization
│   └── Analytics Pipeline
├── External Integrations
│   ├── LinkedIn API
│   ├── Indeed Scraper
│   ├── Glassdoor Integration
│   ├── Email Services
│   └── SMS Providers
├── Internal Services
│   ├── Database Operations
│   ├── File Processing
│   ├── AI Service Calls
│   └── API Endpoints
└── Monitoring & Alerts
    ├── Workflow Health
    ├── Error Handling
    ├── Performance Metrics
    └── Audit Logs
```

## Workflow Design Patterns

### 1. Job Scraping Workflow
```
workflows/job-scraping/
├── linkedin-scraper.json
├── indeed-scraper.json
├── glassdoor-scraper.json
├── job-aggregation.json
└── data-validation.json
```

#### LinkedIn Job Scraping Workflow
```json
{
  "name": "LinkedIn Job Scraper",
  "nodes": [
    {
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "hour": 6,
              "minute": 0
            },
            {
              "hour": 12,
              "minute": 0
            },
            {
              "hour": 18,
              "minute": 0
            }
          ]
        }
      },
      "name": "Schedule Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "position": [250, 300]
    },
    {
      "parameters": {
        "url": "https://www.linkedin.com/jobs/search",
        "responseFormat": "json",
        "options": {
          "headers": {
            "User-Agent": "Mozilla/5.0 (compatible; JobBot/1.0)"
          }
        }
      },
      "name": "Fetch LinkedIn Jobs",
      "type": "n8n-nodes-base.httpRequest",
      "position": [450, 300]
    },
    {
      "parameters": {
        "functionCode": "// Extract job data from LinkedIn response\nconst jobs = [];\nconst htmlContent = items[0].json.data;\n\n// Parse HTML and extract job listings\nconst jobElements = parseJobElements(htmlContent);\n\njobElements.forEach(element => {\n  jobs.push({\n    title: extractTitle(element),\n    company: extractCompany(element),\n    location: extractLocation(element),\n    description: extractDescription(element),\n    postedDate: extractPostedDate(element),\n    url: extractJobUrl(element),\n    source: 'linkedin'\n  });\n});\n\nreturn jobs.map(job => ({ json: job }));"
      },
      "name": "Parse Job Data",
      "type": "n8n-nodes-base.function",
      "position": [650, 300]
    },
    {
      "parameters": {
        "resource": "job",
        "operation": "create",
        "requestOptions": {
          "url": "{{$env.API_BASE_URL}}/api/v1/jobs/bulk",
          "method": "POST",
          "headers": {
            "Authorization": "Bearer {{$env.API_TOKEN}}",
            "Content-Type": "application/json"
          }
        }
      },
      "name": "Save to Database",
      "type": "n8n-nodes-base.httpRequest",
      "position": [850, 300]
    },
    {
      "parameters": {
        "conditions": {
          "boolean": [
            {
              "value1": "={{$json.statusCode}}",
              "operation": "equal",
              "value2": 201
            }
          ]
        }
      },
      "name": "Check Save Success",
      "type": "n8n-nodes-base.if",
      "position": [1050, 300]
    },
    {
      "parameters": {
        "message": "Successfully scraped {{$json.count}} jobs from LinkedIn",
        "channel": "#job-scraping",
        "webhook": "{{$env.SLACK_WEBHOOK_URL}}"
      },
      "name": "Notify Success",
      "type": "n8n-nodes-base.slack",
      "position": [1250, 200]
    },
    {
      "parameters": {
        "message": "Error scraping LinkedIn jobs: {{$json.error}}",
        "channel": "#alerts",
        "webhook": "{{$env.SLACK_WEBHOOK_URL}}"
      },
      "name": "Notify Error",
      "type": "n8n-nodes-base.slack",
      "position": [1250, 400]
    }
  ],
  "connections": {
    "Schedule Trigger": {
      "main": [
        [
          {
            "node": "Fetch LinkedIn Jobs",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Fetch LinkedIn Jobs": {
      "main": [
        [
          {
            "node": "Parse Job Data",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Parse Job Data": {
      "main": [
        [
          {
            "node": "Save to Database",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Save to Database": {
      "main": [
        [
          {
            "node": "Check Save Success",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Check Save Success": {
      "main": [
        [
          {
            "node": "Notify Success",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Notify Error",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

### 2. Auto-Apply Workflow
```json
{
  "name": "Auto Apply to Jobs",
  "nodes": [
    {
      "parameters": {
        "resource": "webhook",
        "httpMethod": "POST",
        "path": "auto-apply"
      },
      "name": "Webhook Trigger",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "url": "{{$env.API_BASE_URL}}/api/v1/cv/{{$json.cvId}}",
        "headers": {
          "Authorization": "Bearer {{$json.userToken}}"
        }
      },
      "name": "Fetch User CV",
      "type": "n8n-nodes-base.httpRequest",
      "position": [450, 300]
    },
    {
      "parameters": {
        "url": "{{$env.AI_SERVICE_URL}}/api/job-match",
        "method": "POST",
        "body": {
          "cv": "{{$json.cv}}",
          "criteria": "{{$json.criteria}}"
        },
        "headers": {
          "Authorization": "Bearer {{$env.AI_SERVICE_TOKEN}}"
        }
      },
      "name": "Find Matching Jobs",
      "type": "n8n-nodes-base.httpRequest",
      "position": [650, 300]
    },
    {
      "parameters": {
        "functionCode": "// Filter jobs based on match score and user preferences\nconst matchingJobs = items[0].json.matches;\nconst filteredJobs = matchingJobs.filter(job => {\n  return job.matchScore >= 0.7 && // 70% match or higher\n         job.canAutoApply === true && // Platform supports auto-apply\n         !job.alreadyApplied; // Haven't applied yet\n});\n\nreturn filteredJobs.map(job => ({ json: job }));"
      },
      "name": "Filter Applicable Jobs",
      "type": "n8n-nodes-base.function",
      "position": [850, 300]
    },
    {
      "parameters": {
        "batchSize": 5,
        "options": {
          "reset": false
        }
      },
      "name": "Split Into Batches",
      "type": "n8n-nodes-base.splitInBatches",
      "position": [1050, 300]
    },
    {
      "parameters": {
        "url": "{{$env.AI_SERVICE_URL}}/api/auto-apply",
        "method": "POST",
        "body": {
          "jobId": "{{$json.id}}",
          "cvId": "{{$json.cvId}}",
          "coverLetter": "{{$json.generatedCoverLetter}}",
          "platform": "{{$json.source}}"
        }
      },
      "name": "Submit Application",
      "type": "n8n-nodes-base.httpRequest",
      "position": [1250, 300]
    },
    {
      "parameters": {
        "amount": 2,
        "unit": "seconds"
      },
      "name": "Rate Limit Delay",
      "type": "n8n-nodes-base.wait",
      "position": [1450, 300]
    },
    {
      "parameters": {
        "url": "{{$env.API_BASE_URL}}/api/v1/applications",
        "method": "POST",
        "body": {
          "jobId": "{{$json.jobId}}",
          "userId": "{{$json.userId}}",
          "cvId": "{{$json.cvId}}",
          "status": "{{$json.applicationStatus}}",
          "autoApplied": true,
          "appliedAt": "{{$now}}"
        }
      },
      "name": "Record Application",
      "type": "n8n-nodes-base.httpRequest",
      "position": [1650, 300]
    },
    {
      "parameters": {
        "resource": "email",
        "operation": "send",
        "toEmail": "{{$json.userEmail}}",
        "subject": "Auto-Applied to {{$json.jobTitle}} at {{$json.companyName}}",
        "message": "We've automatically applied to a job that matches your profile!"
      },
      "name": "Send Notification",
      "type": "n8n-nodes-base.emailSend",
      "position": [1850, 300]
    }
  ]
}
```

### 3. Notification Workflow
```json
{
  "name": "Multi-Channel Notifications",
  "nodes": [
    {
      "parameters": {
        "resource": "queue",
        "queue": "notifications"
      },
      "name": "Queue Trigger",
      "type": "n8n-nodes-base.redis",
      "position": [250, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{$json.type}}",
              "operation": "equal",
              "value2": "email"
            }
          ]
        }
      },
      "name": "Check Notification Type",
      "type": "n8n-nodes-base.switch",
      "position": [450, 300]
    },
    {
      "parameters": {
        "service": "gmail",
        "options": {
          "ccEmail": "",
          "bccEmail": ""
        },
        "subject": "={{$json.subject}}",
        "message": "={{$json.message}}",
        "toEmail": "={{$json.recipient}}"
      },
      "name": "Send Email",
      "type": "n8n-nodes-base.emailSend",
      "position": [650, 200]
    },
    {
      "parameters": {
        "resource": "message",
        "operation": "send",
        "body": "={{$json.message}}",
        "from": "{{$env.TWILIO_PHONE_NUMBER}}",
        "to": "={{$json.recipient}}"
      },
      "name": "Send SMS",
      "type": "n8n-nodes-base.twilio",
      "position": [650, 300]
    },
    {
      "parameters": {
        "url": "{{$env.PUSH_NOTIFICATION_URL}}",
        "method": "POST",
        "body": {
          "userId": "={{$json.userId}}",
          "title": "={{$json.title}}",
          "message": "={{$json.message}}",
          "data": "={{$json.data}}"
        }
      },
      "name": "Send Push Notification",
      "type": "n8n-nodes-base.httpRequest",
      "position": [650, 400]
    },
    {
      "parameters": {
        "url": "{{$env.API_BASE_URL}}/api/v1/notifications/{{$json.id}}/status",
        "method": "PATCH",
        "body": {
          "status": "sent",
          "sentAt": "{{$now}}"
        }
      },
      "name": "Update Status",
      "type": "n8n-nodes-base.httpRequest",
      "position": [850, 300]
    }
  ]
}
```

## Workflow Configuration

### Environment Setup
```yaml
# docker-compose.yml for n8n
version: '3.8'
services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
      - N8N_HOST=${N8N_HOST}
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - NODE_ENV=production
      - WEBHOOK_URL=https://your-domain.com
      - GENERIC_TIMEZONE=UTC
      - N8N_LOG_LEVEL=info
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - n8n_data:/home/node/.n8n
      - ./workflows:/home/node/.n8n/workflows
    depends_on:
      - postgres
    networks:
      - n8n_network

  postgres:
    image: postgres:13
    restart: always
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - n8n_network

volumes:
  n8n_data:
  postgres_data:

networks:
  n8n_network:
```

### Workflow Environment Variables
```bash
# .env for n8n workflows
API_BASE_URL=https://api.yourapp.com
AI_SERVICE_URL=https://ai.yourapp.com
API_TOKEN=your_api_token
AI_SERVICE_TOKEN=your_ai_service_token

# External Services
LINKEDIN_API_KEY=your_linkedin_key
INDEED_API_KEY=your_indeed_key
GLASSDOOR_API_KEY=your_glassdoor_key

# Notification Services
SENDGRID_API_KEY=your_sendgrid_key
TWILIO_ACCOUNT_SID=your_twilio_sid
TWILIO_AUTH_TOKEN=your_twilio_token
TWILIO_PHONE_NUMBER=your_twilio_number
SLACK_WEBHOOK_URL=your_slack_webhook

# Database
POSTGRES_DB=n8n_db
POSTGRES_USER=n8n_user
POSTGRES_PASSWORD=your_secure_password

# n8n Configuration
N8N_USER=admin
N8N_PASSWORD=your_n8n_password
N8N_HOST=workflows.yourapp.com
```

## Custom Nodes

### Job Scraper Node
```javascript
// nodes/JobScraper/JobScraper.node.ts
import {
  IExecuteFunctions,
  INodeExecutionData,
  INodeType,
  INodeTypeDescription,
} from 'n8n-workflow';

export class JobScraper implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'Job Scraper',
    name: 'jobScraper',
    group: ['transform'],
    version: 1,
    description: 'Scrapes job listings from various platforms',
    defaults: {
      name: 'Job Scraper',
    },
    inputs: ['main'],
    outputs: ['main'],
    properties: [
      {
        displayName: 'Platform',
        name: 'platform',
        type: 'options',
        options: [
          {
            name: 'LinkedIn',
            value: 'linkedin',
          },
          {
            name: 'Indeed',
            value: 'indeed',
          },
          {
            name: 'Glassdoor',
            value: 'glassdoor',
          },
        ],
        default: 'linkedin',
        description: 'The job platform to scrape',
      },
      {
        displayName: 'Search Keywords',
        name: 'keywords',
        type: 'string',
        default: '',
        placeholder: 'e.g., software engineer, python developer',
        description: 'Keywords to search for',
      },
      {
        displayName: 'Location',
        name: 'location',
        type: 'string',
        default: '',
        placeholder: 'e.g., New York, NY',
        description: 'Job location',
      },
      {
        displayName: 'Max Results',
        name: 'maxResults',
        type: 'number',
        default: 50,
        description: 'Maximum number of jobs to scrape',
      },
    ],
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    for (let i = 0; i < items.length; i++) {
      const platform = this.getNodeParameter('platform', i) as string;
      const keywords = this.getNodeParameter('keywords', i) as string;
      const location = this.getNodeParameter('location', i) as string;
      const maxResults = this.getNodeParameter('maxResults', i) as number;

      try {
        const scraper = new JobScraperService(platform);
        const jobs = await scraper.scrapeJobs({
          keywords,
          location,
          maxResults,
        });

        for (const job of jobs) {
          returnData.push({
            json: job,
          });
        }
      } catch (error) {
        throw new Error(`Job scraping failed: ${error.message}`);
      }
    }

    return [returnData];
  }
}
```

### AI Job Matcher Node
```javascript
// nodes/AIJobMatcher/AIJobMatcher.node.ts
export class AIJobMatcher implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'AI Job Matcher',
    name: 'aiJobMatcher',
    group: ['transform'],
    version: 1,
    description: 'Matches jobs to user CV using AI',
    defaults: {
      name: 'AI Job Matcher',
    },
    inputs: ['main'],
    outputs: ['main'],
    properties: [
      {
        displayName: 'CV Data',
        name: 'cvData',
        type: 'json',
        default: '{}',
        description: 'User CV data in JSON format',
      },
      {
        displayName: 'Match Threshold',
        name: 'threshold',
        type: 'number',
        typeOptions: {
          minValue: 0,
          maxValue: 1,
          numberStepSize: 0.1,
        },
        default: 0.7,
        description: 'Minimum match score (0-1)',
      },
      {
        displayName: 'Include Reasons',
        name: 'includeReasons',
        type: 'boolean',
        default: true,
        description: 'Include match reasoning in output',
      },
    ],
  };

  async execute(this: IExecuteFunctions): Promise<INodeExecutionData[][]> {
    const items = this.getInputData();
    const returnData: INodeExecutionData[] = [];

    for (let i = 0; i < items.length; i++) {
      const cvData = this.getNodeParameter('cvData', i) as object;
      const threshold = this.getNodeParameter('threshold', i) as number;
      const includeReasons = this.getNodeParameter('includeReasons', i) as boolean;
      const jobData = items[i].json;

      try {
        const matcher = new AIJobMatcherService();
        const matchResult = await matcher.matchJob({
          cv: cvData,
          job: jobData,
          threshold,
          includeReasons,
        });

        if (matchResult.score >= threshold) {
          returnData.push({
            json: {
              ...jobData,
              matchScore: matchResult.score,
              matchReasons: includeReasons ? matchResult.reasons : undefined,
              recommendations: matchResult.recommendations,
            },
          });
        }
      } catch (error) {
        throw new Error(`AI job matching failed: ${error.message}`);
      }
    }

    return [returnData];
  }
}
```

## Workflow Monitoring

### Health Check Workflow
```json
{
  "name": "System Health Check",
  "nodes": [
    {
      "parameters": {
        "triggerTimes": {
          "item": [
            {
              "minute": 0
            }
          ]
        }
      },
      "name": "Every Hour",
      "type": "n8n-nodes-base.scheduleTrigger",
      "position": [250, 300]
    },
    {
      "parameters": {
        "functionCode": "const services = [\n  { name: 'API Gateway', url: process.env.API_BASE_URL + '/health' },\n  { name: 'User Service', url: process.env.USER_SERVICE_URL + '/health' },\n  { name: 'Job Service', url: process.env.JOB_SERVICE_URL + '/health' },\n  { name: 'AI Service', url: process.env.AI_SERVICE_URL + '/health' }\n];\n\nreturn services.map(service => ({ json: service }));"
      },
      "name": "Get Service URLs",
      "type": "n8n-nodes-base.function",
      "position": [450, 300]
    },
    {
      "parameters": {
        "url": "={{$json.url}}",
        "options": {
          "timeout": 5000
        }
      },
      "name": "Check Service Health",
      "type": "n8n-nodes-base.httpRequest",
      "position": [650, 300]
    },
    {
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$json.statusCode}}",
              "operation": "equal",
              "value2": 200
            }
          ]
        }
      },
      "name": "Check Status",
      "type": "n8n-nodes-base.if",
      "position": [850, 300]
    },
    {
      "parameters": {
        "message": "🚨 Service {{$node['Get Service URLs'].json.name}} is down! Status: {{$json.statusCode}}",
        "channel": "#alerts"
      },
      "name": "Alert on Failure",
      "type": "n8n-nodes-base.slack",
      "position": [1050, 400]
    },
    {
      "parameters": {
        "functionCode": "const results = items.map(item => ({\n  service: item.json.name,\n  status: item.json.statusCode === 200 ? 'healthy' : 'unhealthy',\n  responseTime: item.json.responseTime,\n  timestamp: new Date().toISOString()\n}));\n\nreturn [{ json: { healthCheck: results } }];"
      },
      "name": "Aggregate Results",
      "type": "n8n-nodes-base.function",
      "position": [1050, 200]
    }
  ]
}
```

### Performance Monitoring
```javascript
// workflows/performance-monitoring.js
const performanceWorkflow = {
  name: "Performance Monitoring",
  nodes: [
    {
      name: "Schedule Trigger",
      type: "n8n-nodes-base.scheduleTrigger",
      parameters: {
        rule: {
          interval: [{ field: "minutes", value: 5 }]
        }
      }
    },
    {
      name: "Collect Metrics",
      type: "n8n-nodes-base.function",
      parameters: {
        functionCode: `
          const metrics = await Promise.all([
            fetch('${process.env.API_BASE_URL}/metrics'),
            fetch('${process.env.AI_SERVICE_URL}/metrics'),
            fetch('${process.env.SCRAPER_SERVICE_URL}/metrics')
          ]);

          const data = await Promise.all(
            metrics.map(response => response.json())
          );

          return data.map(metric => ({ json: metric }));
        `
      }
    },
    {
      name: "Check Thresholds",
      type: "n8n-nodes-base.function",
      parameters: {
        functionCode: `
          const alerts = [];
          
          items.forEach(item => {
            const metric = item.json;
            
            // CPU usage > 80%
            if (metric.cpu_usage > 0.8) {
              alerts.push({
                type: 'cpu_high',
                service: metric.service,
                value: metric.cpu_usage,
                threshold: 0.8
              });
            }
            
            // Memory usage > 85%
            if (metric.memory_usage > 0.85) {
              alerts.push({
                type: 'memory_high',
                service: metric.service,
                value: metric.memory_usage,
                threshold: 0.85
              });
            }
            
            // Response time > 2 seconds
            if (metric.avg_response_time > 2000) {
              alerts.push({
                type: 'response_time_high',
                service: metric.service,
                value: metric.avg_response_time,
                threshold: 2000
              });
            }
          });

          return alerts.map(alert => ({ json: alert }));
        `
      }
    },
    {
      name: "Send Alerts",
      type: "n8n-nodes-base.slack",
      parameters: {
        channel: "#performance-alerts",
        message: "⚠️ Performance Alert: {{$json.type}} for {{$json.service}}\nValue: {{$json.value}}\nThreshold: {{$json.threshold}}"
      }
    }
  ]
};
```

## Workflow Deployment

### CI/CD Pipeline for Workflows
```yaml
# .github/workflows/deploy-n8n-workflows.yml
name: Deploy n8n Workflows

on:
  push:
    branches: [main]
    paths: ['workflows/**']
  
  pull_request:
    branches: [main]
    paths: ['workflows/**']

jobs:
  validate-workflows:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install n8n CLI
        run: npm install -g n8n
        
      - name: Validate Workflow Syntax
        run: |
          for file in workflows/*.json; do
            echo "Validating $file"
            node -e "JSON.parse(require('fs').readFileSync('$file', 'utf8'))"
          done
          
      - name: Test Workflows
        run: |
          # Start n8n in test mode
          n8n start --tunnel &
          sleep 30
          
          # Import and test workflows
          for file in workflows/*.json; do
            echo "Testing $file"
            # Add workflow testing logic here
          done

  deploy-staging:
    needs: validate-workflows
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Staging n8n
        run: |
          # Deploy workflows to staging environment
          curl -X POST "${{ secrets.N8N_STAGING_URL }}/api/v1/workflows/import" \
            -H "Authorization: Bearer ${{ secrets.N8N_API_KEY }}" \
            -H "Content-Type: application/json" \
            -d @workflows/job-scraping.json

  deploy-production:
    needs: deploy-staging
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Production n8n
        run: |
          # Deploy workflows to production environment
          for file in workflows/*.json; do
            curl -X POST "${{ secrets.N8N_PROD_URL }}/api/v1/workflows/import" \
              -H "Authorization: Bearer ${{ secrets.N8N_PROD_API_KEY }}" \
              -H "Content-Type: application/json" \
              -d @"$file"
          done
```

### Workflow Backup Strategy
```bash
#!/bin/bash
# scripts/backup-workflows.sh

# Backup n8n workflows
echo "Starting n8n workflow backup..."

# Create backup directory with timestamp
BACKUP_DIR="backups/workflows/$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Export all workflows
curl -X GET "${N8N_URL}/api/v1/workflows" \
  -H "Authorization: Bearer ${N8N_API_KEY}" \
  -o "${BACKUP_DIR}/all_workflows.json"

# Export individual workflows
WORKFLOW_IDS=$(curl -s -X GET "${N8N_URL}/api/v1/workflows" \
  -H "Authorization: Bearer ${N8N_API_KEY}" | jq -r '.data[].id')

for id in $WORKFLOW_IDS; do
  curl -X GET "${N8N_URL}/api/v1/workflows/${id}" \
    -H "Authorization: Bearer ${N8N_API_KEY}" \
    -o "${BACKUP_DIR}/workflow_${id}.json"
done

# Compress backup
tar -czf "${BACKUP_DIR}.tar.gz" -C backups/workflows "$(basename "$BACKUP_DIR")"
rm -rf "$BACKUP_DIR"

# Upload to cloud storage (optional)
aws s3 cp "${BACKUP_DIR}.tar.gz" "s3://your-backup-bucket/n8n-workflows/"

echo "Backup completed: ${BACKUP_DIR}.tar.gz"
```

## Integration Patterns

### API Integration Pattern
```javascript
// Custom API connector for internal services
class InternalAPIConnector {
  constructor(baseURL, apiKey) {
    this.baseURL = baseURL;
    this.apiKey = apiKey;
  }

  async makeRequest(endpoint, method = 'GET', data = null) {
    const url = `${this.baseURL}${endpoint}`;
    const options = {
      method,
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
      },
    };

    if (data && method !== 'GET') {
      options.body = JSON.stringify(data);
    }

    const response = await fetch(url, options);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  }

  // User service methods
  async getUser(userId) {
    return this.makeRequest(`/users/${userId}`);
  }

  async updateUserPreferences(userId, preferences) {
    return this.makeRequest(`/users/${userId}/preferences`, 'PUT', preferences);
  }

  // Job service methods
  async createJob(jobData) {
    return this.makeRequest('/jobs', 'POST', jobData);
  }

  async searchJobs(criteria) {
    const queryString = new URLSearchParams(criteria).toString();
    return this.makeRequest(`/jobs/search?${queryString}`);
  }

  // Application methods
  async createApplication(applicationData) {
    return this.makeRequest('/applications', 'POST', applicationData);
  }

  async updateApplicationStatus(applicationId, status) {
    return this.makeRequest(`/applications/${applicationId}/status`, 'PATCH', { status });
  }
}
```

### Error Handling and Retry Logic
```javascript
// Robust error handling for workflows
class WorkflowErrorHandler {
  static async executeWithRetry(fn, maxRetries = 3, delay = 1000) {
    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fn();
      } catch (error) {
        if (i === maxRetries - 1) {
          throw error;
        }
        
        // Exponential backoff
        await this.sleep(delay * Math.pow(2, i));
      }
    }
  }

  static async sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }

  static handleWorkflowError(error, context) {
    const errorData = {
      message: error.message,
      stack: error.stack,
      workflow: context.workflow,
      node: context.node,
      timestamp: new Date().toISOString(),
    };

    // Log error
    console.error('Workflow Error:', errorData);

    // Send to monitoring service
    this.sendToMonitoring(errorData);

    // Trigger alert if critical
    if (this.isCriticalError(error)) {
      this.sendAlert(errorData);
    }

    return errorData;
  }

  static isCriticalError(error) {
    const criticalPatterns = [
      'ECONNREFUSED',
      'timeout',
      'authentication',
      'authorization'
    ];

    return criticalPatterns.some(pattern => 
      error.message.toLowerCase().includes(pattern)
    );
  }

  static async sendAlert(errorData) {
    // Send to Slack, email, or other alerting system
    await fetch(process.env.ALERT_WEBHOOK_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        text: `🚨 Critical Workflow Error: ${errorData.message}`,
        attachments: [{
          color: 'danger',
          fields: [
            { title: 'Workflow', value: errorData.workflow, short: true },
            { title: 'Node', value: errorData.node, short: true },
            { title: 'Time', value: errorData.timestamp, short: true }
          ]
        }]
      })
    });
  }
}
```

This workflow architecture provides a robust foundation for automating complex business processes while maintaining scalability, reliability, and monitoring capabilities.
