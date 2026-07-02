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
