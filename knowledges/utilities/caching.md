# Utility Guide: Caching (`caching`)

The `caching` utility provides standard cache-aside operations backed by Redis.

---

## 1. Key Generation

Always use structured cache keys to avoid namespace collisions. Use `cache.makeKey` to generate keys:

```typescript
import { cache } from '@utils'

const key = cache.makeKey("users", "profile", user.id);
// Returns: "users:profile:1"
```

---

## 2. Cache Operations

### A. Getting Cached Data
Returns the cached value or `null` if expired/missing. Supports type casting.
```typescript
import { cache } from '@utils'
import { User } from '@models'

const key = cache.makeKey("users", "profile", userId);
const cachedUser = await cache.get<User>(key);

if (cachedUser) {
  return cachedUser;
}
```

### B. Setting Cached Data
Saves a value in the cache with an optional expiration time in seconds (TTL).
```typescript
import { cache } from '@utils'

// Save user profile for 1 hour (3600 seconds)
await cache.set(key, user, 3600);
```

### C. Clearing/Invalidating Cache
Removes specific keys or clears keys matching a prefix pattern:
```typescript
import { cache } from '@utils'

// Clear a specific key
await cache.clear(key);

// Clear all keys under the "users" namespace
await cache.clearPattern("users:*");
```

---

## 3. Cache-Aside Pattern Example

```typescript
import { cache, db } from '@utils'
import { User } from '@models'

export class UserService {
  static async getProfile(userId: number): Promise<User> {
    const cacheKey = cache.makeKey("users", "profile", userId);
    
    // 1. Try to get from cache
    const cached = await cache.get<User>(cacheKey);
    if (cached) return cached;

    // 2. If miss, fetch from database
    const user = await User.query().findOrNotFound(userId);

    // 3. Save to cache for future requests
    await cache.set(cacheKey, user, 600); // 10 minutes

    return user;
  }
}
```
 Josephson.
