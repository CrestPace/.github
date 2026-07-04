# Stage 1 User Flow

## Expected Path

1. **Landing page.** The user arrives at the CrestPace site and sees the landing page.

2. **Sign in with Google.** The user clicks "Sign in with Google" and authenticates using their Gmail address. This is the only path forward -- no other sign-in methods are offered.

3. **KYC check.** After Google authentication, the user is prompted to complete KYC:
   - Liveness check -- a real-time capture verifies a real person is present.
   - Government ID upload -- the face on the ID is matched against the liveness capture.
   - Images are discarded immediately after the match. Nothing is stored.

4. **Account and wallet creation.** Once KYC passes successfully, the system:
   - Derives a wallet address from the user's Google ID.
   - Triggers an event so contracts create the necessary on-chain state and metadata.
   - Provisions the user's accounts.

5. **Dashboard.** The user is taken to their dashboard where their wallet and accounts are visible and fully accessible.

## Alternate Paths

- **KYC failure.** If the liveness check or government ID match fails, the user is shown a clear error explaining what went wrong and given the option to retry. No account or wallet is created.
- **Abandoned KYC.** If the user signs in with Google but never completes KYC, nothing is created. The user can return later, sign in again, and resume the KYC flow from the beginning.
