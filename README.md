<p align="center">
  <img src="./assets/logo.png" alt="Pump Pool logo" width="160">
</p>

<h1 align="center">Pump Pool</h1>

<p align="center">
  The first big-liquidity memecoin launch on Pump.fun — powered by the new Pump.fun × Meteora pool API.
</p>

<p align="center">
  <a href="https://github.com/PumpPools/Pump-Pool">GitHub</a> ·
  <a href="https://pump.fun">Pump.fun</a> ·
  <a href="https://meteora.ag">Meteora</a>
</p>

---

## What is Pump Pool?

**Pump Pool** is a Solana project built on top of the Pump.fun bonding-curve protocol, extended with **Meteora** liquidity infrastructure. It lets creators launch memecoins on Pump.fun and graduate them into **real, deep liquidity pools** — programmatically, through the official APIs.

On **2 July 2026**, Pump.fun and Meteora updated their APIs to let developers **create liquidity pools directly** — no manual UI steps, no workarounds, no duct-taped scripts. Pool creation, migration, and LP management can now be driven end-to-end from code.

We used that update on day one.

**Pump Pool is the first project to launch a big-liquidity memecoin on Pump.fun using the new API integration.**

---

## Why this matters

Before this API update, launching on Pump.fun meant:

1. Create a token on the bonding curve
2. Wait for the curve to fill
3. Hope migration to a DEX worked cleanly
4. Manually seed liquidity somewhere else if you wanted depth

Now Pump.fun and Meteora expose pool creation **directly through their API layer**. That means:

- **Programmatic pool deployment** — create and configure pools from your own stack
- **Meteora-native liquidity** — graduated tokens land in Meteora AMM infrastructure, not a black box
- **Composable launches** — bonding curve → migration → locked LP in one pipeline
- **Builder-friendly** — TypeScript/Rust clients, Anchor programs, and CPI hooks for automation

Pump Pool sits at the intersection of all of this: a reference implementation and a live proof that the new flow works at scale.

---

## How it works

### 1. Bonding curve (Pump.fun)

Tokens launch on a **constant-product bonding curve** (`x × y = k`) with standard Pump.fun parameters:

| Parameter | Value |
|---|---|
| Initial virtual token reserves | 1,073,000,000,000,000 |
| Initial virtual SOL reserves | 15,000,000,000 |
| Initial real token reserves | 793,100,000,000,000 |
| Total token supply | 1,000,000,000,000,000 |

Trading happens on-curve until the migration threshold is hit.

### 2. Migration (Meteora)

When the bonding curve completes (~42.5 SOL raised, supply exhausted):

1. Remaining supply is minted
2. Protocol fees route to the multisig
3. A **CPI call creates a Meteora Dynamic AMM pool** with matched token + SOL liquidity
4. LP is **locked** with claim authority assigned to the protocol multisig

### 3. Direct pool API (new — July 2026)

The updated Pump.fun / Meteora API now exposes **direct pool creation endpoints** that previously required UI interaction or undocumented flows. Pump Pool uses this to:

- Initialize pools with custom config
- Seed liquidity programmatically
- Lock LP and assign fee claim authority
- Migrate from bonding curve to AMM in a single orchestrated pipeline

---

## Project structure

```
pump-pool/
├── assets/              # Brand assets (logo)
├── programs/pump-fun/   # Anchor smart contract (bonding curve + Meteora migration)
├── clients/
│   ├── js/              # TypeScript SDK + generated Kinobi client
│   └── rust/            # Rust client
├── idls/                # Program IDL
├── tests/               # Integration tests (Mocha + Bankrun)
├── migrations/          # Deployment scripts
└── configs/             # Validator, build, and codegen configs
```

---

## Core features

- **Bonding curve trading** — buy/sell along the Pump.fun curve until graduation
- **Automated Meteora migration** — CPI into Meteora Dynamic AMM on curve completion
- **LP locking** — liquidity locked at migration with escrowed fee claims
- **Admin controls** — parameter updates, fee withdrawal, whitelist support
- **JS + Rust clients** — generated from IDL via Kinobi for type-safe integration

---

## Getting started

### Prerequisites

```
anchor  : v0.31.0
solana  : v1.18.18
rustc   : v1.75.0
pnpm    : v9.x
```

### Install

```bash
git clone https://github.com/PumpPools/Pump-Pool.git
cd Pump-Pool
pnpm install
```

### Build & test

```bash
pnpm programs:build
pnpm test
```

### Configure cluster

Update `Anchor.toml`:

```toml
[provider]
cluster = "https://api.devnet.solana.com"
wallet = "~/.config/solana/id.json"
```

---

## On-chain proof

| | |
|---|---|
| **Mainnet CA** | [`9MHPjXpZXgJrB4NiJVFStE5qy7Nqp7yaYpaqNe5jNfMw`](https://solscan.io/account/9MHPjXpZXgJrB4NiJVFStE5qy7Nqp7yaYpaqNe5jNfMw) |
| **Devnet CA** | [`9MHPjXpZXgJrB4NiJVFStE5qy7Nqp7yaYpaqNe5jNfMw`](https://solscan.io/account/9MHPjXpZXgJrB4NiJVFStE5qy7Nqp7yaYpaqNe5jNfMw?cluster=devnet) |
| **Create bonding curve** | [`32JZPj…aM2S`](https://solscan.io/tx/32JZPjV9JAakkvJyqQMpFhW8ziNqZV3QN2jbsY1W6GRsuDtq69HPtXsQE1RpAyUE2rw1DW5usozh4oueWqb1aM2S?cluster=devnet) |
| **Swap (with fees)** | [`TrzFL5…wi8`](https://solscan.io/tx/TrzFL5aMu6vGEdFHhpHiDQhYrCYEmo4swSDLrN7avxSQwP3LrweSMPFTDZYmVYMwVCsTHSKWRxJkt9xXboQAwi8?cluster=devnet) |
| **Meteora migration** | [`496nHg…nUVU`](https://solscan.io/tx/496nHgdFz5QYR4ah1pTsmnkQJSMiF8Z3FaJhSNgQKaHp5UFL1rQekdEoUG7vWRozSKg3YHJivQsHZk7b8TxfnUVU?cluster=devnet) |

**Protocol multisig:** `Br4NUsLoHRgAcxTBsDwgnejnjqMe5bkyio1YCrM3gWM2`

---

## API integration

Pump Pool is built to consume the **updated Pump.fun SDK** and **Meteora DLMM / Dynamic AMM SDKs** for:

| Operation | Stack |
|---|---|
| Token creation | Pump.fun `create` / `create_v2` |
| Curve trading | Pump.fun bonding curve program |
| Pool creation | Meteora Dynamic AMM (via new direct API) |
| LP management | Meteora pool + lock CPI |
| Fee routing | Protocol multisig + fee vault |

If you're building on the new API, this repo is the reference implementation.

---

## Links

- **Repo:** [github.com/PumpPools/Pump-Pool](https://github.com/PumpPools/Pump-Pool)
- **Pump.fun docs:** [github.com/PumpPools/pump-public-docs](https://github.com/PumpPools/pump-public-docs)
- **Meteora:** [meteora.ag](https://meteora.ag)

---

## License

See repository for license details.
