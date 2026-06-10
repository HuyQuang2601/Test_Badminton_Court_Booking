# Badminton Court Booking System

Backend RESTful API for managing users, badminton courts, time slots, booking approval, JWT authentication, token blacklisting, AOP audit logging, and image upload abstraction.

## Tech Stack

- Java 17
- Spring Boot 4
- Spring Security JWT
- Spring Data JPA
- H2 local database by default
- MySQL driver available for deployment
- AspectJ AOP for audit logs

## Run Locally

```powershell
.\gradlew.bat bootRun
```

H2 console:

```text
http://localhost:8080/h2-console
JDBC URL: jdbc:h2:mem:badminton
Username: sa
Password: <empty>
```

## Seed Accounts

All seed accounts use password:

```text
password123
```

| Username | Role |
| --- | --- |
| admin | ROLE_ADMIN |
| manager | ROLE_MANAGER |
| customer | ROLE_CUSTOMER |

## Main Endpoints

Auth:

```text
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
POST /api/v1/auth/change-password
POST /api/v1/auth/forgot-password
```

Public:

```text
GET /api/v1/courts
GET /api/v1/courts/{id}
GET /api/v1/time-slots
```

Admin:

```text
GET /api/v1/admin/users
GET /api/v1/admin/users/{id}
POST /api/v1/admin/users
PUT /api/v1/admin/users/{id}
DELETE /api/v1/admin/users/{id}
```

Manager/Admin:

```text
GET /api/v1/manager/courts
POST /api/v1/manager/courts
PUT /api/v1/manager/courts/{id}
DELETE /api/v1/manager/courts/{id}
GET /api/v1/manager/time-slots
POST /api/v1/manager/time-slots
PUT /api/v1/manager/time-slots/{id}
DELETE /api/v1/manager/time-slots/{id}
GET /api/v1/manager/bookings/pending
GET /api/v1/manager/bookings?date=2026-06-09&status=CONFIRMED
PATCH /api/v1/manager/bookings/{id}/approve
PATCH /api/v1/manager/bookings/{id}/reject
PATCH /api/v1/manager/bookings/{id}/check-in
```

Customer:

```text
GET /api/v1/customer/bookings
POST /api/v1/customer/bookings
GET /api/v1/customer/bookings/available-courts?date=2026-06-09&timeSlotId=1
```

Files:

```text
POST /api/v1/files/upload
POST /api/v1/files/courts/{courtId}/images
```

## Response Format

Successful responses are wrapped as:

```json
{
  "success": true,
  "message": "Created successfully",
  "data": {}
}
```

Errors are returned as:

```json
{
  "timestamp": "2026-06-09T13:00:00",
  "status": 400,
  "error": "Bad Request",
  "message": "Validation message",
  "path": "/api/v1/example"
}
```

## Notes

- JWT access tokens are short-lived and refresh tokens are persisted in the database.
- Logout stores the current access token hash in `token_blacklist`.
- Booking creation is audited by `LoggingAspect` using `@AfterReturning` and `@AfterThrowing`.
- Upload uses a `CloudStorageService` abstraction. The default `dev` provider returns a stable mock cloud URL so the project can run without external credentials. Replace it with a Cloudinary/S3 implementation for production.
