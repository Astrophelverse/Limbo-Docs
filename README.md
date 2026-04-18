# LIMBO
### *Your contract won't leave the same.*

Limbo is an automated smart contract security auditing bot built in Rust, delivered through Telegram. Protocols submit Solidity source or a project ZIP, pay USDT, and receive a premium PDF audit report — fully automated, no humans required for Small and Pro tiers.

---

## How It Works

```
/start → submit contract → choose tier → pay USDT
  → tools run in parallel → correlation engine validates
  → Foundry verifies exploits → AI explains everything
  → PDF delivered → funds swept automatically
  → LimboRewards token issued
```

---

## Stack

| Layer | Technology |
|---|---|
| Bot | Rust + Teloxide |
| Async runtime | Tokio |
| Static analysis | Slither |
| Fuzzing | Echidna (protocol-aware invariants) |
| Symbolic execution | Mythril |
| Formal verification | Halmos |
| PoC generation + verification | Foundry |
| Correlation engine | Custom Rust |
| AI layer | Gemini Flash → Claude Haiku at scale |
| Payments | ethers-rs on-chain USDT monitoring |
| Key security | AES-256-GCM + zeroize |
| PDF | printpdf |
| LimboRewards | Chainlink VRF |

---

## Tiers

| Tier | Price | Depth | LimboRewards Tokens |
|---|---|---|---|
| Small | $50 USDT | Static, fuzzing, correlation, AI | 1 |
| Pro | $150 USDT | Everything + Foundry PoC verification | 3 |
| Enterprise | Custom | Everything + manual review | 10 |

---

## Networks

| Network | Gas | Confirmations |
|---|---|---|
| Polygon | Limbo covers it | 5 |
| Arbitrum | Limbo covers it | 10 |
| Ethereum | User pays | 12 |

---

## Install

```bash
git clone https://github.com/astrophel2/limbo
cd limbo
chmod +x install.sh
./install.sh
```

The installer asks 11 questions and configures everything.

---

## PDF Structure

```
Cover               Risk score, finding counts, protocol metadata
Executive summary   AI-written protocol overview and risk assessment
CONFIRMED           Foundry-verified real exploits
HIGH CONFIDENCE     2+ tools agreed — fix before mainnet
SINGLE TOOL         One tool flagged it — human review recommended
FALSE POSITIVES     Filtered out, listed for transparency
Sign-off            "Your contract has been to Limbo."
```

---

## LimboRewards

Every paid scan issues a LimboRewards draw token. When a milestone is reached, Chainlink VRF selects a winner on-chain and pays out in USDT automatically. The contract is open source — every draw is publicly verifiable. See [DOCS.md](DOCS.md) for full details.

---

## Roadmap

- [ ] Multichain Monday — Solana / Anchor program support
- [ ] LimboRewards contract deployment
- [ ] Limbo Verified badge for audited protocols
- [ ] Partnership integrations — Cantina, Sherlock, HackenProof, Cyfrin
- [ ] Referral program — 15% revenue share
- [ ] Public risk leaderboard
- [ ] API access for CI/CD integration

---

## License

Business Source License 1.1 — source-available, non-commercial use permitted. Converts to MIT on 2028-01-01. See [LICENSE](LICENSE).

---

*Powered by Astrophel — powered by stardust 💀*
*Contact: @astrophel2 on Telegram*
