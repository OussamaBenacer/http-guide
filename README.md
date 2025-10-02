# HTTP Status Codes & Response Structure Guide

## 📋 Standard API Response Structure

### Success Response
```json
{
  "success": true,
  "status": 200,
  "message": "Operation completed successfully",
  "data": {
    // Your actual response data
  },
  "meta": {  // Optional - for pagination, timestamps, etc.
    "page": 1,
    "limit": 20,
    "total": 100,
    "timestamp": "2025-10-02T10:30:00Z"
  }
}
```

### Error Response
```json
{
  "success": false,
  "status": 400,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": [  // Optional - for validation errors
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "timestamp": "2025-10-02T10:30:00Z"  // Optional
}
```

---

## 🔵 1xx – Informational (rare in APIs)

### 100 Continue
Server tells the client to continue sending the request (used in large uploads).

### 101 Switching Protocols
WebSocket handshake (switching from HTTP to WebSocket).

---

## 🟢 2xx – Success

### 200 OK
Standard success response for GET, PUT, PATCH, DELETE, etc.

### 201 Created
When a new resource is created (e.g., user signup, new post).

### 202 Accepted
Request accepted but not completed yet (async tasks, queues).

### 204 No Content
Request succeeded but nothing to return (e.g., DELETE request).

---

## 🟡 3xx – Redirection

### 301 Moved Permanently
Resource moved (e.g., redirect to new API version).

### 302 Found
Temporary redirect.

### 304 Not Modified
Used for caching (client's cached version is still valid).

---

## 🔴 4xx – Client Errors

### 400 Bad Request
Invalid request (malformed JSON, wrong body, invalid query params).

### 401 Unauthorized
Authentication required (user not logged in or token missing/invalid).

### 403 Forbidden
Authenticated but not allowed (e.g., normal user trying admin route).

### 404 Not Found
Resource doesn't exist.

### 405 Method Not Allowed
Wrong HTTP method (e.g., POST on a GET-only endpoint).

### 409 Conflict
Request conflicts with existing resource (e.g., duplicate email, username already taken).

### 422 Unprocessable Entity
Validation failed (valid JSON format but data doesn't meet business rules).

### 429 Too Many Requests
Rate limiting (too many API calls in a short time).

---

## 🔴 5xx – Server Errors

### 500 Internal Server Error
Generic server error (something broke on the server side).

### 502 Bad Gateway
Upstream server failed (e.g., reverse proxy issue, database connection failed).

### 503 Service Unavailable
Server temporarily down (maintenance, overloaded).

### 504 Gateway Timeout
Server took too long to respond.

---

## ✅ Best Practices

### Status Code Usage
- Use **2xx** for success
- Use **4xx** for client mistakes (bad request, auth issues, validation)
- Use **5xx** for server mistakes (crashes, timeouts, service down)

### Response Structure
- Always include `success` and `status` fields
- Wrap successful data in a `data` field
- Wrap errors in an `error` object with `code` and `message`
- Use consistent field naming (camelCase or snake_case)
- Include pagination info in `meta` for list endpoints

### Error Handling
- **400 vs 422**: Use 400 for malformed requests, 422 for validation errors
- **401 vs 403**: Use 401 for "not authenticated", 403 for "not authorized"
- **409**: Use for conflicts like duplicate resources
- Always provide clear, actionable error messages

### Common Patterns

**Pagination (List Endpoints):**
```json
{
  "success": true,
  "status": 200,
  "data": [
    { "id": 1, "name": "Item 1" },
    { "id": 2, "name": "Item 2" }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

**Async/Background Operations (Long-running tasks):**
Async operations are tasks that take too long to complete immediately (e.g., video processing, bulk imports, report generation, sending emails to 10,000 users). Instead of making the client wait, return 202 and let them check status later.

```json
{
  "success": true,
  "status": 202,
  "message": "Video processing started",
  "data": {
    "jobId": "xyz-789",
    "status": "processing",
    "statusUrl": "/api/jobs/xyz-789",
    "estimatedCompletion": "2025-10-02T10:35:00Z"
  }
}
```

Then the client can poll `/api/jobs/xyz-789` to check progress.

**Validation Errors (422):**
```json
{
  "success": false,
  "status": 422,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please fix the following errors",
    "details": [
      {
        "field": "email",
        "message": "Email format is invalid"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters"
      }
    ]
  }
}
```

**Rate Limiting (429):**
When a user exceeds the rate limit, tell them:
1. **How long to wait** (always recommended - improves UX)
2. **When they can retry** (optional but helpful)

```json
{
  "success": false,
  "status": 429,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again in 60 seconds",
    "retryAfter": 60,  // Seconds to wait
    "limit": 100,      // Optional: requests per window
    "remaining": 0,    // Optional: requests left
    "resetAt": "2025-10-02T10:31:00Z"  // Optional: when limit resets
  }
}
```

**Should you always include retry time in 429?**
- **YES, highly recommended** - Users/clients need to know how long to wait
- Without it, clients might retry immediately and waste resources
- Use `retryAfter` (seconds) or `resetAt` (timestamp)
- Standard HTTP header: `Retry-After: 60` (also include in response body)

**Server Errors (500):**
```json
{
  "success": false,
  "status": 500,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Something went wrong. Please try again later",
    "requestId": "req_abc123"  // Optional: for tracking/support
  }
}
```
