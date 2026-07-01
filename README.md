# IdentityReportService Package

## What is included

- Source for `IdentityReportService`
- Shared contracts used by the service
- External database script: `database/IdentityDb.sql`

## Connection contract

This repo is the auth/report side of the library system. Keep these values aligned across all services:

- `Jwt__Issuer` = `LibraryAuth`
- `Jwt__Audience` = `LibraryUsers`
- `Jwt__Key` = same secret in every service
- `InternalApi__Key` = same internal secret in every service

Other services call this repo for internal reader/report data and event ingestion. When you run services separately, the other teams must point to this service's public URL for:

- `book.borrowed`
- `book.returned`
- `fine.paid`

If this service is moved to another machine, keep its host/port reachable from `CirculationService`.

## Database setup

1. Open SQL Server Management Studio or Azure Data Studio.
2. Connect to your SQL Server instance.
3. Run `database/IdentityDb.sql`.
4. Verify the database name matches the service connection string.

## Default connection string

```json
Server=localhost,1433;Database=IdentityDb;User Id=sa;Password=Your_password123;TrustServerCertificate=True;Encrypt=False
```

## Required environment variables

If you run this service outside the main compose stack, set these values explicitly:

```bash
ConnectionStrings__IdentityDb=Server=YOUR_SQL_SERVER;Database=IdentityDb;User Id=sa;Password=...;TrustServerCertificate=True;Encrypt=False
Jwt__Issuer=LibraryAuth
Jwt__Audience=LibraryUsers
Jwt__Key=ChangeThisKeyToSomethingAtLeast32CharsLong!
Jwt__ExpiryMinutes=480
InternalApi__Key=LibraryInternalSecretChangeMe!
```

## Run locally

```bash
dotnet restore
dotnet run --project src/IdentityReportService/IdentityReportService.csproj
```

## Run with Docker

```bash
docker build -f src/IdentityReportService/Dockerfile -t identityreportservice .
docker run --rm -p 5003:8080 identityreportservice
```

## Notes

- This service only needs SQL Server to start.
- It issues JWT tokens and stores reader/report projections.
- Demo accounts are seeded in the SQL script.
- Keep the shared JWT and internal API secrets synchronized with the other two repos.
- Circulation must be able to reach the `/integration/events/*` endpoints on this service for report projections to stay in sync.
