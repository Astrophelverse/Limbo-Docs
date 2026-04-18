# Limbo Documentation
### Your contract won't leave the same

---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Bot Commands & Flow](#bot-commands--flow)
4. [Tiers & Pricing](#tiers--pricing)
5. [Networks & Payments](#networks--payments)
6. [How The Scanner Works](#how-the-scanner-works)
7. [The Correlation Engine](#the-correlation-engine)
8. [Understanding Your PDF Report](#understanding-your-pdf-report)
9. [Enterprise Tier](#enterprise-tier)
10. [LimboRewards](#limborewards)
11. [Security & Privacy](#security--privacy)
12. [Self-Hosting](#self-hosting)
13. [FAQ](#faq)

---

## Overview

Limbo is an automated smart contract security auditing bot that lives entirely inside Telegram. Protocols submit Solidity source or a project ZIP, pay USDT, and receive a premium PDF audit report — automatically, with no human intervention required for Small and Pro tiers.

Unlike tools that run a single scanner and dump raw output, Limbo runs four industry-standard security tools simultaneously, cross-validates their findings through a correlation engine, verifies exploits using Foundry, and delivers a structured report in the same format a professional audit firm would produce — at a fraction of the cost.

---

## Getting Started

### As A Protocol

1. Open Limbo on Telegram and send `/start`
2. Paste your contract address, Solidity source, or upload a `.zip` of your project
3. Choose your tier — Small, Pro, or Enterprise
4. Choose your network — Polygon, Arbitrum, or Ethereum
5. Send the exact USDT amount to the generated address
6. Limbo scans automatically once payment confirms on-chain
7. Receive your PDF report and your LimboRewards draw token

### As A Self-Hoster

See [Self-Hosting](#self-hosting).

---

## Bot Commands & Flow

| Command | Description |
|---|---|
| `/start` | Begin a new audit session |

**Full session flow:**

```
/start
  ↓
Submit contract (address / Solidity source / .zip)
  ↓
Select tier
  ↓
Select network
  ↓
Send USDT to generated one-time address
  ↓
Bot monitors blockchain every 30 seconds
  ↓
Payment confirmed at required threshold → scan begins
  ↓
Slither + Echidna + Mythril run in parallel
  ↓
Correlation engine cross-validates all findings
  ↓
Foundry attempts to verify exploits
  ↓
AI enriches each finding with explanation, PoC and fix
  ↓
PDF delivered
  ↓
"Your contract has been to Limbo."
  ↓
LimboRewards token issued to your wallet
```

---

## Tiers & Pricing

### Small — $50 USDT

Best for quick pre-deployment checks and catching common vulnerabilities.

- Static analysis (Slither)
- Adversarial fuzzing (Echidna)
- Symbolic execution (Mythril)
- Correlation engine
- AI explanations and fix recommendations
- PDF report
- 1 LimboRewards draw token

### Pro — $150 USDT

Best for protocols approaching mainnet with significant TVL.

Everything in Small, plus:

- Full Foundry PoC generation
- Foundry exploit verification — findings confirmed or theoretical
- Deeper AI analysis focused on economic logic
- Multi-vector attack scenario generation
- Extended fuzzing with protocol-specific invariants
- 3 LimboRewards draw tokens

### Enterprise — Custom Pricing

Best for protocols with large TVL, institutional requirements, or pre-audit screening needs.

Everything in Pro, plus:

- Manual review by a senior auditor
- Composability and cross-protocol analysis
- Economic impact quantification per finding
- Custom scope definition
- Direct bot communication — auditor identity stays private
- Limbo Verified badge
- Negotiated timeline
- 10 LimboRewards draw tokens

To begin: select Enterprise in the bot and answer the intake questions.

---

## Networks & Payments

| Network | Token | Gas | Confirmations |
|---|---|---|---|
| Polygon | USDT PoS | Limbo pays | 5 (~10 seconds) |
| Arbitrum | USDT | Limbo pays | 10 (~2 seconds) |
| Ethereum | USDT ERC-20 | You pay | 12 (~3 minutes) |

Each session generates a unique throwaway wallet address. Send the exact USDT amount shown. Limbo monitors the blockchain every 30 seconds. Once the required confirmations are reached the scan begins automatically and funds sweep to the operations wallet after delivery.

**Important:**
- Send the exact amount — no more, no less
- Use the correct network — USDT on Polygon is a different contract to USDT on Ethereum
- Payment windows expire after 2 hours
- On Ethereum you must include a small amount of ETH to cover the sweep gas fee

---

## How The Scanner Works

Limbo runs all tools concurrently using Tokio async workers. Each tool has a hard timeout enforced at the process level, preventing any single contract from exhausting server memory.

### Slither — Static Analysis

Analyses contract source without executing it. Detects reentrancy, access control issues, unchecked return values, integer overflows and more using both default and custom DeFi-specific detectors.

**Timeout:** 120 seconds

### Echidna — Adversarial Fuzzing

Property-based fuzzing to break protocol invariants under adversarial conditions. Limbo generates invariants specific to the detected protocol type rather than using generic defaults.

**Protocol-specific invariant examples:**

Lending:
```
∀ state after any transaction sequence:
  total_borrows ≤ total_deposits − reserve_factor

∀ flash loan F:
  balance_before ≤ balance_after + fee
```

AMM:
```
∀ swap:
  k_before ≤ k_after

∀ adversarial swap sequence:
  price_impact matches expected_slippage
```

Bridge:
```
∀ lock(amount, destination):
  outstanding_claims(destination) ≤ locked_total(source)
```

**Timeout:** 300 seconds

### Mythril — Symbolic Execution

Builds a symbolic model and explores all possible execution paths mathematically. Effective at finding vulnerabilities that require specific transaction sequences or precise value manipulation. Runs in an isolated Docker container.

**Timeout:** 180 seconds

### Halmos — Formal Verification

Mathematical proof that invariants hold for all possible inputs — the same category of analysis as Certora Prover, available as open source.

---

## The Correlation Engine

After all tools complete, the correlation engine processes findings through five stages:

**Normalisation** — titles are mapped to canonical form so the same bug named differently by different tools groups correctly.

**Grouping** — findings with the same normalised title are grouped and their sources tracked.

**Confidence scoring:**

| Condition | Confidence |
|---|---|
| Foundry exploit ran successfully | CONFIRMED |
| 2+ tools independently found it | HIGH CONFIDENCE |
| 1 tool found it | SINGLE TOOL |
| Info severity, single tool | FALSE POSITIVE |

**Severity escalation** — confirmed findings are upgraded: Medium → High, High → Critical.

**Deduplication** — findings are deduplicated and sorted by severity within each confidence tier.

The result is a PDF containing only validated, actionable findings.

---

## Understanding Your PDF Report

### Cover Page

Protocol identifier, tier, network, protocol type, date, risk score, finding counts by category.

### Risk Score

| Score | Meaning |
|---|---|
| 0–39 | Low risk — minor issues only |
| 40–69 | Medium risk — fix before mainnet |
| 70–100 | High risk — critical issues present |

### Executive Summary

AI-generated overview of what the protocol does, the most significant risks found, and the overall security posture.

### CONFIRMED VULNERABILITIES

Foundry-verified exploits. Proven real. Each finding includes severity, confidence, tools that found it, Foundry verification status, vulnerability description, PoC code, and recommended fix.

### HIGH CONFIDENCE FINDINGS

Two or more tools agreed. Not Foundry-verified yet but highly likely real. Fix before mainnet.

### SINGLE TOOL FINDINGS

One tool flagged these. Human review recommended.

### FALSE POSITIVES — FILTERED

Detected but ruled out. Listed for transparency.

### Sign-off

*Your contract has been to Limbo.*

---

## Enterprise Tier

Enterprise engagements are handled entirely through the bot. The auditor's identity stays private throughout.

**Intake process:**

1. Select Enterprise in the bot
2. Answer 5 questions: protocol name, TVL, scope, timeline, contact
3. Review and confirm your submission
4. Limbo notifies the auditor privately
5. The auditor replies through the bot with a custom quote
6. Pay to the provided address
7. Full audit begins with direct bot communication throughout

---

## LimboRewards

LimboRewards is a verifiable onchain draw mechanism that rewards the Limbo user community. Every paid scan issues draw tokens. When a milestone is reached, a winner is selected automatically using Chainlink VRF and paid in USDT. No one — including the Limbo team — can influence or predict the outcome.

### Why This Is Not A Lottery

The product is real and valuable. Users pay for a security audit. The draw token is a bonus that comes with the purchase — the same model as McDonald's Monopoly. Prize funds come from Limbo's operating revenue, not from user payments. You pay for an audit. The token is free.

### The Flow

```
User pays for scan
  ↓
Scan completes — PDF delivered
  ↓
LimboRewards contract issues draw token to user's wallet
  ↓
Progress bar advances toward milestone (e.g. 100 scans)
  ↓
Milestone reached
  ↓
Chainlink VRF generates a verifiable random number on-chain
  ↓
Winner selected from token holders — no manipulation possible
  ↓
Prize paid automatically in USDT to winner's wallet
  ↓
New round begins
```

### Token Allocation

| Tier | Draw Tokens Per Scan |
|---|---|
| Small | 1 |
| Pro | 3 |
| Enterprise | 10 |

Higher tiers receive more entries proportional to their contribution.

### Prize Structure

Prizes are funded at predetermined revenue milestones. The contract does not release a prize until confirmed operating balance is sufficient. Prize funds are never sourced directly from user payments.

### Verifiability

The LimboRewards contract is open source. Anyone can verify the token issuance logic, Chainlink VRF integration, milestone thresholds, prize payment logic, and full history of past draws and winners. Chainlink VRF provides cryptographic proof that every random number was generated fairly.

---

## Security & Privacy

### Payment Security

Every session generates a unique throwaway wallet. Private keys are encrypted in memory using AES-256-GCM and wiped immediately after the sweep transaction completes. Funds sweep to the operations wallet automatically after PDF delivery. Sessions expire after 2 hours.

### Your Contract

Contracts are written to ephemeral temp directories deleted automatically on scan completion via Rust RAII. No source code is stored permanently. ZIP uploads are stream-extracted — only `.sol`, `.json` and `.toml` files are processed. Build folders including `node_modules`, `target/`, `artifacts/`, `cache/` and `out/` are skipped entirely.

### Confirmation Thresholds

Polygon 5, Arbitrum 10, Ethereum 12. These prevent accepting payments from transactions later reversed in a chain reorganisation.

---

## Self-Hosting

### Requirements

- Ubuntu 22.04 or 24.04
- 2GB RAM minimum, 4GB recommended
- 20GB disk space
- Docker installed (for Mythril)
- Public internet access

### Install

```bash
git clone https://github.com/astrophel2/limbo
cd limbo
chmod +x install.sh
./install.sh
```

### What The Installer Asks

| Prompt | Where To Get It |
|---|---|
| Telegram Bot Token | @BotFather |
| Your Telegram ID | @userinfobot |
| Enterprise notify chat ID | Same as your Telegram ID |
| Gemini API Key | aistudio.google.com |
| Polygon RPC URL | alchemy.com free tier |
| Arbitrum RPC URL | alchemy.com free tier |
| Ethereum RPC URL | alchemy.com (optional) |
| Wallet address | Any EVM wallet — same address for all chains |
| Polygonscan API Key | polygonscan.com free |
| Arbiscan API Key | arbiscan.io free |

### Running

```bash
# Standard
./target/release/limbo

# With logs
RUST_LOG=info ./target/release/limbo

# Persistent session (survives terminal close)
screen -S limbo
./target/release/limbo
# Ctrl+A then D to detach, screen -r limbo to return
```

### Blacklisting Your Own Contracts

Add to `.env`:

```
BLACKLIST=0xYourContract1,0xYourContract2
```

---

## FAQ

**Can I upload a full Hardhat or Foundry project?**
Yes. Upload as `.zip`. Limbo stream-extracts only `.sol`, `.json` and `.toml` files and skips `node_modules`, `target/`, `artifacts/`, `cache/` and `out/` entirely.

**What Solidity versions are supported?**
0.6.x through 0.8.x. Most tools work best with 0.8.x.

**What if the scan fails?**
The bot will notify you. For paid scans that fail due to a Limbo error, contact @astrophel2 for a refund or rescan.

**How long does a scan take?**
Small: 3–8 minutes. Pro: 5–15 minutes. Depends on contract size and complexity.

**Can I scan by contract address?**
Yes. Limbo fetches verified source from the relevant block explorer automatically.

**Is my contract code stored?**
No. All files are written to ephemeral temp directories deleted automatically when the scan completes.

**Can I get a refund?**
Payments are final for completed scans. If a scan fails due to a Limbo error you will be rescanned or refunded.

**What is the Enterprise turnaround time?**
Negotiated per engagement. Typically 3–7 business days for a full manual review.

**How do I claim my LimboRewards tokens?**
After your scan completes the bot provides your token receipt. Full token issuance and the WebApp draw tracker are part of the LimboRewards rollout.

---

*Limbo — Powered by Astrophel 💀*
*Contact: @astrophel2 on Telegram*
