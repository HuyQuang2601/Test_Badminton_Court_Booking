# Testing Guide

This guide tests the main features required by the SRS and grading sheet.

## 1. Start The Project

Run the server:

```powershell
.\gradlew.bat bootRun
```

Base URL:

```text
http://localhost:8080
```

Run automated build/context test:

```powershell
.\gradlew.bat test
```

Expected result:

```text
BUILD SUCCESSFUL
```

## 2. Seed Accounts

The project creates these accounts automatically on startup.

| Username | Password | Role |
| --- | --- | --- |
| admin | password123 | ROLE_ADMIN |
| manager | password123 | ROLE_MANAGER |
| customer | password123 | ROLE_CUSTOMER |

## 3. H2 Database Console

Open:

```text
http://localhost:8080/h2-console
```

Use:

```text
JDBC URL: jdbc:h2:mem:badminton
Username: sa
Password: <empty>
```

Useful tables to inspect:

```sql
SELECT * FROM USERS;
SELECT * FROM USER_ROLES;
SELECT * FROM COURTS;
SELECT * FROM TIME_SLOTS;
SELECT * FROM BOOKINGS;
SELECT * FROM REFRESH_TOKENS;
SELECT * FROM TOKEN_BLACKLIST;
SELECT * FROM AUDIT_LOGS;
```

## 4. Postman Setup

Create these Postman variables:

```text
baseUrl = http://localhost:8080
adminToken =
managerToken =
customerToken =
refreshToken =
bookingId =
```

For secured requests, add header:

```text
Authorization: Bearer {{customerToken}}
```

Change the token variable depending on the role being tested.

## 5. FR-01 Login And JWT

Request:

```http
POST {{baseUrl}}/api/v1/auth/login
Content-Type: application/json
```

Body:

```json
{
  "username": "customer",
  "password": "password123"
}
```

Expected:

```text
HTTP 200
success = true
data.accessToken exists
data.refreshToken exists
data.tokenType = Bearer
```

Save:

```text
customerToken = data.accessToken
refreshToken = data.refreshToken
```

Repeat login for manager:

```json
{
  "username": "manager",
  "password": "password123"
}
```

Save:

```text
managerToken = data.accessToken
```

Repeat login for admin:

```json
{
  "username": "admin",
  "password": "password123"
}
```

Save:

```text
adminToken = data.accessToken
```

Test wrong password:

```json
{
  "username": "customer",
  "password": "wrong"
}
```

Expected:

```text
HTTP 401
```

## 6. FR-02 Refresh Token

Request:

```http
POST {{baseUrl}}/api/v1/auth/refresh
Content-Type: application/json
```

Body:

```json
{
  "refreshToken": "{{refreshToken}}"
}
```

Expected:

```text
HTTP 200
data.accessToken exists
```

Update:

```text
customerToken = data.accessToken
```

## 7. FR-04 Register Customer

Request:

```http
POST {{baseUrl}}/api/v1/auth/register
Content-Type: application/json
```

Body:

```json
{
  "username": "student01",
  "email": "student01@example.com",
  "password": "password123",
  "fullName": "Student 01",
  "phone": "0912345678"
}
```

Expected:

```text
HTTP 201
data.username = student01
data.roles contains ROLE_CUSTOMER
```

Test duplicate username by sending again.

Expected:

```text
HTTP 409
```

## 8. Public Court And Time Slot APIs

Get courts:

```http
GET {{baseUrl}}/api/v1/courts
```

Expected:

```text
HTTP 200
data contains seed courts
```

Get one court:

```http
GET {{baseUrl}}/api/v1/courts/1
```

Expected:

```text
HTTP 200
data.id = 1
```

Get time slots:

```http
GET {{baseUrl}}/api/v1/time-slots
```

Expected:

```text
HTTP 200
data contains seed time slots
```

## 9. FR-05 Admin User CRUD, Search, Pagination

Use:

```text
Authorization: Bearer {{adminToken}}
```

Search users:

```http
GET {{baseUrl}}/api/v1/admin/users?keyword=admin&page=0&size=10
```

Expected:

```text
HTTP 200
data.content contains admin
```

Create manager user:

```http
POST {{baseUrl}}/api/v1/admin/users
Content-Type: application/json
Authorization: Bearer {{adminToken}}
```

Body:

```json
{
  "username": "manager02",
  "email": "manager02@example.com",
  "password": "password123",
  "fullName": "Manager 02",
  "phone": "0902222222",
  "roles": ["ROLE_MANAGER"],
  "enabled": true
}
```

Expected:

```text
HTTP 201
```

Get user:

```http
GET {{baseUrl}}/api/v1/admin/users/1
Authorization: Bearer {{adminToken}}
```

Expected:

```text
HTTP 200
```

Update user:

```http
PUT {{baseUrl}}/api/v1/admin/users/1
Content-Type: application/json
Authorization: Bearer {{adminToken}}
```

Body:

```json
{
  "email": "admin@badminton.local",
  "fullName": "System Admin Updated",
  "phone": "0900000000",
  "roles": ["ROLE_ADMIN"],
  "enabled": true
}
```

Expected:

```text
HTTP 200
data.fullName = System Admin Updated
```

Authorization check:

```http
GET {{baseUrl}}/api/v1/admin/users
Authorization: Bearer {{customerToken}}
```

Expected:

```text
HTTP 403
```

## 10. Manager Court CRUD

Use:

```text
Authorization: Bearer {{managerToken}}
```

List managed courts:

```http
GET {{baseUrl}}/api/v1/manager/courts?page=0&size=10
Authorization: Bearer {{managerToken}}
```

Expected:

```text
HTTP 200
```

Create court:

```http
POST {{baseUrl}}/api/v1/manager/courts
Content-Type: application/json
Authorization: Bearer {{managerToken}}
```

Body:

```json
{
  "name": "Court B1",
  "address": "District 3, Ho Chi Minh City",
  "description": "New indoor court",
  "hourlyPrice": 150000,
  "active": true,
  "imageUrls": []
}
```

Expected:

```text
HTTP 201
```

Update court:

```http
PUT {{baseUrl}}/api/v1/manager/courts/1
Content-Type: application/json
Authorization: Bearer {{managerToken}}
```

Body:

```json
{
  "name": "Court A1 Updated",
  "address": "District 1, Ho Chi Minh City",
  "description": "Updated court description",
  "hourlyPrice": 130000,
  "active": true,
  "imageUrls": []
}
```

Expected:

```text
HTTP 200
data.name = Court A1 Updated
```

## 11. Manager Time Slot CRUD

Use:

```text
Authorization: Bearer {{managerToken}}
```

List time slots:

```http
GET {{baseUrl}}/api/v1/manager/time-slots
Authorization: Bearer {{managerToken}}
```

Expected:

```text
HTTP 200
```

Create time slot:

```http
POST {{baseUrl}}/api/v1/manager/time-slots
Content-Type: application/json
Authorization: Bearer {{managerToken}}
```

Body:

```json
{
  "startTime": "22:00:00",
  "endTime": "23:00:00",
  "active": true
}
```

Expected:

```text
HTTP 201
```

Invalid time slot:

```json
{
  "startTime": "23:00:00",
  "endTime": "22:00:00",
  "active": true
}
```

Expected:

```text
HTTP 400
```

## 12. FR-06 Customer Booking

Use:

```text
Authorization: Bearer {{customerToken}}
```

Check available courts:

```http
GET {{baseUrl}}/api/v1/customer/bookings/available-courts?date=2026-06-10&timeSlotId=1
Authorization: Bearer {{customerToken}}
```

Expected:

```text
HTTP 200
data contains available courts
```

Create booking:

```http
POST {{baseUrl}}/api/v1/customer/bookings
Content-Type: application/json
Authorization: Bearer {{customerToken}}
```

Body:

```json
{
  "courtId": 1,
  "bookingDate": "2026-06-10",
  "timeSlotId": 1,
  "note": "Test booking"
}
```

Expected:

```text
HTTP 201
data.status = PENDING
```

Save:

```text
bookingId = data.id
```

Test duplicate booking with the same body.

Expected:

```text
HTTP 409
```

## 13. FR-07 Customer Booking History

Request:

```http
GET {{baseUrl}}/api/v1/customer/bookings
Authorization: Bearer {{customerToken}}
```

Expected:

```text
HTTP 200
data contains the created booking
```

## 14. FR-08 Manager Approve, Reject, Check-In

Use:

```text
Authorization: Bearer {{managerToken}}
```

View pending bookings:

```http
GET {{baseUrl}}/api/v1/manager/bookings/pending
Authorization: Bearer {{managerToken}}
```

Expected:

```text
HTTP 200
data contains booking with PENDING status
```

Approve booking:

```http
PATCH {{baseUrl}}/api/v1/manager/bookings/{{bookingId}}/approve
Authorization: Bearer {{managerToken}}
```

Expected:

```text
HTTP 200
data.status = CONFIRMED
```

Check confirmed bookings:

```http
GET {{baseUrl}}/api/v1/manager/bookings?date=2026-06-10&status=CONFIRMED
Authorization: Bearer {{managerToken}}
```

Expected:

```text
HTTP 200
data contains approved booking
```

Check in:

```http
PATCH {{baseUrl}}/api/v1/manager/bookings/{{bookingId}}/check-in
Authorization: Bearer {{managerToken}}
```

Expected:

```text
HTTP 200
data.status = CHECKED_IN
```

Reject flow test:

```text
Create another booking on a different court/time slot, then call:
PATCH /api/v1/manager/bookings/{newBookingId}/reject
```

Expected:

```text
HTTP 200
data.status = REJECTED
```

## 15. FR-09 Upload Court Images

Use Postman `form-data`.

Generic upload:

```http
POST {{baseUrl}}/api/v1/files/upload
Authorization: Bearer {{managerToken}}
Content-Type: multipart/form-data
```

Body:

```text
key: file
type: File
value: select a .png or .jpg file
```

Expected:

```text
HTTP 200
data.url exists
```

Upload court image:

```http
POST {{baseUrl}}/api/v1/files/courts/1/images
Authorization: Bearer {{managerToken}}
Content-Type: multipart/form-data
```

Body:

```text
key: file
type: File
value: select a .png or .jpg file
```

Expected:

```text
HTTP 200
data.imageUrls contains uploaded URL
```

Invalid file type test:

```text
Upload a .txt file
```

Expected:

```text
HTTP 400
Only PNG and JPG images are allowed
```

Note:

```text
Current implementation uses dev cloud storage and returns a mock URL.
This is enough to test API behavior, but not a real Cloudinary/S3 integration.
```

## 16. FR-10 Change Password And Forgot Password

Change password:

```http
POST {{baseUrl}}/api/v1/auth/change-password
Content-Type: application/json
Authorization: Bearer {{customerToken}}
```

Body:

```json
{
  "currentPassword": "password123",
  "newPassword": "newPassword123"
}
```

Expected:

```text
HTTP 200
```

Login again with old password.

Expected:

```text
HTTP 401
```

Login with new password.

Expected:

```text
HTTP 200
```

Forgot password:

```http
POST {{baseUrl}}/api/v1/auth/forgot-password
Content-Type: application/json
```

Body:

```json
{
  "email": "customer@badminton.local"
}
```

Expected:

```text
HTTP 200
Message says reset instructions will be sent if email exists
```

Note:

```text
Forgot password is currently a safe stub. It does not send real email or reset token.
```

## 17. FR-03 Logout And Token Blacklist

Use a valid token:

```http
POST {{baseUrl}}/api/v1/auth/logout
Authorization: Bearer {{customerToken}}
```

Expected:

```text
HTTP 200
```

Use the same old token again:

```http
GET {{baseUrl}}/api/v1/customer/bookings
Authorization: Bearer {{customerToken}}
```

Expected:

```text
HTTP 403
Token has been revoked
```

Check H2:

```sql
SELECT * FROM TOKEN_BLACKLIST;
```

Expected:

```text
At least one row exists
```

## 18. AOP Audit Log Test

Create a successful booking.

Then check H2:

```sql
SELECT * FROM AUDIT_LOGS;
```

Expected:

```text
Row with success = TRUE and message starts with [AUDIT - SUCCESS]
```

Create duplicate booking with same `courtId`, `bookingDate`, and `timeSlotId`.

Expected API result:

```text
HTTP 409
```

Check H2 again:

```sql
SELECT * FROM AUDIT_LOGS;
```

Expected:

```text
Row with success = FALSE and message starts with [AUDIT - FAILED]
```

## 19. Security Matrix Tests

No token accessing customer API:

```http
GET {{baseUrl}}/api/v1/customer/bookings
```

Expected:

```text
HTTP 403 or 401
```

Customer accessing admin API:

```http
GET {{baseUrl}}/api/v1/admin/users
Authorization: Bearer {{customerToken}}
```

Expected:

```text
HTTP 403
```

Customer accessing manager API:

```http
GET {{baseUrl}}/api/v1/manager/courts
Authorization: Bearer {{customerToken}}
```

Expected:

```text
HTTP 403
```

Manager accessing customer API:

```http
GET {{baseUrl}}/api/v1/customer/bookings
Authorization: Bearer {{managerToken}}
```

Expected:

```text
HTTP 403
```

Public API without token:

```http
GET {{baseUrl}}/api/v1/courts
```

Expected:

```text
HTTP 200
```

## 20. Grading Checklist

Basic functions:

| Code | Feature | How to verify |
| --- | --- | --- |
| FR-01 | Login JWT | Section 5 |
| FR-02 | Refresh Token | Section 6 |
| FR-03 | Logout blacklist | Section 17 |
| FR-04 | Register | Section 7 |
| FR-05 | User CRUD/search/page | Section 9 |
| FR-06 | Create booking | Section 12 |
| FR-07 | Booking history | Section 13 |
| FR-08 | Approve/reject booking | Section 14 |
| FR-09 | Upload court images | Section 15 |
| FR-10 | Change/forgot password | Section 16 |

Advanced functions:

| Code | Feature | Current status |
| --- | --- | --- |
| FR-11 | AOP logging | Partial: booking audit exists, full execution-time logging not yet implemented |
| FR-12 | 10 unit tests | Not completed yet |
| FR-13 | Redis TokenBlacklist | Not completed yet, currently DB blacklist |
