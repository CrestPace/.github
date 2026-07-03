# Features

This document lists features planned for CrestPace that are not yet covered in the main [README](README.md). Each entry describes what the feature does at a high level. The architectural rationale behind these features lives in [architecture_decisions.md](architecture_decisions.md).

## Schedule Sending (Allowances and Payroll)

Schedule the automated transfer of funds to specific usernames or wallet addresses at defined intervals (e.g., at the end of each month). This covers use cases such as recurring allowances, payroll disbursements, and subscription-style payouts.

The scheduling engine is built in-house rather than relying on external payment orchestration services. It handles:
- One-time scheduled transfers executed at a future date.
- Recurring transfers with configurable intervals (daily, weekly, monthly, etc.).
- Retry logic for failed transfers due to insufficient balances or network conditions.
- Notifications when a scheduled transfer succeeds, fails, or is approaching execution.

## Bulk Instant Transfers

Send funds to a large number of recipients in a single action, without scheduling. This is an instant, one-to-many transaction designed for scenarios such as batch payroll runs, airdrops, or mass refunds.

Unlike scheduled sends, bulk transfers execute immediately when the user confirms. The system processes all recipients in the batch and returns a per-recipient status (success, insufficient balance, invalid recipient, etc.) so the sender can audit the result without inspecting each transfer individually.

## QR Code Scanning (Phase 2)

In-app QR code scanning allows a user to scan a QR code displayed on another device (phone, tablet, or desktop) to quickly populate a recipient's payment details. This eliminates manual entry of usernames or wallet addresses for in-person transfers.

This feature is deferred to Phase 2 to keep Phase 1 focused on core payment flows and scheduled transfers.

## Deposit and Lock Funds (Fixed-Term Staking)

Lock a specific amount of funds for a set period of time and earn a fixed percentage reward in return. This is a simple fixed-deposit model: the user chooses an amount, locks it for a term (starting at 30 days), and receives the reward when the term ends.

Early unlocking is supported but penalized. If a user unlocks funds before the term expires, the reward percentage is slashed based on the remaining balance of the locked tokens. This protects the integrity of the fixed-term model while giving users an escape hatch when needed.
