# Architecture Decisions

This document records the key technical decisions behind CrestPace. Read it before contributing. Skipping it leads to assumptions that are likely wrong.

## Phase 1: Testnet-Driven Engineering

Phase 1 integrates with a real blockchain on testnet. Testnet tokens are used for all balances and transactions rather than virtual or simulated balances. This adds real weight to the project and ensures it is not a front-end-only placeholder.

Live token prices are fetched from external APIs. The only external dependencies in Phase 1 are:

- **Price feed APIs** for current token prices.
- **RPC providers** for blockchain communication.

A faucet will be used to dispense testnet tokens for the supported currencies. If faucet access proves unreliable for a given token, mock tokens will be used as a fallback.

The blockchain will be chosen for speed, strong performance, and a positive developer experience.

## Principle: Minimal External Dependencies

CrestPace is built to rely on as few external services as possible. Standard industry features that most fintechs buy off the shelf are built from scratch within the project.

This applies across the board:

- **Recurring payments and sweeps** are not implemented via Stripe, Plaid, or any payment orchestration service. The scheduling engine, balance sweeping logic, and retry mechanisms are all built in-house.
- **Interest calculations and disbursements** use an internal ledger and compounding engine rather than an external banking API.
- **Stop-loss and portfolio management** logic runs on the backend, not through a brokerage integration or market-data provider.
- **Notifications** are delivered through an in-house push infrastructure rather than a SaaS notification platform.

### Trade-off: Why Build from Scratch

- **Primary reason:** To display engineering brilliance. Building the hard way, correctly, is the point.
- **Significant factor:** The experience gained from implementing standard industry features at a low level is invaluable.
- **Secondary benefit:** Full control over every component, no vendor lock-in, no usage-based pricing surprises, and the ability to tailor each feature to the platform's exact needs rather than working around a third party's API limitations.

## KYC and Identity Verification

KYC is implemented in-house. Image processing and part of the verification pipeline run inside a Trusted Execution Environment (TEE) to prevent the leaking of personal data.

The initial scope covers:

1. **Liveness check** -- Verifies that a real person is present during onboarding. Built from scratch, not delegated to an external identity provider.
2. **Government ID processing** -- Used solely to pattern-match the face on the official document against the liveness capture. It is not used to confirm nationality or any other demographic attribute.

Images captured during verification are ephemeral. Once the face on the government ID has been matched against the liveness capture, the images are discarded. They are never persisted to a database, file system, or any other storage medium.

Risk scoring is planned for a later phase.

## User Onboarding and Fund Allocation

CrestPace avoids on-ramping real currencies and stays off mainnets during early phases. Instead of requiring users to deposit real funds, an allocation method is used to seed each user's balance. The goal is to make the experience as realistic as possible without touching real money.

The fund allocation process is split across two phases.

### Phase 1: Manual Allocation

After completing the KYC flow (liveness check and government ID match), the user manually enters the deposit amount they want allocated to their account. No AI integration, no web search -- just a direct number from the user. The system accepts the figure and proceeds to token selection.

### Phase 2: AI-Driven Allocation

The manual flow is replaced with an automated salary lookup:

1. **User Details.** After KYC, the user provides employment details: job title, geographic location, specific role, and industry (e.g., a software engineer at an agricultural startup in Lagos).

2. **AI-Driven Fund Allocation.** The system performs a web search to determine the average salary for that specific role in that specific geographic area. The retrieved figure becomes the user's initial deposit amount. This ensures each user starts with a balance that reflects real-world earning data rather than an arbitrary fixed number.

### Token Selection (Both Phases)

Regardless of how the deposit amount is determined, the user selects which tokens to allocate their deposit to and defines the percentage each token should receive. For example, a user with a $5,000 deposit might allocate 60% to USDC, 30% to ETH, and 10% to BTC. The system distributes the deposit across the chosen tokens according to the user's specified percentages.

### Design Rationale

- **No real on-ramp, no mainnet risk.** Early development and testing happen entirely on testnets with testnet tokens. There is zero financial liability.
- **Phase 1 ships faster.** Manual allocation removes the AI integration from the critical path, so the onboarding flow can be built and tested without depending on an external search pipeline.
- **Phase 2 adds realism.** Once the core platform is stable, the AI-driven salary lookup produces deposit amounts that feel authentic. A software engineer in San Francisco and a teacher in Nairobi will receive materially different starting balances, which mirrors how the platform would behave in production.
- **User agency over token mix.** Pushing all deposits into a single default token would force users into an allocation they did not choose. Letting the user define percentages up front respects their preferences and educates them about multi-token portfolio construction from day one.
- **Built in-house.** The scheduling engine for recurring allowances and the bulk transfer processor (see [features.md](features.md)) handle the disbursement of these allocated funds without external payment services, consistent with the principle of minimal external dependencies.

## QR Code Scanning (Phase 2)

In-app QR code scanning will be added in Phase 2. A user with the device running the app (or web interface) can scan a QR code displayed on another device to resolve a recipient's payment details without manual typing. The scanner reads the encoded username or wallet address and passes it to the P2P transfer flow.

This feature is deferred to Phase 2 so Phase 1 stays focused on core transfers, scheduled sends, and bulk payments. The username-based P2P routing system already provides a simple, error-resistant way to send funds without QR codes, so QR scanning is a convenience layer rather than a foundational requirement.

## Deposit and Lock Funds (Fixed-Term Staking)

Users can lock funds for a fixed term and earn a percentage-based reward at maturity. This is a simple staking model, not a dynamic yield product. The architecture is designed around three rules:

1. **Fixed term.** The initial term is 30 days. Shorter or longer terms may be added later based on demand, but there is no variable-duration logic in Phase 1.
2. **Fixed reward rate.** For every 30-day period, the current placeholder rate is 2%. This figure is not final. Market research will be conducted later to determine a defensible methodology for setting the actual percentage (e.g., benchmarking against comparable on-chain staking yields or traditional fixed-deposit rates in target regions).
3. **Early unlock with slashing.** Users can unlock funds before the term ends, but the reward is reduced. The penalty is calculated against the remaining balance of the locked tokens at the time of early unlock, so partial unlocks are implicitly penalized as well. The exact slashing curve will be defined when the rate methodology is finalized.

### Design Rationale

- **Predictable, not algorithmic.** A fixed rate keeps the smart contract and backend simple. There is no need for an on-chain oracle, yield aggregator, or dynamic rate engine. The rate is set administratively and updated only when market research justifies a change.
- **Slashing protects the model.** Without a penalty for early exits, users could deposit right before the reward payout and earn a full-term return for a short lock, breaking the economics. Slashing ensures rewards are proportional to the actual lock duration.
- **Separate from the interest savings feature.** The savings feature in the main README pays interest on held balances without a lock. This feature is a distinct product: voluntary lock-up in exchange for a higher, guaranteed return.

## P2P Transfers

Peer-to-peer transfers use unique usernames as the routing identifier. No wallet addresses are exposed to the sender. The system resolves the username to the underlying wallet address internally, validates the recipient, and executes the transfer. The goal is to keep the user experience simple and enjoyable, while the backend handles address management, multi-token conversion, and fallback logic.

## Multi-Token Payments

When a user initiates a send and their balance in the selected token is insufficient, the system surfaces the shortfall and prompts the user to select which tokens they want to swap to cover the remainder. Swaps are never performed behind the scenes without explicit user consent. The user must actively choose which tokens to convert before the transaction proceeds.

## Parallel Application Instance

A parallel version of CrestPace runs entirely on platform-native tokens whose prices are determined by the [Token-Prices-Algorithm](https://github.com/CrestPace/Token-Prices-Algorithm) rather than external exchanges. This instance is identical in features but operates in a controlled economic environment. It is used for testing, demonstration, and as a sandbox for the pricing algorithm itself.

## Implementation Language Strategy

- **Go** for the majority of backend services: API layer, business logic, transaction engine, ledger, and interest calculations. Chosen for performance, simplicity, and strong concurrency primitives.
- **Rust** for performance-critical paths that sit close to the blockchain: RPC communication, event listening, and token price polling. Chosen for safety guarantees and low-level control when interacting with nodes directly.
- **Solidity** for smart contracts.
