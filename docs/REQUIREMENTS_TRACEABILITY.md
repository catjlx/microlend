# Requirement traceability (SC6113 brief → this repository)

This table maps the SC6113 brief to this repository and to the submitted documents: Technical Report v1.4, User Manual v1.2 and Team Contribution Report v1.1 (referred to below as "Technical Report" §n). Status is as of 14 October 2026.

## Mandatory functional requirements

| Requirement | Where it is implemented | Status |
|---|---|---|
| Role-based authentication, ≥ 3 roles | Wallet-signature login + JWT (`backend/src/auth.js`); roles Admin / Officer / Auditor read on-chain from `LoanManager`, plus Borrower / Lender for any wallet; role-gated routes (`requireRole`) and role-gated UI tabs (`frontend/src/App.jsx`); on-chain `AccessControl` | Done. Role matrix: `ARCHITECTURE.md` §7 |
| MetaMask integration and wallet transaction signing | `frontend/src/context/AppContext.jsx` (connect, network switch, `personal_sign`, every contract call through the signer, `runTx` status toasts) | Done |
| ≥ 3 interacting smart contracts | `CreditRegistry`, `LendingPool`, `CollateralVault`, `LoanManager` (+ `MockStablecoin`); `LoanManager` calls the other three | Done. Diagram: `ARCHITECTURE.md` §2 |
| ≥ 10 blockchain transaction types | 19 listed in `ARCHITECTURE.md` §8 | Done. Technical Report Table 15; the main lifecycle was executed end to end on Sepolia on 13 October 2026 and recorded in the demonstration video |
| Backend REST API | `backend/src/routes/*.js` (public, session, ledger, score, staff) | Done |
| Persistent data storage | SQLite (dev) / PostgreSQL (cloud) via knex, tables in `backend/src/db.js` | Done |
| Comprehensive exception handling | `backend/src/errors.js` (HTTP, validation, chain errors), contract custom errors mapped to messages in `frontend/src/lib/format.js` | Done |
| Responsive dashboard: history, event logs, visualisations | Overview, Lend, Audit pages with Recharts; event log with filters, paging, CSV export | Done. Phone-width (390 px) check and screenshots: Technical Report §8, Figure 9 |
| Gas estimation, optimisation analysis | `npm run test:gas` → `docs/gas-report.txt` (hardhat-gas-reporter) | Done. Table in `TESTING.md` §4; optimisation analysis in Technical Report §9.4 |
| Smart-contract testing | 49 tests in `test/` | Done. 49/49 contract and 16/16 backend tests pass; coverage 97.1% statements, 75.9% branches (Technical Report §9.1, `docs/evidence/03-coverage.txt`) |
| Public cloud deployment | `backend/Dockerfile`, `render.yaml`, `frontend/vercel.json`, `.github/workflows/ci.yml` | Done. Sepolia contracts and Render deployment (11 October 2026); URLs and addresses in `README.md` and Technical Report §10, Appendix B |

## Rubric (100 marks)

| Criterion (marks) | Evidence in repo | Covered in the submitted documents |
|---|---|---|
| Real-world problem analysis and validation (15) | `REPORT_GUIDE.md` §2 checklist; model card; simulation | Technical Report §1, §2, §9.7 (evidence table, seven-respondent survey, stakeholders) and §11.2 (ethics and regulation) |
| System architecture and technical design (10) | `ARCHITECTURE.md` (components, contracts, state machine, ER, sequence, deployment) | Technical Report §3 and its figures |
| Smart-contract design and security (15) | 4 contracts; `SECURITY.md`; Slither config | Technical Report §5, §6 (Slither before and after, triage); `docs/SLITHER_TRIAGE.md` |
| Frontend development and UX (10) | `frontend/` (5 role-based pages, responsive CSS, accessible labels, error messages) | Technical Report §8 (structured expert review; the moderated usability test is listed as not yet run) and the User Manual |
| Backend, API and database integration (10) | `backend/` (REST, indexer, DB, scoring service) | Technical Report §7; API summary in `README.md` §5 |
| Blockchain integration and functionality (10) | Event indexer, EIP-712 attestations, wallet flows | Technical Report §5 and Table 23 (Sepolia run of 13 October 2026, recorded in the demonstration video) |
| Testing, evaluation and impact measurement (10) | Contract + API + scoring tests; model evaluation (`ml/metrics.json`); portfolio simulation (`ml/portfolio_results.json`) | Technical Report §9 (results, coverage, gas, success indicators) |
| Cloud deployment and DevOps (5) | CI workflow, Dockerfile, Render blueprint | Technical Report §10, Appendix B Tables 27-28; CI evidence in Figures 11-12 |
| Innovation and practical feasibility (5) | Progressive credit (history bonus → tier up → collateral down), verifiable off-chain scoring (EIP-712), permissionless default handling, first-loss reserve analysis | Technical Report §4, §5, §12; limits in §11 and `SECURITY.md` §3 |
| Technical report (5) | `REPORT_GUIDE.md` outline | Technical Report v1.4 |
| Presentation and demonstration (5) | `scripts/seed.js` gives a ready demo dataset; demo script in `USER_MANUAL.md` §8 | Presentation and demonstration videos (links in `README.md`) |

## Deliverables

| Deliverable | Source |
|---|---|
| 10-15 page report | Technical Report v1.4 (outline in `REPORT_GUIDE.md`) |
| Architecture, contract interaction, data model, process flow, UI diagrams | `ARCHITECTURE.md` (Mermaid) + screenshots |
| GitHub repository with docs and contribution history | this repository; the commit and pull-request history is analysed in the Contribution Report |
| Public URL, contract deployment details, testing report | `README.md` (URLs, addresses), `deployments/sepolia.json`, `docs/evidence/11-deploy-sepolia.txt`; `TESTING.md`; independent MicroLend Test Report v1.0 (standalone document) |
| Slides and 10-minute demo | presentation and demonstration videos (links in `README.md`) |
| User manual and individual contribution report | User Manual v1.2 and Team Contribution Report v1.1; `docs/USER_MANUAL.md` is the shorter in-repository manual |
