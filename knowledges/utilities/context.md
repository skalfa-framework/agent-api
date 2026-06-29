# Utility Guide: Controller Context (`context`)

The `context` utility defines the extended properties and methods available on Elysia's HTTP context, typed as `ControllerContext`.

---

## 1. Type Definition

The `ControllerContext` is declared as a module extension of Elysia's `Context`:

```typescript
import { Context } from "elysia";

declare module "elysia" {
  interface ControllerContext extends Context {
    getQuery: {
      paginate:          number;
      page:              number;
      sort:              string[];
      filter:            Record<string, string>;
      search:            string;
      searchable:        string[];
      selectable:        string[];
      selectableOption:  string[];
      expand:            string[];
    };
    
    payload:                 Record<string, any>;
    user?:                   any;
    permissions?:            string[];
    
    validation:              <T extends object>(rules: Record<string, any>) => Promise<void>;
    responseData:            (data: any[], totalRow?: number, message?: string) => any;
    responseSaved:           (data: any, message?: string) => any;
    responseSuccess:         (data: any, message?: string) => any;
    responseError:           (error: any, section?: string, message?: string) => any;
    responseForbidden:       (message?: string) => any;
    
    uploadFile:              (file: File, folder?: string, options?: any) => Promise<string>;
    deleteFile:              (filePath: string) => Promise<boolean>;
  }
}
```

---

## 2. Usage in Controllers

Every controller method receives this context as the first argument:

```typescript
import { ControllerContext } from "elysia";

export class MyController {
  static async doSomething(c: ControllerContext) {
    const userId = c.user?.id;
    const body = c.payload;
    
    c.responseSuccess({ userId, body });
  }
}
```
