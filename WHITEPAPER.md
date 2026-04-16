# Limbo: Automated Adversarial Security Analysis for EVM Smart Contracts

**Version:** 1.0
**Author:** Astrophel
**Contact:** @astrophel2 (Telegram)
**Date:** 2025

---

## Abstract

Smart contract exploits have drained over $3 billion from DeFi protocols since 2020. The majority of these exploits targeted economic logic vulnerabilities — attack surfaces that existing automated tools fail to detect reliably. Manual audits are expensive, slow, and inaccessible to smaller protocols. Limbo is an automated security analysis system delivered through Telegram that combines static analysis, adversarial fuzzing, symbolic execution and formal verification into a single pipeline. A cross-validation correlation engine eliminates false positives, Foundry-based exploit verification confirms real vulnerabilities, and AI-powered reporting translates raw findings into actionable audit documents. Limbo makes institutional-grade security analysis accessible to any EVM protocol at a fraction of the cost and time of a manual audit.

---

## 1. Introduction

### 1.1 The Problem

The EVM smart contract ecosystem has accumulated hundreds of billions of dollars in total value locked. This concentration of value has made protocols a primary target for adversarial actors, resulting in a continuous stream of high-profile exploits.

Post-mortem analysis of major exploits reveals a consistent pattern: the vulnerabilities that cause the most damage are rarely simple code bugs. They are economic logic attacks — price manipulation through flash loans, oracle dependencies that can be influenced within a single transaction, incentive misalignments that allow an attacker to extract value at the expense of other participants.

Existing automated tools such as Slither, Mythril and Echidna are highly effective at detecting known code-level vulnerability patterns. They are significantly less effective at reasoning about economic attack surfaces because this requires understanding the protocol's intended invariants — properties that should always hold regardless of what sequence of transactions an adversary executes.

Manual audits fill this gap but introduce two critical constraints: cost and time. A professional audit from a reputable firm costs between $10,000 and $100,000 and takes weeks to schedule. This creates a two-tier security landscape where well-funded protocols receive thorough reviews and smaller protocols deploy with minimal scrutiny.

### 1.2 The Opportunity

The gap between automated tools and manual audits is not primarily a technical gap — it is an integration and intelligence gap. The tools exist. What is missing is a system that:

1. Runs all relevant tools with protocol-aware configuration
2. Cross-validates findings to eliminate noise
3. Verifies that flagged vulnerabilities are actually exploitable
4. Delivers results in a format that protocol teams can act on

Limbo is built on the thesis that this gap can be closed with careful engineering, without requiring a human auditor for the majority of common vulnerability classes.

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
                    └─────────────────┘
```

### 2.2 Technology Stack

| Component | Technology | Rationale |
|---|---|---|
| Bot runtime | Rust + Teloxide | Memory safety, concurrency, performance |
| Async executor | Tokio | Parallel tool execution |
| Static analysis | Slither | Industry standard, extensible detector API |
| Fuzzing | Echidna | Property-based, supports custom invariants |
| Symbolic execution | Mythril | Deep path exploration |
| Formal verification | Halmos | Mathematical proof of invariants |
| PoC generation | Foundry | Industry standard testing framework |
| AI enrichment | Gemini 3 Flash | Cost-effective, strong coding capability |
| Payment | ethers-rs | On-chain USDT monitoring |
| Key security | AES-256-GCM + zeroize | Private keys encrypted, wiped post-sweep |
| PDF generation | printpdf | Native Rust, no external dependencies |

---

## 3. Scanner Design

### 3.1 Protocol Type Detection

Before any security tool runs, Limbo determines the protocol type using AI analysis of the contract source. This classification drives the configuration of subsequent analysis stages.

Supported protocol types: `lending`, `amm`, `governance`, `bridge`, `staking`, `nft`, `token`, `defi`

### 3.2 Static Analysis

Slither is executed with informational findings suppressed to reduce noise. Custom detectors targeting DeFi-specific patterns supplement the default ruleset. These custom detectors are derived from real findings discovered by the Limbo team during competitive audit participation on Code4rena, Immunefi and Cantina.

### 3.3 Adversarial Fuzzing

Echidna is configured with protocol-aware invariants rather than generic properties. This is the primary mechanism by which Limbo detects economic logic vulnerabilities that code-level analysis cannot find.

**Invariant strategy by protocol type:**

*Lending protocols:*
```
∀ state s after any sequence of transactions T:
  total_borrows(s) ≤ total_deposits(s) − reserve(s)
  
∀ flash_loan F:
  balance_before ≤ balance_after + fee
  
∀ liquidation L:
  health_factor(borrower) < threshold → liquidation_valid
```

*AMM protocols:*
```
∀ swap(tokenIn, amountIn) → amountOut:
  k_before ≤ k_after  (invariant preservation)
  
∀ sequence S of swaps:
  price_impact(S) matches expected_slippage(S)
```

*Bridge protocols:*
```
∀ lock(amount, destination):
  outstanding_claims(destination) ≤ locked_total(source)
```

The fuzzer uses an adversarial sender pool simulating multiple independent actors including flash loan providers, to test whether invariants hold under coordinated attack scenarios.

### 3.4 Symbolic Execution

Mythril performs bounded symbolic execution with a 90-second timeout per contract. It is particularly effective at finding vulnerabilities that require specific transaction sequences or precise value manipulation that random fuzzing is unlikely to reach.

### 3.5 Formal Verification

Halmos provides unbounded formal verification for selected properties. Unlike fuzzing, which tests many inputs, formal verification proves that a property holds for all possible inputs. This is applied to the most critical invariants identified during protocol type classification.

### 3.6 Execution Model

All tools run concurrently using Tokio async workers with hard timeout enforcement:

```
Slither:  120 seconds
Echidna:  300 seconds
Mythril:   90 seconds
Halmos:   120 seconds
```

If any tool exceeds its timeout the process is killed and the scan continues with results from the remaining tools. This ensures the server cannot be OOM-killed by a complex contract.

---

## 4. Correlation Engine

### 4.1 Problem

Running multiple security tools against the same contract produces overlapping and contradictory output. The same vulnerability may be named differently by different tools. Some findings are genuine vulnerabilities. Others are false positives caused by tool limitations or insufficient context. Without correlation, the PDF would be noise.

### 4.2 Approach

The correlation engine processes all raw findings through five stages:

**Stage 1 — Normalisation**
Finding titles are normalised to a canonical form, stripping tool prefixes and mapping synonyms (e.g. "reentrance", "re-entrancy" and "reentrancy" all map to the same canonical key).

**Stage 2 — Grouping**
Findings with the same normalised title are grouped. The group tracks which tools contributed each finding.

**Stage 3 — Confidence scoring**

| Condition | Confidence |
|---|---|
| Foundry exploit ran successfully | CONFIRMED |
| 2+ tools independently found it | HIGH CONFIDENCE |
| 1 tool found it | SINGLE TOOL |
| Info severity + single tool | FALSE POSITIVE |

**Stage 4 — Severity escalation**
Confirmed findings have their severity escalated: Medium → High, High → Critical. This reflects the increased certainty that a confirmed, exploitable vulnerability represents.

**Stage 5 — Deduplication**
Within each confidence tier, findings are deduplicated and sorted by severity.

### 4.3 Foundry Verification

For all findings that receive a PoC from the AI layer, Limbo attempts to execute the exploit using Foundry against a local fork. The PoC test is written to fail (assert revert) when the exploit succeeds. If Forge reports a test failure, the finding is marked CONFIRMED.

This binary verification is the most important signal in the entire pipeline. A CONFIRMED finding means an attacker could have run this exact code and extracted funds.

---

## 5. AI Layer

### 5.1 Role

The AI layer does not perform security analysis. Security analysis is the exclusive domain of the deterministic tools described above. The AI layer has three responsibilities:

1. Protocol type classification from source code
2. Translation of raw tool output into human-readable explanations
3. Generation of Foundry PoC code for enrichment of findings

### 5.2 Model Selection

Limbo uses Gemini 3 Flash for the AI layer, selected for its strong performance on SWE-bench coding benchmarks and cost efficiency suitable for a per-scan pricing model. The system is designed to allow model substitution; Claude Haiku 4.5 will be evaluated as volume scales.

### 5.3 Prompt Architecture

Each finding is enriched with a structured prompt that provides:
- Full contract source (truncated to 6000 tokens for cost efficiency)
- Finding title, severity, description and location
- Protocol type context
- Tier-appropriate depth instruction
- Required output schema (JSON)

The depth instruction varies by tier to control analysis thoroughness and token consumption.

### 5.4 Limitations

The AI layer can generate incorrect PoC code or mischaracterise vulnerabilities. This is why AI output is used for enrichment only — the underlying security determination comes from deterministic tools. All AI-generated PoC code that is not Foundry-verified is clearly labelled as unverified in the PDF.

---

## 6. Payment Architecture

### 6.1 Design Principles

- Zero manual intervention for Small and Pro tiers
- Zero persistent storage of sensitive material
- Cryptographic guarantees for private key security
- Configurable confirmation thresholds per network

### 6.2 Session Lifecycle

```
1. User selects tier and network
2. Limbo generates ephemeral wallet (ethers-rs LocalWallet)
3. Private key encrypted with AES-256-GCM, ephemeral key
4. Encrypted key stored in session alongside wallet address
5. User sends USDT to wallet address
6. Monitor polls blockchain every 30 seconds
7. On N confirmations: scan begins
8. On scan completion: sweep transaction constructed
9. Sweep transaction signed with decrypted key
10. Key zeroized from memory immediately
11. USDT arrives in operations wallet
```

### 6.3 Confirmation Thresholds

| Network | Confirmations | Rationale |
|---|---|---|
| Polygon | 5 | Fast finality, low reorg risk |
| Arbitrum | 10 | Optimistic rollup, conservative threshold |
| Ethereum | 12 | Standard finality assumption |

### 6.4 Gas Management

For Polygon and Arbitrum, Limbo maintains a gas feeder mechanism. Before constructing the sweep transaction, a small amount of native token (0.1 MATIC / 0.001 ETH) is sent to the ephemeral wallet to cover transaction fees. This cost is absorbed into Limbo's operating margin.

For Ethereum, the user is required to include sufficient ETH alongside their USDT payment to cover gas.

### 6.5 Session Expiry

Sessions expire after 120 minutes. Expired sessions are removed from memory and the user is notified. Funds sent to expired session addresses can be recovered on request.

---

## 7. Security Properties

| Property | Mechanism |
|---|---|
| Private key confidentiality | AES-256-GCM encryption + zeroize on use |
| Payment replay prevention | Unique address per session |
| False payment prevention | Minimum confirmation thresholds |
| Denial of service prevention | Hard tool timeouts |
| Disk exhaustion prevention | Rust RAII tempdir cleanup |
| Memory exhaustion prevention | ZIP stream extraction, node_modules skip |
| Admin impersonation prevention | Telegram ID verification (cannot be spoofed) |
| Contract self-scan prevention | Configurable blacklist |

---

## 8. Competitive Landscape

| Tool | Automated | Economic Logic | PDF Output | Telegram Native | Price |
|---|---|---|---|---|---|
| **Limbo** | ✓ | ✓ | ✓ | ✓ | $50–$150 |
| Slither (standalone) | ✓ | ✗ | ✗ | ✗ | Free |
| Mythril (standalone) | ✓ | ✗ | ✗ | ✗ | Free |
| Certora | ✓ | Partial | ✗ | ✗ | Enterprise |
| Manual audit | ✗ | ✓ | ✓ | ✗ | $10k–$100k |

Limbo occupies a unique position: automated delivery with economic logic analysis at a price point accessible to protocols of any size.

---

## 9. Roadmap

### Phase 1 — Launch (Current)
- EVM smart contract support (Solidity)
- Polygon + Arbitrum primary networks
- Small / Pro / Enterprise tiers
- Correlation engine
- Foundry verification
- Automated USDT payment

### Phase 2 — Growth
- Partnership integrations (Cantina, Sherlock, HackenProof, Cyfrin)
- Referral program (15% revenue share)
- Limbo Verified badge for audited protocols
- Public risk leaderboard
- Historical scan database

### Phase 3 — Multichain Monday
- Solana smart contract support (Anchor programs)
- Rust-based contract analysis toolchain
- Cross-chain composability analysis

### Phase 4 — Platform
- Announcement and channel management system
- White-label offering for audit firms
- API access for CI/CD pipeline integration

---

## 10. Conclusion

Limbo addresses the automation gap in smart contract security by combining deterministic analysis tools with a correlation layer, exploit verification and AI-assisted reporting. The result is a security analysis product that produces output comparable to a preliminary manual audit at a fraction of the cost and time, delivered natively within the communication platform where the DeFi developer community already operates.

The economic attack surfaces that have caused the most damage to DeFi protocols are detectable with protocol-aware fuzzing. The false positive problem that makes raw tool output unusable is solvable with cross-validation. The accessibility barrier that excludes smaller protocols from serious security review is removable with automation and appropriate pricing.

Limbo makes institutional-grade security analysis available to every EVM protocol. Your contract won't leave the same.

---

*Astrophel — powered by stardust 💀*
*Contact: @astrophel2 on Telegram*
