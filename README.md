# AcademicHub

A blockchain-based academic record verification system. A university admin issues a student's record on the Ethereum **Sepolia** testnet and stores the certificate on **IPFS** (via Pinata). Anyone, such as a recruiter or another institution, can then verify the record using just the Student ID.

## Why this exists

Paper and PDF certificates are easy to forge and slow to verify. AcademicHub makes the record tamper-proof (stored on-chain) and the certificate file permanent and content-addressed (stored on IPFS), so verification takes seconds and needs no middleman.

## Architecture

```
Admin / Recruiter (browser)
        |
        v
 Express API (Node.js)
   |            |
   |            +--> Pinata (IPFS)   certificate file -> CID
   |
   +--> ethers.js --> Sepolia smart contract   (name, major, GPA, certificate link)
```

| Layer | Technology |
|-------|------------|
| Backend | Node.js, Express |
| Blockchain | Solidity smart contract on Sepolia, ethers.js v6 |
| File storage | IPFS via Pinata |
| Upload handling | Multer (in-memory) |

## Smart contract

- Deployed on Sepolia at: `0xB37d42767c6e16B14AA82962A274fff941E8975d`
- Key functions:
  - `issueRecord(studentId, name, major, gpa, cid)` : admin-only write
  - `getStudentData(studentId)` : public read, returns name, major, GPA and certificate link
  - `universityAdmin()` : returns the admin address
- Event: `RecordUpdated(studentId, recordType)`

Contract source: see `contracts/` (add your `.sol` file here).

## API

### `GET /api/verify/:studentId`
Public. Reads the record from the blockchain.

```json
{
  "success": true,
  "studentId": "S1001",
  "name": "...",
  "major": "...",
  "gpa": "...",
  "certificateLink": "https://gateway.pinata.cloud/ipfs/<CID>",
  "ipfsLink": "ipfs://<CID>",
  "altGatewayLink": "https://ipfs.io/ipfs/<CID>"
}
```

### `POST /api/issue`
Admin only. `multipart/form-data` with fields: `studentId`, `name`, `major`, `gpa`, `adminKey`, and a `certificate` file. The server uploads the file to IPFS, then writes the record on-chain and returns the transaction hash and Etherscan link.

### `GET /api/health`
Returns `{ "status": "ok" }`.

## Getting started

**Prerequisites:** Node.js 18+, a Sepolia RPC URL (Infura or Alchemy), a Pinata account, and a Sepolia wallet with test ETH.

```bash
git clone <your-repo-url>
cd <repo-folder>
npm install
cp env.example .env     # then fill in your own values
npm start               # or: npm run dev
```

### Environment variables

| Variable | Description |
|----------|-------------|
| `SEPOLIA_RPC_URL` | Sepolia RPC endpoint (Infura or Alchemy) |
| `ADMIN_PRIVATE_KEY` | Private key of the contract admin wallet (use a Sepolia-only test wallet) |
| `PINATA_API_KEY` | Pinata API key |
| `PINATA_SECRET_KEY` | Pinata secret key |
| `ADMIN_PASSWORD` | Password required by `/api/issue` |
| `PORT` | Optional, defaults to `3001` |

> **Security:** never commit your real `.env`. It is listed in `.gitignore`.

## Screenshots

_Add a screenshot or demo GIF of the issue and verify flows here._

## Possible improvements

- Replace the shared admin password with wallet-signature authentication
- Add rate limiting and input validation
- Support revoking or updating records
- Add automated tests for the API routes

## Author

Built by **THG Future Data Labs**.
