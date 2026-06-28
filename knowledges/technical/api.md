# Panduan Teknis: Layanan API (API Service)

Layanan API di Skalfa API memisahkan tanggung jawab antara routing (pemetaan endpoint), controller (pengolah request/response), dan model (representasi data).

---

## 1. Rute (Route)

Route hanya bertugas mendefinisikan HTTP method dan path API. Di Skalfa API, route dikelompokkan menggunakan `group` agar namespace jelas dan mudah dikembangkan.

```typescript
// src/routes/index.ts
import { Elysia } from 'elysia'
import { api, Middleware } from '@utils'
import { AuthController, UserController } from '@controllers'

export const routes = (app: Elysia) => app.group('/api', (route) => {
  // Rute publik
  route.post('/login', AuthController.login)

  // Menggunakan middleware otentikasi untuk rute di bawahnya
  route.use(Middleware.Auth)

  // Rute terproteksi
  route.get('/me', AuthController.me)

  // Rute CRUD otomatis untuk User
  api(route, '/users', UserController)

  return route
})
```

---

## 2. Pengendali (Controller)

Controller menjadi titik masuk utama pengolahan request. Di sinilah validasi, pengecekan hak akses (permission), pemanggilan model/service, dan penentuan response dilakukan.

```typescript
// src/controllers/user/user.controller.ts
import type { ControllerContext } from 'elysia'
import { db, permission } from '@utils'
import { User } from '@models'

// Daftarkan permission modul ini
export const UserPermission = permission.register({
  '100': {
    name: 'User Management',
    accesses: {
      '01': 'View',
      '02': 'Create',
      '03': 'Update',
      '04': 'Delete',
    }
  },
})

export class UserController {
  static async index(c: ControllerContext) {
    // Pengecekan permission
    UserPermission.have('01').guard(c)

    // Ambil data menggunakan ORM dan resolve query parameter otomatis
    const users = await User.query().resolve(c)

    c.responseData(users.data, users.total)
  }

  static async store(c: ControllerContext) {
    UserPermission.have('02').guard(c)

    // Validasi payload request
    c.validation({
      name:  'required|string|max:100',
      email: 'required|email|unique:users,email'
    })

    const trx = await db.transaction()

    try {
      // Menyimpan data beserta relasinya secara otomatis menggunakan .pump()
      let record = await new User().pump(c.body, { trx })
      await trx.commit()
      c.responseSaved(record)
    } catch (err) {
      await trx.rollback()
      c.responseError(err as Error, 'Create User')
    }
  }
}
```

---

## 3. Model

Model merepresentasikan tabel atau entity data, mendefinisikan kolom, relasi, dan konfigurasi data.

```typescript
// src/models/iam/user.model.ts
import { Model, Field, SoftDelete } from '@utils'

export class User extends Model {
  static table = "users";

  @Field(['string', 'fillable', 'selectable', 'searchable'])
  name!: string

  @Field(['string', 'fillable', 'selectable', 'searchable'])
  email!: string

  @SoftDelete()
  deleted_at!: Date
}
```
*Catatan untuk Agen: Jaga agar route tidak berisi logika bisnis. Semua proses logika didelegasikan ke controller atau service.*
