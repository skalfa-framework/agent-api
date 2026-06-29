# Technical Guide: Queue Workers (`queue`)

Skalfa API uses Redis to handle asynchronous background tasks (queues) via the `queue` utility. This prevents long-running operations from blocking the main HTTP thread.

---

## 1. Adding Jobs to the Queue

Jobs are dispatched to the queue using the `queue.add` method. You must provide a job name and a payload.

```typescript
// app/controllers/auth/register.controller.ts
import { ControllerContext } from 'elysia'
import { queue } from '@utils'

export class RegisterController {
  static async register(c: ControllerContext) {
    await c.validation({
      email:    ["required", "email"],
      password: ["required"]
    })

    // Create user logic...
    const user = { id: 1, email: c.payload.email };

    // Dispatch welcome email to queue
    await queue.add("send-welcome-email", {
      userId: user.id,
      email:  user.email
    })

    c.responseSuccess(user, "Registration successful")
  }
}
```

---

## 2. Defining Queue Workers

Workers are defined in `app/jobs/queues/` and process jobs asynchronously.

```typescript
// app/jobs/queues/email.worker.ts
import { queue, mail } from '@utils'
import { welcomeTemplate } from '@templates'

queue.worker("send-welcome-email", async (job) => {
  const { email, userId } = job.data;
  
  console.log(`✉️ Sending welcome email to user ${userId}...`);
  
  await mail.send({
    to:      email,
    subject: "Welcome to Skalfa!",
    html:    welcomeTemplate("User")
  });
  
  console.log(`✅ Welcome email sent to ${email}`);
})
```

---

## 3. Running Workers

In production, queue workers must be run in a separate process:
```bash
bun run app/jobs/queues/workers.ts
```
In development, workers are automatically started alongside the main server when running `npm run dev`.
