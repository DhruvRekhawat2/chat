# YourFinAdvisor - System Architecture Documentation

## Table of Contents
1. [Executive Summary](#executive-summary)
2. [System Overview](#system-overview)
3. [Architecture Patterns](#architecture-patterns)
4. [Component Architecture](#component-architecture)
5. [Data Flow](#data-flow)
6. [Technology Stack](#technology-stack)
7. [Deployment Architecture](#deployment-architecture)
8. [Security Architecture](#security-architecture)
9. [Performance & Scalability](#performance--scalability)
10. [Development Workflow](#development-workflow)
11. [Monitoring & Observability](#monitoring--observability)

---

## Executive Summary

YourFinAdvisor is a comprehensive financial advisory chat platform built as a modern, scalable microservices architecture. The system provides AI-powered financial guidance through web and mobile interfaces, with real-time chat capabilities, file processing, and secure authentication.

**Key Features:**
- AI-powered financial advisory conversations
- Multi-platform support (Web + Mobile)
- Real-time streaming chat responses
- File upload and processing capabilities
- Secure JWT-based authentication
- Cloud-native deployment on Google Cloud Platform

---

## System Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              YourFinAdvisor Platform                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Web Client    │    │  Mobile Client  │    │    External Services    │  │
│  │   (React/Vite)  │    │ (React Native)  │    │     (OpenAI API)        │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                        API Gateway / Load Balancer                      │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Backend Services                              │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │  Chat Service   │  │  File Service   │  │    Auth Service         │  │  │
│  │  │   (FastAPI)     │  │   (FastAPI)     │  │    (FastAPI)            │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Data Layer                                    │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Firestore     │  │  Google Cloud   │  │    Google Cloud         │  │  │
│  │  │   (NoSQL DB)    │  │   Storage       │  │     Logging             │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Monorepo Structure

```
your-fin-advisor-chat/
├── chatservice/                    # Backend API Service
│   ├── api/v1/                    # API endpoints
│   ├── core/                      # Configuration & core logic
│   ├── db/                        # Database layer
│   ├── models/                    # Data models
│   ├── services/                  # Business logic services
│   ├── security/                  # Authentication & authorization
│   └── tests/                     # Backend tests
│
├── wealthaiagent/                 # Frontend Monorepo
│   ├── apps/
│   │   ├── web/                   # React Web Application
│   │   └── mobile/                # React Native Mobile App
│   ├── packages/                  # Shared packages
│   │   ├── hooks/                 # Shared React hooks
│   │   └── types/                 # Shared TypeScript types
│   └── scripts/                   # Build & deployment scripts
```

---

## Architecture Patterns

### 1. Microservices Architecture
- **Service Separation**: Chat, File, and Auth services are independently deployable
- **API-First Design**: RESTful APIs with OpenAPI/Swagger documentation
- **Stateless Services**: Each service maintains no local state

### 2. Event-Driven Architecture
- **Real-time Communication**: WebSocket connections for live chat
- **Streaming Responses**: Server-sent events for AI response streaming
- **Asynchronous Processing**: File upload and processing

### 3. Monorepo Pattern
- **Shared Code**: Common types and hooks across web and mobile
- **Unified Tooling**: Single build system with Turborepo
- **Consistent Development**: Shared linting, testing, and deployment

---

## Component Architecture

### Backend Service Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              FastAPI Application                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Middleware    │    │   API Router    │    │    Exception Handler    │  │
│  │   (CORS, Auth)  │    │   (v1/api)      │    │                         │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Service Layer                                 │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │  Chat Service   │  │  AI Service     │  │    File Service         │  │  │
│  │  │                 │  │                 │  │                         │  │  │
│  │  │ • Chat CRUD     │  │ • OpenAI API    │  │ • File Upload           │  │  │
│  │  │ • Message Mgmt  │  │ • Streaming     │  │ • File Processing       │  │  │
│  │  │ • Real-time     │  │ • Context Mgmt  │  │ • Storage Integration   │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Data Access Layer                             │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Firestore     │  │   Google Cloud  │  │    Configuration        │  │  │
│  │  │   Client        │  │   Storage       │  │    Management           │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Frontend Architecture

#### Web Application (React + Vite)
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Web Application                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   App Router    │    │   State Mgmt    │    │    UI Components        │  │
│  │  (React Router) │    │   (Zustand)     │    │   (Radix UI + Custom)   │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Feature Modules                               │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Chat Module   │  │   File Module   │  │    Auth Module          │  │  │
│  │  │                 │  │                 │  │                         │  │  │
│  │  │ • Chat Window   │  │ • File Upload   │  │ • Login/Logout          │  │  │
│  │  │ • Message List  │  │ • File Preview  │  │ • User Profile          │  │  │
│  │  │ • Input Area    │  │ • File Renderer │  │ • Session Mgmt          │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Service Layer                                 │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │  API Client     │  │  WebSocket      │  │    Local Storage        │  │  │
│  │  │  (HTTP Client)  │  │  Connection     │  │    Management           │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Mobile Application (React Native + Expo)
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            Mobile Application                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Navigation    │    │   State Mgmt    │    │    UI Components        │  │
│  │  (Expo Router)  │    │   (Zustand)     │    │   (NativeWind + Custom) │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Feature Modules                               │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Chat Module   │  │   File Module   │  │    Auth Module          │  │  │
│  │  │                 │  │                 │  │                         │  │  │
│  │  │ • Chat Interface│  │ • File Picker   │  │ • Biometric Auth        │  │  │
│  │  │ • Message List  │  │ • File Preview  │  │ • Secure Storage        │  │  │
│  │  │ • Voice Input   │  │ • Native Share  │  │ • Push Notifications    │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Native Services                               │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │  API Client     │  │  Device APIs    │  │    Local Storage        │  │  │
│  │  │  (HTTP Client)  │  │  (Camera, etc.) │  │    (AsyncStorage)       │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow

### Chat Flow Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   API       │    │   AI        │    │   Database  │
│  (Web/Mobile)│    │  Gateway    │    │  Service    │    │  (Firestore)│
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       │ 1. Send Message   │                   │                   │
       │──────────────────▶│                   │                   │
       │                   │ 2. Validate &     │                   │
       │                   │    Authenticate   │                   │
       │                   │──────────────────▶│                   │
       │                   │                   │ 3. Process with   │
       │                   │                   │    OpenAI API     │
       │                   │                   │──────────────────▶│
       │                   │                   │                   │ 4. Store
       │                   │                   │                   │    Message
       │                   │                   │                   │◀─────────
       │                   │                   │ 5. Stream Response│
       │                   │                   │◀──────────────────│
       │ 6. Stream Chunks  │                   │                   │
       │◀──────────────────│                   │                   │
       │                   │                   │                   │
```

### File Upload Flow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │    │   API       │    │   File      │    │   Google    │
│  (Web/Mobile)│    │  Gateway    │    │  Service    │    │   Cloud     │
│             │    │             │    │             │    │   Storage   │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       │ 1. Upload File    │                   │                   │
       │──────────────────▶│                   │                   │
       │                   │ 2. Validate File  │                   │
       │                   │    & Size         │                   │
       │                   │──────────────────▶│                   │
       │                   │                   │ 3. Store in GCS   │
       │                   │                   │──────────────────▶│
       │                   │                   │                   │ 4. Return
       │                   │                   │                   │    URL
       │                   │                   │◀──────────────────│
       │ 5. File URL       │                   │                   │
       │◀──────────────────│                   │                   │
       │                   │                   │                   │
```

---

## Technology Stack

### Backend Stack
| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Framework** | FastAPI | Latest | High-performance async web framework |
| **Language** | Python | 3.11+ | Backend development |
| **Database** | Firestore | Latest | NoSQL document database |
| **Storage** | Google Cloud Storage | Latest | File storage |
| **AI/ML** | OpenAI GPT-4o | Latest | Natural language processing |
| **Authentication** | JWT | HS256 | Token-based authentication |
| **Containerization** | Docker | Latest | Application containerization |
| **Deployment** | Google Cloud Run | Latest | Serverless container platform |

### Frontend Stack

#### Web Application
| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Framework** | React | 19.0.0 | UI library |
| **Build Tool** | Vite | 6.3.1 | Fast build tool |
| **Styling** | Tailwind CSS | 4.1.4 | Utility-first CSS framework |
| **UI Components** | Radix UI | Latest | Accessible component primitives |
| **State Management** | Zustand | 5.0.4 | Lightweight state management |
| **Routing** | React Router DOM | 7.5.3 | Client-side routing |
| **Authentication** | Clerk | 5.30.4 | Authentication service |
| **Charts** | Recharts | 2.15.3 | Data visualization |

#### Mobile Application
| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Framework** | React Native | 0.79.5 | Cross-platform mobile development |
| **Platform** | Expo | 53.0.20 | Development platform |
| **Styling** | NativeWind | 4.1.23 | Tailwind CSS for React Native |
| **Navigation** | Expo Router | 5.1.4 | File-based routing |
| **State Management** | Zustand | 4.5.0 | State management |
| **Authentication** | Clerk | 5.0.0 | Authentication service |

### DevOps & Infrastructure
| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Monorepo** | Turborepo | Latest | Build system for monorepos |
| **Package Manager** | pnpm | 8+ | Fast, disk space efficient package manager |
| **CI/CD** | Google Cloud Build | Latest | Continuous integration/deployment |
| **Container Registry** | Google Artifact Registry | Latest | Container image storage |
| **Monitoring** | Google Cloud Logging | Latest | Application logging |
| **Secrets Management** | Google Secret Manager | Latest | Secure secret storage |

---

## Deployment Architecture

### Google Cloud Platform Deployment

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Google Cloud Platform                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Cloud DNS     │    │   Load Balancer │    │    Cloud Armor          │  │
│  │   (Domain Mgmt) │    │   (HTTPS)       │    │    (DDoS Protection)    │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Cloud Run Services                            │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │  Chat Service   │  │  File Service   │  │    Auth Service         │  │  │
│  │  │   (Auto-scaling)│  │   (Auto-scaling)│  │    (Auto-scaling)       │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Data & Storage                                │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Firestore     │  │  Cloud Storage  │  │    Secret Manager       │  │  │
│  │  │   (Database)    │  │   (Files)       │  │    (Secrets)            │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Observability                                 │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │  Cloud Logging   │  │  Cloud Monitoring│  │    Error Reporting      │  │  │
│  │  │   (Logs)        │  │   (Metrics)     │  │    (Error Tracking)     │  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Deployment Pipeline

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Git       │    │   Cloud     │    │   Artifact  │    │   Cloud     │
│  Repository │    │   Build     │    │  Registry   │    │    Run      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │                   │
       │ 1. Push Code      │                   │                   │
       │──────────────────▶│                   │                   │
       │                   │ 2. Build Docker   │                   │
       │                   │    Image          │                   │
       │                   │──────────────────▶│                   │
       │                   │                   │ 3. Push Image     │
       │                   │                   │──────────────────▶│
       │                   │                   │                   │ 4. Deploy
       │                   │                   │                   │    Service
       │                   │                   │                   │◀─────────
       │                   │                   │                   │
```

---

## Security Architecture

### Authentication & Authorization

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Security Layers                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Client Auth   │    │   API Gateway   │    │    Service Layer        │  │
│  │                 │    │                 │    │                         │  │
│  │ • JWT Tokens    │    │ • CORS Policy   │    │ • Role-based Access     │  │
│  │ • Clerk Auth    │    │ • Rate Limiting │    │ • Resource Validation   │  │
│  │ • Secure Storage│    │ • Request Validation│  │ • Input Sanitization   │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Data Security                                 │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Encryption    │  │   Access Control│  │    Audit Logging         │  │  │
│  │  │                 │  │                 │  │                         │  │  │
│  │  │ • TLS/SSL       │  │ • Firestore     │  │ • Request Logging       │  │  │
│  │  │ • Data at Rest  │  │   Security Rules│  │ • Error Tracking        │  │  │
│  │  │ • Secrets Mgmt  │  │ • IAM Policies  │  │ • Performance Monitoring│  │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Security Features

1. **Authentication**
   - JWT-based token authentication
   - Clerk integration for user management
   - Secure token storage (mobile: Keychain, web: secure cookies)

2. **Authorization**
   - Role-based access control
   - Resource-level permissions
   - API endpoint protection

3. **Data Protection**
   - TLS/SSL encryption in transit
   - Data encryption at rest
   - Secure secret management via Google Secret Manager

4. **Input Validation**
   - Request validation using Pydantic models
   - SQL injection prevention
   - XSS protection

5. **Infrastructure Security**
   - Google Cloud Security Command Center
   - Cloud Armor DDoS protection
   - Regular security updates

---

## Performance & Scalability

### Scalability Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Scalability Architecture                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Horizontal    │    │   Vertical      │    │    Caching Strategy     │  │
│  │   Scaling       │    │   Scaling       │    │                         │  │
│  │                 │    │                 │    │                         │  │
│  │ • Auto-scaling  │    │ • Resource      │    │ • Response Caching      │  │
│  │ • Load Balancing│    │   Optimization  │    │ • Database Query Cache  │  │
│  │ • Microservices │    │ • Memory Mgmt   │    │ • CDN for Static Assets │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Performance Optimization                      │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Database      │  │   API           │  │    Frontend             │  │
│  │  │   Optimization  │  │   Optimization  │  │    Optimization         │  │
│  │  │                 │  │                 │  │                         │  │
│  │  │ • Indexing      │  │ • Pagination    │  │ • Code Splitting        │  │
│  │  │ • Query         │  │ • Streaming     │  │ • Lazy Loading          │  │
│  │  │   Optimization  │  │ • Compression   │  │ • Bundle Optimization   │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Performance Metrics

| Metric | Target | Monitoring |
|--------|--------|------------|
| **API Response Time** | < 200ms | Cloud Monitoring |
| **Chat Message Latency** | < 100ms | Real-time monitoring |
| **File Upload Time** | < 5s (10MB) | Performance tracking |
| **Page Load Time** | < 2s | Web Vitals |
| **Database Query Time** | < 50ms | Firestore monitoring |
| **Uptime** | 99.9% | Cloud Monitoring |

### Optimization Strategies

1. **Backend Optimization**
   - Async/await for non-blocking operations
   - Database query optimization
   - Response streaming for real-time chat
   - Connection pooling

2. **Frontend Optimization**
   - Code splitting and lazy loading
   - Bundle optimization with Vite
   - Image optimization
   - Caching strategies

3. **Infrastructure Optimization**
   - Auto-scaling based on demand
   - CDN for static assets
   - Load balancing across regions
   - Database read replicas

---

## Development Workflow

### Development Environment Setup

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Development Workflow                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Local         │    │   Staging       │    │    Production           │  │
│  │   Development   │    │   Environment   │    │    Environment          │  │
│  │                 │    │                 │    │                         │  │
│  │ • Docker Compose│    │ • Cloud Run     │    │ • Cloud Run             │  │
│  │ • Hot Reload    │    │ • Test Data     │    │ • Production Data       │  │
│  │ • Local DB      │    │ • CI/CD Pipeline│    │ • Monitoring            │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Development Tools                             │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Code Quality  │  │   Testing       │  │    Deployment           │  │
│  │  │                 │  │                 │  │                         │  │
│  │  │ • ESLint        │  │ • Jest          │  │ • Cloud Build           │  │
│  │  │ • Prettier      │  │ • MSW           │  │ • GitHub Actions        │  │
│  │  │ • TypeScript    │  │ • E2E Tests     │  │ • Automated Testing     │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Development Commands

```bash
# Monorepo Management
./scripts/install-all.sh     # Install all dependencies
./scripts/dev-all.sh         # Start all development servers
./scripts/build-all.sh       # Build all packages
./scripts/test-all.sh        # Run all tests
./scripts/lint-all.sh        # Lint all code

# Individual Services
cd chatservice
python -m uvicorn app:app --reload  # Start backend

cd wealthaiagent/apps/web
pnpm dev                        # Start web app

cd wealthaiagent/apps/mobile
pnpm start                      # Start mobile app
```

### Testing Strategy

1. **Unit Tests**
   - Backend: pytest with FastAPI TestClient
   - Frontend: Jest with React Testing Library
   - Coverage targets: >80%

2. **Integration Tests**
   - API endpoint testing
   - Database integration tests
   - External service mocking

3. **End-to-End Tests**
   - User journey testing
   - Cross-browser testing
   - Mobile app testing

4. **Performance Tests**
   - Load testing with Locust
   - API performance benchmarks
   - Frontend performance monitoring

---

## Monitoring & Observability

### Monitoring Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              Observability Stack                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Application   │    │   Infrastructure│    │    Business Metrics     │  │
│  │   Monitoring    │    │   Monitoring    │    │                         │  │
│  │                 │    │                 │    │                         │  │
│  │ • Error Tracking│    │ • Resource Usage│    │ • User Engagement       │  │
│  │ • Performance   │    │ • Auto-scaling  │    │ • Chat Volume           │  │
│  │ • API Metrics   │    │ • Health Checks │    │ • File Upload Stats     │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│           │                       │                           │              │
│           └───────────────────────┼───────────────────────────┘              │
│                                   │                                          │
│  ┌─────────────────────────────────────────────────────────────────────────┐  │
│  │                           Logging & Tracing                             │  │
│  │  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────┐  │  │
│  │  │   Structured    │  │   Distributed   │  │    Alerting             │  │
│  │  │   Logging       │  │   Tracing       │  │                         │  │
│  │  │                 │  │                 │  │                         │  │
│  │  │ • Google Cloud  │  │ • Request       │  │ • Error Alerts          │  │
│  │  │   Logging       │  │   Tracing       │  │ • Performance Alerts    │  │
│  │  │ • JSON Format   │  │ • Span Tracking │  │ • SLA Monitoring        │  │
│  │  └─────────────────┘  └─────────────────┘  └─────────────────────────┘  │
│  └─────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Key Metrics & Alerts

1. **Application Metrics**
   - Request/response times
   - Error rates and types
   - API endpoint usage
   - Chat message volume

2. **Infrastructure Metrics**
   - CPU and memory usage
   - Network I/O
   - Database performance
   - Storage utilization

3. **Business Metrics**
   - Active users
   - Chat session duration
   - File upload success rate
   - User engagement patterns

4. **Alerting Rules**
   - Error rate > 5%
   - Response time > 2s
   - Service downtime
   - Resource exhaustion

### Logging Strategy

1. **Structured Logging**
   - JSON format for machine readability
   - Consistent log levels (DEBUG, INFO, WARN, ERROR)
   - Contextual information in each log

2. **Log Aggregation**
   - Centralized logging in Google Cloud Logging
   - Log retention policies
   - Search and filtering capabilities

3. **Security Logging**
   - Authentication events
   - Authorization failures
   - Data access patterns
   - Compliance reporting

---

## Conclusion

YourFinAdvisor is built as a modern, scalable, and secure financial advisory platform that leverages cloud-native technologies and best practices. The architecture supports high availability, real-time communication, and seamless user experiences across web and mobile platforms.

### Key Strengths

1. **Scalability**: Microservices architecture with auto-scaling capabilities
2. **Security**: Multi-layered security with JWT authentication and encryption
3. **Performance**: Optimized for speed with streaming responses and caching
4. **Maintainability**: Monorepo structure with shared code and consistent tooling
5. **Observability**: Comprehensive monitoring and logging for operational excellence

### Future Enhancements

1. **Advanced AI Features**: Multi-modal AI capabilities, voice interactions
2. **Analytics Dashboard**: Real-time business intelligence and user analytics
3. **Multi-tenant Support**: B2B capabilities for financial institutions
4. **Mobile Push Notifications**: Enhanced mobile engagement features
5. **Advanced Security**: Biometric authentication, advanced fraud detection

This architecture provides a solid foundation for scaling the platform and adding new features while maintaining high performance, security, and reliability standards.
