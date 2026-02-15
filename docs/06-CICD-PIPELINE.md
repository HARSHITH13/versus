# CI/CD Pipeline Documentation

## 🎯 Overview
This document describes the Continuous Integration and Continuous Deployment (CI/CD) pipelines for the Versus Tournament Management System using Azure DevOps.

## 🏗️ Pipeline Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Developer Workflow                         │
│                                                              │
│  Git Push → Azure DevOps Repos → Trigger Build Pipeline     │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────┐
│                   BUILD PIPELINE                             │
│                                                              │
│  1. Checkout Code                                           │
│  2. Restore Dependencies                                    │
│  3. Build Solution                                          │
│  4. Run Unit Tests                                          │
│  5. Run Integration Tests                                   │
│  6. Code Coverage Analysis                                  │
│  7. Security Scanning                                       │
│  8. Build Docker Images                                     │
│  9. Push to Azure Container Registry                        │
│  10. Publish Artifacts                                      │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────┴────────────────────────────────────┐
│                  RELEASE PIPELINE                            │
│                                                              │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
│  │     DEV      │   │   STAGING    │   │  PRODUCTION  │   │
│  │              │   │              │   │              │   │
│  │ • Auto       │   │ • Auto       │   │ • Manual     │   │
│  │ • No Tests   │   │ • Smoke Test │   │ • Approval   │   │
│  │ • Fast       │   │ • E2E Tests  │   │ • Full Tests │   │
│  └──────────────┘   └──────────────┘   └──────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 📋 Repository Structure

```
versus/
├── .azuredevops/
│   ├── pipelines/
│   │   ├── build-api.yml
│   │   ├── build-web.yml
│   │   ├── build-combined.yml
│   │   └── infrastructure.yml
│   └── templates/
│       ├── build-template.yml
│       ├── test-template.yml
│       ├── docker-template.yml
│       └── deploy-template.yml
├── src/
├── tests/
├── scripts/
│   ├── build/
│   ├── test/
│   └── deploy/
└── infrastructure/
    ├── bicep/
    └── terraform/
```

## 🔨 Build Pipeline

### azure-pipelines-build.yml
```yaml
trigger:
  branches:
    include:
      - main
      - develop
      - feature/*
  paths:
    include:
      - src/**
      - tests/**

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '10.x'
  dockerRegistryServiceConnection: 'versus-acr-connection'
  imageRepository: 'versus-api'
  containerRegistry: 'versusacr.azurecr.io'
  dockerfilePath: 'src/Versus.Api/Dockerfile'
  tag: '$(Build.BuildId)'
  vmImageName: 'ubuntu-latest'

stages:
- stage: Build
  displayName: 'Build and Test'
  jobs:
  - job: BuildAPI
    displayName: 'Build API'
    pool:
      vmImage: $(vmImageName)
    
    steps:
    # Install .NET SDK
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        packageType: 'sdk'
        version: $(dotnetVersion)
        installationPath: $(Agent.ToolsDirectory)/dotnet

    # Restore NuGet packages
    - task: DotNetCoreCLI@2
      displayName: 'Restore NuGet Packages'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'
        feedsToUse: 'select'

    # Build solution
    - task: DotNetCoreCLI@2
      displayName: 'Build Solution'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '--configuration $(buildConfiguration) --no-restore'

    # Run unit tests
    - task: DotNetCoreCLI@2
      displayName: 'Run Unit Tests'
      inputs:
        command: 'test'
        projects: '**/Versus.*.UnitTests/*.csproj'
        arguments: '--configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage" --logger trx'
        publishTestResults: true

    # Run integration tests
    - task: DotNetCoreCLI@2
      displayName: 'Run Integration Tests'
      inputs:
        command: 'test'
        projects: '**/Versus.*.IntegrationTests/*.csproj'
        arguments: '--configuration $(buildConfiguration) --no-build --collect:"XPlat Code Coverage" --logger trx'
        publishTestResults: true

    # Code coverage report
    - task: PublishCodeCoverageResults@1
      displayName: 'Publish Code Coverage'
      inputs:
        codeCoverageTool: 'Cobertura'
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
        failIfCoverageEmpty: false

    # Security scanning with WhiteSource Bolt
    - task: WhiteSource@21
      displayName: 'Security Scan'
      inputs:
        cwd: '$(System.DefaultWorkingDirectory)'

    # Publish application for deployment
    - task: DotNetCoreCLI@2
      displayName: 'Publish API'
      inputs:
        command: 'publish'
        publishWebProjects: false
        projects: 'src/Versus.Api/Versus.Api.csproj'
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/api'
        zipAfterPublish: true

    # Build Docker image
    - task: Docker@2
      displayName: 'Build Docker Image'
      inputs:
        containerRegistry: $(dockerRegistryServiceConnection)
        repository: $(imageRepository)
        command: 'build'
        Dockerfile: $(dockerfilePath)
        tags: |
          $(tag)
          latest

    # Push Docker image to ACR
    - task: Docker@2
      displayName: 'Push Docker Image to ACR'
      inputs:
        containerRegistry: $(dockerRegistryServiceConnection)
        repository: $(imageRepository)
        command: 'push'
        tags: |
          $(tag)
          latest

    # Publish build artifacts
    - task: PublishBuildArtifacts@1
      displayName: 'Publish Artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'

- stage: BuildWeb
  displayName: 'Build Web UI'
  jobs:
  - job: BuildBlazor
    displayName: 'Build Blazor App'
    pool:
      vmImage: $(vmImageName)
    
    steps:
    - task: UseDotNet@2
      displayName: 'Install .NET SDK'
      inputs:
        packageType: 'sdk'
        version: $(dotnetVersion)

    - task: DotNetCoreCLI@2
      displayName: 'Build Blazor App'
      inputs:
        command: 'build'
        projects: 'src/Versus.Web/Versus.Web.csproj'
        arguments: '--configuration $(buildConfiguration)'

    - task: DotNetCoreCLI@2
      displayName: 'Publish Blazor App'
      inputs:
        command: 'publish'
        publishWebProjects: false
        projects: 'src/Versus.Web/Versus.Web.csproj'
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/web'

    # Build Docker image for Blazor
    - task: Docker@2
      displayName: 'Build Blazor Docker Image'
      inputs:
        containerRegistry: $(dockerRegistryServiceConnection)
        repository: 'versus-web'
        command: 'build'
        Dockerfile: 'src/Versus.Web/Dockerfile'
        tags: |
          $(tag)
          latest

    - task: Docker@2
      displayName: 'Push Blazor Docker Image'
      inputs:
        containerRegistry: $(dockerRegistryServiceConnection)
        repository: 'versus-web'
        command: 'push'
        tags: |
          $(tag)
          latest

    - task: PublishBuildArtifacts@1
      displayName: 'Publish Web Artifacts'
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)/web'
        ArtifactName: 'web-drop'
```

## 🚀 Release Pipeline

### azure-pipelines-release.yml
```yaml
trigger: none # Manual trigger only

resources:
  pipelines:
  - pipeline: buildPipeline
    source: 'Versus-Build'
    trigger:
      branches:
        include:
        - main
        - develop

variables:
  - group: versus-dev-variables
  - group: versus-staging-variables
  - group: versus-prod-variables

stages:
# ========================================
# DEVELOPMENT ENVIRONMENT
# ========================================
- stage: DeployDev
  displayName: 'Deploy to Development'
  condition: eq(variables['Build.SourceBranch'], 'refs/heads/develop')
  jobs:
  - deployment: DeployToAzure
    displayName: 'Deploy to Azure Dev'
    environment: 'versus-dev'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          # Download artifacts
          - download: buildPipeline
            artifact: drop

          # Azure Login
          - task: AzureCLI@2
            displayName: 'Azure Login'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az account show

          # Database Migration
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Run EF Core Migrations'
            inputs:
              azureSubscription: 'versus-azure-connection'
              authenticationType: 'servicePrincipal'
              serverName: '$(sqlServerName)'
              databaseName: '$(sqlDatabaseName)'
              sqlUsername: '$(sqlUsername)'
              sqlPassword: '$(sqlPassword)'
              deployType: 'SqlTask'
              sqlInline: |
                -- Run migrations using SQL scripts
                -- Or use EF Core migrations in next step

          # Deploy to Azure Container Apps
          - task: AzureCLI@2
            displayName: 'Deploy to Container Apps'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az containerapp update \
                  --name versus-api-dev \
                  --resource-group $(resourceGroup) \
                  --image $(containerRegistry)/versus-api:$(Build.BuildId)

          # Restart App Service (alternative)
          - task: AzureWebApp@1
            displayName: 'Deploy to App Service'
            inputs:
              azureSubscription: 'versus-azure-connection'
              appType: 'webAppContainer'
              appName: '$(appServiceName)'
              package: '$(Pipeline.Workspace)/buildPipeline/drop/api/*.zip'

          # Smoke Test
          - task: PowerShell@2
            displayName: 'Run Smoke Tests'
            inputs:
              targetType: 'inline'
              script: |
                $response = Invoke-RestMethod -Uri "https://$(appUrl)/health" -Method Get
                if ($response.status -ne "Healthy") {
                  Write-Error "Health check failed!"
                  exit 1
                }
                Write-Host "Health check passed!"

# ========================================
# STAGING ENVIRONMENT
# ========================================
- stage: DeployStaging
  displayName: 'Deploy to Staging'
  dependsOn: DeployDev
  condition: succeeded()
  jobs:
  - deployment: DeployToAzureStaging
    displayName: 'Deploy to Azure Staging'
    environment: 'versus-staging'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: buildPipeline
            artifact: drop

          # Database Migration
          - task: AzureCLI@2
            displayName: 'Run Database Migrations'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Download migration bundle
                dotnet tool install --global dotnet-ef
                
                # Apply migrations
                dotnet ef database update \
                  --connection "$(connectionString)" \
                  --project $(Pipeline.Workspace)/buildPipeline/drop/api

          # Deploy API
          - task: AzureCLI@2
            displayName: 'Deploy API to Container Apps'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az containerapp update \
                  --name versus-api-staging \
                  --resource-group $(resourceGroup) \
                  --image $(containerRegistry)/versus-api:$(Build.BuildId) \
                  --set-env-vars \
                    "ConnectionStrings__DefaultConnection=$(connectionString)" \
                    "AzureAd__TenantId=$(azureAdTenantId)"

          # Deploy Web UI
          - task: AzureCLI@2
            displayName: 'Deploy Web UI to Container Apps'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az containerapp update \
                  --name versus-web-staging \
                  --resource-group $(resourceGroup) \
                  --image $(containerRegistry)/versus-web:$(Build.BuildId)

          # Wait for deployment
          - task: PowerShell@2
            displayName: 'Wait for Deployment'
            inputs:
              targetType: 'inline'
              script: |
                Start-Sleep -Seconds 30

          # Smoke Tests
          - task: DotNetCoreCLI@2
            displayName: 'Run Smoke Tests'
            inputs:
              command: 'test'
              projects: '**/Versus.SmokeTests/*.csproj'
              arguments: '--configuration Release'

          # E2E Tests with Playwright
          - task: PowerShell@2
            displayName: 'Run E2E Tests'
            inputs:
              filePath: 'scripts/test/run-e2e-tests.ps1'
              arguments: '-BaseUrl $(appUrl)'

          # Performance Tests
          - task: AzureLoadTest@1
            displayName: 'Run Performance Tests'
            inputs:
              azureSubscription: 'versus-azure-connection'
              loadTestConfigFile: 'tests/performance/load-test.yaml'
              resourceGroup: $(resourceGroup)

# ========================================
# PRODUCTION ENVIRONMENT
# ========================================
- stage: DeployProduction
  displayName: 'Deploy to Production'
  dependsOn: DeployStaging
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: ApprovalGate
    displayName: 'Manual Approval Required'
    environment: 'versus-production'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to production..."

  - deployment: DeployToProduction
    displayName: 'Deploy to Azure Production'
    dependsOn: ApprovalGate
    environment: 'versus-production'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - download: buildPipeline
            artifact: drop

          # Backup current deployment
          - task: AzureCLI@2
            displayName: 'Backup Current Deployment'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Tag current image as backup
                az acr import \
                  --name $(containerRegistry) \
                  --source $(containerRegistry)/versus-api:latest \
                  --image versus-api:backup-$(date +%Y%m%d-%H%M%S)

          # Database Migration with Backup
          - task: SqlAzureDacpacDeployment@1
            displayName: 'Backup & Migrate Database'
            inputs:
              azureSubscription: 'versus-azure-connection'
              authenticationType: 'servicePrincipal'
              serverName: '$(sqlServerName)'
              databaseName: '$(sqlDatabaseName)'
              deployType: 'SqlTask'
              sqlInline: |
                -- Create backup before migration
                BACKUP DATABASE [$(sqlDatabaseName)] 
                TO URL = '$(backupStorageUrl)/$(sqlDatabaseName)-$(Build.BuildId).bacpac'

          # Blue-Green Deployment
          - task: AzureCLI@2
            displayName: 'Deploy to Green Slot'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Deploy to green revision
                az containerapp revision copy \
                  --name versus-api-prod \
                  --resource-group $(resourceGroup) \
                  --image $(containerRegistry)/versus-api:$(Build.BuildId) \
                  --revision-suffix green-$(Build.BuildId)

          # Traffic Splitting (10% to new version)
          - task: AzureCLI@2
            displayName: 'Route 10% Traffic to New Version'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az containerapp ingress traffic set \
                  --name versus-api-prod \
                  --resource-group $(resourceGroup) \
                  --revision-weight latest=90 green-$(Build.BuildId)=10

          # Monitor for 10 minutes
          - task: PowerShell@2
            displayName: 'Monitor New Version'
            inputs:
              targetType: 'inline'
              script: |
                Write-Host "Monitoring new version for 10 minutes..."
                Start-Sleep -Seconds 600
                
                # Check Application Insights for errors
                # If error rate > threshold, rollback
                
                Write-Host "Monitoring complete. Proceeding with full rollout."

          # Full Traffic Switch
          - task: AzureCLI@2
            displayName: 'Route 100% Traffic'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az containerapp ingress traffic set \
                  --name versus-api-prod \
                  --resource-group $(resourceGroup) \
                  --revision-weight green-$(Build.BuildId)=100

          # Purge CDN Cache
          - task: AzureCLI@2
            displayName: 'Purge CDN Cache'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az cdn endpoint purge \
                  --resource-group $(resourceGroup) \
                  --profile-name $(cdnProfileName) \
                  --name $(cdnEndpointName) \
                  --content-paths "/*"

          # Post-deployment Tests
          - task: DotNetCoreCLI@2
            displayName: 'Run Production Smoke Tests'
            inputs:
              command: 'test'
              projects: '**/Versus.SmokeTests/*.csproj'
              arguments: '--configuration Release --filter "Category=Smoke"'

          # Create Release Tag
          - task: PowerShell@2
            displayName: 'Create Git Release Tag'
            inputs:
              targetType: 'inline'
              script: |
                git tag -a "release-$(Build.BuildId)" -m "Production release $(Build.BuildId)"
                git push origin "release-$(Build.BuildId)"

          # Send Notifications
          - task: PowerShell@2
            displayName: 'Send Deployment Notification'
            inputs:
              targetType: 'inline'
              script: |
                # Send email/Slack notification
                Write-Host "Deployment completed successfully!"
```

## 🔄 Rollback Procedure

### rollback-pipeline.yml
```yaml
trigger: none

parameters:
- name: targetRevision
  displayName: 'Rollback to Revision'
  type: string
  default: 'previous'

stages:
- stage: Rollback
  displayName: 'Rollback Production'
  jobs:
  - deployment: RollbackDeployment
    displayName: 'Rollback to Previous Version'
    environment: 'versus-production'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureCLI@2
            displayName: 'Rollback Container App'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                # Get previous revision
                PREVIOUS_REVISION=$(az containerapp revision list \
                  --name versus-api-prod \
                  --resource-group $(resourceGroup) \
                  --query "[1].name" -o tsv)
                
                # Switch traffic back
                az containerapp ingress traffic set \
                  --name versus-api-prod \
                  --resource-group $(resourceGroup) \
                  --revision-weight $PREVIOUS_REVISION=100

          - task: SqlAzureDacpacDeployment@1
            displayName: 'Restore Database Backup'
            inputs:
              azureSubscription: 'versus-azure-connection'
              serverName: '$(sqlServerName)'
              databaseName: '$(sqlDatabaseName)'
              deployType: 'SqlTask'
              sqlInline: |
                -- Restore from backup if needed
                RESTORE DATABASE [$(sqlDatabaseName)]
                FROM URL = '$(backupStorageUrl)/$(sqlDatabaseName)-$(targetRevision).bacpac'
```

## 🔧 Infrastructure as Code Pipeline

### infrastructure-pipeline.yml
```yaml
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - infrastructure/**

variables:
  - group: versus-infrastructure

stages:
- stage: ValidateInfrastructure
  displayName: 'Validate Infrastructure'
  jobs:
  - job: ValidateBicep
    displayName: 'Validate Bicep Templates'
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: AzureCLI@2
      displayName: 'Validate Bicep'
      inputs:
        azureSubscription: 'versus-azure-connection'
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az bicep build --file infrastructure/bicep/main.bicep
          az deployment group validate \
            --resource-group $(resourceGroup) \
            --template-file infrastructure/bicep/main.bicep \
            --parameters infrastructure/bicep/parameters.json

- stage: DeployInfrastructure
  displayName: 'Deploy Infrastructure'
  dependsOn: ValidateInfrastructure
  jobs:
  - deployment: DeployAzureResources
    displayName: 'Deploy Azure Resources'
    environment: 'versus-infrastructure'
    pool:
      vmImage: 'ubuntu-latest'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureCLI@2
            displayName: 'Deploy Bicep Template'
            inputs:
              azureSubscription: 'versus-azure-connection'
              scriptType: 'bash'
              scriptLocation: 'inlineScript'
              inlineScript: |
                az deployment group create \
                  --resource-group $(resourceGroup) \
                  --template-file infrastructure/bicep/main.bicep \
                  --parameters infrastructure/bicep/parameters.json
```

## 📊 Pipeline Best Practices

### 1. Variable Groups in Azure DevOps

#### versus-dev-variables
```
appServiceName: versus-api-dev
resourceGroup: versus-dev-rg
sqlServerName: versus-dev-sql
sqlDatabaseName: versus-dev-db
appUrl: https://versus-dev.azurewebsites.net
```

#### versus-staging-variables
```
appServiceName: versus-api-staging
resourceGroup: versus-staging-rg
sqlServerName: versus-staging-sql
sqlDatabaseName: versus-staging-db
appUrl: https://versus-staging.azurewebsites.net
```

#### versus-prod-variables
```
appServiceName: versus-api-prod
resourceGroup: versus-prod-rg
sqlServerName: versus-prod-sql
sqlDatabaseName: versus-prod-db
appUrl: https://api.versus-sports.com
```

### 2. Service Connections
- **versus-azure-connection**: Azure Resource Manager connection
- **versus-acr-connection**: Azure Container Registry connection
- **versus-github-connection**: GitHub repository connection (if using GitHub)

### 3. Environments
Configure environments in Azure DevOps with approvals:
- **versus-dev**: Auto-deploy, no approval
- **versus-staging**: Auto-deploy, no approval
- **versus-production**: Manual approval required

### 4. Branch Policies
```
main branch:
- Require pull request reviews (2 approvers)
- Require linked work items
- Require successful build
- Require code coverage > 80%

develop branch:
- Require pull request reviews (1 approver)
- Require successful build
```

## 🔒 Security in Pipelines

### 1. Secrets Management
```yaml
# Use Azure Key Vault task
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'versus-azure-connection'
    KeyVaultName: 'versus-keyvault'
    SecretsFilter: '*'
    RunAsPreJob: true
```

### 2. Dependency Scanning
```yaml
# WhiteSource Bolt
- task: WhiteSource@21
  inputs:
    cwd: '$(System.DefaultWorkingDirectory)'

# Snyk Security Scan
- task: SnykSecurityScan@1
  inputs:
    serviceConnectionEndpoint: 'snyk-connection'
    testType: 'app'
    monitorWhen: 'always'
```

### 3. Container Scanning
```yaml
# Trivy container scanner
- task: Bash@3
  displayName: 'Scan Docker Image'
  inputs:
    targetType: 'inline'
    script: |
      docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
        aquasec/trivy image --severity HIGH,CRITICAL \
        $(containerRegistry)/versus-api:$(tag)
```

## 📈 Monitoring & Notifications

### 1. Build Quality Widgets
- Code coverage trends
- Test pass rate
- Build duration
- Deployment frequency

### 2. Notifications
```yaml
# Send notification on failure
- task: PowerShell@2
  condition: failed()
  displayName: 'Send Failure Notification'
  inputs:
    targetType: 'inline'
    script: |
      # Send to Slack/Teams/Email
      Invoke-RestMethod -Uri $(slackWebhook) -Method Post -Body @{
        text = "Build failed: $(Build.DefinitionName) #$(Build.BuildNumber)"
      } | ConvertTo-Json
```

## 🎯 Next Steps
1. Review [Project Setup Guide](./07-PROJECT-SETUP.md)
2. Set up Azure DevOps organization and project
3. Create service connections
4. Configure variable groups
5. Import pipeline templates
6. Run first build
