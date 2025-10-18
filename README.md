# QuantumDrop API - Setup Guide

## 🎯 Overview
Backend API for QuantumDrop - a quantum-secure, anonymous dead-drop platform using BlockDAG and IPFS.

## 📋 Prerequisites
- Node.js v18+
- BlockDAG Awakening Testnet RPC access
- IPFS node or Infura/Pinata account
- Deployed QuantumDrop smart contract

## 🚀 Quick Start

### 1. Install Dependencies
```bash
npm install
```

### 2. Configure Environment
```bash
cp .env.example .env
```

Edit `.env` with your configurations:
- **BLOCKDAG_RPC_URL**: BlockDAG testnet RPC endpoint
- **BLOCKDAG_PRIVATE_KEY**: Ephemeral wallet for anonymous transactions
- **QUANTUM_DROP_CONTRACT_ADDRESS**: Deployed contract address
- **IPFS_HOST**: IPFS node (Infura: `ipfs.infura.io`)
- **PINATA_API_KEY/SECRET**: (Optional) For IPFS pinning

### 3. Deploy Smart Contract
See `contracts/README.md` for deployment instructions.

### 4. Run Development Server
```bash
npm run dev
```

Server runs on `http://localhost:3000`

## 🛣️ API Routes

### POST `/api/upload`
**Purpose**: Store encrypted file metadata on BlockDAG (sender flow)

**Request Body**:
```json
{
  "cid": "QmXxx...xxx",
  "encryptedSymKey": "base64_encoded_key",
  "passphraseHash": "64_char_keccak256_hash",
  "zkProof": {
    "pi_a": ["...", "...", "..."],
    "pi_b": [["...", "..."], ["...", "..."], ["...", "..."]],
    "pi_c": ["...", "...", "..."],
    "protocol": "groth16",
    "curve": "bn128"
  },
  "expiryDays": 7
}
```

**Response** (201 Created):
```json
{
  "success": true,
  "dropId": "0xabc123...",
  "txHash": "0xdef456...",
  "explorerUrl": "https://awakening.bdagscan.com/tx/0xdef456...",
  "expiresAt": 1729296000,
  "message": "Drop stored successfully. Share passphrase securely."
}
```

**Compliance**:
- NIST SP 800-53 SI-10 (Input validation)
- ISO 27001 A.10 (Cryptography), A.12.4 (Logging)

### POST `/api/claim`
**Purpose**: Claim file and burn metadata (receiver flow)

**Request Body**:
```json
{
  "passphraseHash": "64_char_keccak256_hash",
  "zkProof": {
    "pi_a": ["...", "...", "..."],
    "pi_b": [["...", "..."], ["...", "..."], ["...", "..."]],
    "pi_c": ["...", "...", "..."],
    "protocol": "groth16",
    "curve": "bn128"
  }
}
```

**Response** (200 OK):
```json
{
  "success": true,
  "cid": "QmXxx...xxx",
  "encryptedSymKey": "base64_encoded_key",
  "txHash": "0xburn123...",
  "message": "Drop claimed successfully. Data burned."
}
```

### GET `/api/status/:passphraseHash`
**Purpose**: Check drop status (optional verification)

**Response** (200 OK):
```json
{
  "success": true,
  "exists": true,
  "claimed": false,
  "expiresAt": 1729296000,
  "createdAt": 1728691200
}
```

## 🏗️ Project Structure
```
src/
├── routes/
│   ├── upload.route.ts      # Upload endpoint
│   ├── claim.route.ts       # Claim endpoint
│   └── status.route.ts      # Status check
├── services/
│   ├── blockdag.service.ts  # BlockDAG interactions
│   ├── ipfs.service.ts      # IPFS operations
│   └── zkp.service.ts       # ZK proof validation
├── middlewares/
│   └── security.middleware.ts # Security headers, logging
└── types/
    ├── upload.types.ts      # Upload types
    ├── claim.types.ts       # Claim types
    └── common.types.ts      # Shared types
```

## 🔒 Security Features
- **Anonymity**: No wallet/IP logging, ephemeral keys
- **Quantum-Resistance**: NIST FIPS 203/204/205 (client-side)
- **One-Time Access**: Data burned post-claim
- **ZK Proofs**: Verifiable anonymity
- **TLS 1.3**: Secure communications (ISO 27001 A.13)

## 🧪 Testing
```bash
# Run tests
npm test

# Test upload endpoint
curl -X POST http://localhost:3000/api/upload \
  -H "Content-Type: application/json" \
  -d @test/fixtures/upload.json
```

## 📝 Compliance Standards
- **NIST SP 800-53**: Access control, audit logging, input validation
- **ISO 27001**: A.10 (Cryptography), A.12 (Logging), A.13 (Secure comms)
- **ISO 27018**: PII protection, data anonymization

## 🌐 BlockDAG Testnet
- **Explorer**: https://awakening.bdagscan.com
- **RPC**: Contact BlockDAG team for testnet access
- **Faucet**: Get testnet tokens for deployment

## 📚 Next Steps
1. Deploy smart contract (see `contracts/QuantumDrop.sol`)
2. Update `QUANTUM_DROP_CONTRACT_ADDRESS` in `.env`
3. Test upload/claim flow
4. Integrate with React frontend
5. Deploy to production (Docker/K8s)

## 🤝 Support
For issues or questions, refer to `taskguide.md` or BlockDAG documentation.
