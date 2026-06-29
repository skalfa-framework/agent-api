# Utility Guide: Mail Dispatcher (`mail`)

The `mail` utility provides a wrapper around Nodemailer to dispatch HTML/text emails.

---

## 1. Sending Simple Emails

Use `mail.send` to send an email.

```typescript
import { mail } from '@utils'

await mail.send({
  to:      "user@example.com",
  subject: "Account Activated",
  html:    "<h3>Your account is now active!</h3>",
  text:    "Your account is now active!" // plain text fallback
});
```

---

## 2. Attachments

You can attach local files or buffers:

```typescript
import { mail } from '@utils'
import path from 'path'

await mail.send({
  to:      "client@example.com",
  subject: "Monthly Report",
  html:    "<p>Please find your monthly report attached.</p>",
  attachments: [
    {
      filename: "report.pdf",
      path:     path.resolve("storage/private/reports/june.pdf")
    }
  ]
});
```

---

## 3. Send to Multiple Recipients

```typescript
import { mail } from '@utils'

await mail.send({
  to:      ["user1@example.com", "user2@example.com"],
  subject: "System Update",
  html:    "<p>The system will be down for maintenance tonight.</p>"
});
```
