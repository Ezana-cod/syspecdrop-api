# Postman API Testing Guide

## Base URL
```
http://localhost:3000
```

---

## 1. Upload Route - Create Drop

**Endpoint**: `POST /api/upload`

**Headers**:
```json
{
  "Content-Type": "application/json"
}
```

**Request Body**:
```json
{
  "file": "SGVsbG8gV29ybGQhIFRoaXMgaXMgYSBzZWNyZXQgZmlsZS4="
}
```

**Notes**:
- `file` is base64-encoded content
- Example decodes to: "Hello World! This is a secret file."
- To encode your own file in terminal: `base64 -i yourfile.txt`

**Expected Response** (201 Created):
```json
{
  "success": true,
  "dropId": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "ipfsHash": "QmXyz...",
  "passphrase": "quantum-secure-drop-anonymous-encrypted-blockdag",
  "txHash": "0x1234567890abcdef...",
  "explorerUrl": "https://explorer.bdagscan.com/tx/0x...",
  "message": "Drop created successfully. Share the passphrase securely."
}
```

**Save These Values**:
- ✅ `dropId` → Use in claim request
- ✅ `passphrase` → Use in claim request
- ✅ `txHash` → Check on BlockDAG explorer

---

## 2. Claim Route - Retrieve File

**Endpoint**: `POST /api/claim`

**Headers**:
```json
{
  "Content-Type": "application/json"
}
```

**Request Body**:
```json
{
  "dropId": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "passphrase": "quantum-secure-drop-anonymous-encrypted-blockdag"
}
```

**Notes**:
- Use the `dropId` and `passphrase` from upload response
- **V3: Can be claimed UNLIMITED times** (metadata is never burned)
- First claim sets `claimed=true` for status tracking
- Subsequent claims still work with correct passphrase

**Expected Response** (200 OK):
```json
{
  "success": true,
  "fileName": "drop-a1b2c3d4.bin",
  "fileContent": "SGVsbG8gV29ybGQhIFRoaXMgaXMgYSBzZWNyZXQgZmlsZS4=",
  "txHash": "0x1234567890abcdef...",
  "message": "Drop claimed successfully. File decrypted."
}
```

**Decode File**:
- `fileContent` is base64-encoded
- In terminal: `echo "SGVsbG8gV29ybGQhIFRoaXMgaXMgYSBzZWNyZXQgZmlsZS4=" | base64 -d`
- Should return: "Hello World! This is a secret file."

---

## 3. Status Route - Check Drop

**Endpoint**: `GET /api/status/:dropId`

**Headers**:
```json
{
  "Content-Type": "application/json"
}
```

**Example URL**:
```
http://localhost:3000/api/status/a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

**Expected Response** (200 OK):
```json
{
  "success": true,
  "dropId": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "exists": true,
  "claimed": false,
  "ipfsHash": "QmXyz...",
  "createdAt": 1729267200,
  "message": "Drop is active and unclaimed"
}
```

**After Claiming**:
```json
{
  "success": true,
  "dropId": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
  "exists": true,
  "claimed": true,
  "message": "Drop has been claimed and metadata burned"
}
```

---

## Testing Scenarios

### Scenario 1: Happy Path
1. **Upload** → Save `dropId` and `passphrase`
2. **Status** → Verify `exists: true`, `claimed: false`
3. **Claim** → Retrieve file with passphrase
4. **Status** → Verify `claimed: true`
5. **Claim Again** → Should fail (already claimed)

### Scenario 2: Invalid Passphrase
1. **Upload** → Save `dropId`
2. **Claim** with wrong passphrase → Should fail with 400 error

### Scenario 3: Non-existent Drop
1. **Status** with random dropId → Should return `exists: false`
2. **Claim** with random dropId → Should fail

---

## Sample Test Files (Base64 Encoded)

### Small Text File
```
SGVsbG8gV29ybGQhIFRoaXMgaXMgYSBzZWNyZXQgZmlsZS4=
```
Decodes to: "Hello World! This is a secret file."

### JSON Document
```
eyJzZWNyZXQiOiAid2hpc3RsZWJsb3dlcl9kb2N1bWVudCIsICJkYXRhIjogIkNvbmZpZGVudGlhbCBpbmZvcm1hdGlvbiJ9
```
Decodes to: `{"secret": "whistleblower_document", "data": "Confidential information"}`

### Larger Text (Lorem Ipsum)
```
TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQsIGNvbnNlY3RldHVyIGFkaXBpc2NpbmcgZWxpdC4gU2VkIGRvIGVpdXNtb2QgdGVtcG9yIGluY2lkaWR1bnQgdXQgbGFib3JlIGV0IGRvbG9yZSBtYWduYSBhbGlxdWEu
```

---

## Error Responses

### 400 Bad Request (Invalid Passphrase)
```json
{
  "success": false,
  "error": "Failed to claim drop",
  "code": "CLAIM_FAILED"
}
```

### 404 Not Found (Drop doesn't exist)
```json
{
  "success": false,
  "error": "Drop not found",
  "code": "DROP_NOT_FOUND"
}
```

### 500 Internal Server Error
```json
{
  "success": false,
  "error": "Internal server error",
  "code": "UPLOAD_ERROR"
}
```

---

## Environment Variables Required

Make sure these are set in `.env`:
```env
BLOCKDAG_RPC_URL=https://rpc.awakening.bdagscan.com
BLOCKDAG_CHAIN_ID=1043
BLOCKDAG_CONTRACT_ADDRESS=0x... # Your deployed contract address
BLOCKDAG_PRIVATE_KEY=0x... # Your wallet private key
IPFS_HOST=127.0.0.1
IPFS_PORT=5001
IPFS_PROTOCOL=http
```

---

## Quick Test Commands (cURL)

### Upload
```bash
curl -X POST http://localhost:3000/api/upload \
  -H "Content-Type: application/json" \
  -d '{"file":"SGVsbG8gV29ybGQhIFRoaXMgaXMgYSBzZWNyZXQgZmlsZS4="}'
```

### Claim
```bash
curl -X POST http://localhost:3000/api/claim \
  -H "Content-Type: application/json" \
  -d '{"dropId":"YOUR_DROP_ID","passphrase":"YOUR_PASSPHRASE"}'
```

### Status
```bash
curl http://localhost:3000/api/status/YOUR_DROP_ID
```
