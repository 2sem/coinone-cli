# coinone-cli

Claude Code plugin marketplace for [coinone-api-cli](https://github.com/2sem/coinone-api-cli) — a developer-friendly CLI for the Coinone cryptocurrency exchange APIs.

## Install

```
/plugin marketplace add 2sem/coinone-cli
```

## Skills

### coinone-cli

Query Coinone market data, inspect balances, and place guarded orders from the command line.

**Requires:** `coinone-api-cli` installed globally

```bash
npm install -g coinone-api-cli
coinone doctor --json
```

Or via Homebrew:

```bash
brew tap 2sem/tap
brew install coinone
```

## Skill Auto-Sync

`coinone-cli/SKILL.md` is synced weekly from [2sem/coinone-api-cli](https://github.com/2sem/coinone-api-cli/tree/main/skills/coinone-api-cli) via GitHub Actions.

## Source

- CLI source: [2sem/coinone-api-cli](https://github.com/2sem/coinone-api-cli)
- npm: [`coinone-api-cli`](https://www.npmjs.com/package/coinone-api-cli)
