# MicroLend - Tokenized Micro-lending for the Unbanked

SC6113 group project. A full-stack DApp where lenders fund a transparent pool, borrowers without bank records build a reusable on-chain credit history, officers approve loans, and auditors can verify every event.

## Live demo and links

* **Live frontend:** https://microlend-web.onrender.com (Sepolia testnet; the free Render tier sleeps, so the first load can take about a minute)
* **Live API:** https://microlend-api.onrender.com ([health check](https://microlend-api.onrender.com/api/health))
* **Demo video:** https://www.youtube.com/watch?v=yrgKagY_duw
* **Presentation video:** https://youtu.be/S3S13By7L0E
* **Source repository:** [catjlx/microlend](https://github.com/catjlx/microlend)
* **Hosting note:** the Render services build from [Garywong007/microlend](https://github.com/Garywong007/microlend) (`main`, commit `1b5496f`), a mirror of this repository. The application source is identical to this repository's `main`; only documentation files differ.

### Sepolia deployment

Network: Ethereum Sepolia (chain id `11155111`). Deployed 2026-10-11 (block `11890702` onwards). Details are also in [`deployments/sepolia.json`](deployments/sepolia.json) and [`docs/evidence/11-deploy-sepolia.txt`](docs/evidence/11-deploy-sepolia.txt).

| Contract | Address (Etherscan) |
|---|---|
| MockStablecoin (mUSDC test token) | [`0x882b02C3eeaC59BC42C283c6C8d32BaA3723B2d1`](https://sepolia.etherscan.io/address/0x882b02C3eeaC59BC42C283c6C8d32BaA3723B2d1#code) |
| CreditRegistry | [`0xa80e874cA4C8C33b7c1887370005823f4cBe248F`](https://sepolia.etherscan.io/address/0xa80e874cA4C8C33b7c1887370005823f4cBe248F#code) |
| LendingPool | [`0x576358F66d8d521bcd02A472Ef87F903f1c4f410`](https://sepolia.etherscan.io/address/0x576358F66d8d521bcd02A472Ef87F903f1c4f410#code) |
| CollateralVault | [`0x9c7b61301E2573f0Ab52aA51669e496E1Cf10B31`](https://sepolia.etherscan.io/address/0x9c7b61301E2573f0Ab52aA51669e496E1Cf10B31#code) |
| LoanManager | [`0x9279A53E8E93cA960f682c3bAEd6cCC940aE0b69`](https://sepolia.etherscan.io/address/0x9279A53E8E93cA960f682c3bAEd6cCC940aE0b69#code) |

All five contracts were submitted for source verification on Etherscan during deployment (see the log above). Initial roles: the deployer `0xC0f1C1a59D70F3afD12776c86811664B08AeEC80` holds Admin, Credit officer and Auditor; the scoring service address is `0xb5767eFeD7b6C51C4DD9484f9c545B96a60EB3dD` (`SCORER_ROLE` on CreditRegistry). Roles can be granted to other wallets from the Admin tab.

### Deployment records (historical)

Two files are kept as dated records of the 11 October 2026 deployment and are cited as evidence in the Contribution Report. They describe the state on that date and are **not** the current project status.

* [`docs/TEAM_HANDOVER.md`](docs/TEAM_HANDOVER.md) (Chinese): deployment handover written on 11 October 2026. It lists the live addresses, service configuration, wallet roles and the planned test sequence at that time. Task statuses in it are superseded; for example, the full Sepolia loan lifecycle was executed on 13 October 2026.
* [`docs/evidence/12-cloud-smoke-check.json`](docs/evidence/12-cloud-smoke-check.json): read-only HTTP snapshot of the live frontend and API taken on 11 October 2026 (health, configuration, indexer lag). It contains no transactions and no credentials.

## Overview

* **Smart contracts** (Solidity 0.8.24, OpenZeppelin 5): `CreditRegistry`, `LendingPool`, `CollateralVault`, `LoanManager` (+ test token `MockStablecoin`)
* **Backend** (Node 20, Express, knex, SQLite/PostgreSQL, ethers v6): wallet-signature login, REST API, chain event indexer, off-chain credit scoring service that signs EIP-712 score attestations
* **Frontend** (React 18, Vite, ethers v6, Recharts, MetaMask): role-based dashboards for borrower, lender, credit officer, auditor and admin
* **Data science** (Python): logistic-regression default model, fairness checks, and a portfolio simulation

> **Status.** Compiled and tested on Ubuntu 22.04 (WSL2) with Node 20: 49/49 contract tests and 16/16 backend tests pass (see `docs/TESTING.md` for coverage and gas). Unaudited coursework code. The full loan lifecycle (score, application, approval, collateral, disbursement, repayment) was executed on Sepolia on 13 October 2026 (Loan #2) and is shown in the demo video.

Docs: [Architecture](docs/ARCHITECTURE.md) · [Security](docs/SECURITY.md) · [Testing](docs/TESTING.md) · [Requirement traceability](docs/REQUIREMENTS_TRACEABILITY.md) · [User manual](docs/USER_MANUAL.md) · [Report and slides guide](docs/REPORT_GUIDE.md) · [Submitted reports](docs/reports/) (Technical Report v1.4, User Manual v1.2, Contribution Report v1.1)

## 1. Prerequisites

* Node.js 20.11+ and npm
* Python 3.10+ in an **isolated environment** (only for `ml/` and Slither): `conda create -n microlend python=3.12 -y && conda activate microlend && pip install -r requirements-dev.txt`, or `python3 -m venv .venv && source .venv/bin/activate`. Never install into the system or conda `base` environment. Record the exact versions with `pip freeze > requirements.lock.txt`.
* MetaMask browser extension
* Optional: `slither-analyzer` (`pip install slither-analyzer`), Docker

## 2. Run everything locally (10 minutes)

```bash
npm run setup                 # root + backend + frontend dependencies
npm run build:contracts       # compile, writes shared/abis/*.json
npm test                      # 49 contract tests
npm run test:gas              # gas table -> docs/gas-report.txt
(cd backend && npm test)      # scoring + API tests

# --- start a local blockchain, deploy, add demo data ---
npm run node                  # terminal 1 (keep running)
npm run deploy:local          # terminal 2: deploys + grants roles -> deployments/localhost.json
npm run seed:local            # demo loans in every state, two lenders, an audit note
npm run local:env             # writes backend/.env and frontend/.env

# --- app ---
(cd backend && npm start)     # terminal 3: API on http://localhost:4000
(cd frontend && npm run dev)  # terminal 4: UI on http://localhost:5173
```

### MetaMask for the local chain

1. Add a network: RPC `http://127.0.0.1:8546`, chain id `31338`, symbol ETH (or just click **Switch network** in the app, which adds it). MicroLend avoids Hardhat's default port 8545 and chain id 31337 so it does not clash with other Hardhat projects in MetaMask.
2. Import accounts with the **public test keys** printed when `npm run node` starts (MicroLend uses its own mnemonic, `scripts/local-mnemonic.js`, so the addresses differ from other Hardhat projects; never use these keys on a real network).

| Account # | Role in the demo |
|---|---|
| #0 | Admin (deployer) |
| #1 | Credit officer |
| #2 | Auditor |
| #3 | Scoring service (used by the backend, not for login) |
| #4, #5 | Lenders A and B |
| #6 - #11 | Borrowers B1 - B6 (B1 repaid, B2 active, B3 pending, B4 approved, B5 defaulted, B6 rejected) |

Switch accounts in MetaMask and reload to see the different role tabs. A fresh wallet can use the **Borrow** and **Lend** tabs; use "Get 1,000 test mUSDC" first.

## 3. Deploy to Sepolia and the cloud

1. Create a wallet with Sepolia ETH (faucet), and an RPC key from Infura or Alchemy. Optionally an Etherscan API key.
2. `cp .env.example .env` and fill `SEPOLIA_RPC_URL`, `DEPLOYER_PRIVATE_KEY`, `ETHERSCAN_API_KEY`. Optional role addresses (`OFFICER_ADDRESS`, `AUDITOR_ADDRESS`, `SCORER_ADDRESS`) can be teammates' wallets.
3. `npm run deploy:sepolia` writes `deployments/sepolia.json` and `shared/abis/*.json`; commit both. Record the contract addresses and Etherscan links in the report.
4. **Backend on Render:** push to GitHub, create a *Blueprint* from `render.yaml` (API service built from `backend/Dockerfile`, PostgreSQL, static site). Set the secret variables: `RPC_URL`, `SCORER_PRIVATE_KEY` (a dedicated key that holds `SCORER_ROLE`), `CORS_ORIGIN` (the frontend URL), `NETWORK=sepolia`. `JWT_SECRET` is generated.
5. **Frontend:** the Blueprint also builds the static site. Alternatively import `frontend/` into Vercel (`frontend/vercel.json`) and set `VITE_API_URL` to the API URL.
6. Check `https://<api>/api/health` and open the site with MetaMask on Sepolia. The free Render tier sleeps, so open the URL a minute before the demo.
7. Put the public URLs into this README and the report.

## 4. Repository layout

```
contracts/      Solidity sources           test/       Hardhat tests (49)
scripts/        deploy, seed, ABI export   deployments/ addresses per network
backend/        API, indexer, scoring      frontend/   React dashboard
ml/             model + portfolio simulation, metrics, synthetic sample
docs/           architecture, security, testing, manual, report guide, images
.github/        CI (compile, test, gas, Slither, backend, frontend, model)
render.yaml     cloud blueprint            shared/abis ABIs shared by backend and frontend
```

## 5. REST API summary

| Method and path | Auth | Purpose |
|---|---|---|
| `GET /api/health` | - | liveness and chain connection |
| `GET /api/config`, `/api/abis` | - | contract addresses and ABIs for the frontend |
| `GET /api/stats`, `/api/pool/history` | - | pool KPIs and snapshot time series |
| `POST /api/auth/nonce`, `/api/auth/verify` | - | wallet-signature login, returns JWT and on-chain roles |
| `GET /api/me`, `PUT /api/profile` | JWT | wallet, roles, credit profile; optional profile data |
| `POST /api/score`, `GET /api/score/latest` | JWT | compute a score, get the signed EIP-712 attestation |
| `GET /api/loans`, `/api/loans/:id` | JWT | own loans (staff: all loans) |
| `GET /api/events`, `/api/events/export.csv` | JWT | event log (own; auditor/admin: all) with CSV export |
| `GET /api/officer/queue` | officer/admin | applications waiting for review with model inputs |
| `GET /api/audit/summary` | auditor/admin | approval/default rates, speed, volumes |

## 6. Limitations

Prototype for coursework: synthetic training data, mock stablecoin, testnet only, self-reported scoring inputs, no KYC/AML, admin keys not behind a multisig. See [docs/SECURITY.md](docs/SECURITY.md) section 3.

## License

MIT
