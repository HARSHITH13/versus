# API Specification

## 🎯 Overview
This document defines the REST API endpoints for the Versus Tournament Management System. All APIs follow RESTful conventions and return JSON responses.

## 🔐 Authentication & Authorization

### Authentication
- **Type**: Bearer Token (JWT)
- **Provider**: Azure AD B2C
- **Header**: `Authorization: Bearer {token}`

### Roles & Permissions
| Role | Permissions |
|------|-------------|
| **Admin** | Full access to all resources |
| **Organizer** | Create/manage tournaments, approve teams/players |
| **Player** | View tournaments, register, submit scores (limited) |
| **Viewer** | Read-only access |

### Common Headers
```http
Authorization: Bearer {jwt_token}
Content-Type: application/json
X-Tenant-Id: {tenant_id}
X-Api-Version: 1.0
```

## 📊 Response Format

### Success Response
```json
{
  "success": true,
  "data": {
    // Response data
  },
  "message": "Operation completed successfully",
  "timestamp": "2026-02-15T10:30:00Z"
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "name",
        "message": "Tournament name is required"
      }
    ]
  },
  "timestamp": "2026-02-15T10:30:00Z"
}
```

### Paginated Response
```json
{
  "success": true,
  "data": {
    "items": [...],
    "pageNumber": 1,
    "pageSize": 10,
    "totalPages": 5,
    "totalCount": 47,
    "hasPreviousPage": false,
    "hasNextPage": true
  }
}
```

## 🌐 API Endpoints

### Base URL
```
Development: https://localhost:7001/api
Staging: https://versus-staging.azurewebsites.net/api
Production: https://api.versus-sports.com/api
```

---

## 1️⃣ Authentication APIs

### POST /auth/login
Login with Azure AD B2C

**Request:**
```json
{
  "token": "azure_ad_b2c_token"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "refresh_token_here",
    "expiresIn": 3600,
    "user": {
      "id": "guid",
      "email": "user@example.com",
      "name": "John Doe",
      "role": "Organizer",
      "tenantId": "tenant_guid"
    }
  }
}
```

### POST /auth/refresh
Refresh access token

**Request:**
```json
{
  "refreshToken": "refresh_token_here"
}
```

**Response:** Same as login

### POST /auth/logout
Logout user

**Response:**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

## 2️⃣ Tournament APIs

### GET /tournaments
Get all tournaments (paginated)

**Query Parameters:**
- `pageNumber` (default: 1)
- `pageSize` (default: 10, max: 100)
- `status` (Draft, Published, InProgress, Completed, Cancelled)
- `searchTerm` (search by name/description)
- `sortBy` (startDate, name, created)
- `sortOrder` (asc, desc)

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "guid",
        "name": "Summer Championship 2026",
        "description": "Annual summer tournament",
        "format": "RoundRobin",
        "sportType": "Football",
        "startDate": "2026-06-01T00:00:00Z",
        "endDate": "2026-06-30T23:59:59Z",
        "status": "Published",
        "bannerImageUrl": "https://...",
        "maxTeams": 16,
        "registeredTeams": 12,
        "organizerName": "John Organizer",
        "venue": "Central Stadium",
        "entryFee": 100.00,
        "prizePool": 5000.00,
        "viewCount": 1234,
        "created": "2026-01-15T10:00:00Z"
      }
    ],
    "pageNumber": 1,
    "pageSize": 10,
    "totalCount": 25,
    "totalPages": 3
  }
}
```

### GET /tournaments/{id}
Get tournament details

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "guid",
    "name": "Summer Championship 2026",
    "description": "Annual summer tournament",
    "format": "RoundRobin",
    "sportType": "Football",
    "startDate": "2026-06-01T00:00:00Z",
    "endDate": "2026-06-30T23:59:59Z",
    "registrationDeadline": "2026-05-25T23:59:59Z",
    "status": "Published",
    "bannerImageUrl": "https://...",
    "venue": "Central Stadium",
    "maxTeams": 16,
    "minTeams": 4,
    "entryFee": 100.00,
    "prizePool": 5000.00,
    "rules": "Tournament rules...",
    "organizerId": "guid",
    "organizerName": "John Organizer",
    "organizerEmail": "organizer@example.com",
    "teams": [
      {
        "id": "guid",
        "name": "Red Dragons",
        "logoUrl": "https://...",
        "registrationDate": "2026-02-10T12:00:00Z",
        "approvalStatus": "Approved"
      }
    ],
    "fixturesGenerated": true,
    "viewCount": 1234,
    "created": "2026-01-15T10:00:00Z",
    "lastModified": "2026-02-01T15:30:00Z"
  }
}
```

### POST /tournaments
Create new tournament (Organizer, Admin)

**Request:**
```json
{
  "name": "Summer Championship 2026",
  "description": "Annual summer tournament",
  "format": "RoundRobin",
  "sportType": "Football",
  "startDate": "2026-06-01T00:00:00Z",
  "endDate": "2026-06-30T23:59:59Z",
  "registrationDeadline": "2026-05-25T23:59:59Z",
  "venue": "Central Stadium",
  "maxTeams": 16,
  "minTeams": 4,
  "entryFee": 100.00,
  "prizePool": 5000.00,
  "rules": "Tournament rules...",
  "bannerImage": "base64_or_url"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "guid",
    "name": "Summer Championship 2026",
    "status": "Draft"
  },
  "message": "Tournament created successfully"
}
```

### PUT /tournaments/{id}
Update tournament (Organizer, Admin)

**Request:** Same as POST

**Response:**
```json
{
  "success": true,
  "message": "Tournament updated successfully"
}
```

### DELETE /tournaments/{id}
Delete tournament (Admin only)

**Response:**
```json
{
  "success": true,
  "message": "Tournament deleted successfully"
}
```

### POST /tournaments/{id}/publish
Publish tournament (Organizer, Admin)

**Response:**
```json
{
  "success": true,
  "message": "Tournament published successfully"
}
```

### POST /tournaments/{id}/cancel
Cancel tournament (Organizer, Admin)

**Request:**
```json
{
  "reason": "Insufficient registrations"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Tournament cancelled successfully"
}
```

### POST /tournaments/{id}/generate-fixtures
Generate fixtures for tournament (Organizer, Admin)

**Response:**
```json
{
  "success": true,
  "data": {
    "fixturesGenerated": 28,
    "totalRounds": 7
  },
  "message": "Fixtures generated successfully"
}
```

---

## 3️⃣ Team APIs

### GET /teams
Get all teams (paginated)

**Query Parameters:**
- `pageNumber`, `pageSize`
- `searchTerm`
- `isActive` (true/false)

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "guid",
        "name": "Red Dragons",
        "shortName": "RDG",
        "logoUrl": "https://...",
        "description": "Championship team",
        "captainName": "John Captain",
        "homeVenue": "Dragon Stadium",
        "primaryColor": "#FF0000",
        "secondaryColor": "#FFFFFF",
        "totalPlayers": 15,
        "tournamentsPlayed": 12,
        "isActive": true,
        "created": "2025-01-01T00:00:00Z"
      }
    ],
    "pageNumber": 1,
    "totalCount": 45
  }
}
```

### GET /teams/{id}
Get team details

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "guid",
    "name": "Red Dragons",
    "shortName": "RDG",
    "logoUrl": "https://...",
    "description": "Championship team",
    "captainId": "guid",
    "captainName": "John Captain",
    "homeVenue": "Dragon Stadium",
    "primaryColor": "#FF0000",
    "secondaryColor": "#FFFFFF",
    "foundedDate": "2020-01-01",
    "players": [
      {
        "id": "guid",
        "name": "John Player",
        "jerseyNumber": 10,
        "position": "Forward",
        "joinDate": "2025-01-01T00:00:00Z",
        "isCaptain": false
      }
    ],
    "statistics": {
      "tournamentsPlayed": 12,
      "matchesWon": 45,
      "matchesLost": 15,
      "matchesDraw": 8,
      "goalsScored": 123,
      "goalsConceded": 45
    },
    "isActive": true,
    "created": "2025-01-01T00:00:00Z"
  }
}
```

### POST /teams
Create new team

**Request:**
```json
{
  "name": "Red Dragons",
  "shortName": "RDG",
  "description": "Championship team",
  "homeVenue": "Dragon Stadium",
  "primaryColor": "#FF0000",
  "secondaryColor": "#FFFFFF",
  "foundedDate": "2020-01-01",
  "logo": "base64_or_url"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "guid",
    "name": "Red Dragons"
  },
  "message": "Team created successfully"
}
```

### PUT /teams/{id}
Update team

**Request:** Same as POST

### DELETE /teams/{id}
Delete team (Admin only)

### POST /teams/{id}/players
Add player to team

**Request:**
```json
{
  "playerId": "guid",
  "jerseyNumber": 10,
  "isCaptain": false
}
```

**Response:**
```json
{
  "success": true,
  "message": "Player added to team successfully"
}
```

### DELETE /teams/{teamId}/players/{playerId}
Remove player from team

---

## 4️⃣ Tournament Registration APIs

### POST /tournaments/{tournamentId}/register
Register team for tournament

**Request:**
```json
{
  "teamId": "guid",
  "paymentMethod": "Card",
  "paymentReference": "PAY123456"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "registrationId": "guid",
    "status": "Pending",
    "paymentStatus": "Pending"
  },
  "message": "Registration submitted successfully"
}
```

### GET /tournaments/{tournamentId}/registrations
Get tournament registrations (Organizer, Admin)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "guid",
      "teamId": "guid",
      "teamName": "Red Dragons",
      "registrationDate": "2026-02-10T12:00:00Z",
      "approvalStatus": "Pending",
      "paymentStatus": "Paid"
    }
  ]
}
```

### PUT /tournaments/{tournamentId}/registrations/{registrationId}/approve
Approve registration (Organizer, Admin)

**Request:**
```json
{
  "seed": 1,
  "groupName": "A"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Registration approved successfully"
}
```

### PUT /tournaments/{tournamentId}/registrations/{registrationId}/reject
Reject registration (Organizer, Admin)

**Request:**
```json
{
  "reason": "Incomplete documentation"
}
```

---

## 5️⃣ Match APIs

### GET /tournaments/{tournamentId}/matches
Get tournament matches

**Query Parameters:**
- `status` (Scheduled, InProgress, Completed)
- `round`
- `teamId`
- `date`

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "guid",
      "matchNumber": 1,
      "round": 1,
      "roundName": "Round 1",
      "scheduledDate": "2026-06-05T15:00:00Z",
      "venue": "Central Stadium",
      "status": "Scheduled",
      "homeTeam": {
        "id": "guid",
        "name": "Red Dragons",
        "logoUrl": "https://..."
      },
      "awayTeam": {
        "id": "guid",
        "name": "Blue Tigers",
        "logoUrl": "https://..."
      },
      "homeScore": null,
      "awayScore": null
    }
  ]
}
```

### GET /matches/{id}
Get match details

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "guid",
    "tournamentId": "guid",
    "tournamentName": "Summer Championship 2026",
    "matchNumber": 1,
    "round": 1,
    "roundName": "Round 1",
    "scheduledDate": "2026-06-05T15:00:00Z",
    "actualStartTime": "2026-06-05T15:05:00Z",
    "actualEndTime": "2026-06-05T16:50:00Z",
    "venue": "Central Stadium",
    "status": "Completed",
    "homeTeam": {
      "id": "guid",
      "name": "Red Dragons",
      "logoUrl": "https://...",
      "lineup": [...]
    },
    "awayTeam": {
      "id": "guid",
      "name": "Blue Tigers",
      "logoUrl": "https://...",
      "lineup": [...]
    },
    "homeScore": 3,
    "awayScore": 2,
    "homeScoreHalfTime": 2,
    "awayScoreHalfTime": 1,
    "winnerId": "guid",
    "events": [
      {
        "id": "guid",
        "type": "Goal",
        "minute": 15,
        "period": "FirstHalf",
        "teamName": "Red Dragons",
        "playerName": "John Player",
        "description": "Header from corner"
      }
    ],
    "referee": "John Referee",
    "attendance": 5000,
    "notes": "Great match"
  }
}
```

### PUT /matches/{id}
Update match details (Organizer, Admin)

**Request:**
```json
{
  "scheduledDate": "2026-06-05T15:00:00Z",
  "venue": "Central Stadium",
  "refereeId": "guid"
}
```

### POST /matches/{id}/start
Start match (Organizer, Admin)

**Response:**
```json
{
  "success": true,
  "message": "Match started",
  "data": {
    "actualStartTime": "2026-06-05T15:05:00Z"
  }
}
```

### POST /matches/{id}/complete
Complete match (Organizer, Admin)

**Request:**
```json
{
  "homeScore": 3,
  "awayScore": 2,
  "homeScoreHalfTime": 2,
  "awayScoreHalfTime": 1,
  "notes": "Great match"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Match completed successfully"
}
```

### POST /matches/{id}/events
Add match event (Organizer, Admin)

**Request:**
```json
{
  "teamId": "guid",
  "playerId": "guid",
  "eventType": "Goal",
  "minute": 15,
  "period": "FirstHalf",
  "description": "Header from corner",
  "assistPlayerId": "guid"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "eventId": "guid"
  },
  "message": "Event added successfully"
}
```

### DELETE /matches/{matchId}/events/{eventId}
Delete match event (Organizer, Admin)

---

## 6️⃣ Points Table APIs

### GET /tournaments/{tournamentId}/points-table
Get points table

**Query Parameters:**
- `groupName` (for group stage tournaments)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "position": 1,
      "teamId": "guid",
      "teamName": "Red Dragons",
      "teamLogo": "https://...",
      "played": 10,
      "won": 7,
      "draw": 2,
      "lost": 1,
      "goalsFor": 25,
      "goalsAgainst": 10,
      "goalDifference": 15,
      "points": 23,
      "form": "WWDWW"
    }
  ]
}
```

### GET /tournaments/{tournamentId}/top-scorers
Get top scorers

**Query Parameters:**
- `limit` (default: 10)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "rank": 1,
      "playerId": "guid",
      "playerName": "John Striker",
      "avatarUrl": "https://...",
      "teamName": "Red Dragons",
      "goals": 15,
      "assists": 7,
      "matchesPlayed": 10,
      "goalsPerMatch": 1.5
    }
  ]
}
```

---

## 7️⃣ Player APIs

### GET /players
Get all players

### GET /players/{id}
Get player details

### POST /players
Create player profile

**Request:**
```json
{
  "jerseyNumber": 10,
  "position": "Forward",
  "preferredFoot": "Right",
  "height": 180,
  "weight": 75,
  "dateOfBirth": "1995-05-15",
  "nationality": "USA",
  "bio": "Professional football player"
}
```

### PUT /players/{id}
Update player profile

### GET /players/{id}/statistics
Get player statistics

**Query Parameters:**
- `tournamentId` (optional)

**Response:**
```json
{
  "success": true,
  "data": {
    "playerId": "guid",
    "playerName": "John Player",
    "overallStatistics": {
      "tournamentsPlayed": 15,
      "matchesPlayed": 120,
      "goals": 45,
      "assists": 25,
      "yellowCards": 5,
      "redCards": 0,
      "minutesPlayed": 9850
    },
    "tournamentStatistics": [
      {
        "tournamentId": "guid",
        "tournamentName": "Summer Championship 2026",
        "matchesPlayed": 10,
        "goals": 5,
        "assists": 3
      }
    ]
  }
}
```

---

## 8️⃣ Notification APIs

### GET /notifications
Get user notifications

**Query Parameters:**
- `isRead` (true/false)
- `pageNumber`, `pageSize`

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "guid",
        "type": "Success",
        "title": "Registration Approved",
        "message": "Your team has been approved for Summer Championship",
        "link": "/tournaments/guid",
        "isRead": false,
        "created": "2026-02-15T10:00:00Z"
      }
    ],
    "unreadCount": 5,
    "pageNumber": 1,
    "totalCount": 25
  }
}
```

### PUT /notifications/{id}/read
Mark notification as read

### PUT /notifications/read-all
Mark all notifications as read

### DELETE /notifications/{id}
Delete notification

---

## 9️⃣ Dashboard & Analytics APIs

### GET /dashboard/stats
Get dashboard statistics (Role-based)

**Response (Organizer):**
```json
{
  "success": true,
  "data": {
    "totalTournaments": 15,
    "activeTournaments": 3,
    "totalTeams": 45,
    "totalPlayers": 350,
    "totalMatches": 180,
    "upcomingMatches": 12,
    "recentActivity": [
      {
        "type": "MatchCompleted",
        "description": "Red Dragons vs Blue Tigers completed",
        "timestamp": "2026-02-15T16:00:00Z"
      }
    ]
  }
}
```

### GET /dashboard/analytics
Get analytics data

**Query Parameters:**
- `startDate`, `endDate`

**Response:**
```json
{
  "success": true,
  "data": {
    "tournamentTrends": [...],
    "participationStats": [...],
    "popularSports": [...],
    "revenueData": [...]
  }
}
```

---

## 🔄 Real-time SignalR Hubs

### TournamentHub

**Connection:**
```
wss://api.versus-sports.com/hubs/tournament
```

**Methods:**

#### Server → Client
```javascript
// Join tournament room
connection.invoke("JoinTournament", tournamentId);

// Receive score update
connection.on("ReceiveScoreUpdate", (matchId, homeScore, awayScore) => {
  // Update UI
});

// Receive points table update
connection.on("ReceivePointsUpdate", (tournamentId, pointsTable) => {
  // Update standings
});

// Receive match status change
connection.on("ReceiveMatchStatus", (matchId, status) => {
  // Update match status
});
```

---

## 📝 HTTP Status Codes

| Code | Description |
|------|-------------|
| 200 | OK - Request successful |
| 201 | Created - Resource created |
| 204 | No Content - Request successful, no content returned |
| 400 | Bad Request - Invalid request data |
| 401 | Unauthorized - Authentication required |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource not found |
| 409 | Conflict - Resource conflict |
| 422 | Unprocessable Entity - Validation failed |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

---

## 🔒 Rate Limiting

```
Anonymous: 100 requests per 15 minutes
Authenticated: 1000 requests per 15 minutes
Organizer/Admin: 5000 requests per 15 minutes
```

---

## 📚 API Versioning

APIs are versioned via URL path:
```
/api/v1/tournaments
/api/v2/tournaments
```

Current version: **v1**

---

## 🎯 Next Steps
1. Review [CI/CD Pipeline](./06-CICD-PIPELINE.md)
2. Review [Project Setup Guide](./07-PROJECT-SETUP.md)
3. Implement APIs using .NET 8 Web API
4. Generate Swagger/OpenAPI documentation
