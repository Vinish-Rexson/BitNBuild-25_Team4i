# 🔖 Proof of Presence — Hackathon Pitch

> *"Every moment deserves a permanent, trustless proof."*

---

## 📌 Executive Summary

**Proof of Presence** is a decentralized attendance verification dApp built on the Solana blockchain. It lets event organizers issue cryptographically verifiable, photo-backed NFTs to their attendees — instantly, gaslessly, and with metadata stored permanently on Arweave. No app download. No gas fees for attendees. No disappearing links.

---

## 🚨 The Problem (In Depth)

Modern event attendance systems are broken in multiple fundamental ways:

### 1. Attendance Records Are Centralized and Fragile
Today, organizers track attendance through Google Sheets, Eventbrite exports, proprietary platforms, or paper sign-in sheets. All of these share a fatal flaw: **a single point of failure**. If the company shuts down, the spreadsheet is deleted, or the platform terms change, your proof of attendance is gone. There is no independent, tamper-proof source of truth.

### 2. Attendees Own Nothing
When someone attends a conference, a hackathon, a community meetup, or an online webinar, they walk away with nothing verifiable. A LinkedIn post or a scanned PDF of a certificate is easily fabricated. There is no standard way to say: *"I was here, and here is cryptographic proof that cannot be faked."*

### 3. Ticket Duplication and Fraud Are Rampant
QR code tickets are regularly screenshotted and shared. Barcodes are duplicated. Manual sign-in sheets are filled in by proxies. Without an on-chain single-use claim mechanism, there is no reliable way to enforce that each physical seat maps to exactly one verified attendee.

### 4. NFT Metadata Is Not Truly Permanent
Many NFT projects store images and metadata on IPFS pinning services that expire when the subscription lapses. The result is "rug metadata" — NFTs whose images break and whose descriptions disappear, rendering them worthless. This is a widespread problem even for reputable projects.

### 5. Gas Fees Create Friction and Exclusion
Even where blockchain-based attendance exists (e.g., POAP on Ethereum), attendees are often required to hold ETH or native tokens to pay for minting. This creates a significant adoption barrier for non-crypto-native audiences who just want to prove they attended an event.

### 6. No Photo-Linked Identity
Existing attendance NFTs are generic images identical for every holder. They carry no personal, moment-specific evidence. There is no way to tell if a token was actually claimed in-person versus gifted or transferred.

---

## 💡 The Solution

Proof of Presence solves every one of these problems in a single, cohesive system:

- **On-chain immutability**: Every claim is recorded as an NFT transaction on Solana — public, permanent, and tamper-proof.
- **Truly permanent storage**: Every selfie and metadata JSON is uploaded to Arweave via Bundlr with a one-time payment, meaning it survives indefinitely without any recurring cost.
- **Single-use claim codes**: Each seat gets a unique code that can be claimed exactly once, preventing duplication.
- **Zero gas for attendees**: The organizer wallet co-signs as the fee payer, so attendees — even those new to crypto — can claim with no SOL balance.
- **Personal photo proof**: Every NFT is tied to a live selfie taken at claim time, creating irrefutable moment-specific evidence.
- **Grouped under event collections**: All attendee NFTs are verified members of the event's Metaplex Collection, so wallets and explorers automatically group them.

---

## ✨ Features — Detailed Breakdown

### 📸 In-Browser Camera Capture
The attendee's device camera opens directly in the browser using the Web `MediaDevices.getUserMedia` API — no app download, no third-party camera software needed. A live preview stream is shown, and the attendee can capture a snapshot and review it before proceeding. A **retake button** lets them reshoot if the image is blurry or poorly framed. The final snapshot is encoded as a base64 PNG and sent securely to the server for upload.

---

### 🖼️ Permanent Arweave Storage via Bundlr
Once captured, the snapshot image is uploaded to **Arweave** — a blockchain-based permanent data storage network — using the **Bundlr (Irys) uploader** integrated via Metaplex Umi. Unlike IPFS free tiers where pins can expire, Arweave stores data permanently with a single upfront payment in SOL or USDC. A corresponding **metadata JSON** (containing name, description, and the Arweave image URI) is also uploaded and pinned permanently. The resulting metadata URI is what gets embedded in the NFT on-chain.

---

### 🏆 Metaplex Collection NFTs
Each event is represented by a **Collection NFT** minted using the Metaplex Token Metadata standard. When attendees claim their individual NFTs, each one is **verified as a member of that event's Collection**. This means:
- Explorers like Solscan and wallets like Phantom automatically group all NFTs from the same event.
- The organizer has cryptographic ownership of the collection and can rotate or revoke authority.
- Attendees see properly branded, grouped proof of their attendance history across multiple events.

---

### ⛽ Gasless Attendee Experience (Organizer Pays)
The minting transaction is built as a **Versioned Transaction** where the organizer's server wallet is set as the **fee payer**. The backend partially signs the transaction with the mint authority and collection authority, then sends it to the attendee for signature. The attendee signs with their wallet (only a 0-lamport self-transfer is required to enforce their signature — costing them nothing). The server then adds the fee-payer signature and broadcasts. **Attendees need zero SOL balance to claim their NFT.**

---

### 🔐 Claim Code System (Reservation + Finalize)
Each seat or attendee slot is assigned a **unique, single-use claim code**. Claim codes are stored in a Supabase database with a status of `unused`, `reserved`, or `claimed`.

The two-step flow prevents race conditions:
1. **Reserve** — When a claim starts, the code is immediately locked as `reserved`. No other session can use the same code simultaneously.
2. **Finalize** — Only after the transaction is confirmed on-chain does the code move to `claimed`, storing the wallet address and transaction signature as an audit trail.

This prevents double-spending, claim sniping, and orphaned mints.

---

### 🔍 Public Verification Portal
Anyone can verify a proof of attendance through a public portal by entering:
- A **claim code** — shows the event, wallet, and transaction signature linked to it.
- A **wallet address** — lists all events the wallet has claimed.
- A **transaction signature** — fetches on-chain confirmation and the associated claim record.

This makes proofs fully independently verifiable by third parties — employers, DAOs, or anyone who needs to confirm attendance without trusting a central authority.

---

### 🗂️ Organizer Dashboard
The organizer-facing interface at `/organizer` allows event managers to:
- **Create events** by entering a name and description. The backend automatically mints a Collection NFT and stores the event record in the database.
- **Generate claim codes** in bulk (1–100 per request). Each code is a unique UUID inserted into the `claims` table with `status: unused`.
- **Download or distribute** the codes as a list or generate QR codes pointing to `/claim/<code>` for printing or sharing.
- **Monitor claim status** to see in real time which codes are unused, reserved, or fully claimed.

---

### 🔄 Multi-Signer Versioned Transaction Architecture
The minting flow relies on Solana **Versioned Transactions** with **multiple partial signers**. This is a sophisticated transaction design that allows:
- The **mint keypair** (generated server-side) to sign as the NFT mint authority.
- The **collection authority** (organizer-controlled keypair) to verify the NFT's collection membership.
- The **attendee's wallet** to sign, providing on-chain proof that the holder personally initiated the claim.
- The **fee payer wallet** to pay all transaction costs without the user needing any SOL.

All signatures are aggregated server-side before broadcast, ensuring atomicity.

---

### ♻️ Live Claim Page Flow (`/claim/[code]`)
The attendee-facing claim page is a guided, single-page experience:
1. Loads the event metadata from Supabase using the scanned claim code.
2. Prompts the attendee to connect their Solana wallet using the Wallet Adapter UI (supports Phantom, Backpack, Solflare, etc.).
3. Opens the device camera for a live selfie capture.
4. Uploads the image and metadata to Arweave and returns a permanent URI.
5. Builds the versioned mint transaction on the server and sends it back partially signed.
6. The attendee signs in their wallet — a frictionless, native experience with no fee prompt.
7. The server finalizes, broadcasts, and confirms the transaction.
8. Displays a success screen with: the minted NFT thumbnail, the Solana Explorer link for the transaction, and a shareable link to the verification portal.

---

## 🛠️ Tech Stack — Explained

| Layer | Technology | Why |
|---|---|---|
| **Frontend Framework** | [Next.js 14](https://nextjs.org/) (TypeScript, App Router) | Full-stack React with API routes, SSR, and TypeScript safety |
| **Blockchain** | [Solana](https://solana.com/) | Sub-second finality, ~$0.00025/tx, ideal for mass minting |
| **NFT Standard** | [Metaplex Umi](https://developers.metaplex.com/) + `mpl-token-metadata` | Industry-standard NFT creation and collection management on Solana |
| **Permanent Storage** | [Arweave](https://www.arweave.org/) via [Bundlr/Irys](https://bundlr.network/) | One-time fee, truly permanent storage — no pin expiry, no IPFS risk |
| **Wallet Integration** | [`@solana/wallet-adapter`](https://github.com/anza-xyz/wallet-adapter) | Plug-and-play support for all major Solana wallets |
| **Database** | [Supabase](https://supabase.com/) (PostgreSQL) | Managed Postgres for storing events, claim codes, and audit trails |
| **Camera** | Browser `Navigator.mediaDevices.getUserMedia` | Native web API — no dependencies or plugins needed |
| **Transactions** | Solana Versioned Transactions | Enables multiple partial signers, essential for gasless co-signing flow |

---

## 🌍 Use Cases

| Audience | Use Case |
|---|---|
| **Hackathon organizers** | Mint provable participation NFTs for every project presenter |
| **Tech conferences** | Distribute session-specific attendance badges to verified attendees |
| **DAOs & Web3 communities** | Prove IRL presence for governance gating or token airdrops |
| **Universities & bootcamps** | Issue verifiable attendance credentials for courses and workshops |
| **Corporate events & summits** | Replace paper badges with wallet-native digital identities |
| **Music & arts festivals** | Give fans a collectible, photo-linked memory of the experience |

---

## 🚀 Why We Win Against the Competition

| Compared To | Our Advantage |
|---|---|
| **POAP (Ethereum)** | Solana = 10,000× cheaper transactions + sub-second confirmation |
| **Google Forms / Eventbrite** | Trustless on-chain records vs. centralized, deletable databases |
| **Generic IPFS NFTs** | Arweave = permanent metadata, zero risk of "rug metadata" |
| **Whitelist-based airdrops** | Attendees claim themselves — no manual wallet collection needed |
| **Traditional certificates** | NFTs are wallet-native, verifiable on-chain, and can't be faked |

---

## 📈 Roadmap — What's Next

- [ ] **AI-generated artwork** — Use the selfie to generate a unique, stylized NFT image via a generative AI model
- [ ] **Organizer analytics dashboard** — Claim rates, wallet demographics, geographic heatmaps
- [ ] **Dynamic NFTs** — NFTs that update their image or metadata post-event (e.g., adding a speaker's signature)
- [ ] **Anchor on-chain program** — Custom Solana program for advanced on-chain validation and event logic
- [ ] **Supabase RLS** — Row Level Security policies to harden the database before production
- [ ] **Mobile PWA** — Progressive Web App support for a native-feel mobile experience
- [ ] **Multi-chain expansion** — Extend to Ethereum L2s (Base, Polygon) for wider ecosystem reach
- [ ] **Token-gated RSVP** — Require an existing NFT or token to unlock a claim code (exclusive events)

---

## 👥 The Team

*Built at **BitNBuild '25** by **Team 4i***

---

> *"Attendance is a moment. Your proof should last forever."*
