### QuantumDrop Task Summary

**Overview**: Develop a decentralized, quantum-secure dead-drop platform for anonymous file transfers (inspired by Assange-era whistleblowing), using IPFS for storage and BlockDAG for on-chain metadata. Core twist: Senders upload encrypted files; receivers claim via passphrase without direct interaction, enforced by smart contracts with one-time access and key burn.

**Key Innovative Feature - ZKPs for Verifiable Anonymity**: Integrate lattice-based ZK-SNARKs (e.g., via circom library) for receivers to prove drop access/validity without revealing passphrase, sender details, or content. This enables "provable receipts" for journalists (e.g., confirming tip legitimacy to editors) while maintaining deniability—no on-chain links between parties. Use post-quantum ZKPs (hash/lattice-based) to resist quantum attacks, aligning with Ethereum's quantum-proofing research.

**Compliance & Cybersecurity Standards**:
- **NIST Inherent Compliance**: Employ PQC standards—FIPS 203 (ML-KEM/Kyber for key encapsulation), FIPS 204 (ML-DSA/Dilithium for signatures), FIPS 205 (SLH-DSA/SPHINCS+ for hash-based sigs)—for all encryption/ZKPs to ensure quantum resistance. Follow NIST SP 800-53 for security controls (e.g., access control, audit logging).
- **ISO 27001 Inherent Compliance**: Build ISMS-aligned features per ISO/IEC 27001:2022—systematic risk assessment, info security policies, and controls for confidentiality/integrity/availability (e.g., Annex A.10 cryptography via TLS 1.3/OpenSSL, A.12 logging/monitoring, A.13 secure comms). Include MFA, event logging, data anonymization/sharding, and audit-ready reports. Extend to ISO 27018 for PII protection in cloud/decentralized setups.

**Compact Implementation Steps**:
1. **Setup**: React frontend + ethers.js; Solidity contract on BlockDAG testnet (EVM-compatible).
2. **Encryption/Upload**: PQC-encrypt file (Kyber/AES hybrid), upload to IPFS, store hashed metadata on-chain.
3. **ZK Integration**: Generate/verify ZKPs for claims; burn keys post-verification.
4. **Security Controls**: Add MFA, logging, auto-expiry for unclaimed drops; test for ISO/NIST alignment (e.g., risk assessments via simulated audits).
5. **Demo/Polish**: End-to-end flow on testnet; document compliance mapping for hackathon pitch.

This ensures high cybersecurity (e.g., against quantum threats, data leaks) while winning on innovation via ZK-enabled anonymity.


QuantumDrop: Optimized End-to-End Anonymous Flow
1. Sender Prepares and Uploads File (Anonymous, Client-Side)

Action: Sender accesses /sender on React SPA.
Process:
Uploads file (e.g., whistleblower PDF) via drag-drop.
Inputs or auto-generates a strong passphrase (256-bit entropy, shown as QR code/copyable text).
Client-side JS encrypts file using NIST FIPS 203 ML-KEM (Kyber via @noble/kyber):
Generates Kyber public/private key pair (quantum-resistant).
Derives AES-256 symmetric key via Kyber encapsulation.
Encrypts file with AES-256.


Uploads encrypted file to IPFS (ipfs-http-client), receiving Content ID (CID).
Generates ZKP (snarkjs) proving file integrity without revealing content.
Computes passphraseHash (keccak256 of passphrase).
Calls backend API (/upload) to store CID|encryptedSymKey|ZKP on BlockDAG via Solidity contract.


Anonymity: No wallet required (API uses ephemeral keys); no sender metadata stored; Tor hides IP.
Compliance: NIST FIPS 203/204 for encryption; ISO 27001 A.10 for cryptographic controls.
Output: Sender shares passphrase off-chain (e.g., encrypted email, physical note).

2. Smart Contract Stores Metadata (BlockDAG Testnet)

Action: Backend API interacts with Solidity contract on BlockDAG’s Awakening Testnet.
Process:
Contract stores mapping: passphraseHash => CID|encryptedSymKey|ZKP.
Emits anonymous event (logged on BlockDAG explorer: https://awakening.bdagscan.com).
Enforces one-time access (deletes data post-claim).


Anonymity: No sender/receiver wallet linkage; public contract allows universal access.
Compliance: ISO 27001 A.12 for audit logging; NIST SP 800-53 AC-3 for access control.
Innovation: Solidity leverages BlockDAG’s EVM for robust, parallelized storage; ZKPs ensure verifiable anonymity.

3. Receiver Claims File (Anonymous, One-Time)

Action: Receiver visits /receiver on SPA, enters passphrase (via Tor/VPN).
Process:
Client computes passphraseHash (keccak256).
Calls API (/claim) to:
Generate ZKP (snarkjs) proving passphrase validity.
Query contract with passphraseHash, retrieve CID|encryptedSymKey|ZKP.
Verify ZKP on-chain (contract validates proof without data exposure).


Fetches encrypted file from IPFS using CID.
Decrypts file client-side (Kyber decapsulation for AES key, then AES-256 decrypt).
Contract burns data (deletes mapping) for one-time access.


Anonymity: No receiver wallet needed; ZKP hides passphrase. API uses ephemeral keys.
Compliance: NIST FIPS 205 for ZKP integrity; ISO 27001 A.13 for secure comms (TLS 1.3).

4. Verification and Audit (Optional, Deniable)

Action: Receiver visits /status to check drop status or generate audit proof.
Process:
Input passphraseHash or tx ID to view logs on BlockDAG explorer (no ID linkage).
Optional: Generate ZKP receipt proving claim (e.g., for journalists to show editors) without revealing file/passphrase.
Unclaimed drops expire after 7 days (contract-enforced).


Anonymity: ZKP receipt conceals sensitive data; explorer shows only hashed metadata.
Compliance: ISO 27001 A.12.4 for monitoring; NIST SP 800-53 AU-2 for auditable events.

5. Demo Flow (Hackathon Pitch)

Sender: Upload dummy “leak” (PDF), show passphrase QR, confirm tx on explorer.
Receiver: Input passphrase, download file, show burn tx (data gone).
Wow Factor: Display ZKP receipt proving claim without leaks; emphasize quantum-secure encryption and NIST/ISO compliance.
Pitch Line: “QuantumDrop: WikiLeaks reimagined—anonymous, quantum-proof dead drops for the decentralized era.”

Tech Stack

Frontend: React SPA (ethers.js, snarkjs, ipfs-http-client, @noble/kyber).
Backend: Node.js API (Express, Dockerized, ethers.js for BlockDAG).
Smart Contract: Solidity (BlockDAG testnet, deploy via Remix/IDE).
Storage: IPFS (pinned via Pinata).
Compliance: NIST FIPS 203/204/205, ISO 27001/27018 (A.10, A.12, A.13).

Why Best Approach

Anonymity: No accounts, IPs, or wallet links; ZKPs enable deniable receipts.
Quantum-Security: PQC (Kyber/Dilithium) protects against quantum attacks.
Solidity Advantage: Mature EVM tools (Remix, Hardhat) ensure reliability; BlockDAG’s DAG scales for high TPS.
Hackathon Appeal: ZKPs, PQC, and whistleblower UX solve real privacy needs with Web3 innovation.
