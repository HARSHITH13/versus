# Database Schema Design

## 🎯 Overview
This document describes the complete database schema for the Versus Tournament Management System using Azure SQL Database with multi-tenant architecture.

## 🏗️ Database Architecture

### Multi-Tenant Strategy
**Shared Database with Row-Level Security (RLS)**
- Every table includes `TenantId` column
- Database-level Row-Level Security policies
- Application-level tenant validation
- Cost-effective for MVP, scalable for growth

## 📊 Entity Relationship Diagram

```
┌──────────────┐       ┌──────────────┐       ┌──────────────┐
│   Tenants    │       │    Users     │       │    Roles     │
│──────────────│       │──────────────│       │──────────────│
│ Id (PK)      │◄──────┤ TenantId(FK) │       │ Id (PK)      │
│ Name         │       │ Id (PK)      │──────►│ Name         │
│ Domain       │       │ Email        │       │ Permissions  │
│ Status       │       │ RoleId (FK)  │       └──────────────┘
│ Created      │       │ FirstName    │
└──────────────┘       │ LastName     │
                       └──────────────┘
                              │
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────┐      ┌──────────────┐     ┌──────────────┐
│ Tournaments  │      │    Teams     │     │   Players    │
│──────────────│      │──────────────│     │──────────────│
│ Id (PK)      │◄─┐   │ Id (PK)      │◄─┐  │ Id (PK)      │
│ TenantId(FK) │  │   │ TenantId(FK) │  │  │ TenantId(FK) │
│ Name         │  │   │ Name         │  │  │ UserId (FK)  │
│ Description  │  │   │ LogoUrl      │  │  │ JerseyNumber │
│ Format       │  │   │ CaptainId    │  │  │ Position     │
│ StartDate    │  │   │ Created      │  │  │ Status       │
│ EndDate      │  │   └──────────────┘  │  └──────────────┘
│ Status       │  │           ▲         │          ▲
│ OrganizerId  │  │           │         │          │
└──────────────┘  │           │         │          │
        │         │   ┌───────┴─────┐   │  ┌───────┴─────┐
        │         │   │TeamTournam. │   │  │PlayerTeams  │
        │         │   │─────────────│   │  │─────────────│
        │         │   │ TeamId (FK) │   │  │ PlayerId(FK)│
        │         └───┤ Tournam.(FK)│   └──┤ TeamId (FK) │
        │             │ Seed        │      │ JoinDate    │
        │             └─────────────┘      └─────────────┘
        │
        ├─────────────────────────────────┐
        │                                 │
        ▼                                 ▼
┌──────────────┐                  ┌──────────────┐
│   Matches    │                  │  Fixtures    │
│──────────────│                  │──────────────│
│ Id (PK)      │                  │ Id (PK)      │
│ TournamentId │                  │ TournamentId │
│ HomeTeamId   │                  │ Round        │
│ AwayTeamId   │                  │ MatchNumber  │
│ MatchDate    │                  │ Generated    │
│ Venue        │                  └──────────────┘
│ Status       │
│ HomeScore    │
│ AwayScore    │
└──────────────┘
        │
        │
        ▼
┌──────────────┐       ┌──────────────┐
│MatchEvents  │       │  PointsTable │
│──────────────│       │──────────────│
│ Id (PK)      │       │ Id (PK)      │
│ MatchId (FK) │       │ TournamentId │
│ TeamId (FK)  │       │ TeamId (FK)  │
│ PlayerId(FK) │       │ Played       │
│ EventType    │       │ Won          │
│ Minute       │       │ Lost         │
│ Description  │       │ Draw         │
└──────────────┘       │ Points       │
                       │ GoalsFor     │
                       │ GoalsAgainst │
                       │ GoalDiff     │
                       │ Position     │
                       └──────────────┘
```

## 📋 Table Definitions

### 1. Tenants Table
```sql
CREATE TABLE [dbo].[Tenants]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [Name] NVARCHAR(200) NOT NULL,
    [Domain] NVARCHAR(100) NOT NULL UNIQUE,
    [ContactEmail] NVARCHAR(256) NOT NULL,
    [ContactPhone] NVARCHAR(20) NULL,
    [Status] INT NOT NULL DEFAULT 1, -- 1=Active, 2=Suspended, 3=Inactive
    [SubscriptionTier] INT NOT NULL DEFAULT 1, -- 1=Free, 2=Basic, 3=Premium
    [MaxTournaments] INT NOT NULL DEFAULT 5,
    [MaxTeamsPerTournament] INT NOT NULL DEFAULT 16,
    [StorageQuotaGB] DECIMAL(10,2) NOT NULL DEFAULT 1.0,
    [Settings] NVARCHAR(MAX) NULL, -- JSON
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [CreatedBy] NVARCHAR(128) NULL,
    [LastModified] DATETIME2 NULL,
    [LastModifiedBy] NVARCHAR(128) NULL
);

CREATE INDEX [IX_Tenants_Domain] ON [dbo].[Tenants] ([Domain]);
CREATE INDEX [IX_Tenants_Status] ON [dbo].[Tenants] ([Status]);
```

### 2. Users Table
```sql
CREATE TABLE [dbo].[Users]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [ExternalId] NVARCHAR(256) NOT NULL, -- Azure AD B2C ObjectId
    [Email] NVARCHAR(256) NOT NULL,
    [FirstName] NVARCHAR(100) NOT NULL,
    [LastName] NVARCHAR(100) NOT NULL,
    [PhoneNumber] NVARCHAR(20) NULL,
    [AvatarUrl] NVARCHAR(500) NULL,
    [RoleId] UNIQUEIDENTIFIER NOT NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [LastLogin] DATETIME2 NULL,
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [CreatedBy] NVARCHAR(128) NULL,
    [LastModified] DATETIME2 NULL,
    [LastModifiedBy] NVARCHAR(128) NULL,
    
    CONSTRAINT [FK_Users_Tenants] FOREIGN KEY ([TenantId]) 
        REFERENCES [dbo].[Tenants]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_Users_Roles] FOREIGN KEY ([RoleId]) 
        REFERENCES [dbo].[Roles]([Id])
);

CREATE UNIQUE INDEX [IX_Users_ExternalId] ON [dbo].[Users] ([ExternalId]);
CREATE INDEX [IX_Users_TenantId] ON [dbo].[Users] ([TenantId]);
CREATE INDEX [IX_Users_Email] ON [dbo].[Users] ([TenantId], [Email]);
```

### 3. Roles Table
```sql
CREATE TABLE [dbo].[Roles]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [Name] NVARCHAR(50) NOT NULL UNIQUE,
    [Description] NVARCHAR(500) NULL,
    [Permissions] NVARCHAR(MAX) NOT NULL, -- JSON array
    [IsSystemRole] BIT NOT NULL DEFAULT 0,
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);

-- Seed data
INSERT INTO [dbo].[Roles] ([Name], [Description], [Permissions], [IsSystemRole])
VALUES 
    ('Admin', 'Full system access', '["*"]', 1),
    ('Organizer', 'Create and manage tournaments', '["tournaments.*","teams.*","matches.*"]', 1),
    ('Player', 'Participate in tournaments', '["tournaments.view","teams.join","matches.view"]', 1),
    ('Viewer', 'Read-only access', '["*.view"]', 1);
```

### 4. Tournaments Table
```sql
CREATE TABLE [dbo].[Tournaments]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [Name] NVARCHAR(200) NOT NULL,
    [Description] NVARCHAR(MAX) NULL,
    [Format] INT NOT NULL, -- 1=RoundRobin, 2=Knockout, 3=League, 4=GroupStage
    [SportType] INT NOT NULL, -- 1=Football, 2=Cricket, 3=Basketball, etc.
    [StartDate] DATETIME2 NOT NULL,
    [EndDate] DATETIME2 NOT NULL,
    [RegistrationDeadline] DATETIME2 NULL,
    [Status] INT NOT NULL DEFAULT 1, -- 1=Draft, 2=Published, 3=InProgress, 4=Completed, 5=Cancelled
    [OrganizerId] UNIQUEIDENTIFIER NOT NULL,
    [BannerImageUrl] NVARCHAR(500) NULL,
    [Venue] NVARCHAR(200) NULL,
    [MaxTeams] INT NOT NULL DEFAULT 16,
    [MinTeams] INT NOT NULL DEFAULT 2,
    [EntryFee] DECIMAL(10,2) NOT NULL DEFAULT 0,
    [PrizePool] DECIMAL(10,2) NULL,
    [Rules] NVARCHAR(MAX) NULL,
    [Settings] NVARCHAR(MAX) NULL, -- JSON
    [FixturesGenerated] BIT NOT NULL DEFAULT 0,
    [ViewCount] INT NOT NULL DEFAULT 0,
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [CreatedBy] NVARCHAR(128) NULL,
    [LastModified] DATETIME2 NULL,
    [LastModifiedBy] NVARCHAR(128) NULL,
    
    CONSTRAINT [FK_Tournaments_Tenants] FOREIGN KEY ([TenantId]) 
        REFERENCES [dbo].[Tenants]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_Tournaments_Organizers] FOREIGN KEY ([OrganizerId]) 
        REFERENCES [dbo].[Users]([Id]),
    CONSTRAINT [CK_Tournaments_Dates] CHECK ([EndDate] > [StartDate])
);

CREATE INDEX [IX_Tournaments_TenantId] ON [dbo].[Tournaments] ([TenantId]);
CREATE INDEX [IX_Tournaments_Status] ON [dbo].[Tournaments] ([TenantId], [Status]);
CREATE INDEX [IX_Tournaments_Dates] ON [dbo].[Tournaments] ([StartDate], [EndDate]);
CREATE INDEX [IX_Tournaments_Organizer] ON [dbo].[Tournaments] ([OrganizerId]);
```

### 5. Teams Table
```sql
CREATE TABLE [dbo].[Teams]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [Name] NVARCHAR(200) NOT NULL,
    [ShortName] NVARCHAR(10) NULL,
    [LogoUrl] NVARCHAR(500) NULL,
    [Description] NVARCHAR(1000) NULL,
    [CaptainId] UNIQUEIDENTIFIER NULL,
    [HomeVenue] NVARCHAR(200) NULL,
    [PrimaryColor] NVARCHAR(7) NULL, -- Hex color #FFFFFF
    [SecondaryColor] NVARCHAR(7) NULL,
    [FoundedDate] DATE NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [CreatedBy] NVARCHAR(128) NULL,
    [LastModified] DATETIME2 NULL,
    [LastModifiedBy] NVARCHAR(128) NULL,
    
    CONSTRAINT [FK_Teams_Tenants] FOREIGN KEY ([TenantId]) 
        REFERENCES [dbo].[Tenants]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_Teams_Captains] FOREIGN KEY ([CaptainId]) 
        REFERENCES [dbo].[Users]([Id])
);

CREATE INDEX [IX_Teams_TenantId] ON [dbo].[Teams] ([TenantId]);
CREATE INDEX [IX_Teams_Name] ON [dbo].[Teams] ([TenantId], [Name]);
```

### 6. Players Table
```sql
CREATE TABLE [dbo].[Players]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [UserId] UNIQUEIDENTIFIER NOT NULL,
    [JerseyNumber] INT NULL,
    [Position] NVARCHAR(50) NULL,
    [PreferredFoot] NVARCHAR(10) NULL, -- Left, Right, Both
    [Height] DECIMAL(5,2) NULL, -- in cm
    [Weight] DECIMAL(5,2) NULL, -- in kg
    [DateOfBirth] DATE NULL,
    [Nationality] NVARCHAR(100) NULL,
    [Bio] NVARCHAR(2000) NULL,
    [Status] INT NOT NULL DEFAULT 1, -- 1=Active, 2=Injured, 3=Suspended, 4=Inactive
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [CreatedBy] NVARCHAR(128) NULL,
    [LastModified] DATETIME2 NULL,
    [LastModifiedBy] NVARCHAR(128) NULL,
    
    CONSTRAINT [FK_Players_Tenants] FOREIGN KEY ([TenantId]) 
        REFERENCES [dbo].[Tenants]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_Players_Users] FOREIGN KEY ([UserId]) 
        REFERENCES [dbo].[Users]([Id])
);

CREATE UNIQUE INDEX [IX_Players_UserId] ON [dbo].[Players] ([TenantId], [UserId]);
CREATE INDEX [IX_Players_TenantId] ON [dbo].[Players] ([TenantId]);
```

### 7. PlayerTeams (Join Table)
```sql
CREATE TABLE [dbo].[PlayerTeams]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [PlayerId] UNIQUEIDENTIFIER NOT NULL,
    [TeamId] UNIQUEIDENTIFIER NOT NULL,
    [JoinDate] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [LeaveDate] DATETIME2 NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [JerseyNumber] INT NULL,
    [IsCaptain] BIT NOT NULL DEFAULT 0,
    [IsViceCaptain] BIT NOT NULL DEFAULT 0,
    
    CONSTRAINT [FK_PlayerTeams_Players] FOREIGN KEY ([PlayerId]) 
        REFERENCES [dbo].[Players]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_PlayerTeams_Teams] FOREIGN KEY ([TeamId]) 
        REFERENCES [dbo].[Teams]([Id])
);

CREATE UNIQUE INDEX [IX_PlayerTeams_Active] ON [dbo].[PlayerTeams] ([PlayerId], [TeamId]) 
    WHERE [IsActive] = 1;
CREATE INDEX [IX_PlayerTeams_Team] ON [dbo].[PlayerTeams] ([TeamId]);
```

### 8. TournamentTeams (Join Table)
```sql
CREATE TABLE [dbo].[TournamentTeams]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TournamentId] UNIQUEIDENTIFIER NOT NULL,
    [TeamId] UNIQUEIDENTIFIER NOT NULL,
    [Seed] INT NULL, -- Seeding for knockout tournaments
    [GroupName] NVARCHAR(20) NULL, -- For group stage tournaments (A, B, C, etc.)
    [RegistrationDate] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [ApprovalStatus] INT NOT NULL DEFAULT 1, -- 1=Pending, 2=Approved, 3=Rejected
    [ApprovedBy] UNIQUEIDENTIFIER NULL,
    [ApprovedDate] DATETIME2 NULL,
    [PaymentStatus] INT NOT NULL DEFAULT 1, -- 1=Pending, 2=Paid, 3=Refunded
    
    CONSTRAINT [FK_TournamentTeams_Tournaments] FOREIGN KEY ([TournamentId]) 
        REFERENCES [dbo].[Tournaments]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_TournamentTeams_Teams] FOREIGN KEY ([TeamId]) 
        REFERENCES [dbo].[Teams]([Id]),
    CONSTRAINT [FK_TournamentTeams_ApprovedBy] FOREIGN KEY ([ApprovedBy]) 
        REFERENCES [dbo].[Users]([Id])
);

CREATE UNIQUE INDEX [IX_TournamentTeams_Unique] ON [dbo].[TournamentTeams] ([TournamentId], [TeamId]);
CREATE INDEX [IX_TournamentTeams_Tournament] ON [dbo].[TournamentTeams] ([TournamentId]);
```

### 9. Matches Table
```sql
CREATE TABLE [dbo].[Matches]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TournamentId] UNIQUEIDENTIFIER NOT NULL,
    [HomeTeamId] UNIQUEIDENTIFIER NOT NULL,
    [AwayTeamId] UNIQUEIDENTIFIER NOT NULL,
    [MatchNumber] INT NOT NULL,
    [Round] INT NOT NULL, -- Round number in tournament
    [RoundName] NVARCHAR(50) NULL, -- Quarter Final, Semi Final, Final, etc.
    [GroupName] NVARCHAR(20) NULL, -- For group stage
    [ScheduledDate] DATETIME2 NOT NULL,
    [ActualStartTime] DATETIME2 NULL,
    [ActualEndTime] DATETIME2 NULL,
    [Venue] NVARCHAR(200) NULL,
    [Status] INT NOT NULL DEFAULT 1, -- 1=Scheduled, 2=InProgress, 3=Completed, 4=Postponed, 5=Cancelled
    [HomeScore] INT NULL,
    [AwayScore] INT NULL,
    [HomeScoreHalfTime] INT NULL,
    [AwayScoreHalfTime] INT NULL,
    [WinnerId] UNIQUEIDENTIFIER NULL,
    [ResultType] INT NULL, -- 1=RegularTime, 2=ExtraTime, 3=Penalties
    [PenaltyScoreHome] INT NULL,
    [PenaltyScoreAway] INT NULL,
    [RefereeId] UNIQUEIDENTIFIER NULL,
    [Attendance] INT NULL,
    [Notes] NVARCHAR(MAX) NULL,
    [IsPlayoff] BIT NOT NULL DEFAULT 0,
    [ParentMatchId] UNIQUEIDENTIFIER NULL, -- For knockout stages
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [CreatedBy] NVARCHAR(128) NULL,
    [LastModified] DATETIME2 NULL,
    [LastModifiedBy] NVARCHAR(128) NULL,
    
    CONSTRAINT [FK_Matches_Tournaments] FOREIGN KEY ([TournamentId]) 
        REFERENCES [dbo].[Tournaments]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_Matches_HomeTeams] FOREIGN KEY ([HomeTeamId]) 
        REFERENCES [dbo].[Teams]([Id]),
    CONSTRAINT [FK_Matches_AwayTeams] FOREIGN KEY ([AwayTeamId]) 
        REFERENCES [dbo].[Teams]([Id]),
    CONSTRAINT [FK_Matches_Winners] FOREIGN KEY ([WinnerId]) 
        REFERENCES [dbo].[Teams]([Id]),
    CONSTRAINT [FK_Matches_Parent] FOREIGN KEY ([ParentMatchId]) 
        REFERENCES [dbo].[Matches]([Id]),
    CONSTRAINT [CK_Matches_Teams] CHECK ([HomeTeamId] <> [AwayTeamId])
);

CREATE INDEX [IX_Matches_Tournament] ON [dbo].[Matches] ([TournamentId]);
CREATE INDEX [IX_Matches_Date] ON [dbo].[Matches] ([ScheduledDate]);
CREATE INDEX [IX_Matches_Status] ON [dbo].[Matches] ([Status]);
CREATE INDEX [IX_Matches_Teams] ON [dbo].[Matches] ([HomeTeamId], [AwayTeamId]);
```

### 10. MatchEvents Table
```sql
CREATE TABLE [dbo].[MatchEvents]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [MatchId] UNIQUEIDENTIFIER NOT NULL,
    [TeamId] UNIQUEIDENTIFIER NOT NULL,
    [PlayerId] UNIQUEIDENTIFIER NULL,
    [EventType] INT NOT NULL, -- 1=Goal, 2=YellowCard, 3=RedCard, 4=Substitution, 5=Injury
    [Minute] INT NOT NULL,
    [Period] INT NOT NULL, -- 1=FirstHalf, 2=SecondHalf, 3=ExtraTime1, 4=ExtraTime2
    [Description] NVARCHAR(500) NULL,
    [AssistPlayerId] UNIQUEIDENTIFIER NULL,
    [SubstitutedPlayerId] UNIQUEIDENTIFIER NULL, -- For substitutions
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [CreatedBy] NVARCHAR(128) NULL,
    
    CONSTRAINT [FK_MatchEvents_Matches] FOREIGN KEY ([MatchId]) 
        REFERENCES [dbo].[Matches]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_MatchEvents_Teams] FOREIGN KEY ([TeamId]) 
        REFERENCES [dbo].[Teams]([Id]),
    CONSTRAINT [FK_MatchEvents_Players] FOREIGN KEY ([PlayerId]) 
        REFERENCES [dbo].[Players]([Id]),
    CONSTRAINT [FK_MatchEvents_AssistPlayers] FOREIGN KEY ([AssistPlayerId]) 
        REFERENCES [dbo].[Players]([Id]),
    CONSTRAINT [FK_MatchEvents_SubstitutedPlayers] FOREIGN KEY ([SubstitutedPlayerId]) 
        REFERENCES [dbo].[Players]([Id])
);

CREATE INDEX [IX_MatchEvents_Match] ON [dbo].[MatchEvents] ([MatchId]);
CREATE INDEX [IX_MatchEvents_Player] ON [dbo].[MatchEvents] ([PlayerId]);
CREATE INDEX [IX_MatchEvents_Type] ON [dbo].[MatchEvents] ([EventType]);
```

### 11. PointsTable Table
```sql
CREATE TABLE [dbo].[PointsTable]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TournamentId] UNIQUEIDENTIFIER NOT NULL,
    [TeamId] UNIQUEIDENTIFIER NOT NULL,
    [GroupName] NVARCHAR(20) NULL,
    [Played] INT NOT NULL DEFAULT 0,
    [Won] INT NOT NULL DEFAULT 0,
    [Draw] INT NOT NULL DEFAULT 0,
    [Lost] INT NOT NULL DEFAULT 0,
    [GoalsFor] INT NOT NULL DEFAULT 0,
    [GoalsAgainst] INT NOT NULL DEFAULT 0,
    [GoalDifference] INT NOT NULL DEFAULT 0,
    [Points] INT NOT NULL DEFAULT 0,
    [Position] INT NOT NULL DEFAULT 0,
    [Form] NVARCHAR(10) NULL, -- Last 5 results: WWDLL
    [LastUpdated] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    
    CONSTRAINT [FK_PointsTable_Tournaments] FOREIGN KEY ([TournamentId]) 
        REFERENCES [dbo].[Tournaments]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_PointsTable_Teams] FOREIGN KEY ([TeamId]) 
        REFERENCES [dbo].[Teams]([Id])
);

CREATE UNIQUE INDEX [IX_PointsTable_Unique] ON [dbo].[PointsTable] ([TournamentId], [TeamId], [GroupName]);
CREATE INDEX [IX_PointsTable_Tournament] ON [dbo].[PointsTable] ([TournamentId]);
CREATE INDEX [IX_PointsTable_Position] ON [dbo].[PointsTable] ([TournamentId], [GroupName], [Position]);
```

### 12. PlayerStatistics Table
```sql
CREATE TABLE [dbo].[PlayerStatistics]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [PlayerId] UNIQUEIDENTIFIER NOT NULL,
    [TournamentId] UNIQUEIDENTIFIER NOT NULL,
    [MatchesPlayed] INT NOT NULL DEFAULT 0,
    [Goals] INT NOT NULL DEFAULT 0,
    [Assists] INT NOT NULL DEFAULT 0,
    [YellowCards] INT NOT NULL DEFAULT 0,
    [RedCards] INT NOT NULL DEFAULT 0,
    [MinutesPlayed] INT NOT NULL DEFAULT 0,
    [CleanSheets] INT NULL, -- For goalkeepers
    [LastUpdated] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    
    CONSTRAINT [FK_PlayerStatistics_Players] FOREIGN KEY ([PlayerId]) 
        REFERENCES [dbo].[Players]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_PlayerStatistics_Tournaments] FOREIGN KEY ([TournamentId]) 
        REFERENCES [dbo].[Tournaments]([Id]) ON DELETE CASCADE
);

CREATE UNIQUE INDEX [IX_PlayerStatistics_Unique] ON [dbo].[PlayerStatistics] ([PlayerId], [TournamentId]);
CREATE INDEX [IX_PlayerStatistics_Tournament] ON [dbo].[PlayerStatistics] ([TournamentId]);
CREATE INDEX [IX_PlayerStatistics_Goals] ON [dbo].[PlayerStatistics] ([TournamentId], [Goals] DESC);
```

### 13. Notifications Table
```sql
CREATE TABLE [dbo].[Notifications]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [UserId] UNIQUEIDENTIFIER NOT NULL,
    [Type] INT NOT NULL, -- 1=Info, 2=Success, 3=Warning, 4=Error
    [Title] NVARCHAR(200) NOT NULL,
    [Message] NVARCHAR(1000) NOT NULL,
    [Link] NVARCHAR(500) NULL,
    [IsRead] BIT NOT NULL DEFAULT 0,
    [ReadAt] DATETIME2 NULL,
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    [ExpiresAt] DATETIME2 NULL,
    
    CONSTRAINT [FK_Notifications_Tenants] FOREIGN KEY ([TenantId]) 
        REFERENCES [dbo].[Tenants]([Id]) ON DELETE CASCADE,
    CONSTRAINT [FK_Notifications_Users] FOREIGN KEY ([UserId]) 
        REFERENCES [dbo].[Users]([Id]) ON DELETE CASCADE
);

CREATE INDEX [IX_Notifications_User] ON [dbo].[Notifications] ([UserId], [IsRead], [Created] DESC);
CREATE INDEX [IX_Notifications_Tenant] ON [dbo].[Notifications] ([TenantId]);
```

### 14. AuditLogs Table
```sql
CREATE TABLE [dbo].[AuditLogs]
(
    [Id] UNIQUEIDENTIFIER NOT NULL PRIMARY KEY DEFAULT NEWID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [UserId] UNIQUEIDENTIFIER NULL,
    [EntityName] NVARCHAR(100) NOT NULL,
    [EntityId] UNIQUEIDENTIFIER NOT NULL,
    [Action] NVARCHAR(50) NOT NULL, -- Create, Update, Delete
    [OldValues] NVARCHAR(MAX) NULL, -- JSON
    [NewValues] NVARCHAR(MAX) NULL, -- JSON
    [IpAddress] NVARCHAR(45) NULL,
    [UserAgent] NVARCHAR(500) NULL,
    [Created] DATETIME2 NOT NULL DEFAULT GETUTCDATE()
);

CREATE INDEX [IX_AuditLogs_Tenant] ON [dbo].[AuditLogs] ([TenantId], [Created] DESC);
CREATE INDEX [IX_AuditLogs_Entity] ON [dbo].[AuditLogs] ([EntityName], [EntityId]);
CREATE INDEX [IX_AuditLogs_User] ON [dbo].[AuditLogs] ([UserId], [Created] DESC);
```

## 🔒 Row-Level Security (RLS)

### Security Policy for Multi-Tenancy
```sql
-- Enable Row-Level Security
ALTER TABLE [dbo].[Tournaments] ENABLE CHANGE_TRACKING;
ALTER TABLE [dbo].[Teams] ENABLE CHANGE_TRACKING;
ALTER TABLE [dbo].[Players] ENABLE CHANGE_TRACKING;

-- Create security policy function
CREATE FUNCTION [dbo].[fn_TenantAccessPredicate](@TenantId UNIQUEIDENTIFIER)
    RETURNS TABLE
    WITH SCHEMABINDING
AS
    RETURN SELECT 1 AS fn_TenantAccessPredicate_result
    WHERE 
        @TenantId = CAST(SESSION_CONTEXT(N'TenantId') AS UNIQUEIDENTIFIER)
        OR IS_MEMBER('db_owner') = 1;
GO

-- Apply security policy to tables
CREATE SECURITY POLICY [dbo].[TenantSecurityPolicy]
    ADD FILTER PREDICATE [dbo].[fn_TenantAccessPredicate]([TenantId]) ON [dbo].[Tournaments],
    ADD FILTER PREDICATE [dbo].[fn_TenantAccessPredicate]([TenantId]) ON [dbo].[Teams],
    ADD FILTER PREDICATE [dbo].[fn_TenantAccessPredicate]([TenantId]) ON [dbo].[Players],
    ADD FILTER PREDICATE [dbo].[fn_TenantAccessPredicate]([TenantId]) ON [dbo].[Users],
    ADD FILTER PREDICATE [dbo].[fn_TenantAccessPredicate]([TenantId]) ON [dbo].[Notifications],
    ADD FILTER PREDICATE [dbo].[fn_TenantAccessPredicate]([TenantId]) ON [dbo].[AuditLogs]
WITH (STATE = ON);
GO

-- Set tenant context in application
-- EXEC sp_set_session_context @key = N'TenantId', @value = @TenantId;
```

## 📊 Views for Common Queries

### Tournament Standings View
```sql
CREATE VIEW [dbo].[vw_TournamentStandings]
AS
SELECT 
    pt.TournamentId,
    pt.TeamId,
    t.Name AS TeamName,
    t.LogoUrl AS TeamLogo,
    pt.GroupName,
    pt.Position,
    pt.Played,
    pt.Won,
    pt.Draw,
    pt.Lost,
    pt.GoalsFor,
    pt.GoalsAgainst,
    pt.GoalDifference,
    pt.Points,
    pt.Form
FROM [dbo].[PointsTable] pt
INNER JOIN [dbo].[Teams] t ON pt.TeamId = t.Id
WHERE t.IsActive = 1;
GO
```

### Top Scorers View
```sql
CREATE VIEW [dbo].[vw_TopScorers]
AS
SELECT 
    ps.TournamentId,
    ps.PlayerId,
    u.FirstName + ' ' + u.LastName AS PlayerName,
    u.AvatarUrl,
    t.Name AS TeamName,
    ps.Goals,
    ps.Assists,
    ps.MatchesPlayed,
    CAST(ps.Goals AS FLOAT) / NULLIF(ps.MatchesPlayed, 0) AS GoalsPerMatch,
    ROW_NUMBER() OVER (PARTITION BY ps.TournamentId ORDER BY ps.Goals DESC, ps.Assists DESC) AS Rank
FROM [dbo].[PlayerStatistics] ps
INNER JOIN [dbo].[Players] p ON ps.PlayerId = p.Id
INNER JOIN [dbo].[Users] u ON p.UserId = u.Id
INNER JOIN [dbo].[PlayerTeams] pt ON p.Id = pt.PlayerId AND pt.IsActive = 1
INNER JOIN [dbo].[Teams] t ON pt.TeamId = t.Id
WHERE ps.Goals > 0;
GO
```

### Upcoming Matches View
```sql
CREATE VIEW [dbo].[vw_UpcomingMatches]
AS
SELECT 
    m.Id AS MatchId,
    m.TournamentId,
    tour.Name AS TournamentName,
    m.MatchNumber,
    m.Round,
    m.RoundName,
    m.ScheduledDate,
    m.Venue,
    m.Status,
    ht.Name AS HomeTeam,
    ht.LogoUrl AS HomeTeamLogo,
    at.Name AS AwayTeam,
    at.LogoUrl AS AwayTeamLogo,
    m.HomeScore,
    m.AwayScore
FROM [dbo].[Matches] m
INNER JOIN [dbo].[Tournaments] tour ON m.TournamentId = tour.Id
INNER JOIN [dbo].[Teams] ht ON m.HomeTeamId = ht.Id
INNER JOIN [dbo].[Teams] at ON m.AwayTeamId = at.Id
WHERE m.Status IN (1, 2) -- Scheduled or InProgress
  AND m.ScheduledDate >= GETUTCDATE();
GO
```

## 🔄 Stored Procedures

### Update Points Table
```sql
CREATE PROCEDURE [dbo].[sp_UpdatePointsTable]
    @MatchId UNIQUEIDENTIFIER
AS
BEGIN
    SET NOCOUNT ON;
    
    DECLARE @TournamentId UNIQUEIDENTIFIER, @HomeTeamId UNIQUEIDENTIFIER, @AwayTeamId UNIQUEIDENTIFIER;
    DECLARE @HomeScore INT, @AwayScore INT, @GroupName NVARCHAR(20);
    
    -- Get match details
    SELECT 
        @TournamentId = TournamentId,
        @HomeTeamId = HomeTeamId,
        @AwayTeamId = AwayTeamId,
        @HomeScore = HomeScore,
        @AwayScore = AwayScore,
        @GroupName = GroupName
    FROM [dbo].[Matches]
    WHERE Id = @MatchId AND Status = 3; -- Completed
    
    IF @TournamentId IS NOT NULL
    BEGIN
        -- Update home team
        UPDATE [dbo].[PointsTable]
        SET 
            Played = Played + 1,
            Won = Won + CASE WHEN @HomeScore > @AwayScore THEN 1 ELSE 0 END,
            Draw = Draw + CASE WHEN @HomeScore = @AwayScore THEN 1 ELSE 0 END,
            Lost = Lost + CASE WHEN @HomeScore < @AwayScore THEN 1 ELSE 0 END,
            GoalsFor = GoalsFor + @HomeScore,
            GoalsAgainst = GoalsAgainst + @AwayScore,
            GoalDifference = (GoalsFor + @HomeScore) - (GoalsAgainst + @AwayScore),
            Points = Points + CASE 
                WHEN @HomeScore > @AwayScore THEN 3 
                WHEN @HomeScore = @AwayScore THEN 1 
                ELSE 0 END,
            LastUpdated = GETUTCDATE()
        WHERE TournamentId = @TournamentId 
          AND TeamId = @HomeTeamId
          AND (GroupName = @GroupName OR @GroupName IS NULL);
        
        -- Update away team
        UPDATE [dbo].[PointsTable]
        SET 
            Played = Played + 1,
            Won = Won + CASE WHEN @AwayScore > @HomeScore THEN 1 ELSE 0 END,
            Draw = Draw + CASE WHEN @AwayScore = @HomeScore THEN 1 ELSE 0 END,
            Lost = Lost + CASE WHEN @AwayScore < @HomeScore THEN 1 ELSE 0 END,
            GoalsFor = GoalsFor + @AwayScore,
            GoalsAgainst = GoalsAgainst + @HomeScore,
            GoalDifference = (GoalsFor + @AwayScore) - (GoalsAgainst + @HomeScore),
            Points = Points + CASE 
                WHEN @AwayScore > @HomeScore THEN 3 
                WHEN @AwayScore = @HomeScore THEN 1 
                ELSE 0 END,
            LastUpdated = GETUTCDATE()
        WHERE TournamentId = @TournamentId 
          AND TeamId = @AwayTeamId
          AND (GroupName = @GroupName OR @GroupName IS NULL);
        
        -- Update positions
        WITH RankedTeams AS (
            SELECT 
                Id,
                ROW_NUMBER() OVER (
                    PARTITION BY TournamentId, GroupName 
                    ORDER BY Points DESC, GoalDifference DESC, GoalsFor DESC
                ) AS NewPosition
            FROM [dbo].[PointsTable]
            WHERE TournamentId = @TournamentId
        )
        UPDATE pt
        SET Position = rt.NewPosition
        FROM [dbo].[PointsTable] pt
        INNER JOIN RankedTeams rt ON pt.Id = rt.Id;
    END
END;
GO
```

## 📈 Performance Optimization

### Indexes Strategy
1. **Clustered Indexes**: Primary keys (GUID)
2. **Non-Clustered Indexes**: Foreign keys, frequently queried columns
3. **Filtered Indexes**: Status columns, IsActive flags
4. **Covering Indexes**: For complex queries

### Query Optimization Tips
```sql
-- Use column store index for analytics
CREATE NONCLUSTERED COLUMNSTORE INDEX [IX_MatchEvents_Analytics]
ON [dbo].[MatchEvents] ([MatchId], [EventType], [Minute], [Created]);

-- Partitioning for large tables (by year)
CREATE PARTITION FUNCTION [PF_ByYear](DATETIME2)
AS RANGE RIGHT FOR VALUES 
    ('2024-01-01', '2025-01-01', '2026-01-01', '2027-01-01');

CREATE PARTITION SCHEME [PS_ByYear]
AS PARTITION [PF_ByYear] ALL TO ([PRIMARY]);

-- Apply partitioning to audit logs
CREATE TABLE [dbo].[AuditLogs] (
    -- columns...
) ON [PS_ByYear] ([Created]);
```

## 🎯 Next Steps
1. Review [API Specification](./05-API-SPECIFICATION.md)
2. Review [CI/CD Pipeline](./06-CICD-PIPELINE.md)
3. Set up database migrations with EF Core
