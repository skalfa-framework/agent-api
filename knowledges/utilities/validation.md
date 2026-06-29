# Utility Guide: Validation (`validation`)

The `validation` utility provides request body and query validation.

---

## 1. Performing Validation

Validate the request payload inside controller methods using `c.validation`:

```typescript
// app/controllers/iam/user.controller.ts
import { ControllerContext } from 'elysia'
import { User } from '@models'

export class UserController {
  static async store(c: ControllerContext) {
    // Throws a 422 Unprocessable Entity error if validation fails
    await c.validation<User>({
      name:     ["required", "max:100"],
      email:    ["required", "email"],
      password: ["required", "min:8"]
    });

    // c.payload contains the validated and filtered data
    const record = await (new User).pump(c.payload);
    c.responseSaved(record);
  }
}
```

---

## 2. Supported Validation Rules

*   `required`: Field cannot be null, undefined, or empty.
*   `email`: Field must be a valid email address.
*   `min:N`: Minimum length (for strings) or minimum value (for numbers) is N.
*   `max:N`: Maximum length (for strings) or maximum value (for numbers) is N.
*   `numeric`: Field must be a number or numeric string.
*   `boolean`: Field must be a boolean or boolean-like value (`true`, `false`, `0`, `1`).
*   `date`: Field must be a valid date string.
*   `json`: Field must be a valid JSON string or object.
*   `url`: Field must be a valid URL string.
*   `uuid`: Field must be a valid UUID.
