# Technical Guide: File Storage (`storage`)

Skalfa API provides file storage management supporting both public (accessible directly via URL) and private (accessible only with specific permissions) files.

---

## 1. Disks Configuration

Files are stored in the `storage/` directory at the project root:
*   **Public Disk** (`storage/public/`): Files are accessible directly via `/storage/public/...` URLs.
*   **Private Disk** (`storage/private/`): Files are protected. Accessing them requires matching permission records in the `storages` and `storage_permissions` tables.

---

## 2. Uploading Files (`uploadFile`)

The `uploadFile` helper is available on the `ControllerContext` `c` (or can be imported directly). It handles writing files to disk and optionally registering metadata in the database.

```typescript
// app/controllers/gallery/photo.controller.ts
import { ControllerContext } from 'elysia'

export class PhotoController {
  static async upload(c: ControllerContext) {
    await c.validation({
      file: ["required"]
    })

    const file = c.body.file as File;

    // 1. Upload to public disk under 'photos' folder
    const publicPath = await c.uploadFile(file, "photos")
    // Returns: "/photos/abc123xyz.jpg" (URL-ready)

    // 2. Upload to private disk with restricted access
    const privatePath = await c.uploadFile(file, "invoices", {
      disk:        "private",
      owner_id:    c.user.id,
      permissions: [
        { role_id: 1 } // Only role ID 1 (Admin) can access
      ]
    })

    c.responseSuccess({ publicPath, privatePath })
  }
}
```

---

## 3. Deleting Files (`deleteFile`)

To delete a file from disk and remove its database record, use the `deleteFile` helper:

```typescript
// app/controllers/gallery/photo.controller.ts
import { ControllerContext } from 'elysia'

export class PhotoController {
  static async destroy(c: ControllerContext) {
    const filePath = "storage/public/photos/abc123xyz.jpg";
    
    const deleted = await c.deleteFile(filePath);
    if (!deleted) {
      return c.responseError("File not found or failed to delete");
    }

    c.responseSuccess(null, "File deleted successfully")
  }
}
```
*Note: `deleteFile` expects the path from the project root (e.g., `storage/public/photos/filename.jpg`).*
