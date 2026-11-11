# Vault (Anchor / Solana)

Week 2 assignment: a Solana vault program built with Anchor, with `initialize`, `deposit`, `withdraw`, and `close` instructions, tested with LiteSVM.

## Overview

Each user gets a personal vault: a PDA that holds SOL, plus a state PDA that stores the bump seeds needed to sign for the vault.

## Accounts and PDAs

| Account | Seeds | Purpose |
|---|---|---|
| `vault_state` | `["state", user]` | Stores `vault_bump` and `state_bump` |
| `vault` | `["vault", user]` | Holds the user's SOL |

## Instructions

- **`initialize`**: Creates `vault_state`, transfers the rent-exempt minimum (for 0 data bytes) from the user to `vault`, and stores both bumps.
- **`deposit(amount)`**: Transfers `amount` from the user to `vault`. `amount` must be greater than 0.
- **`withdraw(amount)`**: Transfers `amount` from `vault` to the user. The vault PDA signs the transfer. `amount` must be greater than 0 and must not exceed the vault balance minus the rent-exempt minimum, so the rent reserve always stays in the vault.
- **`close`**: Transfers all remaining lamports from `vault` to the user (the vault PDA signs), and closes `vault_state`, returning its rent to the user.

## Errors

- `InvalidAmount`: The amount is zero. Note that the message text says "Deposit amount must be greater than zero", and the same error is also used by `withdraw`.
- `InsufficientBalance`: A withdrawal would take more than the available balance (vault balance minus the rent reserve).

## Tests

`tests/test_initialize.rs` runs the full flow in LiteSVM (a local simulated Solana environment):

1. Airdrop SOL to a test user and load the built program.
2. `initialize`: check that `vault_state` exists and `vault` holds the rent-exempt amount.
3. `deposit`: check the balance after depositing.
4. `withdraw`: check that the vault balance equals rent + deposit − withdrawal.
5. `close`: check that `vault_state` is closed and `vault` is empty.

`src/lib.rs` also has a unit test (`test_id`) that checks the program ID.

## How to run

```bash
anchor build
anchor test
```

Current result: all tests pass (`test result: ok`). See the screenshot in `docs/test-results.png`.

## Environment

- anchor-cli 1.1.2
- solana-cli 3.1.10
- Rust 1.89.0 (pinned in `rust-toolchain.toml`)
- Windows 11 with WSL2 (Ubuntu)

## Notes

- The program ID was synced with `anchor keys sync`.
- The starter code comes from `moses7054/anchor_vault_starter_q2_26`. The `withdraw` and `close` instructions and their tests were completed by me.
- The Escrow program (make, take, refund, update) is not included in this repository yet.
