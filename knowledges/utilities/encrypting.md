# Utility Guide: Encryption & Hashing (`encrypting`)

The `encrypting` utility provides cryptographic functions, including two-way encryption (AES, TripleDES) and one-way hashing (SHA256, SHA512, MD5).

---

## 1. Two-Way Encryption (AES / TripleDES)

Encrypts data with a secret key so it can be decrypted later.

### A. AES Encryption & Decryption
```typescript
import { encrypt } from '@utils'

const secretKey = "my-super-secret-key";
const sensitiveData = "secret password 123";

// Encrypt
const encrypted = encrypt.aesEncrypt(sensitiveData, secretKey);

// Decrypt
const decrypted = encrypt.aesDecrypt(encrypted, secretKey);
// => "secret password 123"
```

### B. TripleDES Encryption & Decryption
```typescript
import { encrypt } from '@utils'

const encrypted = encrypt.tripleDesEncrypt("hello", secretKey);
const decrypted = encrypt.tripleDesDecrypt(encrypted, secretKey);
```

---

## 2. One-Way Hashing (SHA / MD5)

Generates a secure hash. Hashing is one-way and cannot be reversed.

### A. SHA-256 Hashing
```typescript
import { encrypt } from '@utils'

const hash = encrypt.sha256("password123");
// Returns: "ef92b778bafe771e8924f72d40537d34..."
```

### B. SHA-512 Hashing
```typescript
import { encrypt } from '@utils'

const hash = encrypt.sha512("password123");
```

### C. MD5 Hashing
```typescript
import { encrypt } from '@utils'

const hash = encrypt.md5("password123");
```

---

## 3. Bcrypt Password Hashing

For user passwords, **always** use bcrypt (supplied via standard packages) rather than simple hashing:
```typescript
import bcrypt from 'bcrypt'

// Hash password before saving
const hash = await bcrypt.hash(plainPassword, 10);

// Verify password during login
const isMatch = await bcrypt.compare(plainPassword, hash);
```
