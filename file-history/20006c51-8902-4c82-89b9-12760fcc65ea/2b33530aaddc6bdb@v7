# Invite-Only Signup with Resend Email

## Summary

1. Add Resend email service for sending invite links
2. Enable invite-only mode (`REQUIRE_INVITE=true`)
3. Bootstrap first admin via `ADMIN_EMAIL` env var
4. Build admin UI page to manage invites

---

## Phase 1: Resend Setup

### 1.1 Get Resend API Key
1. Sign up at https://resend.com
2. Add and verify your domain (or use their test domain for dev)
3. Create API key → copy it

### 1.2 Install Go SDK
```bash
cd backend
go get github.com/resend/resend-go/v2
```

### 1.3 Create Email Service

**New file: `backend/internal/email/email.go`**

```go
package email

import (
    "fmt"
    "os"
    "github.com/resend/resend-go/v2"
)

var client *resend.Client

func Init() {
    apiKey := os.Getenv("RESEND_API_KEY")
    if apiKey != "" {
        client = resend.NewClient(apiKey)
    }
}

func SendInviteEmail(toEmail, inviteURL string) error {
    if client == nil {
        // No email configured - log to console instead
        fmt.Printf("\n[EMAIL] Invite for %s: %s\n\n", toEmail, inviteURL)
        return nil
    }

    fromEmail := os.Getenv("EMAIL_FROM")
    if fromEmail == "" {
        fromEmail = "noreply@yourdomain.com"
    }

    _, err := client.Emails.Send(&resend.SendEmailRequest{
        From:    fromEmail,
        To:      []string{toEmail},
        Subject: "You're invited to Astrotask",
        Html: fmt.Sprintf(`
            <h2>You've been invited!</h2>
            <p>Click the link below to create your account:</p>
            <p><a href="%s">Sign up for Astrotask</a></p>
            <p>This link expires in 7 days.</p>
        `, inviteURL),
    })
    return err
}
```

---

## Phase 2: Wire Up Email Sending

### 2.1 Update Invitation Handler

**Modify: `backend/internal/handlers/invitations.go`**

In the `Create` function, after generating the invite URL:

```go
// Send invite email
if err := email.SendInviteEmail(email, inviteURL); err != nil {
    log.Printf("Failed to send invite email: %v", err)
    // Don't fail the request - still return the URL
}
```

### 2.2 Initialize Email in main.go

```go
import "simple_task_tracker/backend/internal/email"

func main() {
    // ... after godotenv.Load()
    email.Init()
    // ...
}
```

---

## Phase 3: Admin Bootstrap

### 3.1 Create Bootstrap Function

**New file: `backend/internal/database/bootstrap.go`**

```go
package database

func BootstrapAdminInvite(db *gorm.DB) {
    adminEmail := os.Getenv("ADMIN_EMAIL")
    if adminEmail == "" {
        return
    }

    // Only bootstrap if no users exist
    var count int64
    db.Model(&models.User{}).Count(&count)
    if count > 0 {
        return
    }

    // Create invite (InvitedByID=0 for system-generated)
    token, invite, err := auth.CreateInvitationToken(db, adminEmail, 0)
    if err != nil {
        log.Printf("Failed to create admin invite: %v", err)
        return
    }

    frontendURL := os.Getenv("FRONTEND_URL")
    if frontendURL == "" {
        frontendURL = "http://localhost:3000"
    }
    inviteURL := frontendURL + "/signup?invite=" + token

    // Send email (or log if no email service)
    email.SendInviteEmail(adminEmail, inviteURL)

    log.Printf("═══════════════════════════════════════════")
    log.Printf("ADMIN INVITE CREATED FOR: %s", invite.Email)
    log.Printf("═══════════════════════════════════════════")
}
```

### 3.2 Call in main.go

```go
// After database.InitDB()
database.BootstrapAdminInvite(db)
```

---

## Phase 4: Environment Variables

### docker-compose.yml
```yaml
environment:
  REQUIRE_INVITE: "true"
  ADMIN_EMAIL: jyn@afterdark.io
  RESEND_API_KEY: re_xxxxx  # Get from resend.com
  EMAIL_FROM: noreply@yourdomain.com
  FRONTEND_URL: http://localhost:3000
```

### For AWS
Same env vars in your ECS task definition or EC2 environment.

---

## Phase 5: Admin UI (Optional - can do later)

Create `/admin/invites` page in frontend to:
- List pending invites
- Create new invites
- Revoke invites

This uses the existing API endpoints:
- `GET /api/v1/admin/invitations` - List
- `POST /api/v1/admin/invitations` - Create
- `DELETE /api/v1/admin/invitations/:id` - Revoke

---

## Files Summary

| File | Change |
|------|--------|
| `backend/go.mod` | Add resend-go dependency |
| `backend/internal/email/email.go` | New - email service |
| `backend/internal/database/bootstrap.go` | New - admin bootstrap |
| `backend/internal/handlers/invitations.go` | Send email on invite create |
| `backend/cmd/server/main.go` | Init email, call bootstrap |
| `docker-compose.yml` | Add env vars |

---

## Flow

1. **First deploy (fresh database):**
   - Server starts, sees ADMIN_EMAIL, no users
   - Creates invite, sends email to you
   - You click link, sign up, become admin

2. **Inviting others:**
   - Call `POST /admin/invitations` with email
   - Resend sends the invite email automatically
   - User clicks link, signs up

3. **Public signup blocked:**
   - `REQUIRE_INVITE=true` rejects signups without invite token
