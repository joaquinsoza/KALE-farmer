# KALE Farmer

An automated farming bot for the [KALE Protocol](https://github.com/kalepail/KALE-sc) on the Stellar blockchain. This farmer participates in the KALE farming game by planting, performing proof-of-work mining, and harvesting rewards.

## What is KALE Farming?

KALE is a Stellar smart contract farming game where participants earn KALE tokens through a three-phase process:

1. **Plant**: Stake KALE tokens (or 0 to start) to enter a farming block
2. **Work**: Submit proof-of-work hashes by finding nonces with maximum leading zeros
3. **Harvest**: Claim rewards based on timing (gap), stake amount, and PoW difficulty (zeros)

Blocks advance every 5 minutes, creating competitive farming rounds.

## Architecture

This farmer consists of two components:

- **Rust Mining Engine** (`src/main.rs`): Multi-threaded Keccak256 hash computation for optimal performance
- **TypeScript Orchestrator** (`bun_scripts/plant_and_work.ts`): Manages the farming lifecycle, block detection, and transaction submission

## Prerequisites

### Required

- **Rust** (1.70+) - For compiling the mining engine
- **Bun** (1.0+) - JavaScript runtime for the orchestrator
- **Stellar Account** with 5-10 XLM for transaction fees
- **LaunchTube Account** - For transaction submission service

### Optional

- **KALE Tokens** - You can start with 0 stake and earn through farming

## Installation

### Step 1: Install Rust

Follow the Stellar smart contract setup guide:

> https://developers.stellar.org/docs/build/smart-contracts/getting-started/setup

Or install directly:
```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Step 2: Install Bun

> https://bun.sh

```bash
curl -fsSL https://bun.sh/install | bash
```

### Step 3: Clone and Build

```bash
git clone https://github.com/kalepail/KALE-farmer.git
cd KALE-farmer

# Build the Rust mining engine
make build

# Install TypeScript dependencies
cd bun_scripts
bun install
```

## Configuration

### Step 1: Create Your Environment File

Copy the example configuration:

```bash
cd bun_scripts
cp .env.example .env
```

### Step 2: Configure Your Environment

Edit `.env` with your settings:

```bash
# Network Selection
ENV=mainnet
RPC_URL=https://mainnet.sorobanrpc.com
NETWORK_PASSPHRASE="Public Global Stellar Network ; September 2015"

# LaunchTube (Transaction Submission Service)
LAUNCHTUBE_URL=https://launchtube.xyz
LAUNCHTUBE_JWT=<your-launchtube-jwt>

# Your Stellar Account
FARMER_PK=<your-public-key>
FARMER_SK=<your-secret-key>

# KALE Contract
CONTRACT_ID=CDL74RF5BLYR2YBLCCI7F5FB6TPSCLKEJUBSD2RSVWZ4YHF3VMFAIGWA

# Farming Parameters
STAKE_AMOUNT=0           # Start with 0 if you don't have KALE yet (in stroops, 1 KALE = 1e7 stroops)
NONCE_COUNT=10000000     # Number of hashes to compute per block
```

### Getting LaunchTube JWT

LaunchTube is a service for submitting gasless Soroban transactions to the Stellar network:

- **Testnet**: Generate a token at [testnet.launchtube.xyz/gen](https://testnet.launchtube.xyz/gen)
- **Mainnet**: Request a token via Stellar Discord's `#launchtube` channel

After obtaining your JWT, add it to your `.env` file.

### Network Options

The farmer supports three networks:

**Mainnet** (Production):
```bash
bun run farm
```

**Testnet** (Testing):
```bash
bun run farm:testnet
```

**Futurenet** (Experimental):
```bash
bun run farm:futurenet
```

## Running the Farmer

### Option 1: Direct Execution (Foreground)

Run the farmer directly in your terminal:

```bash
cd bun_scripts
bun run farm
```

This runs in the foreground and will stop if you close the terminal or the process crashes.

### Option 2: PM2 Process Manager (Recommended for 24/7 Operation)

For production use, PM2 will automatically restart the farmer if it crashes and keep it running in the background:

#### First Time Setup

Install PM2 globally (one-time):
```bash
npm install -g pm2
```

Or use the local PM2 from the project (no global install needed):
```bash
cd bun_scripts
bun install  # Installs PM2 as dev dependency
```

#### Starting the Farmer with PM2

**Mainnet:**
```bash
cd bun_scripts
bun run pm2:start
# or with npx: npx pm2 start ../ecosystem.config.cjs --only kale-farmer
```

**Testnet:**
```bash
bun run pm2:start:testnet
```

**Futurenet:**
```bash
bun run pm2:start:futurenet
```

#### Managing PM2 Processes

**Check status:**
```bash
bun run pm2:status
# Shows: PID, status, CPU, memory, uptime, restarts
```

**View logs (live tail):**
```bash
bun run pm2:logs
# Press Ctrl+C to exit log view
```

**Monitor resource usage:**
```bash
bun run pm2:monit
# Real-time CPU and memory monitoring
```

**Restart the farmer:**
```bash
bun run pm2:restart
```

**Stop the farmer:**
```bash
bun run pm2:stop
```

**Remove from PM2:**
```bash
bun run pm2:delete
```

#### PM2 Features

- **Auto-restart on crash**: If the process exits unexpectedly, PM2 automatically restarts it
- **Memory limits**: Restarts if memory exceeds 500MB (prevents runaway processes)
- **Exponential backoff**: Prevents rapid restart loops on persistent errors
- **Log management**: Stores logs in `bun_scripts/logs/` directory
- **Startup on boot**: Configure PM2 to start farming on system restart:
  ```bash
  pm2 startup
  pm2 save
  ```

#### PM2 Log Files

Logs are stored in `bun_scripts/logs/`:
- `kale-farmer-out.log` - Standard output (farming activity)
- `kale-farmer-error.log` - Error output
- Similar files for testnet and futurenet variants

### Expected Output

```
95825 { max_gap: 51, max_stake: 99384900n, max_zeros: 9, min_gap: 2, min_stake: 0n, min_zeros: 5 }
Successfully planted 10
Booting... 0
Proc booted
Hashrate: 28.66 MH/s
["1346520", "00000052594ee8c93fc0da184d5b087eac35b96e7a475c1f37d14e0cbfcbdef0"]
Waiting 3s before submitting work to ensure gap >= 1...
Successfully worked [result] 5
```

### What's Happening?

1. **Block Detection**: Polls contract every 5 seconds for new blocks
2. **Planting**: Stakes your configured amount (or 0) to enter the block
3. **Mining**: Spawns Rust subprocess to find best nonce with maximum leading zeros
4. **Work Submission**: Waits for minimum ledger gap, then submits PoW result
5. **Harvesting**: Claims rewards from previous blocks automatically

### Monitoring Performance

- **Hashrate**: Shows your mining performance in MH/s (megahashes per second)
- **Leading Zeros**: More zeros = higher PoW difficulty = better rewards
- **Gap**: Ledgers between plant and work (affects rewards)
- **Stake**: Your staked amount (affects rewards)

## Harvesting Rewards

Rewards are automatically harvested when new blocks arrive. You can also manually harvest previous blocks:

```bash
bun run harvest        # Mainnet
bun run harvest:testnet  # Testnet
```

This harvests the previous 24 hours of blocks (288 blocks at 5-minute intervals).

## Farming Strategy

### Starting with 0 KALE

Your `.env.example` has `STAKE_AMOUNT=0`, perfect for starting:

**Pros**:
- No initial KALE investment needed
- Earn rewards through timing (gap) and PoW (zeros)
- Build up your balance over time

**Cons**:
- Lower rewards per block (missing stake factor)

### Farming with Stake

Once you have KALE, increase your stake for better rewards:

```bash
STAKE_AMOUNT=100000000  # 10 KALE (in stroops)
```

Your reward is calculated from three normalized factors:
- **Gap**: Timing between plant and work (ledgers passed)
- **Stake**: Amount of KALE staked
- **Zeros**: Proof-of-work difficulty (leading hex zeros)

## Troubleshooting

### Error: `GapCountTooLow` (Error #15)

This means work was submitted too quickly after planting. The farmer includes automatic delay handling, but if you still see this:
- Ensure you're running the latest version
- Check that `plantedAt` tracking is working
- The minimum delay is 10 seconds (2+ ledgers)

### Error: `HashInvalid` (Error #13)

The submitted hash doesn't match the computed hash:
- Verify your Rust binary is up to date (`make build`)
- Check that `FARMER_PK` in `.env` matches your actual public key

### Error: `ZeroCountTooLow` (Error #7)

Your PoW submission has too few leading zeros:
- Increase `NONCE_COUNT` for more hash attempts
- Ensure Rust binary is compiled in release mode
- Check your hashrate - low rates may indicate CPU throttling

### Low Hashrate

If your hashrate is below 10 MH/s:
- Compile in release mode: `make build`
- Close other CPU-intensive applications
- Check CPU temperature (thermal throttling)
- Consider running on a more powerful machine

### Transaction Failures

- Ensure you have enough XLM for transaction fees (5-10 XLM recommended)
- Verify LaunchTube JWT is valid
- Check RPC endpoint is responding

## Advanced: Contributing to Development

If you want to modify or contribute to the KALE farmer:

### Install Stellar CLI

> https://developers.stellar.org/docs/build/smart-contracts/getting-started/setup

### Add Network Configuration

```bash
stellar network add mainnet \
    --rpc-url "https://mainnet.sorobanrpc.com" \
    --network-passphrase "Public Global Stellar Network ; September 2015" \
    --global
```

### Rebuild Contract Bindings

When the KALE contract is updated, regenerate TypeScript bindings:

```bash
# For mainnet
make bindings-mainnet

# For testnet
make bindings-testnet
```

Then manually copy the generated SDK:

```bash
# Copy generated bindings
cp -r ./bun_scripts/kale-sc-sdk__raw/src/index.ts ./bun_scripts/kale-sc-sdk/src/index.ts

# Rebuild SDK package
cd ./bun_scripts/kale-sc-sdk
bun run build
cd ..
bun install --force
```

## Project Structure

```
├── src/
│   └── main.rs                 # Rust mining engine
├── bun_scripts/
│   ├── plant_and_work.ts       # Main farming orchestrator
│   ├── harvest.ts              # Reward harvesting
│   ├── utils.ts                # Contract utilities
│   ├── console-utils.ts        # Logging helpers
│   └── package.json            # Dependencies
├── ext/
│   └── kale-sc-sdk/            # Smart contract SDK
├── Cargo.toml                  # Rust dependencies
├── Makefile                    # Build automation
└── README.md
```

## Resources

- **KALE Smart Contract**: https://github.com/kalepail/KALE-sc
- **Stellar Docs**: https://developers.stellar.org
- **Soroban Docs**: https://soroban.stellar.org
- **LaunchTube**: https://launchtube.xyz

## License

See the original [KALE-sc repository](https://github.com/kalepail/KALE-sc) for license information.

## Support

For issues or questions:
- Open an issue on GitHub
- Check the KALE smart contract documentation
- Review Stellar/Soroban developer docs
