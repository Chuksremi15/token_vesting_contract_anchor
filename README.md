# Token Vesting Smart Contract

A Solana-based token vesting smart contract built with Anchor framework that enables companies to create and manage token vesting schedules for their employees with support for cliffs and linear vesting.

## 🚀 Features

- **Company Vesting Accounts**: Create isolated vesting accounts per company
- **Employee Vesting Schedules**: Set up individual vesting schedules for employees
- **Cliff Support**: Implement vesting cliffs to prevent early token claims
- **Linear Vesting**: Gradual token release over time
- **Secure Treasury Management**: Company-controlled treasury accounts
- **PDA-based Architecture**: Uses Program Derived Addresses for secure account management
- **Cross-Program Invocation**: Integrates with SPL Token program for secure transfers

## 🛠️ Installation

1. **Clone the repository**

   ```bash
   git clone <repository-url>
   cd token_vesting
   ```

2. **Install dependencies**

   ```bash
   cd anchor
   yarn install
   ```

3. **Build the program**

   ```bash
   anchor build
   ```

4. **Generate TypeScript types**
   ```bash
   anchor build
   ```

## 🛠️ Architecture

### Core Components

#### 1. **VestingAccount**

- Represents a company's vesting program
- Contains treasury token account and company metadata
- PDA derived using company name as seed

#### 2. **EmployeeAccount**

- Individual employee vesting schedule
- Tracks vesting parameters and withdrawal history
- PDA derived using beneficiary and vesting account

#### 3. **Treasury Token Account**

- Company-controlled token account for vesting distributions
- PDA derived using company name and "vesting_treasury" seed

### Account Structure

```rust
// VestingAccount - Company level
pub struct VestingAccount {
    pub owner: Pubkey,                    // Company owner
    pub mint: Pubkey,                     // Token mint
    pub treasury_token_account: Pubkey,   // Treasury PDA
    pub company_name: String,             // Company identifier
    pub treasury_bump: u8,                // Treasury PDA bump
    pub bump: u8,                         // Vesting account bump
}

// EmployeeAccount - Employee level
pub struct EmployeeAccount {
    pub beneficiary: Pubkey,              // Employee wallet
    pub start_time: i64,                  // Vesting start timestamp
    pub end_time: i64,                    // Vesting end timestamp
    pub total_amount: i64,                // Total tokens to vest
    pub total_withdrawn: i64,             // Already claimed amount
    pub cliff_time: i64,                  // Cliff end timestamp
    pub vesting_account: Pubkey,          // Parent vesting account
    pub bump: u8,                         // Employee account bump
}
```

## 🛠️ Instructions

### 1. Create Vesting Account

Creates a new company vesting account with treasury.

```typescript
await program.methods
  .createVestingAccount("Company Name")
  .accounts({
    signer: companyOwner.publicKey,
    vestingAccount: vestingAccountPda,
    mint: tokenMint.publicKey,
    treasuryTokenAccount: treasuryPda,
    tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
    systemProgram: anchor.web3.SystemProgram.programId,
  })
  .signers([companyOwner])
  .rpc();
```

### 2. Create Employee Vesting

Sets up a vesting schedule for an employee.

```typescript
await program.methods
  .createEmployeeVesting(
    startTime, // Unix timestamp
    endTime, // Unix timestamp
    totalAmount, // Token amount (with decimals)
    cliffTime // Unix timestamp
  )
  .accounts({
    owner: companyOwner.publicKey,
    beneficiary: employee.publicKey,
    vestingAccount: vestingAccountPda,
    employeeAccount: employeeAccountPda,
    systemProgram: anchor.web3.SystemProgram.programId,
  })
  .signers([companyOwner])
  .rpc();
```

### 3. Claim Tokens

Allows employees to claim their vested tokens.

```typescript
await program.methods
  .claimTokens("Company Name")
  .accounts({
    beneficiary: employee.publicKey,
    employeeAccount: employeeAccountPda,
    vestingAccount: vestingAccountPda,
    mint: tokenMint.publicKey,
    treasuryTokenAccount: treasuryPda,
    employeeTokenAccount: employeeAta,
    tokenProgram: anchor.utils.token.TOKEN_PROGRAM_ID,
    associatedTokenProgram: anchor.utils.token.ASSOCIATED_TOKEN_PROGRAM_ID,
    systemProgram: anchor.web3.SystemProgram.programId,
  })
  .signers([employee])
  .rpc();
```

## 🧪 Testing

Run the test suite:

```bash
anchor test
```

Or run with verbose output:

```bash
anchor test --skip-lint
```

## 🚀 Deployment

### Local Development

1. **Start local validator**

   ```bash
   solana-test-validator
   ```

2. **Deploy to localnet**

   ```bash
   anchor deploy
   ```

3. **Update program ID** (if needed)
   - Copy the new program ID from deployment output
   - Update `declare_id!()` in `lib.rs`
   - Update `Anchor.toml` programs section

### Mainnet/Devnet

1. **Switch to target cluster**

   ```bash
   solana config set --url <cluster-url>
   ```

2. **Deploy**
   ```bash
   anchor deploy
   ```

## 📊 Vesting Calculation

The vesting calculation follows a linear vesting model:

```rust
// Calculate vested amount
let time_since_start = now.saturating_sub(employee_account.start_time);
let total_vesting_time = employee_account.end_time.saturating_sub(employee_account.start_time);
let vested_amount = if now >= employee_account.end_time {
    employee_account.total_amount
} else {
    (employee_account.total_amount * time_since_start) / total_vesting_time
};

// Calculate claimable amount
let claimable_amount = vested_amount.saturating_sub(employee_account.total_withdrawn);
```

## 🔒 Security Features

- **PDA-based accounts**: All accounts use Program Derived Addresses for security
- **Access control**: `has_one` constraints ensure proper authorization
- **Cliff protection**: Prevents early token claims
- **Atomic operations**: All state changes are atomic
- **Input validation**: Comprehensive error handling and validation

## 🛡️ Error Handling

The program includes custom error codes:

```rust
#[error_code]
pub enum ErrorCode {
    #[msg("Claiming is not available yet.")]
    ClaimNotAvailableYet,
    #[msg("There is nothing to claim.")]
    NothingToClaim,
}
```
