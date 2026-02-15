# Versus - Sports Tournament Management System

<div align="center">

![.NET](https://img.shields.io/badge/.NET-10.0-512BD4?logo=dotnet)
![Azure](https://img.shields.io/badge/Azure-Cloud%20Native-0089D6?logo=microsoft-azure)
![Docker](https://img.shields.io/badge/Docker-Containerized-2496ED?logo=docker)
![License](https://img.shields.io/badge/License-MIT-green.svg)

**A Cloud-Native Sports Tournament Management System built with .NET 8 and Azure**

[Features](#-features) • [Architecture](#-architecture) • [Documentation](#-documentation) • [Getting Started](#-getting-started) • [Contributing](#-contributing)

</div>

---

## 🎯 Overview

**Versus** is a comprehensive, cloud-native Sports Tournament Management System designed to help organizers create, manage, and run tournaments efficiently. Built with modern technologies and following best practices, it demonstrates enterprise-level architecture patterns and Azure cloud services integration.

### Key Highlights
- 🏆 **Tournament Management**: Create and manage tournaments with multiple formats (Round Robin, Knockout, League, Group Stage)
- 📊 **Automatic Fixture Generation**: Smart algorithms to generate fair match schedules
- 📈 **Live Points Table**: Real-time standings and statistics
- 👥 **Multi-tenant SaaS**: Isolated data for each organization
- 🔐 **Role-based Access Control**: Admin, Organizer, Player, and Viewer roles
- 🚀 **Cloud-Native**: Built for Azure with auto-scaling and high availability
- 🔄 **Real-time Updates**: Live score updates using SignalR
- 📱 **Responsive UI**: Modern Blazor WebAssembly frontend

## ✨ Features

### For Organizers
- ✅ Create and configure tournaments
- ✅ Manage team registrations and approvals
- ✅ Automatic fixture generation
- ✅ Live score management
- ✅ Points table tracking
- ✅ Player statistics
- ✅ Match scheduling and venue management
- ✅ Tournament analytics and reporting

### For Players/Teams
- ✅ Register for tournaments
- ✅ View match schedules
- ✅ Track statistics and rankings
- ✅ Receive real-time notifications
- ✅ View historical performance

### For Spectators
- ✅ Browse tournaments
- ✅ View live scores
- ✅ Follow favorite teams
- ✅ Access tournament statistics

## 🏗️ Architecture

Versus follows **Clean Architecture** principles with clear separation of concerns:

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
│              Web API + Blazor WebAssembly                    │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────┐
│                   Application Layer                          │
│        Use Cases (CQRS) + Business Logic                    │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────┐
│                     Domain Layer                             │
│          Entities + Value Objects + Domain Events           │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────┐
│                  Infrastructure Layer                        │
│        Data Access + External Services + Azure              │
└─────────────────────────────────────────────────────────────┘
```

### Technology Stack

| Category | Technologies |
|----------|-------------|
| **Backend** | .NET 8, ASP.NET Core Web API, C# 12 |
| **Frontend** | Blazor WebAssembly, MudBlazor |
| **Database** | Azure SQL Database, Entity Framework Core |
| **Caching** | Azure Redis Cache |
| **Storage** | Azure Blob Storage |
| **Messaging** | Azure Service Bus, Azure Event Grid |
| **Real-time** | Azure SignalR Service |
| **Identity** | Azure AD B2C |
| **Monitoring** | Application Insights, Azure Monitor |
| **CI/CD** | Azure DevOps |
| **Containers** | Docker, Azure Container Apps |
| **IaC** | Bicep / Terraform |
| **Testing** | NUnit, FluentAssertions, Moq, Testcontainers |

## 📚 Documentation

Comprehensive documentation is available in the `/docs` folder:

| Document | Description |
|----------|-------------|
| [📐 System Architecture](./docs/01-SYSTEM-ARCHITECTURE.md) | High-level architecture, multi-tenancy, security |
| [☁️ Azure Services](./docs/02-AZURE-SERVICES.md) | Azure services configuration and integration |
| [🏛️ Clean Architecture](./docs/03-CLEAN-ARCHITECTURE.md) | Application layers, DDD, CQRS patterns |
| [🗄️ Database Design](./docs/04-DATABASE-DESIGN.md) | Complete database schema and relationships |
| [🔌 API Specification](./docs/05-API-SPECIFICATION.md) | RESTful API endpoints documentation |
| [🚀 CI/CD Pipeline](./docs/06-CICD-PIPELINE.md) | Azure DevOps build and release pipelines |
| [⚙️ Project Setup](./docs/07-PROJECT-SETUP.md) | Step-by-step setup guide for development |

## 🚀 Getting Started

### Prerequisites
- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)
- [Visual Studio 2022](https://visualstudio.microsoft.com/) or [VS Code](https://code.visualstudio.com/)
- Azure Subscription (free tier available)

### Quick Start

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/versus.git
   cd versus
   ```

2. **Start local infrastructure**
   ```bash
   docker-compose up -d
   ```

3. **Run database migrations**
   ```bash
   cd src/Versus.Infrastructure
   dotnet ef database update --startup-project ../Versus.Api
   ```

4. **Run the API**
   ```bash
   cd src/Versus.Api
   dotnet run
   ```

5. **Run the Web App**
   ```bash
   cd src/Versus.Web
   dotnet run
   ```

6. **Access the applications**
   - API: https://localhost:7001
   - Swagger: https://localhost:7001/swagger
   - Web App: https://localhost:7002

For detailed setup instructions, see [Project Setup Guide](./docs/07-PROJECT-SETUP.md).

## 🛠️ Development

### Solution Structure
```
versus/
├── src/
│   ├── Versus.Domain/           # Domain entities, value objects
│   ├── Versus.Application/      # Use cases, DTOs, interfaces
│   ├── Versus.Infrastructure/   # Data access, external services
│   ├── Versus.Api/             # REST API, controllers
│   ├── Versus.Web/             # Blazor WebAssembly UI
│   └── Versus.Shared/          # Shared constants, utilities
├── tests/
│   ├── Versus.Domain.UnitTests/
│   ├── Versus.Application.UnitTests/
│   ├── Versus.Application.IntegrationTests/
│   └── Versus.Api.IntegrationTests/
├── docs/                        # Documentation
└── scripts/                     # Build and deployment scripts
```

### Running Tests
```bash
# Run all tests
dotnet test

# Run with coverage
dotnet test --collect:"XPlat Code Coverage"

# Run specific test project
dotnet test tests/Versus.Domain.UnitTests/
```

### Docker Build
```bash
# Build API image
docker build -t versus-api:latest -f src/Versus.Api/Dockerfile .

# Build Web image
docker build -t versus-web:latest -f src/Versus.Web/Dockerfile .

# Run with docker-compose
docker-compose -f docker-compose.yml -f docker-compose.override.yml up
```

## 🔧 Configuration

### Environment Variables
```bash
# Database
ConnectionStrings__DefaultConnection=Server=...

# Azure Services
Azure__StorageConnectionString=...
Azure__ServiceBusConnectionString=...
Azure__RedisConnectionString=...

# Authentication
AzureAd__TenantId=...
AzureAd__ClientId=...
Jwt__SecretKey=...

# Application Insights
ApplicationInsights__InstrumentationKey=...
```

### User Secrets (Development)
```bash
cd src/Versus.Api
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your-connection-string"
dotnet user-secrets set "Jwt:SecretKey" "your-secret-key"
```

## 🚢 Deployment

### Azure Deployment
```bash
# Login to Azure
az login

# Deploy infrastructure
az deployment group create \
  --resource-group versus-prod-rg \
  --template-file infrastructure/bicep/main.bicep \
  --parameters infrastructure/bicep/parameters.json

# Deploy application
az containerapp update \
  --name versus-api-prod \
  --resource-group versus-prod-rg \
  --image versusacr.azurecr.io/versus-api:latest
```

For complete CI/CD setup, see [CI/CD Pipeline Documentation](./docs/06-CICD-PIPELINE.md).

## 🧪 Testing Strategy

- **Unit Tests**: Domain and application logic
- **Integration Tests**: API endpoints and database interactions
- **E2E Tests**: Complete user workflows with Playwright
- **Performance Tests**: Load testing with Azure Load Testing
- **Security Tests**: Dependency scanning with WhiteSource Bolt

Target: **80%+ code coverage**

## 📊 Monitoring

- **Application Insights**: Performance, exceptions, dependencies
- **Azure Monitor**: Infrastructure metrics and alerts
- **Log Analytics**: Centralized logging
- **Azure Dashboard**: Custom dashboards for key metrics

## 🔒 Security

- ✅ Azure AD B2C authentication
- ✅ JWT token-based authorization
- ✅ Role-based access control (RBAC)
- ✅ Row-level security for multi-tenancy
- ✅ HTTPS only
- ✅ Secrets in Azure Key Vault
- ✅ Managed identities for Azure services
- ✅ WAF enabled on Azure Front Door
- ✅ DDoS protection

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guidelines](CONTRIBUTING.md) before submitting pull requests.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 👥 Authors

- **Your Name** - *Initial work* - [@yourusername](https://github.com/yourusername)

## 🙏 Acknowledgments

- Clean Architecture principles by Robert C. Martin
- Domain-Driven Design by Eric Evans
- Microsoft Azure documentation and best practices
- .NET Community

## 📞 Support

- 📧 Email: support@versus-sports.com
- 📖 Documentation: [/docs](/docs)
- 🐛 Issues: [GitHub Issues](https://github.com/yourusername/versus/issues)
- 💬 Discussions: [GitHub Discussions](https://github.com/yourusername/versus/discussions)

---

<div align="center">

**Built with ❤️ using .NET 8 and Azure**

[⬆ Back to Top](#versus---sports-tournament-management-system)

</div>
Versus is a full-stack Sports Tournament Management System that enables seamless tournament creation, team management, match scheduling and live score tracking
