---
name: coinone-cli
description: Use the Coinone CLI to query Coinone public and private APIs from the command line, including guarded live order placement when explicitly requested. Requires coinone-api-cli installed globally.
---

Use this skill when the user wants to work with Coinone exchange data through the CLI.

## When to Use This Skill

Activate this skill when the user wants to:

- call Coinone public APIs from the command line
- inspect safe read-only private API data with env-based auth
- place or cancel guarded orders when the user explicitly asks for a real trading action
- script Coinone queries for developers or AI agents
- diagnose installation or runtime problems with the `coinone` CLI

## Installation

```bash
npm install -g coinone-api-cli
coinone --help
coinone doctor --json
```

Homebrew:

```bash
brew tap 2sem/tap
brew install coinone
coinone doctor --json
```

## Core Usage Rules

- Use `--json` for agent and automation workflows.
- Never print, hardcode, or request private credentials in normal output.
- Private auth must come from environment variables:
  - `COINONE_ACCESS_TOKEN`
  - `COINONE_SECRET_KEY`
- Use `--timeout <ms>` in automation to fail fast.

## Supported Command Surface

### Local diagnostics

- `coinone doctor`

### Public

- `coinone markets list`
- `coinone markets get <targetCurrency> --quote <quoteCurrency>`
- `coinone currencies list`
- `coinone currencies get <currency>`
- `coinone ticker get <targetCurrency> --quote <quoteCurrency>`
- `coinone ticker list [--quote <quoteCurrency>]`
- `coinone orderbook get <targetCurrency> --quote <quoteCurrency> [--size <n>]`
- `coinone trades list <targetCurrency> --quote <quoteCurrency> [--size <n>]`
- `coinone range-units get <targetCurrency> --quote <quoteCurrency>`

### Private read-only

- `coinone auth status`
- `coinone balances list`
- `coinone balances get <currency>`
- `coinone fees list`
- `coinone fees get --quote <quoteCurrency> --target <targetCurrency>`
- `coinone orders active [--quote <quoteCurrency>] [--target <targetCurrency>] [--type <type>]`
- `coinone orders get <orderId> --quote <quoteCurrency> --target <targetCurrency> [--user-order-id <id>]`
- `coinone orders completed --from <timestamp-ms|iso> --to <timestamp-ms|iso> [--size <1-100>] [--to-trade-id <id>] [--quote <quoteCurrency> --target <targetCurrency>]`

### Guarded private writes

- `coinone orders place --quote <quoteCurrency> --target <targetCurrency> --side <buy|sell> --type limit --price <string> --qty <string> [--post-only] [--user-order-id <id>] (--dry-run | --confirm live)`
- `coinone orders cancel --order-id <id> --quote <quoteCurrency> --target <targetCurrency> [--user-order-id <id>] --confirm live`

## Recommended Patterns

### Public market data

```bash
coinone ticker get btc --quote krw --json
coinone markets list --json
coinone orderbook get btc --quote krw --size 10 --json
```

### Private auth check

```bash
export COINONE_ACCESS_TOKEN="your-access-token"
export COINONE_SECRET_KEY="your-secret-key"
coinone doctor --json
coinone auth status --json
```

### Private read-only examples

```bash
coinone balances list --json
coinone fees get --quote krw --target btc --json
coinone orders completed --from 2026-01-01T00:00:00Z --to 2026-01-02T00:00:00Z --json
```

### Guarded write sequence

```bash
coinone doctor --json
coinone markets get usdc --quote krw --json
coinone orders place --quote krw --target usdc --side buy --type limit --price 1519 --qty 4 --dry-run --json
coinone orders place --quote krw --target usdc --side buy --type limit --price 1519 --qty 4 --confirm live --json
coinone orders get <orderId> --quote krw --target usdc --json
coinone orders cancel --order-id <orderId> --quote krw --target usdc --confirm live --json
```

### Update

```bash
npm install -g coinone-api-cli

# Homebrew
brew update && brew upgrade coinone
```

## Safety and Validation

- Never place or cancel a real order unless the user explicitly requests it.
- For trading actions, always run this sequence unless the user explicitly says otherwise:
  1. `doctor`
  2. optional `markets get` to inspect pair metadata
  3. `orders place --dry-run`
  4. only then `orders place --confirm live`
  5. `orders get` or `orders active`
  6. `orders cancel --confirm live` if cleanup is needed

## Common Failure Cases

- Missing private env vars → run `coinone doctor --json`
- `coinone fees ...` returns `Invalid API permission` (code `40`) → enable **고객 정보** permission on the Coinone API key
- `coinone` not found after global install → compare `npm bin -g` with shell `PATH`
- Homebrew install issues → run `brew tap 2sem/tap && brew reinstall coinone`
- Timeout/network failures → retry with `--timeout <ms>` adjusted upward
- `orders completed` window error → ensure `--from <= --to` and window ≤ 90 days
- Coinone `107 Parameter error` on live order → rerun `markets get` for the pair and verify min/max constraints
