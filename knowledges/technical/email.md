# Technical Guide: Email Service (`email`)

Skalfa API provides email sending capabilities built on top of Nodemailer, accessible via the `mail` utility.

---

## 1. Configuration

SMTP credentials and sender configuration are loaded from environment variables in `.env`:
```env
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=your_username
MAIL_PASSWORD=your_password
MAIL_FROM_ADDRESS=noreply@skalfa.com
MAIL_FROM_NAME="Skalfa API"
```

---

## 2. Sending Emails

Emails are sent using the `mail.send` method, which supports HTML content, attachments, and recipient lists.

```typescript
// app/controllers/auth/verification.controller.ts
import { ControllerContext } from 'elysia'
import { mail } from '@utils'

export class VerificationController {
  static async sendCode(c: ControllerContext) {
    await c.validation({
      email: ["required", "email"]
    })

    const code = Math.floor(100000 + Math.random() * 900000).toString();

    await mail.send({
      to:      c.payload.email,
      subject: "Verification Code - Skalfa API",
      html:    `<p>Your verification code is: <strong>${code}</strong></p>`
    })

    c.responseSuccess(null, "Verification email sent successfully")
  }
}
```

---

## 3. Email Templates

For structured emails, place templates in `app/outputs/mails/` as HTML files or string builders, and load them dynamically:

```typescript
// app/outputs/mails/welcome.template.ts
export function welcomeTemplate(name: string): string {
  return `
    <div style="font-family: sans-serif; padding: 20px;">
      <h2>Welcome, ${name}!</h2>
      <p>Thank you for registering on Skalfa API.</p>
    </div>
  `;
}
```
Use it in your controller or service:
```typescript
await mail.send({
  to:      user.email,
  subject: "Welcome to Skalfa!",
  html:    welcomeTemplate(user.name)
})
```
