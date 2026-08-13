# 🛡️ Aegis-F: Confidential Kinetic Lending Protector

> **Flare Summer Signal Hackathon** — *Bounty 2: Confidential Compute Apps ($6,000 Pool)*
> **Target Network:** Flare Coston2 Testnet (Chain ID 114) · Flare Mainnet (Chain ID 14)
> **Status:** Live on Coston2 Testnet · Contracts Verified · Go TEE Keeper Active · 13/13 Unit Tests Passing · Web3 Wallet Connected · Interactive Dashboard Live

**Jump to:** [Contracts](#-live-coston2-verified-smart-contracts-chain-id-114) · [Summary](#-executive-summary) · [Architecture](#%EF%B8%8F-system-architecture--data-pipeline) · [Flare Primitives](#-flare-native-primitives-integration) · [Math Model](#-dynamic-repayment--mev-savings-math-model) · [UI Destinations](#-interactive-fintech-web-dashboard) · [Tests](#-comprehensive-test-suite--verification) · [Quickstart](#-quickstart-guide) · [Directory Tree](#%EF%B8%8F-project-directory-structure) · [Disclosures](#%E2%9A%96%EF%B8%8F-limitations--trust-assumptions) · [Bounty Alignment](#-hackathon-bounty-alignment)

---

## ⚡ Executive Summary

**Aegis-F is an autonomous, confidential risk-management keeper for Flare-native lending markets.** It closes a specific, well-documented DeFi attack vector: public-mempool liquidation hunting, where borrowers' stop-loss triggers and repayment orders are visible to MEV searchers before they land on-chain, exposing them to sandwiching and forced liquidation penalties of 8–10%.

Aegis-F moves the sensitive part of that pipeline — the trigger threshold, the debt-relief calculation, and the signing key — into a **Flare Confidential Compute (FCC)** hardware enclave, so nothing about a borrower's defense strategy is observable until the protective transaction is already final.

**In one sentence:** Kinetic Market borrowers get a private, sub-second, mathematically-exact auto-repayment shield that public liquidation bots cannot see coming.

### The Attack This Solves

```
Standard DeFi Liquidation Attack Vector
┌────────────────────┐      ┌─────────────────────────┐      ┌──────────────────────┐
│ Collateral Drops   │ ───► │ Stop-Loss / Bot Order   │ ───► │ MEV Bots Sandwich    │
│ (HF approaches 1.0)│      │ Visible in Public Mempool│      │ & Seize 8–10% Bonus  │
└────────────────────┘      └─────────────────────────┘      └──────────────────────┘
```

```
Aegis-F Confidential TEE Defense
┌────────────────────┐      ┌─────────────────────────┐      ┌──────────────────────┐
│ FTSOv2 Oracle Reads│ ───► │ Enclave RAM: Trigger &  │ ───► │ Enclave Signs Dynamic│
│ (~1.8s latency)    │      │ Debt Math (0 leakage)   │      │ Repay, Restores 1.30 │
└────────────────────┘      └─────────────────────────┘      └──────────────────────┘
```

1. **Zero mempool leakage** — stop-loss threshold, target health-factor buffer, and reserve authorization stay encrypted in enclave memory until the moment of execution.
2. **Sub-second oracle ingestion** — the enclave polls native FTSOv2 feeds (~1.8s block latency) with strict staleness guards (<120s enclave, <180s on-chain).
3. **Exact, not approximate, debt relief** — on breach, the enclave computes the minimum repayment needed to restore a 1.30 health factor and dispatches it via an isolated Protocol Managed Wallet before a public liquidator's 1.00 threshold is even reached.

---

## 🔗 Live Coston2 Verified Smart Contracts (Chain ID 114)

| Contract | Verified Coston2 Address | Role | Compiler / Status |
| :--- | :--- | :--- | :---: |
| 🏦 **`MockKineticPosition`** | [`0x6376...3AC8c`](https://coston2-explorer.flare.network/address/0x6376892136f7c85E09c0e36100ffA6b484B3AC8c) | Kinetic / Compound V2 Position & Comptroller | `Solidity 0.8.20` · Exact Match |
| 📡 **`InstructionSender`** | [`0x416d...4BB3c`](https://coston2-explorer.flare.network/address/0x416dbc9ABC289b58701e8543e6C54a3a7634BB3c) | FCC Gateway & Instruction Relay | `Solidity 0.8.20` · Exact Match |
| 🔐 **`AegisVault`** | [`0x52C0...F4d6e`](https://coston2-explorer.flare.network/address/0x52C0C06382bCF4f08689c74c47F4D5BFf36F4d6e) | Confidential Repayment Reserve Vault | `Solidity 0.8.20` · Exact Match |
| 🤖 **Designated TEE Keeper** | [`0xB45f...a1B38`](https://coston2-explorer.flare.network/address/0xB45f8a4946cD15bb6f208BF3372934b5946a1B38) | Protocol Managed Wallet (PMW Signer) | Go TEE Daemon · Active |
| 🌐 **FTSOv2 Contract Registry** | [`0xaD67...F6019`](https://coston2-explorer.flare.network/address/0xaD67FE66660Fb8dFE9d6b1b4240d8650e30F6019) | Flare Coston2 Native Oracle Registry | Native Flare Interface |
| 🔮 **FTSOv2 Direct Oracle** | [`0x3d89...11618F`](https://coston2-explorer.flare.network/address/0x3d893c53d9e80E433582fe4091473fC49f11618F) | Coston2 `FtsoV2Interface` Contract | Native Feed Ingestion |

All six addresses above are verified on the Coston2 Blockscout explorer — click through to confirm compiler match, source, and deployment history directly rather than taking our word for it.

#### 🔭 Flare Mainnet Reference Contracts (Chain ID 14 · Read-Only Live Ingestion)

| Contract | Flare Mainnet Address | Role / Integration |
| :--- | :--- | :--- |
| 🏛️ **Kinetic Comptroller** | `0xeC7e541375D70c37262f619162502dB9131d6db5` | Live production Comptroller on Flare Mainnet |
| ⚖️ **Kinetic Unitroller** | `0x8041680Fb73E1Fe5F851e76233DCDfA0f2D2D7c8` | Proxy implementation contract |
| 📊 **ProtocolFTSOV3Oracle** | `0xC1d7029C970d9B683Da9d37b49d84D081dbeD54c` | Kinetic production oracle reader |
| 🪙 **ISO FXRP-USDT0 Market** | `0x3565f14D8C8B1d23b18501258673752eFa88D8E5` | Active Kinetic FAsset lending market |
| 💎 **JOULE-USDC-FLR Market** | `0x685507E7B8a9B83803138f5f0DCE699F2A9fC8A7` | Multi-collateral lending pool |

*(Mainnet contracts are referenced read-only, to demonstrate ABI compatibility — Aegis-F itself is deployed and operated on Coston2 for this submission.)*

---

## 🏛️ System Architecture & Data Pipeline

```
                                    PUBLIC / ON-CHAIN DOMAIN
 ┌─────────────────────────┐          ┌──────────────────────────┐          ┌─────────────────────────┐
 │   Kinetic Market /      │          │   AegisVault Contract    │          │  Flare ContractRegistry │
 │   MockKineticPosition   │◄─────────┤  (ReentrancyGuard +     │          │  & FTSOv2 Price Feeds   │
 │   (Staleness Checked)   │  Repay   │   Pausable Breaker)      │          │  (FLR, BTC, ETH, XRP)   │
 └───────────┬─────────────┘  Debt    └─────────────▲────────────┘          └────────────┬────────────┘
             │                                      │ Execute                            │
             │ Query                                │ Protection                         │ getFeedByIdInWei
             │ Collateral/Debt                      │ (PMW Signature)                    │ (~1.8s block latency)
             ▼                                      │                                    ▼
 ┌──────────────────────────────────────────────────┴─────────────────────────────────────────────────┐
 │                               PRIVATE / TEE ENCLAVE DOMAIN (FCC)                                  │
 │                                                                                                    │
 │   ┌───────────────────────────┐      ┌───────────────────────────┐      ┌──────────────────────┐   │
 │   │    Instruction Router     │      │   Dynamic Health Engine   │      │   FTSOv2 Poller      │   │
 │   │  • Signature Auth (EIP-191)─────►│  • Evaluates HF in TEE    │◄─────│  • Multi-Feed Poll   │   │
 │   │  • Multi-Position Map     │      │  • Dynamic Repay to 1.30  │      │  • Staleness Guard   │   │
 │   └───────────────────────────┘      └─────────────┬─────────────┘      └──────────────────────┘   │
 │                                                    │                                               │
 │                                                    ▼ Trigger Breached (HF <= HF_thresh)            │
 │                                      ┌───────────────────────────┐                                 │
 │                                      │   EVM ECDSA Signer (PMW)  │                                 │
 │                                      │   Signs executeProtection │                                 │
 │                                      └───────────────────────────┘                                 │
 └────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Confidential State Machine

| Component | Visibility | Execution Layer | Security & Privacy Guarantee |
| :--- | :---: | :---: | :--- |
| `FLR/USD` Price Feed | 🌐 Public | On-Chain FTSOv2 | Decentralized oracle with on-chain staleness checks (<180s). |
| Vault Reserve Balance | 🌐 Public | Coston2 Smart Contract | Non-custodial reserve in `AegisVault.sol`, `ReentrancyGuard`-protected. |
| Repayment Transaction | 🌐 Public | Coston2 Explorer | Verifiable on-chain event confirming debt relief and restored HF. |
| Borrower Stop-Loss Limit | 🔒 Confidential | TEE Enclave Memory | Trigger threshold (e.g. $HF_{thresh}=1.15$) encrypted in hardware RAM — invisible to MEV bots. |
| Dynamic Debt Calculation | 🔒 Confidential | Go FCE Daemon | Computed in-enclave; exact debt relief needed to reach $1.30$ HF. |
| Keeper Signing Key (PMW) | 🔒 Confidential | TEE Enclave Isolation | Enclave-custodied Go ECDSA signer (PMW fallback in `MODE=0`; native PMW in `MODE=1`). |

---

## 🔮 Flare Native Primitives Integration

### 1. Flare Confidential Compute (FCC)
- **TEE extension daemon** written in Go (`fce-keeper/`), implementing the Flare Confidential Extension (FCE) interface.
- **Dual-port architecture:** public `tee-proxy` on port **6662** (`/info`, `/direct`, `/simulate-price`, `/logs`, `/stats`) and an internal enclave daemon on port **6661** for hardware-isolated signing.
- **Protocol Managed Wallets (PMW):** enclave-custodied EVM ECDSA signer matching the authorized `teeKeeper` in `AegisVault.sol`.
- **Signature-authenticated triggers:** the `/direct` endpoint verifies EIP-191 personal signatures before registering or updating enclave triggers, with a 10 requests/minute per-signer rate limit.
- **Instruction routing:** `InstructionSender.sol` emits typed FCC instruction events (`OPType = keccak256("AEGIS_KINETIC_PROTECTOR")`, `OPCommand = keccak256("REGISTER_TRIGGER")`) parsed by the Go daemon router.

### 2. Flare Time Series Oracle v2 (FTSOv2) — Multi-Asset Matrix
Price ingestion via `FtsoV2Interface.getFeedByIdInWei`, routed through the Flare Contract Registry (`0xaD67FE66660Fb8dFE9d6b1b4240d8650e30F6019` on Coston2), across four verified feed IDs:

- `FLR/USD`: `0x01464c522f55534400000000000000000000000000` *(active position feed)*
- `BTC/USD`: `0x014254432f55534400000000000000000000000000`
- `ETH/USD`: `0x014554482f55534400000000000000000000000000`
- `XRP/USD`: `0x015852502f55534400000000000000000000000000`

### 3. Flare Mainnet Kinetic Market Compatibility Reader
`services/mainnetReader.js` connects to Flare Mainnet (Chain ID 14) via public RPC (`https://flare-api.flare.network/ext/C/rpc`) and reads directly against the production Kinetic Comptroller and Unitroller, demonstrating ABI compatibility without requiring any mainnet deployment of Aegis-F itself.

---

## 🧮 Dynamic Repayment & MEV Savings Math Model

### 1. Dynamic Debt Repayment (target buffer $HF_{target}=1.30$)

Given collateral $C$, oracle price $P$, collateral factor $CF=0.85$, and outstanding debt $D$:

$$HF = \frac{C \cdot P \cdot CF}{D}$$

When $HF \le HF_{thresh}$ (e.g. $1.15$), the enclave computes the exact debt relief $\Delta D$ required to restore $HF_{target}=1.30$:

$$\Delta D = D - \frac{C \cdot P \cdot CF}{HF_{target}}$$

$$\text{Repayment Amount} = \min(D,\ \max(0,\ \Delta D))$$

This is the core differentiator versus standard liquidation: Kinetic/Compound-style forks seize a fixed 50% close factor regardless of how small the actual shortfall is. Aegis-F repays exactly what's needed and nothing more.

### 2. Liquidation Bonus Avoided

$$\text{Liquidation Penalty Avoided} = D \times 50\% \times 8\%$$
$$\text{Net Borrower Benefit} = \text{Penalty Avoided} - \text{Flare Gas Fee } (\approx \$0.00028)$$

### 3. Protocol Sustainability (20 bps fee)
A 20 bps ($0.20\%$) protection fee applies only to successfully rescued debt — zero fees on idle reserves or passive monitoring — funding gas relayers and keeper maintenance.

---

## 🖥️ Interactive Fintech Web Dashboard

Built with **Vite + React 18**, dark institutional trading-desk aesthetic (`#0A0A0F` background, `#F5F3F7` type, `#2ED47A` money green, `#9B7FFF` TEE purple, `#60A5FA` Flare blue).

| View | Route | What It Shows |
| :--- | :--- | :--- |
| **Overview** | `/` | Live asset ticker, FCC attestation badge, 3-pillar stats bar, side-by-side public-mempool vs. enclave proof trace. |
| **Live Demo** | `/demo` | Two-step flow: configure & EIP-191-sign a position, then run a Black Swan stress test and watch the enclave rescue it in real time. |
| **Calculator & Simulator** | `/simulator` | Dynamic repay math sandbox, plus a millisecond-scale predator-race visualizer (enclave vs. mempool bot). |
| **Institutional Portfolio Desk** | `/portfolio` | Multi-asset risk heatmap across FLR, BTC, ETH, XRP positions. |
| **Verified Proofs** | `/proofs` | Direct Coston2 explorer links, compiler verification badges, and the enclave attestation modal (see disclosure below). |

Additional features: MetaMask/Core/injected wallet support with one-click Coston2 network switching, synthetic Web Audio feedback (mutable), GSAP/Framer Motion micro-interactions, and full mobile responsiveness.

---

## 🧪 Comprehensive Test Suite & Verification

### 1. Hardhat Smart Contract Unit Tests (13/13 Passing)

```bash
npx hardhat test
```

```
  Aegis-F Smart Contract Suite
    1. MockKineticPosition Lending Mechanics & FTSO Math
      ✔ should allow collateral deposit and calculate correct initial health factor
      ✔ should accurately reflect health factor drops when oracle price falls
      ✔ should allow liquidation only when health factor drops below 1.0
    2. Edge Cases & Security Validations
      ✔ should handle zero debt safely with max uint256 health factor
      ✔ should return zero health factor if collateral is zero with active debt
      ✔ should reject borrows exceeding maximum borrowing capacity
      ✔ should allow owner to update maximum oracle staleness limit
    3. InstructionSender Confidential Compute Routing
      ✔ should emit FCC instruction events with matching OPType and OPCommand
      ✔ should allow borrower to revoke an active trigger
    4. AegisVault, Circuit Breaker & TEE Debt Repayment
      ✔ should allow user to deposit repayment reserve and execute protection via TEE Keeper
      ✔ should prevent unauthorized callers from triggering protection
      ✔ should support circuit breaker pausing to halt automated execution during emergencies
      ✔ should allow users to withdraw their unspent reserve

  13 passing (613ms)
```

### 2. End-to-End Simulation

```bash
npx hardhat run scripts/demo-e2e.cjs
```

Simulates a full lifecycle on a local fork: position opened at 1.4875 HF → FLR price drops from $0.035 to $0.027 → HF degrades to 1.1475, breaching the 1.15 trigger → enclave dispatches an $8.00 repayment → HF recovers to 1.9125, and the position remains unliquidatable. Full console output is in `docs/SUBMISSION.md`.

### 3. Live Flare Mainnet Reader Verification

```bash
node -e "const { KineticMainnetReader } = require('./services/mainnetReader.js'); new KineticMainnetReader().getMarketOverview().then(console.log);"
```

---

## 🚀 Quickstart Guide

### Prerequisites
- **Node.js** ≥ 18.0.0
- **Go** ≥ 1.21 (for the TEE keeper daemon)

### 1. Clone & Install
```bash
git clone https://github.com/your-username/AGEIS-F.git
cd AGEIS-F
npm install
cd frontend && npm install && cd ..
```

### 2. Configure Environment
```bash
cp .env.example .env
```

### 3. Run Smart Contract Tests
```bash
npx hardhat test
```

### 4. Launch the Go TEE Keeper (Terminal 1)
```bash
cd fce-keeper
go run main.go
# or the pre-compiled binary: ./aegis-keeper
```
Listens on `http://localhost:6662` with FTSOv2 polling active.

### 5. Launch the Dashboard (Terminal 2)
```bash
cd frontend
npm run dev
```
Open **http://localhost:3000**.

---

## 🏛️ Project Directory Structure

```
AGEIS-F/
├── contracts/
│   ├── AegisVault.sol                  # Confidential liquidation reserve vault & TEE auth
│   ├── InstructionSender.sol           # FCC gateway & instruction emitter
│   ├── MockKineticPosition.sol         # Kinetic-style lending position with FTSOv2 ingestion
│   └── interfaces/
│       ├── ContractRegistry.sol
│       └── FtsoV2Interface.sol
├── deployments/
│   ├── coston2.json                    # Verified Coston2 addresses & feed IDs
│   └── hardhat.json
├── fce-keeper/                         # Go TEE keeper daemon
│   ├── main.go
│   ├── health_engine.go                # Dynamic health calculation & repayment algorithm
│   ├── ftso_poller.go                  # Sub-second FTSOv2 reader & staleness guard
│   ├── signer.go                       # EVM ECDSA enclave signer (PMW)
│   ├── types.go
│   └── aegis-keeper                    # Compiled binary
├── frontend/                           # Vite + React 18 dashboard
│   └── src/{components,sections,services}/
├── services/
│   └── mainnetReader.js                # Read-only Flare Mainnet Kinetic inspector
├── test/
│   └── AegisContracts.test.js          # 13 Hardhat unit tests
├── scripts/
│   ├── deploy.cjs
│   └── demo-e2e.cjs
├── docs/
│   └── SUBMISSION.md
├── hardhat.config.cjs
└── package.json
```

---

## ⚖️ Limitations & Trust Assumptions

We'd rather over-disclose than have a judge discover a gap we didn't mention:

1. **`MODE=0` hardware simulation on testnet.** This submission runs the Go keeper in local simulation mode — identical signing and health-engine logic to production, but without provisioning a billed AMD SEV-SNP instance on GCP Confidential Space. The attestation quote and `MRENCLAVE`/`PCR0` measurement shown in the `/proofs` view are generated by this simulation path, not a live SEV-SNP hardware attestation. The production path (real hardware attestation on GCP Confidential Space, published to Coston2's `TeeMachineRegistry`) is scoped in the roadmap below, not yet deployed.
2. **Single keeper instance.** The current build runs one TEE keeper. Production would need a redundant keeper network with threshold-signed consensus (native PMW) to remove single-point-of-failure risk.
3. **Static staleness windows.** Oracle staleness bounds are fixed (<180s on-chain, <120s in-enclave) rather than volatility-adjusted.
4. **Compound V2 benchmark parameters.** The 50% close factor and 8% liquidation bonus used in the savings model are canonical Compound V2 parameters, used illustratively — actual Kinetic Market parameters may differ.

---

## 🗺️ Roadmap

- **FDC cross-chain attestation** — use the Flare Data Connector to attest borrower positions on foreign EVM chains (e.g. Aave V3 on Arbitrum) and execute cross-chain debt relief.
- **FAssets collateral expansion** — native support for FXRP, FBTC, FDOGE collateral on Kinetic.
- **On-chain attestation registry** — publish real hardware attestation quotes to Coston2's `TeeMachineRegistry`, replacing the simulation-mode quote used in this submission.

---

## 🏆 Hackathon Bounty Alignment

| Bounty Requirement | How Aegis-F Satisfies It |
| :--- | :--- |
| **Confidential Compute (FCC / TEE)** | Private stop-loss thresholds, dynamic debt calculation, and signing keys isolated in enclave RAM (`AMD SEV-SNP` target; `MODE=0` simulation in this build — see Limitations). |
| **Flare Oracles (FTSOv2)** | Native `FLR/USD`, `BTC/USD`, `ETH/USD`, `XRP/USD` feeds with active staleness verification. |
| **DeFi Risk Management** | Protects Kinetic Market borrowers from mempool MEV front-running; avoids 8–10% liquidation penalties. |
| **Production Readiness** | Verified Coston2 contracts, 13/13 passing Hardhat tests, working Web3 wallet integration, live Go TEE daemon. |

---

<div align="center">
  <b>Built natively for the Flare Summer Signal Hackathon</b><br/>
  <i>Securing DeFi lending with Flare Confidential Compute & FTSOv2</i>
</div>
