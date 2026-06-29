# Technical Guide: API Service (`api`)

The API service in Skalfa API separates responsibilities between routing (endpoint mapping), controllers (request/response handling), and models (data representation).

---

## 1. Routes

Routes are only responsible for defining the HTTP method and the API path. In Skalfa API, routes are grouped using `.group` to keep the namespace clear and maintainable.

```typescript
// app/routes/index.ts
import { Elysia } from 'elysia'
import { api, Middleware } from '@utils'
import { AuthController, UserController } from '@controllers'

export const routes = (app: Elysia) => app.group('/api', (route) => {
  // Public route
  route.post('/login', AuthController.login)

  // Apply authentication middleware for routes below
  route.use(Middleware.Auth)

  // Protected route
  route.get('/me', AuthController.me)

  // Automatic CRUD routes for User
  api(route, '/users', UserController)

  return route
})
```

---

## 2. Controllers

Controllers receive request data, validate inputs, check permissions, and delegate the business logic to either inline CRUD or service objects before returning a response.

### Controller Requirements:
1.  Must inherit from `BaseController` (if using blueprint/magic methods) or be a plain class.
2.  Methods must be `static` and receive `c: ControllerContext`.
3.  Must apply permission guards at the beginning of the method.
4.  Must validate inputs before processing.

```typescript
// app/controllers/iam/user.controller.ts
import { ControllerContext } from 'elysia'
import { permission } from '@utils'
import { User } from '@models'

const p = permission.register({
  "100": {
    name: "User Management",
    accesses: {
      "01": "View",
      "02": "Create"
    }
  }
})

export class UserController {
  // GET /api/users
  static async index(c: ControllerContext) {
    p.have("100.01").guard(c)

    // Eager load relations and resolve query automatically
    const { data, total } = await User.query().resolve(c)
    c.responseData(data, total)
  }

  // POST /api/users
  static async store(c: ControllerContext) {
    p.have("100.02").guard(c)

    await c.validation<User>({
      name:  ["required", "max:100"],
      email: ["required", "email"]
    })

    const record = await (new User).pump(c.payload)
    c.responseSaved(record)
  }
}
```

---

## 3. Controller Context (`c`)

The `ControllerContext` extends Elysia's default context with several helper properties and methods:

*   `c.user`: Contains the authenticated user object (populated by `Middleware.Auth`).
*   `c.permissions`: Array of permission keys owned by the user (e.g., `["100.01", "100.02"]`).
*   `c.payload`: The validated and filtered request body.
*   `c.getQuery`: Parsed query parameters (pagination, filters, search, sorting).
