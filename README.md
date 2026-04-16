# LIMBO v2
### *Your contract won't leave the same.*

Limbo is a premium Telegram bot that audits smart contracts automatically. Protocols submit code or a ZIP, pay USDT, and receive a dark premium PDF audit report — no humans, no waiting, fully automated.

--- 

## What's New In v2

- **ZIP upload** — audit entire protocol repos, not just single files
- **Streaming extractor** — skips node_modules to prevent OOM on free tier
- **Correlation engine** — cross-validates findings across all tools, scores confidence
- **Foundry PoC verification** — actually runs exploits to confirm findings are real
- **Polygon + Arbitrum primary** — Limbo auto-covers gas fees, ETH optional
- **5/10/12 confirmation thresholds** — no fake payment exploits
- **Encrypted private keys** — never stored plain in memory, auto-wiped after sweep
- **Auto-sweep** — USDT moves to your wallet automatically after scan
- **Admin bypass** — your Telegram ID gets free scans forever
- **Contract blacklist** — protect your own contracts
- **Enterprise bot intake** — full two-way communication through the bot, your identity stays hidden
- **Hard timeouts** — Mythril/Echidna can't OOM kill the server
- **Interactive installer** — one script configures everything

---

## Stack

| Layer | Technology |
|---|---|
| Bot | Rust + Teloxide |
| Async | Tokio |
| Static Analysis | Slither |
| Fuzzing | Echidna (protocol-aware invariants) |
| Symbolic Execution | Mythril |
| Formal Verification | Halmos |
| PoC Generation + Verification | Foundry |
| Correlation Engine | Custom Rust |
| AI Layer | Gemini 3 Flash (→ Claude Haiku at scale) |
| Payments | ethers-rs on-chain USDT monitoring |
| Key Security | AES-256-GCM + zeroize |
| PDF | printpdf |

---

## Install

```bash
git clone https://github.com/yourname/limbo
cd limbo
chmod +x install.sh
./install.sh
```

The script installs all tools and asks you 11 questions. That's it.

---

## Pricing

| Tier | Price | Engine | Output |
|---|---|---|---|
| Small | $50 USDT | Gemini Flash basic | PDF with findings + fixes |
| Pro | $150 USDT | Gemini Flash deep | PDF + Foundry PoCs |
| Enterprise | Custom | Max depth + manual | Full audit + your review |

---

## Networks

| Network | Gas | Confirmations |
|---|---|---|
| Polygon | Limbo covers it | 5 |
| Arbitrum | Limbo covers it | 10 |
| Ethereum | User pays | 12 |

---

## Bot Flow

```
/start
  → Paste address / Solidity source / upload .zip
  → Choose tier
  → Choose network
  → Send USDT to generated address
  → Payment confirmed on-chain automatically
  → All tools run in parallel with hard timeouts
  → Correlation engine cross-validates findings
  → Foundry verifies exploits are real
  → AI enriches with explanations + PoCs + fixes
  → PDF delivered
  → Funds swept to your wallet automatically
  → "Your contract has been to Limbo."
```

---

## PDF Sections

```
Cover               Protocol, tier, risk score, finding counts
Executive Summary   AI-written protocol overview
CONFIRMED           Foundry-verified real exploits
HIGH CONFIDENCE     2+ tools agreed, not yet verified
SINGLE TOOL         One tool found it, needs human review
FALSE POSITIVES     Filtered out, listed for transparency
Sign-off            "Your contract has been to Limbo."
```

---

## Roadmap

- [ ] Multichain Monday — Solana support announcement
- [ ] HackenProof / Cantina / Sherlock partnership integration
- [ ] Limbo Verified badge for protocols
- [ ] Public risk leaderboard
- [ ] Referral program (15% cut per referred scan)
- [ ] Switch to Claude Haiku 4.5 at scale
- [ ] v4 — Announcement + channel management system

---

*Powered by Astrophel — powered by stardust 💀*
