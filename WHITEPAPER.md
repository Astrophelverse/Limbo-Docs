# Limbo: Automated Adversarial Security Analysis for EVM Smart Contracts

**Version:** 1.0
**Author:** Astrophel
**Contact:** @astrophel2 on Telegram
**Date:** 2025

---

## Abstract

Smart contract exploits have drained over $3 billion from DeFi protocols since 2020. The majority of these incidents involved economic logic vulnerabilities — attack surfaces that existing automated tools fail to detect reliably. Manual audits are expensive, slow, and inaccessible to smaller protocols. Limbo is an automated security analysis system delivered through Telegram that combines static analysis, adversarial fuzzing with protocol-aware invariants, symbolic execution and formal verification into a single coordinated pipeline. A cross-validation correlation engine eliminates false positives. Foundry-based exploit verification confirms that flagged vulnerabilities are real. An AI layer translates raw findings into actionable audit documents. Limbo makes institutional-grade security analysis accessible to any EVM protocol at a fraction of the cost and time of a manual audit. An onchain incentive layer — LimboRewards — transforms every paid scan into a verifiable community participation event powered by Chainlink VRF.

---

## 1. Introduction

### 1.1 The Problem

The EVM smart contract ecosystem has accumulated hundreds of billions of dollars in total value locked. This concentration of value makes protocols a continuous target for adversarial actors.

Post-mortem analysis of major exploits reveals a consistent pattern. The vulnerabilities that cause the most damage are rarely simple code bugs. They are economic logic attacks — price manipulation through flash loans, oracle dependencies that can be influenced within a single transaction, incentive misalignments that allow an attacker to extract value at the expense of other participants.

Existing automated tools are highly effective at detecting known code-level vulnerability patterns. They are significantly less effective at reasoning about economic attack surfaces, which requires understanding a protocol's intended invariants — properties that should hold regardless of what sequence of transactions an adversary executes.

Manual audits fill this gap but introduce two critical constraints: cost and time. A professional audit from a reputable firm costs between $10,000 and $100,000 and takes weeks to schedule. This creates a two-tier security landscape where well-funded protocols receive thorough reviews and smaller protocols deploy with minimal scrutiny.

### 1.2 The Opportunity

The gap between automated tools and manual audits is not primarily a technical gap. It is an integration and intelligence gap. The tools exist. What is missing is a system that runs all relevant tools with protocol-aware configuration, cross-validates findings to eliminate noise, verifies that flagged vulnerabilities are actually exploitable, and delivers results in a format that protocol teams can act on immediately.

Limbo closes this gap without requiring a human auditor for the majority of common vulnerability classes.

---

## 2. System Architecture

### 2.1 Overview

```
                    ┌─────────────────┐
                    │  Telegram Bot   │
                    │  (Teloxide/Rust)│
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
     ┌────────▼──────┐  ┌───▼────┐  ┌─────▼──────┐
     │ Payment Layer │  │ ZIP    │  │ Enterprise │
     │ USDT on-chain │  │Extract │  │  Intake    │
     └────────┬──────┘  └───┬────┘  └─────┬──────┘
              └──────────────┼─────────────┘
                             │
                    ┌────────▼────────┐
                    │  Scanner Engine │
                    │   (Tokio async) │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
   ┌──────▼──────┐   ┌───────▼──────┐   ┌──────▼──────┐
   │   Slither   │   │   Echidna    │   │   Mythril   │
   │   (static)  │   │  (fuzzing)   │   │ (symbolic)  │
   └──────┬──────┘   └───────┬──────┘   └──────┬──────┘
          └──────────────────┼──────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Correlation    │
                    │    Engine       │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │    Foundry      │
                    │ PoC Verification│
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │   AI Layer      │
                    │ (Gemini Flash)  │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  PDF Generator  │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  LimboRewards   │
                    │ (Chainlink VRF) │
                    └─────────────────┘
```

### 2.2 Technology Stack

| Component | Technology | Rationale |
|---|---|---|
| Bot runtime | Rust + Teloxide | Memory safety, concurrency, performance |
| Async executor | Tokio | Parallel tool execution with timeout enforcement |
| Static analysis | Slither | Industry standard, extensible custom detector API |
| Fuzzing | Echidna | Property-based, supports protocol-aware invariants |
| Symbolic execution | Mythril | Deep path exploration, Docker-isolated |
| Formal verification | Halmos | Mathematical proof — same category as Certora Prover |
| PoC generation | Foundry | Industry standard testing framework |
| AI enrichment | Gemini Flash / Claude Haiku | Cost-efficient, strong on Solidity |
| Payments | ethers-rs on-chain | USDT monitoring, AES-256-GCM key security |
| PDF generation | printpdf | Native Rust, no external dependencies |
| LimboRewards | Chainlink VRF | Verifiable on-chain randomness |

---

## 3. Scanner Design

### 3.1 Protocol Type Detection

Before any security tool runs, Limbo determines the protocol type using AI analysis of the contract source. This classification drives invariant generation and analysis depth for subsequent stages.

Supported types: `lending`, `amm`, `governance`, `bridge`, `staking`, `nft`, `token`, `defi`

### 3.2 Static Analysis

Slither runs with informational findings suppressed to reduce noise. Custom detectors targeting DeFi-specific patterns supplement the default ruleset. These detectors are derived from real vulnerabilities discovered by the Limbo team during competitive audit participation on Code4rena, Immunefi and Cantina.

### 3.3 Adversarial Fuzzing

Echidna is configured with protocol-aware invariants rather than generic properties. This is the primary mechanism for detecting economic logic vulnerabilities that code-level analysis cannot find.

**Invariant strategy by protocol type:**

*Lending:*
```
∀ state s after any sequence of transactions T:
  total_borrows(s) ≤ total_deposits(s) − reserve(s)

∀ flash loan F:
  balance_before ≤ balance_after + fee

∀ liquidation L:
  health_factor(borrower) < threshold → liquidation_valid
```

*AMM:*
```
∀ swap(tokenIn, amountIn) → amountOut:
  k_before ≤ k_after

∀ adversarial sequence S:
  price_impact(S) matches expected_slippage(S)
```

*Bridge:*
```
∀ lock(amount, destination):
  outstanding_claims(destination) ≤ locked_total(source)
```

The fuzzer uses an adversarial sender pool including flash loan providers to test whether invariants hold under coordinated attack scenarios.

### 3.4 Symbolic Execution

Mythril performs bounded symbolic execution with timeout enforcement. It is effective at finding vulnerabilities requiring specific transaction sequences or precise value manipulation that random fuzzing rarely reaches. It runs in an isolated Docker container to prevent host memory exhaustion.

### 3.5 Formal Verification

Halmos provides unbounded formal verification for selected properties. Unlike fuzzing, which tests many inputs, formal verification proves that a property holds for all possible inputs.

### 3.6 Execution Model

All tools run concurrently via Tokio async workers with hard timeout enforcement:

```
Slither:   120 seconds
Echidna:   300 seconds
Mythril:   180 seconds
Halmos:    120 seconds
```

If any tool exceeds its timeout, the process is killed and the scan continues with results from the remaining tools. This ensures the server cannot be OOM-killed by a complex contract.

---

## 4. Correlation Engine

### 4.1 Problem

Running multiple security tools against the same contract produces overlapping and contradictory output. Without correlation, the PDF would be noise. The correlation engine is what makes Limbo's output usable.

### 4.2 Approach

Five stages: normalisation of finding titles across tools, grouping by canonical key, confidence scoring based on tool agreement and Foundry verification, severity escalation for confirmed findings, and deduplication.

**Confidence tiers:**

| Condition | Confidence |
|---|---|
| Foundry exploit ran successfully | CONFIRMED |
| 2+ tools independently found it | HIGH CONFIDENCE |
| 1 tool found it | SINGLE TOOL |
| Info severity + single tool | FALSE POSITIVE |

Confirmed findings have severity escalated: Medium → High, High → Critical. This reflects the increased certainty that a proven, exploitable vulnerability represents.

### 4.3 Foundry Verification

For all AI-generated PoC code, Limbo attempts to execute the exploit using Foundry against a local fork. PoC tests are written to fail when the exploit succeeds. If Forge reports a test failure, the finding is marked CONFIRMED. This binary verification is the most important signal in the pipeline — a CONFIRMED finding means an attacker could have run this exact code and extracted funds.

---

## 5. AI Layer

### 5.1 Role

The AI layer does not perform security analysis. That is the exclusive domain of the deterministic tools. The AI layer has three responsibilities: protocol type classification, translation of raw tool output into human-readable explanations, and generation of Foundry PoC code for each finding.

### 5.2 Model Selection

Limbo uses Gemini Flash for the AI layer, selected for strong performance on coding benchmarks and cost efficiency suitable for a per-scan pricing model. The architecture supports model substitution; Claude Haiku 4.5 is targeted for production scale.

### 5.3 Limitations

The AI layer can generate incorrect PoC code or mischaracterise vulnerabilities. This is why AI output is used for enrichment only — the underlying security determination comes from deterministic tools. All AI-generated PoC code that is not Foundry-verified is clearly labelled as unverified in the PDF.

---

## 6. Payment Architecture

### 6.1 Design Principles

Zero manual intervention for Small and Pro tiers. Zero persistent storage of sensitive material. Cryptographic guarantees for private key security. Configurable confirmation thresholds per network.

### 6.2 Session Lifecycle

```
1.  User selects tier and network
2.  Limbo generates ephemeral wallet (ethers-rs LocalWallet)
3.  Private key encrypted with AES-256-GCM, ephemeral session key
4.  User sends USDT to wallet address
5.  Monitor polls blockchain every 30 seconds
6.  On N confirmations: scan begins
7.  On scan completion: sweep transaction constructed
8.  Sweep signed with decrypted key
9.  Key zeroized from memory immediately
10. USDT arrives in operations wallet
11. LimboRewards token issued
```

### 6.3 Confirmation Thresholds

| Network | Confirmations | Rationale |
|---|---|---|
| Polygon | 5 | Fast finality, low reorg risk |
| Arbitrum | 10 | Optimistic rollup, conservative threshold |
| Ethereum | 12 | Standard finality assumption |

### 6.4 Gas Management

For Polygon and Arbitrum, a gas feeder mechanism sends a small amount of native token to the ephemeral wallet before the sweep transaction. This cost is absorbed into operating margin. For Ethereum, the user includes ETH alongside their USDT payment.

---

## 7. LimboRewards

### 7.1 Overview

LimboRewards is an onchain incentive mechanism that rewards the Limbo user community. Every paid scan issues draw tokens. When a milestone is reached, a winner is selected using Chainlink VRF and paid in USDT automatically. No party — including Limbo — can influence or predict the outcome.

### 7.2 Mechanism

```
User completes a paid scan
  ↓
LimboRewards contract issues draw tokens proportional to tier
  ↓
Tokens accumulate toward a milestone (e.g. 100 scans per round)
  ↓
Milestone reached
  ↓
Chainlink VRF fulfillRandomWords called on-chain
  ↓
Verifiable random index selects winner from token holders
  ↓
Prize paid in USDT directly to winner's wallet
  ↓
New round begins
```

### 7.3 Token Allocation

| Tier | Draw Tokens |
|---|---|
| Small | 1 |
| Pro | 3 |
| Enterprise | 10 |

### 7.4 Prize Funding

Prizes are funded from operating revenue at predetermined milestones. The contract enforces that no prize is released until sufficient balance is confirmed. Prize funds are never sourced directly from user payments.

### 7.5 Why This Is Not A Lottery

The product is real and independently valuable. Users pay for a security audit. The draw token is a bonus issued on completion of that purchase. This is structurally identical to purchase-based promotional draws that operate legally across many jurisdictions. The prize is funded by the operator, not pooled from participants.

### 7.6 Verifiability

The LimboRewards contract is open source. Anyone can verify token issuance logic, Chainlink VRF integration, milestone thresholds, prize payment logic and the full history of draws and winners. Chainlink VRF provides a cryptographic proof that each random number was generated honestly and was not manipulated before use.

---

## 8. Security Properties

| Property | Mechanism |
|---|---|
| Private key confidentiality | AES-256-GCM encryption + zeroize on use |
| Payment replay prevention | Unique address per session |
| False payment prevention | Minimum confirmation thresholds |
| Denial of service prevention | Hard tool timeouts |
| Disk exhaustion prevention | Rust RAII tempdir cleanup |
| Memory exhaustion prevention | ZIP stream extraction, build folder skip |
| Admin impersonation prevention | Telegram ID verification |
| Contract self-scan prevention | Configurable address blacklist |
| Draw manipulation prevention | Chainlink VRF cryptographic proof |

---

## 9. Competitive Landscape

| Tool | Automated | Economic Logic | PDF Output | Telegram Native | LimboRewards | Price |
|---|---|---|---|---|---|---|
| **Limbo** | ✓ | ✓ | ✓ | ✓ | ✓ | $50–$150 |
| Slither (standalone) | ✓ | ✗ | ✗ | ✗ | ✗ | Free |
| Mythril (standalone) | ✓ | ✗ | ✗ | ✗ | ✗ | Free |
| Certora | ✓ | Partial | ✗ | ✗ | ✗ | Enterprise |
| Manual audit | ✗ | ✓ | ✓ | ✗ | ✗ | $10k–$100k |

---

## 10. Roadmap

### Phase 1 — Launch
EVM smart contract support, Polygon and Arbitrum primary networks, Small / Pro / Enterprise tiers, correlation engine, Foundry verification, automated USDT payment.

### Phase 2 — Growth
LimboRewards contract deployment, partnership integrations with Cantina / Sherlock / HackenProof / Cyfrin, referral program, Limbo Verified badge, public risk leaderboard.

### Phase 3 — Multichain Monday
Solana smart contract support for Anchor programs, Rust-based analysis toolchain, cross-chain composability analysis.

### Phase 4 — Platform
API access for CI/CD pipeline integration, white-label offering for audit firms, announcement and channel management system.

---

## 11. Conclusion

Limbo addresses the automation gap in smart contract security by combining deterministic analysis tools with a correlation layer, exploit verification and AI-assisted reporting. LimboRewards adds a verifiable community incentive layer that converts every scan into a reason to participate and share.

The economic attack surfaces responsible for the largest DeFi losses are detectable with protocol-aware fuzzing. The false positive problem that makes raw tool output unusable is solvable with cross-validation. The accessibility barrier that excludes smaller protocols from serious security review is removable with automation and appropriate pricing.

Limbo makes institutional-grade security analysis available to every EVM protocol.

Your contract won't leave the same.

---

*Astrophel — powered by stardust 💀*
*Contact: @astrophel2 on Telegram*
