# PM2 Process Manager

PM2 keeps your farmer running 24/7 with automatic restarts on crashes.

## Why Use PM2?

- Auto-restarts on crash (network errors, LaunchTube issues, etc.)
- Runs in background, survives SSH disconnects
- Memory limits (500MB) prevent runaway processes
- Built-in logging and monitoring

## Setup

Install PM2:
```bash
npm install -g pm2
```

Or use without installing:
```bash
npx pm2 <command>
```

## Usage

From the `bun_scripts/` directory:

```bash
# Start
bun run pm2:start              # mainnet
bun run pm2:start:testnet      # testnet
bun run pm2:start:futurenet    # futurenet

# Monitor
bun run pm2:status             # check status
bun run pm2:logs               # view logs
bun run pm2:monit              # resource usage

# Control
bun run pm2:restart            # restart
bun run pm2:stop               # stop
bun run pm2:delete             # remove from PM2
```

## Auto-start on Boot

```bash
pm2 startup
pm2 save
```

## Logs

Logs are in `bun_scripts/pm2/logs/`:
- `kale-farmer-out.log` - farming activity
- `kale-farmer-error.log` - errors

View directly:
```bash
tail -f pm2/logs/kale-farmer-out.log
```

## Troubleshooting

**Process keeps restarting?**
```bash
bun run pm2:logs  # check what's failing
```

**Already running?**
```bash
bun run pm2:delete  # remove and restart
```

Common issues:
- Invalid LaunchTube JWT
- Not enough XLM for fees
- RPC endpoint down

## Resources

- [PM2 Docs](https://pm2.keymetrics.io/docs/usage/quick-start/)
