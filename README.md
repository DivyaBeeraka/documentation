# BIGHAT UGC API Documentation  

### Tech Stack

- **Frontend:** React  
- **Backend:** .NET  
- **Database:** MongoDB  

# Frontend Integration Guide

This document defines the **API contract** between the BIGHAT UGC frontend and backend.

It explains:

- Why each API exists  
- What logic it performs  
- How the frontend should interact with it  

This allows frontend developers to confidently build features **without reading backend code**.

> **Project Scope (Important):**  
> In this project, users can **only submit/upload content**.  
> Users **cannot view or fetch their uploaded content**.  
> Therefore, no `GET` APIs for user-uploaded content are used in the frontend.


# Base URL

```
{API_BASE_URL}/api/v1
```

All endpoints listed below are relative to this base URL.

### Example

```
POST {API_BASE_URL}/api/v1/auth/login
```


# Common Rules

### Required Headers

### For JSON Requests

```http
Content-Type: application/json
Authorization: Bearer <JWT_TOKEN> // Required for protected routes
```

### For File Uploads

```http
Content-Type: multipart/form-data
Authorization: Bearer <JWT_TOKEN>
```


# Standard Response Formats

### Standard Error Format

```json
{
  "error": true,
  "message": "Human readable error message",
  "code": "ERROR_CODE"
}
```

### Validation Error Format

```json
{
  "error": true,
  "message": "Validation failed",
  "fields": {
    "email": "Invalid email format",
    "password": "Password must be at least 8 characters"
  }
}
```


# Authentication APIs

These APIs manage user access and identity.


### 1. POST `/auth/login`

### Purpose

Authenticate an existing user and grant access to protected features such as:

- Profile  
- Upload forms  

### Backend Logic

- Validate input  
- Verify hashed password  
- Generate JWT token  
- Return token + user profile  

### Used In Frontend

- `/login`  
- `/` (default route)  

### Request Body

```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

### Validation Rules

| Field    | Rule                             |
|----------|----------------------------------|
| email    | Required, valid email format     |
| password | Required, minimum 4 characters   |

### Success Response (200)

```json
{
  "token": "jwt_access_token",
  "user": {
    "id": "u123",
    "name": "John Doe",
    "email": "user@example.com"
  }
}
```

### Error Responses

| Status Code | Meaning               |
|------------|----------------------|
| 401        | Invalid credentials  |
| 422        | Validation error     |


### 2. POST `/auth/register`

### Purpose

Create a new user account on the platform.

### Backend Logic

- Validate inputs  
- Check email uniqueness  
- Hash password  
- Store user  
- Return created user ID  

### Used In Frontend

- `/new-user`  

### Request Body

```json
{
  "name": "John Doe",
  "email": "user@example.com",
  "password": "password123"
}
```

### Validation Rules

| Field    | Rule                                |
|----------|-------------------------------------|
| name     | Required, minimum 2 characters      |
| email    | Required, unique, valid format      |
| password | Required, minimum 4 characters      |

### Success Response (201)

```json
{
  "message": "User registered successfully",
  "userId": "u123"
}
```

### Error Responses

| Status Code | Meaning               |
|------------|----------------------|
| 409        | Email already exists |
| 422        | Validation error     |


### 3. POST `/auth/forgot-password`

### Purpose

Allow users to recover access to their account.

### Backend Logic

- Validate email  
- Check if user exists  
- Generate reset token  
- Send reset link via email  

### Used In Frontend

- `/forgot-password`  

### Request Body

```json
{
  "email": "user@example.com"
}
```

### Validation Rules

| Field | Rule                         |
|-------|------------------------------|
| email | Required, valid email format |

### Success Response (200)

```json
{
  "message": "Password reset link sent"
}
```

### Error Responses

| Status Code | Meaning          |
|------------|-----------------|
| 404        | Email not found |


# User Profile APIs


### 4. GET `/users/me`

### Purpose

Fetch logged-in user profile information (for displaying user info in UI only).

### Backend Logic

- Validate JWT token  
- Identify user  
- Fetch user data from database  

### Used In Frontend

- `/user-profile`  
- `/home`  

### Success Response (200)

```json
{
  "id": "u123",
  "name": "John Doe",
  "email": "user@example.com",
  "avatar": "https://cdn.example.com/avatar.png"
}
```

### Error Responses

| Status Code | Meaning        |
|------------|---------------|
| 401        | Unauthorized  |
| 404        | User not found |


### 5. PUT `/users/me`

### Purpose

Allow users to update their profile details.

### Backend Logic

- Validate JWT token  
- Validate input  
- Update allowed fields  
- Persist changes  

### Used In Frontend

- `/user-profile`  

### Request Body

```json
{
  "name": "Updated Name",
  "avatar": "https://cdn.example.com/new.png"
}
```

### Validation Rules

| Field  | Rule                          |
|--------|-------------------------------|
| name   | Optional, minimum 2 characters |
| avatar | Optional, valid URL            |

### Success Response (200)

```json
{
  "message": "Profile updated successfully"
}
```


# Content Submission APIs (Users Can Only Upload)


### 6. POST `/ugc/reels`

### Purpose

Allow users to upload short-form video content (UGC).  
Users cannot view or manage uploaded videos in this project.

### Backend Logic

- Validate JWT token  
- Validate file type & size  
- Upload video to storage  
- Save metadata in database  

### Used In Frontend

- `/reel`  

### Headers

```http
Authorization: Bearer <JWT_TOKEN>
Content-Type: multipart/form-data
```

### Form Fields

| Field       | Type | Rule                                   |
|------------|------|------------------------------------------|
| video       | File | Required, mp4, max 100MB                |
| title       | Text | Required, minimum 3 characters          |
| description | Text | Optional, max 500 characters            |

### Success Response (201)

```json
{
  "message": "Reel uploaded successfully",
  "reelId": "r123"
}
```

### Error Responses

| Status Code | Meaning                  |
|------------|--------------------------|
| 413        | File too large           |
| 415        | Unsupported file format  |


### 7. POST `/ugc/product-video-url`

### Purpose

Allow users to submit externally hosted product videos (e.g., YouTube links).  
Users cannot view submitted URLs later.

### Backend Logic

- Validate JWT token  
- Validate URL format  
- Associate URL with product  
- Store reference  

### Used In Frontend

- `/product-url-insert`  

### Request Body

```json
{
  "productId": "p123",
  "videoUrl": "https://youtube.com/watch?v=example"
}
```

### Validation Rules

| Field     | Rule                 |
|-----------|----------------------|
| productId | Required             |
| videoUrl  | Required, valid URL  |

### Success Response (201)

```json
{
  "message": "Video URL submitted successfully"
}
```


# API Behavior Rules (Frontend Expectations)

| Status Code | Required Frontend Behavior     |
|------------|---------------------------------|
| 401        | Redirect to Login               |
| 403        | Show permission error           |
| 5xx        | Show generic error message      |
| Timeout    | Retry once automatically        |


# API Contract Checklist

- All used endpoints are documented  
- Purpose & backend logic defined  
- Validation rules clearly written  
- Standard error format enforced  
- Authentication rules specified  


# Final Notes for Frontend Developers

- Store JWT securely.  
- Attach `Authorization` header for protected routes.  
- Map validation errors to form fields.  
- Use global API error handling.  
- Handle token expiry (logout user if expired).  


This document is the **single source of truth** for frontend-backend communication in the **BIGHAT UGC platform**.
