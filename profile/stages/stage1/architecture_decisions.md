# Stage 1 Architecture Decisions

## Single Authentication Method: Google Only

Signing in with Google is the only authentication method, at least initially. This is a deliberate constraint.

### Why Not Email and Password

Handling email-and-password auth means building password reset flows, hashing and salting logic, breach detection, and all the associated support burden. That is not where the engineering effort should go in Stage 1.

### Why Not Phone Number Verification

Phone number verification requires an SMS provider, which introduces an external third-party dependency and ongoing cost. This goes against the architecture's principle of minimizing reliance on external services unless absolutely required.

### Why Not Privy or Another Auth Aggregator

Services like Privy could automate multi-method authentication, but they are external dependencies. Relying on a third-party auth aggregator contradicts the core architectural decision to build as much as possible in-house.

### Why Google

Google sign-in is the most convenient option for most users. It reduces the number of things being built simultaneously and keeps the authentication surface area small, which is better for security and maintainability. The Google ID also serves as a stable, unique identifier for deriving other platform-specific identifiers downstream.

## Delayed Account and Wallet Creation

Nothing is created on the backend until KYC is complete. The sign-in step authenticates the user, but no wallet, no accounts, and no on-chain state exist until after the liveness check and government ID match pass.

This ensures:

- No orphaned or half-created accounts from users who abandon the KYC flow.
- Wallets are only provisioned for verified identities.
- The contract deployment event is triggered once, cleanly, with all required metadata available.

## Wallet Derivation from Google ID

The user's wallet address is derived from their Google ID. This provides a deterministic way to link an authenticated identity to an on-chain address without storing the mapping in a separate lookup table that could become a point of failure or attack.

## Schema Design and Security (Cross-Stage Concern)

The schema is the most important consideration that carries across all stages. Every piece of data must be tied to the authenticated user in a way that prevents one user from accessing another user's resources by manipulating an ID, a query parameter, or any other client-supplied value.

Insecure direct object references (IDOR) are the most commonly discovered vulnerability on platforms like HackerOne. If CrestPace is going to be enterprise-grade and production-ready, the schema and every query built on top of it must be designed to prevent these vulnerabilities from day one. Row-level ownership, proper authorization checks, and never trusting a client-supplied identifier alone are not optional -- they are the baseline.
