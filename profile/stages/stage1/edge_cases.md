# Stage 1 Edge Cases

This document tracks everything that could go wrong during Stage 1. The goal is to handle every path with proper error messages, solid error handling, and retry logic where appropriate -- so the auth layer is production-grade from the start.

## Authentication Edge Cases

- **Google sign-in popup blocked.** The user's browser blocks the Google sign-in popup. The UI must detect this and show a message telling the user to allow popups for the site.
- **Google token expired mid-session.** If the Google ID token expires while the user is in the KYC flow, the session should refresh transparently or, if that fails, prompt the user to re-authenticate without losing KYC progress.
- **Google account has no associated email.** Edge case: some Google accounts (e.g., legacy or enterprise-managed) may not expose an email claim. The system must handle a missing email gracefully -- either by requiring it or by falling back to the Google ID alone.
- **Duplicate Google sign-in.** If a user signs in with a Google account that already has a completed KYC and account, they should be taken directly to the dashboard, not forced through KYC again.
- **Multiple sign-in attempts.** Rapid repeated sign-in attempts from the same IP or device should not create multiple pending KYC states or trigger rate-limiting that locks out legitimate users.

## KYC Edge Cases

- **Camera permission denied.** The liveness check requires camera access. If the user denies the permission, show a clear message explaining why it is needed and how to enable it in their browser settings.
- **Poor lighting or blurry capture.** The liveness check or ID photo may fail because of image quality. Provide real-time feedback (e.g., "Move to a brighter area" or "Hold the camera steady") rather than a generic failure message.
- **Unsupported ID format.** If the user uploads a government ID that the processing pipeline cannot parse (wrong country, expired, damaged barcode), the system should surface which specific issue was detected and whether a different ID type is acceptable.
- **Face mismatch.** If the face on the government ID does not match the liveness capture, the user should be told explicitly that the faces did not match and given the option to retry. Multiple retries should not silently lock the user out without explanation.
- **Image upload failures.** If the ID upload or liveness capture fails due to network issues, the system should retry automatically at least once before surfacing an error.
- **KYC timeout.** If the user takes too long in the KYC flow (session timeout or inactivity), they should be returned to the sign-in step with a clear message. No partial KYC state should persist beyond the session.

## Wallet and Account Creation Edge Cases

- **Contract event failure.** If the event triggered after KYC to create on-chain state fails (network error, gas issues, contract revert), the system must retry with backoff and, if it ultimately fails, create the user's account in a pending state with a clear error message rather than silently leaving the user in limbo.
- **Wallet address collision.** Although the wallet is derived from the Google ID, the derivation algorithm should be tested against known collision scenarios. If a collision is ever detected, the system must fail closed -- do not overwrite an existing wallet.
- **Partial state after creation failure.** If account creation partially succeeds (e.g., wallet created but accounts not provisioned), the system must detect the incomplete state on the next sign-in and either complete the setup or surface a clear error with a path to resolution.

## Security Edge Cases

- **IDOR (Insecure Direct Object Reference).** This is the most critical class of vulnerability. Every endpoint, every query, and every state transition must verify that the authenticated user owns the resource being accessed. Never trust a client-supplied ID alone. Row-level ownership checks must be present on every data access path. This is not a Stage 1-only concern -- it applies across all stages -- but the schema and authorization patterns established in Stage 1 set the foundation for everything that follows.
- **Session hijacking.** Google ID tokens should be validated on every request, not just at sign-in. Token expiry and signature verification must happen server-side.
- **CSRF on the KYC submission endpoint.** The KYC submission endpoint must include CSRF protection so an attacker cannot trick a signed-in user into submitting a fraudulent KYC on their behalf.
- **Rate limiting.** The sign-in and KYC endpoints must be rate-limited to prevent abuse (credential enumeration, denial-of-wallet creation, spam).
