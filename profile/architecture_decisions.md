# Architecture Decisions

This document records the key technical decisions behind CrestPace. Read it before contributing. Skipping it leads to assumptions that are likely wrong.

## Phase 1: Testnet-Driven Engineering

Phase 1 integrates with a real blockchain on testnet. Testnet tokens are used for all balances and transactions rather than virtual or simulated balances. This adds real weight to the project and ensures it is not a front-end-only placeholder.

Live token prices are fetched from external APIs. The only external dependencies in Phase 1 are:

- **Price feed APIs** for current token prices.
- **RPC providers** for blockchain communication.

A faucet will be used to dispense testnet tokens for the supported currencies. If faucet access proves unreliable for a given token, mock tokens will be used as a fallback.

The blockchain will be chosen for speed, strong performance, and a positive developer experience.

## Token Mirroring

CrestPace deploys its own versions of tokens on the BNB testnet. Each mirrored token represents a real-world cryptocurrency (BTC, ETH, USDC, etc.) and is used for actual transactions on the platform.

Smart contracts cannot make external calls, so prices are not stored on-chain. Instead, the price of the real-world counterpart is fetched from external APIs on-demand. The architecture is event-driven: when a user triggers a transaction request, the backend fetches the exact current price immediately and uses that price for that particular transaction.

This means every transaction uses the real-world price at the moment it is executed, not a stale price written to a contract earlier.

### How It Works

- A mirrored USDC, ETH, BTC, or any other supported token is deployed as a smart contract on BNB testnet. These contracts handle transfers and balances, not price storage.
- When a user initiates a transaction, the backend fetches the current price of the real-world token from an external API.
- The fetched price is used for that specific transaction: conversions, swaps, balance calculations, stop-loss triggers, and interest calculations all use the price fetched at the point of the request.
- No price is written to the contract. No oracle is consulted on-chain. The fetch happens off-chain in the backend, and the result is applied to the transaction directly.

### Trade-Off

- **Up-front complexity.** Deploying and maintaining mirrored tokens for every supported asset adds initial contract work. The price-fetching pipeline also needs to be built and hardened with fallbacks.
- **Event-driven accuracy.** Because the price is fetched at the point of each transaction, the system always uses the most current price available. There is no risk of transacting against a stale on-chain price that was written minutes or hours ago.
- **Price feeds remain an external dependency.** Every transaction depends on an external API call succeeding. The fallback system (see below) is what makes this reliable.

## Price Feed Fallback System

The off-chain price feed service depends on external APIs. To prevent downtime, a three-tier fallback system is used.

### Primary API

The main option is a primary API with a very generous free tier. This should allow fetching prices as many times as possible for free.

### Fallbacks

Fallbacks are essential in two cases:

1. **Quota exhaustion.** If the monthly limit on the first API is surpassed, the fallback APIs cover the remaining requests. The background cron job for regular price refreshes is designed so that it does not exhaust the monthly quota on its own.
2. **Event-driven capacity.** Whenever a user triggers a transaction, the system still needs to fetch prices and update the tokens. Enough request capacity must be left on the primary API for these event-driven requests, and the fallbacks absorb the overflow.

### Redundancy Scenarios

1. If the first API fails, the system moves to the second one.
2. In the unlikely scenario that the second API also fails, a third API serves as the backup.
3. Alternatively, if quotas on both the first and second APIs are exhausted due to high request volume, the third API covers the remaining amount needed before the end of the month.

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

After completing the KYC flow (liveness check and government ID match), the user provides employment details: industry, exact role, and the city they work from. The system performs a web search to determine the average salary for that specific role in that specific geographic area.

The user then chooses a percentage of that salary to use as their deposit amount. This ensures each user starts with a balance that reflects real-world earning data rather than an arbitrary fixed number, while still giving the user control over how much of their salary they want to allocate.

### Token Selection

The user selects which tokens to allocate their deposit to and defines the percentage each token should receive. For example, a user with a $5,000 deposit might allocate 60% to USDC, 30% to ETH, and 10% to BTC. The system distributes the deposit across the chosen tokens according to the user's specified percentages.

### Design Rationale

- **No real on-ramp, no mainnet risk.** Early development and testing happen entirely on testnets with testnet tokens. There is zero financial liability.
- **Realistic starting balances.** The salary lookup produces deposit amounts that feel authentic. A software engineer in San Francisco and a teacher in Nairobi will receive materially different starting balances, which mirrors how the platform would behave in production.
- **User agency over deposit size.** Letting the user choose a percentage of their salary rather than forcing the full amount respects their preferences and mirrors real life, where you decide how much of your pay to deposit in your bank.
- **User agency over token mix.** Pushing all deposits into a single default token would force users into an allocation they did not choose. Letting the user define percentages up front respects their preferences and educates them about multi-token portfolio construction from day one.
- **Built in-house.** The scheduling engine for recurring allowances and the bulk transfer processor (see [features.md](features.md)) handle the disbursement of these allocated funds without external payment services, consistent with the principle of minimal external dependencies.

## QR Code Scanning (Phase 2)

In-app QR code scanning will be added in Phase 2. A user with the device running the app (or web interface) can scan a QR code displayed on another device to resolve a recipient's payment details without manual typing. The scanner reads the encoded username or wallet address and passes it to the P2P transfer flow.

This feature is deferred to Phase 2 so Phase 1 stays focused on core transfers, scheduled sends, and bulk payments. The username-based P2P routing system already provides a simple, error-resistant way to send funds without QR codes, so QR scanning is a convenience layer rather than a foundational requirement.

## Deposit and Lock Funds (Fixed-Term Staking)

Users can lock funds for a fixed term and earn a percentage-based reward. This is a simple staking model, not a dynamic yield product. The architecture is designed around three rules:

1. **Fixed term.** The initial term is 30 days. Shorter or longer terms may be added later based on demand, but there is no variable-duration logic in Phase 1.
2. **Fixed reward rate.** For every 30-day period, the current placeholder rate is 2%. This figure is not final. Market research will be conducted later to determine a defensible methodology for setting the actual percentage (e.g., benchmarking against comparable on-chain staking yields or traditional fixed-deposit rates in target regions).
3. **Daily interest accrual.** Interest is calculated and added daily based on the deposited amount, not as a lump sum at maturity. On full withdrawal, the user receives the principal plus the interest that was already accrued daily. On partial withdrawal, the interest is recalculated going forward using the new lower balance. This keeps partial withdrawals fair without any complex penalty logic.

### Design Rationale

- **Predictable, not algorithmic.** A fixed rate keeps the smart contract and backend simple. There is no need for an on-chain oracle, yield aggregator, or dynamic rate engine. The rate is set administratively and updated only when market research justifies a change.
- **Daily accrual is simpler than slashing.** The original idea was to invalidate interest on early full withdrawal and pay based on what's left for partial. Daily accrual replaces that: the user keeps what was earned up to the point of withdrawal, and partial withdrawals just recalculate on the new balance. No penalty, no slashing curve to define.

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


## Token Allocation and Circulation

Even if this isn't a real bank, we still want to mimic the real ones as much as possible.

So first, how do we get the tokens (remember we choose Web3 not just because we can actually carry out transfers of tokens with monetary value, but cos if we didn't then we would basically be storing user's money as just numbers. It won't hold weight. The thought of it would make me quit doing this).

With cryptocurrencies, even though on testnet, we carry out actual blockchain interactions that can be verified. We aren't mocking the money here.

So, how will we do this?

Minting... We mint our own tokens. Here's the catch: They are going to be mirrors of real popular cryptocurrencies. What this means is our minted token price = price of real token e.g. BTC.

Well, we might not be able to create a token literally called BTC even on testnet cos it's probably taken but that's fine. Whatever name we assign will still have part of the original token name in it. Plus the users will still see the names they are familiar with on the frontend. This is to improve User Experience.

The backend will handle all the mapping. In fact, a better method will be to make sure that they are mapped in the frontend, that way no need for backend mapping. However, the backend will still validate token symbol to prevent any potential security vulnerabilities.

So how do we determine token supply? Do we just mint a billion tokens for each token we will be mirroring and call it a day? NO! Absolutely not! It defies the real bank mimicking idea.

So, what do we do? Well, we mint on demand hence the Genius Onboarding flow where we accurately get the average salary of the individual and ask how many percent to deposit.

Why this?

Let me explain. In the real world you want to use this neobank for regular day-to-day transactions without having to use your standard bank. You receive your pay for the month, ponder on the amount to deposit in the neobank, then you proceed to send that amount there.

That's exactly the flow we are trying to recreate here...

Now for actual token allocation, just like you would choose how much to spend on different things for the month or how much to invest in different stocks and obviously how much to put in different cryptocurrencies. We allow you to set the percentage of your deposited amount you allocate for each token.

Now after you have allocated, we mint the exact (quantity + 5%) in case the price changes within the short time. Although we fetch the price just before minting, crypto is volatile. Now here's the catch. We also mint the stablecoin equivalent, i.e. your preferred stablecoin choosing when they wanted to lock funds. Same stuff even if they didn't choose to lock funds they will still have to put preferred stablecoin.

Now why also mint stablecoins.
I thought of something: we only actually have access to the locked funds... The other tokens in circulation are in the user's active accounts. So when we want to implement stop losses, how do we get a massive amount of stablecoin equivalent of a user's active balance... Simple, by minting that exact amount at the beginning.

Now here's the moment I realised this was a genius architecture by me:
Later on when the market stabilizes a bit, users want to buy tokens again. Well guess what, since we are mimicking real banks, when the stop losses feature swaps from the tokens they own to stablecoins, we end up storing the tokens they held before. ..Yh maybe I didn't specify but we have our own store where:
  1. The deposit to activate stop loss is stored
  2. The swaps when stop loss activates is stored.
This enables a proper cycle.
  However, I quickly noticed flaws: It's crypto, what if the user's portfolio improved in value.
  Well it's crypto, another person's portfolio definitely reduced in value. So they balance each other out. The store is like an aggregate pool. It's not separated per user.

  Even with that, what if we apply the same + 5% logic to the stablecoins we mint... The possibility of encountering any trouble = very low.
  Now there's also the way banks work. I had to read articles and watch videos about how real banks, Neobanks, ledgers work.

  The main part has to do with lending where they bet not everyone is going to withdraw all their funds at once. Adding this scenario, it's impossible for any problems to occur.

  Talk about sheer human intelligence. IMPORTANT: NO AI was consulted or assisted to the creation of this near-perfect system. Proudly made by a human who hasn't yet fried his brain cells.


SWAPS:
   Now the part I also want to talk about... In this case I have not been able to find a better solution than what individual banks give. However, remember the + 5%, the excess is also stored in our pool. And I meant +5% for each token.

   When a user requests a swap we first check if we can fulfill with only our pool without the reserves for that token going below 5%.

   If it does then we have to act like banks... use someone's/a group of people's savings account...
   I haven't yet come up with an architecture for using a group of people...

   For the single person it will be the one with the least number of transactions.

   Now does this mean the user will wake up to see a debit alert or the balance decrease? Nope...
   Meaning I'll have to properly label this particular operation such that the backend doesn't trigger any updates to the user's viewed balance. This makes me want to implement a dual balance architecture:

   Here the user has the virtual balance: what he/she sees.   AND
   The actual balance: This one has all the user's token movement. In fact, this will be directly tied to the ledger.
   I haven't thought about this enough yet though.

   There is one caveat though: we have to make the virtual balance + non-lending operations from the ledger source of truth.

   Honestly the virtual = actual - lending operations.

   This is better and more direct meaning we approve any transaction the user requests using virtual as source of truth. The actual is only going to be referenced to see if we need to borrow or use our pool to restock the user's balance behind the scenes. Making actual = virtual once again.

  I have tried my best to design the architecture in a way that doesn't mint tokens again after the onboarding, causing it to be more like a real bank works.
   