# TrustVault · Track 4: Trust, but Verify

 Yugaantar DevForge Track 4**  
> Post-Quantum Cryptography, Identity Lifecycle & Access Control

[![Tests](https://img.shields.io/badge/automated%20tests-161%2F161%20passing-10b981.svg)](#quick-start--verification)
[![Validator](https://img.shields.io/badge/toolkit%20validator-22%2F22%20PASS-10b981.svg)](#step-3-validate-with-official-toolkit)
[![PQC Standard](https://img.shields.io/badge/NIST%20FIPS-203%20ML--KEM%20%7C%20204%20ML--DSA-3b82f6.svg)](#core-capabilities)
[![Dependencies](https://img.shields.io/badge/dependencies-0%20runtime%20npm-blueviolet.svg)](#prerequisites)

---

## 📌 Project Overview

**TrustVault** is a zero-trust access control and security operations platform built for the post-quantum era. It protects sensitive assets, enforces real-time device revocations across distributed nodes, and logs audit events to an immutable, tamper-evident ledger.

### What Makes TrustVault Different?
* **Native Post-Quantum Cryptography**: Uses NIST-standardized **FIPS 203 ML-KEM-768** (key encapsulation) and **FIPS 204 ML-DSA-65** (digital signatures) built directly into the Node.js / OpenSSL standard library.
* **Zero Runtime Dependencies**: 100% pure standard library (`node:crypto`, `node:fs`, `node:path`). No fragile third-party npm packages.
* **Sub-20ms Device Revocation**: When a device is lost or compromised, access is revoked across all nodes in under **20 milliseconds**.
* **Explainable Detection (No Black-Box AI)**: Detects privilege escalation, clock manipulation, and lateral movement using deterministic graph analysis with zero hallucination.
* **Interactive Operations Dashboard**: Clean, responsive UI with live demonstration walkthroughs, node topology status, and visual hash-chain verification.

---

## 🚀 Quick Start & Verification

### Prerequisites
* **Node.js**: [v24.11.1](https://nodejs.org/) (native OpenSSL 3.5.4 with PQC support)
* **Python**: 3.12+ (only used to run the official hackathon toolkit validator)

---

### Step 1: Verify Node Environment
Make sure Node.js v24.11.1 is active in your terminal:
```bash
node -v
# Output should be: v24.11.1
```

### Step 2: Clone the Project
Clone the repository alongside the official toolkit:
```bash
git clone https://github.com/veryVanshika801/trust-vault-track4.git
cd trust-vault-track4

# Clone the official evaluation toolkit into the parent directory
git clone https://github.com/ScalerOpenSourceLabs/Track-4-Toolkit.git ../Track-4-Toolkit
```

### Step 3: Validate with Official Toolkit
Run the official challenge validator against the service transport:
```bash
python3 ../Track-4-Toolkit/validate.py --entry "$PWD/track4_service.cjs"
```
**Expected Output:** `PASS 22/22 (Local scenario passed.)`

### Step 4: Run the Complete 161-Test Suite
Run the full automated test suite covering all 12 core challenge objectives:
```bash
node --test --test-concurrency=1 tests/*.cjs
```
**Expected Output:**
```text
ℹ tests 161
ℹ suites 0
ℹ pass 161
ℹ fail 0
```

### Step 5: Launch the Security Dashboard
Start the local operations dashboard:
```bash
node bin/dashboard.cjs
```
Open **[http://127.0.0.1:3400](http://127.0.0.1:3400)** in your browser to view the interactive demo, manage users, and inspect real-time audit decisions.

---

## 🏗️ Architecture & Flow

```text
       [ User / API Request (JSON over stdin) ]
                          │
                          ▼
              [ track4_service.cjs ]
            (Fail-Closed CLI Transport)
                          │
                          ▼
                 [ team_app.cjs ]
    ┌─────────────────────┼─────────────────────┐
    ▼                     ▼                     ▼
[ Authorization Guard ] [ Envelope Encryption ] [ PQC ML-KEM-768 ]
 • Bounded Maps/Sets     • AES-256-GCM           • 1184-byte Public Key
 • Prototype-safe        • Unique per-asset DEK  • Shared-secret Proof
    │                     │                     │
    ▼                     ▼                     ▼
[ Device Lifecycle ]    [ Atomic Persistence ] [ PQC Messaging ]
 • ML-DSA-65 Signatures  • Crash-safe fsync      • Ephemeral Prekeys
 • Sub-20ms Revocation   • Atomic lock files     • 1,024 Replay Window
                          │
                          ▼
              [ $TRACK4_STATE_DIR ]
         ├── kek.bin           (0600, key encryption key)
         ├── store.enc         (0600, encrypted assets)
         ├── audit.log         (0600, SHA-256 hash-chained log)
         └── audit_index.json  (0600, fast O(1) audit index)
```

---

## 🛡️ 12 Core Challenge Capabilities

| # | Challenge Objective | How TrustVault Solves It | Status |
|:---:|:---|:---|:---:|
| **1** | **Key Rotation** | Monotonic lifecycle (`Active` $\rightarrow$ `Retiring` $\rightarrow$ `Retired`). Enforces a clean 60s cutoff across all cluster nodes. | **100% PASS** |
| **2** | **Device Revocation** | Revokes lost devices across 5 nodes in **20 ms**. Maya re-enrols a new phone seamlessly while Alex remains unaffected. | **100% PASS** |
| **3** | **Crash Recovery** | Write-Ahead Journaling (WAL) and atomic `fsync` prevent half-written or corrupted state if a process crashes mid-operation. | **100% PASS** |
| **4** | **Backup Integrity** | Backups use Scrypt KDF + AES-256-GCM and ML-DSA-65 digital signatures. Restoring an old backup preserves existing revocations. | **100% PASS** |
| **5** | **Tamper-Evident Logs** | Canonical SHA-256 hash chains with periodic digital signatures. Any bit-flip, deleted line, or swap is immediately detected. | **100% PASS** |
| **6** | **Routine vs Suspicious** | Deterministic rules flag unapproved role changes and anomalous reads without any machine learning hallucinations. | **100% PASS** |
| **7** | **Lateral Movement** | Graph correlation tracks multi-hop traversals across unexpected services while ignoring benign isolated denials. | **100% PASS** |
| **8** | **Clock Synchronization** | Normalizes time offsets ($\pm 5,000\text{ ms}$ threshold) and flags ambiguous event orderings within a $\pm 500\text{ ms}$ uncertainty window. | **100% PASS** |
| **9** | **Replay Protection** | Persistent 1,024-sequence sliding window per sender. Duplicate messages, sequence reuse, and restarts are safely dropped. | **100% PASS** |
| **10** | **Forward Secrecy** | Per-session ephemeral ML-KEM-768 prekeys destroyed in `finally` blocks; past messages cannot be cracked if identity keys leak later. | **100% PASS** |
| **11** | **Message Chunking** | Splits large messages into chunks signed with ML-DSA-65. Detects tampered chunks (`CHUNK_AUTH_FAILED`) and missing frames. | **100% PASS** |
| **12** | **Session Resumption** | Resumption tickets bind peer identities using timing-safe comparisons (`crypto.timingSafeEqual`) and strict TTL bounds. | **100% PASS** |

---

## 📊 Measured Benchmarks & Metrics

| Metric | Target | Measured Result | Evidence Source |
|:---|:---:|:---:|:---|
| **Revocation Propagation** | $< 100\text{ ms}$ | **20 ms** | 5-node cluster simulation with partition catch-up |
| **Grace Period Cutoff** | Configurable | **60 s** (500 ms test) | In-flight requests succeed; post-cutoff rejected |
| **Clock Skew Tolerance** | Configurable | **$\pm 5,000\text{ ms}$** | Events within $\pm 500\text{ ms}$ marked as uncertain order |
| **Detector F1 Score** | $100\%$ | **100.0%** | Evaluated on official practice set (22 events) |
| **Median Time to Alert** | $< 200\text{ ms}$ | **80 ms** | Synthetic attack benchmark suite |
| **False Positive Rate** | $0$ | **0 / 1,000** | 1,000 synthetic routine benign events |
| **In-Memory Crypto Speed** | $< 10\text{ ms}$ | **< 2 ms** | Steady-state native ML-KEM and AES-GCM operations |

---

## 🔒 Security Hardening

* **Log Injection Defense (CWE-117)**: Automatically strips carriage returns (`\r`), newlines (`\n`), and tabs (`\t`) from user input before writing to audit logs.
* **Prototype Pollution Safe**: Uses native `Map` and `Set` collections; user-supplied keys like `__proto__` cannot alter JavaScript prototypes.
* **Path Traversal Immunity**: Filenames are strictly fixed or SHA-256 hashed; user IDs never dictate disk file paths.
* **Early Denial**: Non-owners and revoked devices are rejected immediately before running post-quantum KEM operations, preventing timing attacks.
* **Safe Alert-Only Default**: Anomaly detection defaults to alert-only logging to prevent denial-of-service against legitimate users; lockout is opt-in (`TRACK4_LIVE_LOCKOUT=1`).

---

## 📂 Repository Layout

```text
.
├── bin/
│   ├── dashboard.cjs       # Security Operations Dashboard server (port 3400)
│   ├── detect.cjs          # Standalone explainable anomaly detector CLI
│   ├── verify_log.cjs      # Audit log verification & tamper-check CLI
│   └── track4_admin.cjs    # Administrative key rotation utility
├── dashboard/
│   └── index.html          # TrustVault visual dashboard interface
├── lib/
│   ├── key_lifecycle.cjs   # Key rotation & write-ahead journal
│   ├── distributed_revocation.cjs # 5-node cluster revocation & device enrolment
│   ├── backup_integrity.cjs# Scrypt + ML-DSA-65 encrypted backup manager
│   ├── tamper_log.cjs      # SHA-256 hash-chained log with signed checkpoints
│   ├── detector.cjs        # Explainable lateral movement & clock-skew engine
│   └── messaging.cjs       # PQC protected messaging, replay & forward secrecy
├── tests/
│   ├── core_objectives_12.test.cjs # 58-test comprehensive verification suite
│   └── *.test.cjs          # Unit, fuzzing, concurrency, and boundary tests
├── team_app.cjs            # Application dispatcher & authorization engine
└── track4_service.cjs      # CLI transport adapter (official test entrypoint)
```

---

## 👥 Team & License

* **Project**: TrustVault
* **Track**: Track 4 – Post-Quantum Cryptography, Identity & Access Control
* **Team**: Ctrl+Alt+Delulu
* **License**: MIT
