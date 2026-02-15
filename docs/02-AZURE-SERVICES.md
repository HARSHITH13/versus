# Azure Services Architecture

## 🎯 Overview
This document details all Azure services used in the Versus Tournament Management System, their configuration, and integration patterns.

## ☁️ Azure Services Breakdown

### 1. Azure App Service
**Purpose**: Host the Web API and Blazor applications

#### Configuration
```yaml
Service Plan:
  - Development: B1 (Basic)
  - Staging: S1 (Standard)
  - Production: P1V3 (Premium V3)

Features:
  - Auto-scaling rules
  - Deployment slots (staging, production)
  - Always On enabled
  - Application Insights integration
  - Managed identities
  - Custom domain with SSL

Auto-Scale Rules:
  - Scale out when CPU > 70%
  - Scale in when CPU < 30%
  - Min instances: 2
  - Max instances: 10
```

#### App Service Plan Structure
```
┌─────────────────────────────────────────────────────┐
│          App Service Plan (Linux)                    │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │  versus-api-app                             │    │
│  │  - .NET 8 Web API                           │    │
│  │  - RESTful endpoints                        │    │
│  │  - Swagger/OpenAPI                          │    │
│  └────────────────────────────────────────────┘    │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │  versus-web-app                             │    │
│  │  - Blazor WebAssembly                       │    │
│  │  - Static files via CDN                     │    │
│  └────────────────────────────────────────────┘    │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │  versus-admin-app                           │    │
│  │  - Admin Portal                             │    │
│  │  - Management Interface                     │    │
│  └────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

---

### 2. Azure SQL Database
**Purpose**: Primary relational data store

#### Configuration
```yaml
Tier: General Purpose
Compute: Serverless (for cost optimization)
  - Min vCores: 2
  - Max vCores: 8
  - Auto-pause delay: 60 minutes

Storage: 
  - Initial: 32 GB
  - Max: 1 TB
  - Auto-grow enabled

Backup:
  - Point-in-time restore: 7 days
  - Long-term retention: 1 year (monthly)
  - Geo-redundant backup: Enabled

Security:
  - Azure AD Authentication
  - Transparent Data Encryption (TDE)
  - Advanced Threat Protection
  - Firewall rules
  - Private endpoint (production)
  - Managed Identity access

Performance:
  - Query Performance Insights
  - Automatic tuning enabled
  - Indexes optimization
```

#### Database Architecture
```
┌─────────────────────────────────────────────────────┐
│         versus-sql-server                            │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │  versus-main-db (Primary)                   │    │
│  │  - All application tables                   │    │
│  │  - Multi-tenant data                        │    │
│  │  - Row-level security                       │    │
│  └────────────────────────────────────────────┘    │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │  versus-analytics-db (Optional)             │    │
│  │  - Read-only replica                        │    │
│  │  - Reporting queries                        │    │
│  │  - Historical data                          │    │
│  └────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

---

### 3. Azure Redis Cache
**Purpose**: Distributed caching for performance

#### Configuration
```yaml
Tier: Standard C1 (Production)
  - Memory: 1 GB
  - SSL only: Enabled
  - Non-SSL port: Disabled

Cache Policies:
  - Eviction: allkeys-lru
  - Max memory policy: volatile-lru
  - Persistence: RDB enabled

Access:
  - Private endpoint
  - Managed identity
  - Connection string in Key Vault

Use Cases:
  - Session management
  - Tournament data cache
  - Points table cache
  - User profile cache
  - API response cache
```

#### Caching Strategy
```
┌─────────────────────────────────────────────────────┐
│              Redis Cache Strategy                    │
│                                                      │
│  Key Pattern          TTL        Use Case           │
│  ─────────────────    ────────   ───────────────    │
│  tournament:{id}      30 min     Tournament details │
│  points:{id}          5 min      Points table       │
│  fixture:{id}         15 min     Fixture details    │
│  user:{id}            60 min     User profile       │
│  team:{id}            30 min     Team info          │
│  session:{token}      24 hours   User sessions      │
│  api:rate:{ip}        1 min      Rate limiting      │
└─────────────────────────────────────────────────────┘
```

---

### 4. Azure Blob Storage
**Purpose**: Store media files, documents, and logs

#### Configuration
```yaml
Account Type: StorageV2 (General Purpose v2)
Performance: Standard
Replication: GRS (Geo-Redundant Storage)
Access Tier: Hot

Containers:
  - tournament-images (Public read)
  - player-avatars (Public read)
  - team-logos (Public read)
  - documents (Private)
  - exports (Private, time-limited SAS)
  - logs (Private, archive after 90 days)

Security:
  - Shared Access Signatures (SAS)
  - Managed Identity access
  - Encryption at rest
  - Soft delete enabled (30 days)
  - Versioning enabled

CDN Integration:
  - Azure Front Door
  - Cache static assets
  - Image optimization
```

#### Storage Structure
```
versus-storage-account
│
├── tournament-images/
│   ├── {tenantId}/{tournamentId}/banner.jpg
│   └── {tenantId}/{tournamentId}/gallery/*
│
├── player-avatars/
│   └── {tenantId}/{playerId}/avatar.jpg
│
├── team-logos/
│   └── {tenantId}/{teamId}/logo.png
│
├── documents/
│   └── {tenantId}/rules/{tournamentId}.pdf
│
├── exports/
│   └── {tenantId}/reports/{reportId}.xlsx
│
└── logs/
    └── {year}/{month}/{day}/application-logs.json
```

---

### 5. Azure Service Bus
**Purpose**: Asynchronous messaging for background jobs

#### Configuration
```yaml
Tier: Standard (Premium for production if needed)
Messaging Units: 1 (Standard), 2 (Premium)

Queues:
  - fixture-generation-queue
    - Max delivery count: 10
    - Lock duration: 5 minutes
    - Dead-letter enabled
  
  - notification-queue
    - Max delivery count: 5
    - Lock duration: 1 minute
    - Duplicate detection: 10 minutes
  
  - email-queue
    - Max delivery count: 3
    - Batch processing enabled
    - Scheduled messages supported

Topics & Subscriptions:
  - tournament-events-topic
    - Subscription: analytics-processor
    - Subscription: notification-handler
    - Subscription: audit-logger

Features:
  - Managed Identity authentication
  - Duplicate detection
  - Dead-letter queues
  - Message sessions
  - Scheduled messages
```

#### Messaging Patterns
```
┌─────────────────────────────────────────────────────────┐
│              Service Bus Message Flow                    │
│                                                          │
│  Tournament Created → fixture-generation-queue           │
│                    ↓                                     │
│              Background Worker                           │
│                    ↓                                     │
│          Generate Fixtures (Long Running)                │
│                    ↓                                     │
│              tournament-events-topic                     │
│                    ↓                                     │
│         ┌──────────┼──────────────┐                     │
│         ↓          ↓               ↓                     │
│    Analytics  Notifications    Audit                    │
│    Processor    Handler         Logger                   │
└─────────────────────────────────────────────────────────┘
```

---

### 6. Azure SignalR Service
**Purpose**: Real-time communication for live updates

#### Configuration
```yaml
Tier: Standard (1 unit = 1000 concurrent connections)
Units: 2 (for high availability)
Service Mode: Default

Features:
  - Automatic scaling
  - Message buffer
  - Connection state management
  - Managed Identity

Real-time Features:
  - Live score updates
  - Points table refresh
  - Match status changes
  - Player registrations
  - Admin notifications
  - Chat (future feature)
```

#### SignalR Hubs
```csharp
// Hub structure
Hubs:
  - TournamentHub
    - Groups: tournament-{id}
    - Methods:
      - JoinTournament(tournamentId)
      - ReceiveScoreUpdate(matchId, score)
      - ReceivePointsUpdate(tournamentId, pointsTable)
      - ReceiveMatchStatus(matchId, status)

  - AdminHub
    - Groups: admin-{tenantId}
    - Methods:
      - ReceivePlayerRegistration(playerId, tournamentId)
      - ReceiveSystemNotification(message)
```

---

### 7. Azure Event Grid
**Purpose**: Event-driven architecture for system events

#### Configuration
```yaml
Topics:
  - versus-events-topic

Event Types:
  - Tournament.Created
  - Tournament.Updated
  - Tournament.Published
  - Match.Scheduled
  - Score.Updated
  - Player.Registered
  - Team.Created

Subscriptions:
  - Event Grid → Azure Functions (serverless processing)
  - Event Grid → Logic Apps (workflows)
  - Event Grid → Webhook (external integrations)

Features:
  - Event filtering
  - Retry policy (max 30 attempts)
  - Dead-letter destination (blob storage)
  - Managed Identity
```

---

### 8. Azure Key Vault
**Purpose**: Secure secrets and certificate management

#### Configuration
```yaml
Tier: Standard
Features:
  - Soft delete enabled
  - Purge protection enabled
  - RBAC-based access

Secrets Stored:
  - Database connection strings
  - Redis connection strings
  - Storage account keys
  - API keys (3rd party)
  - JWT signing keys
  - SendGrid API key
  - Azure AD B2C secrets

Certificates:
  - SSL/TLS certificates
  - Auto-renewal enabled

Access Policies:
  - App Service: Get, List secrets
  - Function Apps: Get secrets
  - Developers: Get, List (dev vault only)
```

---

### 9. Azure Application Insights
**Purpose**: Application monitoring and diagnostics

#### Configuration
```yaml
Features:
  - Application Performance Management (APM)
  - Live Metrics Stream
  - Application Map
  - Smart Detection
  - Custom metrics and events
  - Log Analytics workspace integration

Monitoring:
  - Request tracking
  - Dependency tracking
  - Exception tracking
  - Custom events
  - Performance counters
  - Availability tests

Alerts:
  - High error rate
  - Slow response times
  - Failed dependencies
  - High CPU/Memory usage
```

#### Telemetry
```csharp
Custom Events:
  - Tournament created
  - Fixture generated
  - Player registered
  - Score updated

Custom Metrics:
  - Active tournaments count
  - Active players count
  - Match completion rate
  - API response times by endpoint

Dependencies:
  - SQL queries
  - Redis cache hits/misses
  - External API calls
```

---

### 10. Azure Front Door / CDN
**Purpose**: Global content delivery and load balancing

#### Configuration
```yaml
Features:
  - SSL/TLS termination
  - WAF (Web Application Firewall)
  - URL-based routing
  - Session affinity
  - Health probes
  - Caching rules
  - Custom domains

WAF Rules:
  - OWASP Top 10 protection
  - Bot protection
  - Rate limiting
  - Geo-filtering (optional)
  - Custom rules

Caching:
  - Static assets: 30 days
  - Images: 7 days
  - API responses: No cache (dynamic)
  - Compression: Enabled (gzip, brotli)

Routing:
  - /api/* → App Service API
  - /static/* → Blob Storage
  - /* → Blazor App
```

---

### 11. Azure Container Apps (Recommended) / AKS
**Purpose**: Container orchestration

#### Azure Container Apps Configuration (Simpler)
```yaml
Environment: versus-container-env

Apps:
  - versus-api-app
    - Min replicas: 2
    - Max replicas: 10
    - CPU: 0.5 cores
    - Memory: 1 GB
    - Ingress: External (HTTPS)
    - Scale rules:
      - HTTP: 100 concurrent requests
      - CPU: 70%

  - versus-worker-app
    - Min replicas: 1
    - Max replicas: 5
    - CPU: 1 core
    - Memory: 2 GB
    - Ingress: Internal only
    - Scale rules:
      - Service Bus queue length > 10

Features:
  - KEDA-based autoscaling
  - Dapr integration (optional)
  - Managed identity
  - Blue-green deployments
  - Traffic splitting
```

#### AKS Configuration (Advanced)
```yaml
Cluster:
  - Node count: 3-10 (autoscale)
  - Node size: Standard_D4s_v3
  - Kubernetes version: 1.28+
  - Network plugin: Azure CNI

Features:
  - Azure AD integration
  - Managed identity
  - Azure Policy
  - Azure Monitor for containers
  - Ingress controller (NGINX)
  - Cert-manager for SSL
  - Helm for deployments

Namespaces:
  - versus-prod
  - versus-staging
  - versus-dev
```

---

### 12. Azure DevOps
**Purpose**: CI/CD pipelines and source control

#### Configuration
```yaml
Repositories:
  - versus-api (Backend)
  - versus-web (Frontend)
  - versus-infrastructure (IaC)

Pipelines:
  - Build Pipeline:
    - Code checkout
    - Restore dependencies
    - Build solution
    - Run unit tests
    - Run integration tests
    - Code coverage analysis
    - Security scanning (WhiteSource Bolt)
    - Docker image build
    - Push to Azure Container Registry
    - Publish artifacts

  - Release Pipeline:
    - Dev → Automatic deployment
    - Staging → Automatic deployment + smoke tests
    - Production → Manual approval + deployment

Artifacts:
  - NuGet packages (shared libraries)
  - Docker images
  - Terraform/Bicep templates

Test Plans:
  - Unit tests
  - Integration tests
  - E2E tests (Playwright/Selenium)
```

---

### 13. Azure AD B2C
**Purpose**: Customer identity and access management

#### Configuration
```yaml
Tenant: versus-b2c.onmicrosoft.com

User Flows:
  - Sign up and sign in
  - Password reset
  - Profile editing
  - Multi-factor authentication (optional)

Custom Policies:
  - Social login (Google, Microsoft, Facebook)
  - Custom branding
  - Custom attributes (PlayerRole, TenantId)

Token Configuration:
  - Access token lifetime: 1 hour
  - Refresh token lifetime: 14 days
  - ID token lifetime: 1 hour

Claims:
  - sub (User ID)
  - email
  - name
  - role (Admin, Organizer, Player)
  - tenant_id
```

---

## 💰 Cost Estimation (Monthly)

### Development Environment
| Service | Tier | Est. Cost |
|---------|------|-----------|
| App Service | B1 | $13 |
| Azure SQL | Serverless (2-4 vCores) | $50-100 |
| Redis Cache | Basic C0 | $17 |
| Storage | Standard | $5 |
| Service Bus | Standard | $10 |
| SignalR | Free tier | $0 |
| Application Insights | 5GB included | $0-10 |
| **Total** | | **~$100-150** |

### Production Environment
| Service | Tier | Est. Cost |
|---------|------|-----------|
| App Service / Container Apps | P1V3 or 2 containers | $150-200 |
| Azure SQL | 4-8 vCores | $200-400 |
| Redis Cache | Standard C1 | $75 |
| Storage | Standard GRS | $30 |
| Service Bus | Standard | $10 |
| SignalR | Standard (2 units) | $100 |
| Front Door / CDN | Standard | $35-50 |
| Application Insights | 10-20 GB | $20-40 |
| Azure AD B2C | 50K MAU free | $0-5 |
| Key Vault | Standard | $3 |
| **Total** | | **~$620-900** |

---

## 🔄 Service Integration Patterns

### 1. API → Database Pattern
```
API → Azure AD Auth → Validate Tenant → Query SQL → Cache in Redis → Return
```

### 2. Background Job Pattern
```
API → Enqueue Message (Service Bus) → Worker picks message → 
Process → Update DB → Publish Event (Event Grid) → Notify (SignalR)
```

### 3. File Upload Pattern
```
API → Generate SAS Token → Client uploads to Blob → 
Blob trigger → Process/Validate → Update DB → CDN cache
```

### 4. Real-time Update Pattern
```
Score Update → API → Update DB → Publish to SignalR Hub → 
Clients receive via WebSocket → UI updates
```

---

## 🛡️ Security Best Practices

1. **Managed Identities**: All service-to-service communication
2. **Private Endpoints**: SQL, Redis, Storage (production)
3. **Key Vault**: No secrets in code or config files
4. **Network Security Groups**: Restrict traffic
5. **DDoS Protection**: Azure DDoS Standard
6. **WAF**: Azure Front Door WAF
7. **Encryption**: At rest and in transit
8. **Monitoring**: Azure Security Center

---

## 📚 Additional Resources

- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/architecture/framework/)
- [Azure Architecture Center](https://learn.microsoft.com/azure/architecture/)
- [Azure Security Best Practices](https://learn.microsoft.com/azure/security/fundamentals/best-practices-and-patterns)

---

## 🎯 Next Steps
1. Set up Azure resources using IaC (Bicep/Terraform)
2. Configure CI/CD pipelines in Azure DevOps
3. Implement monitoring and alerting
4. Set up disaster recovery procedures
