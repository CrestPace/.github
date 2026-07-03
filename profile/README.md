# CrestPace

## 👋 Welcome

**CrestPace** is a crypto-native neobank built for peak performance and fast transaction times. The name reflects the platform's emphasis on providing peak quality services with low-latency.

It is a one-stop fintech application designed to solve all of your financial needs, combining traditional banking features with the speed and flexibility of cryptocurrency.

&nbsp;

## ✨ Features

### 💳 Virtual Accounts and Cards

Create virtual accounts and access full card details (card number, CVV, expiry, and billing address). Virtual cards are provisioned quickly and ready for use within the blink of an eye.

&nbsp;

### 💸 Transaction Management

Send payments and receive deposits in the cryptocurrency (or multiple cryptocurrencies) of your choice. All transactions are optimized for speed, giving the platform its name.

&nbsp;

### 🔄 Peer-to-Peer Transfers

Send funds to other CrestPace users by their unique username. No wallet addresses, no QR codes -- just type a username and hit send.

&nbsp;

### 🔁 Automatic Conversion and Liquidity

Balances are automatically converted between cryptocurrencies, stablecoins, and native fiat (such as USD) based on the user's preferences and the transaction context.

**Multi-token payment support:** If the balance in one token is insufficient for a transaction, the system will pull from multiple tokens in the user's wallet based on the user's choice and convert them on the fly to complete the send.

&nbsp;

### 💰 Fixed Interest Savings

Earn interest on saved cryptocurrency held within the bank. Interest payments are disbursed in either USDT or USDC stable coin.

&nbsp;

### 📊 Inbuilt Portfolio Management

Manage a crypto portfolio directly inside the app. Includes automatic stop-losses that convert cryptocurrencies to stablecoins during market downturns based on user-defined thresholds, protecting value without manual intervention.

&nbsp;

### 🏦 Lending Services

Borrow and lend funds through the platform. CrestPace functions as a full fintech institution by enabling credit markets for its users.

&nbsp;

### 🔔 Transaction Notifications

Push and in-app notifications for all account activity: incoming deposits, completed sends, failed conversions, stop-loss triggers, and interest payouts.

&nbsp;

### 🛡️ Compliance and KYC

Identity verification and transaction monitoring. Includes a liveness check built from scratch in-house. Risk scoring will also be added.

&nbsp;

## 🧱 Architecture

Before contributing or making assumptions about how anything works, read [architecture_decisions.md](architecture_decisions.md). This document tracks the key technical decisions that shape the entire codebase.

Skipping it leads to assumptions that are likely wrong. For example: CrestPace relies on as few external services as possible. Standard industry features are coded from scratch rather than outsourced to third-party APIs.

Features currently in planning are tracked separately in [features.md](features.md).

&nbsp;

## 📂 Repositories

| Repository | Description |
|---|---|
| [Backend](https://github.com/CrestPace/Backend) | Core server-side logic, APIs, and infrastructure |
| [contract](https://github.com/CrestPace/contract) | Smart contracts for the platform's on-chain functionality |
| [contract-listener](https://github.com/CrestPace/contract-listener) | Listens for contract events and errors, polishes price data, and manages RPC connections |
| [App-UI](https://github.com/CrestPace/App-UI) | Mobile application user interface |
| [Web-UI](https://github.com/CrestPace/Web-UI) | Website and web-based user interface |
| [Token-Prices-Algorithm](https://github.com/CrestPace/Token-Prices-Algorithm) | Custom algorithm that determines token prices for the platform's native tokens, designed to simulate real-world crypto market dynamics |

&nbsp;

## 🛠️ Tech Stack

**Frontend:** SvelteKit and UI libraries.  
Frontend will mostly be developed by AI, but the architecture, user flow, and backend communication logic will be designed by hand.

**Backend:** Go and Rust.  
Mainly Go, with Rust used for RPC node communication, event listening, and token price polling.

**Contracts:** Solidity.  
The target blockchain is yet to be finalized.

&nbsp;

## 👥 Contributors

The core maintainers are [Rehoboth](https://github.com/AlphaTechini) and [Emmanuel](https://github.com/emma31-dev).

Any vulnerability finding, feature request, or suggested change should be communicated to them first before implementation to avoid unnecessary waste of time.

The backend and contracts are where contributors are needed most. Preferably 2 max for frontend.

Backend and contracts will aim to have at least **50% of code handwritten**.

All AI changes to these repos will be verified manually.

&nbsp;

## 🔍 Review

**CodeRabbit** will be used in this repo and all PRs must resolve any issue raised by CodeRabbit that does not deviate from the architecture.

> AI development rules for this org: [AGENTS.md](AGENTS.md)
