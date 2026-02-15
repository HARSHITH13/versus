# Clean Architecture Implementation in .NET 8

## 🎯 Overview
This document describes the Clean Architecture implementation for the Versus Tournament Management System using .NET 8, following Domain-Driven Design (DDD) principles and SOLID principles.

## 🏗️ Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                            │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │   Web API          │  │   Blazor Web UI    │                │
│  │   Controllers      │  │   Pages/Components │                │
│  │   Middleware       │  │   State Management │                │
│  │   Filters          │  │   SignalR Clients  │                │
│  └────────────────────┘  └────────────────────┘                │
└────────────────────┬───────────────────────────────────────────┘
                     │ (depends on)
┌────────────────────┴───────────────────────────────────────────┐
│                    APPLICATION LAYER                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │   Use Cases / Application Services                        │  │
│  │   - Commands (CQRS Write)                                │  │
│  │   - Queries (CQRS Read)                                  │  │
│  │   - DTOs (Data Transfer Objects)                         │  │
│  │   - Mapping Profiles (AutoMapper)                        │  │
│  │   - Validators (FluentValidation)                        │  │
│  │   - Interfaces (IServices, IRepositories)                │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────┬───────────────────────────────────────────┘
                     │ (depends on)
┌────────────────────┴───────────────────────────────────────────┐
│                    DOMAIN LAYER (Core)                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │   Entities                                                │  │
│  │   Value Objects                                           │  │
│  │   Domain Events                                           │  │
│  │   Domain Services                                         │  │
│  │   Aggregates                                              │  │
│  │   Specifications                                          │  │
│  │   Enums                                                   │  │
│  │   Exceptions                                              │  │
│  │   Interfaces (abstractions only)                         │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────┘
                     ↑
                     │ (depends on)
┌────────────────────┴───────────────────────────────────────────┐
│                    INFRASTRUCTURE LAYER                         │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │   Data Access (EF Core)                                  │  │
│  │   Repositories Implementation                            │  │
│  │   External Services                                      │  │
│  │   - Azure Storage Service                                │  │
│  │   - Email Service (SendGrid)                             │  │
│  │   - Cache Service (Redis)                                │  │
│  │   - Messaging Service (Service Bus)                      │  │
│  │   Identity (Azure AD B2C)                                │  │
│  │   Logging & Monitoring                                   │  │
│  │   Background Jobs                                        │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## 📁 Solution Structure

```
Versus.sln
│
├── src/
│   │
│   ├── 1. Core Layer
│   │   ├── Versus.Domain/                    (Core Business Logic)
│   │   │   ├── Entities/
│   │   │   │   ├── Tournament.cs
│   │   │   │   ├── Match.cs
│   │   │   │   ├── Team.cs
│   │   │   │   ├── Player.cs
│   │   │   │   ├── Fixture.cs
│   │   │   │   └── Tenant.cs
│   │   │   ├── ValueObjects/
│   │   │   │   ├── Score.cs
│   │   │   │   ├── Address.cs
│   │   │   │   └── DateRange.cs
│   │   │   ├── Aggregates/
│   │   │   │   ├── TournamentAggregate.cs
│   │   │   │   └── MatchAggregate.cs
│   │   │   ├── Enums/
│   │   │   │   ├── TournamentStatus.cs
│   │   │   │   ├── MatchStatus.cs
│   │   │   │   └── UserRole.cs
│   │   │   ├── Events/
│   │   │   │   ├── TournamentCreatedEvent.cs
│   │   │   │   ├── MatchCompletedEvent.cs
│   │   │   │   └── PlayerRegisteredEvent.cs
│   │   │   ├── Exceptions/
│   │   │   │   ├── DomainException.cs
│   │   │   │   └── InvalidTournamentException.cs
│   │   │   ├── Interfaces/
│   │   │   │   └── IRepository.cs
│   │   │   └── Specifications/
│   │   │       └── TournamentSpecifications.cs
│   │   │
│   │   └── Versus.Application/              (Use Cases & Business Rules)
│   │       ├── Common/
│   │       │   ├── Interfaces/
│   │       │   │   ├── IApplicationDbContext.cs
│   │       │   │   ├── ICacheService.cs
│   │       │   │   ├── IStorageService.cs
│   │       │   │   ├── IEmailService.cs
│   │       │   │   ├── ICurrentUserService.cs
│   │       │   │   └── IDateTime.cs
│   │       │   ├── Behaviors/
│   │       │   │   ├── ValidationBehavior.cs
│   │       │   │   ├── LoggingBehavior.cs
│   │       │   │   └── PerformanceBehavior.cs
│   │       │   ├── Exceptions/
│   │       │   │   ├── ValidationException.cs
│   │       │   │   └── NotFoundException.cs
│   │       │   └── Models/
│   │       │       ├── Result.cs
│   │       │       └── PaginatedList.cs
│   │       │
│   │       ├── Features/
│   │       │   ├── Tournaments/
│   │       │   │   ├── Commands/
│   │       │   │   │   ├── CreateTournament/
│   │       │   │   │   │   ├── CreateTournamentCommand.cs
│   │       │   │   │   │   ├── CreateTournamentCommandHandler.cs
│   │       │   │   │   │   └── CreateTournamentCommandValidator.cs
│   │       │   │   │   ├── UpdateTournament/
│   │       │   │   │   ├── DeleteTournament/
│   │       │   │   │   └── PublishTournament/
│   │       │   │   ├── Queries/
│   │       │   │   │   ├── GetTournaments/
│   │       │   │   │   │   ├── GetTournamentsQuery.cs
│   │       │   │   │   │   └── GetTournamentsQueryHandler.cs
│   │       │   │   │   ├── GetTournamentById/
│   │       │   │   │   └── GetTournamentStats/
│   │       │   │   └── DTOs/
│   │       │   │       ├── TournamentDto.cs
│   │       │   │       └── TournamentDetailDto.cs
│   │       │   │
│   │       │   ├── Matches/
│   │       │   │   ├── Commands/
│   │       │   │   │   ├── UpdateScore/
│   │       │   │   │   └── CompleteMatch/
│   │       │   │   ├── Queries/
│   │       │   │   │   └── GetMatches/
│   │       │   │   └── DTOs/
│   │       │   │
│   │       │   ├── Players/
│   │       │   │   ├── Commands/
│   │       │   │   │   ├── RegisterPlayer/
│   │       │   │   │   └── ApprovePlayer/
│   │       │   │   ├── Queries/
│   │       │   │   └── DTOs/
│   │       │   │
│   │       │   ├── Teams/
│   │       │   │   ├── Commands/
│   │       │   │   ├── Queries/
│   │       │   │   └── DTOs/
│   │       │   │
│   │       │   └── Fixtures/
│   │       │       ├── Commands/
│   │       │       │   └── GenerateFixtures/
│   │       │       ├── Queries/
│   │       │       └── DTOs/
│   │       │
│   │       └── DependencyInjection.cs
│   │
│   ├── 2. Infrastructure Layer
│   │   └── Versus.Infrastructure/           (External Concerns)
│   │       ├── Data/
│   │       │   ├── ApplicationDbContext.cs
│   │       │   ├── Configurations/
│   │       │   │   ├── TournamentConfiguration.cs
│   │       │   │   ├── MatchConfiguration.cs
│   │       │   │   └── PlayerConfiguration.cs
│   │       │   ├── Interceptors/
│   │       │   │   ├── AuditableEntityInterceptor.cs
│   │       │   │   └── TenantInterceptor.cs
│   │       │   ├── Migrations/
│   │       │   └── Repositories/
│   │       │       ├── GenericRepository.cs
│   │       │       └── TournamentRepository.cs
│   │       │
│   │       ├── Identity/
│   │       │   ├── IdentityService.cs
│   │       │   └── CurrentUserService.cs
│   │       │
│   │       ├── Services/
│   │       │   ├── AzureBlobStorageService.cs
│   │       │   ├── RedisCacheService.cs
│   │       │   ├── ServiceBusService.cs
│   │       │   ├── EmailService.cs
│   │       │   ├── SignalRService.cs
│   │       │   └── DateTimeService.cs
│   │       │
│   │       ├── BackgroundJobs/
│   │       │   ├── FixtureGenerationJob.cs
│   │       │   └── TournamentReminderJob.cs
│   │       │
│   │       └── DependencyInjection.cs
│   │
│   ├── 3. Presentation Layer
│   │   ├── Versus.Api/                      (REST API)
│   │   │   ├── Controllers/
│   │   │   │   ├── TournamentsController.cs
│   │   │   │   ├── MatchesController.cs
│   │   │   │   ├── PlayersController.cs
│   │   │   │   └── TeamsController.cs
│   │   │   ├── Hubs/
│   │   │   │   ├── TournamentHub.cs
│   │   │   │   └── AdminHub.cs
│   │   │   ├── Middleware/
│   │   │   │   ├── ExceptionHandlingMiddleware.cs
│   │   │   │   ├── TenantResolutionMiddleware.cs
│   │   │   │   └── RequestLoggingMiddleware.cs
│   │   │   ├── Filters/
│   │   │   │   ├── ApiExceptionFilterAttribute.cs
│   │   │   │   └── ValidateModelStateAttribute.cs
│   │   │   ├── Extensions/
│   │   │   │   └── ServiceCollectionExtensions.cs
│   │   │   ├── Program.cs
│   │   │   ├── appsettings.json
│   │   │   └── Dockerfile
│   │   │
│   │   └── Versus.Web/                      (Blazor WebAssembly)
│   │       ├── Pages/
│   │       │   ├── Tournaments/
│   │       │   │   ├── TournamentList.razor
│   │       │   │   ├── TournamentDetail.razor
│   │       │   │   └── CreateTournament.razor
│   │       │   ├── Matches/
│   │       │   └── Players/
│   │       ├── Shared/
│   │       │   ├── MainLayout.razor
│   │       │   ├── NavMenu.razor
│   │       │   └── Components/
│   │       ├── Services/
│   │       │   ├── TournamentService.cs
│   │       │   ├── AuthService.cs
│   │       │   └── SignalRConnectionService.cs
│   │       ├── Program.cs
│   │       ├── wwwroot/
│   │       └── Dockerfile
│   │
│   └── 4. Shared/Cross-Cutting
│       └── Versus.Shared/
│           ├── Constants/
│           ├── Extensions/
│           └── Utilities/
│
├── tests/
│   ├── Versus.Domain.UnitTests/
│   ├── Versus.Application.UnitTests/
│   ├── Versus.Application.IntegrationTests/
│   └── Versus.Api.IntegrationTests/
│
└── scripts/
    ├── database/
    └── deployment/
```

## 🎯 Key Patterns & Principles

### 1. Domain-Driven Design (DDD)

#### Entities
```csharp
// Domain/Entities/Tournament.cs
namespace Versus.Domain.Entities;

public class Tournament : BaseAuditableEntity
{
    private readonly List<Match> _matches = new();
    private readonly List<Team> _teams = new();
    private readonly List<Player> _players = new();

    public string Name { get; private set; }
    public string Description { get; private set; }
    public TournamentFormat Format { get; private set; }
    public DateRange Duration { get; private set; }
    public TournamentStatus Status { get; private set; }
    public string TenantId { get; private set; }
    public string OrganizerId { get; private set; }
    
    public IReadOnlyCollection<Match> Matches => _matches.AsReadOnly();
    public IReadOnlyCollection<Team> Teams => _teams.AsReadOnly();
    public IReadOnlyCollection<Player> Players => _players.AsReadOnly();

    private Tournament() { } // EF Core

    public static Tournament Create(
        string name, 
        string description, 
        TournamentFormat format,
        DateRange duration,
        string tenantId,
        string organizerId)
    {
        var tournament = new Tournament
        {
            Name = name,
            Description = description,
            Format = format,
            Duration = duration,
            Status = TournamentStatus.Draft,
            TenantId = tenantId,
            OrganizerId = organizerId
        };

        tournament.AddDomainEvent(new TournamentCreatedEvent(tournament));
        return tournament;
    }

    public void Publish()
    {
        if (Status != TournamentStatus.Draft)
            throw new InvalidTournamentException("Only draft tournaments can be published");

        if (_teams.Count < 2)
            throw new InvalidTournamentException("Tournament must have at least 2 teams");

        Status = TournamentStatus.Published;
        AddDomainEvent(new TournamentPublishedEvent(this));
    }

    public void AddTeam(Team team)
    {
        if (Status != TournamentStatus.Draft)
            throw new InvalidTournamentException("Cannot add teams to published tournament");

        _teams.Add(team);
    }

    public void GenerateFixtures()
    {
        if (Status != TournamentStatus.Published)
            throw new InvalidTournamentException("Only published tournaments can generate fixtures");

        // Domain logic for fixture generation
        var fixtureGenerator = new FixtureGenerator(Format);
        var fixtures = fixtureGenerator.Generate(_teams);
        
        foreach (var fixture in fixtures)
        {
            _matches.Add(fixture);
        }

        AddDomainEvent(new FixturesGeneratedEvent(this));
    }
}
```

#### Value Objects
```csharp
// Domain/ValueObjects/Score.cs
namespace Versus.Domain.ValueObjects;

public class Score : ValueObject
{
    public int HomeScore { get; private set; }
    public int AwayScore { get; private set; }

    private Score() { }

    public Score(int homeScore, int awayScore)
    {
        if (homeScore < 0) throw new ArgumentException("Score cannot be negative", nameof(homeScore));
        if (awayScore < 0) throw new ArgumentException("Score cannot be negative", nameof(awayScore));

        HomeScore = homeScore;
        AwayScore = awayScore;
    }

    public bool IsWinner(TeamPosition position)
    {
        return position switch
        {
            TeamPosition.Home => HomeScore > AwayScore,
            TeamPosition.Away => AwayScore > HomeScore,
            _ => false
        };
    }

    public bool IsDraw() => HomeScore == AwayScore;

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return HomeScore;
        yield return AwayScore;
    }
}

// Domain/ValueObjects/DateRange.cs
public class DateRange : ValueObject
{
    public DateTime StartDate { get; private set; }
    public DateTime EndDate { get; private set; }

    private DateRange() { }

    public DateRange(DateTime startDate, DateTime endDate)
    {
        if (startDate >= endDate)
            throw new ArgumentException("Start date must be before end date");

        StartDate = startDate;
        EndDate = endDate;
    }

    public bool IsActive(DateTime currentDate)
    {
        return currentDate >= StartDate && currentDate <= EndDate;
    }

    public int DurationInDays() => (EndDate - StartDate).Days;

    protected override IEnumerable<object> GetEqualityComponents()
    {
        yield return StartDate;
        yield return EndDate;
    }
}
```

#### Domain Events
```csharp
// Domain/Events/TournamentCreatedEvent.cs
namespace Versus.Domain.Events;

public class TournamentCreatedEvent : BaseEvent
{
    public TournamentCreatedEvent(Tournament tournament)
    {
        Tournament = tournament;
    }

    public Tournament Tournament { get; }
}
```

### 2. CQRS Pattern (Command Query Responsibility Segregation)

#### Command
```csharp
// Application/Features/Tournaments/Commands/CreateTournament/CreateTournamentCommand.cs
namespace Versus.Application.Features.Tournaments.Commands.CreateTournament;

public record CreateTournamentCommand : IRequest<Result<Guid>>
{
    public string Name { get; init; }
    public string Description { get; init; }
    public TournamentFormat Format { get; init; }
    public DateTime StartDate { get; init; }
    public DateTime EndDate { get; init; }
    public List<Guid> TeamIds { get; init; }
}

// Application/Features/Tournaments/Commands/CreateTournament/CreateTournamentCommandValidator.cs
public class CreateTournamentCommandValidator : AbstractValidator<CreateTournamentCommand>
{
    public CreateTournamentCommandValidator()
    {
        RuleFor(v => v.Name)
            .NotEmpty().WithMessage("Tournament name is required.")
            .MaximumLength(200).WithMessage("Tournament name must not exceed 200 characters.");

        RuleFor(v => v.Description)
            .MaximumLength(2000).WithMessage("Description must not exceed 2000 characters.");

        RuleFor(v => v.StartDate)
            .GreaterThan(DateTime.UtcNow).WithMessage("Start date must be in the future.");

        RuleFor(v => v.EndDate)
            .GreaterThan(v => v.StartDate).WithMessage("End date must be after start date.");

        RuleFor(v => v.TeamIds)
            .NotEmpty().WithMessage("At least 2 teams are required.")
            .Must(teams => teams.Count >= 2).WithMessage("At least 2 teams are required.");
    }
}

// Application/Features/Tournaments/Commands/CreateTournament/CreateTournamentCommandHandler.cs
public class CreateTournamentCommandHandler : IRequestHandler<CreateTournamentCommand, Result<Guid>>
{
    private readonly IApplicationDbContext _context;
    private readonly ICurrentUserService _currentUser;
    private readonly IMapper _mapper;

    public CreateTournamentCommandHandler(
        IApplicationDbContext context,
        ICurrentUserService currentUser,
        IMapper mapper)
    {
        _context = context;
        _currentUser = currentUser;
        _mapper = mapper;
    }

    public async Task<Result<Guid>> Handle(CreateTournamentCommand request, CancellationToken cancellationToken)
    {
        // Get teams
        var teams = await _context.Teams
            .Where(t => request.TeamIds.Contains(t.Id))
            .ToListAsync(cancellationToken);

        if (teams.Count != request.TeamIds.Count)
            return Result<Guid>.Failure("One or more teams not found");

        // Create tournament using domain logic
        var dateRange = new DateRange(request.StartDate, request.EndDate);
        var tournament = Tournament.Create(
            request.Name,
            request.Description,
            request.Format,
            dateRange,
            _currentUser.TenantId,
            _currentUser.UserId);

        // Add teams
        foreach (var team in teams)
        {
            tournament.AddTeam(team);
        }

        _context.Tournaments.Add(tournament);
        await _context.SaveChangesAsync(cancellationToken);

        return Result<Guid>.Success(tournament.Id);
    }
}
```

#### Query
```csharp
// Application/Features/Tournaments/Queries/GetTournaments/GetTournamentsQuery.cs
namespace Versus.Application.Features.Tournaments.Queries.GetTournaments;

public record GetTournamentsQuery : IRequest<Result<PaginatedList<TournamentDto>>>
{
    public int PageNumber { get; init; } = 1;
    public int PageSize { get; init; } = 10;
    public TournamentStatus? Status { get; init; }
    public string SearchTerm { get; init; }
}

// Application/Features/Tournaments/Queries/GetTournaments/GetTournamentsQueryHandler.cs
public class GetTournamentsQueryHandler : IRequestHandler<GetTournamentsQuery, Result<PaginatedList<TournamentDto>>>
{
    private readonly IApplicationDbContext _context;
    private readonly IMapper _mapper;
    private readonly ICurrentUserService _currentUser;
    private readonly ICacheService _cache;

    public GetTournamentsQueryHandler(
        IApplicationDbContext context,
        IMapper mapper,
        ICurrentUserService currentUser,
        ICacheService cache)
    {
        _context = context;
        _mapper = mapper;
        _currentUser = currentUser;
        _cache = cache;
    }

    public async Task<Result<PaginatedList<TournamentDto>>> Handle(
        GetTournamentsQuery request, 
        CancellationToken cancellationToken)
    {
        var cacheKey = $"tournaments:{_currentUser.TenantId}:{request.PageNumber}:{request.PageSize}:{request.Status}:{request.SearchTerm}";
        
        var cached = await _cache.GetAsync<PaginatedList<TournamentDto>>(cacheKey);
        if (cached != null)
            return Result<PaginatedList<TournamentDto>>.Success(cached);

        var query = _context.Tournaments
            .Where(t => t.TenantId == _currentUser.TenantId)
            .AsQueryable();

        if (request.Status.HasValue)
            query = query.Where(t => t.Status == request.Status.Value);

        if (!string.IsNullOrWhiteSpace(request.SearchTerm))
            query = query.Where(t => t.Name.Contains(request.SearchTerm));

        var tournaments = await query
            .OrderByDescending(t => t.Created)
            .ProjectTo<TournamentDto>(_mapper.ConfigurationProvider)
            .PaginatedListAsync(request.PageNumber, request.PageSize);

        await _cache.SetAsync(cacheKey, tournaments, TimeSpan.FromMinutes(5));

        return Result<PaginatedList<TournamentDto>>.Success(tournaments);
    }
}
```

### 3. MediatR Pipeline Behaviors

```csharp
// Application/Common/Behaviors/ValidationBehavior.cs
public class ValidationBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        if (_validators.Any())
        {
            var context = new ValidationContext<TRequest>(request);

            var validationResults = await Task.WhenAll(
                _validators.Select(v => v.ValidateAsync(context, cancellationToken)));

            var failures = validationResults
                .Where(r => r.Errors.Any())
                .SelectMany(r => r.Errors)
                .ToList();

            if (failures.Any())
                throw new ValidationException(failures);
        }

        return await next();
    }
}

// Application/Common/Behaviors/LoggingBehavior.cs
public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;
    private readonly ICurrentUserService _currentUser;

    public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger, ICurrentUserService currentUser)
    {
        _logger = logger;
        _currentUser = currentUser;
    }

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        var requestName = typeof(TRequest).Name;
        var userId = _currentUser.UserId ?? "Anonymous";

        _logger.LogInformation("Versus Request: {Name} {@UserId} {@Request}",
            requestName, userId, request);

        var response = await next();

        _logger.LogInformation("Versus Response: {Name} {@Response}", requestName, response);

        return response;
    }
}

// Application/Common/Behaviors/PerformanceBehavior.cs
public class PerformanceBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    private readonly Stopwatch _timer;
    private readonly ILogger<PerformanceBehavior<TRequest, TResponse>> _logger;

    public PerformanceBehavior(ILogger<PerformanceBehavior<TRequest, TResponse>> logger)
    {
        _timer = new Stopwatch();
        _logger = logger;
    }

    public async Task<TResponse> Handle(TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken cancellationToken)
    {
        _timer.Start();

        var response = await next();

        _timer.Stop();

        var elapsedMilliseconds = _timer.ElapsedMilliseconds;

        if (elapsedMilliseconds > 500)
        {
            var requestName = typeof(TRequest).Name;

            _logger.LogWarning("Versus Long Running Request: {Name} ({ElapsedMilliseconds} milliseconds) {@Request}",
                requestName, elapsedMilliseconds, request);
        }

        return response;
    }
}
```

### 4. Repository Pattern

```csharp
// Application/Common/Interfaces/IRepository.cs
public interface IRepository<T> where T : BaseEntity
{
    Task<T> GetByIdAsync(Guid id, CancellationToken cancellationToken = default);
    Task<IReadOnlyList<T>> GetAllAsync(CancellationToken cancellationToken = default);
    Task<IReadOnlyList<T>> GetAsync(ISpecification<T> spec, CancellationToken cancellationToken = default);
    Task<T> AddAsync(T entity, CancellationToken cancellationToken = default);
    Task UpdateAsync(T entity, CancellationToken cancellationToken = default);
    Task DeleteAsync(T entity, CancellationToken cancellationToken = default);
    Task<int> CountAsync(ISpecification<T> spec, CancellationToken cancellationToken = default);
}

// Infrastructure/Data/Repositories/GenericRepository.cs
public class GenericRepository<T> : IRepository<T> where T : BaseEntity
{
    protected readonly ApplicationDbContext _context;

    public GenericRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public virtual async Task<T> GetByIdAsync(Guid id, CancellationToken cancellationToken = default)
    {
        return await _context.Set<T>().FindAsync(new object[] { id }, cancellationToken);
    }

    public virtual async Task<IReadOnlyList<T>> GetAllAsync(CancellationToken cancellationToken = default)
    {
        return await _context.Set<T>().ToListAsync(cancellationToken);
    }

    public virtual async Task<IReadOnlyList<T>> GetAsync(ISpecification<T> spec, CancellationToken cancellationToken = default)
    {
        return await ApplySpecification(spec).ToListAsync(cancellationToken);
    }

    public virtual async Task<T> AddAsync(T entity, CancellationToken cancellationToken = default)
    {
        await _context.Set<T>().AddAsync(entity, cancellationToken);
        await _context.SaveChangesAsync(cancellationToken);
        return entity;
    }

    public virtual async Task UpdateAsync(T entity, CancellationToken cancellationToken = default)
    {
        _context.Entry(entity).State = EntityState.Modified;
        await _context.SaveChangesAsync(cancellationToken);
    }

    public virtual async Task DeleteAsync(T entity, CancellationToken cancellationToken = default)
    {
        _context.Set<T>().Remove(entity);
        await _context.SaveChangesAsync(cancellationToken);
    }

    public virtual async Task<int> CountAsync(ISpecification<T> spec, CancellationToken cancellationToken = default)
    {
        return await ApplySpecification(spec).CountAsync(cancellationToken);
    }

    private IQueryable<T> ApplySpecification(ISpecification<T> spec)
    {
        return SpecificationEvaluator<T>.GetQuery(_context.Set<T>().AsQueryable(), spec);
    }
}
```

## 📦 Dependency Injection Configuration

```csharp
// Application/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        services.AddValidatorsFromAssembly(Assembly.GetExecutingAssembly());
        services.AddMediatR(cfg => {
            cfg.RegisterServicesFromAssembly(Assembly.GetExecutingAssembly());
            cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(ValidationBehavior<,>));
            cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(LoggingBehavior<,>));
            cfg.AddBehavior(typeof(IPipelineBehavior<,>), typeof(PerformanceBehavior<,>));
        });
        services.AddAutoMapper(Assembly.GetExecutingAssembly());

        return services;
    }
}

// Infrastructure/DependencyInjection.cs
public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration configuration)
    {
        // Database
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(
                configuration.GetConnectionString("DefaultConnection"),
                b => b.MigrationsAssembly(typeof(ApplicationDbContext).Assembly.FullName)));

        services.AddScoped<IApplicationDbContext>(provider => provider.GetRequiredService<ApplicationDbContext>());

        // Repositories
        services.AddScoped(typeof(IRepository<>), typeof(GenericRepository<>));

        // Services
        services.AddSingleton<ICacheService, RedisCacheService>();
        services.AddScoped<IStorageService, AzureBlobStorageService>();
        services.AddScoped<IEmailService, EmailService>();
        services.AddScoped<IDateTime, DateTimeService>();
        services.AddScoped<ICurrentUserService, CurrentUserService>();

        // Azure Services
        services.AddStackExchangeRedisCache(options =>
        {
            options.Configuration = configuration.GetConnectionString("Redis");
        });

        services.AddAzureClients(builder =>
        {
            builder.AddBlobServiceClient(configuration.GetConnectionString("AzureStorage"));
            builder.AddServiceBusClient(configuration.GetConnectionString("ServiceBus"));
        });

        return services;
    }
}
```

## 🎯 Next Steps
1. Review [Database Design](./04-DATABASE-DESIGN.md)
2. Review [API Specification](./05-API-SPECIFICATION.md)
3. Review [CI/CD Pipeline](./06-CICD-PIPELINE.md)
