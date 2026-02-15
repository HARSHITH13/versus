# Versus - Quick Reference Guide

## 🚀 5-Minute Quick Start

```bash
# 1. Clone and navigate
git clone https://github.com/yourusername/versus.git
cd versus

# 2. Start local services
docker-compose up -d

# 3. Apply database migrations
cd src/Versus.Infrastructure
dotnet ef database update --startup-project ../Versus.Api

# 4. Run the API
cd ../Versus.Api
dotnet run

# 5. Run the Web App (in another terminal)
cd ../Versus.Web
dotnet run
```

🎉 **Done!** Access at:
- API: https://localhost:7001/swagger
- Web: https://localhost:7002

---

## 📂 Project Structure at a Glance

```
versus/
├── src/
│   ├── Versus.Domain/          ← Entities, Value Objects (No dependencies)
│   ├── Versus.Application/     ← CQRS Commands/Queries (Depends on Domain)
│   ├── Versus.Infrastructure/  ← EF Core, Azure Services (Depends on Application)
│   ├── Versus.Api/            ← REST API Controllers (Depends on Infrastructure)
│   └── Versus.Web/            ← Blazor UI (Depends on Shared)
├── tests/                     ← Unit & Integration Tests
└── docs/                      ← Architecture Documentation
```

---

## 🔑 Key Technologies

| What | Technology |
|------|------------|
| **Backend** | .NET 8, C# 12 |
| **Frontend** | Blazor WebAssembly |
| **Database** | Azure SQL, EF Core |
| **Cache** | Redis |
| **Storage** | Azure Blob Storage |
| **Auth** | Azure AD B2C, JWT |
| **Real-time** | SignalR |
| **CI/CD** | Azure DevOps |
| **Container** | Docker, Azure Container Apps |

---

## 🏗️ Architecture Patterns

### Clean Architecture Layers
```
Presentation → Application → Domain ← Infrastructure
     ↓              ↓           ↓
  API/Web      Use Cases   Entities
```

### CQRS (Command Query Responsibility Segregation)
- **Commands**: Change state (`CreateTournamentCommand`)
- **Queries**: Read data (`GetTournamentsQuery`)
- **MediatR**: Handles both

### Domain-Driven Design
- **Entities**: `Tournament`, `Match`, `Team`, `Player`
- **Value Objects**: `Score`, `DateRange`
- **Aggregates**: `TournamentAggregate`
- **Domain Events**: `TournamentCreatedEvent`

---

## 💾 Database Schema (Key Tables)

| Table | Purpose |
|-------|---------|
| `Tenants` | Multi-tenant organizations |
| `Users` | User accounts (Azure AD B2C) |
| `Tournaments` | Tournament details |
| `Teams` | Team registration |
| `Players` | Player profiles |
| `Matches` | Match fixtures |
| `PointsTable` | Tournament standings |
| `MatchEvents` | Goals, cards, etc. |

**Multi-Tenancy**: Every table has `TenantId` with Row-Level Security (RLS)

---

## 🔌 API Endpoints (Sample)

```
POST   /api/tournaments              Create tournament
GET    /api/tournaments              List tournaments
GET    /api/tournaments/{id}         Get tournament details
POST   /api/tournaments/{id}/publish Publish tournament

POST   /api/teams                    Create team
GET    /api/teams/{id}               Get team details

POST   /api/matches/{id}/events      Add match event (goal, card)
PUT    /api/matches/{id}/complete    Complete match

GET    /api/tournaments/{id}/points-table   Get standings
GET    /api/tournaments/{id}/top-scorers    Get top scorers
```

📖 **Full API Docs**: [docs/05-API-SPECIFICATION.md](./docs/05-API-SPECIFICATION.md)

---

## 🎭 User Roles & Permissions

| Role | Can Do |
|------|--------|
| **Admin** | Everything |
| **Organizer** | Create/manage tournaments, approve teams |
| **Player** | Register, view schedules, limited updates |
| **Viewer** | Read-only access |

---

## ☁️ Azure Services Used

| Service | Purpose |
|---------|---------|
| **App Service / Container Apps** | Host API & Web |
| **Azure SQL** | Primary database |
| **Redis Cache** | Distributed caching |
| **Blob Storage** | Images, documents |
| **Service Bus** | Async messaging |
| **SignalR** | Real-time updates |
| **Event Grid** | Event-driven architecture |
| **Application Insights** | Monitoring & logging |
| **Azure AD B2C** | Authentication |
| **Azure Front Door** | CDN & WAF |
| **Key Vault** | Secrets management |

---

## 🧪 Testing Commands

```bash
# Run all tests
dotnet test

# Tests with coverage
dotnet test --collect:"XPlat Code Coverage"

# Specific project
dotnet test tests/Versus.Domain.UnitTests/

# Watch mode (continuous testing)
dotnet watch test --project tests/Versus.Domain.UnitTests/
```

---

## 🐳 Docker Commands

```bash
# Start local infrastructure
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f sqlserver
docker-compose logs -f redis

# Rebuild and start
docker-compose up -d --build

# Build API image
docker build -t versus-api:latest -f src/Versus.Api/Dockerfile .

# Run API container
docker run -p 8080:8080 versus-api:latest
```

---

## 🔧 Common Configuration

### appsettings.Development.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=VersusDb;User Id=sa;Password=YourStrong@Passw0rd;TrustServerCertificate=True;",
    "Redis": "localhost:6379"
  },
  "Jwt": {
    "SecretKey": "your-32-char-secret-key-here",
    "Issuer": "VersusApi",
    "Audience": "VersusWeb"
  }
}
```

### User Secrets (Development)
```bash
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your-connection-string"
dotnet user-secrets set "Jwt:SecretKey" "your-secret-key"
```

---

## 🚢 Deployment

### Azure CLI
```bash
# Login
az login

# Create Resource Group
az group create --name versus-prod-rg --location eastus

# Deploy Container App
az containerapp update \
  --name versus-api-prod \
  --resource-group versus-prod-rg \
  --image versusacr.azurecr.io/versus-api:latest
```

### Azure DevOps
1. Push to `develop` → Auto-deploy to Dev
2. Push to `main` → Auto-deploy to Staging → Manual approval → Production

📖 **Full CI/CD Guide**: [docs/06-CICD-PIPELINE.md](./docs/06-CICD-PIPELINE.md)

---

## 📊 Monitoring URLs (Local)

| Service | URL |
|---------|-----|
| API | https://localhost:7001 |
| Swagger | https://localhost:7001/swagger |
| Web App | https://localhost:7002 |
| Seq Logs | http://localhost:5341 |
| MailHog | http://localhost:8025 |
| Azurite | http://localhost:10000 |

---

## 🐛 Troubleshooting

### SQL Server won't start
```bash
docker logs versus-sqlserver
# Check password meets complexity requirements
```

### Migration fails
```bash
# Clean and recreate
dotnet ef database drop --force --startup-project ../Versus.Api
dotnet ef database update --startup-project ../Versus.Api
```

### Port already in use
```bash
# Find process using port 7001
netstat -ano | findstr :7001
# Kill process
taskkill /PID <process-id> /F
```

### Redis connection fails
```bash
# Test Redis connectivity
docker exec -it versus-redis redis-cli ping
# Should return: PONG
```

---

## 📚 Documentation Index

| Document | What's Inside |
|----------|---------------|
| [System Architecture](./docs/01-SYSTEM-ARCHITECTURE.md) | High-level design, multi-tenancy, scalability |
| [Azure Services](./docs/02-AZURE-SERVICES.md) | Azure setup, configuration, costs |
| [Clean Architecture](./docs/03-CLEAN-ARCHITECTURE.md) | Layers, CQRS, DDD patterns |
| [Database Design](./docs/04-DATABASE-DESIGN.md) | Schema, tables, relationships |
| [API Specification](./docs/05-API-SPECIFICATION.md) | Endpoints, authentication, examples |
| [CI/CD Pipeline](./docs/06-CICD-PIPELINE.md) | Build & release pipelines |
| [Project Setup](./docs/07-PROJECT-SETUP.md) | Step-by-step setup guide |

---

## 🎓 Learning Path

1. **Start Here**: [README.md](./README.md)
2. **Understand Architecture**: [01-SYSTEM-ARCHITECTURE.md](./docs/01-SYSTEM-ARCHITECTURE.md)
3. **Setup Environment**: [07-PROJECT-SETUP.md](./docs/07-PROJECT-SETUP.md)
4. **Explore Code**: Review `src/Versus.Domain` first (no dependencies)
5. **Run Tests**: `dotnet test`
6. **Make Changes**: Follow [CONTRIBUTING.md](./CONTRIBUTING.md)
7. **Deploy**: [06-CICD-PIPELINE.md](./docs/06-CICD-PIPELINE.md)

---

## 💡 Pro Tips

- Use `dotnet watch run` for hot reload during development
- Keep Application Insights open during testing for real-time diagnostics
- Use Azure Storage Explorer for blob inspection
- Redis Insight for cache debugging
- Azure Data Studio for SQL queries
- Install VS Code extensions: C# Dev Kit, Azure Tools

---

## 📞 Quick Links

- **Swagger UI**: https://localhost:7001/swagger
- **Health Check**: https://localhost:7001/health
- **Azure Portal**: https://portal.azure.com
- **Azure DevOps**: https://dev.azure.com

---

## 🎯 Next Steps

1. ✅ Clone repository
2. ✅ Start Docker containers
3. ✅ Run migrations
4. ✅ Start API & Web
5. 👉 Read [System Architecture](./docs/01-SYSTEM-ARCHITECTURE.md)
6. 👉 Implement your first feature
7. 👉 Deploy to Azure

---

**Happy Coding! 🚀**

Need help? Check the [full documentation](./docs/) or open an issue!
