# Utility Guide: Authentication (`auth`)

The `auth` utility provides token-based session management, JWT operations, and email verification token handling.

---

## 1. Access Token Management

Skalfa API uses database-backed personal access tokens. The token returned to the client is formatted as `id|plainToken`.

### A. Creating an Access Token
Generates a new access token and saves its hash to the database.
```typescript
import { auth } from '@utils'

const { token, record } = await auth.createAccessToken(user.id);
// token is "1|abc123xyz..." (send this to the client)
```

### B. Verifying an Access Token
Validates the plain token against the hash stored in the database. Returns the session and user if valid.
```typescript
import { auth } from '@utils'

const session = await auth.verifyAccessToken(tokenString);
if (!session) {
  throw new Error("Unauthorized");
}

const { user, token } = session;
```

### C. Revoking an Access Token
Deletes the token record from the database.
```typescript
import { auth } from '@utils'

await auth.revokeAccessToken(tokenId);
```

---

## 2. Email Mail Tokens

Used for temporary operations like email verification or password resets. These tokens are time-bound.

### A. Creating a Mail Token
```typescript
import { auth } from '@utils'

const { token } = await auth.createUserMailToken(user.id);
// Send this token to the user's email
```

### B. Verifying a Mail Token
Verifies the token and ensures it has not expired.
```typescript
import { auth } from '@utils'

const isValid = await auth.verifyUserMailToken(user.id, tokenString);
if (!isValid) {
  throw new Error("Invalid or expired token");
}
```
