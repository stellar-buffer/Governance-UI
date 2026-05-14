# Governance UI — Powered by Drips Wave

A Next.js-based governance interface for stellar DAOs, built on top of the [SPL Governance](https://github.com/stellar-labs/stellar-program-library/tree/master/governance) program. This UI enables token holders, NFT communities, and multi-sig participants to create proposals, cast votes, manage treasuries, and interact with a rich ecosystem of voter-weight plugins — all from a single, unified interface.

This project participates in [Drips Wave](https://docs.drips.network/wave/#drips-wave), a recurring open-source bounty program that lets contributors **Fix, Merge, and Earn** on a predictable monthly cycle.

---

## Table of Contents

- [What is Drips Wave?](#what-is-drips-wave)
- [Architecture Overview](#architecture-overview)
- [Project Structure](#project-structure)
- [Voter Weight Plugins](#voter-weight-plugins)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [Available Scripts](#available-scripts)
- [Contributing via Drips Wave](#contributing-via-drips-wave)

---

## What is Drips Wave?

[Drips Wave](https://docs.drips.network/wave/#drips-wave) is a recurring bounty cycle designed to help open-source ecosystems attract developers, accelerate maintenance, and grow their communities. Instead of one-off grants or long-term commitments, Wave creates a predictable rhythm of contribution.

### How a Wave Works

```
┌─────────────────────────────────────────────────────────────────┐
│                        DRIPS WAVE LIFECYCLE                     │
│                                                                 │
│  1. SCOPING          2. SPRINT             3. REWARD            │
│  ┌──────────┐        ┌──────────┐          ┌──────────┐         │
│  │Maintainer│──────▶ │Contributor│────────▶ │ Payout   │        │
│  │tags issues│       │submits PRs│          │ by Points│        │
│  │w/ points │        │& earns pts│          │  share   │        │
│  └──────────┘        └──────────┘          └──────────┘         │
│                                                                 │
│  ◀──────────────── ~1 month cycle ──────────────────▶           │
└─────────────────────────────────────────────────────────────────┘
```

### Point System

| Complexity | Description                                      | Points |
|------------|--------------------------------------------------|--------|
| Trivial    | Typos, minor copy changes, small bug fixes       | 100    |
| Medium     | Standard features or involved bug fixes          | 150    |
| High       | Complex architecture, refactors, new integrations| 200    |

Your payout is proportional to your share of total points earned across all contributors in a given Wave cycle. If the reward pool is $50,000 and you earned 5% of all points, you receive $2,500.

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                          governance-ui                               │
│                                                                      │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────────────────┐  │
│  │  Next.js    │   │   Zustand    │   │   @tanstack/react-query  │  │
│  │  Pages /    │──▶│   Stores     │──▶│   Server State Cache     │  │
│  │  App Router │   │  (UI State)  │   │   (On-chain Data)        │  │
│  └─────────────┘   └──────────────┘   └──────────────────────────┘  │
│         │                                          │                 │
│         ▼                                          ▼                 │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    stellar Web3 Layer                        │    │
│  │                                                             │    │
│  │  @stellar/web3.js  ──▶  RPC Node (Mainnet / Devnet)         │    │
│  │  @coral-xyz/anchor ──▶  Program IDLs                       │    │
│  │  @stellar/spl-governance ──▶  Governance Program            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│         │                                                            │
│         ▼                                                            │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                  Voter Weight Plugin System                 │    │
│  │                                                             │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │    │
│  │  │  VSR     │ │  NFT     │ │  Helium  │ │  Quadratic   │  │    │
│  │  │ (Locked  │ │  Vote    │ │  Vote    │ │  Plugin      │  │    │
│  │  │ Tokens)  │ │  Plugin  │ │  Plugin  │ │              │  │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │    │
│  │  │  Drift   │ │  Pyth    │ │  Parcl   │ │  Gateway     │  │    │
│  │  │  Stake   │ │  Network │ │  Vote    │ │  (Civic)     │  │    │
│  │  │  Voter   │ │  Plugin  │ │  Plugin  │ │  Plugin      │  │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │    │
│  └─────────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────────┘
```

### Data Flow for Proposal Voting

```
User Wallet
    │
    ▼
ConnectWalletButton
    │
    ▼
useRealmVoterWeightPlugins()   ◀── resolves active plugin chain
    │
    ├──▶ VoteStakeRegistry (VSR)   ──▶ locked token multiplier
    ├──▶ NftVotePlugin             ──▶ NFT collection weight
    ├──▶ HeliumVotePlugin          ──▶ sub-DAO position weight
    └──▶ QuadraticPlugin           ──▶ sqrt(tokens) weight
    │
    ▼
castVote(proposal, side)
    │
    ▼
@stellar/spl-governance  ──▶  on-chain vote record
```

---

## Project Structure

```
governance-ui/
├── pages/                    # Next.js routes
│   ├── dao/[symbol]/         # DAO-specific pages (proposals, treasury, members)
│   ├── realms/               # Realm discovery & listing
│   └── api/                  # API routes (notifications, etc.)
│
├── components/               # Shared React components
│   ├── VotePanel/            # Voting UI (cast, relinquish, results)
│   ├── ProposalVotingPower/  # Per-plugin voting power display
│   ├── TreasuryAccount/      # Treasury management UI
│   └── GovernancePower/      # Governance power breakdown
│
├── VoterWeightPlugins/       # Plugin abstraction layer
│   ├── clients/              # Per-plugin client wrappers
│   ├── hooks/                # useVoterWeightPlugins, useVsrClient, etc.
│   └── lib/                  # Shared plugin utilities
│
├── VoteStakeRegistry/        # VSR (locked token voting) plugin
├── NftVotePlugin/            # NFT-based voting plugin
├── HeliumVotePlugin/         # Helium sub-DAO voting plugin
├── DriftStakeVoterPlugin/    # Drift protocol staking plugin
├── PythVotePlugin/           # Pyth Network staking plugin
├── ParclVotePlugin/          # Parcl protocol voting plugin
├── GatewayPlugin/            # Civic identity gateway plugin
├── QuadraticPlugin/          # Quadratic voting plugin
│
├── actions/                  # stellar transaction builders
│   ├── castVote.ts
│   ├── createProposal.ts
│   ├── executeInstructions.ts
│   └── ...
│
├── hooks/                    # Global React hooks
├── stores/                   # Zustand global state
├── models/                   # TypeScript types & account models
└── utils/                    # Helpers, formatting, validation
```

---

## Voter Weight Plugins

The UI supports a composable plugin system where each DAO can configure one or more voter weight plugins. Plugins modify how much voting power a wallet has.

### Vote Stake Registry (VSR)

Tokens locked for longer periods receive a multiplier on their voting power. The longer the lockup, the higher the weight.

```typescript
// hooks/useVsrMode.ts — detect if VSR is active for the current realm
import { useRealmVoterWeightPlugins } from 'hooks/useRealmVoterWeightPlugins'

const { isReady, plugins } = useRealmVoterWeightPlugins()
const vsrActive = plugins?.voterWeight.some(p => p.name === 'VSR')
```

### NFT Vote Plugin

Voting power is derived from NFT ownership within a configured collection. Each NFT counts as one vote (or a configurable weight).

```typescript
// NftVotePlugin/accounts.ts — fetch NFT voter weight accounts
import { getNftVoterWeightRecord } from 'NftVotePlugin/accounts'

const voterWeightRecord = await getNftVoterWeightRecord(
  connection,
  programId,
  realm,
  walletPublicKey
)
```

### Helium Vote Plugin

Positions in Helium sub-DAOs (IOT, MOBILE, HNT) are converted to voting power based on lockup duration and sub-DAO multipliers.

```typescript
// HeliumVotePlugin/utils/calcPositionVotingPower.ts
import { calcPositionVotingPower } from 'HeliumVotePlugin/utils/calcPositionVotingPower'

const power = calcPositionVotingPower({
  position,
  registrar,
  unixNow: Date.now() / 1000,
})
```

### Quadratic Plugin

Voting power is the square root of token balance, reducing the influence of large holders and giving smaller participants a proportionally larger voice.

---

## Getting Started

### Prerequisites

- Node.js `18.19.0` (use [nvm](https://github.com/nvm-sh/nvm) or [volta](https://volta.sh/))
- Yarn `1.22.19`

```bash
# Install the correct Node version
nvm use
# or with volta (auto-detected from package.json)

# Install dependencies
yarn setup
```

### Running the Development Server

```bash
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

### Running Tests

```bash
# Run all tests once
yarn test

# Full CI check (lint + type-check + test)
yarn test-all
```

---

## Configuration

Copy `.env.sample` to `.env.local` and fill in the values:

```bash
cp .env.sample .env.local
```

| Variable                              | Description                                      | Default                              |
|---------------------------------------|--------------------------------------------------|--------------------------------------|
| `MAINNET_RPC`                         | Mainnet RPC endpoint                             | `https://mango.rpcpool.com`          |
| `DEVNET_RPC`                          | Devnet RPC endpoint                              | `https://mango.devnet.rpcpool.com`   |
| `DEFAULT_GOVERNANCE_PROGRAM_ID`       | Override the default SPL Governance program ID   | Devnet test program                  |
| `NEXT_PUBLIC_JUPTER_SWAP_API_ENDPOINT`| Jupiter V6 Swap API (self-hosted or paid)        | `https://quote-api.jup.ag/v6`        |
| `NEXT_PUBLIC_API_ENDPOINT`            | Realms GraphQL API                               | `https://api.realms.today/graphql`   |
| `NEXT_PUBLIC_HELIUS_MAINNET_RPC`      | Helius RPC for compressed NFT support            | —                                    |
| `NEXT_PUBLIC_HELIUS_DEVNET_RPC`       | Helius Devnet RPC                                | —                                    |

### Custom Jupiter Swap Endpoint

You can point the UI at a self-hosted [Jupiter V6 Swap API](https://station.jup.ag/docs/apis/self-hosted) or a paid hosted endpoint:

```bash
NEXT_PUBLIC_JUPTER_SWAP_API_ENDPOINT=https://your-custom-jupiter-endpoint.com/v6
```

---

## Available Scripts

| Script              | Description                                          |
|---------------------|------------------------------------------------------|
| `yarn dev`          | Start the Next.js development server                 |
| `yarn build`        | Build for production                                 |
| `yarn start`        | Start the production server                          |
| `yarn type-check`   | Run TypeScript type checking                         |
| `yarn lint`         | Run ESLint across all `.ts`, `.tsx`, `.js` files     |
| `yarn format`       | Auto-format with Prettier                            |
| `yarn test`         | Run Jest test suite                                  |
| `yarn test-all`     | Lint + type-check + test (full CI pipeline)          |
| `yarn analyze`      | Build with bundle analyzer enabled                   |
| `yarn create-proposal` | CLI script to create a proposal programmatically |

---

## Contributing via Drips Wave

This repository participates in [Drips Wave](https://docs.drips.network/wave/#drips-wave). Wave is a monthly open-source sprint where contributors earn rewards for merged pull requests.

### How to Participate

1. **Browse open issues** tagged with a Wave label at [drips.network/wave](https://www.drips.network/wave).
2. **Log in with GitHub** and apply to work on an issue that interests you.
3. **Get assigned** — a maintainer will review your profile and assign you to the issue.
4. **Submit a PR** — fix the issue and open a pull request against this repository.
5. **Earn points** — once your PR is merged and the issue is marked resolved, you earn points toward the Wave reward pool.

### Reward Distribution

At the end of each Wave cycle, the total reward budget is split proportionally among all contributors based on their share of points earned:

```
Your Reward = (Your Points / Total Points in Wave) × Total Reward Pool
```

### Issue Complexity Guide for This Repo

| Complexity | Examples in this codebase                                              |
|------------|------------------------------------------------------------------------|
| Trivial    | Fix a typo in a component, correct a broken link, update a constant    |
| Medium     | Add a missing loading state, fix a hook dependency array, improve a11y |
| High       | Implement a new voter weight plugin, refactor a store, add a new page  |

### Development Workflow

```bash
# 1. Fork and clone the repo
git clone https://github.com/your-fork/governance-ui.git
cd governance-ui

# 2. Install dependencies
yarn setup

# 3. Create a feature branch
git checkout -b fix/your-issue-description

# 4. Make your changes, then verify
yarn test-all

# 5. Push and open a PR
git push origin fix/your-issue-description
```

> The maintained version of this project lives at [Mythic-Project/governance-ui](https://github.com/Mythic-Project/governance-ui).

---

## License

[MIT](./LICENSE) — stellar Maintainers
