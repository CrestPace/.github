# Stage 1 Features

## Google Authentication

Sign in with Google is the only authentication method. Using a Gmail address to sign in automatically creates everything in the background -- wallet address, account scaffolding -- but nothing is final until KYC is complete. No email-and-password flow, no phone number verification, no additional sign-in providers. Just Google.

## In-House KYC (Know Your Customer)

KYC is built from scratch in-house. It includes:

- **Liveness check** -- verifies a real person is present during onboarding.
- **Government ID match** -- pattern-matches the face on the official document against the liveness capture. Not used to confirm nationality or any other demographic attribute.

Images captured during verification are ephemeral. Once the face on the government ID has been matched against the liveness capture, the images are discarded. They are never persisted to any storage medium.

## Wallet and Account Creation

- A wallet address is derived from the user's Google ID.
- The wallet and associated accounts are not created until KYC is successfully completed.
- After KYC passes, an event is triggered so contracts can do their work -- setting up the necessary on-chain parameters and metadata.
- Once complete, the user's dashboard becomes accessible with their wallet and accounts visible.
