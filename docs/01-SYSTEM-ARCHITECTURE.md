# Sports Tournament Management System - System Architecture

## 🎯 Overview
The **Versus** Sports Tournament Management System is a cloud-native, multi-tenant SaaS application built on .NET 8 and Azure services. It enables organizations to create and manage sports tournaments with automated fixture generation, real-time points tracking, and role-based access control.

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │   Web App    │  │  Mobile App  │  │  Admin Panel │             │
│  │   (Angular)  │  │   (Future)   │  │  (Angular)   │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└────────────────────────────┬─────────────────────────────────────────┘
                             │ HTTPS
┌────────────────────────────┴─────────────────────────────────────────┐
│                      AZURE FRONT DOOR / CDN                           │
│  - SSL Termination                                                    │
│  - WAF (Web Application Firewall)                                     │
│  - Static Content Caching                                             │
│  - Global Load Balancing                                              │
└────────────────────────────┬─────────────────────────────────────────┘
                             │
┌────────────────────────────┴─────────────────────────────────────────┐
│                      API GATEWAY LAYER                                │
│                   (Azure API Management)                              │
│  - Rate Limiting                                                      │
│  - Request Throttling                                                 │
│  - API Versioning                                                     │
│  - Authentication/Authorization                                       │
└────────────────────────────┬─────────────────────────────────────────┘
                             │
┌────────────────────────────┴─────────────────────────────────────────┐
│                    APPLICATION LAYER                                  │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │           Azure App Service / Container Apps                 │   │
│  │                                                               │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │   │
│  │  │ Tournament   │  │   Player     │  │  Fixture     │     │   │
│  │  │   Service    │  │   Service    │  │  Service     │     │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘     │   │
│  │                                                               │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │   │
│  │  │  Identity    │  │ Notification │  │  Reporting   │     │   │
│  │  │   Service    │  │   Service    │  │   Service    │     │   │
│  │  └──────────────┘  └──────────────┘  └──────────────┘     │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────┬─────────────────────────────────────────┘
                             │
┌────────────────────────────┴─────────────────────────────────────────┐
│                      DATA LAYER                                       │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │  Azure SQL   │  │   Redis      │  │    Cosmos    │             │
│  │  Database    │  │   Cache      │  │      DB      │             │
│  │  (Primary)   │  │ (Distributed)│  │  (Events)    │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐                                │
│  │  Blob        │  │  Table       │                                │
│  │  Storage     │  │  Storage     │                                │
│  │  (Media)     │  │  (Logs)      │                                │
│  └──────────────┘  └──────────────┘                                │
└────────────────────────────┬─────────────────────────────────────────┘
                             │
┌────────────────────────────┴─────────────────────────────────────────┐
│                   MESSAGING & EVENTS                                  │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │  Service     │  │  Event Grid  │  │  SignalR     │             │
│  │    Bus       │  │              │  │  Service     │             │
│  │ (Async Ops)  │  │ (Events)     │  │ (Real-time)  │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└────────────────────────────┬─────────────────────────────────────────┘
                             │
┌────────────────────────────┴─────────────────────────────────────────┐
│              MONITORING & OBSERVABILITY                               │
│                                                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │
│  │ Application  │  │   Log        │  │   Azure      │             │
│  │   Insights   │  │  Analytics   │  │   Monitor    │             │
│  └──────────────┘  └──────────────┘  └──────────────┘             │
└───────────────────────────────────────────────────────────────────────┘
```

## 🔐 Security Architecture

### Authentication & Authorization
- **Azure AD B2C** - User authentication and identity management
- **JWT Tokens** - Secure API access
- **Role-Based Access Control (RBAC)**
  - **Admin**: Full system access
  - **Organizer**: Create/manage tournaments, approve players
  - **Player**: Register, view tournaments, submit scores
  - **Viewer**: Read-only access

### Security Measures
- **Azure Key Vault** - Secrets and certificate management
- **Managed Identities** - Service-to-service authentication
- **WAF** - Protection against OWASP Top 10
- **DDoS Protection** - Azure DDoS Standard
- **SSL/TLS** - End-to-end encryption

## 🎨 Multi-Tenant Architecture

### Tenant Isolation Strategy
```
┌─────────────────────────────────────────────────────────────┐
│                    Tenant Isolation                          │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │         Database Per Tenant (Option 1)              │    │
│  │  - Complete isolation                               │    │
│  │  - Higher cost                                      │    │
│  │  - Easier scaling                                   │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │      Schema Per Tenant (Option 2)                   │    │
│  │  - Balanced approach                                │    │
│  │  - Moderate cost                                    │    │
│  │  - Good isolation                                   │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │    Shared Schema with TenantId (Option 3) ✓        │    │
│  │  - Cost effective                                   │    │
│  │  - Row-level security                               │    │
│  │  - Best for MVP                                     │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

**Recommended: Shared Schema with TenantId**
- Every table has a `TenantId` column
- Row-Level Security (RLS) enforced at database level
- Application layer validates tenant context
- Easy migration to schema-per-tenant if needed

## 📊 Data Flow Architecture

### Tournament Creation Flow
```
User → API Gateway → Tournament Service → Azure SQL → Event Grid → 
Notification Service → SignalR → Real-time Update to Clients
```

### Fixture Generation Flow
```
Organizer → Tournament Service → Fixture Generator (Background Job) →
Azure Service Bus → Fixture Processor → Azure SQL → Event Notification
```

### Live Score Update Flow
```
Score Update → API → Validation → Azure SQL → SignalR Hub → 
WebSocket → Real-time to Connected Clients
```

## 🚀 Deployment Architecture

### Environments
1. **Development** - Azure App Service (Basic Tier)
2. **Staging** - Azure App Service (Standard Tier)
3. **Production** - Azure Container Apps / AKS

### Container Strategy
```
┌─────────────────────────────────────────────────────────┐
│              Azure Container Registry (ACR)              │
│                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐       │
│  │    API     │  │   Web UI   │  │  Worker    │       │
│  │   Image    │  │   Image    │  │   Image    │       │
│  └────────────┘  └────────────┘  └────────────┘       │
└─────────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────────┐
│         Azure Container Apps (Recommended)              │
│                        OR                               │
│         Azure Kubernetes Service (AKS)                  │
│                                                          │
│  - Auto-scaling (KEDA)                                  │
│  - Blue-Green Deployment                                │
│  - Rolling Updates                                      │
│  - Health Checks                                        │
└─────────────────────────────────────────────────────────┘
```

## 📈 Scalability & Performance

### Scaling Strategy
- **Horizontal Scaling**: Auto-scale based on CPU/Memory/Request count
- **Database Scaling**: Azure SQL Elastic Pool
- **Caching**: Redis Cache for frequently accessed data
- **CDN**: Static assets and media files
- **Load Balancing**: Azure Front Door

### Performance Optimizations
- **Caching Strategy**
  - Tournament details: 30 minutes
  - Points table: 5 minutes
  - User profiles: 1 hour
- **Database Indexing**: Strategic indexes on frequently queried columns
- **Connection Pooling**: Optimized database connections
- **Async Processing**: Background jobs for heavy operations

## 🔄 Disaster Recovery & High Availability

### Backup Strategy
- **Azure SQL**: Automated backups with 7-day retention
- **Geo-Replication**: Secondary region for failover
- **Blob Storage**: GRS (Geo-Redundant Storage)

### High Availability
- **SLA Target**: 99.9% uptime
- **Multi-Region Deployment**: Primary (East US), Secondary (West US)
- **Automated Failover**: Azure Traffic Manager
- **Health Probes**: Continuous monitoring

## 📝 Technology Stack Summary

| Layer | Technology |
|-------|------------|
| Frontend | Angular 18, TypeScript, RxJS |
| Backend | .NET 8, ASP.NET Core Web API |
| Database | Azure SQL Database |
| Cache | Azure Redis Cache |
| Storage | Azure Blob Storage |
| Messaging | Azure Service Bus, Event Grid |
| Real-time | Azure SignalR Service |
| Identity | Azure AD B2C |
| Monitoring | Application Insights, Log Analytics |
| CI/CD | Azure DevOps |
| Container | Docker, Azure Container Apps/AKS |
| API Gateway | Azure API Management |
| CDN | Azure Front Door |

## 🎯 Architecture Principles

1. **Clean Architecture**: Separation of concerns with distinct layers
2. **SOLID Principles**: Maintainable and testable code
3. **Domain-Driven Design**: Rich domain models
4. **CQRS Pattern**: Separate read and write operations
5. **Event-Driven**: Asynchronous communication where applicable
6. **Microservices Ready**: Modularity for future service extraction
7. **Cloud-Native**: Leveraging Azure PaaS services
8. **Security First**: Zero-trust security model
9. **Observability**: Comprehensive logging and monitoring
10. **Cost Optimization**: Right-sizing resources

## 📚 Next Steps
1. Review [Azure Services Architecture](./02-AZURE-SERVICES.md)
2. Review [Clean Architecture Implementation](./03-CLEAN-ARCHITECTURE.md)
3. Review [Database Design](./04-DATABASE-DESIGN.md)
4. Review [API Specifications](./05-API-SPECIFICATION.md)
