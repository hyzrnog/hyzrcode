# HYZR — Perps Terminal

Hyperliquid-style cross-asset perpetuals trading terminal.

**Stack:** Next.js 16 (App Router) · React 19 · TypeScript · Tailwind CSS 4 · Prisma + SQLite · bun

**State:** includes everything through `fx24` — full wallet logic (multi-wallet pairing, per-wallet
persisted trading accounts, live on-chain balances, signature ownership verification, and
Hyperliquid agent approval for live trading). See `worklog.md` for the complete fx0 → fx24 history.

---

## Quick start

```bash
bun install            # or: npm install

# 1) Point Prisma at the bundled SQLite db (relative to prisma/schema.prisma)
#    .env ships with an absolute path from the original machine — edit it:
#    DATABASE_URL=file:../db/custom.db

bunx prisma generate   # regenerates the Prisma client
bun run dev            # dev server on http://localhost:3000
```

`db/custom.db` is included (tracker watches + demo data). If you'd rather start from an empty
database, delete it and run `bun run db:push`.

## Production build

```bash
bash scripts/build-prod.sh
```

This is the canonical prod script used throughout the project: builds the standalone output,
syncs `public/` static assets into it, kills whatever holds port 3000, and restarts the
standalone server. First paint of charts/candles backfills for ~35–50s after a cold start.

## Wallet logic (fx24) — what's implemented

- **Pairing:** Phantom, Solflare, MetaMask and Coinbase Wallet, each via its REAL injected
  provider (`window.phantom.solana`, `window.solflare`, EIP-6963 for MetaMask/Coinbase), with
  install-link fallback, reject/pending error mapping, live account-change tracking and
  persistence across reloads.
- **Multiple wallets:** every paired wallet is saved to `hyzr-wallet`; switch the ACTIVE wallet
  from the Fund modal manager, the WalletMenu strip, or the footer "Active wallets" popover.
- **Per-wallet trading accounts:** balance, positions, orders, fills, leverage, margin mode and
  PnL persist per address under `hyzr-perps-account:<address>` (no wallet paired = the legacy
  demo account). The whole terminal — trade panel, DOM, portfolio, footer PnL, TopNav pill —
  reads/writes the active wallet's account.
- **Live balances:** SOL (public RPC JSON-RPC) and ETH/USDC (via the wallet's own injected
  provider on mainnet/Base/BNB/Optimism/Polygon) poll every 30s for every paired wallet.
- **Ownership verification:** real signature challenge (Solana `signMessage` / EVM `personal_sign`),
  verified shield persists in the wallet list.
- **Live trading:** "Approve & Go Live" in Settings generates an agent keypair in-browser and
  gets it approved by the connected EVM wallet through the real Hyperliquid EIP-712
  `AgentApprove` flow (`eth_signTypedData_v4` → `/api/approveAgent`), then orders sign locally
  and go straight to `/exchange`. Solana wallets can't approve HL agents (EVM signature
  required) — the UI explains this inline.

## QA

Playwright scripts live in `scripts/qa_fx*.ts` (run with `bun`). `scripts/build-prod.sh` +
`scripts/qa_fx24_wallet_logic.ts` are the latest pair. QA screenshots (not included in this
archive) were rendered to `qa-shots/`.

## Not included in this archive

`node_modules/`, `.next/`, `.git/` (185MB of scaffold history), `qa-shots/`, `skills/`,
`upload/`, `tool-results/`, runtime icon/tracker caches, `tsconfig.tsbuildinfo`.
