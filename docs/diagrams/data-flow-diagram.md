# Data Flow Diagrams (DFD)

## Overview

This document contains comprehensive Data Flow Diagrams for the AI-Powered Job Seeker Platform, showing how data moves through the system at different levels of detail.

## Table of Contents
1. [Context Diagram (Level 0)](#context-diagram-level-0)
2. [System Overview Diagram (Level 1)](#system-overview-diagram-level-1)
3. [Detailed Process Diagrams (Level 2)](#detailed-process-diagrams-level-2)
4. [Data Dictionary](#data-dictionary)

## Context Diagram (Level 0)

The context diagram shows the system as a single process with external entities and data flows.

```mermaid
graph TD
    %% External Entities
    JS[Job Seekers]
    R[Recruiters]
    JB[Job Boards<br/>LinkedIn, Indeed, Glassdoor]
    AI[AI Services<br/>OpenAI, Claude]
    ES[Email Service<br/>SendGrid]
    SMS[SMS Service<br/>Twilio]
    PS[Payment System<br/>Stripe]
    
    %% Main System
    HRP[HR Platform<br/>System]
    
    %% Job Seeker Flows
    JS -->|Registration Data, CV Data, Job Applications| HRP
    HRP -->|Job Recommendations, Application Status, Notifications| JS
    
    %% Recruiter Flows
    R -->|Job Postings, Candidate Reviews, Interview Scheduling| HRP
    HRP -->|Candidate Profiles, Application Analytics, Notifications| R
    
    %% External Service Flows
    HRP -->|Job Search Queries| JB
    JB -->|Job Listings, Company Data| HRP
    
    HRP -->|CV Analysis, Job Matching Requests| AI
    AI -->|Analysis Results, Recommendations| HRP
    
    HRP -->|Email Templates, Notifications| ES
    ES -->|Delivery Status| HRP
    
    HRP -->|SMS Alerts| SMS
    SMS -->|Delivery Status| HRP
    
    HRP -->|Payment Requests| PS
    PS -->|Payment Confirmations| HRP
    
    %% Styling
    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef system fill:#f3e5f5,stroke:#4a148c,stroke-width:3px
    
    class JS,R,JB,AI,ES,SMS,PS external
    class HRP system
```

## System Overview Diagram (Level 1)

The Level 1 DFD breaks down the main system into major processes and data stores.

```mermaid
graph TD
    %% External Entities
    JS[Job Seekers]
    R[Recruiters]
    JB[Job Boards]
    AI[AI Services]
    NS[Notification Services]
    
    %% Main Processes
    P1[1.0<br/>User Management<br/>& Authentication]
    P2[2.0<br/>CV Management<br/>& Builder]
    P3[3.0<br/>Job Management<br/>& Search]
    P4[4.0<br/>Application<br/>Processing]
    P5[5.0<br/>AI Analysis<br/>& Matching]
    P6[6.0<br/>Workflow<br/>Automation]
    P7[7.0<br/>Notification<br/>Management]
    P8[8.0<br/>Analytics<br/>& Reporting]
    
    %% Data Stores
    D1[(D1: Users)]
    D2[(D2: CVs)]
    D3[(D3: Jobs)]
    D4[(D4: Applications)]
    D5[(D5: AI Models)]
    D6[(D6: Analytics)]
    D7[(D7: Notifications)]
    D8[(D8: Sessions)]
    
    %% User Management Flows
    JS -->|Registration, Login| P1
    R -->|Registration, Login| P1
    P1 -->|User Data| D1
    P1 <-->|Session Data| D8
    P1 -->|Authentication Status| JS
    P1 -->|Authentication Status| R
    
    %% CV Management Flows
    JS -->|CV Data, Templates| P2
    P2 <-->|CV Storage| D2
    P2 -->|CV Updates| JS
    P2 -->|CV Data| P5
    
    %% Job Management Flows
    R -->|Job Postings| P3
    JB -->|External Jobs| P3
    P3 <-->|Job Storage| D3
    P3 -->|Job Listings| JS
    P3 -->|Job Data| P5
    
    %% Application Processing Flows
    JS -->|Applications| P4
    P4 <-->|Application Data| D4
    P4 -->|Application Status| JS
    P4 -->|Application Updates| R
    P4 -->|Application Data| P6
    
    %% AI Analysis Flows
    P5 -->|Analysis Requests| AI
    AI -->|Analysis Results| P5
    P5 <-->|Model Data| D5
    P5 -->|Recommendations| JS
    P5 -->|Match Results| P4
    
    %% Workflow Automation Flows
    P6 -->|Job Scraping| JB
    P6 -->|Auto Applications| P4
    P6 -->|Notification Triggers| P7
    P6 -->|Analytics Events| P8
    
    %% Notification Flows
    P7 -->|Email/SMS| NS
    P7 <-->|Notification Logs| D7
    P7 -->|Notifications| JS
    P7 -->|Notifications| R
    
    %% Analytics Flows
    P8 <-->|Analytics Data| D6
    P8 -->|Reports| R
    P8 -->|Insights| JS
    
    %% Styling
    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef datastore fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    
    class JS,R,JB,AI,NS external
    class P1,P2,P3,P4,P5,P6,P7,P8 process
    class D1,D2,D3,D4,D5,D6,D7,D8 datastore
```

## Detailed Process Diagrams (Level 2)

### 2.1 User Management & Authentication (Process 1.0)

```mermaid
graph TD
    %% External Entities
    JS[Job Seekers]
    R[Recruiters]
    
    %% Sub-processes
    P11[1.1<br/>User Registration]
    P12[1.2<br/>User Authentication]
    P13[1.3<br/>Profile Management]
    P14[1.4<br/>Role Management]
    
    %% Data Stores
    D1[(D1: Users)]
    D8[(D8: Sessions)]
    D9[(D9: Audit Logs)]
    
    %% Registration Flow
    JS -->|Registration Data| P11
    R -->|Registration Data| P11
    P11 -->|Validate & Store| D1
    P11 -->|Registration Status| JS
    P11 -->|Registration Status| R
    P11 -->|Audit Entry| D9
    
    %% Authentication Flow
    JS -->|Login Credentials| P12
    R -->|Login Credentials| P12
    P12 -->|Verify| D1
    P12 -->|Session Data| D8
    P12 -->|Auth Token| JS
    P12 -->|Auth Token| R
    P12 -->|Login Event| D9
    
    %% Profile Management Flow
    JS -->|Profile Updates| P13
    R -->|Profile Updates| P13
    P13 <-->|Profile Data| D1
    P13 -->|Updated Profile| JS
    P13 -->|Updated Profile| R
    
    %% Role Management Flow
    P14 <-->|Role Data| D1
    P14 -->|Role Assignments| P13
    P14 -->|Permission Updates| D8
    
    %% Styling
    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef datastore fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    
    class JS,R external
    class P11,P12,P13,P14 process
    class D1,D8,D9 datastore
```

### 2.2 CV Management & Builder (Process 2.0)

```mermaid
graph TD
    %% External Entities
    JS[Job Seekers]
    AI[AI Services]
    
    %% Sub-processes
    P21[2.1<br/>CV Creation]
    P22[2.2<br/>CV Editing]
    P23[2.3<br/>Template Management]
    P24[2.4<br/>CV Export]
    P25[2.5<br/>CV Analysis]
    
    %% Data Stores
    D2[(D2: CVs)]
    D10[(D10: Templates)]
    D11[(D11: Skills Database)]
    D5[(D5: AI Models)]
    
    %% CV Creation Flow
    JS -->|CV Data, Template Choice| P21
    P21 -->|Template| D10
    P21 -->|Skills Validation| D11
    P21 -->|New CV| D2
    P21 -->|Created CV| JS
    
    %% CV Editing Flow
    JS -->|CV Updates| P22
    P22 <-->|CV Data| D2
    P22 -->|Updated CV| JS
    P22 -->|CV for Analysis| P25
    
    %% Template Management Flow
    P23 <-->|Templates| D10
    P23 -->|Available Templates| P21
    P23 -->|Template Updates| P22
    
    %% CV Export Flow
    JS -->|Export Request| P24
    P24 -->|CV Data| D2
    P24 -->|Template| D10
    P24 -->|PDF/DOCX| JS
    
    %% CV Analysis Flow
    P25 -->|Analysis Request| AI
    AI -->|Analysis Results| P25
    P25 <-->|Analysis Data| D5
    P25 -->|Recommendations| JS
    P25 -->|Skills Extracted| D11
    
    %% Styling
    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef datastore fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    
    class JS,AI external
    class P21,P22,P23,P24,P25 process
    class D2,D10,D11,D5 datastore
```

### 2.3 Job Management & Search (Process 3.0)

```mermaid
graph TD
    %% External Entities
    R[Recruiters]
    JS[Job Seekers]
    JB[Job Boards]
    
    %% Sub-processes
    P31[3.1<br/>Job Posting]
    P32[3.2<br/>Job Search]
    P33[3.3<br/>Job Scraping]
    P34[3.4<br/>Job Matching]
    P35[3.5<br/>Job Management]
    
    %% Data Stores
    D3[(D3: Jobs)]
    D12[(D12: Companies)]
    D13[(D13: Locations)]
    D14[(D14: Search History)]
    
    %% Job Posting Flow
    R -->|Job Details| P31
    P31 -->|Company Data| D12
    P31 -->|Location Data| D13
    P31 -->|New Job| D3
    P31 -->|Posted Job| R
    
    %% Job Search Flow
    JS -->|Search Criteria| P32
    P32 -->|Search Query| D3
    P32 -->|Search History| D14
    P32 -->|Job Results| JS
    P32 -->|Matching Request| P34
    
    %% Job Scraping Flow
    P33 -->|Scraping Request| JB
    JB -->|Job Data| P33
    P33 -->|External Jobs| D3
    P33 -->|Company Updates| D12
    
    %% Job Matching Flow
    P34 -->|Job Data| D3
    P34 -->|Matched Jobs| JS
    P34 -->|Match Analytics| D14
    
    %% Job Management Flow
    R -->|Job Updates| P35
    P35 <-->|Job Data| D3
    P35 -->|Updated Job| R
    P35 -->|Status Changes| JS
    
    %% Styling
    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef datastore fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    
    class R,JS,JB external
    class P31,P32,P33,P34,P35 process
    class D3,D12,D13,D14 datastore
```

### 2.4 Application Processing (Process 4.0)

```mermaid
graph TD
    %% External Entities
    JS[Job Seekers]
    R[Recruiters]
    JB[Job Boards]
    
    %% Sub-processes
    P41[4.1<br/>Application<br/>Submission]
    P42[4.2<br/>Application<br/>Tracking]
    P43[4.3<br/>Auto-Apply<br/>Processing]
    P44[4.4<br/>Application<br/>Review]
    P45[4.5<br/>Interview<br/>Scheduling]
    
    %% Data Stores
    D4[(D4: Applications)]
    D2[(D2: CVs)]
    D3[(D3: Jobs)]
    D15[(D15: Interviews)]
    D16[(D16: Application History)]
    
    %% Application Submission Flow
    JS -->|Application Data| P41
    P41 -->|CV Data| D2
    P41 -->|Job Data| D3
    P41 -->|New Application| D4
    P41 -->|Application Confirmation| JS
    P41 -->|Application Alert| R
    
    %% Application Tracking Flow
    JS -->|Status Request| P42
    R -->|Status Update| P42
    P42 <-->|Application Status| D4
    P42 -->|Status History| D16
    P42 -->|Status Update| JS
    
    %% Auto-Apply Processing Flow
    P43 -->|Application Data| JB
    JB -->|Submission Status| P43
    P43 -->|Auto Application| D4
    P43 -->|Auto-Apply Report| JS
    
    %% Application Review Flow
    R -->|Review Data| P44
    P44 <-->|Application Data| D4
    P44 -->|Review Results| R
    P44 -->|Review Status| JS
    P44 -->|Interview Request| P45
    
    %% Interview Scheduling Flow
    P45 <-->|Interview Data| D15
    P45 -->|Interview Details| JS
    P45 -->|Interview Details| R
    P45 -->|Calendar Updates| D4
    
    %% Styling
    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef datastore fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    
    class JS,R,JB external
    class P41,P42,P43,P44,P45 process
    class D4,D2,D3,D15,D16 datastore
```

### 2.5 AI Analysis & Matching (Process 5.0)

```mermaid
graph TD
    %% External Entities
    AI[AI Services<br/>OpenAI, Claude]
    
    %% Sub-processes
    P51[5.1<br/>CV Analysis]
    P52[5.2<br/>Job Matching]
    P53[5.3<br/>Skill Extraction]
    P54[5.4<br/>Recommendation<br/>Engine]
    P55[5.5<br/>Conversational<br/>AI]
    
    %% Data Stores
    D2[(D2: CVs)]
    D3[(D3: Jobs)]
    D5[(D5: AI Models)]
    D17[(D17: Embeddings)]
    D18[(D18: Chat History)]
    D11[(D11: Skills Database)]
    
    %% CV Analysis Flow
    P51 -->|CV Text| AI
    AI -->|Analysis Results| P51
    P51 -->|CV Data| D2
    P51 -->|Analysis Report| D5
    P51 -->|Extracted Skills| P53
    
    %% Job Matching Flow
    P52 -->|CV & Job Data| AI
    AI -->|Match Scores| P52
    P52 -->|CV Embeddings| D17
    P52 -->|Job Embeddings| D17
    P52 -->|Match Results| D5
    
    %% Skill Extraction Flow
    P53 -->|Text Analysis| AI
    AI -->|Extracted Skills| P53
    P53 -->|Skills Update| D11
    P53 -->|Skill Mappings| D5
    
    %% Recommendation Engine Flow
    P54 -->|User Preferences| D5
    P54 -->|Job Recommendations| AI
    AI -->|Personalized Recommendations| P54
    P54 -->|Recommendation History| D5
    
    %% Conversational AI Flow
    P55 -->|Chat Query| AI
    AI -->|Chat Response| P55
    P55 <-->|Conversation| D18
    P55 -->|Context Data| D5
    
    %% Styling
    classDef external fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef process fill:#e8f5e8,stroke:#2e7d32,stroke-width:2px
    classDef datastore fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    
    class AI external
    class P51,P52,P53,P54,P55 process
    class D2,D3,D5,D17,D18,D11 datastore
```

## Data Dictionary

### Data Stores

| ID | Name | Description | Data Elements |
|----|------|-------------|---------------|
| D1 | Users | User account information | user_id, email, password_hash, profile_data, role, created_at, updated_at |
| D2 | CVs | Resume/CV documents and data | cv_id, user_id, title, sections, template_id, version, created_at |
| D3 | Jobs | Job postings and listings | job_id, title, company, description, requirements, location, salary, posted_at |
| D4 | Applications | Job applications and status | app_id, user_id, job_id, cv_id, status, applied_at, updated_at |
| D5 | AI Models | AI analysis data and models | model_id, type, data, embeddings, created_at, updated_at |
| D6 | Analytics | System analytics and metrics | event_id, user_id, event_type, data, timestamp |
| D7 | Notifications | Notification logs and templates | notif_id, user_id, type, content, status, sent_at |
| D8 | Sessions | User session data | session_id, user_id, token, expires_at, created_at |
| D9 | Audit Logs | System audit trail | log_id, user_id, action, details, timestamp |
| D10 | Templates | CV templates | template_id, name, structure, preview, created_at |
| D11 | Skills Database | Skills and competencies | skill_id, name, category, synonyms, level |
| D12 | Companies | Company information | company_id, name, description, industry, location |
| D13 | Locations | Geographic locations | location_id, city, state, country, coordinates |
| D14 | Search History | User search patterns | search_id, user_id, query, filters, timestamp |
| D15 | Interviews | Interview scheduling | interview_id, app_id, datetime, type, status |
| D16 | Application History | Application status history | history_id, app_id, old_status, new_status, timestamp |
| D17 | Embeddings | Vector embeddings for ML | embedding_id, entity_type, entity_id, vector_data |
| D18 | Chat History | AI conversation history | chat_id, user_id, message, response, timestamp |

### Data Flows

| Flow | Description | Data Elements |
|------|-------------|---------------|
| Registration Data | User signup information | email, password, name, role, preferences |
| Login Credentials | Authentication data | email/username, password, 2FA_code |
| CV Data | Resume information | personal_info, experience, education, skills, achievements |
| Job Posting | Job creation data | title, description, requirements, company, location, salary |
| Application Data | Job application info | job_id, cv_id, cover_letter, additional_documents |
| Search Criteria | Job search parameters | keywords, location, salary_range, job_type, experience_level |
| Analysis Results | AI analysis output | scores, recommendations, extracted_data, confidence_levels |
| Notification Data | Alert information | recipient, type, content, priority, delivery_method |

### External Entities

| Entity | Description | Data In | Data Out |
|--------|-------------|---------|-----------|
| Job Seekers | Platform users seeking employment | Registration, CV, Applications, Search | Job recommendations, Notifications, Reports |
| Recruiters | Platform users posting jobs | Registration, Job postings, Reviews | Applications, Analytics, Notifications |
| Job Boards | External job listing sites | Job data, Company info | Scraping requests, API calls |
| AI Services | External AI/ML providers | Analysis requests, Text data | Analysis results, Recommendations |
| Notification Services | Email/SMS providers | Messages, Recipients | Delivery status, Responses |
| Payment System | Payment processing | Payment requests, User data | Transaction status, Receipts |

This comprehensive DFD documentation provides a complete view of how data flows through the AI-Powered Job Seeker Platform at multiple levels of detail, enabling developers and stakeholders to understand the system's data processing requirements and dependencies.
