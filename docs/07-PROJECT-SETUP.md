# Project Setup Guide

## 🎯 Overview
Complete step-by-step guide to set up the Versus Tournament Management System development environment and deploy to Azure.

## 📋 Prerequisites

### Required Software
- **Windows 11** (or Windows 10, macOS, Linux)
- **.NET 8 SDK** - [Download](https://dotnet.microsoft.com/download/dotnet/8.0)
- **Visual Studio 2022** (17.12+) or **VS Code**
- **Docker Desktop** - [Download](https://www.docker.com/products/docker-desktop)
- **Azure CLI** - [Download](https://docs.microsoft.com/cli/azure/install-azure-cli)
- **Git** - [Download](https://git-scm.com/downloads)
- **SQL Server Management Studio (SSMS)** (optional)
- **Postman** or **Insomnia** for API testing (optional)

### Azure Subscription
- Active Azure subscription
- Contributor or Owner role
- Azure DevOps organization (free tier available)

### Development Tools (Optional but Recommended)
- **Azure Storage Explorer**
- **Azure Data Studio**
- **Redis Insight**
- **Docker Desktop with Kubernetes**

## 🔧 Local Development Setup

### Step 1: Clone the Repository
```powershell
# Create project directory
mkdir "D:\Learning\Projects\Versus - Sports Tournament Management System"
cd "D:\Learning\Projects\Versus - Sports Tournament Management System"

# Initialize Git repository
git init
git remote add origin <your-repo-url>
git pull origin main
```

### Step 2: Install .NET 8 SDK
```powershell
# Verify installation
dotnet --version
# Should output: 8.0.x

# Install EF Core tools
dotnet tool install --global dotnet-ef
dotnet tool update --global dotnet-ef

# Verify EF Core tools
dotnet ef --version
```

### Step 3: Create Solution Structure
```powershell
# Create solution
dotnet new sln -n Versus

# Create Domain project
dotnet new classlib -n Versus.Domain -o src/Versus.Domain -f net8.0
dotnet sln add src/Versus.Domain/Versus.Domain.csproj

# Create Application project
dotnet new classlib -n Versus.Application -o src/Versus.Application -f net8.0
dotnet sln add src/Versus.Application/Versus.Application.csproj

# Create Infrastructure project
dotnet new classlib -n Versus.Infrastructure -o src/Versus.Infrastructure -f net8.0
dotnet sln add src/Versus.Infrastructure/Versus.Infrastructure.csproj

# Create API project
dotnet new webapi -n Versus.Api -o src/Versus.Api -f net8.0
dotnet sln add src/Versus.Api/Versus.Api.csproj

# Create Angular project (handled separately with Angular CLI)
# cd src
# ng new Versus.Web --routing --style=scss --skip-git

# Create Shared project
dotnet new classlib -n Versus.Shared -o src/Versus.Shared -f net8.0
dotnet sln add src/Versus.Shared/Versus.Shared.csproj

# Create Test projects
dotnet new nunit -n Versus.Domain.UnitTests -o tests/Versus.Domain.UnitTests -f net8.0
dotnet sln add tests/Versus.Domain.UnitTests/Versus.Domain.UnitTests.csproj

dotnet new nunit -n Versus.Application.UnitTests -o tests/Versus.Application.UnitTests -f net8.0
dotnet sln add tests/Versus.Application.UnitTests/Versus.Application.UnitTests.csproj

dotnet new nunit -n Versus.Application.IntegrationTests -o tests/Versus.Application.IntegrationTests -f net8.0
dotnet sln add tests/Versus.Application.IntegrationTests/Versus.Application.IntegrationTests.csproj

dotnet new nunit -n Versus.Api.IntegrationTests -o tests/Versus.Api.IntegrationTests -f net8.0
dotnet sln add tests/Versus.Api.IntegrationTests/Versus.Api.IntegrationTests.csproj
```

### Step 4: Add Project References
```powershell
# Application references Domain
dotnet add src/Versus.Application/Versus.Application.csproj reference src/Versus.Domain/Versus.Domain.csproj

# Infrastructure references Application and Domain
dotnet add src/Versus.Infrastructure/Versus.Infrastructure.csproj reference src/Versus.Application/Versus.Application.csproj
dotnet add src/Versus.Infrastructure/Versus.Infrastructure.csproj reference src/Versus.Domain/Versus.Domain.csproj

# API references Infrastructure, Application, and Domain
dotnet add src/Versus.Api/Versus.Api.csproj reference src/Versus.Infrastructure/Versus.Infrastructure.csproj
dotnet add src/Versus.Api/Versus.Api.csproj reference src/Versus.Application/Versus.Application.csproj
dotnet add src/Versus.Api/Versus.Api.csproj reference src/Versus.Domain/Versus.Domain.csproj
dotnet add src/Versus.Api/Versus.Api.csproj reference src/Versus.Shared/Versus.Shared.csproj

# Note: Angular Web project is separate and doesn't use project references
# It will consume the API via HTTP
```

### Step 5: Install NuGet Packages

#### Domain Project
```powershell
cd src/Versus.Domain
# No external dependencies - pure domain logic
```

#### Application Project
```powershell
cd src/Versus.Application
dotnet add package MediatR
dotnet add package FluentValidation
dotnet add package FluentValidation.DependencyInjectionExtensions
dotnet add package AutoMapper
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection
dotnet add package Microsoft.Extensions.Logging.Abstractions
```

#### Infrastructure Project
```powershell
cd src/Versus.Infrastructure
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Azure.Storage.Blobs
dotnet add package Azure.Messaging.ServiceBus
dotnet add package StackExchange.Redis
dotnet add package Microsoft.Azure.SignalR
dotnet add package Microsoft.Identity.Web
dotnet add package Serilog
dotnet add package Serilog.Sinks.ApplicationInsights
dotnet add package Hangfire
dotnet add package Hangfire.SqlServer
```

#### API Project
```powershell
cd src/Versus.Api
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add package Microsoft.Identity.Web
dotnet add package Swashbuckle.AspNetCore
dotnet add package Serilog.AspNetCore
dotnet add package Azure.Monitor.OpenTelemetry.AspNetCore
dotnet add package Microsoft.ApplicationInsights.AspNetCore
dotnet add package FluentValidation.AspNetCore
dotnet add package Microsoft.AspNetCore.SignalR.Client
dotnet add package HealthChecks.UI.Client
dotnet add package AspNetCore.HealthChecks.SqlServer
dotnet add package AspNetCore.HealthChecks.Redis
dotnet add package AspNetCore.HealthChecks.AzureStorage
```

#### Web Project (Angular)
```powershell
cd src/Versus.Web
npm install

# Install additional packages
npm install --save @angular/material @angular/cdk
npm install --save @microsoft/signalr
npm install --save rxjs
```

#### Test Projects
```powershell
cd tests/Versus.Domain.UnitTests
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
dotnet add package FluentAssertions
dotnet add package Moq
dotnet add package Bogus

cd ../Versus.Application.UnitTests
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
dotnet add package FluentAssertions
dotnet add package Moq
dotnet add package Microsoft.EntityFrameworkCore.InMemory

cd ../Versus.Application.IntegrationTests
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
dotnet add package FluentAssertions
dotnet add package Microsoft.AspNetCore.Mvc.Testing
dotnet add package Testcontainers
dotnet add package Testcontainers.MsSql

cd ../Versus.Api.IntegrationTests
dotnet add package NUnit
dotnet add package NUnit3TestAdapter
dotnet add package FluentAssertions
dotnet add package Microsoft.AspNetCore.Mvc.Testing
dotnet add package Testcontainers
```

### Step 6: Docker Compose for Local Development
Create `docker-compose.yml` in root:
```yaml
version: '3.8'

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2022-latest
    container_name: versus-sqlserver
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=YourStrong@Passw0rd
      - MSSQL_PID=Developer
    ports:
      - "1433:1433"
    volumes:
      - sqlserver-data:/var/opt/mssql
    networks:
      - versus-network

  redis:
    image: redis:7-alpine
    container_name: versus-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - versus-network

  azurite:
    image: mcr.microsoft.com/azure-storage/azurite
    container_name: versus-azurite
    ports:
      - "10000:10000"  # Blob service
      - "10001:10001"  # Queue service
      - "10002:10002"  # Table service
    volumes:
      - azurite-data:/data
    networks:
      - versus-network

  seq:
    image: datalust/seq:latest
    container_name: versus-seq
    environment:
      - ACCEPT_EULA=Y
    ports:
      - "5341:80"
    volumes:
      - seq-data:/data
    networks:
      - versus-network

volumes:
  sqlserver-data:
  redis-data:
  azurite-data:
  seq-data:

networks:
  versus-network:
    driver: bridge
```

Start containers:
```powershell
docker-compose up -d
```

### Step 7: Configuration Files

#### appsettings.Development.json (API)
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "Microsoft.EntityFrameworkCore": "Information"
    }
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost,1433;Database=VersusDb;User Id=sa;Password=YourStrong@Passw0rd;TrustServerCertificate=True;",
    "Redis": "localhost:6379",
    "AzureStorage": "UseDevelopmentStorage=true"
  },
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "yourdomain.onmicrosoft.com",
    "TenantId": "your-tenant-id",
    "ClientId": "your-client-id"
  },
  "Jwt": {
    "SecretKey": "your-super-secret-key-min-32-characters-long",
    "Issuer": "VersusApi",
    "Audience": "VersusWeb",
    "ExpirationMinutes": 60
  },
  "Azure": {
    "SignalR": {
      "ConnectionString": ""
    },
    "ServiceBus": {
      "ConnectionString": ""
    },
    "ApplicationInsights": {
      "ConnectionString": ""
    }
  },
  "Cors": {
    "AllowedOrigins": ["https://localhost:7002", "http://localhost:5002"]
  }
}
```

### Step 8: Initialize Database
```powershell
cd src/Versus.Infrastructure

# Create initial migration
dotnet ef migrations add InitialCreate --startup-project ../Versus.Api/Versus.Api.csproj

# Apply migration to database
dotnet ef database update --startup-project ../Versus.Api/Versus.Api.csproj

# Verify database
# Connect to SQL Server using SSMS or Azure Data Studio
# Server: localhost,1433
# Username: sa
# Password: YourStrong@Passw0rd
```

### Step 9: Run the Application
```powershell
# Terminal 1 - Run API
cd src/Versus.Api
dotnet run

# Terminal 2 - Run Angular Web
cd src/Versus.Web
npm start

# API will be available at: https://localhost:7001
# Swagger UI: https://localhost:7001/swagger
# Web App will be available at: http://localhost:4200
```

### Step 10: Verify Setup
```powershell
# Test API health endpoint
curl https://localhost:7001/health

# Run unit tests
dotnet test

# Run with code coverage
dotnet test --collect:"XPlat Code Coverage"
```

## ☁️ Azure Setup

### Step 1: Login to Azure
```powershell
# Login to Azure
az login

# Set subscription
az account set --subscription "Your Subscription Name"

# Verify
az account show
```

### Step 2: Create Resource Groups
```powershell
# Create resource groups for each environment
az group create --name versus-dev-rg --location eastus
az group create --name versus-staging-rg --location eastus
az group create --name versus-prod-rg --location eastus
```

### Step 3: Create Azure Resources (Development)
```powershell
# Variables
$resourceGroup = "versus-dev-rg"
$location = "eastus"

# Create Azure SQL Server
az sql server create `
  --name versus-dev-sql `
  --resource-group $resourceGroup `
  --location $location `
  --admin-user sqladmin `
  --admin-password "YourStrong@Passw0rd123"

# Create Azure SQL Database
az sql db create `
  --resource-group $resourceGroup `
  --server versus-dev-sql `
  --name versus-dev-db `
  --service-objective GP_S_Gen5_2 `
  --compute-model Serverless `
  --auto-pause-delay 60

# Create Storage Account
az storage account create `
  --name versusdevstorage `
  --resource-group $resourceGroup `
  --location $location `
  --sku Standard_LRS `
  --kind StorageV2

# Create Redis Cache
az redis create `
  --name versus-dev-redis `
  --resource-group $resourceGroup `
  --location $location `
  --sku Basic `
  --vm-size c0

# Create Application Insights
az monitor app-insights component create `
  --app versus-dev-insights `
  --location $location `
  --resource-group $resourceGroup

# Create Azure Container Registry
az acr create `
  --resource-group $resourceGroup `
  --name versusdevcr `
  --sku Basic

# Create App Service Plan
az appservice plan create `
  --name versus-dev-plan `
  --resource-group $resourceGroup `
  --location $location `
  --sku B1 `
  --is-linux

# Create App Service for API
az webapp create `
  --resource-group $resourceGroup `
  --plan versus-dev-plan `
  --name versus-dev-api `
  --deployment-container-image-name mcr.microsoft.com/dotnet/samples:aspnetapp

# Create App Service for Web
az webapp create `
  --resource-group $resourceGroup `
  --plan versus-dev-plan `
  --name versus-dev-web `
  --deployment-container-image-name mcr.microsoft.com/dotnet/samples:aspnetapp
```

### Step 4: Configure Container Apps (Alternative to App Service)
```powershell
# Create Container Apps environment
az containerapp env create `
  --name versus-dev-env `
  --resource-group $resourceGroup `
  --location $location

# Create Container App for API
az containerapp create `
  --name versus-api-dev `
  --resource-group $resourceGroup `
  --environment versus-dev-env `
  --image mcr.microsoft.com/dotnet/samples:aspnetapp `
  --target-port 8080 `
  --ingress external `
  --min-replicas 1 `
  --max-replicas 3

# Create Container App for Web
az containerapp create `
  --name versus-web-dev `
  --resource-group $resourceGroup `
  --environment versus-dev-env `
  --image mcr.microsoft.com/dotnet/samples:aspnetapp `
  --target-port 8080 `
  --ingress external `
  --min-replicas 1 `
  --max-replicas 3
```

### Step 5: Set Up Azure AD B2C
```powershell
# Create Azure AD B2C tenant (via Portal - cannot be scripted)
# 1. Go to Azure Portal
# 2. Create Azure AD B2C resource
# 3. Create user flows
# 4. Register applications
# 5. Configure custom branding
```

### Step 6: Create Azure DevOps Project
```powershell
# Install Azure DevOps CLI extension
az extension add --name azure-devops

# Login to Azure DevOps
az devops login

# Create new project
az devops project create `
  --name "Versus" `
  --description "Sports Tournament Management System" `
  --visibility private

# Create service connection (manual via Portal)
# Azure DevOps → Project Settings → Service connections
# Create:
# - Azure Resource Manager connection
# - Azure Container Registry connection
```

### Step 7: Configure Key Vault
```powershell
# Create Key Vault
az keyvault create `
  --name versus-dev-kv `
  --resource-group $resourceGroup `
  --location $location

# Add secrets
az keyvault secret set --vault-name versus-dev-kv --name "ConnectionStrings--DefaultConnection" --value "your-connection-string"
az keyvault secret set --vault-name versus-dev-kv --name "Jwt--SecretKey" --value "your-jwt-secret"
az keyvault secret set --vault-name versus-dev-kv --name "AzureAd--ClientSecret" --value "your-client-secret"

# Grant App Service access to Key Vault
$apiPrincipalId = (az webapp identity assign --name versus-dev-api --resource-group $resourceGroup --query principalId -o tsv)
az keyvault set-policy --name versus-dev-kv --object-id $apiPrincipalId --secret-permissions get list
```

## 🚀 Deployment

### Manual Deployment
```powershell
# Build Docker image
docker build -t versusdevcr.azurecr.io/versus-api:latest -f src/Versus.Api/Dockerfile .

# Login to ACR
az acr login --name versusdevcr

# Push image
docker push versusdevcr.azurecr.io/versus-api:latest

# Deploy to Container App
az containerapp update `
  --name versus-api-dev `
  --resource-group $resourceGroup `
  --image versusdevcr.azurecr.io/versus-api:latest
```

### Using Azure DevOps
See [CI/CD Pipeline Documentation](./06-CICD-PIPELINE.md)

## 📊 Monitoring Setup

### Application Insights
```powershell
# Get instrumentation key
az monitor app-insights component show `
  --app versus-dev-insights `
  --resource-group $resourceGroup `
  --query instrumentationKey -o tsv

# Add to appsettings.json
"ApplicationInsights": {
  "InstrumentationKey": "your-key-here"
}
```

### Setup Alerts
```powershell
# Create alert rule for high error rate
az monitor metrics alert create `
  --name "High Error Rate" `
  --resource-group $resourceGroup `
  --scopes /subscriptions/{subscription-id}/resourceGroups/$resourceGroup/providers/Microsoft.Web/sites/versus-dev-api `
  --condition "count requests/failed > 10" `
  --window-size 5m `
  --evaluation-frequency 1m
```

## 🔍 Verification Checklist

- [ ] .NET 8 SDK installed
- [ ] Docker Desktop running
- [ ] Local SQL Server container running
- [ ] Local Redis container running
- [ ] Database created and migrated
- [ ] API running at https://localhost:7001
- [ ] Swagger UI accessible
- [ ] Web app running at https://localhost:7002
- [ ] Unit tests passing
- [ ] Azure resource groups created
- [ ] Azure SQL Database created and accessible
- [ ] Azure Storage Account created
- [ ] Azure Redis Cache created
- [ ] Application Insights configured
- [ ] Azure DevOps project created
- [ ] CI/CD pipelines configured

## 🐛 Troubleshooting

### Issue: Connection to SQL Server failed
```powershell
# Check if container is running
docker ps

# Check SQL Server logs
docker logs versus-sqlserver

# Verify connection string in appsettings.json
```

### Issue: Redis connection failed
```powershell
# Verify Redis is running
docker ps | grep redis

# Test connection
redis-cli -h localhost -p 6379 ping
# Should return: PONG
```

### Issue: EF Core migrations fail
```powershell
# Clean and rebuild
dotnet clean
dotnet build

# Remove migrations folder
# Delete Migrations folder in Infrastructure project

# Recreate migration
dotnet ef migrations add InitialCreate --startup-project ../Versus.Api/Versus.Api.csproj
```

### Issue: Docker build fails
```powershell
# Clear Docker cache
docker system prune -a

# Rebuild image
docker build --no-cache -t versus-api:latest -f src/Versus.Api/Dockerfile .
```

## 📚 Next Steps
1. Implement domain entities (see [Clean Architecture](./03-CLEAN-ARCHITECTURE.md))
2. Create database configurations (see [Database Design](./04-DATABASE-DESIGN.md))
3. Implement API endpoints (see [API Specification](./05-API-SPECIFICATION.md))
4. Set up CI/CD (see [CI/CD Pipeline](./06-CICD-PIPELINE.md))
5. Configure Azure services (see [Azure Services](./02-AZURE-SERVICES.md))

## 🆘 Support
- Azure Documentation: https://docs.microsoft.com/azure
- .NET Documentation: https://docs.microsoft.com/dotnet
- EF Core Documentation: https://docs.microsoft.com/ef/core
- SignalR Documentation: https://docs.microsoft.com/aspnet/core/signalr
