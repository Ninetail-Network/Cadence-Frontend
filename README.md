# Cadence Frontend

This is the user-facing app for **Cadence**, a Stellar-native platform where
fans send creators anonymous messages, creators can respond by text or by
having an AI voice agent call the fan back, and both sides build a daily
on-chain check-in streak that pays out rewards. Full project context lives
in the [top-level README](../README.md).

This app is the only place a real person actually interacts with Cadence —
everything the contracts and backend do (escrow, streak tracking, EchoCall
orchestration) is invisible unless it's surfaced here clearly: an honest
anonymous inbox for fans, a manageable queue and one-click voice-call
trigger for creators, and a streak view simple enough that checking in
becomes a genuine daily habit rather than a chore.

Next.js application with two surfaces sharing one codebase: the **Fan
Portal** (send anonymous messages, check in daily) and the **Creator
Dashboard** (manage inbox, trigger EchoCall, track streak/monetization).

## Stack

- **Framework:** Next.js (App Router), TypeScript, Tailwind CSS
- **Wallets:** Freighter, Albedo, xBull — via `@stellar/wallets-kit` (or
  `@creit.tech/stellar-wallets-kit`)
- **Data:** React Query for API calls, WebSocket subscription for live inbox
- **Stellar:** `@stellar/stellar-sdk` in the browser for building/signing
  transactions before sending to the backend for submission

## Folder structure

```
frontend/
├── app/
│   ├── (fan)/
│   │   ├── page.tsx              # landing + wallet connect
│   │   ├── send/page.tsx         # compose an anonymous Veil message
│   │   └── pulse/page.tsx      # fan's own streak calendar + check-in button
│   ├── (creator)/
│   │   ├── dashboard/page.tsx    # inbox, filters (pending/answered/published)
│   │   ├── message/[id]/page.tsx # message detail, reply, trigger EchoCall
│   │   ├── calls/page.tsx        # EchoCall history + transcripts
│   │   └── settings/page.tsx     # unlock pricing, subscription tiers
│   └── layout.tsx
├── components/
│   ├── WalletConnectButton.tsx
│   ├── StreakCalendar.tsx        # visual streak/Pulse heatmap
│   ├── MessageCard.tsx
│   ├── EchoCallModal.tsx         # prompt input + live call status
│   └── UnlockPaywall.tsx
├── lib/
│   ├── stellar.ts                # tx-building helpers, wallet-kit wrapper
│   ├── api.ts                    # typed fetch wrapper for backend
│   └── hooks/
│       ├── useStreak.ts
│       ├── useInbox.ts
│       └── useCall.ts
└── package.json
```

## Key screens

### Fan Portal
- **Send a message** — write anonymously, pick a creator, optionally attach
  an unlock payment (amount + asset picker: XLM or USDC), sign with wallet,
  submit.
- **Pulse** — a GitHub-style streak heatmap; one-tap "Check in today"
  button that builds and signs the `pulse.checkin` transaction.

### Creator Dashboard
- **Inbox** — paginated list of pending/answered/published messages, sorted
  optionally by fan streak length (reward loyal fans with faster answers).
- **Message detail** — reply by text (releases escrow) or **Start
  EchoCall**: a modal where the creator types a plain-English prompt (mirrors
  Phone Claw's UX — phone number + instructions) and watches call status
  live, then reviews the transcript once done.
- **Pulse** — the creator's own streak, plus aggregate fan engagement
  stats (active streaks, milestone payouts issued).
- **Settings** — set per-message unlock price, subscription tiers, and
  streak-based perks (e.g., "fans with 7+ day streaks skip the paywall").

## Environment variables

```
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_STELLAR_NETWORK=testnet
NEXT_PUBLIC_SOROBAN_RPC_URL=https://soroban-testnet.stellar.org
NEXT_PUBLIC_VEIL_ESCROW_CONTRACT_ID=...
NEXT_PUBLIC_PULSE_CONTRACT_ID=...
NEXT_PUBLIC_VEIL_TOKEN_CONTRACT_ID=...
```

## Local development

```bash
cd frontend
npm install
npm run dev
```

## Design notes

- Wallet signing happens **client-side only** — private keys never touch
  the backend. The frontend builds an unsigned XDR transaction, the wallet
  extension signs it, and the signed XDR is what gets POSTed to the backend
  for submission to Soroban RPC.
- The `StreakCalendar` component is shared between fan and creator views —
  same data shape (`StreakInfo` from the contract), different framing copy.
- EchoCall's prompt box intentionally mirrors Phone Claw's plain-English
  style: a phone number field + a free-text instruction box, so creators
  don't need to think in "scripts," just intent.
