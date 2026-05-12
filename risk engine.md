1. Expanded security model

1.1 Assets and trust boundaries

Core assets:

- On‑chain state:
  - Compliance registry accounts
  - Attestation records
  - Program configuration (admin, risk params, circuit IDs)
- Off‑chain assets:
  - Daemon signing keys
  - Verification configuration (circuit versions, risk rules)
  - Logs and audit trails
- Client‑side assets:
  - User keys (wallets)
  - Generated proofs (before submission)
  - Session tokens / nonces

Trust boundaries:

- Client ↔ Daemon API
- Daemon ↔ Solana RPC
- Daemon internal queues ↔ verification workers
- On‑chain program ↔ other Solana programs

Each boundary is where you enforce auth, integrity, replay protection, and rate limiting.

---

1.2 Threat classes and mitigations

A. Malicious prover / client

Goal: Submit invalid or manipulated proofs, or bypass risk checks.

Mitigations:

- Deterministic verification in daemon:
  - Verification logic is pure and stateless per proof.
- Canonical proof format:
  - Strict schema validation before verification.
- Risk engine gating:
  - No attestation if risk score exceeds configured thresholds.
- On‑chain re‑verification (lightweight):
  - Program checks:
  \[
  \text{hash(proof)} = \text{stored\_hash}
  \]
  and validates attestation signature and freshness.

---

B. Replay attacks

Goal: Reuse old valid proofs/attestations to gain access or re‑assert a prior state.

Mitigations:

- Nonces and epochs:
  - Each proof bound to:
  \[
  (\text{subject}, \text{nonce}, \text{epoch})
  \]
- On‑chain freshness checks:
  - Program stores last used nonce/epoch per subject.
- Attestation expiry:
  - Attestations include valid_until timestamp/slot.
- Idempotent registry updates:
  - If same (subject, proof_id) seen again → no state change.

---

C. Sybil and identity abuse

Goal: Create many identities to bypass risk or limits.

Mitigations:

- Identity binding:
  - Registry binds:
  \[
  \text{wallet\pubkey} \leftrightarrow \text{identity\handle}
  \]
- Cross‑proof correlation in daemon:
  - Risk engine tracks:
    - Proof frequency per subject
    - Shared attributes (e.g., device, IP class, optional signals)
- Rate limiting per subject / per IP / per wallet:
  - Configurable thresholds in daemon.

---

D. Daemon compromise

Goal: Attacker controls daemon to issue false attestations or bypass verification.

Mitigations:

- Dedicated signing keys with strict scope:
  - Daemon uses attestation key only for signing results.
  - Key is not a Solana authority key.
- Key management:
  - HSM or KMS‑backed keys where possible.
  - Rotation protocol:
    - New key announced on‑chain in config.
    - Grace period where both old and new accepted.
- On‑chain verification of daemon identity:
  - Program stores allowed attestation pubkeys.
  - Any attestation must be signed by one of these keys.
- Audit logging:
  - Every attestation logged with:
    - Proof hash
    - Subject
    - Risk score
    - Daemon instance ID
- Multi‑daemon quorum (optional future):
  - Require M‑of‑N attestations for high‑risk operations.

---

E. Network‑level attacks (MITM, tampering)

Goal: Intercept or modify traffic between client and daemon, or daemon and RPC.

Mitigations:

- TLS everywhere for client ↔ daemon.
- Signed payloads:
  - Client signs proof submission with wallet key:
  \[
  \text{sig} = \text{Sign}_{\text{wallet}}(\text{hash(proof || nonce)})
  \]
- Daemon verifies client signature before processing.
- RPC integrity:
  - Use trusted RPC providers.
  - Optional: cross‑check critical writes via multiple RPC endpoints.

---

F. On‑chain program abuse

Goal: Malicious programs or callers try to misuse Aurora’s program.

Mitigations:

- Strict instruction validation:
  - All accounts and PDAs derived and checked.
- Access control:
  - Only specific instructions can mutate registry.
  - Optional: allow‑list of caller programs for certain ops.
- Config immutability / controlled mutability:
  - Critical config changes gated by:
    - Multisig
    - Time‑lock
    - Governance program

---

1.3 Security invariants

You can literally list these in SECURITY.md:

- Invariant 1: No registry entry is updated without a valid, non‑expired attestation signed by an authorized daemon key.
- Invariant 2: No attestation is produced without a successfully verified proof and a risk score within configured bounds.
- Invariant 3: Each (subject, proof_id) pair can only advance state once (idempotent updates).
- Invariant 4: All state transitions are fully reconstructible from logs + on‑chain history.

---

2. More diagrams

2.1 Sequence diagram: proof → attestation → registry

`mermaid
sequenceDiagram
    participant U as User / Wallet
    participant S as Aurora SDK
    participant D as Verification Daemon
    participant R as Risk Engine
    participant P as Solana Program
    participant C as Compliance Registry Account
    
    U->>S: Request ZK proof generation
    S->>S: Generate proof (local / circuit)
    S->>U: Request wallet signature over proof hash
    U->>S: Signed payload
    S->>D: Submit {proof, signature, nonce, subject}
    D->>D: Validate signature & schema
    D->>R: Evaluate risk(proof, subject)
    R-->>D: risk_score, decision
    D->>D: Verify proof (ZK verification)
    D->>D: Build attestation {proofhash, subject, riskscore, valid_until}
    D->>D: Sign attestation with daemon key
    D->>P: Submit on‑chain tx with attestation
    P->>P: Verify daemon signature & freshness
    P->>C: Update compliance registry state
    P-->>D: Success / failure
    D-->>S: Result (accepted / rejected)
    S-->>U: Final status
`

---

2.2 Component diagram: security‑relevant pieces

`mermaid
flowchart LR
    
    subgraph Client
        W[Wallet Keys]
        SDK[Aurora SDK<br/>TS + WASM]
    end
    
    subgraph OffChain[Off‑Chain]
        API[Daemon API Gateway]
        V[Verifier Workers]
        RE[Risk Engine]
        AK[Attestation Key Store]
        LOG[Audit Logs]
    end
    
    subgraph OnChain[Solana]
        PROG[Aurora Program]
        REG[Compliance Registry Accounts]
        CFG[Config PDA<br/>(allowed keys, params)]
    end
    
    W --> SDK
    SDK --> API
    API --> V
    V --> RE
    V --> AK
    AK --> PROG
    RE --> PROG
    PROG --> REG
    PROG --> CFG
    PROG --> LOG:::offchain
    
    classDef offchain fill=#222,color=#fff,stroke=#555;
`

(You can drop the classDef if you want it simpler.)

---

2.3 Trust boundary diagram

`mermaid
flowchart TB
    
    subgraph Zone1[User / Client Zone]
        C1[Wallet + SDK]
    end
    
    subgraph Zone2[Daemon Zone]
        G[API Gateway]
        Q[Queues]
        VW[Verifier Workers]
        RK[Risk Engine]
        KS[Key Store]
    end
    
    subgraph Zone3[Solana Network]
        SP[Aurora Program]
        SR[Registry Accounts]
    end
    
    C1 -->|TLS + signed payloads| G
    G --> Q
    Q --> VW
    VW --> RK
    VW --> KS
    KS -->|Signed attestations| SP
    RK --> SP
    SP --> SR
`

You can annotate this later with:

- Boundary 1: Client ↔ Daemon (TLS, signatures, rate limiting)  
- Boundary 2: Daemon ↔ Solana (RPC integrity, key management)  

---

2.4 Optional: risk engine decision tree (simple)

`mermaid
flowchart TD
    
    A[Verified proof?] -->|No| R1[Reject]
    A -->|Yes| B[Compute risk score]
    
    B --> C{Score <= low_threshold?}
    C -->|Yes| L[Accept - Low Risk]
    C -->|No| D{Score <= high_threshold?}
    D -->|Yes| M[Accept - Medium Risk<br/>Flag for review]
    D -->|No| H[Reject - High Risk]
`

---

3. Where this plugs into the repo

Concrete places to use this:

- docs/architecture.md
  - Include component diagram + sequence diagram.
- SECURITY.md
  - Include threat classes, mitigations, and invariants.
- docs/risk-engine.md
  - Include risk decision tree.
- README.md (later)
  - Include one high‑level architecture diagram + one short sequence diagram.
The risk engine is a deterministic scoring system that evaluates:

- Proof metadata  
- Submission frequency  
- Identity binding  
- Optional device/IP signals  
- Historical behavior  

Decision Tree

`mermaid
flowchart TD

    A[Verified proof?] -->|No| R1[Reject]
    A -->|Yes| B[Compute risk score]

    B --> C{Score <= low_threshold?}
    C -->|Yes| L[Accept - Low Risk]
    C -->|No| D{Score <= high_threshold?}
    D -->|Yes| M[Accept - Medium Risk<br/>Flag for review]
    D -->|No| H[Reject - High Risk]
`

---

6. Security Architecture

Aurora enforces four core invariants:

1. No registry update occurs without a valid, non‑expired attestation.  
2. No attestation is produced without a verified proof + acceptable risk score.  
3. Each (subject, proof_id) pair is idempotent.  
4. All state transitions are fully auditable.

---

6.1 Threat Classes & Controls

Malicious Prover
- Strict schema validation  
- Deterministic verification  
- Risk gating  
- On‑chain attestation verification  

Replay Attacks
- Nonces  
- Epochs  
- Attestation expiry  
- On‑chain freshness checks  

Sybil Attacks
- Identity binding  
- Rate limiting  
- Cross‑proof correlation  

Daemon Compromise
- Attestation key isolation  
- Key rotation  
- On‑chain allow‑list of daemon keys  
- Audit logs  

Network Attacks
- TLS  
- Signed payloads  
- RPC integrity checks  

On‑Chain Abuse
- PDA‑based access control  
- Strict instruction validation  
- Config immutability  

---

7. Data Flow Diagram

`mermaid
flowchart LR

    subgraph Client
        P[Proof Generator]
        S[SDK]
        W[Wallet]
    end

    subgraph OffChain
        G[API Gateway]
        Q[Queue]
        V[Verifier]
        R[Risk Engine]
        K[Attestation Key]
    end

    subgraph OnChain
        O[Aurora Program]
        C[Compliance Registry]
    end

    P --> S
    S --> W
    S --> G
    G --> Q
    Q --> V
    V --> R
    V --> K
    K --> O
    R --> O
    O --> C
`

---

8. Operational Model

Batching
- Daemon batches proofs for throughput  
- Ensures deterministic ordering  
- Reduces RPC load  

Monitoring
- Prometheus metrics  
- Structured logs  
- Health endpoints  

Failure Modes
- Proof invalid → reject  
- Risk too high → reject  
- Attestation expired → reject  
- RPC failure → retry queue  

---

9. Module‑Level Responsibilities

| Module | Responsibility |
|-------|----------------|
| sdk/ | Proof generation, signing, WASM bindings |
| daemon/verification | ZK verification |
| daemon/risk | Risk scoring |
| daemon/attestation | Attestation creation + signing |
| programs/compliance-registry | On‑chain state transitions |
| tests/e2e | Full pipeline validation |

---

10. Summary

Aurora is a deterministic, multi‑layer ZK trust framework with:

- Strong separation of concerns  
- Clear trust boundaries  
- A secure attestation pipeline  
- A compliance‑grade risk engine  
- Full Solana integration  
- WASM‑compatible SDK  

This architecture enables high‑throughput, privacy‑preserving identity verification across the GitDigital ecosystem.

---

