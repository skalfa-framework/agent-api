# Utility Guide: Controller Helper (`controller`)

The `controller` utility provides standard response formats, pagination parsers, and validation triggers for Elysia controllers.

---

## 1. Query Parameters Parser (`c.getQuery`)

Automatically parses and normalizes query parameters from the request URL:
*   `page`: Page number (defaults to `1`).
*   `paginate`: Number of items per page (defaults to `10`).
*   `search`: Search query string.
*   `sort`: Array of sorting fields (e.g. `["created_at desc"]`).
*   `filter`: Object containing key-value filters.
*   `expand`: Relations to eagerly load.

---

## 2. Standard Responses

Always use the context helper methods to return HTTP responses. This ensures a consistent JSON structure across the entire API.

### A. `c.responseData(data, totalRow, message)`
Used for returning lists of records, supporting pagination headers.
```typescript
static async index(c: ControllerContext) {
  const { data, total } = await User.query().resolve(c);
  c.responseData(data, total);
}
// Returns: { status: 200, body: { data: [...], total: 100 } }
```

### B. `c.responseSaved(data, message)`
Used after creating or updating a record.
```typescript
static async store(c: ControllerContext) {
  const record = await (new User).pump(c.payload);
  c.responseSaved(record, "User created successfully");
}
// Returns: { status: 201, body: { data: {...}, message: "User created successfully" } }
```

### C. `c.responseSuccess(data, message)`
Used for general successful operations.
```typescript
static async doAction(c: ControllerContext) {
  c.responseSuccess(null, "Action completed");
}
```

### D. `c.responseError(error, section, message)`
Handles errors and formats them into a standard error response.
```typescript
try {
  // logic...
} catch (err) {
  c.responseError(err, "UserController.store", "Failed to save user");
}
```
*Note: If the application is running in development mode, the error stack trace will be included in the response.*
