# Account lockout and unlock flow

How a user gets locked after 5 wrong passwords, how they get unlocked, and how sessions (OAuth tokens) behave around lock, logout, and password reset. There are **two parallel systems**: the monolith (`digima-backend-app`, old flow) and the auth-api (`digima-backend-auth-api`, new flow).

## Old flow — monolith (`digima-backend-app`)

### Lock (5 failed logins)

1. `app/Services/Lockout/Bouncer.php` counts failed attempts in cache, key = `username|ip`.
   - Config: `config/auth.php` → `auth.lockout.max_failed_attempts = 5`, remembered for 300s.
   - Invoked from `app/Jobs/Oauth/IssueAccessTokenJob.php` (login/token endpoint).
2. On the 5th failure it fires `LockoutEvent`.
3. `app/Listeners/Oauth/LockAuthenticatableListener.php` handles it:
   - Sends `LockoutNotification` email.
   - Dispatches `LockUserJob` / `LockAdministratorJob` → `LockableTrait::lock()` sets `locked_at = now()` (and syncs `is_locked` on `UserReference`).
   - **Revokes ALL active access tokens** via `revokeInitiatorValidAccessTokens()` → every non-expired, non-revoked token is revoked (refresh tokens cascade).
4. Login rejects locked users at `app/Services/AuthManager.php:218` (`isLocked()` → `LockedOutUserException`).

> ⚠️ Lockout kicks out **every device**, including ones already logged in. If device B fails 5 attempts, device A's session dies immediately (token revoked → next request 401, refresh also revoked).

### Unlock — password reset only

There is **no admin unlock endpoint and no unlock button anywhere**. The only way out is self-service password reset:

1. Guest routes: `routes/user/v2/guest.php` and `routes/admin/v2/guest.php` — `POST /password-resets` (request email), `PUT /password-resets` (submit new password).
2. `PasswordResetsController` → `app/Jobs/PasswordReset/ResetPasswordJob.php` updates the password and fires `PasswordReset\CompletedEvent`.
3. Only listener: `UnlockAuthenticatableListener` → `UnlockUserJob` / `UnlockAdministratorJob` → `LockableTrait::unlock()` nulls `locked_at`, sets `is_locked = false` on `UserReference`.
4. The job also syncs the new password to auth-api (`updateOrMigrateUserToAuthApi()` → `UserMigrateRequestedEvent`).

### adminweb (`digima-admin-web-app`)

Not involved in unlock. Only shows `is_locked` as a **read-only column** in the user references list (`src/domains/users/views/user-references/Index.jsx`).

## New flow — auth-api (`digima-backend-auth-api`)

- Own lockout in `domain/services/token/password_grant.go`: max **5 failed attempts**, counter in **Redis** per username (`infra/database/repositories/redis/login_attempt/repository.go`).
- Lock lasts **30 minutes** and **expires automatically** (Redis TTL) — no unlock action exists or is needed.
- Successful login calls `LoginAttemptRepository.Reset()` to clear the counter.
- The lock only blocks token issuance — **existing sessions survive** (no token revocation on lock).
- No password-reset flow lives in auth-api; the monolith owns it and pushes the new password via the `UserService.UpdatePassword` gRPC.

## Session (token) behavior summary

| Action | Monolith | auth-api |
|---|---|---|
| Lockout (5 fails) | Revokes **all** tokens — every device logged out | Blocks new logins only; existing sessions survive |
| Logout | Revokes **current token only** (+ its refresh tokens) — other devices stay logged in. `User/v2/Oauth/AccessTokensController` → `RevokeAccessTokenJob($currentToken)` | — |
| Password reset / change | **Does NOT revoke any tokens** — other devices stay logged in | `UpdatePassword` does NOT touch tokens either (only user `Delete` wipes them) |

## Known gaps

- **Password reset does not invalidate existing sessions** (both systems). If someone stole the password and is logged in, resetting doesn't evict them. OWASP recommends invalidating all sessions on password change/reset. Possible fix: add a token-revocation listener on `PasswordReset\CompletedEvent` (reuse the lockout listener's logic) + revoke tokens in auth-api's `UpdatePassword`.
- Lock/unlock are asymmetric on purpose: lockout revokes tokens, unlock just clears `locked_at` (relies on tokens having been revoked at lock time).
- There is no "revoke all tokens for a user" admin flow; admin can only revoke one token at a time (`Admin/v2/Oauth/AccessTokensController`).
