# Utility Guide: Permissions & RBAC (`permission`)

The `permission` utility provides feature-based and access-key-based authorization.

---

## 1. Registering Permissions

Permissions are registered in controllers or config files. Each feature has a unique 3-digit key, and accesses are defined as 2-digit keys.

```typescript
import { permission } from '@utils'

export const UserPermission = permission.register({
  "100": {
    name: "User Management",
    accesses: {
      "01": "View",
      "02": "Create",
      "03": "Update",
      "04": "Delete"
    }
  }
})
```

---

## 2. Guarding Endpoints

Use the `.guard(c)` method at the beginning of controller actions. If the user lacks the required permission, a `403 Forbidden` response is returned immediately.

### A. Basic Guard
```typescript
static async index(c: ControllerContext) {
  UserPermission.have('01').guard(c);
  
  // Logic here only runs if user has "100.01" permission
}
```
*Note: If no feature key is specified, it defaults to the feature key it was registered under (e.g., '01' is normalized to '100.01').*

### B. OR Permissions
Allows access if the user has at least one of the specified permissions:
```typescript
UserPermission
  .have('01')
  .orHave('02')
  .guard(c);
```

---

## 3. Retrieving Available Permissions

Use these helpers to list all registered features and accesses (e.g., to display in a role management UI):

```typescript
import { permission } from '@utils'

const features = permission.getFeatures();
const accesses = permission.getAccesses();
```
