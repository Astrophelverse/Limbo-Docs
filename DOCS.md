# Limbo Documentation
### v2 — Your contract won't leave the same.

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
10. [Security & Privacy](#security--privacy)
11. [Self-Hosting](#self-hosting)
12. [FAQ](#faq)

---

## Overview

Limbo is an automated smart contract security auditing bot that lives entirely inside Telegram. Protocols submit their Solidity code or a project ZIP, pay USDT, and receive a premium PDF audit report — automatically, with no human intervention required for Small and Pro tiers.

**What makes Limbo different:**

Unlike tools that run a single scanner and dump raw output, Limbo runs four industry-standard security tools simultaneously, cross-validates their findings through a correlation engine, attempts to verify exploits using Foundry, and delivers everything as a structured PDF report — the same format a professional audit firm would produce.

---

## Getting Started

### As A Protocol

1. Open the Limbo bot on Telegram
2. Send `/start`
3. Paste your contract address, Solidity source, or upload a `.zip` of your project
4. Choose your tier (Small / Pro / Enterprise)
5. Choose your network (Polygon / Arbitrum / Ethereum)
6. Send the exact USDT amount to the generated address
7. Wait — Limbo scans automatically once payment confirms
8. Receive your PDF report

### As A Self-Hoster

See [Self-Hosting](#self-hosting) below.

---

## Bot Commands & Flow

| Command | Description |
|---|---|
| `/start` | Begin a new audit session |

**Full session flow:**

```
/start
  ↓
Submit contract (address / source / .zip)
  ↓
Select tier
  ↓
Select network
  ↓
Send USDT to generated address
  ↓
[Bot monitors chain every 30 seconds]
  ↓
Payment confirmed → scan begins
  ↓
[Slither + Echidna + Mythril run in parallel]
  ↓
[Correlation engine cross-validates findings]
  ↓
[Foundry attempts to verify exploits]
  ↓
[AI enriches findings with explanations + PoCs + fixes]
  ↓
PDF delivered to chat
  ↓
"Your contract has been to Limbo."
```

---

## Tiers & Pricing

### Small — $50 USDT

Best for: quick pre-deployment check, catching obvious bugs.

- Static analysis (Slither)
- Fuzzing (Echidna)
- Symbolic execution (Mythril)
- Correlation engine
- AI explanations and fix recommendations
- PDF report

### Pro — $150 USDT

Best for: protocols approaching mainnet launch, serious security review.

Everything in Small, plus:

- Full Foundry PoC generation
- Foundry exploit verification (confirmed vs theoretical)
- Deeper AI analysis with economic logic focus
- Multi-vector attack scenario generation
- Extended fuzzing with protocol-specific invariants

### Enterprise — Custom Pricing

Best for: protocols with significant TVL, pre-audit screening, institutional requirements.

Everything in Pro, plus:

- Manual review by a senior auditor
- Composability and cross-protocol analysis
- Economic impact quantification
- Custom scope definition
- Direct communication channel through the bot
- Limbo Verified badge (coming soon)
- Turnaround time negotiated per engagement

To begin: select Enterprise in the bot and answer the intake questions.

---

## Networks & Payments

| Network | Token | Gas | Confirmations Required |
|---|---|---|---|
| Polygon | USDT PoS | Limbo pays | 5 (~10 seconds) |
| Arbitrum | USDT | Limbo pays | 10 (~2 seconds) |
| Ethereum | USDT ERC-20 | You pay | 12 (~3 minutes) |

**How payment works:**

Each session generates a unique throwaway wallet address. You send the exact USDT amount to that address. Limbo monitors the blockchain every 30 seconds. Once the required number of confirmations is reached, the scan begins automatically.

After the scan, your funds are swept automatically to the Limbo operations wallet.

**Important:**
- Send the exact amount shown
- Use the correct network — USDT on Polygon is not the same token as USDT on Ethereum
- Payment windows expire after 2 hours

**Ethereum note:**
When using Ethereum, you must include a small amount of ETH in the same wallet to cover gas for the sweep transaction.

---

## How The Scanner Works

Limbo runs four tools in parallel using Tokio async workers. Each tool has a hard timeout to prevent OOM crashes on the server.

### Slither (Static Analysis)

Slither analyses the contract's source code without executing it. It checks for known vulnerability patterns including reentrancy, access control issues, unchecked return values, integer overflows and more.

**Timeout:** 120 seconds

### Echidna (Fuzzing)

Echidna uses property-based fuzzing to break invariants. Limbo does not use generic invariants. Instead, it first detects the protocol type (lending, AMM, governance, bridge, staking) and generates custom invariants appropriate to that type.

**Examples of protocol-specific invariants:**
- Lending: total borrows cannot exceed total deposits minus reserve factor, regardless of flash loan manipulation
- AMM: the xy=k invariant must hold after every swap even when the attacker controls transaction ordering
- Bridge: locked funds must always equal or exceed outstanding claims on the destination chain

**Timeout:** 300 seconds

### Mythril (Symbolic Execution)

Mythril builds a symbolic model of the contract and explores all possible execution paths mathematically. It is particularly effective at finding integer overflows, reentrancy, and dangerous delegatecall patterns.

**Timeout:** 90 seconds

### Halmos (Formal Verification)

Halmos provides formal verification — mathematical proof that certain properties hold for all possible inputs. This is the same category of tool used by Certora Prover, now available as open source.

---

## The Correlation Engine

This is the core of what separates Limbo from running tools manually.

After all tools complete, the correlation engine:

1. **Groups findings** by normalised title across tools (accounts for different naming conventions)
2. **Counts how many tools agree** on each finding
3. **Assigns confidence:**
   - `CONFIRMED` — Foundry exploit ran successfully, finding is proven real
   - `HIGH CONFIDENCE` — 2 or more tools independently found the same issue
   - `SINGLE TOOL` — one tool flagged it, needs human verification
   - `FALSE POSITIVE` — ruled out after cross-validation
4. **Upgrades severity** for confirmed findings (a confirmed Medium becomes High, confirmed High becomes Critical)
5. **Filters noise** — Info-severity single-tool findings are marked as false positives

This means the PDF you receive contains only validated, actionable findings — not a raw dump of scanner output.

---

## Understanding Your PDF Report

### Cover Page

- Protocol identifier
- Tier used
- Network
- Protocol type (detected automatically)
- Date and time
- Risk Score (0–100)
- Finding counts by category

### Risk Score

| Score | Meaning |
|---|---|
| 0–39 | Low risk — minor issues found |
| 40–69 | Medium risk — address before mainnet |
| 70–100 | High risk — critical issues present |

### Executive Summary

An AI-generated paragraph explaining what the protocol does, the most important security concerns found, and the overall security posture.

### CONFIRMED VULNERABILITIES

Findings where Foundry successfully ran an exploit. These are proven real. Fix these before doing anything else.

Each finding includes:
- Severity badge
- Confidence badge
- Which tools found it
- Foundry verified indicator (if applicable)
- Description of the vulnerability and its impact
- Proof of Concept (Foundry test code)
- Recommended fix in Solidity

### HIGH CONFIDENCE FINDINGS

Two or more tools independently found these issues. Not yet Foundry-verified but highly likely real. Fix before mainnet.

### SINGLE TOOL FINDINGS

One tool found these. Could be real, could be context-dependent. Recommended for human review.

### FALSE POSITIVES — FILTERED

Listed for transparency. These were detected but ruled out by the correlation engine.

### Sign-off

*Your contract has been to Limbo.*

---

## Enterprise Tier

Enterprise engagements are handled entirely through the bot. Your identity stays private throughout.

**Intake process:**

1. Select Enterprise in the bot
2. Answer 5 questions: protocol name, TVL, scope, timeline, contact
3. Review and confirm your submission
4. Limbo notifies the auditor privately
5. The auditor replies through the bot with a custom quote
6. You pay to the provided address
7. Full audit begins

**What enterprise includes:**
- Everything in Pro
- Manual auditor review of all findings
- Cross-protocol composability analysis
- Economic impact quantification per finding
- Negotiated timeline
- Direct communication via bot throughout

---

## Security & Privacy

### Payment security

- Every session generates a unique throwaway wallet
- Private keys are encrypted in memory using AES-256-GCM
- Keys are wiped from memory immediately after the sweep transaction
- Funds sweep to the operations wallet automatically after scan delivery
- Payment windows expire after 2 hours

### Your contract

- Contracts are written to ephemeral temp directories
- Directories are deleted automatically when the scan completes (Rust RAII)
- No contract code is stored permanently on Limbo servers
- ZIP uploads are stream-extracted — only .sol, .json and .toml files are processed

### Confirmation thresholds

- Polygon: 5 confirmations
- Arbitrum: 10 confirmations
- Ethereum: 12 confirmations

These thresholds prevent accepting payments from orphaned transactions.

---

## Self-Hosting

### Requirements

- Ubuntu 22.04 or 24.04
- 2GB RAM minimum (4GB recommended for Pro/Enterprise scans)
- 10GB disk space
- Public internet access

### Install

```bash
git clone https://github.com/astrophel2/limbo
cd limbo
chmod +x install.sh
./install.sh
```

The installer will ask you for:

| Prompt | Where to get it |
|---|---|
| Telegram Bot Token | @BotFather on Telegram |
| Your Telegram ID | @userinfobot on Telegram |
| Enterprise notify chat | Same as your Telegram ID |
| Gemini API Key | aistudio.google.com |
| Polygon RPC URL | alchemy.com (free tier) |
| Arbitrum RPC URL | alchemy.com (free tier) |
| Ethereum RPC URL | alchemy.com (optional) |
| Wallet address | Any EVM wallet (MetaMask, Rabby, Trust) |
| Polygonscan API Key | polygonscan.com (free) |
| Arbiscan API Key | arbiscan.io (free) |

### Running

```bash
# Standard
./target/release/limbo

# With logs
RUST_LOG=info ./target/release/limbo

# Background
nohup ./target/release/limbo &
```

### Blacklisting your own contracts

In your `.env` file:

```
BLACKLIST=0xYourContract1,0xYourContract2
```

---

## FAQ

**Can I upload a full Hardhat or Foundry project?**
Yes. Upload it as a `.zip`. Limbo extracts only `.sol`, `.json` and `.toml` files and skips `node_modules` entirely.

**What Solidity versions are supported?**
0.6.x through 0.8.x. Most tools work best with 0.8.x.

**What if the scan fails?**
The bot will notify you. For paid scans that fail due to a Limbo error, contact @astrophel2 for a refund or rescan.

**How long does a scan take?**
Small: 3–8 minutes. Pro: 5–12 minutes. Depends on contract complexity.

**Can I scan a contract by address?**
Yes. Limbo fetches the source from Etherscan/Polygonscan/Arbiscan automatically.

**Is my contract code stored?**
No. All files are written to ephemeral temp directories and deleted automatically when the scan completes.

**Can I get a refund?**
Payments are final for completed scans. If a scan fails due to a Limbo error you will be rescanned or refunded.

**What is the Enterprise turnaround time?**
Negotiated per engagement. Typically 3–7 business days for a full manual review.

---

*Limbo v2 — Powered by Astrophel 💀*
