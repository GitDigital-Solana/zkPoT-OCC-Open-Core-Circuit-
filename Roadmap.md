ROADMAP.md — Aurora zk‑Cryptography Framework

Aurora’s roadmap is organized into three layers of progression:

- Protocol Layer — circuits, verification, attestation  
- Platform Layer — SDK, daemon, registry, risk engine  
- Ecosystem Layer — GitDigital integrations, enterprise modules, multi‑chain support  

Each quarter includes deliverables, milestones, and audit‑ready acceptance criteria.

---

Q2 2026 — Stabilization & Circuit Consolidation

Protocol Layer
- Consolidate all ZK circuits into a unified versioned registry  
- Introduce circuit metadata (hash, version, proving key ID)  
- Add deterministic proof serialization format  
- Add proof size reduction pass (WASM‑friendly)

Platform Layer
- Daemon v2:
  - Parallel verification queues  
  - Configurable worker pools  
  - Deterministic batching  
- SDK improvements:
  - WASM performance pass  
  - Mobile‑safe bindings  
  - Stronger type‑safe error surface  

Ecosystem Layer
- GitDigital identity binding v1  
- Developer onboarding examples (wallets, exchanges, dApps)

Acceptance Criteria
- All circuits versioned and discoverable  
- Daemon handles 10k proofs/hour on commodity hardware  
- SDK stable in browser + Node + React Native  

---

Q3 2026 — Expansion & Multi‑Proof Capabilities

Protocol Layer
- Multi‑proof aggregation support  
- Proof‑of‑identity + proof‑of‑attributes combined circuits  
- Circuit upgrade governance (time‑locked)

Platform Layer
- Risk Engine v2:
  - Behavioral scoring  
  - Optional device/IP correlation  
  - Pluggable risk modules  
- Attestation v2:
  - Multi‑attestation bundles  
  - Expiry + epoch enforcement  

Ecosystem Layer
- GitDigital governance integration  
- GitDigital multi‑AI orchestration hooks  
- Exchange‑grade compliance workflows

Acceptance Criteria
- Multi‑proof aggregation reduces RPC load by 40%  
- Risk engine supports pluggable scoring modules  
- Governance can approve/deny circuit upgrades  

---

Q4 2026 — zk‑Compression, Multi‑Chain, Enterprise

Protocol Layer
- zk‑Compression integration for:
  - Proof batching  
  - State compression  
  - Lower on‑chain footprint  
- Multi‑chain verification (Ethereum, Solana, L2s)

Platform Layer
- Daemon v3:
  - Horizontal scaling  
  - Multi‑chain attestation routing  
  - Multi‑daemon quorum mode (M‑of‑N attestations)  
- SDK v3:
  - Unified multi‑chain client  
  - Circuit auto‑discovery  
  - Attestation caching  

Ecosystem Layer
- Enterprise compliance module:
  - SOC2‑aligned audit logs  
  - Enterprise‑grade risk scoring  
  - Policy‑driven attestation rules  
- GitDigital personal stability layer integration  
- GitDigital professional services integration

Acceptance Criteria
- zk‑Compression reduces on‑chain storage by 70%  
- Multi‑chain verification stable across Solana + 1 EVM chain  
- Enterprise module passes internal audit  

---

2027+ — Long‑Term Vision

Protocol Layer
- Universal proof format  
- Recursive proofs for identity + behavior  
- Zero‑knowledge revocation proofs  

Platform Layer
- Fully decentralized attestation network  
- Multi‑daemon consensus  
- Zero‑trust daemon clusters  

Ecosystem Layer
- GitDigital global trust graph  
- Federated identity across chains  
- AI‑driven risk modeling (non‑sensitive, privacy‑preserving)

---

Strategic Themes

1. Determinism
Every component — circuits, daemon, SDK, registry — must behave deterministically.

2. Auditability
All state transitions must be reconstructible from logs + on‑chain history.

3. Modularity
Each layer must be independently testable, replaceable, and upgradable.

4. Privacy
Zero‑knowledge first. No sensitive data ever leaves the client.

5. Ecosystem Integration
Aurora is the trust layer for GitDigital’s multi‑AI, multi‑chain ecosystem.

---
