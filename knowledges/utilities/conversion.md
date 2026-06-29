# Utility Guide: Case Conversion (`conversion`)

The `conversion` utility provides functions to transform string casing (snake_case, camelCase, PascalCase, etc.) and singular/plural forms.

---

## 1. Case Conversions

### A. Snake Case (`strSnake`)
Converts camelCase, PascalCase, or space-separated strings to snake_case.
```typescript
import { conversion } from '@utils'

conversion.strSnake("MyVariableName"); 
// => "my_variable_name"
```

### B. Camel Case (`strCamel`)
Converts snake_case, kebab-case, or space-separated strings to camelCase.
```typescript
import { conversion } from '@utils'

conversion.strCamel("my_variable_name");
// => "myVariableName"
```

### C. Pascal Case (`strPascal`)
Converts strings to PascalCase.
```typescript
import { conversion } from '@utils'

conversion.strPascal("my variable_name");
// => "MyVariableName"
```

### D. Slug Case (`strSlug`)
Converts strings to url-friendly slugs, removing special characters.
```typescript
import { conversion } from '@utils'

conversion.strSlug("Hello World!");
// => "hello-world"
```

---

## 2. Pluralization & Singularization

### A. Plural Form (`strPlural`)
Converts an English word to its plural form.
```typescript
import { conversion } from '@utils'

conversion.strPlural("product");
// => "products"
```

### B. Singular Form (`strSingular`)
Converts an English word to its singular form.
```typescript
import { conversion } from '@utils'

conversion.strSingular("products");
// => "product"
```
