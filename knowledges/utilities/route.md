# Utility Guide: Route Grouping (`route`)

The `route` utility provides helpers to map endpoints to controllers and register resources.

---

## 1. Route Grouping

Group routes using `app.group` to organize endpoints:

```typescript
// app/routes/index.ts
import { Elysia } from 'elysia'
import { AuthController } from '@controllers'

export const routes = (app: Elysia) => app.group('/api', (route) => {
  route.post('/login', AuthController.login);
  
  return route;
})
```

---

## 2. Resource Routing (`api`)

The `api` helper automatically maps standard RESTful CRUD endpoints to a controller class:

```typescript
import { Elysia } from 'elysia'
import { api } from '@utils'
import { UserController } from '@controllers'

export const routes = (app: Elysia) => app.group('/api', (route) => {
  // Registers all CRUD endpoints for /users
  api(route, '/users', UserController);
  
  return route;
})
```

### Auto-Mapped Endpoints:
| Method | Path | Controller Method | Description |
| :--- | :--- | :--- | :--- |
| `GET` | `/users` | `UserController.index` | List records |
| `POST` | `/users` | `UserController.store` | Create record |
| `GET` | `/users/:id` | `UserController.show` | Show single record |
| `PUT`/`PATCH` | `/users/:id` | `UserController.update` | Update record |
| `DELETE` | `/users/:id` | `UserController.destroy` | Delete record |
