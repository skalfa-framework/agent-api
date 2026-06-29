# Technical Guide: Middleware

Middleware in Skalfa API is used to intercept HTTP requests before they reach the controller handler. It handles cross-cutting concerns like authentication, logging, CORS, and request parsing.

---

## 1. Global Middleware

Global middleware is executed for every incoming request. It is registered directly in `app/app.ts`:

```typescript
// app/app.ts
import { Elysia } from 'elysia'
import { cors } from '@elysia/cors'
import { logger } from '@utils'

const app = new Elysia()
  .use(cors()) // Global CORS middleware
  .use(logger) // Global logger middleware
```

---

## 2. Local/Route-Specific Middleware

Local middleware is applied to specific route groups or individual endpoints. The most common local middleware is `Middleware.Auth`.

```typescript
// app/routes/index.ts
import { Elysia } from 'elysia'
import { Middleware } from '@utils'
import { UserController } from '@controllers'

export const routes = (app: Elysia) => app.group('/api', (route) => {
  // Apply Auth middleware to all routes registered under this group
  route.use(Middleware.Auth)

  route.get('/users', UserController.index)
  
  return route
})
```

---

## 3. Creating Custom Middleware

Custom middleware is defined using Elysia's `.derive` or `.onBeforeHandle` hooks.

```typescript
// app/middleware/ip-filter.middleware.ts
import { Elysia } from 'elysia'

export const ipFilter = (allowedIps: string[]) => (app: Elysia) => 
  app.onBeforeHandle(({ request, set }) => {
    const clientIp = request.headers.get("x-forwarded-for") || "unknown";
    
    if (!allowedIps.includes(clientIp)) {
      set.status = 403;
      return { error: "Forbidden: IP not allowed" };
    }
  })
```
Register it in your routes or globally:
```typescript
route.use(ipFilter(["127.0.0.1"]))
```
---

## 4. Execution Order

Middleware is executed in the order of its registration. Always register logging and CORS middleware first, followed by body parsers, authentication, and finally specific route guards.
